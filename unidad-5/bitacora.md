
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
