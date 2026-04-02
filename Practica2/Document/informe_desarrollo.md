# Informe de Desarrollo — Práctica 2: Red de Hospitales Metropolitano
**Curso:** Redes de Computadoras 1  
**Universidad:** San Carlos de Guatemala — Facultad de Ingeniería  
**Carnet:** 20238204  
**Fecha:** 2026

---

## 1. Descripción General de la Implementación

Esta práctica consiste en el diseño e implementación de una red LAN segmentada para 6 hospitales metropolitanos de Guatemala, utilizando Cisco Packet Tracer. La red demuestra **4 tecnologías avanzadas de switching**:

1. **VLANs (Virtual Local Area Networks)** — Segmentación lógica de la red
2. **VTP (VLAN Trunking Protocol)** — Administración centralizada de VLANs
3. **STP Rapid PVST+ (Spanning Tree Protocol)** — Prevención de bucles y redundancia
4. **EtherChannel PAgP** — Agregación de enlaces para mayor ancho de banda

Cada tecnología resuelve un problema específico en redes empresariales.

---

## 2. Conceptos Fundamentales

### 2.1 ¿QUÉ ES UNA VLAN?

**Definición:** Una VLAN es una red lógica que agrupa dispositivos de forma independiente de su ubicación física. Permite comunicación entre equipos en el mismo dominio de broadcast aunque estén en diferentes switches físicos.

**Sin VLANs:** Todos los dispositivos conectados a los switches comparten el MISMO domino de broadcast. Un broadcast de una PC llega a TODAS las demás.

**Con VLANs:** Los broadcasts se aíslan por VLAN. Un broadcast en VLAN 14 (Quirófanos) **nunca** llega a dispositivos en VLAN 24 (UCI).

**Ejemplo en esta práctica:**
```
VLAN 14 (Quirófanos):    192.168.4.0/25  → Médicos, cirujanos, equipos de quirófano
VLAN 24 (UCI):           192.168.4.128/26 → Monitores, equipos de monitoreo  
VLAN 34 (Administración): 192.168.4.192/26 → Personal administrativo
VLAN 44 (Urgencias):     192.168.5.0/25  → Triaje, registros de pacientes
VLAN 99 (Nativa):        192.168.5.128/28 → Administración de switches (SVI)
VLAN 999 (Blackhole):    Sin IP          → Seguridad: puertos no usados
```

**Ventajas de VLANs:**
- ✅ **Seguridad:** El tráfico de UCI no se mezcla con administrativo
- ✅ **Eficiencia:** Reducen el tamaño del dominio de broadcast
- ✅ **Administración:** Fácil añadir/mover dispositivos sin cambiar cableado
- ✅ **Control de QoS:** Se pueden priorizar VLANs críticas (UCI) sobre administrativas

---

### 2.2 ¿QUÉ ES VTP?

**Definición:** VTP es un protocolo que **propaga automáticamente la configuración de VLANs** desde un servidor central a todos los clientes en el dominio.

**Sin VTP:** Tendrías que crear manualmente cada VLAN en cada uno de los 6 switches (SW-CORE, SW-H1, SW-H2, SW-H3, SW-H4, SW-H5). Eso son 6 × 6 VLANs = 36 comandos manuales.

**Con VTP:** Creas las VLANs UNA SOLA VEZ en SW-CORE y automáticamente aparecen en todos los switches clientes.

**Modos VTP:**
- 🔴 **Server:** Crea y modifica VLANs, las propaga a otros switches
- 🟢 **Client:** Recibe VLANs del servidor, no puede crearlas localmente
- 🟡 **Transparent:** Ignora VTP pero puede retransmitir mensajes VTP

**En esta práctica:**
- SW-CORE = **Server** (dominio `202308204`, contraseña `area1`, versión 2)
- SW-H1 a SW-H5 = **Clientes** (reciben todas las VLANs del core)
- SW-H6 = Transparente (solo tiene 1 hub, no necesita VTP)

**Requisito crítico:** VTP solo funciona a través de **puertos troncales**. Sin trunk configurado, no hay propagación.

---

### 2.3 ¿QUÉ ES STP RAPID PVST+?

**El Problema:** Imagina que SW-CORE conecta a SW-H1 por DOS caminos diferentes:
```
SW-CORE ──[Fut1]──┐
                  ├─→ SW-H1
SW-CORE ──[Fut2]──┘
```

Sin STP, un paquete que sale del core puede entrar por Fut1, llegar a H1, y H1 lo reenviará tanto a Fut1 como a Fut2. El paquete regresa al core por Fut2, el core lo vuelve a reenviar... **¡BUCLE INFINITO!** Esto causa:
- 🔴 Incremento exponencial del tráfico
- 🔴 Consumo total del acceso
- 🔴 Caída de la red

**La Solución — STP Rapid PVST+:**
- Detecta automaticamente todos los bucles potenciales
- **Bloquea puertos redundantes** para eliminar el bucle (solo mantiene UN camino activo)
- Si el puerto activo falla, **automáticamente activa el redundante** (reconvergencia)
- **PVST+ = Per-VLAN Spanning Tree Plus:** Hace esto **por cada VLAN** (6 árboles diferentes)

**Rapid = Rápido:** Converge en <2 segundos vs 50 segundos del STP clásico (802.1D)

**En esta práctica:**
```
Root Bridge (SW-CORE)
   ↓
   ├─→ SW-H1 [Designated ports activos]
   ├─→ SW-H2 [Designated ports activos]
   ├─→ SW-H3 [Designated ports activos]
   ├─→ SW-H4 [Designated ports activos]
   └─→ SW-H5 [Designated ports activos]
```

El Root Bridge es SW-CORE porque tiene la prioridad más baja (se configuró explícitamente con `root primary`).

---

### 2.4 ¿QUÉ ES ETHERCHANNEL?

**El Problema:** El enlace entre SW-CORE y SW-H1 es un cuello de botella:
- Un solo puerto GigabitEthernet = 1 Gbps de capacidad
- Si 50 hospitales envían datos simultáneamente, se congestiona

**La Solución — EtherChannel:**
Agrupa **3 enlaces físicos** en UNA interfaz lógica (**Port-Channel**) que:
- ✅ **Suma el ancho de banda:** 3 × 1Gbps = 3 Gbps
- ✅ **Proporciona redundancia:** Si 1 enlace falla, los otros 2 siguen funcionando
- ✅ **Balancea el tráfico:** Los paquetes se distribuyen entre los 3 enlaces

