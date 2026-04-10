# Docker Rootless Setup (Linux Mint) — Full Guide

## Overview
This document summarizes a full setup of Docker Engine in rootless mode,
including troubleshooting and secure configuration.

---

## 1. Docker vs Docker Desktop

- Docker Desktop:
  - Uses VM (KVM/QEMU)
  - Requires GNOME terminal dependency
  - Heavier

- Docker Engine:
  - Native Linux
  - No VM
  - Recommended

---

## 2. Install Docker Engine

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg
````

Add repo and install Docker CE.

---

## 3. Non-root vs Rootless

### docker group (NOT fully secure)

```bash
sudo usermod -aG docker $USER
```

* No sudo needed
* Still root daemon

---

### Rootless Docker (secure)

Install:

```bash
sudo apt install uidmap docker-ce-rootless-extras
```

Setup:

```bash
dockerd-rootless-setuptool.sh install
systemctl --user start docker
sudo loginctl enable-linger $USER
```

---

## 4. Zsh configuration

Add to `~/.zshrc`:

```bash
export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock
```

---

## 5. Verify

```bash
docker info
```

Expected:

```
Security Options:
  rootless
  seccomp
```

---

## 6. Fix credential error

Edit:

```bash
nano ~/.docker/config.json
```

Remove:

```json
"credsStore": "desktop"
```

---

## 7. Portainer (rootless fix)

```bash
docker volume create portainer_data

docker run -d \
  -p 127.0.0.1:9000:9000 \
  --name portainer \
  --restart=unless-stopped \
  -v /run/user/$(id -u)/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

---

## 8. Security Best Practices

* Avoid:

  * `--privileged`
  * `-v /:/host`
* Use:

  * `--read-only`
  * `--security-opt no-new-privileges`
* Bind to localhost:

  ```bash
  -p 127.0.0.1:PORT:PORT
  ```

---

## 9. Key Concepts

* Rootless Docker = no root daemon
* Uses user namespaces
* Safer than standard Docker
* Not full VM isolation

---

## 10. Notes

* Data stored in:

  ```
  ~/.local/share/docker
  ```
* Ports <1024 not available
* Some containers may need tweaks

---

## Conclusion

This setup provides:

* Strong isolation
* No system risk
* Ideal for local dev and OpenClaw experiments
