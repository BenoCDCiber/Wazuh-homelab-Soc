# SOC Home Lab — Wazuh SIEM

Laboratorio casero de seguridad defensiva construido para practicar habilidades de SOC Analyst: despliegue de un SIEM, integración de agentes, generación de detecciones y análisis de eventos.

## 🎯 Objetivo

Simular un entorno de monitoreo de seguridad (SOC) con detección de amenazas en tiempo real, replicando tareas reales de un analista: instalación de herramientas, troubleshooting, generación de ataques controlados y análisis de alertas.

## 🏗️ Arquitectura

El lab está compuesto por 3 máquinas virtuales distribuidas en 2 computadoras físicas conectadas en red:

| Máquina | Rol | Software |
|---|---|---|
| Ubuntu Server | SIEM / Servidor central | Wazuh 4.9.2 (indexer + manager + dashboard) |
| Windows 10 Pro | Endpoint monitoreado | Agente Wazuh + Sysmon (config SwiftOnSecurity) |
| Kali Linux | Atacante (simulación) | Nmap, Hydra |

Red configurada en modo **Bridged** para permitir comunicación entre máquinas en distintas computadoras físicas.

## 🛠️ Instalación

Ver documentación detallada en [`01-setup/`](./01-setup/).

Resumen de componentes desplegados:
1. Ubuntu Server 24.04 con IP estática
2. Wazuh instalado vía script oficial (`wazuh-install.sh -a`)
3. Agente Wazuh desplegado en Windows 10 desde el dashboard
4. Sysmon instalado con configuración de [SwiftOnSecurity](https://github.com/SwiftOnSecurity/sysmon-config)
5. Integración de Sysmon con Wazuh vía `ossec.conf` (canal `Microsoft-Windows-Sysmon/Operational`)

## 🐛 Problemas resueltos durante el despliegue

Documentar el troubleshooting es tan valioso como el resultado final — así se ve el proceso real de un analista:

- **Wazuh indexer fallando al iniciar:** causado por espacio en disco insuficiente (`No space left on device`). Resuelto extendiendo el volumen LVM con `lvextend` + `resize2fs`.
- **Permisos de archivo de configuración:** el usuario `wazuh-indexer` no tenía permisos de lectura sobre `/etc/default/wazuh-indexer`, corregido con `chown`.
- **Sysmon no capturaba tráfico SMB (puertos 135/139/445):** limitación conocida de Sysmon con el stack de red en modo kernel de Windows — documentado como hallazgo técnico.
- **Conflicto de IP duplicada** en la red del lab, resuelto reasignando IPs estáticas fuera del rango DHCP del router.

Ver detalle completo en [`lessons-learned.md`](./lessons-learned.md).

## 🚨 Detecciones documentadas

### Detección 1: Fuerza bruta RDP con Hydra

- **Herramienta de ataque:** Hydra (RDP module)
- **Objetivo:** Endpoint Windows 10, puerto 3389
- **Resultado:** 11 intentos fallidos de autenticación + 1 exitoso, detectados en tiempo real
- **Categorización Wazuh:** `authentication_failed`, `windows_security`
- **Evidencia:** ver [`02-detections/01-rdp-bruteforce.md`](./02-detections/01-rdp-bruteforce.md)

![Detección de fuerza bruta RDP](./screenshots/rdp-bruteforce-detection.png)

## 📚 Herramientas y tecnologías

`Wazuh` `Sysmon` `VMware` `Ubuntu Server` `Windows 10` `Kali Linux` `Nmap` `Hydra` `LVM` `SSH`

## 🎓 Aprendizajes clave

- Configuración end-to-end de un SIEM open source desde cero
- Integración de fuentes de logs (Windows Event Log + Sysmon) con un SIEM
- Troubleshooting real de infraestructura (disco, permisos, redes)
- Interpretación de alertas y mapeo a framework MITRE ATT&CK
- Simulación de ataques controlados para validar detecciones

---

📫 **Contacto:** [tu LinkedIn aquí]
