# Computación en la Nube — UAO Virtual

Repositorio de prácticas de la especialización en Computación en la Nube.

**Estudiante:** Milton Jaramillo
**Docente:** Prof. Heberth Fabián Martínez Velásquez
**Institución:** UAO Virtual
**Año:** 2026

---

## Estructura del repositorio

```
compunube/
├── Instaladores/               # Instaladores usados en las prácticas (no versionados)
├── Modulo1/
│   ├── guias/                  # Guías del módulo 1
│   └── entrega/                # Informe, evidencias y video
│       ├── Informe_Practica1_Vagrant_MiltonJaramillo_v2.pdf
│       └── Evidencia/          # Capturas de pantalla del proceso
├── Modulo2/
│   ├── documentos/             # Guías del módulo 2
│   ├── Practica1_LXD/
│   │   └── entrega/
│   │       ├── Informe_LXD_Milton_Jaramillo.pdf
│   │       └── evidencia/      # 22 capturas del proceso LXD
│   ├── Practica2_M2A3/
│   │   └── entrega/
│   │       ├── Informe_HAProxy_LXD_Milton_Jaramillo.pdf
│   │       └── evidencia/      # 9 capturas del proceso HAProxy + JMeter
│   └── Microproyecto1_HAProxy/
│       └── entrega/
│           ├── Microproyecto1_ClusterLXD_HAProxy_Milton_Jaramillo.pdf
│           └── evidencia/      # 11 capturas: cluster, contenedores, HAProxy, páginas UAO, JMeter
├── Modulo3/
│   ├── documentos/             # Guías del módulo 3
│   └── Microproyecto2_kubernetes_azure/
│       ├── README.md
│       └── entrega/
│           ├── Microproyecto2_kubernetes_azure_Milton_Jaramillo.pdf
│           └── evidencia/      # 17 capturas del proceso AKS
└── README.md
```

---

## Módulo 1 — Práctica 1: Configuración entorno virtualizado con Vagrant

### Descripción
Configuración de un entorno virtualizado usando **Vagrant** y **VirtualBox** con dos máquinas virtuales Ubuntu comunicadas en red privada.

### Herramientas utilizadas
| Herramienta | Versión |
|---|---|
| VirtualBox | 7.2.8 |
| Vagrant | 2.4.9 |
| Plugin vagrant-vbguest | 0.32.0 |
| Ubuntu (bento/ubuntu-20.04) | 20.04 LTS |

### Máquinas virtuales configuradas
| VM | IP | Rol |
|---|---|---|
| servidorUbuntu | 192.168.100.3 | Servidor |
| clienteUbuntu | 192.168.100.2 | Cliente |

### Cómo levantar el entorno
```bash
# Clonar el repositorio
git clone https://github.com/milton329/compunube.git

# Ir a la carpeta del módulo 1
cd "Modulo 1"

# Levantar las VMs
vagrant up

# Conectarse al servidor
vagrant ssh servidorUbuntu

# Conectarse al cliente
vagrant ssh clienteUbuntu

# Ver estado de las VMs
vagrant status

# Apagar las VMs
vagrant halt
```

### Box publicado en Vagrant Cloud
**https://app.vagrantup.com/milton329/boxes/ubuntu-20.04-practica1**

---

## Módulo 2 — Práctica 1: Linux Containers (LXD)

### Descripción
Instalación y configuración de **LXD** sobre las VMs Vagrant. Creación y gestión de contenedores Linux, snapshots, servidor web Apache y configuración SSH.

### Herramientas utilizadas
| Herramienta | Versión |
|---|---|
| LXD | 4.0 |
| Apache2 | 2.4.41 |
| OpenSSH Server | 8.2p1 |
| Ubuntu 20.04 | LTS |

### Contenedores configurados
| Contenedor | Puerto Web | Puerto SSH | Descripción |
|---|---|---|---|
| web | 5080 | — | Servidor Apache de prueba |
| server | 5081 | 2223 | Servidor final con Apache + SSH |

### Comandos principales
```bash
# Instalar LXD
sudo apt-get install lxd -y
newgrp lxd
lxd init --auto

# Crear contenedor
lxc launch ubuntu:20.04 server

# Reenvío de puertos
lxc config device add server myport80 proxy listen=tcp:192.168.100.3:5081 connect=tcp:127.0.0.1:80

# SSH al contenedor
ssh -p 2223 remoto@192.168.100.3

# Transferir archivo
scp -P 2223 archivo.txt remoto@192.168.100.3:~/
```

