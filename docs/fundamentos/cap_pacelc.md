---
title: "Teoremas CAP y PACELC"
description: "Consistency, Availability, Partition tolerance: trade-offs fundamentales en sistemas distribuidos"
nav_order: 30
tags: [cap, pacelc, availability, consistency, partitions]
---

# Teoremas CAP y PACELC

!!! abstract "En Pocas Palabras"
    
    - **CAP:** No puedes tener Consistency, Availability y Partition tolerance **simultáneamente**
    - **PACELC:** Incluso sin particiones, debes elegir entre Latency y Consistency

Estos teoremas no son limitaciones académicas abstractas, sino **realidades operativas** que enfrentas diariamente al diseñar sistemas distribuidos.

## El Teorema CAP

### Definición Formal

**En presencia de particiones de red (P), debes elegir entre Consistency (C) y Availability (A).**

`mermaid
graph TB
    subgraph "Sistema Normal"
        C1[Consistency] 
        A1[Availability]
        P1[Partition Tolerance]
        C1 --- A1
        A1 --- P1
        P1 --- C1
    end
    
    subgraph "Durante Partición"
        C2[Consistency]
        A2[Availability] 
        P2[Partition Tolerance]
        P2 --- C2
        P2 --- A2
        C2 -.->|"❌ Choose only ONE"| A2
    end
    
    style P2 fill:#ff9999
`

### Las Tres Propiedades

=== "Consistency"

    **Todos los nodos ven los mismos datos al mismo tiempo**
    
    - Operaciones aparecen **atómicas** e **instantáneas**
    - Equivalente a un sistema **centralizado**
    - Requiere **coordinación** entre nodos
    
    `java
    // Ejemplo: Read-after-write consistency
    database.put("balance", 1000);
    int balance = database.get("balance"); 
    assert balance == 1000; // Siempre verdadero
    `

=== "Availability"

    **El sistema responde a todas las peticiones (exitosa o fallida)**
    
    - Cada nodo **funcionando** debe responder
    - No hay **timeouts infinitos** o fallos
    - Operación termina en **tiempo finito**
    
    `java
    // Ejemplo: Sistema siempre responde
    try {
        String result = service.getData();
        // result puede ser stale, pero nunca timeout
        return result;
    } catch (TimeoutException e) {
        // ❌ Viola availability
        throw e; 
    }
    `

=== "Partition Tolerance"

    **El sistema continúa funcionando a pesar de pérdidas de mensajes**
    
    - Nodos **aislados** por fallos de red
    - Mensajes **perdidos** o **retrasados**
    - **Split-brain** scenarios
    
    `java
    // Red partition detected
    if (canReachMajority()) {
        // Continue serving requests
        return processRequest(request);
    } else {
        // Choose: reject (CP) or serve stale (AP)
        return chooseCAStrategy(request);
    }
    `

### CAP en la Práctica

#### Sistemas CP (Consistency + Partition Tolerance)

**Sacrifican Availability durante particiones**

`java
public class CPBankingSystem {
    private final ConsensusModule consensus;
    private final QuorumManager quorum;
    
    public void transfer(String from, String to, BigDecimal amount) {
        if (!quorum.hasMajority()) {
            // 🚫 Rechazar durante partición para mantener consistency
            throw new ServiceUnavailableException("No quorum available");
        }
        
        // Usar consenso para garantizar consistency
        TransferOperation op = new TransferOperation(from, to, amount);
        consensus.propose(op).join(); // Puede fallar si partición ocurre
    }
    
    public BigDecimal getBalance(String account) {
        if (!quorum.hasMajority()) {
            // 🚫 No servir datos potencialmente stale
            throw new ServiceUnavailableException("No quorum available");
        }
        
        return readFromMajority(account);
    }
}
`

**Ejemplos reales:** etcd, ZooKeeper, sistemas bancarios

#### Sistemas AP (Availability + Partition Tolerance)

