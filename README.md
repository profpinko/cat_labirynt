# cat_labirynt
Wstępna gra z kotem w labiryncie
<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Labirynt Czarnego Kota</title>
  <style>
    body { background: #111; color: #eee; font-family: sans-serif; text-align: center; }
    canvas { display: block; margin: 20px auto; background: #222; border: 2px solid #555; }
  </style>
</head>
<body>
  <h1>Labirynt Czarnego Kota</h1>
  <p>Poprowadź dusze do wyjścia, zbierz tuńczyka, unikaj złych duchów.</p>
  <canvas id="gameCanvas" width="640" height="640"></canvas>
  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const tileSize = 64;

    const levels = [
      [
        [1,1,1,1,1,1,1,1,1,1],
        [1,0,2,0,1,0,3,0,4,1],
        [1,0,1,0,1,0,1,0,0,1],
        [1,0,1,0,0,0,1,0,0,1],
        [1,0,1,1,1,0,1,0,5,1],
        [1,0,0,0,1,0,1,0,0,1],
        [1,1,1,0,1,0,0,0,0,1],
        [1,4,0,0,1,1,1,1,0,1],
        [1,0,3,0,0,2,0,0,0,1],
        [1,1,1,1,1,1,1,1,1,1]
      ],
      [
        [1,1,1,1,1,1,1,1,1,1],
        [1,2,0,0,1,0,4,0,3,1],
        [1,0,1,0,1,0,1,0,0,1],
        [1,0,1,0,0,0,1,1,0,1],
        [1,0,1,1,1,0,0,0,5,1],
        [1,0,0,0,1,1,1,0,0,1],
        [1,0,1,0,0,2,0,1,0,1],
        [1,0,1,1,1,1,0,1,0,1],
        [1,3,0,0,4,0,0,0,2,1],
        [1,1,1,1,1,1,1,1,1,1]
      ],
      [
        [1,1,1,1,1,1,1,1,1,1],
        [1,0,3,0,1,4,0,2,0,1],
        [1,0,1,0,1,1,0,1,0,1],
        [1,2,1,0,0,0,0,1,0,1],
        [1,0,1,1,1,1,0,1,5,1],
        [1,0,0,0,0,1,0,1,0,1],
        [1,1,1,1,0,1,0,1,0,1],
        [1,4,0,0,0,1,0,1,2,1],
        [1,0,0,3,0,0,0,0,0,1],
        [1,1,1,1,1,1,1,1,1,1]
      ],
      [
        [1,1,1,1,1,1,1,1,1,1],
        [1,0,0,2,0,1,0,3,0,1],
        [1,0,1,1,0,1,0,1,0,1],
        [1,4,1,0,0,0,0,1,0,1],
        [1,0,1,0,1,1,0,1,5,1],
        [1,0,1,0,1,0,0,1,0,1],
        [1,0,0,0,1,0,1,1,0,1],
        [1,1,1,0,1,0,0,0,2,1],
        [1,3,0,0,0,4,0,2,0,1],
        [1,1,1,1,1,1,1,1,1,1]
      ],
      [
        [1,1,1,1,1,1,1,1,1,1],
        [1,2,0,0,4,0,0,3,0,1],
        [1,0,1,0,1,1,0,1,0,1],
        [1,0,1,0,0,0,0,1,0,1],
        [1,0,1,1,1,1,0,1,5,1],
        [1,0,0,0,1,0,0,1,0,1],
        [1,1,1,0,1,0,1,1,0,1],
        [1,3,0,0,0,2,0,0,4,1],
        [1,0,0,4,0,0,2,0,0,1],
        [1,1,1,1,1,1,1,1,1,1]
      ]
    ];

    let currentLevel = 0, map, totalSouls;
    let player = { x:1, y:1, souls:0, tuna:0 }, gameOver=false, gameWin=false;

    function loadLevel(i) {
      map = levels[i].map(r=>r.slice());
      totalSouls = map.flat().filter(v=>v===2).length;
      player = { x:1, y:1, souls:0, tuna:0 };
      gameOver = gameWin = false;
      document.querySelector('p').textContent = `Poziom ${i+1}: poprowadź dusze, zbierz tuńczyka, unikaj duchów`;
      draw();
    }

    function draw() {
      ctx.clearRect(0,0,canvas.width,canvas.height);
      for (let y=0;y<map.length;y++){
        for (let x=0;x<map[y].length;x++){
          const t=map[y][x], px=x*tileSize, py=y*tileSize;
          ctx.fillStyle='#222'; ctx.fillRect(px,py,tileSize,tileSize);
          if(t===1) ctx.fillStyle='#444',ctx.fillRect(px,py,tileSize,tileSize);
          if(t===2) { ctx.fillStyle='#fff'; ctx.beginPath(); ctx.arc(px+32,py+32,16,0,2*Math.PI); ctx.fill(); }
          if(t===3) ctx.fillStyle='#ff0',ctx.fillRect(px+16,py+16,32,32);
          if(t===4) { ctx.fillStyle='#f00'; ctx.beginPath(); ctx.arc(px+32,py+32,21,0,2*Math.PI); ctx.fill(); }
          if(t===5) ctx.fillStyle='#0f0',ctx.fillRect(px,py,tileSize,tileSize);
        }
      }
      ctx.fillStyle='#000'; ctx.beginPath(); ctx.arc(player.x*tileSize+32,player.y*tileSize+32,21,0,2*Math.PI); ctx.fill();
      ctx.fillStyle='#fff'; ctx.fillRect(player.x*tileSize+24,player.y*tileSize+24,4,8);
      ctx.fillRect(player.x*tileSize+40,player.y*tileSize+24,4,8);
      ctx.fillStyle='#eee'; ctx.font='20px sans-serif';
      ctx.fillText(`Poziom: ${currentLevel+1}`,10,600);
      ctx.fillText(`Dusze: ${player.souls}/${totalSouls}`,10,630);
      ctx.fillText(`Tuńczyk: ${player.tuna}`,200,630);
    }

    function move(dx,dy) {
      if(gameOver||gameWin) return;
      const nx=player.x+dx, ny=player.y+dy, t=map[ny]?.[nx];
      if (t === undefined || t === 1) return;
      if(t===4){ gameOver=true; alert('Złapany przez złego ducha! Gra zakończona.'); return; }
      if(t===2) player.souls++,map[ny][nx]=0;
      if(t===3) player.tuna++,map[ny][nx]=0;
      if(t===5 && player.souls===totalSouls){
        gameWin=true;
        if(currentLevel<levels.length-1){ currentLevel++; alert('Poziom ukończony!'); loadLevel(currentLevel); }
        else alert('Wszystkie poziomy ukończone! Wygrałeś!');
        return;
      }
      player.x=nx; player.y=ny;
      requestAnimationFrame(draw);
    }

    document.addEventListener('keydown',e=>{
      const m={ArrowUp:[0,-1],ArrowDown:[0,1],ArrowLeft:[-1,0],ArrowRight:[1,0]};
      if(m[e.key]) move(...m[e.key]);
    });

    loadLevel(currentLevel);
  </script>
</body>
</html>
