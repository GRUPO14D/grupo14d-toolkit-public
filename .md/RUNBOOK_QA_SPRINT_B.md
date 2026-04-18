# RUNBOOK QA — Sprint B `grupo14d-telemetry`

**Para:** Gemini CLI (em nova sessão)
**Papel:** Quality Assurance Engineer + Arquiteto Revisor
**Contexto prévio:**
- Você revisou o `SDD_GRUPO14D_TELEMETRY_V0.1.md` (parecer: APROVADO com ressalvas)
- Você validou o `RUNBOOK_SPRINT_A.md` (parecer: APROVADO com ressalvas menores)
- O SDD v1.1 incorporou suas ressalvas
- O Sprint A incorporou o QA que você fez nele
- O **Sprint B** incorporou as 3 diretrizes arquiteturais que você sugeriu: `threading.Lock`, `_send_payload` central, política de erro distinguida

**Agora:** o Claude Code terminou o **Sprint B** seguindo o `RUNBOOK_SPRINT_B.md`. Seu trabalho é validar que a implementação **realmente** segue o design acordado.

---

## ⚠️ MUDANÇA EM RELAÇÃO AO QA DO SPRINT A

No Sprint A você marcou vários itens "✅ Em conformidade" sem mostrar execução. Dessa vez **não aceitamos isso**. Para cada item do checklist, você deve:

1. **Rodar o comando exato** que o runbook pede
2. **Colar o output literal** (ou um trecho relevante) na seção 2
3. **Citar arquivo:linha** pra cada afirmação que depende do código

Itens marcados ✅ sem evidência executada → **inválidos**, precisam ser refeitos. Isso vale pra cada sub-seção do checklist.

Se o comando der erro, cole o erro e marque ❌. Não racionalize falha como "provavelmente está ok".

---

## 0. Princípios

1. **Você NÃO escreve código.** Nem correção, nem patch.
2. **Você NÃO altera arquivos do repo.**
3. **Evidência obrigatória:** comando executado + output literal, ou citação `arquivo:linha`.
4. **Distinga:** desvios do runbook, bugs reais, sugestões arquiteturais (pro Sprint C).
5. **Não invente contexto.** Se o runbook é ambíguo, pergunte ao Davi.
6. **Timebox: 90-120 min.** Qualidade > cobertura. Entregue parcial marcando o que faltou.

---

## 1. Contexto operacional

### 1.1 O que você valida

O Sprint B implementou:
- `client.py` — classe `Telemetry` completa
- `events.py` — enums e validação de `rpa_name`
- `_internal.py` — helpers de UUID, hostname, timestamp
- `__init__.py` atualizado — `Telemetry` real, os outros 3 ainda `None`
- `tests/test_smoke.py`, `tests/test_telemetry_payload.py`, `tests/test_heartbeat_timing.py`, `tests/conftest.py`
- Commit único em branch `feat/sprint-b-client` (sem merge)

### 1.2 Não está no escopo do Sprint B

Esses DEVEM continuar placeholders:
- `heartbeat.py` — docstring apenas
- `config.py` — docstring apenas
- `HeartbeatDaemon` no `__init__` — `None`
- `load_config` no `__init__` — `None`
- `TelemetryConfig` no `__init__` — `None`
- `Telemetry.from_config()` — `raise NotImplementedError`

Se algo desses foi implementado, é **desvio de escopo** — severidade 🟡.

### 1.3 Onde procurar

```bash
cd ~/Trabalho/grupo14d-telemetry
git checkout feat/sprint-b-client
git log --oneline -5
```

Remoto: `https://github.com/A-DAVI/grupo14d-telemetry/tree/feat/sprint-b-client`

Arquivos de referência:
- `SDD_GRUPO14D_TELEMETRY_V1.1.md` — contrato v1 (seções 3-4)
- `RUNBOOK_SPRINT_B.md` — o que foi pedido
- `RUNBOOK_QA_SPRINT_A.md` — seu QA anterior (pra comparação)

---

## 2. Checklist de validação (EVIDÊNCIA OBRIGATÓRIA)

### 2.1 Branch e commit

Comando:
```bash
git branch --show-current
git log --oneline -3
git status
```

**Cole o output literal aqui no relatório.**

