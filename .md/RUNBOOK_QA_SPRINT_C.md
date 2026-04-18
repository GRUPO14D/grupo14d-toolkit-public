# RUNBOOK QA — Sprint C `grupo14d-telemetry`

**Para:** Gemini CLI (em nova sessão)
**Papel:** Quality Assurance Engineer + Arquiteto Revisor
**Contexto prévio:**
- Você revisou o `SDD_GRUPO14D_TELEMETRY_V0.1.md` (parecer: APROVADO com ressalvas)
- Você validou os Sprints A e B (ambos APROVADO)
- O Sprint B fechou com 24 testes verdes e stress test de thread safety passando exato (10000/10000)
- O Sprint C incorporou suas sugestões do QA Sprint B: delegação via `telemetry.send()`, blindagem do `detalhes_provider`, `interval_seconds` como float com mínimo 0.1

**Agora:** o Claude Code terminou o **Sprint C** seguindo o `RUNBOOK_SPRINT_C.md`. Seu trabalho é validar que a implementação do `HeartbeatDaemon` está correta, thread-safe e não comprometeu o que o Sprint B entregou.

---

## ⚠️ REGRAS DE EVIDÊNCIA (mesmo padrão do QA Sprint B)

Para cada item do checklist, você deve:

1. **Rodar o comando exato** que o runbook pede
2. **Colar o output literal** (ou trecho relevante)
3. **Citar `arquivo:linha`** pra cada afirmação que depende do código

Itens marcados "✅ Em conformidade" sem comando executado + output colado → **inválidos**.

Se o comando der erro, cole o erro e marque ❌. Não racionalize falha como "provavelmente ok".

---

## 0. Princípios

1. **Você NÃO escreve código.** Nem correção, nem patch.
2. **Você NÃO altera arquivos do repo.**
3. **Evidência obrigatória:** comando + output literal, ou `arquivo:linha`.
4. **Distinga:** desvios do runbook, bugs reais, sugestões arquiteturais (pro Sprint D).
5. **Não invente contexto.** Se o runbook é ambíguo, pergunte ao Davi.
6. **Timebox: 90-120 min.** Entregue parcial marcando o que faltou se estourar.
7. **Regressão do Sprint B é crítico.** O `client.py` **não pode** ter sido tocado.

---

## 1. Contexto operacional

### 1.1 O que você valida

O Sprint C implementou:
- `heartbeat.py` — classe `HeartbeatDaemon` completa
- `__init__.py` atualizado — `HeartbeatDaemon` real, `TelemetryConfig`/`load_config` ainda `None`
- `examples/with_heartbeat.py` — exemplo executável com `psutil`
- `pyproject.toml` — grupo opcional `[examples]` com `psutil`
- `tests/test_heartbeat_daemon.py` — ~20 testes
- Commit único em branch `feat/sprint-c-heartbeat` (sem merge)

### 1.2 Decisões arquiteturais fixadas no Sprint C

Estas foram **decididas pelo Davi antes** do Sprint C executar — valide que foram seguidas:

1. **`event_name` parametrizável** — default `"heartbeat"`, mas aceita `"watcher_heartbeat"` (IRPF legado)
2. **`interval_seconds: float`** com mínimo 0.1 — raise se menor
3. **Delegação via `telemetry.send()`** — daemon não chama `_send_payload` privado
4. **`detalhes_provider` blindado** — exceção no provider não para o daemon, emite heartbeat com `{"provider_error": "..."}`
5. **`threading.Event.wait()`** em vez de `time.sleep()` — stop instantâneo
6. **`start()`/`stop()` idempotentes** com `threading.Lock`, `join()` fora do lock, timeout 2s
7. **`__exit__` não envia evento especial** em exceção pendente — só para o daemon
8. **Múltiplos daemons sobre mesma Telemetry** — permitido tecnicamente, mas não recomendado (só documentar)
9. **Thread com nome `HeartbeatDaemon-<rpa_name>`**

### 1.3 Não está no escopo do Sprint C

Esses DEVEM continuar placeholders:
- `config.py` — docstring apenas
- `TelemetryConfig` no `__init__` — `None`
- `load_config` no `__init__` — `None`
- `Telemetry.from_config()` — `raise NotImplementedError`

E o mais importante:
- **`client.py` NÃO pode ter sido alterado**. Diff contra master vazio pra esse arquivo. Se foi tocado, é desvio grave 🔴.

### 1.4 Onde procurar

```bash
cd ~/Trabalho/grupo14d-telemetry
git checkout feat/sprint-c-heartbeat
git log --oneline -5
```

