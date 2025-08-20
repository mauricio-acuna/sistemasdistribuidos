---
title: "Detección de Fallos"
description: "Algoritmos y patrones para detectar fallos en sistemas distribuidos"
nav_order: 2
---

# Detección de Fallos en Sistemas Distribuidos

!!! quote "Michael J. Fischer"
    
    "En un sistema distribuido asíncrono, es imposible distinguir entre un proceso lento y uno que ha fallado."

## 🎯 Resumen Ejecutivo

La **detección de fallos** es fundamental para la construcción de sistemas distribuidos resilientes. Aunque es **imposible** detectar fallos con 100% de precisión en sistemas asíncronos, podemos construir **detectores probabilísticos** efectivos.

### Conceptos Clave
- **Failure Detectors**: Clasificación según completeness y accuracy
- **Heartbeats**: Protocolo básico de monitoreo
- **φ-accrual**: Detector probabilístico adaptativo
- **Gossip Protocol**: Detección distribuida escalable

### Takeaways Principales
- Los failure detectors **perfectos** son imposibles en sistemas asíncronos
- Los detectores **eventually perfect** son suficientes para consenso
- **φ-accrual** proporciona detección adaptativa y robusta
- **Gossip protocols** escalan horizontalmente para grandes clusters

---

## 🚨 Tipos de Fallos

### 📋 Clasificación de Fallos

`java
public enum FailureType {
    /**
     * El proceso se detiene completamente y no responde
     */
    CRASH_STOP("Process stops executing and never recovers"),
    
    /**
     * El proceso se detiene pero puede recuperarse
     */
    CRASH_RECOVERY("Process may stop and later recover"),
    
    /**
     * Fallos en la red que particionan el sistema
     */
    NETWORK_PARTITION("Network splits cluster into disjoint groups"),
    
    /**
     * El proceso envía mensajes incorrectos o inconsistentes
     */
    BYZANTINE("Process behaves arbitrarily or maliciously"),
    
    /**
     * El proceso funciona más lento de lo esperado
     */
    PERFORMANCE_DEGRADATION("Process responds but very slowly");
    
    private final String description;
    
    FailureType(String description) {
        this.description = description;
    }
}
`

### 🔍 Failure Detectors - Clasificación Teórica

`java
public interface FailureDetector {
    /**
     * Consulta si un proceso es sospechoso de haber fallado
     */
    boolean isSuspected(ProcessId process);
    
    /**
     * Propiedades del failure detector
     */
    Properties getProperties();
}

public class Properties {
    // COMPLETENESS: Todo proceso fallido es eventualmente sospechado
    public enum Completeness {
        WEAK,      // Al menos un proceso correcto sospecha del fallido
        STRONG     // Todos los procesos correctos sospechan del fallido
    }
    
    // ACCURACY: Procesos correctos no son sospechados incorrectamente
    public enum Accuracy {
        WEAK,              // Algún proceso correcto nunca es sospechado
        STRONG,            // Ningún proceso correcto es sospechado
        EVENTUALLY_WEAK,   // Eventualmente algún proceso correcto no es sospechado
        EVENTUALLY_STRONG  // Eventualmente ningún proceso correcto es sospechado
    }
}
`

---

## 💓 Heartbeat - Detector Básico

### 🔧 Implementación Simple

`java
public class HeartbeatFailureDetector implements FailureDetector {
    private final long heartbeatTimeout;
    private final Map<ProcessId, Long> lastHeartbeat;
    private final Set<ProcessId> suspectedProcesses;
    private final ScheduledExecutorService scheduler;
    
    public HeartbeatFailureDetector(long timeoutMs) {
        this.heartbeatTimeout = timeoutMs;
        this.lastHeartbeat = new ConcurrentHashMap<>();
        this.suspectedProcesses = ConcurrentHashMap.newKeySet();
        this.scheduler = Executors.newScheduledThreadPool(2);
        
        // Verificar timeouts periódicamente
        scheduler.scheduleAtFixedRate(this::checkTimeouts, 
                timeoutMs / 2, timeoutMs / 2, TimeUnit.MILLISECONDS);
    }
    
    public void receiveHeartbeat(ProcessId process) {
        lastHeartbeat.put(process, System.currentTimeMillis());
        suspectedProcesses.remove(process);
        
        log.debug("Heartbeat received from {}", process);
    }
    
    @Override
    public boolean isSuspected(ProcessId process) {
        return suspectedProcesses.contains(process);
    }
    
    private void checkTimeouts() {
        long now = System.currentTimeMillis();
        
        lastHeartbeat.entrySet().removeIf(entry -> {
            ProcessId process = entry.getKey();
            long lastSeen = entry.getValue();
            
            if (now - lastSeen > heartbeatTimeout) {
                suspectedProcesses.add(process);
                log.warn("Process {} suspected failed (last seen {} ms ago)", 
                        process, now - lastSeen);
                return true;
            }
            return false;
        });
    }
}
`

