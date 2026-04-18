# RUNBOOK — Auditoria Técnica de **UM** RPA (gera `CONTEXT.md`)

**Para:** Gemini CLI
**Escopo desta sessão:** **EXATAMENTE 1 RPA.** Não comece um segundo. Não agregue.
**Entregável único:** `{repo}/CONTEXT.md` preenchido e salvo em disco.

---

## ⚠️ LEIA ANTES DE COMEÇAR

Uma execução anterior deste trabalho foi **rejeitada** porque o Gemini entregou apenas uma síntese consolidada sem criar os arquivos `CONTEXT.md` reais. Esta versão do runbook é mais rigorosa especificamente pra impedir isso.

**Regras inegociáveis desta sessão:**

1. Você vai analisar **exatamente 1 RPA**. Ao terminar, **pare**. Não "adiante" outro.
2. O **primeiro artefato** que você cria é o `CONTEXT.md` vazio com template, antes de qualquer análise. Isso prova que o arquivo existe.
3. Você preenche o `CONTEXT.md` **seção por seção**, salvando após cada seção. Sem acumular.
4. Ao fim de cada seção, você verifica o arquivo em disco com `wc -l` e reporta no chat — isso prova progresso real.
5. Sem síntese executiva especulativa. Só evidência concreta.
6. Se não conseguir cobrir alguma seção, escreve **"Não analisado nesta sessão — motivo: ..."**. Não omite a seção.

**Você NÃO pode:**
- Fazer resumo no fim em vez de preencher o arquivo
- Analisar outros RPAs "de passagem"
- Pular seções porque "pareciam redundantes"
- Entregar só o que está no chat — o entregável é o arquivo em disco

---

## 0. Contexto

O Grupo 14D tem 15 RPAs sob `/Users/davicassoli/Trabalho/14D-PROJETOS/`. Cada um deve receber um `CONTEXT.md` rigoroso na raiz. Esta sessão cobre 1. Outras sessões vão cobrir os demais.

O Davi informou no início da conversa **qual RPA** analisar. Se não informou, **pare e pergunte** antes de tudo.

---

## 1. Passo 0 — Setup e confirmação (5 min)

### 1.1 Confirme com o Davi qual RPA analisar

No chat, antes de qualquer ação:
```
Vou analisar o RPA: {NOME}. Caminho: /Users/davicassoli/Trabalho/14D-PROJETOS/{NOME}.
Confirma?
```

Aguarde confirmação explícita. Se o Davi não confirmar ou pedir outro, ajuste.

### 1.2 Valide acesso

```bash
cd /Users/davicassoli/Trabalho/14D-PROJETOS/{RPA}
pwd
ls -la | head -30
```

Cole o output no chat. Se o diretório não existir, **pare** e reporte.

---

## 2. Passo 1 — Criar `CONTEXT.md` vazio (obrigatório, 2 min)

**Antes de analisar qualquer código**, crie o arquivo com o template abaixo. Isso é verificado depois.

### 2.1 Se já existir CONTEXT.md

```bash
if [ -f CONTEXT.md ]; then
  mv CONTEXT.md CONTEXT.md.bak.$(date +%Y%m%d-%H%M%S)
  echo "Backup criado"
fi
```

### 2.2 Crie o arquivo novo com template vazio

Escreva **exatamente** este conteúdo em `CONTEXT.md`:

````markdown
# CONTEXT — {NOME-DO-RPA}

> Auditoria técnica gerada por Gemini CLI em {DATA-HORA}.
> Status: **EM PREENCHIMENTO**.

---

## 1. Visão geral
_Pendente de preenchimento._

## 2. Estrutura e fluxo principal
_Pendente de preenchimento._

## 3. Achados 🔴 críticos
_Pendente de preenchimento._

## 4. Achados 🟡 importantes
_Pendente de preenchimento._

## 5. Achados ⚪ cosméticos
_Pendente de preenchimento._

## 6. Performance
_Pendente de preenchimento._

## 7. Build e empacotamento
_Pendente de preenchimento._

## 8. Dependências, testes, logging, segurança
_Pendente de preenchimento._

