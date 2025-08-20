---
title: "Consenso en Sistemas Distribuidos"
description: "Raft, Paxos, leader election y replicación de logs para lograr acuerdo bajo fallos"
nav_order: 10
tags: [consensus, raft, paxos, leader-election]
---

# Consenso en Sistemas Distribuidos

!!! abstract "Definición"
    
    **Consenso** es el problema fundamental de lograr **acuerdo** entre múltiples nodos en un sistema distribuido, incluso en presencia de **fallos** de nodos y **particiones de red**.

El consenso es la piedra angular de los sistemas distribuidos robustos. Sin él, es imposible mantener **consistencia**, **orden de operaciones** y **estado coherente** cuando los nodos pueden fallar o desconectarse.

## El Problema Fundamental

### Escenario Típico

Imagina un sistema bancario distribuido donde múltiples réplicas deben acordar el orden de las transacciones:

`mermaid
sequenceDiagram
    participant C as Cliente
    participant A as Nodo A
    participant B as Nodo B  
    participant C as Nodo C
    
    C->>A: transfer(, Alice→Bob)
    A->>B: proponer: tx-123
    A->>C: proponer: tx-123
    
    Note over A,C: ¿Cómo garantizar que todos <br/>apliquen tx-123 en el mismo orden?
    
    B-->>A: accept tx-123
    C-->>A: accept tx-123
    A->>C: ✅ Consenso logrado
`

### Desafíos Clave

!!! danger "Fallos Posibles"
    
    - **Nodos lentos** (GC pauses, CPU saturation)
    - **Nodos caídos** (crash failures)
    - **Particiones de red** (split-brain scenarios)
    - **Mensajes perdidos** o **reordenados**
    - **Relojes desincronizados**

## Algoritmos de Consenso

### 1. Raft - Consenso Entendible

**Raft** fue diseñado específicamente para ser más fácil de entender e implementar que Paxos.

#### Componentes Principales

=== "Leader Election"

    `mermaid
    stateDiagram-v2
        [*] --> Follower
        Follower --> Candidate : timeout
        Candidate --> Leader : majority votes
        Candidate --> Follower : higher term seen
        Leader --> Follower : higher term seen
        
        state Leader {
            [*] --> SendHeartbeats
            SendHeartbeats --> ReplicateLogs
        }
    `

=== "Log Replication"

    `mermaid
    sequenceDiagram
        participant L as Leader
        participant F1 as Follower 1
        participant F2 as Follower 2
        
        L->>F1: AppendEntries(entry=X, prevIndex=5)
        L->>F2: AppendEntries(entry=X, prevIndex=5)
        
        F1-->>L: success=true
        F2-->>L: success=true
        
        Note over L: Majority achieved, commit entry X
        
        L->>F1: AppendEntries(commitIndex=6)
        L->>F2: AppendEntries(commitIndex=6)
`

=== "Safety Properties"

    **Raft garantiza:**
    
    - **Election Safety:** máximo un líder por term
    - **Leader Append-Only:** líder nunca sobrescribe su log
    - **Log Matching:** logs idénticos en índices/terms iguales
    - **Leader Completeness:** entradas comprometidas en logs futuros
    - **State Machine Safety:** misma secuencia aplicada por todos

#### Implementación de Leader Election

`java
public class RaftNode {
    private volatile NodeState state = NodeState.FOLLOWER;
    private volatile int currentTerm = 0;
    private volatile String votedFor = null;
    private long lastHeartbeat = System.currentTimeMillis();
    
    // Election timeout: 150-300ms random
    private final int electionTimeout = 150 + random.nextInt(150);
    
    public void startElection() {
        state = NodeState.CANDIDATE;
        currentTerm++;
        votedFor = nodeId;
        lastHeartbeat = System.currentTimeMillis();
        
        int votes = 1; // vote for self
        
        for (String peerId : peers) {
            RequestVoteRequest request = new RequestVoteRequest(
                currentTerm, nodeId, lastLogIndex, lastLogTerm
            );
            
            // Envío asíncrono a cada peer
            sendRequestVoteAsync(peerId, request)
                .thenAccept(response -> {
                    if (response.voteGranted) {
                        votes.incrementAndGet();
                        
                        if (votes > peers.size() / 2) {
                            becomeLeader();
                        }
                    }
                });
        }
    }
    
    private void becomeLeader() {
        if (state == NodeState.CANDIDATE) {
            state = NodeState.LEADER;
            logger.info("Became leader for term {}", currentTerm);
            
            // Enviar heartbeats inmediatamente
            sendHeartbeats();
        }
    }
}
`

