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

Lo siguiente a configurar fue el NAT, para esta parte he configurado una política de NAT que va desde el port2 (usuarios) a port1(red externa):

<img width="1802" height="187" alt="image" src="https://github.com/user-attachments/assets/9ca5e16b-2b8f-42cb-87ba-baddebbbbc1a" />

He configurado dentro de la política de NAT que cuando se salga use la dirección ip de la interfaz saliente:

<img width="722" height="215" alt="image" src="https://github.com/user-attachments/assets/d87e212f-9ebf-411a-8b48-7538ca3eb7f8" />

Política 1: Permitir Usuarios a WEB-Server (443).

Para hacer esta parte creé una política enlazada con port2(usuarios) y port3(servidores), hice unos objetos para hacer referencia a los usuarios de la vlan 10 y al web server para así especificar que admito el acceso al web server desde los usuarios por https:

<img width="1502" height="142" alt="image" src="https://github.com/user-attachments/assets/db2b17ff-99c0-467a-bee6-0881155c543e" />

Política 2: Bloquear Usuarios a DB-Server (3306)

Esta política es bastante parecida a la anterior, los puertos del forti son los mismos y el origen sigue siendo los usuarios, solo cambié el destino a un objeto creado para el DB_Server e hice que negara el acceso en lugar de permitirlo como en el caso pasado, claro que también especifiqué el servicio de mysql para ir como se pide:

<img width="1500" height="186" alt="image" src="https://github.com/user-attachments/assets/3539910d-7777-4323-b1a2-a5c56813c9d2" />


Activar DPI

Para esta parte modifiqué el custom deep-inspection para que este cumpliera con lo que busco y luego pueda ser implementando, coloqué que el DPI bloque certificados bloqueados, como no confiables y también configuré deep scan ssh.

<img width="1227" height="622" alt="image" src="https://github.com/user-attachments/assets/3335c758-1170-4ad2-9b37-315a36eae53b" />

Luego de personalizar mi DPI, coloqué una política que analice todo el tráfico usando este DPI:

<img width="1562" height="312" alt="image" src="https://github.com/user-attachments/assets/b3183f03-5dc5-45b6-92d9-1567c906be25" />

El WEB-Server solo puede comunicarse con el DB-Server en puerto 3306, nada más.

Para lograr esto configuré dos políticas cada una enlazada tanto en entrada como en salida al port3 (ya que maneja a los servidores), en la primera política acepté la comunicación del web_server al db_server específicamente por el servicio de mysql, que vendría siendo el puerto 3306. La segunda política la hice para que negase todo los servicios, pero como la política para permitir el acceso por mysql está primero a esta, tiene prioridad, permitiendo así solo ese acceso:   

<img width="1477" height="130" alt="image" src="https://github.com/user-attachments/assets/d91b3ed7-f2ea-41ee-8b24-74d7bc3e8437" />

Agregar filtrado de aplicaciones para bloquean descargas de .exe desde web.

Para lograr esto primero fui a security profiles, para configurar en file filter para bloquear cualquier archivo .exe que intente entrar por http:

<img width="1232" height="665" alt="image" src="https://github.com/user-attachments/assets/8ab77878-ff03-4f55-9e7e-76e276dc0170" />

También creé un sensor de aplicación para negar lo mismo desde el apartado de application control:

<img width="1107" height="580" alt="image" src="https://github.com/user-attachments/assets/527d2333-14a1-48f7-8aea-69de1127079f" />

Por último, enlace estos bloqueos .exe a algunas de las políticas que creé anteriormente como la de usuarios a web server:

<img width="1400" height="435" alt="image" src="https://github.com/user-attachments/assets/feddb6e4-cded-4077-ba24-c79543848ac4" />

Implementar rate limiting para evitar DoS.

Para esto primero fui a traffic shaping y creé un nuevo traffic shaper que permitiese solo 5mbs para mi ejemplo:

<img width="1187" height="576" alt="image" src="https://github.com/user-attachments/assets/4e0fe35d-9e2f-4c25-aff4-a549ea9a5a2f" />

Luego edité una política de traffic shaping para especificar que entraría por el port1 que lleva a la red externa y saldría por el puerto para usuarios:

<img width="1242" height="852" alt="image" src="https://github.com/user-attachments/assets/bf9dcde9-4a8b-47cf-b97c-3558591ec9f2" />

Para dar protección extra hice una IPv4 DoS Policy la cual iría de usuarios al server_web 

<img width="1217" height="705" alt="image" src="https://github.com/user-attachments/assets/26f7cfa5-e1fa-482a-ac67-e8f30cdca4c8" />

Para estar bloqueando intentos repetidos de tcp_syn_flood, tcp_port_scan y udp_flood

<img width="926" height="581" alt="image" src="https://github.com/user-attachments/assets/541c5c4f-d817-4dad-8a63-142b97c99241" />


############### Running-configs #################

Switch1-A

Switch1-A#show running-config
Building configuration...

Current configuration : 4282 bytes

! Last configuration change at 02:29:34 UTC Fri Sep 25 2026

version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
service compress-config
!
hostname Switch1-A
!
boot-start-marker
boot-end-marker
!
!
enable secret 5 $1$tCOd$cx4VtY4jM88hjc/.1udTu.
!
username admin secret 5 $1$ZiGg$58KfqquCd3JWIBg0.3W/Z/
no aaa new-model
!
!
!
!
!
!
!
!
no ip domain-lookup
ip domain-name laboratorio.local
ip cef
no ipv6 cef
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface GigabitEthernet0/0
 description Enlace Fortigate port2 - Usuarios
 switchport access vlan 10
 switchport mode access
 switchport port-security maximum 2
 switchport port-security
 negotiation auto
!
interface GigabitEthernet0/1
 description Enlace Fortigate port3 - Servidores
 switchport access vlan 20
 switchport trunk allowed vlan 10,20
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport port-security maximum 2
 switchport port-security
 negotiation auto
!
interface GigabitEthernet0/2
 description Web Server
 switchport access vlan 20
 switchport mode access
 switchport port-security maximum 2
 switchport port-security
 negotiation auto
!
interface GigabitEthernet0/3
 description DB Server
 switchport access vlan 20
 switchport mode access
 switchport port-security maximum 2
 switchport port-security
 negotiation auto
!
interface GigabitEthernet1/0
 description PC1 - Usuarios
 switchport access vlan 10
 switchport mode access
 negotiation auto
!
interface GigabitEthernet1/1
 negotiation auto
!
interface GigabitEthernet1/2
 negotiation auto
!
interface GigabitEthernet1/3
 negotiation auto
!
interface GigabitEthernet2/0
 negotiation auto
!
interface GigabitEthernet2/1
 negotiation auto
!
interface GigabitEthernet2/2
 negotiation auto
!
interface GigabitEthernet2/3
 negotiation auto

interface GigabitEthernet3/0
 negotiation auto

interface GigabitEthernet3/1
 negotiation auto

interface GigabitEthernet3/2
 negotiation auto

interface GigabitEthernet3/3
 negotiation auto

interface Vlan90
 ip address 10.78.7.145 255.255.255.248

ip forward-protocol nd

ip http server
ip http secure-server

ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr






control-plane


^C
banner motd ^C Acceso restringido: solo personal autorizado por Airam Brazoban ^C
!
line con 0
line aux 0
line vty 0 4
 login local
 transport input ssh
!
!
end


















