---
title: "Tiempo y Relojes Distribuidos"
description: "Fundamentos del ordenamiento temporal en sistemas distribuidos"
nav_order: 1
---

# Tiempo y Relojes Distribuidos

!!! quote "Leslie Lamport"
    
    "La manera más fácil de entender un sistema distribuido es verlo como un conjunto de eventos conectados por relaciones de causalidad, no por tiempo absoluto."

## 🎯 Resumen Ejecutivo

**El tiempo es fundamental** para coordinar operaciones en sistemas distribuidos, pero los **relojes físicos** no pueden sincronizarse perfectamente. Los **relojes lógicos** nos permiten establecer **orden causal** entre eventos sin depender del tiempo físico.

### Conceptos Clave
- **Happens-Before**: Relación fundamental de causalidad
- **Relojes Lógicos**: Lamport timestamps y Vector clocks  
- **Relojes Híbridos**: Combinan tiempo físico y lógico
- **Sincronización**: NTP y protocolo de Berkeley

### Takeaways Principales
- El tiempo físico **no** es confiable en sistemas distribuidos
- Los relojes lógicos **capturan causalidad** entre eventos
- Vector clocks **detectan concurrencia** y ordenamiento parcial
- Hybrid Logical Clocks **aproximan tiempo real** manteniendo orden causal

---

## 🕐 El Problema del Tiempo Distribuido

### ⚠️ Limitaciones del Tiempo Físico

`java
// ❌ PROBLEMÁTICO: Confiar en System.currentTimeMillis()
public class DistributedCounter {
    private long lastUpdate = System.currentTimeMillis();
    private int value = 0;
    
    // ¡Clock skew puede causar inconsistencias!
    public boolean tryUpdate(int newValue, long timestamp) {
        if (timestamp > lastUpdate) {  // Falso positivo posible
            value = newValue;
            lastUpdate = timestamp;
            return true;
        }
        return false;
    }
}
`

### 🚨 Problemas Fundamentales

1. **Clock Skew**: Relojes locales driftan a diferentes velocidades
2. **Clock Drift**: Relojes se desalinean progresivamente  
3. **Network Delay**: Timestamps en mensajes sufren latencia variable
4. **Leap Seconds**: Ajustes de tiempo pueden causar inconsistencias

---

## 🧠 Relojes Lógicos de Lamport

### 📐 Definición Formal

Los **Lamport timestamps** asignan números enteros a eventos de forma que:

> Si evento  happens-before evento , entonces L(a) < L(b)

### 🔧 Algoritmo

`java
public class LamportClock {
    private final AtomicLong clock = new AtomicLong(0);
    
    /**
     * Incrementa el reloj para un evento local
     */
    public long tick() {
        return clock.incrementAndGet();
    }
    
    /**
     * Actualiza el reloj al recibir un mensaje
     */
    public long receive(long messageClock) {
        long currentClock = clock.get();
        long newClock = Math.max(currentClock, messageClock) + 1;
        clock.set(newClock);
        return newClock;
    }
    
    /**
     * Obtiene timestamp para enviar en un mensaje
     */
    public long send() {
        return tick();
    }
}
`

### 🎯 Ejemplo Práctico

`java
// Sistema de chat distribuido con Lamport clocks
public class DistributedChat {
    private final LamportClock clock = new LamportClock();
    private final String nodeId;
    
    public void sendMessage(String content, Set<String> recipients) {
        long timestamp = clock.send();
        
        Message msg = new Message(nodeId, content, timestamp);
        recipients.forEach(recipient -> 
            networkService.send(recipient, msg));
        
        log.info("Sent message at L-time {}: {}", timestamp, content);
    }
    
    public void receiveMessage(Message msg) {
        long localTime = clock.receive(msg.getTimestamp());
        
        // Los mensajes se ordenan por Lamport timestamp
        messageQueue.add(msg);
        messageQueue.sort(Comparator.comparing(Message::getTimestamp));
        
        log.info("Received message at L-time {}: {}", 
                localTime, msg.getContent());
    }
}
`

### ⚠️ Limitación de Lamport Clocks

`java
// Los Lamport clocks NO pueden detectar concurrencia
Event a = new Event("node1", 5);  // L(a) = 5
Event b = new Event("node2", 7);  // L(b) = 7

// ¿L(a) < L(b) significa que a → b?
// ¡NO! Podrían ser concurrentes (a || b)
`

---

## 🧭 Vector Clocks

### 📊 Definición

