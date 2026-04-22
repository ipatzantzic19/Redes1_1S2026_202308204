# 📋 PLANIFICACIÓN — Proyecto 2: BanTech GT
**Redes de Computadoras 1 — USAC FIUSAC**
**Carnet:** 202308204 | **XX = 04** | **Y = 4**

---

## 1. Datos de Identificación

| Parámetro | Valor |
|-----------|-------|
| Carnet | 202308204 |
| XX (dos últimos dígitos) | 04 |
| Y (último dígito) | 4 |
| Dominio VTP | `bantech04` |
| Password VTP | `cisco` |
| Fecha límite entrega | 29/04/2026 |
| Calificación | 30/04/2026 |

---

## 2. Resumen de VLANs por Sede

### Sede Occidente (Agencia Regional Comercial)
| VLAN Nombre | Área | Hosts | ID VLAN (1Y, 2Y...) |
|-------------|------|-------|----------------------|
| Cajas | Transacciones críticas | 45 | **14** |
| Asesores | Servicio al cliente | 30 | **24** |
| Gerencia | Administración local | 10 | **34** |
| Seguridad | Cámaras y Biométricos | 12 | **44** |

### Sede Norte (Centro de Autorización de Créditos)
| VLAN Nombre | Área | Hosts | ID VLAN |
|-------------|------|-------|---------|
| Análisis | Analistas de Riesgo | 50 | **54** |
| Auditoría | Revisión de cuentas | 25 | **64** |
| Legal | Contratos | 14 | **74** |

### Sede Oriente (Centro Financiero Legado)
| VLAN Nombre | Área | Hosts | ID VLAN |
|-------------|------|-------|---------|
| Bóveda | Operaciones de valores | 40 | **84** |
| Plataforma | Atención VIP | 60 | **94** |

### Data Center y Sede Central (Alta Seguridad)
| VLAN Nombre | Área | Hosts | ID VLAN |
|-------------|------|-------|---------|
| Core_BD | Servidores de Base de Datos | 14 | **14** |
| Web_Apps | Servidores de Banca Virtual | 28 | **24** |
| NOC | Monitoreo central | 10 | **34** |

> **Nota:** Las VLANs del Data Center comparten números con Occidente. Son segmentos distintos pero con IDs iguales — esto es correcto según la fórmula 1Y, 2Y, 3Y.

---

## 3. Plan de Subnetting

### Espacio de direcciones propuesto

Se utilizará el bloque `192.168.XX.0/24` → `192.168.4.0/24` para las redes LAN internas de cada sede, y `10.0.0.0/8` para los enlaces punto a punto del backbone.

---

### 3.1 Backbone Core (enlaces punto a punto — FLSM /30)

Bloque asignado para backbone: **10.10.0.0/24** → dividido en /30 para cada enlace P2P.

| Enlace | Red | Máscara | Gateway A | Gateway B | Broadcast |
|--------|-----|---------|-----------|-----------|-----------|
| Core1 (SW) ↔ Core2 (SW) (EtherChannel/fibra, GigE1/1/1-2-PC1) | 10.10.0.0 | /30 | 10.10.0.1 | 10.10.0.2 | 10.10.0.3 |
| Core1 (SW) ↔ Core3 (Router) (GigE1/1/3-GigE0/0) | 10.10.0.4 | /30 | 10.10.0.5 | 10.10.0.6 | 10.10.0.7 |
| Core2 (SW) ↔ Core3 (Router) (GigE1/1/3-GigE0/1) | 10.10.0.8 | /30 | 10.10.0.9 | 10.10.0.10 | 10.10.0.11 |
| Core1 (SW) ↔ R-Occidente (GigE1/1/4-GigE0/0) EIGRP | 10.10.0.12 | /30 | 10.10.0.13 | 10.10.0.14 | 10.10.0.15 |
| Core2 (SW) ↔ R-Norte (GigE1/1/4-GigE0/0) RIPv2 | 10.10.0.16 | /30 | 10.10.0.17 | 10.10.0.18 | 10.10.0.19 |
| Core3 (Router) ↔ MS1-Oriente (GigE0/2-GigE0/1) OSPF | 10.10.0.20 | /30 | 10.10.0.21 | 10.10.0.22 | 10.10.0.23 |
| Core3 (Router) ↔ MS2-Oriente (GigE0/3-GigE0/1) OSPF | 10.10.0.24 | /30 | 10.10.0.25 | 10.10.0.26 | 10.10.0.27 |
| Core1 (SW) ↔ R-Central1 (GigE1/1/5-GigE0/0) estáticas | 10.10.0.28 | /30 | 10.10.0.29 | 10.10.0.30 | 10.10.0.31 |
| Core2 (SW) ↔ R-Central2 (GigE1/1/5-GigE0/0) estáticas | 10.10.0.32 | /30 | 10.10.0.33 | 10.10.0.34 | 10.10.0.35 |
| Core3 (Router) ↔ Serial WAN (Serial0/0/0) | 10.10.0.36 | /30 | 10.10.0.37 | 10.10.0.38 | 10.10.0.39 |

