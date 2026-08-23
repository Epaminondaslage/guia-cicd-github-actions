# Apêndice D — Linux para operação de CI/CD

## Sistema

| Comando | Descrição |
|---|---|
| `uname -a` | Mostra kernel, arquitetura e nome do host |
| `cat /etc/os-release` | Exibe a distribuição e versão do sistema operacional |
| `hostnamectl` | Mostra hostname e informações gerais do sistema |
| `uptime` | Mostra há quanto tempo o sistema está no ar e a carga média |

## CPU e memória

| Comando | Descrição |
|---|---|
| `lscpu` | Mostra informações da CPU |
| `free -h` | Mostra uso de memória RAM e swap em formato legível |
| `top` | Monitor interativo de processos e consumo de recursos |
| `htop` | Versão interativa e colorida do `top` |

## Disco

| Comando | Descrição |
|---|---|
| `df -h` | Mostra espaço em disco por partição, em formato legível |
| `df -i` | Mostra uso de inodes por partição |
| `du -sh *` | Mostra o tamanho de cada item do diretório atual |
| `lsblk` | Lista dispositivos de bloco e suas partições |

## Processos

| Comando | Descrição |
|---|---|
| `ps aux` | Lista todos os processos em execução |
| `pgrep -af PROCESSO` | Busca processos pelo nome, mostrando PID e linha de comando |
| `kill PID` | Envia sinal de término a um processo |
| `kill -9 PID` | Força o encerramento de um processo |
| `pkill -f PROCESSO` | Encerra processos cuja linha de comando casa com o padrão |

## Rede

| Comando | Descrição |
|---|---|
| `ip addr` | Mostra interfaces de rede e endereços IP |
| `ip route` | Mostra a tabela de rotas |
| `ss -lntp` | Lista portas TCP em escuta e os processos associados |
| `ping HOST` | Testa conectividade com um host |
| `curl -I https://github.com` | Faz uma requisição HTTP e mostra apenas os cabeçalhos de resposta |

## DNS

| Comando | Descrição |
|---|---|
| `getent hosts HOST` | Resolve um host usando o mecanismo de resolução do sistema |
| `resolvectl status` | Mostra o estado da resolução de DNS via systemd-resolved |

## systemd

| Comando | Descrição |
|---|---|
| `systemctl status SERVICO` | Mostra o estado atual de um serviço |
| `sudo systemctl start SERVICO` | Inicia um serviço |
| `sudo systemctl stop SERVICO` | Para um serviço |
| `sudo systemctl restart SERVICO` | Reinicia um serviço |
| `sudo systemctl enable SERVICO` | Habilita o início automático do serviço no boot |
| `sudo systemctl enable --now SERVICO` | Habilita o serviço no boot e o inicia imediatamente |
| `sudo systemctl disable SERVICO` | Desabilita o início automático do serviço no boot |
| `sudo systemctl daemon-reload` | Recarrega as definições de unidades do systemd |
| `systemctl is-active SERVICO` | Mostra se o serviço está ativo |
| `systemctl list-units --failed` | Lista unidades que falharam |

Runner self-hosted do GitHub Actions: o instalador (`./svc.sh install` / `./svc.sh install USUARIO`) cria um serviço `actions.runner.*`; use `sudo ./svc.sh start`, `sudo ./svc.sh stop` e `sudo ./svc.sh status` (wrappers sobre `systemctl`), ou `systemctl status actions.runner.*` diretamente.

## Logs

| Comando | Descrição |
|---|---|
| `journalctl -u SERVICO` | Mostra o log de um serviço |
| `journalctl -u SERVICO -f` | Acompanha o log de um serviço em tempo real |
| `journalctl -u SERVICO --since "1 hour ago"` | Mostra o log de um serviço a partir de um horário |
| `journalctl -u SERVICO -n 200 --no-pager` | Mostra as últimas 200 linhas do log, sem paginação |
| `journalctl -p err` | Mostra apenas entradas de nível erro ou mais grave |
| `journalctl -p err -b` | Mostra entradas de nível erro do boot atual |
| `journalctl --disk-usage` | Mostra o espaço em disco ocupado pelos logs |
| `sudo journalctl --vacuum-time=7d` | Remove logs com mais de 7 dias |

