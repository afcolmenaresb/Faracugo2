<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Misión Cumpleaños ❤️</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;600&display=swap');
        body { font-family: 'Poppins', sans-serif; background: #fdf2f8; }
        .hidden { display: none; }
        .fade-in { animation: fadeIn 0.5s ease-in; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">

    <div id="app-container" class="max-w-md w-full bg-white rounded-2xl shadow-xl p-8 text-center fade-in">
        
        <div id="login-screen">
            <h1 class="text-2xl font-bold text-pink-600 mb-4">¡Hola Cumpleañera! 🎉</h1>
            <p class="text-gray-600 mb-6">Para iniciar esta misión, ingresa el código que venía en tu postre.</p>
            <input type="number" id="access-code" placeholder="Código secreto" class="w-full p-3 border rounded-lg mb-4 text-center text-xl tracking-widest outline-none focus:border-pink-500">
            <button onclick="login()" class="w-full bg-pink-500 text-white py-3 rounded-lg font-semibold hover:bg-pink-600 transition">Entrar</button>
            <p id="login-error" class="text-red-500 text-sm mt-2 hidden">Código incorrecto 😢</p>
        </div>

        <div id="block-screen" class="hidden">
            <div class="text-6xl mb-4">⛔</div>
            <h2 class="text-xl font-bold text-red-500 mb-2">¡Respuesta Incorrecta!</h2>
            <p class="text-gray-600 mb-4">Debes esperar para intentar de nuevo. No intentes recargar, el tiempo sigue corriendo.</p>
            <div id="countdown" class="text-3xl font-mono font-bold text-gray-800">00:00</div>
        </div>

        <div id="quiz-screen" class="hidden">
            <div class="mb-4">
                <span id="progress-text" class="text-xs font-bold text-pink-400 tracking-widest uppercase">MISTERIO 1/3</span>
            </div>
            <h2 id="question-text" class="text-xl font-bold text-gray-800 mb-6">¿Pregunta?</h2>
            <div id="options-container" class="space-y-3">
                </div>
        </div>

        <div id="success-screen" class="hidden">
            <div class="text-6xl mb-4">✈️🎂</div>
            <h1 class="text-3xl font-bold text-pink-600 mb-2">¡SORPRESA!</h1>
            <p class="text-gray-700 mb-4">Prepara las maletas porque nos vamos...</p>
            
            <div class="bg-pink-50 rounded-lg p-4 text-left text-sm space-y-2 mb-4">
                <p>🗓️ <strong>11/06:</strong> Vuelo a <span class="text-pink-600 font-bold">San Andrés 🏝️</span></p>
                <p>🗓️ <strong>14/06:</strong> Rumbo a <span class="text-pink-600 font-bold">Bogotá, Desierto de la Tatacoa y Eje Cafetero ☕</span></p>
                <p>🗓️ <strong>20/06:</strong> Regreso a casa 🏠</p>
            </div>
            <p class="italic text-gray-500 text-xs">¡Te amo!</p>
        </div>

    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, doc, getDoc, setDoc, updateDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

        // --- 1. PEGA AQUÍ TU CONFIGURACIÓN DE FIREBASE ---
        const firebaseConfig = {
            apiKey: "AIzaSyA9PxB95Z9Vli6k93mpI_1Q72cau9IPDv0",
            authDomain: "sorpresacumple-5ef77.firebaseapp.com",
            projectId: "sorpresacumple-5ef77",
            storageBucket: "sorpresacumple-5ef77.firebasestorage.app",
            messagingSenderId: "169449956478",
            appId: "1:169449956478:web:11524ccde5969abbd8642f"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);

        // --- 2. TUS PREGUNTAS ---
        // Puedes agregar más. "correct" es el índice de la respuesta correcta (empieza en 0).
        const questions = [
            {
                q: "¿Cuál fue la fecha exacta de nuestra primera cita?",
                options: ["14 de Febrero", "10 de Octubre", "25 de Diciembre"], 
                correct: 1 // La opción correcta sería "10 de Octubre"
            },
            {
                q: "¿Qué comida pedimos la vez que nos reímos hasta llorar?",
                options: ["Pizza", "Sushi", "Hamburguesas"], 
                correct: 0
            },
            {
                q: "Si pudiéramos vivir en una película, ¿cuál ha dicho Andrés que sería?",
                options: ["Bee Movie", "Avengers", "Diario de una pasión"], 
                correct: 0 // Referencia a tu broma de Bee Movie ;)
            }
        ];

        // --- LÓGICA DEL SISTEMA ---
        let currentUserId = null;
        let timerInterval = null;

        window.login = async () => {
            const code = document.getElementById('access-code').value;
            // El código de la servilleta es 1106
            if (code !== "1106") {
                document.getElementById('login-error').classList.remove('hidden');
                return;
            }

            currentUserId = "novia_" + code; // ID único en la base de datos
            await checkStatus();
        };

        async function checkStatus() {
            const docRef = doc(db, "sorpresa", currentUserId);
            const docSnap = await getDoc(docRef);

            if (!docSnap.exists()) {
                // Primera vez que entra, creamos su registro
                await setDoc(docRef, { currentQuestion: 0, blockedUntil: null });
                showScreen('quiz-screen');
                loadQuestion(0);
            } else {
                const data = docSnap.data();
                
                // Verificamos si está bloqueada
                if (data.blockedUntil && data.blockedUntil.toMillis() > Date.now()) {
                    showBlockScreen(data.blockedUntil.toMillis());
                } else if (data.currentQuestion >= questions.length) {
                    showScreen('success-screen');
                } else {
                    showScreen('quiz-screen');
                    loadQuestion(data.currentQuestion);
                }
            }
            
            // Escuchar cambios en tiempo real (por si cambia de dispositivo)
            onSnapshot(docRef, (doc) => {
                const data = doc.data();
                if (data.blockedUntil && data.blockedUntil.toMillis() > Date.now()) {
                    showBlockScreen(data.blockedUntil.toMillis());
                }
            });
        }

        function loadQuestion(index) {
            document.getElementById('progress-text').innerText = `MISTERIO ${index + 1}/${questions.length}`;
            document.getElementById('question-text').innerText = questions[index].q;
            const container = document.getElementById('options-container');
            container.innerHTML = '';

            questions[index].options.forEach((opt, i) => {
                const btn = document.createElement('button');
                btn.className = "w-full p-4 text-left border rounded-lg hover:bg-pink-50 hover:border-pink-300 transition";
                btn.innerText = opt;
                btn.onclick = () => submitAnswer(index, i);
                container.appendChild(btn);
            });
        }

        async function submitAnswer(qIndex, answerIndex) {
            const isCorrect = questions[qIndex].correct === answerIndex;
            const docRef = doc(db, "sorpresa", currentUserId);

            if (isCorrect) {
                // Respuesta correcta: Avanzar
                if (qIndex + 1 >= questions.length) {
                    await updateDoc(docRef, { currentQuestion: qIndex + 1 });
                    showScreen('success-screen');
                } else {
                    await updateDoc(docRef, { currentQuestion: qIndex + 1 });
                    loadQuestion(qIndex + 1);
                }
            } else {
                // Respuesta incorrecta: BLOQUEO DE 10 MINUTOS
                // Cambia 10 * 60 * 1000 por el tiempo que quieras (en milisegundos)
                const releaseTime = new Date(Date.now() + 10 * 60 * 1000); 
                await updateDoc(docRef, { blockedUntil: releaseTime });
                showBlockScreen(releaseTime.getTime());
            }
        }

        function showBlockScreen(timestamp) {
            showScreen('block-screen');
            if (timerInterval) clearInterval(timerInterval);
            
            timerInterval = setInterval(() => {
                const now = Date.now();
                const diff = timestamp - now;

                if (diff <= 0) {
                    clearInterval(timerInterval);
                    checkStatus(); // Recargar estado para volver al quiz
                    return;
                }

                const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
                const seconds = Math.floor((diff % (1000 * 60)) / 1000);
                document.getElementById('countdown').innerText = 
                    `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
            }, 1000);
        }

        function showScreen(id) {
            ['login-screen', 'block-screen', 'quiz-screen', 'success-screen'].forEach(s => {
                document.getElementById(s).classList.add('hidden');
            });
            document.getElementById(id).classList.remove('hidden');
        }
    </script>
</body>
</html>1
