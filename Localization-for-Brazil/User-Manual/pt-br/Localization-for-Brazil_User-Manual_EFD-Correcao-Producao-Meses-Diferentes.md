<!-- Commits: 8f87ece5 -->
# EFD ICMS/IPI: Correção de Registros de Produção (K230/K235) entre Meses

**Data**: Maio/2025

---

## Sumário

Esta correção resolve um problema na geração dos registros de produção K230/K250 (itens produzidos) e K235/K255 (itens consumidos) da EFD ICMS/IPI quando a **ordem de produção ou montagem inicia em um mês e é concluída em outro**. Anteriormente, os consumos realizados no mês da apuração para ordens ainda não finalizadas não eram incluídos no bloco K, e as datas de início/fim da produção podiam estar incorretas.

## 1. Contexto

### Registros Envolvidos

| Registro | Descrição |
|----------|-----------|
| K230 | Item produzido — produção própria |
| K235 | Insumo consumido — vinculado ao K230 |
| K250 | Item produzido — produção em terceiros |
| K255 | Insumo consumido — vinculado ao K250 |

Esses registros informam à Receita Federal os detalhes de produção e consumo de materiais no período da apuração fiscal. O K230 contém a data de início e de fim da ordem de produção (`DT_INI_OP` e `DT_FIN_OP`), o K250 contém a data da produção (`DT_PROD`), e os registros K235/K255 listam os insumos consumidos.

### Quando se aplica

Empresas que utilizam **ordens de produção** ou **ordens de montagem** no Business Central e geram o bloco K da EFD ICMS/IPI, especialmente quando ordens de produção se estendem por mais de um mês.

---

## 2. Problema Anterior

### Cenário

Uma ordem de produção inicia em abril e é concluída em maio. A apuração fiscal é de maio.

### Comportamento incorreto

- Os **consumos de maio** para ordens que começaram em abril **não apareciam** no bloco K da apuração de maio, pois o sistema só buscava consumos vinculados a saídas (output) registradas no mesmo período
- As **datas de início e fim** dos registros K230 eram obtidas dos campos do cabeçalho da ordem de produção, que nem sempre refletiam as datas reais dos movimentos de estoque
- Ordens de produção **ainda não finalizadas** no período da apuração não geravam registros K230, mesmo que já tivessem consumos no período

### Impacto

Registros de produção incompletos ou ausentes na EFD ICMS/IPI, podendo gerar inconsistências na validação do arquivo pelo PVA (Programa Validador e Assinador).

---

## 3. Comportamento Corrigido

### Datas de início e fim

As datas de início e fim do registro K230 agora são obtidas diretamente dos **movimentos de estoque** (Item Ledger Entry) da ordem, garantindo que reflitam as datas reais de produção:

- **Data de início**: data do primeiro movimento de estoque da ordem
- **Data de fim**: data do último movimento de estoque da ordem (apenas para ordens finalizadas)

### Consumos de ordens não finalizadas

O sistema agora identifica consumos realizados no período da apuração que pertencem a ordens de produção/montagem **ainda não concluídas**. Para esses casos:

- Um registro K230/K250 é gerado com **quantidade zero** (indicando produção em andamento)
- Os registros K235/K255 de consumo são gerados normalmente, vinculados à ordem em andamento

### Filtro por período nos consumos

Os movimentos de consumo agora são filtrados pelo período da apuração, evitando que consumos de meses anteriores sejam duplicados na apuração atual.