## Usuários e grupos

| Comando | Descrição |
|---|---|
| `id` | Mostra UID, GID e grupos do usuário atual |
| `groups` | Lista os grupos do usuário atual |
| `sudo adduser usuario` | Cria um novo usuário |
| `sudo usermod -aG docker usuario` | Adiciona um usuário ao grupo `docker` |

## Permissões

| Comando | Descrição |
|---|---|
| `ls -la` | Lista arquivos com permissões, dono e grupo |
| `chmod +x script.sh` | Torna um arquivo executável |
| `chmod 750 diretorio/` | Define permissões restritas em um diretório |
| `chown usuario:grupo arquivo` | Altera dono e grupo de um arquivo |
| `chown -R runner:runner /home/runner/_work` | Altera dono e grupo recursivamente |

Evite `chmod 777` como correção genérica.

## SSH

| Comando | Descrição |
|---|---|
| `ssh usuario@host` | Conecta a um host remoto via SSH |
| `ssh-keygen -t ed25519` | Gera um novo par de chaves SSH |
| `ssh-keyscan HOST` | Obtém a chave pública de host de um servidor remoto |

## Pacotes Ubuntu

| Comando | Descrição |
|---|---|
| `sudo apt update` | Atualiza a lista de pacotes disponíveis |
| `sudo apt upgrade` | Atualiza os pacotes instalados |
| `apt search PACOTE` | Busca um pacote pelo nome |
| `apt policy PACOTE` | Mostra versões disponíveis e instalada de um pacote |

## Arquivos

| Comando | Descrição |
|---|---|
| `cp -r origem/ destino/` | Copia um diretório recursivamente |
| `mv origem destino` | Move ou renomeia um arquivo ou diretório |
| `rm -rf diretorio/` | Remove um diretório e seu conteúdo, sem confirmação |
| `mkdir -p caminho/subpasta` | Cria um diretório e seus pais, se necessário |
| `find . -name "*.log" -mtime +7 -delete` | Remove arquivos `.log` com mais de 7 dias |
| `grep -R "erro" /var/log/` | Busca recursivamente pelo termo "erro" em `/var/log/` |

## Compressão

| Comando | Descrição |
|---|---|
| `tar -czf arquivo.tar.gz diretorio/` | Compacta um diretório em um arquivo `.tar.gz` |
| `tar -xzf arquivo.tar.gz` | Extrai um arquivo `.tar.gz` |

## Cron

| Comando | Descrição |
|---|---|
| `crontab -e` | Edita a crontab do usuário atual |
| `crontab -l` | Lista as entradas da crontab do usuário atual |
| `crontab -u usuario -l` | Lista a crontab de um usuário específico |
| `0 3 * * * find /home/runner/_work -mtime +3 -delete` | Exemplo: limpeza noturna do workdir do runner |

## Firewall UFW

| Comando | Descrição |
|---|---|
| `sudo ufw status` | Mostra o estado do firewall UFW e as regras ativas |

Altere regras de firewall apenas com plano para evitar perda de acesso remoto.

## Fontes

- [systemctl(1) — Linux man page](https://man7.org/linux/man-pages/man1/systemctl.1.html) — `enable --now`, `daemon-reload` e demais subcomandos de gerenciamento de serviços.
- [journalctl(1) — Linux man page](https://man7.org/linux/man-pages/man1/journalctl.1.html) — `-u`, `--since`, `-p err`, `--disk-usage` e `--vacuum-time`.
- [Configuring the self-hosted runner application as a service](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/configure-the-application) — `./svc.sh install/start/stop/status` como wrapper de systemd para o runner self-hosted.

**Fim do Apêndice D**