Remoto: `https://github.com/A-DAVI/grupo14d-telemetry/tree/feat/sprint-c-heartbeat`

---

## 2. Checklist de validação (EVIDÊNCIA OBRIGATÓRIA)

### 2.1 Branch e commit

Comando:
```bash
git branch --show-current
git log --oneline -5
git status
git log main..feat/sprint-c-heartbeat --oneline
```

**Cole o output literal.**

Valide:
- [ ] Branch atual é `feat/sprint-c-heartbeat`
- [ ] Último commit tem mensagem começando com `feat(telemetry): implementa HeartbeatDaemon`
- [ ] `git log main..` mostra **exatamente 1 commit** (Sprint C, não agrupou)
- [ ] `git status` limpo
- [ ] Branch **não mergeada** na main

### 2.2 Regressão Sprint B — `client.py` intocado

**Este é o item mais importante desta seção.** Se o Claude Code alterou `client.py`, é desvio grave.

Comando:
```bash
git diff main -- src/grupo14d_telemetry/client.py
git diff main -- src/grupo14d_telemetry/events.py
git diff main -- src/grupo14d_telemetry/_internal.py
```

**Cole o output literal.**

Valide:
- [ ] `git diff main -- src/grupo14d_telemetry/client.py` retorna **vazio** (ou só mudança trivial como removação de linha em branco)
- [ ] `events.py` e `_internal.py` também intocados ou com alteração mínima justificável
- [ ] Se houver alteração em `client.py`, cole o diff completo e marque 🔴 BLOQUEANTE

Verificação adicional — Sprint B continua passando:
```bash
pip install -e '.[test]'
pytest tests/test_smoke.py tests/test_telemetry_payload.py tests/test_heartbeat_timing.py -v 2>&1 | tail -30
```

**Cole as últimas ~30 linhas.**

Valide:
- [ ] **TODOS** os testes do Sprint B continuam verdes (24 originais)

### 2.3 Arquivos do Sprint C

Comando:
```bash
find src/grupo14d_telemetry tests examples -type f | sort
wc -l src/grupo14d_telemetry/heartbeat.py tests/test_heartbeat_daemon.py examples/with_heartbeat.py 2>/dev/null
```

**Cole o output.**

Valide:
- [ ] `src/grupo14d_telemetry/heartbeat.py` existe e tem >120 linhas
- [ ] `tests/test_heartbeat_daemon.py` existe e tem >150 linhas
- [ ] `examples/with_heartbeat.py` existe
- [ ] `src/grupo14d_telemetry/config.py` **continua placeholder** (sem código executável)

### 2.4 `config.py` ainda é placeholder

Comando:
```bash
cat src/grupo14d_telemetry/config.py
```

**Cole o output.**

Valide:
- [ ] Só docstring, nenhum `import`, nenhuma classe, nenhuma função
- [ ] Docstring menciona Sprint D

### 2.5 `__init__.py`

Comando:
```bash
cat src/grupo14d_telemetry/__init__.py
```

**Cole o output.**

Valide:
- [ ] Importa `Telemetry` de `.client`
- [ ] Importa `HeartbeatDaemon` de `.heartbeat`
- [ ] `TelemetryConfig = None`
- [ ] `load_config = None`
- [ ] `__all__` tem exatamente 5 elementos

Verificação dinâmica:
```bash
python -c "from grupo14d_telemetry import Telemetry, HeartbeatDaemon, TelemetryConfig, load_config, __version__; print(type(Telemetry).__name__, type(HeartbeatDaemon).__name__, TelemetryConfig, load_config, __version__)"
```

**Cole o output.**

Esperado: `type HeartbeatDaemon como 'type', TelemetryConfig e load_config None, __version__ 0.1.0.dev0`

### 2.6 `pyproject.toml` — grupo `[examples]`

Comando:
```bash
grep -A 10 "optional-dependencies" pyproject.toml
```

**Cole o output.**

Valide:
- [ ] Existe `test = ["pytest>=8.0"]`
- [ ] Existe `examples = ["psutil>=5.9"]` (ou similar)
- [ ] `psutil` **NÃO** aparece em `dependencies` principal — só em `[examples]`

### 2.7 `heartbeat.py` — as 9 decisões arquiteturais

Essa é **a seção crítica**. Cada decisão precisa ter evidência.

#### 2.7.1 `event_name` parametrizável com default

Comando:
```bash
grep -n "event_name\|Event.HEARTBEAT" src/grupo14d_telemetry/heartbeat.py
```

