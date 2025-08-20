---
title: "Concurrencia Multihilo"
description: "Estructuras lock-free, thread pools aislados, backpressure y patrones para máximo rendimiento"
nav_order: 2
---

# Concurrencia Multihilo

!!! abstract "¿Por qué Concurrencia?"
    
    Los sistemas distribuidos modernos manejan **miles de requests por segundo** con **latencias de microsegundos**. Solo con **concurrencia cuidadosamente diseñada** puedes aprovechar el hardware moderno para alcanzar estos objetivos.

La concurrencia en sistemas distribuidos va más allá de simplemente "usar threads". Requiere:

- **Thread pools especializados** por tipo de carga
- **Estructuras de datos lock-free** para hot paths
- **Backpressure sofisticado** para prevenir cascadas de fallo
- **Patrones de procesamiento** que maximicen throughput

## Arquitectura de Concurrencia de ODIN

`mermaid
graph TB
    subgraph "Client Layer"
        RPC[RPC Requests]
        API[API Gateway]
    end
    
    subgraph "Thread Pools Aislados"
        RPCIO[rpc-io Pool<br/>NIO, timeouts cortos]
        STATE[state-machine Pool<br/>CPU-bound, sized = cores]
        REPL[replication Pool<br/>I/O a seguidores]
        DBIO[db-io Pool<br/>Async drivers]
        BG[bg-maintenance Pool<br/>Snapshots, metrics]
    end
    
    subgraph "Lock-Free Structures"
        QUEUES[Bounded Queues<br/>MPMC, backpressure]
        CACHE[Cache Layer<br/>Concurrent maps]
        METRICS[Metrics<br/>Atomic counters]
    end
    
    RPC --> RPCIO
    API --> RPCIO
    RPCIO --> STATE
    STATE --> REPL
    STATE --> DBIO
    BG --> CACHE
    
    RPCIO -.-> QUEUES
    STATE -.-> QUEUES
    REPL -.-> QUEUES
    
    style RPCIO fill:#e3f2fd
    style STATE fill:#f3e5f5
    style REPL fill:#e8f5e8
    style DBIO fill:#fff3e0
    style BG fill:#fce4ec
`

## Principios de Diseño

### 1. Separation of Concerns por Thread Pool

=== "rpc-io Pool"

    **Responsabilidad:** I/O de clientes y servidores
    
    `java
    ThreadPoolExecutor rpcIoPool = new ThreadPoolExecutor(
        16, 16,                                    // Fixed size
        60, TimeUnit.SECONDS,                      // Keep alive
        new ArrayBlockingQueue<>(1024),            // Bounded queue
        new NamedThreadFactory("rpc-io"),          // Naming
        new ThreadPoolExecutor.AbortPolicy()       // Fail fast
    );
    `
    
    - **Threads:** 16-32 (I/O bound)
    - **Queue:** Bounded, fail-fast cuando llena
    - **Timeouts:** Cortos (1-5 segundos)

=== "state-machine Pool"

    **Responsabilidad:** Aplicar entradas del log de consenso
    
    `java
    ForkJoinPool stateMachinePool = new ForkJoinPool(
        Math.max(1, Runtime.getRuntime().availableProcessors() - 1),
        ForkJoinPool.defaultForkJoinWorkerThreadFactory,
        null,
        false  // FIFO scheduling
    );
    `
    
    - **Threads:** ≈ cores físicos (CPU bound)
    - **Scheduling:** FIFO para predicibilidad
    - **Work stealing:** Habilitado para balance

=== "replication Pool"

    **Responsabilidad:** I/O a seguidores, batching
    
    `java
    ThreadPoolExecutor replicationPool = new ThreadPoolExecutor(
        8, 16,                                     // Elastic sizing
        30, TimeUnit.SECONDS,
        new LinkedBlockingQueue<>(512),            // Larger queue
        new NamedThreadFactory("replication"),
        new CallerRunsPolicy()                     // Backpressure
    );
    `
    
    - **Batching:** Coalescing de flushes
    - **Timeouts:** Más largos para seguidores lentos
    - **Backpressure:** Caller-runs para throttling

