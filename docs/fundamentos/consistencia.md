---
title: "Modelos de Consistencia"
description: "Linealizable, secuencial, causal y eventual: garantías, trade-offs y patrones de implementación"
nav_order: 20
tags: [consistency, linearizability, eventual, causal]
---

# Modelos de Consistencia

!!! abstract "Definición"
    
    **Consistencia** define qué **garantías** ofrece un sistema distribuido sobre el **orden** y **visibilidad** de las operaciones realizadas por diferentes clientes.

La elección del modelo de consistencia es una de las decisiones arquitecturales más importantes, con impactos directos en **performance**, **disponibilidad** y **complejidad** del sistema.

## El Espectro de Consistencia

`mermaid
graph LR
    A[Linearizable] --> B[Secuencial]
    B --> C[Causal]
    C --> D[Eventual]
    
    A -.-> E[Más fuerte<br/>Más lenta]
    D -.-> F[Más débil<br/>Más rápida]
    
    style A fill:#ff9999
    style D fill:#99ff99
`

### Analogía: El Restaurante Distribuido

Imagina un **restaurante con múltiples ubicaciones** donde los clientes pueden hacer pedidos y consultar el menú:

=== "Linearizable"

    **Como si fuera un solo restaurante**
    
    - Todos ven **exactamente** los mismos platos disponibles **al mismo tiempo**
    - Si un plato se agota en cualquier sucursal, **inmediatamente** desaparece de todos los menús
    - **Lento:** cada cambio requiere coordinación entre todas las sucursales
    
=== "Secuencial"

    **Orden global pero con retraso**
    
    - Todos ven los **mismos cambios** en el **mismo orden**
    - Pero pueden verlos en **momentos diferentes**
    - Si Plato A se agota antes que B, todos verán A→B, nunca B→A
    
=== "Causal"

    **Causa y efecto respetados**
    
    - Si el chef dice "el pollo está listo" y luego "añádanlo al menú"
    - **Nunca** verás el pollo en el menú **antes** del anuncio del chef
    - Pero cambios **independientes** pueden verse en cualquier orden
    
=== "Eventual"

    **Eventualmente todos coinciden**
    
    - Las sucursales **eventualmente** tendrán el mismo menú
    - Pero temporalmente pueden mostrar **información diferente**
    - **Rápido:** cada sucursal actualiza de forma independiente

## 1. Consistencia Linealizable

!!! success "Definición"
    
    Todas las operaciones aparecen como si fueran **atómicas** y ocurrieran en **algún punto** entre su invocación y respuesta, respetando el **orden tiempo real**.

### Propiedades

- **Atomicidad:** cada operación es instantánea
- **Orden real:** si op1 termina antes que op2 comience, op1 < op2 en el orden global
- **Visibilidad inmediata:** efectos visibles instantáneamente por todos

### Ejemplo: Contador Bancario

`java
public class LinearizableCounter {
    private final AtomicLong value = new AtomicLong(0);
    private final List<Node> replicas;
    
    public long increment() {
        // Coordinación síncrona con mayoría
        long newValue = coordinateWithMajority(current -> current + 1);
        
        // Todos los nodos deben confirmar antes de retornar
        return newValue;
    }
    
    public long get() {
        // Lectura también requiere coordinación para garantizar 
        // que vemos el valor más reciente
        return readFromMajority();
    }
    
    private long coordinateWithMajority(LongUnaryOperator operation) {
        // Two-phase commit o consenso para garantizar atomicidad
        ConsensusRequest request = new ConsensusRequest(operation);
        ConsensusResult result = raftConsensus.propose(request);
        
        return result.getValue();
    }
    
    private long readFromMajority() {
        // Read-through consensus para garantizar linearizabilidad
        List<Long> values = replicas.parallelStream()
            .map(Node::getValue)
            .collect(toList());
            
        // Verificar que mayoría tiene el mismo valor
        return findMajorityValue(values);
    }
}
`

### Test de Linearizabilidad

`java
@Test
public void testLinearizability() {
    LinearizableCounter counter = new LinearizableCounter();
    
    // Timeline:
    // T1: increment() [start=0ms, end=100ms] → result=1
    // T2: get() [start=50ms, end=60ms] → result=?
    
    CompletableFuture<Long> increment = CompletableFuture.supplyAsync(() -> {
        sleep(100); // simular latencia
        return counter.increment(); // debe retornar 1
    });
    
    sleep(50); // T2 comienza después de que T1 ya comenzó
    
    long getValue = counter.get();
    
    // VÁLIDO: T2 puede ver 0 o 1
    // INVÁLIDO: T2 ve 2 (valor que no existe aún)
    
    assertTrue(getValue == 0 || getValue == 1);
    assertEquals(1L, increment.get());
}
`

