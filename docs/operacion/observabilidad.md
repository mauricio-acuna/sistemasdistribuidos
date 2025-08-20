---
title: "Observabilidad en Sistemas Distribuidos"
description: "OpenTelemetry, métricas, trazas y dashboards para sistemas de producción complejos"
nav_order: 40
tags: [observability, metrics, tracing, monitoring]
---

# Observabilidad en Sistemas Distribuidos

!!! abstract "Los Tres Pilares"
    
    - **📊 Métricas:** ¿Qué está pasando? (SLIs, alertas)
    - **🔍 Trazas:** ¿Por qué está pasando? (Request flow, latencia)
    - **📝 Logs:** ¿Cómo está pasando? (Contexto, debugging)

En sistemas distribuidos complejos, la observabilidad no es opcional: es la diferencia entre **detectar problemas en segundos** vs **descubrirlos por quejas de usuarios**.

## Arquitectura de Observabilidad

`mermaid
graph TB
    subgraph "Application Layer"
        APP1[Service A]
        APP2[Service B] 
        APP3[Service C]
    end
    
    subgraph "Instrumentation"
        OTEL[OpenTelemetry SDK]
        PROM[Prometheus Client]
        LOG[Structured Logging]
    end
    
    subgraph "Collection & Processing"
        JAEGER[Jaeger<br/>Distributed Tracing]
        PROMETHEUS[Prometheus<br/>Metrics Collection]
        LOKI[Loki<br/>Log Aggregation]
    end
    
    subgraph "Visualization & Alerting"
        GRAFANA[Grafana Dashboards]
        ALERT[AlertManager]
        ONCALL[On-call Systems]
    end
    
    APP1 --> OTEL
    APP2 --> OTEL  
    APP3 --> OTEL
    
    APP1 --> PROM
    APP2 --> PROM
    APP3 --> PROM
    
    APP1 --> LOG
    APP2 --> LOG
    APP3 --> LOG
    
    OTEL --> JAEGER
    PROM --> PROMETHEUS
    LOG --> LOKI
    
    JAEGER --> GRAFANA
    PROMETHEUS --> GRAFANA
    LOKI --> GRAFANA
    
    PROMETHEUS --> ALERT
    ALERT --> ONCALL
    
    style OTEL fill:#e3f2fd
    style PROMETHEUS fill:#f3e5f5
    style GRAFANA fill:#e8f5e8
`

## Métricas (SLIs) Fundamentales

### Golden Signals

Los **4 indicadores** que todo sistema distribuido debe monitorear:

=== "Latencia"

    **Tiempo que toma servir una request**
    
    `java
    @Component
    public class LatencyMetrics {
        
        private final Timer requestLatency = Timer.build()
            .name("http_request_duration_seconds")
            .help("HTTP request latency")
            .labelNames("method", "endpoint", "status")
            .quantile(0.5, 0.05)    // p50 ±5%
            .quantile(0.95, 0.01)   // p95 ±1%  
            .quantile(0.99, 0.001)  // p99 ±0.1%
            .register();
            
        public void recordLatency(String method, String endpoint, 
                                  int statusCode, Duration latency) {
            requestLatency
                .labels(method, endpoint, String.valueOf(statusCode))
                .observe(latency.toNanos() / 1_000_000_000.0);
        }
    }
    
    // Usage in interceptor
    @RestControllerAdvice  
    public class LatencyInterceptor {
        
        @Around("@annotation(Timed)")
        public Object measureLatency(ProceedingJoinPoint joinPoint) throws Throwable {
            Timer.Sample sample = Timer.start();
            
            try {
                Object result = joinPoint.proceed();
                
                latencyMetrics.recordLatency(
                    getHttpMethod(),
                    getEndpoint(joinPoint), 
                    200,
                    sample.stop()
                );
                
                return result;
                
            } catch (Exception e) {
                latencyMetrics.recordLatency(
                    getHttpMethod(),
                    getEndpoint(joinPoint),
                    getErrorCode(e),
                    sample.stop()
                );
                
                throw e;
            }
        }
    }
    `

