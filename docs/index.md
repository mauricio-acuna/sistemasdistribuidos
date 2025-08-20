---
title: "ODIN - Guía de Sistemas Distribuidos"
description: "Guía técnica completa para sistemas distribuidos, consenso, concurrencia multihilo y alta performance"
hide:
  - navigation
  - toc
---

# ODIN - Guía de Sistemas Distribuidos

<div class="grid cards" markdown>

-   :material-account-group-outline:{ .lg .middle } **Para Equipos de Plataforma**

    ---

    Documentación técnica avanzada para ingenieros que construyen y mantienen sistemas distribuidos complejos de alta performance.

    [:octicons-arrow-right-24: Comenzar](fundamentos/)

-   :material-lightbulb-outline:{ .lg .middle } **Fundamentos Teóricos**

    ---

    Consenso, consistencia, CAP/PACELC, relojes lógicos y detección de fallos. Los pilares conceptuales de sistemas distribuidos.

    [:octicons-arrow-right-24: Explorar fundamentos](fundamentos/)

-   :material-cpu-64-bit:{ .lg .middle } **Concurrencia Multihilo**

    ---

    Estructuras lock-free, thread pools aislados, backpressure y patrones avanzados para máximo rendimiento.

    [:octicons-arrow-right-24: Ver concurrencia](concurrencia/)

-   :material-chart-line:{ .lg .middle } **Observabilidad & SLOs**

    ---

    Métricas, trazas, dashboards y alertas para sistemas de producción con requisitos de alta disponibilidad.

    [:octicons-arrow-right-24: Aprender observabilidad](operacion/)

</div>

## ¿Por qué esta guía?

ODIN implementa una arquitectura multithreaded y distribuida que aborda **consenso**, **consistencia**, **tolerancia a particiones** y **detección de fallos**. Esta guía surge de la necesidad de:

!!! success "Objetivos Clave"
    
    - **Homogeneizar conocimiento** del equipo en fundamentos distribuidos
    - **Reducir incidentes** por saturación de pools, deadlocks y regresiones de consistencia  
    - **Acelerar decisiones** de diseño con plantillas, checklists y evidencia de benchmarks

## SLOs de Referencia

Los sistemas que construimos deben cumplir estándares de clase mundial:

| Métrica | Objetivo | Umbral Crítico |
|---------|----------|----------------|
| **Disponibilidad** | ≥ 99.95% mensual | < 99.9% |
| **Latencia p95** | < 20ms (lectura) | > 50ms |
| **Latencia p99** | < 60ms (lectura) | > 120ms |
| **Escritura con consenso p99** | < 120ms | > 200ms |
| **Cambios de líder** | < 2/hora | > 5/hora |
| **MTTR Sev-2** | < 20 min | > 45 min |

## Principios de Diseño

=== "Complejidad vs Performance"

    Solo adoptar complejidad que reduzca latencia o aumente throughput con **beneficios demostrables**.

=== "Separation of Concerns"

    Pools aislados por tipo de carga; límites y backpressure en cada frontera.

=== "Fail-Fast & Observability-First"

    Timeouts, circuit breakers, métricas y trazas desde el día 0.

=== "Idempotencia & Reintentos"

    Handlers externos idempotentes; exactamente-una-vez solo donde sea garantizable.

## Navegación Rápida

<div class="grid cards" markdown>

-   **[Consenso](fundamentos/consenso.md)**
    
    Raft, Paxos, leader election y replicación de logs

-   **[Thread Pools](concurrencia/thread_pools.md)**
    
    Dimensionamiento, aislamiento y topologías recomendadas

-   **[Backpressure](concurrencia/backpressure.md)**
    
    Control de flujo y prevención de cascadas de fallo

-   **[Observabilidad](operacion/observabilidad.md)**
    
    OpenTelemetry, Prometheus y dashboards de referencia

-   **[JCStress](pruebas/jcstress.md)**
    
    Pruebas de concurrencia y validación de estructuras lock-free

-   **[Runbooks](operacion/runbooks.md)**
    
    Procedimientos operativos para incidentes comunes

</div>

## Recursos Destacados

!!! tip "Papers Fundamentales"
    
    - [Lamport - Time, Clocks, and the Ordering of Events](recursos/biblioteca.md#lamport-time-clocks)
    - [Ongaro & Ousterhout - In Search of an Understandable Consensus Algorithm (Raft)](recursos/biblioteca.md#raft-paper)
    - [Abadi - PACELC](recursos/biblioteca.md#pacelc)
    - [Dean & Barroso - The Tail at Scale](recursos/biblioteca.md#tail-at-scale)

!!! example "Herramientas Esenciales"
    
    - **[jcstress](recursos/herramientas.md#jcstress)** - Pruebas de concurrencia
    - **[Jepsen](recursos/herramientas.md#jepsen)** - Validación de consistencia
    - **[async-profiler](recursos/herramientas.md#async-profiler)** - Profiling de performance
    - **[TLA+](recursos/herramientas.md#tla-plus)** - Verificación formal

---

**Última actualización:** Agosto 2025 | **Versión:** 1.0 | **Estado:** Producción
