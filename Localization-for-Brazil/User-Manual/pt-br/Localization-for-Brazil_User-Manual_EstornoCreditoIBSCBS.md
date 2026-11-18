# Estorno de Crédito IBS/CBS (gEstornoCred)

**Data**: Março/2026

---

## Sumário

Esta atualização implementa o fluxo completo de **Estorno de Crédito** para os impostos IBS e CBS da Reforma Tributária, conforme grupo `gEstornoCred` (tag UB116) definido na NT 2025.002 v1.33. O valor de estorno é informado manualmente pelo usuário na subpage de impostos do documento e contabilizado automaticamente com partida dobrada (débito na conta de despesa + crédito na conta de estorno).

## 1. Contexto

### O que é

O Estorno de Crédito é um mecanismo da Reforma Tributária que obriga o contribuinte a devolver créditos de IBS/CBS previamente apropriados, quando a operação se enquadra em determinadas classificações tributárias (`cClassTrib`). A obrigatoriedade é determinada pelo indicador `ind_gEstornoCred` da tabela de classificações tributárias publicada pela Receita Federal.

### Quando se aplica

O estorno se aplica quando a **Classificação Tributária IBS/CBS** (`IBSCBSTaxClassification`) associada à linha do documento possui o campo **Estorno de Crédito** (`CreditReversal`) marcado como verdadeiro. Esse indicador é importado automaticamente a partir dos dados oficiais da classificação tributária, via ação **Baixar Classificações Tributárias** na página **Classificações Tributárias IBS/CBS** (`IBSCBSTaxClassifications`).

### Como funciona

1. Durante o cálculo de impostos, o sistema identifica as linhas IBS/CBS cuja classificação requer estorno
2. O usuário informa manualmente o valor de estorno para cada tipo de imposto (CBS, IBS UF, IBS Municipal) na subpage de impostos
3. No registro do documento, o sistema contabiliza o estorno com partida dobrada
4. Na geração da NF-e, os valores são enviados no grupo `gEstornoCred` do XML

---

<a id="configuracao-contabilizacao"></a>

## 2. Configuração da Contabilização

### Criando um código de contabilização para estorno

1. Navegue até **Contabilizações de Imposto** (`TaxPostings`) via pesquisa do Business Central
2. Crie um novo registro ou abra um existente dedicado ao IBS/CBS com estorno
3. No **Cartão Contabilização Imposto** (`TaxPostingCard`), no grupo **Faturamento**, marque o campo **Estorno de Crédito** (`PostingCreditReversal`)

### Comportamento ao ativar o Estorno de Crédito

Ao marcar **Estorno de Crédito**, o sistema automaticamente:

- Ativa **Despesa** (`PostingExpense`) como verdadeiro
- Limpa todos os demais campos do grupo **Faturamento** (**Custo/Preço Item**, **Financeiro**, **Imposto a Pagar**, **Crédito Imposto**)
- Desabilita o grupo **Pagamento/Recebimento** e as **linhas de contabilização** (`TaxPostingLine`)

Isso garante que o código de contabilização de estorno seja exclusivo — não pode ser combinado com outros tipos de lançamento.

> **Nota**: Esse código deve ser utilizado exclusivamente em jurisdições/área fiscal configuradas para impostos IBS/CBS que exigem estorno de crédito.

---

<a id="configuracao-contas"></a>

## 3. Configuração das Contas Contábeis

### Definindo as contas de estorno

1. Navegue até **Contas Contabilização Impostos** (`TaxPostingAccounts`) via pesquisa
2. Para cada combinação de **Classificação Imposto** (CBS, IBS UF, IBS Mun) e filtro desejado, preencha:

| Campo | Descrição |
|-------|-----------|
| **Tipo Conta Estorno de Crédito** (`CreditReversalAccountType`) | Tipo da conta para registrar o crédito do estorno (normalmente **Conta Contábil**) |
| **Nº Conta Estorno de Crédito** (`CreditReversalAccountNo`) | Número da conta contábil de estorno de crédito |
| **Tipo Conta Despesa** (`ExpenseAccountType`) | Tipo da conta para registrar o débito da despesa |
| **N° Conta Despesa** (`ExpenseAccountNo`) | Número da conta contábil de despesa do estorno |

> **Importante**: Ambas as contas (Estorno de Crédito e Despesa) precisam estar preenchidas para que a contabilização funcione corretamente, pois o estorno gera dois lançamentos.

---

<a id="preenchimento"></a>

## 4. Preenchimento do Valor de Estorno

### Onde informar

O valor de estorno é informado na **subpage de impostos** do documento (pedido de venda, fatura de compra, documento fiscal, etc.).

### Campos na subpage de impostos

