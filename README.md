# -<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nexus Game Vault — Виртуальная библиотека игр (1990–2026)</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <!-- FontAwesome для иконок -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Plus+Jakarta+Sans:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <!-- Three.js для 3D коробок -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    
    <style>
        @keyframes scaleUp {
            from { opacity: 0; transform: scale(0.95); }
            to { opacity: 1; transform: scale(1); }
        }
        .animate-scaleUp { animation: scaleUp 0.25s cubic-bezier(0.16, 1, 0.3, 1) forwards; }
        .perspective-1000 { perspective: 1000px; }
        
        ::-webkit-scrollbar { width: 8px; height: 8px; }
        ::-webkit-scrollbar-track { background: #090a0f; }
        ::-webkit-scrollbar-thumb { background: #1e293b; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #06b6d4; }

        .view-wall {
            grid-template-columns: repeat(auto-fill, minmax(130px, 1fr)) !important;
            gap: 12px !important;
        }
        .view-wall .game-card-img { height: 180px !important; }

        .view-list {
            display: flex !important;
            flex-direction: column !important;
            gap: 8px !important;
        }
        .view-list .game-item-card {
            display: flex;
            flex-direction: row;
            align-items: center;
            padding: 12px 16px;
            background: #111622;
            border: 1px solid #1e293b;
            border-radius: 12px;
            transition: all 0.2s;
        }
        .view-list .game-item-card:hover {
            border-color: #06b6d4;
            transform: translateX(4px);
        }
    </style>
</head>
<body class="bg-[#090a0f] text-slate-100 font-['Plus_Jakarta_Sans',sans-serif] min-h-screen flex flex-col selection:bg-cyan-500 selection:text-black">

    <!-- Верхняя панель навигации -->
    <header class="sticky top-0 z-40 backdrop-blur-xl bg-[#0b0e14]/80 border-b border-cyan-500/20 px-4 lg:px-8 py-4 flex flex-col md:flex-row items-center justify-between gap-4">
        <div class="flex items-center gap-3">
            <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-cyan-500 to-indigo-600 flex items-center justify-center shadow-lg shadow-cyan-500/30 animate-pulse">
                <i class="fa-solid fa-gamepad text-xl text-black font-bold"></i>
            </div>
            <div>
                <h1 class="text-xl font-extrabold tracking-wider font-['Orbitron'] bg-gradient-to-r from-cyan-400 via-teal-300 to-indigo-400 bg-clip-text text-transparent">NEXUS VAULT</h1>
                <p class="text-xs text-slate-400">Виртуальная библиотека 1990 — 2026</p>
            </div>
        </div>

        <!-- Поиск -->
        <div class="w-full md:w-80 relative">
            <i class="fa-solid fa-magnifying-glass absolute left-3.5 top-1/2 -translate-y-1/2 text-slate-400"></i>
            <input type="text" id="searchInput" placeholder="Поиск игры, жанра, разработчика..." class="w-full bg-[#131823] border border-slate-800 rounded-xl pl-10 pr-4 py-2 text-sm text-slate-200 placeholder-slate-500 focus:outline-none focus:border-cyan-500 transition-all shadow-inner">
        </div>

        <!-- Управление видом и избранным -->
        <div class="flex items-center gap-3 w-full md:w-auto justify-end">
            <button id="favoritesBtn" class="flex items-center gap-2 px-3.5 py-2 rounded-xl bg-[#131823] border border-slate-800 hover:border-cyan-500/50 text-slate-300 text-sm transition-all">
                <i class="fa-solid fa-heart text-rose-500"></i>
                <span class="hidden sm:inline">Избранное</span>
                <span id="favCount" class="bg-rose-500/20 text-rose-400 px-1.5 py-0.5 rounded-md text-xs font-bold">0</span>
            </button>

            <!-- Переключатель видов -->
            <div class="flex items-center bg-[#131823] border border-slate-800 rounded-xl p-1 gap-1">
                <button data-view="grid" class="view-btn p-2 rounded-lg text-cyan-400 bg-cyan-500/25 transition-all active" title="Сетка">
                    <i class="fa-solid fa-grip"></i>
                </button>
                <button data-view="wall" class="view-btn p-2 rounded-lg text-slate-400 hover:text-white transition-all" title="Стена боксов">
                    <i class="fa-solid fa-border-all"></i>
                </button>
                <button data-view="coverflow" class="view-btn p-2 rounded-lg text-slate-400 hover:text-white transition-all" title="Карусель (Альбом)">
                    <i class="fa-solid fa-layer-group"></i>
                </button>
                <button data-view="list" class="view-btn p-2 rounded-lg text-slate-400 hover:text-white transition-all" title="Список">
                    <i class="fa-solid fa-list"></i>
                </button>
            </div>
        </div>
    </header>

    <!-- Фильтры -->
    <nav class="bg-[#0b0e14]/50 border-b border-slate-800/60 px-4 lg:px-8 py-3 flex flex-wrap items-center gap-4 text-sm z-30">
        <div class="flex items-center gap-2">
            <span class="text-slate-400 text-xs font-semibold uppercase tracking-wider">Год:</span>
            <select id="yearFilter" class="bg-[#131823] border border-slate-800 rounded-lg px-3 py-1.5 text-slate-200 focus:outline-none focus:border-cyan-500">
                <option value="all">Все годы (1990 - 2026)</option>
                <option value="90s">1990–1999 (Ретро)</option>
                <option value="00s">2000–2009 (Классика)</option>
                <option value="10s">2010–2019 (Эра HD)</option>
                <option value="20s">2020–2026 (Некстген)</option>
            </select>
        </div>

        <div class="flex items-center gap-2">
            <span class="text-slate-400 text-xs font-semibold uppercase tracking-wider">Платформа:</span>
            <select id="platformFilter" class="bg-[#131823] border border-slate-800 rounded-lg px-3 py-1.5 text-slate-200 focus:outline-none focus:border-cyan-500">
                <option value="all">Все платформы</option>
                <option value="PC">PC (Windows / Linux)</option>
                <option value="PlayStation">PlayStation (PS1-PS5)</option>
                <option value="Xbox">Xbox (Series / One / 360)</option>
                <option value="Nintendo">Nintendo (Switch / Wii / 64)</option>
                <option value="SEGA">SEGA (Mega Drive / Saturn)</option>
            </select>
        </div>

        <div class="flex items-center gap-2">
            <span class="text-slate-400 text-xs font-semibold uppercase tracking-wider">Жанр:</span>
            <select id="genreFilter" class="bg-[#131823] border border-slate-800 rounded-lg px-3 py-1.5 text-slate-200 focus:outline-none focus:border-cyan-500">
                <option value="all">Все жанры</option>
                <option value="Экшен">Экшен</option>
                <option value="RPG">RPG</option>
                <option value="Шутер">Шутер</option>
                <option value="Стратегия">Стратегия</option>
                <option value="Приключения">Приключения</option>
                <option value="Гонки">Гонки</option>
                <option value="Хоррор">Хоррор</option>
            </select>
        </div>

        <div class="ml-auto text-xs text-slate-400">
            Найдено игр: <span id="gameCount" class="text-cyan-400 font-bold">0</span>
        </div>
    </nav>

    <!-- Основная рабочая область -->
    <main class="flex-1 p-4 lg:p-8 relative overflow-hidden" id="mainContainer">
        <!-- Сетка -->
        <div id="gamesContainer" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-6 transition-all duration-300"></div>

        <!-- Режим Coverflow (Альбом) -->
        <div id="coverflowContainer" class="hidden h-[600px] flex flex-col items-center justify-center relative">
            <div id="coverflowTrack" class="w-full flex items-center justify-center relative h-[450px] perspective-1000"></div>
            <div class="flex items-center gap-6 mt-6">
                <button id="cfPrev" class="w-12 h-12 rounded-full bg-[#131823] border border-slate-700 hover:border-cyan-500 text-white flex items-center justify-center shadow-lg transition-all hover:scale-110">
                    <i class="fa-solid fa-chevron-left"></i>
                </button>
                <div id="cfTitle" class="text-center font-['Orbitron'] text-lg text-cyan-300 tracking-wider">Название игры</div>
                <button id="cfNext" class="w-12 h-12 rounded-full bg-[#131823] border border-slate-700 hover:border-cyan-500 text-white flex items-center justify-center shadow-lg transition-all hover:scale-110">
                    <i class="fa-solid fa-chevron-right"></i>
                </button>
            </div>
        </div>

        <!-- Заглушка -->
        <div id="noResults" class="hidden flex flex-col items-center justify-center py-24 text-center">
            <div class="w-20 h-20 rounded-2xl bg-slate-800/50 flex items-center justify-center text-slate-500 text-3xl mb-4 border border-slate-700">
                <i class="fa-solid fa-ghost"></i>
            </div>
            <h3 class="text-lg font-bold text-slate-300 mb-1">Игры не найдены</h3>
            <p class="text-sm text-slate-500">Попробуйте изменить параметры фильтрации или поисковый запрос</p>
        </div>
    </main>

    <!-- Модальное окно 3D бокса -->
    <div id="gameModal" class="fixed inset-0 z-50 bg-black/80 backdrop-blur-md hidden flex items-center justify-center p-4">
        <div class="bg-[#10141d] border border-cyan-500/30 rounded-2xl w-full max-w-4xl max-h-[90vh] overflow-y-auto shadow-2xl shadow-cyan-500/10 flex flex-col md:flex-row relative animate-scaleUp">
            
            <button id="closeModal" class="absolute top-4 right-4 z-10 w-10 h-10 rounded-full bg-slate-900/80 border border-slate-700 hover:border-rose-500 hover:text-rose-400 text-slate-400 flex items-center justify-center transition-all">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>

            <div class="w-full md:w-1/2 bg-gradient-to-b from-[#151b28] to-[#0a0d14] p-6 flex flex-col items-center justify-center relative border-b md:border-b-0 md:border-r border-slate-800">
                <div class="absolute top-4 left-4 text-xs font-['Orbitron'] text-cyan-400 tracking-wider flex items-center gap-2">
                    <span class="w-2 h-2 rounded-full bg-cyan-400 animate-ping"></span> 3D ВИРТУАЛЬНЫЙ БОКС
                </div>
                <div id="canvas3DContainer" class="w-full h-80 cursor-grab active:cursor-grabbing flex items-center justify-center"></div>
                <p class="text-xs text-slate-400 mt-2 text-center"><i class="fa-solid fa-hand-pointer mr-1"></i> Зажмите и вращайте мышкой</p>
            </div>

            <div class="w-full md:w-1/2 p-6 md:p-8 flex flex-col justify-between">
                <div>
                    <div class="flex items-center gap-2 mb-3">
                        <span id="modalPlatform" class="text-xs px-2.5 py-1 rounded-md bg-cyan-500/10 text-cyan-400 border border-cyan-500/20 font-semibold">PC</span>
                        <span id="modalYear" class="text-xs text-slate-400 font-medium">1993</span>
                        <span id="modalGenre" class="text-xs text-indigo-400 font-medium">Шутер</span>
                    </div>

                    <h2 id="modalTitle" class="text-2xl md:text-3xl font-extrabold font-['Orbitron'] text-white mb-4">Название игры</h2>
                    
                    <div class="flex items-center gap-4 mb-6">
                        <div class="flex items-center gap-1.5 bg-amber-500/10 border border-amber-500/20 px-3 py-1 rounded-xl text-amber-400 text-sm font-bold">
                            <i class="fa-solid fa-star"></i>
                            <span id="modalRating">9.5</span>
                        </div>
                        <div class="text-xs text-slate-400">
                            Разработчик: <span id="modalDev" class="text-slate-200 font-semibold">Id Software</span>
                        </div>
                    </div>

                    <p id="modalDesc" class="text-sm text-slate-300 leading-relaxed mb-6">Описание игры...</p>
                </div>

                <div class="flex items-center gap-4 pt-4 border-t border-slate-800">
                    <button id="modalFavBtn" class="flex-1 py-3 px-4 rounded-xl bg-[#182030] hover:bg-slate-800 border border-slate-700 font-medium text-sm transition-all flex items-center justify-center gap-2">
                        <i class="fa-solid fa-heart text-rose-500"></i>
                        <span id="modalFavText">В избранное</span>
                    </button>
                    <button id="launchBtn" class="flex-1 py-3 px-4 rounded-xl bg-gradient-to-r from-cyan-500 to-indigo-600 hover:from-cyan-400 hover:to-indigo-500 text-black font-bold text-sm transition-all shadow-lg shadow-cyan-500/20 flex items-center justify-center gap-2">
                        <i class="fa-solid fa-play"></i> Играть / Запустить
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Футер -->
    <footer class="bg-[#0b0e14] border-t border-slate-800/80 py-6 px-4 text-center text-xs text-slate-500">
        <p>Nexus Vault Game Library &copy; 1990–2026. Все виртуальные обложки и данные каталогизированы для демонстрации.</p>
    </footer>

    <!-- Скрипт логики и базы данных -->
    <script>
        const gamesDatabase = [
            { id: 1, title: "Super Mario World", year: 1990, platform: "Nintendo", genre: "Приключения", rating: 9.8, dev: "Nintendo", desc: "Культовый платформер для SNES, ставший эталоном жанра на десятилетия вперёд.", image: "https://images.unsplash.com/photo-1550745165-9bc0b252726f?q=80&w=600" },
            { id: 2, title: "Sonic the Hedgehog", year: 1991, platform: "SEGA", genre: "Экшен", rating: 9.3, dev: "SEGA", desc: "Самый быстрый синий ёж в истории видеоигр на легендарной консоли SEGA Mega Drive.", image: "https://images.unsplash.com/photo-1612287230202-1ff1d85d1bdf?q=80&w=600" },
            { id: 3, title: "DOOM", year: 1993, platform: "PC", genre: "Шутер", rating: 9.9, dev: "id Software", desc: "Прародитель жанра 3D-шутеров, перевернувший индустрию ПК-гейминга.", image: "https://images.unsplash.com/photo-1542751371-adc38448a05e?q=80&w=600" },
            { id: 4, title: "Warcraft: Orcs & Humans", year: 1994, platform: "PC", genre: "Стратегия", rating: 9.0, dev: "Blizzard Entertainment", desc: "Легендарная стратегия в реальном времени, положившая начало великой вселенной.", image: "https://images.unsplash.com/photo-1614680376593-902f749f7ffc?q=80&w=600" },
            { id: 5, title: "Chrono Trigger", year: 1995, platform: "Nintendo", genre: "RPG", rating: 9.9, dev: "Square", desc: "Шедевральная японская ролевая игра с путешествиями во времени.", image: "https://images.unsplash.com/photo-1538481199705-c710c4e965fc?q=80&w=600" },
            { id: 6, title: "Resident Evil", year: 1996, platform: "PlayStation", genre: "Хоррор", rating: 9.5, dev: "Capcom", desc: "Игра, которая популяризировала жанр Survival Horror на первой PlayStation.", image: "https://images.unsplash.com/photo-1509198397868-475647b2a1e5?q=80&w=600" },
            { id: 7, title: "Final Fantasy VII", year: 1997, platform: "PlayStation", genre: "RPG", rating: 9.8, dev: "Square", desc: "Эпическая RPG, изменившая представление о кинематографичности в видеоиграх.", image: "https://images.unsplash.com/photo-1511512578047-dfb367046420?q=80&w=600" },
            { id: 8, title: "Half-Life", year: 1998, platform: "PC", genre: "Шутер", rating: 9.7, dev: "Valve", desc: "Инновационный научно-фантастический шутер с сюжетной подачей без загрузочных экранов.", image: "https://images.unsplash.com/photo-1552820728-8b83bb6b773f?q=80&w=600" },
            { id: 9, title: "Silent Hill", year: 1999, platform: "PlayStation", genre: "Хоррор", rating: 9.4, dev: "Konami", desc: "Психологический хоррор с неповторимой давящей атмосферой туманного города.", image: "https://images.unsplash.com/photo-1518709268805-4e9042af9f23?q=80&w=600" },
            { id: 10, title: "The Sims", year: 2000, platform: "PC", genre: "Приключения", rating: 9.1, dev: "Maxis", desc: "Революционный симулятор реальной человеческой жизни.", image: "https://images.unsplash.com/photo-1589241062272-c0a000071dfa?q=80&w=600" },
            { id: 11, title: "Grand Theft Auto III", year: 2001, platform: "PlayStation", genre: "Экшен", rating: 9.6, dev: "DMA Design", desc: "Прорывной трехмерный криминальный экшен в открытом мегаполисе.", image: "https://images.unsplash.com/photo-1534447677768-be436bb09401?q=80&w=600" },
            { id: 12, title: "Warcraft III: Reign of Chaos", year: 2002, platform: "PC", genre: "Стратегия", rating: 9.8, dev: "Blizzard", desc: "Великая стратегия с героями и культовой историей, породившая жанр MOBA.", image: "https://images.unsplash.com/photo-1563089145-599997674d42?q=80&w=600" },
            { id: 13, title: "Max Payne 2", year: 2003, platform: "PC", genre: "Шутер", rating: 9.5, dev: "Remedy", desc: "Нуарный боевик с механикой замедления времени (Bullet Time).", image: "https://images.unsplash.com/photo-1579373903781-fd5c0c30c4cd?q=80&w=600" },
            { id: 14, title: "Grand Theft Auto: San Andreas", year: 2004, platform: "PlayStation", genre: "Экшен", rating: 9.9, dev: "Rockstar Games", desc: "Масштабное приключение Си-Джея в штате Сан-Андреас.", image: "https://images.unsplash.com/photo-1542751110-97427bbecf20?q=80&w=600" },
            { id: 15, title: "Resident Evil 4", year: 2005, platform: "Nintendo", genre: "Хоррор", rating: 9.6, dev: "Capcom", desc: "Переосмысление сериала с видом из-за плеча, задавшее стандарт экшенов на годы вперед.", image: "https://images.unsplash.com/photo-1511512578047-dfb367046420?q=80&w=600" },
            { id: 16, title: "The Elder Scrolls IV: Oblivion", year: 2006, platform: "PC", genre: "RPG", rating: 9.4, dev: "Bethesda Game Studios", desc: "Масштабная фэнтезийная ролевая игра в провинции Сиродил.", image: "https://images.unsplash.com/photo-1579783900882-c0d3dad7b119?q=80&w=600" },
            { id: 17, title: "BioShock", year: 2007, platform: "Xbox", genre: "Шутер", rating: 9.7, dev: "2K Games", desc: "Утопический подводный город Восторг и глубокая философская история.", image: "https://images.unsplash.com/photo-1550745165-9bc0b252726f?q=80&w=600" },
            { id: 18, title: "Grand Theft Auto IV", year: 2008, platform: "PlayStation", genre: "Экшен", rating: 9.5, dev: "Rockstar Games", desc: "Мрачный и реалистичный Либерти-Сити глазами Нико Берика.", image: "https://images.unsplash.com/photo-1518709268805-4e9042af9f23?q=80&w=600" },
            { id: 19, title: "Uncharted 2: Among Thieves", year: 2009, platform: "PlayStation", genre: "Приключения", rating: 9.6, dev: "Naughty Dog", desc: "Кинематографичный приключенческий экшен с поисками Шамбалы.", image: "https://images.unsplash.com/photo-1538481199705-c710c4e965fc?q=80&w=600" },
            { id: 20, title: "Red Dead Redemption", year: 2010, platform: "Xbox", genre: "Приключения", rating: 9.7, dev: "Rockstar Games", desc: "Драматичная история заката эпохи Дикого Запада.", image: "https://images.unsplash.com/photo-1509198397868-475647b2a1e5?q=80&w=600" },
            { id: 21, title: "The Elder Scrolls V: Skyrim", year: 2011, platform: "PC", genre: "RPG", rating: 9.8, dev: "Bethesda", desc: "Легендарная RPG про драконорожденного в заснеженном Скайриме.", image: "https://images.unsplash.com/photo-1614680376593-902f749f7ffc?q=80&w=600" },
            { id: 22, title: "Far Cry 3", year: 2012, platform: "PC", genre: "Шутер", rating: 9.0, dev: "Ubisoft", desc: "Тропический остров, безумный Ваас Монтенегро и выживание в джунглях.", image: "https://images.unsplash.com/photo-1552820728-8b83bb6b773f?q=80&w=600" },
            { id: 23, title: "Grand Theft Auto V", year: 2013, platform: "PlayStation", genre: "Экшен", rating: 9.8, dev: "Rockstar Games", desc: "Три главных героя и огромный солнечный Лос-Сантос.", image: "https://images.unsplash.com/photo-1534447677768-be436bb09401?q=80&w=600" },
            { id: 24, title: "Dragon Age: Inquisition", year: 2014, platform: "PC", genre: "RPG", rating: 8.9, dev: "BioWare", desc: "Масштабное фэнтезийное приключение во главе Инквизиции.", image: "https://images.unsplash.com/photo-1511512578047-dfb367046420?q=80&w=600" },
            { id: 25, title: "The Witcher 3: Wild Hunt", year: 2015, platform: "PC", genre: "RPG", rating: 10.0, dev: "CD Projekt Red", desc: "Признанный шедевр о поисках Цири и монстрах Континента.", image: "https://images.unsplash.com/photo-1579783900882-c0d3dad7b119?q=80&w=600" },
            { id: 26, title: "Battlefield 1", year: 2016, platform: "PC", genre: "Шутер", rating: 9.1, dev: "EA DICE", desc: "Масштабные и зрелищные сражения времен Первой мировой войны.", image: "https://images.unsplash.com/photo-1542751371-adc38448a05e?q=80&w=600" },
            { id: 27, title: "The Legend of Zelda: BOTW", year: 2017, platform: "Nintendo", genre: "Приключения", rating: 9.9, dev: "Nintendo", desc: "Беспрецедентная свобода исследования открытого мира Хайрула.", image: "https://images.unsplash.com/photo-1550745165-9bc0b252726f?q=80&w=600" },
            { id: 28, title: "God of War", year: 2018, platform: "PlayStation", genre: "Экшен", rating: 9.8, dev: "Santa Monica Studio", desc: "Новая глава жизни спартанского бога войны Кратоса в скандинавских мифах.", image: "https://images.unsplash.com/photo-1538481199705-c710c4e965fc?q=80&w=600" },
            { id: 29, title: "Death Stranding", year: 2019, platform: "PlayStation", genre: "Приключения", rating: 9.0, dev: "Kojima Productions", desc: "Уникальный симулятор курьера в постапокалиптическом мире от Хидео Кодзимы.", image: "https://images.unsplash.com/photo-1518709268805-4e9042af9f23?q=80&w=600" },
            { id: 30, title: "Cyberpunk 2077", year: 2020, platform: "PC", genre: "RPG", rating: 9.0, dev: "CD Projekt Red", desc: "Неоновый Найт-Сити, киберимпланты и рок-музыкант в голове.", image: "https://images.unsplash.com/photo-1542751110-97427bbecf20?q=80&w=600" },
            { id: 31, title: "Forza Horizon 5", year: 2021, platform: "Xbox", genre: "Гонки", rating: 9.4, dev: "Playground Games", desc: "Фестиваль скорости в живописных и разнообразных ландшафтах Мексики.", image: "https://images.unsplash.com/photo-1511512578047-dfb367046420?q=80&w=600" },
            { id: 32, title: "Elden Ring", year: 2022, platform: "PC", genre: "RPG", rating: 9.7, dev: "FromSoftware", desc: "Величественное мрачное фэнтези в открытом мире от Хидэтака Миядзаки.", image: "https://images.unsplash.com/photo-1579783900882-c0d3dad7b119?q=80&w=600" },
            { id: 33, title: "Baldur's Gate 3", year: 2023, platform: "PC", genre: "RPG", rating: 9.9, dev: "Larian Studios", desc: "Величайшая партийная RPG с невероятной вариативностью выбора и сюжетом.", image: "https://images.unsplash.com/photo-1614680376593-902f749f7ffc?q=80&w=600" },
            { id: 34, title: "Black Myth: Wukong", year: 2024, platform: "PC", genre: "Экшен", rating: 9.3, dev: "Game Science", desc: "Эпический экшен по мотивам романа «Путешествие на Запад» на движке Unreal Engine 5.", image: "https://images.unsplash.com/photo-1552820728-8b83bb6b773f?q=80&w=600" },
            { id: 35, title: "Grand Theft Auto VI", year: 2025, platform: "PlayStation", genre: "Экшен", rating: 10.0, dev: "Rockstar Games", desc: "Долгожданное возвращение в Вайс-Сити с Люсией и Джейсоном.", image: "https://images.unsplash.com/photo-1534447677768-be436bb09401?q=80&w=600" },
            { id: 36, title: "Nexus Chronicles", year: 2026, platform: "PC", genre: "RPG", rating: 9.6, dev: "Nexus Studios", desc: "Инновационная игра нового поколения с полной интеграцией виртуальной реальности.", image: "https://images.unsplash.com/photo-1550745165-9bc0b252726f?q=80&w=600" }
        ];

        let currentView = 'grid';
        let favorites = JSON.parse(localStorage.getItem('nexus_favs')) || [];
        let activeGame = null;
        let cfIndex = 0;
        let threeRenderer = null, threeScene = null, threeAnimId = null;

        document.addEventListener('DOMContentLoaded', () => {
            initUI();
            filterAndRenderGames();
        });

        function initUI() {
            document.getElementById('searchInput').addEventListener('input', filterAndRenderGames);
            document.getElementById('yearFilter').addEventListener('change', filterAndRenderGames);
            document.getElementById('platformFilter').addEventListener('change', filterAndRenderGames);
            document.getElementById('genreFilter').addEventListener('change', filterAndRenderGames);

            document.querySelectorAll('.view-btn').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    document.querySelectorAll('.view-btn').forEach(b => b.classList.remove('active', 'bg-cyan-500/25', 'text-cyan-400'));
                    const target = e.currentTarget;
                    target.classList.add('active', 'bg-cyan-500/25', 'text-cyan-400');
                    currentView = target.dataset.view;
                    filterAndRenderGames();
                });
            });

            let showOnlyFavs = false;
            document.getElementById('favoritesBtn').addEventListener('click', () => {
                showOnlyFavs = !showOnlyFavs;
                document.getElementById('favoritesBtn').classList.toggle('border-cyan-500', showOnlyFavs);
                document.getElementById('favoritesBtn').classList.toggle('bg-cyan-500/10', showOnlyFavs);
                filterAndRenderGames(showOnlyFavs);
            });

            document.getElementById('closeModal').addEventListener('click', closeGameModal);
            document.getElementById('gameModal').addEventListener('click', (e) => {
                if (e.target === document.getElementById('gameModal')) closeGameModal();
            });

            document.getElementById('cfPrev').addEventListener('click', () => {
                const filtered = getFilteredGames();
                if (!filtered.length) return;
                cfIndex = (cfIndex - 1 + filtered.length) % filtered.length;
                renderCoverflow(filtered);
            });
            document.getElementById('cfNext').addEventListener('click', () => {
                const filtered = getFilteredGames();
                if (!filtered.length) return;
                cfIndex = (cfIndex + 1) % filtered.length;
                renderCoverflow(filtered);
            });

            updateFavCounter();
        }

        function getFilteredGames() {
            const query = document.getElementById('searchInput').value.toLowerCase();
            const yearVal = document.getElementById('yearFilter').value;
            const platformVal = document.getElementById('platformFilter').value;
            const genreVal = document.getElementById('genreFilter').value;

            return gamesDatabase.filter(game => {
                const matchQuery = game.title.toLowerCase().includes(query) || 
                                   game.dev.toLowerCase().includes(query) || 
                                   game.genre.toLowerCase().includes(query);
                let matchYear = true;
                if (yearVal === '90s') matchYear = game.year >= 1990 && game.year <= 1999;
                else if (yearVal === '00s') matchYear = game.year >= 2000 && game.year <= 2009;
                else if (yearVal === '10s') matchYear = game.year >= 2010 && game.year <= 2019;
                else if (yearVal === '20s') matchYear = game.year >= 2020 && game.year <= 2026;

                const matchPlatform = platformVal === 'all' || game.platform === platformVal;
                const matchGenre = genreVal === 'all' || game.genre === genreVal;

                return matchQuery && matchYear && matchPlatform && matchGenre;
            });
        }

        function filterAndRenderGames(onlyFavs = false) {
            let filtered = getFilteredGames();
            if (onlyFavs) filtered = filtered.filter(g => favorites.includes(g.id));

            document.getElementById('gameCount').textContent = filtered.length;
            
            const container = document.getElementById('gamesContainer');
            const coverflowContainer = document.getElementById('coverflowContainer');
            const noResults = document.getElementById('noResults');

            if (filtered.length === 0) {
                container.classList.add('hidden');
                coverflowContainer.classList.add('hidden');
                noResults.classList.remove('hidden');
                return;
            }

            noResults.classList.add('hidden');

            if (currentView === 'coverflow') {
                container.classList.add('hidden');
                coverflowContainer.classList.remove('hidden');
                cfIndex = 0;
                renderCoverflow(filtered);
                return;
            }

            coverflowContainer.classList.add('hidden');
            container.classList.remove('hidden');

            if (currentView === 'grid') {
                container.className = "grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-6";
            } else if (currentView === 'wall') {
                container.className = "grid view-wall";
            } else if (currentView === 'list') {
                container.className = "view-list";
            }

            container.innerHTML = filtered.map(game => {
                const isFav = favorites.includes(game.id);
                if (currentView === 'list') {
                    return `
                        <div class="game-item-card cursor-pointer" onclick="openGameModal(${game.id})">
                            <img src="${game.image}" alt="${game.title}" class="w-16 h-16 object-cover rounded-lg mr-4">
                            <div class="flex-1">
                                <div class="flex items-center gap-2 text-xs text-cyan-400 font-semibold">
                                    <span>${game.platform}</span> &bull; <span>${game.year}</span>
                                </div>
                                <h4 class="text-white font-bold font-['Orbitron'] text-base">${game.title}</h4>
                                <p class="text-xs text-slate-400">${game.dev} | ${game.genre}</p>
                            </div>
                            <div class="text-amber-400 font-bold text-sm flex items-center gap-1 mr-4">
                                <i class="fa-solid fa-star"></i> ${game.rating}
                            </div>
                        </div>
                    `;
                }

                return `
                    <div class="bg-[#111622] border border-slate-800/80 hover:border-cyan-500/50 rounded-2xl overflow-hidden shadow-xl hover:shadow-cyan-500/10 transition-all duration-300 flex flex-col group cursor-pointer" onclick="openGameModal(${game.id})">
                        <div class="relative overflow-hidden ${currentView === 'wall' ? 'h-[180px]' : 'h-52'}">
                            <img src="${game.image}" alt="${game.title}" class="w-full h-full object-cover group-hover:scale-110 transition-transform duration-500">
                            <div class="absolute inset-0 bg-gradient-to-t from-[#111622] via-transparent to-transparent"></div>
                            <span class="absolute top-3 left-3 bg-black/60 backdrop-blur-md text-cyan-400 border border-cyan-500/30 text-[10px] font-bold px-2.5 py-1 rounded-lg">
                                ${game.platform}
                            </span>
                            <button onclick="event.stopPropagation(); toggleFavorite(${game.id})" class="absolute top-3 right-3 w-8 h-8 rounded-full bg-black/60 backdrop-blur-md flex items-center justify-center text-slate-300 hover:text-rose-500 transition-colors">
                                <i class="fa-${isFav ? 'solid' : 'regular'} fa-heart ${isFav ? 'text-rose-500' : ''}"></i>
                            </button>
                        </div>
                        <div class="p-4 flex-1 flex flex-col justify-between">
                            <div>
                                <div class="flex items-center justify-between text-xs text-slate-400 mb-1">
                                    <span>${game.year}</span>
                                    <span class="text-indigo-400">${game.genre}</span>
                                </div>
                                <h3 class="font-['Orbitron'] font-bold text-white text-base group-hover:text-cyan-400 transition-colors line-clamp-1">${game.title}</h3>
                            </div>
                            <div class="flex items-center justify-between mt-4 pt-3 border-t border-slate-800/60 text-xs">
                                <span class="text-slate-400 truncate max-w-[120px]">${game.dev}</span>
                                <span class="text-amber-400 font-bold flex items-center gap-1">
                                    <i class="fa-solid fa-star"></i> ${game.rating}
                                </span>
                            </div>
                        </div>
                    </div>
                `;
            }).join('');
        }

        function renderCoverflow(filtered) {
            const track = document.getElementById('coverflowTrack');
            const titleEl = document.getElementById('cfTitle');
            if (!filtered.length) return;

            if (cfIndex >= filtered.length) cfIndex = 0;
            activeGame = filtered[cfIndex];
            titleEl.textContent = `${activeGame.title} (${activeGame.year})`;

            track.innerHTML = filtered.map((game, idx) => {
                let offset = idx - cfIndex;
                let translateX = offset * 160;
                let scale = offset === 0 ? 1.1 : Math.max(0.7, 1 - Math.abs(offset) * 0.15);
                let rotateY = offset === 0 ? 0 : (offset < 0 ? 45 : -45);
                let zIndex = 100 - Math.abs(offset);
                let opacity = Math.abs(offset) > 3 ? 0 : 1;

                return `
                    <div onclick="openGameModal(${game.id})" class="absolute transition-all duration-500 cursor-pointer w-56 h-80 rounded-2xl overflow-hidden border-2 ${offset === 0 ? 'border-cyan-400 shadow-2xl shadow-cyan-500/30' : 'border-slate-800 opacity-70'}"
                         style="transform: translateX(${translateX}px) scale(${scale}) rotateY(${rotateY}deg); z-index: ${zIndex}; opacity: ${opacity};">
                        <img src="${game.image}" class="w-full h-full object-cover">
                        <div class="absolute inset-0 bg-gradient-to-t from-black/80 via-transparent to-transparent flex items-end p-4">
                            <span class="text-white font-bold font-['Orbitron'] text-sm">${game.title}</span>
                        </div>
                    </div>
                `;
            }).join('');
        }

        function toggleFavorite(id) {
            if (favorites.includes(id)) favorites = favorites.filter(favId => favId !== id);
            else favorites.push(id);
            localStorage.setItem('nexus_favs', JSON.stringify(favorites));
            updateFavCounter();
            filterAndRenderGames();
            if (activeGame && activeGame.id === id) updateModalFavButton();
        }

        function updateFavCounter() {
            document.getElementById('favCount').textContent = favorites.length;
        }

        function openGameModal(id) {
            activeGame = gamesDatabase.find(g => g.id === id);
            if (!activeGame) return;

            document.getElementById('modalPlatform').textContent = activeGame.platform;
            document.getElementById('modalYear').textContent = activeGame.year;
            document.getElementById('modalGenre').textContent = activeGame.genre;
            document.getElementById('modalTitle').textContent = activeGame.title;
            document.getElementById('modalRating').textContent = activeGame.rating;
            document.getElementById('modalDev').textContent = activeGame.dev;
            document.getElementById('modalDesc').textContent = activeGame.desc;

            updateModalFavButton();
            document.getElementById('gameModal').classList.remove('hidden');
            initThreeDBox(activeGame);
        }

        function closeGameModal() {
            document.getElementById('gameModal').classList.add('hidden');
            stopThreeD();
        }

        function updateModalFavButton() {
            const isFav = favorites.includes(activeGame.id);
            document.getElementById('modalFavText').textContent = isFav ? 'В избранном' : 'В избранное';
            document.querySelector('#modalFavBtn i').className = `fa-${isFav ? 'solid' : 'regular'} fa-heart text-rose-500`;
        }

        document.getElementById('modalFavBtn').addEventListener('click', () => {
            if (activeGame) toggleFavorite(activeGame.id);
        });

        function initThreeDBox(game) {
            const container = document.getElementById('canvas3DContainer');
            container.innerHTML = '';
            const width = container.clientWidth || 320;
            const height = 300;

            const scene = new THREE.Scene();
            threeScene = scene;

            const camera = new THREE.PerspectiveCamera(45, width / height, 0.1, 1000);
            camera.position.z = 5.5;

            const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
            renderer.setSize(width, height);
            renderer.setPixelRatio(window.devicePixelRatio);
            container.appendChild(renderer.domElement);
            threeRenderer = renderer;

            scene.add(new THREE.AmbientLight(0xffffff, 0.8));
            const pointLight = new THREE.PointLight(0x06b6d4, 2, 50);
            pointLight.position.set(5, 5, 5);
            scene.add(pointLight);

            const geometry = new THREE.BoxGeometry(2.2, 3.0, 0.5);
            const loader = new THREE.TextureLoader();

            const defaultMaterials = [
                new THREE.MeshStandardMaterial({ color: 0x1b2436 }),
                new THREE.MeshStandardMaterial({ color: 0x1b2436 }),
                new THREE.MeshStandardMaterial({ color: 0x1b2436 }),
                new THREE.MeshStandardMaterial({ color: 0x1b2436 }),
                new THREE.MeshStandardMaterial({ color: 0x06b6d4, roughness: 0.2 }),
                new THREE.MeshStandardMaterial({ color: 0x0b0e14 })
            ];

            const mesh = new THREE.Mesh(geometry, defaultMaterials);
            scene.add(mesh);

            loader.load(game.image, (texture) => {
                mesh.material = [
                    new THREE.MeshStandardMaterial({ color: 0x111622 }),
                    new THREE.MeshStandardMaterial({ color: 0x111622 }),
                    new THREE.MeshStandardMaterial({ color: 0x111622 }),
                    new THREE.MeshStandardMaterial({ color: 0x111622 }),
                    new THREE.MeshStandardMaterial({ map: texture, roughness: 0.3 }),
                    new THREE.MeshStandardMaterial({ color: 0x0b0e14 })
                ];
            });

            let isDragging = false;
            let prevMousePos = { x: 0, y: 0 };

            renderer.domElement.addEventListener('mousedown', (e) => {
                isDragging = true;
                prevMousePos = { x: e.clientX, y: e.clientY };
            });

            window.addEventListener('mousemove', (e) => {
                if (!isDragging) return;
                mesh.rotation.y += (e.clientX - prevMousePos.x) * 0.01;
                mesh.rotation.x += (e.clientY - prevMousePos.y) * 0.01;
                prevMousePos = { x: e.clientX, y: e.clientY };
            });

            window.addEventListener('mouseup', () => { isDragging = false; });

            function animate() {
                threeAnimId = requestAnimationFrame(animate);
                if (!isDragging) mesh.rotation.y += 0.005;
                renderer.render(scene, camera);
            }
            animate();
        }

        function stopThreeD() {
            if (threeAnimId) cancelAnimationFrame(threeAnimId);
            if (threeRenderer) {
                threeRenderer.dispose();
                threeRenderer = null;
            }
        }
    </script>
</body>
</html>