### Implementación con Raft

`java
public class LinearizableKVStore {
    private final RaftConsensusModule raft;
    private final ConcurrentHashMap<String, String> state = new ConcurrentHashMap<>();
    
    public String put(String key, String value) {
        // Proponer operación a través de Raft
        PutOperation op = new PutOperation(key, value);
        raft.propose(op).join(); // esperar a que se replique a mayoría
        
        return value;
    }
    
    public String get(String key) {
        // Opción 1: Read-through consensus (más lento pero linearizable)
        ReadOperation op = new ReadOperation(key);
        return raft.propose(op).join();
        
        // Opción 2: Local read con read-index (más rápido)
        // return readWithReadIndex(key);
    }
    
    private String readWithReadIndex(String key) {
        // Raft optimization: confirmar que somos líder actual
        long readIndex = raft.confirmLeadership();
        
        // Esperar a que nuestro log esté actualizado hasta readIndex
        raft.waitForAppliedIndex(readIndex);
        
        // Ahora la lectura local es linearizable
        return state.get(key);
    }
    
    // State machine para aplicar operaciones consensuadas
    @RaftStateMachine
    public void apply(Operation operation) {
        if (operation instanceof PutOperation) {
            PutOperation put = (PutOperation) operation;
            state.put(put.key, put.value);
        }
    }
}
`

### Costo de Linearizabilidad

| Aspecto | Impacto |
|---------|---------|
| **Latencia** | 2-3x más alta (coordinación requerida) |
| **Throughput** | Limitado por el nodo más lento |
| **Disponibilidad** | Requiere mayoría funcionando |
| **Escalabilidad** | No escala horizontalmente para escrituras |

## 2. Consistencia Secuencial

!!! info "Definición"
    
    Todas las operaciones aparecen en **algún orden secuencial total**, y cada cliente ve las operaciones en **orden de programa**.

Más relajada que linearizabilidad: **no** requiere respetar orden tiempo real.

### Ejemplo: Bulletin Board

`java
public class SequentialBulletinBoard {
    private final List<String> messages = new CopyOnWriteArrayList<>();
    private final AtomicLong timestamp = new AtomicLong(0);
    
    public void post(String message) {
        long ts = timestamp.incrementAndGet();
        MessageWithTimestamp msg = new MessageWithTimestamp(message, ts);
        
        // Difundir a todas las réplicas con timestamp
        replicas.forEach(replica -> replica.receive(msg));
    }
    
    public List<String> getMessages() {
        // Cada réplica ordena por timestamp antes de retornar
        return messages.stream()
            .sorted(Comparator.comparing(MessageWithTimestamp::getTimestamp))
            .map(MessageWithTimestamp::getMessage)
            .collect(toList());
    }
}
`

### Diferencia con Linearizabilidad

`java
// Timeline real:
// T1: post("A") [0ms-10ms]
// T2: post("B") [5ms-15ms]  
// T3: getMessages() [20ms]

// LINEARIZABLE: debe retornar ["A", "B"] porque A terminó antes que B comenzara
// SECUENCIAL: puede retornar ["A", "B"] o ["B", "A"] porque se solapan
`

## 3. Consistencia Causal

!!! info "Definición"
    
    Operaciones **causalmente relacionadas** son vistas por todos los nodos en el **mismo orden**. Operaciones **concurrentes** pueden ser vistas en **diferente orden**.

### Relación Happens-Before

Operación A → B (A causa B) si:

1. **Mismo proceso:** A y B en el mismo cliente y A antes que B
2. **Mensaje:** A es envío, B es recepción del mismo mensaje  
3. **Transitividad:** A → C y C → B, entonces A → B

### Implementación con Vector Clocks