---

### 3.2 Sede Occidente — VLSM (basado en 192.168.10.0/24)

Orden de subnetting: de mayor a menor número de hosts.

| VLAN | Hosts req. | Hosts útiles necesarios | Red | Máscara | Rango útil | Broadcast | Gateway |
|------|-----------|------------------------|-----|---------|------------|-----------|---------|
| Cajas (VLAN 14) | 45 | 62 (/26) | 192.168.10.0 | /26 | .1 – .62 | .63 | 192.168.10.1 |
| Asesores (VLAN 24) | 30 | 30 (/27) | 192.168.10.64 | /27 | .65 – .94 | .95 | 192.168.10.65 |
| Seguridad (VLAN 44) | 12 | 14 (/28) | 192.168.10.96 | /28 | .97 – .110 | .111 | 192.168.10.97 |
| Gerencia (VLAN 34) | 10 | 14 (/28) | 192.168.10.112 | /28 | .113 – .126 | .127 | 192.168.10.113 |
| Enlace R-Occidente | 2 | 2 (/30) | 192.168.10.128 | /30 | .129 – .130 | .131 | — |

---

### 3.3 Sede Norte — VLSM (basado en 192.168.20.0/24)

| VLAN | Hosts req. | Red | Máscara | Rango útil | Broadcast | Gateway |
|------|-----------|-----|---------|------------|-----------|---------|
| Análisis (VLAN 54) | 50 | 192.168.20.0 | /26 | .1 – .62 | .63 | 192.168.20.1 |
| Auditoría (VLAN 64) | 25 | 192.168.20.64 | /27 | .65 – .94 | .95 | 192.168.20.65 |
| Legal (VLAN 74) | 14 | 192.168.20.96 | /28 | .97 – .110 | .111 | 192.168.20.97 |

---

### 3.4 Sede Oriente — VLSM (basado en 192.168.30.0/24)

| VLAN | Hosts req. | Red | Máscara | Rango útil | Broadcast | Gateway Virtual (HSRP) |
|------|-----------|-----|---------|------------|-----------|------------------------|
| Plataforma (VLAN 94) | 60 | 192.168.30.0 | /26 | .1 – .62 | .63 | 192.168.30.1 |
| Bóveda (VLAN 84) | 40 | 192.168.30.64 | /26 | .65 – .126 | .127 | 192.168.30.65 |

**HSRP Oriente:**
- MS1 → Active (prioridad 110) | MS2 → Standby (prioridad 100)
- VLAN 94 gateway virtual: `192.168.30.1`
- VLAN 84 gateway virtual: `192.168.30.65`

---

### 3.5 Data Center / Sede Central — VLSM (basado en 192.168.40.0/24)

| VLAN | Hosts req. | Red | Máscara | Rango útil | Broadcast | Gateway |
|------|-----------|-----|---------|------------|-----------|---------|
| Web_Apps (VLAN 24) | 28 | 192.168.40.0 | /27 | .1 – .30 | .31 | 192.168.40.1 |
| Core_BD (VLAN 14) | 14 | 192.168.40.32 | /28 | .33 – .46 | .47 | 192.168.40.33 |
| NOC (VLAN 34) | 10 | 192.168.40.48 | /28 | .49 – .62 | .63 | 192.168.40.49 |

