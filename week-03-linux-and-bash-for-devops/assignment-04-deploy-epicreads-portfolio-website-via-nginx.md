# Assignment 4 — Deploy EpicReads Portfolio Website via Nginx

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will deploy a static portfolio website on an Ubuntu VM using Nginx. You will download the website template, add your ownership proof in the footer, deploy the files to the Nginx web root, and verify the website is publicly accessible via a browser.

---

# Task 0 — Pre-flight Check

## Goal

Verify the Ubuntu VM and Nginx are ready for deployment.

### Evidence

#### Screenshot 0 — Output of `sudo systemctl status nginx --no-pager` showing Active (running)

<img width="523" height="210" alt="image" src="https://github.com/user-attachments/assets/eef058f8-21ae-4313-acb4-6ac72bca0054" />

---

# Task 1 — Get the Website Source Code

## Goal

Download and extract the portfolio website template.

### Evidence

#### Screenshot 1 — Output of `ls -la` showing the extracted project folder

<img width="371" height="113" alt="image" src="https://github.com/user-attachments/assets/036b9cff-6d35-43ca-b4dd-beda2fce72cf" />


---

# Task 2 — Add Ownership Proof (Anti-Copy Change)

## Goal

Update the website footer with your deployment details.

### Evidence

#### Screenshot 2 — Nano editor open with the updated footer showing your Full Name, Group, Week, and Date

<img width="563" height="315" alt="image" src="https://github.com/user-attachments/assets/d3ca6390-7284-4686-9d60-9c121b1764ad" />


---

# Task 3 — Deploy Website via Nginx

## Goal

Deploy the portfolio website to the Nginx web root.

### Evidence

#### Screenshot 3 — Output of `sudo nginx -t` showing configuration test successful

<img width="342" height="35" alt="image" src="https://github.com/user-attachments/assets/7b53d3bb-c59b-4b6f-a91f-3e299875671e" />


---

#### Screenshot 4 — Output of `ls /var/www/html` showing deployed website files

<img width="479" height="44" alt="image" src="https://github.com/user-attachments/assets/485523cd-e005-4b96-9b89-2014ddb4f8e1" />


---

# Task 4 — Verify Website is Live

## Goal

Verify the deployed website is publicly accessible and the footer contains your details.

### Evidence
<img width="668" height="139" alt="image" src="https://github.com/user-attachments/assets/45650d04-60f9-4a41-82a5-ce540005f028" />

#### Screenshot 5 — Output of `curl ifconfig.me` showing the server's public IP address


---<img width="380" height="53" alt="image" src="https://github.com/user-attachments/assets/b4184393-124c-494a-8f16-f160311785fb" />


#### Screenshot 6 — Browser showing the live website with your Full Name and deployment details in the footer

<img width="371" height="134" alt="image" src="https://github.com/user-attachments/assets/2814a4d9-9741-40d1-88a9-a0fd08f75f12" />

<img width="680" height="335" alt="image" src="https://github.com/user-attachments/assets/4f968860-a94e-483a-a85a-17ad74db6a1a" />

---

# Task 5 — Mini Real DevOps Operational Check

## Goal

Verify the deployed website and Nginx service are healthy.

### Evidence

#### Screenshot 7 — Output of `systemctl is-enabled nginx`

<img width="446" height="53" alt="image" src="https://github.com/user-attachments/assets/9260dc5d-39c1-4d25-a8ee-0b6e85f3157f" />


---

#### Screenshot 8 — Output of `curl -I http://localhost` showing 200 OK

<img width="354" height="84" alt="image" src="https://github.com/user-attachments/assets/8a22144d-9402-459a-9147-56e26c4f83c2" />


---

# LinkedIn Post (Mandatory)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`Add your URL here`

---

#### Screenshot — Published LinkedIn post showing the live website with your Full Name in the footer

Add your screenshot here.

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Ownership proof in the footer is mandatory
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [ ] Screenshot 0: Nginx service status (active/running)
- [ ] Screenshot 1: Website files downloaded and extracted
- [ ] Screenshot 2: Footer updated with Full Name, Group, Week, and Date
- [ ] Screenshot 3: Nginx configuration test successful
- [ ] Screenshot 4: Website files deployed to /var/www/html
- [ ] Screenshot 5: Public IP retrieved
- [ ] Screenshot 6: Live website accessible in browser with footer details
- [ ] Screenshot 7: Nginx enabled on boot
- [ ] Screenshot 8: Local HTTP response returns 200 OK
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
