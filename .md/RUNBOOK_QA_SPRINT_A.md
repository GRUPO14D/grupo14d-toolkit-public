# RUNBOOK QA — Sprint A `grupo14d-telemetry`

**Para:** Gemini CLI (em nova sessão)
**Papel:** Quality Assurance Engineer + Arquiteto Revisor
**Contexto prévio:** Você já revisou o `SDD_GRUPO14D_TELEMETRY_V0.1.md` (parecer: APROVADO com ressalvas). A v1.1 do SDD incorporou todas suas ressalvas — ver changelog na seção inicial do SDD.
**Agora:** o Claude Code terminou o **Sprint A (scaffolding)** seguindo o `RUNBOOK_SPRINT_A.md`. Seu trabalho é **validar o resultado** e **sugerir melhorias arquiteturais** pro Sprint B não nascer torto.

---

## 0. Princípios do seu trabalho

1. **Você NÃO escreve código.** Nenhuma linha. Nem correção, nem patch. Apenas leitura, análise e relatório.
2. **Você NÃO altera arquivos do repo.** Só produz o relatório final.
3. **Critique com base em evidência.** Se apontar problema, cite arquivo e linha. "Parece frágil" sem fonte é descartado.
4. **Distingua claramente:** (a) desvios do runbook, (b) bugs reais, (c) sugestões arquiteturais opcionais.
5. **Não invente contexto.** Se algo não está documentado no SDD v1.1, pergunte ao Davi, não deduza.
6. **Timebox: 60-90 min.** Qualidade > cobertura. Se estourar, entrega parcial marcando o que faltou.

---

## 1. Contexto operacional

### 1.1 O que você está validando

O Sprint A tinha um escopo estrito: **scaffolding apenas**. Sem lógica de negócio, sem `HeartbeatDaemon`, sem loader de config.

Entregáveis esperados:
- Estrutura de pastas
- `pyproject.toml` instalável via `pip install -e .`
- Módulos placeholder (sem lógica) com docstrings apontando pra qual sprint implementa cada um
- `__init__.py` exportando 4 símbolos como `None` + `__version__`
- README, CHANGELOG, `.gitignore`
- Commit único pushed

Ver `RUNBOOK_SPRINT_A.md` seção 12 pra lista completa de critérios.

### 1.2 Onde procurar

Três fontes autorizadas:

1. **Repo local:** `~/Trabalho/grupo14d-telemetry/` (ou caminho que o Davi informar)
2. **Remoto:** `https://github.com/A-DAVI/grupo14d-telemetry` (verificar que commit foi pushed)
3. **SDD v1.1:** `SDD_GRUPO14D_TELEMETRY_V1.1.md` — fonte de verdade pra decisões arquiteturais

**Não toque em nenhum outro diretório.** Especialmente: não leia os 15 repos dos RPAs — eles não são afetados nesta sprint.

### 1.3 Como acessar os arquivos

Comandos úteis (somente leitura):

```bash
cd ~/Trabalho/grupo14d-telemetry
ls -la                          # estrutura raiz
find . -type f -not -path './.git/*' | sort   # todos os arquivos
tree -L 3 -I '.git|__pycache__|*.egg-info'    # visão hierárquica
git log --oneline -5            # commits recentes
git show HEAD --stat            # arquivos tocados no último commit
```

Pra leitura:

```bash
cat pyproject.toml
cat src/grupo14d_telemetry/__init__.py
cat src/grupo14d_telemetry/_version.py
# ... etc
```

---

## 2. Checklist de validação (obrigatório)

Passe por cada item. Marque `✅`, `❌` ou `⚠️` (com justificativa).

### 2.1 Estrutura de pastas

Referência: `RUNBOOK_SPRINT_A.md` seção 2.

Execute `find . -type f -not -path './.git/*' | sort` e confira:

- [ ] `pyproject.toml` na raiz
- [ ] `README.md` na raiz
- [ ] `CHANGELOG.md` na raiz
- [ ] `.gitignore` na raiz
- [ ] `src/grupo14d_telemetry/__init__.py`
- [ ] `src/grupo14d_telemetry/_version.py`
- [ ] `src/grupo14d_telemetry/client.py`
- [ ] `src/grupo14d_telemetry/heartbeat.py`
- [ ] `src/grupo14d_telemetry/config.py`
- [ ] `src/grupo14d_telemetry/events.py`
- [ ] `src/grupo14d_telemetry/_internal.py`
- [ ] `tests/__init__.py`
- [ ] `scripts/.gitkeep` (ou algum marcador)
- [ ] `examples/.gitkeep` (ou algum marcador)

**Desvio?** Se algum arquivo extra existe que não estava no runbook, registre. Se falta algum, registre também.

### 2.2 `_version.py`

- [ ] Arquivo define `__version__` como string
- [ ] Valor é exatamente `"0.1.0.dev0"` (não `0.1.0`, não `0.1.0-dev`, não `"0.1.dev0"`)
- [ ] Docstring do módulo presente

### 2.3 `__init__.py`

- [ ] Importa `__version__` de `_version`
- [ ] Define `Telemetry = None`
- [ ] Define `HeartbeatDaemon = None`
- [ ] Define `TelemetryConfig = None`
- [ ] Define `load_config = None`
- [ ] `__all__` lista exatamente os 5 símbolos: `['Telemetry', 'HeartbeatDaemon', 'TelemetryConfig', 'load_config', '__version__']`
- [ ] Docstring do módulo explica que é v0.1 scaffolding

### 2.4 Placeholders dos 5 módulos

Cada módulo (`client.py`, `heartbeat.py`, `config.py`, `events.py`, `_internal.py`) deve:

- [ ] Conter **apenas** docstring — nenhum `import`, nenhuma classe, nenhuma função
- [ ] Docstring indicar qual Sprint implementa (B, C ou D)
- [ ] Listar resumidamente o que vai ser implementado

**Desvio grave:** se algum módulo tem código executável. Isso fere o escopo do Sprint A.

### 2.5 `pyproject.toml`

Referência: `RUNBOOK_SPRINT_A.md` seção 7.

- [ ] `[build-system]` usa `setuptools>=68` e `wheel`
- [ ] `[project]` tem `name = "grupo14d-telemetry"`
- [ ] `dynamic = ["version"]` presente
- [ ] `requires-python = ">=3.11"`
- [ ] `dependencies` contém exatamente `pyyaml>=6.0` (nada a mais, nada a menos)
- [ ] `[project.optional-dependencies]` tem `test = ["pytest>=8.0"]`
- [ ] `[tool.setuptools.dynamic]` aponta pra `grupo14d_telemetry._version.__version__`
- [ ] `[tool.setuptools.packages.find]` usa `where = ["src"]`
- [ ] Classifier `"Private :: Do Not Upload"` presente
- [ ] Autor: Davi Cassoli Lira

### 2.6 `.gitignore`

Mínimo coberto:

- [ ] Python: `__pycache__/`, `*.pyc`, `*.egg-info/`
- [ ] Build: `build/`, `dist/`
- [ ] Venv: `.venv/`, `venv/`
- [ ] Testes: `.pytest_cache/`, `.coverage`
- [ ] IDE: `.vscode/`, `.idea/`, `.DS_Store`
- [ ] Segredos: `.env`, `api_key.txt`, `.secrets/`

### 2.7 `README.md`

- [ ] Status marcado como "v0.1 em desenvolvimento / scaffolding"
- [ ] Seção de instalação dev (`pip install -e .`)
- [ ] Seção de instalação de consumo com `git+https://${GITHUB_PAT}@...` (conforme ADR-06)
- [ ] Lista API pública prevista
- [ ] Exemplo de uso previsto (chama `from_config()` e `HeartbeatDaemon`)
- [ ] Menciona compatibilidade Python ≥3.11 e PyInstaller

### 2.8 `CHANGELOG.md`

- [ ] Seção `[Unreleased]` presente
- [ ] Entrada `[0.1.0.dev0]` com data `2026-04-17` (ou posterior)
- [ ] Formato Keep a Changelog respeitado

### 2.9 Git

```bash
git log --oneline -5
git show HEAD --stat
```