## 9. Top 5 ações recomendadas
_Pendente de preenchimento._

## 10. Limitações desta auditoria
_Pendente de preenchimento._
````

Substitua `{NOME-DO-RPA}` e `{DATA-HORA}` pelos valores reais.

### 2.3 Valide que o arquivo existe

```bash
ls -la CONTEXT.md
wc -l CONTEXT.md
head -10 CONTEXT.md
```

Cole o output no chat. **Só avance pro Passo 2 depois que o Davi ver isso.**

---

## 3. Passo 2 — Coleta de dados brutos (15 min)

Execute **todos** os comandos abaixo. Salve os outputs num buffer (você vai citar depois). Não analise ainda — só colete.

```bash
echo "=== LOC ==="
find . -name "*.py" -not -path "*/venv/*" -not -path "*/\.*" | xargs wc -l | tail -1

echo "=== Estrutura ==="
find . -type f -name "*.py" -not -path "*/venv/*" -not -path "*/\.*" | sort | head -50

echo "=== Git ==="
git log -1 --format="%h %ci %an %s" 2>/dev/null || echo "Sem git"
git log --since="30 days ago" --oneline 2>/dev/null | wc -l

echo "=== README e docs ==="
ls README* 2>/dev/null
ls *.spec 2>/dev/null
ls *.bat 2>/dev/null
ls requirements*.txt 2>/dev/null

echo "=== TODOs/FIXMEs ==="
grep -rn "TODO\|FIXME\|XXX\|HACK" --include="*.py" . 2>/dev/null | head -30

echo "=== Prints em produção ==="
grep -rn "^\s*print(" --include="*.py" . 2>/dev/null | head -20

echo "=== Bare except ==="
grep -rn "except:\|except Exception" --include="*.py" . 2>/dev/null | head -20

echo "=== Credenciais suspeitas ==="
grep -rniE "password\s*=\s*['\"]" --include="*.py" . 2>/dev/null | head -5
grep -rniE "api_key\s*=\s*['\"]" --include="*.py" . 2>/dev/null | head -5
grep -rniE "token\s*=\s*['\"]" --include="*.py" . 2>/dev/null | head -5

echo "=== Paths hardcoded ==="
grep -rnE "['\"][A-Z]:\\\\" --include="*.py" . 2>/dev/null | head -10
grep -rnE "['\"]\\\\\\\\[a-zA-Z]" --include="*.py" . 2>/dev/null | head -10
grep -rnE "['\"]/home/|'/Users/" --include="*.py" . 2>/dev/null | head -10

echo "=== time.sleep ==="
grep -rn "time\.sleep(" --include="*.py" . 2>/dev/null | head -20

echo "=== requests sem sessão ==="
grep -rn "requests\.\(get\|post\|put\|delete\)" --include="*.py" . 2>/dev/null | head -20

echo "=== Selenium sem WebDriverWait ==="
grep -rn "webdriver\|WebDriver" --include="*.py" . 2>/dev/null | head -10
grep -rn "WebDriverWait\|implicitly_wait" --include="*.py" . 2>/dev/null | head -10

echo "=== pyautogui ==="
grep -rn "pyautogui\|import autogui" --include="*.py" . 2>/dev/null | head -10

echo "=== Sintaxe válida ==="
python -m py_compile $(find . -name "*.py" -not -path "*/venv/*" -not -path "*/\.*") 2>&1 | head -20

echo "=== Ruff (se disponível) ==="
command -v ruff && ruff check . --select=E,F,W --statistics 2>&1 | head -30 || echo "ruff indisponível"

echo "=== Vulture (se disponível) ==="
command -v vulture && vulture . --min-confidence 80 2>&1 | head -20 || echo "vulture indisponível"
```

**Reporte no chat:** "Coleta concluída. Principais observações iniciais: (2-3 linhas)."

---

## 4. Passo 3 — Preencher seção 1 (Visão geral)

Edite o `CONTEXT.md`, substitua a seção 1 por:

````markdown
## 1. Visão geral