---

## Módulo 2 — Práctica 2 (M2A3): Balanceo de Carga con HAProxy y LXD

### Descripción
Implementación de un **balanceador de carga** usando **HAProxy** y contenedores **LXD** sobre las VMs Vagrant. Dos servidores Apache reciben tráfico balanceado via Round Robin, con pruebas de fallo y pruebas de carga con JMeter.

### Herramientas utilizadas
| Herramienta | Versión |
|---|---|
| LXD | 4.0.9 |
| HAProxy | 2.0.33 |
| Apache2 | 2.4.41 |
| Apache JMeter | 5.6.3 |
| Java | 1.8.0_491 |

### Arquitectura implementada
```
[Anfitrión Windows]
        │  http://192.168.100.3
        ▼
[servidorUbuntu VM - 192.168.100.3]
        │
        ▼
[Contenedor haproxy - 10.149.16.127]  ← HAProxy Round Robin
        ├──▶ [Contenedor web1 - 10.149.16.40]   Apache "Hello from web1"
        └──▶ [Contenedor web2 - 10.149.16.101]  Apache "Hello from web2"
```

### Comandos principales
```bash
# Crear los 3 contenedores de una vez
lxc launch ubuntu:20.04 web1 && lxc launch ubuntu:20.04 web2 && lxc launch ubuntu:20.04 haproxy

# Instalar Apache en web1 y web2
lxc exec web1 -- bash -c "apt install -y apache2 && echo 'Hello from web1' > /var/www/html/index.html && systemctl start apache2"
lxc exec web2 -- bash -c "apt install -y apache2 && echo 'Hello from web2' > /var/www/html/index.html && systemctl start apache2"

# Instalar HAProxy
lxc exec haproxy -- bash -c "apt install -y haproxy"

# Port forwarding para acceso desde el anfitrión
lxc config device add haproxy http proxy listen=tcp:0.0.0.0:80 connect=tcp:10.149.16.127:80

# Verificar balanceo
curl 10.149.16.127 && curl 10.149.16.127
```

### Accesos
| Recurso | URL |
|---|---|
| Balanceador de carga | http://192.168.100.3/ |
| Dashboard HAProxy | http://192.168.100.3/haproxy?stats (admin/admin) |

### Resultados JMeter
| Métrica | Valor |
|---|---|
| Total peticiones | 10,000 |
| Throughput | 379.1 req/seg |
| Tiempo promedio | 358 ms |
| Error rate | 0.45% |

---

## Módulo 2 — Microproyecto 1 (M2A4): Cluster LXD + HAProxy + Servidores Backup

### Descripción
Implementación de un **cluster LXD multi-nodo** con balanceo de carga **HAProxy**, servidores de producción y backup, página de error 503 personalizada con diseño UAO y pruebas de carga con JMeter.

Los contenedores se distribuyen entre dos nodos Vagrant: **web1 y web2** (producción) en servidorUbuntu, **web3 y web4** (backup) en clienteUbuntu. El balanceador HAProxy atiende requests en Round Robin entre producción; cuando ambos caen, activa automáticamente los backups. Si todos los servidores están caídos, muestra una página 503 personalizada con identidad visual UAO.

Las páginas web de cada servidor tienen diseño propio: **verde** para producción y **naranja** para backup, con header UAO Virtual, gradiente institucional y tarjetas con nombre del servidor. Se usó base64 para transferir los archivos HTML a los contenedores al no tener `/vagrant` montado.

El reto principal fue la **comunicación cross-node**: los contenedores en clienteUbuntu no son alcanzables directamente desde el contenedor haproxy en servidorUbuntu. Se resolvió con proxy devices LXD en clienteUbuntu que exponen web3 y web4 en la IP de la VM (192.168.100.2:8083 y 8084).

### Herramientas utilizadas
| Herramienta | Versión |
|---|---|
| LXD Cluster | 4.0.9 |
| HAProxy | 2.0.33 |
| Apache2 | 2.4.41 |
| Apache JMeter | 5.6.3 |
| Java | 1.8.0_491 |

### Arquitectura implementada
```
[Anfitrión Windows]
        │  http://192.168.100.3
        ▼
[servidorUbuntu - 192.168.100.3]  ←── Nodo 1 del Cluster LXD
        │  (haproxy, web1, web2)
        ▼
[Contenedor haproxy]  ← HAProxy Round Robin
        ├──▶ [web1 - servidorUbuntu]  Produccion (verde)
        ├──▶ [web2 - servidorUbuntu]  Produccion (verde)
        ├──▶ [web3 - clienteUbuntu]   Backup (naranja)  ← 192.168.100.2:8083
        └──▶ [web4 - clienteUbuntu]   Backup (naranja)  ← 192.168.100.2:8084

[clienteUbuntu - 192.168.100.2]  ←── Nodo 2 del Cluster LXD
        │  (web3, web4)
```

