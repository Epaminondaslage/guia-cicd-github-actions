# Volume 12 — Infraestrutura do Servidor CI/CD

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 12_INFRAESTRUTURA_SERVIDOR_CICD.md  
**Versão:** 0.1.0

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

# 2. Físico ou virtual

Um runner pode operar em:

```text
máquina física
VM
cloud VM
```

Para começar, VM é uma escolha forte por:

- isolamento;
- snapshots;
- facilidade de reconstrução;
- limites de recursos.

---

# 3. Não misturar funções críticas

Evite:

```text
mesmo host:
PROD + banco PROD + runner CI
```

Prefira separação lógica ou física.

---

# 4. Dimensionamento

CI simples:

```text
2-4 vCPU
4-8 GB RAM
SSD
```

CI + E2E:

```text
4-8+ vCPU
8-16+ GB RAM
SSD/NVMe
```

Meça antes de ampliar.

---

# 5. Storage

Builds e Docker consomem disco.

Planeje:

- workspace;
- Docker layers;
- browsers;
- artifacts temporários;
- logs.

---

# 6. SSD/NVMe

E2E e builds se beneficiam de I/O rápido.

Disco lento pode parecer problema de CPU.

---

# 7. Filesystem

Monitore:

```bash
df -h
df -i
```

Inodes também podem acabar.

---

# 8. Ubuntu Server LTS

Preferência por versão LTS suportada.

Benefícios:

- estabilidade;
- documentação;
- patches;
- compatibilidade.

---

# 9. Instalação mínima

Instale apenas o necessário.

Menos serviços:

```text
menor superfície
menor manutenção
```

---

# 10. Hostname

Exemplo:

```text
runner-ci-01
runner-e2e-01
runner-deploy-01
```

Nomes devem expressar função.

---

# 11. Usuários

```text
admin
github-runner
deploy
```

Separe funções.

---

# 12. Root

Não use login root rotineiramente.

---

# 13. SSH

Preferência:

```text
public key authentication
```

Desabilitar password login pode ser apropriado após garantir acesso por chave.

---

# 14. sshd_config

Mudanças devem ser feitas com cuidado e sessão de fallback para evitar lockout.

---

# 15. Firewall

Ubuntu pode usar UFW ou nftables conforme política.

Runner normalmente precisa principalmente de tráfego de saída HTTPS para GitHub.

---

# 16. Runner não precisa de porta pública

O runner inicia conexão com o GitHub.

Não abra porta de internet apenas para "receber jobs".

---

# 17. DNS

Hosts internos devem possuir nomes previsíveis.

Evite espalhar IPs hardcoded por scripts.

---

# 18. NTP

Hora correta é essencial para:

- TLS;
- logs;
- Git;
- tokens;
- auditoria.

Use sincronização de tempo.

---

# 19. Timezone

Servidor pode permanecer em UTC.

Aplicação trata timezone da regra de negócio.

---

# 20. Updates

Rotina:

```bash
sudo apt update
sudo apt upgrade
```

Planeje reboot quando kernel/bibliotecas exigirem.

---

# 21. Patches automáticos

Atualizações automáticas de segurança podem ser úteis.

Avalie janela de manutenção.

---

# 22. Docker

Instale pelo repositório oficial e mantenha versão suportada.

---

# 23. Docker data root

Por padrão:

```text
/var/lib/docker
```

Planeje espaço.

---

# 24. Disco separado

Em runners pesados, volume/disco dedicado para Docker pode simplificar capacidade e manutenção.

---

# 25. Docker log rotation

Configure para evitar crescimento ilimitado.

---

# 26. Runner directory

Exemplo:

```text
/home/github-runner/actions-runner
```

---

# 27. Workspace

```text
_work/
```

é descartável do ponto de vista de negócio.

Não armazene fonte única de dados ali.

---

# 28. Cache

Cache pode ser perdido.

Pipeline deve continuar correto após cache vazio.

---

# 29. Backup do runner

Backup importante:

- scripts;
- IaC;
- docs;
- configs.

Não é necessário preservar builds temporários.

---

# 30. Reconstrução

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

# 31. Infrastructure as Code

Evolução recomendada:

```text
Ansible
Terraform/OpenTofu
cloud-init
```

dependendo do ambiente.

---

# 32. Ansible

Open source e útil para:

- pacotes;
- usuários;
- Docker;
- configs;
- hardening.

---

# 33. OpenTofu

Alternativa open source para provisioning declarativo em provedores compatíveis.

Pode ser útil em cloud/virtualização.

---

# 34. Proxmox

Em infraestrutura local, Proxmox VE pode hospedar VMs/containers, dependendo das necessidades.

Runner de Docker costuma ser simples em VM.

---

# 35. Snapshots

Snapshot não substitui backup.

É útil antes de mudanças de infraestrutura.

---

# 36. UPS

Infraestrutura local deve considerar energia.

Uma UPS reduz:

- corrupção;
- interrupção de builds;
- downtime.

---

# 37. Shutdown controlado

Em queda prolongada, servidores devem poder desligar corretamente.