- [ ] Existe commit com mensagem começando com `chore: scaffolding inicial do grupo14d-telemetry`
- [ ] Commit contém todos os arquivos esperados (mínimo 11 arquivos versionados)
- [ ] Commit foi pushed pro remote (`git status` mostra branch sincronizado)
- [ ] Histórico tem **apenas um commit** do Sprint A (não múltiplos)

### 2.10 Validação de instalação

Execute em venv **novo e limpo** (não reuse um venv existente):

```bash
python -m venv /tmp/qa_venv_sprint_a
source /tmp/qa_venv_sprint_a/bin/activate

cd ~/Trabalho/grupo14d-telemetry
pip install -e . 2>&1 | tee /tmp/qa_install.log
```

Valide:

- [ ] Instalação termina com `Successfully installed grupo14d-telemetry-0.1.0.dev0`
- [ ] Zero `ERROR` no log
- [ ] Apenas 1 warning esperado: sobre `Private :: Do Not Upload` (ou nenhum)

Depois:

```bash
python -c "import grupo14d_telemetry; print(grupo14d_telemetry.__version__)"
# esperado: 0.1.0.dev0

python -c "from grupo14d_telemetry import Telemetry, HeartbeatDaemon, TelemetryConfig, load_config, __version__; print('OK')"
# esperado: OK

python -c "import grupo14d_telemetry; print(sorted(grupo14d_telemetry.__all__))"
# esperado: ['HeartbeatDaemon', 'Telemetry', 'TelemetryConfig', '__version__', 'load_config']
```

- [ ] Versão imprime `0.1.0.dev0`
- [ ] Import dos 5 símbolos funciona sem erro
- [ ] `__all__` contém exatamente os 5 esperados

Limpeza:

```bash
deactivate
rm -rf /tmp/qa_venv_sprint_a
```

---

## 3. Caça-bugs independente (obrigatório)

Vá além do checklist. Procure problemas que o runbook **não cobriu** mas que podem morder no Sprint B:

### 3.1 Inconsistências entre arquivos

Compare valores duplicados em múltiplos arquivos — eles precisam bater:

- Versão em `_version.py` vs `CHANGELOG.md` (seção `[0.1.0.dev0]`)
- Python mínimo no `pyproject.toml` (`>=3.11`) vs README
- Nome do pacote: `grupo14d-telemetry` (PyPI, com hífen) vs `grupo14d_telemetry` (import, com underscore) — ambos devem coexistir, confirmar que estão corretos
- URL do repo no `pyproject.toml` vs README

### 3.2 Armadilhas do `pip install -e .`

- [ ] Instalação criou `grupo14d_telemetry.egg-info/` na raiz (esperado) — ele está no `.gitignore`?
- [ ] Após instalação, `python -c "import grupo14d_telemetry; print(grupo14d_telemetry.__file__)"` retorna caminho dentro de `src/` (não em `site-packages`)
- [ ] Mudança manual em `_version.py` (teste: altere para `"0.1.0.dev1"`, reimporte, confirme que vê novo valor — depois reverta)

### 3.3 Resiliência da estrutura

- [ ] `src/grupo14d_telemetry/__init__.py` importa `_version` via caminho **absoluto** (`from grupo14d_telemetry._version import __version__`) e não relativo (`from ._version import ...`)?

  **Por quê importa:** PyInstaller tem histórico de quebrar imports relativos em alguns modos. Absoluto é mais seguro pra v0.1 dado que ADR-05 menciona PyInstaller como constraint.

  Se o Claude Code escolheu relativo, não é bug — é trade-off. Registre na seção de sugestões.

### 3.4 Limpeza

- [ ] Não há arquivos temporários no repo (ex: `.DS_Store`, `__pycache__/`, `*.egg-info/` versionados)
- [ ] `git ls-files` não mostra lixo

### 3.5 Permissões e encoding

- [ ] Todos os arquivos `.py` são UTF-8 sem BOM
- [ ] Terminações de linha consistentes (LF preferível; se houver mistura CRLF+LF, sinalizar)
- [ ] Nenhum arquivo com permissão executável indevida (`.py` não precisa ser executável)

---

## 4. Sugestões arquiteturais pro Sprint B

Este é o valor que você agrega além da validação. O Sprint B vai implementar a classe `Telemetry` + contract_version + testes. Com o SDD v1.1 em mãos, **antecipe problemas** que o Claude Code provavelmente vai encontrar.