**Cole o output.**

Valide:
- [ ] `__init__` aceita parâmetro `event_name`
- [ ] Default é `Event.HEARTBEAT.value` (ou string `"heartbeat"`)
- [ ] `self.event_name` guardado e usado em `_emit_once`

#### 2.7.2 `interval_seconds: float` com mínimo 0.1

Comando:
```bash
grep -n "interval_seconds\|_MIN_INTERVAL\|0\.1" src/grupo14d_telemetry/heartbeat.py
```

**Cole o output.**

Valide:
- [ ] Tipo anotado como `float` no `__init__`
- [ ] Validação `if interval_seconds < 0.1` (ou constante `_MIN_INTERVAL`) presente
- [ ] Raise `ValueError` se menor

Teste dinâmico:
```bash
python <<'EOF'
from grupo14d_telemetry import Telemetry, HeartbeatDaemon

tel = Telemetry("rpa-teste", "http://x")

# Devem passar
for v in [0.1, 0.5, 1.0, 60.0, 300.0]:
    try:
        HeartbeatDaemon(tel, interval_seconds=v)
        print(f"OK: {v}")
    except Exception as e:
        print(f"FALHOU: {v} — {e}")

# Devem falhar
for v in [0.01, 0.0, -1, -0.5]:
    try:
        HeartbeatDaemon(tel, interval_seconds=v)
        print(f"ACEITOU ERRADO: {v}")
    except ValueError:
        print(f"REJEITOU: {v}")
EOF
```

**Cole o output.**

Esperado: 5 OK, 4 REJEITOU, zero "ACEITOU ERRADO".

#### 2.7.3 Delegação via `telemetry.send()`

Comando:
```bash
grep -n "telemetry.send\|self.telemetry\._\|telemetry\._send_payload" src/grupo14d_telemetry/heartbeat.py
```

**Cole o output.**

Valide:
- [ ] `self.telemetry.send(...)` é chamado (API pública)
- [ ] **NENHUMA** chamada a `self.telemetry._send_payload(...)` ou outro método privado (`_dispatch`, `_post`)
- [ ] Se aparecer método privado, é desvio 🟡

#### 2.7.4 `detalhes_provider` blindado

Comando:
```bash
grep -n "_resolve_detalhes\|_emit_once\|provider_error\|detalhes_provider" src/grupo14d_telemetry/heartbeat.py
```

**Cole o output.**

Valide:
- [ ] Método `_resolve_detalhes` existe
- [ ] Chamada ao provider protegida por try/except
- [ ] Em erro, `detalhes` recebe `{"provider_error": "..."}`
- [ ] Daemon continua rodando (não retorna / não levanta)

Teste dinâmico — provider que explode:
```bash
python <<'EOF'
import time
from unittest.mock import patch
from grupo14d_telemetry import Telemetry, HeartbeatDaemon

captured = []
def fake_dispatch(self, p): captured.append(p)

call_count = {"n": 0}
def provider_quebrado():
    call_count["n"] += 1
    raise RuntimeError("provider morto")

with patch.object(Telemetry, '_dispatch', fake_dispatch):
    tel = Telemetry("rpa-teste", "http://x")
    hb = HeartbeatDaemon(
        tel,
        empresa="e",
        interval_seconds=0.1,
        detalhes_provider=provider_quebrado,
    )
    hb.start()
    time.sleep(0.5)
    hb.stop()

print(f"Provider foi chamado {call_count['n']} vezes")
print(f"Heartbeats capturados: {len(captured)}")
for p in captured[:3]:
    print(f"  detalhes={p.get('detalhes')}")
EOF
```

**Cole o output.**

Esperado:
- Provider chamado >= 3 vezes (daemon não parou)
- Heartbeats capturados >= 3
- `detalhes` contém `{"provider_error": "RuntimeError: provider morto"}` em cada

#### 2.7.5 `threading.Event.wait()` em vez de `time.sleep`

Comando:
```bash
grep -n "time\.sleep\|Event\|_stop_event" src/grupo14d_telemetry/heartbeat.py
```

**Cole o output.**

Valide:
- [ ] `self._stop_event = threading.Event()` no `__init__`
- [ ] `self._stop_event.wait(timeout=...)` usado no loop
- [ ] **NENHUM** `time.sleep(...)` no módulo (exceto se houver em código mocked de teste, mas `heartbeat.py` não pode ter)

