<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Мой Кейс Бот</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { overflow: hidden; font-family: Arial; background: #0f0f1a; user-select: none; -webkit-tap-highlight-color: transparent; color: white; }

        #coinCounter {
            position: absolute; top: 15px; left: 15px;
            background: rgba(0,0,0,0.7); backdrop-filter: blur(10px);
            padding: 10px 20px; border-radius: 30px;
            border: 2px solid rgba(255,215,0,0.4);
            color: #ffd700; font-size: 22px; font-weight: bold;
            display: flex; align-items: center; gap: 8px; z-index: 15;
            pointer-events: none;
        }
        #coinCounter span { font-size: 24px; }

        #status {
            position: absolute; top: 15px; left: 50%; transform: translateX(-50%);
            color: white; background: rgba(0,0,0,0.6); padding: 8px 20px;
            border-radius: 20px; font-size: 13px; backdrop-filter: blur(5px);
            border: 1px solid rgba(255,255,255,0.1); pointer-events: none; z-index: 10;
            white-space: nowrap;
        }

        #cases {
            display: flex; justify-content: center; align-items: center;
            height: 100vh; gap: 20px; flex-wrap: wrap; padding: 20px;
            overflow-y: auto;
        }

        .case {
            width: 160px; height: 220px;
            background: linear-gradient(145deg, #1e1e3a, #2a2a4a);
            border-radius: 20px;
            border: 2px solid rgba(255,255,255,0.1);
            display: flex; flex-direction: column; align-items: center;
            justify-content: center; gap: 12px; cursor: pointer;
            transition: all 0.3s; touch-action: manipulation;
            box-shadow: 0 8px 30px rgba(0,0,0,0.4);
            position: relative;
            overflow: hidden;
        }
        .case:active { transform: scale(0.95); }
        .case .case-emoji { font-size: 60px; }
        .case .case-name { font-size: 16px; font-weight: bold; }
        .case .case-price { font-size: 18px; color: #ffd700; font-weight: bold; }
        .case .case-rarity { font-size: 12px; opacity: 0.6; }

        #rouletteContainer {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.95); backdrop-filter: blur(20px);
            z-index: 100; display: none; justify-content: center; align-items: center;
            flex-direction: column;
        }
        #rouletteContainer.active { display: flex; }

        #rouletteWindow {
            width: 90%; max-width: 500px; height: 120px;
            background: #1a1a2e; border-radius: 15px;
            border: 2px solid rgba(255,215,0,0.3);
            overflow: hidden; position: relative;
        }

        #rouletteTrack {
            display: flex; position: absolute; top: 0; left: 0;
            height: 100%; align-items: center;
            transition: transform 6s cubic-bezier(0.1, 0.8, 0.1, 1);
        }

        .roulette-item {
            flex: 0 0 100px; height: 80px;
            display: flex; flex-direction: column; align-items: center;
            justify-content: center; gap: 4px;
            background: rgba(255,255,255,0.05); margin: 0 5px;
            border-radius: 10px; border: 2px solid rgba(255,255,255,0.1);
        }
        .roulette-item .item-emoji { font-size: 30px; }
        .roulette-item .item-name { font-size: 10px; text-align: center; }
        .roulette-item.rare { border-color: #3498db; background: rgba(52,152,219,0.1); }
        .roulette-item.epic { border-color: #9b59b6; background: rgba(155,89,182,0.1); }
        .roulette-item.legendary { border-color: #ffd700; background: rgba(255,215,0,0.1); }

        #roulettePointer {
            position: absolute; top: 0; left: 50%; transform: translateX(-50%);
            width: 4px; height: 100%; background: #ffd700;
            z-index: 5; box-shadow: 0 0 20px rgba(255,215,0,0.5);
        }

        #rouletteResult {
            margin-top: 30px; text-align: center; display: none;
        }
        #rouletteResult.show { display: block; }
        #rouletteResult .result-emoji { font-size: 80px; }
        #rouletteResult .result-name { font-size: 24px; font-weight: bold; margin-top: 10px; }
        #rouletteResult .result-rarity { font-size: 14px; opacity: 0.6; margin-top: 5px; }
        #rouletteResult .result-btn {
            margin-top: 20px; padding: 12px 40px; border-radius: 30px;
            border: none; background: #3498db; color: white;
            font-size: 18px; font-weight: bold; cursor: pointer;
            touch-action: manipulation;
        }
        #rouletteResult .result-btn:active { transform: scale(0.95); }

        #inventoryToggle {
            position: absolute; bottom: 30px; right: 20px;
            background: rgba(155,89,182,0.8); backdrop-filter: blur(10px);
            border: 1px solid rgba(255,255,255,0.2); color: white;
            width: 60px; height: 60px; border-radius: 50%; font-size: 28px;
            cursor: pointer; z-index: 20; touch-action: manipulation;
            box-shadow: 0 4px 20px rgba(0,0,0,0.4);
        }
        #inventoryToggle:active { transform: scale(0.9); }

        #inventoryMenu {
            position: absolute; bottom: -100%; left: 0; width: 100%;
            background: rgba(20,20,40,0.98); backdrop-filter: blur(20px);
            border-top: 1px solid rgba(155,89,182,0.3);
            padding: 20px 16px 30px; z-index: 30;
            transition: bottom 0.4s cubic-bezier(0.4,0,0.2,1);
            max-height: 65vh; overflow-y: auto; border-radius: 20px 20px 0 0;
        }
        #inventoryMenu.open { bottom: 0; }
        #inventoryMenu::-webkit-scrollbar { width: 3px; }
        #inventoryMenu::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.2); border-radius: 10px; }

        .menu-header {
            display: flex; justify-content: space-between; align-items: center;
            color: white; margin-bottom: 15px; padding-bottom: 10px;
            border-bottom: 1px solid rgba(255,255,255,0.05);
        }
        .menu-header h3 { font-size: 18px; font-weight: 600; }
        .menu-close {
            background: rgba(255,255,255,0.1); border: none; color: white;
            font-size: 24px; width: 40px; height: 40px; border-radius: 50%;
            cursor: pointer; touch-action: manipulation;
        }
        .menu-close:active { transform: scale(0.9); }

        .inventory-item {
            display: flex; justify-content: space-between; align-items: center;
            padding: 10px 14px; margin-bottom: 6px;
            background: rgba(255,255,255,0.05); border-radius: 12px;
            border-left: 3px solid #ffd700;
        }
        .inventory-item .info { display: flex; align-items: center; gap: 10px; }
        .inventory-item .emoji { font-size: 28px; }
        .inventory-item .name { font-size: 14px; }
        .inventory-item .rarity { font-size: 11px; opacity: 0.6; }
        .inventory-item .sell-btn {
            background: rgba(46,204,113,0.2); border: 1px solid #2ecc71;
            color: white; padding: 6px 14px; border-radius: 20px;
            font-size: 12px; cursor: pointer; touch-action: manipulation;
        }
        .inventory-item .sell-btn:active { transform: scale(0.9); }
        .inventory-item.rare { border-left-color: #3498db; }
        .inventory-item.epic { border-left-color: #9b59b6; }
        .inventory-item.legendary { border-left-color: #ffd700; }

        #emptyInventory {
            text-align: center; color: rgba(255,255,255,0.4); padding: 30px;
            font-size: 14px;
        }

        #info {
            position: absolute; bottom: 10px; left: 0; width: 100%;
            text-align: center; color: rgba(255,255,255,0.2); font-size: 11px;
            pointer-events: none; z-index: 5;
        }

        @media (max-width: 500px) {
            .case { width: 130px; height: 180px; }
            .case .case-emoji { font-size: 45px; }
            .case .case-name { font-size: 14px; }
            .case .case-price { font-size: 16px; }
            #inventoryToggle { bottom: 20px; right: 15px; width: 54px; height: 54px; font-size: 24px; }
            #coinCounter { font-size: 16px; padding: 6px 14px; top: 10px; left: 10px; }
            #coinCounter span { font-size: 18px; }
            #status { font-size: 11px; padding: 6px 14px; }
            .inventory-item .name { font-size: 12px; }
            .inventory-item .sell-btn { font-size: 11px; padding: 4px 10px; }
        }
    </style>
</head>
<body>

    <div id="coinCounter">
        <span>🪙</span> <span id="coinCount">0</span>
    </div>

    <div id="status">🎰 Выбери кейс</div>

    <div id="cases"></div>

    <div id="rouletteContainer">
        <div id="rouletteWindow">
            <div id="roulettePointer"></div>
            <div id="rouletteTrack"></div>
        </div>
        <div id="rouletteResult">
            <div class="result-emoji">🎁</div>
            <div class="result-name">Приз</div>
            <div class="result-rarity">Обычный</div>
            <button class="result-btn" id="resultBtn">Забрать</button>
        </div>
    </div>

    <button id="inventoryToggle">🎒</button>
    <div id="inventoryMenu">
        <div class="menu-header">
            <h3>🎒 Инвентарь</h3>
            <button class="menu-close" id="closeInventory">✕</button>
        </div>
        <div id="inventoryList"></div>
    </div>

    <div id="info">Крути кейсы и собирай скины!</div>

    <script>
        const rarities = {
            common: { name: 'Обычный', color: '#95a5a6' },
            rare: { name: 'Редкий', color: '#3498db' },
            epic: { name: 'Эпический', color: '#9b59b6' },
            legendary: { name: 'Легендарный', color: '#ffd700' },
        };

        const allItems = [
            { id: 'c1', name: 'Ржавый ключ', emoji: '🔑', rarity: 'common', price: 5 },
            { id: 'c2', name: 'Старая монета', emoji: '🪙', rarity: 'common', price: 5 },
            { id: 'c3', name: 'Кусок ткани', emoji: '🧵', rarity: 'common', price: 5 },
            { id: 'c4', name: 'Деревяшка', emoji: '🪵', rarity: 'common', price: 5 },
            { id: 'c5', name: 'Камень', emoji: '🪨', rarity: 'common', price: 5 },
            { id: 'r1', name: 'Серебряный ключ', emoji: '🔑', rarity: 'rare', price: 15 },
            { id: 'r2', name: 'Синий кристалл', emoji: '💎', rarity: 'rare', price: 15 },
            { id: 'r3', name: 'Кожаные перчатки', emoji: '🧤', rarity: 'rare', price: 15 },
            { id: 'r4', name: 'Зелье силы', emoji: '🧪', rarity: 'rare', price: 15 },
            { id: 'e1', name: 'Золотой ключ', emoji: '🔑', rarity: 'epic', price: 40 },
            { id: 'e2', name: 'Фиолетовый кристалл', emoji: '💜', rarity: 'epic', price: 40 },
            { id: 'e3', name: 'Плащ героя', emoji: '🧥', rarity: 'epic', price: 40 },
            { id: 'e4', name: 'Корона', emoji: '👑', rarity: 'epic', price: 40 },
            { id: 'l1', name: 'Алмазный ключ', emoji: '🔑', rarity: 'legendary', price: 100 },
            { id: 'l2', name: 'Красный кристалл', emoji: '❤️', rarity: 'legendary', price: 100 },
            { id: 'l3', name: 'Крылья ангела', emoji: '🪽', rarity: 'legendary', price: 100 },
            { id: 'l4', name: 'Меч легенд', emoji: '⚔️', rarity: 'legendary', price: 100 },
        ];

        const cases = [
            { id: 'case1', name: 'Бронзовый кейс', emoji: '📦', price: 10, rarity: 'common', chances: { common: 70, rare: 25, epic: 4, legendary: 1 } },
            { id: 'case2', name: 'Серебряный кейс', emoji: '🎁', price: 30, rarity: 'rare', chances: { common: 40, rare: 40, epic: 15, legendary: 5 } },
            { id: 'case3', name: 'Золотой кейс', emoji: '🏆', price: 100, rarity: 'epic', chances: { common: 10, rare: 30, epic: 45, legendary: 15 } },
            { id: 'case4', name: 'Алмазный кейс', emoji: '💎', price: 250, rarity: 'legendary', chances: { common: 0, rare: 15, epic: 50, legendary: 35 } },
        ];

        let coins = parseInt(localStorage.getItem('caseCoins')) || 100;
        let inventory = JSON.parse(localStorage.getItem('caseInventory')) || [];

        const coinDisplay = document.getElementById('coinCount');
        function updateCoins() {
            coinDisplay.textContent = coins;
            localStorage.setItem('caseCoins', coins);
        }
        updateCoins();

        function saveInventory() {
            localStorage.setItem('caseInventory', JSON.stringify(inventory));
        }

        function renderCases() {
            const container = document.getElementById('cases');
            container.innerHTML = cases.map(c => `
                <div class="case" data-id="${c.id}">
                    <div class="case-emoji">${c.emoji}</div>
                    <div class="case-name">${c.name}</div>
                    <div class="case-price">🪙 ${c.price}</div>
                    <div class="case-rarity">${rarities[c.rarity].name}</div>
                </div>
            `).join('');

            container.querySelectorAll('.case').forEach(el => {
                el.addEventListener('click', () => openCase(el.dataset.id));
            });
        }

        function openCase(id) {
            const c = cases.find(x => x.id === id);
            if (!c) return;
            if (coins < c.price) {
                document.getElementById('status').textContent = '❌ Недостаточно монет!';
                setTimeout(() => document.getElementById('status').textContent = '🎰 Выбери кейс', 1500);
                return;
            }
            coins -= c.price;
            updateCoins();

            const rarity = getRandomRarity(c.chances);
            const items = allItems.filter(i => i.rarity === rarity);
            const item = items[Math.floor(Math.random() * items.length)];

            showRoulette(c, item);
        }

        function getRandomRarity(chances) {
            const roll = Math.random() * 100;
            let cumulative = 0;
            const order = ['legendary', 'epic', 'rare', 'common'];
            for (const r of order) {
                cumulative += chances[r] || 0;
                if (roll < cumulative) return r;
            }
            return 'common';
        }

        function showRoulette(c, winningItem) {
            const container = document.getElementById('rouletteContainer');
            const track = document.getElementById('rouletteTrack');
            const resultDiv = document.getElementById('rouletteResult');

            container.classList.add('active');
            resultDiv.classList.remove('show');

            const items = [];
            const all = [...allItems];
            for (let i = 0; i < 30; i++) {
                items.push(all[Math.floor(Math.random() * all.length)]);
            }
            items[20] = winningItem;

            track.innerHTML = items.map(item => {
                return `<div class="roulette-item ${item.rarity}">
                    <div class="item-emoji">${item.emoji}</div>
                    <div class="item-name">${item.name}</div>
                </div>`;
            }).join('');

            track.style.transition = 'none';
            track.style.transform = 'translateX(0)';
            void track.offsetWidth;

            const itemWidth = 110;
            const targetPos = -(20 * itemWidth) + (document.getElementById('rouletteWindow').offsetWidth / 2) - (itemWidth / 2);

            track.style.transition = 'transform 6s cubic-bezier(0.1, 0.8, 0.1, 1)';
            track.style.transform = `translateX(${targetPos}px)`;

            setTimeout(() => {
                resultDiv.classList.add('show');
                const r = rarities[winningItem.rarity];
                resultDiv.querySelector('.result-emoji').textContent = winningItem.emoji;
                resultDiv.querySelector('.result-name').textContent = winningItem.name;
                resultDiv.querySelector('.result-rarity').textContent = r.name;
                resultDiv.querySelector('.result-rarity').style.color = r.color;

                inventory.push(winningItem);
                saveInventory();
                renderInventory();

                document.getElementById('status').textContent = `🎁 Выпало: ${winningItem.name}`;
                setTimeout(() => document.getElementById('status').textContent = '🎰 Выбери кейс', 2000);
            }, 6200);
        }

        document.getElementById('resultBtn').addEventListener('click', () => {
            document.getElementById('rouletteContainer').classList.remove('active');
            renderCases();
        });

        function renderInventory() {
            const list = document.getElementById('inventoryList');
            if (inventory.length === 0) {
                list.innerHTML = '<div id="emptyInventory">Инвентарь пуст 😢</div>';
                return;
            }
            list.innerHTML = inventory.map((item, index) => {
                const r = rarities[item.rarity];
                return `<div class="inventory-item ${item.rarity}">
                    <div class="info">
                        <span class="emoji">${item.emoji}</span>
                        <div>
                            <div class="name">${item.name}</div>
                            <div class="rarity" style="color:${r.color}">${r.name}</div>
                        </div>
                    </div>
                    <button class="sell-btn" data-index="${index}">🪙 ${item.price}</button>
                </div>`;
            }).join('');

            list.querySelectorAll('.sell-btn').forEach(btn => {
                btn.addEventListener('click', () => {
                    const idx = parseInt(btn.dataset.index);
                    const item = inventory[idx];
                    if (!item) return;
                    coins += item.price;
                    updateCoins();
                    inventory.splice(idx, 1);
                    saveInventory();
                    renderInventory();
                    document.getElementById('status').textContent = `💰 Продано за ${item.price} 🪙`;
                    setTimeout(() => document.getElementById('status').textContent = '🎰 Выбери кейс', 1500);
                });
            });
        }

        const invMenu = document.getElementById('inventoryMenu');
        document.getElementById('inventoryToggle').onclick = () => {
            invMenu.classList.toggle('open');
            renderInventory();
        };
        document.getElementById('closeInventory').onclick = () => {
            invMenu.classList.remove('open');
        };

        renderCases();
        renderInventory();

        console.log('🎰 Игра Крутка Кейсов запущена!');
    </script>
</body>
</html>
