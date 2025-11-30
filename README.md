# WireGuard Docker (Userspace) Lab

This project spins up a 3-node WireGuard network using **Docker + Rocky Linux 9 + wireguard-go (userspace)**:

- `wg-router` – WireGuard hub (10.0.0.1)
- `wg-client1` – peer (10.0.0.2)
- `wg-client2` – peer (10.0.0.3)

All WireGuard runs in **userspace mode** (`wireguard-go`), so it works on macOS / Docker Desktop without kernel modules.

## Requirements

- Docker
- docker-compose
- Ansible
- community.docker collection:

```bash
ansible-galaxy collection install community.docker
```

## Layout

- `Dockerfile.r9` – Rocky Linux 9 image with Python, WireGuard tools, wireguard-go
- `docker-compose.yml` – 3 containers on `172.28.0.0/16`
- `ansible/` – playbooks to configure WireGuard and bring up interfaces

## Usage

### 1) Build and start containers

```bash
docker-compose build
docker-compose up -d
```

### 2) Run Ansible

```bash
cd ansible
ansible-playbook site.yml
```

This will:

1. Generate WireGuard keypairs on all three containers.
2. Create `/etc/wireguard/wg0.conf` on each (router + peers).
3. Bring up `wg0` using userspace `wireguard-go` via `wg-quick`.

## Testing

From your host:

```bash
docker exec -it wg-client1 bash
ping 10.0.0.3
```

And:

```bash
docker exec -it wg-client2 bash
ping 10.0.0.2
```

If you see replies, your hub-and-spoke WireGuard network is working.

## Notes

- WireGuard runs in **userspace mode**, no kernel module or `modprobe` is used.
- The router routes traffic between the two peers over the WireGuard network.
