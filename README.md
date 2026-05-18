# 🛡️ DMZ Lab - Cisco Packet Tracer

Configuración de una red segura con Zona Desmilitarizada (DMZ) usando un router Cisco ISR simulado en Cisco Packet Tracer, aplicando NAT estático y ACLs extendidas para controlar el tráfico entre LAN, DMZ e Internet.

## Objetivo

Implementar una arquitectura de seguridad perimetral con tres zonas de red separadas, garantizando que:
- Solo el tráfico HTTP (puerto 80) desde Internet pueda llegar al servidor web en la DMZ.
- La red DMZ no pueda iniciar conexiones hacia la LAN interna.
- Los usuarios internos puedan acceder al servidor web de la DMZ.

## Contenido del repositorio

```
dmz-lab/
├── informe/
│   └── Informe_DMZ_Laboratorio.md 
├── evidencias/              
│   ├── config_pc_internal.png
│   ├── config_pc_external.png
│   ├── interfaces_router.png
│   ├── config_nat.png
│   ├── config_acls.png
│   ├── web_desde_external.png
│   ├── web_desde_internal.png
│   ├── ping_bloqueado_external.png
│   ├── ping_bloqueado_dmz_lan.png
│   └── actividad_completada.png
└── README.md
```

## Topología implementada

| Zona | Red | Interfaz | Dispositivo |
|------|-----|----------|-------------|
| LAN Interna | 192.168.1.0/24 | GigabitEthernet0/0 | PC_Internal |
| DMZ | 192.168.2.0/24 | GigabitEthernet0/1 | Web_DMZ |
| Red Externa (WAN) | 192.168.3.0/24 | GigabitEthernet0/2 | PC_External |

## Tecnologías utilizadas

- Cisco Packet Tracer
- Router Cisco ISR (IOS 15.1)
- NAT estático
- ACLs extendidas nombradas


### Contributors

Este laboratorio fue desarrollado como parte del curso de redes en [4Geeks Academy Coding Bootcamp](https://4geeksacademy.com/us/coding-bootcamp).
Puedes encontrar más recursos y plantillas en la [página de GitHub de la escuela](https://github.com/4geeksacademy/).
