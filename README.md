# penales.<!doctype html>

<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Juego de Penales - HTML/CSS/JS (principiantes)</title>
  <style>
    /* Estilos simples y claros para principiantes */
    :root{--green:#0b8a3e;--grass:#7bd26b;--white:#fff;--accent:#ffcc00}
    *{box-sizing:border-box;font-family:system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial}
    body{display:flex;align-items:center;justify-content:center;min-height:100vh;margin:0;background:linear-gradient(180deg,#7ec9a4 0%, #eafaf0 100%)}
    .container{width:960px;max-width:96vw;background:linear-gradient(#ffffffcc,#f6fff9);border-radius:14px;padding:18px;box-shadow:0 8px 30px rgba(20,60,40,0.12)}header{display:flex;align-items:center;gap:12px}
header h1{font-size:20px;margin:0}
.field{position:relative;margin-top:14px;height:360px;border-radius:8px;overflow:hidden;background:linear-gradient(#57b36a,#3b8f4b);display:flex;align-items:flex-end;justify-content:center;padding-bottom:36px}

/* Portería */
.goal{position:absolute;top:24px;left:50%;transform:translateX(-50%);width:66%;height:36%;border:6px solid rgba(255,255,255,0.9);border-bottom-width:12px;border-radius:8px;background:rgba(255,255,255,0.03);display:flex;align-items:center;justify-content:center}
.net{position:absolute;inset:0;pointer-events:none}

/* Jugador y arquero */
.player{position:absolute;bottom:32px;left:50%;transform:translateX(-50%);display:flex;flex-direction:column;align-items:center;gap:6px}
.ball{width:28px;height:28px;border-radius:50%;background:radial-gradient(circle at 30% 30%,#fff,#e6e6e6);border:2px solid #333;display:flex;align-items:center;justify-content:center}
.goalie{position:absolute;top:calc(24px + 6%);left:50%;transform:translateX(-50%);width:90px;height:120px;border-radius:10px;display:flex;align-items:flex-start;justify-content:center}
.goalie .head{width:40px;height:40px;background:#fbe3d7;border-radius:50%;margin-top:6px}
.goalie .body{width:60px;height:66px;background:#1d4b8f;border-radius:10px;margin-top:6px}

/* HUD */
.hud{display:flex;gap:12px;align-items:center;margin-top:12px}
.score{background:#fff;padding:10px;border-radius:8px;box-shadow:0 3px 10px rgba(0,0,0,0.06)}
button{background:var(--green);color:var(--white);border:none;padding:10px 14px;border-radius:8px;cursor:pointer;font-weight:600}
button.secondary{background:transparent;border:2px solid var(--green);color:var(--green);font-weight:700}

/* Controles: dirección */
.controls{display:flex;gap:8px;align-items:center}
.dir{width:80px;height:44px;border-radius:8px;background:#fff;border:2px solid #ddd;display:flex;align-items:center;justify-content:center;cursor:pointer;font-weight:700}
.dir.active{outline:3px solid rgba(11,138,62,0.15)}

/* Mensajes centrales */
.overlay{position:absolute;inset:0;display:flex;align-items:center;justify-content:center}
.message{background:rgba(255,255,255,0.95);padding:16px 20px;border-radius:12px;box-shadow:0 8px 30px rgba(0,0,0,0.08);text-align:center}

footer{display:flex;justify-content:space-between;align-items:center;margin-top:12px}
small{color:#666}

/* Simple animación de la pelota (usada por JS dinámicamente) */

  </style>
</head>
<body>
  <div class="container">
    <header>
      <h1>Juego de Penales - ¡Fácil y para principiantes!</h1>
      <div style="margin-left:auto">Rondas: <strong id="rounds">5</strong></div>
    </header><div class="field" id="field">
  <div class="goal" id="goal">
    <div class="net"></div>
    <div class="goalie" id="goalie" aria-hidden="true">
      <div class="head"></div>
      <div class="body"></div>
    </div>
  </div>

  <div class="player" style="bottom:20px">
    <div class="ball" id="ball" title="Pelota"></div>
  </div>

  <div class="overlay" id="overlay" style="pointer-events:none;">
    <div class="message" id="message" style="display:none"></div>
  </div>
</div>

<div class="hud">
  <div class="score">Puntuación: <strong id="score">0</strong></div>
  <div class="score">Intentos: <strong id="attempt">0</strong>/<strong id="total">5</strong></div>

  <div class="controls" style="margin-left:auto">
    <div class="dir" data-dir="left" id="left">IZQ</div>
    <div class="dir" data-dir="center" id="center">CENTRO</div>
    <div class="dir" data-dir="right" id="right">DER</div>
    <button id="kick">Patear</button>
    <button id="reset" class="secondary">Reiniciar</button>
  </div>
</div>

<footer>
  <small>Usa los botones o las teclas ← ↑ → para elegir dirección y presiona "Patear".</small>
  <small>Hecho con HTML/CSS/JS — código comentado para aprender.</small>
</footer>

  </div>  <script>
    // Variables principales
    const scoreEl = document.getElementById('score');
    const attemptEl = document.getElementById('attempt');
    const totalEl = document.getElementById('total');
    const ball = document.getElementById('ball');
    const goalie = document.getElementById('goalie');
    const message = document.getElementById('message');
    const overlay = document.getElementById('overlay');
    const leftBtn = document.getElementById('left');
    const centerBtn = document.getElementById('center');
    const rightBtn = document.getElementById('right');
    const kickBtn = document.getElementById('kick');
    const resetBtn = document.getElementById('reset');

    // Estado del juego
    let selected = 'center'; // dirección elegida por el jugador
    let score = 0;
    let attempt = 0;
    const total = 5; // rondas por defecto
    let inAction = false; // evita doble clicks mientras una animación corre

    totalEl.textContent = total;

    // Función para actualizar UI
    function updateUI(){
      scoreEl.textContent = score;
      attemptEl.textContent = attempt;
      // marcar dirección activa
      [leftBtn,centerBtn,rightBtn].forEach(btn=>btn.classList.toggle('active', btn.dataset.dir===selected));
    }

    // Selección por botones
    [leftBtn,centerBtn,rightBtn].forEach(btn=>{
      btn.addEventListener('click', ()=>{ if(inAction) return; selected = btn.dataset.dir; updateUI(); });
    });

    // Atajos de teclado: izquierda, arriba, derecha
    window.addEventListener('keydown', (e)=>{
      if(inAction) return;
      if(e.key === 'ArrowLeft'){ selected='left'; updateUI(); }
      if(e.key === 'ArrowUp'){ selected='center'; updateUI(); }
      if(e.key === 'ArrowRight'){ selected='right'; updateUI(); }
      if(e.key.toLowerCase() === ' ' || e.key.toLowerCase() === 'k'){ // espacio o k para patear
        kickBtn.click();
      }
    });

    // Lógica del arquero (elige hacia donde se lanza)
    function goalieChoice(){
      // arquero tiene una cierta probabilidad de adivinar la dirección
      const choices = ['left','center','right'];
      // probabilidad de adivinar: 33% base + pequeño margen si ya fallaste mucho
      const guess = choices[Math.floor(Math.random()*3)];
      return guess;
    }

    // Animar pelota hacia la dirección y comprobar si gol
    function animateKick(direction, callback){
      inAction = true;
      message.style.display = 'none';

      // Posiciones objetivo dependientes del tamano de la porteria
      const goalRect = document.getElementById('goal').getBoundingClientRect();
      const fieldRect = document.getElementById('field').getBoundingClientRect();
      const ballRect = ball.getBoundingClientRect();

      // calcular coordenadas relativas dentro del campo (px)
      const startX = ballRect.left + ballRect.width/2 - fieldRect.left;
      const startY = ballRect.top + ballRect.height/2 - fieldRect.top;

      // destino aproximado dentro de la porteria
      const targetY = 70; // hacia arriba del campo
      let targetX = fieldRect.width/2; // centro por defecto
      if(direction==='left') targetX = fieldRect.width*0.33;
      if(direction==='right') targetX = fieldRect.width*0.67;

      const duration = 550; // ms
      const start = performance.now();

      function step(now){
        const t = Math.min(1, (now-start)/duration);
        // animación simple: movimiento lineal con ease-out
        const ease = 1 - Math.pow(1-t, 3);
        const x = startX + (targetX - startX) * ease;
        const y = startY + (targetY - startY) * ease;
        ball.style.transform = `translate(${x - fieldRect.width/2}px, ${y - (fieldRect.height - 40)}px)`; // ajuste para centrar sobre el jugador

        if(t < 1){
          requestAnimationFrame(step);
        } else {
          setTimeout(()=>{ callback(); }, 160);
        }
      }
      requestAnimationFrame(step);
    }

    // Animar arquero (se lanza a una dirección)
    function animateGoalie(guess){
      // mover arquero a la izquierda/centro/derecha con una pequeña transformación
      const field = document.getElementById('field');
      const fieldWidth = field.getBoundingClientRect().width;
      let offset = 0; // px
      if(guess==='left') offset = -fieldWidth*0.18;
      if(guess==='right') offset = fieldWidth*0.18;
      goalie.style.transition = 'transform 400ms ease-out';
      goalie.style.transform = `translateX(${offset}px)`;
      // volver a centro tras 700ms
      setTimeout(()=>{ goalie.style.transform = 'translateX(0px)'; }, 1000);
    }

    // Mensajes transitorios
    function showMessage(text, timeout=1100){
      message.textContent = text; message.style.display = 'block';
      setTimeout(()=>{ message.style.display='none'; }, timeout);
    }

    // Acción de patear
    kickBtn.addEventListener('click', ()=>{
      if(inAction) return;
      if(attempt >= total) return; // ya se jugaron todas las rondas

      attempt++;
      updateUI();

      // el arquero adivina
      const guess = goalieChoice();
      animateGoalie(guess);

      // animar la pelota hacia la dirección seleccionada
      animateKick(selected, ()=>{
        // determinar si fue gol: si el arquero adivinó la misma dirección -> puede atajar
        const atajoExitoso = (guess === selected) && (Math.random() > 0.25); // si adivina, 75% de chances de atajar
        if(atajoExitoso){
          showMessage('¡Atajó el arquero!');
        } else {
          score++;
          showMessage('¡GOOOOL!');
        }
        updateUI();

        // reset pelota a su posición inicial visual
        ball.style.transition = 'transform 280ms ease-in';
        setTimeout(()=>{ ball.style.transition = ''; ball.style.transform=''; inAction=false; }, 380);

        // Si se terminaron las rondas, mostrar resultado final
        if(attempt >= total){
          setTimeout(()=>{
            showFinal();
          }, 600);
        }
      });
    });

    function showFinal(){
      const text = `Juego finalizado. Puntuaci\u00f3n: ${score} de ${total}.`;
      message.textContent = text; message.style.display='block';
      inAction = true; // bloqueamos hasta reinicio
    }

    // Reiniciar juego
    resetBtn.addEventListener('click', resetGame);
    function resetGame(){
      score = 0; attempt = 0; selected = 'center'; inAction=false;
      message.style.display='none';
      ball.style.transform=''; goalie.style.transform='translateX(0px)';
      updateUI();
    }

    // inicialización
    updateUI();

    // Pequeños consejos para quien aprende:
    // - Lee los comentarios de este archivo y modifica const total para cambiar la cantidad de rondas.
    // - Prueba cambiar la dificultad (por ejemplo reducir o aumentar la probabilidad de atajar en goalieChoice()).
    // - Observa cómo se usan getBoundingClientRect() y transform para mover elementos en pantalla.
  </script></body>
</html>