Los **Vector Clocks** mantienen un vector de timestamps, uno por cada nodo del sistema, permitiendo detectar **relaciones causales completas**.

### 🔧 Implementación

`java
public class VectorClock {
    private final String nodeId;
    private final Map<String, Long> clock;
    private final Set<String> knownNodes;
    
    public VectorClock(String nodeId, Set<String> nodes) {
        this.nodeId = nodeId;
        this.knownNodes = new HashSet<>(nodes);
        this.clock = new ConcurrentHashMap<>();
        
        // Inicializar todos los clocks en 0
        nodes.forEach(node -> clock.put(node, 0L));
    }
    
    /**
     * Incrementa el reloj local para un evento
     */
    public synchronized VectorClock tick() {
        clock.compute(nodeId, (k, v) -> v + 1);
        return copy();
    }
    
    /**
     * Actualiza el vector clock al recibir un mensaje
     */
    public synchronized VectorClock receive(VectorClock messageClock) {
        // Tomar el máximo para cada componente
        knownNodes.forEach(node -> {
            long localTime = clock.getOrDefault(node, 0L);
            long messageTime = messageClock.clock.getOrDefault(node, 0L);
            clock.put(node, Math.max(localTime, messageTime));
        });
        
        // Incrementar nuestro propio reloj
        return tick();
    }
    
    /**
     * Determina la relación causal entre dos vector clocks
     */
    public CausalRelation compareTo(VectorClock other) {
        boolean thisHappensBeforeOther = true;
        boolean otherHappensBeforeThis = true;
        
        for (String node : knownNodes) {
            long thisTime = this.clock.getOrDefault(node, 0L);
            long otherTime = other.clock.getOrDefault(node, 0L);
            
            if (thisTime > otherTime) {
                otherHappensBeforeThis = false;
            }
            if (otherTime > thisTime) {
                thisHappensBeforeOther = false;
            }
        }
        
        if (thisHappensBeforeOther && !otherHappensBeforeThis) {
            return CausalRelation.HAPPENS_BEFORE;
        } else if (otherHappensBeforeThis && !thisHappensBeforeOther) {
            return CausalRelation.HAPPENS_AFTER;
        } else if (thisHappensBeforeOther && otherHappensBeforeThis) {
            return CausalRelation.CONCURRENT;  // Mismo evento
        } else {
            return CausalRelation.CONCURRENT;  // Eventos concurrentes
        }
    }
}

enum CausalRelation {
    HAPPENS_BEFORE, HAPPENS_AFTER, CONCURRENT
}
`

### 🎯 Ejemplo: Sistema de Versionado Distribuido

`java
public class DistributedVersionControl {
    private final VectorClock currentVersion;
    private final Map<VectorClock, Document> versions;
    
    public void createNewVersion(Document doc) {
        VectorClock newVersion = currentVersion.tick();
        versions.put(newVersion, doc);
        
        log.info("Created version {}", newVersion);
    }
    
    public MergeResult merge(VectorClock otherVersion) {
        CausalRelation relation = currentVersion.compareTo(otherVersion);
        
        switch (relation) {
            case HAPPENS_BEFORE:
                // Fast-forward merge
                return MergeResult.fastForward(otherVersion);
                
            case HAPPENS_AFTER:
                // Ya tenemos la versión más reciente
                return MergeResult.noAction();
                
            case CONCURRENT:
                // Merge conflict - requiere resolución manual
                return MergeResult.conflict(currentVersion, otherVersion);
                
            default:
                throw new IllegalStateException("Unexpected relation: " + relation);
        }
    }
}
`

---

## ⚡ Hybrid Logical Clocks (HLC)

### 🎯 Mejor de Ambos Mundos

Los **HLC** combinan tiempo físico (para aproximar tiempo real) con tiempo lógico (para mantener causalidad).

### 📐 Algoritmo HLC