#### Log Replication con Backpressure

`java
public class RaftLogReplication {
    private final BlockingQueue<LogEntry> pendingEntries;
    private final Map<String, Integer> nextIndex;
    private final Map<String, Integer> matchIndex;
    
    public CompletableFuture<Boolean> appendEntry(LogEntry entry) {
        // Backpressure: rechazar si cola llena
        if (!pendingEntries.offer(entry)) {
            return CompletableFuture.completedFuture(false);
        }
        
        // Replicar a mayoría de followers
        List<CompletableFuture<Boolean>> futures = new ArrayList<>();
        
        for (String followerId : followers) {
            CompletableFuture<Boolean> future = sendAppendEntries(followerId, entry);
            futures.add(future);
        }
        
        // Esperar mayoría
        return waitForMajority(futures);
    }
    
    private CompletableFuture<Boolean> waitForMajority(List<CompletableFuture<Boolean>> futures) {
        int requiredAcks = (followers.size() + 1) / 2; // +1 por el líder
        AtomicInteger acks = new AtomicInteger(1); // líder ya tiene el entry
        
        CompletableFuture<Boolean> result = new CompletableFuture<>();
        
        for (CompletableFuture<Boolean> future : futures) {
            future.thenAccept(success -> {
                if (success && acks.incrementAndGet() >= requiredAcks) {
                    result.complete(true);
                }
            });
        }
        
        return result;
    }
}
`

### 2. Paxos - El Algoritmo Original

**Paxos** es más general pero también más complejo de implementar correctamente.

#### Fases de Paxos

=== "Phase 1: Prepare"

    `mermaid
    sequenceDiagram
        participant P as Proposer
        participant A1 as Acceptor 1
        participant A2 as Acceptor 2
        participant A3 as Acceptor 3
        
        P->>A1: prepare(n=5)
        P->>A2: prepare(n=5)
        P->>A3: prepare(n=5)
        
        A1-->>P: promise(n=5, accepted=null)
        A2-->>P: promise(n=5, accepted=(3,"value"))
        A3-->>P: promise(n=5, accepted=null)
        
        Note over P: Received majority promises
    `

=== "Phase 2: Accept"

    `mermaid
    sequenceDiagram
        participant P as Proposer
        participant A1 as Acceptor 1
        participant A2 as Acceptor 2
        participant A3 as Acceptor 3
        
        Note over P: Use highest numbered accepted value<br/>or propose new value
        
        P->>A1: accept(n=5, value="new")
        P->>A2: accept(n=5, value="new") 
        P->>A3: accept(n=5, value="new")
        
        A1-->>P: accepted(n=5, value="new")
        A2-->>P: accepted(n=5, value="new")
        A3-->>P: rejected(higher_n=7)
        
        Note over P: Majority accepted, value chosen
    `

#### Multi-Paxos para Secuencias

`java
public class MultiPaxosInstance {
    private final Map<Integer, PaxosInstance> instances = new ConcurrentHashMap<>();
    private volatile int leaderPromiseNumber = 0;
    private volatile boolean isLeader = false;
    
    public CompletableFuture<Void> propose(int instanceId, Object value) {
        if (!isLeader) {
            return electLeader().thenCompose(v -> propose(instanceId, value));
        }
        
        PaxosInstance instance = instances.computeIfAbsent(
            instanceId, 
            id -> new PaxosInstance(id, leaderPromiseNumber)
        );
        
        return instance.propose(value);
    }
    
    private CompletableFuture<Void> electLeader() {
        int promiseNumber = generatePromiseNumber();
        
        // Phase 1: Send prepare to all acceptors
        return sendPrepareToMajority(promiseNumber)
            .thenAccept(responses -> {
                if (receivedMajorityPromises(responses)) {
                    isLeader = true;
                    leaderPromiseNumber = promiseNumber;
                }
            });
    }
}
`

### 3. Comparación de Algoritmos

