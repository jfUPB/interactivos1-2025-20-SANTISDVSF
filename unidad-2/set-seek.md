# Unidad 2

## 🔎 Fase: Set + Seek  

### Actividad 01
***¿Como funciona este ejemplo?***
Este programa en MicroPython para micro:bit implementa el parpadeo independiente de dos LEDs en la matriz, utilizando una máquina de estados para cada píxel.

Se define una clase Pixel, encargada de gestionar el comportamiento de un LED en una posición específica (pixelX, pixelY). Cada instancia alterna entre encendido (valor 9) y apagado (valor 0) según un intervalo de tiempo configurado por el usuario.

La lógica de parpadeo se ejecuta continuamente mediante el método update(), que se llama dentro de un bucle infinito, permitiendo que cada LED parpadee de manera independiente.

***¿Cuales son los estados de este programa?***
Init: Inicializa el tiempo (startTime) y muestra el estado inicial del pixel.
WaitTimeout: Espera a que pase el tiempo para alternar el estado del píxel (encendido o apagado) y actualizar el tiempo de referencia.
De esta manera cada pixel es independiente y parpadea en su propio intervalo de tiempo sin necesidad de de interrupciones ni eventos externos. 

***¿Cuales son los eventos/inputs del sistema?***
El script no depende de un input ya que sus eventos internos son el paso del tiempo medido con  utime.ticks_ms() y utime.ticks_diff().

***¿Cuales son las acciones que realiza el programa?***

Inicializar el tiempo de referencia:
Se guarda el tiempo actual utilizando utime.ticks_ms() cuando el estado es "Init".

Encender o apagar el píxel:
Se utiliza display.set_pixel(x, y, valor) para mostrar el estado del pixel en la matriz LED. El valor cambia entre 0 (apagado) y 9 (encendido).

Alternar el estado del brillo:
Cuando ha pasado el intervalo de tiempo definido, el valor del pixel se alterna de 0 a 9.

Actualizar el tiempo de referencia nuevamente:
Cada vez que se cambia el estado del pixel, se actualiza startTime para medir el siguiente intervalo correctamente.

Mantenerse en el estado "WaitTimeout":
Una vez inicializado, el objeto permanece en este estado ejecutando las acciones anteriores en cada ciclo de actualizacion.

# Actividad 2

  ### *1. Codigo del semaforo*  
  
``` python
from microbit import *
import utime

class Pixel:
    def __init__(self, pixelX, pixelY):
        self.pixelX = pixelX
        self.pixelY = pixelY

    def on(self):
        display.set_pixel(self.pixelX, self.pixelY, 9)

    def off(self):
        display.set_pixel(self.pixelX, self.pixelY, 0)

rojo = Pixel(2, 3)
amarillo = Pixel(2, 2)
verde = Pixel(2, 1)
#yo sigma
while True:
    rojo.on()
    amarillo.off()
    verde.off()
    sleep(3000)

    rojo.off()
    amarillo.on()
    verde.off()
    sleep(1000)

    rojo.off()
    amarillo.off()
    verde.on()
    sleep(3000)
```

### **2. los estados, eventos y acciones del codigo

### Estados
El sistema tiene tres estados principales, uno por cada luz del semáforo:

- *Rojo*: LED rojo encendido
- *Amarillo*: LED amarillo encendido
- *Verde*: LED verde encendido

### Eventos
Los cambios de estado se dan por eventos de tiempo:

- Después de *3 segundos* → pasa de *Rojo* a *Amarillo*
- Después de *1 segundo* → pasa de *Amarillo* a *Verde*
- Después de *3 segundos* → pasa de *Verde* a *Rojo*

### Acciones
Cada estado enciende un LED y apaga los otros:

- *Rojo*:  
  - rojo.on()
  - amarillo.off()
  - verde.off()

- *Amarillo*:  
  - amarillo.on()  
  - rojo.off()  
  - verde.off()

- *Verde*:  
  - verde.on()  
  - rojo.off()  
  - amarillo.off()