`java
public class CausalKVStore {
    private final Map<String, VersionedValue> store = new ConcurrentHashMap<>();
    private final VectorClock vectorClock;
    private final String nodeId;
    
    public void put(String key, String value, VectorClock clientClock) {
        // Incrementar nuestro reloj y merge con cliente
        vectorClock.increment(nodeId);
        VectorClock newClock = vectorClock.merge(clientClock);
        
        VersionedValue versionedValue = new VersionedValue(value, newClock);
        store.put(key, versionedValue);
        
        // Propagar a otros nodos de forma asíncrona
        propagateAsync(key, versionedValue);
    }
    
    public VersionedValue get(String key, VectorClock clientClock) {
        VersionedValue value = store.get(key);
        
        if (value != null && value.getClock().happensBefore(clientClock)) {
            // El cliente ya vio una versión posterior causalmente
            return fetchCausallyConsistentValue(key, clientClock);
        }
        
        return value;
    }
    
    private void propagateAsync(String key, VersionedValue value) {
        // Solo enviar a nodos que no han visto esta versión
        replicas.parallelStream()
            .filter(replica -> !replica.hasVersion(value.getClock()))
            .forEach(replica -> {
                CompletableFuture.runAsync(() -> 
                    replica.receive(key, value)
                );
            });
    }
    
    public void receive(String key, VersionedValue incoming) {
        VersionedValue current = store.get(key);
        
        if (current == null || incoming.getClock().dominates(current.getClock())) {
            // Versión más nueva causalmente
            store.put(key, incoming);
        } else if (current.getClock().isConcurrentWith(incoming.getClock())) {
            // Versiones concurrentes - usar resolución de conflictos
            VersionedValue resolved = resolveConflict(current, incoming);
            store.put(key, resolved);
        }
        // Si current domina incoming, ignorar (versión más vieja)
    }
}

public class VectorClock {
    private final Map<String, Long> clock = new ConcurrentHashMap<>();
    
    public void increment(String nodeId) {
        clock.compute(nodeId, (id, current) -> (current == null ? 0 : current) + 1);
    }
    
    public VectorClock merge(VectorClock other) {
        VectorClock merged = new VectorClock();
        
        Set<String> allNodes = Sets.union(clock.keySet(), other.clock.keySet());
        
        for (String nodeId : allNodes) {
            long thisClock = clock.getOrDefault(nodeId, 0L);
            long otherClock = other.clock.getOrDefault(nodeId, 0L);
            merged.clock.put(nodeId, Math.max(thisClock, otherClock));
        }
        
        return merged;
    }
    
    public boolean happensBefore(VectorClock other) {
        return clock.entrySet().stream()
            .allMatch(entry -> {
                String nodeId = entry.getKey();
                long thisClock = entry.getValue();
                long otherClock = other.clock.getOrDefault(nodeId, 0L);
                return thisClock <= otherClock;
            }) && !equals(other);
    }
    
    public boolean isConcurrentWith(VectorClock other) {
        return !happensBefore(other) && !other.happensBefore(this);
    }
}
`

### Ejemplo de Causality

`java
// Red social con posts y comentarios
public class CausalSocialNetwork {
    
    public void timeline() {
        // Alice: publica post
        VectorClock aliceClock = new VectorClock().increment("alice");
        socialNetwork.post("alice", "Hermoso día!", aliceClock);
        
        // Bob: ve el post de Alice y comenta
        VersionedPost alicePost = socialNetwork.getPost("alice", aliceClock);
        VectorClock bobClock = new VectorClock().merge(alicePost.getClock()).increment("bob");
        socialNetwork.comment("bob", "Totalmente de acuerdo", bobClock);
        
        // GARANTÍA CAUSAL: todos los nodos verán el post de Alice 
        // ANTES del comentario de Bob (causalmente relacionados)
        
        // Charlie: comenta el post original (concurrente con Bob)
        VectorClock charlieClock = new VectorClock().merge(alicePost.getClock()).increment("charlie");
        socialNetwork.comment("charlie", "¡Sí!", charlieClock);
        
        // Los comentarios de Bob y Charlie pueden aparecer en cualquier orden
        // (son concurrentes causalmente)
    }
}
`

## 4. Consistencia Eventual

!!! warning "Definición"
    
    Si no hay más actualizaciones, **eventualmente** todos los nodos convergerán al **mismo estado**. No hay garantías de **cuándo** ocurrirá la convergencia.

### Características

- **Alta disponibilidad:** cada nodo puede servir lecturas/escrituras
- **Baja latencia:** no requiere coordinación
- **Resolución de conflictos:** necesaria para convergencia
- **AP en CAP:** prioriza availability y partition tolerance

### CRDT (Conflict-free Replicated Data Types)

