# 📝 Decisões do projeto

Registre aqui cada decisão importante: **o que** decidiu, **as opções** e **o porquê**. Isso vira ótimo material para entrevistas.

| # | Decisão | Opções consideradas | Escolha | Motivo |
|---|---|---|---|---|
| 1 | Tipo de chave primária | `bigint identity` x `uuid` | | |
| 2 | Valores de domínio | `ENUM` x `CHECK` x tabela de domínio | | |
| 3 | Exclusão de registros | física x lógica (`deleted_at`) | | |
| 4 | Precisão numérica | `numeric(p, s)` | | |
| 5 | `ON DELETE` das ordens de manutenção | `RESTRICT` x `CASCADE` x `SET NULL` | | |
| 6 | Ferramenta de migrations | Flyway x script próprio | | |