### 📊 Heartbeat con Jitter

`java
public class AdaptiveHeartbeatSender {
    private final ProcessId localId;
    private final Set<ProcessId> peers;
    private final NetworkService network;
    private final Random random = new Random();
    
    public void startSendingHeartbeats(long baseInterval) {
        peers.forEach(peer -> {
            // Agregar jitter para evitar thundering herd
            long jitter = (long) (baseInterval * 0.1 * random.nextGaussian());
            long interval = Math.max(100, baseInterval + jitter);
            
            scheduler.scheduleAtFixedRate(
                () -> sendHeartbeatTo(peer),
                0, interval, TimeUnit.MILLISECONDS
            );
        });
    }
    
    private void sendHeartbeatTo(ProcessId peer) {
        try {
            HeartbeatMessage msg = new HeartbeatMessage(
                localId, 
                System.currentTimeMillis(),
                getLocalMetrics()
            );
            
            network.sendAsync(peer, msg)
                   .exceptionally(ex -> {
                       log.debug("Failed to send heartbeat to {}: {}", 
                               peer, ex.getMessage());
                       return null;
                   });
                   
        } catch (Exception e) {
            log.error("Error sending heartbeat to {}", peer, e);
        }
    }
}
`

---

## 📈 φ-accrual (Phi-Accrual) Failure Detector

### 🧠 Concepto Fundamental

En lugar de binario (fallido/vivo), φ-accrual produce un **valor continuo de sospecha** basado en la distribución histórica de llegadas de heartbeats.

### 📐 Algoritmo

`java
public class PhiAccrualFailureDetector implements FailureDetector {
    private final double phiThreshold;
    private final int maxSampleSize;
    private final Map<ProcessId, ArrivalWindow> arrivalWindows;
    
    public PhiAccrualFailureDetector(double phiThreshold) {
        this.phiThreshold = phiThreshold;  // Típicamente 8.0-12.0
        this.maxSampleSize = 1000;
        this.arrivalWindows = new ConcurrentHashMap<>();
    }
    
    public void heartbeatReceived(ProcessId process) {
        long now = System.currentTimeMillis();
        
        arrivalWindows.computeIfAbsent(process, 
            k -> new ArrivalWindow(maxSampleSize))
            .addArrival(now);
    }
    
    @Override
    public boolean isSuspected(ProcessId process) {
        return getPhi(process) > phiThreshold;
    }
    
    /**
     * Calcula el valor phi (sospecha) para un proceso
     */
    public double getPhi(ProcessId process) {
        ArrivalWindow window = arrivalWindows.get(process);
        if (window == null || window.size() < 2) {
            return 0.0;  // Insuficientes datos
        }
        
        long now = System.currentTimeMillis();
        long timeSinceLastHeartbeat = now - window.getLastArrival();
        
        // Calcular distribución de intervalos históricos
        double mean = window.getMean();
        double variance = window.getVariance();
        
        if (variance < 1e-6) {
            variance = 1.0;  // Evitar división por cero
        }
        
        // Phi basado en distribución normal
        double phi = -Math.log10(probabilityOfArrival(
            timeSinceLastHeartbeat, mean, variance));
            
        return Math.max(0.0, phi);
    }
    
    private double probabilityOfArrival(long interval, double mean, double variance) {
        if (interval <= mean) {
            return 1.0;
        }
        
        // Usar distribución exponencial modificada
        double lambda = 1.0 / mean;
        return Math.exp(-lambda * (interval - mean));
    }
}

class ArrivalWindow {
    private final CircularBuffer<Long> arrivals;
    private volatile long lastArrival;
    
    public ArrivalWindow(int maxSize) {
        this.arrivals = new CircularBuffer<>(maxSize);
    }
    
    public synchronized void addArrival(long timestamp) {
        if (lastArrival > 0) {
            long interval = timestamp - lastArrival;
            arrivals.add(interval);
        }
        lastArrival = timestamp;
    }
    
    public double getMean() {
        return arrivals.stream()
                .mapToLong(Long::longValue)
                .average()
                .orElse(1000.0);  // Default 1 segundo
    }
    
    public double getVariance() {
        double mean = getMean();
        return arrivals.stream()
                .mapToDouble(interval -> Math.pow(interval - mean, 2))
                .average()
                .orElse(1000.0);
    }
}
`

