# Práctica 1 — IaaS: Azure Virtual Machines

**Universidad Autónoma de Occidente — UAO Virtual**  
**Materia:** Computación en la Nube  
**Profesor:** Oscar Mondragón  
**Estudiante:** Milton Giovanny Jaramillo Herrera  
**Fecha:** Junio 2026  

---

## Descripción

Implementación de **Infrastructure as a Service (IaaS)** usando **Azure Virtual Machines**. A diferencia de AKS (PaaS) trabajado en el Módulo 3, con IaaS se accede al nivel más bajo de abstracción de la nube: la máquina virtual como unidad de cómputo con control total sobre el sistema operativo, el almacenamiento y la red.

Se crean y configuran máquinas virtuales Linux y Windows en Azure Portal, se gestiona almacenamiento adicional con discos administrados, se automatizan despliegues mediante plantillas ARM y se establecen conexiones remotas vía SSH y RDP.

---

## Estructura del proyecto

```
Practica1_AzureVM/
├── README.md
└── entrega/
    ├── Informe_Practica_IaaS_AzureVM_MiltonJaramillo.pdf
    └── evidencia/                    # 14 capturas de pantalla numeradas
```

---

## Objetivos

- Crear una VM Ubuntu Server 24.04 en Azure Portal, asignar IP pública estática y conectarse por SSH desde Windows
- Agregar un disco de datos administrado, formatearlo y montarlo en Linux
- Desplegar una VM con Docker preinstalado usando la plantilla ARM `docker-simple-on-ubuntu`
- Explorar el catálogo de plantillas ARM de inicio rápido de Azure
- Crear una VM Windows Server 2025 y conectarse mediante escritorio remoto (RDP)

---

## Herramientas utilizadas

| Herramienta | Versión | Uso |
|---|---|---|
| Azure Portal | — | Creación y gestión visual de VMs |
| Azure CLI | 2.87.0 | Gestión de recursos desde Windows PowerShell |
| Azure Virtual Machines | — | IaaS — VMs Linux y Windows |
| Azure Managed Disks | SSD Premium | Almacenamiento de datos adicional |
| Azure ARM Templates | — | Despliegue automatizado de infraestructura |
| SSH (OpenSSH) | Windows built-in | Conexión remota a VMs Linux |
| RDP (mstsc) | Windows built-in | Escritorio remoto a VM Windows |
| icacls | Windows built-in | Gestión de permisos de archivos .pem |

---

## Arquitectura implementada

```
[Windows 11 — Azure CLI + PowerShell + mstsc]
        │
        ▼
[Azure — Grupo de recursos: rg-practica4 — East US 2]
        │
        ├── vm-ubuntu-practica4  (Ubuntu 24.04 LTS, Standard_D2s_v3)
        │       ├── IP pública: 104.209.154.0  ← SSH port 22
        │       ├── IP privada: 10.0.0.4
        │       └── disk-datos-practica4 (SSD Premium 4 GiB) → /mnt/datadisk
        │
        ├── MyDockerVM  (Ubuntu 22.04 LTS, Standard_D2s_v3)
        │       ├── IP pública: 20.96.2.138  ← SSH port 22
        │       └── Docker Engine 29.6.0 (instalado via extensión ARM)
        │
        └── vm-windows-practica4  (Windows Server 2025 Datacenter, Standard_D2s_v3)
                ├── IP pública: 20.96.119.206  ← RDP port 3389
                └── IP privada: 10.0.0.5
```

---

## Ejercicio 1 — VM Ubuntu Server + SSH

Se creó la VM `vm-ubuntu-practica4` en Azure Portal. La cuenta Azure for Students no asigna IP pública por defecto, por lo que se asignó una IP pública estática via Azure CLI. El archivo `.pem` requirió corrección de permisos con `icacls` ya que SSH en Windows rechaza claves con permisos demasiado abiertos.

