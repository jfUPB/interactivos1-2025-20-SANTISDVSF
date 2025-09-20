
# Evidencias de la unidad 4

## Código

[[Enlace a la aplicación a modificar](URL)](http://www.generative-gestaltung.de/2/sketches/?02_M/M_1_2_01)

Código a modificar:

``` js

// P_2_1_2_01
//
// Generative Gestaltung – Creative Coding im Web
// ISBN: 978-3-87439-902-9, First Edition, Hermann Schmidt, Mainz, 2018
// Benedikt Groß, Hartmut Bohnacker, Julia Laub, Claudius Lazzeroni
// with contributions by Joey Lee and Niels Poldervaart
// Copyright 2018
//
// http://www.generative-gestaltung.de
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

/**
 * changing size and position of circles in a grid
 *
 * MOUSE
 * position x          : circle position
 * position y          : circle size
 * left click          : random position
 *
 * KEYS
 * s                   : save png
 */
'use strict';

var tileCount = 20;
var actRandomSeed = 0;

var circleAlpha = 130;
var circleColor;

function setup() {
  createCanvas(600, 600);
  noFill();
  circleColor = color(0, 0, 0, circleAlpha);
}

function draw() {
  translate(width / tileCount / 2, height / tileCount / 2);

  background(255);

  randomSeed(actRandomSeed);

  stroke(circleColor);
  strokeWeight(mouseY / 60);

  for (var gridY = 0; gridY < tileCount; gridY++) {
    for (var gridX = 0; gridX < tileCount; gridX++) {

      var posX = width / tileCount * gridX;
      var posY = height / tileCount * gridY;

      var shiftX = random(-mouseX, mouseX) / 20;
      var shiftY = random(-mouseX, mouseX) / 20;

      ellipse(posX + shiftX, posY + shiftY, mouseY / 15, mouseY / 15);
    }
  }
}

function mousePressed() {
  actRandomSeed = random(100000);
}

function keyReleased() {
  if (key == 's' || key == 'S') saveCanvas(gd.timestamp(), 'png');
}

```

[Enlace a la aplicación modificada](URL) https://editor.p5js.org/SANTISDVSF/sketches/T87WalDHh

Código modificado:

``` js

// ====== Variables del sketch (tomado y adaptado del ejemplo) ======
let c;
let lineModuleSize = 0;
let angle = 0;
let angleSpeed = 1;
const lineModule = [];
let lineModuleIndex = 0;
let clickPosX = 0;
let clickPosY = 0;

// ====== Soporte WebSerial (ASCII, coma-separado, fin de línea \n) ======
let port;
let connectBtn;
let connectionInitialized = false;
let microBitConnected = false;

// FSM
const STATES = {
  WAIT_MICROBIT_CONNECTION: "WAIT_MICROBIT_CONNECTION",
  RUNNING: "RUNNING",
};
let appState = STATES.WAIT_MICROBIT_CONNECTION;

// Datos recibidos del micro:bit
let microBitX = 0;
let microBitY = 0;
let microBitAState = false;
let microBitBState = false;
let prevmicroBitAState = false;
let prevmicroBitBState = false;

// === MODO DEMO (para casa sin micro:bit). Pon DEMO=true para simular datos. ===
const DEMO = false;
let demoT = 0;

function preload() {
  lineModule[1] = loadImage("02.svg");
  lineModule[2] = loadImage("03.svg");
  lineModule[3] = loadImage("04.svg");
  lineModule[4] = loadImage("05.svg");
}

function setup() {
  createCanvas(windowWidth, windowHeight);
  background(255);

  // Serial
  port = createSerial();
  connectBtn = createButton("Connect to micro:bit");
  connectBtn.position(10, 10);
  connectBtn.mousePressed(connectBtnClick);

  // Colores por defecto
  c = color(181, 157, 0);
  noCursor();
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}

function connectBtnClick() {
  if (!port.opened()) {
    // “MicroPython” + 115200 como en la guía
    port.open("MicroPython", 115200);
    connectionInitialized = false;
  } else {
    port.close();
  }
}

function updateButtonStates(newAState, newBState) {
  // Evento "pressed" de A (subida)
  if (newAState === true && prevmicroBitAState === false) {
    lineModuleSize = random(50, 160);  // tamaño nuevo
    clickPosX = microBitX;             // guardar posición de “click”
    clickPosY = microBitY;
    // print("A pressed");
  }
  // Evento "released" de B (bajada)
  if (newBState === false && prevmicroBitBState === true) {
    c = color(random(255), random(255), random(255), random(80, 100));
    // print("B released");
  }
  prevmicroBitAState = newAState;
  prevmicroBitBState = newBState;
}

function draw() {
  // --------- Botón y estado de conexión ----------
  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
    microBitConnected = false;
  } else {
    microBitConnected = true;
    connectBtn.html("Disconnect");

    // Limpiar buffer SOLO una vez al abrir
    if (!connectionInitialized) {
      port.clear();
      connectionInitialized = true;
    }

    // Leer paquetes completos “hasta \n”
    if (port.availableBytes() > 0) {
      let data = port.readUntil("\n");
      if (data) {
        data = data.trim();               // quitar \n
        const values = data.split(",");   // x,y,A,B
        if (values.length === 4) {
          microBitX = int(values[0]) + windowWidth / 2;
          microBitY = int(values[1]) + windowHeight / 2;
          microBitAState = values[2].toLowerCase() === "true";
          microBitBState = values[3].toLowerCase() === "true";
          updateButtonStates(microBitAState, microBitBState);
        } else {
          // Paquete parcial (pasa cuando el frame cae entre envíos); ignorar
          // print("Paquete incompleto (no 4 valores).");
        }
      }
    }
  }

  // --------- DEMO (sin micro:bit) ---------
  if (DEMO) {
    // trayectoria en 8 con “click” cada cierto tiempo y color aleatorio al soltar
    demoT += 0.02;
    microBitX = windowWidth / 2 + cos(demoT) * 200;
    microBitY = windowHeight / 2 + sin(demoT * 2) * 120;
    const a = (floor(demoT * 10) % 40) < 10;   // A true por ráfagas
    const b = (floor(demoT * 10) % 40) === 20; // B se “suelta” cada tanto
    microBitAState = a;
    microBitBState = b;
    updateButtonStates(microBitAState, microBitBState);
    microBitConnected = true; // para que pase a RUNNING
  }

  // --------- FSM por frame ---------
  switch (appState) {
    case STATES.WAIT_MICROBIT_CONNECTION:
      // No dibujamos nada hasta conectar
      if (microBitConnected === true) {
        // Preparar estado
        strokeWeight(0.75);
        c = color(181, 157, 0);
        noCursor();
        appState = STATES.RUNNING;
      }
      break;

    case STATES.RUNNING:
      // Si se pierde conexión, regresar a espera
      if (!microBitConnected && !DEMO) {
        cursor();
        appState = STATES.WAIT_MICROBIT_CONNECTION;
        break;
      }

      // “Dibujo cuando A está presionado” (igual que el mouseDown del original)
      if (microBitAState === true) {
        let x = microBitX;
        let y = microBitY;

        // Mantén SHIFT para forzar horizontal/vertical, como el original
        if (keyIsPressed && keyCode === SHIFT) {
          if (abs(clickPosX - x) > abs(clickPosY - y)) {
            y = clickPosY;
          } else {
            x = clickPosX;
          }
        }

        push();
        translate(x, y);
        rotate(radians(angle));
        if (lineModuleIndex !== 0) {
          tint(c);
          image(lineModule[lineModuleIndex], 0, 0, lineModuleSize, lineModuleSize);
        } else {
          stroke(c);
          line(0, 0, lineModuleSize, lineModuleSize);
        }
        angle += angleSpeed;
        pop();
      }
      break;
  }
}

// ====== Controles de teclado (del ejemplo original) ======
function keyPressed() {
  if (keyCode === UP_ARROW)   lineModuleSize += 5;
  if (keyCode === DOWN_ARROW) lineModuleSize -= 5;
  if (keyCode === LEFT_ARROW) angleSpeed -= 0.5;
  if (keyCode === RIGHT_ARROW)angleSpeed += 0.5;
}

function keyReleased() {
  if (key === "s" || key === "S") {
    const ts = year() + nf(month(), 2) + nf(day(), 2) + "_" + nf(hour(), 2) + nf(minute(), 2) + nf(second(), 2);
    saveCanvas(ts, "png");
  }
  if (keyCode === DELETE || keyCode === BACKSPACE) background(255);

  // invertir sentido
  if (key === "d" || key === "D") {
    angle += 180;
    angleSpeed *= -1;
  }

  // colores por defecto 1..4
  if (key === "1") c = color(181, 157, 0);
  if (key === "2") c = color(0, 130, 164);
  if (key === "3") c = color(87, 35, 129);
  if (key === "4") c = color(197, 0, 123);

  // módulos SVG 5..9
  if (key === "5") lineModuleIndex = 0;
  if (key === "6") lineModuleIndex = 1;
  if (key === "7") lineModuleIndex = 2;
  if (key === "8") lineModuleIndex = 3;
  if (key === "9") lineModuleIndex = 4;
}

```

## Video

[Video demostratativo][ https://youtu.be/xqK1A7_DlEw](https://youtu.be/N9Z4aeJUogo)








