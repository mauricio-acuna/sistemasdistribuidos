---
title: "Glosario de Términos"
description: "Definiciones precisas de conceptos clave en sistemas distribuidos y concurrencia"
nav_order: 100
---

# Glosario de Términos

## A

**Atomicidad**
: Propiedad que garantiza que una operación se ejecuta completamente o no se ejecuta en absoluto. No hay estados intermedios visibles.

**Actor Model**
: Modelo de concurrencia donde "actores" son unidades computacionales que se comunican únicamente por paso de mensajes.

**Availability (Disponibilidad)**
: En CAP, garantía de que el sistema responde a todas las peticiones (exitosas o fallidas) en tiempo finito.

**Async/Await**
: Patrón de programación que permite escribir código asíncrono de forma secuencial y legible.

## B

**Backpressure**
: Mecanismo de control de flujo que evita saturación propagando señales de límite hacia los productores upstream.

**Bounded Queue**
: Cola con capacidad máxima fija. Cuando está llena, las nuevas inserciones fallan o bloquean según la política configurada.

**Byzantine Fault**
: Fallo donde un nodo puede comportarse arbitrariamente (maliciosamente), incluyendo envío de mensajes contradictorios.

**Bulkhead Pattern**
: Patrón de aislamiento que separa recursos críticos en "compartimentos" independientes para contener fallos.

## C

**CAP Theorem**
: Teorema que establece que en presencia de particiones de red, debes elegir entre Consistency y Availability.

**CAS (Compare-And-Swap)**
: Operación atómica que actualiza un valor solo si coincide con un valor esperado. Base de algoritmos lock-free.

**Causal Consistency**
: Modelo de consistencia donde operaciones causalmente relacionadas son vistas en el mismo orden por todos los nodos.

**Circuit Breaker**
: Patrón que "abre" automáticamente cuando detecta fallos en un servicio downstream, previniendo cascadas de fallo.

**Consensus**
: Problema fundamental de lograr acuerdo entre múltiples nodos en presencia de fallos.

**Consistency**
: En CAP, garantía de que todos los nodos ven los mismos datos al mismo tiempo.

**CRDT (Conflict-free Replicated Data Type)**
: Estructura de datos que puede ser replicada y actualizada concurrentemente sin coordinación, convergiendo automáticamente.

## D

**Deadlock**
: Situación donde dos o más threads se bloquean mutuamente esperando recursos que el otro posee.

**Disruptor Pattern**
: Patrón de ultra-baja latencia que usa ring buffers y evita locks para máximo throughput.

**Distributed Tracing**
: Técnica para rastrear requests a través de múltiples servicios en sistemas distribuidos.

## E

**Eventual Consistency**
: Modelo de consistencia donde el sistema converge a un estado consistente si no hay más actualizaciones.

**Event Sourcing**
: Patrón donde el estado se deriva de una secuencia inmutable de eventos en lugar de almacenar estado actual.

## F

**Failure Detector**
: Componente que monitora la salud de otros nodos y reporta sospechas de fallo.

**False Sharing**
: Problema de performance donde threads acceden a diferentes variables que están en la misma cache line.

**Fencing Token**
: Token único creciente usado para prevenir split-brain scenarios garantizando orden de operaciones.

**FLP Impossibility**
: Teorema que establece la imposibilidad de consenso determinístico en sistemas asíncronos con un nodo fallido.

## G

**Gossip Protocol**
: Protocolo de comunicación peer-to-peer donde nodos intercambian información de forma epidémica.

**Grace Period**
: Tiempo de espera antes de considerar definitivamente que un nodo ha fallado.

## H

**Happens-Before**
: Relación de orden parcial entre eventos en sistemas distribuidos que captura causalidad potencial.

**Hazard Pointer**
: Técnica de gestión de memoria para estructuras lock-free que protege objetos de ser liberados prematuramente.

**Heartbeat**
: Mensaje periódico enviado para indicar que un nodo está vivo y funcionando.

**HLC (Hybrid Logical Clock)**
: Reloj que combina tiempo físico y lógico para mantener orden causal con aproximación al tiempo real.

## I

**Idempotencia**
: Propiedad donde aplicar una operación múltiples veces produce el mismo resultado que aplicarla una vez.

**Isolation**
: Propiedad ACID que garantiza que operaciones concurrentes no interfieren entre sí.

## J

**Jitter**
: Variación aleatoria añadida a timeouts para evitar efectos thundering herd.

## L

**Lamport Clock**
: Reloj lógico que asigna timestamps para mantener orden causal entre eventos.

**Leader Election**
: Proceso de seleccionar un nodo coordinador en un sistema distribuido.

**Linearizability**
: Modelo de consistencia más fuerte donde operaciones aparecen atómicas y en orden tiempo real.

**Liveness**
: Propiedad que garantiza que algo bueno eventualmente ocurrirá (progreso).

**Lock-Free**
: Algoritmo que garantiza progreso global del sistema sin usar locks mutuamente excluyentes.

## M

**MPMC (Multi-Producer Multi-Consumer)**
: Cola que permite múltiples threads productores y consumidores concurrentemente.

**MTTR (Mean Time To Recovery)**
: Tiempo promedio para recuperarse de un fallo.

## P