`java
public class HybridLogicalClock {
    private final AtomicLong logicalTime = new AtomicLong(0);
    private final Clock physicalClock;
    
    public HybridLogicalClock() {
        this.physicalClock = Clock.systemUTC();
    }
    
    /**
     * Genera un timestamp HLC para un evento local
     */
    public synchronized HLCTimestamp now() {
        long physicalNow = physicalClock.millis();
        long logical = logicalTime.get();
        
        if (physicalNow > logical) {
            // El tiempo físico avanzó - usar y resetear lógico
            logicalTime.set(physicalNow);
            return new HLCTimestamp(physicalNow, 0);
        } else {
            // Usar tiempo lógico incrementado
            long newLogical = logicalTime.incrementAndGet();
            return new HLCTimestamp(logical, newLogical - logical);
        }
    }
    
    /**
     * Actualiza HLC al recibir un timestamp externo
     */
    public synchronized HLCTimestamp receive(HLCTimestamp remote) {
        long physicalNow = physicalClock.millis();
        long logical = logicalTime.get();
        
        long maxPhysical = Math.max(physicalNow, remote.physical);
        
        if (maxPhysical == logical) {
            // Mismo tiempo lógico - incrementar contador
            logicalTime.incrementAndGet();
            return new HLCTimestamp(logical, 
                Math.max(0, remote.logical) + 1);
        } else {
            // Nuevo tiempo lógico
            logicalTime.set(maxPhysical);
            return new HLCTimestamp(maxPhysical, 0);
        }
    }
}

public class HLCTimestamp implements Comparable<HLCTimestamp> {
    public final long physical;  // Componente de tiempo físico
    public final long logical;   // Componente de tiempo lógico
    
    public HLCTimestamp(long physical, long logical) {
        this.physical = physical;
        this.logical = logical;
    }
    
    @Override
    public int compareTo(HLCTimestamp other) {
        int physicalComp = Long.compare(this.physical, other.physical);
        if (physicalComp != 0) return physicalComp;
        
        return Long.compare(this.logical, other.logical);
    }
    
    @Override
    public String toString() {
        return String.format("HLC(%d, %d)", physical, logical);
    }
}
`

### 🌟 Ventajas de HLC

1. **Aproxima tiempo real**: Timestamps cercanos al tiempo físico
2. **Mantiene causalidad**: Garantiza happens-before ordering
3. **Compacto**: Solo 64-96 bits vs. N*64 bits de vector clocks
4. **Compatible**: Se integra fácilmente con sistemas existentes

---

## 🔧 Sincronización de Relojes Físicos

### 📡 Network Time Protocol (NTP)

`java
public class NTPClient {
    private static final int NTP_PORT = 123;
    private final SocketAddress ntpServer;
    
    public ClockSyncResult synchronize() throws IOException {
        // Enviar request NTP
        long t1 = System.currentTimeMillis();  // Tiempo local de envío
        
        byte[] request = createNTPRequest();
        DatagramSocket socket = new DatagramSocket();
        socket.send(new DatagramPacket(request, request.length, ntpServer));
        
        // Recibir response
        byte[] response = new byte[48];
        DatagramPacket packet = new DatagramPacket(response, response.length);
        socket.receive(packet);
        
        long t4 = System.currentTimeMillis();  // Tiempo local de recepción
        
        // Extraer timestamps del servidor
        long t2 = extractServerReceiveTime(response);  // Servidor recibió
        long t3 = extractServerTransmitTime(response); // Servidor envió
        
        // Calcular offset y delay
        long offset = ((t2 - t1) + (t3 - t4)) / 2;
        long delay = (t4 - t1) - (t3 - t2);
        
        return new ClockSyncResult(offset, delay);
    }
}
`

### 🔄 Algoritmo de Berkeley

`java
public class BerkeleyTimeSync {
    private final List<ClockService> slaves;
    
    /**
     * Sincroniza relojes usando el algoritmo de Berkeley
     */
    public void synchronizeClocks() {
        // 1. Recopilar tiempos de todos los nodos
        List<Long> times = new ArrayList<>();
        times.add(System.currentTimeMillis());  // Master time
        
        for (ClockService slave : slaves) {
            try {
                long slaveTime = slave.getCurrentTime();
                times.add(slaveTime);
            } catch (Exception e) {
                log.warn("Failed to get time from slave: {}", e.getMessage());
            }
        }
        
        // 2. Calcular tiempo promedio (excluyendo outliers)
        long averageTime = calculateRobustAverage(times);
        
        // 3. Enviar ajustes a cada esclavo
        long masterAdjustment = averageTime - System.currentTimeMillis();
        adjustLocalClock(masterAdjustment);
        
        for (ClockService slave : slaves) {
            try {
                long slaveCurrentTime = slave.getCurrentTime();
                long slaveAdjustment = averageTime - slaveCurrentTime;
                slave.adjustClock(slaveAdjustment);
            } catch (Exception e) {
                log.error("Failed to adjust slave clock: {}", e.getMessage());
            }
        }
    }
    
    private long calculateRobustAverage(List<Long> times) {
        // Eliminar outliers usando IQR
        times.sort(Long::compareTo);
        int n = times.size();
        
        if (n < 3) return times.stream().mapToLong(Long::longValue).sum() / n;
        
        int q1Index = n / 4;
        int q3Index = 3 * n / 4;
        long iqr = times.get(q3Index) - times.get(q1Index);
        long lowerBound = times.get(q1Index) - (long)(1.5 * iqr);
        long upperBound = times.get(q3Index) + (long)(1.5 * iqr);
        
        return times.stream()
                .filter(t -> t >= lowerBound && t <= upperBound)
                .mapToLong(Long::longValue)
                .sum() / times.size();
    }
}
`

