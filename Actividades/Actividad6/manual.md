# Actividad Práctica 6 — Inter VLAN Routing
**Universidad San Carlos de Guatemala | Facultad de Ingeniería | ECYS**  
**Curso:** Redes de Computadoras 1 | **Ponderación:** 1 pt | **Tiempo estimado:** 30 min

---

## Tabla de Contenidos

1. [Conceptos Previos](#1-conceptos-previos)
2. [Topología de Red](#2-topología-de-red)
3. [Paso 1 — Crear la topología en Cisco Packet Tracer](#3-paso-1--crear-la-topología-en-cisco-packet-tracer)
4. [Paso 2 — Configurar las VLANs en los switches](#4-paso-2--configurar-las-vlans-en-los-switches)
5. [Paso 3 — Configurar puertos de acceso (hacia las PCs)](#5-paso-3--configurar-puertos-de-acceso-hacia-las-pcs)
6. [Paso 4 — Configurar puertos troncales (entre switches)](#6-paso-4--configurar-puertos-troncales-entre-switches)
7. [Paso 5 — Configurar las PCs con sus IPs](#7-paso-5--configurar-las-pcs-con-sus-ips)
8. [Paso 6 — Configurar el Router (Router-on-a-Stick)](#8-paso-6--configurar-el-router-router-on-a-stick)
9. [Paso 7 — Verificar conectividad con Ping](#9-paso-7--verificar-conectividad-con-ping)
10. [Entregables](#10-entregables)

---

## 1. Conceptos Previos

Antes de comenzar, es importante entender los siguientes conceptos:

### ¿Qué es una VLAN?
Una **VLAN (Virtual Local Area Network)** es una segmentación lógica de una red física. Permite agrupar dispositivos en redes separadas aunque estén conectados al mismo switch físico. Las VLANs mejoran la seguridad y el rendimiento al aislar el tráfico.

### ¿Qué es Inter VLAN Routing?
Por defecto, los dispositivos en diferentes VLANs **no pueden comunicarse entre sí** (ese es el propósito del aislamiento). El **Inter VLAN Routing** es la técnica que permite que el tráfico cruce de una VLAN a otra, usando un **router** como intermediario.

### ¿Qué es Router-on-a-Stick?
Es la técnica más común para Inter VLAN Routing. Consiste en conectar el router a un switch mediante un **único enlace físico** (troncal), pero configurar múltiples **subinterfaces lógicas** en ese enlace, una por cada VLAN. El router recibe tráfico de una VLAN, lo enruta y lo reenvía a otra.

### ¿Qué es un Puerto Troncal (Trunk)?
Un puerto configurado en modo **trunk** transporta tráfico de **múltiples VLANs** simultáneamente. Utiliza etiquetas 802.1Q para identificar a qué VLAN pertenece cada trama. Se usa en los enlaces entre switches y entre el switch y el router.

### ¿Qué es un Puerto de Acceso (Access)?
Un puerto en modo **access** pertenece a una única VLAN. Se conecta directamente a los dispositivos finales (PCs). El dispositivo final no necesita saber nada sobre VLANs.

### ¿Qué es VLSM?
**VLSM (Variable Length Subnet Mask)** permite dividir una red en subredes de diferentes tamaños según las necesidades. En esta práctica, la red `172.15.XX.0/24` ya fue dividida así:

| VLAN | Nombre | Red | Máscara | Hosts disponibles |
|------|--------|-----|---------|-------------------|
| 10   | ADMON  | 172.15.XX.0/26   | 255.255.255.192 | 62 hosts |
| 30   | IT     | 172.15.XX.64/27  | 255.255.255.224 | 30 hosts |
| 20   | RRHH   | 172.15.XX.96/28  | 255.255.255.240 | 14 hosts |

> **Nota:** Reemplaza `XX` con los dos últimos dígitos de tu número de carnet.

---

## 2. Topología de Red

La red a implementar tiene la siguiente estructura:

```
                    [Router ISR4331 - ROAS]
                           |
              (enlace troncal al Switch1)
                           |
               +-----------+-----------+
               |                       |
          [Switch1]               [Switch2]
               |                       |
               +----------+------------+
                          |
                     [Switch3]
                    /    |    \
                  /      |      \
              [PC01]  [PC02]  [PC03]
             VLAN10  VLAN20  VLAN30
```

**Dispositivos requeridos en Packet Tracer:**
- 1x Router ISR4331 (o similar)
- 3x Switch 2960-24TT
- 3x PC-PT

**Diagrama de la topología (captura de Packet Tracer):**

> 📸 **[IMAGEN 1 — Topología completa en Packet Tracer]**
> *(Inserta aquí una captura de pantalla mostrando todos los dispositivos conectados y los nombres visibles)*

---

## 3. Paso 1 — Crear la topología en Cisco Packet Tracer

### 3.1 Abrir Cisco Packet Tracer
1. Abre Cisco Packet Tracer (descárgalo desde https://legacy.netacad.com/portal/resources/packet-tracer si no lo tienes).
2. Crea un nuevo proyecto en blanco.

### 3.2 Agregar los dispositivos
En la barra inferior de dispositivos:
- Selecciona **Network Devices > Routers** → arrastra un **ISR4331** al área de trabajo.
- Selecciona **Network Devices > Switches** → arrastra **3 switches 2960-24TT**.
- Selecciona **End Devices** → arrastra **3 PC-PT**.

### 3.3 Nombrar los dispositivos
Haz doble clic en cada dispositivo y en la pestaña **Config** cambia el nombre (Display Name):
- Router → `ROAS`
- Switches → `Switch1`, `Switch2`, `Switch3`
- PCs → `PC01`, `PC02`, `PC03`

### 3.4 Conectar los dispositivos
Usa el cable **Copper Straight-Through** (para diferentes tipos de dispositivos) o **Copper Cross-Over** según corresponda. En Packet Tracer puedes usar el cable automático (rayo ⚡):

| Desde | Puerto | Hacia | Puerto |
|-------|--------|-------|--------|
| ROAS  | GigabitEthernet0/0 | Switch1 | FastEthernet0/1 |
| Switch1 | FastEthernet0/2 | Switch2 | FastEthernet0/1 |
| Switch1 | FastEthernet0/3 | Switch3 | FastEthernet0/1 |
| Switch2 | FastEthernet0/2 | Switch3 | FastEthernet0/2 |
| Switch3 | FastEthernet0/3 | PC01 | FastEthernet0 |
| Switch3 | FastEthernet0/4 | PC02 | FastEthernet0 |
| Switch3 | FastEthernet0/5 | PC03 | FastEthernet0 |

---

## 4. Paso 2 — Configurar las VLANs en los switches

Hay que crear las VLANs en **cada switch** (Switch1, Switch2 y Switch3). Haz clic en el switch → pestaña **CLI**.

### Comandos para Switch1, Switch2 y Switch3 (repetir en cada uno):

```
Switch> enable
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name ADMON
Switch(config-vlan)# exit
Switch(config)# vlan 20
Switch(config-vlan)# name RRHH
Switch(config-vlan)# exit
Switch(config)# vlan 30
Switch(config-vlan)# name IT
Switch(config-vlan)# exit
Switch(config)# end
Switch# write memory
```

**¿Qué hace esto?**
- `vlan 10` — crea la VLAN con ID 10
- `name ADMON` — le asigna un nombre descriptivo
- `write memory` — guarda la configuración

> 💡 **Tip:** Puedes verificar que las VLANs se crearon con `Switch# show vlan brief`

---

## 5. Paso 3 — Configurar puertos de acceso (hacia las PCs)

Los puertos que conectan a las PCs deben estar en modo **access** y asignados a su VLAN correspondiente. Estos comandos van solo en **Switch3** (donde están conectadas las PCs).

### PC01 → VLAN 10 (puerto FastEthernet0/3)

```
Switch3(config)# interface fastEthernet 0/3
Switch3(config-if)# switchport mode access
Switch3(config-if)# switchport access vlan 10
Switch3(config-if)# exit
```

### PC02 → VLAN 20 (puerto FastEthernet0/4)

```
Switch3(config)# interface fastEthernet 0/4
Switch3(config-if)# switchport mode access
Switch3(config-if)# switchport access vlan 20
Switch3(config-if)# exit
```

### PC03 → VLAN 30 (puerto FastEthernet0/5)

```
Switch3(config)# interface fastEthernet 0/5
Switch3(config-if)# switchport mode access
Switch3(config-if)# switchport access vlan 30
Switch3(config-if)# exit
Switch3(config)# end
Switch3# write memory
```

**¿Qué hace esto?**
- `switchport mode access` — el puerto opera en modo acceso (una sola VLAN, sin etiquetas 802.1Q)
- `switchport access vlan 10` — asigna el puerto a la VLAN 10

---

## 6. Paso 4 — Configurar puertos troncales (entre switches)

Los puertos que conectan switches entre sí, y el enlace del switch hacia el router, deben estar en modo **trunk** para transportar todas las VLANs.

### En Switch1:

```
Switch1> enable
Switch1# configure terminal

! Puerto hacia el Router
Switch1(config)# interface fastEthernet 0/1
Switch1(config-if)# switchport mode trunk
Switch1(config-if)# exit

! Puerto hacia Switch2
Switch1(config)# interface fastEthernet 0/2
Switch1(config-if)# switchport mode trunk
Switch1(config-if)# exit

! Puerto hacia Switch3
Switch1(config)# interface fastEthernet 0/3
Switch1(config-if)# switchport mode trunk
Switch1(config-if)# exit

Switch1(config)# end
Switch1# write memory
```

### En Switch2:

```
Switch2> enable
Switch2# configure terminal

! Puerto hacia Switch1
Switch2(config)# interface fastEthernet 0/1
Switch2(config-if)# switchport mode trunk
Switch2(config-if)# exit

! Puerto hacia Switch3
Switch2(config)# interface fastEthernet 0/2
Switch2(config-if)# switchport mode trunk
Switch2(config-if)# exit

Switch2(config)# end
Switch2# write memory
```

### En Switch3:

```
Switch3> enable
Switch3# configure terminal

! Puerto hacia Switch1
Switch3(config)# interface fastEthernet 0/1
Switch3(config-if)# switchport mode trunk
Switch3(config-if)# exit

! Puerto hacia Switch2
Switch3(config)# interface fastEthernet 0/2
Switch3(config-if)# switchport mode trunk
Switch3(config-if)# exit

Switch3(config)# end
Switch3# write memory
```

> 💡 **Verificación:** Usa `show interfaces trunk` para confirmar que los puertos están en modo trunk.

---

## 7. Paso 5 — Configurar las PCs con sus IPs

Haz clic en cada PC → pestaña **Desktop** → **IP Configuration**.

> **Importante:** Reemplaza `XX` con los dos últimos dígitos de tu número de carnet.

### PC01 — VLAN 10 (ADMON)

| Campo | Valor |
|-------|-------|
| IP Address | `172.15.XX.10` |
| Subnet Mask | `255.255.255.192` |
| Default Gateway | `172.15.XX.1` |

### PC02 — VLAN 20 (RRHH)

| Campo | Valor |
|-------|-------|
| IP Address | `172.15.XX.100` |
| Subnet Mask | `255.255.255.240` |
| Default Gateway | `172.15.XX.97` |

### PC03 — VLAN 30 (IT)

| Campo | Valor |
|-------|-------|
| IP Address | `172.15.XX.80` |
| Subnet Mask | `255.255.255.224` |
| Default Gateway | `172.15.XX.65` |

> 📸 **[IMAGEN 2 — Configuración IP de PC01]**
> *(Captura mostrando la ventana IP Configuration de PC01 con los valores ingresados)*

> 📸 **[IMAGEN 3 — Configuración IP de PC02]**
> *(Captura mostrando la ventana IP Configuration de PC02 con los valores ingresados)*

> 📸 **[IMAGEN 4 — Configuración IP de PC03]**
> *(Captura mostrando la ventana IP Configuration de PC03 con los valores ingresados)*

---

## 8. Paso 6 — Configurar el Router (Router-on-a-Stick)

Esta es la parte más importante. El router necesita **subinterfaces** — una por cada VLAN — sobre su única interfaz física. Cada subinterfaz tendrá la IP que corresponde al **Gateway** de su VLAN.

Haz clic en el Router ROAS → pestaña **CLI**.

```
Router> enable
Router# configure terminal
Router(config)# hostname ROAS

! Activar la interfaz física principal (sin IP, solo se enciende)
ROAS(config)# interface gigabitEthernet 0/0
ROAS(config-if)# no shutdown
ROAS(config-if)# exit

! Subinterfaz para VLAN 10 - ADMON
ROAS(config)# interface gigabitEthernet 0/0.10
ROAS(config-subif)# encapsulation dot1Q 10
ROAS(config-subif)# ip address 172.15.XX.1 255.255.255.192
ROAS(config-subif)# exit

! Subinterfaz para VLAN 20 - RRHH
ROAS(config)# interface gigabitEthernet 0/0.20
ROAS(config-subif)# encapsulation dot1Q 20
ROAS(config-subif)# ip address 172.15.XX.97 255.255.255.240
ROAS(config-subif)# exit

! Subinterfaz para VLAN 30 - IT
ROAS(config)# interface gigabitEthernet 0/0.30
ROAS(config-subif)# encapsulation dot1Q 30
ROAS(config-subif)# ip address 172.15.XX.65 255.255.255.224
ROAS(config-subif)# exit

ROAS(config)# end
ROAS# write memory
```

**¿Qué hace cada comando?**

| Comando | Explicación |
|---------|-------------|
| `interface gig0/0.10` | Crea una subinterfaz lógica `.10` sobre la interfaz física `gig0/0` |
| `encapsulation dot1Q 10` | Le dice al router que esta subinterfaz pertenece a la VLAN 10 (usando el estándar 802.1Q) |
| `ip address 172.15.XX.1 255.255.255.192` | Asigna la IP del Gateway para esa VLAN |
| `no shutdown` en la interfaz física | Activa la interfaz física (sin esto, nada funciona) |

> 💡 **Verificación:** Usa `show ip interface brief` para ver que todas las subinterfaces están **up/up**.

> 📸 **[IMAGEN 5 — Configuración del Router: subinterfaces]**
> *(Captura de la CLI del router mostrando los comandos de configuración de las subinterfaces)*

> 📸 **[IMAGEN 6 — show ip interface brief en el Router]**
> *(Captura mostrando que las subinterfaces están en estado up/up)*

---

## 9. Paso 7 — Verificar conectividad con Ping

Ahora comprobaremos que el Inter VLAN Routing funciona correctamente realizando pings entre PCs de **diferentes VLANs**.

### 9.1 Ping de PC01 a PC02 (VLAN 10 → VLAN 20)

En **PC01** → Desktop → **Command Prompt**:

```
C:\> ipconfig
```
*(Verifica que muestra la IP y gateway correctos)*

```
C:\> ping 172.15.XX.100
```
*(Reemplaza XX con tu carnet. Debes ver 4 respuestas exitosas)*

**Resultado esperado:**
```
Pinging 172.15.XX.100 with 32 bytes of data:
Reply from 172.15.XX.100: bytes=32 time<1ms TTL=127
Reply from 172.15.XX.100: bytes=32 time<1ms TTL=127
Reply from 172.15.XX.100: bytes=32 time<1ms TTL=127
Reply from 172.15.XX.100: bytes=32 time<1ms TTL=127

Ping statistics for 172.15.XX.100:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

> 📸 **[IMAGEN 7 — Ping exitoso de PC01 a PC02]**
> *(Captura mostrando primero el resultado de `ipconfig` en PC01 y luego el ping exitoso hacia 172.15.XX.100)*

---

### 9.2 Ping de PC02 a PC03 (VLAN 20 → VLAN 30)

En **PC02** → Desktop → **Command Prompt**:

```
C:\> ipconfig
```
*(Verifica que muestra la IP y gateway correctos)*

```
C:\> ping 172.15.XX.80
```
*(Reemplaza XX con tu carnet. Debes ver 4 respuestas exitosas)*

**Resultado esperado:**
```
Pinging 172.15.XX.80 with 32 bytes of data:
Reply from 172.15.XX.80: bytes=32 time<1ms TTL=127
Reply from 172.15.XX.80: bytes=32 time<1ms TTL=127
Reply from 172.15.XX.80: bytes=32 time<1ms TTL=127
Reply from 172.15.XX.80: bytes=32 time<1ms TTL=127

Ping statistics for 172.15.XX.80:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

> 📸 **[IMAGEN 8 — Ping exitoso de PC02 a PC03]**
> *(Captura mostrando primero el resultado de `ipconfig` en PC02 y luego el ping exitoso hacia 172.15.XX.80)*

---

## 10. Entregables

Según la rúbrica de calificación, debes entregar:

| Criterio | Descripción | Puntos |
|----------|-------------|--------|
| **Topología 1** | Captura de la topología completa en Packet Tracer con todos los dispositivos correctamente conectados | 20 pts |
| **Conexión InterVLAN 1** | Captura de `ipconfig` en PC01 + ping exitoso de PC01 hacia PC02 | 40 pts |
| **Conexión InterVLAN 2** | Captura de `ipconfig` en PC02 + ping exitoso de PC02 hacia PC03 | 40 pts |
| **Total** | | **100 pts (= 1 pt)** |

### Resumen de capturas necesarias:

- [ ] 📸 Imagen 1: Topología completa en Packet Tracer
- [ ] 📸 Imagen 2: IP Configuration de PC01
- [ ] 📸 Imagen 3: IP Configuration de PC02
- [ ] 📸 Imagen 4: IP Configuration de PC03
- [ ] 📸 Imagen 5: CLI del Router con configuración de subinterfaces
- [ ] 📸 Imagen 6: `show ip interface brief` en el Router
- [ ] 📸 Imagen 7: `ipconfig` + ping PC01 → PC02 (exitoso)
- [ ] 📸 Imagen 8: `ipconfig` + ping PC02 → PC03 (exitoso)

---

## Solución de Problemas (Troubleshooting)

Si los pings no funcionan, revisa lo siguiente en orden:

1. **¿El primer ping falla pero los siguientes sí?** — Normal. El primer ping activa el ARP. No hay problema.
2. **¿Todos los pings fallan?**
   - Verifica que la interfaz física del router esté en `no shutdown`.
   - Verifica que `encapsulation dot1Q` tenga el número de VLAN correcto.
   - Verifica que el gateway en cada PC coincida exactamente con la IP de la subinterfaz del router.
3. **¿Los switches muestran puertos en naranja?** — Espera unos segundos, el protocolo STP converge. Los puertos deben ponerse en verde.
4. **¿El comando `show vlan brief` no muestra las VLANs?** — Repite el proceso de creación de VLANs en ese switch.
5. **¿El trunk no funciona?** — Verifica con `show interfaces trunk` que el puerto aparece en la lista.

---

*Documento preparado para la Actividad Práctica 6 — Redes de Computadoras 1 — ECYS, FIUSAC*