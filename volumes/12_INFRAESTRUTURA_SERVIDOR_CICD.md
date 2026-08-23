# Volume 12 — Infraestrutura do Servidor CI/CD

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 12_INFRAESTRUTURA_SERVIDOR_CICD.md  
**Versão:** 0.2.0

---

## 1. Objetivo

Definir uma infraestrutura estável, reconstruível e segura para hospedar self-hosted runners e serviços auxiliares de CI/CD.

Arquitetura inicial:

```text
Internet
   |
GitHub
   |
HTTPS
   |
Runner VM
Ubuntu Server
   |
   +-- GitHub Actions Runner
   +-- Docker
   +-- ferramentas de CI
```

Separadamente:

```text
DEV server
PROD server
```

---

## 2. Físico ou virtual

Um runner pode operar em:

```text
máquina física
VM
container (Docker)
LXC/LXD
cloud VM
Kubernetes (via Actions Runner Controller)
```

Para começar, VM é uma escolha forte por:

- isolamento;
- snapshots;
- facilidade de reconstrução;
- limites de recursos.

### 2.1 Trade-offs de isolamento (VM vs container vs LXC)

O nível de isolamento necessário depende do que o runner vai executar:

```text
VM (KVM/QEMU)     -> isolamento forte: kernel próprio, hypervisor separa hardware
LXC/container      -> isolamento fraco/médio: kernel compartilhado com o host
Docker-in-Docker   -> risco adicional: runner dentro de container que sobe outros containers
```

Pontos críticos:

- container e LXC compartilham o kernel do host. Uma fuga de container (container breakout) compromete o host inteiro, não apenas o job;
- VM isola o kernel, reduzindo o raio de explosão de uma falha ou exploração no job;
- self-hosted runners **nunca** devem rodar workflows de PRs de forks externos sem revisão manual (`pull_request_target` e execução automática em fork são vetores clássicos de exec arbitrário). Trate isso como controle de segurança, não como detalhe de infraestrutura;
- se o repositório aceita contribuições externas, prefira runners efêmeros em VM descartável por job, ou GitHub-hosted runners para PRs de fork e self-hosted apenas para o repositório interno/branches protegidas;
- LXC privilegiado (`privileged: true`) para dar acesso ao Docker do host anula praticamente todo o isolamento: evite. Se precisar de Docker dentro do LXC, prefira `keyctl`/nesting controlado ou mova a carga para VM.

Resumindo o trade-off: LXC/container ganham em densidade e velocidade de provisionamento; VM ganha em segurança quando o conteúdo do job não é 100% confiável.

---

## 3. Não misturar funções críticas

Evite:

```text
mesmo host:
PROD + banco PROD + runner CI
```

Prefira separação lógica ou física.

---

## 4. Dimensionamento

Referências de partida por tipo de carga (ajuste com métricas reais do seu pipeline):

| Tipo de carga | Recursos | Observação |
|---|---|---|
| Lint/testes unitários simples (Node/PHP pequenos) | 2 vCPU / 4 GB RAM / 20-40 GB SSD | — |
| Build + testes unitários médios (monorepo, várias linguagens) | 4 vCPU / 8 GB RAM / 60-100 GB SSD/NVMe | — |
| CI + E2E (Playwright/Cypress, browsers headless) | 4-8 vCPU / 8-16 GB RAM / 100-200 GB NVMe | cada browser headless facilmente consome 500MB-1GB+ de RAM por worker |
| Build de imagens Docker pesadas / matrizes paralelas | 8+ vCPU / 16-32 GB RAM / 200+ GB NVMe | múltiplas camadas simultâneas competem por I/O e CPU |
| Deploy runner (só orquestra SSH/API, não builda) | 1-2 vCPU / 2 GB RAM / 20 GB SSD | — |

Notas de dimensionamento:

