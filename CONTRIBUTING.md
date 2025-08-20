# 🤝 Guía de Contribución - ODIN

¡Gracias por tu interés en contribuir a ODIN! Esta guía te ayudará a hacer contribuciones valiosas y efectivas.

## 🎯 Filosofía de Contribución

ODIN aspira a ser **la referencia definitiva** para sistemas distribuidos, combinando:

- **Rigor Académico**: Fundamentos teóricos sólidos con referencias apropiadas
- **Pragmatismo Industrial**: Soluciones que funcionan en producción
- **Claridad Pedagógica**: Explicaciones graduales con analogías efectivas
- **Código Funcional**: Implementaciones completas y testeadas

## 📋 Tipos de Contribuciones

### 📚 Contenido Técnico

#### ✍️ Nuevos Artículos
- **Papers Implementation**: Implementar algoritmos de papers académicos
- **Case Studies**: Análisis de sistemas reales (Kafka, Cassandra, etc.)
- **Deep Dives**: Exploración profunda de conceptos específicos
- **Troubleshooting Guides**: Soluciones a problemas comunes

#### 🔧 Mejoras de Contenido Existente
- Añadir ejemplos de código
- Mejorar explicaciones complejas
- Actualizar referencias y links
- Corregir errores técnicos

### 💻 Código y Ejemplos

#### 🚀 Implementaciones
- Algoritmos de consenso (Raft, Paxos, PBFT)
- Estructuras de datos lock-free
- Patrones de concurrencia
- Herramientas de testing

#### 📊 Benchmarks
- Performance comparisons
- Throughput/latency measurements
- Memory usage analysis
- Scaling characteristics

### 🧪 Testing y Verificación

#### ⚡ Tests de Concurrencia
- jcstress stress tests
- Property-based testing
- Chaos engineering scenarios
- Load testing suites

#### 🔬 Verificación Formal
- TLA+ specifications
- Model checking
- Invariant verification
- Safety/liveness proofs

## 🔄 Flujo de Contribución

### 1. Preparación

`ash
# Fork del repositorio
git clone https://github.com/tu-usuario/guide.git
cd guide

# Configurar upstream
git remote add upstream https://github.com/odin-distributed-systems/guide.git

# Instalar dependencias
pip install -r requirements.txt
npm install
`

### 2. Desarrollo

`ash
# Crear branch feature
git checkout -b feature/nueva-funcionalidad

# Trabajar en changes...
# Ejecutar servidor local para preview
mkdocs serve

# Ejecutar tests
npm test
python -m pytest tests/
`

### 3. Submit

`ash
# Commit con mensaje descriptivo
git add .
git commit -m "feat(consenso): añadir implementación Multi-Paxos

- Implementar algoritmo Multi-Paxos completo
- Añadir tests de correctness con jcstress  
- Incluir benchmark vs Raft
- Documentar trade-offs de performance"

# Push y crear PR
git push origin feature/nueva-funcionalidad
`

## 📝 Standards de Contenido

### 📖 Estructura de Artículos

`markdown
---
title: "Título Descriptivo"
description: "Resumen de 1-2 líneas"
nav_order: 10
---

# Título Principal

## Resumen Ejecutivo
- Problema que resuelve
- Conceptos clave 
- Takeaways principales

## Contexto y Motivación
- ¿Por qué es importante?
- Problemas que resuelve
- Limitaciones de enfoques anteriores

## Fundamentos Teóricos
- Definiciones formales
- Propiedades y garantías
- Trade-offs fundamentales

## Implementación Práctica
- Código funcional y comentado
- Casos de uso reales
- Consideraciones de performance

## Testing y Verificación
- Tests de correctness
- Benchmarks de performance  
- Herramientas de verificación

## Referencias y Lecturas
- Papers académicos originales
- Implementaciones de referencia
- Recursos adicionales
`

### 💻 Standards de Código

