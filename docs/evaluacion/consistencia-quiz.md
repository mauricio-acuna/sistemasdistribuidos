---
title: "Quiz: Modelos de Consistencia"
description: "Evaluación sobre consistencia eventual, CAP, PACELC y modelos de consistencia"
---

# 🔄 Quiz: Modelos de Consistencia

<div id="quiz-container">
    <div class="quiz-header">
        <h3>Pregunta <span id="current-question">1</span> de <span id="total-questions">12</span></h3>
        <div class="progress-bar">
            <div id="progress-fill"></div>
        </div>
    </div>

    <div id="question-container"></div>

    <div class="quiz-controls">
        <button id="prev-btn" onclick="previousQuestion()">← Anterior</button>
        <button id="next-btn" onclick="nextQuestion()">Siguiente →</button>
        <button id="submit-btn" onclick="submitQuiz()">Finalizar Quiz</button>
    </div>
</div>

<div id="results-container" style="display: none;">
    <div class="score-display">
        <h2>📊 Resultados del Quiz</h2>
        <div id="score-circle">
            <span id="score-percentage">0%</span>
        </div>
        <p id="score-message"></p>
    </div>
    
    <div class="detailed-results">
        <h3>📝 Revisión Detallada</h3>
        <div id="answers-review"></div>
    </div>
    
    <div class="action-buttons">
        <button id="retry-btn" onclick="retryQuiz()">🔄 Reintentar</button>
        <button id="next-topic-btn" onclick="goToNextTopic()">➡️ Siguiente Tema</button>
        <a href="index.md" class="btn">📚 Volver a Evaluaciones</a>
    </div>
</div>

