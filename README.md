# 🚀 Jenkins Apache Logs Pipeline

## 📌 Task: 09 CI/CD Jenkins

This repository contains a Jenkins Pipeline solution for installing and verifying an Apache/httpd web server on a remote Linux VM.

The pipeline is stored in a `Jenkinsfile`, uploaded to GitHub, and executed in Jenkins using **Pipeline from SCM**.

---

## ✅ What Was Implemented

The task required creating a Jenkins pipeline that:

* Installs Jenkins and required plugins.
* Uses a `Jenkinsfile` stored in a GitHub repository.
* Connects to a remote Linux VM.
* Detects the operating system.
* Installs Apache2/httpd.
* Starts and enables the web server.
* Verifies that the web server is working.
* Reads the Apache/httpd access log.
* Checks the log file for `4xx` and `5xx` HTTP errors.
* Provides screenshots or a short wrap-up video for submission.

---

## 🧱 Project Architecture

```text
GitHub Repository
      │
      │ Jenkinsfile
      ▼
Jenkins Pipeline from SCM
      │
      │ SSH connection
      ▼
Remote Ubuntu VM
      │
      ├── Detect OS
      ├── Install Apache2
      ├── Start Apache2 service
      ├── Create test web page
      ├── Check HTTP response
      └── Analyze Apache access logs
```

---

## 🖥️ Environment

| Component          | Value                                                           |
| ------------------ | --------------------------------------------------------------- |
| Jenkins controller | Windows local machine                                           |
| Jenkins workspace  | `C:\ProgramData\Jenkins\.jenkins\workspace\jenkins-apache-logs` |
| Target VM          | Ubuntu Linux VM                                                 |
| Target OS          | Ubuntu 26.04 LTS                                                |
| Target VM IP       | `192.168.56.102`                                                |
| SSH user           | `nick`                                                          |
| Web server         | Apache2                                                         |
| Repository         | `jenkins-apache-logs`                                           |

---

## 🔗 Repository

```text
https://github.com/unikol/jenkins-apache-logs
```

---

## 🔐 Jenkins Credentials

The pipeline uses Jenkins Global Credentials for secure SSH authentication.

Credential type:

```text
SSH Username with private key
```

Credential ID:

```text
jenkins-apache-ssh-key
```

SSH user:

```text
nick
```

The private SSH key is stored only in Jenkins Credentials and is **not stored in the GitHub repository**.

---

## ⚙️ Jenkins Plugins Used

The following Jenkins plugins were used or required:

```text
Pipeline
Git plugin
Credentials Binding
SSH Agent / SSH credentials support
```

---

## 🧩 Pipeline Stages

The final Jenkins pipeline contains the following stages:

```text
1. Checkout
2. Prepare deployment scripts
3. Test SSH connection
4. Install and start Apache/httpd
5. Check web server logs for 4xx and 5xx
6. Post Actions
```

---

## 🔄 Pipeline Workflow

### 1. Checkout Repository

Jenkins retrieves the project from GitHub:

```text
GitHub → Jenkins Pipeline from SCM → Jenkinsfile
```

---

### 2. Prepare Deployment Scripts

The pipeline dynamically creates two helper scripts:

```text
deploy_apache.sh
check_logs.sh
```

These scripts are later copied to the remote VM through `scp`.

---

### 3. Test SSH Connection

Jenkins checks that it can connect to the VM:

```bash
hostname
whoami
sudo -n whoami
```

Expected result:

```text
ubuntu-26
nick
root
```

This confirms that Jenkins can connect to the VM and run privileged commands using passwordless sudo.

---

### 4. Install and Start Apache2/httpd

The pipeline detects the package manager:

```text
apt-get → apache2
dnf/yum → httpd
```

For Ubuntu, the pipeline installs Apache2:

```bash
sudo apt-get update -y
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y apache2
```

Then it enables and starts the service:

```bash
sudo systemctl enable apache2
sudo systemctl restart apache2
```

---

### 5. Create Test Web Page

The pipeline creates a simple test page:

```html
<h1>Deployed by Jenkins Pipeline</h1>
<p>Apache/httpd installation completed successfully.</p>
```

The page is saved to:

```text
/var/www/html/index.html
```

---

### 6. Verify HTTP Response

The pipeline verifies the web server with:

```bash
curl -I http://localhost
```

Successful result:

```text
HTTP/1.1 200 OK
Server: Apache/2.4.66 (Ubuntu)
```

---

### 7. Check Apache/httpd Logs

The pipeline searches for the access log:

```text
/var/log/apache2/access.log
/var/log/httpd/access_log
```

For Ubuntu, the log file was:

```text
/var/log/apache2/access.log
```

The pipeline checks for `4xx` and `5xx` HTTP status codes using `awk`.

Example logic:

```bash
awk '$9 ~ /^[45][0-9][0-9]$/ {print}' /var/log/apache2/access.log
```

---

## ✅ Final Result

The Jenkins pipeline completed successfully.

Important successful output:

```text
HTTP/1.1 200 OK
No 4xx or 5xx errors found in the access log.
Deployment completed successfully. Apache/httpd is installed and logs were checked.
Finished: SUCCESS
```

---

## 🌐 Deployed Web Page

The deployed Apache page is available from the host machine:

```text
http://192.168.56.102
```

Expected page content:

```text
Deployed by Jenkins Pipeline
Apache/httpd installation completed successfully.
```
---

## 🧠 Problems Solved During Implementation

### Problem 1: Jenkins on Windows and SSH Key Permissions

Jenkins was running as:

```text
nt authority\system
```

Windows OpenSSH rejected the temporary Jenkins SSH key because permissions were too open.

The pipeline fixed this using:

```bat
icacls "%SSH_KEY_FILE%" /inheritance:r
icacls "%SSH_KEY_FILE%" /grant:r "%CURRENT_USER%:R"
```

---

### Problem 2: Invalid SSH Key Format

At one stage, Jenkins reported:

```text
Load key "...ssh-key-SSH_KEY_FILE": invalid format
```

The issue was fixed by correctly adding the **private SSH key** to Jenkins Credentials instead of using an incorrect or incomplete key.

---

### Problem 3: Groovy `$` Escaping Issue

The Jenkinsfile initially failed because Groovy interpreted `$` inside a shell command.

Instead of using:

```bash
sed -i 's/\r$//'
```

the pipeline was changed to:

```bash
tr -d '\r'
```

This removed Windows CRLF characters from shell scripts before running them on Linux.

---

## 📚 What I Learned

During this task, I practiced:

* Jenkins Pipeline from SCM.
* Jenkinsfile creation.
* GitHub integration with Jenkins.
* Jenkins Credentials usage.
* SSH key authentication from Jenkins to Linux VM.
* Windows Jenkins controller limitations.
* Remote Linux server configuration.
* Apache2 installation automation.
* Service verification with `systemctl`.
* HTTP verification with `curl`.
* Apache access log analysis.
* Handling 4xx and 5xx HTTP errors in CI/CD pipeline logic.

---

## 🏁 Summary

This project demonstrates a Jenkins deployment pipeline that connects to a remote Linux VM, installs Apache2/httpd, verifies the web server status, and checks access logs for client and server errors.

The final build status was:

SUCCESS