**Protocolos de Negociación:**
- 🔴 **PAgP (Port Aggregation Protocol):** Propietario de Cisco
  - **Desirable:** Inicia la negociación (SW-CORE)
  - **Auto:** Responde a la negociación (SW-H1)
- 🟢 **LACP (Link Aggregation Control Protocol):** Estándar IEEE 802.3ad
  - **Active:** Inicia
  - **Passive:** Responde

**En esta práctica:** PAgP porque usamos switches Cisco.

```
Configuración Física:
SW-CORE                          SW-H1
├─ Gi0/1 ─┐                   ┌─ Gi0/1
├─ Gi0/2 ─┼─── Cables ───────┼─ Gi0/2
└─ Fa0/24 ┘                   └─ Fa0/24

Resultado Lógico (lo que "ve" el SW-CORE):
┌─ Port-Channel1 ─ 3 Gbps ─→ SW-H1
```

**Comandos clave:**
```
interface range Gi0/1-2, Fa0/24    ← Seleccionar 3 interfaces
channel-group 1 mode desirable     ← Activar PAgP en DESIRABLE (inicia negociación)
```

---

## 3. Procesos de Implementación Detallados

### Fase 1 — Diseño y Planificación

#### 3.1.1 Decisión de Topología: Estrella Extendida

**Opciones evaluadas:**

| Topología | Ventajas | Desventajas | Elegida |
|-----------|----------|-------------|--------|
| **En Anillo** | Redundancia nativa | Compleja, STP complejo, difícil agregar switches | ❌ |
| **En Malla Completa** | Máxima redundancia | Cableado excesivo, costoso | ❌ |
| **Estrella Extendida** | Fácil de administrar, ideal para EtherChannel, escalable | Punto único de fallo en core (se mitiga con redundancia) | ✅ |
| **Árbol** | Simple, escalable | Poco redundante | ❌ Parcial |

**Elegida: Estrella Extendida**

```
                    ┌─ SW-CORE ─┐  ← Root Bridge
                    │   (CORE)   │
      ┌─────────────┼────────────┼─────────────────┐
      │             │            │                 │
     SW-H1         SW-H2        SW-H3            SW-H4
     (H1)          (H2)         (H3)             (H4)
      ├─ HUB-A      ├─ HUB-A    ├─ HUB-A        ├─ HUB-A
      ├─ HUB-B      ├─ HUB-B    ├─ HUB-B        ├─ HUB-B
      └─ HUB-C      └─ HUB-C    └─ HUB-C        └─ HUB-B
        │ │ │         │ │         │ │ │           │ │
        PCs PCs       PCs PCs      PCs PCs         PCs PCs
```

**Razones de esta decisión:**
1. **Facilita EtherChannel:** Los 3 enlaces entre core y hospital van por una ruta coherente
2. **Simplifica administración:** Todas las VLANs se crean en core y se propagan
3. **Root Bridge centralizado:** STP es sencillo con el core como raíz
4. **Escalabilidad:** Agregar un nuevo hospital = conectarlo al core

---

#### 3.1.2 Definición de VLANs por Área Funcional

**Análisis de funciones del hospital:**

| Área | Tipo de Tráfico | Criticidad | Usuarios Estimados | VLAN Asignada | Máscara |
|------|-----------------|-----------|-------------------|---------------|---------|
| Quirófanos | Equipos médicos, órdenes de cirugía | **MUY CRÍTICA** | 80 usuarios | 14 | /25 (126 hosts) |
| UCI | Monitores, vital signs, alertas | **MUY CRÍTICA** | 62 usuarios | 24 | /26 (62 hosts) |
| Administración | Correos, reportes, RRHH | Normal | 62 usuarios | 34 | /26 (62 hosts) |
| Urgencias | Triaje, registros, derivaciones | **CRÍTICA** | 126 usuarios | 44 | /25 (126 hosts) |
| Nativa (Management) | Administración de switches | Interna | 14 IPs | 99 | /28 (14 hosts) |
| Blackhole | Seguridad (puertos deshabilitados) | Seguridad | 0 | 999 | N/A |

**Fórmula VLAN ID usada:** `VLAN_ID = [Número de Área] + [X]` donde X = 4 (del carnet)
- Área 1 (Quirófanos): 1 + 4 = **VLAN 14** ✅
- Área 2 (UCI): 2 + 4 = **VLAN 24** ✅
- Área 3 (Admin): 3 + 4 = **VLAN 34** ✅
- Área 4 (Urgencias): 4 + 4 = **VLAN 44** ✅

---

### Fase 2 — Construcción de Topología en Packet Tracer

#### 3.2.1 Pasos de Construcción