`java
// G-Counter: Grow-only counter usando CRDT
public class GCounter {
    private final Map<String, Long> counters = new ConcurrentHashMap<>();
    private final String nodeId;
    
    public void increment() {
        counters.compute(nodeId, (id, current) -> (current == null ? 0 : current) + 1);
    }
    
    public long getValue() {
        return counters.values().stream().mapToLong(Long::longValue).sum();
    }
    
    public void merge(GCounter other) {
        for (Map.Entry<String, Long> entry : other.counters.entrySet()) {
            String nodeId = entry.getKey();
            long otherValue = entry.getValue();
            
            counters.compute(nodeId, (id, current) -> 
                Math.max(current == null ? 0 : current, otherValue)
            );
        }
    }
    
    // Propagación en background
    @Scheduled(fixedDelay = 1000)
    public void gossipUpdates() {
        replicas.forEach(replica -> {
            try {
                GCounter theirState = replica.getState();
                merge(theirState);
                replica.merge(this);
            } catch (Exception e) {
                // Manejo de errores de red
                logger.warn("Failed to gossip with {}", replica.getId(), e);
            }
        });
    }
}

// PN-Counter: Increment/Decrement counter
public class PNCounter {
    private final GCounter positiveCounter;
    private final GCounter negativeCounter;
    
    public PNCounter(String nodeId) {
        this.positiveCounter = new GCounter(nodeId);
        this.negativeCounter = new GCounter(nodeId);
    }
    
    public void increment() {
        positiveCounter.increment();
    }
    
    public void decrement() {
        negativeCounter.increment();
    }
    
    public long getValue() {
        return positiveCounter.getValue() - negativeCounter.getValue();
    }
    
    public void merge(PNCounter other) {
        positiveCounter.merge(other.positiveCounter);
        negativeCounter.merge(other.negativeCounter);
    }
}
`

### Resolución de Conflictos

`java
public class EventualKVStore {
    private final Map<String, VersionedValue> store = new ConcurrentHashMap<>();
    
    public void put(String key, String value) {
        VersionedValue current = store.get(key);
        long newVersion = (current == null) ? 1 : current.getVersion() + 1;
        
        VersionedValue newValue = new VersionedValue(value, newVersion, nodeId, System.currentTimeMillis());
        store.put(key, newValue);
        
        // Propagación asíncrona
        gossipToReplicas(key, newValue);
    }
    
    public String get(String key) {
        VersionedValue value = store.get(key);
        return value != null ? value.getValue() : null;
    }
    
    public void receive(String key, VersionedValue incoming) {
        store.compute(key, (k, current) -> {
            if (current == null) {
                return incoming;
            }
            
            // Estrategias de resolución de conflictos
            return resolveConflict(current, incoming);
        });
    }
    
    private VersionedValue resolveConflict(VersionedValue current, VersionedValue incoming) {
        // Estrategia 1: Last-Writer-Wins (LWW)
        if (incoming.getTimestamp() > current.getTimestamp()) {
            return incoming;
        } else if (incoming.getTimestamp() == current.getTimestamp()) {
            // Tie-breaker por nodeId
            return incoming.getNodeId().compareTo(current.getNodeId()) > 0 ? incoming : current;
        }
        return current;
        
        // Estrategia 2: Versioning con merging
        // return mergeVersions(current, incoming);
        
        // Estrategia 3: Application-specific conflict resolution
        // return applicationMerge(current, incoming);
    }
}
`

## Comparación de Modelos

### Tabla de Trade-offs

| Modelo | Latencia | Throughput | Disponibilidad | Complejidad | Uso Típico |
|--------|----------|------------|-----------------|-------------|------------|
| **Linealizable** | ⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐⭐ | Contadores bancarios, inventarios |
| **Secuencial** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | Logs de eventos, feeds ordenados |
| **Causal** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Redes sociales, colaboración |
| **Eventual** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | DNS, CDN, cachés |

### Matriz de Decisión

`mermaid
quadrantChart
    title Elección de Modelo de Consistencia
    x-axis Low Latency --> High Latency
    y-axis Weak Consistency --> Strong Consistency
    
    quadrant-1 Eventual Consistency
    quadrant-2 Linearizability  
    quadrant-3 Causal Consistency
    quadrant-4 Sequential Consistency
    
    DNS: [0.9, 0.1]
    CDN: [0.8, 0.2]
    Social Media: [0.7, 0.6]
    Banking: [0.2, 0.9]
    Inventory: [0.3, 0.8]
    Logs: [0.4, 0.7]
`

## Patrones de Implementación

### 1. Read-Your-Writes

`java
public class ReadYourWritesStore {
    private final Map<String, String> localCache = new ConcurrentHashMap<>();
    private final Set<String> writtenKeys = ConcurrentHashMap.newKeySet();
    
    public void put(String key, String value) {
        // Escribir localmente primero
        localCache.put(key, value);
        writtenKeys.add(key);
        
        // Propagar asincrónicamente
        propagateAsync(key, value);
    }
    
    public String get(String key) {
        // Si escribimos esta key, leer de cache local
        if (writtenKeys.contains(key)) {
            return localCache.get(key);
        }
        
        // Sino, leer del store distribuido
        return distributedStore.get(key);
    }
}
`