**Sacrifican Consistency durante particiones**

`java
public class APContentDelivery {
    private final LocalCache cache;
    private final List<Replica> replicas;
    
    public String getContent(String key) {
        // ✅ Siempre responder, incluso si data es stale
        String content = cache.get(key);
        
        if (content == null) {
            // Intentar cualquier réplica disponible
            for (Replica replica : replicas) {
                try {
                    content = replica.get(key);
                    if (content != null) {
                        cache.put(key, content);
                        return content;
                    }
                } catch (NetworkException e) {
                    // Continue trying other replicas
                    continue;
                }
            }
        }
        
        return content; // Puede ser null o stale, pero no timeout
    }
    
    public void putContent(String key, String content) {
        // ✅ Aceptar write localmente
        cache.put(key, content);
        
        // Propagar asincrónicamente (eventual consistency)
        propagateEventually(key, content);
    }
    
    private void propagateEventually(String key, String content) {
        CompletableFuture.runAsync(() -> {
            for (Replica replica : replicas) {
                try {
                    replica.put(key, content);
                } catch (Exception e) {
                    // Log error pero no fallar la operación original
                    logger.warn("Failed to propagate to {}", replica.getId(), e);
                    // Retry logic aquí...
                }
            }
        });
    }
}
`

**Ejemplos reales:** DNS, CDNs, DynamoDB, Cassandra

### Common Misconceptions

!!! warning "❌ Malentendidos Frecuentes"
    
    1. **"CAP significa elegir 2 de 3"** 
       - ❌ Falso: Partition tolerance no es opcional en sistemas distribuidos
       - ✅ Verdad: Eliges CP o AP cuando ocurren particiones
    
    2. **"CAP es una elección única y global"**
       - ❌ Falso: Puedes ser CP para algunas operaciones, AP para otras
       - ✅ Verdad: Diferentes subsistemas pueden hacer diferentes trade-offs
    
    3. **"Eventual consistency = AP"**
       - ❌ Falso: Puedes tener eventual consistency con CP
       - ✅ Verdad: Es sobre qué garantías ofreces durante particiones

## El Teorema PACELC

**Extensión de CAP que considera latencia incluso sin particiones**

!!! info "PACELC Explicado"
    
    **Si hay Partición (P):** elige Availability (A) o Consistency (C)
    
    **Sino (E - Else):** elige Latency (L) o Consistency (C)

### ¿Por qué PACELC?

CAP solo considera el comportamiento **durante particiones**, pero la mayoría del tiempo los sistemas **no** están particionados. PACELC reconoce que incluso en operación normal, hay trade-offs entre **latencia** y **consistencia**.

`mermaid
flowchart TD
    A[Request Arrives] --> B{Network Partition?}
    B -->|Yes| C{CAP Choice}
    B -->|No| D{PACELC Choice}
    
    C -->|CP| E[Reject Request<br/>Maintain Consistency]
    C -->|AP| F[Serve Request<br/>Risk Inconsistency]
    
    D -->|PC| G[Coordinate Replicas<br/>Higher Latency]
    D -->|PL| H[Serve Locally<br/>Risk Inconsistency]
    
    style B fill:#ffeb3b
    style C fill:#ff9999
    style D fill:#4caf50
`

### Clasificación PACELC de Sistemas

| Sistema | Durante Partición | Sin Partición | Tipo |
|---------|-------------------|---------------|------|
| **MongoDB** | CP | PC | PC/PC |
| **Redis Cluster** | AP | PC | PA/PC |
| **Cassandra** | AP | PL | PA/PL |
| **DynamoDB** | AP | PL | PA/PL |
| **etcd** | CP | PC | PC/PC |
| **BigTable** | CP | PC | PC/PC |

### Implementando PACELC Choices

#### PC/PC: Priorizar Consistency siempre