Teste dinâmico — stop rápido mesmo com interval grande:
```bash
python <<'EOF'
import time
from unittest.mock import patch
from grupo14d_telemetry import Telemetry, HeartbeatDaemon

with patch.object(Telemetry, '_dispatch', lambda self, p: None):
    tel = Telemetry("rpa-teste", "http://x")
    hb = HeartbeatDaemon(tel, interval_seconds=10.0)  # interval grande
    hb.start()
    time.sleep(0.2)  # deixa entrar no wait

    inicio = time.monotonic()
    hb.stop()
    duracao = time.monotonic() - inicio

print(f"stop() duração: {duracao:.3f}s")
assert duracao < 1.0, "stop() não pode demorar mais que 1s"
print("OK — stop responsivo")
EOF
```

**Cole o output.**

Esperado: duração < 1s, última linha "OK — stop responsivo".

#### 2.7.6 `start()`/`stop()` idempotentes + Lock

Comando:
```bash
grep -n "self._lock\|self._started\|with self._lock" src/grupo14d_telemetry/heartbeat.py
```

**Cole o output.**

Valide:
- [ ] `self._lock = threading.Lock()` no `__init__`
- [ ] `with self._lock:` envolve modificação de `self._started`
- [ ] `start()` verifica `self._started` e retorna cedo se já iniciado
- [ ] `stop()` verifica `self._started` e retorna cedo se não iniciado
- [ ] `join()` é chamado **fora** do `with self._lock` (evita deadlock se thread travar)

Teste dinâmico — idempotência:
```bash
python <<'EOF'
from unittest.mock import patch
from grupo14d_telemetry import Telemetry, HeartbeatDaemon

with patch.object(Telemetry, '_dispatch', lambda self, p: None):
    tel = Telemetry("rpa-teste", "http://x")
    hb = HeartbeatDaemon(tel, interval_seconds=0.5)

    # stop antes de start — noop
    hb.stop()
    print(f"stop antes de start: is_running={hb.is_running}")

    hb.start()
    hb.start()  # segunda é no-op
    print(f"depois de 2x start: is_running={hb.is_running}")

    hb.stop()
    hb.stop()  # segunda é no-op
    print(f"depois de 2x stop: is_running={hb.is_running}")
EOF
```

**Cole o output.**

Esperado:
```
stop antes de start: is_running=False
depois de 2x start: is_running=True
depois de 2x stop: is_running=False
```

#### 2.7.7 `__exit__` NÃO suprime exceção

Comando:
```bash
grep -n "__exit__\|__enter__\|return True" src/grupo14d_telemetry/heartbeat.py
```

**Cole o output.**

Valide:
- [ ] `__exit__` chama `self.stop()`
- [ ] `__exit__` **NÃO retorna True** (retorno implícito None permite propagação da exceção)

Teste dinâmico:
```bash
python <<'EOF'
from unittest.mock import patch
from grupo14d_telemetry import Telemetry, HeartbeatDaemon

with patch.object(Telemetry, '_dispatch', lambda self, p: None):
    tel = Telemetry("rpa-teste", "http://x")
    try:
        with HeartbeatDaemon(tel, interval_seconds=0.1):
            raise ValueError("exceção de teste")
    except ValueError as e:
        print(f"OK — exceção propagou: {e}")
    else:
        print("FALHOU — exceção foi suprimida")
EOF
```

**Cole o output.** Esperado: "OK — exceção propagou: exceção de teste".

#### 2.7.8 Múltiplos daemons sobre mesma Telemetry — permitido

Comando:
```bash
grep -n "multiple\|múltipl\|single\|uma única\|uma instância" src/grupo14d_telemetry/heartbeat.py
```

**Cole o output.**

Valide:
- [ ] Docstring menciona que múltiplos daemons são permitidos tecnicamente mas não recomendados
- [ ] **NENHUM** bloqueio no código (registry, singleton, assert) impede múltiplas instâncias

Teste dinâmico — dois daemons simultâneos não raise:
```bash
python <<'EOF'
import time
from unittest.mock import patch
from grupo14d_telemetry import Telemetry, HeartbeatDaemon

captured = []
with patch.object(Telemetry, '_dispatch', lambda self, p: captured.append(p)):
    tel = Telemetry("rpa-teste", "http://x")
    hb1 = HeartbeatDaemon(tel, empresa="e", interval_seconds=0.15)
    hb2 = HeartbeatDaemon(tel, empresa="e", interval_seconds=0.15)

    hb1.start()
    hb2.start()
    time.sleep(0.5)
    hb1.stop()
    hb2.stop()

print(f"Total heartbeats com 2 daemons: {len(captured)}")
EOF
```