Valide:
- [ ] Branch atual é `feat/sprint-b-client`
- [ ] Último commit tem mensagem começando com `feat(telemetry): implementa Telemetry com contrato v1`
- [ ] Commit contém pelo menos: `client.py`, `events.py`, `_internal.py`, `__init__.py`, `tests/conftest.py`, `tests/test_telemetry_payload.py`, `tests/test_heartbeat_timing.py`
- [ ] `git status` limpo (nenhum arquivo não versionado do Sprint B)
- [ ] Branch **não foi mergeada** na main (`git log main..feat/sprint-b-client` mostra commits)

### 2.2 Estrutura de arquivos

Comando:
```bash
find src tests -type f -name "*.py" | sort
wc -l src/grupo14d_telemetry/*.py tests/*.py
```

**Cole o output literal.**

Valide:
- [ ] `src/grupo14d_telemetry/client.py` existe e tem >200 linhas
- [ ] `src/grupo14d_telemetry/events.py` existe com enums
- [ ] `src/grupo14d_telemetry/_internal.py` existe
- [ ] `src/grupo14d_telemetry/heartbeat.py` continua placeholder (docstring apenas)
- [ ] `src/grupo14d_telemetry/config.py` continua placeholder
- [ ] `tests/conftest.py`, `tests/test_smoke.py`, `tests/test_telemetry_payload.py`, `tests/test_heartbeat_timing.py` existem

### 2.3 Heartbeat e Config continuam placeholders

Comandos:
```bash
cat src/grupo14d_telemetry/heartbeat.py
echo "---"
cat src/grupo14d_telemetry/config.py
```

**Cole o output.**

Valide:
- [ ] Ambos contêm **apenas docstring**, nenhum `import`, nenhuma classe, nenhuma função
- [ ] Docstring menciona o sprint que vai implementar (C e D respectivamente)

### 2.4 `__init__.py` — API pública

Comando:
```bash
cat src/grupo14d_telemetry/__init__.py
```

**Cole o output.**

Valide:
- [ ] Importa `Telemetry` de `grupo14d_telemetry.client`
- [ ] `HeartbeatDaemon = None`
- [ ] `TelemetryConfig = None`
- [ ] `load_config = None`
- [ ] `__all__` tem exatamente 5 elementos
- [ ] Importa `__version__` de `_version`

Verificação dinâmica:
```bash
python -c "from grupo14d_telemetry import Telemetry, HeartbeatDaemon, TelemetryConfig, load_config, __version__; print(Telemetry, HeartbeatDaemon, TelemetryConfig, load_config, __version__)"
```

**Cole o output esperado:**
`<class 'grupo14d_telemetry.client.Telemetry'> None None None 0.1.0.dev0`

### 2.5 `events.py` — enums e validação

Comando:
```bash
cat src/grupo14d_telemetry/events.py
```

**Cole o output.**

Valide (com `grep -n` ou inspeção):
- [ ] Regex é **exatamente** `^[a-z0-9]+(-[a-z0-9]+)*$` (cite a linha)
- [ ] Enum `Event` herda de `(str, Enum)` — permite comparação direta com string
- [ ] Enum `Event` tem 8 valores: `RPA_STARTED`, `RPA_FINISHED`, `RPA_ERROR`, `HEARTBEAT`, `AUTOMATION_STARTED`, `AUTOMATION_FINISHED`, `WATCHER_STARTED`, `WATCHER_HEARTBEAT`
- [ ] Enum `Status` tem 4 valores: `SUCCESS`, `ERROR`, `WARNING`, `RUNNING`
- [ ] `validate_rpa_name` **raise** `ValueError` (não log)
- [ ] `warn_unknown_event` **log WARNING** (não raise)

Teste dinâmico:
```bash
python <<'EOF'
from grupo14d_telemetry.events import Event, Status, validate_rpa_name, RPA_NAME_PATTERN

print("Event count:", len(list(Event)))
print("Status count:", len(list(Status)))
print("Regex:", RPA_NAME_PATTERN.pattern)

# Validações — devem passar
for name in ["rpa-irpf", "cartorios", "rebnic-acumuladores", "a1-b2-c3"]:
    validate_rpa_name(name)
    print(f"OK: {name}")

# Devem falhar
for name in ["trots_alt_entrada", "Rebnic-Acumuladores", "-rpa", "rpa-", "rpa--irpf", ""]:
    try:
        validate_rpa_name(name)
        print(f"FALHA NÃO DETECTADA: {name}")
    except ValueError as e:
        print(f"REJEITADO: {name}")
EOF
```

