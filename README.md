# Apache Guacamole deployment procedure

## Container stack deployment
1. Clone this repository.

2. Edit the `docker-compose.yml` file. Replace both database user and root passwords with secure passwords.

3. Run docker compose.
```bash
docker compose up -d
```
or create a Stack in Portainer.

Once the containers are up, its time to set up the database.

## MySQL database setup
1. Get into `guac-sql` container. From the host server, run:
```bash
docker exec -it guac-sql /bin/bash
```
or access it via portainer if you prefer. 

Once inside the container, run:
```bash
mysql -u root -p
```
You will be prompted for the root password.

2. Upon loging in, create the database.
```sql
CREATE DATABASE guacamole_db;
```

3. Then, create a new user.
```sql
CREATE USER 'guacamole_user'@'%' IDENTIFIED BY 'pass123';
GRANT SELECT,INSERT,UPDATE,DELETE ON guacamole_db.* TO 'guacamole_user'@'%';
FLUSH PRIVILEGES;
```
4. Confirm the user was created (Optional)
```sql
SELECT USER FROM mysql.user; 
```
Leave this terminal open. You will need to come back.

5. Initialize the MySQL database
- Log into the host server and run the following command:
```bash
docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
```
- Copy the generated script to the `guacamole-sql` container
```bash
sudo docker cp ./initdb.sql guac-sql:/initdb.sql
```
- Go back to the container terminal and run the following:
```sql
use guacamole_db;
```
```sql
source initdb.sql;
```

At this point you should have successfully configured the database. You can now open guacamole's Web Interface at `http://<server-ip>:8080/guacamole`
