---
title: "Quiz: Detección de Fallos"
description: "Evaluación sobre failure detectors y algoritmos de detección"
---

# 🚨 Quiz: Detección de Fallos

<div id="quiz-app">
    <div v-if="!showResults" class="quiz-container">
        <div class="quiz-header">
            <h3>Pregunta {{ currentQuestion + 1 }} de {{ questions.length }}</h3>
            <div class="progress-bar">
                <div class="progress-fill" :style="{ width: progressPercent + '%' }"></div>
            </div>
        </div>

        <div class="question-card">
            <h4>{{ questions[currentQuestion].question }}</h4>
            <div class="options">
                <div 
                    v-for="(option, index) in questions[currentQuestion].options" 
                    :key="index"
                    class="option"
                    :class="{ selected: selectedAnswers[currentQuestion] === index }"
                    @click="selectAnswer(index)"
                >
                    <input 
                        type="radio" 
                        :name="'q' + currentQuestion" 
                        :value="index"
                        :checked="selectedAnswers[currentQuestion] === index"
                    >
                    <span>{{ option }}</span>
                </div>
            </div>
        </div>

        <div class="quiz-controls">
            <button 
                @click="previousQuestion" 
                :disabled="currentQuestion === 0"
                class="btn btn-secondary"
            >
                ← Anterior
            </button>
            
            <button 
                v-if="currentQuestion < questions.length - 1"
                @click="nextQuestion"
                class="btn btn-primary"
            >
                Siguiente →
            </button>
            
            <button 
                v-else
                @click="submitQuiz"
                class="btn btn-success"
                :disabled="!allQuestionsAnswered"
            >
                Finalizar Quiz
            </button>
        </div>
    </div>

    <div v-if="showResults" class="results-container">
        <div class="score-display">
            <h2>📊 Resultados</h2>
            <div class="score-circle" :class="scoreClass">
                <span class="score-percentage">{{ scorePercent }}%</span>
            </div>
            <p class="score-message">{{ scoreMessage }}</p>
        </div>
        
        <div class="detailed-results">
            <h3>📝 Revisión Detallada</h3>
            <div 
                v-for="(question, index) in questions" 
                :key="index"
                class="review-item"
                :class="{ correct: isCorrect(index), incorrect: !isCorrect(index) }"
            >
                <h4>Pregunta {{ index + 1 }}: {{ isCorrect(index) ? '✅' : '❌' }}</h4>
                <p><strong>Pregunta:</strong> {{ question.question }}</p>
                <p><strong>Tu respuesta:</strong> {{ question.options[selectedAnswers[index]] }}</p>
                <p v-if="!isCorrect(index)"><strong>Respuesta correcta:</strong> {{ question.options[question.correct] }}</p>
                <p><strong>Explicación:</strong> {{ question.explanation }}</p>
            </div>
        </div>
        
        <div class="action-buttons">
            <button 
                v-if="scorePercent < 70" 
                @click="retryQuiz"
                class="btn btn-warning"
            >
                🔄 Reintentar
            </button>
            
            <button 
                v-if="scorePercent >= 70" 
                @click="nextTopic"
                class="btn btn-success"
            >
                ➡️ Siguiente Tema
            </button>
            
            <a href="index.md" class="btn btn-info">📚 Volver a Evaluaciones</a>
        </div>
    </div>
</div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
const { createApp, ref, computed } = Vue;

