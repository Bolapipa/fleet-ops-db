# migrations/

Scripts que criam e evoluem o schema, aplicados **em ordem** (Fase 2).

- Padrão de nome: `V001__create_vessel.sql`, `V002__create_equipment.sql`, ...
- Uma mudança lógica por arquivo.
- **Nunca edite uma migration já aplicada.** Para corrigir, crie uma nova.