| Aspecto | Raft | Paxos | Multi-Paxos |
|---------|------|-------|-------------|
| **Comprensibilidad** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| **Eficiencia** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Simplicidad implementación** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| **Flexibilidad** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Latencia steady-state** | 1 RTT | 2 RTT | 1 RTT |

## Problemas Comunes y Soluciones

### 1. Split-Brain y Fencing

!!! danger "Problema: Split-Brain"
    
    Cuando una partición de red hace que múltiples nodos se crean líderes simultáneamente.

**Solución: Quorum + Fencing Tokens**

`java
public class FencingTokenManager {
    private volatile long currentFencingToken = 0;
    
    public boolean validateLeadership(long fencingToken) {
        // Solo el token más alto puede realizar escrituras
        return fencingToken >= currentFencingToken;
    }
    
    public long grantLeadership() {
        return ++currentFencingToken;
    }
}

public class RaftLeader {
    private final FencingTokenManager fencingManager;
    private long myFencingToken;
    
    public void becomeLeader() {
        myFencingToken = fencingManager.grantLeadership();
        state = NodeState.LEADER;
    }
    
    public boolean appendEntry(LogEntry entry) {
        if (!fencingManager.validateLeadership(myFencingToken)) {
            // Perdí liderazgo, step down
            state = NodeState.FOLLOWER;
            return false;
        }
        
        return replicateToMajority(entry);
    }
}
`

### 2. Cascading Failures por GC Pauses

!!! warning "Problema"
    
    GC pauses largas pueden causar timeouts en cadena y elecciones innecesarias.

**Solución: φ-accrual Failure Detection**

`java
public class PhiAccrualFailureDetector {
    private final Deque<Long> heartbeatHistory = new ArrayDeque<>();
    private final double suspicionThreshold;
    
    public double phi(long now, long lastHeartbeat) {
        long timeSinceLastHeartbeat = now - lastHeartbeat;
        
        if (heartbeatHistory.isEmpty()) {
            return 0.0;
        }
        
        // Calcular media y desviación de intervalos históricos
        double mean = calculateMean();
        double stddev = calculateStdDev(mean);
        
        // φ value basado en distribución normal
        double phi = -Math.log10(1.0 - cumulativeDistribution(
            timeSinceLastHeartbeat, mean, stddev
        ));
        
        return phi;
    }
    
    public boolean isNodeSuspect(long now, long lastHeartbeat) {
        return phi(now, lastHeartbeat) > suspicionThreshold;
    }
    
    public void recordHeartbeat(long timestamp) {
        if (!heartbeatHistory.isEmpty()) {
            long interval = timestamp - heartbeatHistory.peekLast();
            heartbeatHistory.addLast(interval);
            
            // Mantener ventana deslizante
            if (heartbeatHistory.size() > SAMPLE_SIZE) {
                heartbeatHistory.removeFirst();
            }
        }
        heartbeatHistory.addLast(timestamp);
    }
}
`

### 3. Log Compaction y Snapshots

`java
public class RaftLogCompaction {
    private volatile long lastSnapshotIndex = 0;
    private volatile int lastSnapshotTerm = 0;
    private byte[] snapshot;
    
    public void compactLog() {
        if (log.size() - lastSnapshotIndex > COMPACTION_THRESHOLD) {
            // Crear snapshot del estado actual
            snapshot = stateMachine.createSnapshot();
            
            long newSnapshotIndex = log.getLastApplied();
            int newSnapshotTerm = log.getEntry(newSnapshotIndex).term;
            
            // Truncar log manteniendo solo entradas post-snapshot
            log.truncatePrefix(newSnapshotIndex);
            
            lastSnapshotIndex = newSnapshotIndex;
            lastSnapshotTerm = newSnapshotTerm;
            
            logger.info("Log compacted, snapshot covers up to index {}", 
                       newSnapshotIndex);
        }
    }
    
    public void handleInstallSnapshot(InstallSnapshotRequest request) {
        if (request.lastIncludedIndex <= lastSnapshotIndex) {
            // Snapshot más viejo, ignorar
            return;
        }
        
        // Aplicar snapshot y truncar log
        stateMachine.restoreFromSnapshot(request.data);
        log.truncateFrom(0);
        
        lastSnapshotIndex = request.lastIncludedIndex;
        lastSnapshotTerm = request.lastIncludedTerm;
    }
}
`