### Configuración del Cluster LXD
```bash
# Bootstrap nodo 1 (servidorUbuntu) con preseed
cat <<EOF | lxd init --preseed
config:
  core.https_address: 192.168.100.3:8443
  core.trust_password: clustersecret
cluster:
  server_name: servidorUbuntu
  enabled: true
EOF

# Unir nodo 2 (clienteUbuntu)
cat <<EOF | lxd init --preseed
cluster:
  enabled: true
  server_name: clienteUbuntu
  server_address: 192.168.100.2:8443
  cluster_address: 192.168.100.3:8443
  cluster_password: clustersecret
  cluster_certificate: |
    <certificado del nodo 1>
EOF
```

### Configuración HAProxy (backup + error personalizado)
```
frontend http
    bind *:80
    default_backend web-backend

backend web-backend
    balance roundrobin
    stats enable
    stats auth admin:admin
    stats uri /haproxy?stats
    server web1 <ip-web1>:80 check
    server web2 <ip-web2>:80 check
    server web3 192.168.100.2:8083 check backup
    server web4 192.168.100.2:8084 check backup
    errorfile 503 /etc/haproxy/errors/503.http
```

### Port forwarding cross-node (clienteUbuntu)
```bash
# Exponer web3 y web4 al exterior para que HAProxy los alcance
lxc config device add web3 http proxy listen=tcp:192.168.100.2:8083 connect=tcp:127.0.0.1:80
lxc config device add web4 http proxy listen=tcp:192.168.100.2:8084 connect=tcp:127.0.0.1:80
```

### Accesos
| Recurso | URL |
|---|---|
| Balanceador de carga | http://192.168.100.3/ |
| Dashboard HAProxy | http://192.168.100.3/haproxy?stats (admin/admin) |

### Resultados JMeter
| Escenario | Muestras | Promedio | Throughput | Error % |
|---|---|---|---|---|
| Normal (100 usuarios, 10 loops) | 1,000 | 39 ms | 104.1 req/seg | 0.00% |
| Estrés (500 usuarios, 20 loops) | 11,000 | 891 ms | 67.6 req/seg | 6.24% |

### Evidencias (11 capturas)
| # | Pantallazo | Descripción |
|---|---|---|
| 01 | cluster_lxd_activo | Cluster LXD con ambos nodos ONLINE |
| 02 | contenedores_distribuidos | lxc list mostrando contenedores en ambos nodos |
| 03 | haproxy_stats_produccion_UP | Dashboard HAProxy todos los servidores UP |
| 04 | backup_servidores_caidos | HAProxy activando backup al caer producción |
| 05 | pagina_error_personalizada | Página 503 UAO cuando todos los servidores caen |
| 06 | pagina_web1_produccion_UAO | Página web1 verde con diseño UAO |
| 07 | pagina_web2_produccion_UAO | Página web2 verde con diseño UAO |
| 08 | pagina_web3_backup_UAO | Página web3 naranja (backup) con diseño UAO |
| 09 | pagina_web4_backup_UAO | Página web4 naranja (backup) con diseño UAO |
| 10 | jmeter_escenario_normal | Summary Report 1000 peticiones, 0% error |
| 11 | jmeter_escenario_estres | Summary Report 11000 peticiones bajo estrés |

---

## Módulo 3 — Microproyecto 2: Kubernetes en Azure (AKS)

### Descripción
Implementación de un clúster de **Azure Kubernetes Service (AKS)** de dos nodos en la nube de Microsoft Azure. Se despliegan dos aplicaciones (clasificación de imágenes con deep learning y Grafana como herramienta de monitoreo) y se demuestra el uso de los servicios de supervisión que provee AKS mediante Azure Monitor.

AKS es un servicio gestionado de Kubernetes que elimina la complejidad operativa del plano de control, permitiendo enfocarse en el despliegue y gestión de aplicaciones en contenedores.

### Objetivos
- Crear y configurar un clúster AKS desde Azure Portal
- Verificar el funcionamiento del clúster mediante Cloud Shell y Azure CLI
- Desplegar una aplicación de clasificación de imágenes con deep learning
- Desplegar una aplicación de interés (Grafana)
- Demostrar el monitoreo y supervisión del clúster con Azure Monitor

