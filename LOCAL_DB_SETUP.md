# How to install MariaDB and phpMyAdmin on Fedora using Podman

Here is a detailed step-by-step guide to install and run MariaDB and phpMyAdmin on Fedora using Podman with separate containers:

## 1. Install Podman

- install podman via `sudo dnf install podman`
- verify podman installation via `podman -v`

## 2. Create a Podman network

```bash
podman network create mynetwork
```

## 3. Run MariaDB container

The next command will pull the latest MariaDB image and run it with environment variables for root password and user credentials, it will also expose port 3306 to the host for possible direct access:
```bash
podman run -d --name mariadb \
  --network mynetwork \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=YourRootPassword \
  -e MYSQL_DATABASE=exampledb \
  -e MYSQL_USER=exampleuser \
  -e MYSQL_PASSWORD=examplepass \
  --restart=always \
  mariadb:latest
```

## 4. Verify MariaDB container

- check if the mariadb container is running via `podman ps`
- Logs to confirm MariaDB started correctly: `podman logs mariadb`
- Connect to MariaDB inside the container to verify database:
```bash
podman exec -it mariadb mysql -u root -p
# Enter Your Root Password when prompted
SHOW DATABASES;
```

## 5. Run phpMyAdmin container

```bash
podman run -d --name phpmyadmin \
  --network mynetwork \
  -p 8080:80 \
  -e PMA_HOST=mariadb \
  --restart=always \
  phpmyadmin/phpmyadmin:latest
```

**Explanation**:
- `-p 8080:80` exposes phpMyAdmin web UI on port 8080.
- `PMA_HOST=mariadb` is how we connect phpmyadmin and mariadb containers within Podman’s internal network 

## 6. Verify phpMyAdmin container

- check if the phpmyadmin container is running via `podman ps`
- check logs for errors: `podman logs phpmyadmin`

## 7. Access phpMyAdmin web UI

- Open a browser and navigate to http://localhost:8080

## 8. Persist MariaDB data on host (optional)

To persist data beyond container lifecycle, we need to:
- create a volume, 
- delete the existing mariadb container,
- and create a new container with the volume mounted on it

```bash
podman volume create mariadb_data

podman rm -f mariadb

podman run -d --name mariadb \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=YourRootPassword \
  -e MYSQL_DATABASE=exampledb \
  -e MYSQL_USER=exampleuser \
  -e MYSQL_PASSWORD=examplepass \
  -v mariadb_data:/var/lib/mysql \
  --restart=always \
  mariadb:latest
```
