---
title: "Autoevaluación - Sistemas Distribuidos"
description: "Evalúa tu conocimiento con preguntas de múltiple opción"
nav_order: 50
---

# 🧠 Autoevaluación - Sistemas Distribuidos

!!! info "Objetivo"
    
    **Valida tu comprensión** de los conceptos fundamentales de sistemas distribuidos mediante preguntas de múltiple opción.
    
    **Meta:** Obtener al menos **70%** de respuestas correctas para considerar el tema dominado.

---

## 📝 Evaluaciones Disponibles

### 🟢 **Nivel Fundamentos**

<div class="evaluation-card">

#### [**�� Tiempo y Relojes Distribuidos**](tiempo-relojes-quiz.md)
- Lamport timestamps
- Vector clocks  
- Hybrid Logical Clocks
- Sincronización de tiempo

**Duración:** ~10 minutos | **Preguntas:** 15

</div>

<div class="evaluation-card">

#### [**🚨 Detección de Fallos**](deteccion-fallos-quiz.md)
- Failure detectors
- φ-accrual algorithm
- SWIM protocol
- Heartbeat patterns

**Duración:** ~8 minutos | **Preguntas:** 12

</div>

<div class="evaluation-card">

#### [**🗳️ Algoritmos de Consenso**](consenso-quiz.md)
- Raft algorithm
- Paxos variants
- Leader election
- Log replication

**Duración:** ~15 minutos | **Preguntas:** 20

</div>

<div class="evaluation-card">

#### [**🔄 Modelos de Consistencia**](consistencia-quiz.md)
- Linearizability
- Sequential consistency
- Causal consistency
- Eventual consistency

**Duración:** ~12 minutos | **Preguntas:** 18

</div>

### 🟡 **Nivel Avanzado**

<div class="evaluation-card">

#### [**⚖️ CAP y PACELC**](cap-pacelc-quiz.md)
- CAP theorem
- PACELC extension
- Trade-off decisions
- Real-world examples

**Duración:** ~10 minutos | **Preguntas:** 15

</div>

<div class="evaluation-card">

#### [**🔒 Algoritmos Lock-Free**](lock-free-quiz.md)
- Compare-and-swap
- ABA problem
- Memory ordering
- Lock-free data structures

**Duración:** ~12 minutos | **Preguntas:** 16

</div>

### 🔴 **Evaluación Integral**

<div class="evaluation-card highlight">

#### [**🏆 Examen Completo de Sistemas Distribuidos**](examen-completo.md)
- Todos los temas fundamentales
- Casos prácticos complejos
- Análisis de trade-offs
- Decisiones arquitectónicas

**Duración:** ~45 minutos | **Preguntas:** 50

</div>

---

## 📊 **Sistema de Puntuación**

### 🎯 **Criterios de Evaluación**

| Puntuación | Nivel | Descripción |
|------------|-------|-------------|
| **90-100%** | 🏆 **Excelente** | Dominio completo del tema |
| **80-89%** | 🥇 **Muy Bueno** | Comprensión sólida |
| **70-79%** | ✅ **Bueno** | Conocimiento adecuado |
| **60-69%** | ⚠️ **Regular** | Necesita refuerzo |
| **< 60%** | ❌ **Insuficiente** | Requiere estudio adicional |

### 🔄 **Política de Reintentos**

- **< 70%:** Botón "Reintentar" disponible inmediatamente
- **≥ 70%:** Opción de "Mejorar Puntuación" después de 24h
- **Intentos ilimitados** para alcanzar el objetivo de aprendizaje

---

## 🎓 **Cómo Usar las Evaluaciones**

### 1️⃣ **Preparación**
- Lee el material correspondiente antes de la evaluación
- Revisa implementaciones de código
- Practica con ejemplos

### 2️⃣ **Durante la Evaluación**
- Lee cada pregunta cuidadosamente
- Considera todas las opciones antes de responder
- No hay límite de tiempo (excepto el examen completo)

### 3️⃣ **Después de la Evaluación**
- Revisa las respuestas incorrectas
- Consulta las explicaciones detalladas
- Repasa el material relacionado si es necesario

---

## 📈 **Progreso de Aprendizaje**

### 🛤️ **Ruta Sugerida**

1. **Fundamentos** (completa en orden):
   - Tiempo y Relojes → Detección de Fallos → Consenso → Consistencia

2. **Teoremas** (después de fundamentos):
   - CAP y PACELC

3. **Implementación** (nivel avanzado):
   - Lock-Free Algorithms

4. **Validación Final**:
   - Examen Completo

### 🏅 **Certificación Informal**

Al completar **todas las evaluaciones con ≥70%**, habrás demostrado:

- ✅ Comprensión de fundamentos teóricos
- ✅ Conocimiento de implementaciones prácticas  
- ✅ Capacidad de análisis de trade-offs
- ✅ Preparación para sistemas distribuidos de producción

---

!!! tip "Consejos para el Éxito"
    
    - **Practica regularmente**: 15-20 minutos diarios son más efectivos que sesiones largas
    - **Implementa los algoritmos**: La programación refuerza la comprensión teórica
    - **Discute con colegas**: Explicar conceptos ayuda a consolidar el conocimiento
    - **Aplica en proyectos reales**: Busca oportunidades para usar estos patrones

!!! warning "Recordatorio Importante"
    
    Las evaluaciones son **herramientas de aprendizaje**, no exámenes competitivos. El objetivo es identificar áreas de mejora y reforzar tu comprensión.

---

<div align="center">

**🚀 ¿Listo para comenzar tu autoevaluación?**

[**Comenzar con Tiempo y Relojes**](tiempo-relojes-quiz.md){ .md-button .md-button--primary }
[**Ir directo al Examen Completo**](examen-completo.md){ .md-button }

</div>

<style>
.evaluation-card {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 16px;
    margin: 12px 0;
    background-color: #f9f9f9;
}

.evaluation-card.highlight {
    border-color: #ff6b35;
    background-color: #fff5f2;
}

.evaluation-card h4 {
    margin-top: 0;
    color: #333;
}
</style>