**Cole o output.**

Esperado: 4 OK, 6 REJEITADO, zero "FALHA NÃO DETECTADA".

### 2.6 `_internal.py` — helpers

Comando:
```bash
cat src/grupo14d_telemetry/_internal.py
```

**Cole o output.**

Valide:
- [ ] `new_session_id()` retorna UUID4 completo (36 chars)
- [ ] `get_hostname()` tem fallback pra `"unknown-host"` em exceção
- [ ] `utc_now_iso()` usa `datetime.now(timezone.utc).isoformat()` — timezone explícito

Teste dinâmico:
```bash
python <<'EOF'
from grupo14d_telemetry._internal import new_session_id, get_hostname, utc_now_iso
import uuid

sid = new_session_id()
print("session_id:", sid, "len:", len(sid))
parsed = uuid.UUID(sid)
print("UUID version:", parsed.version)

print("hostname:", get_hostname())

ts = utc_now_iso()
print("timestamp:", ts)
print("tem +00:00?", "+00:00" in ts)
EOF
```

**Cole o output.**

Valide:
- [ ] `session_id` tem 36 caracteres
- [ ] UUID version é 4
- [ ] timestamp contém `+00:00` (UTC explícito)

### 2.7 `client.py` — as 3 diretrizes do QA Sprint A

Essa é **a seção crítica**. Verifique que as 3 diretrizes foram implementadas conforme acordado.

#### 2.7.1 Diretriz 1: `threading.Lock`

Comando:
```bash
grep -n "threading" src/grupo14d_telemetry/client.py
grep -n "_health_lock" src/grupo14d_telemetry/client.py
grep -n "with self._health_lock" src/grupo14d_telemetry/client.py
```

**Cole o output.**

Valide:
- [ ] `threading` é importado
- [ ] `self._health_lock = threading.Lock()` no `__init__`
- [ ] `with self._health_lock:` aparece no mínimo 4 vezes (em `_on_failure`, `_on_success`, `is_healthy` getter, `last_error` getter)
- [ ] Mutações de `_consecutive_failures` e `_last_error` **nunca** ocorrem fora do `with self._health_lock`

#### 2.7.2 Diretriz 2: `_send_payload` central

Comando:
```bash
grep -n "contract_version" src/grupo14d_telemetry/client.py
grep -n "_send_payload" src/grupo14d_telemetry/client.py
```

**Cole o output.**

Valide:
- [ ] `"contract_version": "v1"` aparece **exatamente uma vez** em `client.py` — dentro de `_send_payload`
- [ ] Nenhum método público (`start`, `finish`, `error`, etc) monta payload direto com chaves `"rpa"`, `"event"`, `"session_id"`, etc.
- [ ] Todos os métodos públicos chamam `self._send_payload(...)` (ou chamam `self.send(...)` que delega)

Teste negativo — não deve haver duplicação:
```bash
grep -n '"rpa":' src/grupo14d_telemetry/client.py
grep -n '"event":' src/grupo14d_telemetry/client.py
grep -n '"session_id":' src/grupo14d_telemetry/client.py
```

Cada comando deve retornar **exatamente 1 linha** (a do `_send_payload`). Se aparecer mais, há duplicação — 🔴 BLOQUEANTE.

#### 2.7.3 Diretriz 3: Política de erro

Comando:
```bash
grep -n "HTTPError\|URLError\|logger.warning\|logger.debug" src/grupo14d_telemetry/client.py
```

**Cole o output.**

Valide:
- [ ] `urllib.error.HTTPError` capturado separadamente de `URLError`
- [ ] `HTTPError` com `e.code in (401, 403)` → `logger.warning`
- [ ] Outros `HTTPError` → `logger.warning`
- [ ] `URLError` → `logger.debug`
- [ ] `except Exception` genérico no final → `logger.warning` (swallow + log, nunca raise)

