# SegRedes
Aquí recopilo las documentaciones requeridas para las prácticas de Seguridad de Redes impartida por el maestro Jonathan Rondon

En este repositorio de github se estará presentando la documentación pertinente a la Primera Práctica de la materia, la cual tiene como principal objetivos utilizar un fortigate para cumplir con ciertos requisitos
Primero, este es el vídeo de Youtube en el que explico mi práctica: https://youtu.be/-QJ5ac82ocA

La topologia solicita lo siguiente
Topología:
Infraestructura: 
1 Fortigate (Toda configuración y demostración debe ser por GUI)
Ruta por defecto.
NAT.
Política 1: Permitir Usuarios a WEB-Server (443).
Política 2: Bloquear Usuarios a DB-Server (3306).
Activar DPI.
Crear una regla que detecte intentos de SQL Injection en tráfico hacia WEB-Server, los bloquee y coloque al atacante en cuarentena.
Inyectar payloads maliciosos y ven cómo FortiGate los bloquea y los loggea.
El WEB-Server solo puede comunicarse con el DB-Server en puerto 3306, nada más.
Agregar filtrado de aplicaciones para bloquean descargas de .exe desde web.
Implementar rate limiting para evitar DoS.
1 Switch
VLAN.
Seguridad de Basica de Redes.
2 Servidores (/28)
Web Server (HTTPS).
DB Server.
1 Usuarios (/25)
Vlan 10.
DHCP.

Así se ve el diagrama de la topología según los dispositivos solicitados:
<img width="832" height="742" alt="image" src="https://github.com/user-attachments/assets/8a7a1f5a-219a-4eaf-81d6-790a24e9ac72" />

He conectado los dispositivos de la siguiente manera:

<img width="500" height="437" alt="image" src="https://github.com/user-attachments/assets/b82906a9-0add-4bc4-9c8b-3b3f9f5e21f8" />

El port1 del fortigate pertenece a la conexión con el ISP para la conexión, Port2 se encarga de los usuarios de la red y Port3 de los servidores.

Ya que en la práctica se solicita que usemos nuestra matricula para el direccionamiento, he usado los últimos 4 digitos de mi matricula (0787) para dividir
la red así:

<img width="707" height="326" alt="image" src="https://github.com/user-attachments/assets/f5b915d9-6194-4704-aa2d-8ffafb54cab0" />