| Campo | Descrição |
|-------|-----------|
| **Estorno de Crédito** (`CreditReversal`) | Indica se a linha requer estorno. Preenchido automaticamente com base na classificação tributária. Não editável. |
| **Valor Estorno de Crédito** (`CreditReversalAmount`) | Valor do estorno a ser informado pelo usuário. Editável somente quando **Estorno de Crédito** = Sim. |

### Como preencher

1. Abra o documento (pedido de venda, fatura de compra, etc.)
2. Acesse a subpage de impostos (ação **Impostos** ou equivalente)
3. Ative a visibilidade das colunas **Estorno de Crédito** e **Valor Estorno de Crédito** via personalização da página
4. Nas linhas de CBS, IBS UF e IBS Municipal que possuem **Estorno de Crédito** = Sim, informe o valor desejado em **Valor Estorno de Crédito**
5. Cada tipo de imposto recebe seu valor separadamente:
   - Linha CBS → valor do `vCBSEstCred`
   - Linha IBS UF → parcela UF do `vIBSEstCred`
   - Linha IBS Municipal → parcela Municipal do `vIBSEstCred`

> **Nota**: Se a classificação tributária não exigir estorno, o campo **Valor Estorno de Crédito** ficará desabilitado e não poderá ser preenchido.

---

<a id="contabilizacao"></a>

## 5. Contabilização

### Lançamentos gerados

Quando o documento é registrado e existe um valor de estorno preenchido, o sistema gera **dois lançamentos contábeis** por linha de imposto com estorno:

| Lançamento | Conta | Valor |
|------------|-------|-------|
| Crédito | **Nº Conta Estorno de Crédito** | Valor do estorno (negativo) |
| Débito | **N° Conta Despesa** | Valor do estorno (positivo) |

### Independência do valor do imposto

A contabilização do estorno de crédito é **independente** do valor do imposto calculado (`TaxAmount`). Isso significa que:

- O estorno é contabilizado mesmo que o valor do imposto na linha seja zero
- O valor contabilizado é sempre o **Valor Estorno de Crédito** informado pelo usuário, não o valor do imposto

### Fluxos suportados

A contabilização funciona nos três fluxos de registro:

- **Vendas** — ao registrar pedidos de venda, faturas e notas de crédito
- **Compras** — ao registrar pedidos de compra, faturas e notas de crédito
- **Documentos fiscais** — ao registrar documentos fiscais manuais

---

<a id="nfe"></a>

## 6. NF-e

### Grupo gEstornoCred no XML

Quando a classificação tributária indica estorno de crédito, o XML da NF-e inclui automaticamente o grupo `gEstornoCred` com os seguintes campos. O grupo é gerado quando a classificação tributária tem o indicador `ind_gEstornoCred` = 1.

| Tag XML | Descrição | Origem |
|---------|-----------|--------|
| `vIBSEstCred` | Valor do estorno de IBS | Soma de **Valor Estorno de Crédito** das linhas IBS UF + IBS Municipal |
| `vCBSEstCred` | Valor do estorno de CBS | **Valor Estorno de Crédito** da linha CBS |

O grupo `gEstornoCred` só é incluído quando a classificação tributária indica estorno ou quando a nota de débito é do tipo 07 (perda em estoque).

---

<a id="resumo-campos-novos"></a>

## 7. Resumo dos Campos Novos

### Tabela Contabilização Imposto (`TaxPostingFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Estorno de Crédito** (`PostingCreditReversal`) | Boolean | Ativa a contabilização de estorno de crédito. Ao ativar, força Despesa=Sim e limpa demais flags. |

### Tabela Contas Contabilização Impostos (`TaxPostingAccountsFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tipo Conta Estorno de Crédito** (`CreditReversalAccountType`) | Option | Tipo da conta para registrar o crédito do estorno |
| **Nº Conta Estorno de Crédito** (`CreditReversalAccountNo`) | Code[20] | Número da conta de estorno de crédito |

### Tabelas Linha Valor Imposto (`TaxAmountLineFNX`) e Mov. Imposto Documento Fiscal (`FiscalDocumentTaxEntryFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Estorno de Crédito** (`CreditReversal`) | Boolean | Indica se a linha requer estorno (preenchido automaticamente) |
| **Valor Estorno de Crédito** (`CreditReversalAmount`) | Decimal | Valor do estorno informado pelo usuário |
| **Registrar Estorno de Crédito** (`PostingCreditReversal`) | Boolean | Flag de contabilização propagado do código de contabilização |

### Tabela Linha Imposto Modificada (`ModifiedTaxAmountLineFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Valor Estorno de Crédito** (`CreditReversalAmount`) | Decimal | Valor de estorno salvo quando o usuário modifica impostos manualmente |