**PACELC**
: Extensión de CAP que considera trade-offs entre Latency y Consistency incluso sin particiones.

**Partition Tolerance**
: En CAP, capacidad del sistema de continuar funcionando a pesar de pérdidas de mensajes.

**Paxos**
: Familia de algoritmos de consenso que garantizan acuerdo incluso con fallos de nodos.

**φ-accrual (Phi-accrual)**
: Detector de fallos probabilístico que produce valores de sospecha graduales en lugar de binarios.

## Q

**Quorum**
: Subconjunto mínimo de nodos requerido para tomar decisiones válidas (típicamente mayoría).

**Queue Depth**
: Número actual de elementos esperando en una cola.

## R

**Raft**
: Algoritmo de consenso diseñado para ser más comprensible que Paxos.

**Race Condition**
: Situación donde el resultado depende del timing relativo de eventos concurrentes.

**Read-Your-Writes**
: Garantía de consistencia donde un cliente siempre ve sus propias escrituras.

**Replication Lag**
: Retraso entre una escritura en el nodo primario y su aparición en réplicas.

## S

**Safety**
: Propiedad que garantiza que algo malo nunca ocurrirá.

**SEDA (Staged Event-Driven Architecture)**
: Arquitectura que descompone servicios en stages conectados por queues.

**Sequential Consistency**
: Modelo donde operaciones aparecen en algún orden secuencial total respetando orden de programa.

**Split-Brain**
: Scenario donde múltiples nodos creen ser el líder simultáneamente debido a particiones de red.

**State Machine Replication**
: Técnica donde múltiples réplicas aplican las mismas operaciones en el mismo orden.

## T

**Thread Pool**
: Conjunto de threads reutilizables para ejecutar tareas, evitando overhead de creación/destrucción.

**Throughput**
: Número de operaciones completadas por unidad de tiempo.

**TLA+ (Temporal Logic of Actions)**
: Lenguaje de especificación formal para sistemas concurrentes y distribuidos.

**Two-Phase Commit (2PC)**
: Protocolo de consenso para transacciones distribuidas con prepare y commit phases.

## V

**Vector Clock**
: Estructura de datos que captura relaciones happens-before entre eventos en sistemas distribuidos.

**Versioning**
: Técnica para manejar actualizaciones concurrentes asignando versiones a datos.

## W

**Wait-Free**
: Algoritmo que garantiza que cada thread completa operaciones en un número finito de pasos.

**Write Amplification**
: Fenómeno donde una escritura lógica resulta en múltiples escrituras físicas.

---

## Acrónimos Comunes

| Acrónimo | Significado | Definición |
|----------|-------------|------------|
| **ACID** | Atomicity, Consistency, Isolation, Durability | Propiedades de transacciones |
| **BASE** | Basically Available, Soft state, Eventual consistency | Alternativa a ACID |
| **CAS** | Compare-And-Swap | Operación atómica fundamental |
| **CDN** | Content Delivery Network | Red de distribución de contenido |
| **CP** | Consistency + Partition tolerance | Elección en CAP theorem |
| **AP** | Availability + Partition tolerance | Elección en CAP theorem |
| **CRDT** | Conflict-free Replicated Data Type | Tipo de dato que converge automáticamente |
| **DHT** | Distributed Hash Table | Tabla hash distribuida |
| **FIFO** | First In, First Out | Orden de procesamiento |
| **HLC** | Hybrid Logical Clock | Reloj híbrido físico-lógico |
| **LIFO** | Last In, First Out | Orden de procesamiento |
| **MPMC** | Multi-Producer Multi-Consumer | Tipo de cola concurrente |
| **MTBF** | Mean Time Between Failures | Tiempo promedio entre fallos |
| **MTTR** | Mean Time To Recovery | Tiempo promedio de recuperación |
| **NUMA** | Non-Uniform Memory Access | Arquitectura de memoria |
| **RPC** | Remote Procedure Call | Llamada a procedimiento remoto |
| **SLA** | Service Level Agreement | Acuerdo de nivel de servicio |
| **SLI** | Service Level Indicator | Indicador de nivel de servicio |
| **SLO** | Service Level Objective | Objetivo de nivel de servicio |
| **SPSC** | Single-Producer Single-Consumer | Tipo de cola simple |

## Fórmulas Importantes

### Dimensionamiento de Thread Pools

`
Threads CPU-bound = # cores físicos ± 1
Threads I/O-bound = # cores × (1 + wait_time/compute_time)
`

### Little's Law

`
Latencia promedio = Throughput × Concurrencia promedio
`

### Utilización vs Latencia

`
Latencia = Tiempo_servicio / (1 - Utilización)
`

### Quorum en Sistemas de N nodos

`
Quorum = floor(N/2) + 1
`

### Error Budget (SLO)

`
Error Budget = (1 - SLO) × Total_requests
`

---

!!! tip "Contribuir al Glosario"
    
    ¿Falta algún término importante? ¿Hay definiciones que se pueden mejorar?
    
    [Contribuye al glosario](https://github.com/odin-distributed-systems/guide/edit/main/docs/referencia/glosario.md) con un pull request.

!!! note "Referencias"
    
    Las definiciones están basadas en papers académicos, documentación oficial y consenso de la industria. Ver [bibliografía](../recursos/biblioteca.md) para referencias específicas.