=== "Throughput"

    **Requests por segundo que maneja el sistema**
    
    `java
    @Component
    public class ThroughputMetrics {
        
        private final Counter requestCount = Counter.build()
            .name("http_requests_total")
            .help("Total HTTP requests")
            .labelNames("method", "endpoint", "status")
            .register();
            
        private final Gauge activeConnections = Gauge.build()
            .name("http_active_connections")
            .help("Currently active HTTP connections")
            .register();
            
        public void incrementRequests(String method, String endpoint, int status) {
            requestCount.labels(method, endpoint, String.valueOf(status)).inc();
        }
        
        public void setActiveConnections(double count) {
            activeConnections.set(count);
        }
    }
    
    // Prometheus queries for alerting:
    // Rate: rate(http_requests_total[1m])
    // Error rate: rate(http_requests_total{status=~"5.."}[1m]) / rate(http_requests_total[1m])
    `

=== "Errores"

    **Rate de requests que fallan**
    
    `java
    @Component
    public class ErrorMetrics {
        
        private final Counter errorCount = Counter.build()
            .name("application_errors_total")
            .help("Application errors by type")
            .labelNames("error_type", "service", "method")
            .register();
            
        private final Histogram errorLatency = Histogram.build()
            .name("error_handling_duration_seconds")
            .help("Time spent handling errors")
            .labelNames("error_type")
            .register();
            
        public void recordError(String errorType, String service, String method) {
            errorCount.labels(errorType, service, method).inc();
        }
        
        public Timer.Sample startErrorHandling(String errorType) {
            return Timer.start(errorLatency.labels(errorType));
        }
    }
    
    @ControllerAdvice
    public class GlobalExceptionHandler {
        
        @ExceptionHandler(BusinessException.class)
        public ResponseEntity<?> handleBusinessException(BusinessException e) {
            errorMetrics.recordError("business", "user-service", "validateUser");
            
            return ResponseEntity.badRequest()
                .body(new ErrorResponse(e.getMessage()));
        }
        
        @ExceptionHandler(TimeoutException.class)
        public ResponseEntity<?> handleTimeout(TimeoutException e) {
            errorMetrics.recordError("timeout", "user-service", "downstream");
            
            return ResponseEntity.status(HttpStatus.GATEWAY_TIMEOUT)
                .body(new ErrorResponse("Service temporarily unavailable"));
        }
    }
    `

=== "Saturación"

    **Qué tan "lleno" está el sistema**
    
    `java
    @Component
    public class SaturationMetrics {
        
        private final Gauge cpuUsage = Gauge.build()
            .name("system_cpu_usage_percent")
            .help("CPU usage percentage")
            .register();
            
        private final Gauge memoryUsage = Gauge.build()
            .name("jvm_memory_usage_percent") 
            .help("JVM memory usage percentage")
            .labelNames("memory_type")
            .register();
            
        private final Gauge threadPoolUtilization = Gauge.build()
            .name("thread_pool_utilization_percent")
            .help("Thread pool utilization")
            .labelNames("pool_name")
            .register();
            
        private final Gauge queueDepth = Gauge.build()
            .name("queue_depth_percent")
            .help("Queue depth as percentage of capacity")
            .labelNames("queue_name")
            .register();
            
        @Scheduled(fixedRate = 5000) // Every 5 seconds
        public void collectSystemMetrics() {
            // CPU usage
            double cpu = getSystemCpuUsage();
            cpuUsage.set(cpu);
            
            // Memory usage
            MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
            MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
            
            double heapUtilization = (double) heapUsage.getUsed() / heapUsage.getMax() * 100;
            memoryUsage.labels("heap").set(heapUtilization);
            
            // Thread pools
            threadPoolExecutors.forEach((name, executor) -> {
                double utilization = (double) executor.getActiveCount() / executor.getMaximumPoolSize() * 100;
                threadPoolUtilization.labels(name).set(utilization);
            });
        }
    }
    `

### Métricas de Consenso y Concurrencia