| Item | Valor |
|------|-------|
| Propósito | (1-2 linhas baseadas em README ou docstrings do ponto de entrada) |
| Status em produção | Ativo / Legado / Em migração (baseado em git log recente) |
| Stack | Python X.Y, libs principais (do requirements.txt ou imports) |
| Ponto de entrada | `caminho/arquivo.py:função` (identifique o main/run) |
| Build | PyInstaller --onefile ou --onedir (do .spec) / nenhum |
| LOC total Python | N linhas |
| Último commit | {hash} em {data} por {autor}, mensagem: "{msg}" |
| Commits últimos 30 dias | N |
| README presente | Sim / Não / Parcial |
| requirements.txt presente | Sim / Não |

### Fluxo principal (resumo de 3-5 passos)

1. ...
2. ...

### Comando de execução conhecido

```bash
(comando do README/.bat/.spec, ou "não identificado")
```
````

Após salvar, execute e cole no chat:
```bash
wc -l CONTEXT.md
grep -c "Pendente" CONTEXT.md
```

Esperado: contagem de "Pendente" caiu de 10 para 9.

---

## 5. Passo 4 — Preencher seção 2 (Estrutura)

Edite a seção 2:

````markdown
## 2. Estrutura e fluxo principal

### Árvore (até 2 níveis, ignorando venv/.git)

```
arquivo1.py (X linhas) — responsabilidade
src/
├── ui/ — (responsabilidade)
└── core/ — (responsabilidade)
```

### Separação de camadas

(GUI separada do domínio? Adapter pattern? Spaghetti?)

### Observações

- Duplicação detectada: (arquivo:linha ou "nenhuma")
- Pastas vazias/mortas: (listar ou "nenhuma")
- Código morto (via vulture ou inspeção): (listar 3-5 ou "nenhum detectado")
````

Após salvar, `grep -c "Pendente" CONTEXT.md` deve dar 8.

---

## 6. Passo 5 — Preencher seções 3, 4, 5 (Achados por severidade)

Este é o **núcleo do valor**. Baseado na coleta do Passo 2 + leitura do ponto de entrada, liste achados.

**Regras duras pra cada achado:**

- Tem `arquivo.py:linha` real (não "por aí" nem "em algum lugar")
- Tem fix sugerido com código concreto (antes / depois)
- Tem estimativa em minutos ou horas (15m, 1h, 4h, 1d)
- Severidade decidida com base em impacto, não em quantidade

**Severidades:**

- 🔴 **Crítico** — perda de dado, corrupção, vazamento de credencial, travamento em produção
- 🟡 **Importante** — risco de bug, manutenibilidade ruim, performance ruim perceptível
- ⚪ **Cosmético** — melhoria de legibilidade, pequena inconsistência

**Mínimo esperado:** pelo menos 3 achados 🟡 ou 🔴. Se você genuinamente não encontrou nada, registra explicitamente:

> "Após análise de X arquivos e Y LOC, nenhum achado crítico identificado. Justificativa: ..."

### Formato por achado

````markdown
### 3.1 Exceções genéricas silenciando falhas de parsing

- **Localização:** `src/parsers/excel.py:45-52`
- **Evidência:**
  ```python
  try:
      return float(valor)
  except Exception:
      return 0.0
  ```
- **Problema:** Qualquer erro de parsing vira `0.0` sem log. Dados corrompidos ficam invisíveis.
- **Impacto:** Conciliação pode fechar com divergência silenciosa; auditoria fica cega.
- **Fix sugerido:**
  ```python
  try:
      return float(valor)
  except (ValueError, TypeError) as e:
      logger.warning("Falha parsing valor=%r (%s); retornando None", valor, e)
      return None
  ```
- **Esforço:** 30min
````

Preencha seções 3, 4 e 5. Após salvar cada uma, reporte `wc -l CONTEXT.md` no chat.

---

## 7. Passo 6 — Preencher seção 6 (Performance)

````markdown
## 6. Performance

### 6.1 Hotspots detectados

| Localização | Problema | Fix sugerido | Esforço |
|-------------|----------|--------------|---------|