### 2. Monotonic Reads

`java
public class MonotonicReadsStore {
    private final AtomicLong readTimestamp = new AtomicLong(0);
    
    public String get(String key) {
        long minTimestamp = readTimestamp.get();
        
        // Buscar réplica con timestamp >= minTimestamp
        for (Replica replica : replicas) {
            if (replica.getTimestamp() >= minTimestamp) {
                String value = replica.get(key);
                
                // Actualizar timestamp para futuras lecturas
                readTimestamp.updateAndGet(current -> 
                    Math.max(current, replica.getTimestamp())
                );
                
                return value;
            }
        }
        
        throw new NoSufficientlyUpdatedReplicaException();
    }
}
`

### 3. Session Consistency

`java
public class SessionConsistentStore {
    private final ThreadLocal<SessionContext> sessionContext = new ThreadLocal<>();
    
    public void put(String key, String value) {
        SessionContext session = getOrCreateSession();
        
        long writeTimestamp = System.currentTimeMillis();
        session.addWrite(key, writeTimestamp);
        
        distributedStore.put(key, value, writeTimestamp);
    }
    
    public String get(String key) {
        SessionContext session = getOrCreateSession();
        
        // Verificar si tenemos writes pendientes para esta key
        Long lastWriteTimestamp = session.getLastWrite(key);
        
        if (lastWriteTimestamp != null) {
            // Asegurar que leemos una versión >= nuestro último write
            return distributedStore.get(key, lastWriteTimestamp);
        }
        
        return distributedStore.get(key);
    }
    
    private SessionContext getOrCreateSession() {
        SessionContext session = sessionContext.get();
        if (session == null) {
            session = new SessionContext();
            sessionContext.set(session);
        }
        return session;
    }
}
`

## Testing de Consistencia

### Jepsen-style Tests

`java
@Test
public void testLinearizability() {
    KVStore store = new DistributedKVStore();
    
    // Historia de operaciones concurrentes
    List<Operation> history = runConcurrentOperations(store);
    
    // Verificar que existe algún orden secuencial que respete:
    // 1. Orden de programa de cada proceso
    // 2. Orden tiempo real de operaciones no solapadas
    assertTrue(isLinearizable(history));
}

public class LinearizabilityChecker {
    public boolean isLinearizable(List<Operation> history) {
        // Generar todos los posibles órdenes lineales
        for (List<Operation> linearOrder : generateLinearOrders(history)) {
            if (isValidLinearization(linearOrder, history)) {
                return true;
            }
        }
        return false;
    }
    
    private boolean isValidLinearization(List<Operation> linearOrder, List<Operation> history) {
        // Verificar que el orden lineal es consistente con:
        // 1. Semántica de las operaciones
        // 2. Orden real de tiempo
        // 3. Orden de programa por proceso
        
        Map<String, String> state = new HashMap<>();
        
        for (Operation op : linearOrder) {
            if (!isValidTransition(state, op)) {
                return false;
            }
            applyOperation(state, op);
        }
        
        return respectsRealTimeOrder(linearOrder, history);
    }
}
`

## Checklist de Consistencia

!!! checklist "Implementación de Consistencia"
    
    **Análisis de Requerimientos:**
    
    - [ ] Identificar operaciones que requieren strong consistency
    - [ ] Evaluar tolerancia a inconsistencias temporales
    - [ ] Definir SLAs de latencia vs consistencia
    - [ ] Considerar patrones de acceso (read-heavy vs write-heavy)
    
    **Diseño:**
    
    - [ ] Elegir modelo de consistencia apropiado
    - [ ] Definir estrategias de resolución de conflictos
    - [ ] Diseñar esquemas de versionado
    - [ ] Planificar propagación de actualizaciones
    
    **Implementación:**
    
    - [ ] Implementar vector clocks o timestamps
    - [ ] Configurar quorum sizes adecuados
    - [ ] Implementar backpressure en escrituras
    - [ ] Añadir métricas de divergencia
    
    **Testing:**
    
    - [ ] Tests de consistencia bajo concurrencia
    - [ ] Validación con particiones de red
    - [ ] Benchmarks de latencia por modelo
    - [ ] Chaos testing con inconsistencias

---

!!! success "Siguiente Tema"
    
    Ahora explora **[CAP y PACELC](cap_pacelc.md)** para entender cómo estos teoremas guían las decisiones de consistencia vs disponibilidad.