## Métricas de Consenso

### KPIs Fundamentales

`java
// Métricas de liderazgo
Counter leaderElections = Counter.build()
    .name("raft_leader_elections_total")
    .help("Total number of leader elections")
    .register();

Histogram electionDuration = Histogram.build()
    .name("raft_election_duration_seconds")
    .help("Time taken for leader elections")
    .buckets(0.1, 0.5, 1.0, 2.0, 5.0)
    .register();

// Métricas de replicación
Summary replicationLatency = Summary.build()
    .name("raft_replication_latency_seconds")
    .help("Time to replicate log entries to majority")
    .quantile(0.5, 0.05)
    .quantile(0.95, 0.01)
    .quantile(0.99, 0.001)
    .register();

Gauge logSize = Gauge.build()
    .name("raft_log_entries")
    .help("Number of entries in Raft log")
    .register();

// Métricas de health del cluster
Gauge clusterSize = Gauge.build()
    .name("raft_cluster_size")
    .help("Number of nodes in cluster")
    .register();

Gauge availableNodes = Gauge.build()
    .name("raft_available_nodes")
    .help("Number of responding nodes")
    .register();
`

### Dashboard de Consenso

`yaml
# Grafana Dashboard JSON snippet
{
  "title": "Raft Consensus Health",
  "panels": [
    {
      "title": "Leader Stability",
      "targets": [
        {
          "expr": "rate(raft_leader_elections_total[5m])",
          "legendFormat": "Elections per second"
        }
      ],
      "alert": {
        "conditions": [
          {
            "query": "A",
            "reducer": {"type": "avg"},
            "evaluator": {"params": [0.01], "type": "gt"}
          }
        ]
      }
    },
    {
      "title": "Replication Latency",
      "targets": [
        {
          "expr": "histogram_quantile(0.95, raft_replication_latency_seconds)",
          "legendFormat": "p95"
        },
        {
          "expr": "histogram_quantile(0.99, raft_replication_latency_seconds)", 
          "legendFormat": "p99"
        }
      ]
    }
  ]
}
`

## Checklist de Implementación

!!! checklist "Consenso en Producción"
    
    **Configuración:**
    
    - [ ] Election timeout con jitter (150-300ms)
    - [ ] Heartbeat interval < election timeout / 10
    - [ ] Quorum size configurado (cluster size / 2 + 1)
    - [ ] Log compaction threshold definido
    
    **Observabilidad:**
    
    - [ ] Métricas de leader elections y duración
    - [ ] Latencia de replicación (p95, p99)
    - [ ] Tamaño de log y rate de crecimiento
    - [ ] Health checks de quorum
    
    **Resilencia:**
    
    - [ ] Fencing tokens implementados
    - [ ] φ-accrual failure detection
    - [ ] Graceful shutdown con leadership transfer
    - [ ] Snapshot/restore procedures
    
    **Performance:**
    
    - [ ] Batching de log entries
    - [ ] Pipeline de AppendEntries
    - [ ] Backpressure en propuestas
    - [ ] Thread pools aislados para consensus vs aplicación

## Recursos Avanzados

### Papers de Referencia

- **[Raft Consensus Algorithm](https://raft.github.io/raft.pdf)** - Ongaro & Ousterhout
- **[Paxos Made Simple](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf)** - Leslie Lamport
- **[Viewstamped Replication](http://pmg.csail.mit.edu/papers/vr-revisited.pdf)** - Liskov & Cowling
- **[Zab: High-performance broadcast](https://web.stanford.edu/class/cs347/reading/zab.pdf)** - Junqueira et al.

### Implementaciones de Referencia

- **[Raft.js](https://github.com/kanaka/raft.js)** - Implementación visual interactiva
- **[Copycat](https://github.com/atomix/copycat)** - Raft para JVM
- **[etcd/raft](https://github.com/etcd-io/etcd/tree/main/raft)** - Implementación en Go
- **[Apache Ratis](https://ratis.apache.org/)** - Raft Java library

---

!!! success "Siguiente Tema"
    
    Ahora que dominas consenso, explora **[Modelos de Consistencia](consistencia.md)** para entender las garantías que ofrecen diferentes niveles de consistencia.