**Cole o output.** Esperado: >= 4 (ambos emitem sem raise).

#### 2.7.9 Nome da thread

Comando:
```bash
grep -n "name=f\|name=\"\|Thread(" src/grupo14d_telemetry/heartbeat.py
```

**Cole o output.**

Valide:
- [ ] `threading.Thread(... name=f"HeartbeatDaemon-{...}")` presente
- [ ] Thread criada é `daemon=True`

Teste dinâmico:
```bash
python <<'EOF'
import threading
import time
from unittest.mock import patch
from grupo14d_telemetry import Telemetry, HeartbeatDaemon

with patch.object(Telemetry, '_dispatch', lambda self, p: None):
    tel = Telemetry("rpa-teste", "http://x")
    hb = HeartbeatDaemon(tel, interval_seconds=0.5)
    hb.start()
    time.sleep(0.1)

    nomes = [t.name for t in threading.enumerate() if "Heartbeat" in t.name]
    print(f"Threads com nome Heartbeat: {nomes}")
    hb.stop()
EOF
```

**Cole o output.** Esperado: `['HeartbeatDaemon-rpa-teste']` (ou similar com o rpa_name).

### 2.8 Testes passam

Comando:
```bash
pytest tests/ -v 2>&1 | tee /tmp/qa_sprint_c_pytest.log | tail -50
```

**Cole as últimas ~50 linhas.**

Valide:
- [ ] **≥44 testes passando** (24 do Sprint B + 20 do Sprint C)
- [ ] **ZERO falhas, ZERO erros**
- [ ] Tempo total < 30s (se > 30s, algum teste espera real time.sleep longo — 🟡)

### 2.9 Cobertura dos testes obrigatórios

Comando:
```bash
grep -c "def test_" tests/test_heartbeat_daemon.py
grep "def test_" tests/test_heartbeat_daemon.py
```

**Cole o output.**

Valide:
- [ ] `test_heartbeat_daemon.py` tem **≥20 testes**
- [ ] Presença de testes cobrindo cada classe/área:
  - `TestInit` (validação de params)
  - `TestTiming` (emissão de pulsos)
  - `TestIdempotency` (start/stop repetidos)
  - `TestStopResponsiveness` (stop rápido)
  - `TestDetalhesProvider` (incluindo provider que quebra)
  - `TestContextManager` (não suprime exceção)
  - `TestIsRunning`
  - `TestThreadSafety` (start/stop concorrentes)

---

## 3. Caça-bugs independente

### 3.1 Stress test — start/stop concorrentes (caso Tkinter real)

Este é o cenário mais perigoso: RPA Tkinter onde thread main e `WM_DELETE_WINDOW` podem chamar stop simultâneo.

```bash
python <<'EOF'
import threading
import time
from unittest.mock import patch
from grupo14d_telemetry import Telemetry, HeartbeatDaemon

with patch.object(Telemetry, '_dispatch', lambda self, p: None):
    tel = Telemetry("rpa-teste", "http://x")
    hb = HeartbeatDaemon(tel, interval_seconds=0.1)

    errors = []

    def worker_start_stop():
        try:
            for _ in range(100):
                hb.start()
                hb.stop()
        except Exception as e:
            errors.append(f"{type(e).__name__}: {e}")

    threads = [threading.Thread(target=worker_start_stop) for _ in range(10)]
    inicio = time.monotonic()
    for t in threads:
        t.start()
    for t in threads:
        t.join(timeout=30)
    duracao = time.monotonic() - inicio

print(f"Duração: {duracao:.2f}s")
print(f"Threads vivas após join: {[t.name for t in threads if t.is_alive()]}")
print(f"Errors: {errors[:5]}")
print(f"Estado final is_running: {hb.is_running}")
assert not any(t.is_alive() for t in threads), "alguma worker travou"
assert len(errors) == 0, "houve exceção em start/stop concorrente"
print("OK — sem deadlock, sem exceção em concorrência")
EOF
```

**Cole o output.**

Valide:
- [ ] Duração < 30s (sem deadlock)
- [ ] Zero threads vivas após join
- [ ] Zero exceções
- [ ] `is_running` final é `False`

Se travar ou der exceção: 🔴 BLOQUEANTE.

### 3.2 Stress test — volume de heartbeats com Lock em uso

Valida que o Lock de `HeartbeatDaemon` não conflita com o Lock de `Telemetry` (herdado do Sprint B).

