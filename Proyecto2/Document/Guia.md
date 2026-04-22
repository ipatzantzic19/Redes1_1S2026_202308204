# 🛠️ GUÍA PASO A PASO — Implementación BanTech GT
**Redes de Computadoras 1 — USAC FIUSAC**
**Carnet:** 202308204 | **XX = 04** | **Y = 4**

---

> **Convención de esta guía:**
> - Los comandos van dentro de bloques de código.
> - Cada sección indica claramente en qué dispositivo se ejecutan.
> - El orden importa: backbone primero, luego sedes, pruebas al final.

---

## FASE 1 — Backbone Core (Núcleo Nacional)

### Paso 1.1 — Colocar dispositivos del backbone en Packet Tracer

1. Abre Cisco Packet Tracer.
2. Arrastra los siguientes dispositivos al área de trabajo:
   - **2 Multilayer Switches** modelo `3650-24PS` → renombrarlos: `Core1` (Switch), `Core2` (Switch)
     - **Rol:** Núcleo principal con EtherChannel LACP (fibra óptica, puertos GigE1/1/1 y GigE1/1/2)
     - **Capacidad:** IP Routing habilitado para OSPF + EIGRP (Core1) / OSPF + RIPv2 (Core2)
   - **1 Router** modelo `4331` o `2911` → renombrar: `Core3` (Router)
     - **Rol:** Conexión a sedes (Oriente) y enlace Serial WAN
     - **Capacidad:** OSPF + enlace serial
   - **2 Routers adicionales** para redistribución: `R-Central1`, `R-Central2` (zona Data Center)
   - **1 Router** por sede de borde: `R-Occidente`, `R-Norte`
   - **2 Multilayer Switches** (modelo 3560 o 3650): `MS1`, `MS2` (para Oriente)
3. Para simular fibra óptica entre Core1 y Core2: usar conexión con 2 cables Ethernet Gigabit en puertos **GigabitEthernet0/0** y **GigabitEthernet0/1** (etiquetados como fibra en la documentación) → Agregar a EtherChannel LACP.
4. Para enlace serial: agregar módulo `WIC-2T` a Core3 (clic en router → pestaña Physical → insertar módulo en slot disponible con router apagado).

---

### Paso 1.2 — Configurar hostnames y contraseñas básicas

Repetir en **CADA dispositivo** del backbone (Core1 [Switch], Core2 [Switch], Core3 [Router], R-Occidente, R-Norte, R-Central1, R-Central2, MS1, MS2):

```
enable
configure terminal
hostname <nombre-del-dispositivo>
enable secret cisco
line console 0
 password cisco
 login
line vty 0 4
 password cisco
 login
service password-encryption
no ip domain-lookup
exit
```

---

### Paso 1.3 — Configurar IP Routing en Core1 y Core2

Como estos son Multilayer Switches, es obligatorio habilitar enrutamiento IP para permitir OSPF y redistribución de rutas.

**En Core1:**
```
configure terminal
ip routing
exit
```

**En Core2:**
```
configure terminal
ip routing
exit
```

---

### Paso 1.4 — Configurar EtherChannel LACP entre Core1 y Core2

Este enlace representa el canal principal de fibra óptica del backbone. Usa 2 interfaces físicas Gigabit agregadas (puertos GigE1/1/1 y GigE1/1/2).

**En Core1 (Multilayer Switch):**
```
configure terminal
interface range GigabitEthernet1/1/1, GigabitEthernet1/1/2
 no switchport
 no shutdown
exit
interface range GigabitEthernet1/1/1, GigabitEthernet1/1/2
 channel-group 1 mode active
exit
interface Port-channel1
 description ETHERCHANNEL-FIBRA-Core1-Core2
 ip address 10.10.0.1 255.255.255.252
 no shutdown
exit
```

**En Core2 (Multilayer Switch):**
```
configure terminal
interface range GigabitEthernet1/1/1, GigabitEthernet1/1/2
 no switchport
 no shutdown
exit
interface range GigabitEthernet1/1/1, GigabitEthernet1/1/2
 channel-group 1 mode active
exit
interface Port-channel1
 description ETHERCHANNEL-FIBRA-Core2-Core1
 ip address 10.10.0.2 255.255.255.252
 no shutdown
exit
```

---

### Paso 1.5 — Configurar enlaces punto a punto restantes del backbone

**Core1 (Switch) ↔ Core3 (Router) - Ethernet Gigabit:**

