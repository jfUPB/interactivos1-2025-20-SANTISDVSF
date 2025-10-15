
# Evidencias de la unidad 7

### ACTIVIDAD 1

URL Dev Tunnel: https://k1c57t7r-3000.use2.devtunnels.ms/mobile/

## ¿Por qué usar Dev Tunnels y no localhost/IP local?
localhost apunta al mismo dispositivo; desde el celular buscaría un servidor en el propio celular. La IP local solo funciona si ambos están en la misma Wi-Fi y sin bloqueos.

##¿Qué hacen npm install y npm start?
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


