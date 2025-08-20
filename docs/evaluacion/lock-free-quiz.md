---
title: "Quiz de Lock-Free Programming"
---

# Quiz de Lock-Free Programming
*Programación Sin Bloqueos: Operaciones Atómicas, Memory Ordering y Estructuras de Datos*

<style>
.quiz-container {
    background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%);
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
    border-color: #ff6b6b;
    background: #fff5f5;
}

.option.selected {
    border-color: #ff6b6b;
    background: #ffe6e6;
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
    background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%);
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
</style>

<div id="quiz-app" class="quiz-container">
    <h2>🔓 Quiz de Lock-Free Programming</h2>
    <p>Evalúa tu conocimiento sobre programación sin bloqueos, operaciones atómicas, memory ordering y estructuras de datos lock-free.</p>
    
    <div v-if="!quizStarted" class="text-center">
        <h3>📋 Instrucciones</h3>
        <ul style="text-align: left; max-width: 600px; margin: 0 auto;">
            <li>12 preguntas sobre programación lock-free</li>
            <li>Cubre operaciones atómicas, memory ordering, ABA problem, y más</li>
            <li>Necesitas 70% para aprobar (9/12 respuestas correctas)</li>
            <li>Puedes reintentar si no alcanzas el mínimo</li>
            <li>Tiempo estimado: 15-20 minutos</li>
        </ul>
        <button @click="startQuiz" class="quiz-button">🚀 Comenzar Quiz</button>
    </div>

    <div v-if="quizStarted && !showResults">
        <div class="progress-bar">
            <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
        </div>
        <p>Pregunta {{ currentQuestion + 1 }} de {{ questions.length }}</p>
        
        <div class="question-card">
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
            
            <div v-if="showAnswer" style="margin-top: 15px; padding: 15px; background: #ffe6e6; border-radius: 8px; color: #2d3748;">
                <strong>💡 Explicación:</strong> {{ questions[currentQuestion].explanation }}
            </div>
        </div>

        <div class="navigation-buttons">
            <button @click="previousQuestion" :disabled="currentQuestion === 0" class="quiz-button">⬅️ Anterior</button>
            <button @click="showAnswer ? nextQuestion() : checkAnswer()" class="quiz-button">
                {{ showAnswer ? 'Siguiente ➡️' : 'Verificar ✓' }}
            </button>
        </div>
    </div>

    <div v-if="showResults" class="result-container" :class="{ 'fail': score < passingScore }">
        <h3>🎯 Resultados del Quiz</h3>
        <div style="font-size: 2em; margin: 20px 0;">
            {{ score }}/{{ questions.length }} ({{ Math.round((score/questions.length)*100) }}%)
        </div>
        
        <div v-if="score >= passingScore">
            <h4>🎉 ¡Excelente trabajo!</h4>
            <p>Has demostrado un sólido conocimiento de programación lock-free y operaciones atómicas.</p>
            <p><strong>Próximo paso:</strong> ¡Completa el examen final integral!</p>
        </div>
        
        <div v-else>
            <h4>📚 Necesitas repasar un poco más</h4>
            <p>Revisa los conceptos de operaciones atómicas, memory ordering y el problema ABA.</p>
            <p><strong>Recomendación:</strong> Estudia las secciones sobre programación concurrente antes de reintentar.</p>
        </div>

        <div style="margin-top: 20px;">
            <button @click="restartQuiz" class="quiz-button">🔄 Reintentar Quiz</button>
            <button @click="goToEvaluation" class="quiz-button">📚 Volver a Evaluación</button>
            <button v-if="score >= passingScore" @click="goToFinalExam" class="quiz-button">🏆 Examen Final</button>
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
            passingScore: 9,
            questions: [
                {
                    question: "¿Cuál es la principal ventaja de la programación lock-free sobre el uso de mutexes?",
                    options: [
                        "Es más fácil de programar",
                        "Evita problemas como deadlocks, priority inversion y convoying",
                        "Siempre es más rápida",
                        "Usa menos memoria"
                    ],
                    correct: 1,
                    explanation: "La programación lock-free evita los problemas inherentes a los locks como deadlocks (interbloqueos), priority inversion (inversión de prioridades) y convoying (convoy de threads esperando el mismo lock)."
                },
                {
                    question: "¿Qué es una operación atómica en el contexto de programación concurrente?",
                    options: [
                        "Una operación que usa un solo hilo",
                        "Una operación que se ejecuta de manera indivisible sin interferencia",
                        "Una operación que es muy rápida",
                        "Una operación que no puede fallar"
                    ],
                    correct: 1,
                    explanation: "Una operación atómica es aquella que se ejecuta de manera indivisible, es decir, otros hilos ven la operación como si ocurriera instantáneamente sin estados intermedios."
                },
                {
                    question: "¿Qué es el problema ABA en programación lock-free?",
                    code: `// Ejemplo del problema ABA\nNode* top = stack.top;  // Lee A\n// Otro hilo: pop A, pop B, push A\nif (compare_and_swap(&stack.top, top, top->next)) {\n    // CAS exitoso, pero el estado cambió!\n}`,
                    options: [
                        "Un problema de rendimiento en operaciones atómicas",
                        "Cuando un valor cambia de A a B y luego de vuelta a A, ocultando modificaciones intermedias",
                        "Un error de compilación en código concurrent",
                        "Un problema de sincronización entre procesos"
                    ],
                    correct: 1,
                    explanation: "El problema ABA ocurre cuando un valor cambia de A a B y luego vuelve a A. Compare-and-Swap ve el mismo valor inicial y final, pero no detecta los cambios intermedios que pueden haber invalidado la operación."
                },
                {
                    question: "¿Cuál es la diferencia entre memory_order_relaxed y memory_order_seq_cst?",
                    options: [
                        "No hay diferencia práctica",
                        "relaxed permite reordering, seq_cst garantiza orden secuencial total",
                        "relaxed es más lento que seq_cst",
                        "seq_cst solo funciona en arquitecturas x86"
                    ],
                    correct: 1,
                    explanation: "memory_order_relaxed permite que las operaciones se reordenen, mientras que memory_order_seq_cst (sequential consistency) garantiza un orden total secuencial consistente entre todos los hilos."
                },
                {
                    question: "¿Qué es Compare-And-Swap (CAS) y por qué es fundamental en programación lock-free?",
                    code: `bool compare_and_swap(T* addr, T expected, T desired) {\n    if (*addr == expected) {\n        *addr = desired;\n        return true;\n    }\n    return false;\n}`,
                    options: [
                        "Una operación que siempre intercambia dos valores",
                        "Una primitiva atómica que actualiza un valor solo si coincide con el esperado",
                        "Un tipo de lock especial",
                        "Una técnica de optimización del compilador"
                    ],
                    correct: 1,
                    explanation: "CAS es una primitiva atómica que actualiza un valor en memoria solo si el valor actual coincide con un valor esperado. Es fundamental porque permite implementar algoritmos lock-free de manera segura."
                },
                {
                    question: "¿Cuál es una solución común para el problema ABA?",
                    options: [
                        "Usar locks tradicionales en su lugar",
                        "Usar versionado o hazard pointers",
                        "Evitar el uso de punteros",
                        "Usar solo memory_order_relaxed"
                    ],
                    correct: 1,
                    explanation: "El versionado (agregar un contador a cada puntero) o hazard pointers (marcadores de seguridad para punteros en uso) son soluciones comunes para prevenir el problema ABA en estructuras de datos lock-free."
                },
                {
                    question: "¿Qué garantiza memory_order_acquire en una operación atómica?",
                    options: [
                        "Que la operación sea más rápida",
                        "Que ninguna operación de lectura o escritura después de esta operación sea reordenada antes de ella",
                        "Que la operación sea atómica",
                        "Que solo un hilo pueda ejecutar la operación"
                    ],
                    correct: 1,
                    explanation: "memory_order_acquire actúa como una barrera de adquisición: ninguna operación de lectura o escritura que aparezca después de esta operación en el código puede ser reordenada para ejecutarse antes de ella."
                },
                {
                    question: "¿Cuál es la diferencia principal entre wait-free y lock-free?",
                    options: [
                        "Wait-free usa locks, lock-free no",
                        "Lock-free garantiza progreso del sistema, wait-free garantiza progreso de cada hilo individual",
                        "No hay diferencia, son sinónimos",
                        "Wait-free es más lento que lock-free"
                    ],
                    correct: 1,
                    explanation: "Lock-free garantiza que al menos un hilo del sistema hará progreso, mientras que wait-free garantiza que cada hilo individual hará progreso en un número finito de pasos."
                },
                {
                    question: "¿Qué es un hazard pointer?",
                    options: [
                        "Un puntero que causa errores de segmentación",
                        "Una técnica para proteger punteros de ser liberados mientras están en uso",
                        "Un tipo de puntero atómico",
                        "Un puntero que apunta a memoria compartida"
                    ],
                    correct: 1,
                    explanation: "Los hazard pointers son una técnica de gestión de memoria para estructuras lock-free que protege punteros de ser liberados mientras otros hilos puedan estar usándolos."
                },
                {
                    question: "¿Por qué las arquitecturas débilmente ordenadas (como ARM) presentan más desafíos para programación lock-free?",
                    options: [
                        "Tienen menos registros disponibles",
                        "Permiten más reordering de operaciones de memoria por hardware",
                        "No soportan operaciones atómicas",
                        "Son más lentas en general"
                    ],
                    correct: 1,
                    explanation: "Las arquitecturas débilmente ordenadas como ARM permiten que el hardware reordene operaciones de memoria más agresivamente, requiriendo barreras de memoria explícitas para garantizar el orden correcto."
                },
                {
                    question: "¿Cuál es el propósito de memory_order_release?",
                    code: `data.store(value, std::memory_order_relaxed);\nflag.store(true, std::memory_order_release); // Publica el dato`,
                    options: [
                        "Hacer la operación más lenta pero segura",
                        "Garantizar que todas las operaciones anteriores sean visibles antes de esta operación",
                        "Liberar memoria automáticamente",
                        "Prevenir que otros hilos lean el valor"
                    ],
                    correct: 1,
                    explanation: "memory_order_release actúa como una barrera de liberación: garantiza que todas las operaciones de lectura y escritura anteriores en el código sean visibles antes de que esta operación sea visible."
                },
                {
                    question: "¿Cuál es una desventaja significativa de los algoritmos lock-free?",
                    options: [
                        "Siempre son más lentos que las versiones con locks",
                        "Complejidad de implementación y debugging, posible starvation de hilos",
                        "No funcionan en sistemas multiprocesador",
                        "Requieren hardware especial"
                    ],
                    correct: 1,
                    explanation: "Los algoritmos lock-free son significativamente más complejos de implementar y debuggear correctamente. Además, aunque garantizan progreso del sistema, pueden sufrir de starvation donde algunos hilos individuales no progresan."
                }
            ]
        };
    },
    computed: {
        progressPercentage() {
            return ((this.currentQuestion + 1) / this.questions.length) * 100;
        }
    },
    methods: {
        startQuiz() {
            this.quizStarted = true;
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
            } else {
                this.showResults = true;
            }
        },
        previousQuestion() {
            if (this.currentQuestion > 0) {
                this.currentQuestion--;
                this.showAnswer = false;
            }
        },
        restartQuiz() {
            this.currentQuestion = 0;
            this.selectedAnswers = {};
            this.showAnswer = false;
            this.showResults = false;
            this.score = 0;
            this.quizStarted = false;
        },
        goToEvaluation() {
            window.location.href = '/evaluacion/';
        },
        goToFinalExam() {
            window.location.href = '/evaluacion/examen-completo/';
        }
    }
}).mount('#quiz-app');
</script>

