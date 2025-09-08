# Unidad 1

## 🛠 Fase: Apply

### Actividad 5
¿Como funciona el sistema fisico interactivo?

Este sistema interacivo es un sistema basico donde el input fue el botón A del micro:bit y el output fue el color de un cuadrado en la pantalla que cambiaba interactivamente.

Al principio el cuadrado es verde. Al presionar el botón A, el micro:bit envía una señal por puerto serial que el programa en p5.js recibe, y el cuadrado cambia a rojo. Cuando se suelta el botón, el micro:bit envía una señal y el color vuelve a verde que era el color que había originalmente. Esta actividad permitió comprender como con un simple boton se pueden controlar elementos visuales digitales, abriendo la puerta a proyectos interactivos que pueen ayudar a explotar más la creatividad. 

### Actividad 6
Control de movimiento con micro:bit

Link programa en el editor de p5.js:
https://editor.p5js.org/SANTISDVSF/sketches/n1Ptt6Huk

CODIGO DEL PROGRAMA:
#### Código del micro:bit (Python)

```
from microbit import *
uart.init(baudrate=115200)

while True:
    if button_a.is_pressed():
        uart.write('L')  # Left
        sleep(150)
    elif button_b.is_pressed():
        uart.write('R')  # Right
        sleep(150)

```

Código en p5.js

```

let port;
let connectBtn;
let connectionInitialized = false;
let x = 200;

function setup() {
  createCanvas(400, 400);
  background(220);
  port = createSerial();
  connectBtn = createButton("Connect to micro:bit");
  connectBtn.position(80, 300);
  connectBtn.mousePressed(connectBtnClick);
}

function draw() {
  background(220);

  if (port.opened() && !connectionInitialized) {
    port.clear();
    connectionInitialized = true;
  }

  if (port.availableBytes() > 0) {
    let dataRx = port.read(1);
    if (dataRx === 'L') {
      x -= 10;
    } else if (dataRx === 'R') {
      x += 10;
    }
  }

  x = constrain(x, 0, width);
  ellipse(x, height / 2, 50, 50);
}

function connectBtnClick() {
  if (!port.opened()) {
    port.open("MicroPython", 115200);
    connectionInitialized = false;
  } else {
    port.close();
  }
}

```