```bash
python <<'EOF'
import time
from unittest.mock import patch
from grupo14d_telemetry import Telemetry, HeartbeatDaemon

captured = []
with patch.object(Telemetry, '_dispatch', lambda self, p: captured.append(p)):
    tel = Telemetry("rpa-teste", "http://x")

    # 5 daemons paralelos com interval baixo
    daemons = [
        HeartbeatDaemon(tel, empresa=f"e{i}", interval_seconds=0.1)
        for i in range(5)
    ]

    inicio = time.monotonic()
    for d in daemons:
        d.start()
    time.sleep(2.0)  # 2s de execução
    for d in daemons:
        d.stop()
    duracao = time.monotonic() - inicio

print(f"Duração total: {duracao:.2f}s")
print(f"Heartbeats capturados: {len(captured)}")
print(f"Todos parados? {all(not d.is_running for d in daemons)}")
# Esperado: ~5 daemons * 2s / 0.1s = ~100 heartbeats (±20)
EOF
```

**Cole o output.**

Valide:
- [ ] Duração total próxima de 2s (não travou)
- [ ] Heartbeats capturados entre 60 e 120 (tolerância razoável)
- [ ] Todos os daemons parados ao fim

### 3.3 Memory / thread leak — 100 ciclos start/stop

```bash
python <<'EOF'
import threading
from unittest.mock import patch
from grupo14d_telemetry import Telemetry, HeartbeatDaemon

with patch.object(Telemetry, '_dispatch', lambda self, p: None):
    tel = Telemetry("rpa-teste", "http://x")

    threads_antes = threading.active_count()
    hb = HeartbeatDaemon(tel, interval_seconds=0.1)

    for i in range(100):
        hb.start()
        hb.stop()

    # Pequena espera pros joins completarem
    import time
    time.sleep(0.5)
    threads_depois = threading.active_count()

    print(f"Threads antes: {threads_antes}")
    print(f"Threads depois: {threads_depois}")
    print(f"Diferença: {threads_depois - threads_antes}")
    # Esperado: diferença <= 1 (thread residual da última pode ainda estar no join)
EOF
```

**Cole o output.**

Valide:
- [ ] Diferença de threads <= 1 (não houve leak)
- [ ] Se > 5, é bug 🟡

### 3.4 Regressão — `is_healthy` de Telemetry continua funcionando

```bash
python <<'EOF'
from grupo14d_telemetry import Telemetry

tel = Telemetry("rpa-teste", "http://x", enabled=False)
print("is_healthy inicial:", tel.is_healthy)
print("last_error inicial:", tel.last_error)

for _ in range(10):
    tel._on_failure("erro regressão")
print("após 10 falhas:", tel.is_healthy, tel.last_error)

tel._on_success()
print("após sucesso:", tel.is_healthy, "last_error persiste:", tel.last_error)
EOF
```

**Cole o output.**

Valide:
- [ ] `is_healthy` inicial: True
- [ ] Após 10 falhas: False
- [ ] Após sucesso: True, mas `last_error` mantém mensagem (não zera)

### 3.5 Imports limpos — sem ciclo

Comando:
```bash
python -c "
import grupo14d_telemetry.heartbeat as h
import grupo14d_telemetry.client as c
import grupo14d_telemetry.events as e
print('heartbeat imports:', sorted(x for x in dir(h) if not x.startswith('_')))
"
```

**Cole o output.**

Valide:
- [ ] `heartbeat.py` usa `TYPE_CHECKING` pra importar `Telemetry` (ou só usa como string no type hint)
- [ ] Sem ciclo `client → heartbeat → client`

Comando adicional:
```bash
grep -n "TYPE_CHECKING\|from grupo14d_telemetry" src/grupo14d_telemetry/heartbeat.py
```

### 3.6 Exemplo executável — `with_heartbeat.py`

Comando:
```bash
cat examples/with_heartbeat.py
```

**Cole o output.**

Valide:
- [ ] Importa `psutil` (coerente com `[examples]` em pyproject.toml)
- [ ] Usa `HeartbeatDaemon` com `detalhes_provider` real
- [ ] Tem `if __name__ == "__main__":`

Não rode o exemplo (requer MONITOR-RPA). Só valide estrutura.

### 3.7 Lixo no repo

Comando:
```bash
git status --ignored
find . -name "__pycache__" -not -path "./.git/*" -not -path "./.venv/*" 2>/dev/null
find . -name ".pytest_cache" -not -path "./.git/*" 2>/dev/null
```

**Cole o output.**

Valide:
- [ ] `__pycache__`, `.pytest_cache` não versionados
- [ ] `.egg-info` só aparece no ignored