Teste dinâmico — simula auth falha e valida nível de log:
```bash
python <<'EOF'
import logging
import urllib.error
from unittest.mock import patch
from grupo14d_telemetry import Telemetry

logging.basicConfig(level=logging.DEBUG, format='%(levelname)s:%(name)s:%(message)s')

tel = Telemetry("rpa-teste", "http://fake-host-that-does-not-exist-xyz.local:8000")

# 1) erro de rede (URLError — host inexistente)
print("=== Teste 1: URLError esperado, nível DEBUG ===")
tel._post({"event": "test", "contract_version": "v1"})

# 2) simula HTTPError 401
print("\n=== Teste 2: HTTPError 401 esperado, nível WARNING ===")
class FakeHTTPError(urllib.error.HTTPError):
    def __init__(self):
        super().__init__(url="http://x", code=401, msg="Unauthorized", hdrs=None, fp=None)

with patch('grupo14d_telemetry.client.urlopen', side_effect=FakeHTTPError()):
    tel._post({"event": "test", "contract_version": "v1"})

print("\n=== is_healthy após falhas:", tel.is_healthy)
print("last_error:", tel.last_error)
EOF
```

**Cole o output.**

Valide:
- [ ] Teste 1 produz log em nível DEBUG (se não aparecer, `basicConfig` está ok mas lib loga OK)
- [ ] Teste 2 produz log em nível WARNING com "Auth failure"
- [ ] `last_error` não é `None`

### 2.8 `Telemetry.from_config` é stub

Comando:
```bash
grep -n "from_config\|NotImplementedError\|TODO" src/grupo14d_telemetry/client.py
```

**Cole o output.**

Valide:
- [ ] Método `from_config` existe
- [ ] Corpo levanta `NotImplementedError`
- [ ] Docstring tem TODO mencionando Sprint D
- [ ] Docstring tem TODO mencionando PyInstaller / `sys._MEIPASS` (4ª diretriz do QA Sprint A)

Teste:
```bash
python <<'EOF'
from grupo14d_telemetry import Telemetry
try:
    Telemetry.from_config()
except NotImplementedError as e:
    print("OK — NotImplementedError:", e)
EOF
```

**Cole o output.**

### 2.9 Testes passam

Comandos:
```bash
pip install -e '.[test]' 2>&1 | tail -5
pytest tests/ -v 2>&1 | tee /tmp/qa_sprint_b_pytest.log
```

**Cole as últimas ~40 linhas do output.**

Valide:
- [ ] Instalação com `[test]` funciona
- [ ] `pytest` reporta **≥20 testes passando**
- [ ] **ZERO falhas, ZERO erros**
- [ ] Tempo total < 30s (se maior, algum teste está batendo na rede real — 🟡)

### 2.10 Cobertura mínima dos testes obrigatórios

Verifique que cada arquivo tem os testes do runbook:

```bash
grep -c "def test_" tests/test_smoke.py
grep -c "def test_" tests/test_telemetry_payload.py
grep -c "def test_" tests/test_heartbeat_timing.py
```

**Cole o output.**

Valide:
- [ ] `test_smoke.py` tem **≥6** testes
- [ ] `test_telemetry_payload.py` tem **≥15** testes (3 classes * média 5 testes)
- [ ] `test_heartbeat_timing.py` tem **≥3** testes

Verificação específica — os testes cobrem os casos do runbook:
```bash
grep "def test_" tests/test_telemetry_payload.py
```

**Cole o output.**

Confirme presença destes testes (nomes podem variar um pouco):
- [ ] `test_start_payload_has_contract_version`
- [ ] Algum teste validando que `status` e `duracao_segundos` aparecem em `finish`
- [ ] `test_timestamp_is_iso_utc`
- [ ] `test_session_id_is_uuid4`
- [ ] Algum teste rejeitando snake_case
- [ ] Algum teste rejeitando PascalCase
- [ ] Algum teste rejeitando hífen duplo ou inicial

---

## 3. Caça-bugs independente

Vá além do checklist. Procure:

### 3.1 Race conditions não cobertas

O `_send_payload` é chamado em thread main. O `_post` roda em thread daemon. `_on_failure` e `_on_success` são chamados em `_post` (thread daemon). `is_healthy` e `last_error` podem ser lidos pela thread main.

