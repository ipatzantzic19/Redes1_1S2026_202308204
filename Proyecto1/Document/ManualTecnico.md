# 📘 Manual Técnico — Proyecto 1: NetCore Academy
**Curso:** Redes de Computadoras 1  
**Universidad:** San Carlos de Guatemala — FIUSAC  
**Estudiante:** _[Tu nombre completo]_  
**Carnet:** `202308204`  
**Fecha de entrega:** 13 de marzo de 2026  

---

## 🔑 Datos de configuración personalizados

| Parámetro | Valor |
|---|---|
| Carnet | `202308204` |
| Último dígito (X) | `4` |
| Penúltimo dígito (#) | `0` → se usa el anterior = `2` |
| Tipo de carnet | **PAR** |
| VLANs IDs | 14, 24, 34, 44, 54 |
| Red base | 192.168.14.0/24 … 192.168.54.0/24 |
| Dominio VTP | `C2_NetCore` |
| Contraseña VTP | `proyecto12026` |
| STP | **PVST** (`spanning-tree mode pvst`) |
| EtherChannel inter-edificios | **LACP** |
| EtherChannel B1↔B2 | **LACP** |
| EtherChannel intra-edificio | **PAgP** |

---

## 📋 Índice

1. [Topología General](#1-topología-general)
2. [Topología por Edificio](#2-topología-por-edificio)
3. [Medios e Interfaces por Segmento](#3-medios-e-interfaces-por-segmento)
4. [Tabla de Dominios de Colisión](#4-tabla-de-dominios-de-colisión)
5. [Tabla de Dominios de Broadcast](#5-tabla-de-dominios-de-broadcast)
6. [Tabla de VLANs](#6-tabla-de-vlans)
7. [Tabla de Direccionamiento IP](#7-tabla-de-direccionamiento-ip)
8. [Comandos por Dispositivo](#8-comandos-por-dispositivo)
9. [Pruebas de Ping](#9-pruebas-de-ping)
10. [Capturas de Validación](#10-capturas-de-validación)
11. [Presupuesto Estimado](#11-presupuesto-estimado)
12. [Fases de Implementación](#12-fases-de-implementación)

---

## 1. Topología General

> 📸 **[CAPTURA REQUERIDA]** Inserta aquí una captura de pantalla de la topología completa en Cisco Packet Tracer mostrando los 4 edificios interconectados.

```
Edificio A ──(Fibra OM3 LACP EtherChannel)──────── Edificio B
     │                                                   │
  (Fibra OM3                                       (Fibra OM3
   LACP EtherChannel)                            LACP EtherChannel)
     │                                                   │
Edificio C ──(Fibra OM3 LACP EtherChannel)──── Edificio D
```

**Descripción:** El campus NetCore Academy está compuesto por 4 edificios interconectados mediante fibra óptica OM3 (100Base-FX). Los enlaces inter-edificio usan EtherChannel con **LACP** (carnet par). Cada edificio tiene su propio switch de agregación (Switch-PT con módulos de fibra FFE).

---

## 2. Topología por Edificio

### 2.1 Edificio A

> 📸 **[CAPTURA REQUERIDA]** Inserta captura del Edificio A en Packet Tracer.

**Función:** Biblioteca, Docencia, Laboratorio  
**Switch distribución:** SW-A1 (Switch-PT)  
**Switches acceso:** SW-A2, SW-A3 (2960-24TT)  
**Access Point:** AC-A1 → SW-A3, VLAN DOCENTES (24)

| Dispositivo | Tipo | VLAN | Conectado a |
|---|---|---|---|
| SW-A1 | Switch-PT | — | SW-B1, SW-C4 (fibra LACP) |
| SW-A2 | 2960-24TT | — | SW-A1 (GigabitEthernet UTP Cat6) |
| SW-A3 | 2960-24TT | — | SW-A1 (GigabitEthernet UTP Cat6) |
| AC-A1 | AccessPoint | 24 | SW-A3 (access) |
| Admin2 | PC | 14 | SW-A2 |
| Laboratorio2 | PC | 44 | SW-A2 |
| Laboratorio4 | PC | 44 | SW-A2 |
| Docencia1 (Smartphone) | Dispositivo WiFi | 24 | AC-A1 |
| Docencia2 (Laptop) | Laptop WiFi | 24 | AC-A1 |
| Docencia3 (Laptop) | Laptop WiFi | 24 | AC-A1 |

> **EtherChannel intra-edificio A:** SW-A2 ↔ SW-A3 con **PAgP** (2 enlaces GigabitEthernet UTP Cat6)

---

### 2.2 Edificio B

> 📸 **[CAPTURA REQUERIDA]** Inserta captura del Edificio B en Packet Tracer.

**Función:** Biblioteca y Administración  
**Switches agregación:** SW-B1, SW-B2 (Switch-PT)  
**Switches acceso:** SW-B3, SW-B4 (2960-24TT)  
**Legacy L1:** Hub-B1 conectado a SW-B2 (FastEthernet)

| Dispositivo | Tipo | VLAN | Conectado a |
|---|---|---|---|
| SW-B1 | Switch-PT | — | SW-A1 (fibra LACP), SW-B2 (fibra LACP) |
| SW-B2 | Switch-PT | — | SW-B1 (fibra LACP), SW-D5 (fibra LACP) |
| SW-B3 | 2960-24TT | — | SW-B1 (GigabitEthernet) |
| SW-B4 | 2960-24TT | — | SW-B1 (GigabitEthernet) |
| Hub-B1 | Hub-PT | — | SW-B2 (FastEthernet) |
| Biblioteca1 | PC | 34 | SW-B3 |
| Docencia6 (Laptop) | Laptop | 24 | SW-B3 |
| Admin1 (Laptop) | Laptop | 14 | SW-B4 |
| Biblioteca2 (Laptop) | Laptop | 34 | SW-B4 |
| Biblioteca3 | PC | 34 | Hub-B1 |
| Biblioteca4 | PC | 34 | Hub-B1 |
| Biblioteca5 | PC | 34 | Hub-B1 |

> **EtherChannel B1↔B2:** LACP (2 enlaces fibra OM3)

---

### 2.3 Edificio C

> 📸 **[CAPTURA REQUERIDA]** Inserta captura del Edificio C en Packet Tracer.

**Función:** Docencia y Biblioteca ligera  
**Punto campus:** SW-C4 (Switch-PT)  
**Núcleo interno:** Hub-C1 (capa 1 — un gran dominio de colisión)  
**Switches acceso:** SW-C1, SW-C2, SW-C3 (2960-24TT)

| Dispositivo | Tipo | VLAN | Conectado a |
|---|---|---|---|
| SW-C4 | Switch-PT | — | SW-A1, SW-D1 (fibra LACP) |
| Hub-C1 | Hub-PT | — | SW-C4, SW-C1, SW-C2, SW-C3 (FastEthernet) |
| SW-C1 | 2960-24TT | — | Hub-C1 (FastEthernet UTP Cat6) |
| SW-C2 | 2960-24TT | — | Hub-C1 (FastEthernet UTP Cat6) |
| SW-C3 | 2960-24TT | — | Hub-C1 (FastEthernet UTP Cat6) |
| Docencia9 | PC | 24 | SW-C1 |
| Docencia7 (Laptop) | Laptop | 24 | SW-C2 |
| Docencia8 | PC | 24 | SW-C2 |
| Biblioteca6 | PC | 34 | SW-C3 |
| Admin3 | PC | 14 | SW-C3 |

---

### 2.4 Edificio D

> 📸 **[CAPTURA REQUERIDA]** Inserta captura del Edificio D en Packet Tracer.

**Función:** Administración, Laboratorio, Visitantes  
**IDF:** SW-D1 (Switch-PT)  
**Salida hacia B:** SW-D5 (Switch-PT)  
**VTP Transparente:** SW-E1 con VLAN 54 local  
**Repeater:** Repeater-D1 entre SW-D2 y SW-D3

| Dispositivo | Tipo | VTP | VLAN | Conectado a |
|---|---|---|---|---|
| SW-D1 | Switch-PT | Cliente | — | SW-C4 (fibra LACP), SW-D5 (fibra trunk) |
| SW-D5 | Switch-PT | Cliente | — | SW-B2 (fibra LACP), SW-D1 (fibra trunk) |
| SW-D2 | 2960-24TT | Cliente | — | SW-D1, SW-D5 (GigabitEthernet) |
| SW-D3 | 2960-24TT | Cliente | — | Repeater-D1 (FastEthernet) |
| SW-D4 | 2960-24TT | Cliente | — | SW-D2 (FastEthernet) |
| SW-E1 | 2960-24TT | **Transparente** | 54 local | SW-D1, SW-D2 (FastEthernet) |
| Repeater-D1 | Repeater-PT | — | — | SW-D2 ↔ SW-D3 |
| AC-D1 | AccessPoint | — | 54 | SW-E1 (GigabitEthernet) |
| Visitantes1 | PC | — | 54 | SW-E1 |
| Visitantes2 | PC | — | 54 | SW-E1 |
| Visitantes3 (Laptop) | Laptop WiFi | — | 54 | AC-D1 |
| Admin4 | PC | — | 14 | SW-D3 |
| Admin5 | PC | — | 14 | SW-D3 |
| Docencia10 (Server) | Server | — | 24 | SW-D3 |
| Laboratorio3 | PC | — | 44 | SW-D4 |
| Biblioteca7 | PC | — | 34 | SW-D4 |

---

## 3. Medios e Interfaces por Segmento

| Segmento | Dispositivos | Medio | Interfaz | Justificación |
|---|---|---|---|---|
| Inter-edificios | SW-A1↔SW-B1, SW-A1↔SW-C4, SW-C4↔SW-D1, SW-D5↔SW-B2, SW-B1↔SW-B2 | Fibra óptica OM3 | FastEthernet módulo FFE (x2 por EtherChannel) | Mayor alcance, inmunidad EMI, agregación LACP |
| D1↔D5 (redundancia interna) | SW-D1 ↔ SW-D5 | Fibra óptica OM3 | FastEthernet módulo FFE (x1 trunk 802.1Q) | Redundancia de capa 2 dentro del Edificio D |
| Distribución edificio | SW-A1↔SW-A2/A3, SW-B1↔SW-B3/B4, SW-D1↔SW-D2, etc. | Cobre UTP Cat6 | GigabitEthernet | Soporta tráfico agregado de múltiples VLANs |
| Acceso a usuarios | PCs/Laptops → switches | Cobre UTP Cat5e | FastEthernet | Suficiente para dispositivos finales |
| Segmentos Legacy | SW-B2↔Hub-B1, Hub-C1↔switches C, SW-D2↔Repeater-D1↔SW-D3 | Cobre UTP Cat5e | FastEthernet | Capa 1: dominio de colisión compartido |
| AP a Switch | AC-A1↔SW-A3, AC-D1↔SW-E1 | Cobre UTP Cat6 | GigabitEthernet | Bridge L2, integración clientes WiFi a VLAN |

---

## 4. Tabla de Dominios de Colisión

> **Dominio de colisión:** segmento donde colisiones son posibles. Cada puerto de switch = 1 dominio. Hubs y repetidores extienden/comparten el dominio.

### Edificio A

| ID | Dispositivos | Tipo |
|---|---|---|
| DC-A-1 | SW-A1 ↔ SW-A2 (enlace físico 1 del Port-Channel) | Punto a punto |
| DC-A-2 | SW-A1 ↔ SW-A2 (enlace físico 2 del Port-Channel) | Punto a punto |
| DC-A-3 | SW-A1 ↔ SW-A3 (enlace físico 1) | Punto a punto |
| DC-A-4 | SW-A1 ↔ SW-A3 (enlace físico 2) | Punto a punto |
| DC-A-5 | SW-A2 ↔ SW-A3 PAgP enlace 1 | Punto a punto |
| DC-A-6 | SW-A2 ↔ SW-A3 PAgP enlace 2 | Punto a punto |
| DC-A-7…n | Cada PC/Laptop ↔ su puerto de switch | Punto a punto |

**Total Edificio A: ~12 dominios individuales**

---

### Edificio B

| ID | Dispositivos | Tipo |
|---|---|---|
| **DC-B-HUB** | **Hub-B1 + SW-B2 puerto + Biblioteca3 + Biblioteca4 + Biblioteca5** | **Compartido (Hub)** |
| DC-B-1…n | Cada enlace punto a punto entre switches y PCs individuales | Punto a punto |

**Total Edificio B: ~11 dominios individuales + 1 dominio compartido por Hub-B1**  
⚠️ Biblioteca3, 4 y 5 comparten el mismo dominio de colisión. Una transmisión simultánea genera colisión para todos.

---

### Edificio C

| ID | Dispositivos | Tipo |
|---|---|---|
| **DC-C-HUB (GRANDE)** | **Hub-C1 + SW-C4 + SW-C1 + SW-C2 + SW-C3** — todos comparten un único dominio | **Compartido (Hub)** |
| DC-C-1…n | Cada PC ↔ su puerto en SW-C1/C2/C3 | Punto a punto |

**Total Edificio C: ~5 dominios individuales + 1 gran dominio del Hub-C1**  
⚠️ Todo el backbone del Edificio C es un único dominio de colisión. Cualquier colisión se propaga a todos los switches C.

---

### Edificio D

| ID | Dispositivos | Tipo |
|---|---|---|
| **DC-D-REP** | **SW-D2 ↔ Repeater-D1 ↔ SW-D3** | **Extendido (Repeater)** |
| DC-D-1…n | Cada enlace punto a punto restante y PC ↔ switch | Punto a punto |

**Total Edificio D: ~14 dominios individuales + 1 extendido por Repeater-D1**

---

## 5. Tabla de Dominios de Broadcast

> **Dominio de broadcast:** todos los dispositivos que reciben un frame broadcast. En capa 2: **1 VLAN = 1 dominio de broadcast** extendido por todos los trunks activos.

| VLAN ID | Nombre | Alcance del dominio de broadcast |
|---|---|---|
| **14** | ADMIN | Admin2 (Ed.A), Admin1 (Ed.B), Admin3 (Ed.C), Admin4 y Admin5 (Ed.D) — todos los switches con VLAN 14 vía VTP |
| **24** | DOCENTES | Docencia1/2/3 WiFi (Ed.A), Docencia6 (Ed.B), Docencia7/8/9 (Ed.C), Docencia10 Server (Ed.D) |
| **34** | BIBLIOTECA | Biblioteca1/2 (Ed.B), Biblioteca3/4/5 vía Hub (Ed.B), Biblioteca6 (Ed.C), Biblioteca7 (Ed.D) |
| **44** | LABORATORIO | Laboratorio2/4 (Ed.A), Laboratorio3 (Ed.D) |
| **54** | VISITANTE | Visitantes1/2 y Visitantes3 WiFi — **contenido solo en SW-E1** |

> ⚠️ **VLAN 54 VISITANTE:** SW-E1 es VTP **Transparent** — la VLAN 54 es local y los broadcasts **NO se propagan** al resto del campus. Dominio de broadcast completamente aislado.

---

## 6. Tabla de VLANs

| VLAN ID | Nombre | Red Asignada | Área |
|---|---|---|---|
| **14** | ADMIN | 192.168.14.0/24 | Administración |
| **24** | DOCENTES | 192.168.24.0/24 | Docencia |
| **34** | BIBLIOTECA | 192.168.34.0/24 | Biblioteca |
| **44** | LABORATORIO | 192.168.44.0/24 | Laboratorio de Redes |
| **54** | VISITANTE | 192.168.54.0/24 | Visitantes |

---

## 7. Tabla de Direccionamiento IP

> Máscara de subred: `255.255.255.0` en todos los dispositivos.

### Edificio A

| Dispositivo | VLAN | Dirección IP | Gateway |
|---|---|---|---|
| Admin2 | 14 – ADMIN | 192.168.14.2 | 192.168.14.1 |
| Laboratorio2 | 44 – LABORATORIO | 192.168.44.2 | 192.168.44.1 |
| Laboratorio4 | 44 – LABORATORIO | 192.168.44.4 | 192.168.44.1 |
| Docencia1 (Smartphone) | 24 – DOCENTES | 192.168.24.11 | 192.168.24.1 |
| Docencia2 (Laptop) | 24 – DOCENTES | 192.168.24.12 | 192.168.24.1 |
| Docencia3 (Laptop) | 24 – DOCENTES | 192.168.24.13 | 192.168.24.1 |

### Edificio B

| Dispositivo | VLAN | Dirección IP | Gateway |
|---|---|---|---|
| Biblioteca1 | 34 – BIBLIOTECA | 192.168.34.10 | 192.168.34.1 |
| Docencia6 (Laptop) | 24 – DOCENTES | 192.168.24.16 | 192.168.24.1 |
| Admin1 (Laptop) | 14 – ADMIN | 192.168.14.11 | 192.168.14.1 |
| Biblioteca2 (Laptop) | 34 – BIBLIOTECA | 192.168.34.20 | 192.168.34.1 |
| Biblioteca3 (Hub) | 34 – BIBLIOTECA | 192.168.34.30 | 192.168.34.1 |
| Biblioteca4 (Hub) | 34 – BIBLIOTECA | 192.168.34.40 | 192.168.34.1 |
| Biblioteca5 (Hub) | 34 – BIBLIOTECA | 192.168.34.50 | 192.168.34.1 |

### Edificio C

| Dispositivo | VLAN | Dirección IP | Gateway |
|---|---|---|---|
| Docencia9 | 24 – DOCENTES | 192.168.24.19 | 192.168.24.1 |
| Docencia7 (Laptop) | 24 – DOCENTES | 192.168.24.17 | 192.168.24.1 |
| Docencia8 | 24 – DOCENTES | 192.168.24.18 | 192.168.24.1 |
| Biblioteca6 | 34 – BIBLIOTECA | 192.168.34.60 | 192.168.34.1 |
| Admin3 | 14 – ADMIN | 192.168.14.13 | 192.168.14.1 |

### Edificio D

| Dispositivo | VLAN | Dirección IP | Gateway |
|---|---|---|---|
| Visitantes1 | 54 – VISITANTE | 192.168.54.1 | 192.168.54.1 |
| Visitantes2 | 54 – VISITANTE | 192.168.54.2 | 192.168.54.1 |
| Visitantes3 | 54 – VISITANTE | 192.168.54.3 | 192.168.54.1 |
| Admin4 | 14 – ADMIN | 192.168.14.14 | 192.168.14.1 |
| Admin5 | 14 – ADMIN | 192.168.14.15 | 192.168.14.1 |
| Docencia10 (Server) | 24 – DOCENTES | 192.168.24.110 | 192.168.24.1 |
| Laboratorio3 | 44 – LABORATORIO | 192.168.44.3 | 192.168.44.1 |
| Biblioteca7 | 34 – BIBLIOTECA | 192.168.34.70 | 192.168.34.1 |

---

## 8. Comandos por Dispositivo

> Configuración base — aplicar en **todos los switches** antes de cualquier otra configuración:

```cisco
enable
configure terminal
hostname [NOMBRE_SWITCH]
no ip domain-lookup
enable secret cisco
line console 0
 password cisco
 login
line vty 0 4
 password cisco
 login
service password-encryption
```

---

### 8.1 SW-A1 — VTP SERVER | ROOT BRIDGE | PVST

```cisco
enable
configure terminal
hostname SW-A1

! ── VTP SERVER ───────────────────────────────────────────
vtp mode server
vtp domain C2_NetCore
vtp version 2

! ── CREAR VLANs (solo en VTP Server) ────────────────────
vlan 14
 name ADMIN
vlan 24
 name DOCENTES
vlan 34
 name BIBLIOTECA
vlan 44
 name LABORATORIO
vlan 54
 name VISITANTE

! ── STP PVST — ROOT BRIDGE prioridad 4096 ───────────────
spanning-tree mode pvst
spanning-tree vlan 14 priority 4096
spanning-tree vlan 24 priority 4096
spanning-tree vlan 34 priority 4096
spanning-tree vlan 44 priority 4096
spanning-tree vlan 54 priority 4096

! ── EtherChannel LACP hacia SW-B1 (fibra OM3 inter-edificios)
interface range FastEthernet0/1 - 2
 channel-protocol lacp
 channel-group 1 mode active
 no shutdown
interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! ── EtherChannel LACP hacia SW-C4 (fibra OM3 inter-edificios)
interface range FastEthernet0/3 - 4
 channel-protocol lacp
 channel-group 2 mode active
 no shutdown
interface port-channel 2
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! ── Trunk GigabitEthernet hacia SW-A2 (UTP Cat6) ────────
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! ── Trunk GigabitEthernet hacia SW-A3 (UTP Cat6) ────────
interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! ── Banner MOTD ──────────────────────────────────────────
banner motd # Bienvenido a Edificio A - NETCORE_202308204 #

! ── Contraseña VTP: configurar AL FINAL ─────────────────
! vtp password proyecto12026

end
write memory
```

---

### 8.2 SW-A2 — VTP CLIENT | Edificio A acceso

```cisco
enable
configure terminal
hostname SW-A2
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

! Trunk hacia SW-A1
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! EtherChannel PAgP con SW-A3 (intra-edificio)
interface range FastEthernet0/1 - 2
 channel-protocol pagp
 channel-group 3 mode desirable
 no shutdown
interface port-channel 3
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! Acceso Admin2 → VLAN 14
interface FastEthernet0/10
 switchport mode access
 switchport access vlan 14
 no shutdown

! Acceso Laboratorio2 → VLAN 44
interface FastEthernet0/11
 switchport mode access
 switchport access vlan 44
 no shutdown

! Acceso Laboratorio4 → VLAN 44
interface FastEthernet0/12
 switchport mode access
 switchport access vlan 44
 no shutdown

end
write memory
```

---

### 8.3 SW-A3 — VTP CLIENT | Edificio A acceso + AP

```cisco
enable
configure terminal
hostname SW-A3
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

! Trunk hacia SW-A1
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! EtherChannel PAgP con SW-A2 (intra-edificio)
interface range FastEthernet0/1 - 2
 channel-protocol pagp
 channel-group 3 mode desirable
 no shutdown
interface port-channel 3
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! AC-A1 → VLAN 24 DOCENTES
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 24
 no shutdown

end
write memory
```

---

### 8.4 SW-B1 — VTP CLIENT | Edificio B agregación

```cisco
enable
configure terminal
hostname SW-B1
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

! EtherChannel LACP hacia SW-A1
interface range FastEthernet0/1 - 2
 channel-protocol lacp
 channel-group 1 mode active
 no shutdown
interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! EtherChannel LACP hacia SW-B2
interface range FastEthernet0/3 - 4
 channel-protocol lacp
 channel-group 4 mode active
 no shutdown
interface port-channel 4
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! Trunk hacia SW-B3 (GigabitEthernet)
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Trunk hacia SW-B4 (GigabitEthernet)
interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

banner motd # Bienvenido a Edificio B - NETCORE_202308204 #
end
write memory
```

---

### 8.5 SW-B2 — VTP CLIENT | Edificio B agregación redundante

```cisco
enable
configure terminal
hostname SW-B2
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

! EtherChannel LACP hacia SW-B1
interface range FastEthernet0/1 - 2
 channel-protocol lacp
 channel-group 4 mode active
 no shutdown
interface port-channel 4
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! EtherChannel LACP hacia SW-D5
interface range FastEthernet0/3 - 4
 channel-protocol lacp
 channel-group 5 mode active
 no shutdown
interface port-channel 5
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! Hub-B1 Legacy — acceso VLAN BIBLIOTECA
interface FastEthernet0/10
 switchport mode access
 switchport access vlan 34
 no shutdown

end
write memory
```

---

### 8.6 SW-B3 — VTP CLIENT | Edificio B acceso

```cisco
enable
configure terminal
hostname SW-B3
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Biblioteca1 → VLAN 34
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 34
 no shutdown

! Docencia6 → VLAN 24
interface FastEthernet0/6
 switchport mode access
 switchport access vlan 24
 no shutdown

end
write memory
```

---

### 8.7 SW-B4 — VTP CLIENT | Edificio B acceso

```cisco
enable
configure terminal
hostname SW-B4
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Admin1 → VLAN 14
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 14
 no shutdown

! Biblioteca2 → VLAN 34
interface FastEthernet0/6
 switchport mode access
 switchport access vlan 34
 no shutdown

! Biblioteca4 → VLAN 34
interface FastEthernet0/7
 switchport mode access
 switchport access vlan 34
 no shutdown

end
write memory
```

---

### 8.8 SW-C4 — VTP CLIENT | Edificio C punto campus

```cisco
enable
configure terminal
hostname SW-C4
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

! EtherChannel LACP hacia SW-A1
interface range FastEthernet0/1 - 2
 channel-protocol lacp
 channel-group 2 mode active
 no shutdown
interface port-channel 2
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! EtherChannel LACP hacia SW-D1
interface range FastEthernet0/3 - 4
 channel-protocol lacp
 channel-group 6 mode active
 no shutdown
interface port-channel 6
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! Trunk hacia Hub-C1 (GigabitEthernet)
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

banner motd # Bienvenido a Edificio C - NETCORE_202308204 #
end
write memory
```

---

### 8.9 SW-C1 / SW-C2 / SW-C3 — VTP CLIENT | Edificio C acceso

```cisco
! ═══ SW-C1 ═══
enable
configure terminal
hostname SW-C1
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Docencia9 → VLAN 24
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 24
 no shutdown
end
write memory

! ═══ SW-C2 ═══
enable
configure terminal
hostname SW-C2
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Docencia7 → VLAN 24
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 24
 no shutdown

! Docencia8 → VLAN 24
interface FastEthernet0/6
 switchport mode access
 switchport access vlan 24
 no shutdown
end
write memory

! ═══ SW-C3 ═══
enable
configure terminal
hostname SW-C3
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Biblioteca6 → VLAN 34
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 34
 no shutdown

! Admin3 → VLAN 14
interface FastEthernet0/6
 switchport mode access
 switchport access vlan 14
 no shutdown
end
write memory
```

---

### 8.10 SW-D1 — VTP CLIENT | IDF Armario de Piso

```cisco
enable
configure terminal
hostname SW-D1
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

! EtherChannel LACP hacia SW-C4
interface range FastEthernet0/1 - 2
 channel-protocol lacp
 channel-group 6 mode active
 no shutdown
interface port-channel 6
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! Trunk directo hacia SW-D5 (fibra OM3, 1 enlace 802.1Q)
interface FastEthernet0/5
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Trunk GigabitEthernet hacia SW-D2
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Trunk hacia SW-E1 (solo VLAN 54)
interface FastEthernet0/10
 switchport mode trunk
 switchport trunk allowed vlan 54
 no shutdown

banner motd # Bienvenido a Edificio D - NETCORE_202308204 #
end
write memory
```

---

### 8.11 SW-D5 — VTP CLIENT | Salida Edificio D hacia B

```cisco
enable
configure terminal
hostname SW-D5
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

! EtherChannel LACP hacia SW-B2
interface range FastEthernet0/1 - 2
 channel-protocol lacp
 channel-group 5 mode active
 no shutdown
interface port-channel 5
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54

! Trunk directo hacia SW-D1 (fibra OM3)
interface FastEthernet0/5
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Trunk GigabitEthernet hacia SW-D2
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

end
write memory
```

---

### 8.12 SW-D2 — VTP CLIENT | Distribución Edificio D

```cisco
enable
configure terminal
hostname SW-D2
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Hacia Repeater-D1 → SW-D3
interface FastEthernet0/3
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Hacia SW-D4
interface FastEthernet0/4
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Hacia SW-E1 (solo VLAN 54)
interface FastEthernet0/10
 switchport mode trunk
 switchport trunk allowed vlan 54
 no shutdown

end
write memory
```

---

### 8.13 SW-D3 — VTP CLIENT | Acceso Edificio D

```cisco
enable
configure terminal
hostname SW-D3
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

! Desde Repeater-D1
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Admin4 → VLAN 14
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 14
 no shutdown

! Admin5 → VLAN 14
interface FastEthernet0/6
 switchport mode access
 switchport access vlan 14
 no shutdown

! Docencia10 Server → VLAN 24
interface FastEthernet0/7
 switchport mode access
 switchport access vlan 24
 no shutdown

end
write memory
```

---

### 8.14 SW-D4 — VTP CLIENT | Acceso Edificio D

```cisco
enable
configure terminal
hostname SW-D4
vtp mode client
vtp domain C2_NetCore
vtp version 2
spanning-tree mode pvst

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 14,24,34,44,54
 no shutdown

! Laboratorio3 → VLAN 44
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 44
 no shutdown

! Biblioteca7 → VLAN 34
interface FastEthernet0/6
 switchport mode access
 switchport access vlan 34
 no shutdown

end
write memory
```

---

### 8.15 SW-E1 — VTP TRANSPARENTE | Visitantes Edificio D

```cisco
enable
configure terminal
hostname SW-E1

! ── VTP TRANSPARENTE — no propaga ni recibe VLANs ────────
vtp mode transparent
vtp domain C2_NetCore
vtp version 2

! ── VLAN 54 LOCAL (creada manualmente, no por VTP) ───────
vlan 54
 name VISITANTE

spanning-tree mode pvst

! Trunk hacia SW-D1 (solo VLAN 54)
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 54
 no shutdown

! Trunk hacia SW-D2 (solo VLAN 54)
interface FastEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan 54
 no shutdown

! AC-D1 → acceso VLAN 54 (GigabitEthernet)
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 54
 no shutdown

! Visitantes1 → VLAN 54
interface FastEthernet0/5
 switchport mode access
 switchport access vlan 54
 no shutdown

! Visitantes2 → VLAN 54
interface FastEthernet0/6
 switchport mode access
 switchport access vlan 54
 no shutdown

banner motd # Bienvenido a Edificio D (Visitantes) - NETCORE_202308204 #
end
write memory
```

---

### 8.16 ⚠️ CONTRASEÑA VTP — Ejecutar AL FINAL en TODOS los switches

> Después de confirmar que todos los clientes recibieron correctamente las VLANs (verificar con `show vlan brief`), ejecutar en **cada switch**:

```cisco
enable
configure terminal
vtp password proyecto12026
end
write memory
```

Ejecutar en: SW-A1, SW-A2, SW-A3, SW-B1, SW-B2, SW-B3, SW-B4, SW-C4, SW-C1, SW-C2, SW-C3, SW-D1, SW-D2, SW-D3, SW-D4, SW-D5, SW-E1

---

## 9. Pruebas de Ping

> **Regla:** 1 ping exitoso (misma VLAN, diferente switch) + 1 ping fallido (diferente VLAN) por cada VLAN = **10 pruebas totales**.

### VLAN 14 — ADMIN

| # | Origen | IP Origen | Destino | IP Destino | Resultado |
|---|---|---|---|---|---|
| 1 | Admin2 — Ed.A | 192.168.14.2 | Admin3 — Ed.C | 192.168.14.13 | ✅ EXITOSO — misma VLAN, diferente edificio |
| 2 | Admin2 — Ed.A | 192.168.14.2 | Biblioteca1 — Ed.B | 192.168.34.10 | ❌ FALLIDO — VLAN 14 vs VLAN 34 |

> 📸 [Captura ping 1 — exitoso] | 📸 [Captura ping 2 — fallido]

---

### VLAN 24 — DOCENTES

| # | Origen | IP Origen | Destino | IP Destino | Resultado |
|---|---|---|---|---|---|
| 3 | Docencia8 — Ed.C | 192.168.24.18 | Docencia10 — Ed.D | 192.168.24.110 | ✅ EXITOSO — misma VLAN, diferente edificio |
| 4 | Docencia8 — Ed.C | 192.168.24.18 | Admin4 — Ed.D | 192.168.14.14 | ❌ FALLIDO — VLAN 24 vs VLAN 14 |

> 📸 [Captura ping 3 — exitoso] | 📸 [Captura ping 4 — fallido]

---

### VLAN 34 — BIBLIOTECA

| # | Origen | IP Origen | Destino | IP Destino | Resultado |
|---|---|---|---|---|---|
| 5 | Biblioteca1 — Ed.B | 192.168.34.10 | Biblioteca6 — Ed.C | 192.168.34.60 | ✅ EXITOSO — misma VLAN, diferente edificio |
| 6 | Biblioteca1 — Ed.B | 192.168.34.10 | Laboratorio2 — Ed.A | 192.168.44.2 | ❌ FALLIDO — VLAN 34 vs VLAN 44 |

> 📸 [Captura ping 5 — exitoso] | 📸 [Captura ping 6 — fallido]

---

### VLAN 44 — LABORATORIO

| # | Origen | IP Origen | Destino | IP Destino | Resultado |
|---|---|---|---|---|---|
| 7 | Laboratorio2 — Ed.A | 192.168.44.2 | Laboratorio3 — Ed.D | 192.168.44.3 | ✅ EXITOSO — misma VLAN, diferente edificio |
| 8 | Laboratorio2 — Ed.A | 192.168.44.2 | Docencia7 — Ed.C | 192.168.24.17 | ❌ FALLIDO — VLAN 44 vs VLAN 24 |

> 📸 [Captura ping 7 — exitoso] | 📸 [Captura ping 8 — fallido]

---

### VLAN 54 — VISITANTE

| # | Origen | IP Origen | Destino | IP Destino | Resultado |
|---|---|---|---|---|---|
| 9 | Visitantes1 — Ed.D | 192.168.54.1 | Visitantes2 — Ed.D | 192.168.54.2 | ✅ EXITOSO — misma VLAN, mismo switch |
| 10 | Visitantes1 — Ed.D | 192.168.54.1 | Admin2 — Ed.A | 192.168.14.2 | ❌ FALLIDO — VLAN 54 vs VLAN 14 (además aislado por VTP Transparent) |

> 📸 [Captura ping 9 — exitoso] | 📸 [Captura ping 10 — fallido]

---

## 10. Capturas de Validación

### 10.1 show spanning-tree

> 📸 **[CAPTURA REQUERIDA]** Ejecutar en SW-A1. Debe mostrar `This bridge is the root` en todas las VLANs (14, 24, 34, 44, 54).

```
SW-A1# show spanning-tree
[INSERTAR CAPTURA]
```

### 10.2 show etherchannel summary

> 📸 **[CAPTURA REQUERIDA]** Ejecutar en SW-A1. Los Port-Channel deben mostrar flags `SU` (Layer2, in use).

```
SW-A1# show etherchannel summary
[INSERTAR CAPTURA]
```

### 10.3 show interfaces trunk

> 📸 **[CAPTURA REQUERIDA]** Ejecutar en SW-A1. Las VLANs 14,24,34,44,54 deben aparecer en "VLANs allowed and active in management domain".

```
SW-A1# show interfaces trunk
[INSERTAR CAPTURA]
```

### 10.4 show vtp status — comparación de los 3 modos

> 📸 **[CAPTURA REQUERIDA]** Ejecutar en SW-A1 (Server), SW-B1 (Client) y SW-E1 (Transparent) para demostrar los 3 modos.

```
SW-A1# show vtp status
[CAPTURA — VTP Operating Mode: Server | Configuration Revision debe ser > 0]

SW-B1# show vtp status
[CAPTURA — VTP Operating Mode: Client | mismo Configuration Revision que Server]

SW-E1# show vtp status
[CAPTURA — VTP Operating Mode: Transparent | Configuration Revision: 0]
```

---

## 11. Presupuesto Estimado

### Equipos activos

| Equipo | Cant. | Precio Unit. (USD) | Total (USD) |
|---|---|---|---|
| Switch Cisco Catalyst 2960-24TT | 12 | $250.00 | $3,000.00 |
| Switch Cisco Switch-PT equiv. (2960-X) | 7 | $350.00 | $2,450.00 |
| Módulo fibra PT-SWITCH-NM-1FFE | 14 | $80.00 | $1,120.00 |
| Hub 10/100 (Hub-PT equiv.) | 2 | $30.00 | $60.00 |
| Regenerador de señal / Repeater | 1 | $45.00 | $45.00 |
| Access Point Cisco Aironet equiv. | 2 | $150.00 | $300.00 |
| **Subtotal equipos** | | | **$6,975.00** |

### Cableado y accesorios

| Material | Cant. | Precio Unit. | Total |
|---|---|---|---|
| Cable Fibra óptica OM3 multimodo | 200 m | $3.50/m | $700.00 |
| Conectores SC/LC para fibra (par) | 24 pares | $8.00 | $192.00 |
| Cable UTP Cat6 (rollo 305 m) | 3 rollos | $85.00 | $255.00 |
| Cable UTP Cat5e (rollo 305 m) | 2 rollos | $60.00 | $120.00 |
| Patch panel 24 puertos Cat6 | 4 | $45.00 | $180.00 |
| Cable de consola USB-RJ45 | 4 | $15.00 | $60.00 |
| Rack de piso 12U con bandeja | 4 | $120.00 | $480.00 |
| **Subtotal cableado** | | | **$1,987.00** |

### Resumen

| Categoría | Subtotal |
|---|---|
| Equipos activos | $6,975.00 |
| Cableado y accesorios | $1,987.00 |
| **TOTAL ESTIMADO** | **$8,962.00** |
| + IVA Guatemala (12%) | $1,075.44 |
| **TOTAL CON IVA** | **$10,037.44** |

> Precios referenciales en USD. No incluye mano de obra de instalación.

---

## 12. Fases de Implementación

### ✅ FASE 1 — Topología física en Packet Tracer
- [ ] Colocar todos los **Switch-PT** e instalar módulos **PT-SWITCH-NM-1FFE** (necesario para puertos de fibra)
- [ ] Colocar todos los **Cisco 2960-24TT**
- [ ] Conectar fibra OM3 **100Base-FX** entre edificios (2 enlaces por EtherChannel + 1 enlace D1↔D5)
- [ ] Conectar **GigabitEthernet UTP Cat6** entre switches de distribución y acceso
- [ ] Conectar **FastEthernet UTP Cat5e** a PCs, Laptops y dispositivos finales
- [ ] Agregar **Hub-B1** (Edificio B) y **Hub-C1** (Edificio C) con sus conexiones
- [ ] Agregar **Repeater-D1** entre SW-D2 y SW-D3
- [ ] Agregar **AC-A1** (SW-A3) y **AC-D1** (SW-E1) y configurar sus SSID
- [ ] Nombrar todos los dispositivos finales según la topología

### ✅ FASE 2 — VTP y VLANs
- [ ] Configurar **SW-A1 como VTP Server**, dominio `C2_NetCore`, versión 2
- [ ] Crear las 5 VLANs en SW-A1: **14-ADMIN, 24-DOCENTES, 34-BIBLIOTECA, 44-LABORATORIO, 54-VISITANTE**
- [ ] Configurar todos los demás switches (menos SW-E1) como **VTP Client**, mismo dominio
- [ ] Configurar **SW-E1 como VTP Transparent** y crear VLAN 54 localmente
- [ ] Verificar en un cliente con `show vlan brief` — debe mostrar las 5 VLANs

### ✅ FASE 3 — Troncales 802.1Q
- [ ] Configurar `switchport mode trunk` en todos los enlaces inter-switch
- [ ] Ajustar `switchport trunk allowed vlan 14,24,34,44,54` en cada trunk
- [ ] Verificar con `show interfaces trunk`

### ✅ FASE 4 — EtherChannel
- [ ] Configurar **LACP** (mode active) en los 5 EtherChannel inter-edificios
- [ ] Configurar **PAgP** (mode desirable) en EtherChannel intra-edificio SW-A2↔SW-A3
- [ ] Verificar con `show etherchannel summary` — estado debe ser **`SU`**

### ✅ FASE 5 — STP PVST
- [ ] Configurar `spanning-tree mode pvst` en todos los switches
- [ ] Configurar `spanning-tree vlan 14,24,34,44,54 priority 4096` en SW-A1
- [ ] Verificar con `show spanning-tree` — SW-A1 debe ser **ROOT BRIDGE** en todas las VLANs
- [ ] Prueba de reconvergencia: deshabilitar un enlace con `shutdown` y verificar que STP recalcula

### ✅ FASE 6 — Puertos de acceso y direccionamiento IP
- [ ] Asignar cada puerto de acceso a su VLAN correspondiente en cada switch
- [ ] Configurar IPs estáticas en todos los PCs, Laptops y Servers según la tabla de la sección 7
- [ ] Conectar dispositivos WiFi al SSID del Access Point correspondiente

### ✅ FASE 7 — Seguridad y Banners
- [ ] Configurar `banner motd` en SW-A1, SW-B1, SW-C4, SW-D1 y SW-E1 (mínimo 1 por edificio = 5)
- [ ] Configurar `enable secret cisco` y contraseñas de consola/vty en todos
- [ ] Activar `vtp password proyecto12026` en **todos** los switches **(AL FINAL)**

### ✅ FASE 8 — Verificación y documentación
- [ ] Realizar las 10 pruebas de ping (sección 9) y capturar resultados
- [ ] Capturar `show spanning-tree`, `show etherchannel summary`, `show interfaces trunk`
- [ ] Capturar `show vtp status` en SW-A1, SW-B1 y SW-E1
- [ ] Insertar capturas de topología completa y por edificio en las secciones correspondientes
- [ ] Subir `README.md` y `Proyecto1_202308204.pkt` al repositorio antes del **13 de marzo 23:59**

---

## 📁 Estructura del Repositorio

```
Redes1_1S_2026_202308204/
└── Proyecto 1/
    ├── README.md                    ← Este archivo (Manual Técnico)
    └── Proyecto1_202308204.pkt      ← Topología completa en Packet Tracer
```

---

*Manual Técnico | Proyecto 1 — NetCore Academy | Redes de Computadoras 1 — FIUSAC 2026 | Carnet: 202308204*