---

## 📚 Recursos Adicionales

### Conceptos Fundamentales
- **Operaciones Atómicas**: fetch_add, compare_exchange, load/store
- **Memory Ordering**: acquire, release, relaxed, seq_cst
- **Problemas Comunes**: ABA problem, false sharing, memory reclamation

### Estructuras de Datos Lock-Free
- **Stack Lock-Free**: Implementación con CAS
- **Queue Lock-Free**: Algoritmo de Michael & Scott
- **Hash Tables**: Técnicas de redimensionamiento lock-free
- **Skip Lists**: Estructuras probabilísticas sin locks

### Herramientas y Técnicas
- **Hazard Pointers**: Gestión segura de memoria
- **Epochs**: Técnica alternativa para memory reclamation
- **Double-Width CAS**: Para evitar el problema ABA
- **Memory Barriers**: Barreras explícitas de memoria

### Librerías y Frameworks
- **Intel TBB**: Threading Building Blocks
- **Folly**: Librerías lock-free de Facebook
- **Boost.Lockfree**: Estructuras de datos sin locks
- **libcds**: Concurrent Data Structures library

### Próximos Pasos
1. 🏆 **[Examen Completo](examen-completo.md)** - Evaluación integral de todos los temas
2. 🛠️ **[Proyectos Prácticos](../proyectos/)** - Implementa estructuras lock-free
3. 📖 **[Bibliografía](../bibliografia/)** - Papers fundamentales sobre lock-free programming

---
*¿Completaste el quiz? ¡Increíble! La programación lock-free es uno de los temas más avanzados en concurrencia.*
