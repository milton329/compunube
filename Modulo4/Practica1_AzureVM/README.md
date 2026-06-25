# Práctica 1 — IaaS: Azure Virtual Machines

**Universidad Autónoma de Occidente — UAO Virtual**  
**Materia:** Computación en la Nube  
**Profesor:** Oscar Mondragón  
**Estudiante:** Milton Giovanny Jaramillo Herrera  
**Fecha:** Junio 2026  

---

## Descripción

Implementación de Infrastructure as a Service (IaaS) usando Azure Virtual Machines. Se crean y configuran máquinas virtuales Linux y Windows en Azure Portal, se gestiona almacenamiento adicional mediante discos administrados y se despliegan entornos usando plantillas ARM de Azure. Se establecen conexiones remotas vía SSH (Linux) y RDP (Windows).

---

## Estructura del proyecto

```
Practica1_AzureVM/
├── README.md
└── entrega/
    ├── Informe_Practica_IaaS_AzureVM_MiltonJaramillo.pdf
    └── evidencia/                    ← 14 capturas de pantalla
```

---

## Ejercicios realizados

### Ejercicio 1 — VM Ubuntu + SSH

Creación de VM Ubuntu Server 24.04 LTS en Azure Portal con IP pública estática asignada via Azure CLI. Conexión SSH desde Windows PowerShell con corrección de permisos del archivo .pem via icacls.

**Parámetros:**
| Parámetro | Valor |
|---|---|
| Grupo de recursos | rg-practica4 |
| Nombre | vm-ubuntu-practica4 |
| Región | East US 2 |
| Imagen | Ubuntu Server 24.04 LTS Gen2 |
| Tamaño | Standard_D2s_v3 (2 vCPU, 8 GiB) |
| Autenticación | SSH Public Key |
| IP pública | 104.209.154.0 (estática) |

**Comandos:**
```bash
# Asignar IP pública via Azure CLI
az network public-ip create --resource-group rg-practica4 --name ip-publica-practica4 --sku Standard --allocation-method Static
az network nic ip-config update --resource-group rg-practica4 --nic-name vm-ubuntu-practica4583 --name ipconfig1 --public-ip-address ip-publica-practica4

# Corregir permisos de la clave en Windows
icacls "C:\Users\USUARIO\practica4.pem" /inheritance:r /grant:r "$($env:USERNAME):(R)"

# Conectarse por SSH
ssh -i "C:\Users\USUARIO\practica4.pem" azureuser@104.209.154.0
```

Evidencia: `evidencia/01_vm_ubuntu_creacion_clave.png` al `evidencia/04_ssh_conexion_vm.png`

---

### Ejercicio 2 — Disco de datos adicional

Creación y adjuntado de disco administrado SSD Premium de 4 GiB desde Azure Portal. Formateo y montaje dentro de la VM Ubuntu vía SSH.

```bash
lsblk                              # verificar disco /dev/sdc de 4 GB
sudo mkfs.ext4 /dev/sdc            # formatear con ext4
sudo mkdir /mnt/datadisk           # crear punto de montaje
sudo mount /dev/sdc /mnt/datadisk  # montar disco
df -h                              # verificar: /dev/sdc 3.9G en /mnt/datadisk
```

**Resultado:** disco montado con 3.7 GB disponibles. UUID: `1a3c2bab-0237-4954-b673-7acb1fbfb99a`

Evidencia: `evidencia/05_disco_adicional_portal.png` y `evidencia/06_disco_montado.png`

---

### Ejercicio 3 — VM con Docker via Template ARM

Despliegue de VM Ubuntu 22.04 con Docker Engine preinstalado usando la plantilla de inicio rápido `application-workloads/docker/docker-simple-on-ubuntu`. La plantilla creó 6 recursos automáticamente en una sola operación.

**Template:** `application-workloads/docker/docker-simple-on-ubuntu` (autor: postboxretinal)

**Recursos creados por la plantilla:**
- `MyDockerVM` — VM Ubuntu 22.04
- `MyDockerVM/installDocker` — extensión de instalación Docker
- `myVMNicD` — interfaz de red
- `MyVNETD` — red virtual
- `myPublicIPD` — IP pública
- `default-NSG` — grupo de seguridad

```bash
# Verificar Docker en la VM desplegada (IP: 20.96.2.138)
docker --version   # Docker version 29.6.0, build fb59821
docker ps          # sin contenedores activos (instalación limpia)
```

Evidencia: `evidencia/07_template_docker_seleccionado.png` al `evidencia/09_docker_verificacion.png`

---

### Ejercicio 4 — Exploración del catálogo de Templates ARM