### 🎯 Ventajas de φ-accrual

1. **Adaptativo**: Se ajusta automáticamente a patrones de red
2. **Probabilístico**: Proporciona confianza graduada
3. **Robusto**: Maneja variabilidad de latencia de red
4. **Configurable**: Threshold φ ajusta sensibilidad

---

## 🗣️ Gossip-Based Failure Detection

### 📡 SWIM Protocol (Scalable Weakly-consistent Infection-style process group Membership)

`java
public class SwimFailureDetector {
    private final ProcessId localId;
    private final Set<ProcessId> members;
    private final Map<ProcessId, MembershipInfo> membershipTable;
    private final NetworkService network;
    private final Random random = new Random();
    
    // Configuración SWIM
    private final int protocolPeriod = 1000;  // ms
    private final int k = 3;  // Número de nodos para indirect ping
    private final int pingTimeout = 500;     // ms
    private final int pingReqTimeout = 700;  // ms
    
    public void startSwimProtocol() {
        scheduler.scheduleAtFixedRate(this::swimCycle, 
                protocolPeriod, protocolPeriod, TimeUnit.MILLISECONDS);
    }
    
    private void swimCycle() {
        // 1. Seleccionar un miembro aleatorio para ping
        ProcessId target = selectRandomMember();
        if (target == null) return;
        
        // 2. Enviar ping directo
        CompletableFuture<Void> directPing = sendPing(target);
        
        try {
            directPing.get(pingTimeout, TimeUnit.MILLISECONDS);
            // Ping exitoso - marcar como vivo
            updateMemberStatus(target, MemberStatus.ALIVE);
            
        } catch (TimeoutException e) {
            // 3. Ping directo falló - intentar ping indirecto
            handleIndirectPing(target);
        } catch (Exception e) {
            log.error("Error in SWIM cycle", e);
        }
        
        // 4. Procesar gossip de membership
        disseminateGossip();
    }
    
    private void handleIndirectPing(ProcessId target) {
        // Seleccionar k nodos aleatorios para ping indirecto
        List<ProcessId> intermediates = selectRandomMembers(k);
        intermediates.remove(target);
        
        List<CompletableFuture<Void>> indirectPings = intermediates.stream()
                .map(intermediate -> sendPingReq(intermediate, target))
                .collect(Collectors.toList());
        
        CompletableFuture<Void> anySuccess = CompletableFuture.anyOf(
                indirectPings.toArray(new CompletableFuture[0])
        ).thenApply(result -> null);
        
        try {
            anySuccess.get(pingReqTimeout, TimeUnit.MILLISECONDS);
            // Al menos un ping indirecto exitoso
            updateMemberStatus(target, MemberStatus.ALIVE);
            
        } catch (TimeoutException e) {
            // Todos los pings indirectos fallaron
            suspectMember(target);
        }
    }
    
    private void suspectMember(ProcessId target) {
        MembershipInfo info = membershipTable.get(target);
        if (info.getStatus() != MemberStatus.SUSPECTED) {
            info.setStatus(MemberStatus.SUSPECTED);
            info.setIncarnation(info.getIncarnation() + 1);
            
            log.warn("Suspecting member {} (incarnation {})", 
                    target, info.getIncarnation());
            
            // Programar confirmación de fallo después del período de sospecha
            scheduler.schedule(() -> confirmFailure(target), 
                    3 * protocolPeriod, TimeUnit.MILLISECONDS);
        }
    }
    
    private void disseminateGossip() {
        // Seleccionar algunos miembros para gossip
        List<ProcessId> gossipTargets = selectRandomMembers(3);
        
        GossipMessage gossip = createGossipMessage();
        gossipTargets.forEach(target -> 
            network.sendAsync(target, gossip));
    }
}

enum MemberStatus {
    ALIVE, SUSPECTED, FAILED, LEFT
}

class MembershipInfo {
    private volatile MemberStatus status;
    private volatile long incarnation;
    private volatile long lastUpdated;
    
    // Constructor, getters, setters...
}
`

