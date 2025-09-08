# Unidad 1

## 🔎 Fase: Set + Seek


### Actividad 01: 
¿Qué es un sistema físico interactivo?
Para mí, un sistema físico interactivo es cualquier cosa que responde cuando interactúas con ella. Es como cuando tocas un botón y algo sucede, o cuando un sensor detecta tu presencia y enciende una luz. Lo importante es que haya una respuesta a lo que haces, y eso hace que la experiencia sea más divertida y sorprendente.

¿Cómo podrías aplicar lo que has visto en tu perfil profesional?
En mi vida profesional, creo que entender los sistemas físicos interactivos me puede ayudar a crear cosas que sean más útiles y atractivas para las personas


### Actividad 02:
¿Qué es el diseño y el arte generativos?
El diseño y arte generativos son formas de crear cosas usando sistemas. Es un proceso donde uno decide las reglas y el sistema se encarga de generar las creaciones.

¿Cómo podrías aplicar lo que has visto en tu perfil profesional?
En mi perfil profesional, podría aplicar el diseño y arte generativos para crear soluciones más creativas y originales, por ende mas utiles y exitosas.

### Actividad 03
Herramientas y tecnologías

¿Qué observé?
Cuando presiono los botones A o B en el micro:bit, el círculo en la pantalla cambia de color.
Cuando sacudo el micro:bit, el círculo se pone verde.
Cuando presiono el botón Send Love en la interfaz, el micro:bit muestra un corazón y luego una carita feliz.

### Actividad 04
CODIGO:

```
function setup() {
  createCanvas(400, 400);
  background(240);
  noLoop();
}

function draw() {
  for (let i = 0; i < 30; i++) {
    let x = random(width);
    let y = random(height);
    let w = random(20, 60);
    let h = random(20, 60);
    fill(random(255), random(255), random(255), 150);
    noStroke();
    rect(x, y, w, h);
  }
}
```

<img width="406" height="396" alt="image" src="https://github.com/user-attachments/assets/b98a8b9b-b383-4ae7-9c79-f44d0446e8d8" />