En Core1 (Switch - puerto GigE1/1/3):
```
interface GigabitEthernet1/1/3
 description ENLACE-Core1-Core3
 no switchport
 ip address 10.10.0.5 255.255.255.252
 no shutdown
exit
```

En Core3 (Router - puerto GigE0/0):
```
interface GigabitEthernet0/0
 description ENLACE-Core3-Core1
 ip address 10.10.0.6 255.255.255.252
 no shutdown
exit
```

**Core2 (Switch) ↔ Core3 (Router) - Redundancia del núcleo:**

En Core2 (Switch - puerto GigE1/1/3):
```
interface GigabitEthernet1/1/3
 description ENLACE-Core2-Core3
 no switchport
 ip address 10.10.0.9 255.255.255.252
 no shutdown
exit
```

En Core3 (Router - puerto GigE0/1):
```
interface GigabitEthernet0/1
 description ENLACE-Core3-Core2
 ip address 10.10.0.10 255.255.255.252
 no shutdown
exit
```

**Core3 ↔ Enlace Serial WAN (simular conectividad remota):**

En Core3 (requiere módulo WIC-2T previamente instalado):
```
interface Serial0/0/0
 description ENLACE-SERIAL-WAN
 ip address 10.10.0.37 255.255.255.252
 clock rate 64000
 no shutdown
exit
```

---

### Paso 1.6 — Configurar enlaces backbone hacia sedes de borde

**Core1 (Switch) ↔ R-Occidente (Router) - Conexión sede Occidente:**

En Core1 (Switch - puerto GigE1/1/4):
```
interface GigabitEthernet1/1/4
 description ENLACE-Core1-R-Occidente
 no switchport
 ip address 10.10.0.13 255.255.255.252
 no shutdown
exit
```

En R-Occidente:
```
interface GigabitEthernet0/0
 description ENLACE-R-Occidente-Core1
 ip address 10.10.0.14 255.255.255.252
 no shutdown
exit
```

**Core2 (Switch) ↔ R-Norte (Router) - Conexión sede Norte:**

En Core2 (Switch - puerto GigE1/1/4):
```
interface GigabitEthernet1/1/4
 description ENLACE-Core2-R-Norte
 no switchport
 ip address 10.10.0.17 255.255.255.252
 no shutdown
exit
```

En R-Norte:
```
interface GigabitEthernet0/0
 description ENLACE-R-Norte-Core2
 ip address 10.10.0.18 255.255.255.252
 no shutdown
exit
```

**Core3 ↔ MS1 y Core3 ↔ MS2 (Oriente, OSPF):**

En Core3:
```
interface GigabitEthernet0/2
 description ENLACE-Core3-MS1-Oriente
 ip address 10.10.0.21 255.255.255.252
 no shutdown
exit
interface GigabitEthernet0/3
 description ENLACE-Core3-MS2-Oriente
 ip address 10.10.0.25 255.255.255.252
 no shutdown
exit
```

En MS1:
```
configure terminal
ip routing
interface GigabitEthernet0/1
 description ENLACE-MS1-Core3
 no switchport
 ip address 10.10.0.22 255.255.255.252
 no shutdown
exit
```

En MS2:
```
configure terminal
ip routing
interface GigabitEthernet0/1
 description ENLACE-MS2-Core3
 no switchport
 ip address 10.10.0.26 255.255.255.252
 no shutdown
exit
```

**Core1 (Switch) ↔ R-Central1 (Router) - Data Center:**

En Core1 (Switch - puerto GigE1/1/5):
```
interface GigabitEthernet1/1/5
 description ENLACE-Core1-R-Central1
 no switchport
 ip address 10.10.0.29 255.255.255.252
 no shutdown
exit
```

En R-Central1 (Router - puerto GigE0/0):
```
interface GigabitEthernet0/0
 description ENLACE-R-Central1-Core1
 ip address 10.10.0.30 255.255.255.252
 no shutdown
exit
```

**Core2 (Switch) ↔ R-Central2 (Router) - Data Center:**

En Core2 (Switch - puerto GigE1/1/5):
```
interface GigabitEthernet1/1/5
 description ENLACE-Core2-R-Central2
 no switchport
 ip address 10.10.0.33 255.255.255.252
 no shutdown
exit
```

En R-Central2 (Router - puerto GigE0/0):
```
interface GigabitEthernet0/0
 description ENLACE-R-Central2-Core2
 ip address 10.10.0.34 255.255.255.252
 no shutdown
exit
```