`java
@Component  
public class DistributedSystemMetrics {
    
    // Raft consensus metrics
    private final Counter leaderElections = Counter.build()
        .name("raft_leader_elections_total")
        .help("Total leader elections")
        .register();
        
    private final Gauge leaderStability = Gauge.build()
        .name("raft_leader_stability_seconds")
        .help("Time since last leader change")
        .register();
        
    private final Histogram replicationLatency = Histogram.build()
        .name("raft_replication_latency_seconds")
        .help("Log replication latency to followers")
        .register();
        
    private final Gauge logSize = Gauge.build()
        .name("raft_log_entries")
        .help("Number of entries in Raft log")
        .register();
        
    // Concurrency metrics
    private final Counter lockContentions = Counter.build()
        .name("lock_contentions_total")
        .help("Lock contention events")
        .labelNames("lock_name")
        .register();
        
    private final Counter casFailures = Counter.build()
        .name("cas_failures_total")
        .help("Failed compare-and-swap operations")
        .labelNames("operation")
        .register();
        
    private final Histogram gcPauseTimes = Histogram.build()
        .name("gc_pause_duration_seconds")
        .help("Garbage collection pause times")
        .labelNames("gc_type")
        .buckets(0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1.0)
        .register();
        
    // Business metrics  
    private final Counter consensusOperations = Counter.build()
        .name("consensus_operations_total")
        .help("Operations requiring consensus")
        .labelNames("operation_type", "status")
        .register();
        
    public void recordLeaderElection() {
        leaderElections.inc();
        leaderStability.setToCurrentTime();
    }
    
    public void recordReplication(Duration latency) {
        replicationLatency.observe(latency.toNanos() / 1_000_000_000.0);
    }
    
    public void recordLockContention(String lockName) {
        lockContentions.labels(lockName).inc();
    }
    
    public void recordCasFailure(String operation) {
        casFailures.labels(operation).inc();
    }
    
    public void recordConsensusOperation(String operationType, String status) {
        consensusOperations.labels(operationType, status).inc();
    }
}
`

## Distributed Tracing

### OpenTelemetry Integration

`java
@Configuration
@EnableAutoConfiguration
public class ObservabilityConfig {
    
    @Bean
    public OpenTelemetry openTelemetry() {
        return OpenTelemetrySDK.builder()
            .setTracerProvider(
                SdkTracerProvider.builder()
                    .addSpanProcessor(BatchSpanProcessor.builder(
                        JaegerGrpcSpanExporter.builder()
                            .setEndpoint("http://jaeger:14250")
                            .build())
                        .setMaxExportBatchSize(512)
                        .setExportTimeout(Duration.ofSeconds(2))
                        .setScheduleDelay(Duration.ofSeconds(1))
                        .build())
                    .setResource(Resource.getDefault()
                        .merge(Resource.builder()
                            .put(ResourceAttributes.SERVICE_NAME, "odin-consensus")
                            .put(ResourceAttributes.SERVICE_VERSION, "1.0.0")
                            .build()))
                    .build())
            .buildAndRegisterGlobal();
    }
    
    @Bean
    public Tracer tracer(OpenTelemetry openTelemetry) {
        return openTelemetry.getTracer("odin-consensus");
    }
}
`

### Instrumentación de Consenso

`java
@Component
public class TracedRaftConsensus {
    
    private final Tracer tracer;
    private final RaftConsensusEngine consensus;
    
    public CompletableFuture<ConsensusResult> propose(ConsensusRequest request) {
        Span span = tracer.spanBuilder("consensus.propose")
            .setSpanKind(SpanKind.INTERNAL)
            .setAttribute("consensus.request_id", request.getId())
            .setAttribute("consensus.operation_type", request.getType())
            .setAttribute("consensus.payload_size", request.getPayloadSize())
            .startSpan();
            
        try (Scope scope = span.makeCurrent()) {
            
            return consensus.propose(request)
                .whenComplete((result, throwable) -> {
                    if (throwable != null) {
                        span.recordException(throwable);
                        span.setStatus(StatusCode.ERROR, throwable.getMessage());
                    } else {
                        span.setAttribute("consensus.term", result.getTerm());
                        span.setAttribute("consensus.index", result.getIndex());
                        span.setAttribute("consensus.leader_id", result.getLeaderId());
                        span.setStatus(StatusCode.OK);
                    }
                    
                    span.end();
                });
                
        } catch (Exception e) {
            span.recordException(e);
            span.setStatus(StatusCode.ERROR, e.getMessage());
            span.end();
            throw e;
        }
    }
    
    public void replicateToFollower(String followerId, LogEntry entry) {
        Span span = tracer.spanBuilder("consensus.replicate")
            .setSpanKind(SpanKind.CLIENT)
            .setAttribute("consensus.follower_id", followerId)
            .setAttribute("consensus.entry_index", entry.getIndex())
            .setAttribute("consensus.entry_term", entry.getTerm())
            .startSpan();
            
        try (Scope scope = span.makeCurrent()) {
            
            // Add trace context to replication RPC
            RpcContext.current().setTraceContext(span.getSpanContext());
            
            replicationClient.sendAppendEntries(followerId, entry);
            
            span.setAttribute("consensus.replication_successful", true);
            span.setStatus(StatusCode.OK);
            
        } catch (Exception e) {
            span.recordException(e);
            span.setAttribute("consensus.replication_successful", false);
            span.setStatus(StatusCode.ERROR, e.getMessage());
            throw e;
        } finally {
            span.end();
        }
    }
}
`

