# Guia Pessoal de CI/CD com GitHub Actions, Self-Hosted Runners e IA

> Esta é a estrutura que uso nos meus projetos pessoais. Serve como ponto de partida e referência de boas práticas, mas exige ajuste antes de ser adotada em projetos corporativos — políticas de segurança, compliance, governança de acesso e requisitos de infraestrutura variam de empresa para empresa.

Guia  cobrindo fundamentos de Git, GitHub Actions, runners self-hosted, Docker no pipeline, testes, deploy contínuo, segurança, observabilidade, infraestrutura, arquiteturas de referência, otimização e governança.

## Sobre vibe coding e IA neste guia

"Vibe coding" — deixar um agente de IA gerar código e aceitar o resultado sem revisão criteriosa — virou um movimento global à medida que ferramentas como GitHub Copilot, Cursor e Claude Code tornaram trivial produzir código funcional em minutos. É rápido para prototipar, mas descontrolado é um risco real em produção: código que "funciona na tela" pode conter dependência inventada, lógica que não bate com o requisito real, ou falha de segurança que só aparece sob carga.

Este guia adota IA no desenvolvimento de forma deliberada, não pelo hype: agentes aceleram a escrita de código, mas nunca substituem a responsabilidade humana pelo que é aprovado e enviado a produção. Na prática isso significa spec e plano revisados antes de codar, revisão humana obrigatória do diff linha a linha antes do merge, e agentes distintos para escrever e para revisar — os mesmos princípios de qualquer boa prática de engenharia, aplicados também ao código que a IA ajuda a gerar. O detalhamento completo está no [Volume 08 — Desenvolvimento com Spec + IA](volumes/08_DESENVOLVIMENTO_SPEC_IA.md), incluindo uma comparação direta entre vibe coding descontrolado e o fluxo adotado aqui.

<p align="center">
  <img src="img/img12_light.png#gh-light-mode-only" alt="Diagrama ilustrativo de vibe coding controlado com boas práticas" width="512">
  <img src="img/img12_dark.png#gh-dark-mode-only" alt="Diagrama ilustrativo de vibe coding controlado com boas práticas" width="512">
</p>



## Estrutura deste Guia

| Caminho | Conteúdo |
|---|---|
| [`00_MASTER_CICD_GitHub_Runners_IA.md`](00_MASTER_CICD_GitHub_Runners_IA.md) | Visão geral e índice mestre |
| [`volumes/`](volumes/) | 16 volumes temáticos |
| [`apendices/`](apendices/) | 8 apêndices de referência rápida |

## Volumes

| # | Título | Arquivo |
|---|---|---|
| 01 | Fundamentos de Git | [volumes/01_FUNDAMENTOS_GIT.md](volumes/01_FUNDAMENTOS_GIT.md) |
| 02 | GitHub Pull Requests | [volumes/02_GITHUB_PULL_REQUESTS.md](volumes/02_GITHUB_PULL_REQUESTS.md) |
| 03 | GitHub Actions | [volumes/03_GITHUB_ACTIONS.md](volumes/03_GITHUB_ACTIONS.md) |
| 04 | Self-Hosted Runners (Ubuntu/Docker) | [volumes/04_SELF_HOSTED_RUNNERS_Ubuntu_Docker.md](volumes/04_SELF_HOSTED_RUNNERS_Ubuntu_Docker.md) |
| 05 | Docker no Pipeline | [volumes/05_DOCKER_NO_PIPELINE.md](volumes/05_DOCKER_NO_PIPELINE.md) |
| 06 | Estratégia Profissional de Testes | [volumes/06_ESTRATEGIA_PROFISSIONAL_DE_TESTES.md](volumes/06_ESTRATEGIA_PROFISSIONAL_DE_TESTES.md) |
| 07 | Pipeline CI Profissional | [volumes/07_PIPELINE_CI_PROFISSIONAL.md](volumes/07_PIPELINE_CI_PROFISSIONAL.md) |
| 08 | Desenvolvimento com Spec + IA | [volumes/08_DESENVOLVIMENTO_SPEC_IA.md](volumes/08_DESENVOLVIMENTO_SPEC_IA.md) |
| 09 | Continuous Deployment | [volumes/09_CONTINUOUS_DEPLOYMENT.md](volumes/09_CONTINUOUS_DEPLOYMENT.md) |
| 10 | Segurança do Pipeline | [volumes/10_SEGURANCA_DO_PIPELINE.md](volumes/10_SEGURANCA_DO_PIPELINE.md) |
| 11 | Observabilidade | [volumes/11_OBSERVABILIDADE.md](volumes/11_OBSERVABILIDADE.md) |
| 12 | Infraestrutura de Servidor CI/CD | [volumes/12_INFRAESTRUTURA_SERVIDOR_CICD.md](volumes/12_INFRAESTRUTURA_SERVIDOR_CICD.md) |
| 13 | Arquiteturas de Referência | [volumes/13_ARQUITETURAS_DE_REFERENCIA.md](volumes/13_ARQUITETURAS_DE_REFERENCIA.md) |
| 14 | Otimização e Escalabilidade | [volumes/14_OTIMIZACAO_E_ESCALABILIDADE.md](volumes/14_OTIMIZACAO_E_ESCALABILIDADE.md) |
| 15 | Governança e Operação | [volumes/15_GOVERNANCA_E_OPERACAO.md](volumes/15_GOVERNANCA_E_OPERACAO.md) |
| 16 | Laboratório Completo | [volumes/16_LABORATORIO_COMPLETO.md](volumes/16_LABORATORIO_COMPLETO.md) |

## Apêndices

| # | Título | Arquivo |
|---|---|---|
| A | Comandos Git | [apendices/APENDICE_A_COMANDOS_GIT.md](apendices/APENDICE_A_COMANDOS_GIT.md) |
| B | GitHub CLI | [apendices/APENDICE_B_GITHUB_CLI.md](apendices/APENDICE_B_GITHUB_CLI.md) |
| C | Docker CLI | [apendices/APENDICE_C_DOCKER_CLI.md](apendices/APENDICE_C_DOCKER_CLI.md) |
| D | Linux para CI/CD | [apendices/APENDICE_D_LINUX_CICD.md](apendices/APENDICE_D_LINUX_CICD.md) |
| E | YAML | [apendices/APENDICE_E_YAML.md](apendices/APENDICE_E_YAML.md) |
| F | Expressões do GitHub Actions | [apendices/APENDICE_F_GITHUB_ACTIONS_EXPRESSOES.md](apendices/APENDICE_F_GITHUB_ACTIONS_EXPRESSOES.md) |
| G | Troubleshooting | [apendices/APENDICE_G_TROUBLESHOOTING.md](apendices/APENDICE_G_TROUBLESHOOTING.md) |
| H | Checklists | [apendices/APENDICE_H_CHECKLISTS.md](apendices/APENDICE_H_CHECKLISTS.md) |
