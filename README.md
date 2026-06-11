# Ansible VM Playbooks

A collection of 30+ Ansible playbooks for common VM management tasks on **Ubuntu/Debian** targets, with full **Ansible AWX** integration for REST API-driven execution.

---

## Playbooks

### Firewall

**`enable_firewall.yml`**  
Use when provisioning a new VM that has no firewall configured. Enables UFW, sets default-deny on incoming traffic, and opens only the ports you specify. Run this as one of the first steps after spinning up a fresh server.

**`add_firewall_ports.yml`**  
Use when you've deployed a new service on an existing VM and need to open additional ports without touching the current ruleset. Safe to run repeatedly — it only adds, never removes.

**`remove_firewall_ports.yml`**  
Use when decommissioning a service or tightening security on a VM. Removes specific UFW rules for ports you no longer need exposed. Run after stopping the corresponding service.

---

### Web Servers & Load Balancing

**`install_nginx.yml`**  
Use when you need a simple web server running on a VM — serving static files, acting as a reverse proxy, or as a base before further nginx configuration. Installs, enables, and verifies nginx is responding.

**`nginx_load_balancer.yml`**  
Use when you have two or more backend app servers and need to distribute traffic across them. Configures nginx as a Layer 7 HTTP load balancer with your choice of algorithm (round-robin, least-conn, or ip_hash for session stickiness). Includes a `/health` endpoint for upstream monitoring.

**`run_http_server.yml`**  
Use when you need a quick file-serving HTTP server without setting up full nginx — e.g. serving build artifacts, internal documentation, or testing static content. Runs Python's built-in HTTP server as a persistent systemd service.

**`deploy_static_website.yml`**  
Use when deploying a static frontend (HTML/CSS/JS) to a VM. Syncs your local build directory to the server, writes a full nginx virtual host config with gzip and cache headers, and optionally wires up an existing SSL certificate. Good for landing pages, docs sites, or SPAs.

---

### System Setup & Package Management

**`apt_install_package.yml`**  
Use any time you need to install one or more apt packages across a fleet of VMs. Accepts a list of packages and handles cache refresh. A general-purpose building block you'll call from other workflows.

**`install_python312.yml`**  
Use when your application requires Python 3.12 but the distro ships an older version. Adds the deadsnakes PPA and installs Python 3.12 with venv and dev headers. Optionally sets it as the system default `python3`.

**`install_nodejs.yml`**  
Use when setting up a Node.js application server. Installs the exact major version you specify (18, 20, or 22 LTS) via NodeSource, and optionally installs Yarn and PM2 for process management.

**`install_java.yml`**  
Use when deploying Java-based applications (Spring Boot, Kafka, Elasticsearch, etc.). Installs OpenJDK (choose JDK or JRE, any supported version), sets `JAVA_HOME` system-wide, and registers the version with `update-alternatives`.

**`setup_swap.yml`**  
Use on VMs with limited RAM (under 2 GB) or before running memory-intensive operations like compilations or database imports. Creates a swap file of configurable size and tunes `vm.swappiness` so the kernel uses swap conservatively.

**`configure_ntp.yml`**  
Use on any VM where accurate time is important — databases, distributed systems, TLS certificate validation, log correlation. Installs chrony, points it at your NTP pool servers, and sets the system timezone.

---

### Users & Access

**`create_user.yml`**  
Use when onboarding a new deploy user, developer, or service account onto a VM. Creates the user, adds them to specified groups (e.g. `sudo`, `docker`), installs their SSH public key, and optionally disables password login for the account.

---

### Databases & Caching

**`setup_postgresql.yml`**  
Use when provisioning a VM as a PostgreSQL database server. Installs the version you specify, creates a database and user with proper privileges, configures `pg_hba.conf` for network access, and optionally opens the port in UFW. Uses `python3-psycopg2` for idempotent Ansible module support.

**`setup_redis.yml`**  
Use when your application needs an in-memory cache or message broker. Installs Redis, configures bind address, max memory with an eviction policy (e.g. `allkeys-lru`), and optional password authentication. Verifies it responds to `PING` before completing.

---

### Docker & Containers

**`install_docker.yml`**  
Use when preparing a VM to run containers. Installs Docker CE from the official Docker apt repository (not the Ubuntu snap), the Compose plugin, and adds specified users to the `docker` group so they can run commands without sudo.

