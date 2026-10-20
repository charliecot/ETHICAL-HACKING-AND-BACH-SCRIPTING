# Linux Deployment Cheat Sheet: Running a Dockerized Application

## 1. Understand the deployment process

Assume you have an application with a `Dockerfile` and possibly a `compose.yaml` file.

The workflow is:

1. Prepare the Linux machine.
2. Install or verify Docker.
3. Obtain the application source code.
4. Configure environment variables and secrets.
5. Build the Docker image.
6. Run the container or containers.
7. Test the application.
8. Configure persistence, logs, and restart policies.
9. Secure the deployment.
10. Update the application when you release new code.

Example architecture:

```text
Browser
   |
   | HTTP/HTTPS
   v
Linux server
   |
   v
Docker
   |
   +---- Django application container
   |
   +---- PostgreSQL container
   |
   +---- Redis container
```

The exact services depend on your application.

---

## 2. Prepare Linux

Check your Linux distribution:

```bash
cat /etc/os-release
```

Check your current user:

```bash
whoami
```

Check your current directory:

```bash
pwd
```

Check disk space:

```bash
df -h
```

Check available memory:

```bash
free -h
```

Check your IP addresses:

```bash
ip addr
```

Check whether Docker is installed:

```bash
docker --version
```

Check Docker Compose:

```bash
docker compose version
```

Check Git:

```bash
git --version
```

If Docker is missing, install Docker Engine using the official instructions for your Linux distribution:

https://docs.docker.com/engine/install/

For Ubuntu, follow the Ubuntu-specific installation instructions.

If you use WSL with Docker Desktop, enable Docker Desktop's WSL integration for your Ubuntu distribution instead of automatically installing a second Docker Engine.

---

## 3. Obtain your application

### Option A: Clone from Git

Create an applications directory:

```bash
mkdir -p ~/apps
cd ~/apps
```

Clone your repository:

```bash
git clone YOUR_REPOSITORY_URL
```

Enter the project directory:

```bash
cd YOUR_PROJECT_DIRECTORY
```

Inspect the files:

```bash
ls -la
```

Find Docker configuration files:

```bash
find . -maxdepth 2 \
  \( -name Dockerfile -o -name compose.yaml -o -name docker-compose.yml \)
```

Replace the placeholder repository URL and directory name with your real values.

### Option B: Application already exists in Linux

Navigate to your project:

```bash
cd /path/to/your/project
```

Check that the Docker configuration is present:

```bash
ls -la
```

---

## 4. Understand the important Docker files

### Dockerfile

The `Dockerfile` defines how Docker builds an image.

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

This is an illustrative Django example, not a production-ready configuration. For production, use a suitable application server, such as Gunicorn with a compatible WSGI application, or an appropriate ASGI server.

### Compose file

The Compose file defines services, networks, volumes, and configuration for multiple containers.

Example:

```yaml
services:
  web:
    build: .
    ports:
      - "8000:8000"
    restart: unless-stopped
```

This example assumes the application listens on port 8000 inside the container.

---

## 5. Configure environment variables

Create an environment file if your project expects one:

```bash
nano .env
```

Example for Django:

```env
DEBUG=False
SECRET_KEY=replace-with-a-secure-random-secret
ALLOWED_HOSTS=localhost,127.0.0.1
```

Use the actual environment variable names required by your application.

Protect the file:

```bash
chmod 600 .env
```

Ensure that `.env` is included in `.gitignore`:

```gitignore
.env
```

Do not commit production secrets to a public repository.

For a public deployment, configure the real domain in `ALLOWED_HOSTS`, configure CORS where necessary, and store credentials securely.

---

## 6. Build the Docker image

If you have a Dockerfile:

```bash
docker build -t myapp:latest .
```

Explanation:

- `docker build`: builds an image.
- `-t`: assigns a name and tag.
- `myapp:latest`: image name and tag.
- `.`: uses the current directory as the build context.