### 4.1 Questões a discutir (registrar cada uma no relatório)

Para cada tópico abaixo, produza: **(a) análise curta**, **(b) recomendação concreta**, **(c) grau de prioridade (alta/média/baixa)**.

1. **Nome e assinatura do método `send()` low-level**
   O SDD v1.1 (seção 4.3) define `send(self, event, empresa="", detalhes=None, **extra)`. Isso é bom ou problemático? Considere: (i) typos de `event` não são detectados; (ii) `**extra` permite enviar campos que podem conflitar com o payload obrigatório (`rpa`, `machine`, etc).

2. **Onde `contract_version` é injetado**
   Deve ser injetado no `send()` low-level (automático pra todo evento) ou em cada método de conveniência (start/finish/error)? Qual a implicação pra o Sprint C (HeartbeatDaemon chama `send()` direto)?

3. **Thread safety de `_consecutive_failures` e `_last_error`**
   O SDD 4.6 introduz contadores internos. Se `send()` é chamado em múltiplas threads daemon (caso comum em Tkinter + heartbeat), precisa de `threading.Lock`? Ou aceita race condition (falsos positivos no `is_healthy`)?

4. **Política de renovação de `_consecutive_failures`**
   Quando vira 0 de novo? Após 1 sucesso? Após N sucessos? O SDD diz "zero após sucesso" implicitamente — isso é suficiente ou gera flapping?

5. **Gestão do `session_id` entre execuções**
   O SDD diz UUID4 por instância. Mas: e se o RPA travar e reiniciar em 2s? Dois `session_id` pra mesma "execução do operador"? Considere se faz sentido expor `session_id` como parâmetro pro RPA sobrescrever.

6. **`Telemetry.from_config()` sem config disponível**
   O que acontece se nenhum dos 6 níveis de cascata (SDD 4.5) retornar config válido? Raise? Instância com `enabled=False`? Warning? A decisão precisa estar explícita antes do Sprint B.

7. **Interação com PyInstaller pra leitura de config**
   Quando o binário está empacotado, `config.ini` não está no mesmo diretório do executável. O loader precisa lidar com `sys._MEIPASS` ou derivados? Isso impacta Sprint D mas deve ser antecipado no design de `Telemetry.from_config()`.

8. **Logging: biblioteca padrão vs config customizada**
   `import logging; logger = logging.getLogger(__name__)` é o padrão. Mas RPAs podem ter loggers silenciados. O pacote deve forçar algum handler mínimo? Ou confiar no consumidor? SDD 4.6 menciona WARNING sem dizer onde o WARNING aparece.

9. **Timestamp: ISO 8601 com micro ou sem?**
   `datetime.now(timezone.utc).isoformat()` gera microssegundos por default. MONITOR-RPA aceita (TIMESTAMPTZ tolerante), mas o IRPF antigo tinha `watcher_heartbeat` com formato específico. Confirmar consistência.

10. **Testes no Sprint B — que framework?**
    SDD diz `pytest` (seção 4.1 e 5.3). Mas o Claude Code pode escolher `unittest` por simplicidade ou `pytest + pytest-mock`. Sugerir a stack antes de ele decidir.

### 4.2 Antecipações de PyInstaller

O `.spec` template chega no Sprint B (SDD 5.1). Mas algumas decisões precisam ser tomadas ANTES:

- O pacote deve ter `py.typed` marker? (SDD menciona em `pyproject.toml` — confirmar que arquivo existe ou se é criado no Sprint B)
- `collect-all` vs `collect-submodules` pro PyInstaller — qual recomendar?
- `pyyaml` tem binários nativos (libyaml) — isso pode complicar `.spec` em Windows. Quando/como o template vai cobrir isso?

### 4.3 Convenções de código que não estão no SDD

Decida e sugira:

- Type hints em todo lugar ou só na API pública? (Recomendação: toda pública; privada opcional)
- Formatter: black? ruff format? nenhum?
- Linter: ruff? nenhum? (SDD 1.3 diz "sem CI/CD" mas isso não significa sem linter local)
- Docstrings: Google style? NumPy? Sphinx?