- RAM costuma ser o gargalo real em E2E com browsers, não CPU;
- disco cresce mais rápido do que se espera por causa de camadas Docker acumuladas e artifacts de teste (vídeos/screenshots de E2E);
- se usar runners efêmeros (ver seção 54), cada job parte de uma imagem/snapshot limpo. Isso simplifica dimensionamento porque não há acúmulo entre execuções, mas exige que o provisionamento (clone de VM, pull de imagem de container) seja rápido o bastante para não virar gargalo de fila.

Meça antes de ampliar.

---

## 5. Storage

Builds e Docker consomem disco.

Planeje:

- workspace (`_work/`);
- Docker layers e cache de build (BuildKit cache, `docker builder prune` periódico);
- cache de dependências (`~/.npm`, `~/.composer/cache`, `~/.cache/pip`, cache de camadas do Actions `actions/cache`);
- browsers (Playwright/Cypress podem baixar centenas de MB por versão de browser);
- artifacts temporários (vídeos, screenshots, coverage);
- logs.

Sem limpeza periódica, o disco de um runner de build tende a crescer continuamente. Agende `docker system prune` (com critério, nunca `--volumes` sem revisar) e rotação de workspace antigo.

---

## 6. SSD/NVMe

E2E e builds se beneficiam de I/O rápido.

Disco lento pode parecer problema de CPU.

NVMe é preferível a SATA SSD quando a carga envolve muitas operações pequenas e simultâneas: é o caso típico de `docker build` com muitas camadas, `npm install`/`composer install` com milhares de arquivos pequenos, e suítes de E2E gravando vídeo/screenshot em paralelo. Nesses cenários, IOPS e latência importam mais que throughput sequencial.

---

## 7. Filesystem

Monitore:

```bash
df -h
df -i
```

Inodes também podem acabar.

---

## 8. Ubuntu Server LTS

Preferência por versão LTS suportada.

Benefícios:

- estabilidade;
- documentação;
- patches;
- compatibilidade.

---

## 9. Instalação mínima

Instale apenas o necessário.

Menos serviços:

```text
menor superfície
menor manutenção
```

---

## 10. Hostname

Exemplo:

```text
runner-ci-01
runner-e2e-01
runner-deploy-01
```

Nomes devem expressar função.

---

## 11. Usuários

```text
admin
github-runner
deploy
```

Separe funções.

---

## 12. Root

Não use login root rotineiramente.

---

## 13. SSH

Preferência:

```text
public key authentication
```

Desabilitar password login pode ser apropriado após garantir acesso por chave.

---

## 14. sshd_config

Mudanças devem ser feitas com cuidado e sessão de fallback para evitar lockout.

---

## 15. Firewall

Ubuntu pode usar UFW ou nftables conforme política.

Runner normalmente precisa principalmente de tráfego de saída HTTPS para GitHub.

Regras mínimas de saída (outbound) para um runner funcional:

| Porta/protocolo | Destino |
|---|---|
| 443/tcp | github.com, api.github.com, *.actions.githubusercontent.com |
| 443/tcp | ghcr.io, registry-1.docker.io, *.docker.io (se usar Docker Hub) |
| 443/tcp | repositórios de pacotes usados (npm registry, packagist, pypi, apt mirrors) |
| 53/udp,tcp | resolvers DNS internos/externos |
| 123/udp | NTP |
| 22/tcp | apenas se o pipeline usar Git sobre SSH ou deploy via SSH |

Não é necessário liberar entrada (inbound) de internet: o runner sempre inicia a conexão (ver seção 16). Se a organização usa uma lista de permissões restritiva, consulte a lista oficial de ranges/domínios do GitHub Actions e mantenha atualizada, pois pode mudar.

---

## 16. Runner não precisa de porta pública

O runner inicia conexão com o GitHub (long poll / WebSocket sobre HTTPS), não o contrário. Não existe "webhook chegando" no runner em si.

Não abra porta de internet apenas para "receber jobs".

Isso vale mesmo para Actions Runner Controller (ARC) no Kubernetes: o listener do ARC também inicia conexão de saída para o GitHub; não é necessário expor endpoint público para o cluster receber jobs.

---

## 17. DNS

Hosts internos devem possuir nomes previsíveis.