---

## 3.5 Tabla de Dispositivos Finales (Hosts / PCs)

### Cantidad de Hosts por Sede y Área

| Sede | VLAN | Área Funcional | Qty Hosts | Rango IPs | Gateway |
|------|------|---|---|---|---|
| **Occidente** | 14 | Cajas | 7 | 192.168.10.1-7 | 192.168.10.1 |
| | 24 | Asesores | 7 | 192.168.10.65-71 | 192.168.10.65 |
| | 34 | Gerencia | 5 | 192.168.10.113-117 | 192.168.10.113 |
| | 44 | Seguridad | 7 | 192.168.10.97-103 | 192.168.10.97 |
| **Norte** | 54 | Análisis | 7 | 192.168.20.1-7 | 192.168.20.1 |
| | 64 | Auditoría | 7 | 192.168.20.65-71 | 192.168.20.65 |
| | 74 | Legal | 5 | 192.168.20.97-101 | 192.168.20.97 |
| **Oriente** | 94 | Plataforma | 7 | 192.168.30.1-7 | 192.168.30.1 (HSRP) |
| | 84 | Bóveda | 7 | 192.168.30.65-71 | 192.168.30.65 (HSRP) |
| **Data Center** | 24 | Web_Apps | 7 | 192.168.40.1-7 | 192.168.40.1 |
| | 14 | Core_BD | 7 | 192.168.40.33-39 | 192.168.40.33 |
| | 34 | NOC | 5 | 192.168.40.49-53 | 192.168.40.49 |

**Total BanTech GT:** 78 dispositivos finales distribuidos en 4 sedes, con redundancia HSRP en Oriente.

---

## 4. Arquitectura General del Backbone

### 4.1 Topología del Backbone Core

```
                    ┌─────────────────────────────────────┐
                    │         BACKBONE NACIONAL            │
                    │                                      │
         ┌──────────┤   Core1 ════ Core2 ════ Core3       │
         │          │    (OSPF)    (OSPF)    (OSPF)        │
         │          │      ║         ║         ║            │
         │          │   [LACP EtherChannel - Fibra]        │
         │          └──────────────────────────────────────┘
         │
    Redistribución
```

**Dispositivos del Backbone:**
- **Core1** — Multilayer Switch 3650-24PS (IP Routing habilitado) 
  - Puertos: GigE1/1/1-2 (EtherChannel fibra), GigE1/1/3 (Core3), GigE1/1/4 (R-Occidente), GigE1/1/5 (R-Central1)
  - Rol: OSPF + EIGRP (redistribuyente)
- **Core2** — Multilayer Switch 3650-24PS (IP Routing habilitado)
  - Puertos: GigE1/1/1-2 (EtherChannel fibra), GigE1/1/3 (Core3), GigE1/1/4 (R-Norte), GigE1/1/5 (R-Central2)
  - Rol: OSPF + RIPv2 (redistribuyente)
- **Core3** — Router 4331 o 2911
  - Puertos: GigE0/0 (Core1), GigE0/1 (Core2), GigE0/2 (MS1), GigE0/3 (MS2), Serial0/0/0 (WAN)
  - Rol: OSPF + Enlace Serial WAN
- **MS1 y MS2** — Switches multicapa en Oriente (con IP Routing)

### 4.2 Dominios de Enrutamiento

| Dominio | Protocolo | Sedes cubiertas | Dispositivos frontera |
|---------|-----------|-----------------|------------------|
| Principal | OSPF Area 0 | Backbone completo + Oriente | Core1 (SW), Core2 (SW), Core3 |
| Expansión Regional | EIGRP AS 100 | Occidente | R-Occidente, Core1 (SW) |
| Red Legada | RIPv2 | Norte | R-Norte, Core2 (SW) |
| Alta Seguridad | Estáticas | Data Center + Central | R-Central1, R-Central2 |

### 4.3 Redistribución de Rutas