### 3.8 Consistência com SDD

Releia SDD v1.1 seção 4.4 e confirme presença:

- [ ] `HeartbeatDaemon(telemetry, empresa, interval_seconds, detalhes_provider, event_name)` — assinatura completa
- [ ] `start()`, `stop()`, `__enter__`, `__exit__`
- [ ] Property `is_running`
- [ ] Uso documentado no docstring com exemplo Tkinter (`WM_DELETE_WINDOW` mencionado? Não obrigatório mas bom)

---

## 4. Sugestões arquiteturais pro Sprint D

Sprint D vai implementar `load_config` + `TelemetryConfig`. Com o Sprint C em mãos, antecipe:

### 4.1 Tópicos obrigatórios (responda cada um)

Para cada tópico: **análise curta + recomendação + prioridade**.

1. **Cascata de 6 níveis do SDD 4.5 é realista?**
   SDD lista: args → env → config.ini → config-global.ini → pipeline.yml → arquivos de chave. Isso vira 6 tentativas de abrir arquivo por instanciação. Overkill ou necessário? Poderia virar 3 (args → env → primeiro arquivo encontrado)?

2. **YAML support — vale adicionar `pyyaml` como dep principal?**
   Só 1 RPA usa YAML (TROTS-IMPORT). Alternativa: `TelemetryConfig.from_yaml()` separado, `pyyaml` só no grupo `[yaml]`. Prós e contras?

3. **PyInstaller + `sys._MEIPASS`**
   Quando empacotado, `config.ini` está onde? Relativo ao executável? No MEIPASS? Sugerir padrão de resolução.

4. **`Telemetry.from_config(config_path=None)` — semântica do None**
   `None` = auto-descobrir (cascata completa). Path explícito = só aquele arquivo. Comportamento quando auto-descoberta não encontra: raise? enabled=False? warning? O SDD menciona mas não é explícito.

5. **Validação estrita vs permissiva no `TelemetryConfig`**
   Se `config.ini` tem `rpa_name = INVALIDO_PASCAL`, a validação kebab-case (do Sprint B) vai raise na instanciação. `TelemetryConfig` deve validar antes (fail fast) ou deixar o `Telemetry.__init__` falhar?

6. **Reconciliação com thread safety do Sprint C**
   `HeartbeatDaemon` já tem seu Lock. `Telemetry` tem outro. `TelemetryConfig` vai ter?  Ou é imutável (dataclass frozen)?

7. **Env vars específicas vs genéricas**
   SDD propõe `RPA_API_KEY`, `MONITOR_RPA_URL`, `RPA_NAME`. Isso pode colidir com outras libs que definem `RPA_*`. Considerar prefixo `G14D_TELEMETRY_*`?

### 4.2 Convenções ainda abertas

1. Dataclass vs classe normal para `TelemetryConfig`? (Sugestão: `@dataclass(frozen=True, slots=True)` — imutável, thread-safe)
2. `load_config()` retorna dict ou `TelemetryConfig`? (API mais limpa?)
3. Logger em `config.py` — mesmo padrão `logging.getLogger(__name__)` dos outros módulos?

---

## 5. Formato do relatório

Arquivo único: `/tmp/qa_sprint_c_report.md`

Estrutura obrigatória:

```markdown
# QA Sprint C — grupo14d-telemetry

**Revisor:** Gemini CLI
**Data:** YYYY-MM-DD
**Branch:** feat/sprint-c-heartbeat
**Commit:** <hash>
**Veredito:** APROVADO | APROVADO COM RESSALVAS | REPROVADO

## 1. Resumo executivo
(5-8 linhas)

## 2. Checklist de validação
### 2.1 Branch e commit
<COLE output de git log/status>

### 2.2 Regressão — client.py intocado
<COLE output de git diff main -- ...>
<COLE últimas linhas do pytest Sprint B>

### 2.3 Arquivos do Sprint C
<COLE output de find/wc>

### 2.4 config.py placeholder
<COLE cat>

### 2.5 __init__.py
<COLE cat + python -c>

### 2.6 pyproject.toml
<COLE grep>

### 2.7 heartbeat.py — 9 decisões
#### 2.7.1 event_name parametrizável
<COLE grep + teste dinâmico>

#### 2.7.2 interval_seconds float com mínimo
<COLE grep + teste 5 OK / 4 REJEITOU>

#### 2.7.3 Delegação via telemetry.send()
<COLE grep>

#### 2.7.4 detalhes_provider blindado
<COLE grep + teste provider quebrado>

#### 2.7.5 threading.Event.wait
<COLE grep + teste stop rápido>

#### 2.7.6 start/stop idempotentes + Lock
<COLE grep + teste idempotência>

#### 2.7.7 __exit__ não suprime
<COLE grep + teste exceção propaga>

#### 2.7.8 Múltiplos daemons permitidos
<COLE grep + teste 2 daemons>

#### 2.7.9 Nome da thread
<COLE grep + teste threading.enumerate>

### 2.8 Testes passam
<COLE últimas ~50 linhas do pytest>

### 2.9 Cobertura de testes obrigatórios
<COLE grep -c + grep "def test_">

## 3. Caça-bugs
### 3.1 Stress start/stop concorrentes
<COLE output completo — é o item mais crítico>

### 3.2 Stress volume de heartbeats com 5 daemons
<COLE output>

### 3.3 Leak check — 100 ciclos
<COLE output>

### 3.4 Regressão is_healthy Sprint B
<COLE output>

### 3.5 Imports limpos
<COLE output>

### 3.6 Exemplo executável
<COLE cat de with_heartbeat.py>

### 3.7 Lixo no repo
<COLE output>

### 3.8 Consistência com SDD
(checklist marcado)

## 4. Bugs encontrados
| ID | Severidade | Descrição | Evidência | Ação sugerida |

## 5. Sugestões arquiteturais pro Sprint D
### 5.1 Tópicos obrigatórios (7)
(cada um: análise + recomendação + prioridade)

### 5.2 Convenções abertas (3)

## 6. Itens aprovados sem ressalva

## 7. Perguntas pro Davi

## 8. Pré-requisitos pro Sprint D (se APROVADO)
```

---

## 6. Critérios de veredito

**APROVADO:**
- Zero bugs 🔴 bloqueantes
- `client.py` intocado (`git diff main` vazio)
- Todos os 24 testes do Sprint B continuam verdes
- `HeartbeatDaemon` passa nos 9 testes dinâmicos das decisões arquiteturais
- Stress test de start/stop concorrentes não travou nem exceção
- Leak check: diferença de threads ≤ 1
- ≥44 testes totais passando (24 + 20)

**APROVADO COM RESSALVAS:**
- Bugs 🟡 ou ⚪ corrigíveis em paralelo
- Alguma decisão arquitetural implementada levemente diferente mas equivalente

**REPROVADO:**
Pelo menos 1 dos abaixo:
- `client.py` foi alterado (desvio de escopo 🔴)
- Stress test de concorrência travou ou deu exceção
- Algum teste do Sprint B quebrou
- `HeartbeatDaemon` chama método privado de `Telemetry` em vez de `send()`
- `detalhes_provider` que raise **para** o daemon
- `stop()` demora >1s com interval grande
- `__exit__` suprime exceção do bloco `with`
- `heartbeat.py` ou `config.py` tem código executável fora do escopo

---

## 7. O que NÃO fazer

- ❌ Não altere arquivos do repo
- ❌ Não rode `with_heartbeat.py` (requer MONITOR-RPA)
- ❌ Não aceite "✅" sem ter rodado o comando e colado output
- ❌ Não faça merge da branch
- ❌ Não crie PR
- ❌ Não sugira implementações específicas pro Sprint D — só direcionamentos
- ❌ Não critique ADRs já cristalizadas no SDD v1.1

---

## 8. Critério de pronto

- [ ] Todas as sub-seções da seção 2 têm **output literal colado**
- [ ] Seção 2.2 (regressão Sprint B) confirmada com `git diff` vazio em `client.py`
- [ ] Seção 2.7 cobre as 9 decisões com grep + teste dinâmico cada
- [ ] Seção 3.1 (stress concorrência) reporta duração, threads vivas, erros
- [ ] Seção 3.3 (leak check) reporta diferença de threads
- [ ] Seção 5 (sugestões pro Sprint D) cobre os 7 tópicos + 3 convenções
- [ ] Veredito claro com justificativa
- [ ] Relatório em `/tmp/qa_sprint_c_report.md`
- [ ] Nenhum arquivo do repo modificado

---

## 9. Ao terminar

Pare. Reporte ao Davi:

1. Caminho do relatório: `/tmp/qa_sprint_c_report.md`
2. Veredito em 1 linha
3. **Top 3 issues mais críticos** (se houver)
4. Resultado específico do stress test de concorrência (seção 3.1)
5. Confirmação se `client.py` foi tocado ou não (seção 2.2)

**Não dispare Sprint D nem faça correções.** Espere decisão do Davi.