**Parámetros de configuración:**
| Parámetro | Valor |
|---|---|
| Grupo de recursos | rg-practica4 |
| Nombre | vm-ubuntu-practica4 |
| Región | East US 2 |
| Imagen | Ubuntu Server 24.04 LTS Gen2 |
| Tamaño | Standard_D2s_v3 (2 vCPU, 8 GiB RAM) |
| Disponibilidad | No se requiere redundancia de infraestructura |
| Autenticación | Clave pública SSH (RSA) |
| Usuario | azureuser |
| IP pública | 104.209.154.0 (estática, SKU Standard) |

**Comandos:**
```bash
# Asignar IP pública via Azure CLI
az network public-ip create --resource-group rg-practica4 --name ip-publica-practica4 --sku Standard --allocation-method Static
az network nic ip-config update --resource-group rg-practica4 --nic-name vm-ubuntu-practica4583 --name ipconfig1 --public-ip-address ip-publica-practica4

# Verificar IP asignada
az vm show --resource-group rg-practica4 --name vm-ubuntu-practica4 --show-details --query publicIps -o tsv

# Corregir permisos de la clave .pem en Windows
Copy-Item "...\vm-ubuntu-practica4_key.pem" "C:\Users\USUARIO\practica4.pem"
icacls "C:\Users\USUARIO\practica4.pem" /inheritance:r /grant:r "$($env:USERNAME):(R)"

# Conectarse por SSH
ssh -i "C:\Users\USUARIO\practica4.pem" azureuser@104.209.154.0
```

**Resultado obtenido:**
```
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.17.0-1018-azure x86_64)
System load:  0.0    Usage of /: 5.8% of 28.02GB    Memory usage: 3%
IPv4 address for eth0: 10.0.0.4
azureuser@vm-ubuntu-practica4:~$
```

Evidencia: `evidencia/01_vm_ubuntu_creacion_clave.png` al `evidencia/04_ssh_conexion_vm.png`

---

## Ejercicio 2 — Disco de datos adicional

Se agregó un disco administrado SSD Premium de 4 GiB (`disk-datos-practica4`) desde la sección **Discos** de la VM en Azure Portal. Al aplicar el cambio la VM se reinició. El disco apareció como `/dev/sdc` sin particionar.

```bash
# Verificar disco nuevo
lsblk
# NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
# sdc       8:32   0    4G  0 disk          ← disco nuevo sin montar

# Formatear con ext4
sudo mkfs.ext4 /dev/sdc

# Crear punto de montaje y montar
sudo mkdir /mnt/datadisk
sudo mount /dev/sdc /mnt/datadisk

# Verificar montaje
df -h
# /dev/sdc   3.9G   24K  3.7G   1% /mnt/datadisk
```

**Resultado:** disco de 3.9 GB montado en `/mnt/datadisk` con 3.7 GB disponibles.  
UUID del sistema de archivos: `1a3c2bab-0237-4954-b673-7acb1fbfb99a`

Evidencia: `evidencia/05_disco_adicional_portal.png` y `evidencia/06_disco_montado.png`

---

## Ejercicio 3 — VM con Docker via Plantilla ARM

Se usó la plantilla de inicio rápido `application-workloads/docker/docker-simple-on-ubuntu` desde **Implementación personalizada** en Azure Portal. La plantilla crea automáticamente todos los recursos necesarios en una sola operación, demostrando el valor de la infraestructura como código (IaC).

**Plantilla:** `application-workloads/docker/docker-simple-on-ubuntu`  
**Autor:** postboxretinal | **Última actualización:** 2026-04-22

**Parámetros configurados:**
| Parámetro | Valor |
|---|---|
| Nombre VM | MyDockerVM |
| DNS | vm-docker-practica4.eastus2.cloudapp.azure.com |
| Tamaño | Standard_D2s_v3 |
| OS | Ubuntu 22.04 LTS Gen2 |
| Usuario | azureuser |