```
PASO 1: Agregar Switches Core
   → 1 × Cisco Catalyst 2960 → Renombrar a SW-CORE
   → Este será el Root Bridge, servidor VTP y punto central

PASO 2: Agregar Switches de Hospital
   → 5 × Cisco Catalyst 2960 → Renombrar a SW-H1, SW-H2, ... SW-H5
   → Cada uno es punto de acceso a un hospital

PASO 3: Agregar Hubs de Acceso
   Para crear EXACTAMENTE los dominios de colisión requeridos:
   
   Hospital Roosevelt (H1): 3 dominios → 3 Hubs (HUB-H1-A, HUB-H1-B, HUB-H1-C)
   Hospital SJDIOS (H2): 2 dominios → 2 Hubs (HUB-H2-A, HUB-H2-B)
   Hospital IGSS (H3): 3 dominios → 3 Hubs (HUB-H3-A, HUB-H3-B, HUB-H3-C)
   Hospital Ortopedia (H4): 2 dominios → 2 Hubs (HUB-H4-A, HUB-H4-B)
   Hospital Infantil (H5): 2 dominios → 2 Hubs (HUB-H5-A, HUB-H5-B)
   Hospital Militar (H6): 1 dominio → 1 Hub (HUB-H6)
   
   TOTAL: 14 Hubs

PASO 4: Agregar PCs
   Calcular distribución:
   - Dominio con 10 usuarios → 10 PCs
   - Dominio con 8 usuarios → 8 PCs
   Total: ~150 PCs
   
   Etiquetar: PC-[VLAN]-[Hospital]-[Número]
   Ejemplo: PC-Q-H1-1 (PC en VLAN Quirófanos, Hospital 1, PC #1)

PASO 5: Conectar Físicamente
   
   a) Apilamiento horizontal:
      SW-CORE ─ Gi0/1 ──→ Gi0/1 ─ SW-H1
      SW-CORE ─ Gi0/2 ──→ Gi0/2 ─ SW-H1
      SW-CORE ─ Fa0/24 ──→ Fa0/24 ─ SW-H1
      (Estos 3 enlaces se agruparán en Port-Channel1)
   
   b) Conectar SW-H1 a su Hub de Acceso:
      SW-H1 ─ Fa0/1 ──→ Fa0/1 ─ HUB-H1-A
      SW-H1 ─ Fa0/6 ──→ Fa0/1 ─ HUB-H1-B
      SW-H1 ─ Fa0/11 ─→ Fa0/1 ─ HUB-H1-C
   
   c) Conectar PCs a Hubs:
      HUB-H1-A ─ Fa0/2 ──→ Fa0 ─ PC-Q-H1-1
      HUB-H1-A ─ Fa0/3 ──→ Fa0 ─ PC-Q-H1-2
      ... y así sucesivamente
   
   d) Repetir para H2, H3, H4, H5 (H6 solo tiene 1 hub)
```

#### 3.2.2 Problema Encontrado: Falta de Puertos

**Descripción:** El switch Cisco 2960 tiene:
- 24 puertos FastEthernet (1 Gbps cada uno)
- 2 puertos GigabitEthernet (1 Gbps cada uno)
- **Total: 26 puertos**

Para EtherChannel necesitamos 3 puertos GigabitEthernet (mínimo de mejor velocidad). Pero el switch solo tiene 2.

**Soluciones evaluadas:**
1. ❌ Usar solo GigabitEthernet existentes (solo 2 enlaces, no es suficiente)
2. ✅ Usar GigabitEthernet + 1 FastEthernet como tercer enlace
3. ❌ Cambiar a switch más potente (no disponible en Packet Tracer)

**Decisión:** **Opción 2 — Usar Gi0/1, Gi0/2 y Fa0/24**

Aunque Fa0/24 tiene la misma velocidad que otros FastEthernet, el channel-group acepta mezclar velocidades (no es recomendado en producción, pero válido en práctica educativa).

---

### Fase 3 — Configuración de VLANs y VTP

#### 3.3.1 Paso 1: Configurar VTP Server en SW-CORE

```Cisco IOS
SW-CORE# configure terminal
SW-CORE(config)# vtp mode server             ← Cambiar a modo SERVIDOR
SW-CORE(config)# vtp domain 202308204        ← Nombre del dominio VTP
SW-CORE(config)# vtp password area1          ← Contraseña compartida (encriptada)
SW-CORE(config)# vtp version 2               ← Versión mejorada (v1 es obsoleta)
SW-CORE(config)# end
```

**Qué hace cada comando:**

| Comando | Función |
|---------|---------|
| `vtp mode server` | Activa el servidor VTP. Este switch PUEDE crear/modificar/propagar VLANs |
| `vtp domain 202308204` | Define nombre del dominio. Los switches fuera de este dominio ignoran los cambios |
| `vtp password area1` | Contraseña compartida (debe ser igual en TODOS los switches del dominio) |
| `vtp version 2` | **Cambios en V2 vs V1:** V2 soporta VLANs extendidas (1006-4094) |
| `end` | Sale del modo de configuración |

**Verificación:**
```
SW-CORE# show vtp status
VTP Version capable      : 1 to 2
VTP version running      : 2
VTP Domain Name          : 202308204
VTP Pruning Mode         : Disabled
VTP Traps Generation     : Enabled
VTP Password (crypted)   : $1$LS$+sXh7XZ6K.J1K. ← Encriptada
Configuration last modified by 0.0.0.0 at 3-1-93 00:04:07
Local updater ID is 0000.2AB7.9B40
VTP Operating Mode       : Server
```

**Matriz de Configuración Revision:** El server cuenta cambios. Si dos servers compiten, el con mayor revision gana (puede causar conflictos).

---

#### 3.3.2 Paso 2: Crear las VLANs en SW-CORE

```Cisco IOS
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

**Qué sucede aquí:**
1. Se crean 6 VLANs con IDs únicos
2. Cada VLAN recibe un nombre descriptivo (recomendado pero no obligatorio)
3. El nombre ayuda a otros administradores a entender el propósito
4. **AUTOMÁTICAMENTE** el servidor VTP v2 incrementa su Configuration Revision
5. En los próximos 5 segundos, llegará un mensaje VTP a los clientes

**Verificación:**
```
SW-CORE# show vlan brief
VLAN Name                             Status    Ports
---- -------------------------------- --------- ---
1    default                          active    Fa0/1-24
14   Quirofanos                       active    
24   UCI_Cuidados_Intensivos          active    
34   Administracion                   active    
44   Urgencias_Emergencias            active    
99   VLAN_Nativa_Troncal              active    
999  Blackhole_No_Usar                active    
```

---

#### 3.3.3 Paso 3: Configurar VTP Clientes en SW-H1, SW-H2, etc.

```Cisco IOS
SW-H1# configure terminal
SW-H1(config)# vtp mode client              ← Cliente: NO puede crear VLANs
SW-H1(config)# vtp domain 202308204         ← DEBE coincidir con server
SW-H1(config)# vtp password area1           ← DEBE coincidir con server
SW-H1(config)# end
```

**Qué sucede:**
1. El switch entra en modo CLIENTE
2. Como cliente, **ignora tráfico que no sea del dominio 202308204**
3. Recibe anuncios VTP del server a través de puertos troncales
4. **Automáticamente adopta todas las VLANs del server**
5. **NO puede crear VLANs locales** (comando vlan fallará)

**Ventaja crítica:** Si el server crea 100 VLANs, los 5 clientes las reciben todas automáticamente. Sin VTP, tomarías 5 × 100 = 500 comandos.

---

#### 3.3.4 Problema: VLANs No Se Propagan

**Síntoma:** 
```
SW-H1# show vlan brief
VLAN Name                             Status    Ports
---- -------------------------------- --------- ---
1    default                          active    Fa0/1-24
```

Solo la VLAN 1 (default). Las VLANs 14, 24, 34, 44, 99 no aparecen.

**Causas evaluadas:**

| # | Causa | Síntoma | Solución |
|---|-------|--------|----------|
| 1 | Dominio VTP incorrecto | `show vtp status` != 202308204 | Cambiar `vtp domain 202308204` |
| 2 | Contraseña incorrecta | `show vtp status` muestra (crypted) diferente | Cambiar `vtp password area1` |
| 3 | **Puerto troncal no configurado** ← ESTA ERA | `show interfaces trunk` sin interfaces activas | Crear trunk con `switchport mode trunk` |
| 4 | Revisión de configuración baja | Server nunca envía | Incrementar revisión en server |

**Razón fundamental:** **VTP solo se propaga a través de puertos troncales.**

¿Por qué? Porque VTP envía mensajes en VLAN 1 nativa con MAC de multicast `01:00:0c:cc:cc:cc`. Si el puerto es access (asignado a una sola VLAN como VLAN 14), ese multicast nunca lo ve.

**Solución aplicada:**

```Cisco IOS
SW-CORE(config)# interface GigabitEthernet0/1
SW-CORE(config-if)# switchport mode trunk        ← SU PRIMERO AQUÍ
SW-CORE(config-if)# switchport trunk native vlan 99
SW-CORE(config-if)# switchport trunk allowed vlan 14,24,34,44,99
SW-CORE(config-if)# no shutdown
SW-CORE(config-if)# exit