---

## 🧪 Testing Temporal

### ⚡ Test de Concurrencia con jcstress

`java
@JCStressTest
@Outcome(id = "BEFORE", expect = Expect.ACCEPTABLE, desc = "A happens before B")
@Outcome(id = "AFTER", expect = Expect.ACCEPTABLE, desc = "B happens before A")
@Outcome(id = "CONCURRENT", expect = Expect.ACCEPTABLE, desc = "A and B concurrent")
public class VectorClockConcurrencyTest {
    
    private final VectorClock clockA = new VectorClock("A", Set.of("A", "B"));
    private final VectorClock clockB = new VectorClock("B", Set.of("A", "B"));
    
    @Actor
    public void actorA(R_Result r) {
        VectorClock eventA = clockA.tick();
        r.r1 = eventA.toString();
    }
    
    @Actor
    public void actorB(R_Result r) {
        VectorClock eventB = clockB.tick();
        r.r2 = eventB.toString();
    }
    
    @Arbiter
    public void analyze(R_Result r, S_Result result) {
        VectorClock vcA = VectorClock.parse(r.r1);
        VectorClock vcB = VectorClock.parse(r.r2);
        
        result.r1 = vcA.compareTo(vcB).toString();
    }
}
`

### 🔬 Property-Based Testing

`java
@Property
public void lamportClockMonotonicity(
    @ForAll("events") List<String> events) {
    
    LamportClock clock = new LamportClock();
    List<Long> timestamps = new ArrayList<>();
    
    // Procesar eventos
    for (String event : events) {
        if (event.startsWith("send")) {
            timestamps.add(clock.send());
        } else if (event.startsWith("tick")) {
            timestamps.add(clock.tick());
        }
    }
    
    // Verificar monotonicidad
    for (int i = 1; i < timestamps.size(); i++) {
        assertThat(timestamps.get(i))
            .isGreaterThan(timestamps.get(i - 1));
    }
}
`

---

## 📚 Referencias y Recursos

### 📖 Papers Fundamentales

1. **Lamport, L.** (1978). *Time, clocks, and the ordering of events in a distributed system*. Communications of the ACM. [📄 PDF](https://lamport.azurewebsites.net/pubs/time-clocks.pdf)

2. **Fidge, C.** (1988). *Timestamps in message-passing systems that preserve the partial ordering*. Australian Computer Science Communications. [📄 PDF](https://fileadmin.cs.lth.se/cs/Personal/Amr_Ergawy/dist-algos-papers/4.pdf)

3. **Kulkarni, S. S., Demirbas, M., Madappa, D., Avva, B., & Leone, M.** (2014). *Logical physical clocks*. International Conference on Principles of Distributed Systems. [📄 PDF](https://cse.buffalo.edu/tech-reports/2014-04.pdf)

### 🛠️ Implementaciones de Referencia

- [**TrueTime**](https://cloud.google.com/spanner/docs/true-time-external-consistency) - Google Spanner's global clock
- [**HLC**](https://github.com/crossbeam-rs/crossbeam/tree/master/crossbeam-utils/src) - Rust implementation
- [**Vector Clocks**](https://github.com/ricardobcl/Dotted-Version-Vectors) - Dotted Version Vectors

### 🔧 Herramientas Prácticas

- **chrony**: Implementación NTP moderna y precisa
- **PTP**: Precision Time Protocol para redes locales
- **TimescaleDB**: Base de datos optimizada para series temporales

---

!!! tip "Próximo Paso"
    
    Ahora que entiendes los fundamentos del tiempo distribuido, explora cómo se aplican en [**Detección de Fallos**](deteccion-fallos.md) para construir sistemas resilientes.

!!! warning "Consideración Práctica"
    
    En producción, siempre combina **múltiples técnicas**: NTP para tiempo físico aproximado, HLC para ordenamiento causal, y timeouts adaptativos para robustez ante variabilidad de red.