**Recursos creados por la plantilla (6 en total):**
- `MyDockerVM` — VM Ubuntu 22.04
- `MyDockerVM/installDocker` — extensión que instala Docker Engine
- `myVMNicD` — interfaz de red
- `MyVNETD` — red virtual
- `myPublicIPD` — dirección IP pública
- `default-NSG` — grupo de seguridad de red

```bash
# Obtener IP de la VM desplegada
az vm show -g rg-practica4 -n MyDockerVM --show-details --query publicIps -o tsv
# 20.96.2.138

# Conectarse y verificar Docker
ssh -i "C:\Users\USUARIO\docker-practica4.pem" azureuser@20.96.2.138
docker --version   # Docker version 29.6.0, build fb59821
docker ps          # sin contenedores activos (instalación limpia)
```

Evidencia: `evidencia/07_template_docker_seleccionado.png` al `evidencia/09_docker_verificacion.png`

---

## Ejercicio 4 — Exploración del catálogo de plantillas ARM

Se exploró el catálogo de plantillas de inicio rápido de Azure en la sección **Implementación personalizada** del Portal. Se identificó la plantilla `quickstarts/microsoft.compute/vm-simple-linux` (autor: bmoore-msft, actualización: 2025-03-13) que despliega una VM Linux simple con el mínimo de parámetros y los últimos parches de seguridad.

> **Restricción de cuota Azure for Students:** Límite de 4 vCPU familia DSv3 en East US 2. Con `vm-ubuntu-practica4` y `MyDockerVM` activas (4 vCPU totales), no fue posible desplegar una tercera VM simultáneamente. Se documentó la exploración del catálogo de templates disponibles (VMs Linux, Windows, aplicaciones web, bases de datos, alta disponibilidad).

Evidencia: `evidencia/10_template_linux_seleccionado.png`

---

## Ejercicio 5 — VM Windows Server 2025 + RDP

Se creó la VM `vm-windows-practica4` con Windows Server 2025 Datacenter Gen2. El puerto RDP (3389) no se abrió durante la creación por lo que fue necesario agregar manualmente una regla de entrada en el NSG. La contraseña inicial requirió reset via Azure CLI por incompatibilidad con los requisitos de complejidad de Windows Server.

**Parámetros de configuración:**
| Parámetro | Valor |
|---|---|
| Nombre | vm-windows-practica4 |
| Imagen | Windows Server 2025 Datacenter Gen2 |
| Tamaño | Standard_D2s_v3 (2 vCPU, 8 GiB RAM) |
| CPU | Intel Xeon Platinum 8272CL @ 2.60 GHz |
| Usuario | azureuser |
| IP pública | 20.96.119.206 |
| Puerto RDP | 3389 (agregado manualmente al NSG) |

**Regla NSG para RDP:**
```
Puerto: 3389 | Protocolo: TCP | Prioridad: 900 | Acción: Permitir
```

**Reset de contraseña via Azure CLI:**
```bash
az vm user update --resource-group rg-practica4 --name vm-windows-practica4 --username azureuser --password "Practica4#2026Az"
```

**Conexión desde Windows 11:**
```
Win+R → mstsc → Equipo: 20.96.119.206 → Usuario: azureuser
```

**Resultado:** Escritorio de Windows Server 2025 con Server Manager activo.  
Sistema operativo confirmado: Microsoft Windows Server 2025 Datacenter.  
Remote Desktop: habilitado | Microsoft Defender Firewall: activo

Evidencia: `evidencia/11_vm_windows_desplegada.png` al `evidencia/14_rdp_windows_servidor.png`

---

## Recursos de Azure utilizados

| Recurso | Nombre | Tipo | Estado |
|---|---|---|---|
| Grupo de recursos | rg-practica4 | Resource Group | Activo |
| VM Linux | vm-ubuntu-practica4 | Virtual Machine | Completado |
| VM Docker | MyDockerVM | Virtual Machine | Activo |
| VM Windows | vm-windows-practica4 | Virtual Machine | Activo |
| IP pública Ubuntu | ip-publica-practica4 | Public IP Static | Asignada |
| Disco de datos | disk-datos-practica4 | Managed Disk SSD 4 GiB | Montado |

