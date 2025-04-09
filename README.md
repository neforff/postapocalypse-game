# postapocalypse-game
норророл
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Иммунный</title>
    <style>
        body {
            font-family: 'Courier New', monospace;
            background-color: #111;
            color: #ccc;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            line-height: 1.6;
        }
        #game-container {
            background-color: #222;
            padding: 20px;
            border-radius: 5px;
            border: 1px solid #444;
            min-height: 300px;
        }
        #text-output {
            margin-bottom: 20px;
            white-space: pre-wrap;
            max-height: 400px;
            overflow-y: auto;
            padding: 10px;
            background-color: #1a1a1a;
            border-radius: 3px;
        }
        #input-container {
            display: flex;
            gap: 10px;
            margin-top: 15px;
        }
        #text-input {
            flex-grow: 1;
            padding: 8px;
            background-color: #333;
            color: #eee;
            border: 1px solid #555;
            border-radius: 3px;
        }
        #submit-btn {
            padding: 8px 15px;
            background-color: #4a4a4a;
            color: #eee;
            border: none;
            border-radius: 3px;
            cursor: pointer;
        }
        #submit-btn:hover {
            background-color: #5a5a5a;
        }
        .death {
            color: #ff5555;
            font-weight: bold;
        }
        .new-character {
            color: #55ff55;
            font-weight: bold;
        }
        .hint {
            color: #aaa;
            font-style: italic;
        }
        .character-panel {
            display: flex;
            justify-content: space-between;
            margin-bottom: 15px;
            padding: 10px;
            background-color: #2a2a2a;
            border-radius: 3px;
        }
        .character-info {
            flex: 1;
        }
        .character-name {
            font-weight: bold;
            color: #eee;
            margin-bottom: 5px;
        }
        .stat-bar {
            height: 20px;
            margin: 5px 0;
            border-radius: 3px;
            overflow: hidden;
            background-color: #333;
        }
        .health-bar {
            background-color: #d9534f;
            height: 100%;
            transition: width 0.3s;
        }
        .hunger-bar {
            background-color: #f0ad4e;
            height: 100%;
            transition: width 0.3s;
        }
        .backpack {
            flex: 0 0 200px;
            margin-left: 15px;
            padding: 10px;
            background-color: #333;
            border-radius: 3px;
            max-height: 120px;
            overflow-y: auto;
        }
        .backpack-title {
            font-weight: bold;
            margin-bottom: 5px;
            color: #eee;
        }
        .item {
            display: inline-block;
            margin: 2px;
            padding: 2px 5px;
            background-color: #444;
            border-radius: 3px;
            font-size: 0.9em;
        }
        .location {
            margin-top: 5px;
            font-style: italic;
            color: #aaa;
        }
    </style>
