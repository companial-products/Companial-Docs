# ECD — Escrituração Contábil Digital

A ECD é a versão digital dos livros contábeis, transmitida anualmente à Receita Federal. Ela exporta o diário contábil completo do BC no formato SPED.

---

## O que é

A Escrituração Contábil Digital substitui os livros contábeis em papel (Diário Geral, Razão, Balancete). Ela contém:

- **Plano de contas** — completo, com mapeamento para o plano referencial fiscal
- **Movimentos contábeis** — todos os Movimentos Contábeis do período
- **Balancetes** — saldos mensais de todas as contas
- **Demonstrações contábeis** — balanço patrimonial e DRE

### Formato

Arquivo texto SPED (tipo `LECD`), separador pipe, transmitido anualmente. O prazo é definido pela Receita Federal e pode ser prorrogado: confira o prazo vigente no guia prático da ECD.

---

## Estrutura de blocos

| Bloco | Nome | Conteúdo |
|-------|------|---------|
| **0** | Abertura | Dados da empresa, inscrições, filiais, participantes |
| **I** | Lançamentos Contábeis | Plano de contas, saldos mensais, lançamentos do diário |
| **J** | Demonstrações Contábeis | Balanço patrimonial, DRE, DLPA, termos de abertura/encerramento |

### Registros importantes

| Registro | O que contém |
|----------|-------------|
| **I050** | Plano de contas — cada conta com código, descrição, tipo (analítica/sintética) |
| **I051** | Mapeamento para plano referencial — vincula cada conta a um código da Receita Federal |
| **I155** | Saldo mensal — saldo anterior, débitos, créditos, saldo final por conta |
| **I200** | Lançamento contábil — cabeçalho de cada lançamento, com número, data e valor |
| **I250** | Partidas do lançamento — conta, centro de custo, valor e indicador de débito ou crédito |
| **J100** | Balanço patrimonial |
| **J150** | DRE (Demonstrativo de Resultado) |

---

## Como a localização implementa

O módulo ECD:

1. Lê o **plano de contas** do BC (Chart of Accounts)
2. Para cada conta, busca o **mapeamento referencial** (Relação Entre Contas)
3. Exporta todos os **movimentos contábeis** do período
4. Calcula **saldos mensais** por conta
5. Gera as **demonstrações contábeis** (balanço, DRE)
6. Inclui **signatários** (contador, responsável)
7. Exporta o arquivo texto SPED

### Plano de contas referencial

O mapeamento do plano de contas da empresa para o plano referencial fiscal é feito na tabela **Relação Entre Contas** (Relation Between Accounts). Cada conta analítica deve ter um código referencial correspondente.

Esse mapeamento é fundamental porque:
- A Receita Federal usa o plano referencial para validar a ECD
- A ECF importa esse mapeamento para construir seus blocos
- Contas sem mapeamento podem gerar inconsistências

---

## Pontos de atenção

- **Plano de contas referencial incompleto** — contas sem mapeamento para o plano referencial geram alertas no PVA ou inconsistências na ECF.
- **Lançamentos de encerramento** — o sistema deve gerar lançamentos de encerramento do exercício para que os saldos estejam corretos.
- **Signatários** — a ECD exige signatários (contador com CRC e responsável legal). Devem estar cadastrados.
- **Saldos cruzados** — a soma dos saldos iniciais + movimentos = saldo final. Qualquer inconsistência gera erro de validação.

---

## Para praticar

1. Localize a configuração da ECD no BC (Diga-me > "Configuração ECD")
2. Verifique se o plano de contas tem mapeamento referencial (Diga-me > "Relação Entre Contas")
3. Confira os signatários cadastrados
4. Se possível, gere um arquivo ECD de teste e observe a estrutura

**Referência**: consulte o manual "Fiscal ECD" disponível na seção de [Manuais](Localization-for-Brazil_User-Manual_Contents.md).