! ← Repetir Gi0/2 y Fa0/24

SW-H1(config)#interface GigabitEthernet0/1
SW-H1(config-if)# switchport mode trunk        ← Y LUEGO AQUÍ (ANTES de VTP)
SW-H1(config-if)# switchport trunk native vlan 99
SW-H1(config-if)# switchport trunk allowed vlan 14,24,34,44,99
SW-H1(config-if)# no shutdown
SW-H1(config-if)# exit
```

**Después de esto,** configurar VTP y las VLANs aparecen en SW-H1 automáticamente.

---

### Fase 4 — Configuración de EtherChannel

#### 3.4.1 ¿Por Qué Necesitamos EtherChannel?

**Problema de banda ancha:**
```
Archivo de 300 MB para respaldar datos de pacientes:
- Por 1 enlace Gi0/1: 300 MB ÷ 1 Gbps = 0.3 segundos
- Pero si 3 hospitales respaldan SIMULTÁNEAMENTE:
  - Hospital 1: 300 MB
  - Hospital 2: 300 MB
  - Hospital 3: 300 MB
  Total: 900 MB por 1 Gbps = 0.9 segundos POR SECUENCIA
  
Colas de cola de espera se forman exponencialmente.
```

**Con EtherChannel:**
- 3 enlaces = 3 Gbps
- 900 MB ÷ 3 Gbps = 0.3 segundos (sin espera)
- **3 × más rápido** para transferencias paralelas

---

#### 3.4.2 Paso 1: Configurar PAgP en SW-CORE (Desirable)

```Cisco IOS
SW-CORE# configure terminal
SW-CORE(config)# interface range GigabitEthernet0/1 - 2, FastEthernet0/24
SW-CORE(config-if-range)# channel-group 1 mode desirable
SW-CORE(config-if-range)# exit
```

**Explicar `interface range`:**
- **GigabitEthernet0/1 - 2**: Selecciona Gi0/1 Y Gi0/2 (rango)
- **FastEthernet0/24**: Añade específicamente Fa0/24
- **Separadas por comas** = Todas se aplica el comando simultáneamente

**Explicar `channel-group 1 mode desirable`:**
- **channel-group 1**: ID del canal (agrupa estas 3 interfaces)
- **mode desirable**: PAgP en modo **DESIRABLE**
  - → "Estoy listo para agrupar, ¿y tú?"
  - → Inicia negociación
  - → Espera respuesta del otro extremo

---

#### 3.4.3 Paso 2: Configurar PAgP en SW-H1 (Auto)

```Cisco IOS
SW-H1(config)# interface range GigabitEthernet0/1 - 2, FastEthernet0/24
SW-H1(config-if-range)# channel-group 1 mode auto
SW-H1(config-if-range)# exit
```

**mode auto** en SW-H1:
- → "Si alguien quiere agrupar, aceptaré"
- → Responde a DESIRABLE (pero no inicia)
- → Complemento perfecto para el DESIRABLE del core

**Tabla de compatibilidades E/A:**

| SW-CORE | SW-H1 | Resultado |
|---------|-------|-----------|
| desirable | auto | ✅ Crea Port-Channel |
| desirable | desirable | ✅ Crea Port-Channel |
| auto | auto | ❌ NO crea (ambos esperan) |
| passive | passive | ❌ NO crea (ambos esperan) |
| desirable | on | ⚠️ Funciona pero sin protocolo |

---

#### 3.4.4 Paso 3: Configurar el Port-Channel como Troncal

```Cisco IOS
SW-CORE(config)# interface port-channel 1
SW-CORE(config-if)# switchport mode trunk                 ← Por qué trunk?
SW-CORE(config-if)# switchport trunk native vlan 99      ← VLAN sin etiquetado
SW-CORE(config-if)# switchport trunk allowed vlan 14,24,34,44,99
SW-CORE(config-if)# exit
```

**¿Por qué es troncal?**
```
Entre SW-CORE y SW-H1 fluye TRÁFICO DE MÚLTIPLES VLANs:
- VLAN 14 (Quirófanos) de Hospital 1
- VLAN 24 (UCI) de Hospital 1
- VLAN 34 (Admin) de Hospital 1
- VLAN 44 (Urgencias) de Hospital 1

Si el puerto fuera ACCESS (solo VLAN 14), 
las demás VLANs NO podrían pasar.

Por eso debe ser TRUNK: Para que TODAS las VLANs atraviesen.
```

**Configuración de la VLAN Nativa:**

```
VLAN Nativa (VLAN 99) = VLAN sin etiquetado 802.1Q

Paquete en VLAN 14 cuando sale por el trunk:
[Etiqueta 802.1Q: VLAN 14] [Datos]