Evite espalhar IPs hardcoded por scripts.

### 17.1 Redundância de resolver

Configure ao menos dois resolvers DNS, com ordem de fallback clara:

```text
resolver primário   -> ex.: DNS interno/autoritativo da rede
resolver secundário -> ex.: resolver externo confiável (1.1.1.1, 8.8.8.8) ou secundário interno
```

Cuidados práticos:

- em `/etc/resolv.conf` (ou no backend usado, `systemd-resolved`/`netplan`), a ordem dos `nameserver` importa como fallback, não como balanceamento. O primeiro que não responder faz o cliente tentar o próximo, com timeout;
- se o resolver primário for um serviço rodando em container (ex.: dentro de um LXC/VM que hospeda o próprio DNS interno), um restart desse container derruba a resolução até o fallback assumir. Teste esse cenário deliberadamente;
- ao reconfigurar DNS em containers/LXC, o `docker restart` do daemon costuma ser necessário para que os containers em execução peguem a nova configuração de rede;
- falha de DNS intermitente é uma causa comum e subestimada de jobs de CI falhando de forma não determinística (timeout ao resolver registry, npm registry, etc.). Antes de suspeitar de "flakiness" do teste, descarte problema de DNS.

---

## 18. NTP

Hora correta é essencial para:

- TLS;
- logs;
- Git;
- tokens;
- auditoria.

Use sincronização de tempo.

---

## 19. Timezone

Servidor pode permanecer em UTC.

Aplicação trata timezone da regra de negócio.

---

## 20. Updates

Rotina:

```bash
sudo apt update
sudo apt upgrade
```

Planeje reboot quando kernel/bibliotecas exigirem.

---

## 21. Patches automáticos

Atualizações automáticas de segurança podem ser úteis.

Avalie janela de manutenção.

---

## 22. Docker

Instale pelo repositório oficial e mantenha versão suportada.

---

## 23. Docker data root

Por padrão:

```text
/var/lib/docker
```

Planeje espaço.

---

## 24. Disco separado

Em runners pesados, volume/disco dedicado para Docker pode simplificar capacidade e manutenção.

---

## 25. Docker log rotation

Configure para evitar crescimento ilimitado.

---

## 26. Runner directory

Exemplo:

```text
/home/github-runner/actions-runner
```

---

## 27. Workspace

```text
_work/
```

é descartável do ponto de vista de negócio.

Não armazene fonte única de dados ali.

---

## 28. Cache

Cache pode ser perdido.

Pipeline deve continuar correto após cache vazio.

---

## 29. Backup do runner

Backup importante:

- scripts;
- IaC;
- docs;
- configs.

Não é necessário preservar builds temporários.

---

## 30. Reconstrução

Meta:

```text
runner perdido
 |
 v
VM nova
 |
 v
provision
 |
 v
registrar
 |
 v
online
```

---

## 31. Infrastructure as Code

Evolução recomendada:

```text
Ansible
Terraform/OpenTofu
cloud-init
```

dependendo do ambiente.

---

## 32. Ansible

Open source e útil para:

- pacotes;
- usuários;
- Docker;
- configs;
- hardening.

---

## 33. OpenTofu

Alternativa open source para provisioning declarativo em provedores compatíveis.

Pode ser útil em cloud/virtualização.

---

## 34. Proxmox

Em infraestrutura local, Proxmox VE pode hospedar VMs/containers, dependendo das necessidades.

Runner de Docker costuma ser simples em VM.

---

## 35. Snapshots

Snapshot não substitui backup.

É útil antes de mudanças de infraestrutura.

---

## 36. UPS

Infraestrutura local deve considerar energia.

Uma UPS reduz:

- corrupção;
- interrupção de builds;
- downtime.

---

## 37. Shutdown controlado

Em queda prolongada, servidores devem poder desligar corretamente.

---

## 38. Rede

Self-hosted runner deve ter conectividade estável.

E2E pode sofrer com perda de pacotes/latência.

---

## 39. VLAN

Uma arquitetura madura pode separar:

