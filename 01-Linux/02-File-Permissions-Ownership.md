# 🔐 Linux File Permissions and Ownership


## 🖼️ Quick Visual Summary

![Quick Summary: Linux File Permissions and Ownership](../assets/topic-summaries/linux-file-permissions-ownership.svg)

> **⚡ 80/20 Summary:** Owner/group/others decide access • rwx maps to read/write/execute • chmod changes modes • chown changes ownership

## 1. 🎯 Overview
In Linux, security is tightly controlled at the file and directory level. Every file is owned by a specific **User** and a specific **Group**, and has strict permissions dictating exactly who can **Read**, **Write**, or **Execute** it.

## 2. 💡 Why This Matters
- **Security & Blast Radius:** If a web server is hacked, correctly set permissions prevent the hacker from modifying system core files or reading database passwords.
- **Application Booting:** Many applications (like SSH keys or Docker sockets) will flat-out refuse to run if their file permissions are too open.
- **Multi-tenant Safety:** Allows multiple engineers to use the same machine safely without accidentally overwriting each other's configurations.

## 3. 🧠 Core Concepts
- **Three Identity Tiers (U-G-O):**
  - **User (u):** The owner of the file.
  - **Group (g):** Other users who belong to the file's group.
  - **Others (o):** Everyone else in the system (the public).
- **Three Permission Types:**
  - **Read (`r` / 4):** View file contents or list directory contents.
  - **Write (`w` / 2):** Modify/delete the file or add files to a directory.
  - **Execute (`x` / 1):** Run the file as a script/program, or enter a directory (`cd`).
- **Root User:** The superuser who bypasses all permission restrictions.

## 4. 🧭 Architecture / Workflow
When you run `ls -l`, you see a string like `-rwxr-xr--`. 
1. The first character determines the type ( `-` for file, `d` for directory, `l` for symlink).
2. The next 3 chars (`rwx`) are the **User** permissions. (Read, Write, Execute).
3. The next 3 chars (`r-x`) are the **Group** permissions. (Read, Execute, No Write).
4. The final 3 chars (`r--`) are **Others'** permissions. (Read only).

## 5. 🛠️ Commands & Practical Usage

Change file permissions using numeric values (The standard way):
```bash
chmod 755 script.sh
```
> *7 (User: r+w+x), 5 (Group: r+x), 5 (Others: r+x).*

Change file ownership (User and Group):
```bash
chown ubuntu:www-data config.yaml
```
> *Changes owner to `ubuntu` and group to `www-data`.*

Recursively fix permissions for an entire web directory:
```bash
chmod -R 644 /var/www/html/
```
> *Changes all files inside to Read/Write for owner, Read for others.*

Add execute permission quickly without numbers:
```bash
chmod +x deploy.sh
```

## 6. ⚙️ Configuration / Code Examples
A common startup script that ensures SSH keys are properly secured before attempting a connection:

```bash
#!/bin/bash

KEY_FILE="~/.ssh/id_rsa"

# Check if the key is too open (e.g. 644 instead of 600)
# This will force the key to Read/Write for OWNER ONLY.
echo "Securing SSH key permissions..."
chmod 600 $KEY_FILE
echo "Key secured. Launching connection..."

ssh -i $KEY_FILE user@production-server
```

## 7. 🧪 Hands-on Step-by-Step

**Step 1: Create a secure configuration file**
```bash
touch secrets.env
```

**Step 2: Check default permissions**
```bash
ls -l secrets.env
# Output usually looks like: -rw-r--r-- (644)
```

**Step 3: Lock down the file so ONLY you can read it**
```bash
chmod 600 secrets.env
```

**Step 4: Verify the lockdown**
```bash
ls -l secrets.env
# Output is now: -rw-------  (No one else can read or write)
```

**Step 5: Change ownership to simulate a service hand-off (needs sudo)**
```bash
sudo chown root:root secrets.env
```

## 8. 🚨 Common Errors & Troubleshooting

- **Error: `Permission denied`**
  - **Issue:** Your user account does not have read/write access to the target file.
  - **Fix:** Prefix command with `sudo` if you need administrative rights, or fix the permissions using `chmod`/`chown`.
- **Error: `bash: ./script.sh: Permission denied`**
  - **Issue:** The script exists, but lacks the "Execute" (`x`) bit.
  - **Fix:** Run `chmod +x script.sh`.
- **Error: `WARNING: UNPROTECTED PRIVATE KEY FILE!`**
  - **Issue:** SSH refuses to use your private key because its permissions are too open (e.g., others can read it).
  - **Fix:** Execute `chmod 600 <your-key.pem>`. 

## 9. ✅ Best Practices
1. **Principle of Least Privilege:** Never give `777` permissions (Read/Write/Execute for everyone) to solve a "Permission denied" error. It creates massive security holes.
2. **Directories need Execute:** A directory must have the Execute (`x`) permission for a user to `cd` into it. 
3. **Use Groups securely:** Instead of making a file public, create a specific Linux group, assign the relevant users to that group, and give the group access.

## 10. 🎤 Interview Questions & Answers

**Q1: What does `chmod 777` do and why is it dangerous?**
**A1:** It gives read, write, and execute permissions to the User, Group, and Everyone else in the world. It is a severe risk because any compromised process or rogue user on the system can modify or execute that file.

**Q2: What is the numeric value to give the owner read/write, and everyone else read-only?**
**A2:** 644. Owner: 4(R)+2(W)=6. Group: 4(R)=4. Others: 4(R)=4.

**Q3: How do you recursively change the owner of a directory and all its contents?**
**A3:** Using the `-R` flag: `chown -R user:group /path/to/folder`.

**Q4: A script you just downloaded says "Permission denied" when you try to run it via `./setup.sh`. Why?**
**A4:** The file lacks execute permissions. Fix it with `chmod +x setup.sh`.

**Q5: What happens if a directory has Read permissions but does NOT have Execute permissions?**
**A5:** You can list the files inside it using `ls`, but you cannot enter it using `cd` or read the contents of the files inside it.

## 11. ⚡ Quick Revision Summary
- **r=4, w=2, x=1**
- **chmod:** Changes the permission bits (e.g., 755, 600).
- **chown:** Changes the user and group ownership of the file.
- **Golden Rules:** SSH keys must be 600. Scripts need `+x` to run. Never use 777.

## 12. 🔗 Official Documentation Links
- [GNU Coreutils: chown](https://www.gnu.org/software/coreutils/manual/html_node/chown-invocation.html)
- [GNU Coreutils: chmod](https://www.gnu.org/software/coreutils/manual/html_node/chmod-invocation.html)
