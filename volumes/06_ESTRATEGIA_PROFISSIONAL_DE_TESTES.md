# Volume 06 — Estratégia Profissional de Testes

**Projeto:** Guia Profissional de CI/CD com GitHub Actions, Self-Hosted Runners e IA  
**Documento:** 06_ESTRATEGIA_PROFISSIONAL_DE_TESTES.md  
**Versão:** 0.1.0  
**Pré-requisitos:** Volumes 01 a 05

---

## 1. Objetivo

Este volume define uma estratégia de testes capaz de crescer com o sistema sem transformar cada Pull Request em uma execução excessivamente lenta.

O princípio central é:

```text
quanto mais barato o teste
mais cedo e mais frequentemente ele deve executar
```

Arquitetura:

```text
PR
 |
 +-- lint
 +-- typecheck
 +-- unit
 +-- integration
 +-- smoke E2E
 |
 v
Merge
 |
 v
DEV
 |
 +-- E2E relacionado
 |
 v
Aprovação
 |
 v
PROD
 |
 +-- health check
 +-- smoke seguro
```

Em paralelo:

```text
Nightly
 |
 +-- regressão completa
 +-- E2E completo
 +-- verificações demoradas
```

---

# 2. Por que os testes ficam lentos

À medida que o sistema cresce, aumentam:

- telas;
- endpoints;
- fluxos;
- integrações;
- combinações de dados;
- browsers;
- dependências externas;
- cenários de regressão.

Se toda PR executar tudo, o tempo tende a crescer continuamente.

A solução não é apenas adicionar CPU.

É necessário classificar os testes.

---

# 3. Pirâmide de testes

```text
             E2E
          /-------\
       Integração
     /-------------\
       Unitários
```

A maior quantidade de testes deve estar na base.

Características:

| Tipo | Velocidade | Isolamento | Custo |
|---|---:|---:|---:|
| Unitário | Alta | Alto | Baixo |
| Integração | Média | Médio | Médio |
| E2E | Baixa | Baixo | Alto |

---

# 4. Testes unitários

Validam pequenas unidades de lógica.

Exemplo:

```javascript
calcularTotal(itens)
```

Testes:

```text
lista vazia
um item
desconto
arredondamento
valor inválido
```

Não precisam abrir browser nem subir banco real.

---

# 5. Características desejadas dos unitários

- rápidos;
- determinísticos;
- isolados;
- fáceis de diagnosticar;
- executáveis localmente;
- numerosos.

Na PR, devem rodar sempre.

---

# 6. Jest e Vitest

Para Node.js/JavaScript:

```text
Vitest
Jest
```

A escolha depende do stack.

Scripts:

```json
{
  "scripts": {
    "test:unit": "vitest run"
  }
}
```

Pipeline:

```bash
npm run test:unit
```

---

# 7. PHPUnit

Para PHP:

```bash
vendor/bin/phpunit
```

Script Composer:

```json
{
  "scripts": {
    "test": "phpunit"
  }
}
```

O importante é expor um comando padronizado ao CI.

---

# 8. Testes de integração

Validam colaboração entre componentes.

Exemplos:

```text
API + MariaDB
Service + Redis
Backend + MQTT
Repository + banco
```

São mais caros que unitários, mas ainda devem ser relativamente rápidos.

---

# 9. Infraestrutura descartável

Com Docker:

```text
CI
 |
 v
docker compose
 |
 +-- MariaDB
 +-- Redis
 +-- Mosquitto
 |
 v
integration tests
 |
 v
down -v
```

Isso evita dependência de infraestrutura compartilhada.

---

# 10. Testes de API

Podem validar endpoints sem browser.

Exemplo:

```text
POST /login
GET /tickets
POST /clientes
```

São geralmente mais rápidos que E2E completo.

Portanto, grande parte da regra funcional pode ser coberta na camada de API.

---

# 11. Contract tests

Testes de contrato verificam compatibilidade entre componentes.

Exemplo:

```text
Frontend espera:

GET /tickets

{
  "items": [...]
}
```

Se backend alterar o contrato inesperadamente, o teste deve detectar.

---

# 12. E2E

End-to-End testa o fluxo real.

```text
Browser
 |
 v
Frontend
 |
 v
API
 |
 v
Banco
```

Exemplo:

```text
usuário abre login
preenche credenciais
entra no dashboard
cria chamado
consulta chamado
```

---

# 13. O valor do E2E

E2E encontra problemas que unitários podem não encontrar:

- roteamento;
- build;
- JavaScript no browser;
- autenticação;
- cookies;
- CORS;
- integração frontend/backend;
- banco;
- comportamento real.

Mas custa mais tempo.

---

# 14. Smoke E2E

Smoke é um subconjunto pequeno e crítico.

Exemplo:

```text
login
dashboard
criação básica
consulta básica
logout
```

Objetivo:

> O núcleo do sistema está funcional?

---

# 15. Full E2E

A suíte completa pode conter:

```text
login
perfis
permissões
cadastros
filtros
relatórios
erros
mobile
workflows especiais
```

Não precisa necessariamente executar em todo push.

---

# 16. Classificação por tags

Playwright:

```typescript
test('login válido @smoke', async ({ page }) => {
  // ...
});
```

Execução:

```bash
npx playwright test --grep @smoke
```

Full:

```bash
npx playwright test
```

---

# 17. Estratégia por evento

## Pull Request

```text
lint
unit
integration
smoke E2E
```

## Merge em main

```text
build
deploy DEV
E2E relacionado
```

## Nightly

```text
full E2E
regression
```

## Produção

```text
health
smoke seguro
```

---

# 18. Testes afetados

Se apenas:

```text
frontend/chamados/
```

mudou, podemos priorizar testes de chamados.

Estratégia futura:

```text
git diff
 |
 v
mapear módulos afetados
 |
 v
selecionar testes
```

Não implemente seleção agressiva sem cobertura alternativa.

---

# 19. Test impact analysis

Ferramentas e arquiteturas podem inferir quais testes são impactados por uma mudança.

Benefício:

```text
1000 testes
 |
 v
executar 150 relevantes
```

Risco:

```text
dependência indireta não mapeada
```

Por isso a suíte completa continua existindo em algum ciclo.

---

# 20. Testes noturnos

Workflow:

```yaml
on:
  schedule:
    - cron: '0 3 * * *'
```

Executar:

```bash
npm run test:e2e
```

É uma boa forma de deslocar testes demorados para fora da PR.

---

# 21. Testes antes de release

Uma release importante pode exigir:

```text
unit
integration
full E2E
security
migration test
backup verification
```

Nem todo commit precisa pagar esse custo.

---

# 22. Paralelização

Playwright suporta múltiplos workers.

Exemplo:

```bash
npx playwright test --workers=4
```

Mais workers não significa sempre melhor.

Limites podem ser:

- CPU;
- RAM;
- banco;
- portas;
- dados compartilhados.

---

# 23. Sharding

Uma suíte pode ser dividida:

```text
Shard 1/4
Shard 2/4
Shard 3/4
Shard 4/4
```

Com múltiplos runners, isso reduz tempo total.

Exemplo conceitual:

```bash
npx playwright test --shard=1/4
```

---

# 24. Runner único

Se existe somente um runner:

```text
shard 1
shard 2
shard 3
```

continuarão em fila.

Nesse caso, sharding não traz ganho real de paralelismo.

---

# 25. Múltiplos runners

```text
runner-e2e-01 -> shard 1
runner-e2e-02 -> shard 2
runner-e2e-03 -> shard 3
runner-e2e-04 -> shard 4
```

Esse é um cenário futuro de escalabilidade.

---

# 26. Testes flaky

Um teste flaky falha sem regressão real.

Exemplo:

```text
execução 1 PASS
execução 2 FAIL
execução 3 PASS
```

Causas comuns:

- race condition;
- timeout;
- seletor instável;
- dado compartilhado;
- dependência externa;
- ordem de testes.

---

# 27. Não normalizar flaky tests

Se a equipe sempre clica:

```text
rerun
```

até passar, o CI perde credibilidade.

Flaky test deve gerar investigação.

---

# 28. Seletores E2E

Prefira seletores estáveis.

Exemplo Playwright:

```typescript
page.getByRole('button', { name: 'Salvar' })
```

ou:

```text
data-testid
```

Evite seletores acoplados a detalhes frágeis de CSS quando não necessário.

---

# 29. Auto-waiting

Frameworks modernos como Playwright aguardam automaticamente certas condições.

Não adicione:

```javascript
waitForTimeout(5000)
```

como solução padrão.

Espere o estado correto.

---

# 30. Dados de teste

Cada teste deve controlar seus dados.

Ruim:

```text
usar cliente "João" que já existe no DEV
```

Melhor:

```text
setup cria cliente único
teste usa
cleanup remove
```

---

# 31. Identificadores únicos

Exemplo:

```text
cliente-ci-12345
```