(tabela; se nenhum achado, escreva "Nenhum hotspot óbvio em análise estática.")

### 6.2 I/O e rede

- Sessões `requests` reutilizadas: Sim / Não / N/A — evidência: `arquivo:linha`
- Timeouts em chamadas externas: Sim / Não / Parcial — evidência
- Selenium com `WebDriverWait`: Sim / Não / N/A — evidência
- `pyautogui` com delays: Sim / Não / N/A — evidência

### 6.3 Oportunidades de paralelização

(listar 1-3 ou "nenhuma óbvia")
````

---

## 8. Passo 7 — Preencher seção 7 (Build)

````markdown
## 7. Build e empacotamento

### 7.1 Configuração atual

- Método: PyInstaller `--onefile` / `--onedir` / outro / nenhum (evidência: `.spec` ou `.bat`)
- `.spec` versionado: Sim / Não
- Hidden imports declarados: N (citar exemplos ou "nenhum")
- `collect-all` / `collect-submodules`: Sim / Não (citar)
- `datas` e `binaries`: caminhos relativos / absolutos / não usa
- Script `.bat` de build presente: Sim / Não

### 7.2 Problemas detectados

(lista com evidência; se nenhum, dizer)

### 7.3 Recomendações pra build mais leve

| Ação | Impacto estimado | Esforço |
|------|------------------|---------|
| ex: `--onefile` → `--onedir` | boot 5x mais rápido | 30min |
| ex: `--exclude-module pandas.tests` | −12 MB | 10min |

(mínimo 1 recomendação concreta, ou justificar "build já otimizado")
````

---

## 9. Passo 8 — Preencher seção 8 (Dependências, testes, logging, segurança)

````markdown
## 8. Dependências, testes, logging, segurança

### 8.1 Dependências

- requirements.txt presente: Sim / Não
- Versões pinadas: Sim / Não / Parcial — exemplo de linha
- Total de deps diretas: N
- Deps suspeitas de não-uso (via imports vs requirements): (listar ou "nenhuma")
- Libs pesadas candidatas a remoção: (listar ou "nenhuma")

### 8.2 Testes

- Framework: pytest / unittest / nenhum
- Número de arquivos de teste: N
- Testes batem em rede real: Sim / Não / Não aplicável
- Cobertura do fluxo principal: alta / média / baixa / zero

### 8.3 Logging

- Usa `logging` stdlib: Sim / Não / Misto
- `print()` em produção: N ocorrências (já coletado no Passo 2)
- Configuração centralizada: Sim / Não
- Persistência em arquivo: Sim / Não — caminho: ...

### 8.4 Segurança (checklist)

- [ ] Nenhuma credencial hardcoded (se não, listar com `arquivo:linha`)
- [ ] `.env` no `.gitignore` (Sim / Não)
- [ ] Nenhum `api_key.txt` versionado
- [ ] HTTPS em todas chamadas externas
- [ ] Inputs sanitizados antes de Selenium/pyautogui

### 8.5 Integração com ecossistema 14D

- Reporta telemetria: Sim / Não — `rpa_name=...`
- Formato de config: `config.ini` / `config-global.ini` / `pipeline.yml` / nenhum
- URL do MONITOR-RPA: (conferir valor real, não assumir)
````

---

## 10. Passo 9 — Preencher seção 9 (Top 5)

Baseado em **tudo** acima, eleja 5 ações priorizadas.

**Ordem não é alfabética.** É impacto × custo, começando pelo que o Davi faria primeiro.

````markdown
## 9. Top 5 ações recomendadas (em ordem)

1. **[🔴 ou 🟡] {título curto}** — esforço: Xh, impacto: (1 linha)
2. **[...] {...}** — esforço, impacto
3. ...
````

---

## 11. Passo 10 — Preencher seção 10 (Limitações)

**Honestidade obrigatória.** O que você NÃO cobriu? Por quê?

````markdown
## 10. Limitações desta auditoria