### Correlación de Traces

`java
@RestController
public class ConsensusApiController {
    
    private final TracedRaftConsensus consensus;
    private final Tracer tracer;
    
    @PostMapping("/consensus/propose")
    public CompletableFuture<ResponseEntity<ConsensusResponse>> propose(
            @RequestBody ConsensusRequest request,
            HttpServletRequest httpRequest) {
            
        // Extract trace context from HTTP headers
        Context extractedContext = openTelemetry.getPropagators()
            .getTextMapPropagator()
            .extract(Context.current(), httpRequest, HttpTextMapGetter.INSTANCE);
            
        Span span = tracer.spanBuilder("api.consensus.propose")
            .setParent(extractedContext)
            .setSpanKind(SpanKind.SERVER)
            .setAttribute("http.method", "POST")
            .setAttribute("http.url", "/consensus/propose")
            .setAttribute("user.id", getCurrentUserId())
            .startSpan();
            
        try (Scope scope = span.makeCurrent()) {
            
            return consensus.propose(request)
                .thenApply(result -> {
                    span.setAttribute("consensus.success", true);
                    span.setStatus(StatusCode.OK);
                    
                    return ResponseEntity.ok(
                        new ConsensusResponse(result)
                    );
                })
                .exceptionally(throwable -> {
                    span.recordException(throwable);
                    span.setStatus(StatusCode.ERROR, throwable.getMessage());
                    
                    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                        .body(new ConsensusResponse(throwable.getMessage()));
                })
                .whenComplete((response, throwable) -> span.end());
                
        } catch (Exception e) {
            span.recordException(e);
            span.setStatus(StatusCode.ERROR, e.getMessage());
            span.end();
            throw e;
        }
    }
}
`

## Structured Logging

### Contextual Logging

`java
@Component
public class StructuredLogger {
    
    private static final Logger logger = LoggerFactory.getLogger(StructuredLogger.class);
    private static final ObjectMapper objectMapper = new ObjectMapper();
    
    public void logConsensusEvent(String event, Object... keyValues) {
        try {
            Map<String, Object> logEntry = new HashMap<>();
            logEntry.put("timestamp", Instant.now().toString());
            logEntry.put("service", "odin-consensus");
            logEntry.put("event", event);
            
            // Add trace context
            Span currentSpan = Span.current();
            if (currentSpan.getSpanContext().isValid()) {
                logEntry.put("trace_id", currentSpan.getSpanContext().getTraceId());
                logEntry.put("span_id", currentSpan.getSpanContext().getSpanId());
            }
            
            // Add key-value pairs
            for (int i = 0; i < keyValues.length; i += 2) {
                if (i + 1 < keyValues.length) {
                    logEntry.put(keyValues[i].toString(), keyValues[i + 1]);
                }
            }
            
            logger.info(objectMapper.writeValueAsString(logEntry));
            
        } catch (Exception e) {
            // Fallback to simple logging if JSON serialization fails
            logger.error("Failed to create structured log entry", e);
            logger.info("event={} data={}", event, Arrays.toString(keyValues));
        }
    }
    
    public void logError(String message, Throwable throwable, Object... keyValues) {
        Map<String, Object> errorEntry = new HashMap<>();
        errorEntry.put("timestamp", Instant.now().toString());
        errorEntry.put("level", "ERROR");
        errorEntry.put("message", message);
        errorEntry.put("exception_type", throwable.getClass().getSimpleName());
        errorEntry.put("exception_message", throwable.getMessage());
        errorEntry.put("stack_trace", getStackTrace(throwable));
        
        // Add context
        for (int i = 0; i < keyValues.length; i += 2) {
            if (i + 1 < keyValues.length) {
                errorEntry.put(keyValues[i].toString(), keyValues[i + 1]);
            }
        }
        
        try {
            logger.error(objectMapper.writeValueAsString(errorEntry));
        } catch (Exception e) {
            logger.error("Structured logging failed: {} Original error: {}", 
                e.getMessage(), message, throwable);
        }
    }
}

// Usage in consensus engine
@Component 
public class RaftConsensusEngine {
    
    private final StructuredLogger structuredLogger;
    
    public void becomeLeader(int term) {
        structuredLogger.logConsensusEvent("leader_elected",
            "term", term,
            "node_id", nodeId,
            "cluster_size", clusterSize,
            "election_duration_ms", electionDuration.toMillis()
        );
    }
    
    public void replicateEntry(LogEntry entry, String followerId) {
        structuredLogger.logConsensusEvent("log_replication_started",
            "entry_index", entry.getIndex(),
            "entry_term", entry.getTerm(),
            "follower_id", followerId,
            "payload_size", entry.getPayload().length
        );
    }
    
    public void handleReplicationError(String followerId, Exception error) {
        structuredLogger.logError("Log replication failed", error,
            "follower_id", followerId,
            "current_term", currentTerm,
            "retry_count", getRetryCount(followerId)
        );
    }
}
`

