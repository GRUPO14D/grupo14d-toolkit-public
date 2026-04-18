# RUNBOOK — Auditoria Técnica dos 15 RPAs (gera `CONTEXT.md` por repo)

**Para:** Gemini CLI (em sessão(ões) dedicada(s))
**Papel:** Senior Code Auditor — análise estática, perf, build, correção
**Autor do pedido:** Davi Cassoli Lira
**Data:** 2026-04-18
**Entregável:** 15 arquivos `CONTEXT.md`, um em cada repositório de RPA

---

## 0. Contexto

O Grupo 14D tem 15 RPAs em produção sob `/Users/davicassoli/Trabalho/14D-PROJETOS/`. Todos já reportam telemetria ao MONITOR-RPA. A biblioteca `grupo14d-telemetry` está em construção paralela (Sprints A-E) e **não é foco desta auditoria** — se aparecer referência a `telemetry.py`, apenas registre.

**Objetivo desta auditoria:**
- Produzir um `CONTEXT.md` em cada repo, rigoroso, que sirva de:
  - Documento de entrada pro próximo dev (ou o próprio Davi em 6 meses)
  - Baseline técnico pra priorizar fixes
  - Mapa de dívida técnica com estimativa
- Caçar bugs, leaks, problemas de performance e build
- Sugerir correções (com código) e estimar esforço

**Não-objetivo:**
- Aplicar correções (Davi aplica depois)
- Refatorar arquitetura (sugestões sim, execução não)
- Rodar os RPAs em produção
- Tocar no MONITOR-RPA ou no `grupo14d-telemetry`

---

## 1. Regras de engajamento

1. **Você NÃO executa o código de produção dos RPAs.** Análise estática + leitura apenas.
2. **Você NÃO altera nenhum arquivo fora do `CONTEXT.md` de cada repo.**
3. **Você pode executar:** linters (`ruff check`, `pylint`) em modo read-only; scripts de medição como `tokei`, `radon cc`, `pyflakes`, `vulture`; `python -m py_compile` pra sanidade sintática.
4. **Evidência obrigatória:** todo achado cita `arquivo:linha` ou comando + output.
5. **Severidade obrigatória:** cada achado recebe 🔴 crítico / 🟡 importante / ⚪ cosmético.
6. **Estimativa obrigatória:** cada achado recebe estimativa em horas (15min, 1h, 4h, 1d, etc).
7. **Timebox por RPA:** 45-60 min. Se estourar, entregue parcial marcando no `CONTEXT.md` o que faltou.
8. **Batching obrigatório** (ver seção 3) — não faça os 15 na mesma sessão.

---

## 2. O que analisar em cada repo

Pra cada RPA, cubra **todas** as áreas abaixo. Cada uma vira seção no `CONTEXT.md`.

### 2.1 Visão geral do projeto

- Propósito do RPA (inferir do código + README + docstrings)
- Stack técnica (Python version, libs principais, framework GUI se tiver)
- Status de produção (ativo / legado / em migração)
- Último commit, autor, frequência de commits recentes
- LOC total, por pasta principal
- Comando de build/run (inferir do README, `.spec`, `.bat`, scripts)

### 2.2 Arquitetura e código

- Ponto de entrada (arquivo + função)
- Estrutura de pastas — faz sentido? há duplicação?
- Separação de camadas (GUI / domínio / infra)?
- Uso de padrões (Adapter, Factory, singleton, etc.)
- Dívida técnica óbvia (ex: `TODO`, `FIXME`, `XXX`, `HACK`)
- Código morto detectado (funções nunca chamadas, imports não usados)

### 2.3 Bugs e erros de código

Caça objetiva. Procure especificamente:

- Exceções genéricas `except Exception` ou `except:` que silenciam problemas
- `print()` em código de produção (deveria ser logging)
- Hardcoded paths (Windows `C:\...`, Unix `/home/...`)
- Credenciais hardcoded (senhas, tokens, API keys)
- URLs hardcoded além do MONITOR-RPA (portais de prefeitura, Domínio, etc.)
- `time.sleep()` fixos que deveriam ser polling ou event-driven
- Falta de timeout em chamadas HTTP, Selenium, pyautogui
- Strings comparadas com `==` quando deveria ser enum
- `open()` sem context manager (`with`)
- Mutação de default args (`def f(x=[])` clássico)
- SQL strings concatenadas (se houver DB)
- `eval`, `exec`, `__import__` dinâmicos sem validação
- Race conditions em threads (se houver threading além do telemetry)

