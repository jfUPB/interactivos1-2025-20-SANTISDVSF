
# Evidencias de la unidad 7

## ACTIVIDAD 1

URL Dev Tunnel: https://k1c57t7r-3000.use2.devtunnels.ms/mobile/

### ¿Por qué usar Dev Tunnels y no localhost/IP local?
localhost apunta al mismo dispositivo; desde el celular buscaría un servidor en el propio celular. La IP local solo funciona si ambos están en la misma Wi-Fi y sin bloqueos.

### ¿Qué hacen npm install y npm start?
npm install instala dependencias (Express y Socket.IO).
npm start inicia el servidor en http://localhost:3000.

En la terminal obersve varios mensajes como:
Server is listening on http://localhost:3000, New client connected, Received touch => {x, y, t}, Client disconnected.

<img width="612" height="129" alt="image" src="https://github.com/user-attachments/assets/50234e5a-387e-4246-a1ce-fe84897418f7" />
<img width="642" height="97" alt="image" src="https://github.com/user-attachments/assets/3b54620e-d887-43b1-ba06-95de0e6d02fa" />

En cuanto al comportamiento el círculo del desktop siguió el touch del móvil en tiempo real, tuvo una latencia baja.

<img width="663" height="897" alt="image" src="https://github.com/user-attachments/assets/42bf4360-d4f0-4421-9f2f-c8fd6be604f1" />
<img width="398" height="762" alt="image" src="https://github.com/user-attachments/assets/b5663f4f-4905-4d45-ae44-68393646cccb" />
<img width="540" height="1200" alt="image" src="https://github.com/user-attachments/assets/bd1d20f4-d64e-4dcc-a4c5-6be9f1f917ba" />


## ACTIVIDAD 2

¿Por qué es necesario Dev Tunnels en este escenario y cómo funciona conceptualmente?

R// Cuando corro el server me queda en http://localhost:3000, pero localhost solo sirve en el mismo equipo. Si desde el celular intento abrir localhost, el celu busca un servidor en el propio celular, no en mi PC. La IP local (tipo 192.168.1.X) solo sirve si el PC y el celu están en la misma Wi-Fi y sin bloqueos. Como yo quiero que funcione estando en otra red o con datos, uso Dev Tunnels de VS Code: me da una URL pública segura que apunta a mi server local. 

¿Qué hace touchMoved() y por qué uso threshold?

R// En p5.js, touchMoved() se ejecuta mientras arrastro el dedo sobre la pantalla. Ahí leo las coordenadas mouseX y mouseY y envío esos datos por Socket.IO hacia el servidor.
Uso un threshold (umbral) para no mandar mensajes por micro movimientos del dedo. Solo envío cuando el cambio es “suficientemente grande”. Eso evita saturar la red y la animación se ve más estable en el computador.

Dev Tunnels: (Ventajas y desventajas)
- Funciona aunque el celu y el PC estén en redes distintas (hasta con datos).
  
- Usa HTTPS y atraviesa firewalls/NAT.
  
X La URL es temporal y hay que tener el túnel activo (y sesión iniciada).

IP local (192.168.1.X): (Ventajas y desventajas)
- Simple si estamos en la misma Wi-Fi.

x No sirve por fuera de la red local y puede bloquearse por firewall.

x No es HTTPS por defecto.
 

## ACTIVIDAD 3

¿Qué hace express.static('public')?

En pocas palabras: le digo al servidor que sirva todo lo que haya en la carpeta public tal cual.
Entonces si entro a:

http://localhost:3000/desktop/  me muestra public/desktop/index.html

http://localhost:3000/mobile/ me muestra public/mobile/index.html

¿Cómo viaja un toque del celular hasta el escritorio? ¿Por qué socket.broadcast.emit?

En el celular, cuando muevo el dedo p5 llama touchMoved() y ahí hago: socket.emit('touch', { x, y, t }) que lo que quiere decir es que mando las coordendas al servidor.
En el servidor (Node), escucho ese mensaje con: socket.on('touch', (data) => { ... }) que significa que recibo las coordenadas.
El servidor reenvía ese dato a los demás con: socket.broadcast.emit('touch', data) o sea, a todos menos al que lo mandó.
En el escritorio, estoy escuchando: socket.on('touch', (data) => { x = data.x; y = data.y; }) eso es que actualizo el círculo.

