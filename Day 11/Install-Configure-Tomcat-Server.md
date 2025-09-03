# Install & Configure Apache Tomcat on App Server 2 (stapp02)

This document contains step-by-step commands to:

- Install Tomcat on App Server 2 (hostname: `stapp02`).
- Configure Tomcat to listen on port `5000`.
- Deploy the `ROOT.war` located on the Jump host at `/tmp/ROOT.war` so the app is available directly at `http://stapp02:5000`.

Assumptions:

- You have SSH access to `stapp02` and the Jump host.
- The Jump host path `/tmp/ROOT.war` contains a valid WAR file.
- `bash` is available on the managed machine (commands below are for `bash`).

Checklist (requirements):

- [x] Install Tomcat on App Server 2 (`stapp02`).
- [x] Configure Tomcat to run on port `5000`.
- [x] Copy & deploy `ROOT.war` from Jump host `/tmp` to Tomcat's `webapps` directory.
- [x] Verify the application responds at `http://stapp02:5000`.

If anything below doesn't match your distro (Debian/Ubuntu vs RHEL/CentOS), use the alternative commands shown.

## 1) Install Java & create tomcat user

On `stapp02` (example for Debian/Ubuntu):

```bash
sudo apt update
sudo apt install -y openjdk-11-jdk wget tar
```

Or on RHEL/CentOS:

```bash
sudo yum install -y java-11-openjdk-devel wget tar
```

Create a dedicated user and group for Tomcat:

```bash
sudo groupadd tomcat || true
sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat || true
```

## 2) Install Tomcat

Pick a Tomcat 9+ binary. Replace `9.0.75` below with the desired stable version if needed.

```bash
cd /tmp
TOMCAT_VER=9.0.75
wget https://downloads.apache.org/tomcat/tomcat-9/v${TOMCAT_VER}/bin/apache-tomcat-${TOMCAT_VER}.tar.gz
sudo mkdir -p /opt/tomcat
sudo tar xzf apache-tomcat-${TOMCAT_VER}.tar.gz -C /opt/tomcat --strip-components=1
sudo chown -R tomcat:tomcat /opt/tomcat
sudo chmod -R u+rX /opt/tomcat
```

## 3) Configure Tomcat to listen on port 5000

Edit the connector in `/opt/tomcat/conf/server.xml` and change the HTTP connector port from `8080` to `5000`.

Quick non-interactive change (will modify the first Connector that contains port="8080"):

```bash
sudo sed -i '0, /Connector port="8080"/s//Connector port="5000"/' /opt/tomcat/conf/server.xml
```

Verify the file contains `port="5000"` for the HTTP connector:

```bash
grep -n "Connector .*port=\"5000\"" -n /opt/tomcat/conf/server.xml || sudo sed -n '1,160p' /opt/tomcat/conf/server.xml
```

Notes:

- If you prefer to edit the file manually:

```bash
sudo vi /opt/tomcat/conf/server.xml
# find <Connector port="8080" ... and change to port="5000"
```

## 4) Deploy `ROOT.war` from the jump host

There are two simple options. Use whichever matches your environment.

Option A — copy from the Jump host to `stapp02` (run on the Jump host):

```bash
# from the Jump host
scp /tmp/ROOT.war <your-ssh-user>@stapp02:/tmp/

# now SSH to stapp02 and move it into Tomcat webapps
ssh <your-ssh-user>@stapp02 'sudo mv /tmp/ROOT.war /opt/tomcat/webapps/ && sudo chown tomcat:tomcat /opt/tomcat/webapps/ROOT.war'
```

Option B — if you are already on `stapp02` and can reach the jump host's filesystem (or mounted location), copy directly:

```bash
sudo cp /tmp/ROOT.war /opt/tomcat/webapps/
sudo chown tomcat:tomcat /opt/tomcat/webapps/ROOT.war
```

Tomcat will automatically extract `ROOT.war` to `/opt/tomcat/webapps/ROOT` after (re)start.

## 5) Create a systemd service (recommended)

Create `/etc/systemd/system/tomcat.service` so Tomcat runs at boot and can be managed via `systemctl`.

```bash
sudo tee /etc/systemd/system/tomcat.service > /dev/null <<'EOF'
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking
User=tomcat
Group=tomcat
Environment=JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now tomcat
```

If your Java path is different, update `JAVA_HOME` accordingly (use `readlink -f /usr/bin/java` to locate it).

Alternatively start manually (quick test):

```bash
sudo -u tomcat /opt/tomcat/bin/startup.sh
```

## 6) Open firewall port 5000 (if firewall is enabled)

On Debian/Ubuntu with ufw:

```bash
sudo ufw allow 5000/tcp
sudo ufw reload
```

On RHEL/CentOS with firewalld:

```bash
sudo firewall-cmd --add-port=5000/tcp --permanent
sudo firewall-cmd --reload
```

## 7) Verify the deployment and endpoint

Check Tomcat logs while it deploys:

```bash
sudo tail -n 200 /opt/tomcat/logs/catalina.out
```

Then test the base URL from a machine that can resolve `stapp02`:

```bash
curl -I http://stapp02:5000
curl http://stapp02:5000
```

Expected quick verification output for the header test:

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
```

If the response is not 200, inspect `/opt/tomcat/logs` (catalina.out, localhost._.log, host-manager._.log) and ensure the `ROOT` webapp exploded under `/opt/tomcat/webapps/ROOT`.

## Troubleshooting checklist

- Permission denied deploying WAR: ensure `tomcat:tomcat` owns `/opt/tomcat` and webapps.
- Port 5000 already in use: `sudo ss -tlnp | grep 5000` or `sudo netstat -tuln | grep 5000`.
- SELinux blocking (RHEL/CentOS): check `sestatus` and audit logs; allow httpd network or set proper file contexts if required.

## Small extras / Automation suggestions

- If you want this done repeatedly across servers, I can produce an Ansible playbook that:
  - Installs Java, downloads & unpacks Tomcat,
  - Configures connector port to `5000`,
  - Copies `ROOT.war` from the jump host, and
  - Ensures the systemd service is enabled.

---

End of instructions. Follow the commands in the order shown and replace `<your-ssh-user>` with your real SSH user.
