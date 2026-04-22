# 🔍 AUDITORÍA DE CONSISTENCIA — Proyecto 2: BanTech GT

**Fecha:** 22 de abril de 2026  
**Carnet:** 202308204  
**Objetivo:** Verificar que Guía.md, ManualTecnico.md y Planificacion.md sean consistentes

---

## 1. DATOS IDENTIFICACIÓN

| Parámetro | Guía.md | Manual Técnico | Planificación | ✅/❌ |
|-----------|---------|----------------|---------------|------|
| Carnet | 202308204 | 202308204 | 202308204 | ✅ |
| XX | 04 | 04 | 04 | ✅ |
| Y | 4 | 4 | 4 | ✅ |
| Dominio VTP | bantech04 | bantech04 | bantech04 | ✅ |
| Password VTP | cisco | cisco | cisco | ✅ |

---

## 2. VLANs

### Sede Occidente (14, 24, 34, 44)

| VLAN | Guía.md | Manual Técnico | Planificación | ✅/❌ |
|------|---------|----------------|---------------|------|
| 14 - Cajas | 192.168.10.0/26 | 192.168.10.0/26 | 192.168.10.0/26 | ✅ |
| 24 - Asesores | 192.168.10.64/27 | 192.168.10.64/27 | 192.168.10.64/27 | ✅ |
| 34 - Gerencia | 192.168.10.112/28 | 192.168.10.112/28 | 192.168.10.112/28 | ✅ |
| 44 - Seguridad | 192.168.10.96/28 | 192.168.10.96/28 | 192.168.10.96/28 | ✅ |

### Sede Norte (54, 64, 74)

| VLAN | Guía.md | Manual Técnico | Planificación | ✅/❌ |
|------|---------|----------------|---------------|------|
| 54 - Análisis | 192.168.20.0/26 | 192.168.20.0/26 | 192.168.20.0/26 | ✅ |
| 64 - Auditoría | 192.168.20.64/27 | 192.168.20.64/27 | 192.168.20.64/27 | ✅ |
| 74 - Legal | 192.168.20.96/28 | 192.168.20.96/28 | 192.168.20.96/28 | ✅ |

### Sede Oriente (84, 94)

| VLAN | Guía.md | Manual Técnico | Planificación | ✅/❌ |
|------|---------|----------------|---------------|------|
| 84 - Bóveda | 192.168.30.64/26 | 192.168.30.64/26 | 192.168.30.64/26 | ✅ |
| 94 - Plataforma | 192.168.30.0/26 | 192.168.30.0/26 | 192.168.30.0/26 | ✅ |

### Data Center (14, 24, 34 — repetidas)

| VLAN | Guía.md | Manual Técnico | Planificación | ✅/❌ |
|------|---------|----------------|---------------|------|
| 14 - Core_BD | 192.168.40.32/28 | 192.168.40.32/28 | 192.168.40.32/28 | ✅ |
| 24 - Web_Apps | 192.168.40.0/27 | 192.168.40.0/27 | 192.168.40.0/27 | ✅ |
| 34 - NOC | 192.168.40.48/28 | 192.168.40.48/28 | 192.168.40.48/28 | ✅ |

---

## 3. CONFIGURACIONES VTP

### Fase 2 — Occidente

- ✅ Paso 2.2: `vtp mode server` en SW-DIST-OCC
- ✅ Paso 2.2: `vtp domain bantech04`
- ✅ Paso 2.2: `vtp version 2` (AGREGADO 22/04/26)
- ✅ Paso 2.2: `vtp password cisco`
- ✅ Paso 2.3: VLANs 14, 24, 34, 44 con nombres

### Fase 3 — Norte

- ✅ Paso 3.2: Hostname, contraseñas en SW-CORE-NORTE, SW-DIST-N1, SW-DIST-N2
- ✅ Paso 3.2: `vtp mode server` en SW-CORE-NORTE
- ✅ Paso 3.2: `vtp mode client` en SW-DIST-N1, SW-DIST-N2
- ✅ Paso 3.2: `vtp version 2` (AGREGADO 22/04/26)
- ✅ Paso 3.2: `vtp password cisco`
- ✅ Paso 3.4: Rapid PVST+ en los 3 switches