### Herramientas utilizadas
| Herramienta | Versión | Uso |
|---|---|---|
| Azure Kubernetes Service (AKS) | Kubernetes 1.34.8 | Orquestador de contenedores en la nube |
| Azure CLI | 2.87.0 | Gestión del clúster desde Windows |
| kubectl | 1.36.1 | Interacción con el clúster Kubernetes |
| TensorFlow / Keras | tensorflow:latest-jupyter | Modelo MobileNetV2 para clasificación de imágenes |
| Grafana | 13.0.2 | Dashboard de monitoreo y visualización |
| Azure Monitor | — | Supervisión nativa de métricas AKS |

### Arquitectura implementada
```
[Windows 11 — Azure CLI + kubectl]
        │
        ▼
[Azure Kubernetes Service — East US 2]
        │
        ├── Plano de control (gestionado por Azure)
        │       ├── API Server  (CPU: 6%, Memoria: 15%)
        │       └── etcd, scheduler, controller-manager
        │
        └── Grupo de nodos — agentpool (2 nodos Standard_D2s_v3)
                ├── aks-agentpool-vmss000000  (CPU: 41%, Mem: 22%)
                └── aks-agentpool-vmss000001  (CPU: 47%, Mem: 22%)
                        │
                        ├── [image-classifier]  tensorflow/tensorflow:latest-jupyter
                        │       └── Service LoadBalancer → 52.177.38.104:8888
                        │
                        └── [grafana]  grafana/grafana:latest
                                └── Service LoadBalancer → 52.184.200.57:3000
```

### Configuración del clúster
| Parámetro | Valor |
|---|---|
| Nombre | aks-microproyecto2 |
| Grupo de recursos | rg-microproyecto2 |
| Región | East US 2 |
| Versión Kubernetes | 1.34.8 |
| Nodos | 2 × Standard_D2s_v3 (2 vCPU, 8 GiB RAM) |
| Sistema operativo | Ubuntu 22.04 LTS |
| Runtime | containerd 1.7.31 |
| Plan de tarifa | Gratis |
| Red | Azure CNI (Superposición) |

> **Nota para cuentas de estudiante:** La Serie B no es compatible con AKS. El tamaño `Standard_DS2_v2` puede estar restringido según la región. Se recomienda usar `Standard_D2s_v3` con escalado manual de 2 nodos para mantenerse dentro de la cuota de 4 vCPU de Azure for Students.

### Paso 1 — Creación y verificación del clúster

**Desde Azure Cloud Shell:**
```bash
# Obtener credenciales del clúster
az aks get-credentials --resource-group rg-microproyecto2 --name aks-microproyecto2

# Verificar nodos
kubectl get nodes
kubectl get nodes -o wide
```

**Desde Azure CLI en Windows (instalación):**
```bash
# Instalar Azure CLI
# Descargar desde: https://aka.ms/installazurecliwindows

# Verificar instalación
az --version

# Iniciar sesión (sin cuenta Microsoft personal, usar device code)
az login --use-device-code

# Instalar kubectl
az aks install-cli

# Obtener credenciales y verificar nodos
az aks get-credentials --resource-group rg-microproyecto2 --name aks-microproyecto2
kubectl get nodes
```

### Paso 2 — Aplicación de clasificación de imágenes

Se desplegó un servidor Jupyter con TensorFlow que implementa clasificación de imágenes usando el modelo preentrenado **MobileNetV2** (ImageNet, 1000 clases).

```bash
# Desplegar
kubectl create deployment image-classifier --image=tensorflow/tensorflow:latest-jupyter
kubectl expose deployment image-classifier --type=LoadBalancer --port=8888

# Obtener IP pública
kubectl get services

# Obtener token de acceso a Jupyter
kubectl logs <nombre-del-pod>
```

**Código de clasificación ejecutado en Jupyter:**
```python
import tensorflow as tf
from tensorflow.keras.applications import MobileNetV2
from tensorflow.keras.applications.mobilenet_v2 import preprocess_input, decode_predictions
import numpy as np
from PIL import Image

model = MobileNetV2(weights='imagenet')

img_array = np.random.randint(50, 200, (224, 224, 3), dtype=np.uint8)
img = Image.fromarray(img_array)
x = np.expand_dims(img_array.astype('float32'), axis=0)
x = preprocess_input(x)

preds = model.predict(x)
for i, (_, label, score) in enumerate(decode_predictions(preds, top=3)[0]):
    print(f"  {i+1}. {label}: {score:.4f}")
```