### 2. Lock-Free Hot Paths

Los paths más críticos evitan locks usando **Compare-And-Swap (CAS)**:

`java
public class LockFreeCounter {
    private final AtomicLong count = new AtomicLong(0);
    
    public long increment() {
        return count.incrementAndGet();  // Single CAS operation
    }
    
    public long addAndGet(long delta) {
        long current, updated;
        do {
            current = count.get();
            updated = current + delta;
        } while (!count.compareAndSet(current, updated));
        
        return updated;
    }
}
`

### 3. Backpressure en Cada Frontera

`java
public interface BackpressureStrategy {
    
    enum Action {
        ACCEPT,           // Procesar request
        REJECT,           // Rechazar con código específico
        THROTTLE,         // Rate limit
        CALLER_RUNS       // Ejecutar en thread del caller
    }
    
    Action decide(QueueMetrics metrics, Request request);
}

public class AdaptiveBackpressure implements BackpressureStrategy {
    
    @Override
    public Action decide(QueueMetrics metrics, Request request) {
        double utilization = metrics.getQueueSize() / (double) metrics.getCapacity();
        
        if (utilization > 0.9) {
            return Action.REJECT;
        } else if (utilization > 0.75) {
            return request.getPriority().isHigh() ? Action.ACCEPT : Action.THROTTLE;
        }
        
        return Action.ACCEPT;
    }
}
`

## Temas Principales

### :material-flash: [Estructuras Lock-Free](lock_free.md)

Técnicas avanzadas para eliminación de contención:

- **Atomic operations:** CAS, fetch-and-add, memory ordering
- **Memory management:** Hazard pointers, epoch-based reclamation
- **Estructuras:** Stacks, queues, hash tables sin locks
- **Problemas:** ABA, false sharing, memory fences

### :material-view-grid: [Thread Pools Aislados](thread_pools.md)

Diseño de pools especializados para máximo rendimiento:

- **Dimensionamiento:** Fórmulas para CPU-bound vs I/O-bound
- **Topologías:** Pipeline, staged, actor-based
- **Configuración:** Queue types, rejection policies, monitoring
- **Patterns:** Bulkheads, circuit breakers, graceful degradation

### :material-water: [Backpressure](backpressure.md)

Control de flujo sofisticado para prevenir overload:

- **Estrategias:** Rate limiting, load shedding, adaptive throttling
- **Implementación:** Token buckets, sliding windows, PID controllers
- **Propagation:** End-to-end backpressure chains
- **Metrics:** Queue depth, rejection rates, latency percentiles

### :material-puzzle: [Patrones Avanzados](patrones.md)

Arquitecturas de procesamiento de alto rendimiento:

- **Actor Model:** Message passing, location transparency
- **SEDA:** Staged event-driven architecture  
- **Disruptor:** Ultra-low latency ring buffers
- **Flow Control:** Reactive streams, demand-driven processing

## Ejemplo Práctico: RPC Handler