Estes não afetam o Sprint A, mas se não forem decididos antes do Sprint B, cada arquivo vai nascer num estilo diferente.

---

## 5. Formato do relatório

Arquivo único: `/tmp/qa_sprint_a_report.md`

Estrutura obrigatória:

```markdown
# QA Sprint A — grupo14d-telemetry

**Revisor:** Gemini CLI
**Data:** YYYY-MM-DD
**Repositório:** ~/Trabalho/grupo14d-telemetry (commit <hash>)
**Veredito geral:** APROVADO | APROVADO COM RESSALVAS | REPROVADO

## 1. Resumo executivo
(3-5 linhas: o que foi validado, qual o veredito, quantos issues bloqueantes)

## 2. Checklist de validação
### 2.1 Estrutura de pastas
### 2.2 _version.py
### 2.3 __init__.py
### 2.4 Placeholders dos módulos
### 2.5 pyproject.toml
### 2.6 .gitignore
### 2.7 README.md
### 2.8 CHANGELOG.md
### 2.9 Git
### 2.10 Instalação e import

(para cada item: ✅ / ❌ / ⚠️ + evidência com arquivo:linha ou comando executado)

## 3. Bugs encontrados
(lista objetiva; severidade: 🔴 bloqueante, 🟡 importante, ⚪ cosmético)

## 4. Sugestões arquiteturais pro Sprint B
### 4.1 Questões a discutir (10 tópicos da seção 4.1 do runbook)
### 4.2 Antecipações de PyInstaller
### 4.3 Convenções de código

## 5. Itens aprovados sem ressalva
(lista rápida de tudo que passou limpo — isso protege o Claude Code de retrabalho)

## 6. Perguntas pro Davi
(o que só ele pode decidir antes do Sprint B disparar)

## 7. Se aprovado: pré-requisitos pro Sprint B
(lista do que ele precisa ter decidido/preparado antes de disparar o próximo runbook)
```

---

## 6. Critérios de veredito

**APROVADO:** zero bugs bloqueantes, zero desvios do checklist. Pode disparar Sprint B sem corrigir nada.

**APROVADO COM RESSALVAS:** bugs 🟡 ou ⚪, mas nada bloqueante. Lista ações corretivas que podem ser feitas no início do Sprint B ou em paralelo.

**REPROVADO:** pelo menos 1 bug 🔴 bloqueante. Sprint B não dispara até correção. Lista ordem de correção.

---

## 7. O que NÃO fazer

- ❌ Não altere arquivos do repo — apenas leia
- ❌ Não rode o código dos RPAs em produção
- ❌ Não toque em `~/Trabalho/14D-PROJETOS/` (os 15 RPAs)
- ❌ Não altere `~/Trabalho/grupo14d-obsidian-vault/`
- ❌ Não sugira implementações específicas ("use asyncio aqui") — sugira **direcionamentos arquiteturais** ("considere se o modelo síncrono atual é suficiente ou se há benefício em async")
- ❌ Não critique decisões já cristalizadas nas ADRs do SDD v1.1 a menos que tenha evidência nova — essas já passaram por 2 revisões
- ❌ Não termine sem o relatório em `/tmp/qa_sprint_a_report.md`

---

## 8. Critério de pronto

- [ ] Todas as 10 sub-seções da seção 2 foram verificadas
- [ ] Seção 3 (bugs) tem decisão para cada item do checklist
- [ ] Seção 4 (sugestões) cobre os 10 tópicos + PyInstaller + convenções
- [ ] Veredito claro: APROVADO / APROVADO COM RESSALVAS / REPROVADO
- [ ] Relatório em `/tmp/qa_sprint_a_report.md`
- [ ] Nenhum arquivo do repo foi modificado
- [ ] Venv temporário `/tmp/qa_venv_sprint_a` foi removido

---

## 9. Ao terminar

**Pare.** Reporte ao Davi com:

1. Caminho do relatório: `/tmp/qa_sprint_a_report.md`
2. Veredito em uma linha
3. Top 3 issues mais críticos (se houver)

Davi vai decidir se libera Sprint B direto, se corrige antes, ou se trás algum issue pra uma rodada de discussão.

**Não dispare Sprint B nem faça correções.** Seu trabalho é validar, não codar.