### 2.4 Performance

- Leitura/gravação repetida do mesmo arquivo em loop
- `pandas` com `.iterrows()` em datasets grandes (lento notório)
- Regex compiladas dentro de loop em vez de no módulo
- `requests` sem sessão reutilizada
- Selenium sem `WebDriverWait` — usa `time.sleep` fixo
- `pyautogui` com delays excessivos
- I/O serial que poderia ser paralelo (ThreadPoolExecutor)
- Consultas ao Domínio em loop sem batch
- Parsing XML/HTML com `lxml` ou `BeautifulSoup`? Qual parser?
- Logs em nível DEBUG vazando em produção (custo de I/O)

### 2.5 Build e empacotamento (crítico pro Grupo 14D)

A maioria usa **PyInstaller** em Windows. Valide:

- `.spec` existe? É gerado toda vez (ruim) ou versionado (bom)?
- Usa `--onefile` (binário único, lento no boot) ou `--onedir` (pasta, boot rápido)?
- `hidden-imports` listados? Pyinstaller costuma perder imports dinâmicos.
- `collect-all` / `collect-submodules` usado onde necessário?
- `datas` e `binaries` apontam pra caminhos relativos ou absolutos?
- Tamanho do build atual (se conseguir inferir de `.spec` + deps)
- Dependências pesadas sem necessidade (ex: `pandas` inteiro quando só usa CSV)
- `pyinstaller` roda limpo ou com warnings?
- Há `.bat` de build? Está reproducível (mesma saída em máquinas diferentes)?
- Scripts de versionamento do binário (ex: `version_info.txt`)?

**Recomendações pra build mais leve:**

- Sugerir `--exclude-module` pra libs não usadas
- Sugerir `UPX` pra compressão (quando aplicável)
- Sugerir `--onedir` sobre `--onefile` se boot time importa
- Listar deps que podem ser removidas (via `vulture` + análise de imports)
- Sugerir troca de libs pesadas por alternativas (`requests` → `httpx` é neutro; `pandas` → `polars` pesado; `openpyxl` → `xlsxwriter` em escrita; etc.)

### 2.6 Dependências

- `requirements.txt` presente?
- Versões pinadas (`==`) ou flutuantes (`>=`)?
- Dependências não usadas (cross-check `pip list` vs imports reais)
- Vulnerabilidades conhecidas (se tiver acesso a `pip-audit`, roda; senão, cite versões e aponta que precisa)
- Libs abandonadas (última release > 2 anos)
- Duplicação de funcionalidade (`requests` + `httpx` + `urllib` no mesmo projeto)

### 2.7 Testes

- Existem? Em `tests/` ou misturados?
- Framework (`pytest`, `unittest`, nada)?
- Cobertura aproximada (se der pra inferir)
- Testes batem em rede/GUI real (ruim) ou mockam (bom)?
- Há testes de regressão do fluxo principal?

### 2.8 Logging e observabilidade

- Usa `logging` stdlib ou `print`?
- Configuração centralizada ou ad-hoc?
- Níveis de log apropriados?
- Logs persistidos em arquivo? Rotativo?
- Integração com telemetria (esperado: `from telemetry import Telemetry` ou similar)

### 2.9 Segurança

- Credenciais em `.env` ou hardcoded?
- `.env` no `.gitignore`?
- `api_key.txt` ou similar versionado por engano?
- Conexões HTTPS em tudo que é externo?
- Sanitização de inputs antes de passar pra Selenium/pyautogui?

### 2.10 Documentação existente

- README presente? Atualizado?
- Docstrings em funções principais?
- Comentários valiosos ou ruído?
- `CHANGELOG.md`? `CONTRIBUTING.md`?

---

## 3. Batching obrigatório (ORDEM DE EXECUÇÃO)

**Não tente analisar os 15 RPAs na mesma sessão.** Qualidade despenca. Batches recomendados, em ordem:

### Batch 1 — Simples / piloto (1 sessão, ~3h)

