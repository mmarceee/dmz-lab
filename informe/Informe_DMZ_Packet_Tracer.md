## Informe de configuración de DMZ con Cisco Packet Tracer 

_Router ISR con red LAN, DMZ y salida a Internet — NAT estático y ACLs con control de estado_ 

## 1. Objetivo del laboratorio 

Configurar una red segmentada en tres zonas (LAN, DMZ e Internet) utilizando un router Cisco ISR, implementando NAT estático para publicar un servidor web de la DMZ hacia Internet y ACLs para controlar el tráfico entre zonas. El objetivo específico de seguridad era permitir que el servidor web de la DMZ fuera accesible tanto desde la red interna (LAN) como desde el exterior, garantizando al mismo tiempo que ningún host de la DMZ pudiera iniciar conexiones hacia la LAN, limitando así el impacto de un eventual compromiso del servidor publicado. 

## 2. Topología implementada 

Cantidad de redes: 3 (LAN, DMZ y red externa/Internet simulada). 

Dispositivos usados: 

- 1 router Cisco ISR (Router_FW) con 3 interfaces GigabitEthernet activas (Gi0/0, Gi0/1, Gi0/2) 

- Switches de acceso para la LAN y la DMZ (según la topología del ejercicio) 

- PC_Internal — host de la red LAN 

- Server_DMZ — servidor web ubicado en la DMZ 

- PC_External — host que simula un usuario de Internet 

Función de cada zona: 

- LAN (192.168.1.0/24): red interna de confianza, conectada a Gi0/0. Solo puede iniciar conexiones hacia la DMZ, nunca al revés. 

- DMZ (192.168.2.0/24): red intermedia conectada a Gi0/1, aloja el Server_DMZ. Recibe conexiones desde la LAN y desde Internet, pero tiene bloqueada la iniciación de tráfico hacia la LAN. 

- Externa / Internet (192.168.3.0/24): conectada a Gi0/2 (WAN), simula la red pública desde donde PC_External accede al servicio publicado mediante NAT. 

## 3. Plan de direccionamiento IP 

|||||
|---|---|---|---|
|**Dispositivo**|**IP**|**Máscara**|**Gateway**|
|||||
|PC_Internal|192.168.1.10|255.255.255.0|192.168.1.1|
|Server_DMZ|192.168.2.10|255.255.255.0|192.168.2.1|
|PC_External|192.168.3.10|255.255.255.0|192.168.3.1|
|Router_FW Gi0/0 (LAN)|192.168.1.1|255.255.255.0|—|
|Router_FW Gi0/1 (DMZ)|192.168.2.1|255.255.255.0|—|



|Router_FW Gi0/2<br>(Ext/WAN)||||
|---|---|---|---|
||192.168.3.1|255.255.255.0|—|



## 4. Configuración aplicada (resumen) 

## 4.1 Interfaces del router 

```
interface GigabitEthernet0/0
 description LAN
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 no shutdown
!
interface GigabitEthernet0/1
 description DMZ
 ip address 192.168.2.1 255.255.255.0
 ip nat inside
 no shutdown
!
interface GigabitEthernet0/2
 description WAN / Internet
 ip address 192.168.3.1 255.255.255.0
 ip nat outside
 no shutdown
```

## 4.2 NAT estático (publicación del servidor DMZ) 

```
ip nat inside source static 192.168.2.10 192.168.3.1
```

Esta regla traduce la IP privada del servidor (192.168.2.10) hacia la IP pública/externa 192.168.3.1, la cual es la que utiliza PC_External para acceder al sitio web. 

## 4.3 ACL — control DMZ → LAN (con established) 

```
access-list 100 permit tcp 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255 established
access-list 100 deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
access-list 100 permit ip any any
!
interface GigabitEthernet0/1
 ip access-group 100 in
```

La primera línea permite el tráfico de retorno TCP (respuestas web) desde la DMZ hacia la LAN, únicamente cuando corresponde a una conexión ya iniciada por la LAN (flags ACK/RST activos). La segunda línea bloquea cualquier conexión nueva originada en la DMZ hacia la LAN. La tercera evita bloquear tráfico de la DMZ hacia otros destinos (por ejemplo, Internet). 

