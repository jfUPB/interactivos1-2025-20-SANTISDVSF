# Unidad 3

## 🔎 Fase: Set + Seek

### Modelado de la bomba 3.0

Modelo — Bomba 3.0 - Máquina de Estados Finitos (FSM)

La mquina de estados finita funciona con 3 elementos clave: Estados, eventos y acciones

##Estados
1. Config (Configuracion)
  - Aquí la bomba está “inactiva”.
  - Se usa para ajustar el tiempo de cuenta regresiva entre 10 y 60 segundos.
  - El usuario puede subir (A) o bajar (B) el tiempo mostrado en el display.
  - Con el evento S (shake), la bomba pasa al modo armado.

2. Armed
  -La bomba empieza a descontar el tiempo cada 1 segundo.
  -Cada tick del reloj (evento TIMER) resta uno al contador.
  -Si el contador llega a 0, la bomba explota (EXPLODED).

En este estado también se puede intentar desarmarla ingresando la clave A, B, A.

Si la clave es correcta → vuelve a CONFIG.

Si es incorrecta → sigue en ARMED.

3. Exploded
   -Aquí la bomba muestra el ícono de calavera (Image.SKULL).
  -Solo se puede reiniciar con el evento T (tocar el logo).
  -Al reiniciarse vuelve a CONFIG con el contador en 20.

## Eventos

A → Botón A, serial o radio (“incrementar tiempo” en CONFIG, o parte de la clave en ARMED).

B → Botón B, serial o radio (“disminuir tiempo” en CONFIG, o parte de la clave en ARMED).

S → Agitar (shake), serial o radio (arma la bomba desde CONFIG).

T → Logo touch, serial o radio (solo sirve para reiniciar desde EXPLODED).

TIMER → Evento lógico cada 1s en ARMED, que descuenta el contador.

## Variables internas de la máquina

-count: almacena el tiempo de cuenta regresiva (mínimo 10, máximo 60).
-startTime: marca de tiempo usada para controlar los ticks de 1s.
-PASSWORD = ['A','B','A']: la clave correcta para desarmar la bomba.
-key[]: buffer temporal donde se guardan las teclas presionadas en ARMED.
-keyindex: posición dentro de la clave; vuelve a 0 al completar los tres intentos.

## Transiciones y acciones
1. CONFIG → CONFIG (A o B)A:

  - El usuario sube o baja el tiempo.

  - Acción: count = min(count+1,60) o count = max(count-1,10).

  - Se actualiza el display.

2. CONFIG → ARMED (S)
   - El usuario agita el micro:bit.

  -Acción: startTime = now().

  -La bomba pasa al modo armado con el contador en pantalla.

3. ARMED → ARMED (TIMER)
  -Cada segundo, count = count - 1.

  -Si count > 0, sigue mostrando el número.

  -Si count == 0, explota.

4. ARMED → CONFIG (clave correcta A-B-A)

  -El usuario introduce la secuencia correcta.

  -Acción: reinicia contador a 20 y vuelve a CONFIG.

5. ARMED → ARMED (clave incorrecta)
   -Si la secuencia no coincide, keyindex vuelve a 0 y la bomba sigue armada.

6. ARMED → EXPLODED (TIMER y count=0)
   - Si el contador llega a 0, muestra calavera.
   - Estado cambia a EXPLODED.

7. EXPLODED → CONFIG (T)
   -Con el logo touch, se reinicia la bomba.
   -Acción: count = 20; startTime = now(); display.show(count).


### Tabla con los vectores de prueba

# Vectores de prueba – Bomba 3.0

| Estado inicial        | Evento disparador | Acciones                                                   | Estado final |
|------------------------|-------------------|------------------------------------------------------------|--------------|
| CONFIG (count=20)     | A                 | count=21; display.show(21)                                | CONFIG       |
| CONFIG (count=60)     | A                 | count=60 (límite); display.show(60)                       | CONFIG       |
| CONFIG (count=20)     | B                 | count=19; display.show(19)                                | CONFIG       |
| CONFIG (count=10)     | B                 | count=10 (límite); display.show(10)                       | CONFIG       |
| CONFIG (count=25)     | S                 | startTime=now(); display.show(count)                      | ARMED        |
| ARMED (count=25)      | TIMER (1s)        | count=24; display.show(24)                                | ARMED        |
| ARMED (count=1)       | TIMER (1s)        | count=0; display.show(0); display.show(SKULL)             | EXPLODED     |
| ARMED (keyindex=0)    | A                 | key[0]=A; keyindex=1; clear                               | ARMED        |
| ARMED (keyindex=1)    | B                 | key[1]=B; keyindex=2; clear                               | ARMED        |
| ARMED (keyindex=2)    | A (clave correcta)| key[2]=A; keyindex=3; match→count=20; display.show(20)    | CONFIG       |
| ARMED (keyindex=2)    | Secuencia inválida| keyindex=0; clear                                         | ARMED        |
| EXPLODED              | T                 | count=20; startTime=now(); display.show(20); clear        | CONFIG       |
| CONFIG                | A (serial)        | count=min(count+1,60); display.show(count); clear         | CONFIG       |
| CONFIG                | A (radio)         | count=min(count+1,60); display.show(count); clear         | CONFIG       |
| ARMED (count=5)       | TIMER (1s)        | count=4; display.show(4)                                  | ARMED        |
| ARMED (count=4, k=2)  | A (final clave)   | key[2]=A; match→count=20; display.show(20); keyindex=0    | CONFIG       |




