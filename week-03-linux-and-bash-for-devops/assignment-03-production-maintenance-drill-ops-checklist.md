# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal

Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI

![alt text](image-12.png)

---

#### Screenshot 2 — Output of `ip a`

![alt text](image-13.png)

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

![alt text](image-14.png)

---

#### Screenshot 4 — Output of `sudo ufw status`

![alt text](image-17.png)

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

In the sudo ss -tulpen output, the line tcp LISTEN 0.0.0.0:80 ... nginx confirms this. The 0.0.0.0 means Nginx is bound to all network interfaces, not just localhost, so it can accept HTTP connections from any IP address, including external traffic from the internet. The process name nginx alongside the port confirms it's specifically Nginx holding this port open, not another service.


---

**2. What proves SSH is active on port 22?**

The same ss -tulpen output shows tcp LISTEN 0.0.0.0:22 ... sshd, confirming the SSH daemon (sshd) is actively listening on port 22 across all interfaces. This is what allows remote login to the server (e.g., via ssh ubuntu@)

---

**3. Did you find any unexpected open ports? Explain briefly.**

No unexpected ports were found. Aside from Nginx (port 80) and SSH (port 22), the only other listening services were chronyd (time sync) and systemd-resolved (DNS resolution), both bound only to loopback addresses (127.0.0.1, 127.0.0.53, 127.0.0.54), meaning they're not reachable from outside the server. This confirms only the two intended services, the web server and SSH, are externally exposed.

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

<img width="495" height="215" alt="image" src="https://github.com/user-attachments/assets/818fc09a-a594-4eca-9b70-cacb3069d4b0" />


---

#### Screenshot 2 — Output of `sudo nginx -t`

![alt text](image-18.png)


---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

![alt text](image-19.png)



### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

If Nginx fails to restart, the website becomes completely unreachable, since Nginx is the only process serving HTTP traffic on port 80. Any user visiting the site would get a connection error or timeout, since nothing would be listening on that port anymore. This is especially risky if the failure happens during a deployment or config change, since it means the site could go down with no automatic recovery, requiring manual intervention to diagnose and fix


**2. What's your basic rollback plan?**

Before making any configuration change, always run sudo nginx -t first to validate the config syntax — this catches most errors before they ever reach a restart. If a restart is attempted and fails, the first step is to check systemctl status nginx --no-pager and sudo journalctl -u nginx --no-pager -n 50 to see the exact error.

If the failure is due to a bad configuration change, the fix is to revert the config file back to its last known-good version (ideally from a backup or version control) and re-run sudo nginx -t followed by sudo systemctl restart nginx again. Keeping a backup copy of the working config before making changes is the simplest safeguard, since it allows an immediate rollback without needing to debug under pressure.

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

Add your screenshot here.

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

![alt text](image-22.png)
---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

![alt text](image-23.png)

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

No errors were found in either the error log or the journalctl output. The error log returned only one entry whihc I mentioned below an it is not an error, and the journalctl entries show only clean Started, Stopped, Reloaded, and Deactivated successfully events — no failed or exited with status lines anywhere.

2026/07/14 02:06:10 [notice] 24079#24079: using inherited sockets from "5;6;" means When Nginx is reloaded or upgraded gracefully, the old master process passes the listening sockets to the new master process so that active connections are not interrupted.

---

**2. If there were no errors, what does that indicate about the system?**

An empty error log and a clean journalctl history indicate that Nginx has not encountered any internal errors, misconfigurations, or failed lifecycle events during the period covered by these logs. This is a positive signal about current system health, but it is not a permanent guarantee

