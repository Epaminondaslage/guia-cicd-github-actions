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
```

## Logs
```bash
journalctl -u SERVICO
journalctl -u SERVICO -f
journalctl -p err
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
chmod
chown
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
cp
mv
rm
mkdir -p
find
grep
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
```

## Firewall UFW
```bash
sudo ufw status
```

Altere regras de firewall apenas com plano para evitar perda de acesso remoto.

**Fim do Apêndice D**
