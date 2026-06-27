# 🚢 Docker Containers and Networking


## 🖼️ Quick Visual Summary

![Quick Summary: Docker Containers and Networking](../assets/topic-summaries/docker-containers-networking.svg)

> **⚡ 80/20 Summary:** Containers are running images • ports expose apps • networks connect services • logs/debug commands find issues

## 1. 🎯 Overview
While a Docker Image is the static blueprint, a **Docker Container** is the living, breathing application. Once running, these isolated containers must communicate with the outside world (like your web browser) or with each other (like a web app talking to a database) using **Docker Networking**.

## 2. 💡 Why This Matters
- **Isolation Security:** By default, containers are locked in a sandbox. They cannot touch the host machine's files or interact with other containers unless explicitly permitted.
- **Port Mapping:** You can run 5 different Node.js microservices, all internally listening on Port 8080, entirely without conflict on your host machine.
- **Microservice Communication:** Complex apps require containers to find and talk to each other without knowing hardcoded, changing IP addresses.

## 3. 🧠 Core Concepts
- **Container Lifecycle:** A container can be Created, Running, Paused, Stopped, or Deleted.
- **Port Binding (`-p`):** The act of punching a hole through the container's sandbox, mapping a port on the host machine to a port inside the container.
- **Bridge Network (Default):** The default network Docker creates. Containers on the same bridge network can talk to each other via IP address.
- **User-Defined Bridge:** A custom network you create. Containers here get DNS resolution, meaning they can talk to each other using container names instead of IP addresses!

## 4. 🧭 Architecture / Workflow
1. **Start:** Administrator executes `docker run`. The daemon allocates CPU/RAM limits and assigns an internal IP address to the new container.
2. **Ingress (Incoming Traffic):** Traffic hits the Host Server on Port `80`. Docker's internal proxy routes that traffic instantly into the specific container on Port `3000`.
3. **Egress (Outgoing Traffic):** Containers can securely reach out to the open internet to download updates or API data via the host's network interface.

## 5. 🛠️ Commands & Practical Usage

Run a container in the background (`-d` for detached) and map ports:
```bash
docker run -d -p 8080:80 nginx
```
> *Maps Host Port 8080 to Container Port 80. Now `http://localhost:8080` hits Nginx.*

List all actively running containers:
```bash
docker ps
```
> *Use `docker ps -a` to see stopped/crashed containers too.*

Step entirely *inside* a running container to debug it:
```bash
docker exec -it <container-id> /bin/bash
```
> *`-it` stands for Interactive Terminal.*

Stop, then completely obliterate a container permanently:
```bash
docker stop <container-id>
docker rm <container-id>
# Or forcibly destroy it in one command:
docker rm -f <container-id>
```

View the live console logs of a running container:
```bash
docker logs -f <container-id>
```

## 6. ⚙️ Configuration / Code Examples
Creating a custom network allowing two disconnected containers to communicate exclusively via DNS names.

```bash
# 1. Create a secure private network space
docker network create my-internal-net

# 2. Start a hidden database container on that network
docker run -d --name my-redis --network my-internal-net redis:alpine

# 3. Start a web app container on the SAME network and expose it to the world
docker run -d --name web-frontend \
  --network my-internal-net \
  -p 80:80 \
  -e REDIS_HOST="my-redis" \
  my-web-image
```
> *Notice we didn't map a port for `my-redis`. It is completely hidden from the internet, but `web-frontend` can reach it simply by using the hostname `my-redis`.*

## 7. 🧪 Hands-on Step-by-Step

**Step 1: Run Nginx without port mapping**
```bash
docker run -d --name invisible-nginx nginx
```

**Step 2: Try to access it**
Open your browser to `http://localhost`. It will **fail**. The application is trapped in the sandbox. Stop and remove it: `docker rm -f invisible-nginx`.

**Step 3: Run Nginx WITH port mapping**
```bash
docker run -d --name visible-nginx -p 9000:80 nginx
```

**Step 4: Witness success**
Open `http://localhost:9000` in your browser. You will see the "Welcome to nginx!" screen.

**Step 5: Jump inside to make a live change**
```bash
docker exec -it visible-nginx /bin/sh
# Now inside the container!
echo "HACKED BY DOIN DEVOPS!" > /usr/share/nginx/html/index.html
exit
```

**Step 6: Refresh your browser tab**
You will see your new custom HTML text immediately reflected. 

## 8. 🚨 Common Errors & Troubleshooting

- **Error: `Bind for 0.0.0.0:8080 failed: port is already allocated`**
  - **Issue:** You tried to run `docker run -p 8080:80`, but an entirely different application (or another container) is already using port 8080 on your host machine.
  - **Fix:** Change the host mapping to an empty port: `docker run -p 8081:80`.
- **Error: `exec failed: unable to start container process: exec: "bash": executable file not found in $PATH`**
  - **Issue:** You ran `docker exec -it <id> bash`, but the container is built on Alpine Linux, which doesn't have bash installed!
  - **Fix:** Fall back to the default lightweight shell by using `sh` instead: `docker exec -it <id> sh`.
- **Error: Web app cannot reach the Database container.**
  - **Issue:** You booted them both using basic `docker run`, meaning they are on the default bridge network where automatic DNS resolution is disabled.
  - **Fix:** Put them on a custom network: `docker network create local-net` and attach them using `--network local-net`.

## 9. ✅ Best Practices

1. **Avoid the Default Bridge Network:** Always create user-defined networks for multi-container applications so they can communicate seamlessly using container names.
2. **Never store data inside containers:** Containers are ephemeral (temporary). If a container crashes, you lose everything inside it perfectly. Real data must live in Docker Volumes.
3. **Use environment variables:** Pass dynamic configuration (like Database passwords) cleanly at runtime using the `-e` flag (`docker run -e DB_PASS=secret123`), never hardcode them in the image.

## 10. 🎤 Interview Questions & Answers

**Q1: If two containers map a port to the same host port (e.g., `-p 80:8080`), what happens?**
**A1:** The first container will boot successfully. The second container will immediately fail with a "Port already allocated" error because the host's external network card can only route one port to one destination at a time.

**Q2: What is the `-d` flag used for in `docker run`?**
**A2:** It runs the container in *Detached* mode. Instead of seizing control of your terminal and spitting out server logs immediately, it pushes the container to the background and gives your terminal prompt back.

**Q3: How do containers communicate if they are running on the "default" bridge network vs a "user-defined" network?**
**A3:** On the default bridge network, they can only communicate via fragile, shifting IP addresses. On a user-defined network, Docker turns on an embedded DNS server, allowing them to communicate safely via reliable container names.

**Q4: If a container is stopped using `docker stop`, is the data inside it lost?**
**A4:** No. The container process stops, but its writable filesystem layer is preserved on disk until you explicitly execute `docker rm`. 

**Q5: What command would you use to forcibly execute a bash shell as the 'root' user inside a running container that normally runs as 'appuser'?**
**A5:** You use the user flag override: `docker exec -u root -it <container-id> bash`.

## 11. ⚡ Quick Revision Summary
- **`docker ps`:** The lifeline to see what is running.
- **`-p HOST:CONTAINER`:** Unlocks the sandbox and lets external traffic in.
- **`docker exec -it`:** The SSH equivalent for stepping inside a live container.
- **`docker network`:** The glue that allows containers to talk securely via internal DNS.

## 12. 🔗 Official Documentation Links
- [Docker Run Reference](https://docs.docker.com/engine/reference/run/)
- [Docker Networking Overview](https://docs.docker.com/network/)