List the images:

```bash
docker images
```

Inspect image details:

```bash
docker image inspect myapp:latest
```

---

## 7. Run a single container

Example:

```bash
docker run -d \
  --name myapp \
  --restart unless-stopped \
  --env-file .env \
  -p 8000:8000 \
  myapp:latest
```

Explanation:

- `-d`: runs in the background.
- `--name`: gives the container a name.
- `--restart unless-stopped`: restarts it after failures or Docker daemon restarts, unless you explicitly stop it.
- `--env-file`: loads environment variables.
- `-p 8000:8000`: maps host port 8000 to container port 8000.
- `myapp:latest`: image to run.

The application must actually listen on port 8000 inside the container.

If you use Compose instead, you generally do not need this separate `docker run` command.

---

## 8. Run a multi-container application with Compose

If your project contains `compose.yaml` or `docker-compose.yml`:

```bash
# Validate the resolved configuration
docker compose config

# Build images and start services
docker compose up -d --build

# List services
docker compose ps

# View logs
docker compose logs

# Follow logs continuously
docker compose logs -f
```

Compose is especially useful when your application depends on PostgreSQL, Redis, a worker process, or other services.

If the application needs database migrations, run them using the command appropriate for your Compose service. For a Django service named `web`, that might be:

```bash
docker compose exec web python manage.py migrate
```

Only run this if your application uses Django and the service is named `web`.

For a new production database, make sure you have a backup and understand the migration before applying it.

---

## 9. Verify that the application is running

List running containers:

```bash
docker ps
```

List all containers, including stopped ones:

```bash
docker ps -a
```

View logs:

```bash
docker logs myapp
```

Follow logs:

```bash
docker logs -f myapp
```

Inspect port mappings:

```bash
docker port myapp
```

Inspect container configuration:

```bash
docker inspect myapp
```

Monitor resource usage:

```bash
docker stats
```

For Compose projects:

```bash
docker compose ps
docker compose logs -f
```

If a container stops unexpectedly, check its logs before restarting it.

Common causes include missing environment variables, database connection failures, incorrect commands, and port conflicts.

---

## 10. Test the HTTP endpoint

Test the application locally:

```bash
curl -I http://localhost:8000/
```

For a Django REST API, test an actual API endpoint:

```bash
curl -i http://localhost:8000/api/products/
```

Replace the URL with an endpoint provided by your application.

Useful HTTP status codes:

- `200`: request succeeded.
- `301` or `302`: redirect.
- `400`: bad request.
- `401`: authentication required or failed.
- `403`: forbidden.
- `404`: resource or URL not found.
- `500`: server-side error.

A successful HTTP response does not necessarily mean every application dependency is healthy. Check logs and database connectivity as well.

---

## 11. Manage the application

### Start a stopped container

```bash
docker start myapp
```

### Stop a container

```bash
docker stop myapp
```

### Restart a container

```bash
docker restart myapp
```

### Remove a container

```bash
docker rm myapp
```

The container must be stopped before removal unless you explicitly force removal.

### Compose lifecycle commands

```bash
docker compose start
docker compose stop
docker compose restart
docker compose down
```

`docker compose down` removes the project's containers and networks. Named volumes normally remain unless you explicitly request their removal.

Avoid `docker compose down -v` when you need to preserve database data.

---

## 12. Update your application

For an application stored in Git:

```bash
# Get the latest code
git pull

# Rebuild images and recreate changed services
docker compose up -d --build

# Check service status
docker compose ps

# Check logs
docker compose logs -f
```

Before updating a production application:

1. Review the code changes.
2. Back up important data.
3. Review database migrations.
4. Build and test the new image.
5. Deploy the new version.
6. Check application health and logs.
7. Have a rollback plan.

For production, prefer explicit image tags or release versions rather than relying entirely on `latest`.

---

## 13. Persistent data and volumes

