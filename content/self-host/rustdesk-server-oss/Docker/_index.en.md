---
title: Docker
weight: 7
---
### Recommended Tutorial:
- [Building Your Own Remote Desktop Solution: RustDesk Self-Hosted on Cloud with Docker (Hetzner)](https://www.linkedin.com/pulse/building-your-own-remote-desktop-solution-rustdesk-cloud-montinaro-bv94f)

### Installing your own server with Docker

#### Requirements:
* **Docker installed**:
  - [Docker Installation guide](https://docs.docker.com/engine/install/)

**Firewall Configuration Requirements**
Please ensure that the following ports are opened in your firewall:

* **Port 21114 (TCP)**: Required for web console functionality, only available in Pro versions.
* **Port 21115 (TCP)**: Used for NAT type testing.
* **Port 21116 (TCP and UDP)**:
  - Both TCP and UDP must be enabled;
    - Port 21116/UDP is used for ID registration and heartbeat services.
    - Port 21116/TCP is used for TCP hole punching and connection services.
* **Port 21118 (TCP)**: Required to support web clients.
* **Port 21117 (TCP)**: Used for Relay services.
* **Port 21119 (TCP)**: Supports web clients, but can be disabled if not needed.
  - **Note**: If you do not require web client support, ports 21118 and 21119 can be closed to improve security.

#### Pulling and Running the Server

To get started with a RustDesk server, first pull the image using the following command:

```sh
sudo docker image pull rustdesk/rustdesk-server
```

Next, run the containers in detached mode.  
```sh
sudo docker run --name hbbs -v ./data:/root -td --net=host --restart unless-stopped rustdesk/rustdesk-server hbbs
sudo docker run --name hbbr -v ./data:/root -td --net=host --restart unless-stopped rustdesk/rustdesk-server hbbr
```
> If you're using Windows, be sure to omit the `sudo` command when running these containers.

#### Troubleshooting Tips
> When using the `--net=host` option with Docker on Linux, be aware that:
  - This option is only supported on Linux platforms.
  - When enabled, `hbbs` and `hbbr` will see the real incoming IP address instead of the container's IP address.
  - If you experience issues or cannot use this option, please remove it.

> Recent logs can be viewed with:
```sh
docker logs hbbs --tail=10
```
> To troubleshoot issues with Docker containers using the -td option, you can switch to interactive mode by running:
`hbbs -it` or `hbbr -it`

### Docker Compose examples  
#### Requirements:  
  - Docker Compose installed
    - [Installation guide](https://docs.docker.com/compose/install/)

```yaml
services:
  hbbs:
    container_name: hbbs
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21118:21118
    image: rustdesk/rustdesk-server:latest
    command: hbbs -r your.relay.domain:21117 
    volumes:
      - ./hbbs:/root
    network_mode: host
    depends_on:
      - hbbr
    restart: unless-stopped
  hbbr:
    container_name: hbbr
    ports:
      - 21117:21117
      - 21119:21119
    image: rustdesk/rustdesk-server:latest
    command: hbbr 
    volumes:
      - ./hbbr:/root
    network_mode: host
    restart: unless-stopped
networks: {}
```
> The first run of hbbs generates a public key used to [configure the client](https://rustdesk.com/docs/en/self-host/client-configuration/#set-key) and connect to the relay server. It can be found in the log files or in the id_ed25519.pub file in your working directory.
```
docker logs hbbs --tail=15
```
```
Key: wPGbKi9hkSdPkAA1hY1XyN1LtXHIDoNotUseThisKey=
```
> To incude the key in the docker compose file use this syntax:
```
    command: hbbs -r your.relay.domain:21117 -k wPGbKi9hkSdPkAA1hY1XyN1LtXHIDoNotUseThisKey=
    command: hbbr -k wPGbKi9hkSdPkAA1hY1XyN1LtXHIDoNotUseThisKey=
```
 > To modify configurations settings, you can use the environment variable feature in the  `docker-compose.yml` file.
     `environment:`
      `- ALWAYS_USE_RELAY=Y`

```yaml
services:
  hbbs:
    container_name: hbbs -k wPGbKipDnD7QNb0vTTcKZ9hkSdPkAA1hY1XyN1LtXHI=
    image: rustdesk/rustdesk-server:latest
    environment:
      - ALWAYS_USE_RELAY=Y
    command: hbbs -r your.relay.domain:21117 -k wPGbKi9hkSdPkAA1hY1XyN1LtXHIDoNotUseThisKey=
    volumes:
      - ./data:/root
    network_mode: "host"

    depends_on:
      - hbbr
    restart: unless-stopped

  hbbr:
    container_name: hbbr
    image: rustdesk/rustdesk-server:latest
    command: hbbr -k wPGbKi9hkSdPkAA1hY1XyN1LtXHIDoNotUseThisKey=
    volumes:
      - ./data:/root
    network_mode: "host"
    restart: unless-stopped
```
## Install with Podman
  Requirements:  
  1. Podman installed
     - [Podman Installation guide](https://podman.io/docs/installation)
### Podman Quadlet examples

If you would like to run the containers with Podman as a systemd service you can use these sample Podman Quadlet configurations:

```ini
[Container]
AutoUpdate=registry
Image=ghcr.io/rustdesk/rustdesk-server:latest
Exec=hbbs
Volume=/path/to/rustdesk-server/data:/root
Network=host

[Service]
Restart=always

[Install]
WantedBy=default.target
```

or

```ini
[Container]
AutoUpdate=registry
Image=ghcr.io/rustdesk/rustdesk-server:latest
Exec=hbbr
Volume=/path/to/rustdesk-server/data:/root
Network=host

[Service]
Restart=always

[Install]
WantedBy=default.target
```
