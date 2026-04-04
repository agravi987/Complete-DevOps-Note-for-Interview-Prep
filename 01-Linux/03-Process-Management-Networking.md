# Linux Process Management and Networking

## 1. Overview
In a DevOps environment, applications (databases, web servers) run as **"Processes"** within Linux. These processes often need to communicate over the network using **Ports**. Mastering how to find, monitor, kill processes, and analyze network traffic is a non-negotiable debugging skill.

## 2. Why This Matters
- **Outage Resolution:** When a Java app consumes 100% CPU, you need to identify the process and kill it immediately.
- **Port Collisions:** When Docker fails to start a container because "Port 80 is already in use," you must know how to find the rogue process holding the port hostage.
- **Firewall Diagnostics:** If a microservice cannot reach the database, you must test the network connectivity locally via the shell.

## 3. Core Concepts
- **PID (Process ID):** A unique number assigned by the Kernel to every running process.
- **Top / Htop:** Tools that provide a real-time, dynamic view of running system processes and CPU/RAM consumption.
- **Daemons / Services:** Processes configured to start automatically on boot and run in the background (managed by `systemd`).
- **Ports & Bindings:** A process "binds" to a network port (like port 443) to listen for incoming traffic. Only one process can bind to a port at a time.

## 4. Architecture / Workflow
1. **Process Creation:** A command is run (e.g., `nginx`), the kernel gives it a PID and allocates memory.
2. **Network Binding:** The process requests the Kernel to bind to TCP Port 80. 
3. **Listening Hook:** Data arriving at the server's network card on Port 80 is routed directly to the Nginx process.
4. **Termination:** The process is stopped cleanly (SIGTERM) or forcefully (SIGKILL), freeing up the RAM and the Port.

## 5. Commands & Practical Usage

See real-time CPU and Memory usage:
```bash
top
# Pro-tip: Install and use 'htop' for a friendlier, colorful UI.
```

Find the PID of a specific running application:
```bash
ps aux | grep java
```
> *`ps aux` lists ALL processes from all users. `grep` filters the output.*

Gracefully ask a process to terminate:
```bash
kill 14592
```

Forcefully assassinate a frozen process (No mercy):
```bash
kill -9 14592
```

Find out which process is listening on Port 8080:
```bash
netstat -tulpn | grep 8080
# Or using the modern equivalent:
ss -tulpn | grep 8080
```

Check if a remote server's database port is open and reachable:
```bash
telnet database.internal 5432
# Alternative using netcat:
nc -zv database.internal 5432
```

## 6. Configuration / Code Examples
A simple `systemd` configuration file (`/etc/systemd/system/myapp.service`) to manage a background application:

```ini
[Unit]
Description=My Custom Go App
After=network.target

[Service]
ExecStart=/opt/myapp/server
Restart=always
User=appuser
Environment="PORT=8080"

[Install]
WantedBy=multi-user.target
```
> *You apply this via: `systemctl enable myapp` and `systemctl start myapp`.*

## 7. Hands-on Step-by-Step (VERY IMPORTANT)

**Step 1: Start a generic background process**
```bash
sleep 1000 &
```
> *The `&` puts it in the background. The terminal will output a PID (e.g., 4231).*

**Step 2: Verify it is running**
```bash
ps aux | grep sleep
```

**Step 3: Forcefully kill the process**
```bash
kill -9 <your-PID-here>
```

**Step 4: Start a temporary web server on port 9000**
```bash
python3 -m http.server 9000 &
```

**Step 5: Prove the port is listening locally**
```bash
curl http://localhost:9000
```

**Step 6: Identify what holds port 9000 and kill it**
```bash
ss -tulpn | grep 9000
# Note the PID, then kill it.
kill -9 <PID>
```

## 8. Common Errors & Troubleshooting

- **Error: `Address already in use (bind failed)`**
  - **Issue:** Your app is trying to start on Port 80, but Nginx or Apache is already running on that port.
  - **Fix:** Use `ss -tulpn | grep :80` to find the PID, and kill/stop the conflicting service.
- **Error: `Connection refused` (via curl/telnet)**
  - **Issue:** The server is reachable, but absolutely nothing is listening on that specific port. The app crashed or is bound to `127.0.0.1` instead of `0.0.0.0`.
  - **Fix:** Check if the app is running (`ps aux`). Check its bind address inside its configuration.
- **Error: `Connection timed out` (via curl/telnet)**
  - **Issue:** A firewall (like AWS Security Groups or `ufw`) is dropping the packets.
  - **Fix:** Open the required port in your infrastructure's firewall rules.

## 9. Best Practices
1. **Never use `kill -9` immediately:** Always try a regular `kill <PID>` first (SIGTERM). This allows the application to cleanly save data and close database connections. Only use `-9` (SIGKILL) if it's completely frozen.
2. **Bind exclusively to localhost if private:** If an application doesn't need to be accessed from the outside (like a local cache), configure it to listen on `127.0.0.1:port` rather than `0.0.0.0:port` for security.
3. **Use Systemd for persistence:** Do not use `tmux` or `screen` to run production apps. Write a `systemd` service file so Linux automatically restarts the app if it crashes.

## 10. Interview Questions & Answers

**Q1: What is a Zombie process?**
**A1:** A process that has completed execution but still has an entry in the process table because its parent process hasn't read its exit status. 

**Q2: How do you check which process is consuming the most memory?**
**A2:** Run the `top` command and press `Shift + M` to sort the live list by RAM consumption.

**Q3: What's the difference between `SIGTERM` (15) and `SIGKILL` (9)?**
**A3:** `SIGTERM` is a polite request asking the process to terminate, allowing it to execute cleanup routines. `SIGKILL` forcefully terminates the process at the kernel level without warning, potentially causing data corruption.

**Q4: A developer says "I can't ping the database server, it's down!". The DB admin says it's running. How do you troubleshoot?**
**A4:** Ping uses ICMP, which is often blocked by firewalls. I would use `nc -zv <db-ip> 5432` or `telnet <db-ip> 5432` to test the actual TCP port connectivity.

**Q5: What command helps you find the PID holding port 443?**
**A5:** `netstat -tulpn | grep 443` or `ss -tulpn | grep 443`.

## 11. Quick Revision Summary
- **ps aux:** Snapshot of all running processes.
- **kill:** Sends termination signals to PIDs. Use `-9` as a last resort.
- **netstat / ss:** Verifies which processes are bound to which network ports.
- **nc / telnet:** Tests firewall and port accessibility across networks.

## 12. Official Documentation Links
- [Linux man-pages: ps(1)](https://man7.org/linux/man-pages/man1/ps.1.html)
- [Linux man-pages: ss(8)](https://man7.org/linux/man-pages/man8/ss.8.html)