**Resultado obtenido:**
```
1. velvet: 0.8226
2. wool: 0.1151
3. lampshade: 0.0120
```

### Paso 3 — Aplicación de interés: Grafana

Se desplegó **Grafana v13.0.2** como plataforma de visualización y monitoreo de métricas.

```bash
# Desplegar
kubectl create deployment grafana --image=grafana/grafana:latest
kubectl expose deployment grafana --type=LoadBalancer --port=3000

# Verificar
kubectl get pods && kubectl get services
```

**Acceso:** `http://<EXTERNAL-IP>:3000`  
**Credenciales iniciales:** `admin` / `admin`

### Paso 4 — Supervisión y monitoreo con Azure Monitor

Se utilizó **Azure Monitor** integrado en AKS para visualizar métricas del clúster en tiempo real.

**Métricas observadas:**
| Métrica | Valor | Descripción |
|---|---|---|
| CPU Usage Percentage (Max) | 47% | Uso máximo de CPU en los nodos |
| API Server Memory Usage | 15% | Memoria del servidor de API Kubernetes |
| Nodos listos | 2/2 | Ambos nodos en estado Ready |
| Pods en ejecución | 25 | Total de pods activos en el clúster |
| Ancho de banda de disco OS | 68% | Uso de I/O en disco del sistema |

**Ruta en Azure Portal:** `aks-microproyecto2 → Supervisión → Métricas`

### Comandos de referencia general
```bash
# Estado general del clúster
kubectl get nodes
kubectl get pods --all-namespaces
kubectl get services

# Ver logs de un pod
kubectl logs <nombre-pod>

# Describir un recurso
kubectl describe pod <nombre-pod>
kubectl describe service <nombre-service>

# Eliminar recursos
kubectl delete deployment <nombre>
kubectl delete service <nombre>

# Eliminar el clúster completo (liberar créditos)
az group delete --name rg-microproyecto2 --yes --no-wait
```

> **Importante:** AKS consume créditos de Azure rápidamente (~$1.50/hora con 2 nodos Standard_D2s_v3). Se recomienda eliminar el grupo de recursos después de cada sesión y recrear el clúster aproximadamente 1 día antes de la sustentación.

### Evidencias (17 capturas)
| # | Archivo | Descripción |
|---|---|---|
| 01 | 01_cluster_aks_creado.png | Implementación completada en Azure Portal |
| 02 | 02_cluster_aks_detalle.png | Detalle del clúster en ejecución (estado, región, versión) |
| 03 | 03_cloud_shell_abierto.png | Cloud Shell de Azure abierto en el navegador |
| 04 | 04_get_credentials.png | Credenciales del clúster obtenidas vía Cloud Shell |
| 05 | 05_kubectl_get_nodes.png | 2 nodos en estado Ready desde Cloud Shell |
| 06 | 06_kubectl_get_nodes_wide.png | Detalle completo de nodos (IP, OS, runtime) |
| 07 | 07_azure_cli_version.png | Azure CLI v2.87.0 instalado en Windows |
| 08 | 08_kubectl_get_nodes_cli_windows.png | 2 nodos Ready verificados desde CLI en Windows |
| 09 | 09_app_clasificacion_jupyter.png | Jupyter Notebook corriendo en AKS con IP pública |
| 10 | 10_clasificacion_imagenes_resultado_1.png | Ejecución del modelo MobileNetV2 con predicciones |
| 11 | 11_grafana_servicio.png | Servicio Grafana expuesto con IP pública asignada |
| 12 | 12_grafana_dashboard.png | Dashboard Grafana v13 en funcionamiento |
| 13 | 13_aks_supervision_parte1.png | Supervisión AKS — nodos, pods, eventos |
| 14 | 13_aks_supervision_parte2.png | Supervisión AKS — CPU por nodo, memoria, disco |
| 15 | 14_aks_metricas_memoria.png | Gráfica API Server Memory Usage (15%) |
| 16 | 15_aks_metricas_cpu_memoria.png | Gráfica CPU Usage con pico de actividad (47%) |

---

## Notas importantes
- Se requiere desactivar **Hyper-V** en Windows 11 para que VirtualBox funcione correctamente:
  ```
  bcdedit /set hypervisorlaunchtype off
  ```
  Luego reiniciar el equipo.
- Los instaladores, videos y archivos grandes no están versionados en git. Se encuentran en las carpetas locales correspondientes.