#### ☕ Java
`java
/**
 * Implementación del algoritmo Raft leader election.
 * 
 * Basado en: "In Search of an Understandable Consensus Algorithm"
 * por Ongaro & Ousterhout (2014)
 */
public class RaftNode {
    // Usar logging estructurado
    private static final Logger log = LoggerFactory.getLogger(RaftNode.class);
    
    // Documentar invariantes importantes
    private final AtomicLong currentTerm = new AtomicLong(0);  // monotonic
    
    /**
     * Inicia election timeout aleatorio para prevenir split votes.
     * 
     * @param baseTimeout timeout base en ms
     * @param jitter factor de randomización [0.1, 0.5]
     */
    public void startElectionTimeout(long baseTimeout, double jitter) {
        // Implementation...
    }
}
`

#### 🔧 Go
`go
// Package raft implementa el algoritmo de consenso Raft
// según la especificación de Ongaro & Ousterhout.
package raft

import (
    "context"
    "sync"
    "time"
    
    "go.uber.org/zap"
)

// RaftNode representa un nodo individual en el cluster Raft
type RaftNode struct {
    mu          sync.RWMutex
    currentTerm uint64        // must be monotonic
    votedFor    *NodeID       // nil if none
    
    logger      *zap.Logger
}
`

### 📊 Benchmarks

`java
@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.SECONDS)
@State(Scope.Benchmark)
public class ConsensusBenchmark {
    
    @Benchmark
    public void raftConsensus(RaftState state) {
        // Realistic workload simulation
        state.raftCluster.propose(state.nextCommand());
    }
    
    @Benchmark  
    public void paxosConsensus(PaxosState state) {
        // Equivalent workload for comparison
        state.paxosCluster.propose(state.nextCommand());
    }
}
`

## 🧪 Testing Guidelines

### ⚡ Concurrency Testing

`java
@JCStressTest
@Outcome(id = "1, 2", expect = Expect.ACCEPTABLE, desc = "Sequential execution")
@Outcome(id = "2, 1", expect = Expect.ACCEPTABLE, desc = "Sequential execution")  
@Outcome(expect = Expect.FORBIDDEN, desc = "Other values forbidden")
public class LockFreeQueueTest {
    
    private final LockFreeQueue<Integer> queue = new LockFreeQueue<>();
    
    @Actor
    public void actor1(II_Result r) {
        queue.offer(1);
        r.r1 = queue.poll();
    }
    
    @Actor
    public void actor2(II_Result r) {
        queue.offer(2);
        r.r2 = queue.poll();
    }
}
`

### 🔬 Property-Based Testing

`java
@Property
public void queueFIFOProperty(@ForAll List<@IntRange(min=1, max=1000) Integer> items) {
    LockFreeQueue<Integer> queue = new LockFreeQueue<>();
    
    // Enqueue all items
    items.forEach(queue::offer);
    
    // Dequeue should maintain FIFO order
    List<Integer> dequeued = new ArrayList<>();
    Integer item;
    while ((item = queue.poll()) != null) {
        dequeued.add(item);
    }
    
    assertThat(dequeued).isEqualTo(items);
}
`

## 📚 Referencias y Citations

### 📝 Formato de Referencias

`markdown
## Referencias

1. **Lamport, L.** (1978). *Time, clocks, and the ordering of events in a distributed system*. Communications of the ACM, 21(7), 558-565. [DOI](https://doi.org/10.1145/359545.359563)

2. **Ongaro, D., & Ousterhout, J.** (2014). *In search of an understandable consensus algorithm*. 2014 USENIX Annual Technical Conference (USENIX ATC 14), 305-319. [PDF](https://raft.github.io/raft.pdf)

3. **Kleppmann, M.** (2017). *Designing Data-Intensive Applications*. O'Reilly Media. ISBN: 978-1449373320.
`

### 🔗 Enlaces en Línea

`markdown
!!! note "Paper Original"
    
    Para profundizar en los fundamentos teóricos, consulta el paper original:
    [*Impossibility of Distributed Consensus with One Faulty Process*](https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf) por Fischer, Lynch & Paterson (1985).

!!! tip "Implementación de Referencia"
    
    Ver implementación oficial de Raft: [hashicorp/raft](https://github.com/hashicorp/raft)
`

## 🎨 Style Guide

### 📖 Escritura Técnica

