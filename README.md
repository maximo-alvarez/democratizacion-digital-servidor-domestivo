# Democratización de la Infraestructura Digital: Un Servidor Doméstico Resiliente

![GitHub Repo stars](https://img.shields.io/github/stars/maximo-alvarez/democratizacion-digital-servidor-domestivo?style=social)
![GitHub forks](https://img.shields.io/github/forks/maximo-alvarez/democratizacion-digital-servidor-domestivo?style=social)
![License](https://img.shields.io/badge/License-MIT-blue.svg)

Este repositorio contiene los archivos de configuración (`docker-compose.yml` y `.env.example`) para implementar un servidor doméstico multifuncional, de bajo costo y resiliente, tal como se describe en el capítulo de libro "Democratización de la infraestructura digital: modelo resiliente de servidor doméstico para la soberanía de datos".

---

## 📖 Sobre el Proyecto

[cite_start]En la última década, la transformación digital ha consolidado un paradigma donde la infraestructura tecnológica está dominada por un número reducido de corporaciones (hiperescaladores)[cite: 1767]. [cite_start]Este modelo de centralización genera una dependencia que afecta la privacidad, la seguridad y la autonomía de los usuarios[cite: 1768].

[cite_start]Frente a este escenario, este proyecto demuestra la **viabilidad técnica, económica y la resiliencia** de un modelo de servidor doméstico autoalojado, utilizando hardware de consumo (Mini PC) y software de código abierto[cite: 1774, 1771]. [cite_start]El objetivo es proporcionar una guía práctica y replicable para que cualquier usuario con interés técnico pueda construir su propia nube personal y recuperar el control sobre sus datos, un paso fundamental hacia la **soberanía digital**[cite: 1798].

[cite_start]Este repositorio ofrece las herramientas necesarias para replicar la infraestructura descrita, permitiendo a los individuos un control efectivo sobre sus servicios y datos[cite: 1754].

---

## 🚀 Servicios Desplegados

Este proyecto utiliza **Docker** para orquestar un conjunto de servicios de código abierto que reemplazan a sus contrapartes comerciales:

* ☁️ **Nextcloud**: Nube personal para archivos, calendarios, contactos y más (Alternativa a Google Drive/Dropbox).
* 🎬 **Jellyfin**: Servidor de medios para tus películas, series y música (Alternativa a Plex/Emby).
* 💬 **Mattermost**: Plataforma de comunicación para equipos (Alternativa a Slack/Microsoft Teams).
* 🎨 **Excalidraw**: Pizarra virtual colaborativa para diagramas y bocetos.
* 🤖 **n8n**: Plataforma de automatización de flujos de trabajo (Alternativa a Zapier/Make.io).
* 🐘 **PostgreSQL**: Robusto sistema de gestión de bases de datos para los servicios.
* 🐳 **Portainer CE**: Interfaz gráfica para gestionar el entorno de Docker.

---

## 🏗️ Arquitectura del Sistema

El modelo se basa en una arquitectura modular y resiliente:

1.  [cite_start]**Hardware**: Un **Mini PC** de bajo consumo (GMKtec N100) conectado a un **SAI/UPS** para garantizar la continuidad energética[cite: 1751, 1793].
2.  [cite_start]**Sistema Operativo**: **Ubuntu Server 24.04 LTS "Noble Numbat"** como base estable y segura[cite: 1751].
3.  **Acceso Seguro (Modular)**:
    * [cite_start]**Método Primario (VPN)**: Uso de **Tailscale** para crear una red privada virtual segura y acceder a los servicios desde cualquier lugar sin exponer puertos[cite: 1752, 539].
    * [cite_start]**Método Opcional (Proxy Inverso)**: Un **VPS** de bajo costo actúa como proxy inverso, permitiendo el acceso a los servicios a través de dominios públicos (ej. `nextcloud.tudominio.com`) de forma segura[cite: 1752, 544, 546, 548, 550, 552, 554, 556].

---

## 🛠️ Guía de Implementación

Sigue estos pasos para replicar el servidor en tu propio hardware.

### Paso 1: Preparación del Servidor Base

1.  **Instala Ubuntu Server 24.04 LTS**.
2.  **Actualiza el sistema** y elimina `snapd` (opcional):
    ```bash
    sudo apt update && sudo apt upgrade -y && sudo apt autoremove --purge snapd -y
    ```
3.  **Instala herramientas básicas**:
    ```bash
    sudo apt install rsync nano htop net-tools iputils-ping locales screen -y
    ```
4.  **Configura la zona horaria y el idioma** (ejemplo para Ecuador):
    ```bash
    sudo locale-gen es_EC.UTF-8
    sudo ln -fs /usr/share/zoneinfo/America/Guayaquil /etc/localtime
    sudo dpkg-reconfigure -f noninteractive tzdata
    ```

### Paso 2: Instalación de Docker y Docker Compose

1.  **Añade el repositorio oficial de Docker**:
    ```bash
    curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) noble stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
2.  **Instala Docker Engine, CLI y Compose**:
    ```bash
    sudo apt-get update
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
    ```
3.  **Añade tu usuario al grupo `docker`** para ejecutar comandos sin `sudo`:
    ```bash
    sudo usermod -aG docker ${USER}
    ```
    > ⚠️ **Nota:** Deberás cerrar sesión y volver a iniciarla para que este cambio surta efecto.

### Paso 3: Configuración del Almacenamiento (Opcional pero Recomendado)

Si tienes un segundo disco para los datos:

1.  **Identifica el disco** (ej. `/dev/sda`):
    ```bash
    lsblk
    ```
2.  **Crea una nueva partición** (ej. `/dev/sda3`) usando `sudo fdisk /dev/sda`.
3.  **Formatea la nueva partición**:
    ```bash
    sudo mkfs.ext4 /dev/sda3
    ```
4.  **Monta la partición y configúrala para que se monte al arranque**:
    ```bash
    # Crea el punto de montaje
    sudo mkdir -p /mnt/data
    
    # Monta el disco
    sudo mount /dev/sda3 /mnt/data
    
    # Obtén el UUID para el montaje automático
    sudo blkid /dev/sda3
    
    # Añade la línea a /etc/fstab (reemplaza el UUID con el tuyo)
    # UUID=6cd6474e-9f04-40d9-b393-70dcfb17a768 /mnt/data ext4 defaults 0 0
    sudo nano /etc/fstab
    
    # Verifica que el montaje funciona
    sudo mount -a
    ```

### Paso 4: Despliegue de Servicios

1.  **Clona este repositorio**:
    ```bash
    git clone [https://github.com/maximo-alvarez/democratizacion-digital-servidor-domestivo.git](https://github.com/maximo-alvarez/democratizacion-digital-servidor-domestivo.git)
    cd democratizacion-digital-servidor-domestivo
    ```
2.  **Configura tus variables de entorno**:
    * Copia la plantilla: `cp .env.example .env`
    * Edita el archivo `.env` con tus propios valores (rutas, dominio, contraseñas): `nano .env`

3.  **Crea los directorios de datos** (si usas una ruta personalizada en `.env`):
    * La variable `DATA_BASE_PATH` en tu `.env` debe apuntar a un directorio existente. Asegúrate de crearlo si es necesario.

4.  **Asigna los permisos correctos** a los directorios de volumen:
    ```bash
    # Lee la ruta base desde tu archivo .env
    export $(grep -v '^#' .env | xargs)
    
    # Asigna permisos
    sudo chown -R 1000:1000 $DATA_BASE_PATH/jellyfin/cache
    sudo chown -R 1000:1000 $DATA_BASE_PATH/jellyfin/config
    sudo chown -R 999:999 $DATA_BASE_PATH/postgres/data
    sudo chown -R 2000:2000 $DATA_BASE_PATH/mattermost/data
    ```

5.  **Inicia todos los servicios**:
    ```bash
    docker compose up -d
    ```

### Paso 5: Configuración de la Red Privada (Tailscale)

1.  **Instala Tailscale** en el servidor:
    ```bash
    curl -fsSL [https://tailscale.com/install.sh](https://tailscale.com/install.sh) | sh
    ```
2.  **Inicia Tailscale y autentícate**:
    ```bash
    sudo tailscale up
    ```
    > Sigue el enlace que te mostrará en la terminal para añadir el servidor a tu cuenta de Tailscale. Instala Tailscale en tus otros dispositivos (laptop, móvil) para acceder a tu servidor de forma segura.

---

## 🚀 Uso y Mantenimiento

* **Ver el estado de los contenedores**: `docker ps -a`
* **Ver los logs de un servicio**: `docker compose logs -f <nombre_del_servicio>` (ej. `nextcloud`)
* **Detener los servicios**: `docker compose down`
* **Actualizar los contenedores**:
    ```bash
    # Descarga las últimas imágenes
    docker compose pull
    
    # Reinicia los servicios con las nuevas imágenes
    docker compose up -d
    ```

---

## 🎓 Cómo Citar este Proyecto

Si utilizas este proyecto como referencia en tu trabajo, por favor cita la publicación académica asociada:

> Alvarez, M. (2025). Democratización de la infraestructura digital: modelo resiliente de servidor doméstico para la soberanía de datos. En *Ciencia, educación y sociedad en red: Perspectivas para la innovación*.

---

## 📄 Licencia

Este proyecto se distribuye bajo la Licencia MIT. Consulta el archivo `LICENSE` para más detalles.
