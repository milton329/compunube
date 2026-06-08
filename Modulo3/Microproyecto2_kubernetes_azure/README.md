# Microproyecto 2 — Kubernetes en Azure (AKS)

**Universidad Autónoma de Occidente — UAO Virtual**  
**Materia:** Computación en la Nube  
**Profesor:** Oscar Mondragón  
**Estudiante:** Milton Giovanny Jaramillo Herrera  
**Fecha:** Junio 2026  

---

## Descripción

Implementación de un clúster de Azure Kubernetes Service (AKS) con despliegue de aplicaciones y demostración de supervisión y monitoreo en la nube de Azure.

---

## Estructura del proyecto

```
Microproyecto2_kubernetes_azure/
├── README.md
├── entrega/          # Informe y documentos de entrega
└── evidencia/        # Capturas de pantalla del proceso
```

---

## Actividades realizadas

### 1. Implementación del clúster AKS

Se creó un clúster de Azure Kubernetes Service con las siguientes características:

| Parámetro | Valor |
|-----------|-------|
| Nombre | aks-microproyecto2 |
| Grupo de recursos | rg-microproyecto2 |
| Región | East US 2 |
| Versión Kubernetes | 1.34.8 |
| Nodos | 2 (Standard_D2s_v3) |
| Sistema operativo | Ubuntu 22.04 LTS |
| Plan | Gratis |

**Verificación mediante Cloud Shell:**
```bash
az aks get-credentials --resource-group rg-microproyecto2 --name aks-microproyecto2
kubectl get nodes
kubectl get nodes -o wide
```

**Verificación mediante Azure CLI (Windows):**
```bash
az login --use-device-code
az aks get-credentials --resource-group rg-microproyecto2 --name aks-microproyecto2
kubectl get nodes
```

Evidencia: `evidencia/01_cluster_aks_creado.png` al `evidencia/08_kubectl_get_nodes_cli_windows.png`

---

### 2. Aplicación de clasificación de imágenes

Se desplegó un Jupyter Notebook con TensorFlow que implementa clasificación de imágenes usando el modelo **MobileNetV2** preentrenado en ImageNet.

**Despliegue:**
```bash
kubectl create deployment image-classifier --image=tensorflow/tensorflow:latest-jupyter
kubectl expose deployment image-classifier --type=LoadBalancer --port=8888
```

**Acceso:** `http://<EXTERNAL-IP>:8888`

**Modelo utilizado:** MobileNetV2 (ImageNet)  
**Resultado de prueba:** clasificación de imagen sintética con predicciones de clase

Evidencia: `evidencia/09_app_clasificacion_jupyter.png`, `evidencia/10_clasificacion_imagenes_resultado_1.png`

---

### 3. Aplicación de interés — Grafana

Se desplegó **Grafana v13.0.2** como aplicación de monitoreo y visualización de métricas.

**Despliegue:**
```bash
kubectl create deployment grafana --image=grafana/grafana:latest
kubectl expose deployment grafana --type=LoadBalancer --port=3000
```

**Acceso:** `http://<EXTERNAL-IP>:3000`  
**Credenciales iniciales:** admin / admin

Evidencia: `evidencia/11_grafana_servicio.png`, `evidencia/12_grafana_dashboard.png`

---

### 4. Supervisión y monitoreo

Se utilizó el servicio de **Métricas de Azure Monitor** integrado en AKS para monitorear el cluster.

**Métricas monitoreadas:**
- CPU Usage Percentage (Máx): 47%
- API Server Memory Usage Percentage: 15%
- Nodos: 2 listos
- Pods en ejecución: 25

Evidencia: `evidencia/13_aks_supervision_parte1.png` al `evidencia/15_aks_metricas_cpu_memoria.png`

---

## Comandos de referencia

```bash
# Ver todos los deployments
kubectl get deployments

# Ver todos los servicios y sus IPs
kubectl get services

# Ver todos los pods
kubectl get pods

# Ver logs de un pod
kubectl logs <nombre-del-pod>

# Eliminar cluster (para ahorrar créditos)
az group delete --name rg-microproyecto2 --yes --no-wait
```

---

## Nota importante

> Los recursos de AKS consumen créditos de Azure rápidamente. Se recomienda eliminar el clúster después de cada sesión de trabajo y recrearlo antes de la sustentación.
