<!-- PR #32 -->

# Aplicação de Adiantamento: Cálculo de Impostos Retidos

**Data**: Abril/2026

---

## Sumário

Esta atualização adiciona o cálculo e a contabilização de **impostos retidos** no momento em que um adiantamento é **aplicado** a uma fatura, e o **estorno** correspondente quando a aplicação é desfeita. A funcionalidade é controlada por um novo parâmetro em **Configurações Fiscais**, o que permite habilitá-la por empresa sem afetar bases existentes.

Antes desta mudança, retenções (IRRF, INSS, ISS, PIS/COFINS/CSLL) eram registradas apenas no posting direto do pagamento. Agora, quando há retenção a registrar e a aplicação ocorre fora do registro do pagamento (telas **Aplicar Movtos. Cliente** e **Aplicar Movtos. Fornecedor**, ou aplicação posterior), o sistema também gera os movimentos contábeis dos impostos retidos.

## 1. Contexto

### O que é

Em operações com **adiantamento** (prepayment), o documento original (fatura, nota fiscal de serviço, etc.) traz retenções calculadas. Quando o adiantamento é aplicado ao documento depois do registro do recebimento/pagamento, o valor líquido financeiro precisa contemplar os impostos retidos — caso contrário, o saldo do cliente ou fornecedor fica incorreto e a contabilização das retenções permanece pendente.

### O que muda

- Ao **aplicar** um adiantamento a uma fatura com retenções, o sistema:
  - Calcula o valor líquido das retenções financeiras
  - Ajusta o **Valor Aplicado** para refletir esse líquido
  - Registra os movimentos de impostos no momento da aplicação (Conta a Pagar, Despesa, Crédito de Imposto)
  - Vincula os movimentos de impostos ao movimento contábil do adiantamento
- Ao **desaplicar** (estornar a aplicação), os movimentos de impostos gerados são **estornados** automaticamente.

### Quando se aplica

- Apenas para aplicações realizadas **fora do registro direto** do pagamento (ex.: aplicação posterior pela página **Aplicar Movtos. Cliente** ou **Aplicar Movtos. Fornecedor**).
- Apenas em operações na **moeda local** (aplicações entre moedas diferentes não acionam o cálculo).
- Apenas quando o documento aplicado tem retenções calculadas (movimentos em `LedgerEntryTaxFNX`).

### Quem deve habilitar

Empresas que utilizam **adiantamento de cliente** ou **adiantamento de fornecedor** com retenções fiscais e que precisam que a contabilização das retenções acompanhe o momento da aplicação.

---

## 2. Configuração

### Passo 1: Habilitar o recurso

1. Navegue até a página **Configurações Fiscais** (`FiscalSetupFNX`)
2. Na seção **Geral**, localize o campo **Calcular Impostos na Aplicação** (`CalcApplicationTaxes`)
3. Marque o campo

A partir desse momento, novas aplicações entre adiantamento e fatura passam a calcular e contabilizar os impostos retidos.

> **Importante**: O campo é desabilitado por padrão. Empresas que já operam com a localização não veem nenhuma mudança de comportamento até habilitarem o parâmetro explicitamente.

### Passo 2: Verificar contas de imposto

Confirme que as **Contas de Lançamento de Impostos** (`TaxPostingAccountsFNX`) estão configuradas para cada combinação de jurisdição/imposto que pode aparecer em retenções aplicadas:

- **Conta a Pagar** (Payable Account) — para a obrigação fiscal a recolher
- **Conta de Despesa** (Expense Account) — quando o imposto compõe o resultado
- **Conta de Crédito de Imposto** (Tax Credit Account) — quando há crédito a aproveitar

Sem essas contas, a aplicação dispara erro e não é concluída.

---

## 3. Comportamento

### Aplicação direta no registro do pagamento

Quando o usuário registra um pagamento com **Aplicar a Doc.** preenchido, o cálculo de impostos retidos continua acontecendo no momento do registro do pagamento — comportamento **inalterado**.

### Aplicação posterior (telas Aplicar Movtos.)

Ao abrir **Aplicar Movtos. Cliente** ou **Aplicar Movtos. Fornecedor** e marcar movimentos para aplicar:

1. O sistema recalcula o **Valor Impostos** (`TaxesAmountFNX`) da linha à medida que a aplicação muda
2. O campo **Valor Impostos** fica visível na tela junto ao valor aplicado, deixando claro o líquido financeiro
3. Quando a aplicação é confirmada, são gerados:
   - O lançamento contábil de quitação do adiantamento (comportamento atual)
   - **Novos lançamentos**: um por imposto retido, atualizando Conta a Pagar / Despesa / Crédito de Imposto conforme a configuração da jurisdição
   - Um lançamento financeiro líquido no cliente/fornecedor para fechar o saldo após retenções