**`deploy_docker_container.yml`**  
Use when you want to run a single Docker container on a VM — specifying image, port mapping, env vars, volume mounts, and restart policy. Waits for the container to reach a running state before completing. Good for self-contained services like monitoring agents, proxies, or single-service apps.

**`deploy_app_with_docker_compose.yml`**  
Use when deploying a multi-service application defined in a `docker-compose.yml`. Upload your own compose file or use the built-in example (nginx + Node.js API + Redis). Handles image pulls, env file upload, and reports container status on completion.

---

### Security & Hardening

**`install_certbot_ssl.yml`**  
Use after deploying nginx when you need a valid TLS certificate for a public-facing domain. Installs Certbot via snap, obtains a Let's Encrypt certificate, and sets up a weekly auto-renewal cron job. Use the `staging: true` flag to test without hitting rate limits.

**`setup_fail2ban.yml`**  
Use on any VM exposed to the public internet to defend against SSH brute-force attacks. Installs fail2ban, configures ban duration and retry limits, and enables the `sshd` jail by default. Additional jails (e.g. nginx) can be enabled via the `f2b_jails` variable.

**`os_hardening_baseline.yml`**  
Use after provisioning any production VM. Applies a security baseline: disables root SSH login, enforces key-only authentication, tunes kernel parameters against network attacks (`tcp_syncookies`, `log_martians`, redirect blocking), installs `auditd`, and disables unnecessary services like `cups` and `avahi-daemon`.

---

### Monitoring & Observability

**`install_prometheus_node_exporter.yml`**  
Use when you want to collect system metrics (CPU, memory, disk, network) from a VM into a Prometheus server. Installs node_exporter as a dedicated system user and systemd service, restricts the metrics port to your Prometheus subnet via UFW, and verifies the `/metrics` endpoint is up.

---

### Operations & Maintenance

**`system_update_reboot.yml`**  
Use during scheduled maintenance windows to fully patch a VM. Runs `apt dist-upgrade`, removes orphaned packages, checks `/var/run/reboot-required`, and reboots only if needed (or if forced). Waits for the host to come back and prints the running kernel version.

**`setup_cron_job.yml`**  
Use when you need a recurring task on a VM — log cleanup, cache warming, health pings, report generation. Define one or more jobs with full cron schedule syntax. Idempotent: re-running updates existing jobs rather than duplicating them.

**`configure_logrotate.yml`**  
Use when an application writes its own log files and you need to prevent disk exhaustion. Writes a logrotate config for each app with compression, retention period, and a post-rotate hook to signal the service. Good follow-up to deploying any long-running service.

**`backup_to_s3.yml`**  
Use for scheduled or on-demand backups of config files, database dumps, or application data to Amazon S3. Compresses each source directory into a dated tarball, uploads to your bucket, and prunes old backups beyond a configurable retention window. Works with IAM instance roles (no keys needed) or explicit credentials via Ansible Vault.

**`mount_nfs_share.yml`**  
Use when VMs need shared storage — for example, mounting a NAS, a shared media volume, or a central config store. Installs NFS client tools, mounts the share, verifies it's accessible, and persists the mount in `/etc/fstab`.

---

### Advanced / Complex

**`setup_wireguard_vpn.yml`**  
Use when you need a fast, lightweight VPN to secure communication between VMs across networks or clouds, or to give remote users access to private infrastructure. Auto-generates server keys if not provided, enables IP forwarding, configures iptables masquerade rules, and writes peer configs. Significantly simpler to operate than OpenVPN.

**`rolling_deploy_zero_downtime.yml`**  
Use when deploying a new application version to a fleet of app servers without taking downtime. Deploys one server at a time (`serial: 1`): drains it from HAProxy, installs the new artifact, waits for the health check to pass, then re-adds it to the load balancer before moving to the next host. If the health check fails on any host, a `rescue:` block automatically rolls back to the previous version and halts the deploy to prevent a bad release propagating further.

---

## AWX Deployment & REST API

AWX exposes every playbook as a REST API endpoint. Use playbooks 31–33 to deploy and configure it on your existing Kubernetes cluster.

### Step 1 — Install the AWX Operator

```bash
ansible-playbook install_awx_operator.yml
# Override operator version if needed:
ansible-playbook install_awx_operator.yml -e "awx_operator_version=2.19.1"
```

