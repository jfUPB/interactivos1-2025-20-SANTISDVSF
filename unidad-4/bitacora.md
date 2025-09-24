
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

[Enlace a la aplicación modificada](URL) [https://editor.p5js.org/SANTISDVSF/sketches/T87WalDHh](https://editor.p5js.org/SANTISDVSF/sketches/m5LsUyjLe)

Código modificado:

``` js

'use strict';

/**
 * Mod de P_2_1_2_01 para micro:bit
 * Mapear:
 *   1) mouseX  -> acelerómetro X
 *   2) mouseY  -> acelerómetro Y
 *   3) click   -> botón A
 *   4) 'S'     -> botón B
 *
 * micro:bit envía: xValue,yValue,aState,bState\n
 */

let tileCount = 20;
let actRandomSeed = 0;
let circleAlpha = 130;
let circleColor;

// datos del micro:bit
let mbX = 0, mbY = 0, mbA = false, mbB = false;
let prevA = false, prevB = false;
let connected = false;

// serial
let serial;
let readoutEl, statusEl, dotEl;

function setup() {
  createCanvas(600, 600);
  noFill();
  circleColor = color(0, 0, 0, circleAlpha);

  // HUD
  readoutEl = document.getElementById('readout');
  statusEl  = document.getElementById('status');
  dotEl     = document.getElementById('dot');

  let btn = document.getElementById('btnConnect');
  if (btn) btn.onclick = requestSerial;

  // Web Serial
  serial = new p5.WebSerial();
  serial.on('connected', () => console.log('WebSerial OK'));
  serial.on('noport', () => setStatus(false, 'sin puerto'));
  serial.on('portavailable', openPort);
  serial.on('requesterror', () => setStatus(false, 'error'));
  serial.on('close', () => setStatus(false, 'cerrado'));
  serial.on('data', serialEvent);
  serial.getPorts();
}

function draw() {
  translate(width / tileCount / 2, height / tileCount / 2);
  background(255);
  randomSeed(actRandomSeed);

  // usa micro:bit si está conectado, si no usa mouse
  const ctrlX = connected ? map(mbX, -1024, 1024, 0, width)  : mouseX;
  const ctrlY = connected ? map(mbY, -1024, 1024, 0, height) : mouseY;

  stroke(circleColor);
  strokeWeight(ctrlY / 60);

  for (let gridY = 0; gridY < tileCount; gridY++) {
    for (let gridX = 0; gridX < tileCount; gridX++) {
      const posX = (width / tileCount) * gridX;
      const posY = (height / tileCount) * gridY;

      const shiftX = random(-ctrlX, ctrlX) / 20;
      const shiftY = random(-ctrlX, ctrlX) / 20;

      ellipse(posX + shiftX, posY + shiftY, ctrlY / 15, ctrlY / 15);
    }
  }

  // botón A => seed nueva (como click)
  if (mbA && !prevA) actRandomSeed = random(100000);

  // botón B => guardar imagen (como tecla S)
  if (mbB && !prevB) {
    const ts = `${year()}${nf(month(),2)}${nf(day(),2)}_${nf(hour(),2)}${nf(minute(),2)}${nf(second(),2)}`;
    saveCanvas(ts, 'png');
  }

  prevA = mbA;
  prevB = mbB;

  // HUD
  if (readoutEl) readoutEl.textContent = `x:${mbX} y:${mbY} | A:${mbA?1:0} B:${mbB?1:0}`;
}

// compatibilidad con mouse/teclado
function mousePressed() { actRandomSeed = random(100000); }
function keyReleased() {
  if (key === 's' || key === 'S') {
    const ts = `${year()}${nf(month(),2)}${nf(day(),2)}_${nf(hour(),2)}${nf(minute(),2)}${nf(second(),2)}`;
    saveCanvas(ts, 'png');
  }
}

// serial helpers
function requestSerial() { serial.requestPort(); }
function openPort() { serial.open({ baudRate: 115200 }); setStatus(true, 'conectado'); }
function setStatus(ok, label){
  connected = !!ok;
  if (statusEl) statusEl.textContent = label || (ok?'conectado':'desconectado');
  if (dotEl) dotEl.classList.toggle('ok', ok);
}

function serialEvent(){
  const line = serial.readLine();
  if (!line) return;
  const parts = line.trim().split(','); // x,y,A,B
  if (parts.length !== 4) return;

  mbX = parseInt(parts[0], 10);
  mbY = parseInt(parts[1], 10);

  const rawA = parts[2].trim();
  const rawB = parts[3].trim();
  mbA = (rawA === 'True' || rawA === 'true' || rawA === '1');
  mbB = (rawB === 'True' || rawB === 'true' || rawB === '1');
}

```

## Video

[Video demostratativo][ https://youtu.be/xqK1A7_DlEw](https://youtu.be/N9Z4aeJUogo)









