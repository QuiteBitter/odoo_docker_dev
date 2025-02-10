# Odoo Development Environment with Docker

This repository provides a Docker Compose-based environment to run and develop against two versions of Odoo – **Odoo 17
** and **Odoo 18** – on your local machine. It also includes a separate PostgreSQL container for the database and an
Nginx reverse proxy to route requests to the correct Odoo instance. The setup uses persistent volumes to ensure that
your data and custom addons persist across container restarts.

## Table of Contents

- [Features](#features)
- [Directory Structure](#directory-structure)
- [Prerequisites](#prerequisites)
- [Setup and Installation](#setup-and-installation)
- [Configuration Files](#configuration-files)
- [Running the Environment](#running-the-environment)
- [Accessing Odoo Instances](#accessing-odoo-instances)
- [Developing Custom Addons](#developing-custom-addons)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

## Features

- **Multiple Odoo Versions:** Run both Odoo 17 and Odoo 18 concurrently in separate containers.
- **Dedicated PostgreSQL Container:** A separate Postgres container hosts your databases, ensuring better isolation and
  ease of management.
- **Nginx Reverse Proxy:** Use a reverse proxy to route HTTP requests to the correct Odoo instance based on hostname.
- **Persistent Storage:** Data (database files, filestore) and custom addons are stored persistently via Docker volumes
  and bind mounts.
- **Developer Friendly:** Easily mount custom addons directories for rapid development and testing.
- **Easy Configuration:** All settings (database connection, addons paths, proxy mode) are set through configuration
  files.

## Directory Structure

A recommended structure for this project is:

```
odoo_docker_dev/
├── docker-compose.yml             # Docker Compose file to define and run the containers.
├── README.md                      # This file.
├── config/
│   ├── odoo17.conf                # Odoo 17 configuration file.
│   ├── odoo18.conf                # Odoo 18 configuration file.
│   └── nginx.conf                 # Nginx reverse proxy configuration.
├── addons_odoo17/                 # Directory for custom Odoo 17 addons.
└── addons_odoo18/                 # Directory for custom Odoo 18 addons.
```

- **docker-compose.yml:** Orchestrates the Odoo 17, Odoo 18, PostgreSQL, and Nginx containers.
- **config/odoo17.conf & config/odoo18.conf:** Contains settings for each Odoo instance (database connection, addons
  path, admin password, etc.).
- **config/nginx.conf:** Configures Nginx to route requests based on hostnames (`odoo17.local` and `odoo18.local`).
- **addons_odoo17/ & addons_odoo18/:** Bind-mounted directories that allow you to develop and test custom Odoo modules.

## Prerequisites

Before starting, ensure you have the following installed:

- **Docker:** [Install Docker](https://docs.docker.com/get-docker/)
- **Docker Compose:** [Install Docker Compose](https://docs.docker.com/compose/install/)

*Optional:* Update your system’s hosts file so your browser can resolve the local hostnames used by the reverse proxy.

### Updating Your Hosts File

For local testing, add the following entries to your hosts file:

- **Linux/Mac:** Edit `/etc/hosts` (with sudo):
  ```
  sudo nano /etc/hosts

-127.0.0.1 odoo17.local

-127.0.0.1 odoo18.local

  ```
- **Windows:** Edit `C:\Windows\System32\drivers\etc\hosts` similarly.

## Setup and Installation

1. **Clone or Download the Repository:**

   ```bash
   git clone https://github.com/yourusername/odoo_docker_dev.git
   cd odoo_docker_dev
   ```

2. **Verify Directory Structure:**

   Ensure that you have the following directories and files:
    - `docker-compose.yml`
    - `config/odoo17.conf`
    - `config/odoo18.conf`
    - `config/nginx.conf`
    - `addons_odoo17/`
    - `addons_odoo18/`

3. **Review and Customize Configuration Files:**

    - **Odoo Config Files:**  
      Open `config/odoo17.conf` and `config/odoo18.conf` to check the database settings, addons paths, and admin
      passwords. For example, an `odoo17.conf` might look like:
      ```ini
      [options]
      db_host = db
      db_port = 5432
      db_user = odoo
      db_password = odoo
      addons_path = /usr/lib/python3/dist-packages/odoo/addons,/mnt/extra-addons
      admin_passwd = admin17
      proxy_mode = True
      ```
      Adjust the `admin_passwd` and other settings as needed.

    - **Nginx Configuration:**  
      Open `config/nginx.conf` to see how the reverse proxy is set up. The file routes:
        - `odoo17.local` to the Odoo 17 container.
        - `odoo18.local` to the Odoo 18 container.

      Confirm that these hostnames match the ones you set in your hosts file.

    - **Docker Compose File:**  
      Check `docker-compose.yml` to review container names, volume mounts, environment variables, and network
      configuration. Adjust any parameters if necessary.

## Running the Environment

Once everything is set up, start your Docker containers:

```bash
docker compose up -d
```

This command will:

- Pull the necessary images (if not already available).
- Create and start containers for PostgreSQL, Odoo 17, Odoo 18, and Nginx.
- Mount volumes for persistent data and your custom addons.

To check the status of your containers:

```bash
docker compose ps
```

You should see containers named `odoo-17`, `odoo-18`, `odoo-db`, and `odoo-proxy` in the running state.

## Accessing Odoo Instances

Open your web browser and use the following URLs:

- **Odoo 17:** [http://odoo17.local](http://odoo17.local)
- **Odoo 18:** [http://odoo18.local](http://odoo18.local)

The first time you access an Odoo instance, you will be prompted to create a new database. Use the admin password
defined in the respective configuration file (e.g., `admin17` for Odoo 17) to proceed.

## Developing Custom Addons

This environment is optimized for development:

- Place your custom modules for Odoo 17 in the `addons_odoo17/` directory.
- Place your custom modules for Odoo 18 in the `addons_odoo18/` directory.

Since these directories are bind-mounted into the respective containers (at `/mnt/extra-addons`), any changes you make
on your host are immediately visible to Odoo. To have Odoo recognize new or updated modules:

- Restart the respective container:
  ```bash
  docker compose restart odoo-17  # or odoo-18
  ```
- Or, log in to Odoo and click on **Update Apps List** (ensure developer mode is enabled).

## Troubleshooting

If you encounter issues, try the following:

- **Container Logs:**  
  View logs for a specific container:
  ```bash
  docker compose logs -f odoo-17
  docker compose logs -f odoo-18
  docker compose logs -f odoo-db
  docker compose logs -f odoo-proxy
  ```

- **Permission Issues:**  
  If you get permission errors (especially with bind mounts for addons or config files), adjust permissions on your
  host:
  ```bash
  chmod -R 777 addons_odoo17 addons_odoo18 config
  ```
  *(Note: Using `777` is acceptable for a local development environment but not recommended for production.)*

- **Database Connection:**  
  Ensure the settings in `odoo17.conf`/`odoo18.conf` (and the environment variables in `docker-compose.yml`) match the
  credentials provided for the PostgreSQL container.

- **Hostnames:**  
  Verify that your hosts file is correctly mapping `odoo17.local` and `odoo18.local` to `127.0.0.1`.

- **Container Restart:**  
  Sometimes a simple restart of the Docker containers can resolve transient issues:
  ```bash
  docker compose restart
  ```

- **Volume Persistence:**  
  Data is persisted using named volumes (`db-data`, `odoo17-data`, `odoo18-data`). If you wish to start over, you can
  remove the containers and volumes (warning: this will erase your data):
  ```bash
  docker compose down -v
  ```

## Additional Resources

- [Official Odoo Docker Documentation](https://hub.docker.com/_/odoo)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Odoo Documentation](https://www.odoo.com/documentation)
- [Nginx Reverse Proxy Guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

---
