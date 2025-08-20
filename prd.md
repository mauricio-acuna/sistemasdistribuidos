---
title: "ODIN — PRD para Web Estática y Guía Técnica" version: 1.0 status: Draft owners:
Platform Engineering / Distributed Systems Guild last_updated: 2025-08-19


---
> Propósito: Este PRD define el contenido y los requisitos para una web estática que prepare al equipo en los conceptos clave de sistemas distribuidos y concurrencia multihilo implementados en ODIN, sirva de base para crear sistemas aún superiores en calidad, y establezca un plan de evolución tecnológica para cargas de muy alta performance.



---
1. Resumen ejecutivo
ODIN implementa una arquitectura multithreaded y distribuida que aborda consenso, consistencia, tolerancia a particiones y detección de fallos. La web resultante debe:
Educar (conceptos, patrones, límites teóricos y trade‑offs).
Operativizar (checklists, perfiles de pools, telemetría, pruebas, despliegue seguro).
Elevar el listón (roadmap de mejoras, tecnologías emergentes, propuestas de investigación aplicada).

Resultados esperados:
Una web navegable por conceptos, con tutoriales prácticos, guías de configuración y recursos gratuitos.
Estándares de calidad: SLOs de latencia/ disponibilidad, criterios de aceptación, dashboards de referencia.


---
2. Objetivos y KPIs
Objetivos
1. Homogeneizar el conocimiento del equipo en fundamentos distribuidos y concurrencia.

2. Reducir incidentes por saturación de pools, deadlocks y regresiones de consistencia.

3. Acelerar decisiones de diseño con plantillas, checklists y evidencia de benchmarks.


KPIs / SLOs sugeridos
Disponibilidad: ≥ 99.95% mensual (errores 5xx + timeouts bajo umbral).
Latencia (p95/p99): p95 < 20 ms; p99 < 60 ms para operaciones típicas de lectura; p99 < 120 ms para escritura con consenso.
Estabilidad de líder: cambios de líder < 2/hora bajo carga nominal.
Colas: backlog medio por pool < 25% del máximo; rechazos < 0.1%.
MTTR: < 20 min en incidentes Sev‑2.


---
3. Alcance y fuera de alcance
En alcance: conceptos, patrones, configuraciones de pools, consenso (Raft/Paxos/derivados), consistencia (modelos y niveles), detección de fallos (φ-accrual), idempotencia, relojes lógicos, backpressure, pruebas (jcstress, Jepsen‑style, chaos), observabilidad (OpenTelemetry, Prometheus), despliegue (canary, feature flags), seguridad básica (mTLS, políticas).
Fuera de alcance: tolerancia a fallos bizantinos avanzada (PBFT/HotStuff) en primera versión; criptografía avanzada; hardware RDMA/SmartNIC en producción (cubierto como evolución).

---
4. Audiencia y stakeholders
Ingeniería de plataforma / backend (Java/JVM).
SRE/Operaciones.
Arquitectura/Staff Engineers.
QA/Performance Engineering.
Product/Delivery (para comprender riesgos/roadmap).


---
5. Arquitectura y principios
5.1 Principios de diseño
Complejidad vs Performance: solo adoptar complejidad que reduzca latencia o aumente throughput con beneficios demostrables.
Separation of Concerns: pools aislados por tipo de carga; límites y backpressure en cada frontera.
Fail‑Fast & Observability‑First: timeouts, circuit breakers, métricas y trazas desde el día 0.
Idempotencia & Reintentos: handlers externos idempotentes; exactamente‑una‑vez solo donde sea garantizable con transacciones.

5.2 Consenso
Patrones: Raft, (Multi)‑Paxos, ZAB, Viewstamped Replication.
Requisitos: leader election, log replication, commit index, quorum, snapshotting.
Invariantes: único líder efectivo; orden total en log comprometido; no pérdida de entradas comprometidas.

5.3 Consistencia
Modelos: Linealizable, Secuencial, Causal, Eventual.
Patrones de cliente: read‑your‑writes, monotonic reads/writes.
Teoremas útiles: CAP (elección CP/AP bajo particiones) y PACELC (latencia vs consistencia sin particiones).

5.4 Detección de fallos
Heartbeats + timeouts; φ‑accrual para umbrales probabilísticos.
Mitigación de falsos positivos (GC, stop‑the‑world) y split‑brain (fencing tokens, leases).

