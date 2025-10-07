# Day 45: Resolve Dockerfile Issues

## 🛠️ Task

While working on it she ran into issues in which the docker build is failing and displaying errors. Look into the issue and fix it to build an image as per details mentioned below:

- The Dockerfile is placed on App Server 2 under `/opt/docker` directory.
- Fix the issues with this file and make sure it is able to build the image.
- Do not change base image, any other valid configuration within Dockerfile, or any of the data been used — for example, `index.html`.

---

## ✅ Corrected Dockerfile

```Dockerfile
FROM httpd:2.4.43

# Modify Apache config to listen on port 8080
RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf

# Uncomment SSL-related modules and config includes
RUN sed -i '/LoadModule\\ ssl_module modules\/mod_ssl.so/s/^#//g' /usr/local/apache2/conf/httpd.conf \
    && sed -i '/LoadModule\\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' /usr/local/apache2/conf/httpd.conf \
    && sed -i '/Include\\ conf\/extra\/httpd-ssl.conf/s/^#//g' /usr/local/apache2/conf/httpd.conf

# Copy SSL certificates
COPY certs/server.crt /usr/local/apache2/conf/server.crt
COPY certs/server.key /usr/local/apache2/conf/server.key

# Copy website content
COPY html/index.html /usr/local/apache2/htdocs/
```

---

## 🔍 Key Fixes

| Issue                         | Fix                                             |
| ----------------------------- | ----------------------------------------------- |
| `IMAGE` keyword is invalid    | Replaced with `FROM`                            |
| `ADD` used for shell commands | Replaced with `RUN`                             |
| Wrong path in some `sed` cmds | Unified to `/usr/local/apache2/conf/httpd.conf` |
| Preserved all data and base   | ✅                                              |

---

## 🚀 Build and Test

```bash
cd /opt/docker
docker build -t nautilus-httpd .
docker run -d -p 8080:8080 nautilus-httpd
curl http://localhost:8080
```
