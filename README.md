# Games
משחקים אינטרקטיבים למצגות
<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>חידון אנרגיה מתחדשת - ניהול כיתה</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Assistant:wght@300;400;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Assistant', sans-serif;
            background: #0f172a;
            color: #f8fafc;
            min-height: 100vh;
        }
        .grid-board {
            display: grid;
            grid-template-columns: repeat(5, 1fr);
            gap: 12px;
            max-width: 1200px;
            margin: 0 auto;
        }
        .category-header {
            background: #1e293b;
            padding: 15px 5px;
            border-radius: 8px;
            text-align: center;
            font-weight: bold;
            border-bottom: 4px solid #10b981;
            height: 80px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.1rem;
        }
        .card {
            aspect-ratio: 16/10;
            background: rgba(30, 41, 59, 0.7);
            border: 2px solid #334155;
            border-radius: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            font-size: 1.8rem;
            cursor: pointer;
            transition: all 0.2s;
        }
        .card:hover:not(.disabled) {
            border-color: #10b981;
            transform: translateY(-3px);
            background: rgba(16, 185, 129, 0.1);
        }
        .card.disabled {
            opacity: 0.3;
            background: #0f172a;
            cursor: default;
        }
        .team-box {
            transition: all 0.3s;
            border: 4px solid transparent;
        }
        .team-active {
            transform: scale(1.05);
            box-shadow: 0 0 20px rgba(255, 255, 255, 0.2);
        }
        .modal-overlay {
            background: rgba(0,0,0,0.85);
            backdrop-filter: blur(5px);
        }
    </style>