---

# 38. Rede

Self-hosted runner deve ter conectividade estável.

E2E pode sofrer com perda de pacotes/latência.

---

# 39. VLAN

Uma arquitetura madura pode separar:

```text
CI VLAN
DEV VLAN
PROD VLAN
management VLAN
```

---

# 40. Firewall entre VLANs

Liberar apenas:

```text
CI -> DEV necessário
Deploy -> PROD necessário
```

---

# 41. Proxy

Se rede usa proxy, runner precisa configuração consistente para Git, npm, Composer e Docker.

---

# 42. DNS interno

Use DNS para serviços.

---

# 43. TLS interno

Certificados internos podem exigir CA confiável instalada no runner.

---

# 44. Secrets do host

Nunca salvar em imagem/template sem criptografia/controle.

---

# 45. SSH agent

Evite deixar chaves administrativas persistentes carregadas desnecessariamente.

---

# 46. Monitoring do host

Instale:

```text
Node Exporter
```

ou solução equivalente.

---

# 47. Métricas mínimas

- CPU;
- load;
- memory;
- swap;
- disk;
- inode;
- network.

---

# 48. Swap

Swap pode evitar OOM abrupto, mas não corrige falta crônica de RAM.

E2E lento por swap excessivo precisa de capacidade.

---

# 49. OOM logs

Verifique kernel/journal em falhas inexplicáveis.

---

# 50. journalctl

Exemplo:

```bash
journalctl -u SERVICO
```

Use para runner/Docker/systemd.

---

# 51. systemd

Gerencia:

- Docker;
- runner;
- exporters.

Serviços devem iniciar após reboot.

---

# 52. Reboot test

Teste reboot controlado antes de depender do runner em produção operacional.

Verifique:

```text
Docker sobe
runner sobe
monitoring sobe
```

---

# 53. Health do runner

Verifique periodicamente:

```text
serviço
GitHub status Idle
Docker
disk
network
```

---

# 54. Multiple runners no mesmo host

É possível, mas compartilham:

- CPU;
- RAM;
- Docker;
- disco.

Uma falha pode afetar todos.

---

# 55. Runner por VM

Mais isolamento:

```text
VM ci
VM e2e
VM deploy
```

---

# 56. Deploy runner

Deve ter rede/permissões distintas do CI.

---

# 57. PROD connectivity

Se deploy via SSH:

```text
runner-deploy -> PROD:22
```

Não liberar CI inteiro.

---

# 58. Bastion

Em ambientes maiores:

```text
deploy runner -> bastion -> PROD
```

pode centralizar acesso.

---

# 59. VPN

WireGuard é opção open source para túneis privados.

Pode conectar ambientes remotos.

---

# 60. WireGuard

Benefícios:

- simples;
- moderno;
- baixo overhead.

Ainda exige boa gestão de chaves e rotas.

---

# 61. Backup

Itens:

```text
configuração
scripts
monitoring config
registry se local
dados críticos
```

---

# 62. 3-2-1

Princípio clássico:

```text
3 cópias
2 mídias
1 offsite
```

A implementação depende do risco.

---

# 63. Restore

Teste restore.

Backup não testado é hipótese.

---

# 64. Registry local

Se hospedar registry próprio, trate como serviço crítico.

---

# 65. Availability

Pergunta:

```text
se runner parar, qual impacto?
```

Talvez deploy atrase, mas produção deve continuar funcionando.

---

# 66. CI não deve ser dependência runtime

Aplicação PROD não deve parar porque runner está offline.

---

# 67. GitHub outage

Tenha capacidade de operar/rollback emergencial controlado se GitHub estiver indisponível.

Documente procedimento break-glass.

---

# 68. Break-glass

Acesso emergencial:

- restrito;
- auditado;
- usado somente em incidente.

---

# 69. Capacity planning

Observe crescimento:

```text
jobs/day
E2E duration
Docker disk
queue time
```

---

# 70. Scale up

Mais CPU/RAM na VM.

---

# 71. Scale out

Adicionar runners.

---

# 72. Quando escalar

Se queue time domina pipeline, scale out pode ser mais eficiente.

---

# 73. Runner labels e capacity

Distribua workloads:

```text
ci
e2e
deploy
```

---

# 74. Documentação do host

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

# 75. Inventário

Mantenha inventário versionado sem secrets.

---

# 76. Checklist de instalação

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

# 77. Checklist de segurança

- [ ] root remoto restrito.
- [ ] passwords desabilitadas quando aplicável.
- [ ] MFA no GitHub.
- [ ] runner sem PROD quando CI.
- [ ] rede segmentada.
- [ ] patches.
- [ ] logs.
- [ ] least privilege.

---

# 78. Arquitetura recomendada final

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

# 79. Próximo volume

**Volume 13 — Arquiteturas de Referência**

Aplicaremos o conjunto em projetos Node.js, PHP, MySQL/MariaDB, MQTT e frontend/backend.

---

**Fim do Volume 12 — Infraestrutura do Servidor CI/CD**