| Punto de Redistribución | Redistribuye |
|------------------------|--------------|
| Core1 (Switch) | OSPF ↔ EIGRP y EIGRP ↔ OSPF |
| Core2 (Switch) | OSPF ↔ RIPv2 y RIPv2 ↔ OSPF |
| R-Central1 | Estáticas → OSPF |

### 4.4 Medios Físicos del Backbone

| Enlace | Medio | Puertos | Justificación |
|--------|-------|--------|----------------|
| Core1(SW) ↔ Core2(SW) | Fibra óptica (EtherChannel LACP) | GigE1/1/1-2 (PC1) | Mayor ancho de banda, menor latencia en núcleo |
| Core1(SW) ↔ Core3 | Ethernet Gigabit | GigE1/1/3 ↔ GigE0/0 | Interconexion local rápida |
| Core2(SW) ↔ Core3 | Ethernet Gigabit | GigE1/1/3 ↔ GigE0/1 | Redundancia del núcleo |
| Core3 ↔ Serial WAN | Enlace Serial | Serial0/0/0 | Simulación de WAN remota |
| Backbone ↔ Sedes | Ethernet Gigabit | GigE1/1/4-5 (SW) | Conectividad de borde |

---

## 5. Diseño por Sede

### 5.1 Sede Occidente — Topología Estrella con Router-on-a-Stick

**Justificación de topología:**
Se elige topología en estrella con un switch de distribución central (SW-DIST-OCC) y switches de acceso por área. Esto facilita la administración centralizada requerida y permite que un problema en el área de asesores no afecte las cajas, ya que cada VLAN está completamente segmentada. El router R-Occidente realiza el inter-VLAN mediante subinterfaces (Router-on-a-Stick), cumpliendo el requerimiento técnico.

**Distribución de Switches por Criticidad:**

| Dispositivo | VLAN(s) | Hosts | Razón del Diseño |
|---|---|---|---|
| **SW-CAJAS** | 14 | 45 | ⭐⭐⭐ Crítica: dedicado a transacciones financieras de cajas |
| **SW-ASESORES** | 24 | 30 | ⭐⭐⭐ Crítica: dedicado a transacciones de servicio al cliente |
| **SW-GERENCIA-SEG** | 34 + 44 | 10 + 12 | ⭐⭐ Soporte: compartido (bajo volumen, tráfico administrativo) |

**Razón técnica de compartir Gerencia y Seguridad:**
- Ambas áreas generan bajo tráfico administrativo (no transaccional)
- Un broadcast storm en Gerencia no afecta cajas ni asesores
- Costo-efectividad: no justifica un switch por solo 10 hosts de Gerencia
- Escalabilidad: si crece, puede separarse fácilmente en el futuro

**Dispositivos propuestos:**
- R-Occidente (router de borde, Router-on-a-Stick)
- SW-DIST-OCC (switch de distribución, VTP Server)
- SW-CAJAS (switch de acceso, VTP Client)
- SW-ASESORES (switch de acceso, VTP Client)
- SW-GERENCIA-SEG (switch de acceso, VTP Client)

**Puertos Trunk:** R-Occidente ↔ SW-DIST-OCC, SW-DIST-OCC ↔ todos los switches de acceso
**Puertos Acceso:** Hosts finales en cada switch de acceso

**Tolerancia a fallos:** El switch de distribución es el punto central; si cae un switch de acceso, solo ese segmento se ve afectado. Las transacciones críticas (Cajas, Asesores) están aisladas entre sí.

---

### 5.2 Sede Norte — Topología Anillo/Triángulo con Rapid PVST+

**Justificación de topología:**
Se implementa una topología en triángulo (anillo de 3 switches) entre el switch principal SW-CORE-NORTE y los switches de distribución SW-DIST-N1 y SW-DIST-N2. Esto crea bucles físicos intencionales que garantizan caminos alternos. Rapid PVST+ gestiona estos bucles y converge rápidamente ante una falla. SW-CORE-NORTE es forzado como Root Bridge porque tiene conectividad directa a R-Norte y es el punto de mayor jerarquía.

