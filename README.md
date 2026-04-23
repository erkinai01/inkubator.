<!DOCTYPE html>
<html lang="kk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Инкубациялық циклды басқару</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700;900&display=swap');
        body { font-family: 'Inter', sans-serif; background-color: #0f172a; color: #f1f5f9; }
        .egg-shape { border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%; }
    </style>
</head>
<body>
    <div id="app" class="min-h-screen flex flex-col">
        <!-- Жүктелу индикаторы басында көрінеді -->
        <div id="loading-screen" class="fixed inset-0 flex flex-col items-center justify-center z-[100] bg-[#0f172a]">
            <div class="animate-spin text-orange-500 mb-4">
                <i data-lucide="refresh-cw" size="48"></i>
            </div>
            <p class="text-slate-400 animate-pulse uppercase tracking-widest text-xs">Жүктелуде...</p>
        </div>

        <header class="bg-[#1e293b] border-b border-slate-700 px-6 py-4 sticky top-0 z-50 shadow-lg">
            <div class="max-w-5xl mx-auto flex justify-between items-center">
                <div class="flex items-center gap-3">
                    <span class="text-2xl">🐤</span>
                    <h1 class="text-xl font-bold bg-gradient-to-r from-orange-400 to-amber-300 bg-clip-text text-transparent">Инкубатор</h1>
                </div>
                <div class="text-xs text-slate-400 hidden sm:block">Бақылау жүйесі v3.0</div>
            </div>
        </header>

        <main id="main-content" class="max-w-5xl mx-auto px-4 py-8 flex-grow w-full hidden">
            <div id="setup-view" class="hidden">
                <div class="bg-[#1e293b] rounded-[2.5rem] p-10 max-w-lg mx-auto border border-slate-700 text-center shadow-2xl">
                    <h2 class="text-2xl font-bold mb-6">Жаңа инкубацияны бастау</h2>
                    <p class="text-slate-400 mb-8">Жұмыртқаларды салған күнді таңдаңыз:</p>
                    <input type="date" id="start-date-input" class="w-full p-4 rounded-2xl bg-slate-900 border-2 border-slate-700 text-white mb-6 outline-none focus:border-orange-500">
                    <button onclick="startIncubation()" class="w-full bg-orange-600 hover:bg-orange-700 py-4 rounded-2xl font-bold transition-all shadow-lg">Бастау</button>
                </div>
            </div>

            <div id="dashboard-view" class="grid grid-cols-1 lg:grid-cols-12 gap-8 hidden">
                <div class="lg:col-span-4 space-y-6">
                    <div class="bg-[#1e293b] rounded-[2.5rem] p-8 border border-slate-700 flex flex-col items-center shadow-xl">
                        <div class="relative w-48 h-64 flex items-center justify-center">
                            <div class="absolute inset-0 bg-slate-100 egg-shape border-4 border-slate-300 shadow-inner overflow-hidden">
                                <div id="embryo-visual" class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-16 h-20 bg-red-200 rounded-full transition-all duration-700"></div>
                            </div>
                        </div>
                        <div class="mt-6 text-center">
                            <div id="display-day" class="text-6xl font-black text-orange-400">1</div>
                            <div class="text-slate-500 uppercase text-[10px] tracking-widest font-bold">Күндік даму</div>
                        </div>
                    </div>
                </div>

                <div class="lg:col-span-8 space-y-6">
                    <div class="bg-[#1e293b] rounded-[2.5rem] p-8 border border-slate-700 shadow-xl">
                        <h2 id="stage-title" class="text-3xl font-bold mb-4">Дамудың басталуы</h2>
                        <p id="stage-desc" class="text-slate-400 mb-8 leading-relaxed"></p>
                        
                        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 mb-8">
                            <div class="bg-slate-900/50 p-4 rounded-2xl border border-slate-700">
                                <i data-lucide="thermometer" class="text-orange-400 mb-2"></i>
                                <div class="text-[10px] text-slate-500 font-bold uppercase mb-1">Температура</div>
                                <div id="val-temp" class="text-xl font-bold">37.8°C</div>
                            </div>
                            <div class="bg-slate-900/50 p-4 rounded-2xl border border-slate-700">
                                <i data-lucide="droplets" class="text-blue-400 mb-2"></i>
                                <div class="text-[10px] text-slate-500 font-bold uppercase mb-1">Ылғалдылық</div>
                                <div id="val-hum" class="text-xl font-bold">55%</div>
                            </div>
                            <div class="bg-slate-900/50 p-4 rounded-2xl border border-slate-700">
                                <i id="turn-icon" data-lucide="rotate-cw" class="text-emerald-400 mb-2"></i>
                                <div class="text-[10px] text-slate-500 font-bold uppercase mb-1">Айналдыру</div>
                                <div id="val-turn" class="text-xl font-bold">Қажет</div>
                            </div>
                        </div>
                        <button onclick="resetIncubation()" class="text-xs text-red-400 hover:underline flex items-center gap-1 opacity-50 hover:opacity-100">
                            <i data-lucide="refresh-cw" size="12"></i> Циклды тоқтату/қайта бастау
                        </button>
                    </div>

                    <div class="bg-[#1e293b] rounded-[2rem] p-6 border border-slate-700 overflow-x-auto">
                        <div class="flex gap-2 min-w-max" id="days-grid"></div>
                    </div>
                </div>
            </div>
        </main>

        <footer class="py-8 border-t border-slate-800/50 mt-auto">
            <div class="max-w-5xl mx-auto px-6 text-center">
                <p class="text-slate-500 text-xs font-bold uppercase tracking-widest mb-1">Жоба авторы</p>
                <p class="text-slate-300 font-medium">Исаханова Еркинай Кайыпрахмановна</p>
            </div>
        </footer>
    </div>

    <script>
        const incubationData = [
            { day: 1, temp: '37.8°C', hum: '55%', turn: true, title: 'Дамудың басталуы', desc: 'Жасушалар бөлініп, бластодерма түзіледі.', emoji: '🥚' },
            { day: 2, temp: '37.8°C', hum: '55%', turn: true, title: 'Қан тамырлары', desc: 'Жүрек соға бастайды. Бірінші қан тамырлары пайда болады.', emoji: '🥚' },
            { day: 3, temp: '37.8°C', hum: '55%', turn: true, title: 'Айналу басталды', desc: 'Эмбрион оң жағына бұрылады. Ми қалыптасады.', emoji: '🥚' },
            { day: 4, temp: '37.8°C', hum: '55%', turn: true, title: 'Ағзалардың бастауы', desc: 'Ми, көз және құлақ дамиды.', emoji: '🥚' },
            { day: 5, temp: '37.8°C', hum: '55%', turn: true, title: 'Көздердің қарайуы', desc: 'Көздері пигментацияланып, қара нүкте болып көрінеді.', emoji: '🥚' },
            { day: 6, temp: '37.8°C', hum: '55%', turn: true, title: 'Тұмсық пен қозғалыс', desc: 'Тұмсық пайда болады. Эмбрион қозғалады.', emoji: '🥚' },
            { day: 7, temp: '37.8°C', hum: '55%', turn: true, title: 'Аяқ-қолдар', desc: 'Аяқтары мен қанаттары ұзарады.', emoji: '🥚' },
            { day: 8, temp: '37.8°C', hum: '55%', turn: true, title: 'Қауырсын нышаны', desc: 'Қауырсын өсетін фолликулалар пайда болады.', emoji: '🥚' },
            { day: 9, temp: '37.8°C', hum: '55%', turn: true, title: 'Құсқа ұқсау', desc: 'Эмбрион енді нағыз балапанға ұқсай бастайды.', emoji: '🥚' },
            { day: 10, temp: '37.8°C', hum: '55%', turn: true, title: 'Тырнақтар', desc: 'Тұмсығы қатая бастайды.', emoji: '🥚' },
            { day: 11, temp: '37.8°C', hum: '55%', turn: true, title: 'Қауырсынның өсуі', desc: 'Денесінде алғашқы мамық қауырсындар көрінеді.', emoji: '🥚' },
            { day: 12, temp: '37.8°C', hum: '55%', turn: true, title: 'Қозғалыстың артуы', desc: 'Балапан жұмыртқа ішінде белсенді.', emoji: '🥚' },
            { day: 13, temp: '37.8°C', hum: '55%', turn: true, title: 'Қаңқаның қатаюы', desc: 'Сүйектері кальций жинап, қатаяды.', emoji: '🥚' },
            { day: 14, temp: '37.8°C', hum: '55%', turn: true, title: 'Басын бұру', desc: 'Балапан басын ауа камерасына қарай бұрады.', emoji: '🥚' },
            { day: 15, temp: '37.8°C', hum: '55%', turn: true, title: 'Мамықтар', desc: 'Балапан толығымен мамықпен жабылады.', emoji: '🥚' },
            { day: 16, temp: '37.8°C', hum: '55%', turn: true, title: 'Сарыуызды игеру', desc: 'Сарыуыз балапанның ішіне тартыла бастайды.', emoji: '🥚' },
            { day: 17, temp: '37.8°C', hum: '55%', turn: true, title: 'Демалуға дайындық', desc: 'Балапан тұмсығын ауа камерасына жақындатады.', emoji: '🥚' },
            { day: 18, temp: '37.5°C', hum: '70%', turn: false, title: 'Лакдаун (Тоқтату)', desc: 'Бұруды тоқтатыңыз! Балапан шығуға дайындалады.', emoji: '🐣' },
            { day: 19, temp: '37.5°C', hum: '70%', turn: false, title: 'Ауамен тыныстау', desc: 'Өкпесі жұмыс істей бастайды.', emoji: '🐣' },
            { day: 20, temp: '37.5°C', hum: '70%', turn: false, title: 'Тесу (Нақлу)', desc: 'Балапан қабықты тесе бастайды.', emoji: '🐣' },
            { day: 21, temp: '37.2°C', hum: '75%', turn: false, title: 'Туылу!', desc: 'Балапан қабықты жарып шығады. Құттықтаймыз!', emoji: '🐥' }
        ];

        let startDate = localStorage.getItem('incubator_start_date');
        let selectedDay = 1;

        function init() {
            lucide.createIcons();
            document.getElementById('loading-screen').classList.add('hidden');
            document.getElementById('main-content').classList.remove('hidden');
            updateUI();
        }

        function updateUI() {
            if (!startDate) {
                document.getElementById('setup-view').classList.remove('hidden');
                document.getElementById('dashboard-view').classList.add('hidden');
            } else {
                document.getElementById('setup-view').classList.add('hidden');
                document.getElementById('dashboard-view').classList.remove('hidden');
                
                const start = new Date(startDate);
                const today = new Date();
                const diff = Math.floor((today - start) / (1000 * 60 * 60 * 24)) + 1;
                const currentDay = diff > 21 ? 21 : (diff < 1 ? 1 : diff);
                
                if (selectedDay === 1 && diff > 1) selectedDay = currentDay;
                
                renderDashboard(currentDay);
            }
        }

        function renderDashboard(currentDay) {
            const data = incubationData[selectedDay - 1];
            document.getElementById('display-day').innerText = selectedDay;
            document.getElementById('stage-title').innerText = data.title;
            document.getElementById('stage-desc').innerText = data.desc;
            document.getElementById('val-temp').innerText = data.temp;
            document.getElementById('val-hum').innerText = data.hum;
            document.getElementById('val-turn').innerText = data.turn ? 'Қажет' : 'Тоқтату';
            document.getElementById('val-turn').className = data.turn ? 'text-xl font-bold text-white' : 'text-xl font-bold text-red-400';
            
            // Күндер торын шығару
            const grid = document.getElementById('days-grid');
            grid.innerHTML = '';
            incubationData.forEach(d => {
                const btn = document.createElement('button');
                btn.className = `w-12 h-12 rounded-xl flex flex-col items-center justify-center transition-all border-2 
                    ${selectedDay === d.day ? 'border-orange-500 bg-orange-500/10' : 'border-slate-800'} 
                    ${d.day < currentDay ? 'opacity-40' : ''}`;
                btn.innerHTML = `<span class="text-[9px] font-bold">${d.day}</span><span>${d.emoji}</span>`;
                btn.onclick = () => { selectedDay = d.day; updateUI(); };
                grid.appendChild(btn);
            });

            // Эмбрион визуализациясы
            const visual = document.getElementById('embryo-visual');
            const scale = 0.5 + (selectedDay * 0.03);
            visual.style.transform = `scale(${scale})`;
            visual.style.opacity = 0.3 + (selectedDay * 0.03);
            visual.style.backgroundColor = selectedDay > 10 ? '#ef4444' : '#fecaca';
        }

        function startIncubation() {
            const date = document.getElementById('start-date-input').value;
            if (date) {
                localStorage.setItem('incubator_start_date', date);
                startDate = date;
                updateUI();
            }
        }

        function resetIncubation() {
            if (confirm('Процесті тоқтатып, басынан бастағыңыз келе ме?')) {
                localStorage.removeItem('incubator_start_date');
                startDate = null;
                selectedDay = 1;
                updateUI();
            }
        }

        window.onload = init;
    </script>
</body>
</html>
