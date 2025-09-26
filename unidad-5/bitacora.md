
# Evidencias de la unidad 5

### ACTIVIDAD 1


<img width="1919" height="948" alt="image" src="https://github.com/user-attachments/assets/5196b5a2-c0b1-4306-9e60-80576803a872" />


Captura del editor de micro:bit (simulador) con el código en Python para enviar los datos en formato ASCII. Se muestran las variables (xValue, yValue, aState, bState) empaquetadas en un string separado por comas y terminado en \n. Esta evidencia prueba que el programa fue cargado y está listo para ejecutarse en el entorno de simulación. En este caso al no tener el microbit en la casa y no pude ir a la ultima clase a probarlo, todo va a ser simulado. 


<img width="1374" height="940" alt="image" src="https://github.com/user-attachments/assets/b0bbe064-77cf-4fa9-836f-47550888eaec" />


Captura de la consola serial del simulador mostrando las salidas ASCII generadas por el micro:bit. Cada línea contiene los cuatro valores enviados (x,y,a,b), separados por comas y terminados en salto de línea. Esta evidencia confirma la comunicación en texto plano (ASCII) y demuestra que los datos cambian correctamente según las muestras definidas en el código.


<img width="716" height="290" alt="image" src="https://github.com/user-attachments/assets/47d4b779-10ca-4838-985d-704d7a03951c" />


Archivo de texto creado manualmente donde se muestran ejemplos de datos enviados en formato ASCII junto con su representación equivalente en binario (hexadecimal) usando el empaquetado >2h2B. Se observa que cada valor entero (x,y) ocupa 2 bytes en big-endian y cada estado de botón (A,B) ocupa 1 byte.


- En ASCII, los valores son legibles para humanos (ejemplo: 500,524,True,False).
- En Hex binario, los mismos datos ocupan menos bytes (01 f4 02 0c 01 00).


El mismo dato puede representarse de dos formas, ASCII es útil para depuración y binario es más compacto y rápido.


<img width="823" height="400" alt="image" src="https://github.com/user-attachments/assets/92a45385-046c-4c40-a775-dba6c909e2e7" />


El botón es fundamental porque reemplaza la conexión real del micro:bit en este entorno de pruebas. Al presionarlo, el sketch comienza a recibir datos simulados, lo que facilita observar el comportamiento del sistema de comunicación serial (ASCII) sin necesidad de hardware. Esto permite verificar el flujo completo.


<img width="1907" height="901" alt="image" src="https://github.com/user-attachments/assets/9e5d8b44-6cb8-43f6-8559-7c61c474a61d" />


Captura del entorno p5.js Web Editor mostrando el sketch en ejecución, el botón Simulate connect, y la consola integrada con los valores recibidos (X, Y, A, B) y eventos generados. Esta evidencia confirma que el sketch procesa correctamente la información proveniente del micro:bit simulado.


Luego de hacer todo este proceso de evidencia puedo responder las preguntas propuestas:


## ¿Qué datos envía el micro:bit?
El micro:bit envía por UART (115200 bps) cuatro campos en ASCII: xValue, yValue, aState, bState.


## ¿Cómo es la estructura del protocolo ASCII usado?
"<x>,<y>,<a>,<b>\n" — comas separan campos, salto de línea marca fin de paquete.


## Muestra y explica la parte del código de p5.js donde lee los datos del micro:bit y los transforma en coordenadas de la pantalla.
En esta simulación inyecté los valores directamente para generar las pruebas y evidencias sin hardware, que fue lo que ya mostre anteriormente


## ¿Cómo se generan los eventos A pressed y B released?
Si A cambia de false a true → A pressed; si B cambia de true a false → B released




### Actividad 2

<img width="1366" height="822" alt="image" src="https://github.com/user-attachments/assets/9e3e8fd4-9cad-4a18-8609-5d0b4a0f2093" />

<img width="1352" height="815" alt="image" src="https://github.com/user-attachments/assets/3845a7b2-ed1e-49d2-b23e-184cc66c7121" />

Se observa cómo se generan 2 enteros de 16 bits y 2 bytes de estados, listos para enviarse por el puerto serial en formato binario.
El hexdump confirma que el protocolo binario genera un paquete compacto de 6 bytes (LEN: 6). La interpretación es:

01 f4 → 500 en big-endian

02 0c → 524 en big-endian

01 → botón A = presionado

00 → botón B = no presionado

Esto demuestra que el formato >2h2B funciona correctamente y elimina la necesidad de enviar un delimitador como en ASCII.

En hardware real, en modo Texto, estos bytes aparecerían como caracteres raros (“basura”), pero en el simulador los imprimimos en hexadecimal para verificar.


https://youtu.be/tZIWI5an55M

En esta fase modifiqué el programa del micro:bit para que los datos binarios no se envíen continuamente, sino únicamente cuando ocurre un evento (por ejemplo, la detección de un gesto shake o la pulsación del botón A). En este caso es el boton a ya que como no tengo el microbit no puedo hacerlo con shake pero basicamente es lo mismo.

