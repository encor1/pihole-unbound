# Ubuntu: Port 53 für Pi-hole freigeben

## 1. Stub-Listener von systemd-resolved deaktivieren

```bash
sudo mkdir -p /etc/systemd/resolved.conf.d

sudo tee /etc/systemd/resolved.conf.d/no-stub.conf >/dev/null <<'EOF'
[Resolve]
DNSStubListener=no
EOF
```

---

## 2. `/etc/resolv.conf` korrigieren

```bash
sudo rm -f /etc/resolv.conf
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

---

## 3. systemd-resolved neu starten

```bash
sudo systemctl restart systemd-resolved
```

---

## 4. Prüfen, ob Port 53 frei ist

```bash
sudo ss -lntup | grep ':53 '
```

Es sollte **kein Prozess** mehr auf Port 53 lauschen.

---

## 5. Docker Compose starten

```bash
docker compose up -d
```

---

## 6. Prüfen, ob Docker nun Port 53 belegt

```bash
sudo ss -lntup | grep ':53 '
```

Jetzt solltest du `docker-proxy` bzw. Docker auf Port 53 sehen.