### 📊 Ventajas de SWIM

1. **Escalabilidad**: Complejidad de mensaje O(1) por nodo
2. **Robustez**: Detección por múltiples rutas
3. **Eficiencia**: Baja overhead de red
4. **Auto-curación**: Recuperación automática de falsos positivos

---

## 🔧 Patterns de Implementación

### 🎛️ Circuit Breaker con Failure Detection

`java
public class FailureDetectorCircuitBreaker {
    private final FailureDetector failureDetector;
    private final ProcessId targetService;
    private volatile CircuitState state = CircuitState.CLOSED;
    private final AtomicLong failureCount = new AtomicLong(0);
    private final AtomicLong lastFailureTime = new AtomicLong(0);
    
    // Configuración
    private final long failureThreshold = 5;
    private final long timeoutMs = 30000;  // 30 segundos
    private final long retryTimeoutMs = 10000;  // 10 segundos
    
    public <T> CompletableFuture<T> execute(Supplier<CompletableFuture<T>> operation) {
        if (state == CircuitState.OPEN) {
            if (shouldAttemptReset()) {
                state = CircuitState.HALF_OPEN;
            } else {
                return CompletableFuture.failedFuture(
                    new CircuitBreakerOpenException("Circuit breaker is OPEN"));
            }
        }
        
        // Verificar failure detector
        if (failureDetector.isSuspected(targetService)) {
            return handleSuspectedFailure();
        }
        
        return operation.get()
                .exceptionally(this::handleOperationFailure)
                .thenApply(this::handleOperationSuccess);
    }
    
    private boolean shouldAttemptReset() {
        return System.currentTimeMillis() - lastFailureTime.get() > retryTimeoutMs;
    }
    
    private <T> CompletableFuture<T> handleSuspectedFailure() {
        onFailure();
        return CompletableFuture.failedFuture(
            new ServiceUnavailableException("Service suspected failed"));
    }
    
    private void onFailure() {
        failureCount.incrementAndGet();
        lastFailureTime.set(System.currentTimeMillis());
        
        if (failureCount.get() >= failureThreshold) {
            state = CircuitState.OPEN;
            log.warn("Circuit breaker opened for service {}", targetService);
        }
    }
}
`

### 🔄 Adaptive Timeout

`java
public class AdaptiveTimeout {
    private final CircularBuffer<Long> responseTimeHistory;
    private final double multiplier;
    private final long minTimeout;
    private final long maxTimeout;
    
    public AdaptiveTimeout(int historySize, double multiplier) {
        this.responseTimeHistory = new CircularBuffer<>(historySize);
        this.multiplier = multiplier;  // Típicamente 2.0 - 3.0
        this.minTimeout = 100;   // 100ms mínimo
        this.maxTimeout = 30000; // 30s máximo
    }
    
    public void recordResponseTime(long responseTimeMs) {
        responseTimeHistory.add(responseTimeMs);
    }
    
    public long getCurrentTimeout() {
        if (responseTimeHistory.isEmpty()) {
            return minTimeout;
        }
        
        // Calcular percentil 95 de tiempos de respuesta históricos
        List<Long> sorted = responseTimeHistory.stream()
                .sorted()
                .collect(Collectors.toList());
        
        int p95Index = (int) (sorted.size() * 0.95);
        long p95ResponseTime = sorted.get(Math.min(p95Index, sorted.size() - 1));
        
        long adaptiveTimeout = (long) (p95ResponseTime * multiplier);
        
        return Math.max(minTimeout, Math.min(maxTimeout, adaptiveTimeout));
    }
}
`

---

## 🧪 Testing Failure Detection

### ⚡ Chaos Engineering