---

### Paso 1.7 — Configurar OSPF Area 0 en el Backbone

OSPF corre en **Core1 y Core2 (como Multilayer Switches con IP Routing habilitado)**, **Core3 (Router)** y también en **MS1, MS2** (Oriente).

**En Core1:**
```
router ospf 1
 router-id 1.1.1.1
 network 10.10.0.0 0.0.0.3 area 0
 network 10.10.0.4 0.0.0.3 area 0
 network 10.10.0.28 0.0.0.3 area 0
 passive-interface GigabitEthernet1/1/4
exit
```

**En Core2:**
```
router ospf 1
 router-id 2.2.2.2
 network 10.10.0.0 0.0.0.3 area 0
 network 10.10.0.8 0.0.0.3 area 0
 network 10.10.0.32 0.0.0.3 area 0
 passive-interface GigabitEthernet1/1/4
exit
```

**En Core3:**
```
router ospf 1
 router-id 3.3.3.3
 network 10.10.0.4 0.0.0.3 area 0
 network 10.10.0.8 0.0.0.3 area 0
 network 10.10.0.20 0.0.0.3 area 0
 network 10.10.0.24 0.0.0.3 area 0
exit
```

**En MS1:**
```
router ospf 1
 router-id 11.11.11.11
 network 10.10.0.20 0.0.0.3 area 0
 network 192.168.30.0 0.0.0.63 area 0
 network 192.168.30.64 0.0.0.63 area 0
exit
```

**En MS2:**
```
router ospf 1
 router-id 12.12.12.12
 network 10.10.0.24 0.0.0.3 area 0
 network 192.168.30.0 0.0.0.63 area 0
 network 192.168.30.64 0.0.0.63 area 0
exit
```

---

### Paso 1.8 — Configurar EIGRP AS 100 (Occidente)

**En Core1:**
```
router eigrp 100
 network 10.10.0.12 0.0.0.3
 no auto-summary
exit
```

**En R-Occidente:**
```
router eigrp 100
 network 10.10.0.12 0.0.0.3
 network 192.168.10.0 0.0.0.255
 no auto-summary
exit
```

---

### Paso 1.9 — Configurar RIPv2 (Norte)

**En Core2:**
```
router rip
 version 2
 network 10.10.0.16
 no auto-summary
exit
```

**En R-Norte:**
```
router rip
 version 2
 network 10.10.0.16
 network 192.168.20.0
 no auto-summary
exit
```

---

### Paso 1.10 — Configurar Rutas Estáticas (Data Center)

**En R-Central1:**
```
ip route 192.168.40.0 255.255.255.0 10.10.0.29
ip route 0.0.0.0 0.0.0.0 10.10.0.29
```

**En R-Central2:**
```
ip route 192.168.40.0 255.255.255.0 10.10.0.33
ip route 0.0.0.0 0.0.0.0 10.10.0.33
```

**En Core1 (rutas hacia Data Center):**
```
ip route 192.168.40.0 255.255.255.0 10.10.0.30
```

**En Core2 (rutas hacia Data Center - ruta de respaldo):**
```
ip route 192.168.40.0 255.255.255.0 10.10.0.34 10
```

---

### Paso 1.11 — Configurar Redistribución de Rutas

**En Core1 (redistribuye OSPF ↔ EIGRP):**
```
router ospf 1
 redistribute eigrp 100 subnets
exit

router eigrp 100
 redistribute ospf 1 metric 10000 100 255 1 1500
exit
```

**En Core2 (redistribuye OSPF ↔ RIPv2):**
```
router ospf 1
 redistribute rip subnets
exit

router rip
 version 2
 redistribute ospf 1 metric 5
exit
```

**En R-Central1 (redistribuye estáticas → OSPF):**
```
router ospf 1
 router-id 5.5.5.5
 network 10.10.0.28 0.0.0.3 area 0
 redistribute static subnets
exit
```

---

## FASE 2 — Sede Occidente

### Paso 2.1 — Colocar dispositivos en Packet Tracer

Agregar a la topología:
- 1 Switch multicapa o switch normal como `SW-DIST-OCC` (VTP Server)
- `SW-CAJAS` (VTP Client)
- `SW-ASESORES` (VTP Client)
- `SW-GERENCIA-SEG` (VTP Client)
- PCs/hosts: mínimo 7 por VLAN

