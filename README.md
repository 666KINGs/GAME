<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Snake Game</title>
  <style>
    body {
      background: url('background.jpg') no-repeat center center fixed;
      background-size: cover;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      height: 100vh;
      margin: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      color: white;
      position: relative;
    }

    body::before {
      content: '';
      position: absolute;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0, 0, 0, 0.6);
      z-index: -1;
    }

    .scoreboard {
      display: flex;
      gap: 40px;
      margin-top: 20px;
      margin-bottom: 10px;
      font-size: 22px;
      font-weight: bold;
      color: #0f0;
      text-shadow: 1px 1px 2px #000;
    }

    canvas {
      background: #000;
      border: 4px solid #0f0;
      box-shadow: 0 0 20px #0f0;
      border-radius: 10px;
    }

    #newRecord {
      margin-top: 10px;
      font-size: 20px;
      font-weight: bold;
      color: gold;
      display: none;
      animation: blink 1s infinite;
    }

    @keyframes blink {
      0%, 100% { opacity: 1; }
      50% { opacity: 0.3; }
    }

    #gameOver {
      margin-top: 20px;
      font-size: 30px;
      font-weight: bold;
      color: #ff4c4c;
      text-shadow: 2px 2px 5px #000;
      display: none;
    }

    button {
      margin-top: 15px;
      padding: 10px 24px;
      font-size: 18px;
      cursor: pointer;
      border: none;
      border-radius: 8px;
      background: linear-gradient(to right, #4CAF50, #45A049);
      color: white;
      box-shadow: 0 4px 6px rgba(0,0,0,0.3);
      transition: 0.3s ease;
    }

    button:hover {
      background: linear-gradient(to right, #45A049, #3e8e41);
      transform: scale(1.05);
    }
  </style>
</head>
<body>

  <div class="scoreboard">
    <div id="score">スコア: 0</div>
    <div id="bestScore">最高得点: 0</div>
  </div>

  <canvas id="game" width="600" height="600"></canvas>
  <div id="newRecord">🎉 New Record!</div>
  <div id="gameOver">失敗しました ! 頑張れ <button id="restartBtn">再起動</button></div>

  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const scoreDisplay = document.getElementById('score');
    const bestScoreDisplay = document.getElementById('bestScore');
    const newRecordDisplay = document.getElementById('newRecord');
    const gameOverDisplay = document.getElementById('gameOver');
    const restartBtn = document.getElementById('restartBtn');

    const box = 20;
    const canvasSize = 600;

    let snake = [];
    let direction = 'RIGHT';
    let food;
    let score;
    let gameInterval;
    let speed;
    let bestScore = localStorage.getItem('bestScore') || 0;

    function init() {
      snake = [{ x: 9 * box, y: 10 * box }];
      direction = 'RIGHT';
      score = 0;
      speed = 140;
      scoreDisplay.textContent = `スコア: ${score}`;
      bestScoreDisplay.textContent = `最高得点: ${bestScore}`;
      newRecordDisplay.style.display = 'none';

      food = {
        x: Math.floor(Math.random() * 30) * box,
        y: Math.floor(Math.random() * 30) * box,
      };

      gameOverDisplay.style.display = 'none';
      if (gameInterval) clearInterval(gameInterval);
      gameInterval = setInterval(draw, speed);
    }

    function draw() {
      ctx.fillStyle = 'rgba(0, 0, 0, 0.92)';
      ctx.fillRect(0, 0, canvasSize, canvasSize);

      const gradient = ctx.createRadialGradient(
        canvasSize / 2, canvasSize / 2, 50,
        canvasSize / 2, canvasSize / 2, 300
      );
      gradient.addColorStop(0, 'rgba(0, 255, 0, 0.15)');
      gradient.addColorStop(1, 'rgba(0, 255, 0, 0)');
      ctx.fillStyle = gradient;
      ctx.fillRect(0, 0, canvasSize, canvasSize);

      ctx.fillStyle = 'white';
      ctx.fillRect(food.x, food.y, box, box);

      for (let i = 0; i < snake.length; i++) {
        const hue = (i * 25) % 360;
        ctx.fillStyle = i === 0 ? 'white' : `hsl(${hue}, 100%, 50%)`;
        ctx.fillRect(snake[i].x, snake[i].y, box, box);
      }

      let snakeX = snake[0].x;
      let snakeY = snake[0].y;

      if (direction === 'LEFT') snakeX -= box;
      else if (direction === 'UP') snakeY -= box;
      else if (direction === 'RIGHT') snakeX += box;
      else if (direction === 'DOWN') snakeY += box;

      if (
        snakeX < 0 || snakeX >= canvasSize ||
        snakeY < 0 || snakeY >= canvasSize ||
        collision({ x: snakeX, y: snakeY }, snake)
      ) {
        clearInterval(gameInterval);
        gameOverDisplay.style.display = 'block';
        return;
      }

      if (snakeX === food.x && snakeY === food.y) {
        score++;
        scoreDisplay.textContent = `スコア: ${score}`;

        if (score > bestScore) {
          bestScore = score;
          localStorage.setItem('bestScore', bestScore);
          bestScoreDisplay.textContent = `最高得点: ${bestScore}`;
          newRecordDisplay.style.display = 'block';
        }

        food = {
          x: Math.floor(Math.random() * 30) * box,
          y: Math.floor(Math.random() * 30) * box,
        };

        if (speed > 30) {
          speed -= 2;
          clearInterval(gameInterval);
          gameInterval = setInterval(draw, speed);
        }
      } else {
        snake.pop();
      }

      snake.unshift({ x: snakeX, y: snakeY });
    }

    function collision(head, array) {
      return array.some(segment => head.x === segment.x && head.y === segment.y);
    }

    document.addEventListener('keydown', (event) => {
      if (event.key === 'ArrowLeft' && direction !== 'RIGHT') direction = 'LEFT';
      else if (event.key === 'ArrowUp' && direction !== 'DOWN') direction = 'UP';
      else if (event.key === 'ArrowRight' && direction !== 'LEFT') direction = 'RIGHT';
      else if (event.key === 'ArrowDown' && direction !== 'UP') direction = 'DOWN';
    });

    restartBtn.addEventListener('click', init);

    init();
  </script>

</body>
</html>