Paquete en VLAN 99 cuando sale por el trunk:
[Datos]  ← Sin etiqueta

¿Por qué? Porque equipos antiguos que no entienden 802.1Q
pueden leer VLAN 99. Device antiguo ignora VLANs,
ve un marco sin etiqueta = VLAN 1 originalmente,
pero en Cisco se cambia a VLAN 99 (más seguro).
```

**VLANs permitidas:**
```
allowed vlan 14,24,34,44,99

Paquete de VLAN 14 → Sale (está en allowed)
Paquete de VLAN 999 → Bloqueado (NO está en allowed)
```

---

#### 3.4.5 Problema: Port-Channel en Estado "SD"

**Síntoma:**
```
SW-CORE# show etherchannel summary
Port-Channel1 ↑ SU Gr ─ PAgP ─ 3 GigabitEthernet0/1(s)
                       ↑ SD Gr ─ Suspended

No está operativo. El estado "SD" = Suspended (suspendido).
```

**Estados posibles:**

| Estado | Significado | Significa |
|--------|-------------|-----------|
| **P** | Bundled | Activo en el bundle |
| **D** | Down | Interface física caída |
| **S** | Suspended | Suspendido por mismatch |
| **U** | In Use | En la negociación |
| **H** | Hot Standby | Reserva activa |

**Por qué falla:**

1. **Causa 1: VLANs permitidas inconsistentes**
   ```
   SW-CORE: allowed vlan 14,24,34,44,99
   SW-H1:   allowed vlan 14,24,34,44,44,99  ← Duplicado VLAN 44
   
   MISMATCH → Suspensión automática
   ```

2. **Causa 2: Modo trunk incorrecto**
   ```
   SW-CORE: switchport mode trunk
   SW-H1:   switchport mode access  ← INCORRECTO
   
   MISMATCH → Suspensión
   ```

3. **Causa 3: VLAN nativa diferente**
   ```
   SW-CORE: native vlan 99
   SW-H1:   native vlan 1  ← Diferente
   
   MISMATCH → Suspensión
   ```

**Solución aplicada:**
```Cisco IOS
! Verificar primero qué está mal:
SW-CORE# show etherchannel 1 detail

! Luego, fomentar consistencia:
interface range Gi0/1-2, Fa0/24
no switchport trunk allowed vlan  ← Limpiar
switchport trunk allowed vlan 14,24,34,44,99  ← Resetear

! Repetir en SW-H1 exactamente igual

! Después de 30 segundos, revisar de nuevo:
show etherchannel summary  ← Debe estar "SU"
```

---

### Fase 5 — Configuración de STP Rapid PVST+

#### 3.5.1 ¿Por Qué STP es Crítico?

**Escenario sin STP:**

```
                    ┌──────────────────┐
                    │    Internet       │
                    └──────────┬────────┘
                              ││
                    ┌────────────────────┐
                    │    SW-CORE         │
                    └─┬────────────┬─────┘
                      │            │
                    [Back]      [Principal]
                      │            │
                    ┌─┴───────────┬─┘
                    │    SW-H1    │
                    └──────────────┘

Paquete llega a SW-CORE por Principal → Se etiqueta con timestamp T1
SW-H1 recibe el paquete
SW-H1 lo reenvía al Back Link (pensando que es un broadcast)
SW-CORE recibe de nuevo → T1 + retardo pequeño
SW-CORE lo reenvía NUEVAMENTE a SW-H1
... Bucle infinito

Consumo de CPU: 100% en segundos
Red: Completamente caída
```

**Con STP:**
```
SW-CORE como Root:
├─ Principal: DESIGNATED port (Activo)
└─ Back: BLOCKED port (Inactivo)

Paquete por Principal solo
Si Principal falla:
├─ Principal: DOWN
├─ Back: LISTENING → LEARNING → FORWARDING (10-15 segundos con Rapid PVST+)

Reconvergencia automática, sin intervención humana.
```

---

#### 3.5.2 Configurar modo Rapid PVST+

```Cisco IOS
! EN TODOS LOS SWITCHES (SW-CORE, SW-H1, ...  SW-H5)

SW-CORE# configure terminal
SW-CORE(config)# spanning-tree mode rapid-pvst
SW-CORE(config)# spanning-tree vlan 14 root primary
SW-CORE(config)# spanning-tree vlan 24 root primary
SW-CORE(config)# spanning-tree vlan 34 root primary
SW-CORE(config)# spanning-tree vlan 44 root primary
SW-CORE(config)# spanning-tree vlan 99 root primary
SW-CORE(config)# end
```

**Explicar cada comando:**

| Comando | Qué Hace |
|---------|----------|
| `spanning-tree mode rapid-pvst` | Activa STP mejorado (Rapid) de Cisco |
| `spanning-tree vlan 14 root primary` | **Para VLAN 14 específicamente**, hacer que este switch sea el Root Bridge colocando su prioridad en 4096 (la mínima automática con versión PVST) |

**¿Por qué "root primary" y no "root priority 4096"?**
```
root primary   → "Quiero ser el root, ajusta automáticamente"
root secondary → "Quiero ser el root si el primary falla"
root priority N → Establecer prioridad específica

root primary es más seguro porque:
1. Calcula automáticamente
2. Si agregas más VLANs después, se ajusta solo
```

**Efecto en prioridades:**

Antes de comando `root primary`:
```
SW-CORE priority = 32768 (default)
SW-H1 priority = 32768 (default)

¿Quién es root? No es claro, podría ser cualquiera.
```

Después de `root primary` en VLAN 14:
```
SW-CORE priority = 4096 (ajustado por Cisco para VLAN 14)
SW-H1 priority = 32768 (sigue el default)

Resultado: SW-CORE es definitivamente el root de VLAN 14
porque 4096 < 32768 (STP elige la prioridad MÁS BAJA)
```

---

#### 3.5.3 Verificación de STP

```Cisco IOS
SW-CORE# show spanning-tree vlan 14

