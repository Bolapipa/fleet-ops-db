# 🧩 Modelo de dados: especificação

Este documento descreve **o que** o banco precisa guardar e **quais regras** ele precisa garantir. O **como**, ou seja, tabelas, tipos, constraints e índices, é decisão sua na Fase 1.

> 💡 Regra de ouro: toda regra de negócio que puder ser garantida pelo banco (constraint, FK, `CHECK`, `UNIQUE`, exclusão) deve ser garantida pelo banco, e não só pela aplicação.

## Entidades

### 1. Embarcação (vessel)
- Identificador interno.
- **Número IMO**: 7 dígitos, **único**, obrigatório.
- Nome, tipo de embarcação (por exemplo PSV, AHTS, OSRV, rebocador), bandeira, ano de construção.
- Porte bruto (DWT), em toneladas, sempre positivo.
- Situação: em operação, em manutenção, parada, desativada.

### 2. Equipamento (equipment)
- Pertence a **uma** embarcação.
- Tipo (motor principal, motor auxiliar, gerador, bomba, guindaste…), fabricante, modelo, número de série.
- Data de instalação.
- O número de série é único **por fabricante**.

### 3. Porto (port)
- Código (por exemplo, no estilo UN/LOCODE, como `BRRIO`), **único**.
- Nome, cidade, país, latitude e longitude (com limites válidos).

### 4. Tripulante (crew_member)
- Nome, documento (único), função (comandante, imediato, maquinista, marinheiro…), data de admissão.
- Situação: ativo ou desligado.

### 5. Embarque (crew_assignment)
- Liga um **tripulante** a uma **embarcação** por um período (data de embarque e de desembarque).
- O desembarque pode estar em aberto (ainda embarcado).
- ⚠️ **Um tripulante não pode estar embarcado em dois lugares ao mesmo tempo**, ou seja, não pode haver sobreposição de períodos para o mesmo tripulante.
  - 🔎 Pesquise: *exclusion constraint* com `tstzrange` / `daterange` e a extensão `btree_gist`.

### 6. Viagem (voyage)
- Embarcação, porto de origem, porto de destino.
- Partida e chegada, previstas e realizadas.
- A chegada deve ser **depois** da partida.
- Situação: planejada, em andamento, concluída, cancelada.

### 7. Abastecimento (bunkering)
- Embarcação, porto, data e hora.
- Volume em m³ (positivo), tipo de combustível, preço por m³.
- O valor total deve ser **consistente** com volume × preço.
  - 🔎 Pesquise: *generated columns*.

### 8. Ordem de manutenção (maintenance_order)
- Equipamento, tipo (preventiva, corretiva, preditiva), prioridade (1 a 5).
- Abertura, início e conclusão. A conclusão não pode ser antes da abertura.
- Situação: aberta, em execução, concluída, cancelada.
- Custo estimado e custo real (não negativos).
- Uma ordem **concluída** precisa ter data de conclusão.

### 9. Leitura de sensor (sensor_reading), tabela de alto volume
- Equipamento, momento da leitura (`timestamptz`), métrica (temperatura, pressão, rotação, vibração…), valor.
- Vai ter **milhões de linhas** (Fase 3) e será **particionada por mês** (Fase 4).
- Consultas típicas: "leituras do equipamento X nas últimas 24 horas" e "média diária por métrica no último mês".

### 10. Auditoria (audit_log), na Fase 5
- Registra quem alterou o quê e quando nas tabelas principais (valor antigo e novo).

## Regras gerais

- Todas as tabelas têm **chave primária**.
- Datas e horas com fuso: use `timestamptz`.
- Tabelas que vão para o lakehouse (Fase 8) precisam de `created_at` e `updated_at`, e o `updated_at` deve se atualizar sozinho em todo `UPDATE`.
- Valores de domínio fixo (tipos, situações): decida entre `CHECK`, `ENUM` ou tabela de domínio, e **registre o porquê** em `docs/DECISOES.md`.
- Nomes em inglês, `snake_case`, tabelas no singular ou no plural, mas **escolha um padrão e mantenha**.

## Perguntas para responder em `docs/DECISOES.md`

1. Chave primária: `bigint identity` ou `uuid`? Por quê?
2. Domínios: `ENUM`, `CHECK` ou tabela de domínio?
3. Exclusão de registros: física (`DELETE`) ou lógica (`deleted_at`)?
4. Precisão de valores monetários e de volume: `numeric(p, s)` com quais `p` e `s`?
5. O que acontece com as ordens de manutenção se um equipamento for removido (`ON DELETE ...`)?
