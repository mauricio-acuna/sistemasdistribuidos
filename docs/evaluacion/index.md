---
title: "Sistema de Autoevaluación ODIN"
---

# 🎯 Sistema de Autoevaluación ODIN
*Evalúa tu conocimiento en Sistemas Distribuidos*

<style>
.eval-container {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    padding: 30px;
    border-radius: 15px;
    margin: 20px 0;
    color: white;
    box-shadow: 0 10px 30px rgba(0,0,0,0.3);
}

.quiz-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
    margin: 30px 0;
}

.quiz-card {
    background: rgba(255,255,255,0.95);
    padding: 25px;
    border-radius: 12px;
    box-shadow: 0 5px 15px rgba(0,0,0,0.1);
    transition: transform 0.3s ease;
    color: #2d3748;
}

.quiz-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 25px rgba(0,0,0,0.2);
}

.quiz-icon {
    font-size: 3em;
    text-align: center;
    margin-bottom: 15px;
}

.quiz-title {
    font-size: 1.4em;
    font-weight: bold;
    margin-bottom: 10px;
    color: #4a5568;
}

.quiz-description {
    margin-bottom: 15px;
    line-height: 1.6;
    color: #718096;
}

.quiz-stats {
    display: flex;
    justify-content: space-between;
    margin-bottom: 15px;
    font-size: 0.9em;
    color: #a0aec0;
}

.quiz-button {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    padding: 12px 24px;
    border-radius: 25px;
    cursor: pointer;
    text-decoration: none;
    display: inline-block;
    transition: all 0.3s ease;
    box-shadow: 0 4px 15px rgba(0,0,0,0.2);
    width: 100%;
    text-align: center;
}

.quiz-button:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(0,0,0,0.3);
    text-decoration: none;
    color: white;
}

.final-exam-card {
    background: linear-gradient(135deg, #2d3748 0%, #1a202c 100%);
    color: white;
    grid-column: 1 / -1;
    padding: 30px;
    text-align: center;
}

.final-exam-card .quiz-button {
    background: linear-gradient(135deg, #f6ad55 0%, #ed8936 100%);
    font-size: 1.2em;
    padding: 15px 40px;
    margin-top: 10px;
}

.progress-section {
    background: rgba(255,255,255,0.1);
    padding: 20px;
    border-radius: 10px;
    margin: 20px 0;
}

.progress-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin: 10px 0;
    padding: 10px;
    background: rgba(255,255,255,0.1);
    border-radius: 8px;
}

.criteria-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 15px;
    margin: 20px 0;
}

.criteria-card {
    background: rgba(255,255,255,0.1);
    padding: 15px;
    border-radius: 8px;
}

.badge {
    display: inline-block;
    padding: 4px 12px;
    border-radius: 12px;
    font-size: 0.8em;
    font-weight: bold;
    margin-left: 10px;
}