VLAN0014
  Spanning tree enabled protocol rstp
  Root ID    Priority    4108      ← 4096 + 12 (VLAN 14)
             Address     ****.****.****.
             This bridge is the root  ← ★ Confirmación de root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
  
  Bridge ID  Priority    4108
             Address     ****.****.****.
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20  sec
  
  Interface           Role Sts Cost      Prio.Nbr Type
  ----------          ---- --- --------- -------- ----
  Po1                 Desg FWD 3         128.27   P2p
  Fa0/1               Desg FWD 19        128.1    P2p
  Fa0/2               Desg FWD 19        128.2    P2p
                                         ↑
                                    DESIGNATED = Envía
                                    FORWARDING = Estado activo
```

**Interpretación:**
- ✅ "This bridge is the root" → SW-CORE es el RT de VLAN 14
- ✅ "Interface Po1 Desg FWD" → Port-Channel1 envía (Designated) y es Forwarding
- ✅ "Interface Fa0/1 Desg FWD" → FastEthernet0/1 también activo

---

### Fase 6 — Asignación de IPs y Pruebas

#### 3.6.1 Subnetting Explicado

```
Red base del carnet XX=4:  192.168.4.0/24
Máscara: 255.255.255.0 = 256 direcciones posibles

Necesidades:
VLAN 14: 126 hosts → /25 (128 direcciones)
VLAN 24: 62 hosts → /26 (64 direcciones)
VLAN 34: 62 hosts → /26 (64 direcciones)

¿Cómo dividir 256 en /25 y /26?

VLAN 14: 192.168.4.0/25 → 192.168.4.0 a 192.168.4.127
VLAN 24: 192.168.4.128/26 → 192.168.4.128 a 192.168.4.191
  ├─ Usables: 192.168.4.130 a 192.168.4.190 (62)
  └─ Gateway: 192.168.4.129
VLAN 34: 192.168.4.192/26 → 192.168.4.192 a 192.168.4.255
  ├─ Usables: 192.168.4.194 a 192.168.4.254 (62)
  └─ Gateway: 192.168.4.193
VLAN 44: 192.168.5.0/25
VLAN 99: 192.168.5.128/28 → 192.168.5.128 a 192.168.5.143

Suma: 126 + 62 + 62 + 126 + 14 = 390 direcciones
(Se desperdician algunas, pero es normal)
```

#### 3.6.2 Configuración de PC

```
Ejemplo: PC-Q-H1-1 (Quirófanos, Hospital 1, PC 1)

IP Address:  192.168.4.2
Subnet Mask: 255.255.255.128
Default Gateway: 192.168.4.1
DNS: 8.8.8.8 (ejemplo)

¿Por qué Gateway 192.168.4.1?
Porque sin router, la Gateway es la dirección de la VLAN en SVI del switch.

En SW-CORE (no se configura, es nativo pero se entiende):
┌─ interface vlan 14
├─ ip address 192.168.4.1 255.255.255.128
└─ (Esta es la ruta por defecto para la VLAN 14)
```

---

#### 3.6.3 Pruebas de Conectividad

**Prueba 1: Ping dentro de la misma VLAN**

```Cisco IOS
PC-Q-H1-1> ping 192.168.4.10  (PC-Q-H2-1 en Hospital 2)

Paso 1: ARP Request
  "¿Quién tiene 192.168.4.10?"
  Destino MAC: FF:FF:FF:FF:FF:FF (broadcast)
  
Envío dentro de VLAN 14:
PC-Q-H1-1 → HUB-H1-A → todos los peers del hub
Pero PC-Q-H2-1 está en otro hospital, otro hub
¿Llega el ARP?

Sí, porque:
1. HUB-H1-A envía a SW-H1 (el hub está conectado al switch)
2. SW-H1 recibe, ve que es broadcast de VLAN 14
3. SW-H1 reenvía por puerto troncal (Po1) al SW-CORE
4. SW-CORE recibe en VLAN 14, reenvía a SW-H2
5. SW-H2 envía a HUB-H2-A
6. PC-Q-H2-1 recibe el ARP Request
7. PC-Q-H2-1 responde con ARP Reply (unicast)

RESULTADO: ✅ ARP encontrada
RESULTADO: ✅ Ping exitoso (ICMP Echo Request/Reply)
```

**Prueba 2: Ping entre diferentes VLANs (sin router)**

```Cisco IOS
PC-Q-H1-1 (VLAN 14, 192.168.4.2) > ping 192.168.4.130 (VLAN 24)

Paso 1: ARP Request
  PC-Q-H1-1: "¿Quién tiene 192.168.4.130?"
  Envía broadcast en VLAN 14
  
Paso 2: El broadcast NUNCA llega a VLAN 24
  ¿Por qué? VLANs son dominios de broadcast SEPARADOS
  Un broadcast en VLAN 14 está etiquetado [802.1Q:VLAN14]
  Los dispositivos en VLAN 24 filtan esa etiqueta y lo ignoran
  
Paso 3: El ARP falla (timeout)
Paso 4: Ping falla

RESULTADO: ❌ Ping falla (comportamiento correcto/esperado)

Nota: Para comunicación entre VLANs se requiere un ROUTER
que tenga interfaces SVI en ambas VLANs.
No hay router en esta práctica, de ahí el aislamiento.
```

---

## 4. Problemas Encontrados y Soluciones

| # | Problema | Síntoma | Causa Raíz | Solución |
|----|-----------|---------|-----------|----------|
| 1 | **VLANs no se propagan** | `show vlan` solo muestra VLAN 1 | Puerto troncal NO configurado | Configurar `switchport mode trunk` ANTES de VTP |
| 2 | **EtherChannel suspendido** | `show etherchannel`: Port-Channel SD | VLANs permitidas inconsistentes | Revisar `show etherchannel detail`, igualar trunk config |
| 3 | **Ping falla en misma VLAN** | ICMP Destination Unreachable | PC con gateway incorrecto | Verificar Gateway = 192.168.X.1 (la SVI del switch) |
| 4 | **Dominios colisión incorrectos** | Documentar dice "3 dominios" pero se ve "1" | PCs conectadas directo al switch, no al hub | Conectar todas las PCs a Hubs primero, luego Hub → Switch |
| 5 | **VLAN nativa mismatch** | STP reporte warnings | Un extremo VLAN 1, otro VLAN 99 | Ambos extremos: `switchport trunk native vlan 99` |
| 6 | **STP Root no elegible** | SW-H1 muestra como root (malo) | No se ejecutó `root primary` en SW-CORE | Ejecutar `spanning-tree vlan [x] root primary` en SW-CORE |
| 7 | **Port-Channel no forma** | Interfaces muestran individual, no agrupadas | PAgP desirable/auto pero incompatible | Cambiar a `mode desirable` ↔ `mode auto` |

---

## 5. Lecciones Aprendidas (Conceptos Clave)

### 5.1 VLANs = Segmentación Lógica

```
CONCEPTO ANTES:  "La red es una sola cosa"
CONCEPTO DESPUÉS: "Puedo dividir la red en grupos lógicos
                    sin hacer cambios físicos de cableado"