Conectar:
- R-Occidente (GigabitEthernet0/1) → SW-DIST-OCC (trunk)
- SW-DIST-OCC → SW-CAJAS (trunk)
- SW-DIST-OCC → SW-ASESORES (trunk)
- SW-DIST-OCC → SW-GERENCIA-SEG (trunk)

---

### Paso 2.2 — Configurar VTP en Occidente

**En SW-DIST-OCC (VTP Server):**
```
configure terminal
vtp mode server
vtp domain bantech04
vtp password cisco
exit
```

**En cada switch cliente (SW-CAJAS, SW-ASESORES, SW-GERENCIA-SEG):**
```
configure terminal
vtp mode client
vtp domain bantech04
vtp password cisco
exit
```

---

### Paso 2.3 — Crear VLANs en SW-DIST-OCC (Server)

Solo se crean en el VTP Server; se propagan automáticamente a los clientes.

```
configure terminal
vlan 14
 name Cajas
vlan 24
 name Asesores
vlan 34
 name Gerencia
vlan 44
 name Seguridad
exit
```

**Verificar propagación en un cliente:**
```
show vlan brief
```

---

### Paso 2.4 — Configurar Puertos Trunk

**En SW-DIST-OCC (hacia R-Occidente y hacia cada switch de acceso):**
```
interface GigabitEthernet0/1
 description TRUNK-HACIA-R-Occidente
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 14,24,34,44
 no shutdown
exit

interface GigabitEthernet0/2
 description TRUNK-HACIA-SW-CAJAS
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 14,24,34,44
 no shutdown
exit

interface GigabitEthernet0/3
 description TRUNK-HACIA-SW-ASESORES
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 14,24,34,44
 no shutdown
exit

interface GigabitEthernet0/4
 description TRUNK-HACIA-SW-GERENCIA-SEG
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 14,24,34,44
 no shutdown
exit
```

> **Nota Packet Tracer:** Si el switch no acepta `switchport trunk encapsulation dot1q`, omite esa línea (switches de capa 2 no la requieren, solo los de capa 3).

---

### Paso 2.5 — Configurar Puertos de Acceso en switches de acceso

**En SW-CAJAS:**
```
interface range FastEthernet0/1 - 10
 switchport mode access
 switchport access vlan 14
 no shutdown
exit
```

**En SW-ASESORES:**
```
interface range FastEthernet0/1 - 8
 switchport mode access
 switchport access vlan 24
 no shutdown
exit
```

**En SW-GERENCIA-SEG:**
```
interface range FastEthernet0/1 - 5
 switchport mode access
 switchport access vlan 34
 no shutdown
exit
interface range FastEthernet0/6 - 10
 switchport mode access
 switchport access vlan 44
 no shutdown
exit
```

---

### Paso 2.6 — Configurar Router-on-a-Stick en R-Occidente

```
configure terminal
interface GigabitEthernet0/1
 no ip address
 no shutdown
exit

interface GigabitEthernet0/1.14
 description SUBINTERFACE-VLAN-Cajas
 encapsulation dot1Q 14
 ip address 192.168.10.1 255.255.255.192
exit

interface GigabitEthernet0/1.24
 description SUBINTERFACE-VLAN-Asesores
 encapsulation dot1Q 24
 ip address 192.168.10.65 255.255.255.224
exit

interface GigabitEthernet0/1.34
 description SUBINTERFACE-VLAN-Gerencia
 encapsulation dot1Q 34
 ip address 192.168.10.113 255.255.255.240
exit

interface GigabitEthernet0/1.44
 description SUBINTERFACE-VLAN-Seguridad
 encapsulation dot1Q 44
 ip address 192.168.10.97 255.255.255.240
exit
```

---

### Paso 2.7 — Configurar IPs en los hosts de Occidente

Configura manualmente en cada PC:

| VLAN | IP ejemplo | Máscara | Gateway |
|------|-----------|---------|---------|
| Cajas (14) | 192.168.10.10 | 255.255.255.192 | 192.168.10.1 |
| Asesores (24) | 192.168.10.70 | 255.255.255.224 | 192.168.10.65 |
| Gerencia (34) | 192.168.10.115 | 255.255.255.240 | 192.168.10.113 |
| Seguridad (44) | 192.168.10.100 | 255.255.255.240 | 192.168.10.97 |

---

## FASE 3 — Sede Norte