**Dispositivos propuestos:**
- R-Norte (router de borde)
- SW-CORE-NORTE (Root Bridge forzado, VTP Server)
- SW-DIST-N1 (VTP Client)
- SW-DIST-N2 (VTP Client)
- SW-ACCESS-ANALISIS, SW-ACCESS-AUDIT, SW-ACCESS-LEGAL (switches de acceso)

**Root Bridge:** SW-CORE-NORTE con `spanning-tree vlan 54,64,74 priority 4096`

**Bucle físico:** SW-CORE-NORTE ↔ SW-DIST-N1 ↔ SW-DIST-N2 ↔ SW-CORE-NORTE
Rapid PVST+ bloquea uno de los puertos redundantes automáticamente.

---

### 5.3 Sede Oriente — HSRP con MS1 y MS2

**Justificación de topología:**
Topología en estrella dual: los switches de acceso se conectan simultáneamente a MS1 (activo) y MS2 (standby). HSRP asegura que si MS1 pierde energía, MS2 toma el rol de gateway sin interrumpir el tráfico. Los hosts usan la IP virtual como gateway, no la IP física de ningún switch.

**Dispositivos propuestos:**
- MS1 (multilayer switch, HSRP Active, prioridad 110)
- MS2 (multilayer switch, HSRP Standby, prioridad 100)
- SW-BOVEDA (acceso, conectado a MS1 y MS2)
- SW-PLATAFORMA (acceso, conectado a MS1 y MS2)

**HSRP Groups:**
- Grupo 84 (VLAN Bóveda): IP virtual `192.168.30.65`, MS1 active
- Grupo 94 (VLAN Plataforma): IP virtual `192.168.30.1`, MS1 active

---

### 5.4 Data Center / Sede Central — EtherChannel + Rutas Estáticas

**Justificación de topología:**
Los servidores manejan ráfagas masivas de tráfico. Se implementa EtherChannel (LACP) entre los switches de distribución del Data Center (SW-DIST-DC1, SW-DIST-DC2) y los switches de acceso de servidores para agregar ancho de banda. El segmento está aislado con rutas estáticas, y R-Central1 redistribuye estas rutas hacia OSPF del backbone.

**Dispositivos propuestos:**
- R-Central1 (redistribuye estáticas → OSPF)
- R-Central2 (router de borde hacia Data Center)
- SW-DIST-DC (distribución, punto EtherChannel)
- SW-ACCESS-BD (acceso servidores BD, EtherChannel con SW-DIST-DC)
- SW-ACCESS-WEB (acceso servidores Web)
- SW-ACCESS-NOC (acceso NOC)

**EtherChannel:** LACP entre SW-DIST-DC ↔ SW-ACCESS-BD (mínimo 2 interfaces físicas)

---

## 6. Resumen de Protocolos y Tecnologías Requeridas

| Tecnología | Sede(s) | Propósito |
|-----------|---------|-----------|
| VLANs + 802.1Q | Todas | Segmentación del tráfico |
| VTP (Server/Client) | Occidente, Norte | Propagación centralizada de VLANs |
| Router-on-a-Stick | Occidente | Inter-VLAN en R-Occidente |
| OSPF Area 0 | Backbone | Protocolo de enrutamiento del núcleo (Core1 SW, Core2 SW, Core3) |
| EIGRP AS 100 | Occidente ↔ Backbone | Expansión regional (Core1 SW) |
| RIPv2 | Norte ↔ Backbone | Red legada (Core2 SW) |
| Rutas Estáticas | Data Center ↔ Backbone | Alta seguridad |
| Redistribución | Core1 (SW), Core2 (SW), R-Central1 | Interoperabilidad entre dominios |
| Rapid PVST+ | Norte | Evitar tormentas de broadcast en anillo |
| HSRP | Oriente (MS1, MS2) | Redundancia de gateway |
| EtherChannel (LACP) | Backbone + Data Center | Agregación de ancho de banda |
| Fibra óptica | Backbone Core1(SW) ↔ Core2(SW) | GigE1/1/1-2 (PC1) - EtherChannel LACP | Enlace de alta velocidad |
| Enlace Serial | Backbone Core3 | Serial0/0/0 (módulo WIC-2T) | Simulación WAN |
| VLSM | Todas las sedes | Optimización de direcciones IP |
| FLSM /30 | Backbone P2P | Direccionamiento de enlaces |