</head>
<body>
    <h1>ИММУННЫЙ</h1>
    <div id="game-container">
        <div id="character-panel" class="character-panel">
            <div class="character-info">
                <div class="character-name" id="character-name">Алексей, врач</div>
                <div>Здоровье:</div>
                <div class="stat-bar">
                    <div class="health-bar" id="health-bar" style="width: 100%"></div>
                </div>
                <div>Голод:</div>
                <div class="stat-bar">
                    <div class="hunger-bar" id="hunger-bar" style="width: 50%"></div>
                </div>
                <div class="location" id="location">Местоположение: больница</div>
            </div>
            <div class="backpack">
                <div class="backpack-title">Рюкзак:</div>
                <div id="inventory">аптечка, фонарик</div>
            </div>
        </div>
        <div id="text-output">Загрузка игры...</div>
        <div id="input-container">
            <input type="text" id="text-input" placeholder="Что вы сделаете?">
            <button id="submit-btn">➤</button>
        </div>
    </div>

    <script>
        // Состояние игры
        const gameState = {
            currentCharacter: 1,
            characters: [
                {
                    name: "Алексей",
                    profession: "врач",
                    inventory: ["аптечка", "фонарик"],
                    health: 100,
                    hunger: 50,
                    location: "больница",
                    clues: [],
                    alive: true
                },
                {
                    name: "Ирина",
                    profession: "ученый",
                    inventory: ["ноутбук", "образцы вируса"],
                    health: 80,
                    hunger: 60,
                    location: "лаборатория",
                    clues: ["вирус искусственного происхождения"],
                    alive: true
                },
                {
                    name: "Дмитрий",
                    profession: "военный",
                    inventory: ["пистолет", "рация"],
                    health: 120,
                    hunger: 40,
                    location: "военная база",
                    clues: ["правительство знало о вирусе"],
                    alive: true
                }
            ],
            gameOver: false,
            daysSurvived: 0,
            virusOrigin: "секретный эксперимент по созданию биологического оружия, который вышел из-под контроля",
            endingsFound: 0
        };

        // Элементы DOM
        const textOutput = document.getElementById('text-output');
        const textInput = document.getElementById('text-input');
        const submitBtn = document.getElementById('submit-btn');
        const characterName = document.getElementById('character-name');
        const healthBar = document.getElementById('health-bar');
        const hungerBar = document.getElementById('hunger-bar');
        const locationElement = document.getElementById('location');
        const inventoryElement = document.getElementById('inventory');

        // Начало игры
        function startGame() {
            updateCharacter();
            printMessage("Год 2024. Неизвестный вирус превратил 99% населения в агрессивных мутантов.\nВы - один из немногих, у кого есть иммунитет.\n\nВаша цель: выжить и раскрыть происхождение вируса.\n\nВведите 'помощь' для списка команд.");
            processLocation();
        }

        // Обновить информацию о текущем персонаже
        function updateCharacter() {
            const char = gameState.characters[gameState.currentCharacter - 1];
            
            // Обновляем панель персонажа
            characterName.textContent = `${char.name}, ${char.profession}`;
            healthBar.style.width = `${char.health}%`;
            hungerBar.style.width = `${char.hunger}%`;
            locationElement.textContent = `Местоположение: ${char.location}`;
            
            // Обновляем рюкзак
            inventoryElement.innerHTML = '';
            char.inventory.forEach(item => {
                const itemElement = document.createElement('span');
                itemElement.className = 'item';
                itemElement.textContent = item;
                inventoryElement.appendChild(itemElement);
            });
            
            // Выводим информацию в консоль
            printMessage(`\n=== ДЕНЬ ${gameState.daysSurvived} ===`);
            printMessage(`Вы играете за ${char.name}, ${char.profession}.`);
            
            if (char.clues.length > 0) {
                printMessage(`\nИзвестные зацепки:`);
                char.clues.forEach(clue => printMessage(`- ${clue}`));
            }
        }

        // Обработка текущей локации
        function processLocation() {
            const char = gameState.characters[gameState.currentCharacter - 1];
            let locationText = "";
            
            switch(char.location) {
                case "больница":
                    locationText = `Вы в полуразрушенной больнице. Вокруг разбросаны медицинские приборы и пустые койки.\nНа севере - аптека, на востоке - выход на улицу.`;
                    break;
                case "лаборатория":
                    locationText = `Вы в секретной лаборатории. Компьютеры все еще работают, но людей нет.\nНа западе - архив, на юге - выход.`;
                    break;
                case "военная база":
                    locationText = `Вы на заброшенной военной базе. Ворота сломаны, но в казармах можно найти припасы.\nНа севере - командный центр, на востоке - склад.`;
                    break;
                case "улица":
                    locationText = `Вы на пустынной улице. Разбитые машины и следы хаоса повсюду.\nНа западе - больница, на востоке - торговый центр.`;
                    break;
                default:
                    locationText = `Вы в неизвестном месте. Оглядитесь вокруг.`;
            }
            
            printMessage(`\n${locationText}`);
        }

        // Обработка команд игрока
        function processCommand(command) {
            if (gameState.gameOver) {
                if (command.toLowerCase() === "новая игра") {
                    resetGame();
                }
                return;
            }

            const char = gameState.characters[gameState.currentCharacter - 1];
            command = command.toLowerCase().trim();
            
            // Общие команды
            if (command === "помощь") {
                printMessage("\nДоступные команды:");
                printMessage("- осмотреться: описание текущего места");
                printMessage("- инвентарь: показать ваш инвентарь");
                printMessage("- идти [направление]: переместиться");
                printMessage("- исследовать: поискать полезные предметы");
                printMessage("- есть [предмет]: употребить пищу");
                printMessage("- использовать [предмет]: применить предмет");
                printMessage("- ждать: пропустить время");
                printMessage("- помощь: показать это сообщение");
                return;
            }
            
            if (command === "осмотреться") {
                processLocation();
                return;
            }
            
            if (command === "инвентарь") {
                printMessage(`\nВаш инвентарь: ${char.inventory.join(", ")}`);
                return;
            }
            
            if (command.startsWith("идти ")) {
                const direction = command.split(" ")[1];
                moveCharacter(direction);
                return;
            }
            
            if (command === "исследовать") {
                searchLocation();
                return;
            }
            
            if (command.startsWith("есть ")) {
                const item = command.split(" ")[1];
                eatItem(item);
                return;
            }
            
            if (command.startsWith("использовать ")) {
                const item = command.split(" ")[1];
                useItem(item);
                return;
            }
            
            if (command === "ждать") {
                passTime();
                return;
            }
            
            // Персонаж-специфичные команды
            if (char.profession === "врач" && command === "анализировать кровь") {
                if (char.inventory.includes("анализы")) {
                    printMessage("\nВы анализируете образцы крови. Вирус явно искусственного происхождения - найдены следы генной модификации.");
                    addClue("вирус создан в лаборатории");
                } else {
                    printMessage("\nУ вас нет образцов для анализа.");
                }
                return;
            }
            
            if (char.profession === "ученый" && command === "изучить образцы") {
                if (char.inventory.includes("образцы вируса")) {
                    printMessage("\nИзучая образцы под микроскопом, вы обнаруживаете маркеры военной лаборатории.");
                    addClue("вирус связан с военными");
                } else {
                    printMessage("\nУ вас нет образцов вируса.");
                }
                return;
            }
            
            if (char.profession === "военный" && command === "проверить рацию") {
                if (char.inventory.includes("рация")) {
                    printMessage("\nВы ловите слабый сигнал: '...это не было запланировано... проект 'Пламя' вышел из-под контроля...'");
                    addClue("проект 'Пламя' связан с вирусом");
                } else {
                    printMessage("\nУ вас нет рации.");
                }
                return;
            }
            
            printMessage("\nНеизвестная команда. Введите 'помощь' для списка доступных команд.");
        }

        // Перемещение персонажа
        function moveCharacter(direction) {
            const char = gameState.characters[gameState.currentCharacter - 1];
            let newLocation = char.location;
            let message = "";
            
            // Проверка на возможность перемещения
            if ((char.location === "больница" && direction === "север") || 
                (char.location === "лаборатория" && direction === "запад")) {
                newLocation = "аптека";
                message = "Вы в аптеке. Полки частично разграблены, но кое-что полезное осталось.";
            } else if (char.location === "больница" && direction === "восток") {
                newLocation = "улица";
                message = "Вы вышли на улицу. Воздух пропитан запахом гари и разложения.";
            } else if (char.location === "лаборатория" && direction === "юг") {
                newLocation = "улица";
                message = "Вы вышли из лаборатории на пустынную улицу.";
            } else if (char.location === "военная база" && direction === "север") {
                newLocation = "командный центр";
                message = "Вы в командном центре. На столе разбросаны документы с грифом 'Совершенно секретно'.";
            } else if (char.location === "военная база" && direction === "восток") {
                newLocation = "склад";
                message = "Вы на складе. Здесь хранятся припасы и оружие.";
            } else if (char.location === "улица" && direction === "запад") {
                newLocation = "больница";
                message = "Вы вошли в больницу. Тишина и темнота.";
            } else if (char.location === "улица" && direction === "восток") {
                newLocation = "торговый центр";
                message = "Вы в торговом центре. Витрины разбиты, везде следы паники.";
            } else {
                printMessage("\nВы не можете пойти в этом направлении.");
                return;
            }
            
            // Проверка на опасность (30% шанс встретить мутанта)
            if (Math.random() < 0.3 && !newLocation.includes("больница") && !newLocation.includes("лаборатория") && !newLocation.includes("база")) {
                encounterMutant();
                return;
            }
            
            char.location = newLocation;
            locationElement.textContent = `Местоположение: ${newLocation}`;
            printMessage(`\nВы идете ${direction}. ${message}`);
            passTime();
            processLocation();
        }

        // Поиск предметов в локации
        function searchLocation() {
            const char = gameState.characters[gameState.currentCharacter - 1];
            const items = {
                "больница": ["бинты", "лекарства", "анализы"],
                "аптека": ["антибиотики", "обезболивающее", "шприц"],
                "лаборатория": ["данные исследований", "флеш-накопитель", "защитный костюм"],
                "архив": ["секретные документы", "журнал экспериментов"],
                "военная база": ["паёк", "боеприпасы", "карта"],
                "командный центр": ["шифрованное сообщение", "ключ от хранилища"],
                "склад": ["консервы", "вода", "радио"],
                "улица": ["бутылка воды", "консервы"],
                "торговый центр": ["еда", "одежда", "аккумулятор"]
            };
            
            const possibleItems = items[char.location] || [];
            if (possibleItems.length === 0) {
                printMessage("\nВы ничего полезного не нашли.");
                passTime();
                return;
            }
            
            // Шанс найти предмет (60%)
            if (Math.random() < 0.6) {
                const foundItem = possibleItems[Math.floor(Math.random() * possibleItems.length)];
                char.inventory.push(foundItem);
                updateInventoryDisplay();
                printMessage(`\nВы нашли: ${foundItem}!`);
                
                // Особые предметы могут давать зацепки
                if (foundItem === "секретные документы") {
                    addClue("вирус разрабатывался как оружие");
                } else if (foundItem === "журнал экспериментов") {
                    addClue("ученые не успели создать вакцину");
                } else if (foundItem === "шифрованное сообщение") {
                    addClue("заражение было случайным");
                }
            } else {
                printMessage("\nВы ничего полезного не нашли.");
            }
            
            // Шанс встретить мутанта при поиске (20%)
            if (Math.random() < 0.2) {
                encounterMutant();
                return;
            }
            
            passTime();
        }

        // Обновление отображения инвентаря
        function updateInventoryDisplay() {
            const char = gameState.characters[gameState.currentCharacter - 1];
            inventoryElement.innerHTML = '';
            char.inventory.forEach(item => {
                const itemElement = document.createElement('span');
                itemElement.className = 'item';
                itemElement.textContent = item;
                inventoryElement.appendChild(itemElement);
            });
        }

        // Встреча с мутантом
        function encounterMutant() {
            const char = gameState.characters[gameState.currentCharacter - 1];
            printMessage("\nВнезапно перед вами появляется мутант! Его кожа покрыта язвами, глаза безумные.");
            
            // Проверка на оружие
            if (char.inventory.includes("пистолет") || char.inventory.includes("нож")) {
                const weapon = char.inventory.includes("пистолет") ? "пистолет" : "нож";
                printMessage(`Вы используете ${weapon} и убиваете мутанта!`);
                
                // Шанс получить травму (30%)
                if (Math.random() < 0.3) {
                    const damage = Math.floor(Math.random() * 20) + 10;
                    char.health -= damage;
                    healthBar.style.width = `${char.health}%`;
                    printMessage(`В ходе борьбы вы получили травму (-${damage} здоровья).`);
                    
                    if (char.health <= 0) {
                        characterDeath("смертельные раны от мутанта");
                        return;
                    }
                }
            } else {
                // Без оружия - высокий шанс смерти
                if (Math.random() < 0.7) {
                    characterDeath("нападение мутанта");
                    return;
                } else {
                    printMessage("Вам чудом удается убежать, но вы сильно ранены.");
                    char.health -= 40;
                    healthBar.style.width = `${char.health}%`;
                    
                    if (char.health <= 0) {
                        characterDeath("смертельные раны от мутанта");
                        return;
                    }
                }
            }
            
            passTime();
        }

        // Использование предмета
        function useItem(item) {
            const char = gameState.characters[gameState.currentCharacter - 1];
            
            if (!char.inventory.includes(item)) {
                printMessage(`\nУ вас нет предмета: ${item}`);
                return;
            }
            
            if (item === "аптечка") {
                char.health = Math.min(char.health + 40, 100);
                healthBar.style.width = `${char.health}%`;
                char.inventory = char.inventory.filter(i => i !== "аптечка");
                updateInventoryDisplay();
                printMessage("\nВы использовали аптечку. Здоровье восстановлено.");
            } else if (item === "фонарик") {
                printMessage("\nВы включили фонарик. Теперь в темных местах лучше видно.");
            } else if (item === "нож") {
                printMessage("\nВы осматриваете нож. Он может пригодиться для защиты.");
            } else if (item === "рация") {
                printMessage("\nВы включаете рацию, но слышите только помехи.");
            } else {
                printMessage(`\nВы не знаете, как использовать ${item}.`);
                return;
            }
            
            passTime();
        }

        // Употребление пищи
        function eatItem(item) {
            const char = gameState.characters[gameState.currentCharacter - 1];
            
            if (!char.inventory.includes(item)) {
                printMessage(`\nУ вас нет предмета: ${item}`);
                return;
            }
            
            const foodItems = ["консервы", "паёк", "еда", "бутылка воды"];
            if (foodItems.includes(item)) {
                char.hunger = Math.max(char.hunger - 30, 0);
                hungerBar.style.width = `${char.hunger}%`;
                char.inventory = char.inventory.filter(i => i !== item);
                updateInventoryDisplay();
                printMessage(`\nВы съели ${item}. Голод уменьшен.`);
            } else {
                printMessage(`\nВы не можете съесть ${item}.`);
                return;
            }
            
            passTime();
        }

        // Пропуск времени
        function passTime() {
            const char = gameState.characters[gameState.currentCharacter - 1];
            
            gameState.daysSurvived++;
            char.hunger += 15;
            hungerBar.style.width = `${char.hunger}%`;
            
            // Проверка голода
            if (char.hunger >= 100) {
                char.health -= 20;
                healthBar.style.width = `${char.health}%`;
                printMessage("\nВы сильно голодаете! Здоровье уменьшено.");
                
                if (char.health <= 0) {
                    characterDeath("голод");
                    return;
                }
            }
            
            // Проверка на раскрытие тайны
            if (char.clues.length >= 3) {
                trySolveMystery();
            }
            
            updateCharacter();
        }

        // Попытка раскрыть тайну вируса
        function trySolveMystery() {
            const char = gameState.characters[gameState.currentCharacter - 1];
            const requiredClues = {
                "врач": ["вирус создан в лаборатории", "вирус связан с военными", "проект 'Пламя' связан с вирусом"],
                "ученый": ["вирус искусственного происхождения", "вирус разрабатывался как оружие", "ученые не успели создать вакцину"],
                "военный": ["правительство знало о вирусе", "заражение было случайным", "проект 'Пламя' связан с вирусом"]
            };
            
            const hasAllClues = requiredClues[char.profession].every(clue => 
                char.clues.includes(clue));
            
            if (hasAllClues) {
                printMessage(`\nСобрав все зацепки, вы понимаете: ${gameState.virusOrigin}`);
                printMessage("\n=== ВЫ ПОБЕДИЛИ! ===");
                printMessage("Вы раскрыли тайну вируса и теперь можете попытаться найти способ спасти человечество.");
                gameState.endingsFound++;
                
                if (gameState.endingsFound < 3) {
                    printMessage(`\nНайдено концовок: ${gameState.endingsFound}/3. Попробуйте сыграть за других персонажей, чтобы узнать больше деталей.`);
                    gameState.gameOver = true;
                } else {
                    printMessage("\nВы раскрыли все тайны вируса! Теперь вы знаете полную правду о катастрофе.");
                    gameState.gameOver = true;
                }
            }
        }

        // Добавление зацепки
        function addClue(clue) {
            const char = gameState.characters[gameState.currentCharacter - 1];
            
            if (!char.clues.includes(clue)) {
                char.clues.push(clue);
                printMessage(`\nНовая зацепка: ${clue}`);
            }
        }

        // Смерть персонажа
        function characterDeath(cause) {
            const char = gameState.characters[gameState.currentCharacter - 1];
            char.alive = false;
            
            printMessage(`\n<span class="death">=== ВЫ УМЕРЛИ ===</span>`);
            printMessage(`Причина: ${cause}.`);
            printMessage(`Вы прожили ${gameState.daysSurvived} дней.`);
            
            // Проверка на наличие живых персонажей
            const aliveCharacters = gameState.characters.filter(c => c.alive);
            
            if (aliveCharacters.length > 0) {
                printMessage(`\n<span class="new-character">Вы начинаете играть за нового персонажа...</span>`);
                gameState.currentCharacter = gameState.characters.findIndex(c => c.alive) + 1;
                gameState.daysSurvived = 0;
                updateCharacter();
                processLocation();
            } else {
                printMessage("\nВсе ваши персонажи погибли. Игра окончена.");
                printMessage("Введите 'новая игра', чтобы начать заново.");
                gameState.gameOver = true;
            }
        }

        // Сброс игры
        function resetGame() {
            gameState.characters.forEach(char => {
                char.alive = true;
                char.health = char.profession === "военный" ? 120 : 100;
                char.hunger = 50;
                char.clues = [];
                
                // Сброс инвентаря к начальному
                if (char.profession === "врач") {
                    char.inventory = ["аптечка", "фонарик"];
                } else if (char.profession === "ученый") {
                    char.inventory = ["ноутбук", "образцы вируса"];
                } else {
                    char.inventory = ["пистолет", "рация"];
                }
                
                // Сброс локации
                if (char.profession === "врач") {
                    char.location = "больница";
                } else if (char.profession === "ученый") {
                    char.location = "лаборатория";
                } else {
                    char.location = "военная база";
                }
            });
            
            gameState.currentCharacter = 1;
            gameState.gameOver = false;
            gameState.daysSurvived = 0;
            gameState.endingsFound = 0;
            
            startGame();
        }

        // Вывод сообщения в игровое окно
        function printMessage(message) {
            const messageElement = document.createElement('div');
            messageElement.innerHTML = message;
            textOutput.appendChild(messageElement);
            textOutput.scrollTop = textOutput.scrollHeight;
        }

        // Обработчик ввода
        submitBtn.addEventListener('click', () => {
            const command = textInput.value;
            textInput.value = '';
            printMessage(`\n> ${command}`);
            processCommand(command);
        });
        
        textInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                const command = textInput.value;
                textInput.value = '';
                printMessage(`\n> ${command}`);
                processCommand(command);
            }
        });

        // Начало игры при загрузке
        window.onload = startGame;
    </script>
</body>
</html>