`java
@Component
public class HighPerformanceRpcHandler {
    
    private final ThreadPoolExecutor rpcPool;
    private final ForkJoinPool stateMachinePool;
    private final BackpressureManager backpressure;
    private final MetricsCollector metrics;
    
    @PostConstruct
    public void initializePools() {
        int cores = Runtime.getRuntime().availableProcessors();
        
        // RPC I/O pool - sized for network I/O
        this.rpcPool = new ThreadPoolExecutor(
            cores * 2, cores * 4,                  // 2-4x cores for I/O
            60, TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(1000),
            new NamedThreadFactory("rpc-handler"),
            (task, executor) -> {
                // Custom rejection: measure and reject with backpressure
                metrics.incrementRejections("rpc-pool-full");
                throw new RpcOverloadException("RPC pool saturated", 
                    Duration.ofMillis(100)); // Retry-after hint
            }
        );
        
        // State machine pool - sized for CPU work
        this.stateMachinePool = new ForkJoinPool(
            Math.max(1, cores - 1),
            ForkJoinPool.defaultForkJoinWorkerThreadFactory,
            (thread, exception) -> {
                // Log uncaught exceptions in state machine
                logger.error("Uncaught exception in state machine", exception);
                metrics.incrementErrors("state-machine-exception");
            },
            false
        );
    }
    
    public CompletableFuture<RpcResponse> handleRequest(RpcRequest request) {
        // Apply backpressure before accepting work
        BackpressureDecision decision = backpressure.shouldAccept(request);
        
        if (decision.isReject()) {
            metrics.incrementRejections(decision.getReason());
            return CompletableFuture.failedFuture(
                new RpcOverloadException("Backpressure applied: " + decision.getReason(),
                    decision.getRetryAfter())
            );
        }
        
        // Process in RPC pool
        return CompletableFuture
            .supplyAsync(() -> {
                try (Timer.Sample sample = metrics.startTimer("rpc-parse")) {
                    return parseAndValidate(request);
                }
            }, rpcPool)
            .thenComposeAsync(validatedRequest -> {
                // CPU-intensive work in state machine pool
                return processInStateMachine(validatedRequest);
            }, stateMachinePool)
            .thenComposeAsync(result -> {
                // I/O back in RPC pool
                return serializeResponse(result);
            }, rpcPool)
            .whenComplete((response, throwable) -> {
                if (throwable != null) {
                    metrics.incrementErrors("rpc-processing-error");
                } else {
                    metrics.incrementSuccess();
                }
            });
    }
    
    private CompletableFuture<ProcessingResult> processInStateMachine(ValidatedRequest request) {
        return CompletableFuture.supplyAsync(() -> {
            try (Timer.Sample sample = metrics.startTimer("state-machine-processing")) {
                
                // State machine work must be:
                // 1. Deterministic
                // 2. CPU-bound  
                // 3. No I/O or blocking calls
                
                return stateMachine.apply(request);
                
            } catch (Exception e) {
                // State machine exceptions are serious
                logger.error("State machine processing failed for request: {}", 
                    request.getId(), e);
                throw new StateMachineException("Processing failed", e);
            }
        }, stateMachinePool);
    }
}
`

## Métricas de Concurrencia

### KPIs Fundamentales

`java
// Thread pool health
Gauge activeThreads = Gauge.build()
    .name("thread_pool_active_threads")
    .help("Currently active threads")
    .labelNames("pool_name")
    .register();

Gauge queueSize = Gauge.build()
    .name("thread_pool_queue_size")
    .help("Tasks waiting in queue")
    .labelNames("pool_name")
    .register();

Counter rejectedTasks = Counter.build()
    .name("thread_pool_rejected_tasks_total")
    .help("Tasks rejected due to saturation")
    .labelNames("pool_name", "reason")
    .register();

// Concurrency contention
Histogram lockWaitTime = Histogram.build()
    .name("lock_wait_time_seconds")
    .help("Time spent waiting for locks")
    .labelNames("lock_name")
    .register();

Counter casFailures = Counter.build()
    .name("cas_failures_total")
    .help("Failed compare-and-swap operations")
    .labelNames("operation")
    .register();

// GC impact on concurrency
Histogram gcPauseTime = Histogram.build()
    .name("gc_pause_duration_seconds")
    .help("GC pause duration")
    .labelNames("gc_type")
    .register();
`

### Dashboard de Concurrencia

`yaml
# Grafana dashboard queries
panels:
  - title: "Thread Pool Utilization"
    targets:
      - expr: "thread_pool_active_threads / thread_pool_max_threads"
        legendFormat: "{{pool_name}} utilization"
    alert:
      condition: "avg() > 0.8"
      
  - title: "Queue Backlog"
    targets:
      - expr: "thread_pool_queue_size"
        legendFormat: "{{pool_name}} queue depth"
    alert:
      condition: "avg() > 100"
      
  - title: "Rejection Rate"
    targets:
      - expr: "rate(thread_pool_rejected_tasks_total[1m])"
        legendFormat: "{{pool_name}} rejections/sec"
`

