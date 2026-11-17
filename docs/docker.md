# Docker

Docker Engine runs on the `server-production` VM. My websites and applications will run in Docker containers.

## Why

I run everything in containers, also Nginx:

- **Isolation:** every app has its own libraries and versions, so they don't conflict. The VM stays clean, only Docker is installed on it.
- **Same image everywhere:** the image I build in GitHub Actions is the same one that runs on the server. No "it works on my machine".
- **Easy deploy and rollback:** deploy is pull the new image and run it. Rollback is run the previous tag.
- **Setup in files:** the Dockerfile and Docker Compose file describe everything. If the server breaks, I can rebuild it fast.
- **Security:** only Nginx is open to the outside. The other containers talk to Nginx over a private Docker network and don't open ports on the VM.
- **Easy to try and remove:** I can test a new tool in a container and delete it without leaving files on the system.
- **Real-world skill:** most companies run their apps in containers, so I practice the same way here.

## Install

I followed only the official guide: [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

1. Removed conflicting packages (none were installed):
   ```bash
   sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc docker-buildx podman-docker containerd runc | cut -f1)
   ```
2. Added Docker's GPG key and `apt` repository:
   ```bash
   sudo apt update
   sudo apt install ca-certificates curl          # certificates for HTTPS, and curl to download the key
   sudo install -m 0755 -d /etc/apt/keyrings      # create the official folder for GPG keys
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc   # download Docker's official public key
   sudo chmod a+r /etc/apt/keyrings/docker.asc    # let everyone read the key (apt reads it as a non-root user)
   ```
   GPG (GNU Privacy Guard) is a tool to sign and check files. Docker uses it to sign its packages, and `apt` uses the key to check that the packages really come from Docker.

   Then I wrote `/etc/apt/sources.list.d/docker.sources` with `nano`:
   ```text
   Types: deb
   URIs: https://download.docker.com/linux/ubuntu
   Suites: resolute
   Components: stable
   Architectures: amd64
   Signed-By: /etc/apt/keyrings/docker.asc
   ```
   `Suites` is the Ubuntu codename (`resolute` = 26.04). I can check it with `. /etc/os-release && echo $VERSION_CODENAME`.

   ```bash
   sudo apt update
   ```

3. Installed Docker:
   ```bash
   sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

4. Checked that it works:
   ```bash
   sudo systemctl status docker   # active (running)
   sudo docker run hello-world    # prints "Hello from Docker!"
   ```

Docker starts on boot (`docker.service` is enabled by the install).

## Containers

All my containers will run in one Docker network, so they can connect to each other.
