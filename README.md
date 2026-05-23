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
│   └── entrega/                # Informe, evidencias y video
│       ├── Informe_Practica1_Vagrant_MiltonJaramillo_v2.pdf
│       └── Evidencia/          # Capturas de pantalla del proceso
├── Modulo2/
│   ├── documentos/             # Guías del módulo 2
│   └── Practica1_LXD/
│       └── entrega/
│           └── evidencia/      # 22 capturas del proceso LXD
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

## Notas importantes
- Se requiere desactivar **Hyper-V** en Windows 11 para que VirtualBox funcione correctamente:
  ```
  bcdedit /set hypervisorlaunchtype off
  ```
  Luego reiniciar el equipo.
- Los instaladores, videos y archivos grandes no están versionados en git. Se encuentran en las carpetas locales correspondientes.