Exploración del catálogo de plantillas de inicio rápido de Azure en "Implementación personalizada". Se identificó la plantilla `quickstarts/microsoft.compute/vm-simple-linux` (autor: bmoore-msft, actualización: 2025-03-13) que despliega una VM Linux simple con los últimos parches de seguridad.

> **Nota:** La suscripción Azure for Students tiene un límite de 4 vCPUs (familia DSv3) en East US 2. Con las dos VMs del ejercicio 1 y 3 activas se agotó la cuota, por lo que se documentó la exploración del catálogo sin despliegue simultáneo de una tercera VM.

Evidencia: `evidencia/10_template_linux_seleccionado.png`

---

### Ejercicio 5 — VM Windows Server 2025 + RDP

Creación de VM Windows Server 2025 Datacenter Gen2 en Azure Portal. Se agregó manualmente la regla de entrada RDP (puerto 3389, TCP) en el NSG. Conexión exitosa desde Windows 11 via Escritorio Remoto (mstsc).

**Parámetros:**
| Parámetro | Valor |
|---|---|
| Nombre | vm-windows-practica4 |
| Imagen | Windows Server 2025 Datacenter Gen2 |
| Tamaño | Standard_D2s_v3 |
| Autenticación | Usuario y contraseña |
| IP pública | 20.96.119.206 |
| Puerto RDP | 3389 |

**Conexión desde Windows:**
```
mstsc   →   equipo: 20.96.119.206   →   usuario: azureuser
```

Evidencia: `evidencia/11_vm_windows_desplegada.png` al `evidencia/14_rdp_windows_servidor.png`

---

## Recursos de Azure utilizados

| Recurso | Nombre | Tipo |
|---|---|---|
| Grupo de recursos | rg-practica4 | Resource Group |
| VM Linux | vm-ubuntu-practica4 | Virtual Machine |
| VM Docker | MyDockerVM | Virtual Machine |
| VM Windows | vm-windows-practica4 | Virtual Machine |
| IP pública Ubuntu | ip-publica-practica4 | Public IP Static |
| Disco de datos | disk-datos-practica4 | Managed Disk 4 GiB |

---

## Evidencias (14 capturas)

| # | Archivo | Descripción |
|---|---|---|
| 01 | 01_vm_ubuntu_creacion_clave.png | Popup descarga clave SSH durante creación VM |
| 02 | 02_vm_ubuntu_desplegada.png | Implementación VM Ubuntu completada |
| 03 | 03_vm_ubuntu_detalle.png | Detalles VM Ubuntu en portal (En ejecución) |
| 04 | 04_ssh_conexion_vm.png | Conexión SSH exitosa desde PowerShell |
| 05 | 05_disco_adicional_portal.png | Disco disk-datos-practica4 en Azure Portal |
| 06 | 06_disco_montado.png | Disco montado en /mnt/datadisk (df -h) |
| 07 | 07_template_docker_seleccionado.png | Template docker-simple-on-ubuntu en catálogo |
| 08 | 08_template_docker_desplegado.png | 6 recursos Docker creados en rg-practica4 |
| 09 | 09_docker_verificacion.png | Docker v29.6.0 verificado en MyDockerVM |
| 10 | 10_template_linux_seleccionado.png | Template vm-simple-linux explorado |
| 11 | 11_vm_windows_desplegada.png | VM Windows implementación completada |
| 12 | 12_vm_windows_detalle.png | Detalles VM Windows con IP pública |
| 13 | 13_rdp_windows_conectado.png | Escritorio RDP conectado a Windows Server 2025 |
| 14 | 14_rdp_windows_servidor.png | Server Manager con usuario azureuser |

---

## Notas técnicas

- Los archivos `.pem` en rutas con espacios requieren copiarse a una ruta simple y ajustar permisos con `icacls`
- La suscripción Azure for Students tiene límite de 3 IPs públicas y 4 vCPUs (DSv3) en East US 2
- Al crear una VM, seleccionar **"No se requiere redundancia de infraestructura"** para evitar errores de zona con Standard_D2s_v3
- El puerto RDP (3389) debe abrirse manualmente en el NSG si no se selecciona durante la creación de la VM

## Referencias

- Portal Azure: https://portal.azure.com/
- Azure Virtual Machines: https://learn.microsoft.com/en-us/azure/virtual-machines/
- Azure ARM Templates: https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/
- Azure CLI: https://learn.microsoft.com/en-us/cli/azure/
- Template Docker on Ubuntu: https://github.com/Azure/azure-quickstart-templates/tree/master/application-workloads/docker/docker-simple-on-ubuntu

---

> Eliminar recursos al terminar para conservar créditos de estudiante:
> ```bash
> az group delete --name rg-practica4 --yes --no-wait
> ```