### Paso 3.1 — Colocar dispositivos

- `SW-CORE-NORTE` (Root Bridge, VTP Server)
- `SW-DIST-N1` (VTP Client)
- `SW-DIST-N2` (VTP Client)
- Switches de acceso: `SW-ACC-ANALISIS`, `SW-ACC-AUDIT`, `SW-ACC-LEGAL`
- PCs: mínimo 7 por VLAN

**Conexiones para crear el anillo/triángulo:**
- SW-CORE-NORTE ↔ SW-DIST-N1 (trunk)
- SW-CORE-NORTE ↔ SW-DIST-N2 (trunk)
- SW-DIST-N1 ↔ SW-DIST-N2 (trunk — este enlace crea el bucle)

---

### Paso 3.2 — Crear VLANs en SW-CORE-NORTE

```
configure terminal
vtp mode server
vtp domain bantech04
vtp password cisco
vlan 54
 name Analisis
vlan 64
 name Auditoria
vlan 74
 name Legal
exit
```

---

### Paso 3.3 — Configurar Rapid PVST+

**En SW-CORE-NORTE (y repetir en SW-DIST-N1, SW-DIST-N2):**
```
configure terminal
spanning-tree mode rapid-pvst
exit
```

**Forzar Root Bridge en SW-CORE-NORTE:**
```
configure terminal
spanning-tree vlan 54 priority 4096
spanning-tree vlan 64 priority 4096
spanning-tree vlan 74 priority 4096
exit
```

**En SW-DIST-N1 y SW-DIST-N2 (prioridad más alta = no serán root):**
```
spanning-tree vlan 54 priority 8192
spanning-tree vlan 64 priority 8192
spanning-tree vlan 74 priority 8192
```

---

### Paso 3.4 — Configurar Trunks en Sede Norte

**En SW-CORE-NORTE:**
```
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 54,64,74
 no shutdown
exit
interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 54,64,74
 no shutdown
exit
interface GigabitEthernet0/3
 description TRUNK-HACIA-R-Norte
 switchport mode trunk
 switchport trunk allowed vlan 54,64,74
 no shutdown
exit
```

**En SW-DIST-N1 (hacia SW-CORE-NORTE y SW-DIST-N2):**
```
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 54,64,74
 no shutdown
exit
interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 54,64,74
 no shutdown
exit
```

---

### Paso 3.5 — Configurar Router-on-a-Stick en R-Norte

```
interface GigabitEthernet0/1
 no ip address
 no shutdown
exit

interface GigabitEthernet0/1.54
 encapsulation dot1Q 54
 ip address 192.168.20.1 255.255.255.192
exit

interface GigabitEthernet0/1.64
 encapsulation dot1Q 64
 ip address 192.168.20.65 255.255.255.224
exit

interface GigabitEthernet0/1.74
 encapsulation dot1Q 74
 ip address 192.168.20.97 255.255.255.240
exit
```

---

### Paso 3.6 — IPs en hosts de Norte

| VLAN | IP ejemplo | Máscara | Gateway |
|------|-----------|---------|---------|
| Análisis (54) | 192.168.20.10 | 255.255.255.192 | 192.168.20.1 |
| Auditoría (64) | 192.168.20.70 | 255.255.255.224 | 192.168.20.65 |
| Legal (74) | 192.168.20.100 | 255.255.255.240 | 192.168.20.97 |

---

## FASE 4 — Sede Oriente

### Paso 4.1 — Configurar VLANs en MS1 (VTP Server de Oriente)

```
configure terminal
ip routing
vtp mode server
vtp domain bantech04
vtp password cisco
vlan 84
 name Boveda
vlan 94
 name Plataforma
exit
```

**En MS2:**
```
configure terminal
ip routing
vtp mode client
vtp domain bantech04
vtp password cisco
exit
```

---

### Paso 4.2 — Configurar interfaces SVI en MS1 y MS2

**En MS1 (interfaces VLAN para HSRP):**
```
interface Vlan84
 ip address 192.168.30.66 255.255.255.192
 no shutdown
exit
interface Vlan94
 ip address 192.168.30.2 255.255.255.192
 no shutdown
exit
```

**En MS2:**
```
interface Vlan84
 ip address 192.168.30.67 255.255.255.192
 no shutdown
exit
interface Vlan94
 ip address 192.168.30.3 255.255.255.192
 no shutdown
exit
```

---

