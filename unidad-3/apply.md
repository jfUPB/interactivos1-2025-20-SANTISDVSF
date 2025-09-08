# Unidad 3


## 🛠 Fase: Apply


## Actividad 6

### Crear la bomba en p5.js

````

/**
 * Bomba 2.0 en p5.js — Máquina de Estados (CONFIG, ARMED, EXPLODED)
 * Controles con teclado:
 * - A: ++count (máx 60) en CONFIG; parte de clave en ARMED
 * - B: --count (mín 10) en CONFIG; parte de clave en ARMED
 * - S: armar (pasar a ARMED)
 * - T: reset desde EXPLODED
 * Temporizador: 1s en ARMED (no bloqueante, usando millis())
 */

let eventQueue = [];

class BombFSM {
  constructor() {
    this.STATE = { CONFIG: 'CONFIG', ARMED: 'ARMED', EXPLODED: 'EXPLODED' };
    this.state = this.STATE.CONFIG;

    this.count = 20;            // [10..60]
    this.lastTick = millis();   // para TIMER de 1s

    this.PASSWORD = ['A','B','A'];
    this.key = new Array(this.PASSWORD.length).fill('');
    this.keyindex = 0;
  }

  // Procesa eventos genéricos
  processEvent(ev) {
    if (!ev) return;

    switch (this.state) {
      case this.STATE.CONFIG:
        if (ev === 'A') {
          this.count = Math.min(this.count + 1, 60);
        } else if (ev === 'B') {
          this.count = Math.max(this.count - 1, 10);
        } else if (ev === 'S') {
          this.lastTick = millis();
          this.state = this.STATE.ARMED;
        }
        break;

      case this.STATE.ARMED:
        if (ev === 'TIMER') {
          this.count -= 1;
          if (this.count <= 0) {
            this.count = 0;
            this.state = this.STATE.EXPLODED;
          }
        } else if (ev === 'A' || ev === 'B') {
          this.key[this.keyindex] = ev;
          this.keyindex++;
          if (this.keyindex === this.PASSWORD.length) {
            // Validar clave
            let ok = true;
            for (let i = 0; i < this.PASSWORD.length; i++) {
              if (this.key[i] !== this.PASSWORD[i]) { ok = false; break; }
            }
            if (ok) {
              this.count = 20;
              this.keyindex = 0;
              this.state = this.STATE.CONFIG;
            } else {
              this.keyindex = 0;
            }
          }
        }
        break;

      case this.STATE.EXPLODED:
        if (ev === 'T') {
          this.count = 20;
          this.lastTick = millis();
          this.state = this.STATE.CONFIG;
        }
        break;
    }
  }

  // Genera el evento TIMER cada 1 segundo
  update() {
    if (this.state === this.STATE.ARMED) {
      const now = millis();
      if (now - this.lastTick >= 1000) {
        this.lastTick = now;
        eventQueue.push('TIMER');
      }
    }
  }

  // Dibujo en canvas (acciones fuera de eventos)
  render() {
    background(20);
    textAlign(CENTER, CENTER);
    noStroke();

    fill(255);
    textSize(20);
    text('BOMBA 2.0 — FSM en p5.js', width/2, 30);

    textSize(16);
    text(`Estado: ${this.state}`, width/2, 70);

    textSize(80);
    if (this.state === this.STATE.EXPLODED) {
      fill(255, 80, 80);
      text('💀', width/2, height/2);
    } else {
      fill(200);
      text(this.count.toString(), width/2, height/2);
    }

    fill(180);
    textSize(14);
    text('Teclado: A(+), B(-), S(armar), T(reset)', width/2, height - 30);
  }
}

let bomb;

function setup() {
  createCanvas(420, 320);
  bomb = new BombFSM();
}

function draw() {
  // 1) Actualización no bloqueante (TIMER)
  bomb.update();

  // 2) Consumir eventos en cola
  while (eventQueue.length > 0) {
    const ev = eventQueue.shift();
    bomb.processEvent(ev);
  }

  // 3) Dibujar
  bomb.render();
}

// Capturar eventos de teclado
function keyPressed() {
  const k = key.toUpperCase();
  if ('ABST'.includes(k)) eventQueue.push(k);
}


````


<img width="476" height="355" alt="image" src="https://github.com/user-attachments/assets/d1c9e01b-6b39-42f4-bedf-6ffcdf9c0dcc" />

## Acitivad 7

### Bomba en p5.js + controles del micro:bit

````

/**
}
break;
case this.STATE.EXPLODED:
if (ev === 'T') { this.count = 20; this.lastTick = millis(); this.state = this.STATE.CONFIG; }
break;
}
}
update() {
if (this.state === this.STATE.ARMED) {
const now = millis();
if (now - this.lastTick >= 1000) { this.lastTick = now; eventQueue.push('TIMER'); }
}
}
render() {
background(10);
textAlign(CENTER,CENTER); noStroke();


fill(255); textSize(20); text('BOMBA 2.0 — p5 + micro:bit (Serial)', width/2, 30);
textSize(16); text(`Estado: ${this.state}`, width/2, 70);


textSize(80);
if (this.state === 'EXPLODED') { fill(255,80,80); text('💀', width/2, height/2); }
else { fill(230); text(this.count.toString(), width/2, height/2); }


fill(180); textSize(14);
const conn = port && port.opened();
text(`Serial: ${conn ? 'Conectado' : 'Desconectado'} — Teclado: A B S T`, width/2, height - 30);
}
}


let bomb;


function setup() {
createCanvas(520, 360);
bomb = new BombFSM();


port = createSerial();
connectBtn = createButton('Conectar micro:bit');
connectBtn.position(20, 20);
connectBtn.mousePressed(() => {
if (!port.opened()) { port.open('MicroPython', 115200); connectionInitialized = false; }
else { port.close(); }
});
}


function draw() {
// Inicialización de Serial (una vez abierto)
if (port && port.opened() && !connectionInitialized) {
port.clear();
connectionInitialized = true;
}


// Lectura no bloqueante del puerto (micro:bit envía caracteres sueltos)
if (port && port.opened()) {
let inByte = port.read(); // suele devolver string con 1 char si hay datos
if (inByte && typeof inByte === 'string' && inByte.length > 0) {
const c = inByte.charAt(0).toUpperCase();
if ('ABST'.includes(c)) eventQueue.push(c);
}
}


// FSM
bomb.update();
while (eventQueue.length > 0) bomb.processEvent(eventQueue.shift());
bomb.render();


// UI botón
if (connectBtn) connectBtn.html(port && port.opened() ? 'Desconectar' : 'Conectar micro:bit');
}


function keyPressed() {
const k = key.toUpperCase();
if ('ABST'.includes(k)) eventQueue.push(k);
}

````

## Código del micro:bit (control por Serial)

````

# MicroPython — Control remoto simple por Serial
# Envía 'A','B','S','T' cuando se activan los sensores/botones
from microbit import *


uart.init(baudrate=115200)


def send(ch):
uart.write(ch)


while True:
if button_a.was_pressed():
send('A')
if button_b.was_pressed():
send('B')
if accelerometer.was_gesture('shake'):
send('S')
if pin_logo.is_touched():
send('T')
sleep(10)

````