---

## 7. Plan de Trabajo (Cronograma Personal)

| Día | Fecha | Actividad |
|-----|-------|-----------|
| 1 | 13/04 | Lectura completa del enunciado, cálculo de VLANs y subnetting |
| 2–3 | 14–15/04 | Diseño lógico de todas las sedes en papel/diagrama |
| 4–5 | 16–17/04 | Construcción del Backbone en Packet Tracer |
| 6–7 | 18–19/04 | Configuración OSPF, EIGRP, RIPv2 en backbone |
| 8 | 20/04 | Redistribución de rutas (Core1 y Core2) |
| 9–10 | 21–22/04 | Sede Occidente: VLANs, VTP, Router-on-a-Stick |
| 11–12 | 23–24/04 | Sede Norte: Anillo, Rapid PVST+, Root Bridge |
| 13–14 | 25–26/04 | Sede Oriente: HSRP con MS1 y MS2 |
| 15 | 27/04 | Data Center: EtherChannel LACP, rutas estáticas |
| 16 | 28/04 | Pruebas de conectividad extremo a extremo, failover |
| 17 | 29/04 | Documentación final, README.md, capturas de pantalla |

---

## 8. Checklist de Verificación Final

### Backbone
- [ ] Dos Multilayer Switches 3650-24PS en el núcleo (Core1, Core2) con IP Routing habilitado
- [ ] Un Router de capa 3 (Core3) para enlace serial WAN
- [ ] EtherChannel LACP en enlace principal (fibra óptica) entre Core1 y Core2
- [ ] OSPF configurado y convergido (Core1, Core2, Core3, MS1, MS2)
- [ ] EIGRP configurado y convergido (AS 100)
- [ ] RIPv2 configurado y convergido
- [ ] Rutas estáticas configuradas
- [ ] Redistribución OSPF ↔ EIGRP en Core1
- [ ] Redistribución OSPF ↔ RIPv2 en Core2
- [ ] Enlace serial presente
- [ ] Enlace de fibra óptica presente

### Sede Occidente
- [ ] VLANs 14, 24, 34, 44 creadas y nombradas
- [ ] VTP Domain `bantech04`, password `cisco`
- [ ] Router-on-a-Stick en R-Occidente (subinterfaces)
- [ ] Trunk 802.1Q entre router y switch
- [ ] Mínimo 7 hosts por área
- [ ] Ping inter-VLAN exitoso

### Sede Norte
- [ ] VLANs 54, 64, 74 creadas y nombradas
- [ ] Bucle físico (triángulo de switches)
- [ ] Rapid PVST+ activado
- [ ] Root Bridge forzado en SW-CORE-NORTE
- [ ] Mínimo 7 hosts por área
- [ ] Prueba de failover STP documentada

### Sede Oriente
- [ ] VLANs 84, 94 creadas y nombradas
- [ ] HSRP configurado en MS1 y MS2
- [ ] IP virtual definida por grupo HSRP
- [ ] Prueba de failover (apagar MS1 → MS2 toma control)
- [ ] Hosts sin perder conectividad en failover

### Data Center
- [ ] VLANs 14, 24, 34 creadas y nombradas
- [ ] EtherChannel LACP (mínimo 2 interfaces)
- [ ] Rutas estáticas en R-Central1 y R-Central2
- [ ] Redistribución estáticas → OSPF en R-Central1
- [ ] Mínimo 7 hosts por área

### General
- [ ] Hostname configurado en todos los routers y switches
- [ ] Nombres de VLAN explícitos en todos los dispositivos
- [ ] Ping exitoso Occidente ↔ Data Center
- [ ] `show ip route` muestra redistribución correcta
- [ ] Repositorio GitHub con commits propios
- [ ] Auxiliar agregado como colaborador
- [ ] README.md completo
- [ ] Archivo .pkt funcional sin errores