1. RPA-SISTEMA-GRUPO14D — conversor utilitário
2. RPA-BALANCETES — reconciliação básica
3. RPA-DROGARIA — conciliação domínio x empresa
4. RPA-TROTS — processamento TROTS
5. RPA-TROTS-ALT-DE-ENTRADA — variante do TROTS

### Batch 2 — Fiscal médio (1 sessão, ~3h)

6. RPA-TROTS-IMPORT — import NFS-e tomados
7. RPA-Conferencia-NFs-e — reconciliação NFS-e
8. RPA-SITTAX — reconciliação tributária
9. RPA-XML-HOSPITAL — processamento XML hospitalar
10. RPA-LANCAMENTOS-REBNIC — Clean Architecture

### Batch 3 — Complexo / crítico (1 sessão, ~3h)

11. RPA-Cartorios — 6 cartórios num só repo, Tkinter
12. RPA-REBNIC — robô conciliador REBNIC-DOMINIO
13. RPA-REBNIC-ALTERACAO-ACUMULADOR-DOMINIO — alteração de acumuladores
14. RPA-COBRANCA-AUTOMATICA — cobrança automática
15. RPA-IRPF — o mais complexo: 3 watchers, queue pg-boss, heartbeat

**Entre batches:** pause. Revise rápido o que produziu. Se houver padrões repetidos, registre no `CONTEXT.md` como "padrão do ecossistema" em vez de repetir 15 vezes.

**Se a sessão estourar no meio de um batch:** complete o RPA em que está, não comece outro, e marque o próximo pra nova sessão.

---

## 4. Template do `CONTEXT.md`

Use **exatamente** esta estrutura em cada repo. Variações quebram a consolidação posterior.

Caminho: `{repo-root}/CONTEXT.md` (sobrescreve se já existir — mas antes cite no topo se tinha conteúdo prévio)

````markdown
# CONTEXT — {NOME-DO-RPA}

> Auditoria técnica automática. Gerada por Gemini CLI em {DATA}.
> Base de conhecimento pra o próximo dev (ou o próprio Davi em 6 meses).
> **Não é substituto de README** — é análise crítica.

---

## 1. Visão geral

| Item | Valor |
|------|-------|
| Propósito | (1-2 linhas) |
| Status em produção | Ativo / Legado / Em migração |
| Stack | Python X.Y, libs principais |
| Ponto de entrada | `caminho/arquivo.py:função` |
| Build | PyInstaller --onefile, etc. |
| LOC total | X linhas (Python) |
| Último commit | {hash}, {data}, {autor} |
| Frequência recente | N commits nos últimos 30 dias |

### Fluxo principal (alto nível)

1. ...
2. ...
3. ...

### Comando de execução

```bash
# (colar comando real encontrado no README/.spec/.bat)
```

---

## 2. Estrutura do repositório

```
arquivo1.py (X linhas) — responsabilidade
src/
├── ui/ — interface Tkinter
├── core/ — lógica de domínio
└── adapters/ — integrações externas
tests/ — Y testes pytest
```

Observações:
- Separação de camadas: (boa / média / fraca)
- Duplicação detectada: ...
- Pastas vazias ou mortas: ...

---

## 3. Achados críticos 🔴

### 3.1 {Título curto do achado}
- **Localização:** `arquivo.py:45-60`
- **Problema:** descrição objetiva
- **Impacto:** o que pode dar errado em produção
- **Fix sugerido:**
  ```python
  # código antes
  ...
  # código depois
  ...
  ```
- **Esforço estimado:** 30 min

### 3.2 ...

---

## 4. Achados importantes 🟡

(mesmo formato da seção 3, para itens não bloqueantes)

---

## 5. Achados cosméticos ⚪

(mesmo formato, lista mais sucinta — pode agrupar)

---

## 6. Performance

### 6.1 Hotspots detectados

| Localização | Problema | Fix sugerido | Esforço |
|-------------|----------|--------------|---------|
| `arquivo:123` | `.iterrows()` em df de 50k linhas | vetorizar com `.apply` ou np | 1h |

### 6.2 I/O e rede

- Sessões `requests` reutilizadas? Sim/Não
- Timeouts em todas chamadas? Sim/Não + exemplos
- Selenium com `WebDriverWait`? Sim/Não

### 6.3 Oportunidades de paralelização

