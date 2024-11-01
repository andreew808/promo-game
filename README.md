gane<!DOCTYPE html> 
<html lang="uk"> 
<head> 
    <meta charset="UTF-8"> 
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
    <title>Ухиляння та збір</title> 
    <style> 
        body { 
            display: flex; 
            align-items: center; 
            justify-content: center; 
            height: 100vh; 
            margin: 0; 
            font-family: Arial, sans-serif; 
            background-color: #000000; /* Чорний фон */ 
            overflow: hidden; 
            color: #ffffff; /* Білий колір тексту */
        } 
        #game-container { 
            position: relative; 
            width: 100vw; 
            height: 100vh; 
            background-color: #000000; /* Чорний фон контейнера */ 
            overflow: hidden; 
            display: none; /* Сховати ігрове поле до натискання Play */
        } 
        #player { 
            position: absolute; 
            bottom: 10px; 
            left: calc(50% - 25px); 
            width: 50px; 
            height: 50px; 
            background-image: url('https://i.postimg.cc/NF3zP4xJ/Untitled337-20241031113103.png');
            background-size: contain; 
            background-repeat: no-repeat; 
        } 
        .item { 
            position: absolute; 
            width: 50px; 
            height: 50px; 
            background-size: contain; 
            background-repeat: no-repeat; 
        } 
        #score-board { 
            position: absolute; 
            top: 20px; 
            left: 20px; 
            font-size: 24px; 
            font-weight: bold; 
        } 
        #goal-board { 
            position: absolute; 
            top: 60px; 
            left: 20px; 
            font-size: 24px; 
            font-weight: bold; 
        } 
        #play-button { 
            padding: 10px 20px; 
            font-size: 24px; 
            cursor: pointer; 
            border: none; 
            background-color: #ffffff; 
            color: #000000; 
            border-radius: 5px; 
        } 
    </style> 
</head> 
<body> 
    <button id="play-button">Play</button>
    <div id="game-container"> 
        <div id="player"></div> 
        <div id="score-board">Рахунок: <span id="score">1000</span></div> 
        <div id="goal-board">Ціль: Досягніть 10,000 очок</div> 
    </div> 
    <script> 
        const gameContainer = document.getElementById('game-container'); 
        const player = document.getElementById('player'); 
        const scoreDisplay = document.getElementById('score'); 
        const playButton = document.getElementById('play-button');
        let score = 1000; // Початковий рахунок
        let gameOver = false; 

        // Запуск гри натисканням кнопки Play
        playButton.addEventListener('click', () => {
            playButton.style.display = 'none';
            gameContainer.style.display = 'block';
            score = 1000;
            scoreDisplay.textContent = score;
            gameOver = false;
            startGame();
        });

        // Обробка руху персонажа за курсором миші 
        document.addEventListener('mousemove', (event) => { 
            if (gameOver) return; 
            const mouseX = event.clientX; 
            const containerWidth = gameContainer.clientWidth; 
            const playerWidth = player.clientWidth; 
            // Обмеження руху персонажа в межах контейнера
            if (mouseX < playerWidth / 2) {
                player.style.left = 0px;
            } else if (mouseX > containerWidth - playerWidth / 2) {
                player.style.left = ${containerWidth - playerWidth}px;
            } else {
                player.style.left = ${mouseX - playerWidth / 2}px; 
            }
        }); 

        // Функція для створення корисного або небезпечного предмета 
        function spawnItem() { 
            const item = document.createElement('div'); 
            item.classList.add('item');
// Випадкове визначення типу предмета
            if (Math.random() > 0.5) { 
                // Корисний предмет
                item.classList.add('good'); 
                const goodImages = [
                    'https://i.postimg.cc/XJmW498F/pngwing-com-17.png', 
                    'https://i.postimg.cc/3r079b8Z/1730365426067.png'
                ];
                item.style.backgroundImage = url('${goodImages[Math.floor(Math.random() * goodImages.length)]}');
                item.dataset.type = 'good'; 
            } else { 
                // Шкідливий предмет
                item.classList.add('bad'); 
                const badImages = [
                    'https://i.postimg.cc/K8fy46fm/Untitled337-20241031110658.png',
                    'https://i.postimg.cc/LXrcr293/Untitled337-20241031110635.png',
                    'https://postimg.cc/dDJxY0zr'
                ];
                item.style.backgroundImage = url('${badImages[Math.floor(Math.random() * badImages.length)]}');
                item.dataset.type = 'bad'; 
            } 

            // Випадкове розташування предмета по горизонталі 
            item.style.left = ${Math.random() * (gameContainer.clientWidth - 50)}px; 
            item.style.top = 0px; 
            gameContainer.appendChild(item); 

            // Рух предмета вниз 
            const fallInterval = setInterval(() => { 
                if (gameOver) { 
                    clearInterval(fallInterval); 
                    return; 
                } 
                // Рухаємо предмет вниз 
                item.style.top = ${item.offsetTop + 5}px; 
                // Перевірка зіткнення з гравцем 
                if (checkCollision(player, item)) { 
                    clearInterval(fallInterval); 
                    item.remove(); 
                    if (item.dataset.type === 'good') { 
                        score += 200; // Додаємо 200 очок за корисний предмет
                        scoreDisplay.textContent = score; 
                        if (score >= 10000) endGame(true); // Завершення гри при досягненні 10,000 очок
                    } else { 
                        score -= 500; // Віднімаємо 500 очок за шкідливий предмет
                        scoreDisplay.textContent = score; 
                        if (score <= 0) endGame(false); // Завершення гри при досягненні нуля очок
                    } 
                } 
                // Видалення предмета, якщо він вийшов за межі поля 
                if (item.offsetTop > gameContainer.clientHeight) { 
                    clearInterval(fallInterval); 
                    item.remove(); 
                } 
            }, 20); 
        } 

        // Перевірка зіткнення 
        function checkCollision(player, item) { 
            const playerRect = player.getBoundingClientRect(); 
            const itemRect = item.getBoundingClientRect(); 
            return !( 
                playerRect.top > itemRect.bottom || 
                playerRect.bottom < itemRect.top || 
                playerRect.right < itemRect.left || 
                playerRect.left > itemRect.right 
            ); 
        } 

        // Кінець гри 
        function endGame(won) { 
            gameOver = true; 
            gameContainer.innerHTML = won ? '<h2>PROMO: LVDKGANG</h2>' : '<h2>Гру закінчено!</h2>'; 
            gameContainer.innerHTML += <p>Ваш рахунок: ${score}</p>; 
            setTimeout(() => {
                playButton.style.display = 'block';
                gameContainer.style.display = 'none';
                gameContainer.innerHTML = '<div id="player"></div><div id="score-board">Рахунок: <span id="score">1000</span></div><div id="goal-board">Ціль: Досягніть 10,000 очок</div>';
            }, 3000);
        } 

        // Функція для початку гри
        function startGame() {
            scoreDisplay.textContent = score;
            setInterval(() => { 
                if (!gameOver) spawnItem(); 
            }, 1000);
setInterval(() => { 
                if (!gameOver) spawnItem(); 
            }, 500); 
        }
    </script> 
</body> 
</html>