`java
public class PCPCDatabase {
    
    public void write(String key, String value) {
        if (isPartitioned()) {
            // CAP: Choose CP - rechazar si no hay quorum
            if (!hasWriteQuorum()) {
                throw new NotEnoughReplicasException();
            }
        }
        
        // PACELC: Choose PC - coordinar con réplicas para consistency
        List<CompletableFuture<Void>> futures = replicas.stream()
            .map(replica -> replica.writeAsync(key, value))
            .collect(toList());
            
        // Esperar a mayoría (más latencia, pero consistente)
        waitForMajority(futures);
    }
    
    public String read(String key) {
        if (isPartitioned() && !hasReadQuorum()) {
            throw new NotEnoughReplicasException();
        }
        
        // Read from majority para garantizar consistency
        return readFromMajority(key);
    }
}
`

#### PA/PL: Priorizar Availability y Latency

`java
public class PAPLDatabase {
    
    public void write(String key, String value) {
        // Siempre aceptar writes localmente
        localStore.put(key, value);
        
        if (isPartitioned()) {
            // CAP: Choose AP - continuar funcionando
            asyncReplication.schedule(key, value);
        } else {
            // PACELC: Choose PL - replicar async para baja latencia
            replicas.parallelStream()
                .forEach(replica -> {
                    CompletableFuture.runAsync(() -> {
                        try {
                            replica.write(key, value);
                        } catch (Exception e) {
                            // Log pero no fallar operación
                            replicationQueue.retry(replica, key, value);
                        }
                    });
                });
        }
    }
    
    public String read(String key) {
        // Siempre servir desde local store (baja latencia)
        return localStore.get(key);
    }
}
`

#### PA/PC: Híbrido según operación

`java
public class PAPCDatabase {
    
    public void criticalWrite(String key, String value) {
        // Operación crítica: elegir consistency sobre latency
        if (!hasWriteQuorum()) {
            throw new ServiceUnavailableException();
        }
        
        // Replicación síncrona
        replicateSync(key, value);
    }
    
    public void regularWrite(String key, String value) {
        // Operación regular: elegir availability/latency
        localStore.put(key, value);
        replicateAsync(key, value);
    }
    
    public String read(String key, ConsistencyLevel level) {
        switch (level) {
            case STRONG:
                return readFromMajority(key);
            case EVENTUAL:
                return localStore.get(key);
            default:
                throw new IllegalArgumentException("Unknown level: " + level);
        }
    }
}
`

## Decisiones de Diseño Prácticas

### Framework de Decisión

`mermaid
flowchart TD
    A[Analizar Requerimientos] --> B{¿Datos críticos?}
    B -->|Sí| C{¿Tolerancia a downtime?}
    B -->|No| D{¿Latencia crítica?}
    
    C -->|Baja| E[CP System<br/>MongoDB, etcd]
    C -->|Alta| F[Híbrido<br/>Different guarantees per operation]
    
    D -->|Sí| G[AP/PL System<br/>Cassandra, DynamoDB]
    D -->|No| H[AP/PC System<br/>Redis with sync replication]
    
    style E fill:#ff9999
    style G fill:#4caf50
    style F fill:#ffeb3b
    style H fill:#2196f3
`

### Patrones de Configuración

#### 1. Feature Flags para CAP Choices

`java
@Component
public class CapAwareService {
    
    @Value("")
    private String capPreference;
    
    @Value("")
    private boolean partitionDetectionEnabled;
    
    public ServiceResponse handleRequest(ServiceRequest request) {
        if (partitionDetectionEnabled && networkPartitioner.isPartitioned()) {
            return handlePartitionedRequest(request);
        }
        
        return handleNormalRequest(request);
    }
    
    private ServiceResponse handlePartitionedRequest(ServiceRequest request) {
        if ("CP".equals(capPreference)) {
            // Priorizar consistency - rechazar si no hay quorum
            if (!quorumManager.hasQuorum()) {
                throw new ServiceUnavailableException("No quorum during partition");
            }
            return processWithStrongConsistency(request);
        } else {
            // Priorizar availability - servir con datos locales
            return processWithLocalData(request);
        }
    }
}
`