- ...

---

## 7. Build e empacotamento

### 7.1 Configuração atual

- Método: PyInstaller `--onefile` / `--onedir`
- `.spec` versionado: Sim/Não
- Hidden imports declarados: N
- Tamanho estimado do binário: ~X MB

### 7.2 Problemas detectados

- ...

### 7.3 Recomendações pra build mais leve

| Ação | Impacto estimado | Esforço |
|------|------------------|---------|
| Trocar `--onefile` por `--onedir` | boot 5x mais rápido | 30 min |
| Adicionar `--exclude-module pandas.tests` | -12 MB | 10 min |
| Remover `requests` (só urllib) | -3 MB | 2h |

---

## 8. Dependências

### 8.1 requirements.txt

```
(colar trecho relevante)
```

### 8.2 Análise

- Total de deps diretas: X
- Versões pinadas: Sim/Não
- Deps não usadas (via imports vs requirements): ...
- Deps pesadas candidatas a remoção: ...
- Libs abandonadas: ...

---

## 9. Testes

- Framework: pytest / unittest / nenhum
- Número de testes: N
- Cobertura estimada: alta / média / baixa / zero
- Testes batem em rede real: Sim/Não
- Testes do fluxo principal cobertos: Sim/Não

### 9.1 Recomendação

(O que adicionar primeiro, em ordem de prioridade)

---

## 10. Logging e observabilidade

- Uso de `logging` stdlib: Sim/Não
- Uso de `print()` em produção: N ocorrências
- Configuração centralizada: Sim/Não
- Integração com telemetria (`telemetry.py` local): Sim/Não
- Logs persistidos: Sim/Não + caminho

---

## 11. Segurança

### 11.1 Checklist

- [ ] Nenhuma credencial hardcoded
- [ ] `.env` no `.gitignore`
- [ ] Nenhum `api_key.txt` versionado
- [ ] HTTPS em todas as chamadas externas
- [ ] Inputs sanitizados antes de Selenium/pyautogui

### 11.2 Achados

(lista, se houver)

---

## 12. Dívida técnica acumulada

Pesquisa por `TODO`, `FIXME`, `XXX`, `HACK`:

| Arquivo:linha | Categoria | Descrição |
|---------------|-----------|-----------|
| `foo.py:45` | TODO | "refatorar quando tiver tempo" |

Total: N marcadores.

---

## 13. Integração com o ecossistema 14D

- Reporta telemetria: Sim/Não → `rpa_name={...}`
- Formato de config: `config.ini` / `config-global.ini` / `pipeline.yml`
- URL do MONITOR-RPA: `http://192.168.1.3:8000`
- Se aplicável, migração pendente pra `grupo14d-telemetry`: breve nota

---

## 14. Sumário executivo de esforço

| Categoria | Itens | Esforço total estimado |
|-----------|-------|------------------------|
| 🔴 Críticos | N | X horas |
| 🟡 Importantes | N | X horas |
| ⚪ Cosméticos | N | X horas |
| Performance | N | X horas |
| Build | N | X horas |
| **Total** | **N** | **X horas** |

---

## 15. Top 5 ações recomendadas (ordem de prioridade)

1. ... (esforço, impacto)
2. ...
3. ...
4. ...
5. ...

---

## 16. Limitações desta auditoria

- Tempo gasto: ~N min
- O que não foi coberto: ...
- Onde a análise pode estar incompleta: ...

---

*Relatório gerado por Gemini CLI em auditoria estática. Não substitui revisão manual nem testes de regressão.*
````

---

## 5. Passo a passo por RPA (execução)

Pra cada RPA do batch atual, siga **exatamente**:

### 5.1 Setup

```bash
cd /Users/davicassoli/Trabalho/14D-PROJETOS/{RPA}
# se existir CONTEXT.md antigo, salva de lado:
if [ -f CONTEXT.md ]; then cp CONTEXT.md CONTEXT.md.bak; fi
```

### 5.2 Coleta de métricas (comandos a rodar)

Execute e **guarde o output** — vai citar no `CONTEXT.md`.