---

## Evidencias (14 capturas)

| # | Archivo | Descripción |
|---|---|---|
| 01 | 01_vm_ubuntu_creacion_clave.png | Popup generación par de claves SSH al crear VM |
| 02 | 02_vm_ubuntu_desplegada.png | Implementación VM Ubuntu completada — 4 recursos OK |
| 03 | 03_vm_ubuntu_detalle.png | Detalles VM Ubuntu: En ejecución, IP privada 10.0.0.4 |
| 04 | 04_ssh_conexion_vm.png | SSH exitoso: Ubuntu 24.04.4 LTS en Azure (IP 104.209.154.0) |
| 05 | 05_disco_adicional_portal.png | disk-datos-practica4: SSD Premium 4 GiB, LUN 0 en Portal |
| 06 | 06_disco_montado.png | mkfs.ext4 + df -h: /dev/sdc 3.9G montado en /mnt/datadisk |
| 07 | 07_template_docker_seleccionado.png | Template docker-simple-on-ubuntu en catálogo ARM |
| 08 | 08_template_docker_desplegado.png | 6 recursos Docker creados en rg-practica4 — todos OK |
| 09 | 09_docker_verificacion.png | Docker version 29.6.0 + docker ps en MyDockerVM |
| 10 | 10_template_linux_seleccionado.png | Template vm-simple-linux explorado — autor bmoore-msft |
| 11 | 11_vm_windows_desplegada.png | VM Windows Server 2025: 3 recursos creados — todos OK |
| 12 | 12_vm_windows_detalle.png | Propiedades VM Windows: IP 20.96.119.206, Standard_D2s_v3 |
| 13 | 13_rdp_windows_conectado.png | Escritorio RDP conectado a 20.96.119.206 desde Windows 11 |
| 14 | 14_rdp_windows_servidor.png | Server Manager: OS Windows Server 2025, usuario azureuser |

---

## Comandos de referencia

```bash
# Login Azure CLI
az login --use-device-code

# Listar VMs del grupo de recursos
az vm list --resource-group rg-practica4 --output table

# Obtener IP pública de una VM
az vm show -g rg-practica4 -n <nombre-vm> --show-details --query publicIps -o tsv

# Asignar IP pública a una NIC existente
az network public-ip create -g rg-practica4 --name <nombre-ip> --sku Standard --allocation-method Static
az network nic ip-config update -g rg-practica4 --nic-name <nombre-nic> --name ipconfig1 --public-ip-address <nombre-ip>

# Resetear contraseña de VM Windows
az vm user update -g rg-practica4 -n <nombre-vm> --username azureuser --password "<nueva-contraseña>"

# Listar IPs públicas disponibles
az network public-ip list --output table

# Eliminar todos los recursos al terminar
az group delete --name rg-practica4 --yes --no-wait
```

---

## Notas técnicas

- Seleccionar **"No se requiere redundancia de infraestructura"** al crear VMs para evitar error de zona con Standard_D2s_v3 en East US 2
- Los archivos `.pem` en rutas con espacios deben copiarse a `C:\Users\USUARIO\` y ajustar permisos con `icacls /inheritance:r`
- La cuenta Azure for Students tiene límite de **3 IPs públicas** y **4 vCPUs familia DSv3** en East US 2
- El puerto RDP (3389) debe abrirse manualmente en el NSG si no se selecciona durante la creación de la VM
- Windows Server requiere contraseñas con mayúsculas, minúsculas, números y símbolos de al menos 12 caracteres

> **Importante:** Las VMs consumen créditos aunque estén detenidas (el disco y la IP pública siguen facturando). Eliminar el grupo de recursos después de cada sesión:
> ```bash
> az group delete --name rg-practica4 --yes --no-wait
> ```
