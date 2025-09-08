# Evidencias de la unidad 4

## Código

[[Enlace a la aplicación a modificar](URL)](http://www.generative-gestaltung.de/2/sketches/?02_M/M_1_2_01)

Código a modificar:

``` js

// M_1_2_01
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
 * order vs random!
 * how to interpolate beetween a free composition (random) and a circle shape (order)
 *
 * MOUSE
 * position x          : fade between random and circle shape
 *
 * KEYS
 * s                   : save png
 */
'use strict';

var sketch = function(p) {

  var actRandomSeed = 0;
  var count = 150;

  p.setup = function() {
    p.createCanvas(800,800);
    p.cursor(p.CROSS);
    p.noStroke();
    p.fill(0,130,164);
  };

  p.draw = function() {
    p.background(255);

    var faderX = p.mouseX / p.width;

    p.randomSeed(actRandomSeed);
    var angle = p.radians(360 / count);
    for (var i = 0; i < count; i++) {
      // positions
      var randomX = p.random(0,p.width);
      var randomY = p.random(0,p.height);
      var circleX = p.width / 2 + p.cos(angle * i) * 300;
      var circleY = p.height / 2 + p.sin(angle * i) * 300;

      var x = p.lerp(randomX,circleX,faderX);
      var y = p.lerp(randomY,circleY,faderX);

      p.ellipse(x,y,11,11);
    };
  };

  p.mousePressed = function() {
    actRandomSeed = p.random(100000);
  };

  p.keyReleased = function(){
    if (p.key == 's' || p.key == 'S') p.saveCanvas(gd.timestamp(), 'png');
  };

};

var myp5 = new p5(sketch);


```

[Enlace a la aplicación modificada](URL) https://editor.p5js.org/SANTISDVSF/sketches/T87WalDHh

Código modificado:

``` js

/****************************************************
 * M_1_2_01 (orden vs random) + micro:bit (p5.webserial)
 * Sensores (4/4):
 *   - xValue  -> fader random ↔ círculo
 *   - yValue  -> tamaño de los puntos (ellipse)
 *   - button A (press)    -> nueva semilla (como mousePressed original)
 *   - button B (release)  -> cambiar color
 * SIN simulación: requiere micro:bit real
 ****************************************************/

// ------- Serial + estados -------
let port, connectBtn, connectionInitialized = false, microBitConnected = false;

const STATES = {
  WAIT_MICROBIT_CONNECTION: "WAIT_MICROBIT_CONNECTION",
  RUNNING: "RUNNING",
};
let appState = STATES.WAIT_MICROBIT_CONNECTION;

// Sensores
let microBitX = 0, microBitY = 0;
let microBitA = false, microBitB = false;
let prevA = false, prevB = false;

// ------- Parámetros del sketch original -------
let actRandomSeed = 0;
let count = 150;
let dotColor;

// Util: -1024..1024 -> 0..1
function tilt01(v) {
  return constrain(map(v, -1024, 1024, 0, 1), 0, 1);
}

function setup() {
  // Igual que el original (800x800)
  createCanvas(800, 800);
  cursor(CROSS);
  noStroke();
  dotColor = color(0, 130, 164); // color inicial del ejemplo

  // WebSerial
  port = createSerial();
  connectBtn = createButton("Connect to micro:bit");
  connectBtn.position(12, 12);
  connectBtn.mousePressed(() => {
    if (!port.opened()) {
      port.open("MicroPython", 115200);
      connectionInitialized = false;
    } else {
      port.close();
    }
  });
}

function updateEdges(newA, newB) {
  // A PRESSED -> nueva semilla (equivale a click en el original)
  if (newA && !prevA) {
    actRandomSeed = random(100000);
  }
  // B RELEASED -> cambiar color
  if (!newB && prevB) {
    dotColor = color(random(255), random(255), random(255));
  }
  prevA = newA;
  prevB = newB;
}

function draw() {
  // ----- gestión de puerto -----
  if (!port.opened()) {
    connectBtn.html("Connect to micro:bit");
    microBitConnected = false;
  } else {
    connectBtn.html("Disconnect");
    microBitConnected = true;

    if (!connectionInitialized) {
      port.clear();
      connectionInitialized = true;
    }
    if (port.availableBytes() > 0) {
      let line = port.readUntil("\n");
      if (line) {
        const vals = line.trim().split(",");
        if (vals.length === 4) {
          microBitX = int(vals[0]);
          microBitY = int(vals[1]);
          microBitA = vals[2].toLowerCase() === "true";
          microBitB = vals[3].toLowerCase() === "true";
          updateEdges(microBitA, microBitB);
        }
      }
    }
  }

  // ----- máquina de estados -----
  switch (appState) {
    case STATES.WAIT_MICROBIT_CONNECTION:
      if (microBitConnected) {
        background(255);
        appState = STATES.RUNNING;
      } else {
        // pantalla de espera
        background(20);
        fill(240);
        noStroke();
        textAlign(CENTER, CENTER);
        textSize(16);
        text("Conecta el micro:bit y pulsa el botón", width/2, height/2);
      }
      break;

    case STATES.RUNNING:
      if (!microBitConnected) {
        appState = STATES.WAIT_MICROBIT_CONNECTION;
        break;
      }

      // --------- Dibujo (igual al original, con fader desde micro:bit) ---------
      background(255);

      // faderX: antes mouseX/width; ahora microBitX
      const faderX = tilt01(microBitX); // 0..1

      // tamaño de punto desde yValue (-1024..1024) -> 5..25 px
      const dotSizeBase = map(microBitY, -1024, 1024, 5, 25);
      const dotSize = constrain(dotSizeBase, 3, 30); // por si acaso

      randomSeed(actRandomSeed);
      const angle = radians(360 / count);
      fill(dotColor);

      for (let i = 0; i < count; i++) {
        // posiciones aleatorias
        const randomX = random(0, width);
        const randomY = random(0, height);
        // posiciones en círculo
        const circleX = width / 2 + cos(angle * i) * 300;
        const circleY = height / 2 + sin(angle * i) * 300;
        // interpolación
        const x = lerp(randomX, circleX, faderX);
        const y = lerp(randomY, circleY, faderX);
        ellipse(x, y, dotSize, dotSize);
      }
      break;
  }
}

// ----- teclas: como el original -----
function keyReleased() {
  if (key === 's' || key === 'S') {
    const ts = year() + nf(month(), 2) + nf(day(), 2) + "_" + nf(hour(), 2) + nf(minute(), 2) + nf(second(), 2);
    saveCanvas(ts, 'png');
  }
}

function windowResized() {
  // Si usas 800x800 fijo, no hace falta redimensionar.
  // Si cambias a pantalla completa, usa:
  // resizeCanvas(windowWidth, windowHeight);
}



```

## Video

[Video demostratativo][ https://youtu.be/xqK1A7_DlEw](https://youtu.be/N9Z4aeJUogo)








