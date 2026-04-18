# grupo14d-toolkit (Public Shell)

Hub de documentação, planejamento e governança para o ecossistema de automação do Grupo 14D.

---

## 🚀 O que é este repositório?

Este é um repositório de **"Casca" (Shell)** que centraliza a inteligência de engenharia por trás dos processos de RPA do Grupo 14D. Ele não contém código de produção, servindo como a **Fonte de Verdade** para:

- **Auditoria Técnica:** Manuais e registros de saúde técnica de mais de 15 RPAs ativos.
- **Especificações (SDD):** Definições rigorosas de novos pacotes e contratos de integração.
- **Planejamento de Sprints:** Cronogramas e diretrizes de QA para evolução do ecossistema.
- **Governança Documental:** Sincronização com o Vault de conhecimento (Obsidian).

---

## 📂 Estrutura de Governança

```text
grupo14d-toolkit/
├── .md/                             # Relatórios e Runbooks
│   ├── RUNBOOK_AUDITORIA_15_RPAS.md # Manual de auditoria em lote
│   ├── RUNBOOK_AUDITORIA_1_RPA.md   # Manual de auditoria granular
│   ├── RUNBOOK_QA_SPRINT_*.md       # Checklists de qualidade por Sprint
│   └── SDD_TELEMETRY_V*.md          # Especificações de Software (SDD)
├── CHANGELOG.md                     # Evolução histórica do Toolkit
└── pyproject.toml                   # Definição de stack e dependências globais
```

---

## 🛠️ Metodologia Operacional

### 1. Auditoria Estática & Rigorosa
Utilizamos um processo de auditoria por robôs (AI-driven) que gera um arquivo `CONTEXT.md` para cada projeto, mapeando:
- **Stack Técnica Real** (pandas, Playwright, SQLite, etc.)
- **Dívida Técnica Acumulada** (Silenciamento de erros, paths hardcoded)
- **Top 5 Ações Prioritárias** (Saneamento e robustez)

### 2. Especificação-Driven Development (SDD)
Nenhuma linha de código é escrita na biblioteca central sem uma especificação formal prévia, garantindo que o contrato de telemetria seja respeitado por todos os 15 RPAs.

### 3. Sprints de Evolução (Em andamento)
- **Sprint A:** Scaffolding e Infraestrutura. (Concluído)
- **Sprint B:** Portabilidade e Contratos de Eventos. (Concluído)
- **Sprint C:** Implementação de Heartbeat e Resiliência. (Em execução)
- **Sprint D:** Centralização de Configurações. (Planejado)

---

## 🛡️ Segurança e Privacidade

Este repositório público foi estruturado para demonstrar o **nível de profissionalismo e metodologia** aplicados no Grupo 14D. 
- **Dados Sensíveis:** IPs, credenciais, nomes de parceiros e caminhos de rede internos são estritamente omitidos ou mascarados.
- **Código de Produção:** Mantido em repositórios privados e ambiente de rede restrito.

---

> Auditoria técnica e documentação orquestradas via Gemini CLI.