### Fase 4 — Oriente

- ✅ Paso 4.1: `vtp mode server` en MS1
- ✅ Paso 4.1: `vtp mode client` en MS2
- ✅ Paso 4.1: `vtp version 2` (AGREGADO 22/04/26)
- ✅ Paso 4.3: HSRP configurado con prioridades

### Fase 5 — Data Center

- ℹ️ Nota: Sin VTP (usa rutas estáticas — correcto)

---

## 4. ARQUITECTURA TOPOLOGÍAS

| Sede | Topología | Guía.md | Manual Técnico | ✅/❌ |
|------|-----------|---------|----------------|------|
| Occidente | Estrella (SW-DIST-OCC central) | ✅ | ✅ | ✅ |
| Norte | Triángulo (Anillo 3 switches) | ✅ | ✅ | ✅ |
| Oriente | Estrella dual (MS1 + MS2 HSRP) | ✅ | ✅ | ✅ |
| Data Center | Dos capas + EtherChannel | ✅ | ✅ | ✅ |

---

## 5. CONFIGURACIONES ESPECIALES

### EtherChannel

| Enlace | Modo | Guía.md | Manual Técnico | ✅/❌ |
|--------|------|---------|----------------|------|
| Core1 ↔ Core2 (Backbone) | LACP active | ✅ | ✅ | ✅ |
| SW-DIST-DC ↔ SW-ACC-BD | LACP active | ✅ | ✅ | ✅ |

### Protocolos de Enrutamiento

| Protocolo | Dispositivos | Guía.md | Manual Técnico | ✅/❌ |
|-----------|--------------|---------|----------------|------|
| OSPF | Core1, Core2, Core3, MS1, MS2 | ✅ | ✅ | ✅ |
| EIGRP AS100 | Core1, R-Occidente | ✅ | ✅ | ✅ |
| RIPv2 | Core2, R-Norte | ✅ | ✅ | ✅ |
| Estáticas | R-Central1, R-Central2 | ✅ | ✅ | ✅ |

### HSRP

| Parámetro | Guía.md | Manual Técnico | ✅/❌ |
|-----------|---------|----------------|------|
| VLAN 84 IP Virtual | 192.168.30.65 | 192.168.30.65 | ✅ |
| VLAN 94 IP Virtual | 192.168.30.1 | 192.168.30.1 | ✅ |
| MS1 Prioridad | 110 | 110 | ✅ |
| MS2 Prioridad | 100 | 100 | ✅ |

---

## 6. CAMBIOS REALIZADOS (22 de abril de 2026)

| Cambio | Archivo | Línea(s) | Descripción |
|--------|---------|----------|-------------|
| ✅ Agregar `vtp version 2` | Guía.md | Fase 2, Paso 2.2 | Sincronizar con Manual Técnico |
| ✅ Agregar `vtp version 2` | Guía.md | Fase 3, Paso 3.2 | Sincronizar con Manual Técnico |
| ✅ Agregar `vtp version 2` | Guía.md | Fase 4, Paso 4.1 | Sincronizar con Manual Técnico |
| ✅ Agregar configuración base | Guía.md | Fase 3, Paso 3.2 | Hostname, contraseñas en todos los switches cliente |
| ✅ Corregir distribución VLANs | Guía.md | Fase 3, Paso 3.4 | Puertos acceso en SW-DIST-N1 y SW-DIST-N2 para 3 VLANs |

---

## 7. RESUMEN DE ESTADO

### General
- **Guía.md:** ✅ 100% Consistente (después de actualizaciones)
- **Manual Técnico:** ✅ 100% Consistente
- **Planificación.md:** ✅ 100% Consistente

### Puntos de atención resueltos
1. ✅ VTP version 2 agregado en todas las fases
2. ✅ Configuración base de switches cliente completa en Fase 3
3. ✅ Distribución simétrica de VLANs en Sede Norte
4. ✅ Puertos de acceso configurados en ambos switches de distribución

---

## 8. VERIFICACIÓN FINAL

**Última auditoría:** 22/04/2026 14:30  
**Estado:** ✅ **APROBADO — Todos los documentos son consistentes**