5.5 Tiempo y orden
NTP insuficiente para orden total; usar Lamport o Vector clocks para happens‑before.
HLC (Hybrid Logical Clocks) como mejora práctica para orden cercano al real.


---
6. Concurrencia multihilo
6.1 Lock‑free & estructuras
Atomics (CAS), memory fences, retry loops. Riesgos: ABA, false sharing. Recolección segura: hazard pointers, epoch‑based reclamation.

6.2 Thread pools aislados
Topología recomendada (ajustable):
rpc-io (I/O cliente/servidor, colas bounded, timeouts cortos).
state-machine (aplicar entradas del log; CPU‑bound, tamaño≈cores físicos).
replication (I/O a seguidores; batching + flush coalescing).
db-io (acceso a almacenamiento; drivers async si existen).
bg-maintenance (snapshots, compaction, métricas).

Dimensionamiento
CPU‑bound: threads ≈ #cores físicos (±1 según cache/misses).
I/O‑bound: threads ≈ cores * (1 + wait/compute).

Colas y rechazo
CPU‑bound → SynchronousQueue o colas pequeñas.
I/O‑bound → Array/LinkedBlockingQueue con límites.
Definir RejectedExecutionHandler que propague backpressure o caller-runs seguro.

6.3 Backpressure y prioridades
Enrutar prioridades (p. ej., control vs data). Evitar head‑of‑line blocking entre pools.
Bulkheads y rate limits por interfaz externa.


---
7. Seguridad (mínimos operativos)
mTLS servicio‑a‑servicio, rotación de certificados.
Autorización con políticas (ABAC/RBAC); fencing tokens para evitar split‑brain.
Cifrado en reposo; secrets gestionados por vault.


---
8. Observabilidad
Métricas: p50/p95/p99 por endpoint y pool; backlog de colas; rechazos; context switches; pausas de GC; cambios de líder; replication lag.
Trazas: OpenTelemetry con trace‑id que cruce RPC → consenso → almacenamiento.
Logs: estructurados (JSON), con correlation ids.
Dashboards de referencia: latencia por percentiles, saturación por pool, estabilidad del líder, errores por tipo.


---
9. Pruebas y validación
Concurrencia: jcstress para primitivas/estructuras; JMH para microbenchmarks.
Integración: tests con randomized scheduling; fuzzing de tiempos.
Chaos: latencia, pérdida de paquetes, particiones, skew de reloj.
Jepsen‑style: validar propiedades de consistencia bajo fallos.
Formal: TLA+ (PlusCal) para invariantes de consenso y colas con backpressure.

Criterios de aceptación
Invariantes satisfechas bajo 3 escenarios de fallo (red, nodo caído, GC larga).
p99 en o por debajo de SLO en ≥ 95% de ejecuciones del benchmark repetido.
Sin deadlocks detectados en 24h de stress.


---
10. Performance engineering
Metodología: curvas latencia‑throughput, identificar punto de rodilla; separación de calorías CPU vs espera I/O.
Entorno: afinidad CPU, irqbalance, NUMA awareness donde aplique; GC configurado y medido.
Herramientas: JMH, wrk/k6, async‑profiler, perf/eBPF, FlameGraphs.
Técnicas: batching, zero‑copy, object pooling prudente, cache‑aware data layout, pinned threads para colas de ultra‑baja latencia (ej. Disruptor), hedged requests para colas largas.


---
11. Despliegue y operativa
Estrategia: feature flags, canary, shadow traffic, graceful degradation.
Alertas: líder flapping, crecimiento sostenido de cola, GC > X ms, tasa de rechazos, p99 fuera de SLO, divergencia de logs.
Procedimientos: snapshot/restore, data repair (compaction, re‑index), runbooks por incidente.


---
12. Información para la web estática
12.1 Mapa del sitio
Inicio (resumen + navegación por conceptos).
Fundamentos (consenso, consistencia, CAP/PACELC, relojes, detección de fallos).
Concurrencia (lock‑free, pools aislados, backpressure, patrones actor/SEDA/Disruptor).
Operación (observabilidad, SLOs, despliegue, runbooks).
Pruebas (jcstress, Jepsen‑style, chaos, TLA+).
Recetas (perfiles de pools, configs, ejemplos de rechazo/backpressure).
Recursos gratuitos (papers, cursos, herramientas).
Glosario.