## Laboratorio Práctico

### 1. Thread Pool Sizing Experiment

`java
@Test
public void findOptimalThreadPoolSize() {
    List<Integer> poolSizes = Arrays.asList(1, 2, 4, 8, 16, 32, 64);
    Map<Integer, PerformanceMetrics> results = new HashMap<>();
    
    for (int poolSize : poolSizes) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            poolSize, poolSize,
            0, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>()
        );
        
        PerformanceMetrics metrics = runLoadTest(executor, Duration.ofMinutes(2));
        results.put(poolSize, metrics);
        
        executor.shutdown();
    }
    
    // Analyze results
    int optimalSize = findOptimalSize(results);
    logger.info("Optimal thread pool size: {}", optimalSize);
}

private PerformanceMetrics runLoadTest(Executor executor, Duration duration) {
    AtomicLong completedTasks = new AtomicLong(0);
    List<Long> latencies = new CopyOnWriteArrayList<>();
    
    long endTime = System.currentTimeMillis() + duration.toMillis();
    
    while (System.currentTimeMillis() < endTime) {
        long startTime = System.nanoTime();
        
        CompletableFuture.runAsync(() -> {
            // Simulate CPU work
            performCpuIntensiveTask();
            
            long latency = System.nanoTime() - startTime;
            latencies.add(latency);
            completedTasks.incrementAndGet();
        }, executor);
        
        Thread.sleep(1); // Control load generation rate
    }
    
    return new PerformanceMetrics(
        completedTasks.get(),
        calculatePercentile(latencies, 95),
        calculatePercentile(latencies, 99)
    );
}
`

### 2. Lock-Free vs Synchronized Benchmark

`java
@Benchmark
@BenchmarkMode(Mode.Throughput)
public class ConcurrencyBenchmark {
    
    private final AtomicLong atomicCounter = new AtomicLong(0);
    private final Object lock = new Object();
    private volatile long synchronizedCounter = 0;
    
    @Benchmark
    @Group("atomic")
    @GroupThreads(4)
    public long atomicIncrement() {
        return atomicCounter.incrementAndGet();
    }
    
    @Benchmark
    @Group("synchronized") 
    @GroupThreads(4)
    public long synchronizedIncrement() {
        synchronized (lock) {
            return ++synchronizedCounter;
        }
    }
    
    @Benchmark
    @Group("lockfree")
    @GroupThreads(4) 
    public long lockFreeIncrement() {
        long current, updated;
        do {
            current = atomicCounter.get();
            updated = current + 1;
        } while (!atomicCounter.compareAndSet(current, updated));
        
        return updated;
    }
}
`

## Checklist de Concurrencia

!!! checklist "Implementación de Concurrencia"
    
    **Diseño de Thread Pools:**
    
    - [ ] Separar pools por tipo de carga (CPU vs I/O)
    - [ ] Dimensionar según cores y características de trabajo
    - [ ] Configurar queues bounded con manejo de overflow
    - [ ] Implementar naming y logging detallado
    
    **Lock-Free Structures:**
    
    - [ ] Identificar hot paths para optimización lock-free
    - [ ] Usar AtomicReference/AtomicLong para contadores
    - [ ] Implementar hazard pointers para memory management
    - [ ] Validar con jcstress para race conditions
    
    **Backpressure:**
    
    - [ ] Definir políticas de rechazo por pool
    - [ ] Implementar métricas de queue depth
    - [ ] Configurar alertas por rejection rate
    - [ ] Propagar backpressure end-to-end
    
    **Monitoring:**
    
    - [ ] Thread pool utilization y queue metrics
    - [ ] GC pause impact en latencias
    - [ ] Context switch rates
    - [ ] Lock contention y CAS failure rates

---

!!! success "Siguiente Tema"
    
    Profundiza en **[Estructuras Lock-Free](lock_free.md)** para maximizar performance en hot paths críticos.