BENEFICIO: Flexibilidad operacional
```

### 5.2 VTP = Escalabilidad Administrativa

```
SIN VTP:  Crear VLAN en 5 switches = 5 × 20 comandos = 100 comandos
CON VTP:  Crear VLAN en 1 switch = 20 comandos
          Propagan automáticamente a 5 switches

1 servidor + N clientes = Eficiencia exponencial
```

### 5.3 STP Rapid PVST+ = Confiabilidad

```
802.1D original:    50 segundos para reconverger (inaceptable en hospitales)
Rapid PVST+:        2 segundos para reconverger (aceptable)

Diferencia = Protección de pacientes
```

### 5.4 EtherChannel = Rendimiento

```
1 × 1 Gbps = Límite de transferencia
3 × 1 Gbps = 3 × Límite de transferencia
6 × 1 Gbps = 6 × Límite de transferencia

Beneficio: Transferencias de datos críticas no se quedan esperando
```

### 5.5 Orden de Configuración IMPORTA

```
CORRECTO:
1. Crear VLANs en Server
2. Configurar Trunks en ambos extremos
3. Configurar VTP Client
4. Resultado: VLANs propagan automáticamente

INCORRECTO (pero muchas personas lo hacen):
1. Configurar VTP Client
2. Crear VLANs en Server
3. Configurar Trunks
4. Resultado: Confusión, VLANs aparecen después de tiempo aleatorio
```

---

## 6. Verificación Final de la Red

```bash
# Checklist de validación:

✅ SW-CORE# show vlan
   → Muestra VLANs 14, 24, 34, 44, 99, 999

✅ SW-H1# show vlan
   → Muestra MISMAS VLANs (propagadas por VTP)

✅ SW-CORE# show vtp status
   → Mode: Server, Domain: 202308204, Version: 2

✅ SW-H1# show vtp status
   → Mode: Client, sincronizado con server

✅ SW-CORE# show etherchannel summary
   → Port-Channel1 SU Ph [GigabitEthernet0/1 GigabitEthernet0/2 FastEthernet0/24]

✅ SW-CORE# show spanning-tree vlan 14
   → "This bridge is the root"

✅ PC-Q-H1-1> ping 192.168.4.10
   → Reply from 192.168.4.10: bytes=32 time=10ms TTL=64 ✓

✅ PC-Q-H1-1> ping 192.168.4.130
   → No response / Destination unreachable (ESPERADO: entre VLANs)