ou:

```text
ticket-${RUN_ID}
```

Evita colisões entre execuções.

---

# 32. Fixtures

Fixtures ajudam a preparar contexto previsível.

Exemplo:

```text
usuário admin
usuário bloqueado
cliente ativo
ticket fechado
```

Fixtures devem ser versionadas e reproduzíveis.

---

# 33. Seed de teste

Fluxo:

```text
banco vazio
 |
 v
migrations
 |
 v
seed test
 |
 v
tests
```

O seed não deve conter dados reais sensíveis.

---

# 34. Test isolation

Um teste não deve depender da execução anterior.

Ideal:

```text
Teste A
independente

Teste B
independente
```

Não:

```text
A cria usuário
B depende daquele usuário
C depende do resultado de B
```

---

# 35. Ordem aleatória

Quando possível, executar testes em ordem variável pode revelar dependências ocultas.

A suíte deveria continuar passando.

---

# 36. Database transaction

Em testes de backend, uma estratégia é abrir uma transaction por teste e fazer rollback.

Isso pode acelerar cleanup.

Depende do framework e do comportamento testado.

---

# 37. Mock

Mock substitui uma dependência.

Exemplo:

```text
serviço de pagamento real
        X
mock
```

Use em unit tests.

Não transforme todos os testes em mocks a ponto de não validar integração real.

---

# 38. Stub

Stub fornece respostas controladas.

Exemplo:

```text
API externa -> resposta fixa
```

Útil para cenários previsíveis.

---

# 39. Fake

Uma implementação simplificada funcional.

Exemplo:

```text
in-memory repository
```

Pode ser útil em testes rápidos.

---

# 40. Serviços externos

Não dependa de:

```text
API pública real
SMTP externo
broker PROD
```

para toda execução de PR.

Use doubles ou ambientes controlados.

---

# 41. Testes contra serviços reais

Podem existir em uma camada específica:

```text
integration-external
```

executada:

- manualmente;
- nightly;
- antes de release.

---

# 42. Testes visuais

E2E funcional não garante aparência.

Visual regression:

```text
baseline screenshot
       |
       v
screenshot atual
       |
       v
comparação
```

Útil para frontend.

---

# 43. Revisão visual humana

Mesmo com teste visual, mudanças intencionais precisam de aprovação.

Fluxo:

```text
diff visual
 |
 v
humano aprova baseline novo
```

---

# 44. Responsive testing

Frontend pode ser validado em viewports:

```text
desktop
tablet
mobile
```

Não é necessário executar dezenas de resoluções em toda PR.

Selecione representações importantes.

---

# 45. Browsers

Playwright:

```text
Chromium
Firefox
WebKit
```

Executar todos em toda PR pode ser caro.

Estratégia:

```text
PR -> Chromium
Nightly -> Chromium + Firefox + WebKit
```

se o requisito de compatibilidade permitir.

---

# 46. Testes de performance

Não são substitutos de E2E.

Podem medir:

- latência;
- throughput;
- uso de CPU;
- memória.

Ferramentas podem incluir k6 ou outras.

---

# 47. Performance gate

Evite bloquear PR por microvariações instáveis.

Use métricas robustas.

Exemplo:

```text
p95 < limite
erro < limite
```

em ambiente controlado.

---

# 48. Load test

Não execute carga pesada no mesmo runner de CI sem planejamento.

Pode interferir em outros jobs.

Prefira ambiente dedicado.

---

# 49. Security tests

Camadas:

```text
dependency scan
static analysis
container scan
secret scan
dynamic tests
```

Serão aprofundadas no Volume 10.

---

# 50. Mutation testing

Mutation testing altera o código propositalmente e verifica se os testes detectam.

É uma técnica avançada para medir força da suíte.

É cara e pode rodar fora da PR cotidiana.

---

# 51. Coverage

Coverage ajuda a identificar áreas não exercitadas.

Mas:

```text
100% coverage
```

não significa:

```text
100% qualidade
```

Coverage mede execução, não qualidade das assertions.

---

# 52. Quality gate de coverage

Pode existir:

```text
mínimo 80%
```

Mas não use um número arbitrário sem contexto.

Priorize código crítico.

---

# 53. Testes por risco

Classifique:

```text
Crítico
Alto
Médio
Baixo
```

Fluxos críticos merecem:

- unit;
- integration;
- E2E;
- monitoramento pós-deploy.

---

# 54. Exemplo de matriz de risco

