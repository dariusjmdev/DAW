# DAW
<div align="center">

# Servidor Ubuntu 22.04 LTS

### Documentación Técnica

**Instalación, configuración y verificación de un servidor virtualizado con acceso remoto seguro por SSH**

</div>

---

| | |
|:--|:--|
| **Autor** | Darius Joanito Marinescu |
| **Fecha** | 20/09/2026 |
| **Sistema operativo** | Ubuntu Server 22.04 LTS |
| **Virtualización** | VirtualBox sobre Windows |
| **Acceso remoto** | SSH con autenticación por clave pública |

---

## Índice

1. [Objetivo](#1-objetivo)
2. [Entorno de virtualización](#2-entorno-de-virtualización)
3. [Instalación de Ubuntu Server](#3-instalación-de-ubuntu-server)
4. [Actualización del sistema](#4-actualización-del-sistema)
5. [Usuario con permisos sudo](#5-usuario-con-permisos-sudo)
6. [Configuración de red](#6-configuración-de-red)
7. [Acceso remoto por SSH](#7-acceso-remoto-por-ssh)
8. [Medidas adicionales de seguridad](#8-medidas-adicionales-de-seguridad)
9. [Resumen técnico del servidor](#9-resumen-técnico-del-servidor)
10. [Resolución de incidencias](#10-resolución-de-incidencias)
11. [Checklist final](#11-checklist-final)
12. [Conclusiones](#12-conclusiones)
13. [Anexo: referencia rápida de comandos](#13-anexo-referencia-rápida-de-comandos)

---

## 1. Objetivo

Este documento describe el proceso completo de creación, instalación, configuración y verificación de un servidor **Ubuntu Server 22.04 LTS** ejecutado en una máquina virtual.

El servidor se ha actualizado, se ha configurado un usuario con permisos `sudo` y se ha habilitado el acceso remoto mediante SSH, incluyendo autenticación por clave pública.

### Fases del proyecto

```text
 Instalacion  -->  Actualizacion  -->  SSH activo y Clave pública  
      |                  |                      |                 
 Snapshot 1         Snapshot 2               Snapshot 3        
```

---

## 2. Entorno de virtualización

### 2.1. Software utilizado

| Componente | Función |
|:--|:--|
| VirtualBox (sobre Windows) | Hipervisor de virtualización |
| VirtualBox Extension Pack | Compatibilidad adicional |

### 2.2. Configuración de la máquina virtual

| Recurso | Valor |
|:--|:--|
| Memoria RAM | 8 GB |
| CPU | 6 núcleos |
| Disco | 25 GB (VDI dinámico) |
| Sistema operativo | Ubuntu Server 22.04 LTS |
| Red | Adaptador puente (Bridge) |
| Gráficos | 16 MB |
| Controlador | SATA |

### 2.3. Justificación técnica

| Recurso | Justificación |
|:--|:--|
| 8 GB de RAM | Permiten ejecutar servicios y tareas administrativas sin limitaciones |
| 6 núcleos | Garantizan fluidez en procesos intensivos, como actualizaciones o compilaciones |
| 25 GB de disco | Suficientes para el sistema, los paquetes y los registros |
| Modo Bridge | Proporciona una IP accesible desde el host, imprescindible para conexiones SSH |
| Ubuntu 22.04 LTS | Versión estable con soporte prolongado |

### 2.4. Esquema de red

```text
+------------------+                                  +-------------------------+
|  Host Windows    |   Red local - adaptador puente   |  VM Ubuntu Server 22.04 |
|  VirtualBox      | <--------------------------->    |  IP 192.168.1.151       |
|  Cliente SSH     |                                  |  OpenSSH - puerto 22    |
+------------------+                                  +-------------------------+
```

### 2.5. Snapshots creados

| N.º | Estado del sistema |
|:--:|:--|
| 1 | Instalación inicial del sistema |
| 2 | Sistema actualizado |
| 3 | SSH configurado |
| 4 | Acceso sin contraseña funcionando |

---

## 3. Instalación de Ubuntu Server

1. Selección del idioma: **Español**.
2. Instalación del sistema base.
3. Creación del usuario inicial: `trabajo`.
4. Activación del servidor **OpenSSH** durante la instalación.
5. Finalización de la instalación y primer arranque.

---

## 4. Actualización del sistema

Comandos ejecutados:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

Comprobación de que no quedan paquetes pendientes de actualizar:

```bash
sudo apt list --upgradable
```

---

## 5. Usuario con permisos sudo

Si el usuario no se creó durante la instalación:

```bash
sudo adduser trabajo
sudo usermod -aG sudo trabajo
```

Verificación del acceso `sudo`:

```bash
su - trabajo
sudo ls /root
```

Si el comando se ejecuta sin errores, el usuario tiene permisos `sudo` correctamente configurados.

---

## 6. Configuración de red

Obtención de la dirección IP del servidor:

```bash
ip a
```

| Parámetro | Valor |
|:--|:--|
| Dirección IP | `192.168.1.151` |
| Uso | Conexiones SSH desde el host Windows |

---

## 7. Acceso remoto por SSH

### 7.1. Comprobación inicial desde Windows

```powershell
ssh trabajo@192.168.1.151
```

Si la conexión se establece, el servicio SSH funciona correctamente.

### 7.2. Configuración de acceso sin contraseña

**Paso 1. Generación de claves en Windows**

```powershell
ssh-keygen -t ed25519
```

La clave pública se obtiene con:

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

**Paso 2. Copia de la clave pública al servidor**

En Ubuntu:

```bash
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
```

Se pega la clave pública generada en Windows.

**Paso 3. Permisos correctos**

```bash
sudo chown -R trabajo:trabajo ~/.ssh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

| Ruta | Permisos | Significado |
|:--|:--|:--|
| `~/.ssh` | `700` | Solo el propietario puede acceder |
| `~/.ssh/authorized_keys` | `600` | Solo el propietario puede leer y escribir |

**Paso 4. Reinicio del servicio SSH**

```bash
sudo systemctl restart ssh
```

**Paso 5. Verificación técnica detallada**

Se ejecuta la conexión en modo detallado para comprobar que el cliente ofrece su clave pública y que la autenticación se completa con ella, sin solicitar contraseña:

```powershell
ssh -v trabajo@192.168.1.151
```

---

## 8. Medidas adicionales de seguridad

### 8.1. Deshabilitar la autenticación por contraseña

```bash
sudo nano /etc/ssh/sshd_config
```

Modificar la directiva:

```text
PasswordAuthentication no
```

Reiniciar el servicio:

```bash
sudo systemctl restart ssh
```

> **Precaución.** Antes de aplicar este cambio, mantener una sesión SSH abierta y comprobar el acceso por clave desde una segunda sesión, para evitar quedar sin acceso al servidor.

### 8.2. Bloquear el acceso del usuario root

```text
PermitRootLogin no
```

### 8.3. Resumen de medidas

| Medida | Configuración | Objetivo |
|:--|:--|:--|
| Sin contraseña | `PasswordAuthentication no` | Eliminar ataques de fuerza bruta sobre contraseñas |
| Sin acceso root | `PermitRootLogin no` | Reducir la superficie de ataque |

---

## 9. Resumen técnico del servidor

| Parámetro | Valor |
|:--|:--|
| Sistema operativo | Ubuntu Server 22.04 LTS |
| Dirección IP | `192.168.1.151` |
| Usuario de trabajo | `trabajo` |
| Servicio SSH | Habilitado y funcionando |
| Autenticación | Clave pública configurada |
| Actualizaciones | Sistema actualizado |
| Permisos administrativos | Usuario con `sudo` verificado |
| Recuperación | Snapshots creados |

---

## 10. Resolución de incidencias

Durante el proceso se identificaron y resolvieron problemas reales:

| Incidencia | Diagnóstico y acción |
|:--|:--|
| Rechazo de claves públicas por parte del servidor SSH | Análisis con `ssh -v` y revisión del registro `/var/log/auth.log` |
| Permisos incorrectos en el directorio `.ssh` | Corrección de propietario y permisos (`700` y `600`) |
| Par de claves del cliente | Regeneración de las claves en Windows |
| Trazabilidad | Documentación completa del proceso y de las soluciones aplicadas |

---

## 11. Checklist final

| Criterio | Estado |
|:--|:--:|
| Máquina virtual funcional | Correcto |
| Ubuntu instalado y actualizado | Correcto |
| SSH operativo | Correcto |
| Acceso sin contraseña | Correcto |
| Usuario sudo funcional | Correcto |
| Documentación completa | Correcto |
| Snapshots creados | Correcto |
| Problemas documentados | Correcto |
| Autonomía demostrada | Correcto |

---

## 12. Conclusiones

El servidor Ubuntu 22.04 ha sido configurado correctamente desde cero, cumpliendo todos los requisitos técnicos y de documentación.

El sistema está actualizado, es accesible por SSH con autenticación por clave pública y cuenta con medidas de seguridad adicionales.

La documentación refleja un proceso completo y ordenado.

---

## 13. Anexo: referencia rápida de comandos

| Equipo | Acción | Comando |
|:--|:--|:--|
| Ubuntu | Actualizar el sistema | `sudo apt update && sudo apt upgrade -y` |
| Ubuntu | Crear usuario con sudo | `sudo adduser trabajo && sudo usermod -aG sudo trabajo` |
| Ubuntu | Consultar la dirección IP | `ip a` |
| Ubuntu | Asignar permisos a `.ssh` | `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` |
| Ubuntu | Reiniciar el servicio SSH | `sudo systemctl restart ssh` |
| Ubuntu | Consultar el estado del firewall | `sudo ufw status` |
| Windows | Generar el par de claves | `ssh-keygen -t ed25519` |
| Windows | Mostrar la clave pública | `type $env:USERPROFILE\.ssh\id_ed25519.pub` |
| Windows | Conectar al servidor | `ssh trabajo@192.168.1.151` |
| Windows | Conectar en modo detallado | `ssh -v trabajo@192.168.1.151` |

---

<div align="center">

**Darius Joanito Marinescu** · Administración de Sistemas · 20/09/2026

</div>