```

---

## 7. Conclusiones y Reflexión

### ¿Qué aprendimos?

1. **VLANs no son "routers"**
   - No enrutan entre VLANs
   - Solo aíslan dominios de broadcast
   - Se necesita un router para comunicación inter-VLAN

2. **La configuración del trunk es FUNDAMENTAL**
   - Sin trunk = Sin VLANs
   - Sin trunk = Sin VTP
   - Trunk = "Autopista de múltiples carriles" (uno por VLAN)

3. **PAgP / LACP hacen el trabajo, pero nosotros diseñamos**
   - Indicamos desirable/auto
   - El protocolo negocia automáticamente
   - Pero los conflictos requieren análisis humano

4. **STP es invisible pero crítico**
   - Bloques puertos automáticamente
   - Reconverge automáticamente
   - Una herramienta "autopoilot" de la red

5. **La escalabilidad viene de automatización**
   - VTP = Automatización de VLANs
   - STP = Automatización de redundancia
   - EtherChannel = Automatización de agregación
   - Menos trabajo humano = Menos errores

### Aplicación en la Industria

En hospitales reales (`IPS`, `UNITEC`, `Aurora`), estas tecnologías se usan exactamente así:
- VLANs por tipo de equipamiento (UCI, Quirófanos, Radiología)
- VTP para simplificar cambios de configuración
- STP Rapid PVST+ para alta disponibilidad (uptime > 99.9%)
- EtherChannel entre edificios (core → hospital satelital)

---

*Fin del Informe Educativo de Desarrollo*  
**Carnet:** 20238204 | **Ciclo:** 2026-01  
**Universidad de San Carlos de Guatemala — Facultad de Ingeniería**

### Fase 1 — Diseño y Planificación

**Decisiones tomadas:**
- Se eligió topología **estrella extendida** con un switch core central y switches de hospital para simplificar la administración y facilitar la implementación de EtherChannel.
- Se definieron 4 áreas funcionales hospitalarias: Quirófanos (VLAN 14), UCI (VLAN 24), Administración (VLAN 34) y Urgencias (VLAN 44).
- Se calculó el subnetting usando la red base `192.168.4.0/24` ajustada al carnet (XX=4).

**Hospitales seleccionados:**
1. Hospital Roosevelt
2. Hospital General San Juan de Dios
3. Hospital de Ginecología y Obstetricia IGSS
4. Hospital Nacional de Ortopedia y Rehabilitación
5. Hospital Infantil de Infectología y Rehabilitación
6. Hospital Centro Médico Militar

### Fase 2 — Construcción de Topología en Packet Tracer

**Pasos seguidos:**
1. Agregar 1 switch Cisco 2960 como **Core Switch**.
2. Agregar 5 switches Cisco 2960 para hospitales H1-H5.
3. Implementar H6 (Hospital Militar) con 1 hub unico de acceso, segun requerimiento de 1 dominio de colision.
4. Agregar hubs de acceso segun los dominios de colision requeridos por hospital.
5. Conectar cada switch de hospital a sus hubs de acceso y luego las PCs a esos hubs para mantener intacto el numero exacto de dominios.
6. Etiquetar todos los dispositivos con sus nombres correspondientes.

**Problema encontrado:** Los switches Cisco 2960 en Packet Tracer solo tienen 24 puertos FastEthernet y 2 GigabitEthernet, lo cual limitó la cantidad de puertos disponibles para el EtherChannel (que requiere 3 puertos GigabitEthernet hacia el core).

**Solución adoptada:** Se utilizaron los puertos GigabitEthernet (Gi0/1 y Gi0/2) más un puerto FastEthernet adicional (Fa0/24) como tercer enlace del EtherChannel. En la documentación se especifica esta configuración.

> **📸 INSERTAR CAPTURA:** Vista general de la topología construida en Packet Tracer.

### Fase 3 — Configuración de VLANs y VTP

**Pasos seguidos:**
1. Configurar SW-CORE como **servidor VTP** con dominio `20238204`.
2. Crear las 6 VLANs en el SW-CORE (14, 24, 34, 44, 99, 999).
3. Configurar SW-H1, SW-H2, SW-H3, SW-H4 y SW-H5 como **clientes VTP**.
4. Verificar propagación con `show vlan brief`.

**Problema encontrado:** Las VLANs no se propagaban a los switches clientes (H1-H5) inicialmente.

**Solución adoptada:** Se verificó que los puertos troncales entre el core y los hospitales estuvieran correctamente configurados con `switchport mode trunk` **antes** de configurar VTP. El VTP solo propaga VLANs a través de puertos troncales activos.

> **📸 INSERTAR CAPTURA:** Salida de `show vlan brief` antes y después de la propagación VTP.

### Fase 4 — Configuración de EtherChannel

**Pasos seguidos:**
1. Agrupar 3 interfaces físicas en SW-CORE con `channel-group 1 mode desirable`.
2. Configurar los mismos puertos en SW-H1 con `channel-group 1 mode auto`.
3. Configurar la interfaz Port-Channel como troncal con VLAN nativa 99.
4. Verificar con `show etherchannel summary`.

**Problema encontrado:** El Port-Channel aparecía en estado "SD" (Suspended) en lugar de "SU".

**Solución adoptada:** Se verificó que ambos extremos del canal estuvieran configurados con el mismo modo de trunk y las mismas VLANs permitidas. La inconsistencia en las VLANs permitidas causaba la suspensión del canal.

> **📸 INSERTAR CAPTURA:** Salida de `show etherchannel summary` con el Port-Channel en estado "SU".

### Fase 5 — Configuración STP Rapid PVST+

**Pasos seguidos:**
1. Configurar `spanning-tree mode rapid-pvst` en todos los switches.
2. Establecer SW-CORE como root bridge con `spanning-tree vlan [id] root primary`.
3. Verificar con `show spanning-tree`.

**Resultado:** SW-CORE aparece con prioridad reducida (4096 + VLAN ID) y estado "This bridge is the root".

> **📸 INSERTAR CAPTURA:** Salida de `show spanning-tree vlan 14` en SW-CORE y en SW-H1.

### Fase 6 — Asignación de IPs y Pruebas

**Pasos seguidos:**
1. Asignar IPs a todas las PCs según la tabla de subnetting.
2. Realizar al menos 10 pings exitosos entre PCs de la misma VLAN.
3. Confirmar que pings entre VLANs distintas fallan (sin router).
4. Capturar análisis de paquetes ARP/ICMP en modo simulación.

> **📸 INSERTAR CAPTURA:** Configuración IP de una PC representativa en cada VLAN.

> **📸 INSERTAR CAPTURA:** Resultados de los 10 pings exitosos (mínimo 2 pings por VLAN).

---

## 3. Problemas Encontrados y Soluciones

| # | Problema | Causa | Solución |
|---|---------|-------|----------|
| 1 | VLANs no se propagan por VTP | Puertos troncales no configurados antes de VTP | Configurar trunk primero, luego VTP |
| 2 | EtherChannel en estado SD | VLANs permitidas inconsistentes entre extremos | Igualar `switchport trunk allowed vlan` en ambos lados |
| 3 | Ping falla dentro de la misma VLAN | PC con gateway incorrecto | Verificar gateway según tabla de subnetting |
| 4 | Dominios de colisión incorrectos | Se mezcló la capa de switches con conexiones directas de PCs | Colgar cada segmento de usuarios en hubs de acceso conectados al switch del hospital |
| 5 | VLAN nativa mismatch | Un extremo con VLAN 1 y otro con VLAN 99 | Configurar `switchport trunk native vlan 99` en ambos extremos |

---

## 4. Verificación del Funcionamiento Correcto

### Comandos de Verificación Utilizados

```bash
# Verificar VLANs
show vlan brief

# Verificar troncales
show interfaces trunk

# Verificar VTP
show vtp status

# Verificar STP
show spanning-tree

# Verificar EtherChannel
show etherchannel summary

# Verificar configuración de interfaces
show interfaces status

# Verificar tabla MAC
show mac address-table

# Verificar IP de SVI
show ip interface brief
```

> **📸 INSERTAR CAPTURA:** Salida de `show interfaces trunk` mostrando puertos troncales activos.

> **📸 INSERTAR CAPTURA:** Salida de `show mac address-table` mostrando entradas para las diferentes VLANs.

---

## 5. Conclusiones

1. **VLANs:** La segmentación por áreas funcionales permite aislar el tráfico médico crítico (UCI, Quirófanos) del tráfico administrativo, mejorando seguridad y rendimiento.

2. **VTP:** La administración centralizada de VLANs mediante VTP reduce significativamente el tiempo de configuración al no tener que crear VLANs manualmente en cada switch.

3. **STP Rapid PVST+:** La versión Rapid de STP mejora los tiempos de convergencia de ~50 segundos (802.1D tradicional) a menos de 2 segundos, crítico en entornos hospitalarios donde la conectividad debe restablecerse rápidamente.

4. **EtherChannel:** La agregación de 3 enlaces físicos en un canal lógico triplica el ancho de banda disponible entre el core y los hospitales, y provee redundancia automática si un enlace falla.

5. **Dominios de Colisión:** El uso estratégico de switches de hospital y hubs de acceso permite cumplir exactamente con los dominios de colisión requeridos por la práctica, siempre conectando los equipos finales a los hubs y no directo al switch.

---

*Fin del Informe de Desarrollo*  
*Carnet: 20238204 | Redes de Computadoras 1 | USAC 2026*