| Função | Risco | Estratégia |
|---|---|---|
| Login | Alto | Unit + API + E2E |
| Relatório visual | Médio | Unit + E2E |
| CSS decorativo | Baixo | Visual/revisão |
| Cobrança | Crítico | múltiplas camadas |

---

# 55. Critérios de aceitação

SPEC:

```text
Usuário bloqueado não consegue autenticar
```

Mapeamento:

```text
unit -> regra de bloqueio
API -> retorna status esperado
E2E -> mensagem correta ao usuário
```

---

# 56. Test traceability

Podemos manter relação:

```text
SPEC-014
 |
 +-- TEST-UNIT-21
 +-- TEST-API-08
 +-- E2E-LOGIN-03
```

Nem sempre precisa ser formal, mas sistemas críticos se beneficiam.

---

# 57. TDD

Test Driven Development:

```text
Red
 |
 v
Green
 |
 v
Refactor
```

Pode ser usado em partes adequadas.

Não é obrigatório para toda funcionalidade.

---

# 58. Desenvolvimento assistido por IA

Fluxo útil:

```text
SPEC
 |
 v
IA propõe testes
 |
 v
humano revisa critérios
 |
 v
implementação
 |
 v
pipeline executa
```

A IA não deve decidir sozinha o que constitui sucesso do negócio.

---

# 59. Test-first para bugs

Ao corrigir bug:

```text
1. criar teste que reproduz
2. confirmar falha
3. corrigir
4. confirmar PASS
```

Isso protege contra regressão futura.

---

# 60. Regression test

Todo bug significativo corrigido deveria, quando possível, deixar um teste.

Assim:

```text
bug antigo
 |
 v
teste permanente
```

---

# 61. Smoke pós-deploy

Produção:

```text
health
 |
 v
login técnico seguro
 |
 v
consulta não destrutiva
```

Deve ser rápido.

---

# 62. Synthetic monitoring

Uma evolução é executar continuamente pequenos fluxos sintéticos.

Exemplo:

```text
a cada 5 minutos
 |
 v
login de monitoramento
 |
 v
endpoint crítico
```

Isso é observabilidade, não CI.

---

# 63. Tempo de feedback

Métrica importante:

```text
commit -> resultado útil
```

Objetivo:

- segundos/minutos para erros simples;
- poucos minutos para CI comum;
- testes longos fora do caminho crítico.

---

# 64. Orçamento de tempo

Exemplo inicial:

```text
lint        < 1 min
unit        < 2 min
integration < 5 min
smoke E2E   < 5 min
```

Não são regras universais.

Use como metas locais depois de medir.

---

# 65. Sequencial versus paralelo

Se runner é único:

```text
lint -> unit -> integration -> E2E
```

Se vários:

```text
lint ----+
unit ----+--> E2E
integration+
```

Otimize conforme capacidade real.

---

# 66. Fail fast

Testes baratos primeiro:

```text
lint
 |
 v
unit
 |
 v
integration
 |
 v
E2E
```

Se lint falha, economizamos os testes caros.

---

# 67. Pipeline híbrido

Podemos paralelizar:

```text
lint + unit
```

e só depois:

```text
integration
E2E
```

Isso equilibra latência e custo.

---

# 68. Cache

Cache pode reduzir:

- npm install;
- Composer;
- browsers;
- Docker layers.

Mas cache incorreto pode gerar bugs.

Sempre permita reconstrução limpa.

---

# 69. Cache poisoning

Em ambientes não confiáveis, cache compartilhado também é superfície de segurança.

Será aprofundado em segurança.

---

# 70. Artifact de falha

Em E2E, preserve:

```text
trace
video
screenshot
report
app logs
```

Somente quando necessário para evitar armazenamento excessivo.

---

# 71. Trace on first retry

Playwright pode ser configurado para gerar trace em retry.

Isso reduz artifact em execuções bem-sucedidas.

---

# 72. Retry

Retries podem ser úteis para coletar evidências, mas não devem mascarar flaky tests.

Exemplo:

```text
CI: 1 retry
Local: 0 retry
```

e relatório deve mostrar que houve retry.

---

# 73. Quarantine

Se um teste está flaky e bloqueia todo desenvolvimento:

```text
quarantine temporária
```

pode ser usada com Issue obrigatória e prazo.

Não transforme quarantine em cemitério permanente.

---

# 74. Test owner

Em equipes, cada suíte ou domínio pode possuir responsável.

Em projeto individual, registre pelo menos:

```text
por que existe
qual requisito cobre
```

---

# 75. Naming