createApp({
    setup() {
        const questions = ref([
            {
                question: "¿Cuál es la principal limitación de los failure detectors en sistemas asíncronos?",
                options: [
                    "No pueden detectar ningún tipo de fallo",
                    "Es imposible distinguir entre un proceso lento y uno fallido",
                    "Solo funcionan en redes de alta velocidad",
                    "Requieren sincronización de relojes perfecta"
                ],
                correct: 1,
                explanation: "En sistemas asíncronos no hay límites de tiempo garantizados, por lo que un proceso que no responde podría estar lento o haber fallado."
            },
            {
                question: "En la clasificación de failure detectors, ¿qué significa 'Strong Completeness'?",
                options: [
                    "Ningún proceso correcto es sospechado",
                    "Todos los procesos correctos sospechan de cualquier proceso fallido",
                    "Solo un proceso correcto sospecha de procesos fallidos",
                    "Los fallos se detectan instantáneamente"
                ],
                correct: 1,
                explanation: "Strong Completeness requiere que eventualmente todos los procesos correctos sospechen de cualquier proceso que realmente haya fallado."
            },
            {
                question: "¿Qué ventaja principal tiene el φ-accrual failure detector sobre los heartbeat básicos?",
                options: [
                    "Usa menos ancho de banda",
                    "Proporciona valores de sospecha graduales en lugar de binarios",
                    "No requiere configuración",
                    "Funciona sin red"
                ],
                correct: 1,
                explanation: "φ-accrual produce un valor continuo de sospecha que se adapta a las condiciones de red, en lugar de una decisión binaria fallido/vivo."
            },
            {
                question: "En el protocolo SWIM, ¿qué sucede si el ping directo falla?",
                options: [
                    "El nodo se marca inmediatamente como fallido",
                    "Se reinicia el protocolo",
                    "Se realiza ping indirecto a través de k nodos aleatorios",
                    "Se espera al siguiente ciclo"
                ],
                correct: 2,
                explanation: "SWIM usa ping indirecto como segunda línea de defensa, pidiendo a otros nodos que verifiquen la conectividad antes de sospechar un fallo."
            },
            {
                question: "¿Cuál es el threshold típico de φ (phi) en production para φ-accrual?",
                options: [
                    "1.0 - 3.0",
                    "5.0 - 7.0", 
                    "8.0 - 12.0",
                    "15.0 - 20.0"
                ],
                correct: 2,
                explanation: "Valores de φ entre 8.0-12.0 proporcionan un buen balance entre detección rápida y falsos positivos en entornos de producción."
            },
            {
                question: "¿Por qué es importante agregar jitter a los heartbeats?",
                options: [
                    "Para mejorar la precisión",
                    "Para evitar thundering herd effects",
                    "Para ahorrar energía",
                    "Para mejorar la seguridad"
                ],
                correct: 1,
                explanation: "El jitter evita que todos los nodos envíen heartbeats simultáneamente, distribuyendo la carga de red y evitando congestión."
            },
            {
                question: "En SWIM, ¿qué información se propaga mediante gossip?",
                options: [
                    "Solo heartbeats",
                    "Información de membership (estado de nodos)",
                    "Datos de aplicación",
                    "Claves de encriptación"
                ],
                correct: 1,
                explanation: "SWIM usa gossip para propagar información de membership, permitiendo que los nodos se enteren de cambios en el estado del cluster."
            },
            {
                question: "¿Cuál es la complejidad de mensaje por nodo en SWIM?",
                options: [
                    "O(1)",
                    "O(log n)",
                    "O(n)",
                    "O(n²)"
                ],
                correct: 0,
                explanation: "SWIM mantiene complejidad O(1) por nodo, lo que lo hace escalable para clusters grandes, a diferencia de enfoques all-to-all."
            },
            {
                question: "¿Qué problema resuelve el 'incarnation number' en SWIM?",
                options: [
                    "Detección de fallos Byzantine",
                    "Prevenir falsos positivos cuando un nodo se recupera",
                    "Ordenamiento de mensajes",
                    "Balanceo de carga"
                ],
                correct: 1,
                explanation: "El incarnation number permite que un nodo que se recupera 'refute' sospechas sobre sí mismo, previniendo falsos positivos persistentes."
            },
            {
                question: "¿Cuál es la principal métrica para evaluar un failure detector?",
                options: [
                    "Throughput de mensajes",
                    "Trade-off entre detection time y false positive rate",
                    "Uso de memoria",
                    "Compatibilidad con protocolos"
                ],
                correct: 1,
                explanation: "Los failure detectors se evalúan principalmente por qué tan rápido detectan fallos reales vs. qué tan frecuentemente reportan falsos positivos."
            }
        ]);

        const currentQuestion = ref(0);
        const selectedAnswers = ref(new Array(questions.value.length).fill(null));
        const showResults = ref(false);

        const progressPercent = computed(() => {
            return ((currentQuestion.value + 1) / questions.value.length) * 100;
        });

        const allQuestionsAnswered = computed(() => {
            return selectedAnswers.value.every(answer => answer !== null);
        });

        const scorePercent = computed(() => {
            if (!showResults.value) return 0;
            const correct = selectedAnswers.value.filter((answer, index) => 
                answer === questions.value[index].correct
            ).length;
            return Math.round((correct / questions.value.length) * 100);
        });

        const scoreClass = computed(() => {
            const score = scorePercent.value;
            if (score >= 90) return 'excellent';
            if (score >= 80) return 'very-good';
            if (score >= 70) return 'good';
            if (score >= 60) return 'regular';
            return 'insufficient';
        });

        const scoreMessage = computed(() => {
            const score = scorePercent.value;
            if (score >= 90) return "🏆 ¡Excelente! Dominas los conceptos de detección de fallos.";
            if (score >= 80) return "🥇 ¡Muy bien! Tienes una comprensión sólida.";
            if (score >= 70) return "✅ ¡Bien! Has alcanzado el nivel mínimo requerido.";
            if (score >= 60) return "⚠️ Regular. Te recomendamos revisar el material.";
            return "❌ Necesitas estudiar más antes de continuar.";
        });

        function selectAnswer(index) {
            selectedAnswers.value[currentQuestion.value] = index;
        }

        function nextQuestion() {
            if (currentQuestion.value < questions.value.length - 1) {
                currentQuestion.value++;
            }
        }

        function previousQuestion() {
            if (currentQuestion.value > 0) {
                currentQuestion.value--;
            }
        }

        function submitQuiz() {
            if (!allQuestionsAnswered.value) {
                alert('Por favor responde todas las preguntas antes de finalizar.');
                return;
            }
            showResults.value = true;
        }

        function isCorrect(index) {
            return selectedAnswers.value[index] === questions.value[index].correct;
        }

        function retryQuiz() {
            currentQuestion.value = 0;
            selectedAnswers.value = new Array(questions.value.length).fill(null);
            showResults.value = false;
        }

        function nextTopic() {
            window.location.href = 'consenso-quiz.md';
        }

        return {
            questions,
            currentQuestion,
            selectedAnswers,
            showResults,
            progressPercent,
            allQuestionsAnswered,
            scorePercent,
            scoreClass,
            scoreMessage,
            selectAnswer,
            nextQuestion,
            previousQuestion,
            submitQuiz,
            isCorrect,
            retryQuiz,
            nextTopic
        };
    }
}).mount('#quiz-app');
</script>