<script>
const questions = [
    {
        question: "¿Qué significa el teorema CAP en sistemas distribuidos?",
        options: [
            "Consistency, Availability, Performance",
            "Consistency, Availability, Partition tolerance",
            "Concurrency, Atomicity, Performance",
            "Coordination, Availability, Persistence"
        ],
        correct: 1,
        explanation: "CAP significa Consistency (Consistencia), Availability (Disponibilidad) y Partition tolerance (Tolerancia a particiones). El teorema establece que solo puedes garantizar 2 de las 3 propiedades simultáneamente."
    },
    {
        question: "En el modelo de consistencia eventual, ¿qué se garantiza?",
        options: [
            "Que todas las lecturas devuelvan el último valor escrito",
            "Que si no hay más actualizaciones, eventualmente todas las réplicas convergerán al mismo estado",
            "Que las escrituras se propagan inmediatamente a todas las réplicas",
            "Que nunca habrá inconsistencias temporales"
        ],
        correct: 1,
        explanation: "La consistencia eventual garantiza que, en ausencia de nuevas actualizaciones, todas las réplicas eventualmente convergerán al mismo estado, pero permite inconsistencias temporales."
    },
    {
        question: "¿Qué añade PACELC al teorema CAP?",
        options: [
            "Consideration sobre performance",
            "Análisis del trade-off entre latencia y consistencia durante operación normal",
            "Soporte para transacciones ACID",
            "Métricas de disponibilidad mejoradas"
        ],
        correct: 1,
        explanation: "PACELC extiende CAP añadiendo que durante operación normal (sin particiones), hay un trade-off entre Latencia (L) y Consistencia (C): PAC-ELC."
    },
    {
        question: "¿Qué caracteriza a la consistencia linealizable?",
        options: [
            "Las operaciones aparecen como si fueran ejecutadas en algún orden secuencial",
            "Cada operación aparece como si fuera ejecutada instantáneamente en algún punto entre su inicio y fin",
            "Las lecturas siempre devuelven el último valor escrito por el mismo cliente",
            "Las transacciones se ejecutan de forma aislada"
        ],
        correct: 1,
        explanation: "La linealizabilidad requiere que cada operación aparezca como si fuera ejecutada atómicamente en algún punto entre su inicio y finalización, respetando el tiempo real."
    },
    {
        question: "En el modelo 'read-your-writes', ¿qué se garantiza?",
        options: [
            "Todas las réplicas se actualizan inmediatamente",
            "Un cliente siempre ve sus propias escrituras en lecturas posteriores",
            "Todas las escrituras son vistas por todos los clientes inmediatamente",
            "Las transacciones son serializables"
        ],
        correct: 1,
        explanation: "Read-your-writes garantiza que un cliente siempre verá el efecto de sus propias escrituras en lecturas subsecuentes, pero no garantiza ver escrituras de otros clientes."
    },
    {
        question: "¿Qué tipo de consistencia ofrece Amazon DynamoDB por defecto?",
        options: [
            "Consistencia fuerte",
            "Consistencia eventual",
            "Linealizabilidad",
            "Consistencia secuencial"
        ],
        correct: 1,
        explanation: "DynamoDB ofrece consistencia eventual por defecto para optimizar rendimiento y disponibilidad, aunque permite solicitar lecturas fuertemente consistentes."
    },
    {
        question: "En consistencia causal, ¿qué relación se debe preservar?",
        options: [
            "El orden temporal absoluto de todas las operaciones",
            "Las operaciones causalmente relacionadas deben verse en el mismo orden por todos",
            "Todas las escrituras deben ser vistas inmediatamente",
            "Solo las operaciones del mismo cliente deben ser ordenadas"
        ],
        correct: 1,
        explanation: "La consistencia causal preserva el orden de operaciones causalmente relacionadas. Si A causó B, todos los procesos verán A antes que B."
    },
    {
        question: "¿Qué significa 'strong consistency' en sistemas distribuidos?",
        options: [
            "Las operaciones son muy rápidas",
            "Todas las lecturas devuelven el valor de la escritura más reciente",
            "El sistema nunca falla",
            "Las transacciones son durables"
        ],
        correct: 1,
        explanation: "Strong consistency garantiza que todas las lecturas devuelvan el valor de la escritura más reciente que se completó antes del inicio de la lectura."
    },
    {
        question: "¿Cuál es la principal ventaja de la consistencia eventual?",
        options: [
            "Garantiza que nunca habrá datos inconsistentes",
            "Permite mayor disponibilidad y mejor rendimiento",
            "Simplifica la programación de aplicaciones",
            "Elimina la necesidad de replicación"
        ],
        correct: 1,
        explanation: "La consistencia eventual permite mayor disponibilidad y mejor rendimiento al no requerir coordinación sincrónica entre réplicas para cada operación."
    },
    {
        question: "¿Qué problema resuelven los 'vector clocks'?",
        options: [
            "Sincronización de tiempo físico",
            "Detección de relaciones causales entre eventos",
            "Compresión de logs de transacciones",
            "Balanceo de carga entre servidores"
        ],
        correct: 1,
        explanation: "Los vector clocks permiten determinar relaciones causales entre eventos en sistemas distribuidos, identificando si dos eventos son concurrentes o uno causó al otro."
    },
    {
        question: "En el contexto de BASE (Basically Available, Soft state, Eventual consistency), ¿qué significa 'Soft state'?",
        options: [
            "El estado se almacena en memoria volátil",
            "El estado puede cambiar sin entrada externa debido a propagación de actualizaciones",
            "El estado es fácil de modificar",
            "El estado se comprime para ahorrar espacio"
        ],
        correct: 1,
        explanation: "Soft state significa que el estado del sistema puede cambiar sin entrada externa, típicamente debido a la propagación eventual de actualizaciones entre réplicas."
    },
    {
        question: "¿Qué modelo de consistencia es más adecuado para un sistema de comentarios en redes sociales?",
        options: [
            "Linealizabilidad",
            "Consistencia eventual",
            "Consistencia secuencial",
            "Consistencia estricta"
        ],
        correct: 1,
        explanation: "Para comentarios en redes sociales, la consistencia eventual es adecuada porque es aceptable que los comentarios aparezcan con ligero retraso, priorizando disponibilidad y rendimiento."
    }
];

let currentQuestionIndex = 0;
let selectedAnswers = [];
let quizStarted = false;

function initializeQuiz() {
    selectedAnswers = new Array(questions.length).fill(null);
    currentQuestionIndex = 0;
    quizStarted = true;
    loadQuestion();
    updateUI();
}

function loadQuestion() {
    const question = questions[currentQuestionIndex];
    const container = document.getElementById('question-container');
    
    let optionsHTML = '';
    for (let i = 0; i < question.options.length; i++) {
        const isSelected = selectedAnswers[currentQuestionIndex] === i;
        optionsHTML += '<div class="option ' + (isSelected ? 'selected' : '') + '" onclick="selectAnswer(' + i + ')">';
        optionsHTML += '<input type="radio" name="answer" value="' + i + '" ' + (isSelected ? 'checked' : '') + '>';
        optionsHTML += '<span>' + question.options[i] + '</span>';
        optionsHTML += '</div>';
    }
    
    container.innerHTML = '<div class="question-card">' +
        '<h4>' + question.question + '</h4>' +
        '<div class="options">' + optionsHTML + '</div>' +
        '</div>';
}