```bash
# Estrutura
find . -type f -name "*.py" -not -path "*/\.*" -not -path "*/venv/*" | head -50

# LOC
find . -name "*.py" -not -path "*/venv/*" -not -path "*/\.*" | xargs wc -l | tail -1

# Git info
git log -1 --format="%h %ci %an %s"
git log --since="30 days ago" --oneline | wc -l

# TODOs e marcadores
grep -rn "TODO\|FIXME\|XXX\|HACK" --include="*.py" . | head -30

# Prints em produção
grep -rn "^\s*print(" --include="*.py" . | head -20

# Bare except
grep -rn "except:\|except Exception" --include="*.py" . | head -20

# Credenciais suspeitas (padrões óbvios — cuidado com falsos positivos)
grep -rni "password\s*=\s*['\"]" --include="*.py" . | head -10
grep -rni "api_key\s*=\s*['\"]" --include="*.py" . | head -10
grep -rni "token\s*=\s*['\"]" --include="*.py" . | head -10

# Hardcoded paths Windows/Unix
grep -rnE "['\"]C:\\\\" --include="*.py" . | head -10
grep -rnE "['\"]/home/" --include="*.py" . | head -10

# Imports dinâmicos
grep -rn "__import__\|importlib\|exec(\|eval(" --include="*.py" . | head -10

# time.sleep fixos
grep -rn "time\.sleep(" --include="*.py" . | head -20

# requests sem session
grep -rn "requests\.\(get\|post\|put\|delete\)" --include="*.py" . | head -20
```

### 5.3 Ferramentas de análise (read-only, rode se disponível)

```bash
# Sintaxe ok?
python -m py_compile $(find . -name "*.py" -not -path "*/venv/*" -not -path "*/\.*") 2>&1 | head -20

# Ruff (se disponível)
ruff check . --select=E,F,W --statistics 2>/dev/null | head -30

# Pyflakes como fallback
python -m pyflakes . 2>/dev/null | head -30

# Vulture — código morto (se disponível)
vulture . --min-confidence 80 2>/dev/null | head -30

# Radon — complexidade ciclomática (se disponível)
radon cc . -a -nc 2>/dev/null | head -20

# Pip-audit — vulnerabilidades (se disponível + requirements.txt)
pip-audit -r requirements.txt 2>/dev/null | head -30
```

**Se alguma ferramenta não existir:** não instale. Registre no `CONTEXT.md` seção 16 (Limitações) como "ferramenta indisponível, análise X não foi feita".

### 5.4 Análise manual (leitura direcionada)

Leia **pelo menos**:
- `README.md` se existir
- Ponto de entrada (`main.py`, `front.py`, `ui/main_window.py`, etc.)
- `requirements.txt`
- `.spec` do PyInstaller se existir
- Qualquer arquivo `.bat` ou `start*.ps1`
- Um dos arquivos maiores de domínio (identifica pelo `wc -l`)

**Não leia:** venv, `.git`, binários, testes de fornecedor, dumps.

### 5.5 Escrita do `CONTEXT.md`

Use o template da seção 4 deste runbook. Preencha **todas** as 16 seções. Nenhuma seção pode ficar em branco — se não tiver achado, escreve "Nenhum achado nesta categoria".

### 5.6 Commit (opcional — o Davi decide depois)

**NÃO commite.** Deixe o `CONTEXT.md` no working tree apenas. O Davi vai decidir se commita direto no repo, se revisa antes, ou se move pro vault.

---

## 6. Critérios de qualidade do relatório

Pra cada `CONTEXT.md` ser aceito:

### 6.1 Completude

- [ ] 16 seções preenchidas (nenhuma em branco)
- [ ] Tabela da seção 1 com todos os campos
- [ ] Pelo menos 3 achados categorizados (🔴/🟡/⚪) — se tudo perfeito, anote isso explicitamente com evidência
- [ ] Seção 14 (esforço) somando corretamente
- [ ] Seção 15 (top 5) em ordem de prioridade

### 6.2 Rigor

- [ ] Todo achado cita `arquivo:linha`
- [ ] Todo achado tem severidade
- [ ] Todo achado tem estimativa
- [ ] Todo fix sugerido tem código "antes/depois" concreto
- [ ] Nenhum achado usa linguagem vaga ("poderia ser melhor", "talvez")

### 6.3 Valor prático

