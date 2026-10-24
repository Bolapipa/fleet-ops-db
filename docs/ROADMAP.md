# 🗺️ Roadmap: guia fase por fase

Cada fase tem:
- **Objetivo:** o que você vai aprender e provar.
- **Tarefas:** o que fazer, na ordem.
- **Entregável:** o que precisa existir no repositório.
- **Pronto quando:** como saber que a fase acabou.
- **Estude:** conceitos e links da documentação oficial.

> ✍️ Regra do projeto: você escreve todo o código. Este guia diz **o que** fazer e **onde pesquisar**, não **como** fazer.
> Ao terminar cada fase: faça commit, marque a issue correspondente e atualize o checklist do README.

Ritmo sugerido: 1 fase por fim de semana. As fases 1, 3 e 4 podem levar 2.

---

## Fase 0: Ambiente

**Objetivo:** subir um PostgreSQL com Docker Compose e conseguir se conectar.

**Tarefas**
1. Siga [SETUP_LINUX_MINT.md](SETUP_LINUX_MINT.md).
2. Crie um `docker-compose.yml` com um serviço PostgreSQL usando a **imagem oficial** `postgres` com a **versão major fixada** (por exemplo `postgres:17`, nunca `latest`).
3. Use um **volume nomeado** para os dados não sumirem ao recriar o container.
4. Coloque usuário, senha e nome do banco (`fleet_ops`) em um arquivo `.env`, que **não vai para o Git**, e crie um `.env.example` sem a senha real, que vai.
5. Adicione um `healthcheck` usando `pg_isready`.
6. Suba o banco e conecte com `psql` e com o DBeaver.

**Entregável:** `docker-compose.yml`, `.env.example`, e uma seção "Como rodar" no README.

**Pronto quando:** `docker compose up -d` sobe o banco, `docker compose ps` mostra `healthy` e `psql` conecta.