#### 2. Consistency Levels Configurables

`java
public enum ConsistencyLevel {
    STRONG,      // CP choice - coordinación completa
    BOUNDED,     // Compromiso - bounded staleness
    SESSION,     // Por-session consistency
    EVENTUAL     // AP choice - eventual convergencia
}

@Service
public class ConfigurableConsistencyService {
    
    public String read(String key, ConsistencyLevel level) {
        switch (level) {
            case STRONG:
                return readWithQuorum(key);
            case BOUNDED:
                return readWithBoundedStaleness(key, Duration.ofSeconds(1));
            case SESSION:
                return readWithSessionConsistency(key);
            case EVENTUAL:
                return readLocal(key);
        }
    }
    
    private String readWithBoundedStaleness(String key, Duration maxStaleness) {
        LocalValue local = localStore.get(key);
        
        if (local == null || local.getAge().compareTo(maxStaleness) > 0) {
            // Data too stale, read from remote
            return readFromReplica(key);
        }
        
        return local.getValue();
    }
}
`

### Métricas para CAP Decisions

`java
@Component
public class CapMetrics {
    
    private final Counter partitionDetectedCounter = Counter.build()
        .name("partition_detected_total")
        .help("Number of network partitions detected")
        .register();
        
    private final Counter cpChoiceCounter = Counter.build()
        .name("cap_cp_choice_total")
        .help("Times CP was chosen during partition")
        .register();
        
    private final Counter apChoiceCounter = Counter.build()
        .name("cap_ap_choice_total")
        .help("Times AP was chosen during partition")
        .register();
        
    private final Histogram consistencyLatency = Histogram.build()
        .name("consistency_operation_duration_seconds")
        .help("Latency of consistency-requiring operations")
        .labelNames("consistency_level")
        .register();
        
    private final Gauge currentConsistencyLevel = Gauge.build()
        .name("current_consistency_level")
        .help("Current consistency level (0=eventual, 1=strong)")
        .register();
        
    public void recordPartition() {
        partitionDetectedCounter.inc();
    }
    
    public void recordCapChoice(CapChoice choice) {
        if (choice == CapChoice.CP) {
            cpChoiceCounter.inc();
        } else {
            apChoiceCounter.inc();
        }
    }
    
    public Timer.Sample startConsistencyTimer(String level) {
        return Timer.start(consistencyLatency.labels(level));
    }
}
`

## Casos de Estudio Reales

### 1. Netflix: Híbrido AP/CP

`java
// Simplified Netflix architecture decisions
public class NetflixStyleSystem {
    
    // AP para viewing data (puede ser stale)
    public UserViewingHistory getViewingHistory(String userId) {
        // Priorizar availability y latency
        return localCache.getViewingHistory(userId);
    }
    
    // CP para billing data (debe ser consistente)
    public void recordPayment(String userId, Payment payment) {
        // Priorizar consistency sobre availability
        if (!billingQuorum.hasQuorum()) {
            throw new PaymentServiceUnavailableException();
        }
        
        billingConsensus.propose(new RecordPaymentOperation(userId, payment));
    }
    
    // Híbrido para recommendations (eventual ok, pero no muy stale)
    public List<Movie> getRecommendations(String userId) {
        RecommendationData data = recommendationCache.get(userId);
        
        if (data.getAge().toMinutes() > 15) {
            // Too stale para recommendations, refresh async
            refreshRecommendationsAsync(userId);
        }
        
        return data.getMovies();
    }
}
`

### 2. Amazon DynamoDB: Tunable Consistency

