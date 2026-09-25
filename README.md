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


##### SCRIPTS ####

Como la mayoría de esta práctica fue realizada a través de la GUI de fortigate, los scripts utilizados para mis dispositivos se centraron en configuraciones
básicas, como de interfaces y de seguridad, estos fueron mis scripts utilizados por cada dispositivo:

##WEB-SERVER##

sudo nano /etc/netplan/00-installer-config.yaml

network:

  version: 2
  
  ethernets:
  
    ens3:
    
      addresses:
      
        - 10.78.7.130/28
        
      routes:
      
        - to: 0.0.0.0/0
        
          via: 10.78.7.129

      nameservers:
      
        addresses: [8.8.8.8, 8.8.4.4]

sudo netplan apply


##DB-SERVER##

sudo nano /etc/netplan/00-installer-config.yaml

network:

  version: 2
  
  ethernets:
  
    ens3:
    
      addresses:
      
        - 10.78.7.131/28
        
      routes:
      
        - to: 0.0.0.0/0
        
          via: 10.78.7.129
          
      nameservers:
      
        addresses: [8.8.8.8, 8.8.4.4]
        

sudo netplan apply


###Fortigate###

conf sys global

set hostname Forti

end

conf sys int

edit port1

set mode static

set ip 192.168.227.130 255.255.255.0

append allowaccess http https ssh

end

execute backup config flash


###Switch1-A###


configure terminal

hostname Switch1-A

banner motd # Acceso restringido: solo personal autorizado por Airam Brazoban #

no ip domain-lookup

service password-encryption

username admin secret cisco123

enable secret cisco123

line vty 0 4

 transport input ssh
 
 login local
 
exit

ip domain-name laboratorio.local

crypto key generate rsa 

! Crear VLANs

vlan 10

 name Usuarios
 
exit

vlan 20

 name Servidores
 
exit

vlan 90

 name Administración
 
exit

! Asignar puerto para PC1 (Usuarios)
interface Gi1/0

 switchport mode access
 
 switchport access vlan 10
 
 description PC1 - Usuarios
 
switchport port-security

switchport port-security maximum 2

switchport port-security violation shutdown

exit

! Asignar puerto para Web Server (Servidores)

interface Gi0/2

 switchport mode access
 
 switchport access vlan 20
 
 description Web Server
 
switchport port-security

switchport port-security maximum 2

switchport port-security violation shutdown

exit


! Asignar puerto para DB Server (Servidores)

interface Gi0/3

 switchport mode access
 
 switchport access vlan 20
 
 description DB Server
 
switchport port-security

switchport port-security maximum 2

switchport port-security violation shutdown

exit


! Puerto hacia Fortigate port2 (Usuarios)

interface Gi0/0

 switchport mode access
 
 switchport access vlan 10
 
 description Enlace Fortigate port2 - Usuarios
 
switchport port-security

switchport port-security maximum 2

switchport port-security violation shutdown

exit

! Puerto hacia Fortigate port3 (Servidores)

interface GigabitEthernet0/1

 description Enlace Fortigate port3 - Servidores
 
 switchport trunk encapsulation dot1q
 
 switchport mode trunk
 
 switchport trunk allowed vlan 10,20
 
 negotiation auto
 
switchport port-security

switchport port-security maximum 2

switchport port-security violation shutdown

exit

interface vlan 90

 ip address 10.78.7.145 255.255.255.248
 
 no shutdown

copy running-config startup-config

######## POLITICAS DEL FORTIGATE ###########

Una de las primeras cosas que se nos pidió para el fortigate fue configurar una ruta estática, yo lo he hecho de la siguiente manera con la ip de salida de la red, haciendo que todo el tráfico desconocido pueda dirigirse hacía allí:

<img width="1567" height="352" alt="image" src="https://github.com/user-attachments/assets/4353da4d-b270-4176-a298-b41341d280fc" />


