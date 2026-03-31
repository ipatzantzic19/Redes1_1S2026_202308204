# Informe de Desarrollo — Práctica 2: Red de Hospitales Metropolitano
**Curso:** Redes de Computadoras 1  
**Universidad:** San Carlos de Guatemala — Facultad de Ingeniería  
**Carnet:** 20238204  
**Fecha:** 2026

---

## 1. Descripción General de la Implementación

Esta práctica consiste en el diseño e implementación de una red LAN segmentada para 6 hospitales metropolitanos de Guatemala, utilizando Cisco Packet Tracer. La red incluye segmentación por VLANs, administración centralizada con VTP, prevención de bucles con STP Rapid PVST+ y agregación de enlaces con EtherChannel PAgP.

---

## 2. Proceso de Implementación

### Fase 1 — Diseño y Planificación

**Decisiones tomadas:**
- Se eligió topología **estrella** con un switch core central para simplificar la administración y facilitar la implementación de EtherChannel.
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
2. Agregar 6 switches Cisco 2960 (uno por hospital).
3. Agregar hubs según los dominios de colisión requeridos por hospital.
4. Conectar PCs representativas a los switches/hubs de cada hospital.
5. Etiquetar todos los dispositivos con sus nombres correspondientes.

**Problema encontrado:** Los switches Cisco 2960 en Packet Tracer solo tienen 24 puertos FastEthernet y 2 GigabitEthernet, lo cual limitó la cantidad de puertos disponibles para el EtherChannel (que requiere 3 puertos GigabitEthernet hacia el core).

**Solución adoptada:** Se utilizaron los puertos GigabitEthernet (Gi0/1 y Gi0/2) más un puerto FastEthernet adicional (Fa0/24) como tercer enlace del EtherChannel. En la documentación se especifica esta configuración.

> **📸 INSERTAR CAPTURA:** Vista general de la topología construida en Packet Tracer.

### Fase 3 — Configuración de VLANs y VTP

**Pasos seguidos:**
1. Configurar SW-CORE como **servidor VTP** con dominio `20238204`.
2. Crear las 6 VLANs en el SW-CORE (14, 24, 34, 44, 99, 999).
3. Configurar todos los SW-Hospital como **clientes VTP**.
4. Verificar propagación con `show vlan brief`.

**Problema encontrado:** Las VLANs no se propagaban a los switches clientes inicialmente.

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
| 4 | Dominios de colisión incorrectos | Hub conectado a hub (crea 1 dominio) | Conectar hubs directamente al switch, no en cadena |
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

5. **Dominios de Colisión:** El uso estratégico de switches y hubs permite cumplir exactamente con los dominios de colisión requeridos por la práctica, demostrando comprensión de la capa física del modelo OSI.

---

*Fin del Informe de Desarrollo*  
*Carnet: 20238204 | Redes de Computadoras 1 | USAC 2026*