Valide:
- [ ] `_on_failure` e `_on_success` **só** são chamados dentro de `_post` (nunca na thread main)
- [ ] `_post` existe em `client.py` e é o único chamador de `_on_failure`/`_on_success`

Teste de stress (simula contenção):
```bash
python <<'EOF'
import threading
from grupo14d_telemetry import Telemetry

tel = Telemetry("rpa-teste", "http://x", enabled=False)

# 100 threads escrevendo failures, 100 lendo is_healthy
def writer():
    for _ in range(1000):
        tel._on_failure("erro-stress")

def reader():
    for _ in range(1000):
        _ = tel.is_healthy
        _ = tel.last_error

threads = [threading.Thread(target=writer) for _ in range(10)]
threads += [threading.Thread(target=reader) for _ in range(10)]

for t in threads:
    t.start()
for t in threads:
    t.join()

print("_consecutive_failures final:", tel._consecutive_failures)
# Esperado: exatamente 10 * 1000 = 10000
print("is_healthy final:", tel.is_healthy)
print("last_error final:", tel.last_error)
EOF
```

**Cole o output.**

Valide:
- [ ] `_consecutive_failures` final é **exatamente 10000** (qualquer valor menor = race condition)
- [ ] Processo termina sem exceção

### 3.2 Payload JSON-serializável

```bash
python <<'EOF'
import json
from unittest.mock import patch
from grupo14d_telemetry import Telemetry

captured = []
def fake_dispatch(self, p): captured.append(p)

with patch.object(Telemetry, "_dispatch", fake_dispatch):
    tel = Telemetry("rpa-teste", "http://x")
    tel.start(empresa="empresa-com-acento-çãõ")
    tel.finish(status="success", empresa="e", duracao_segundos=3.14)
    tel.error("erro com unicode: 🚨", empresa="e")
    tel.automation_start(empresa="e", detalhes={"records": 10, "nested": {"a": 1}})
    tel.send("heartbeat", empresa="e", detalhes={"cpu": 23.5})

for p in captured:
    try:
        s = json.dumps(p, ensure_ascii=False)
        print(f"OK [{p['event']}] {len(s)} chars")
    except Exception as e:
        print(f"FAIL [{p.get('event')}] {e}")
EOF
```

**Cole o output.** Esperado: 5 × "OK".

### 3.3 Threads daemon

Comando:
```bash
grep -n "daemon=True\|Thread(" src/grupo14d_telemetry/client.py
```

**Cole o output.**

Valide:
- [ ] Toda `threading.Thread(...)` instanciada tem `daemon=True` (compat Tkinter, SDD 4.7)

### 3.4 Rede real nos testes

```bash
grep -rn "urlopen\|Request\|http://\|https://" tests/
```

**Cole o output.**

Valide:
- [ ] Nenhum teste bate em rede real (só URLs fake tipo `http://x`)
- [ ] Nenhum teste depende do MONITOR-RPA estar no ar

### 3.5 Imports limpos

Comando:
```bash
python -c "
import grupo14d_telemetry.client as c
import grupo14d_telemetry.events as e
import grupo14d_telemetry._internal as i
print('client:', sorted(x for x in dir(c) if not x.startswith('_')))
print('events:', sorted(x for x in dir(e) if not x.startswith('_')))
print('_internal:', sorted(x for x in dir(i) if not x.startswith('_')))
"
```

**Cole o output.**

Valide:
- [ ] `client.py` não importa de `heartbeat.py` ou `config.py` (ainda não existem)
- [ ] `events.py` não importa de `client.py` (evita ciclo)
- [ ] `_internal.py` não importa nada do próprio pacote (é folha)

### 3.6 Consistência com o SDD

Releia o SDD v1.1 seção 4.3 (API da classe `Telemetry`) e confirme:

- [ ] Todos os métodos públicos listados existem: `start`, `finish`, `error`, `automation_start`, `automation_finish`, `watcher_started`, `send`, `is_healthy`, `last_error`
- [ ] `is_healthy` e `last_error` são `@property`, não métodos
- [ ] `finish` e `automation_finish` aceitam `duracao_segundos` como parâmetro

### 3.7 Lixo no repo

```bash
git status --ignored
find . -name "__pycache__" -not -path "./.git/*"
find . -name "*.egg-info" -not -path "./.git/*"
find . -name ".pytest_cache" -not -path "./.git/*"
```