## Dashboards de Referencia

### Consensus Health Dashboard

`yaml
# Grafana dashboard for Raft consensus
dashboard:
  title: "ODIN Consensus Health"
  panels:
    - title: "Leader Stability"
      type: "stat"
      targets:
        - expr: "time() - raft_leader_stability_seconds"
          legendFormat: "Time since last election (seconds)"
      alert:
        condition: "< 300"  # Alert if leader changes more than every 5 minutes
        
    - title: "Replication Latency"
      type: "graph"
      targets:
        - expr: "histogram_quantile(0.95, raft_replication_latency_seconds)"
          legendFormat: "p95 replication latency"
        - expr: "histogram_quantile(0.99, raft_replication_latency_seconds)"
          legendFormat: "p99 replication latency"
      alert:
        condition: "p95 > 0.1"  # Alert if p95 > 100ms
        
    - title: "Consensus Throughput"
      type: "graph"
      targets:
        - expr: "rate(consensus_operations_total{status='success'}[1m])"
          legendFormat: "Successful consensus ops/sec"
        - expr: "rate(consensus_operations_total{status='failure'}[1m])"
          legendFormat: "Failed consensus ops/sec"
          
    - title: "Log Growth Rate"
      type: "graph"
      targets:
        - expr: "rate(raft_log_entries[5m])"
          legendFormat: "Log entries per second"
        - expr: "raft_log_entries"
          legendFormat: "Total log entries"
          
    - title: "Error Rates by Type"
      type: "pie"
      targets:
        - expr: "sum by (error_type) (rate(application_errors_total[5m]))"
          legendFormat: "{{error_type}}"
`

### Performance Dashboard

`yaml
dashboard:
  title: "ODIN Performance Metrics"
  panels:
    - title: "Request Latency Distribution"
      type: "heatmap"
      targets:
        - expr: "rate(http_request_duration_seconds_bucket[5m])"
          format: "heatmap"
          
    - title: "Thread Pool Utilization"
      type: "graph"
      targets:
        - expr: "thread_pool_utilization_percent"
          legendFormat: "{{pool_name}} utilization"
      alert:
        condition: "avg() > 80"
        
    - title: "GC Impact"
      type: "graph"
      targets:
        - expr: "rate(gc_pause_duration_seconds_sum[1m])"
          legendFormat: "GC time per second"
        - expr: "histogram_quantile(0.99, gc_pause_duration_seconds)"
          legendFormat: "p99 GC pause time"
      alert:
        condition: "p99 > 0.1"  # Alert if p99 GC pause > 100ms
        
    - title: "Lock Contention"
      type: "graph"
      targets:
        - expr: "rate(lock_contentions_total[1m])"
          legendFormat: "Lock contentions per second"
        - expr: "rate(cas_failures_total[1m])"
          legendFormat: "CAS failures per second"
`

## Alerting Estratégico

### SLO-based Alerts