`java
// DynamoDB-style configurable consistency
public class TunableConsistencyKVStore {
    
    public String read(String key, ReadConsistency consistency) {
        switch (consistency) {
            case STRONG:
                // CP choice: read from majority
                return stronglyConsistentRead(key);
                
            case EVENTUAL:
                // AP choice: read from any replica
                return eventuallyConsistentRead(key);
        }
    }
    
    private String stronglyConsistentRead(String key) {
        List<String> values = replicas.parallelStream()
            .map(replica -> replica.get(key))
            .collect(toList());
            
        // Return majority value
        return findMajorityValue(values);
    }
    
    private String eventuallyConsistentRead(String key) {
        // Read from any available replica (lower latency)
        for (Replica replica : replicas) {
            try {
                return replica.get(key);
            } catch (Exception e) {
                continue; // Try next replica
            }
        }
        
        throw new AllReplicasUnavailableException();
    }
}
`

## Testing CAP Behaviors

### 1. Partition Testing

`java
@Test
public void testCapBehaviorDuringPartition() {
    // Setup: cluster with 3 nodes
    DistributedSystem system = new DistributedSystem(3);
    
    // Write initial data
    system.write("key1", "value1");
    
    // Simulate network partition: isolate 1 node
    system.partition(Arrays.asList(0, 1), Arrays.asList(2));
    
    if (system.isCP()) {
        // CP system: majority side should work, minority should fail
        assertTrue(system.partition1().write("key2", "value2"));
        assertThrows(ServiceUnavailableException.class, 
                    () -> system.partition2().write("key3", "value3"));
    } else {
        // AP system: both sides should accept writes
        assertTrue(system.partition1().write("key2", "value2"));
        assertTrue(system.partition2().write("key3", "value3"));
        
        // But reads might return different values
        assertPossiblyDifferent(
            system.partition1().read("key1"),
            system.partition2().read("key1")
        );
    }
}
`

### 2. Latency vs Consistency Testing

`java
@Test 
public void testPacelcTradeoffs() {
    ConfigurableSystem system = new ConfigurableSystem();
    
    // Test PC: consistency over latency
    system.setConsistencyMode(ConsistencyMode.STRONG);
    
    long startTime = System.currentTimeMillis();
    system.write("key", "value");
    long pcLatency = System.currentTimeMillis() - startTime;
    
    // Test PL: latency over consistency
    system.setConsistencyMode(ConsistencyMode.EVENTUAL);
    
    startTime = System.currentTimeMillis();
    system.write("key", "value2");
    long plLatency = System.currentTimeMillis() - startTime;
    
    // PL should be significantly faster
    assertTrue(plLatency < pcLatency / 2);
    
    // But PC should provide stronger guarantees
    assertEquals("value2", system.strongRead("key"));
    // eventualRead might return stale data
}
`

## Checklist CAP/PACELC

!!! checklist "Diseño con CAP/PACELC"
    
    **Análisis de Requerimientos:**
    
    - [ ] Identificar operaciones que requieren strong consistency
    - [ ] Evaluar tolerancia a downtime vs datos stale
    - [ ] Definir SLAs de latencia por tipo de operación
    - [ ] Considerar patrones de fallo (particiones vs nodos caídos)
    
    **Decisiones de Arquitectura:**
    
    - [ ] Elegir CP, AP, o híbrido por subsistema
    - [ ] Definir estrategia PACELC para operación normal
    - [ ] Configurar quorum sizes y timeouts
    - [ ] Planificar degradation graceful
    
    **Implementación:**
    
    - [ ] Detectar particiones de red
    - [ ] Implementar consistency levels configurables
    - [ ] Añadir circuit breakers por CAP choice
    - [ ] Métricas de consistency vs latency
    
    **Operación:**
    
    - [ ] Monitoring de particiones y quorum health
    - [ ] Alertas por degradación de consistency
    - [ ] Runbooks para scenarios de partición
    - [ ] Tuning de parámetros según observabilidad

---

!!! success "Siguiente Tema"
    
    Continúa con **[Tiempo y Relojes](relojes.md)** para entender cómo medir y ordenar eventos en sistemas distribuidos sin un reloj global confiable.
