# Disk is full: finding what ate the space

## 1. Which filesystem?

```bash
df -h
df -i        # also check inodes: millions of tiny files can fill a disk too
```

## 2. What's big?

```bash
sudo du -xh / --max-depth=1 2>/dev/null | sort -h | tail
sudo du -xh /var --max-depth=2 2>/dev/null | sort -h | tail
```

`-x` keeps `du` on one filesystem so it doesn't wander into mounts.

## 3. Usual suspects

| Where | Cleanup |
|---|---|
| `/var/log`, journald | `sudo journalctl --vacuum-size=500M`; check logrotate config |
| Docker | `docker system df`, then `docker system prune` (add `-a` / `--volumes` carefully) |
| `/tmp`, core dumps | remove old files, check `/var/lib/systemd/coredump` |
| Package cache | `sudo apt clean` |

## 4. "df says full, du says not"

A process still holds a **deleted** file open, so the space isn't released:

```bash
sudo lsof +L1
```

Restart that process (often a service writing logs) to free the space.

## Prevent it

- Log rotation with size limits
- Docker log driver limits: `"log-opts": {"max-size": "10m", "max-file": "3"}`
- An alert on disk usage > 80% (node_exporter + Prometheus rule)
