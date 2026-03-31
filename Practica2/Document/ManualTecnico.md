# Manual Técnico — Práctica 2: Red de Hospitales Metropolitano
**Curso:** Redes de Computadoras 1  
**Universidad:** San Carlos de Guatemala — Facultad de Ingeniería  
**Carnet:** 20238204  
**Fecha:** 2026  

---

## Índice
1. [Análisis del Problema](#1-análisis-del-problema)
2. [Hospitales Seleccionados](#2-hospitales-seleccionados)
3. [Topología de Red](#3-topología-de-red)
4. [Dominios de Colisión](#4-dominios-de-colisión)
5. [VLANs](#5-vlans)
6. [Subnetting](#6-subnetting)
7. [Configuración VTP](#7-configuración-vtp)
8. [Configuración STP — Rapid PVST+](#8-configuración-stp--rapid-pvst)
9. [EtherChannel (PAgP)](#9-etherchannel-pagp)
10. [Configuración de Switches](#10-configuración-de-switches)
11. [Asignación de Direcciones IP](#11-asignación-de-direcciones-ip)
12. [Pruebas de Conectividad](#12-pruebas-de-conectividad)
13. [Análisis de Paquetes ARP/ICMP](#13-análisis-de-paquetes-arpicmp)

---

## 1. Análisis del Problema

La red de salud nacional requiere modernizar la infraestructura de comunicación entre hospitales metropolitanos. Se diseña una red LAN segmentada con VLANs por área funcional, aplicando protocolos de administración (VTP), redundancia (STP Rapid PVST+) y agregación de enlaces (EtherChannel PAgP).

### Parámetros derivados del carnet 20238204
| Parámetro | Valor |
|-----------|-------|
| Carnet | 20238204 |
| Último dígito (X) | **4** |
| Dígitos XX para subnetting | **4** |
| Dominio VTP | `20238204` |
| Red base | `192.168.4.0/24`, `192.168.5.0/24`, `192.168.6.0/24` |

### Fórmula VLAN ID
```
VLAN ID = [Número de Área][X] = [Número de Área][4]
Ejemplo: Área 1 → VLAN 14
```

---

## 2. Hospitales Seleccionados

Se seleccionaron 6 hospitales reales del área metropolitana de Guatemala:

| # | Nombre del Hospital | Abreviatura |
|---|---------------------|-------------|
| 1 | Hospital Roosevelt | H-ROOSE |
| 2 | Hospital General San Juan de Dios | H-SJDIOS |
| 3 | Hospital de Ginecología y Obstetricia IGSS | H-IGSS |
| 4 | Hospital Nacional de Ortopedia y Rehabilitación | H-ORTOP |
| 5 | Hospital Infantil de Infectología y Rehabilitación | H-INFANT |
| 6 | Hospital Centro Médico Militar | H-MILITA |

---

## 3. Topología de Red

### Tipo de Topología: Estrella Extendida (Híbrida)

Se utiliza una **topología de estrella** entre hospitales, con un **switch central (Core Switch)** que interconecta los switches principales de cada hospital. Dentro de cada hospital se utiliza una topología interna con switches y hubs según los dominios de colisión requeridos.

**Justificación:**
- La topología estrella centraliza la administración y facilita el aislamiento de fallas.
- Un único punto central permite implementar EtherChannel (PAgP) de forma eficiente.
- La redundancia se garantiza mediante STP Rapid PVST+.
- La escalabilidad es mayor que una topología en anillo.

### Diagrama de Topología General

```
                        [CORE SWITCH]
                       (Switch Central)
                     /    |    |    |    \
                    /     |    |    |     \
              [SW-H1] [SW-H2][SW-H3][SW-H4][SW-H5][SW-H6]
```

> **📸 INSERTAR CAPTURA:** Topología completa en Cisco Packet Tracer (vista general de todos los hospitales conectados al core switch).

---

## 4. Dominios de Colisión

### Conceptos Aplicados
- **Switch:** Cada puerto es un dominio de colisión independiente → N puertos = N dominios.
- **Hub:** Todos los puertos comparten el mismo dominio de colisión → 1 Hub = 1 dominio total.

### Diseño por Hospital

| Hospital | Hosts Mínimos | Dominios Requeridos | Diseño Propuesto |
|----------|--------------|---------------------|-----------------|
| H-ROOSE (H1) | 63 | 3 | 1 Switch principal + 2 Hubs (cada hub = 1 dominio; switch crea 1 dominio para uplink) |
| H-SJDIOS (H2) | 25 | 2 | 1 Switch principal + 1 Hub |
| H-IGSS (H3) | 32 | 3 | 1 Switch principal + 2 Hubs |
| H-ORTOP (H4) | 15 | 2 | 1 Switch principal + 1 Hub |
| H-INFANT (H5) | 26 | 2 | 1 Switch principal + 1 Hub |
| H-MILITA (H6) | 8 | 1 | 1 Hub único (todo en el mismo dominio de colisión) |

### Explicación Técnica

**Hospital 1 — 3 dominios de colisión:**
```
[SW-H1] ──── [HUB-A] ──── PCs (dominio 1 = HUB-A)
          └── [HUB-B] ──── PCs (dominio 2 = HUB-B)
          └── Puerto uplink al Core (dominio 3 = segmento switch)
```

**Hospital 6 — 1 dominio de colisión:**
```
[HUB] ──── PC1, PC2, PC3 ... PC8  (todo un solo dominio)
```

> **📸 INSERTAR CAPTURA:** Vista interna de cada hospital mostrando switches y hubs en Packet Tracer.

---

## 5. VLANs

### Áreas Funcionales Identificadas (mínimo 4)

| Área # | Nombre del Área | VLAN ID (área+4) | Descripción |
|--------|----------------|-----------------|-------------|
| 1 | Quirófanos | **14** | Área de cirugías y procedimientos quirúrgicos |
| 2 | UCI / Cuidados Intensivos | **24** | Unidad de cuidados intensivos |
| 3 | Administración | **34** | Personal administrativo y directivo |
| 4 | Urgencias / Emergencias | **44** | Atención de emergencias y urgencias |
| — | Nativa (Troncales) | **99** | VLAN nativa para enlaces troncales |
| — | Blackhole (puertos sin uso) | **999** | Puertos no utilizados (seguridad) |

### Justificación de VLANs
- **VLAN 14 (Quirófanos):** Aísla el tráfico crítico de cirugías para garantizar ancho de banda y seguridad.
- **VLAN 24 (UCI):** Prioridad alta; el tráfico de monitoreo de pacientes no debe mezclarse con tráfico administrativo.
- **VLAN 34 (Administración):** Segmenta el tráfico de correo, reportes y sistemas hospitalarios.
- **VLAN 44 (Urgencias):** Acceso rápido a sistemas de triaje y registro de pacientes.
- **VLAN 99 (Nativa):** Estándar de seguridad para troncales; evita VLAN hopping en VLAN 1.
- **VLAN 999 (Blackhole):** Puertos no utilizados asignados aquí para prevenir acceso no autorizado.

> **📸 INSERTAR CAPTURA:** Salida del comando `show vlan brief` en el switch principal mostrando todas las VLANs configuradas.

---

## 6. Subnetting

### Red Base Asignada
Según carnet XX = 4:
- Red 1: `192.168.4.0/24`
- Red 2: `192.168.5.0/24`  
- Red 3: `192.168.6.0/24`

### Cálculo de Subnetting por VLAN

Se realiza subnetting para asignar **una subred por VLAN por grupo de hospitales**. Se usa la red `192.168.4.0/24` como base y se divide en subredes según la cantidad de hosts requeridos.

#### Tabla de Subnetting Completa

| VLAN | Área | Red de Subred | Máscara | Prefijo | Gateway | Rango Usable | Broadcast | Hosts Disponibles |
|------|------|--------------|---------|---------|---------|--------------|-----------|-------------------|
| 14 | Quirófanos | 192.168.4.0 | 255.255.255.128 | /25 | 192.168.4.1 | 192.168.4.2 – 192.168.4.126 | 192.168.4.127 | 126 |
| 24 | UCI | 192.168.4.128 | 255.255.255.192 | /26 | 192.168.4.129 | 192.168.4.130 – 192.168.4.190 | 192.168.4.191 | 62 |
| 34 | Administración | 192.168.4.192 | 255.255.255.192 | /26 | 192.168.4.193 | 192.168.4.194 – 192.168.4.254 | 192.168.4.255 | 62 |
| 44 | Urgencias | 192.168.5.0 | 255.255.255.128 | /25 | 192.168.5.1 | 192.168.5.2 – 192.168.5.126 | 192.168.5.127 | 126 |
| 99 | Nativa (mgmt) | 192.168.5.128 | 255.255.255.240 | /28 | 192.168.5.129 | 192.168.5.130 – 192.168.5.142 | 192.168.5.143 | 14 |

#### Justificación de Máscaras
- **VLAN 14 (/25 → 126 hosts):** Quirófanos concentra el mayor número de dispositivos médicos y PCs entre los 6 hospitales (total estimado: ~80 hosts).
- **VLAN 24 (/26 → 62 hosts):** UCI tiene equipos de monitoreo pero en menor cantidad.
- **VLAN 34 (/26 → 62 hosts):** Personal administrativo distribuido en 6 hospitales.
- **VLAN 44 (/25 → 126 hosts):** Urgencias requiere capacidad alta para registros y triaje.
- **VLAN 99 (/28 → 14 hosts):** Solo para interfaces de administración de switches (6 switches + core).

> **📸 INSERTAR CAPTURA:** Tabla de subnetting en el documento o captura del cálculo en una herramienta.

---

## 7. Configuración VTP

### Parámetros VTP
| Parámetro | Valor |
|-----------|-------|
| Dominio | `20238204` |
| Contraseña Área 1 | `area1` |
| Contraseña Área 2 | `area2` |
| Contraseña Área 3 | `area3` |
| Contraseña Área 4 | `area4` |
| Switch Servidor | Core Switch (SW-CORE) |
| Switches Clientes | SW-H1, SW-H2, SW-H3, SW-H4, SW-H5, SW-H6 |
| Versión VTP | 2 |

### Comandos — Switch Servidor (SW-CORE)

```
SW-CORE# configure terminal
SW-CORE(config)# vtp mode server
SW-CORE(config)# vtp domain 20238204
SW-CORE(config)# vtp password area1
SW-CORE(config)# vtp version 2
SW-CORE(config)# end
SW-CORE# show vtp status
```

### Comandos — Switches Clientes (ejemplo SW-H1)

```
SW-H1# configure terminal
SW-H1(config)# vtp mode client
SW-H1(config)# vtp domain 20238204
SW-H1(config)# vtp password area1
SW-H1(config)# end
SW-H1# show vtp status
```

> **📸 INSERTAR CAPTURA:** Salida de `show vtp status` en el switch servidor y en al menos un cliente, mostrando sincronización del dominio.

---

## 8. Configuración STP — Rapid PVST+

### Descripción
Se implementa **Rapid PVST+ (Per-VLAN Spanning Tree Plus)** para evitar bucles de red. El **SW-CORE** es designado como Root Bridge para todas las VLANs.

### Configuración Root Bridge (SW-CORE)

```
SW-CORE# configure terminal
SW-CORE(config)# spanning-tree mode rapid-pvst
SW-CORE(config)# spanning-tree vlan 14 root primary
SW-CORE(config)# spanning-tree vlan 24 root primary
SW-CORE(config)# spanning-tree vlan 34 root primary
SW-CORE(config)# spanning-tree vlan 44 root primary
SW-CORE(config)# spanning-tree vlan 99 root primary
SW-CORE(config)# end
SW-CORE# show spanning-tree vlan 14
```

### Configuración Switches de Acceso (ejemplo SW-H1)

```
SW-H1# configure terminal
SW-H1(config)# spanning-tree mode rapid-pvst
SW-H1(config)# end
SW-H1# show spanning-tree
```

### Justificación del Root Bridge
El **SW-CORE** es elegido como Root Bridge porque:
1. Es el punto central de la topología estrella.
2. Concentra todo el tráfico inter-hospital.
3. Tiene la mayor capacidad de procesamiento.
4. Garantiza que los caminos óptimos siempre pasen por el núcleo.

> **📸 INSERTAR CAPTURA:** Salida de `show spanning-tree` mostrando SW-CORE como Root Bridge con estado "This bridge is the root".

---

## 9. EtherChannel (PAgP)

### Descripción
Se agrupan **3 enlaces físicos** entre el SW-CORE y los switches principales de hospital usando el protocolo **PAgP (Port Aggregation Protocol)** de Cisco.

### Parámetros EtherChannel
| Parámetro | Valor |
|-----------|-------|
| Protocolo | PAgP |
| Número de enlaces | 3 por canal |
| Channel-group | 1 (para el primer par de switches) |
| Modo SW-CORE | `desirable` (inicia negociación) |
| Modo SW-Hospital | `auto` (responde a negociación) |

### Comandos — SW-CORE (hacia SW-H1, ejemplo)

```
SW-CORE# configure terminal
SW-CORE(config)# interface range GigabitEthernet0/1 - 3
SW-CORE(config-if-range)# channel-group 1 mode desirable
SW-CORE(config-if-range)# exit
SW-CORE(config)# interface port-channel 1
SW-CORE(config-if)# switchport mode trunk
SW-CORE(config-if)# switchport trunk native vlan 99
SW-CORE(config-if)# switchport trunk allowed vlan 14,24,34,44,99
SW-CORE(config-if)# end
SW-CORE# show etherchannel summary
```

### Comandos — SW-H1

```
SW-H1# configure terminal
SW-H1(config)# interface range GigabitEthernet0/1 - 3
SW-H1(config-if-range)# channel-group 1 mode auto
SW-H1(config-if-range)# exit
SW-H1(config)# interface port-channel 1
SW-H1(config-if)# switchport mode trunk
SW-H1(config-if)# switchport trunk native vlan 99
SW-H1(config-if)# switchport trunk allowed vlan 14,24,34,44,99
SW-H1(config-if)# end
SW-H1# show etherchannel summary
```

> **📸 INSERTAR CAPTURA:** Salida de `show etherchannel summary` mostrando el Port-Channel en estado "SU" (bundled y up).

---

## 10. Configuración de Switches

### 10.1 Configuración Básica — Plantilla para todos los switches

```
! =============================================
! CONFIGURACIÓN BÁSICA — reemplazar HOSTNAME y PASSWORD
! =============================================
Switch> enable
Switch# configure terminal
Switch(config)# hostname [SW-CORE | SW-H1 | SW-H2 | ...]
Switch(config)# enable secret 20238204
Switch(config)# line console 0
Switch(config-line)# password 20238204
Switch(config-line)# login
Switch(config-line)# exit
Switch(config)# line vty 0 4
Switch(config-line)# password 20238204
Switch(config-line)# login
Switch(config-line)# exit
Switch(config)# service password-encryption
Switch(config)# no ip domain-lookup
Switch(config)# banner motd # Red Hospitales Metropolitano - Acceso Autorizado #
```

### 10.2 Creación de VLANs (Solo en SW-CORE — se propagan por VTP)

```
SW-CORE(config)# vlan 14
SW-CORE(config-vlan)# name Quirofanos
SW-CORE(config-vlan)# exit
SW-CORE(config)# vlan 24
SW-CORE(config-vlan)# name UCI_Cuidados_Intensivos
SW-CORE(config-vlan)# exit
SW-CORE(config)# vlan 34
SW-CORE(config-vlan)# name Administracion
SW-CORE(config-vlan)# exit
SW-CORE(config)# vlan 44
SW-CORE(config-vlan)# name Urgencias_Emergencias
SW-CORE(config-vlan)# exit
SW-CORE(config)# vlan 99
SW-CORE(config-vlan)# name VLAN_Nativa_Troncal
SW-CORE(config-vlan)# exit
SW-CORE(config)# vlan 999
SW-CORE(config-vlan)# name Blackhole_No_Usar
SW-CORE(config-vlan)# exit
```

### 10.3 Configuración de Puertos de Acceso (ejemplo SW-H1)

```
! --- Puertos VLAN 14 (Quirófanos) ---
SW-H1(config)# interface range FastEthernet0/1 - 5
SW-H1(config-if-range)# switchport mode access
SW-H1(config-if-range)# switchport access vlan 14
SW-H1(config-if-range)# spanning-tree portfast
SW-H1(config-if-range)# exit

! --- Puertos VLAN 24 (UCI) ---
SW-H1(config)# interface range FastEthernet0/6 - 10
SW-H1(config-if-range)# switchport mode access
SW-H1(config-if-range)# switchport access vlan 24
SW-H1(config-if-range)# spanning-tree portfast
SW-H1(config-if-range)# exit

! --- Puertos VLAN 34 (Administración) ---
SW-H1(config)# interface range FastEthernet0/11 - 15
SW-H1(config-if-range)# switchport mode access
SW-H1(config-if-range)# switchport access vlan 34
SW-H1(config-if-range)# spanning-tree portfast
SW-H1(config-if-range)# exit

! --- Puertos VLAN 44 (Urgencias) ---
SW-H1(config)# interface range FastEthernet0/16 - 20
SW-H1(config-if-range)# switchport mode access
SW-H1(config-if-range)# switchport access vlan 44
SW-H1(config-if-range)# spanning-tree portfast
SW-H1(config-if-range)# exit

! --- Puertos NO UTILIZADOS → VLAN 999 (Blackhole) ---
SW-H1(config)# interface range FastEthernet0/21 - 24
SW-H1(config-if-range)# switchport mode access
SW-H1(config-if-range)# switchport access vlan 999
SW-H1(config-if-range)# shutdown
SW-H1(config-if-range)# exit
```

### 10.4 Configuración de Puerto Troncal (uplink al Core)

```
! --- Puerto troncal hacia SW-CORE ---
SW-H1(config)# interface GigabitEthernet0/1
SW-H1(config-if)# switchport mode trunk
SW-H1(config-if)# switchport trunk native vlan 99
SW-H1(config-if)# switchport trunk allowed vlan 14,24,34,44,99
SW-H1(config-if)# exit
```

### 10.5 Interfaz de Administración (SVI — Switch Virtual Interface)

```
! --- IP de administración en VLAN 99 ---
SW-H1(config)# interface vlan 99
SW-H1(config-if)# ip address 192.168.5.130 255.255.255.240
SW-H1(config-if)# no shutdown
SW-H1(config-if)# exit
SW-H1(config)# ip default-gateway 192.168.5.129
```

> **📸 INSERTAR CAPTURA:** Salida de `show running-config` del switch principal mostrando todas las configuraciones aplicadas.

> **📸 INSERTAR CAPTURA:** Salida de `show interfaces trunk` mostrando los puertos troncales activos con la VLAN nativa 99.

---

## 11. Asignación de Direcciones IP

### Tabla Completa de IPs por Hospital y VLAN

#### VLAN 14 — Quirófanos (192.168.4.0/25)

| Dispositivo | Hospital | IP Asignada | Máscara | Gateway |
|-------------|----------|-------------|---------|---------|
| PC-Q-H1-1 | Roosevelt | 192.168.4.2 | 255.255.255.128 | 192.168.4.1 |
| PC-Q-H1-2 | Roosevelt | 192.168.4.3 | 255.255.255.128 | 192.168.4.1 |
| PC-Q-H2-1 | San Juan de Dios | 192.168.4.10 | 255.255.255.128 | 192.168.4.1 |
| PC-Q-H3-1 | IGSS | 192.168.4.20 | 255.255.255.128 | 192.168.4.1 |
| PC-Q-H4-1 | Ortopedia | 192.168.4.30 | 255.255.255.128 | 192.168.4.1 |
| PC-Q-H5-1 | Hospital Infantil | 192.168.4.40 | 255.255.255.128 | 192.168.4.1 |
| PC-Q-H6-1 | Centro Médico Militar | 192.168.4.50 | 255.255.255.128 | 192.168.4.1 |

#### VLAN 24 — UCI (192.168.4.128/26)

| Dispositivo | Hospital | IP Asignada | Máscara | Gateway |
|-------------|----------|-------------|---------|---------|
| PC-U-H1-1 | Roosevelt | 192.168.4.130 | 255.255.255.192 | 192.168.4.129 |
| PC-U-H1-2 | Roosevelt | 192.168.4.131 | 255.255.255.192 | 192.168.4.129 |
| PC-U-H2-1 | San Juan de Dios | 192.168.4.140 | 255.255.255.192 | 192.168.4.129 |
| PC-U-H3-1 | IGSS | 192.168.4.150 | 255.255.255.192 | 192.168.4.129 |
| PC-U-H4-1 | Ortopedia | 192.168.4.160 | 255.255.255.192 | 192.168.4.129 |
| PC-U-H5-1 | Hospital Infantil | 192.168.4.170 | 255.255.255.192 | 192.168.4.129 |

#### VLAN 34 — Administración (192.168.4.192/26)

| Dispositivo | Hospital | IP Asignada | Máscara | Gateway |
|-------------|----------|-------------|---------|---------|
| PC-A-H1-1 | Roosevelt | 192.168.4.194 | 255.255.255.192 | 192.168.4.193 |
| PC-A-H2-1 | San Juan de Dios | 192.168.4.200 | 255.255.255.192 | 192.168.4.193 |
| PC-A-H3-1 | IGSS | 192.168.4.210 | 255.255.255.192 | 192.168.4.193 |
| PC-A-H4-1 | Ortopedia | 192.168.4.220 | 255.255.255.192 | 192.168.4.193 |
| PC-A-H5-1 | Hospital Infantil | 192.168.4.230 | 255.255.255.192 | 192.168.4.193 |
| PC-A-H6-1 | Centro Médico Militar | 192.168.4.240 | 255.255.255.192 | 192.168.4.193 |

#### VLAN 44 — Urgencias (192.168.5.0/25)

| Dispositivo | Hospital | IP Asignada | Máscara | Gateway |
|-------------|----------|-------------|---------|---------|
| PC-E-H1-1 | Roosevelt | 192.168.5.2 | 255.255.255.128 | 192.168.5.1 |
| PC-E-H2-1 | San Juan de Dios | 192.168.5.10 | 255.255.255.128 | 192.168.5.1 |
| PC-E-H3-1 | IGSS | 192.168.5.20 | 255.255.255.128 | 192.168.5.1 |
| PC-E-H4-1 | Ortopedia | 192.168.5.30 | 255.255.255.128 | 192.168.5.1 |
| PC-E-H5-1 | Hospital Infantil | 192.168.5.40 | 255.255.255.128 | 192.168.5.1 |
| PC-E-H6-1 | Centro Médico Militar | 192.168.5.50 | 255.255.255.128 | 192.168.5.1 |

#### VLAN 99 — Administración de Switches (192.168.5.128/28)

| Dispositivo | IP Asignada | Máscara | Gateway |
|-------------|-------------|---------|---------|
| SW-CORE (SVI) | 192.168.5.129 | 255.255.255.240 | — |
| SW-H1 (SVI) | 192.168.5.130 | 255.255.255.240 | 192.168.5.129 |
| SW-H2 (SVI) | 192.168.5.131 | 255.255.255.240 | 192.168.5.129 |
| SW-H3 (SVI) | 192.168.5.132 | 255.255.255.240 | 192.168.5.129 |
| SW-H4 (SVI) | 192.168.5.133 | 255.255.255.240 | 192.168.5.129 |
| SW-H5 (SVI) | 192.168.5.134 | 255.255.255.240 | 192.168.5.129 |
| SW-H6 (SVI) | 192.168.5.135 | 255.255.255.240 | 192.168.5.129 |

> **📸 INSERTAR CAPTURA:** Pantalla de configuración IP de al menos 2 PCs en diferentes VLANs (ventana de propiedades en Packet Tracer).

---

## 12. Pruebas de Conectividad

### 12.1 Pruebas Dentro de la Misma VLAN (deben ser exitosas ✅)

#### VLAN 14 — Quirófanos
```
PC-Q-H1-1> ping 192.168.4.10
! (ping de Roosevelt a San Juan de Dios, misma VLAN 14)
```
> **📸 INSERTAR CAPTURA:** Resultado del ping exitoso entre PCs de VLAN 14 (mínimo 10 pings exitosos).

```
PC-Q-H2-1> ping 192.168.4.20
! (ping de San Juan de Dios a IGSS, misma VLAN 14)
```
> **📸 INSERTAR CAPTURA:** Resultado del ping VLAN 14 - segundo par de dispositivos.

#### VLAN 24 — UCI
```
PC-U-H1-1> ping 192.168.4.140
! (ping de Roosevelt a San Juan de Dios, misma VLAN 24)
```
> **📸 INSERTAR CAPTURA:** Resultado del ping exitoso entre PCs de VLAN 24.

#### VLAN 34 — Administración
```
PC-A-H1-1> ping 192.168.4.200
! (ping de Roosevelt a San Juan de Dios, misma VLAN 34)
```
> **📸 INSERTAR CAPTURA:** Resultado del ping exitoso entre PCs de VLAN 34.

#### VLAN 44 — Urgencias
```
PC-E-H1-1> ping 192.168.5.10
! (ping de Roosevelt a San Juan de Dios, misma VLAN 44)
```
> **📸 INSERTAR CAPTURA:** Resultado del ping exitoso entre PCs de VLAN 44.

### 12.2 Pruebas Entre VLANs Distintas (deben fallar ❌ — correcto comportamiento)

```
PC-Q-H1-1> ping 192.168.4.130
! (intento de ping de VLAN 14 a VLAN 24 — DEBE FALLAR)
```
> **📸 INSERTAR CAPTURA:** Ping fallido entre VLANs distintas (confirma aislamiento correcto).

### 12.3 Resumen de Pruebas

| Prueba | Origen | Destino | VLAN | Resultado Esperado |
|--------|--------|---------|------|--------------------|
| 1 | PC-Q-H1-1 (4.2) | PC-Q-H2-1 (4.10) | 14→14 | ✅ Exitoso |
| 2 | PC-Q-H2-1 (4.10) | PC-Q-H3-1 (4.20) | 14→14 | ✅ Exitoso |
| 3 | PC-Q-H3-1 (4.20) | PC-Q-H5-1 (4.40) | 14→14 | ✅ Exitoso |
| 4 | PC-U-H1-1 (4.130) | PC-U-H2-1 (4.140) | 24→24 | ✅ Exitoso |
| 5 | PC-U-H2-1 (4.140) | PC-U-H4-1 (4.160) | 24→24 | ✅ Exitoso |
| 6 | PC-A-H1-1 (4.194) | PC-A-H3-1 (4.210) | 34→34 | ✅ Exitoso |
| 7 | PC-A-H3-1 (4.210) | PC-A-H6-1 (4.240) | 34→34 | ✅ Exitoso |
| 8 | PC-E-H1-1 (5.2) | PC-E-H3-1 (5.20) | 44→44 | ✅ Exitoso |
| 9 | PC-E-H3-1 (5.20) | PC-E-H5-1 (5.40) | 44→44 | ✅ Exitoso |
| 10 | PC-E-H5-1 (5.40) | PC-E-H6-1 (5.50) | 44→44 | ✅ Exitoso |
| 11 | PC-Q-H1-1 (4.2) | PC-U-H1-1 (4.130) | 14→24 | ❌ Falla (aislamiento) |

---

## 13. Análisis de Paquetes ARP/ICMP

### 13.1 Modo Simulación en Packet Tracer

Para analizar paquetes ARP e ICMP:
1. Cambiar a **Modo Simulación** (botón en esquina inferior derecha de Packet Tracer).
2. Filtrar por **ARP** e **ICMP** en "Event List Filters".
3. Ejecutar un ping entre dos PCs de la misma VLAN.
4. Observar los pasos: ARP Request → ARP Reply → ICMP Echo Request → ICMP Echo Reply.

### 13.2 Flujo ARP (antes del primer ping)

```
Paso 1: PC-Q-H1-1 no conoce la MAC de PC-Q-H2-1
        → Envía ARP Request (broadcast) dentro de VLAN 14
        → Destino MAC: FF:FF:FF:FF:FF:FF

Paso 2: PC-Q-H2-1 recibe el ARP Request
        → Responde con ARP Reply (unicast)
        → Destino MAC: MAC de PC-Q-H1-1

Paso 3: PC-Q-H1-1 actualiza su tabla ARP
        → Ahora conoce MAC↔IP de PC-Q-H2-1
```

> **📸 INSERTAR CAPTURA:** Vista del modo simulación mostrando el paquete ARP Request en VLAN 14 (sobre de color diferente en la animación de Packet Tracer).

> **📸 INSERTAR CAPTURA:** Detalle del paquete ARP en la ventana de simulación (PDU Information) mostrando: MAC origen, MAC destino FF:FF:FF:FF:FF:FF, IP origen e IP destino.

### 13.3 Flujo ICMP (ping)

```
Paso 1: ICMP Echo Request
        PC-Q-H1-1 → PC-Q-H2-1
        Protocolo: ICMP Type 8 (Echo Request)

Paso 2: ICMP Echo Reply
        PC-Q-H2-1 → PC-Q-H1-1
        Protocolo: ICMP Type 0 (Echo Reply)
```

> **📸 INSERTAR CAPTURA:** Detalle del paquete ICMP en modo simulación mostrando Type, Code y datos del paquete.

### 13.4 Comportamiento en VLANs Diferentes

Cuando se intenta hacer ping entre VLAN 14 y VLAN 24:
- El ARP Request de VLAN 14 **no llega** a hosts de VLAN 24 (el switch no reenvía broadcast entre VLANs diferentes sin un router).
- El ping falla con "Request Timeout".

> **📸 INSERTAR CAPTURA:** Modo simulación mostrando que el paquete ARP no cruza entre VLANs (queda detenido en el switch).

---

## Estándar TIA/EIA-568B

### Tabla de Colores — Patch Cables 568B

| Pin | Color |
|-----|-------|
| 1 | Blanco/Naranja |
| 2 | Naranja |
| 3 | Blanco/Verde |
| 4 | Azul |
| 5 | Blanco/Azul |
| 6 | Verde |
| 7 | Blanco/Café |
| 8 | Café |

### Tipos de Cable Aplicados

| Conexión | Tipo de Cable | Justificación |
|----------|--------------|---------------|
| PC → Switch | Straight-through | Dispositivos de diferente tipo |
| PC → Hub | Straight-through | Dispositivos de diferente tipo |
| Switch → Switch | Crossover | Dispositivos del mismo tipo (o Auto-MDIX) |
| Hub → Switch | Crossover | Mismo nivel OSI |
| Switch → Router | Straight-through | Diferente tipo |

> En Cisco Packet Tracer, se selecciona automáticamente el tipo de cable correcto usando "Copper Straight-through" o "Copper Cross-over" según corresponde.

---

*Fin del Manual Técnico*  
*Carnet: 20238204 | Redes de Computadoras 1 | USAC 2026*