- [ ] Seção 7 (build) tem pelo menos 1 recomendação concreta com impacto estimado
- [ ] Seção 15 (top 5) reflete o que o Davi faria primeiro, não lista alfabética
- [ ] Seção 16 (limitações) é honesta — não esconde o que não foi coberto

---

## 7. Padrões que provavelmente vão se repetir em vários RPAs

Se você encontrar estes padrões, **referencie** em cada `CONTEXT.md` em vez de reanalisar do zero:

- Uso de `telemetry.py` local (15 cópias — vai virar `grupo14d-telemetry` em breve)
- `requirements.txt` sem pinning
- Configuração em `config.ini` com `[telemetry]` apontando pra `http://192.168.1.3:8000`
- PyInstaller `--onefile` com `.spec` customizado
- Tkinter em Windows
- Ausência de testes automatizados
- `print()` em produção em vez de logging
- `time.sleep()` fixos em código Selenium

Ao encontrar, mencione breve ("padrão do ecossistema 14D, ver também RPA-X") em vez de escrever análise longa repetida.

---

## 8. Formato de entrega

### Por batch:

1. **3-5 `CONTEXT.md`** criados em cada repo do batch
2. **Relatório curto no chat** ao Davi (não um arquivo novo, só mensagem):
   ```
   Batch {N} concluído. RPAs cobertos: X, Y, Z.

   Top achados críticos do batch:
   - RPA-X: {achado} — {esforço}
   - RPA-Y: {achado} — {esforço}

   RPAs com padrões preocupantes: {lista}
   RPAs mais saudáveis: {lista}

   Tempo gasto: N min. Sessão pronta pra batch {N+1}.
   ```

### Entre batches:

Davi revisa o batch anterior, aprova, e libera você pro próximo. **Não avance sem aprovação.**

### Ao fim dos 3 batches:

Gere um 16º arquivo opcional: `/tmp/auditoria-consolidada-15-rpas.md` com:
- Tabela resumo dos 15 RPAs com esforço total e top-3 achados
- Padrões sistêmicos (o que se repete em N RPAs)
- Recomendações arquiteturais transversais (ex: "todos precisam de pinning de deps")
- Top 10 ações priorizadas no ecossistema inteiro

---

## 9. O que NÃO fazer

- ❌ Não analise todos os 15 na mesma sessão — batching é obrigatório
- ❌ Não rode o código dos RPAs
- ❌ Não altere nada fora do `CONTEXT.md` de cada repo
- ❌ Não commite nenhum arquivo (Davi decide)
- ❌ Não invente achados — todo item precisa de `arquivo:linha`
- ❌ Não use linguagem vaga — "poderia" / "talvez" / "provavelmente" são inaceitáveis
- ❌ Não toque em `grupo14d-telemetry` nem no `MONITOR-RPA`
- ❌ Não exceda 60 min por RPA — entregue parcial marcando limitação
- ❌ Não instale ferramentas novas no sistema (use o que tem)
- ❌ Não gere fix que implique arquitetura nova — sugira, mas mantenha escopo em "fix"

---

## 10. Critério de pronto (por RPA)

- [ ] `CONTEXT.md` existe na raiz do repo
- [ ] 16 seções do template preenchidas
- [ ] Todo achado tem arquivo:linha + severidade + esforço
- [ ] Nenhum arquivo além do `CONTEXT.md` foi tocado
- [ ] Comandos da seção 5.2 foram executados e outputs citados quando relevantes
- [ ] Seção 15 (top 5) tem 5 ações priorizadas

## 11. Critério de pronto (por batch)

- [ ] Todos os RPAs do batch têm `CONTEXT.md`
- [ ] Relatório de chat enviado ao Davi com top achados
- [ ] Padrões repetidos entre RPAs do batch identificados

## 12. Critério de pronto (global, fim dos 3 batches)

- [ ] 15 `CONTEXT.md` existem, um em cada repo
- [ ] `/tmp/auditoria-consolidada-15-rpas.md` criado
- [ ] Padrões sistêmicos documentados
- [ ] Top 10 ações do ecossistema priorizadas

---

## 13. Ao terminar cada batch

Pare. Reporte ao Davi conforme seção 8. **Não comece o próximo batch** até ele aprovar.

Se estiver indeciso se algo é achado ou ruído, **registre como ⚪ cosmético** com nota — é mais fácil Davi descartar do que achar depois.
