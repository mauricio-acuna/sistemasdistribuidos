---
title: "Fundamentos de Sistemas Distribuidos"
description: "Conceptos esenciales: consenso, consistencia, CAP/PACELC, tiempo y detección de fallos"
nav_order: 1
---

# Fundamentos de Sistemas Distribuidos

Los sistemas distribuidos presentan desafíos únicos que no existen en sistemas centralizados. Esta sección cubre los conceptos teóricos fundamentales que todo ingeniero debe dominar para construir sistemas robustos y escalables.

## Mapa Conceptual

`mermaid
mindmap
  root((Sistemas Distribuidos))
    Consenso
      Raft
      Paxos
      Leader Election
      Log Replication
    Consistencia
      Linealizable
      Secuencial
      Causal
      Eventual
    Tiempo
      Lamport Clocks
      Vector Clocks
      HLC
      NTP Limitations
    Fallos
      Detección
      φ-accrual
      Heartbeats
      Timeouts
    Teoremas
      CAP
      PACELC
      FLP
`

## Temas Principales

### :material-vote: [Consenso](consenso.md)

El problema fundamental de lograr acuerdo en sistemas distribuidos bajo fallos.

- **Algoritmos:** Raft, Paxos, ZAB, Viewstamped Replication
- **Componentes:** Leader election, log replication, commit index
- **Garantías:** Único líder, orden total, durabilidad

### :material-database-sync: [Consistencia](consistencia.md)

Modelos de consistencia y sus trade-offs en términos de performance y disponibilidad.

- **Modelos:** Linealizable, secuencial, causal, eventual
- **Patrones:** Read-your-writes, monotonic reads/writes
- **Implementación:** Quorum systems, versioning

### :material-scale-balance: [CAP y PACELC](cap_pacelc.md)

Teoremas fundamentales que guían decisiones de diseño arquitectural.

- **CAP:** Consistency, Availability, Partition tolerance
- **PACELC:** Extension considering Latency vs Consistency
- **Decisiones:** CP vs AP bajo particiones

### :material-clock-outline: [Tiempo y Relojes](relojes.md)

Ordenamiento de eventos y sincronización temporal en sistemas distribuidos.

- **Problemas:** Relatividad, drift, sincronización
- **Soluciones:** Lamport clocks, vector clocks, HLC
- **Aplicaciones:** Causality, snapshots, ordering

### :material-heart-pulse: [Detección de Fallos](fallos.md)

Mecanismos para identificar y manejar fallos de nodos y red.

- **Técnicas:** Heartbeats, timeouts, φ-accrual
- **Challenges:** False positives, split-brain
- **Mitigación:** Jitter, backoff, fencing

## Prerrequisitos

!!! note "Conocimientos Previos"
    
    - Programación concurrente básica
    - Conceptos de red (latencia, throughput, particiones)
    - Sistemas operativos (threads, memoria, I/O)
    - Bases de datos (ACID, transacciones)

## Ruta de Aprendizaje Sugerida

`mermaid
flowchart TD
    A[Tiempo y Relojes] --> B[Detección de Fallos]
    B --> C[Consenso]
    C --> D[Consistencia]
    D --> E[CAP y PACELC]
    E --> F[Aplicación Práctica]
    
    style A fill:#e1f5fe
    style F fill:#e8f5e8
`

1. **[Tiempo y Relojes](relojes.md)** - Base para entender ordenamiento
2. **[Detección de Fallos](fallos.md)** - Prerrequisito para consenso
3. **[Consenso](consenso.md)** - Algoritmos fundamentales
4. **[Consistencia](consistencia.md)** - Modelos y garantías
5. **[CAP y PACELC](cap_pacelc.md)** - Trade-offs y decisiones

## Laboratorios Prácticos

!!! lab "Ejercicios Recomendados"
    
    **MIT 6.824/6.5840 Labs:**
    
    - [ ] **Lab 1:** MapReduce
    - [ ] **Lab 2:** Raft leader election
    - [ ] **Lab 3:** Raft log replication  
    - [ ] **Lab 4:** Fault-tolerant key/value service
    
    **TLA+ Models:**
    
    - [ ] **Two-phase commit** specification
    - [ ] **Raft consensus** with invariants
    - [ ] **Logical clocks** ordering properties

## Recursos Adicionales

### Papers Fundamentales

- **Lamport (1978)** - [Time, Clocks, and the Ordering of Events](../recursos/biblioteca.md#lamport-time)
- **Fischer, Lynch, Paterson (1985)** - [Impossibility of Distributed Consensus](../recursos/biblioteca.md#flp)
- **Lamport (2001)** - [Paxos Made Simple](../recursos/biblioteca.md#paxos-simple)
- **Ongaro & Ousterhout (2014)** - [In Search of an Understandable Consensus Algorithm](../recursos/biblioteca.md#raft)

### Cursos y Videos

- **MIT 6.824** - Distributed Systems ([videos](../recursos/cursos.md#mit-6824))
- **Martin Kleppmann** - Distributed Systems lectures ([Cambridge](../recursos/cursos.md#kleppmann))
- **TLA+ Video Course** - Leslie Lamport ([YouTube](../recursos/cursos.md#tla-course))

### Herramientas de Validación

- **[TLA+](../recursos/herramientas.md#tla-plus)** - Especificación y verificación formal
- **[Jepsen](../recursos/herramientas.md#jepsen)** - Testing de consistencia
- **[Chaos Engineering](../pruebas/chaos.md)** - Simulación de fallos

---

!!! success "Siguiente Paso"
    
    Comenzar con **[Tiempo y Relojes](relojes.md)** para establecer los fundamentos de ordenamiento temporal en sistemas distribuidos.