it only reflects that nothing went wrong during the window actually checked. New issues could still appear as traffic, config changes, or system conditions change, so logs like these need to be checked periodically rather than just once.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Yes. The curl request appeared in access.log as a GET / request from the server's own public IP with a 200 status and the user agent curl/8.18.0. This confirms the full traffic path is working end-to-end: the request left the client, traveled through the network, reached Nginx, was processed and served correctly, and was logged — proving there's no break anywhere in that chain..

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`
![alt text](image-24.png)


#### Screenshot 2 — Output of `free -h`

![alt text](image-25.png)

---

#### Screenshot 3 — Output of `df -h`

![alt text](image-26.png)

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

![alt text](image-27.png)



### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

None of the three resources show any critical signal at this moment — CPU is idle, memory has healthy available headroom with zero swap pressure, and disk is at a comfortable 59%. If forced to rank which one deserves the closest ongoing attention as this server scales, it would be disk, since it's the resource most likely to silently creep upward over time (via log growth or package cache accumulation) without any obvious symptom until it's suddenly critical — unlike CPU or memory pressure, which usually show visible slowness first.

---

**2. What happens if disk becomes 100% full in a production server?**

Logs stop being able to write new entries, which is especially dangerous because that's often exactly when you need logs most — during an active incident. Applications (including build tools and package managers) can fail or crash if they need scratch space to write temporary files. If a database were running locally, it could refuse writes or become corrupted. In severe cases, the OS itself can become unstable — even basic operations like logging in via SSH can fail if there's truly no disk space left for the system to work with.


---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

![alt text](image-28.png)

---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

![alt text](image-29.png)

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

![alt text](image-30.png)


---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

ls -lah /var/www/html confirmed the presence of a genuine Create React App production build — index.html, a static/ folder with compiled JS/CSS bundles, and standard CRA metadata files — all owned by www-data, the user Nginx's worker processes run as.

grep -R "Deployed by" confirmed the specific custom identifying text was compiled into the live JavaScript bundle and matched the original source via the accompanying source map — proving this exact build, not a stale or generic one, is what's live.

grep -n "try_files" confirmed Nginx's config correctly falls back to index.html for unmatched routes, ensuring the SPA behaves correctly for all application routes, not just the homepage.

Finally, this was cross-checked against the earlier curl test in Task 3, which showed the live server actually returning this exact index.html content over HTTP — tying the on-disk files to what's genuinely being served to real users.
---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

![alt text](image-31.png)


---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)


![alt text](image-32.png)

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

<img width="500" height="42" alt="image" src="https://github.com/user-attachments/assets/1d2a5dc6-d0cd-4633-b7ac-e8261db11b0a" />


---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

The web root directory (/var/www/html) — the exact path Nginx serves content from — was emptied of all deployment files. Nginx itself remained running and correctly configured, but with no content present and no fallback file available either, it returned a 500 Internal Server Error instead of serving the React application.

---

**2. How did you fix the issue?**

The original deployment had been safely backed up beforehand (moved to html_backup rather than deleted), so recovery involved removing the empty broken directory and moving the backup back into place at the correct path. Nginx was restarted to ensure it was serving cleanly from the restored files, and recovery was confirmed externally via curl -I, which returned 200 OK with identical content metadata (Content-Length, Last-Modified, ETag) to the pre-incident state — proving the exact same build was successfully restored.
---

**3. How can you avoid this kind of issue in real production systems?**

Automated pre-deployment backups, so every release can be instantly rolled back without manual intervention.

Deploying to a versioned, separate directory and atomically switching a symlink (e.g., /var/www/current) to point to it, 3. rather than overwriting the live directory in place — this way a failed deploy never leaves the live path empty or half-written.

CI/CD pipeline checks that verify a deployment actually succeeded (e.g., confirming index.html exists and is non-empty) before marking the release complete.

Post-deployment health checks/monitoring that automatically verify the live site returns a healthy 200 response immediately after every deploy, catching this kind of failure within seconds rather than relying on someone noticing manually.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

![alt text](image-33.png)

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![alt text](image-34.png)
---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

The web root directory (/var/www/html) — the exact path Nginx serves content from — was emptied of all deployment files. Nginx itself remained running and correctly configured, but with no content present and no fallback file available either, it returned a 500 Internal Server Error instead of serving the React application.
---

**2. How did you fix the issue and restore the application?**

The original deployment had been backed up by moving it to `html_backup`, enabling a quick recovery. The empty broken directory was removed, the backup was restored to its original location, and Nginx was restarted. Recovery was verified using `curl -I`, which returned **200 OK** with matching `Content-Length`, `Last-Modified`, and `ETag` values, confirming the exact same website build had been successfully restored.


---

**3. What steps would you take to prevent this kind of issue in real production systems?**

**Key deployment best practices (in brief):**

* **Automated pre-deployment backups:** Create a backup before every deployment so the previous version can be restored instantly without manual intervention.
* **Atomic deployments:** Deploy the new release to a versioned directory and switch a symlink (e.g., `/var/www/current`) only after deployment succeeds, preventing partial or broken updates.
* **CI/CD deployment validation:** Automatically verify that critical files (such as `index.html`) exist and are valid before marking the deployment as successful.
* **Post-deployment health checks:** Run automated health checks after every deployment to confirm the application returns a healthy HTTP 200 response, enabling immediate detection and rollback of failed releases.


---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

**SSH key-based authentication** is more secure than passwords because it uses a **public-private key pair**. The **public key** is stored on the server, while the **private key** stays securely on the user's device and is never shared.

Since the private key is never transmitted over the network, it is much harder to intercept or steal. SSH keys are also far more complex than passwords, making them highly resistant to brute-force attacks.

Additionally, SSH keys simplify access management by allowing each user to have a unique key, so administrators can easily grant or revoke access without changing passwords for everyone.


**1. Why is SSH key-based authentication more secure than sharing passwords?**

SSH key-based authentication is more secure than sharing passwords because it uses a pair of cryptographic keys: a public key and a private key. The public key is stored on the server, while the private key remains on the user's machine and is never shared.

Unlike passwords, SSH keys are not transmitted over the network during authentication, reducing the risk of interception or theft. SSH keys are also much longer and more complex than passwords, making them extremely difficult to crack through brute-force attacks.

Another advantage is better access management. Each user can have their own SSH key, allowing administrators to grant or revoke access for individual users without changing passwords for everyone.
---

**2. Why should only required ports be open on a production server?**

Only the required ports should be open on a production server to minimize security risks and reduce the attack surface. Every open port represents a potential entry point that attackers can use to exploit vulnerabilities or gain unauthorized access.

By allowing only necessary ports, such as port 80 for HTTP, port 443 for HTTPS, and port 22 for SSH when required, organizations can better protect their applications and data from unauthorized access and cyberattacks.

Restricting unnecessary ports also improves system security, simplifies firewall management, and helps meet security best practices and compliance requirements. Therefore, following the principle of least privilege for network access is essential for maintaining a secure production environment.

---

**3. Why is it important for Nginx to be enabled on boot?**

It is important for Nginx to be enabled on boot so that the web server starts automatically whenever the server is restarted or recovers from a power failure. This ensures that the application or website remains accessible without requiring manual intervention from an administrator.

Automatically starting Nginx improves service availability, reduces downtime, and is considered a best practice for production environments.
---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Sharing secrets, keys, or credentials publicly can lead to unauthorized access to systems, applications, and cloud resources. Attackers can use exposed credentials to steal data, modify resources, disrupt services, or incur unexpected costs.

Publicly exposed credentials may also result in data breaches, security incidents, and compliance violations. Therefore, secrets and credentials should always be stored securely and never shared in public repositories, screenshots, or documents.
---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

Cloud resources should be stopped or terminated when they are no longer needed to avoid unnecessary costs and optimize resource usage. Unused resources such as virtual machines, storage volumes, and databases can continue to incur charges even when they are not actively being used.

Removing unused resources also improves security by reducing the attack surface and helps maintain a clean and efficient cloud environment.
---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:
https://www.linkedin.com/posts/binod-mahato-54b61624_dmi-cohort-4-live-micro-internship-waiting-share-7483961520016502784-_AAD/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAUTBIsB0-X0m-Hgz7jv1B_1Okv8NMfRKGc

---

#### Screenshot — Published LinkedIn post

![alt text](image-35.png)

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [ ] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [ ] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [ ] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [ ] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [ ] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [ ] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [ ] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [ ] Task 8: Security & Reliability Notes answered
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