Nomes de teste devem descrever comportamento.

Bom:

```text
usuário bloqueado recebe erro ao autenticar
```

Ruim:

```text
teste 7
```

---

# 76. Arrange / Act / Assert

Estrutura:

```text
Arrange
prepara

Act
executa

Assert
verifica
```

Melhora legibilidade.

---

# 77. Given / When / Then

Alternativa:

```text
Given usuário bloqueado
When tenta autenticar
Then acesso é negado
```

Útil para regras de negócio.

---

# 78. Testes de data/hora

Evite depender do relógio real.

Use:

- clock injetável;
- fake timers;
- datas fixas.

Isso reduz flakiness.

---

# 79. Testes de timezone

Se o sistema possui regras locais, inclua cenários de timezone explicitamente.

Datas são fonte comum de bugs.

---

# 80. Random

Se utiliza dados aleatórios, preserve seed para reproduzir falha.

Exemplo:

```text
seed=48372
```

Sem reprodução, debugging fica difícil.

---

# 81. CI reproducibility

Uma falha no CI deve ser reproduzível localmente.

Por isso scripts padronizados:

```bash
npm run test:ci
```

ou:

```bash
make test-ci
```

são úteis.

---

# 82. Comando único

Ideal:

```bash
npm run ci
```

pode executar a sequência local equivalente aos checks rápidos.

---

# 83. E2E local

Desenvolvedor deve poder executar:

```bash
npm run test:e2e:smoke
```

sem depender do GitHub.

CI não deve ser o único lugar onde testes funcionam.

---

# 84. Docker test environment

Um comando:

```bash
docker compose -f compose.test.yml up --abort-on-container-exit
```

pode ser útil em determinadas arquiteturas.

O desenho deve ser ajustado ao projeto.

---

# 85. Testcontainers

Bibliotecas Testcontainers permitem subir containers diretamente a partir dos testes.

Vantagem:

```text
teste controla infraestrutura
```

Pode ser interessante para Node/PHP conforme suporte.

É uma alternativa a Compose.

---

# 86. Compose versus Testcontainers

Compose:

```text
bom para stack explícita
```

Testcontainers:

```text
bom para infraestrutura controlada pelo código de teste
```

Escolha conforme complexidade.

---

# 87. Banco compartilhado no DEV

E2E de PR não deve poluir banco DEV compartilhado.

Prefira ambiente isolado.

Se isso ainda não for possível, use namespaces/dados únicos e cleanup rigoroso.

---

# 88. Preview environments

Uma evolução futura:

```text
PR #42
 |
 v
ambiente preview-42
```

E2E roda ali.

Depois do merge:

```text
ambiente destruído
```

É poderoso, mas aumenta infraestrutura.

---

# 89. Sem staging

Se o fluxo é:

```text
DEV -> PROD
```

DEV deve funcionar como principal ambiente de validação integrada.

Isso aumenta importância de:

- dados adequados;
- smoke;
- validação visual;
- gate humano.

---

# 90. Gate humano

Automação responde:

```text
tecnicamente passou?
```

Humano responde:

```text
quero publicar agora?
```

São decisões diferentes.

---

# 91. Frontend

Para frontend, inclua:

```text
E2E funcional
visual
responsive
acessibilidade
```

conforme risco.

---

# 92. Accessibility

Ferramentas automáticas podem detectar parte dos problemas de acessibilidade.

Não substituem revisão completa, mas ajudam.

Pode ser um check específico.

---

# 93. Testes de autorização

Não teste apenas:

```text
usuário autorizado consegue
```

Teste também:

```text
usuário não autorizado não consegue
```

Security behavior merece testes negativos.

---

# 94. Testes de erro

Inclua:

- timeout;
- banco indisponível;
- payload inválido;
- autenticação inválida;
- recurso inexistente.

Sistemas falham também nos caminhos de erro.

---

# 95. Resiliência

Alguns testes podem validar:

```text
reconexão MQTT
retry controlado
circuit breaker
```

Especialmente em automação/IoT.

---

# 96. Testes MQTT

Exemplo:

```text
publish comando
 |
 v
aplicação processa
 |
 v
status publicado
 |
 v
assert
```

Use broker isolado.

---

# 97. Mensagens assíncronas

Testes assíncronos devem esperar condição real, não dormir tempo arbitrário.

Exemplo:

```text
aguardar mensagem até timeout
```

não:

```text
sleep 10 e torcer
```

---

# 98. Teste de idempotência

Para webhooks, jobs e eventos:

```text
mesmo evento duas vezes
```

não deveria criar efeitos duplicados quando a operação deve ser idempotente.

---

# 99. Teste de migrations

CI deve validar:

```text
banco vazio -> latest
```

E, quando importante:

```text
versão anterior -> latest
```

---

# 100. Teste de rollback de migration

Só se a estratégia de banco realmente suportar rollback.

Muitas migrations de produção são melhor tratadas como forward-only com expand/contract.

---

# 101. Testes de backup/restore

Backup deve ser testado.

Fluxo:

```text
backup
 |
 v
restore em ambiente isolado
 |
 v
validação
```

Não precisa executar em toda PR.

Pode ser tarefa periódica.

---

# 102. Testes de deploy

Podemos testar scripts de deploy em ambiente de laboratório.

O script de PROD não deveria ser a primeira vez executado durante uma emergência.

---

# 103. Teste de rollback

Laboratório:

```text
deploy versão A
deploy B
simular falha
rollback A
```

Meça e documente.

---

# 104. Quality gates sugeridos

PR:

```text
lint PASS
unit PASS
integration PASS
build PASS
smoke E2E PASS
```

Nightly:

```text
full E2E
visual selecionado
security ampliado
```

Produção:

```text
health PASS
smoke seguro PASS
```

---

# 105. Quando bloquear merge

Bloqueie por testes que são:

- confiáveis;
- relevantes;
- determinísticos;
- obrigatórios.

Não transforme testes experimentais em required check antes de estabilizá-los.

---

# 106. Dashboard de testes

Métricas úteis:

```text
pass rate
duration
flaky rate
top failures
slowest tests
```

Com o tempo, isso orienta otimização.

---

# 107. Slow test report

Classifique os testes mais lentos.

Frequentemente poucos testes dominam o tempo total.

---

# 108. Otimização

Ordem sugerida:

```text
1. medir
2. remover waits desnecessários
3. reduzir setup repetido
4. paralelizar
5. selecionar testes
6. adicionar runners
```

Não comece comprando hardware sem medir.

---

# 109. Teste como documentação

Um bom teste explica comportamento esperado.

Quando a SPEC muda, testes devem mudar conscientemente.

---

# 110. IA e revisão de testes

Use IA para:

- sugerir casos de borda;
- detectar cobertura ausente;
- explicar falhas;
- gerar scaffolding.

Mas revise:

- assertions;
- mocks;
- critérios;
- risco de teste tautológico.

---

# 111. Tautological test

Exemplo ruim:

```text
implementação retorna X
teste copia a mesma lógica e calcula X
```

O teste não oferece independência real.

---

# 112. Testes de regressão dirigidos por incidentes

Após incidente:

```text
post-mortem
 |
 v
teste que teria detectado
```

Isso aumenta maturidade do sistema.

---

# 113. Checklist de PR

- [ ] Unitários pertinentes.
- [ ] Integração pertinente.
- [ ] E2E crítico.
- [ ] Dados isolados.
- [ ] Sem dependência de PROD.
- [ ] Sem waits arbitrários.
- [ ] Falhas produzem evidência.
- [ ] Testes novos representam a SPEC.
- [ ] Tempo do pipeline continua aceitável.

---

# 114. Checklist de E2E

- [ ] Fluxo realmente merece E2E.
- [ ] Seletor estável.
- [ ] Dados únicos.
- [ ] Setup previsível.
- [ ] Cleanup.
- [ ] Timeout razoável.
- [ ] Trace/screenshot configurados.
- [ ] Não depende da ordem.
- [ ] Não depende de serviço externo instável.

---

# 115. Estratégia inicial recomendada

```text
PR:
  lint
  unit
  integration
  Chromium smoke E2E

DEV:
  E2E do domínio alterado
  validação visual

Nightly:
  full E2E
  múltiplos browsers se necessário

PROD:
  health
  smoke não destrutivo
```

---

# 116. Meta de evolução

Quando o sistema crescer:

```text
test impact analysis
sharding
múltiplos runners
preview environments
visual regression
synthetic monitoring
```

devem ser adicionados apenas quando trouxerem valor mensurável.

---

# 117. Próximo volume

**Volume 07 — Pipeline CI Profissional**

O próximo volume consolidará:

- workflows completos;
- Node.js;
- PHP;
- banco;
- MQTT;
- Docker;
- quality gates;
- artifacts;
- cache;
- self-hosted runners;
- PR checks.

---

**Fim do Volume 06 — Estratégia Profissional de Testes**
