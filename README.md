# ansible-automation-playbooks

Ansible playbooks and templates built from real production deployments at
**Incresol Software Services**. Used to automate deployments, server configuration,
and infrastructure setup across multiple client environments.

---

## Repository Structure

```
ansible-automation-playbooks/
├── playbooks/
│   ├── deploy-angular-nginx.yml       # Deploy Angular dist/ to Nginx
│   ├── deploy-springboot-tomcat.yml   # Deploy Spring Boot JAR to Tomcat 9
│   ├── setup-nginx.yml                # Install + configure Nginx
│   ├── setup-mongodb-replica.yml      # MongoDB 3-node replica set
│   ├── setup-docker.yml               # Install Docker + Docker Compose
│   └── initial-server-setup.yml       # New server bootstrap + hardening
├── roles/
│   ├── webserver/
│   │   └── templates/
│   │       ├── nginx-angular.conf.j2          # Nginx config for Angular SPA
│   │       └── nginx-reverse-proxy.conf.j2    # Nginx reverse proxy + load balancer
│   └── database/
│       └── templates/
│           └── mongod.conf.j2                 # MongoDB replica set config
├── inventory/
│   └── hosts.ini                      # Server groups and connection details
├── group_vars/
│   ├── webservers.yml                 # Variables for web servers
│   └── appservers.yml                 # Variables for app servers
└── README.md
```

---

## Playbook Details

### 1. deploy-angular-nginx.yml
**Projects: AspTax, P-Collab**

Deploys a built Angular application (`dist/` folder) to an Nginx web server.

**What it does:**
- Checks Nginx is installed and running
- Creates a timestamped backup of the existing deployment
- Copies Angular `dist/<app-name>/` to the Nginx web root
- Deploys Nginx site configuration from template
- Validates config with `nginx -t` before reloading

```bash
ansible-playbook playbooks/deploy-angular-nginx.yml \
  --extra-vars "env=dev app_name=asptax-web"
```

---

### 2. deploy-springboot-tomcat.yml
**Projects: Nagpur Metro, Aragen, Kaveri Seeds, Sonic Biochem, Daimler**

Deploys a Spring Boot JAR to a Tomcat 9 server with backup and health check.

**What it does:**
- Backs up existing JAR with timestamp
- Stops Tomcat → copies new JAR → starts Tomcat
- Waits for port 8080 to be available
- Runs health check on `/actuator/health` endpoint
- Auto rollback logic built into Jenkins pipeline

```bash
ansible-playbook playbooks/deploy-springboot-tomcat.yml \
  --extra-vars "env=prod app_name=my-app"
```

---

### 3. setup-nginx.yml
**Projects: AspTax, Node App High-Availability**

Installs Nginx and configures it as a reverse proxy or static file server.

**What it does:**
- Installs Nginx
- Opens ports 80 and 443 in UFW firewall
- Deploys reverse proxy config from Jinja2 template
- Removes default Nginx site

```bash
ansible-playbook playbooks/setup-nginx.yml
```

---

### 4. setup-mongodb-replica.yml
**Project: Node Application High-Availability Deployment**

Sets up a 3-node MongoDB replica set for high availability and automatic failover.

**Replica set topology:**
```
mongo-primary   (priority: 2)  ← handles all writes
mongo-secondary (priority: 1)  ← can serve reads, takes over if primary fails
mongo-arbiter   (arbiterOnly)  ← votes in elections, no data stored
```

**What it does:**
- Installs MongoDB 6.0 on all 3 nodes
- Deploys `mongod.conf` with replica set name
- Initiates replica set from primary node
- Verifies replica set status

```bash
ansible-playbook playbooks/setup-mongodb-replica.yml
```

---

### 5. setup-docker.yml
**Projects: AspTax containerization, all Docker-based deployments**

Installs Docker Engine and Docker Compose on Ubuntu servers.

**What it does:**
- Removes old Docker versions
- Adds Docker official repository
- Installs Docker CE + Docker Compose plugin
- Adds user to docker group (no sudo needed)
- Runs `hello-world` to verify installation

```bash
ansible-playbook playbooks/setup-docker.yml
```

---

### 6. initial-server-setup.yml
**Projects: On-Premises Server Maintenance, IDigiPro**

Bootstrap and harden a fresh Ubuntu/CentOS server before any application deployment.

**What it does:**
- Updates all system packages
- Creates a `devops` user with SSH key access
- Hardens SSH — disables root login and password auth
- Configures UFW firewall (allow SSH, HTTP, HTTPS only)
- Sets timezone to Asia/Kolkata
- Installs essential tools: git, vim, htop, fail2ban, logrotate
- Configures log rotation for application logs

```bash
ansible-playbook playbooks/initial-server-setup.yml
```

---

## Nginx Templates

### nginx-angular.conf.j2
Serves Angular SPA with `try_files` routing (required for Angular router), static asset
caching, gzip compression, and security headers.

### nginx-reverse-proxy.conf.j2
Configures Nginx as a reverse proxy with `upstream` load balancing across 3 backend
servers using `least_conn` algorithm — used in the Node app high-availability project.

---

## Prerequisites

```bash
# Install Ansible
pip install ansible

# Verify
ansible --version

# Test connectivity to all servers
ansible all -i inventory/hosts.ini -m ping
```

---

## How to Use

**1. Clone this repo**
```bash
git clone https://github.com/your-username/ansible-automation-playbooks.git
cd ansible-automation-playbooks
```

**2. Update inventory**

Edit `inventory/hosts.ini` — replace the placeholder IPs with your actual server IPs.

**3. Run a playbook**
```bash
# Dry run first (check mode — no changes applied)
ansible-playbook playbooks/deploy-angular-nginx.yml --check

# Full run
ansible-playbook playbooks/deploy-angular-nginx.yml \
  --extra-vars "app_name=asptax-web env=prod"
```

---

## Real Projects These Playbooks Supported

| Project | Client / Platform | Playbooks Used |
|---|---|---|
| AspTax | Incresol (tax platform) | deploy-angular-nginx, setup-nginx, setup-docker |
| P-Collab | Incresol (collaboration tool) | deploy-angular-nginx, setup-nginx |
| Nagpur Metro | Metro rail authority | deploy-springboot-tomcat |
| Aragen | Life sciences company | deploy-springboot-tomcat |
| Kaveri Seeds | Agriculture company | deploy-springboot-tomcat |
| Daimler | Automotive client | deploy-springboot-tomcat |
| Node App HA | Intelisa Bosch | setup-mongodb-replica, setup-nginx |
| IDigiPro | Digital signature platform | initial-server-setup |
| On-Premises Infra | Incresol internal | initial-server-setup, setup-docker |

---

## Tools & Versions

| Tool | Version |
|---|---|
| Ansible | 2.14+ |
| Python | 3.10+ |
| Ubuntu | 20.04 / 22.04 LTS |
| Nginx | 1.24+ |
| MongoDB | 6.0 |
| Docker | 24.x |
| Java | 8 (OpenJDK) |
| Tomcat | 9.x |

---

## Author

**Pavan Kishore Nakka**
DevOps & Cloud Engineer | 3+ Years Experience
AWS Certified Solutions Architect – Associate | AWS Certified Cloud Practitioner

[LinkedIn](https://www.linkedin.com/in/nakka-pavan-kishore-103226221/)