```text
CI VLAN
DEV VLAN
PROD VLAN
management VLAN
```

---

## 40. Firewall entre VLANs

Liberar apenas:

```text
CI -> DEV necessário
Deploy -> PROD necessário
```

---

## 41. Proxy

Se rede usa proxy, runner precisa configuração consistente para Git, npm, Composer e Docker.

---

## 42. DNS interno

Use DNS para serviços.

---

## 43. TLS interno

Certificados internos podem exigir CA confiável instalada no runner.

---

## 44. Secrets do host

Nunca salvar em imagem/template sem criptografia/controle.

---

## 45. SSH agent

Evite deixar chaves administrativas persistentes carregadas desnecessariamente.

---

## 46. Monitoring do host

Instale:

```text
Node Exporter
```

ou solução equivalente.

---

## 47. Métricas mínimas

- CPU;
- load;
- memory;
- swap;
- disk;
- inode;
- network.

---

## 48. Swap

Swap pode evitar OOM abrupto, mas não corrige falta crônica de RAM.

E2E lento por swap excessivo precisa de capacidade.

---

## 49. OOM logs

Verifique kernel/journal em falhas inexplicáveis.

---

## 50. journalctl

Exemplo:

```bash
journalctl -u SERVICO
```

Use para runner/Docker/systemd.

---

## 51. systemd

Gerencia:

- Docker;
- runner;
- exporters.

Serviços devem iniciar após reboot.

---

## 52. Reboot test

Teste reboot controlado antes de depender do runner em produção operacional.

Verifique:

```text
Docker sobe
runner sobe
monitoring sobe
```

---

## 53. Health do runner

Verifique periodicamente:

```text
serviço
GitHub status Idle
Docker
disk
network
```

---

## 54. Multiple runners no mesmo host

É possível, mas compartilham:

- CPU;
- RAM;
- Docker;
- disco.

Uma falha pode afetar todos.

### 54.1 Runners persistentes vs efêmeros

Boa prática atual (2025/2026): preferir **runners efêmeros** (`--ephemeral` no registro do runner) em vez de runners persistentes de longa duração.

```text
runner persistente -> registra uma vez, roda N jobs indefinidamente
runner efêmero      -> registra, roda exatamente 1 job, desregistra e é destruído
```

Motivos para preferir efêmero:

- elimina resíduo entre jobs (variáveis de ambiente, arquivos deixados, processos zumbis, estado do Docker): cada job começa limpo;
- reduz a superfície de ataque: um job malicioso ou comprometido não deixa persistência no runner para o próximo job herdar;
- combina bem com IaC (seção 31): o runner "nasce" de uma imagem/template já validado e morre depois de um job, então o provisionamento vira o ponto central de manutenção em vez do runner individual.

O custo é operacional: cada job paga o tempo de provisionamento (subir VM/container, registrar, baixar runner). Isso é mitigado com templates/imagens pré-aquecidas ou com Actions Runner Controller (ver 54.2).

### 54.2 Actions Runner Controller (ARC) e runner scale sets

Para quem já opera Kubernetes, o **Actions Runner Controller (ARC)** é a alternativa oficial da GitHub a manter VMs/LXC de runner manualmente. Ele gerencia **runner scale sets**: grupos de runners efêmeros provisionados sob demanda como pods, escalando de zero conforme a fila de jobs.

```text
GitHub Actions -> fila de jobs
       |
       v
ARC controller (no cluster)
       |
       v
runner scale set -> cria/destrói pods efêmeros conforme demanda
```

Vantagens do ARC frente a runners manuais em VM/LXC:

- escala automaticamente com a fila (scale to zero quando ocioso, reduzindo custo em cloud);
- isolamento por pod/job por padrão (efêmero nativo);
- integra com o mesmo tooling de observabilidade/deploy já usado para as demais cargas em Kubernetes.

Trade-off: exige operar um cluster Kubernetes (complexidade adicional) e ainda herda os mesmos cuidados de isolamento de container discutidos na seção 2.1: pods não são VMs. Para cargas que exigem isolamento de kernel forte (ex.: build de PRs de forks não confiáveis), combine com runtimes com sandbox reforçado (gVisor/Kata Containers) ou mantenha essa carga fora do ARC, em VM dedicada.

