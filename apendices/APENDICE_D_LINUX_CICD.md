# Apêndice D — Linux para Operação de CI/CD

## Sistema
```bash
uname -a
cat /etc/os-release
hostnamectl
uptime
```

## CPU e memória
```bash
lscpu
free -h
top
htop
```

## Disco
```bash
df -h
df -i
du -sh *
lsblk
```

## Processos
```bash
ps aux
pgrep -af PROCESSO
kill PID
kill -9 PID
pkill -f PROCESSO
```

## Rede
```bash
ip addr
ip route
ss -lntp
ping HOST
curl -I https://github.com
```

## DNS
```bash
getent hosts HOST
resolvectl status
```

## systemd
```bash
systemctl status SERVICO
sudo systemctl start SERVICO
sudo systemctl stop SERVICO
sudo systemctl restart SERVICO
sudo systemctl enable SERVICO
sudo systemctl enable --now SERVICO
sudo systemctl disable SERVICO
sudo systemctl daemon-reload
systemctl is-active SERVICO
systemctl list-units --failed
```

Runner self-hosted do GitHub Actions: o instalador (`./svc.sh install` / `./svc.sh install USUARIO`) cria um serviço `actions.runner.*`; use `sudo ./svc.sh start`, `sudo ./svc.sh stop` e `sudo ./svc.sh status` (wrappers sobre `systemctl`), ou `systemctl status actions.runner.*` diretamente.

## Logs
```bash
journalctl -u SERVICO
journalctl -u SERVICO -f
journalctl -u SERVICO --since "1 hour ago"
journalctl -u SERVICO -n 200 --no-pager
journalctl -p err
journalctl -p err -b
journalctl --disk-usage
sudo journalctl --vacuum-time=7d
```

## Usuários e grupos
```bash
id
groups
sudo adduser usuario
sudo usermod -aG docker usuario
```

## Permissões
```bash
ls -la
chmod +x script.sh
chmod 750 diretorio/
chown usuario:grupo arquivo
chown -R runner:runner /home/runner/_work
```

Evite `chmod 777` como correção genérica.

## SSH
```bash
ssh usuario@host
ssh-keygen -t ed25519
ssh-keyscan HOST
```

## Pacotes Ubuntu
```bash
sudo apt update
sudo apt upgrade
apt search PACOTE
apt policy PACOTE
```

## Arquivos
```bash
cp -r origem/ destino/
mv origem destino
rm -rf diretorio/
mkdir -p caminho/subpasta
find . -name "*.log" -mtime +7 -delete
grep -R "erro" /var/log/
```

## Compressão
```bash
tar -czf arquivo.tar.gz diretorio/
tar -xzf arquivo.tar.gz
```

## Cron
```bash
crontab -e
crontab -l
crontab -u usuario -l
# ex.: limpeza noturna do workdir do runner
# 0 3 * * * find /home/runner/_work -mtime +3 -delete
```

## Firewall UFW
```bash
sudo ufw status
```

Altere regras de firewall apenas com plano para evitar perda de acesso remoto.

**Fim do Apêndice D**