#### ✅ Recomendado
- **Claridad**: "El algoritmo Raft divide el consenso en líder election, log replication y safety"
- **Precisión**: "Raft garantiza linearizability bajo particiones minoritarias"
- **Contexto**: "A diferencia de Paxos, Raft está diseñado para ser comprensible"

#### ❌ Evitar
- Vaguedad: "Raft es mejor que otros algoritmos"
- Jargon sin explicar: "Utiliza φ-accrual failure detection"
- Afirmaciones sin respaldo: "Este es el algoritmo más rápido"

### 🖼️ Diagramas y Visualizaciones

`python
# Usar herramientas estándar para consistency
import matplotlib.pyplot as plt
import seaborn as sns

# Configuración consistente
plt.style.use('seaborn-whitegrid')
sns.set_palette("husl")

# Títulos descriptivos
plt.title('Raft vs Paxos: Latencia de Consenso (95th percentile)')
plt.xlabel('Cluster Size (nodes)')
plt.ylabel('Latency (ms)')
`

## 🏆 Proceso de Review

### 📋 Checklist para PRs

#### ✅ Contenido
- [ ] Información técnicamente correcta
- [ ] Referencias apropiadas incluidas
- [ ] Ejemplos de código compilan y funcionan
- [ ] Tests incluidos para nuevo código
- [ ] Typos y gramática revisados

#### ✅ Estructura  
- [ ] Navegación/orden lógico
- [ ] Headers descriptivos
- [ ] Links internos funcionan
- [ ] Metadata completada

#### ✅ Calidad
- [ ] Añade valor al contenido existente
- [ ] Apropiado para audiencia target
- [ ] Consistente con style guide
- [ ] Imágenes optimizadas (<500KB)

### 🔄 Proceso de Review

1. **Auto-review**: Usar checklist antes de submit
2. **Automated checks**: CI ejecuta tests y linting
3. **Peer review**: Al menos 1 maintainer aprueba
4. **Final review**: Core team hace merge

## 🐛 Reportar Issues

### 🚨 Bugs Técnicos

`markdown
**Descripción**: Breve descripción del bug

**Pasos para Reproducir**:
1. Ir a página X
2. Ejecutar código Y  
3. Observar resultado Z

**Resultado Esperado**: Qué debería pasar

**Resultado Actual**: Qué pasa realmente

**Environment**:
- OS: [e.g. macOS 13.0]
- Browser: [e.g. Chrome 108]
- MkDocs version: [e.g. 1.4.2]
`

### 💡 Feature Requests

`markdown
**Problema**: ¿Qué problema esto resolvería?

**Solución Propuesta**: Descripción de la feature

**Alternativas**: Otras soluciones consideradas

**Contexto Adicional**: Screenshots, ejemplos, etc.
`

## 🎓 Recursos para Contributors

### 📚 Lectura Recomendada

- [*Designing Data-Intensive Applications*](https://dataintensive.net/) por Martin Kleppmann
- [*Distributed Algorithms*](https://www.distributed-algorithms.com/) por Nancy Lynch  
- [Papers We Love - Distributed Systems](https://github.com/papers-we-love/papers-we-love/tree/master/distributed_systems)

### 🛠️ Herramientas Útiles

- **Diagramas**: [Excalidraw](https://excalidraw.com/), [draw.io](https://draw.io/)
- **Benchmarking**: [JMH](https://github.com/openjdk/jmh), [hyperfine](https://github.com/sharkdp/hyperfine)
- **Testing**: [jcstress](https://github.com/openjdk/jcstress), [QuickCheck](https://hackage.haskell.org/package/QuickCheck)
- **Formal Methods**: [TLA+](https://lamport.azurewebsites.net/tla/tla.html), [Alloy](https://alloytools.org/)

## 📞 Contacto

- **Discussions**: [GitHub Discussions](https://github.com/odin-distributed-systems/guide/discussions)
- **Issues**: [GitHub Issues](https://github.com/odin-distributed-systems/guide/issues)
- **Email**: maintainers@odin-guide.dev

---

## 🎉 ¡Gracias por Contribuir!

Cada contribución, por pequeña que sea, hace que ODIN sea mejor para toda la comunidad de sistemas distribuidos.

**¡Esperamos ver tus ideas en acción!** 🚀