# Actividad 3
## 1. ¿Por qué decimos que este programa permite realizar de manera concurrente varias tareas?

Aunque MicroPython en Micro:bit no ejecuta tareas en paralelo real, este programa simula concurrencia porque:

Usa una máquina de estados para cambiar entre estados según eventos (botón A) y tiempo transcurrido.

Mientras espera el tiempo definido por interval, no bloquea el ciclo principal, por lo que puede:

Revisar continuamente si el botón A fue presionado.

Revisar si ya pasó el tiempo para cambiar de estado.

Es decir, parece que monitorea el tiempo y la entrada del botón al mismo tiempo, aunque en realidad se hace de forma secuencial en un while True rápido.

## 2. Estados, eventos y acciones del programa

Estados definidos:

STATE_INIT → estado inicial, muestra cara feliz por primera vez.

STATE_HAPPY → muestra Image.HAPPY.

STATE_SMILE → muestra Image.SMILE.

STATE_SAD → muestra Image.SAD.

Eventos que generan cambios de estado:

Presionar botón A (button_a.was_pressed()).

Cumplir el tiempo del intervalo (utime.ticks_diff(...) > interval).

Acciones que realiza el sistema:

Mostrar una imagen en la matriz LED (display.show(Image.XXX)).

Reiniciar el temporizador (start_time = utime.ticks_ms()).

Ajustar el nuevo intervalo (interval = XXX_INTERVAL).

## 3. Vectores de prueba del programa

### Vector de prueba 1: Cambio automático HAPPY → SMILE
Condiciones iniciales:

Micro:bit recién encendido.

Estado actual: STATE_HAPPY (después de STATE_INIT).

Evento:

Dejar pasar 1.5 segundos sin presionar ningún botón.

Resultado esperado:

Pasa a STATE_SMILE.

Acciones: mostrar Image.SMILE y reiniciar temporizador a 1 s.

Resultado real:

 Image.SMILE se muestra en pantalla.

 Intervalo actualizado a SMILE_INTERVAL.

### Vector de prueba 2: Presionar botón A en estado SMILE
Condiciones iniciales:

Estado actual: STATE_SMILE.

Evento:

Presionar botón A.

Resultado esperado:

Pasa a STATE_HAPPY.

Acciones: muestra Image.HAPPY y reinicia temporizador a 1.5 s.

Resultado real:

 La cara feliz aparece.

 Se reinicia el conteo de tiempo.

### Vector de prueba 3: Cambio automático SMILE → SAD
Condiciones iniciales:

Estado actual: STATE_SMILE.

Evento:

Dejar pasar 1 segundo sin presionar botón.

Resultado esperado:

Pasa a STATE_SAD.

Acciones: mostrar Image.SAD y reiniciar temporizador a 2 s.

Resultado real:

 La cara triste aparece.

 Se reinicia el temporizador correctamente.













### ACTIVIDAD 1

CODIGO:

```
from microbit import *
import utime
class Pixel:
    def __init__(self,pixelX,pixelY,initState,interval):
        self.state = "Init"
        self.startTime = 0
        self.interval = interval
        self.pixelX = pixelX
        self.pixelY = pixelY
        self.pixelState = initState
    def update(self):
        if self.state == "Init":
            self.startTime = utime.ticks_ms()
            self.state = "WaitTimeout"
            display.set_pixel(self.pixelX,self.pixelY,self.pixelState)
        elif self.state == "WaitTimeout":
            if utime.ticks_diff(utime.ticks_ms(),self.startTime) > self.interval:
                self.startTime = utime.ticks_ms()
                if self.pixelState == 9:
                    self.pixelState = 0
                else:
                    self.pixelState = 9
                display.set_pixel(self.pixelX,self.pixelY,self.pixelState)
pixel1 = Pixel(0,0,0,1000)
pixel2 = Pixel(4,4,0,500)
while True:
    pixel1.update()
    pixel2.update()
````
## ¿Como funciona este ejemplo?
R/ Basicamente ese programa crea una clase llamada Pixel, que lo que hace es que representa un Led individual en la pantalla del microbit. Cada Pixel tiene su propia posición, y su propio temporizador y recuerda si está encendido o apagado. Cuando se crean dos de estos píxeles el programa consigue que dos luces distintas parpadeen al mismo tiempo, pero cada una con su propio ritmo, osea que no dependen la una de la otra. Dentro de la clase hay un método llamado update(). Lo que hace es que se encarga de revisar si ya pasó cierto tiempo desde el último cambio, y si si paso cambia el estado del Led.

## ¿Cuáles son los estados en el programa?
R// El programa funciona con dos estados principales para cada LED. 
"Init" es como cuando el LED se está preparando, entonces aquí se configura el tiempo que debe esperar antes de cambiar y se muestra si empieza encendido o apagado. El otro estado es "WaitTimeout", en este estado, el LED simplemente espera a que pase el tiempo que se le puso, entonces una vez que ese tiempo termina, cambia su estado.

## ¿Cuáles son los eventos/inputs en el programa?
R// En este programa los cambios de estado ocurren automáticamente sin necesidad de interacción del usuario. Empezando el programa, el método update() hace que cada LED pase de su estado inicial a uno de espera. Luego cuando transcurre el tiempo (el tiempo que se asigno), el LED cambia de encendido a apagado o al reves y reinicia su temporizador, todo el proceso depende únicamente del paso del tiempo, no de entradas externas.

## ¿Cuáles son las acciones en el programa?
R// Cuando ocurre un cambio dentro del programa, este realiza varias acciones de forma automática. Por ejemplo, primero reinicia el temporizador para empezar a contar el tiempo nuevamente, despues actualiza el estado del LED, indicando si debe esperar o cambiar. Y luego enciende o apaga el LED en su lugar correspondiente dentro de la matriz del microbit. Y por ultimo ajusta el brillo del LED, poniéndolo en 9 si está encendido o en 0 si está apagado para que se vea claramente el parpadeo.

### ACTIVIDAD 2

Codigo que soluciona el problema:

````

from microbit import *
import utime

class Semaforo:
    def __init__(self, x):
        self.x = x
        self.estado = 0  # 0: rojo, 1: amarillo, 2: verde
        self.tiempos = [2000, 1000, 2000]  # ms para cada color
        self.ultimo_cambio = utime.ticks_ms()
        self.dibujar()

    def dibujar(self):
        # Apaga todos
        for y in range(3):
            display.set_pixel(self.x, y, 0)
        # Enciende el color correspondiente
        if self.estado == 0:
            display.set_pixel(self.x, 0, 9)  # Rojo arriba
        elif self.estado == 1:
            display.set_pixel(self.x, 1, 9)  # Amarillo medio
        elif self.estado == 2:
            display.set_pixel(self.x, 2, 9)  # Verde abajo

    def update(self):
        ahora = utime.ticks_ms()
        if utime.ticks_diff(ahora, self.ultimo_cambio) > self.tiempos[self.estado]:
            self.estado = (self.estado + 1) % 3
            self.ultimo_cambio = ahora
            self.dibujar()

semaforo = Semaforo(2)  # Columna central

while True:
    semaforo.update()

````

Explicacion e identificación de estados, eventos y acciones:

R// En este codigo el estado representa en que color esta el semaforo en cada momento, en este codigo se guardo en la variable self.estado dentro de la clase Semaforo. 

````

self.estado = 0  # 0: rojo, 1: amarillo, 2: verde

````

El estado cambia para indicar el color actual del semáforo.

R// Ahora, el evento es lo que provoca que el estado cambie. En el codigo el evento seria el paso del tiempo, Cada vez que pasa el tiempo necesario para cada color , ocurre el evento de cambio de color. Esto se controla en el metodo update.

````

if utime.ticks_diff(ahora, self.ultimo_cambio) > self.tiempos[self.estado]:
    # Aquí ocurre el evento: ha pasado el tiempo necesario

````

Por ultimo, la acción es lo que hace el semáforo cuando ocurre el evento. En este caso la acción es cambiar al siguiente color y mostrarlo en la micro:bi

````

self.estado = (self.estado + 1) % 3  # Cambia al siguiente estado
self.ultimo_cambio = ahora           # Actualiza el tiempo del último cambio
self.dibujar()                       # Dibuja el nuevo color en la micro:bit

````


### ACTIVIDAD 3

## ¿Por qué este programa permite realizar tareas concurrentes?
R// Aunque el microbit solo puede hacer una cosa a la vez, este programa está diseñado para que parezca que hace varias al mismo tiempo. El programa revisa todo el tiempo dos cosas, primero si ya es hora de cambiar la imagen y si el botón A fue presionado. Esto permite que el microbit muestre una secuencia de imágenes y al mismo tiempo esté atento a la interacción del usuario. Si el botón se presiona, el programa puede interrumpir lo que estaba haciendo y responder de inmediato, entonces aunque todo se hace paso a paso, el resultado se siente como si las tareas ocurrieran en paralelo.

## Identifica los estados, eventos y acciones en el programa.
R// Primero tenemos varios estados, el primero es "STATE_INIT" que es el estado inicial del sistema. Aquí se muestra la cara feliz y se inicializa el temporizador. Luego tenemos "STATE_HAPPY", que es donde el microbit muestra una cara feliz. Permanece en este estado durante un intervalo definido, a menos que ocurra un evento (como la pulsación del botón A). En tercer lgar esta "STATE_SMILE" que es cuando el microbit muestra una cara sonriente, se permanece en este estado durante un intervalo definido, a menos que ocurra un evento. Y por ultimo tenemso "STATE_SAD", que es cuando el microbit muestra una cara triste.

Ahora ya viendo los eventos, tenemos la expiracion del tiempo del estado y cuando se pulsa el boton A, son los dos eventos. La expiración del tiempo del estado ocurre cuando el tiempo transcurrido en el estado actual supera el intervalo asignado. y la pulsación del botón A ocurre claramente cuando el usuario presiona el botón A y dependiendo del estado en el que se encuentre el sistema, la pulsación del botón A puede causar una transición diferente.

Y ya por ultimo las acciones que vendrian siendo mostrar una imagen como primera accion, tambien reiniciar el temporizador que es otra accion, actualizar el intervalo es otra accion que es cuando se asigna el valor del intervalo correspondiente al nuevo estado y por ultimo la transición de estado que es cuando se actualiza la variable current_state para reflejar el nuevo estado en el que se encuentra el sistema.

## Describe y aplica al menos 3 vectores de prueba para el programa. Para definir un vector de prueba debes llevar al sistema a un estado, generar los eventos y observar el estado siguiente y las acciones que ocurrirán. Por tanto, un vector de prueba tiene unas condiciones iniciales del sistema, unos resultados esperados y los resultados realmente obtenidos. Si el resultado obtenido es igual al esperado entonces el sistema pasó el vector de prueba, de lo contrario el sistema puede tener un error.

R// VECTOR 1 DE PRUEBA: De STATE_HAPPY a STATE_SAD

El estado inicial es "STATE_HAPPY" - Luego se realiza el evento el cual es presionar el boton A, luego el sistema muestra la imagen SAD, cambia al estado STATE_SAD y reinicia el tiempo, ademas de cambiar al intervalo a SAD_INTERVAL.

VECTOR 2 DE PRUEBA: De STATE_SAD a STATE_SMILE

El estado inicial es "STATE_SAD" - Luego se cumple el intervalo de tiempo, luego el sistema muestra la imagen SMILE, cambia al estado STATE_SMILE y reinicia el tiempo, ademas de cambiar al intervalo a SMILE_INTERVAL.

VECTOR 3 DE PRUEBA: De STATE_SMILE a STATE_HAPPY

El estado inicial es "STATE_SMILE" - Luego se cumple el intervalo de tiempo, luego el sistema muestra la imagen HAPPY, cambia al estado STATE_HAPPY y reinicia el tiempo, ademas de cambiar al intervalo a HAPPY_INTERVAL.