`yaml
# Prometheus alerting rules
groups:
  - name: "slo_alerts"
    rules:
      - alert: "HighErrorRate"
        expr: |
          (
            rate(http_requests_total{status=~"5.."}[5m]) / 
            rate(http_requests_total[5m])
          ) > 0.01
        for: "2m"
        labels:
          severity: "page"
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{  | humanizePercentage }} over the last 5 minutes"
          
      - alert: "HighLatency"
        expr: |
          histogram_quantile(0.95, 
            rate(http_request_duration_seconds_bucket[5m])
          ) > 0.02
        for: "2m"
        labels:
          severity: "page"
        annotations:
          summary: "High latency detected"
          description: "95th percentile latency is {{  }}s"
          
      - alert: "ConsensusLeaderFlapping"
        expr: "increase(raft_leader_elections_total[5m]) > 3"
        for: "1m"
        labels:
          severity: "critical"
        annotations:
          summary: "Consensus leader flapping"
          description: "{{  }} leader elections in the last 5 minutes"
          
      - alert: "ThreadPoolSaturation"
        expr: "thread_pool_utilization_percent > 90"
        for: "30s"
        labels:
          severity: "warning"
        annotations:
          summary: "Thread pool near saturation"
          description: "Thread pool {{ .pool_name }} is {{  }}% utilized"
`

### Incident Response Integration

`java
@Component
public class IncidentResponseIntegration {
    
    private final SlackNotifier slackNotifier;
    private final PagerDutyService pagerDuty;
    private final MetricsCollector metrics;
    
    @EventListener
    public void handleCriticalAlert(CriticalAlertEvent event) {
        
        // Create incident context
        IncidentContext context = IncidentContext.builder()
            .alertName(event.getAlertName())
            .severity(event.getSeverity())
            .affectedServices(event.getAffectedServices())
            .currentMetrics(gatherCurrentMetrics())
            .recentTraces(gatherRecentTraces())
            .logContext(gatherLogContext(event.getTimeWindow()))
            .build();
        
        // Page on-call engineer
        if (event.getSeverity() == Severity.CRITICAL) {
            pagerDuty.createIncident(context);
        }
        
        // Notify team channel
        slackNotifier.sendAlert(context);
        
        // Auto-collect diagnostic info
        diagnosticCollector.gatherDiagnostics(context);
        
        // Update incident metrics
        metrics.incrementIncidents(event.getSeverity().toString());
    }
    
    private Map<String, Object> gatherCurrentMetrics() {
        return Map.of(
            "error_rate", getCurrentErrorRate(),
            "p95_latency", getCurrentP95Latency(),
            "thread_pool_utilization", getThreadPoolUtilization(),
            "consensus_leader", getCurrentLeader(),
            "active_connections", getActiveConnections()
        );
    }
    
    private List<TraceData> gatherRecentTraces() {
        return traceService.getTracesInTimeWindow(
            Instant.now().minus(Duration.ofMinutes(5)),
            Instant.now()
        ).stream()
        .filter(trace -> trace.hasErrors() || trace.isSlowRequest())
        .limit(10)
        .collect(toList());
    }
}
`

## Checklist de Observabilidad

!!! checklist "Implementación de Observabilidad"
    
    **Métricas:**
    
    - [ ] Golden signals (latencia, throughput, errores, saturación)
    - [ ] Métricas de consenso (leader stability, replication latency)
    - [ ] Métricas de concurrencia (thread pools, lock contention)
    - [ ] Métricas de negocio (operations/sec, success rates)
    
    **Tracing:**
    
    - [ ] OpenTelemetry integration en todos los servicios
    - [ ] Propagación de context entre servicios
    - [ ] Instrumentación de operaciones críticas
    - [ ] Sampling inteligente para reducir overhead
    
    **Logging:**
    
    - [ ] Structured logging con JSON
    - [ ] Correlación con trace IDs
    - [ ] Log levels apropiados por entorno
    - [ ] Contexto suficiente para debugging
    
    **Alerting:**
    
    - [ ] SLO-based alerts, no threshold-based
    - [ ] Runbooks vinculados a cada alerta
    - [ ] Escalation policies por severidad
    - [ ] Alert fatigue prevention

---

!!! success "Siguiente Tema"
    
    Aplica estos conceptos en **[SLOs y Métricas](slos.md)** para definir objetivos de nivel de servicio específicos para tu sistema.