**Cole o output.**

Valide:
- [ ] `__pycache__`, `*.egg-info`, `.pytest_cache` **não** versionados (aparecem em `--ignored` ou não aparecem)

---

## 4. Sugestões arquiteturais pro Sprint C

O Sprint C vai implementar `HeartbeatDaemon`. Com o código do Sprint B em mãos, **antecipe problemas**:

### 4.1 Tópicos obrigatórios (responda cada um com análise + recomendação)

1. **Thread daemon dentro de thread daemon**
   `HeartbeatDaemon` vai rodar numa thread. Dentro dela, chama `tel.send("heartbeat")`, que também spawna thread daemon pro `_post`. Isso cria uma hierarquia 2-níveis. Há risco de starvation de threads em RPAs longos? Vale o daemon chamar `_post` diretamente (síncrono, sem spawn)?

2. **Coleta de `detalhes_provider` em contexto de erro**
   Se `detalhes_provider` lançar exceção (ex: `psutil.Process()` falha), o daemon inteiro deve parar ou continuar enviando heartbeat sem detalhes?

3. **Uso do enum `Event.HEARTBEAT`**
   O SDD define tanto `heartbeat` quanto `watcher_heartbeat`. O `HeartbeatDaemon` default deve emitir qual? Deve ser configurável?

4. **`interval_seconds=60` no contexto de teste**
   Testes precisam rodar rápido. O `HeartbeatDaemon` aceita interval < 1s pra testes? Ou o teste é forçado a esperar 60s real?

5. **Múltiplos `HeartbeatDaemon` sobre a mesma `Telemetry`**
   Faz sentido? Se sim, o `session_id` compartilhado é um problema pro servidor distinguir qual daemon está pulsando?

6. **Comportamento no `__exit__` quando há exceção pendente**
   Se o bloco `with HeartbeatDaemon(...)` sai por exceção, o daemon deve enviar um último heartbeat marcando a saída? Ou apenas parar em silêncio?

7. **`stop()` chamado antes de `start()`**
   Deve ser idempotente (no-op)? Raise? O SDD diz "idempotente" — confirmar.

### 4.2 Convenções que ainda estão abertas

1. Formato de docstring usado no Sprint B (Google? NumPy? informal?) — vale fixar ou deixar livre?
2. Type hints estão em 100% da API pública? Os privados também recebem?
3. `logger = logging.getLogger(__name__)` aparece em cada módulo ou só em `client.py`?

---

## 5. Formato do relatório

Arquivo único: `/tmp/qa_sprint_b_report.md`

Estrutura obrigatória:

```markdown
# QA Sprint B — grupo14d-telemetry

**Revisor:** Gemini CLI
**Data:** YYYY-MM-DD
**Branch:** feat/sprint-b-client
**Commit:** <hash>
**Veredito:** APROVADO | APROVADO COM RESSALVAS | REPROVADO

## 1. Resumo executivo
(5-8 linhas: o que foi validado, veredito, quantos issues bloqueantes, destaque do que passou e do que falhou)

## 2. Checklist de validação
### 2.1 Branch e commit
<COLE O OUTPUT DO git log, git status>
Marque ✅/❌/⚠️ com evidência.

### 2.2 Estrutura de arquivos
<COLE O OUTPUT DO find, wc -l>
...

### 2.3 heartbeat.py e config.py continuam placeholders
<COLE O cat DE AMBOS>
...

### 2.4 __init__.py
<COLE O cat e o output do python -c "from ... import ...">
...

### 2.5 events.py
<COLE O cat E O TESTE DINÂMICO DE VALIDAÇÃO>
...

### 2.6 _internal.py
<COLE O cat E O TESTE DINÂMICO>
...

### 2.7 client.py — 3 diretrizes
#### 2.7.1 threading.Lock
<COLE grep DE threading E with self._health_lock>

#### 2.7.2 _send_payload central
<COLE grep DE contract_version E os greps de "rpa":, "event":, etc>

#### 2.7.3 Política de erro
<COLE grep DE HTTPError/URLError + teste dinâmico de log>

### 2.8 from_config stub
<COLE grep E TESTE NotImplementedError>

### 2.9 Testes passam
<COLE AS ÚLTIMAS ~40 LINHAS DO pytest>

### 2.10 Cobertura dos testes obrigatórios
<COLE grep -c "def test_" e grep "def test_">

## 3. Caça-bugs
### 3.1 Race conditions
<COLE O OUTPUT DO STRESS TEST — valide _consecutive_failures == 10000>

### 3.2 Payload JSON-serializável
<COLE OS 5 "OK">

### 3.3 Threads daemon
<COLE grep daemon=True>

### 3.4 Rede real nos testes
<COLE grep>

### 3.5 Imports limpos
<COLE OUTPUT>

### 3.6 Consistência com SDD
(lista dos métodos + confirmação de presença)

### 3.7 Lixo no repo
<COLE OUTPUT DE git status --ignored E find>

## 4. Bugs encontrados
| ID | Severidade | Descrição | Evidência (arquivo:linha ou comando) | Ação sugerida |

Severidades: 🔴 bloqueante, 🟡 importante, ⚪ cosmético

## 5. Sugestões arquiteturais pro Sprint C
### 5.1 Tópicos obrigatórios (7 da seção 4.1 do runbook)
(para cada: análise curta + recomendação + prioridade)

### 5.2 Convenções ainda abertas
(3 perguntas da seção 4.2 com recomendação)

## 6. Itens aprovados sem ressalva
(lista rápida — protege Claude Code de retrabalho)

## 7. Perguntas pro Davi
(o que só ele pode decidir)

## 8. Se aprovado: pré-requisitos pro Sprint C
(checklist do que precisa estar decidido antes do próximo runbook disparar)
```

---

## 6. Critérios de veredito

**APROVADO:**
- Zero bugs 🔴 bloqueantes
- Zero desvios de escopo (heartbeat/config continuam placeholders)
- Todos os 20+ testes passando
- Stress test de 10k falhas bate exatamente `_consecutive_failures == 10000`
- 3 diretrizes arquiteturais claramente implementadas
- Pode disparar Sprint C sem correção

**APROVADO COM RESSALVAS:**
- Bugs 🟡 ou ⚪ que podem ser corrigidos no início do Sprint C ou em paralelo
- Nenhum impacto em correção do contrato v1 ou thread safety

**REPROVADO:**
- Pelo menos 1 bug 🔴 bloqueante. Exemplos de bloqueante:
  - Race condition no stress test (`_consecutive_failures` ≠ 10000)
  - `contract_version` não sai em algum payload
  - `HTTPError 401` loga DEBUG em vez de WARNING
  - Algum método público monta payload sem passar por `_send_payload`
  - `heartbeat.py` ou `config.py` foi implementado (desvio de escopo severo)
  - Testes falhando

---

## 7. O que NÃO fazer

- ❌ Não alterar arquivos do repo
- ❌ Não rodar código dos 15 RPAs em produção
- ❌ Não aceitar "✅" sem ter rodado o comando e colado output
- ❌ Não fazer merge da branch `feat/sprint-b-client`
- ❌ Não criar PR
- ❌ Não sugerir implementações específicas pro Sprint C (só direcionamentos arquiteturais — "considere X" não "implemente Y assim")
- ❌ Não criticar decisões já cristalizadas no SDD v1.1 sem evidência nova

---

## 8. Critério de pronto

- [ ] Todas as 10 sub-seções da seção 2 têm **output literal colado**
- [ ] Seção 3 (caça-bugs) tem os 7 testes rodados com output
- [ ] Seção 3.1 (stress test) reporta `_consecutive_failures` exato
- [ ] Seção 4 (sugestões) cobre os 7 tópicos obrigatórios + 3 convenções
- [ ] Veredito claro com justificativa
- [ ] Relatório em `/tmp/qa_sprint_b_report.md`
- [ ] Venv temporário limpo (se criou algum)
- [ ] Nenhum arquivo do repo modificado

---

## 9. Ao terminar

**Pare.** Reporte ao Davi:

1. Caminho do relatório: `/tmp/qa_sprint_b_report.md`
2. Veredito em uma linha
3. Top 3 issues mais críticos (se houver)
4. Resultado do stress test de thread safety (mais importante item)

**Não dispare Sprint C nem faça correções.** Espere decisão do Davi.