</head>
<body class="p-4">

    <!-- Setup Screen -->
    <div id="setup-screen" class="fixed inset-0 z-[100] bg-slate-900 flex items-center justify-center p-6">
        <div class="bg-slate-800 p-8 rounded-2xl shadow-2xl max-w-md w-full border border-slate-700">
            <h1 class="text-3xl font-bold text-emerald-400 mb-6 text-center">הגדרת קבוצות</h1>
            
            <div class="space-y-6">
                <div>
                    <label class="block mb-2 font-bold">קבוצה 1:</label>
                    <div class="flex gap-2">
                        <input type="text" id="team1-name-input" value="קבוצה א'" class="bg-slate-700 border border-slate-600 rounded p-2 flex-grow text-white">
                        <input type="color" id="team1-color-input" value="#10b981" class="w-12 h-10 border-none rounded cursor-pointer">
                    </div>
                </div>
                <div>
                    <label class="block mb-2 font-bold">קבוצה 2:</label>
                    <div class="flex gap-2">
                        <input type="text" id="team2-name-input" value="קבוצה ב'" class="bg-slate-700 border border-slate-600 rounded p-2 flex-grow text-white">
                        <input type="color" id="team2-color-input" value="#3b82f6" class="w-12 h-10 border-none rounded cursor-pointer">
                    </div>
                </div>
                <button onclick="startGame()" class="w-full bg-emerald-500 hover:bg-emerald-400 text-slate-900 font-bold py-3 rounded-lg transition text-xl mt-4">מתחילים!</button>
            </div>
        </div>
    </div>

    <!-- Main Game UI -->
    <div id="game-ui" class="hidden">
        <header class="flex justify-between items-start mb-8 max-w-6xl mx-auto">
            <div id="team1-ui" class="team-box p-6 rounded-xl w-64 text-center">
                <h2 id="team1-display-name" class="text-2xl font-bold mb-2"></h2>
                <div class="text-4xl font-black" id="team1-score">0</div>
                <div id="turn-1" class="mt-2 text-sm font-bold opacity-0 uppercase tracking-widest">תור הפעולה!</div>
            </div>

            <div class="text-center">
                <h1 class="text-4xl font-black text-white mb-2">ECO-TRIVIA</h1>
                <p class="text-slate-400">ניהול מורה: בחר שאלה ותעד ניקוד</p>
                <button onclick="switchTurn()" class="mt-4 bg-slate-700 hover:bg-slate-600 px-4 py-1 rounded text-sm">החלף תור ידנית</button>
            </div>

            <div id="team2-ui" class="team-box p-6 rounded-xl w-64 text-center">
                <h2 id="team2-display-name" class="text-2xl font-bold mb-2"></h2>
                <div class="text-4xl font-black" id="team2-score">0</div>
                <div id="turn-2" class="mt-2 text-sm font-bold opacity-0 uppercase tracking-widest">תור הפעולה!</div>
            </div>
        </header>

        <div id="board" class="grid-board">
            <!-- Headers and Cards injected by JS -->
        </div>
    </div>

    <!-- Question Modal -->
    <div id="modal" class="fixed inset-0 hidden z-50 modal-overlay flex items-center justify-center p-4">
        <div class="bg-slate-800 border-2 border-slate-600 p-8 rounded-2xl max-w-3xl w-full shadow-2xl relative">
            <div class="flex justify-between mb-4">
                <span id="modal-category" class="text-emerald-400 font-bold"></span>
                <span id="modal-points" class="text-2xl font-black italic text-slate-500"></span>
            </div>
            
            <h2 id="question-text" class="text-2xl md:text-3xl font-bold mb-8 leading-tight text-center"></h2>

            <!-- Feedback Area -->
            <div id="feedback-box" class="hidden mb-6 p-4 rounded-lg text-center font-bold text-xl animate-bounce"></div>

            <div id="options-container" class="grid grid-cols-1 gap-3 mb-8"></div>

            <!-- Teacher Reveal Area (For Open Questions) -->
            <div id="answer-reveal-area" class="hidden mb-6 text-center">
                <button id="reveal-btn" onclick="revealAnswer()" class="bg-slate-700 hover:bg-slate-600 text-slate-300 px-4 py-2 rounded-lg text-sm mb-2 border border-slate-600 transition">הצג תשובה נכונה למורה</button>
                <div id="revealed-answer-text" class="hidden p-4 bg-slate-900/50 rounded-lg text-emerald-300 font-bold border border-emerald-900/30"></div>
            </div>

            <div id="teacher-controls" class="hidden border-t border-slate-700 pt-6">
                <div class="flex justify-center gap-4">
                    <button onclick="handleResult(true)" class="bg-emerald-600 hover:bg-emerald-500 px-8 py-3 rounded-xl font-bold text-white shadow-lg">נכון! (+ניקוד)</button>
                    <button onclick="handleResult(false)" class="bg-red-600 hover:bg-red-500 px-8 py-3 rounded-xl font-bold text-white shadow-lg">טעות (העבר תור)</button>
                </div>
            </div>

            <div id="post-check-controls" class="hidden flex justify-center gap-4">
                <button onclick="closeModal()" class="bg-slate-600 hover:bg-slate-500 px-8 py-2 rounded-lg">חזור ללוח</button>
                <button id="retry-btn" onclick="resetForRetry()" class="bg-amber-600 hover:bg-amber-500 px-8 py-2 rounded-lg">אפשר ניסיון נוסף</button>
            </div>
        </div>
    </div>

    <script>
        const categories = [
            "משבר האנרגיה ויסודות",
            "אנרגיה סולארית",
            "תחנות הידרואלקטריות",
            "תחנות רוח",
            "תחנות ביומסה וביוגז"
        ];

        const scoreSteps = [10, 25, 50, 80, 100];

        const questions = [
            // Cat 1
            { cat: 0, pts: 10, type: 'multi', q: "מהי ההגדרה המדויקת ביותר לאנרגיה מתחדשת לפי המקורות?", options: ["אנרגיה המופקת מנפט וגז טבעי.", "אנרגיה המופקת ממשאבים טבעיים שאינם נגמרים ואינם מזהמים, כמו שמש, מים ורוח.", "אנרגיה שניתן להשתמש בה רק פעם אחת.", "אנרגיה המופקת משריפת פחם בתחנות כוח."], a: 1 },
            { cat: 0, pts: 25, type: 'open', q: "ציינו שתי בעיות מרכזיות הקשורות לשימוש במקורות אנרגיה מתכלים (כמו פחם ונפט).", a: "זיהום אוויר, התחממות גלובלית, ופגיעה באנושות ובבעלי חיים." },
            { cat: 0, pts: 50, type: 'multi', q: "איזה יתרון כלכלי מצוין במקורות בנוגע לשימוש באנרגיה מתחדשת?", options: ["עלויות הקמה נמוכות מאוד מההתחממות.", "צורך תמידי בקניית דלקים יקרים.", "חיסכון כלכלי לאורך זמן בשל עלויות תחזוקה נמוכות לאחר ההשקעה הראשונית.", "היא תמיד זולה יותר מתחנות כוח פחמיות כבר ביום הראשון."], a: 2 },
            { cat: 0, pts: 80, type: 'open', q: "מדוע אנרגיה מתחדשת מסייעת לביטחון האנרגטי של מדינה?", a: "מכיוון שהיא אינה תלויה בדלקים שעלולים להיגמר (כמו נפט וגז), והמשאבים הטבעיים תמיד יהיו זמינים לנו." },
            { cat: 0, pts: 100, type: 'open', q: "הסבירו את הקשר בין שריפת דלקים מאובנים לבין 'התחממות גלובלית'.", a: "שריפת פחם/נפט פולטת גזי חממה המזהמים את האוויר וגורמים ללכידת חום ולהתחממות כדור הארץ." },
            // Cat 2
            { cat: 1, pts: 10, type: 'multi', q: "מהו השם הנוסף של פאנלים סולאריים לפי המקורות?", options: ["קולטים הידראוליים.", "לוחות פוטו-וולטאיים.", "טורבינות שמש.", "ממירים תרמיים."], a: 1 },
            { cat: 1, pts: 25, type: 'open', q: "מהו תפקידו של ה'ממיר' (Inverter) במערכת סולארית?", a: "להפוך את הזרם החשמלי הישיר (DC) הנוצר בפאנלים לזרם חילופין (AC) המשמש בבית." },
            { cat: 1, pts: 50, type: 'multi', q: "כיצד פועלת תחנה 'תרמו-סולארית'?", options: ["המרת אור השמש ישירות לחשמל באמצעות סיליקון.", "שימוש באור השמש ליצירת חום, שמרתיח מים לקיטור המניע טורבינה וגנרטור.", "ניצול הרוח הנוצרת מחום השמש.", "שאיבת מים למקום גבוה באמצעות חום השמש."], a: 1 },
            { cat: 1, pts: 80, type: 'open', q: "מהם שני החסרונות המרכזיים של ייצור חשמל מאנרגיה סולארית?", a: "אי-ייצור חשמל בלילה, וירידת יעילות משמעותית במזג אוויר מעונן/חורפי." },
            { cat: 1, pts: 100, type: 'open', q: "תארו את התהליך הפיזיקלי המתרחש בתוך תא פוטו-וולטאי בעת פגיעת אור השמש.", a: "פוטונים פוגעים בסיליקון, משחררים אלקטרונים, וכך יוצרים זרם חשמלי (DC)." },
            // Cat 3
            { cat: 2, pts: 10, type: 'multi', q: "מהו מקור המילה 'הידרו' ביוונית עתיקה?", options: ["רוח", "שמש", "מים", "חשמל"], a: 2 },
            { cat: 2, pts: 25, type: 'open', q: "הסבירו בקצרה איך נוצר חשמל בתחנה הידרואלקטרית.", a: "תנועת מים זורמים מסובבת טורבינה, שמסובבת גנרטור המייצר אנרגיה חשמלית." },
            { cat: 2, pts: 50, type: 'multi', q: "מה קורה בתחנת 'אגירה שאובה' כאשר יש עודף חשמל ברשת?", options: ["המים משוחררים לנהר.", "משתמשים בחשמל כדי לשאוב מים מהמאגר התחתון בחזרה לעליון.", "הטורבינות עובדות מהר יותר.", "המערכת נכבית לחלוטין."], a: 1 },
            { cat: 2, pts: 80, type: 'open', q: "ציינו שני סוגים של תחנות כוח המנצלות את מי הים.", a: "ייצור חשמל על ידי גלים, ותחנות המנצלות את הגאות והשפל." },
            { cat: 2, pts: 100, type: 'open', q: "מדוע בישראל מספר התחנות ההידרואלקטריות מוגבל יחסית לעולם?", a: "מקורות מים מצומצמים, חוסר בנחלים עם זרימה חזקה או הפרשי גובה מספיקים." },
            // Cat 4
            { cat: 3, pts: 10, type: 'multi', q: "איזו המרת אנרגיה מתבצעת בטורבינת רוח?", options: ["כימית לחשמלית.", "קינטית (תנועה) לחשמלית.", "חום למכנית.", "פוטנציאלית לקול."], a: 1 },
            { cat: 3, pts: 25, type: 'open', q: "מדוע מתארים טורבינת רוח כ'הפוכה ממאוורר'?", a: "מאוורר משתמש בחשמל ליצירת רוח, טורבינה משתמשת ברוח ליצירת חשמל." },
            { cat: 3, pts: 50, type: 'open', q: "מהו החלק שמעלה את מהירות הסיבוב פי כמה וכמה בתוך הטורבינה?", a: "תיבת הילוכים (Gearbox) עם גלגלי שיניים שמכפילה את המהירות מ-12 לערך 100 סיבובים לדקה." },
            { cat: 3, pts: 80, type: 'open', q: "ציינו השפעה שלילית אחת שיכולה להיות לטורבינות רוח.", a: "פגיעה בעופות ועטלפים בגלל הלהבים או הרעש." },
            { cat: 3, pts: 100, type: 'open', q: "תנו דוגמה הממחישה את הגודל העצום של טורבינה מודרנית.", a: "גובה של 80-120 מטר (בניין 30 קומות) או להב באורך 50-60 מטר." },
            // Cat 5
            { cat: 4, pts: 10, type: 'multi', q: "מה נחשב כ'ביומסה' לפי המקורות?", options: ["סלעים ומינרלים.", "חומר אורגני מצמחים, בעלי חיים או פסולת שלהם.", "פלסטיק וזכוכית.", "גז טבעי מהים."], a: 1 },
            { cat: 4, pts: 25, type: 'open', q: "מהו הגז הדליק שנוצר בתהליך פירוק הפסולת האורגנית?", a: "גז מתאן (CH4)." },
            { cat: 4, pts: 50, type: 'multi', q: "מהו 'עיכול אנאירובי'?", options: ["שריפת פסולת.", "תסיסה של חומר אורגני בסביבה ללא חמצן בעזרת חיידקים.", "מיון פסולת.", "ייבוש פסולת בשמש."], a: 1 },
            { cat: 4, pts: 80, type: 'open', q: "איזה אתר בישראל מוזכר כדוגמה להפקת אנרגיה מפסולת בשיטה זו?", a: "פארק המיחזור חיריה." },
            { cat: 4, pts: 100, type: 'open', q: "מהו היתרון המרכזי של ביומסה בטיפול באשפה?", a: "הפיכת מטרד סביבתי (פסולת פולטת גזי חממה) למשאב לייצור חשמל וחום." },
        ];

        const praiseMessages = ["אלופים!", "כל הכבוד!", "תשובה מדויקת!", "פשוט מצוין!", "ידע זה כוח!"];
        const encourageMessages = ["לא נורא, בפעם הבאה!", "כמעט! המשיכו לנסות.", "טעות היא פתח ללמידה.", "אל תוותרו, אתם בדרך הנכונה!"];

        let team1 = { name: "קבוצה א'", score: 0, color: "#10b981" };
        let team2 = { name: "קבוצה ב'", score: 0, color: "#3b82f6" };
        let activeTeam = 1;
        let currentQ = null;

        function startGame() {
            team1.name = document.getElementById('team1-name-input').value;
            team1.color = document.getElementById('team1-color-input').value;
            team2.name = document.getElementById('team2-name-input').value;
            team2.color = document.getElementById('team2-color-input').value;

            document.getElementById('team1-display-name').innerText = team1.name;
            document.getElementById('team1-ui').style.color = team1.color;
            document.getElementById('team1-ui').style.borderColor = team1.color;
            
            document.getElementById('team2-display-name').innerText = team2.name;
            document.getElementById('team2-ui').style.color = team2.color;
            document.getElementById('team2-ui').style.borderColor = team2.color;

            document.getElementById('setup-screen').classList.add('hidden');
            document.getElementById('game-ui').classList.remove('hidden');
            
            initBoard();
            updateTurnUI();
        }

        function initBoard() {
            const board = document.getElementById('board');
            board.innerHTML = '';
            
            categories.forEach(cat => {
                const head = document.createElement('div');
                head.className = 'category-header';
                head.innerText = cat;
                board.appendChild(head);
            });

            scoreSteps.forEach(pts => {
                for (let i = 0; i < 5; i++) {
                    const card = document.createElement('div');
                    card.className = 'card';
                    card.id = `card-${i}-${pts}`;
                    card.innerText = pts;
                    card.onclick = () => openQuestion(i, pts);
                    board.appendChild(card);
                }
            });
        }

        function updateTurnUI() {
            document.getElementById('team1-ui').classList.toggle('team-active', activeTeam === 1);
            document.getElementById('team2-ui').classList.toggle('team-active', activeTeam === 2);
            document.getElementById('turn-1').style.opacity = activeTeam === 1 ? '1' : '0';
            document.getElementById('turn-2').style.opacity = activeTeam === 2 ? '1' : '0';
        }

        function switchTurn() {
            activeTeam = activeTeam === 1 ? 2 : 1;
            updateTurnUI();
        }

        function openQuestion(cat, pts) {
            const cardId = `card-${cat}-${pts}`;
            if (document.getElementById(cardId).classList.contains('disabled')) return;

            currentQ = questions.find(q => q.cat === cat && q.pts === pts);
            
            document.getElementById('modal-category').innerText = categories[cat];
            document.getElementById('modal-points').innerText = pts + " נק'";
            document.getElementById('question-text').innerText = currentQ.q;
            
            const optCont = document.getElementById('options-container');
            optCont.innerHTML = '';
            document.getElementById('teacher-controls').classList.add('hidden');
            document.getElementById('post-check-controls').classList.add('hidden');
            document.getElementById('feedback-box').classList.add('hidden');
            
            // Open Question Reset
            document.getElementById('answer-reveal-area').classList.add('hidden');
            document.getElementById('revealed-answer-text').classList.add('hidden');
            document.getElementById('reveal-btn').classList.remove('hidden');

            if (currentQ.type === 'multi') {
                currentQ.options.forEach((opt, idx) => {
                    const btn = document.createElement('button');
                    btn.className = 'w-full text-right p-4 rounded-xl border-2 border-slate-700 hover:border-slate-500 bg-slate-700/50 transition text-lg';
                    btn.innerText = opt;
                    btn.onclick = () => checkMulti(idx, btn);
                    optCont.appendChild(btn);
                });
            } else {
                document.getElementById('answer-reveal-area').classList.remove('hidden');
                document.getElementById('revealed-answer-text').innerText = currentQ.a;
                document.getElementById('teacher-controls').classList.remove('hidden');
            }

            document.getElementById('modal').classList.remove('hidden');
        }

        function revealAnswer() {
            document.getElementById('revealed-answer-text').classList.remove('hidden');
            document.getElementById('reveal-btn').classList.add('hidden');
        }

        function checkMulti(idx, btnElement) {
            const isCorrect = idx === currentQ.a;
            const btns = document.querySelectorAll('#options-container button');
            btns.forEach(b => b.disabled = true);

            if (isCorrect) {
                btnElement.classList.add('bg-emerald-600', 'border-emerald-400');
                showFeedback(true);
                addScore();
                document.getElementById(`card-${currentQ.cat}-${currentQ.pts}`).classList.add('disabled');
            } else {
                btnElement.classList.add('bg-red-600', 'border-red-400');
                showFeedback(false);
            }
            
            document.getElementById('post-check-controls').classList.remove('hidden');
        }

        function handleResult(isCorrect) {
            if (isCorrect) {
                showFeedback(true);
                addScore();
                document.getElementById(`card-${currentQ.cat}-${currentQ.pts}`).classList.add('disabled');
            } else {
                showFeedback(false);
                switchTurn();
            }
            document.getElementById('teacher-controls').classList.add('hidden');
            document.getElementById('post-check-controls').classList.remove('hidden');
        }

        function showFeedback(isCorrect) {
            const fb = document.getElementById('feedback-box');
            fb.classList.remove('hidden');
            if (isCorrect) {
                fb.innerText = praiseMessages[Math.floor(Math.random() * praiseMessages.length)];
                fb.className = "mb-6 p-4 rounded-lg text-center font-bold text-2xl animate-bounce text-emerald-400";
            } else {
                fb.innerText = encourageMessages[Math.floor(Math.random() * encourageMessages.length)];
                fb.className = "mb-6 p-4 rounded-lg text-center font-bold text-xl text-amber-400";
            }
        }

        function addScore() {
            if (activeTeam === 1) {
                team1.score += currentQ.pts;
                document.getElementById('team1-score').innerText = team1.score;
            } else {
                team2.score += currentQ.pts;
                document.getElementById('team2-score').innerText = team2.score;
            }
        }

        function resetForRetry() {
            openQuestion(currentQ.cat, currentQ.pts);
        }

        function closeModal() {
            document.getElementById('modal').classList.add('hidden');
        }
    </script>
</body>
</html>
