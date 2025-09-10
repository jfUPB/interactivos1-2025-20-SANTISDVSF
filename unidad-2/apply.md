# Unidad 2


## 🛠 Fase: Apply

### ACTIVIDAD 4:
Diseño de la lógica de una bomba temporizada

##DIAGRAMA:

<img width="1215" height="711" alt="ter" src="https://github.com/user-attachments/assets/aa6adfe8-2652-4791-ac61-5684a37ec802" />

### ACTIVIDAD 5:

##Codigo:

````

from microbit import *
import utime
import music

STATE_INIT      = 0
STATE_CONFIG    = 1
STATE_COUNTDOWN = 2
STATE_EXPLODED  = 3

DEFAULT_TIME = 20
MIN_TIME     = 10
MAX_TIME     = 60
TICK_MS      = 1000

current_state = STATE_INIT
set_time   = DEFAULT_TIME
remaining  = 0
start_time = 0

try:
    touch_read = pin_logo.is_touched
except:
    touch_read = lambda: False

touch_prev = False
def touch_was_pressed():
    global touch_prev
    now = touch_read()
    fired = (now and not touch_prev)
    touch_prev = now
    return fired

def show_seconds(sec):
    display.scroll(str(sec), delay=60, wait=False, loop=False)

def play_explosion():
    music.play(['E3:2', 'D3:2', 'C3:2', 'B2:2', 'A2:4'], wait=False)

while True:
    if current_state == STATE_INIT:
        set_time = DEFAULT_TIME
        display.show(str(set_time))
        current_state = STATE_CONFIG

    elif current_state == STATE_CONFIG:
        if button_a.was_pressed():
            set_time = min(set_time + 1, MAX_TIME)
            display.show(str(set_time))
        elif button_b.was_pressed():
            set_time = max(set_time - 1, MIN_TIME)
            display.show(str(set_time))
        elif accelerometer.was_gesture('shake'):
            remaining  = set_time
            start_time = utime.ticks_ms()
            show_seconds(remaining)
            current_state = STATE_COUNTDOWN

    elif current_state == STATE_COUNTDOWN:
        if touch_was_pressed():
            display.show(str(set_time))
            current_state = STATE_CONFIG
        elif utime.ticks_diff(utime.ticks_ms(), start_time) >= TICK_MS:
            start_time = utime.ticks_ms()
            remaining -= 1
            if remaining > 0:
                show_seconds(remaining)
            else:
                display.show(Image.SKULL)
                play_explosion()
                current_state = STATE_EXPLODED

    elif current_state == STATE_EXPLODED:
        if touch_was_pressed():
            set_time = DEFAULT_TIME
            display.show(str(set_time))
            current_state = STATE_CONFIG

    utime.sleep_ms(10)

````

## Vectores de prueba basicos

1. En INIT, sin entrada, se espera que set_time se ponga en 20, se muestre "20" y pase a CONFIG.
2. En CONFIG, si se presiona Botón A y set_time es menor que 60, se espera que aumente en 1, se muestre el nuevo valor y permanezca en CONFIG.
3. En CONFIG, si se presiona Botón B y set_time es mayor que 10, se espera que disminuya en 1, se muestre el nuevo valor y permanezca en CONFIG.
4. En CONFIG, si se detecta un shake (ARMED), se espera que remaining tome el valor de set_time, se actualice start_time con ticks_ms(), se muestre remaining y se pase a COUNTDOWN.
5. En COUNTDOWN, si pasa un segundo y remaining es mayor que 1, se espera que remaining disminuya en 1, se muestre el nuevo valor y permanezca en COUNTDOWN.
6. En COUNTDOWN, si pasa un segundo y remaining es igual a 1, se espera que se muestre la imagen SKULL, se reproduzca la explosión y se pase a EXPLODED.
7. En COUNTDOWN, si se detecta un toque (TOUCH), se espera que se muestre set_time y se pase a CONFIG.
8. En EXPLODED, si se detecta un toque (TOUCH), se espera que set_time vuelva a 20, se muestre "20" y se pase a CONFIG.