`java
@Component
public class FailureInjector {
    private final NetworkService network;
    private final Random random = new Random();
    
    /**
     * Simula fallos de red aleatorios
     */
    public void startNetworkChaos(double failureRate) {
        // Interceptar mensajes salientes
        network.addInterceptor(message -> {
            if (random.nextDouble() < failureRate) {
                log.info("Chaos: Dropping message to {}", message.getTarget());
                return false;  // Drop mensaje
            }
            return true;  // Permitir mensaje
        });
    }
    
    /**
     * Simula latencia variable de red
     */
    public void startLatencyChaos(long baseLatency, double variability) {
        network.addLatencySimulator((source, target) -> {
            double factor = 1.0 + (random.nextGaussian() * variability);
            return (long) (baseLatency * Math.max(0.1, factor));
        });
    }
    
    /**
     * Simula particiones de red
     */
    public void simulateNetworkPartition(Set<ProcessId> partition1, 
                                       Set<ProcessId> partition2, 
                                       Duration duration) {
        
        NetworkPartition partition = new NetworkPartition(partition1, partition2);
        network.addPartition(partition);
        
        // Restaurar conectividad después del tiempo especificado
        scheduler.schedule(() -> {
            network.removePartition(partition);
            log.info("Network partition healed");
        }, duration.toMillis(), TimeUnit.MILLISECONDS);
    }
}
`

### �� Property-Based Testing

`java
@Property
public void failureDetectorCompleteness(
    @ForAll("failureScenarios") List<FailureEvent> events) {
    
    PhiAccrualFailureDetector detector = new PhiAccrualFailureDetector(8.0);
    Set<ProcessId> actuallyFailed = new HashSet<>();
    Set<ProcessId> detectedFailed = new HashSet<>();
    
    // Simular eventos
    for (FailureEvent event : events) {
        switch (event.type) {
            case HEARTBEAT:
                detector.heartbeatReceived(event.process);
                break;
                
            case PROCESS_FAILURE:
                actuallyFailed.add(event.process);
                break;
                
            case TIME_ADVANCE:
                // Verificar detecciones después de avance de tiempo
                for (ProcessId p : actuallyFailed) {
                    if (detector.isSuspected(p)) {
                        detectedFailed.add(p);
                    }
                }
                break;
        }
    }
    
    // Verificar completeness: todos los fallos son eventualmente detectados
    assertThat(detectedFailed).containsAll(actuallyFailed);
}
`

---

## 📚 Referencias y Recursos

### 📖 Papers Fundamentales

1. **Chandra, T. D., & Toueg, S.** (1996). *Unreliable failure detectors for reliable distributed systems*. Journal of the ACM. [📄 PDF](https://www.cs.cornell.edu/home/sam/FDpapers/CT96-JACM.pdf)

2. **Hayashibara, N., Defago, X., Yared, R., & Katayama, T.** (2004). *The φ accrual failure detector*. IEEE Symposium on Reliable Distributed Systems. [📄 PDF](https://pdfs.semanticscholar.org/11ae/4c0c0d0c36dc177c1fff5eb84fa49aa3e1a8.pdf)

3. **Das, A., Gupta, I., & Motivala, A.** (2002). *SWIM: Scalable weakly-consistent infection-style process group membership protocol*. IEEE International Conference on Dependable Systems and Networks. [📄 PDF](https://www.cs.cornell.edu/projects/Quicksilver/public_pdfs/SWIM.pdf)

### 🛠️ Implementaciones de Referencia

- [**Akka Cluster**](https://github.com/akka/akka/tree/main/akka-cluster) - φ-accrual implementation
- [**HashiCorp Serf**](https://github.com/hashicorp/serf) - SWIM protocol implementation  
- [**Apache Cassandra**](https://github.com/apache/cassandra/blob/trunk/src/java/org/apache/cassandra/gms/FailureDetector.java) - φ-accrual failure detector

### 🔧 Herramientas Prácticas

- **Jepsen**: Framework para testing de sistemas distribuidos
- **Chaos Monkey**: Netflix's chaos engineering tool
- **Pumba**: Chaos testing for Docker containers

---

!!! tip "Próximo Paso"
    
    Con los fundamentos de detección de fallos claros, ahora puedes explorar [**Algoritmos de Consenso**](consenso.md) que dependen de failure detectors para funcionar correctamente.

!!! warning "Consideración de Producción"
    
    En sistemas de producción, combina **múltiples detectores**: φ-accrual para adaptabilidad, heartbeats para simplicidad, y SWIM para escalabilidad. Ajusta thresholds basándose en SLOs reales.