function selectAnswer(index) {
    selectedAnswers[currentQuestionIndex] = index;
    loadQuestion();
}

function updateUI() {
    document.getElementById('current-question').textContent = currentQuestionIndex + 1;
    document.getElementById('total-questions').textContent = questions.length;
    
    const progress = ((currentQuestionIndex + 1) / questions.length) * 100;
    document.getElementById('progress-fill').style.width = progress + '%';
    
    document.getElementById('prev-btn').disabled = currentQuestionIndex === 0;
    document.getElementById('next-btn').style.display = 
        currentQuestionIndex === questions.length - 1 ? 'none' : 'inline-block';
    document.getElementById('submit-btn').style.display = 
        currentQuestionIndex === questions.length - 1 ? 'inline-block' : 'none';
}

function nextQuestion() {
    if (currentQuestionIndex < questions.length - 1) {
        currentQuestionIndex++;
        loadQuestion();
        updateUI();
    }
}

function previousQuestion() {
    if (currentQuestionIndex > 0) {
        currentQuestionIndex--;
        loadQuestion();
        updateUI();
    }
}

function submitQuiz() {
    for (let i = 0; i < selectedAnswers.length; i++) {
        if (selectedAnswers[i] === null) {
            alert('Por favor responde la pregunta ' + (i + 1) + ' antes de finalizar.');
            currentQuestionIndex = i;
            loadQuestion();
            updateUI();
            return;
        }
    }
    
    showResults();
}

function showResults() {
    let correctCount = 0;
    
    for (let i = 0; i < questions.length; i++) {
        if (selectedAnswers[i] === questions[i].correct) {
            correctCount++;
        }
    }
    
    const percentage = Math.round((correctCount / questions.length) * 100);
    
    document.getElementById('quiz-container').style.display = 'none';
    document.getElementById('results-container').style.display = 'block';
    
    document.getElementById('score-percentage').textContent = percentage + '%';
    
    const scoreCircle = document.getElementById('score-circle');
    const scoreMessage = document.getElementById('score-message');
    const retryBtn = document.getElementById('retry-btn');
    const nextBtn = document.getElementById('next-topic-btn');
    
    if (percentage >= 90) {
        scoreCircle.className = 'score-circle excellent';
        scoreMessage.textContent = '🏆 ¡Excelente! Dominas los modelos de consistencia.';
        retryBtn.style.display = 'none';
        nextBtn.style.display = 'inline-block';
    } else if (percentage >= 80) {
        scoreCircle.className = 'score-circle very-good';
        scoreMessage.textContent = '🥇 ¡Muy bien! Tienes una comprensión sólida de consistencia.';
        retryBtn.style.display = 'none';
        nextBtn.style.display = 'inline-block';
    } else if (percentage >= 70) {
        scoreCircle.className = 'score-circle good';
        scoreMessage.textContent = '✅ ¡Bien! Has alcanzado el nivel mínimo requerido.';
        retryBtn.style.display = 'none';
        nextBtn.style.display = 'inline-block';
    } else if (percentage >= 60) {
        scoreCircle.className = 'score-circle regular';
        scoreMessage.textContent = '⚠️ Regular. Te recomendamos revisar el material sobre consistencia.';
        retryBtn.style.display = 'inline-block';
        nextBtn.style.display = 'none';
    } else {
        scoreCircle.className = 'score-circle insufficient';
        scoreMessage.textContent = '❌ Necesitas estudiar más sobre modelos de consistencia antes de continuar.';
        retryBtn.style.display = 'inline-block';
        nextBtn.style.display = 'none';
    }
    
    showDetailedReview();
}

function showDetailedReview() {
    const reviewContainer = document.getElementById('answers-review');
    let reviewHTML = '';
    
    for (let i = 0; i < questions.length; i++) {
        const question = questions[i];
        const userAnswer = selectedAnswers[i];
        const isCorrect = userAnswer === question.correct;
        
        reviewHTML += '<div class="review-item ' + (isCorrect ? 'correct' : 'incorrect') + '">';
        reviewHTML += '<h4>Pregunta ' + (i + 1) + ': ' + (isCorrect ? '✅' : '❌') + '</h4>';
        reviewHTML += '<p><strong>Pregunta:</strong> ' + question.question + '</p>';
        reviewHTML += '<p><strong>Tu respuesta:</strong> ' + question.options[userAnswer] + '</p>';
        if (!isCorrect) {
            reviewHTML += '<p><strong>Respuesta correcta:</strong> ' + question.options[question.correct] + '</p>';
        }
        reviewHTML += '<p><strong>Explicación:</strong> ' + question.explanation + '</p>';
        reviewHTML += '</div>';
    }
    
    reviewContainer.innerHTML = reviewHTML;
}