Installs `kubectl` and `kustomize` on the control node (if missing), creates the `awx` namespace, and deploys the AWX Operator. Runs against `localhost` — your machine needs `kubectl` access to the cluster.

### Step 2 — Deploy the AWX Instance

```bash
# NodePort (simplest — access via node IP)
ansible-playbook deploy_awx_instance.yml

# Ingress (production — requires an ingress controller)
ansible-playbook deploy_awx_instance.yml \
  -e "expose_method=ingress awx_hostname=awx.example.com"
```

Creates the AWX Custom Resource, waits up to 15 minutes for the full stack (web, task worker, PostgreSQL) to come up, and prints the auto-generated admin password. Credentials are also saved to `/tmp/awx_credentials.txt`.

### Step 3 — Bootstrap AWX with your playbooks

```bash
ansible-playbook bootstrap_awx_config.yml \
  -e "awx_url=http://NODE_IP:30080" \
  -e "awx_password=<password from step 2>" \
  -e "github_repo=https://github.com/cloudfabrix/ansible-playbooks.git" \
  -e "ssh_private_key_path=~/.ssh/id_rsa"
```

Configures AWX entirely via its REST API — creates an organization, SSH machine credential, Git-backed project, inventory (matching `inventory.ini`), and a **job template for every one of the 30 playbooks**. Variables and target host limits can be overridden per-launch.

---

### REST API Usage

Once bootstrapped, every playbook is launchable via a simple HTTP call.

#### List all job templates
```bash
curl -s -u admin:<password> http://NODE_IP:30080/api/v2/job_templates/ | jq '.results[] | {id, name}'
```

#### Launch a playbook
```bash
# Launch "Install Docker" on all hosts
curl -s -u admin:<password> \
  -X POST http://NODE_IP:30080/api/v2/job_templates/<id>/launch/ \
  -H "Content-Type: application/json" \
  -d '{}'

# Launch with host limit and variable override
curl -s -u admin:<password> \
  -X POST http://NODE_IP:30080/api/v2/job_templates/<id>/launch/ \
  -H "Content-Type: application/json" \
  -d '{
    "limit": "web_servers",
    "extra_vars": {"node_major_version": "22"}
  }'
```

#### Poll job status
```bash
# Returns: pending, waiting, running, successful, failed
curl -s -u admin:<password> \
  http://NODE_IP:30080/api/v2/jobs/<job_id>/ | jq '.status'
```

#### Stream job output
```bash
curl -s -u admin:<password> \
  "http://NODE_IP:30080/api/v2/jobs/<job_id>/stdout/?format=txt"
```

#### Token-based auth (better for automation)
```bash
# Create a token
TOKEN=$(curl -s -u admin:<password> \
  -X POST http://NODE_IP:30080/api/v2/tokens/ \
  -H "Content-Type: application/json" \
  -d '{"description":"CI token","application":null,"scope":"write"}' \
  | jq -r '.token')

# Use the token
curl -s -H "Authorization: Bearer $TOKEN" \
  -X POST http://NODE_IP:30080/api/v2/job_templates/<id>/launch/ \
  -H "Content-Type: application/json" \
  -d '{"limit": "app1"}'
```

#### AWX Playbooks Reference

| File | When to use |
|------|-------------|
| `install_awx_operator.yml` | First-time AWX setup — installs the AWX Operator into your K8s cluster |
| `deploy_awx_instance.yml` | Creates the AWX instance, waits for readiness, retrieves admin password |
| `bootstrap_awx_config.yml` | Configures AWX via API: org, credentials, project, inventory, and all 30 job templates |

---

## Tips

**Dry-run before applying:**
```bash
ansible-playbook os_hardening_baseline.yml -i inventory.ini --check --diff
```

**Limit to specific hosts:**
```bash
ansible-playbook system_update_reboot.yml -i inventory.ini --limit web_servers
```

**Protect secrets with Vault:**
```bash
ansible-vault encrypt_string 'mysecretpassword' --name 'db_password'
ansible-playbook setup_postgresql.yml -i inventory.ini --ask-vault-pass
```

## Tested On

- Ubuntu 22.04 LTS (Jammy)
- Ubuntu 24.04 LTS (Noble)
- Debian 12 (Bookworm)
