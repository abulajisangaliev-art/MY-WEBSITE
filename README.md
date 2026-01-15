<!DOCTYPE html>
<html lang="kk">
<head>
    <meta charset="UTF-8">
    <title>Ramadan Quest: Space & Drive</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        
        body {
            font-family: 'Segoe UI', sans-serif;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            /* Киношный ғарыш фоны */
            background: radial-gradient(circle at center, #1b2735 0%, #090a0f 100%);
            color: white;
        }

        /* Айналып тұрған планета */
        .planet {
            position: absolute;
            width: 400px;
            height: 400px;
            background: url('https://upload.wikimedia.org/wikipedia/commons/2/22/Earth_Western_Hemisphere_transparent_background.png');
            background-size: cover;
            border-radius: 50%;
            box-shadow: inset -20px -20px 50px #000, 0 0 50px rgba(0, 149, 255, 0.3);
            z-index: 1;
            animation: rotatePlanet 60s linear infinite;
            opacity: 0.7;
            right: -100px;
            top: -50px;
        }

        @keyframes rotatePlanet { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }

        /* Негізгі карта */
        .main-card {
            background: rgba(255, 255, 255, 0.05);
            backdrop-filter: blur(20px);
            width: 600px;
            padding: 40px;
            border-radius: 30px;
            border: 1px solid rgba(255,255,255,0.1);
            z-index: 10;
            text-align: center;
        }

        .task-box {
            text-align: left;
            background: rgba(255,255,255,0.1);
            padding: 15px;
            border-radius: 15px;
            margin: 10px 0;
            border-left: 4px solid #00d4ff;
        }

        /* Uber Машина анимациясы */
        .track {
            width: 100%;
            height: 40px;
            background: rgba(255,255,255,0.1);
            border-radius: 20px;
            margin: 20px 0;
            position: relative;
            overflow: hidden;
        }
        #car {
            position: absolute;
            left: 0;
            top: 5px;
            font-size: 25px;
            transition: 1s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }

        /* Тест */
        #quiz-modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.9);
            z-index: 100;
            justify-content: center; align-items: center;
        }
        .quiz-content { background: #1a1a1a; padding: 30px; border-radius: 20px; width: 400px; }
        .opt {
            display: block; width: 100%; padding: 12px; margin: 8px 0;
            background: #333; color: white; border: none; border-radius: 10px; cursor: pointer;
        }
        .correct { background: #0084ff !important; }
        .wrong { background: #ff3b3b !important; }

        .btn {
            background: #00d4ff; color: #000; border: none; padding: 15px 30px;
            border-radius: 10px; font-weight: bold; cursor: pointer; width: 100%;
        }
    </style>
</head>
<body>

    <div class="planet"></div>

    <div class="main-card">
        <h1 id="day-display">1-Күн</h1>
        
        <div class="track">
            <div id="car">🚕</div>
        </div>
        <p id="car-status">Uber: Сұрақтарға жауап беріп, межеге жет!</p>

        <div class="task-box">
            <small style="color:#00d4ff">КҮН АМАЛЫ</small>
            <p id="action-text">Жүктелуде...</p>
        </div>
        
        <div class="task-box">
            <small style="color:#00d4ff">ЗІКІР</small>
            <p id="zikir-text">Жүктелуде...</p>
        </div>

        <button class="btn" onclick="startQuiz()">Сұраққа жауап беру (0/5)</button>
        <button id="next-day-btn" class="btn" style="margin-top:10px; display:none; background:#4CAF50" onclick="showHadith()">Келесі күнге өту →</button>
    </div>

    <div id="quiz-modal">
        <div class="quiz-content">
            <h3 id="q-text" style="margin-bottom:15px"></h3>
            <div id="options-container"></div>
        </div>
    </div>

    <script>
        let currentDay = 1;
        let score = 0;
        let answeredQuestions = [];

        const questionsPool = [
            { q: "Пайғамбарымыз (с.а.у) қай қалада дүниеге келген?", a: ["Мекке", "Медине", "Шам"], c: 0 },
            { q: "Құранда неше сүре бар?", a: ["110", "114", "120"], c: 1 },
            { q: "Ораза парыз болған ай?", a: ["Мәуліт", "Рамазан", "Ражаб"], c: 1 },
            { q: "Ең алғашқы азаншы кім?", a: ["Біләл", "Әбу Бәкір", "Омар"], c: 0 },
            { q: "Мұсылманның бір күндік парыз намазы нешеу?", a: ["3", "5", "6"], c: 1 },
            { q: "Ең үлкен сүре қайсы?", a: ["Бақара", "Ясин", "Ықылас"], c: 0 },
            { q: "Зекет деген не?", a: ["Намаз түрі", "Міндетті садақа", "Ораза түрі"], c: 1 }
        ];

        const daysData = {
            1: { action: "Рамазанға ниет ету", zikir: "Субханаллаһ (100)", hadith: "Ораза - тозақтан қорғайтын қалқан." },
            2: { action: "Ауызашарда дұға жасау", zikir: "Әлхамдулиллаһ (100)", hadith: "Ауыз ашардағы дұға қайтарылмайды." }
        };

        function updateDisplay() {
            document.getElementById('day-display').innerText = `${currentDay}-Күн`;
            document.getElementById('action-text').innerText = daysData[currentDay]?.action || "Жақсылық жасау";
            document.getElementById('zikir-text').innerText = daysData[currentDay]?.zikir || "Ықылас сүресі";
            score = 0;
            updateCar();
            document.getElementById('next-day-btn').style.display = 'none';
        }

        function updateCar() {
            const car = document.getElementById('car');
            const progress = (score / 5) * 85; 
            car.style.left = progress + '%';
            if(score >= 5) {
                document.getElementById('car-status').innerText = "Uber келді! Тапсырма орындалды! ✅";
                document.getElementById('next-day-btn').style.display = 'block';
            } else {
                document.getElementById('car-status').innerText = `Сұрақтар: ${score}/5`;
            }
        }

        function startQuiz() {
            if(score >= 5) return alert("Бүгінгі Uber сапары аяқталды!");
            showQuestion();
        }

        function showQuestion() {
            const modal = document.getElementById('quiz-modal');
            const qData = questionsPool[Math.floor(Math.random() * questionsPool.length)];
            
            document.getElementById('q-text').innerText = qData.q;
            const container = document.getElementById('options-container');
            container.innerHTML = '';

            qData.a.forEach((opt, i) => {
                const b = document.createElement('button');
                b.className = 'opt';
                b.innerText = opt;
                b.onclick = () => {
                    if(i === qData.c) {
                        b.classList.add('correct');
                        score++;
                        confetti({ particleCount: 50, spread: 40 });
                        setTimeout(() => { modal.style.display='none'; updateCar(); }, 600);
                    } else {
                        b.classList.add('wrong');
                        setTimeout(() => { modal.style.display='none'; }, 600);
                    }
                };
                container.appendChild(b);
            });
            modal.style.display = 'flex';
        }

        function showHadith() {
            const h = daysData[currentDay]?.hadith || "Алла разы болсын!";
            alert(`МАША-АЛЛАҺ!\n\nХадис: ${h}`);
            currentDay++;
            updateDisplay();
        }

        updateDisplay();
    </script>
</body>
</html>