A container's writable layer is not a reliable place for important data.

List Docker volumes:

```bash
docker volume ls
```

Inspect a volume:

```bash
docker volume inspect VOLUME_NAME
```

In Compose, a database might use a named volume:

```yaml
services:
  db:
    image: postgres:16
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Use a PostgreSQL image version compatible with your application. Changing major database versions requires a proper upgrade procedure.

Remember:

- Containers can be replaced.
- Named volumes can persist beyond container removal.
- Volumes are not backups.
- Back up databases and uploaded media separately.
- Test restoring your backups.

---

## 14. Inspect Linux networking

List listening TCP ports:

```bash
sudo ss -ltnp
```

Check a specific port:

```bash
sudo ss -ltnp | grep ':8000'
```

Test a local connection:

```bash
curl -v http://localhost:8000/
```

Check the Linux firewall if applicable:

```bash
sudo ufw status
```

On a public server, only expose ports that are required.

For example, you might expose HTTPS on port 443 while keeping PostgreSQL and internal application ports inaccessible from the public internet.

Docker-published ports can interact with firewall rules differently from ordinary host processes, so follow Docker's official networking and firewall guidance.

---

## 15. Use Apache or Nginx for a public website

For a production web application, a common architecture is:

```text
Internet
   |
   v
HTTPS :443
   |
   v
Apache or Nginx
   |
   v
Application container
   |
   +---- Database
   |
   +---- Redis
```

The reverse proxy can handle incoming HTTPS requests and forward traffic to the application.

For Django, the application container should run a suitable production application server. Do not expose Django's development server as your public production server.

The proxy, application, and database should have deliberate network boundaries. Do not publish internal database ports unnecessarily.

For HTTPS, configure a valid TLS certificate and renew it before expiry.

---

## 16. Basic security checklist

- Set Django `DEBUG=False`.
- Generate a strong `SECRET_KEY`.
- Configure `ALLOWED_HOSTS`.
- Keep passwords and secrets out of Git.
- Do not expose database ports publicly.
- Use HTTPS for public applications.
- Keep the operating system and images updated.
- Use non-root application users where practical.
- Review file permissions.
- Configure logging and health checks.
- Back up database data and uploaded files.
- Test backup restoration.
- Review firewall and Docker networking rules.

---

## 17. Troubleshooting quick reference

| Problem | Commands to investigate |
|---|---|
| Container will not start | `docker ps -a`, `docker logs myapp` |
| Port already in use | `sudo ss -ltnp` |
| HTTP request fails | `curl -v http://localhost:8000/` |
| Wrong port mapping | `docker port myapp` |
| Compose service fails | `docker compose ps`, `docker compose logs` |
| Container exits repeatedly | `docker inspect myapp`, `docker logs myapp` |
| Disk space is low | `df -h`, `docker system df` |
| Environment variable is missing | Review application configuration and container environment |
| Database connection fails | Review database logs, service name, credentials, and network configuration |

Do not delete Docker volumes or run aggressive cleanup commands while diagnosing a production database issue.

---

## 18. Final deployment checklist

Before considering the deployment complete, confirm:

- [ ] Docker is installed and working.
- [ ] Application source code is available.
- [ ] Dockerfile and Compose configuration are valid.
- [ ] Secrets and environment variables are configured.
- [ ] Images build successfully.
- [ ] Containers remain running.
- [ ] Application endpoints respond correctly.
- [ ] Database migrations have been applied safely.
- [ ] Persistent storage is configured.
- [ ] Backups exist and restoration has been tested.
- [ ] Restart policies and health checks are configured.
- [ ] Firewall, HTTPS, and public networking are configured where needed.
- [ ] Logs and an update or rollback procedure are available.

**Remember:** A Dockerized application is not automatically a production-ready deployment. Docker packages and runs the application; production readiness also requires configuration, persistence, security, monitoring, and recovery procedures.
