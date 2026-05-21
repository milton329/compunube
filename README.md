# Computación en la Nube — UAO Virtual

Repositorio de prácticas de la especialización en Computación en la Nube.

**Estudiante:** Milton Jaramillo
**Docente:** Prof. Oscar Mondragón
**Institución:** UAO Virtual
**Año:** 2026

---

## Estructura del repositorio

```
compunube/
├── Instaladores/               # Instaladores usados en las prácticas (no versionados)
├── Modulo1/
│   ├── guias/                  # Guías del módulo 1
│   │   ├── M1A2_Guia1_ComputacionNube.pdf
│   │   └── M1A2_Guia2_ComputacionNube.pdf
│   └── entrega/                # Informe y evidencias de la práctica 1
│       ├── Informe_Practica1_Vagrant_MiltonJaramillo_v2.pdf
│       └── Evidencia/          # Capturas de pantalla del proceso
├── Modulo2/
│   └── Practica2_LXD/          # Contenedores con LXD (próximamente)
├── Modulo3/                    # Próximamente
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
El box modificado está disponible públicamente en:
**https://app.vagrantup.com/milton329/boxes/ubuntu-20.04-practica1**

### Repositorio GitHub
**https://github.com/milton329/compunube**

---

## Notas importantes
- Se requiere desactivar **Hyper-V** en Windows 11 para que VirtualBox funcione correctamente:
  ```
  bcdedit /set hypervisorlaunchtype off
  ```
  Luego reiniciar el equipo.
- Los instaladores no están versionados en git por su tamaño (>100MB). Se encuentran en la carpeta `Instaladores/` de forma local.