12.2 Estructura de carpetas sugerida
/site  /content    index.md    fundamentos/      consenso.md      consistencia.md      cap_pacelc.md      relojes.md      fallos.md    concurrencia/      lock_free.md      thread_pools.md      backpressure.md      patrones.md    operacion/      observabilidad.md      despliegue.md      runbooks.md    pruebas/      jcstress.md      jepsen_style.md      chaos.md      tla_plus.md    recetas/      perfiles_pools.md      configs.md      ejemplos.md    recursos.md    glosario.md  /assets  /layouts (según generador)
12.3 Front‑matter (YAML) por página
---title: "Consenso (Raft vs Paxos)"description: "Cómo logramos decisiones únicas y orden total bajo fallos"nav_order: 10tags: [consensus, raft, paxos]---
12.4 Componentes de contenido
Callouts: Nota/Advertencia/Tip.
Diagramas: secuencias (PlantUML/Mermaid) para elecciones de líder, replicación, backpressure.
Tablas: comparativas (CP vs AP; colas; modelos de consistencia).

12.5 SEO y accesibilidad
Metadatos por página; títulos h1→h3 bien jerarquizados; contraste AA; navegación por teclado.


---
13. Plantillas y ejemplos
13.1 Tabla comparativa de consistencia
Modelo	Garantía	Uso típico	Coste
Linealizable	Orden real y único	Contadores bancarios	Latencia alta bajo particionesSecuencial	Orden por cliente	Logs, colas	Menos estrictaCausal	Respeta dependencias	Feeds sociales	Metadatos extraEventual	Convergencia con tiempo	Cachés, catálogos	Resolución de conflictos

13.2 Pseudocódigo de dimensionamiento de pools
int cores = Runtime.getRuntime().availableProcessors();int cpuPool = Math.max(cores, cores - 1);int ioPool = (int) Math.ceil(cores * (1.0 + waitOverCompute)); // waitOverCompute = W/C
13.3 Reglas de backpressure
Límite superior por cola/pool; si se alcanza → rechazo explícito con código/razón y retry-after.
Caller-runs solo en trayectorias idempotentes.
Telemetría: mediana, p95, p99 y backlog por minuto.


---
14. Roadmap de evolución tecnológica
Ahora (0–3 meses)
Estabilizar pools y backpressure; dashboards y alertas mínimas.
Pruebas jcstress/JMH en estructuras críticas; chaos básico.
Modelo TLA+ de consenso + invariantes de colas.

Siguiente (3–9 meses)
Migración evaluada a Java 21 LTS para Virtual Threads (Loom) y structured concurrency.
HLC para orden cercano al real; hedged requests en RPC.
Aeron / Chronicle Queue / Disruptor para trays de baja latencia.
eBPF‑based observability (profiling continuo, TCP drops, colas de NIC).
Evaluación de NATS JetStream / Redpanda según casos.

Más adelante (9–18 meses)
User‑space networking (DPDK/io_uring) donde aplique; QUIC para RPC multi‑stream con mejor control de head‑of‑line.
CRDTs en dominios AP; EPaxos/Flexible Paxos para reducir latencia multi‑datacenter.
RDMA/SmartNIC en entornos controlados; kernel‑bypass.

Investigación aplicada
Adaptive backpressure con RL ligero; priorización dinámica.
Scheduling consciente de NUMA y cache locality.


---
15. Riesgos y mitigaciones
Riesgo	Impacto	Prob.	Mitigación
Config incorrecta de timeouts	Caídas de líder/timeout storms	Media	Valores por entorno + jitter + testsBackpressure ausente	Efecto dominó entre servicios	Alta	Límites por cola + rechazo + dashboardsGC pausas largas	Falsos positivos de fallo	Media	Tuning GC + φ‑accrual + slow follower handlingCambios no validados	Reversiones/bugs críticos	Media	Canary + feature flags + shadow


---
16. Requisitos no funcionales
Compatibilidad: Java 17 (target), evaluar Java 21 en roadmap.
Portabilidad: contenedores; readiness/liveness probes.
Seguridad: mTLS, cifrado en reposo, rotación de secretos.
Documentación: cada cambio con decision record (ADR) y pruebas adjuntas.