### Paso 4.3 — Configurar HSRP en MS1 y MS2

**En MS1 (Active):**
```
interface Vlan84
 standby 84 ip 192.168.30.65
 standby 84 priority 110
 standby 84 preempt
exit
interface Vlan94
 standby 94 ip 192.168.30.1
 standby 94 priority 110
 standby 94 preempt
exit
```

**En MS2 (Standby):**
```
interface Vlan84
 standby 84 ip 192.168.30.65
 standby 84 priority 100
exit
interface Vlan94
 standby 94 ip 192.168.30.1
 standby 94 priority 100
exit
```

---

### Paso 4.4 — Conectar switches de acceso de Oriente a MS1 y MS2

Conecta `SW-BOVEDA` y `SW-PLATAFORMA` simultáneamente a MS1 y MS2:

**En SW-BOVEDA:**
```
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 84,94
 no shutdown
exit
interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 84,94
 no shutdown
exit
```

**Puertos de acceso en SW-BOVEDA:**
```
interface range FastEthernet0/1 - 10
 switchport mode access
 switchport access vlan 84
 no shutdown
exit
```

**En SW-PLATAFORMA (acceso a VLAN 94):**
```
interface range FastEthernet0/1 - 10
 switchport mode access
 switchport access vlan 94
 no shutdown
exit
```

---

### Paso 4.5 — IPs en hosts de Oriente

| VLAN | IP ejemplo | Máscara | Gateway (Virtual HSRP) |
|------|-----------|---------|------------------------|
| Bóveda (84) | 192.168.30.70 | 255.255.255.192 | **192.168.30.65** |
| Plataforma (94) | 192.168.30.10 | 255.255.255.192 | **192.168.30.1** |

> ⚠️ Los hosts deben usar la **IP virtual** del HSRP como gateway, nunca la IP física de MS1 ni MS2.

---

## FASE 5 — Data Center y Sede Central

### Paso 5.1 — Colocar dispositivos

- `SW-DIST-DC` (switch de distribución)
- `SW-ACC-BD` (acceso a servidores de BD — EtherChannel con SW-DIST-DC)
- `SW-ACC-WEB` (acceso servidores Web)
- `SW-ACC-NOC` (acceso NOC)
- Servidores o PCs representando servidores (mínimo 7 por área)

---

### Paso 5.2 — Crear VLANs en SW-DIST-DC

```
configure terminal
vlan 14
 name Core_BD
vlan 24
 name Web_Apps
vlan 34
 name NOC
exit
```

---

### Paso 5.3 — Configurar EtherChannel LACP entre SW-DIST-DC y SW-ACC-BD

**En SW-DIST-DC:**
```
interface range GigabitEthernet0/1 - 2
 channel-group 1 mode active
 no shutdown
exit
interface Port-channel1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34
 no shutdown
exit
```

**En SW-ACC-BD:**
```
interface range GigabitEthernet0/1 - 2
 channel-group 1 mode active
 no shutdown
exit
interface Port-channel1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34
 no shutdown
exit
```

**Verificar EtherChannel:**
```
show etherchannel summary
```

---

### Paso 5.4 — Configurar puertos de acceso en Data Center

**En SW-ACC-BD:**
```
interface range FastEthernet0/1 - 7
 switchport mode access
 switchport access vlan 14
 no shutdown
exit
```

**En SW-ACC-WEB:**
```
interface range FastEthernet0/1 - 7
 switchport mode access
 switchport access vlan 24
 no shutdown
exit
```

**En SW-ACC-NOC:**
```
interface range FastEthernet0/1 - 7
 switchport mode access
 switchport access vlan 34
 no shutdown
exit
```

---

### Paso 5.5 — Configurar Router-on-a-Stick en R-Central2

```
interface GigabitEthernet0/1
 no ip address
 no shutdown
exit

interface GigabitEthernet0/1.14
 encapsulation dot1Q 14
 ip address 192.168.40.33 255.255.255.240
exit

interface GigabitEthernet0/1.24
 encapsulation dot1Q 24
 ip address 192.168.40.1 255.255.255.224
exit

interface GigabitEthernet0/1.34
 encapsulation dot1Q 34
 ip address 192.168.40.49 255.255.255.240
exit
```

---

### Paso 5.6 — IPs en hosts del Data Center