- Tempo total gasto: ~N min
- Ferramentas indisponíveis: (ruff? vulture? pip-audit?) — áreas afetadas
- Arquivos grandes pulados por timebox: (listar)
- Áreas com análise superficial: (listar)
- Dúvidas pro Davi: (listar ou "nenhuma")
````

---

## 12. Passo 11 — Remover o marcador "EM PREENCHIMENTO"

No topo do arquivo, mude:

```
> Status: **EM PREENCHIMENTO**.
```

Para:

```
> Status: **CONCLUÍDO** em {DATA-HORA}.
> Tempo total: ~N min.
> Auditor: Gemini CLI.
```

---

## 13. Validação final (OBRIGATÓRIO antes de reportar)

Execute no chat:

```bash
cd /Users/davicassoli/Trabalho/14D-PROJETOS/{RPA}

echo "=== Arquivo existe? ==="
ls -la CONTEXT.md

echo "=== Linhas totais ==="
wc -l CONTEXT.md

echo "=== Nenhuma seção 'Pendente' sobrou? ==="
grep -c "Pendente de preenchimento" CONTEXT.md
# esperado: 0

echo "=== Todas as 10 seções existem? ==="
grep -c "^## " CONTEXT.md
# esperado: 10

echo "=== Há achados com arquivo:linha? ==="
grep -cE ":[0-9]+" CONTEXT.md
# esperado: >= 3 (cada achado cita pelo menos uma linha)

echo "=== Marcador concluído presente? ==="
grep -c "CONCLUÍDO" CONTEXT.md
# esperado: 1
```

**Cole TODOS os outputs no chat final.** Isso é a prova de vida do relatório.

Se qualquer um falhar (Pendente > 0, seções < 10, etc), **volte e corrija**. Não reporte incompleto.

---

## 14. Reporte final (o que mandar pro Davi)

Quando tudo validado:

```
✅ Auditoria concluída: RPA-{NOME}

Arquivo: /Users/davicassoli/Trabalho/14D-PROJETOS/{NOME}/CONTEXT.md
Tamanho: N linhas
Seções: 10/10 preenchidas
Achados: N 🔴 | N 🟡 | N ⚪
Top ação: {primeira do top-5}
Tempo: ~N min

Outputs de validação:
(colar os 5 blocos do passo 13)

Próximo RPA sugerido: ... (baseado no contexto de ordem)
```

---

## 15. O que NÃO fazer

- ❌ Analisar mais de 1 RPA nesta sessão
- ❌ Gerar síntese executiva "consolidada de vários RPAs"
- ❌ Pular seções porque "parece óbvio" — escreve "Nenhum achado" mas não deixa em branco
- ❌ Reportar conclusão sem rodar o Passo 13 (validação)
- ❌ Inventar `arquivo:linha` — se não tem evidência, não inclui achado
- ❌ Adicionar seção que não está no template (mantém 10 exatas)
- ❌ Alterar qualquer arquivo do repo além do `CONTEXT.md`
- ❌ Executar código do RPA (análise é estática)

---

## 16. Se algo der errado

Se você estourou timebox (>75 min), **pare**:
- Preenche até onde deu
- No Passo 11 (Limitações), marca o que faltou
- No Passo 12 (status), escreve `PARCIAL` em vez de `CONCLUÍDO`
- No reporte final, diz claramente "análise parcial, faltou X"

Se travou em alguma etapa, **avisa o Davi no chat**:
```
BLOQUEADO no Passo N. Motivo: ...
O que fiz até agora: ...
O que preciso: ...
```

Não tenta resolver sozinho se há ambiguidade real.

---

## 17. Uma observação sobre qualidade

Este runbook foi reescrito porque uma versão anterior aceitou síntese superficial como entrega. A diferença desta versão: **artefato primeiro, análise depois**. Crie o arquivo vazio no Passo 1, preencha seção por seção, valide no Passo 13. Se você fizer isso, a qualidade vem naturalmente — porque você não vai conseguir "pular" sem deixar rastro visível no `grep -c "Pendente"`.

Obrigado pelo rigor. Ao final, reporta o que aprendeu de novo sobre esse RPA que não estava no runbook — isso é o que nenhum template captura.
