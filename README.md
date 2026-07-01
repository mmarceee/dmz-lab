# DMZ Lab — Cisco Packet Tracer

## Objetivo

Laboratorio de configuración de una **DMZ (zona desmilitarizada)** en un router Cisco ISR utilizando Packet Tracer. El ejercicio implementa:

- Segmentación de red en tres zonas: **LAN**, **DMZ** e **Internet**.
- **NAT estático** para publicar un servidor web ubicado en la DMZ hacia una IP pública.
- **ACLs (Access Control Lists)** para permitir el acceso al servidor tanto desde la red interna como desde el exterior, mientras se bloquea cualquier conexión iniciada desde la DMZ hacia la LAN.

El objetivo de seguridad central era garantizar que, ante un eventual compromiso del servidor publicado, un atacante no pudiera usarlo como punto de pivote hacia la red interna.

## Contenido del repositorio

```
dmz-lab/
├── README.md
├── dmz-lab.pkt                          # Archivo final de Packet Tracer con la topología y configuración
├── informe/
│   └── Informe_DMZ_Laboratorio.md       # Informe completo del laboratorio (objetivo, topología, configuración, verificaciones y conclusiones)
└── evidencias/
    ├── ping_pc_internal.png             # Ping desde PC_Internal al gateway y al servidor DMZ
    ├── web_acceso_interno.png           # Acceso web desde PC_Internal al servidor DMZ
    ├── web_acceso_externo.png           # Acceso web desde PC_External a la IP pública
    ├── bloqueo_dmz_lan.png              # Intento fallido de conexión desde la DMZ hacia la LAN
    ├── nat_translations.png             # Salida de `show ip nat translations`
    └── access_lists.png                 # Salida de `show access-lists`
```

## Resumen técnico

| Elemento | Detalle |
|---|---|
| Red LAN | `192.168.1.0/24` |
| Red DMZ | `192.168.2.0/24` |
| Red externa (Internet simulado) | `192.168.3.0/24` |
| NAT estático | `192.168.2.10` (Server_DMZ) ↔ `192.168.3.1` (IP pública) |
| Control DMZ → LAN | ACL con `established`, permitiendo solo tráfico de retorno |

Detalle completo de la configuración, el diagnóstico de los problemas encontrados y las soluciones aplicadas en [`informe/Informe_DMZ_Laboratorio.md`](informe/Informe_DMZ_Laboratorio.md).

## Cómo abrir el laboratorio

1. Instalar [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer).
2. Abrir el archivo `dmz-lab.pkt`.
3. Revisar la configuración de las interfaces del router (`Router_FW`) y las ACLs aplicadas para reproducir las pruebas descritas en el informe.

## Autor

Ezequiel Marcenal