| VLAN | IP ejemplo | Máscara | Gateway |
|------|-----------|---------|---------|
| Core_BD (14) | 192.168.40.35 | 255.255.255.240 | 192.168.40.33 |
| Web_Apps (24) | 192.168.40.5 | 255.255.255.224 | 192.168.40.1 |
| NOC (34) | 192.168.40.52 | 255.255.255.240 | 192.168.40.49 |

---

## FASE 6 — Pruebas de Conectividad y Failover

### Paso 6.1 — Verificar OSPF

```
show ip ospf neighbor
show ip route ospf
```
Esperado: ver todos los vecinos OSPF en estado FULL.

### Paso 6.2 — Verificar EIGRP

```
show ip eigrp neighbors
show ip route eigrp
```

### Paso 6.3 — Verificar RIPv2

```
show ip rip database
show ip route rip
```

### Paso 6.4 — Verificar Redistribución

En Core1:
```
show ip route
```
Debes ver rutas marcadas con `O E2` (OSPF externas desde EIGRP) y `D EX` en R-Occidente.

En Core2:
```
show ip route
```
Debes ver rutas marcadas con `O E2` (OSPF externas desde RIP) y `R` en R-Norte.

### Paso 6.5 — Ping extremo a extremo

Desde una PC en Occidente (192.168.10.10) hacer ping a una PC en Data Center (192.168.40.35):
```
ping 192.168.40.35
```
Debe responder exitosamente.

### Paso 6.6 — Probar Failover HSRP en Oriente

1. Verificar estado HSRP antes:
```
show standby brief
```
MS1 debe aparecer como `Active`.

2. Apagar MS1 (en Packet Tracer: clic en MS1 → Physical → apagar el switch, o usar `shutdown` en la interfaz hacia los access switches).

3. Desde un host de Oriente, hacer ping continuo:
```
ping 192.168.30.70 repeat 100
```

4. Durante el ping, apagar MS1.

5. Verificar en MS2:
```
show standby brief
```
MS2 debe aparecer ahora como `Active`.

6. Los pings deben continuar con mínimas pérdidas (solo las que ocurren durante la convergencia HSRP ~3 segundos).

### Paso 6.7 — Probar Failover STP en Norte

1. Verificar topología STP antes:
```
show spanning-tree vlan 54
```
Identificar qué puerto está en estado `BLK` (Blocking) en SW-DIST-N1 o SW-DIST-N2.

2. Desconectar el cable entre SW-CORE-NORTE y SW-DIST-N1 (o apagar la interfaz).

3. Observar cómo Rapid PVST+ converge y el puerto bloqueado pasa a `FWD` (Forwarding).

4. Verificar conectividad se mantiene:
```
ping 192.168.20.10
```

### Paso 6.8 — Verificar EtherChannel en Data Center

```
show etherchannel summary
```
Debe mostrar el Port-channel en estado `SU` (State Up).

### Paso 6.9 — Verificar VTP

En un switch cliente de Occidente o Norte:
```
show vtp status
show vlan brief
```
Las VLANs deben aparecer sin haberlas creado manualmente en el cliente.

---

## FASE 7 — Documentación Final

### Paso 7.1 — Capturas requeridas

Tomar capturas de pantalla de:

1. `show ip route` en Core1, Core2, Core3 (redistribución visible)
2. `show ip ospf neighbor` en todos los routers del backbone
3. `show standby brief` en MS1 y MS2 antes y después del failover
4. `show spanning-tree vlan 54` en SW-CORE-NORTE
5. `show etherchannel summary` en SW-DIST-DC
6. `show vtp status` y `show vlan brief` en un cliente VTP
7. Ping exitoso Occidente → Data Center
8. Ping exitoso Norte → Oriente
9. Ping exitoso durante failover HSRP
10. Topología completa visible en Packet Tracer

### Paso 7.2 — Organizar el repositorio GitHub

Estructura recomendada:
```
BanTech-GT-Redes1/
├── README.md                  ← Manual Técnico completo
├── BanTech_GT.pkt             ← Archivo Packet Tracer
├── capturas/
│   ├── backbone/
│   ├── occidente/
│   ├── norte/
│   ├── oriente/
│   ├── datacenter/
│   └── pruebas/
└── docs/
    └── tablas_subnetting.md
```

### Paso 7.3 — Agregar auxiliares como colaboradores

En GitHub: Settings → Collaborators → Add people

Sección A: `JosselineM7`, `PabloR-RC1`, `RoVas97`
Sección N: `CFerSazo7`, `sneeg123`