El paquete se envía solo bajo condición de evento.

El tamaño del paquete sigue siendo 6 bytes fijos (LEN: 6).

La estructura de datos binarios (>2h2B) se mantiene constante e independiente de la frecuencia de envío.

<img width="1362" height="912" alt="image" src="https://github.com/user-attachments/assets/19e9b567-28c8-489c-9a60-b1bf4e9b06d3" />

<img width="1357" height="913" alt="image" src="https://github.com/user-attachments/assets/32b3ed9f-bf88-4215-bd45-a1ca3ec9849a" />

En este experimento configuré el micro:bit para que enviara dos representaciones del mismo paquete de datos: primero en formato binario (struct.pack('>2h2B')) y luego en formato ASCII (cadena separada por comas y terminada en \n).

<img width="1367" height="914" alt="image" src="https://github.com/user-attachments/assets/e9f12591-addd-438d-a99f-2fd383a1ccac" />

<img width="1341" height="921" alt="image" src="https://github.com/user-attachments/assets/dc381edd-884e-404c-99e5-46ee6b08c291" />

En esta parte realicé pruebas con valores negativos en el eje X del acelerómetro, por ejemplo -45 y -300. El objetivo fue observar cómo se representan estos números cuando se envían en formato binario con struct.pack('>2h2B').

Esta evidencia confirma que el formato binario maneja correctamente tanto números positivos como negativos. En la transmisión, el receptor debe interpretar los 2 bytes como un entero con signo para recuperar el valor real.

### Actividad 3


<img width="1753" height="800" alt="image" src="https://github.com/user-attachments/assets/49a64da3-ca4a-4f92-9989-be5f9833c0ed" />

En el lector binario sin framing logré decodificar correctamente el primer paquete (x=500, y=524, A=1, B=0).
Sin embargo, al llegar bytes extra o basura, la lectura se desincronizó y los valores aparecieron corruptos (x=39304, y=30465, A=244, B=2).

<img width="1835" height="899" alt="image" src="https://github.com/user-attachments/assets/a31d82f0-47a4-4cd4-911f-8e5ab14678b9" />

En esta prueba modifiqué el protocolo para que cada paquete binario incluyera un byte de longitud (LEN=6) al inicio.
De esta forma el receptor en p5.js pudo identificar el inicio y el fin de cada paquete y leer siempre la cantidad exacta de bytes. El framing eliminó el problema de desincronización observado en la prueba anterior. Aunque existan bytes extra o errores en el flujo, el receptor puede recuperar la alineación al leer el encabezado LEN. Este mecanismo hace al protocolo binario más confiable.

Al leer los datos binarios sin framing, observé que la llegada de bytes basura o desalineados hacía que la interpretación de los paquetes se corrompiera. Esto generaba valores incoherentes en la consola, demostrando un problema de desincronización.

Con la implementación de un protocolo de framing (encabezado con el byte de longitud LEN=6), el receptor pudo detectar el inicio exacto de cada paquete y leer siempre la cantidad correcta de bytes. Esto resolvió el problema y permitió interpretar correctamente los valores de X, Y y los botones.


### Rúbrica – Justificación Personal

1. Profundidad de la indagación → 4.7
Durante las actividades formulé preguntas que no solo buscaban ejecutar los ejemplos, sino comprender el porqué de los protocolos. Analicé causas de errores como la desincronización, comparé el tamaño real entre ASCII y binario y reflexioné sobre en qué casos convendría un protocolo menos eficiente pero más legible.

2. Calidad de la experimentación → 3.5 
Reconozco que mi experimentación fue sólida pero limitada porque no pude probar con el micro:bit físico, lo cual habría enriquecido la práctica. Sin embargo, pude compensar esta limitación con el uso del simulador, provoqué errores de desincronización, probé números negativos para verificar representación en complemento a dos, y diseñé un modo evento para verificar que los paquetes mantenían tamaño constante. Mis experimentos fueron efectivos, pero al no usar hardware real, por eso me pongo esa nota.

3. Análisis y reflexión → 4.5
En mi bitácora no me limité a describir lo que veía. Por ejemplo, no solo anoté los bytes, sino que expliqué su significado campo por campo. Tambien mostré cómo el framing resolvía la desincronización y lo relacioné con la teoría de delimitación de paquetes. Entonces pienso que el analisis y la reflexion fueron mas alla y por eso el 4.5

4. Apropiación y articulación de conceptos → 4.0 
Logré explicar con mis propias palabras conceptos clave como framing, struct.pack, representación binaria con signo y la idea de flujo de bytes asíncrono. También comprendí cómo usar DataView en JavaScript para interpretar datos binarios. Sin embargo reconozco que mi articulación aún puede mejorar, en algunos casos me quedé en explicaciones claras pero básicaspor eso me pongo 4.0

NOTA FINAL: 4.1





