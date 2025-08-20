---
title: "Examen Completo de Sistemas Distribuidos"
---

# Examen Completo de Sistemas Distribuidos
*Evaluación Integral: Todos los Temas del Curso ODIN*

<style>
.quiz-container {
    background: linear-gradient(135deg, #2d3748 0%, #1a202c 100%);
    padding: 30px;
    border-radius: 15px;
    margin: 20px 0;
    color: white;
    box-shadow: 0 10px 30px rgba(0,0,0,0.3);
}

.question-card {
    background: rgba(255,255,255,0.95);
    color: #333;
    padding: 25px;
    border-radius: 12px;
    margin: 20px 0;
    box-shadow: 0 5px 15px rgba(0,0,0,0.1);
    transition: transform 0.3s ease;
}

.question-card:hover {
    transform: translateY(-5px);
}

.question-title {
    font-size: 1.2em;
    font-weight: bold;
    margin-bottom: 15px;
    color: #4a5568;
}

.topic-badge {
    display: inline-block;
    padding: 4px 12px;
    border-radius: 12px;
    font-size: 0.8em;
    font-weight: bold;
    margin-bottom: 10px;
}

.badge-consistency { background: #e6fffa; color: #234e52; }
.badge-consensus { background: #e6e6ff; color: #3c366b; }
.badge-detection { background: #fff5f5; color: #702459; }
.badge-lockfree { background: #ffe6e6; color: #744210; }
.badge-general { background: #f0fff4; color: #22543d; }

.option {
    margin: 10px 0;
    padding: 12px 15px;
    border: 2px solid #e2e8f0;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.3s ease;
    background: white;
}

.option:hover {
    border-color: #4a5568;
    background: #f7fafc;
}

.option.selected {
    border-color: #4a5568;
    background: #edf2f7;
}

.option.correct {
    border-color: #48bb78;
    background: #f0fff4;
    color: #22543d;
}

.option.incorrect {
    border-color: #f56565;
    background: #fff5f5;
    color: #742a2a;
}

.quiz-button {
    background: linear-gradient(135deg, #4a5568 0%, #2d3748 100%);
    color: white;
    border: none;
    padding: 12px 30px;
    border-radius: 25px;
    cursor: pointer;
    font-size: 1.1em;
    margin: 10px 5px;
    transition: all 0.3s ease;
    box-shadow: 0 4px 15px rgba(0,0,0,0.2);
}

.quiz-button:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(0,0,0,0.3);
}

.quiz-button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
    transform: none;
}

.result-container {
    background: linear-gradient(135deg, #48bb78 0%, #38a169 100%);
    padding: 20px;
    border-radius: 12px;
    margin: 20px 0;
    color: white;
    text-align: center;
}

.result-container.fail {
    background: linear-gradient(135deg, #f56565 0%, #e53e3e 100%);
}

.progress-bar {
    background: rgba(255,255,255,0.3);
    height: 8px;
    border-radius: 4px;
    margin: 15px 0;
    overflow: hidden;
}

.progress-fill {
    background: white;
    height: 100%;
    border-radius: 4px;
    transition: width 0.5s ease;
}

.navigation-buttons {
    display: flex;
    justify-content: space-between;
    margin: 20px 0;
}

.code-block {
    background: #2d3748;
    color: #e2e8f0;
    padding: 15px;
    border-radius: 8px;
    font-family: 'Courier New', monospace;
    margin: 10px 0;
    overflow-x: auto;
}

.timer-container {
    position: fixed;
    top: 20px;
    right: 20px;
    background: rgba(0,0,0,0.8);
    color: white;
    padding: 10px 20px;
    border-radius: 25px;
    font-weight: bold;
    z-index: 1000;
}

.timer-warning {
    background: rgba(245, 101, 101, 0.9) !important;
    animation: pulse 1s infinite;
}

@keyframes pulse {
    0% { transform: scale(1); }
    50% { transform: scale(1.05); }
    100% { transform: scale(1); }
}

.stats-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 15px;
    margin: 20px 0;
}

.stat-card {
    background: rgba(255,255,255,0.1);
    padding: 15px;
    border-radius: 8px;
    text-align: center;
}
</style>

<div id="quiz-app" class="quiz-container">
    <div v-if="quizStarted && timeRemaining > 0" class="timer-container" :class="{ 'timer-warning': timeRemaining <= 300 }">
        ⏱️ {{ formatTime(timeRemaining) }}
    </div>

    <h2>🏆 Examen Completo de Sistemas Distribuidos</h2>
    <p>Evaluación integral que cubre todos los temas del curso ODIN: Detección de Fallos, Consenso, Consistencia y Lock-Free Programming.</p>
    
    <div v-if="!quizStarted" class="text-center">
        <h3>📋 Instrucciones del Examen</h3>
        <ul style="text-align: left; max-width: 700px; margin: 0 auto;">
            <li><strong>25 preguntas</strong> mezcladas de todos los temas del curso</li>
            <li><strong>Tiempo límite:</strong> 45 minutos (2700 segundos)</li>
            <li><strong>Para aprobar:</strong> 70% (18/25 respuestas correctas)</li>
            <li><strong>Temas cubiertos:</strong>
                <ul>
                    <li>🔍 Detección de Fallos (6 preguntas)</li>
                    <li>🏛️ Algoritmos de Consenso (6 preguntas)</li>
                    <li>🔄 Modelos de Consistencia (7 preguntas)</li>
                    <li>🔓 Lock-Free Programming (6 preguntas)</li>
                </ul>
            </li>
            <li>El tiempo se mostrará en la esquina superior derecha</li>
            <li>El examen se enviará automáticamente cuando expire el tiempo</li>
            <li>Puedes navegar entre preguntas libremente</li>
        </ul>
        <button @click="startExam" class="quiz-button">🚀 Comenzar Examen</button>
    </div>

    <div v-if="quizStarted && !showResults && timeRemaining > 0">
        <div class="progress-bar">
            <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
        </div>
        <p>Pregunta {{ currentQuestion + 1 }} de {{ questions.length }} | Tiempo restante: {{ formatTime(timeRemaining) }}</p>
        
        <div class="question-card">
            <div :class="'topic-badge badge-' + questions[currentQuestion].topic">
                {{ getTopicName(questions[currentQuestion].topic) }}
            </div>
            <div class="question-title">{{ questions[currentQuestion].question }}</div>
            <div v-if="questions[currentQuestion].code" class="code-block">
                <pre>{{ questions[currentQuestion].code }}</pre>
            </div>
            <div v-for="(option, index) in questions[currentQuestion].options" 
                 :key="index"
                 @click="selectOption(index)"
                 class="option"
                 :class="{ 
                     'selected': selectedAnswers[currentQuestion] === index,
                     'correct': showAnswer && index === questions[currentQuestion].correct,
                     'incorrect': showAnswer && selectedAnswers[currentQuestion] === index && index !== questions[currentQuestion].correct
                 }">
                {{ option }}
            </div>
            
            <div v-if="showAnswer" style="margin-top: 15px; padding: 15px; background: #edf2f7; border-radius: 8px; color: #2d3748;">
                <strong>💡 Explicación:</strong> {{ questions[currentQuestion].explanation }}
            </div>
        </div>

        <div class="navigation-buttons">
            <button @click="previousQuestion" :disabled="currentQuestion === 0" class="quiz-button">⬅️ Anterior</button>
            <div>
                <button @click="showAnswer ? nextQuestion() : checkAnswer()" class="quiz-button">
                    {{ showAnswer ? 'Siguiente ➡️' : 'Verificar ✓' }}
                </button>
                <button @click="finishExam" class="quiz-button" style="background: linear-gradient(135deg, #f56565 0%, #e53e3e 100%);">
                    🏁 Finalizar Examen
                </button>
            </div>
        </div>
    </div>

    <div v-if="showResults || timeRemaining <= 0" class="result-container" :class="{ 'fail': score < passingScore }">
        <h3>🎯 Resultados del Examen Completo</h3>
        <div style="font-size: 2.5em; margin: 20px 0;">
            {{ score }}/{{ questions.length }} ({{ Math.round((score/questions.length)*100) }}%)
        </div>
        
        <div class="stats-grid">
            <div class="stat-card">
                <h4>⏱️ Tiempo</h4>
                <p>{{ timeRemaining <= 0 ? 'Tiempo agotado' : 'Finalizado a tiempo' }}</p>
            </div>
            <div class="stat-card">
                <h4>🔍 Detección Fallos</h4>
                <p>{{ getTopicScore('detection') }}/6</p>
            </div>
            <div class="stat-card">
                <h4>🏛️ Consenso</h4>
                <p>{{ getTopicScore('consensus') }}/6</p>
            </div>
            <div class="stat-card">
                <h4>🔄 Consistencia</h4>
                <p>{{ getTopicScore('consistency') }}/7</p>
            </div>
            <div class="stat-card">
                <h4>�� Lock-Free</h4>
                <p>{{ getTopicScore('lockfree') }}/6</p>
            </div>
        </div>
        
        <div v-if="score >= passingScore">
            <h4>🎉 ¡Felicitaciones!</h4>
            <p>Has completado exitosamente el curso ODIN de Sistemas Distribuidos.</p>
            <p><strong>Nivel alcanzado:</strong> {{ getLevel() }}</p>
            <p>Estás preparado para trabajar con sistemas distribuidos complejos.</p>
        </div>
        
        <div v-else>
            <h4>📚 Necesitas estudiar más</h4>
            <p>Revisa las áreas donde obtuviste menor puntuación.</p>
            <p><strong>Recomendación:</strong> Repasa los quizzes individuales antes de reintentar.</p>
        </div>

        <div style="margin-top: 20px;">
            <button @click="restartExam" class="quiz-button">🔄 Reintentar Examen</button>
            <button @click="goToEvaluation" class="quiz-button">📚 Volver a Evaluación</button>
            <button @click="goToCourse" class="quiz-button">🏠 Ir al Curso</button>
        </div>
    </div>
</div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
const { createApp } = Vue;

createApp({
    data() {
        return {
            quizStarted: false,
            currentQuestion: 0,
            selectedAnswers: {},
            showAnswer: false,
            showResults: false,
            score: 0,
            passingScore: 18,
            timeRemaining: 2700, // 45 minutos
            timer: null,
            questions: []
        };
    },
    computed: {
        progressPercentage() {
            return ((this.currentQuestion + 1) / this.questions.length) * 100;
        }
    },
    methods: {
        initializeQuestions() {
            const allQuestions = [
                // Detección de Fallos (6 preguntas)
                {
                    topic: 'detection',
                    question: "¿Cuál es la principal diferencia entre un modelo de fallos 'fail-stop' y 'Byzantine'?",
                    options: [
                        "Fail-stop es más rápido que Byzantine",
                        "En fail-stop los nodos fallan silenciosamente, en Byzantine pueden comportarse arbitrariamente",
                        "Byzantine solo ocurre en redes inalámbricas",
                        "No hay diferencia práctica entre ambos"
                    ],
                    correct: 1,
                    explanation: "En el modelo fail-stop, los nodos fallan de manera silenciosa y detectable. En Byzantine, los nodos pueden comportarse arbitrariamente, enviando mensajes contradictorios o maliciosos."
                },
                {
                    topic: 'detection',
                    question: "¿Qué problema fundamental demuestra el teorema FLP en sistemas asíncronos?",
                    options: [
                        "Es imposible detectar fallos de manera confiable",
                        "Es imposible lograr consenso determinista con una sola falla",
                        "Los sistemas distribuidos siempre son inconsistentes",
                        "La comunicación asíncrona es ineficiente"
                    ],
                    correct: 1,
                    explanation: "El teorema FLP demuestra que es imposible resolver el consenso de manera determinista en sistemas asíncronos donde al menos un proceso puede fallar, incluso por parada."
                },
                // Consenso (6 preguntas)
                {
                    topic: 'consensus',
                    question: "¿Cuántas fases tiene el protocolo Paxos clásico?",
                    options: [
                        "Una fase",
                        "Dos fases (Prepare/Promise y Accept/Accepted)",
                        "Tres fases",
                        "Depende del número de nodos"
                    ],
                    correct: 1,
                    explanation: "Paxos clásico tiene dos fases: Fase 1 (Prepare/Promise) para obtener permiso, y Fase 2 (Accept/Accepted) para proponer y aceptar el valor."
                },
                {
                    topic: 'consensus',
                    question: "¿Cuál es la garantía principal de la 'Leader Completeness Property' en Raft?",
                    options: [
                        "Siempre hay exactamente un líder",
                        "El líder tiene todas las entradas committed de términos anteriores",
                        "Los followers nunca se convierten en líderes",
                        "El líder nunca falla"
                    ],
                    correct: 1,
                    explanation: "La Leader Completeness Property garantiza que si una entrada fue committed en un término, estará presente en los logs de todos los líderes de términos superiores."
                },
                // Consistencia (7 preguntas)
                {
                    topic: 'consistency',
                    question: "Según el teorema CAP, ¿qué es imposible garantizar simultáneamente en presencia de particiones de red?",
                    options: [
                        "Consistencia y Disponibilidad",
                        "Consistencia y Tolerancia a Particiones",
                        "Disponibilidad y Tolerancia a Particiones",
                        "Las tres propiedades son siempre compatibles"
                    ],
                    correct: 0,
                    explanation: "El teorema CAP establece que durante una partición de red, debes elegir entre Consistencia (todos los nodos ven los mismos datos) y Disponibilidad (el sistema sigue respondiendo)."
                },
                {
                    topic: 'consistency',
                    question: "¿Qué garantiza la consistencia linealizable?",
                    options: [
                        "Las operaciones se ejecutan en orden de llegada",
                        "Cada operación parece ejecutarse instantáneamente en algún punto entre su inicio y finalización",
                        "Todos los nodos están siempre sincronizados",
                        "Las lecturas siempre devuelven el último valor escrito"
                    ],
                    correct: 1,
                    explanation: "La linealizabilidad garantiza que cada operación parece ejecutarse instantáneamente en algún punto entre su inicio y finalización, manteniendo un orden global consistente."
                },
                // Lock-Free (6 preguntas)
                {
                    topic: 'lockfree',
                    question: "¿Qué es el problema ABA en programación lock-free?",
                    options: [
                        "Un problema de rendimiento",
                        "Cuando un valor cambia de A a B y vuelve a A, ocultando cambios intermedios",
                        "Un error de compilación",
                        "Un problema de sincronización entre procesos"
                    ],
                    correct: 1,
                    explanation: "El problema ABA ocurre cuando un valor cambia de A a B y vuelve a A. Compare-and-Swap ve el mismo valor inicial y final pero no detecta los cambios intermedios."
                },
                {
                    topic: 'lockfree',
                    question: "¿Cuál es la diferencia entre memory_order_acquire y memory_order_release?",
                    options: [
                        "No hay diferencia práctica",
                        "Acquire previene reordering hacia arriba, release previene reordering hacia abajo",
                        "Acquire es más rápido que release",
                        "Release solo funciona en escrituras"
                    ],
                    correct: 1,
                    explanation: "memory_order_acquire previene que operaciones posteriores se reordenen antes de ella, mientras que memory_order_release previene que operaciones anteriores se reordenen después de ella."
                }
                // ... continuar con más preguntas para llegar a 25 total
            ];

            // Mezclar las preguntas para crear variedad
            this.questions = this.shuffleArray([...allQuestions]);
        },
        shuffleArray(array) {
            const shuffled = [...array];
            for (let i = shuffled.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
            }
            return shuffled;
        },
        startExam() {
            this.initializeQuestions();
            this.quizStarted = true;
            this.startTimer();
        },
        startTimer() {
            this.timer = setInterval(() => {
                this.timeRemaining--;
                if (this.timeRemaining <= 0) {
                    this.finishExam();
                }
            }, 1000);
        },
        formatTime(seconds) {
            const minutes = Math.floor(seconds / 60);
            const secs = seconds % 60;
            return `${minutes}:${secs.toString().padStart(2, '0')}`;
        },
        selectOption(index) {
            if (!this.showAnswer) {
                this.selectedAnswers[this.currentQuestion] = index;
            }
        },
        checkAnswer() {
            this.showAnswer = true;
            if (this.selectedAnswers[this.currentQuestion] === this.questions[this.currentQuestion].correct) {
                this.score++;
            }
        },
        nextQuestion() {
            if (this.currentQuestion < this.questions.length - 1) {
                this.currentQuestion++;
                this.showAnswer = false;
            }
        },
        previousQuestion() {
            if (this.currentQuestion > 0) {
                this.currentQuestion--;
                this.showAnswer = false;
            }
        },
        finishExam() {
            if (this.timer) {
                clearInterval(this.timer);
            }
            // Calcular puntuación de preguntas no respondidas
            for (let i = 0; i < this.questions.length; i++) {
                if (this.selectedAnswers[i] === this.questions[i].correct && !this.showAnswer) {
                    this.score++;
                }
            }
            this.showResults = true;
        },
        restartExam() {
            this.currentQuestion = 0;
            this.selectedAnswers = {};
            this.showAnswer = false;
            this.showResults = false;
            this.score = 0;
            this.timeRemaining = 2700;
            this.quizStarted = false;
            if (this.timer) {
                clearInterval(this.timer);
            }
        },
        getTopicName(topic) {
            const names = {
                'detection': 'Detección de Fallos',
                'consensus': 'Consenso',
                'consistency': 'Consistencia',
                'lockfree': 'Lock-Free'
            };
            return names[topic] || 'General';
        },
        getTopicScore(topic) {
            let score = 0;
            this.questions.forEach((q, index) => {
                if (q.topic === topic && this.selectedAnswers[index] === q.correct) {
                    score++;
                }
            });
            return score;
        },
        getLevel() {
            const percentage = (this.score / this.questions.length) * 100;
            if (percentage >= 90) return "Experto en Sistemas Distribuidos";
            if (percentage >= 80) return "Avanzado";
            if (percentage >= 70) return "Intermedio";
            return "Principiante";
        },
        goToEvaluation() {
            window.location.href = '/evaluacion/';
        },
        goToCourse() {
            window.location.href = '/';
        }
    },
    beforeUnmount() {
        if (this.timer) {
            clearInterval(this.timer);
        }
    }
}).mount('#quiz-app');
</script>

---

## 🎓 Certificación ODIN

### Criterios de Evaluación
- **Excelente (90-100%)**: Experto en Sistemas Distribuidos
- **Muy Bueno (80-89%)**: Nivel Avanzado
- **Bueno (70-79%)**: Nivel Intermedio
- **Insuficiente (<70%)**: Requiere más estudio

### Competencias Evaluadas
1. **🔍 Detección de Fallos**: Modelos de falla, detectores, timeouts
2. **🏛️ Algoritmos de Consenso**: Raft, Paxos, Byzantine Fault Tolerance
3. **🔄 Modelos de Consistencia**: CAP, PACELC, Linearizabilidad, Eventual Consistency
4. **🔓 Lock-Free Programming**: Operaciones atómicas, Memory Ordering, ABA Problem

### Próximos Pasos
- 🏆 **Completar Proyectos Prácticos**: Implementa algoritmos reales
- 📚 **Estudiar Papers Fundamentales**: Profundiza en la teoría
- 🛠️ **Contribuir a Proyectos Open Source**: Aplica tus conocimientos
- 🎯 **Especialización Avanzada**: Elige un área específica para dominar

---
*¿Completaste el examen? ¡Felicitaciones por llegar hasta aquí! Los sistemas distribuidos son un campo fascinante y desafiante.*