¿Por qué uso socket.broadcast.emit y no io.emit o socket.emit?

R// El socket.emit se lo mandaría solo al que lo mandó entonces no me sirve. El io.emit se lo mandaría a todos, incluyendo al emisor cosa que no es necesario, y socket.broadcast.emit se lo manda a todos los demás que es justo lo que quiero, que lo reciban los escritorios, no el mismo celular.

Si conectaras dos computadores de escritorio y un móvil a este servidor, y movieras el dedo en el móvil, ¿Quién recibiría el mensaje retransmitido por el servidor? ¿Por qué?

R// Lo recibirian los dos computadores, el celular es el que emite, el servidor reenvía con broadcast (o sea, a todos menos al que emitió). Como los dos que faltan son los escritorios, los dos lo reciben y se mueven.

¿Qué información útil te proporcionan los mensajes console.log en el servidor durante la ejecución?

R// Los logs son las señales de vida del sistema por así decirlo: arranco bien, llegaron los toques, y quién está conectado.

## ACTIVIDAD 4
DIAGRAMA 

![Texto](https://github.com/user-attachments/assets/83d5fb2f-e1d2-4f8a-86b7-50385416c696)

## ACTIVIDAD 5

Construí una visual que llamé “Una Aventura”. La idea es un ambiente vintage, un disco de vinilo al fondo, humo suave que se mueve y unas ondas que reaccionan a la música. Desde el celular puedo cambiar el color del humo con un toque, y mover el dedo para ajustar cosas como el volumen y la longitud de las ondas. 

Enlaces usados:

PC: http://localhost:3000/desktop/

Cel: https://k1c57t7r-3000.use2.devtunnels.ms/mobile/

CODIGOS:

SERVER
````
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = new Server(server, { cors: { origin: "*" } });

app.use(express.static('public'));

io.on('connection', (socket) => {
  console.log('New client connected:', socket.id);

  // Reenvía cualquier 'touch' que llegue (sean moves o taps) a los demás
  socket.on('touch', (data) => {
    socket.broadcast.emit('touch', data);
  });

  socket.on('disconnect', (reason) => {
    console.log('Client disconnected:', socket.id, 'Reason:', reason);
  });
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Server is listening on http://localhost:${PORT}`);
});

````

DESKTOP INDEX
````

<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <title>Una Aventura — Desktop</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <!-- p5 y p5.sound -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.4/p5.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.4/addons/p5.sound.min.js"></script>
  <!-- socket.io -->
  <script src="/socket.io/socket.io.js"></script>
  <!-- tu sketch -->
  <script defer src="./sketch.js?v=6"></script>
  <style>
    :root { color-scheme: dark; }
    body { margin:0; background:#0f0c0a; color:#e7d9c8; font-family:system-ui; overflow:hidden; }
    .ui {
      position: fixed; top: 12px; left: 12px;
      background: rgba(0,0,0,.35); padding: 10px 14px; border-radius: 12px;
      backdrop-filter: blur(6px); font-size:14px;
    }
    code { background: rgba(255,255,255,.08); padding:2px 6px; border-radius:6px; }
  </style>
</head>
<body>
  <div class="ui">
    <div><b>Una Aventura — Desktop</b></div>
    <div style="opacity:.85;margin-top:6px">
      En el celu abre <code>/mobile/</code> — Tap: color · X: longitud de ondas · Y: giro vinilo
    </div>
  </div>
</body>
</html>

````

DESKTOP SKETCH

````

let socket;

// Estado visual
let warm = 0.7;            // 0=frío, 1=cálido (toggle con tap/tecla C)
let density = 0.8;         // densidad base del humo (constante aquí)
let puff = 0;              // (no usado ahora)

// Audio
let sound = null;
let amp, fft;

// Humo
const SMOKE_COUNT = 200;
let particles = [];
let t = 0;

// Ondas (control X del cel o flechas izq/der)
let waveAmpMul = 1.0;          // multiplicador de altura
let waveAmpMulTarget = 1.0;

// Volumen (control Y del cel o flechas arriba/abajo)
let volTarget = 0.7;           // 0..1
let volCurrent = 0.7;

// HUD y diagnóstico
let lastMobileAt = 0;          // millis() del último paquete móvil
let mobileOK = false;          // si llegaron moves recientemente
let toastMsg = '';
let toastUntil = 0;

function preload() {
  sound = loadSound('/assets/una.aventura.mp3');
}

function setup() {
  let canvas = createCanvas(window.innerWidth, window.innerHeight);
  noStroke();

  for (let i = 0; i < SMOKE_COUNT; i++) {
    particles.push(newSmoke(random(width), height + random(60)));
  }

  amp = new p5.Amplitude();
  fft = new p5.FFT(0.9, 256);

  // Socket: datos del celular
  socket = io();
  socket.on('touch', onTouchFromMobile);

  // Iniciar audio (autoplay puede requerir click)
  userStartAudio().then(() => {
    if (sound && !sound.isPlaying()) sound.loop();
  });
  canvas.mousePressed(() => {
    if (sound && !sound.isPlaying()) sound.loop();
  });

  // volumen inicial
  if (sound) sound.setVolume(volCurrent);
}

function onTouchFromMobile(d) {
  // Log por consola para depurar si llegan paquetes
  console.log('from mobile:', d);

  if (d.type === 'tap') {
    warm = (warm > 0.5) ? 0.15 : 0.9; // alterna frío ↔ cálido
    showToast('Color del humo cambiado (tap)');
  } else if (d.type === 'move') {
    const nx = constrain(d.nx ?? 0.5, 0, 1);
    const ny = constrain(d.ny ?? 0.5, 0, 1);

    // X: ALTURA DE ONDAS: 0→suave, 1→muy altas
    waveAmpMulTarget = map(nx, 0, 1, 0.6, 2.8);

    // Y: VOLUMEN (arriba = más fuerte)
    volTarget = map(1 - ny, 0, 1, 0.0, 1.0);

    lastMobileAt = millis();
    mobileOK = true;
  }
}

function draw() {
  background(15, 12, 10);

  // Suavizado de altura de ondas y volumen
  waveAmpMul = lerp(waveAmpMul, waveAmpMulTarget, 0.2);
  volCurrent = lerp(volCurrent, volTarget, 0.15);
  if (sound) sound.setVolume(volCurrent);

  drawVinyl();
  drawSmoke();
  drawWaves();

  drawHUD();   // ← Indicadores en pantalla
  vignette();
  filmGrain(220);
}

function drawVinyl() {
  push();
  translate(width/2, height/2);
  // (Sin giro; si quieres giro fijo: rotate(frameCount * 0.002);)

  const r = min(width, height) * 0.6;
  fill(30, 26, 22, 140);
  circle(0, 0, r);

  stroke(0,0,0,40); noFill();
  for (let i=0;i<18;i++) circle(0,0,map(i,0,17,r*0.15,r*0.48));
  noStroke();
  pop();
}

function drawSmoke() {
  const level = amp.getLevel();
  const audioBoost = 1 + level * 1.2;

  t += 0.005;
  const speedUp = (0.4 + density * 0.6) * audioBoost;

  for (let p of particles) {
    const angle = noise(p.x*0.002, p.y*0.002, t) * TWO_PI * 2.0 - PI;
    p.vx = cos(angle) * 0.4;
    p.vy = -1.0 * speedUp + sin(angle) * 0.2;

    p.x += p.vx; p.y += p.vy;
    p.life -= 0.004 * (1 + density*0.2) * audioBoost;

    const base = lerpColor(color(210,210,210,10), color(224,198,160,10), warm);
    fill(red(base), green(base), blue(base), 26);
    circle(p.x, p.y, p.size);

    if (p.life <= 0 || p.y < -40) {
      Object.assign(p, newSmoke(random(width*0.3, width*0.7), height + random(20)));
    }
  }
}

function drawWaves() {
  const c1 = lerpColor(color('#a8a8a8'), color('#e4c6a0'), warm);
  const c2 = color(red(c1), green(c1), blue(c1), 30);

  const wave = fft.waveform(); // -1..1
  const midY = height * 0.72;
  // altura base de ondas por nivel de audio
  const baseAmp = map(amp.getLevel(), 0, 0.5, 40, 160, true);
  const ampY = baseAmp * waveAmpMul;

  const L = wave.length;

  stroke(c2); strokeWeight(6); noFill();
  beginShape();
  for (let i = 0; i < L; i++) {
    const x = map(i, 0, L-1, 0, width);
    const y = midY + wave[i] * ampY * 1.2;
    vertex(x, y);
  }
  endShape();

  stroke(c1); strokeWeight(2.5); noFill();
  beginShape();
  for (let i = 0; i < L; i++) {
    const x = map(i, 0, L-1, 0, width);
    const y = midY + wave[i] * ampY;
    vertex(x, y);
  }
  endShape();

  stroke(c1); strokeWeight(1.2);
  beginShape();
  for (let i = 0; i < L; i++) {
    const x = map(i, 0, L-1, 0, width);
    const y = midY + 8 + wave[i] * (ampY * 0.85);
    vertex(x, y);
  }
  endShape();
}

function drawHUD() {
  // Panel
  push();
  const pad = 12;
  const w = 330, h = 128;
  fill(0, 0, 0, 120);
  rect(pad, pad, w, h, 12);

  // Textos
  fill(231, 217, 200);
  textAlign(LEFT, TOP);
  textSize(14);

  const volPct = (volCurrent * 100).toFixed(0) + '%';
  const ampStr = waveAmpMul.toFixed(2);
  const clrStr = (warm > 0.5) ? 'CÁLIDO' : 'FRÍO';
  const audioStr = (sound && sound.isPlaying()) ? 'PLAY' : 'PAUSA';

  text(`Una Aventura — HUD`, pad+12, pad+8);
  text(`Volumen: ${volPct}   (↑ / ↓)`, pad+12, pad+32);
  text(`Altura ondas: x${ampStr}   (← / →)`, pad+12, pad+52);
  text(`Color humo: ${clrStr}   (C)`, pad+12, pad+72);
  text(`Audio: ${audioStr}   (P para Play/Pause)`, pad+12, pad+92);

  // Estado móvil
  const since = millis() - lastMobileAt;
  const mobOK = mobileOK && since < 2000;
  const mobMsg = mobOK ? 'Móvil: OK (recibiendo)' : 'Móvil: sin movimiento';
  fill(mobOK ? color(120, 200, 120) : color(220, 160, 80));
  text(mobMsg, pad+12, pad+h-18);

  pop();

  // Toast temporal
  if (millis() < toastUntil) {
    push();
    const s = 16;
    textAlign(CENTER, CENTER);
    textSize(s);
    fill(0, 0, 0, 150);
    rect(width/2 - 200, height - 80, 400, 40, 10);
    fill(255);
    text(toastMsg, width/2, height - 60);
    pop();
  }
}

function showToast(msg, ms=1200) {
  toastMsg = msg;
  toastUntil = millis() + ms;
}

function keyPressed() {
  // Volumen: ↑/↓
  if (keyCode === UP_ARROW) {
    volTarget = constrain(volTarget + 0.1, 0, 1);
    showToast(`Volumen: ${(volTarget*100).toFixed(0)}%`);
  } else if (keyCode === DOWN_ARROW) {
    volTarget = constrain(volTarget - 0.1, 0, 1);
    showToast(`Volumen: ${(volTarget*100).toFixed(0)}%`);
  }

  // Altura de ondas: ←/→
  if (keyCode === RIGHT_ARROW) {
    waveAmpMulTarget = constrain(waveAmpMulTarget + 0.2, 0.6, 3.0);
    showToast(`Altura ondas: x${waveAmpMulTarget.toFixed(2)}`);
  } else if (keyCode === LEFT_ARROW) {
    waveAmpMulTarget = constrain(waveAmpMulTarget - 0.2, 0.6, 3.0);
    showToast(`Altura ondas: x${waveAmpMulTarget.toFixed(2)}`);
  }

  // Toggle color: C
  if (key.toLowerCase() === 'c') {
    warm = (warm > 0.5) ? 0.15 : 0.9;
    showToast(`Color humo: ${(warm>0.5)?'CÁLIDO':'FRÍO'}`);
  }

  // Play/Pause: P
  if (key.toLowerCase() === 'p' && sound) {
    if (sound.isPlaying()) { sound.pause(); showToast('Audio: PAUSA'); }
    else { sound.loop(); showToast('Audio: PLAY'); }
  }

  // Reset: R
  if (key.toLowerCase() === 'r') {
    waveAmpMulTarget = 1.0;
    volTarget = 0.7;
    warm = 0.7;
    showToast('Reset valores');
  }
}

function newSmoke(x,y){
  return { x,y, vx:0,vy:-1, size: random(10,28)*(0.9 + density*0.2), life: random(0.6,1.0) };
}

function vignette(){
  push(); noStroke();
  for(let i=0;i<8;i++){ fill(0,0,0,map(i,0,7,0,160)*0.5); rect(i,i,width-i*2,height-i*2); }
  pop();
}

function filmGrain(n=200){
  loadPixels();
  for(let i=0;i<n;i++){
    const x=floor(random(width)), y=floor(random(height)), idx=4*(y*width+x);
    const v=random(20);
    pixels[idx]  = max(0, pixels[idx]  - v);
    pixels[idx+1]= max(0, pixels[idx+1]- v);
    pixels[idx+2]= max(0, pixels[idx+2]- v);
  }
  updatePixels();
}

function windowResized(){ resizeCanvas(window.innerWidth, window.innerHeight); }

````

MOBILE INDEX

````

<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <title>Una Aventura — Mobile</title>
  <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.4/p5.min.js"></script>
  <script src="/socket.io/socket.io.js"></script>
  <script defer src="./sketch.js?v=8"></script>
  <style>
    :root { color-scheme: dark; }
    /* 👇 esto permite que p5 reciba touchmove sin que el navegador haga scroll/gestos */
    html, body, canvas { touch-action: none; }
    body { margin:0; background:#000; color:#fff; font-family:system-ui; }
    .hint {
      position:fixed; top:10px; left:10px; right:10px;
      text-align:center; opacity:.9; font-size:14px;
    }
  </style>
</head>
<body>
  <div class="hint">Tap: color del humo · X: altura de ondas (arr/der) · Y: volumen (arr/abajo)</div>
</body>
</html>

````

MOBILE SKETCH

````

// CONTROLES:
// - Tap: cambia color del humo (toggle)
// - X (izq/der): ALTURA DE LAS ONDAS (más notorio)
// - Y (arriba/abajo): VOLUMEN DE LA CANCIÓN (0..100%)

let socket;
let lastX = null, lastY = null;
// Umbral bajito para no perder movimientos cortos
const THRESHOLD = 2;
let tapArmed = true;

function setup() {
  createCanvas(window.innerWidth, window.innerHeight);
  socket = io();
  textAlign(CENTER, CENTER);
  noStroke();
}

function draw() {
  background(0);
  fill(255);
  textSize(18);
  text('Una Aventura — controles:\nTap: color del humo\nX: altura de ondas\nY: volumen de la canción', width/2, height/2);
}

function touchStarted() {
  tapArmed = true;
  // Emite la posición inicial (a veces el primer move no llega)
  const nx = constrain(mouseX / width, 0, 1);
  const ny = constrain(mouseY / height, 0, 1);
  socket.emit('touch', { type: 'move', nx, ny, t: Date.now() });
  lastX = mouseX; lastY = mouseY;
}

function touchMoved() {
  emitIfMoved(mouseX, mouseY);
  return false; // evita scroll/gestos del navegador
}

// Respaldo: algunos navegadores disparan 'mouseDragged' en vez de 'touchMoved'
function mouseDragged() {
  emitIfMoved(mouseX, mouseY);
}

function emitIfMoved(x, y) {
  const nx = constrain(x / width, 0, 1);
  const ny = constrain(y / height, 0, 1);
  if (lastX === null || dist(x, y, lastX, lastY) > THRESHOLD) {
    socket.emit('touch', { type: 'move', nx, ny, t: Date.now() });
    lastX = x; lastY = y;
    tapArmed = false;
  }
}

function touchEnded() {
  if (tapArmed) socket.emit('touch', { type: 'tap', t: Date.now() });
  tapArmed = true;
}

function windowResized(){ resizeCanvas(window.innerWidth, window.innerHeight); }

````

<img width="1908" height="937" alt="image" src="https://github.com/user-attachments/assets/91bcf64c-9ed5-47b9-b213-f2eadc613ab8" />
![WhatsApp Image 2025-10-16 at 9 58 06 PM (1)](https://github.com/user-attachments/assets/c9432f88-278a-464e-8b25-082f6f3490b5)

## AUTOEVALUACIÓN

Realice conscientemente las 5 actividades propuestas por el profe y organice toda la evidencia en la bitacora, asi que todo quedo completo, por lo tanto mi nota propuesta es un 5.0