Para infraestrutura pequena/local (ex.: um host Proxmox único), VMs ou LXC gerenciados manualmente (ou via Ansible/Terraform) seguem sendo uma opção mais simples do que introduzir Kubernetes só para rodar runners.

---

## 55. Runner por VM

Mais isolamento:

```text
VM ci
VM e2e
VM deploy
```

Isso continua válido com runners efêmeros: cada tipo de carga pode ter seu próprio template/imagem base (VM ou container), dimensionado conforme a seção 4, e provisionado sob demanda.

---

## 56. Deploy runner

Deve ter rede/permissões distintas do CI.

---

## 57. PROD connectivity

Se deploy via SSH:

```text
runner-deploy -> PROD:22
```

Não liberar CI inteiro.

---

## 58. Bastion

Em ambientes maiores:

```text
deploy runner -> bastion -> PROD
```

pode centralizar acesso.

---

## 59. VPN

WireGuard é opção open source para túneis privados.

Pode conectar ambientes remotos.

---

## 60. WireGuard

Benefícios:

- simples;
- moderno;
- baixo overhead.

Ainda exige boa gestão de chaves e rotas.

---

## 61. Backup

Itens:

```text
configuração
scripts
monitoring config
registry se local
dados críticos
```

---

## 62. 3-2-1

Princípio clássico:

```text
3 cópias
2 mídias
1 offsite
```

A implementação depende do risco.

---

## 63. Restore

Teste restore.

Backup não testado é hipótese.

---

## 64. Registry local

Se hospedar registry próprio, trate como serviço crítico.

---

## 65. Availability

Pergunta:

```text
se runner parar, qual impacto?
```

Talvez deploy atrase, mas produção deve continuar funcionando.

---

## 66. CI não deve ser dependência runtime

Aplicação PROD não deve parar porque runner está offline.

---

## 67. GitHub outage

Tenha capacidade de operar/rollback emergencial controlado se GitHub estiver indisponível.

Documente procedimento break-glass.

---

## 68. Break-glass

Acesso emergencial:

- restrito;
- auditado;
- usado somente em incidente.

---

## 69. Capacity planning

Observe crescimento:

```text
jobs/day
E2E duration
Docker disk
queue time
```

---

## 70. Scale up

Mais CPU/RAM na VM.

---

## 71. Scale out

Adicionar runners.

---

## 72. Quando escalar

Se queue time domina pipeline, scale out pode ser mais eficiente.

---

## 73. Runner labels, grupos e capacity

Labels distribuem workloads para o hardware certo:

```text
ci
e2e
deploy
```

O workflow escolhe o runner certo combinando labels no `runs-on`:

```yaml
runs-on: [self-hosted, linux, e2e]
```

### 73.1 Runner groups

Em organizações com múltiplos repositórios/produtos, **runner groups** (recurso de nível organização/enterprise) controlam quais repositórios podem usar quais runners. Esse é o mecanismo correto para garantir que um runner de um produto nunca seja acionável por workflows de outro produto, análogo ao isolamento de fila/banco/container que já se pratica entre produtos na aplicação.

```text
grupo "produto-a-runners" -> só repositórios do produto A
grupo "produto-b-runners" -> só repositórios do produto B
```

Combine labels (o tipo de carga: ci/e2e/deploy) com grupos (quem pode acessar) em vez de depender só de labels. Labels sozinhas não impedem que outro repositório da mesma organização "peça" aquele runner se ele estiver num grupo compartilhado.

---

## 74. Documentação do host

Crie:

```text
docs/infrastructure/runner-ci-01.md
```

Com:

- função;
- SO;
- CPU/RAM;
- IP/DNS;
- labels;
- backup;
- owner;
- recovery.

---

## 75. Inventário

Mantenha inventário versionado sem secrets.

---

## 76. Checklist de instalação