**Estude:** imagem oficial do Postgres (https://hub.docker.com/_/postgres), Docker Compose (https://docs.docker.com/compose/), volumes e variáveis de ambiente.

---

## Fase 1: Modelagem relacional

**Objetivo:** transformar a especificação em um modelo relacional normalizado.

**Tarefas**
1. Leia [MODELO_DE_DADOS.md](MODELO_DE_DADOS.md) com calma.
2. Desenhe o **diagrama ER em Mermaid** (`erDiagram`) em `docs/DIAGRAMA_ER.md`, com tabelas, colunas principais e cardinalidades.
3. Responda às 5 perguntas do modelo em `docs/DECISOES.md`.
4. Confira se o modelo está na **3ª forma normal**. Se desnormalizar algo, justifique.
5. Liste, para cada regra de negócio, **qual constraint** vai garanti-la.

**Entregável:** `docs/DIAGRAMA_ER.md` e `docs/DECISOES.md`.

**Pronto quando:** todas as entidades e regras do modelo estão cobertas no diagrama e na lista de constraints.

**Estude:** formas normais (1FN, 2FN, 3FN), cardinalidade, tipos de dados (https://www.postgresql.org/docs/current/datatype.html), constraints (https://www.postgresql.org/docs/current/ddl-constraints.html), Mermaid ER (https://mermaid.js.org/syntax/entityRelationshipDiagram.html).

---

## Fase 2: Migrations versionadas

**Objetivo:** criar o schema por scripts versionados, como código.

**Tarefas**
1. Escolha uma ferramenta: **Flyway** (via Docker) ou um script próprio em Python que aplica os arquivos em ordem e registra os já aplicados em uma tabela de controle. Registre a escolha em `DECISOES.md`.
2. Crie as migrations em `migrations/` com o padrão `V001__create_vessel.sql`, `V002__...`
3. Um arquivo por mudança lógica. **Nunca edite uma migration já aplicada**: crie uma nova.
4. Inclua todas as constraints da Fase 1, incluindo a *exclusion constraint* do embarque.
5. Crie o trigger que atualiza `updated_at` automaticamente.

**Entregável:** pasta `migrations/` completa e instruções de como aplicar no README.

**Pronto quando:** um banco vazio fica com o schema completo com **um único comando**, e todas as regras de negócio são recusadas pelo banco quando violadas. Teste inserindo dados inválidos de propósito.

**Estude:** Flyway (https://documentation.red-gate.com/flyway), `CREATE TABLE`, `ALTER TABLE`, `btree_gist` e exclusion constraints (https://www.postgresql.org/docs/current/btree-gist.html), triggers.

---

## Fase 3: Dados sintéticos em volume

**Objetivo:** popular o banco com dados realistas e em volume.

**Tarefas**
1. Crie um ambiente virtual Python (`python3 -m venv .venv`) e um `requirements.txt` com as versões fixadas.
2. Em `seeds/`, escreva um gerador com **Faker** (locale `pt_BR`) para as tabelas cadastrais: cerca de 30 embarcações, 300 equipamentos, 50 portos, 500 tripulantes, e embarques sem sobreposição.
3. Gere viagens, abastecimentos e ordens de manutenção coerentes, respeitando todas as constraints.
4. Gere **pelo menos 10 milhões** de leituras de sensores. Compare a velocidade de três jeitos: `INSERT` linha a linha, `executemany` e `COPY`. Anote os tempos.
5. Também pesquise como gerar dados direto no SQL com `generate_series`.
6. Torne o gerador **reprodutível**: mesma *seed* gera os mesmos dados.

**Entregável:** `seeds/`, `requirements.txt` e a tabela de tempos de carga em `docs/RESULTADOS.md`.

**Pronto quando:** um comando recria o banco do zero com todos os dados, sem violar nenhuma constraint.

**Estude:** Faker (https://faker.readthedocs.io), psycopg 3 e `COPY` (https://www.psycopg.org/psycopg3/docs/basic/copy.html), `generate_series` (https://www.postgresql.org/docs/current/functions-srf.html).

---

## Fase 4: Performance ⭐ a fase que mais impressiona

**Objetivo:** medir, otimizar e provar com números.

**Tarefas**
1. Escreva em `sql/queries/` pelo menos **6 consultas de negócio**. Por exemplo:
   - leituras de um equipamento nas últimas 24 horas;
   - média diária por métrica no último mês;
   - ordens de manutenção abertas por embarcação e prioridade;
   - consumo de combustível por embarcação e mês;
   - tripulação embarcada em uma data;
   - top 10 equipamentos com mais manutenções corretivas.
2. Rode cada uma com `EXPLAIN (ANALYZE, BUFFERS)` **antes** de qualquer índice e salve o plano em `benchmarks/`.
3. Crie índices adequados: B-tree simples, **compostos**, **parciais** (por exemplo, só ordens abertas) e **BRIN** para a série temporal.
4. **Particione** `sensor_reading` por mês (*declarative partitioning*) e compare.
5. Rode de novo e salve os planos **depois**.
6. Monte em `docs/RESULTADOS.md` uma tabela com a consulta, o tempo antes, o tempo depois, o ganho e o motivo.
7. Também compare o **tamanho** dos índices B-tree e BRIN.

**Entregável:** `sql/queries/`, `benchmarks/` e a tabela de resultados.

**Pronto quando:** cada otimização tem um número de antes e depois e uma explicação do porquê funcionou.

**Estude:** `EXPLAIN` (https://www.postgresql.org/docs/current/using-explain.html), tipos de índice (https://www.postgresql.org/docs/current/indexes-types.html), BRIN (https://www.postgresql.org/docs/current/brin.html), particionamento (https://www.postgresql.org/docs/current/ddl-partitioning.html), `ANALYZE` e estatísticas.

---

## Fase 5: Lógica no banco

**Objetivo:** usar o banco além do `SELECT`.

**Tarefas**
1. Crie **views** para consultas frequentes, como frota atual com situação e tripulação embarcada.
2. Crie uma **materialized view** de indicadores diários de sensores e documente como e quando atualizá-la (`REFRESH ... CONCURRENTLY`).
3. Escreva uma **função PL/pgSQL**, por exemplo "abrir ordem de manutenção" com validações.
4. Crie a tabela `audit_log` e um **trigger de auditoria** genérico que registra o valor antigo e o novo, em JSONB.
5. Demonstre **transações**: um script que mostra `READ COMMITTED` e `REPEATABLE READ` lado a lado em duas sessões, e explique o resultado.
6. Use **window functions** em pelo menos duas consultas, por exemplo média móvel de temperatura e ranking de consumo.

**Entregável:** `sql/views/`, `sql/functions/`, novas migrations e explicações em `docs/`.

**Pronto quando:** cada objeto tem um exemplo de uso documentado.

**Estude:** views e materialized views, PL/pgSQL (https://www.postgresql.org/docs/current/plpgsql.html), triggers, isolamento de transações (https://www.postgresql.org/docs/current/transaction-iso.html), window functions (https://www.postgresql.org/docs/current/tutorial-window.html).

---

## Fase 6: Segurança

**Objetivo:** controlar quem acessa o quê.

**Tarefas**
1. Crie roles: `app_writer` (aplicação), `analyst_ro` (somente leitura) e `etl_reader` (extração da Fase 8).
2. Aplique o **princípio do menor privilégio** com `GRANT` e `REVOKE`, incluindo `ALTER DEFAULT PRIVILEGES`.
3. Implemente **Row-Level Security**: por exemplo, um usuário de uma embarcação só vê as ordens de manutenção dela.
4. Escreva testes que provem que cada role consegue o que deve e **não consegue** o que não deve.

**Entregável:** migrations de segurança e `docs/SEGURANCA.md`.

**Pronto quando:** os testes de permissão passam.

**Estude:** roles e privilégios (https://www.postgresql.org/docs/current/user-manag.html), RLS (https://www.postgresql.org/docs/current/ddl-rowsecurity.html).

---

## Fase 7: Testes, backup e CI

**Objetivo:** automatizar a qualidade, como num projeto de verdade.

**Tarefas**
1. Escreva testes com **pytest** em `tests/` para:
   - as constraints (inserir dados inválidos deve falhar);
   - as funções e os triggers;
   - as permissões da Fase 6.
2. Crie um workflow em `.github/workflows/ci.yml` que, a cada push e PR:
   - sobe um Postgres como *service container*;
   - aplica as migrations;
   - roda os testes.
3. Documente **backup e restore** com `pg_dump` e `pg_restore`, e teste restaurar num banco novo.
4. Adicione o badge do CI no README.

**Entregável:** `tests/`, `ci.yml`, `docs/BACKUP.md` e o badge.

**Pronto quando:** o CI fica verde no GitHub e um restore completo funciona.

**Estude:** pytest (https://docs.pytest.org), service containers no GitHub Actions (https://docs.github.com/actions/using-containerized-services/creating-postgresql-service-containers), `pg_dump` (https://www.postgresql.org/docs/current/app-pgdump.html).

---

## Fase 8: Ponte com o lakehouse

**Objetivo:** ligar este projeto ao seu ponto forte: levar os dados do OLTP para a Bronze.

**Tarefas**
1. **Extração incremental por watermark:** em `extract/`, um script Python que lê só as linhas com `updated_at` maior que o último valor processado e grava em Parquet. Guarde a marca d'água numa tabela ou num arquivo de controle.
2. Trate o problema das **exclusões**: o watermark não as captura. Proponha uma solução, como soft delete ou a tabela de auditoria da Fase 5.
3. **CDC com replicação lógica:** configure `wal_level=logical`, crie uma *publication* e consuma as mudanças, por exemplo com `pgoutput` ou `wal2json`. Opcional e avançado: Debezium.
4. Carregue os arquivos na Bronze de um **Databricks Free Edition** pessoal e monte uma pequena Silver.
5. Compare watermark e CDC: latência, complexidade, exclusões e carga no banco.

**Entregável:** `extract/` e `docs/INTEGRACAO_LAKEHOUSE.md` com o diagrama e a comparação.

**Pronto quando:** uma alteração feita no Postgres aparece na Bronze.

**Estude:** replicação lógica (https://www.postgresql.org/docs/current/logical-replication.html), decodificação lógica (https://www.postgresql.org/docs/current/logicaldecoding.html), Parquet com pyarrow.

> ⚠️ Use só uma conta **pessoal** de Databricks. Nunca o workspace da empresa.

---

## ✅ Ao terminar o projeto

- Atualize o README com resultados reais: tempos, tamanhos e diagramas.
- Grave um GIF curto ou tire prints do DBeaver e do CI verde.
- Adicione o projeto ao portfólio (https://bolapipa.github.io) como um novo job no DAG.
- Escreva um post no LinkedIn sobre o que mais aprendeu na Fase 4.
