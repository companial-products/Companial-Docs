# DCTF — Declaração de Débitos e Créditos Tributários Federais

A DCTF é a declaração federal que informa à Receita Federal quais tributos a empresa apurou no período e como pretende extingui-los (pagamento, compensação, suspensão).

> **Nota**: a Receita Federal substituiu a DCTF pela DCTFWeb. Os débitos de tributos federais que vão para a DCTFWeb fora do eSocial e da EFD-Reinf são informados pelo arquivo MIT, descrito em [MIT — Módulo de Inclusão de Tributos](Localization-for-Brazil_User-Manual_MIT-Modulo-Inclusao-Tributos.md). Este capítulo descreve a DCTF gerada para o programa PGD, que vale só para os períodos anteriores à substituição.

---

## O que é

A DCTF é uma declaração **mensal** que funciona como um "resumo de impostos federais". Para cada tributo apurado, a empresa informa:

- **Quanto** deve (débito)
- **Como** vai pagar (DARF, compensação, suspensão)
- **Quando** pagou ou vai pagar

Os tributos cobertos incluem: IRPJ, IRRF, IPI, IOF, CSLL, PIS/PASEP, COFINS, CIDE, entre outros.

---

## Como a localização implementa

### Fluxo

```
1. CRIAR DECLARAÇÃO
   Informar ano, mês e situação (Normal, Extinção, Cisão, Incorporação)

2. SUGERIR LINHAS
   O sistema lê os movimentos de fornecedores do período
   (Mov. Fornecedor com Jurisdição Imposto e tipo DARF)
   e gera automaticamente:
   - Linhas de débito/crédito (R10) — por código de receita
   - Linhas de pagamento (R11) — DARFs vinculados

3. AJUSTAR
   O usuário pode ajustar valores, adicionar compensações ou suspensões

4. EXPORTAR
   Gerar arquivo no formato oficial para importação no PGD da Receita Federal
```

### Estrutura do arquivo

| Registro | O que contém |
|----------|-------------|
| **R0** | Cabeçalho — dados da empresa |
| **R01** | Configuração — regime de tributação |
| **R02** | Empresa — CNPJ, endereço |
| **R03** | Representante legal e responsável |
| **R10** | Débitos/Créditos — um por código de receita |
| **R11** | Pagamentos — DARFs vinculados a cada débito |
| **T9** | Totalizador |

### Grupos tributários

A DCTF organiza os tributos em 12 grupos: IRPJ, IRRF, IPI, IOF, CSLL, PIS/PASEP, COFINS, CPMF, CIDE, RET, CSRF, COSIRF. O código do grupo é derivado do código de receita do imposto.

---

## Pontos de atenção

- **DARF sem vínculo** — se um DARF foi pago mas não está vinculado à linha de débito na DCTF, o pagamento não será reconhecido pela Receita Federal.
- **Código de receita errado** — cada imposto tem um código específico de receita. Código errado = débito não reconhecido.
- **Declaração retificadora** — para corrigir uma DCTF já transmitida, é necessário gerar uma retificadora com o número do recibo da original.
- **Prazo** — nos períodos em que valia, a DCTF era transmitida até o 15º dia útil do 2º mês subsequente ao período de apuração.

---

## Para praticar

1. Localize a configuração da DCTF no BC (Diga-me > "Configuração DCTF")
2. Abra uma declaração existente e observe as linhas de débito e pagamento
3. Execute a sugestão automática para um mês e confira os valores sugeridos
4. Compare os valores da DCTF com as apurações de IRPJ/CSLL, PIS/COFINS do mesmo período

**Referência**: consulte o manual "DCTF" disponível na seção de [Manuais](Localization-for-Brazil_User-Manual_Contents.md).
