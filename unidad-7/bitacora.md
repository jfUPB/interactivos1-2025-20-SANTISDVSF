
# Evidencias de la unidad 7

## ACTIVIDAD 1

URL Dev Tunnel: https://k1c57t7r-3000.use2.devtunnels.ms/mobile/

### ¿Por qué usar Dev Tunnels y no localhost/IP local?
localhost apunta al mismo dispositivo; desde el celular buscaría un servidor en el propio celular. La IP local solo funciona si ambos están en la misma Wi-Fi y sin bloqueos.

### ¿Qué hacen npm install y npm start?
npm install instala dependencias (Express y Socket.IO).
npm start inicia el servidor en http://localhost:3000.

En la terminal obersve varios mensajes como:
Server is listening on http://localhost:3000, New client connected, Received touch => {x, y, t}, Client disconnected.

<img width="612" height="129" alt="image" src="https://github.com/user-attachments/assets/50234e5a-387e-4246-a1ce-fe84897418f7" />
<img width="642" height="97" alt="image" src="https://github.com/user-attachments/assets/3b54620e-d887-43b1-ba06-95de0e6d02fa" />

En cuanto al comportamiento el círculo del desktop siguió el touch del móvil en tiempo real, tuvo una latencia baja.

<img width="663" height="897" alt="image" src="https://github.com/user-attachments/assets/42bf4360-d4f0-4421-9f2f-c8fd6be604f1" />
<img width="398" height="762" alt="image" src="https://github.com/user-attachments/assets/b5663f4f-4905-4d45-ae44-68393646cccb" />
<img width="540" height="1200" alt="image" src="https://github.com/user-attachments/assets/bd1d20f4-d64e-4dcc-a4c5-6be9f1f917ba" />


## ACTIVIDAD 2

¿Por qué es necesario Dev Tunnels en este escenario y cómo funciona conceptualmente?

R// Cuando corro el server me queda en http://localhost:3000, pero localhost solo sirve en el mismo equipo. Si desde el celular intento abrir localhost, el celu busca un servidor en el propio celular, no en mi PC. La IP local (tipo 192.168.1.X) solo sirve si el PC y el celu están en la misma Wi-Fi y sin bloqueos. Como yo quiero que funcione estando en otra red o con datos, uso Dev Tunnels de VS Code: me da una URL pública segura que apunta a mi server local. 

¿Qué hace touchMoved() y por qué uso threshold?

R// En p5.js, touchMoved() se ejecuta mientras arrastro el dedo sobre la pantalla. Ahí leo las coordenadas mouseX y mouseY y envío esos datos por Socket.IO hacia el servidor.
Uso un threshold (umbral) para no mandar mensajes por micro movimientos del dedo. Solo envío cuando el cambio es “suficientemente grande”. Eso evita saturar la red y la animación se ve más estable en el computador.

Dev Tunnels: (Ventajas y desventajas)
- Funciona aunque el celu y el PC estén en redes distintas (hasta con datos).
  
- Usa HTTPS y atraviesa firewalls/NAT.
  
X La URL es temporal y hay que tener el túnel activo (y sesión iniciada).

IP local (192.168.1.X): (Ventajas y desventajas)
- Simple si estamos en la misma Wi-Fi.

x No sirve por fuera de la red local y puede bloquearse por firewall.

x No es HTTPS por defecto.
 

## ACTIVIDAD 3