<style>
.quiz-container, .results-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: #f9f9f9;
}

.quiz-header {
    margin-bottom: 20px;
}

.progress-bar {
    width: 100%;
    height: 10px;
    background-color: #e0e0e0;
    border-radius: 5px;
    overflow: hidden;
    margin-top: 10px;
}

.progress-fill {
    height: 100%;
    background-color: #4caf50;
    transition: width 0.3s ease;
}

.question-card {
    margin-bottom: 20px;
    padding: 20px;
    background: white;
    border-radius: 6px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.question-card h4 {
    margin-bottom: 15px;
    color: #333;
}

.options {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.option {
    display: flex;
    align-items: center;
    padding: 12px;
    border: 2px solid #ddd;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.2s;
    background: white;
}

.option:hover {
    border-color: #2196f3;
    background-color: #f0f8ff;
}

.option.selected {
    border-color: #4caf50;
    background-color: #e8f5e8;
}

.option input[type="radio"] {
    margin-right: 12px;
}

.quiz-controls {
    display: flex;
    justify-content: space-between;
    margin-top: 20px;
}

.btn {
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
    text-decoration: none;
    display: inline-block;
    transition: background-color 0.2s;
}

.btn-primary { background-color: #2196f3; color: white; }
.btn-secondary { background-color: #6c757d; color: white; }
.btn-success { background-color: #4caf50; color: white; }
.btn-warning { background-color: #ff9800; color: white; }
.btn-info { background-color: #17a2b8; color: white; }

.btn:disabled {
    background-color: #ccc;
    cursor: not-allowed;
}

.score-display {
    text-align: center;
    margin-bottom: 30px;
}

.score-circle {
    width: 120px;
    height: 120px;
    border-radius: 50%;
    border: 8px solid;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 20px auto;
    font-size: 24px;
    font-weight: bold;
    background-color: white;
}

.score-circle.excellent { border-color: #4caf50; color: #4caf50; }
.score-circle.very-good { border-color: #8bc34a; color: #8bc34a; }
.score-circle.good { border-color: #ffc107; color: #ffc107; }
.score-circle.regular { border-color: #ff9800; color: #ff9800; }
.score-circle.insufficient { border-color: #f44336; color: #f44336; }

.review-item {
    margin-bottom: 20px;
    padding: 15px;
    border-radius: 5px;
    border-left: 4px solid;
}

.review-item.correct {
    border-left-color: #4caf50;
    background-color: #f1f8e9;
}

.review-item.incorrect {
    border-left-color: #f44336;
    background-color: #ffebee;
}

.action-buttons {
    text-align: center;
    margin-top: 30px;
}

.action-buttons .btn {
    margin: 0 10px;
}
</style>