## 4.4 ACL — publicación web hacia Internet 

```
access-list 101 permit tcp any host 192.168.3.1 eq 80
access-list 101 deny ip any 192.168.1.0 0.0.0.255
access-list 101 permit ip any any
!
interface GigabitEthernet0/2
 ip access-group 101 in
```

Esta ACL, aplicada en la interfaz WAN, permite explícitamente el tráfico HTTP (puerto 80) desde cualquier origen hacia la IP pública del servidor, y bloquea de forma preventiva cualquier tráfico entrante desde Internet directamente hacia la LAN. 

## 5. Verificaciones realizadas 

||||
|---|---|---|
|**Prueba**|**Resultado**|**Observación**|
||||
|Ping desde PC_Internal al router (gateway)|✅OK|Confirma conectividad L3<br>básica en la LAN|
|Ping desde PC_Internal a Server_DMZ|✅OK|Ruta LAN → DMZ correcta|
|Acceso HTTP desde PC_Internal al Server_DMZ|✅OK|Solucionado con ACL 100 +<br>established|
|Acceso HTTP desde PC_External a 192.168.3.1|✅OK|Solucionado con ip nat<br>outside en Gi0/2|
|Intento de conexión nueva DMZ → LAN|✅Bloqueado|Confirma aislamiento de la<br>DMZ (deny ip)|
|show ip nat translations|✅OK|Traducción estática visible:<br>192.168.2.10 ↔ 192.168.3.1|



Comandos de verificación utilizados: 

```
show ip interface brief
show running-config | include nat
show ip nat translations
show ip nat statistics
show access-lists
```

## 6. Conclusiones y recomendaciones 

Este laboratorio permitió aplicar de forma práctica dos conceptos centrales de seguridad perimetral: la segmentación en zonas de confianza mediante DMZ y el filtrado de tráfico con ACLs stateless. El principal aprendizaje fue entender que las ACLs de Cisco IOS no mantienen tablas de estado por sí mismas: la palabra clave 'established' es una aproximación basada en flags TCP (ACK/RST), útil para tráfico de retorno, pero no equivalente a un firewall stateful real y no aplicable directamente a UDP o ICMP. 

El segundo aprendizaje relevante fue la importancia de la correcta asignación de roles NAT (inside/outside) en cada interfaz: un error aparentemente menor —asignar 'ip nat inside' a la interfaz WAN— fue suficiente para que el router no realizara ninguna traducción, incluso teniendo la regla estática correctamente definida. 

Recomendaciones para futuros laboratorios similares: 

- Verificar conectividad básica (ping) en cada segmento antes de aplicar ACLs, para descartar errores de direccionamiento antes de sospechar de las reglas de filtrado. 

- Usar 'show ip nat statistics' como primer paso de diagnóstico NAT, ya que muestra de inmediato qué interfaces están marcadas como inside/outside. 

- Documentar el orden de las líneas dentro de cada ACL, dado que Cisco IOS aplica coincidencia de la primera regla que aplica (top-down) y se detiene ahí. 

- Para servicios distintos a TCP (ICMP, UDP) que deban responder desde la DMZ hacia la LAN, diseñar reglas explícitas basadas en IP/puerto, ya que 'established' no es válido para esos protocolos. 

## 7. Capturas de evidencia 

**==> picture [433 x 70] intentionally omitted <==**

**----- Start of picture text -----**<br>
© PC_Intemal = O x<br>Physical Config Desktop Programming Attributes<br>Web Browser x<br>= => URL http./192.168.2.10] Go Stop<br>**----- End of picture text -----**<br>


Welcome to Cisco Packet Tracer. Opening doors to new opportunities. Mind Wide Open. 

## Quick Links: 

- « WELCOME! 

- « Copyrights * Image page *« Image 

## Top C:\>ping 152.168.1.10 Pinging 192.168.1.10 with 32 bytes of data: 

Reply from 192.168.2.1: Destination host unreachable. Reply from 192.168_2.1: Destination host unreachable. Reply from 192.168_2.1: Destination host unreachable. Reply from 192.168.2.1: Destination host unreachable. Ping statistics for 192.168.1_10: Packets: Sent = 4, Received = 0, Lost = 4 (100% loss), 