### Estorno (desaplicar)

Ao desaplicar um adiantamento que gerou impostos:

- O sistema localiza os movimentos de imposto vinculados pelo identificador `GLEntryPaymentTaxRelationFNX`
- Marca cada movimento de imposto como **estornado** (`PaymentReversed`) e zera os valores de pagamento daquele movimento
- Gera lançamentos contábeis com sinal invertido (`Reversing Entry`) para reverter os efeitos no Razão

### Validação de excesso

Se o total de impostos retidos a aplicar **excede** o valor disponível para aplicação, o sistema interrompe a operação com a mensagem:

> *O valor de impostos excedeu o valor aplicado. Aplicado: %1 / Impostos: %2*

Isso protege contra cenários onde uma combinação inconsistente de adiantamento, documento e retenções produziria saldo negativo.

### Exceção: documento maior que adiantamento

Quando o valor do documento é **maior** que o valor do adiantamento, o ajuste de retenção sobre o valor aplicado **não é necessário** — o adiantamento é integralmente baixado e o saldo restante segue para liquidação normal pelo pagamento principal. Nesse cenário o sistema pula o ajuste e mantém o valor aplicado igual ao saldo do adiantamento.

---

## 4. Exemplo Prático

**Cenário**: fatura de serviço recebida de fornecedor PJ no valor de R$ 10.000,00, com retenção de IRRF de R$ 150,00 (1,5%). A empresa havia feito um pagamento antecipado de R$ 5.000,00 ao mesmo fornecedor antes de receber a fatura.

### Estado inicial

| Documento | Tipo | Valor | Saldo |
| --- | --- | --- | --- |
| PG-ADTO-001 | Pagamento Antecipado a Fornecedor | R$ 5.000,00 | R$ 5.000,00 (em aberto) |
| FC-1234 | Fatura de Compra de Serviço | R$ 10.000,00 | R$ 10.000,00 (em aberto) |
| Movimento IRRF | Imposto retido no pag. da fatura | R$ 150,00 | calculado |

### Aplicação PG-ADTO-001 → FC-1234 (com flag habilitada)

Quando o usuário aplica o adiantamento à fatura pela tela **Aplicar Movtos. Fornecedor**, o sistema gera os seguintes lançamentos:

| Movimento gerado | Conta | Débito | Crédito |
| --- | --- | --- | --- |
| Quitação adiantamento | Fornecedor (Conta a Pagar) | R$ 5.000,00 | — |
| Quitação adiantamento | Adiantamento a Fornecedor | — | R$ 5.000,00 |
| IRRF retido na aplicação | IRRF a Recolher | — | R$ 75,00 |
| Líquido financeiro do IRRF | Fornecedor (Conta a Pagar) | R$ 75,00 | — |

> Como o adiantamento (R$ 5.000) é menor que a fatura (R$ 10.000), a retenção é registrada **proporcionalmente** ao que foi aplicado: R$ 75,00 de IRRF nesta aplicação. Os R$ 75,00 restantes são registrados quando o saldo da fatura (R$ 5.000) for liquidado pelo pagamento principal ao fornecedor.

**Efeito no saldo do fornecedor após a aplicação**:

- Saldo da fatura FC-1234: R$ 10.000,00 − R$ 5.000,00 (adiantamento) − R$ 75,00 (IRRF retido) = **R$ 4.925,00 a pagar**
- Adiantamento PG-ADTO-001 totalmente quitado
- Obrigação **IRRF a Recolher** registrada: R$ 75,00 (a ser apurada e paga ao fisco)

### Estorno (desaplicação)

Ao desaplicar a operação pela página **Movimentos do Fornecedor** (ação **Estornar Aplicação**), o sistema:

- Gera lançamentos com sinal invertido nas mesmas contas (`Reversing Entry`), restaurando o saldo do adiantamento e da fatura
- Estorna o movimento de IRRF a Recolher, marcando o movimento de imposto como **estornado**
- Saldo do fornecedor volta ao estado anterior à aplicação

---

## 5. Resumo dos Campos Novos

### Tabela Configurações Fiscais (`FiscalSetupFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Calcular Impostos na Aplicação** (`CalcApplicationTaxes`) | Boolean | Habilita o cálculo e a contabilização de impostos retidos quando movimentos são aplicados a um adiantamento, incluindo o estorno na desaplicação. Padrão: desabilitado. |

### Páginas Aplicar Movtos. Cliente / Aplicar Movtos. Fornecedor

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Valor Impostos** (`TaxesAmountFNX`) | Decimal | Total de impostos retidos brasileiros (IRRF, INSS, ISS, PIS, COFINS, CSLL) calculados para o movimento a partir das entradas de imposto relacionadas. Atualizado automaticamente conforme a aplicação muda. |
