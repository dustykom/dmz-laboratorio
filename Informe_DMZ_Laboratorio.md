# Informe de Configuración de DMZ con Cisco Packet Tracer

---

## 1. Objetivo del Laboratorio

El objetivo de este laboratorio fue configurar una arquitectura de red segura basada en una Zona Desmilitarizada (DMZ) utilizando un router Cisco ISR simulado en Cisco Packet Tracer. Se implementaron las siguientes tecnologías:

- Segmentación de red en tres zonas: LAN Interna, DMZ y Red Externa (WAN).
- NAT estático para exponer el servidor web de la DMZ hacia Internet.
- ACLs extendidas para controlar y filtrar el tráfico entre zonas.
- Verificación de conectividad y cumplimiento de las políticas de seguridad definidas.

---

## 2. Topología Implementada

La red se compone de tres zonas separadas, conectadas a través de un único router (Router_FW) que actúa como firewall perimetral:

| Zona | Red | Interfaz del Router | Descripción |
|------|-----|---------------------|-------------|
| LAN Interna | 192.168.1.0/24 | GigabitEthernet0/0 | Red de usuarios internos (PC_Internal) |
| DMZ | 192.168.2.0/24 | GigabitEthernet0/1 | Red del servidor web público (Web_DMZ) |
| Red Externa (WAN) | 192.168.3.0/24 | GigabitEthernet0/2 | Red externa / Internet simulado (PC_External) |

**Dispositivos utilizados:** 1 Router Cisco ISR (Router_FW), 1 PC interna (PC_Internal), 1 Servidor web en DMZ (Web_DMZ), 1 PC externa simulando Internet (PC_External).

---

## 3. Plan de Direccionamiento IP

| Dispositivo | IP Asignada | Máscara de Subred | Gateway |
|-------------|-------------|-------------------|---------|
| PC_Internal | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| Web_DMZ (Server) | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| PC_External | 192.168.3.10 | 255.255.255.0 | 192.168.3.1 |
| Router_FW Gi0/0 (LAN) | 192.168.1.1 | 255.255.255.0 | — |
| Router_FW Gi0/1 (DMZ) | 192.168.2.1 | 255.255.255.0 | — |
| Router_FW Gi0/2 (WAN) | 192.168.3.1 | 255.255.255.0 | — |

---

## 4. Configuración Aplicada

### 4.1 Configuración de Interfaces

Se configuraron las tres interfaces del router con sus respectivas IPs y se levantaron con `no shutdown`:

```bash
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
interface GigabitEthernet0/2
 ip address 192.168.3.1 255.255.255.0
 no shutdown
```

### 4.2 Configuración de NAT Estático

Se configuró NAT estático para traducir la IP pública `192.168.3.1` (WAN) hacia la IP privada del servidor web `192.168.2.10` (DMZ):

```bash
interface GigabitEthernet0/1
 ip nat inside
interface GigabitEthernet0/2
 ip nat outside
ip nat inside source static 192.168.2.10 192.168.3.1
```

### 4.3 ACL 1 — Acceso Web desde Internet a DMZ

Permite únicamente tráfico HTTP (TCP puerto 80) hacia la IP pública del servidor. Todo lo demás, incluido ICMP, queda denegado implícitamente:

```bash
ip access-list extended ACL_WAN_TO_DMZ
 permit tcp any host 192.168.3.1 eq 80
 deny ip any any
interface GigabitEthernet0/2
 ip access-group ACL_WAN_TO_DMZ in
```

### 4.4 ACL 2 — Seguridad DMZ hacia LAN (Crítica)

Bloquea cualquier intento de conexión originado desde la DMZ hacia la LAN. Permite respuestas TCP establecidas para que la comunicación LAN → DMZ funcione correctamente:

```bash
ip access-list extended ACL_DMZ_TO_LAN
 permit tcp 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255 established
 deny ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
 deny icmp 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
 permit ip any any
interface GigabitEthernet0/1
 ip access-group ACL_DMZ_TO_LAN in
```

---

## 5. Verificaciones Realizadas

| Prueba | Origen | Destino | Protocolo | Resultado Esperado | Resultado Obtenido |
|--------|--------|---------|-----------|--------------------|--------------------|
| Ping interno al router LAN | PC_Internal | 192.168.1.1 | ICMP | Exitoso | ✅ Exitoso |
| Ping DMZ al router DMZ | Web_DMZ | 192.168.2.1 | ICMP | Exitoso | ✅ Exitoso |
| Acceso web desde PC_External | PC_External | 192.168.3.1 (puerto 80) | TCP | Exitoso | ✅ Exitoso |
| Acceso web desde PC_Internal | PC_Internal | 192.168.2.10 (puerto 80) | TCP | Exitoso | ✅ Exitoso |
| Ping desde DMZ a LAN (bloqueado) | Web_DMZ | 192.168.1.10 | ICMP | Fallo | ✅ Fallo (bloqueado) |
| Ping desde PC_External a router WAN | PC_External | 192.168.3.1 | ICMP | Fallo (ACL) | ✅ Fallo (bloqueado) |

Adicionalmente, se verificó con `show ip interface brief` que todas las interfaces estaban en estado UP/UP, y con `show running-config` que las ACLs y el NAT estaban correctamente aplicados.

---

## 6. Conclusiones y Recomendaciones

Este laboratorio permitió comprender la arquitectura de seguridad perimetral basada en DMZ y su implementación práctica en equipos Cisco. Los principales aprendizajes fueron:

- **Orden de las reglas en ACLs:** Cisco las evalúa secuencialmente y detiene en la primera coincidencia, por lo que las reglas más específicas deben ir primero.
- **Keyword `established`:** Permite un modelo stateless que emula comportamiento stateful, permitiendo respuestas TCP sin abrir la red a conexiones iniciadas desde la DMZ.
- **NAT estático:** Esencial para exponer servicios internos sin revelar la IP real del servidor, añadiendo una capa de abstracción y seguridad.
- **Verificación previa:** Comprobar la conectividad básica entre interfaces antes de aplicar ACLs evita confundir errores de red con bloqueos de ACL.

**Recomendaciones para entornos de producción:**

- Implementar un firewall con inspección stateful en lugar de ACLs estáticas para mayor robustez.
- Agregar logging a las ACLs (`deny ... log`) para monitorear intentos de acceso no autorizado.
- Usar IDS/IPS en la interfaz DMZ para detectar comportamientos anómalos desde el servidor comprometido.

---

## 7. Capturas de Evidencia

- Configuración IP de PC_Internal: IP `192.168.1.10`, máscara `255.255.255.0`, gateway `192.168.1.1`
- Configuración IP de PC_External: IP `192.168.3.10`, máscara `255.255.255.0`, gateway `192.168.3.1`
- `show ip interface brief`: Gi0/0, Gi0/1 y Gi0/2 todas en estado UP/UP
- Configuración de NAT: `ip nat inside source static 192.168.2.10 192.168.3.1`
- Configuración de ACLs: `ACL_WAN_TO_DMZ` en Gi0/2 inbound y `ACL_DMZ_TO_LAN` en Gi0/1 inbound
- Acceso web exitoso desde PC_External al servidor DMZ vía NAT (`http://192.168.3.1`)
- Acceso web exitoso desde PC_Internal al servidor DMZ (`http://192.168.2.10`)
- Ping bloqueado desde PC_External hacia `192.168.3.1` (100% packet loss)
- Ping bloqueado desde Web_DMZ hacia PC_Internal `192.168.1.10` (100% packet loss)
- Pantalla final de Packet Tracer: *"Congratulations on completing this activity!"*