.badge-beginner { background: #fed7d7; color: #c53030; }
.badge-intermediate { background: #fef5e7; color: #dd6b20; }
.badge-advanced { background: #e6fffa; color: #00b5d8; }
.badge-expert { background: #e6f6ff; color: #0078d4; }
</style>

<div class="eval-container">
    <h2>🎓 Bienvenido al Sistema de Evaluación</h2>
    <p>Pon a prueba tus conocimientos en Sistemas Distribuidos con nuestros quizzes especializados. Cada quiz está diseñado para evaluar aspectos específicos del curso ODIN.</p>
    
    <div class="progress-section">
        <h3>📊 Criterios de Evaluación</h3>
        <div class="criteria-grid">
            <div class="criteria-card">
                <h4>🥇 Experto (90-100%)</h4>
                <p>Dominio completo del tema. Listo para trabajar en sistemas distribuidos complejos.</p>
            </div>
            <div class="criteria-card">
                <h4>🥈 Avanzado (80-89%)</h4>
                <p>Conocimiento sólido. Capaz de diseñar y implementar soluciones distribuidas.</p>
            </div>
            <div class="criteria-card">
                <h4>🥉 Intermedio (70-79%)</h4>
                <p>Comprensión básica. Necesita práctica adicional en algunos conceptos.</p>
            </div>
            <div class="criteria-card">
                <h4>📚 Principiante (<70%)</h4>
                <p>Requiere estudio adicional antes de aplicar los conceptos.</p>
            </div>
        </div>
    </div>
</div>

<div class="quiz-grid">
    <!-- Quiz 1: Detección de Fallos -->
    <div class="quiz-card">
        <div class="quiz-icon">🔍</div>
        <div class="quiz-title">Detección de Fallos</div>
        <div class="quiz-description">
            Evalúa tu comprensión de modelos de falla, detectores de fallos, timeouts y técnicas de heartbeat en sistemas distribuidos.
        </div>
        <div class="quiz-stats">
            <span>⏱️ 15-20 min</span>
            <span>📝 10 preguntas</span>
            <span>🎯 70% para aprobar</span>
        </div>
        <div style="margin-bottom: 10px;">
            <strong>Temas cubiertos:</strong>
            <ul style="margin: 5px 0; font-size: 0.9em;">
                <li>Modelos de falla (crash, omission, Byzantine)</li>
                <li>Detectores de fallos (Perfect, Eventually Perfect)</li>
                <li>Algoritmos de heartbeat y timeout</li>
                <li>Phi Accrual Failure Detector</li>
            </ul>
        </div>
        <a href="deteccion-fallos-quiz/" class="quiz-button">🚀 Comenzar Quiz</a>
    </div>

    <!-- Quiz 2: Algoritmos de Consenso -->
    <div class="quiz-card">
        <div class="quiz-icon">🏛️</div>
        <div class="quiz-title">Algoritmos de Consenso</div>
        <div class="quiz-description">
            Pon a prueba tu conocimiento sobre Raft, Paxos, Byzantine Fault Tolerance y otros algoritmos fundamentales de consenso.
        </div>
        <div class="quiz-stats">
            <span>⏱️ 20-25 min</span>
            <span>�� 15 preguntas</span>
            <span>🎯 70% para aprobar</span>
        </div>
        <div style="margin-bottom: 10px;">
            <strong>Temas cubiertos:</strong>
            <ul style="margin: 5px 0; font-size: 0.9em;">
                <li>Algoritmo Raft (liderazgo, replicación, safety)</li>
                <li>Paxos clásico y Multi-Paxos</li>
                <li>Byzantine Fault Tolerance (PBFT)</li>
                <li>Propiedades de safety y liveness</li>
            </ul>
        </div>
        <a href="consenso-quiz/" class="quiz-button">🚀 Comenzar Quiz</a>
    </div>

    <!-- Quiz 3: Modelos de Consistencia -->
    <div class="quiz-card">
        <div class="quiz-icon">🔄</div>
        <div class="quiz-title">Modelos de Consistencia</div>
        <div class="quiz-description">
            Explora tu comprensión del teorema CAP, PACELC, consistencia eventual, linearizabilidad y otros modelos de consistencia.
        </div>
        <div class="quiz-stats">
            <span>⏱️ 15-20 min</span>
            <span>📝 12 preguntas</span>
            <span>🎯 70% para aprobar</span>
        </div>
        <div style="margin-bottom: 10px;">
            <strong>Temas cubiertos:</strong>
            <ul style="margin: 5px 0; font-size: 0.9em;">
                <li>Teorema CAP y sus implicaciones</li>
                <li>PACELC: extensión del teorema CAP</li>
                <li>Consistencia eventual y strong consistency</li>
                <li>Linearizabilidad y causal consistency</li>
            </ul>
        </div>
        <a href="consistencia-quiz/" class="quiz-button">🚀 Comenzar Quiz</a>
    </div>

    <!-- Quiz 4: Lock-Free Programming -->
    <div class="quiz-card">
        <div class="quiz-icon">🔓</div>
        <div class="quiz-title">Lock-Free Programming</div>
        <div class="quiz-description">
            Evalúa tu dominio de programación sin bloqueos, operaciones atómicas, memory ordering y estructuras de datos lock-free.
        </div>
        <div class="quiz-stats">
            <span>⏱️ 15-20 min</span>
            <span>📝 12 preguntas</span>
            <span>🎯 70% para aprobar</span>
        </div>
        <div style="margin-bottom: 10px;">
            <strong>Temas cubiertos:</strong>
            <ul style="margin: 5px 0; font-size: 0.9em;">
                <li>Operaciones atómicas y Compare-and-Swap</li>
                <li>Memory ordering (acquire, release, relaxed)</li>
                <li>Problema ABA y sus soluciones</li>
                <li>Diferencias entre wait-free y lock-free</li>
            </ul>
        </div>
        <a href="lock-free-quiz/" class="quiz-button">🚀 Comenzar Quiz</a>
    </div>

    <!-- Examen Final -->
    <div class="quiz-card final-exam-card">
        <div class="quiz-icon">🏆</div>
        <div class="quiz-title">Examen Completo de Sistemas Distribuidos</div>
        <div class="quiz-description">
            Evaluación integral que combina todos los temas del curso ODIN. Demuestra tu dominio completo de los sistemas distribuidos.
        </div>
        <div class="quiz-stats" style="color: rgba(255,255,255,0.8);">
            <span>⏱️ 45 minutos</span>
            <span>📝 25 preguntas</span>
            <span>🎯 70% para certificarse</span>
        </div>
        <div style="margin-bottom: 15px;">
            <strong>Incluye preguntas de:</strong>
            <ul style="margin: 10px 0; font-size: 0.9em; color: rgba(255,255,255,0.9);">
                <li>🔍 Detección de Fallos (6 preguntas)</li>
                <li>🏛️ Algoritmos de Consenso (6 preguntas)</li>
                <li>🔄 Modelos de Consistencia (7 preguntas)</li>
                <li>🔓 Lock-Free Programming (6 preguntas)</li>
            </ul>
        </div>
        <a href="examen-completo/" class="quiz-button">🏆 Tomar Examen Final</a>
    </div>
</div>

---

## 📈 Ruta de Aprendizaje Recomendada

### 🎯 Para Principiantes
1. **📚 Estudia el contenido teórico** del curso ODIN
2. **🔍 Comienza con Detección de Fallos** - conceptos fundamentales
3. **🏛️ Continúa con Consenso** - algoritmos core
4. **🔄 Aprende Consistencia** - modelos y teoremas
5. **🔓 Finaliza con Lock-Free** - programación avanzada

### 🎯 Para Estudiantes Intermedios
1. **🏛️ Consenso** y **🔄 Consistencia** en paralelo
2. **🔍 Detección de Fallos** para reforzar fundamentos
3. **🔓 Lock-Free Programming** para técnicas avanzadas
4. **🏆 Examen Final** para certificación

### 🎯 Para Expertos
- **🏆 Toma el Examen Final directamente** para validar conocimientos
- **🔄 Revisa áreas específicas** según resultados
- **🛠️ Procede a proyectos prácticos** de implementación

---

## 🎓 Certificación ODIN

Al completar exitosamente el **Examen Final** (70% o más), obtienes:

- ✅ **Certificación ODIN** en Sistemas Distribuidos
- 📊 **Análisis detallado** de fortalezas por tema
- 🎯 **Nivel alcanzado** (Intermedio/Avanzado/Experto)
- 🚀 **Recomendaciones** para próximos pasos

---

## 💡 Consejos para el Éxito

### 📚 Preparación
- **Lee cuidadosamente** cada sección del curso
- **Entiende los conceptos** antes de memorizar
- **Practica con ejemplos** reales de implementación

### 🎯 Durante los Quizzes
- **Lee cada pregunta completamente** antes de responder
- **Usa el tiempo sabiamente** - no te quedes atascado
- **Revisa tus respuestas** si tienes tiempo

### 🔄 Después de cada Quiz
- **Revisa las explicaciones** de respuestas incorrectas
- **Estudia los temas** donde obtuviste menor puntuación
- **Practica con ejemplos adicionales** antes del siguiente quiz

---

*¿Listo para comenzar? ¡Elige tu primer quiz y demuestra tu conocimiento en sistemas distribuidos!*