---
17. Entregables
1. Web estática (carpeta /site) lista para publicar (GitHub Pages, Netlify o similar).

2. Dashboards de ejemplo (Prometheus/Grafana) exportables.

3. Plantillas de configs de pools y runbooks.

4. Checklist de aceptación por release.



---
18. Recursos gratuitos (curados)
Fundamentos / papers
Lamport — Time, Clocks, and the Ordering of Events in a Distributed System.
Fischer, Lynch, Paterson — Impossibility of Distributed Consensus with One Faulty Process (FLP).
Lamport — Paxos Made Simple.
Ongaro & Ousterhout — In Search of an Understandable Consensus Algorithm (Raft) (incluye sitio raft.github.io).
Fox & Brewer — Harvest, Yield, and Scalable Tolerant Systems.
Abadi — PACELC (artículos y paper resumen).
Dean & Barroso — The Tail at Scale.
Chandy & Lamport — Distributed Snapshots.
Hayashibara et al. — Φ‑accrual failure detector.
Welsh et al. — SEDA.
LMAX — Disruptor (paper + repo).

Cursos abiertos
MIT 6.824/6.5840 — Distributed Systems (videos + labs Raft/KV).
Charlas/notas de Martin Kleppmann (Cambridge) sobre sistemas distribuidos y DDIA.
TLA+ Video Course (YouTube) + TLA+ Hyperbook.
Google SRE Book (web abierta).

Herramientas
Jepsen (blog/casos de estudio).
jcstress y JMH (OpenJDK).
Prometheus + Grafana.
OpenTelemetry (trazas/métricas).
async‑profiler y FlameGraphs.

Java/JVM
Java Memory Model (JLS §17), java.util.concurrent, ForkJoinPool, CompletableFuture.
LMAX Disruptor (repo y wiki).


---
19. Glosario
Backpressure: mecanismo de control de flujo que evita saturación propagando límite al productor.
Quorum: subconjunto de nodos suficiente para decidir (p.ej., mayoría).
Linealizabilidad: operaciones parecen atómicas y en orden real.
HLC: reloj lógico híbrido (físico + lógico) para orden cercano al tiempo real.


---
20. Anexos
20.1 Checklist de salida a producción
[ ] Timeouts con jitter por RPC/cliente.
[ ] Idempotencia en handlers externos.
[ ] Límite de colas y rate limits documentados.
[ ] Dashboards p95/p99 por pool y endpoint.
[ ] Canary + feature flags + rollback plan.
[ ] Runbooks para: líder flapping, cola saturada, GC larga, partición de red.

20.2 Prompt sugerido (Claude Sonnet 4)
> Instrucción: "Genera una web estática (MkDocs o Docusaurus) a partir del PRD adjunto. Crea la estructura de carpetas indicada en /site, divide el contenido en páginas por sección y añade navegación lateral, breadcrumbs y buscador. Inserta callouts, tablas y diagramas Mermaid para consenso, replicación y backpressure. Usa front‑matter YAML por página con title, description, tags y nav_order. Exporta un paquete listo para publicar en GitHub Pages con un tema claro y accesible (AA), y un índice con tarjetas por sección. No inventes contenido: usa literalmente los textos del PRD."


20.3 Snippets de configuración orientativos
Java (pools)
var rpcIo = new ThreadPoolExecutor(  16, 16, 60, TimeUnit.SECONDS,  new ArrayBlockingQueue<>(1024),  new NamedThreadFactory("rpc-io"),  new ThreadPoolExecutor.AbortPolicy() // rechaza y eleva backpressure);
var stateMachine = new ForkJoinPool(Math.max(1, Runtime.getRuntime().availableProcessors()-1));
Prometheus (métricas básicas)
- job_name: 'odin'  static_configs:    - targets: ['odin-a:9090','odin-b:9090']
Alerta (ejemplo)
- alert: QueueBacklogHigh  expr: odin_pool_queue_size > 0.8 * odin_pool_queue_max  for: 5m  labels: { severity: 'page' }  annotations:    summary: 'Backlog alto en ${pool}'    description: 'Rechazos inminentes; verificar productores y límites.'


no olvidar subir todo al repositorio

https://github.com/mauricio-acuna/sistemasdistribuidos.git