- [ ] Ubuntu LTS.
- [ ] hostname.
- [ ] admin key.
- [ ] usuário runner.
- [ ] updates.
- [ ] firewall.
- [ ] Docker.
- [ ] runner.
- [ ] monitoring.
- [ ] disk alerts.
- [ ] backup config.
- [ ] reboot test.
- [ ] recovery docs.

---

## 77. Checklist de segurança

- [ ] root remoto restrito.
- [ ] passwords desabilitadas quando aplicável.
- [ ] MFA no GitHub.
- [ ] runner sem PROD quando CI.
- [ ] rede segmentada.
- [ ] patches.
- [ ] logs.
- [ ] least privilege.
- [ ] runner efêmero quando possível.
- [ ] PRs de forks externos não rodam automaticamente em self-hosted.
- [ ] runner group restringe o runner ao(s) repositório(s) do produto correto.
- [ ] DNS com resolver secundário configurado e testado.

---

## 78. Arquitetura recomendada final

```text
                 GitHub
                    |
       +------------+------------+
       |                         |
       v                         v
  runner-ci VM              runner-e2e VM
       |
       v
      DEV
       |
       v
  runner-deploy VM
       |
       v
      PROD
```

---

## 79. Próximo volume

**Volume 13 — Arquiteturas de Referência**

Aplicaremos o conjunto em projetos Node.js, PHP, MySQL/MariaDB, MQTT e frontend/backend.

---

**Fim do Volume 12 — Infraestrutura do Servidor CI/CD**

## Fontes

### GitHub Actions — self-hosted runners e rede

- [About self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners) — descreve os modelos de hospedagem possíveis para um runner (física, VM, container, on-premises, cloud), base da seção 2.
- [Self-hosted runners reference](https://docs.github.com/en/actions/reference/runners/self-hosted-runners) — confirma que o runner abre conexão HTTPS de saída (porta 443) para o GitHub e lista os domínios necessários (github.com, api.github.com, *.actions.githubusercontent.com, ghcr.io, etc.), sustentando as seções 15 e 16 (firewall de saída e ausência de porta pública).
- [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions) — afirma que "self-hosted runners should almost never be used for public repositories" e alerta sobre `pull_request_target`/`workflow_run` com PRs não confiáveis, sustentando a seção 2.1 sobre PRs de forks externos.
- [Managing access to self-hosted runners using groups](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/managing-access-to-self-hosted-runners-using-groups) — documenta runner groups e políticas de acesso por repositório, base da seção 73.1.
- [Using labels with self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/using-labels-with-self-hosted-runners) — documenta a criação e atribuição de labels a runners, base da seção 73.
- [Autoscaling with self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/autoscaling-with-self-hosted-runners) — descreve o Actions Runner Controller (ARC) como "the recommended Kubernetes-based solution for autoscaling self-hosted runners", base da seção 54.2.

### Docker

- [Runtime options with Memory, CPUs, and GPUs (resource constraints)](https://docs.docker.com/config/containers/resource_constraints/) — documentação oficial sobre limites de CPU/memória por container, relevante ao dimensionamento (seção 4) e à convivência de múltiplos runners no mesmo host (seção 54).
- [Docker build cache](https://docs.docker.com/build/cache/) — explica o funcionamento do cache de camadas/BuildKit, sustentando a seção 5 (Storage) sobre cache de build e `docker builder prune`.

### systemd

- [systemd.service(5)](https://man7.org/linux/man-pages/man5/systemd.service.5.html) — man page oficial (espelhado do repositório upstream do systemd) sobre unidades de serviço, base da seção 51 (gestão de Docker/runner via systemd).
- [journalctl(1)](https://man7.org/linux/man-pages/man1/journalctl.1.html) — man page oficial do comando usado para inspecionar logs de serviços (`journalctl -u SERVICO`), base da seção 50.

### VPN

- [WireGuard — site oficial](https://www.wireguard.com/) — descreve o WireGuard como "an extremely simple yet fast and modern VPN that utilizes state-of-the-art cryptography", sustentando as seções 59 e 60.
