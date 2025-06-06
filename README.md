# bombandapplegame
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>사과와 폭탄 게임 (폭탄 증가 + 이미지 개선)</title>
<style>
  body {
    margin: 0;
    background: linear-gradient(to bottom, #87ceeb 0%, #ffffff 100%);
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    user-select: none;
  }
  canvas {
    background: #a0d8f7;
    border: 2px solid #333;
    border-radius: 10px;
    display: block;
  }
</style>
</head>
<body>
<canvas id="gameCanvas" width="400" height="600"></canvas>

<script>
(() => {
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

  const basket = {
    x: canvas.width / 2 - 25,
    y: canvas.height - 60,
    width: 50,
    height: 30,
    speed: 7,
  };

  let score = 0;
  let gameOver = false;

  const items = [];
  const itemSize = 30;
  const spawnRate = 0.03; // 아이템 생성 확률

  // 폭탄 출현 확률 (초기 20%, 게임 진행마다 증가)
  let bombProbability = 0.2;

  // 폭탄 효과음만 남김
  const soundBomb = new Audio('https://actions.google.com/sounds/v1/explosions/explosion.ogg');

  // 키보드 이벤트
  const keys = {};
  window.addEventListener('keydown', e => {
    if (e.key === 'ArrowLeft' || e.key === 'a') keys.left = true;
    if (e.key === 'ArrowRight' || e.key === 'd') keys.right = true;
  });
  window.addEventListener('keyup', e => {
    if (e.key === 'ArrowLeft' || e.key === 'a') keys.left = false;
    if (e.key === 'ArrowRight' || e.key === 'd') keys.right = false;
  });

  // 터치 이벤트 (모바일)
  let touchX = null;
  canvas.addEventListener('touchstart', e => {
    touchX = e.touches[0].clientX;
  });
  canvas.addEventListener('touchmove', e => {
    const newX = e.touches[0].clientX;
    const diff = newX - touchX;
    basket.x += diff;
    if (basket.x < 0) basket.x = 0;
    if (basket.x > canvas.width - basket.width) basket.x = canvas.width - basket.width;
    touchX = newX;
  });
  canvas.addEventListener('touchend', e => {
    touchX = null;
  });

  // 사과와 폭탄 이미지 직접 그리기 함수
  function drawApple(x, y, size) {
    // 빨간 사과 동그란 모양 + 잎사귀
    ctx.fillStyle = 'red';
    ctx.beginPath();
    ctx.ellipse(x + size/2, y + size/2, size/2 * 0.8, size/2, 0, 0, Math.PI * 2);
    ctx.fill();

    // 잎사귀 그리기
    ctx.fillStyle = 'green';
    ctx.beginPath();
    ctx.moveTo(x + size/2, y + size*0.2);
    ctx.lineTo(x + size/2 + 5, y + size*0.1);
    ctx.lineTo(x + size/2 + 2, y + size*0.3);
    ctx.fill();

    // 꼭지 그리기
    ctx.strokeStyle = 'brown';
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(x + size/2, y + size*0.1);
    ctx.lineTo(x + size/2, y);
    ctx.stroke();
  }

  function drawBomb(x, y, size) {
    // 검정색 폭탄 본체
    ctx.fillStyle = 'black';
    ctx.beginPath();
    ctx.arc(x + size/2, y + size/2, size/2 * 0.8, 0, Math.PI * 2);
    ctx.fill();

    // 불꽃 모양 (주황색/노란색)
    const flameX = x + size/2;
    const flameY = y + size/4;
    const flameRadius = size/6;

    // 주황색 원
    ctx.fillStyle = 'orange';
    ctx.beginPath();
    ctx.arc(flameX, flameY, flameRadius, 0, Math.PI * 2);
    ctx.fill();

    // 노란색 원 (더 작게)
    ctx.fillStyle = 'yellow';
    ctx.beginPath();
    ctx.arc(flameX, flameY, flameRadius * 0.6, 0, Math.PI * 2);
    ctx.fill();

    // 폭탄 꼭지 (회색)
    ctx.strokeStyle = 'gray';
    ctx.lineWidth = 3;
    ctx.beginPath();
    ctx.moveTo(x + size/2, y + size*0.1);
    ctx.lineTo(x + size/2, y);
    ctx.stroke();
  }

  function createItem() {
    // 시간이 지날수록 bombProbability 증가 (최대 70%)
    bombProbability += 0.0005;
    if (bombProbability > 0.7) bombProbability = 0.7;

    const type = Math.random() < bombProbability ? 'bomb' : 'apple';
    const x = Math.random() * (canvas.width - itemSize);
    const speed = 3 + Math.random() * 2;
    items.push({ x, y: 0, size: itemSize, type, speed });
  }

  function update() {
    if (gameOver) return;

    // 바구니 이동
    if (keys.left) basket.x -= basket.speed;
    if (keys.right) basket.x += basket.speed;
    if (basket.x < 0) basket.x = 0;
    if (basket.x > canvas.width - basket.width) basket.x = canvas.width - basket.width;

    // 아이템 위치 업데이트
    items.forEach(item => {
      item.y += item.speed;
    });

    // 충돌 검사 및 처리
    for (let i = items.length - 1; i >= 0; i--) {
      const item = items[i];
      if (
        item.y + item.size > basket.y &&
        item.x + item.size > basket.x &&
        item.x < basket.x + basket.width
      ) {
        if (item.type === 'bomb') {
          soundBomb.play();
          gameOver = true;
        } else if (item.type === 'apple') {
          score++;
          items.splice(i, 1);
        }
      } else if (item.y > canvas.height) {
        items.splice(i, 1);
      }
    }

    // 아이템 생성
    if (Math.random() < spawnRate) createItem();
  }

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 바구니
    ctx.fillStyle = 'brown';
    ctx.fillRect(basket.x, basket.y, basket.width, basket.height);

    // 아이템 그리기
    items.forEach(item => {
      if (item.type === 'apple') {
        drawApple(item.x, item.y, item.size);
      } else {
        drawBomb(item.x, item.y, item.size);
      }
    });

    // 점수 표시
    ctx.fillStyle = '#222';
    ctx.font = '24px Arial';
    ctx.fillText(`점수: ${score}`, 10, 30);

    // 게임 오버 표시
    if (gameOver) {
      ctx.fillStyle = 'rgba(0,0,0,0.5)';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      ctx.fillStyle = 'white';
      ctx.font = '48px Arial';
      ctx.textAlign = 'center';
      ctx.fillText('게임 오버', canvas.width / 2, canvas.height / 2 - 20);

      ctx.font = '24px Arial';
      ctx.fillText(`최종 점수: ${score}`, canvas.width / 2, canvas.height / 2 + 30);

      ctx.font = '20px Arial';
      ctx.fillText('새로고침 해서 다시 도전하세요!', canvas.width / 2, canvas.height / 2 + 70);
      ctx.textAlign = 'start';
    }
  }

  function gameLoop() {
    update();
    draw();
    requestAnimationFrame(gameLoop);
  }

  gameLoop();
})();
</script>
</body>
</html>