function retryQuiz() {
    selectedAnswers = new Array(questions.length).fill(null);
    currentQuestionIndex = 0;
    
    document.getElementById('quiz-container').style.display = 'block';
    document.getElementById('results-container').style.display = 'none';
    
    loadQuestion();
    updateUI();
}

function goToNextTopic() {
    window.location.href = 'lock-free-quiz.md';
}

document.addEventListener('DOMContentLoaded', function() {
    initializeQuiz();
});
</script>

<style>
.quiz-container, .results-container {
    max-width: 800px;
    margin: 20px auto;
    padding: 30px;
    border: 1px solid #ddd;
    border-radius: 10px;
    background-color: #f9f9f9;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}

.quiz-header {
    margin-bottom: 25px;
    text-align: center;
}

.quiz-header h3 {
    margin-bottom: 15px;
    color: #333;
}

.progress-bar {
    width: 100%;
    height: 12px;
    background-color: #e0e0e0;
    border-radius: 6px;
    overflow: hidden;
}

#progress-fill {
    height: 100%;
    background: linear-gradient(90deg, #ff6b6b, #4ecdc4);
    transition: width 0.4s ease;
    width: 8.33%;
}

.question-card {
    background: white;
    padding: 25px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    margin-bottom: 20px;
}

.question-card h4 {
    margin-bottom: 20px;
    color: #333;
    font-size: 18px;
    line-height: 1.4;
}

.options {
    display: flex;
    flex-direction: column;
    gap: 12px;
}

.option {
    display: flex;
    align-items: center;
    padding: 15px;
    border: 2px solid #ddd;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.3s ease;
    background: white;
}

.option:hover {
    border-color: #ff6b6b;
    background-color: #fff5f5;
    transform: translateY(-1px);
}

.option.selected {
    border-color: #4ecdc4;
    background-color: #f0fdfa;
}

.option input[type="radio"] {
    margin-right: 15px;
    transform: scale(1.2);
}

.quiz-controls {
    display: flex;
    justify-content: space-between;
    margin-top: 25px;
}

.quiz-controls button {
    padding: 12px 24px;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-size: 16px;
    font-weight: 500;
    transition: all 0.2s ease;
}

#prev-btn {
    background-color: #6c757d;
    color: white;
}

#next-btn {
    background-color: #ff6b6b;
    color: white;
}

#submit-btn {
    background-color: #4ecdc4;
    color: white;
    display: none;
}

.quiz-controls button:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.quiz-controls button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
    transform: none;
    box-shadow: none;
}

.score-display {
    text-align: center;
    margin-bottom: 40px;
}

.score-circle {
    width: 140px;
    height: 140px;
    border-radius: 50%;
    border: 10px solid;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 25px auto;
    font-size: 28px;
    font-weight: bold;
    background-color: white;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.score-circle.excellent { border-color: #4ecdc4; color: #4ecdc4; }
.score-circle.very-good { border-color: #45b7b8; color: #45b7b8; }
.score-circle.good { border-color: #feca57; color: #feca57; }
.score-circle.regular { border-color: #ff9ff3; color: #ff9ff3; }
.score-circle.insufficient { border-color: #ff6b6b; color: #ff6b6b; }

#score-message {
    font-size: 18px;
    font-weight: 500;
    margin-top: 15px;
}

.detailed-results {
    margin-top: 30px;
}

.detailed-results h3 {
    margin-bottom: 20px;
    color: #333;
}

.review-item {
    margin-bottom: 25px;
    padding: 20px;
    border-radius: 8px;
    border-left: 5px solid;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.review-item.correct {
    border-left-color: #4ecdc4;
    background-color: #f0fdfa;
}

.review-item.incorrect {
    border-left-color: #ff6b6b;
    background-color: #fff5f5;
}

.review-item h4 {
    margin-bottom: 10px;
    color: #333;
}

.review-item p {
    margin-bottom: 8px;
    line-height: 1.5;
}

.action-buttons {
    text-align: center;
    margin-top: 40px;
}

.action-buttons button, 
.action-buttons .btn {
    margin: 0 10px;
    padding: 12px 24px;
    border: none;
    border-radius: 6px;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;
}

#retry-btn {
    background-color: #ff9ff3;
    color: white;
}

#next-topic-btn {
    background-color: #4ecdc4;
    color: white;
}

.btn {
    background-color: #ff6b6b;
    color: white;
}

.action-buttons button:hover,
.action-buttons .btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}
</style>
