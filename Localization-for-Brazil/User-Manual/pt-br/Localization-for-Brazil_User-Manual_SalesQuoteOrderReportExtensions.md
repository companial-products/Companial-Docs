# Reports Padrão: Impostos Brasileiros em Cotações e Ordens de Venda

**Data**: Março/2026

---

## Sumário

Esta atualização adiciona colunas de **impostos brasileiros** (IPI, ST, CBS, IBS UF, IBS Mun, IS) aos reports padrão **Standard Sales - Quote** (1304) e **Standard Sales - Order Conf.** (1305). Os valores são calculados pelo engine de impostos da localização (`TaxCalculateFNX`) e disponibilizados no dataset do report para uso em layouts customizados (Word/RDLC).

## 1. Contexto

### O que é

Os reports padrão de cotação e ordem de venda do Business Central não possuem colunas para impostos brasileiros. Para que layouts customizados possam exibir valores de IPI, ST, CBS, IBS UF, IBS Mun e IS, foram criadas extensões de report que adicionam essas colunas ao dataset, tanto por linha quanto em totais.

### Quando se aplica

As colunas de impostos são preenchidas automaticamente sempre que o report for executado. Os valores dependem da configuração fiscal do documento (NCM, CFOP, matriz fiscal, etc.). Se o documento não possuir impostos brasileiros configurados, os valores serão zero.

---

## 2. Reports Estendidos

| **Report Extension** | **ID** | **Report Base** | **Arquivo** |
| --- | --- | --- | --- |
| StandardSalesQuoteLocFNX | 52112423 | Standard Sales - Quote (1304) | `ReportExtensions/StandardSalesQuote.ReportExt.al` |
| StandardSalesOrderConfLocFNX | 52112424 | Standard Sales - Order Conf. (1305) | `ReportExtensions/StandardSalesOrderConf.ReportExt.al` |

Ambas as extensões seguem a mesma estrutura e lógica de cálculo.

---

## 3. Colunas Adicionadas

### Dataitem Header

| **Coluna** | **Tipo** | **Descrição** |
| --- | --- | --- |
| BRTaxesCalculatedFNX | Boolean | Indica se o cálculo de impostos brasileiros foi executado com sucesso |

### Dataitem Line (por linha do documento)

| **Coluna** | **Tipo** | **Descrição** |
| --- | --- | --- |
| IPIAmountFNX | Decimal | Valor do IPI da linha |
| IPIBaseAmountFNX | Decimal | Base de cálculo do IPI da linha |
| STAmountFNX | Decimal | Valor do ICMS-ST da linha |
| STBaseAmountFNX | Decimal | Base de cálculo do ICMS-ST da linha |
| CBSAmountFNX | Decimal | Valor da CBS da linha |
| CBSBaseAmountFNX | Decimal | Base de cálculo da CBS da linha |
| IBSUFAmountFNX | Decimal | Valor do IBS UF da linha |
| IBSUFBaseAmountFNX | Decimal | Base de cálculo do IBS UF da linha |
| IBSMunAmountFNX | Decimal | Valor do IBS Municipal da linha |
| IBSMunBaseAmountFNX | Decimal | Base de cálculo do IBS Municipal da linha |
| ISAmountFNX | Decimal | Valor do IS da linha |
| ISBaseAmountFNX | Decimal | Base de cálculo do IS da linha |

### Dataitem Totals (totais do documento)

| **Coluna** | **Tipo** | **Descrição** |
| --- | --- | --- |
| TotalIPIAmountFNX | Decimal | Total do IPI no documento |
| TotalIPIBaseAmountFNX | Decimal | Total da base de cálculo do IPI no documento |
| TotalSTAmountFNX | Decimal | Total do ICMS-ST no documento |
| TotalSTBaseAmountFNX | Decimal | Total da base de cálculo do ICMS-ST no documento |
| TotalCBSAmountFNX | Decimal | Total da CBS no documento |
| TotalCBSBaseAmountFNX | Decimal | Total da base de cálculo da CBS no documento |
| TotalIBSUFAmountFNX | Decimal | Total do IBS UF no documento |
| TotalIBSUFBaseAmountFNX | Decimal | Total da base de cálculo do IBS UF no documento |
| TotalIBSMunAmountFNX | Decimal | Total do IBS Municipal no documento |
| TotalIBSMunBaseAmountFNX | Decimal | Total da base de cálculo do IBS Municipal no documento |
| TotalISAmountFNX | Decimal | Total do IS no documento |
| TotalISBaseAmountFNX | Decimal | Total da base de cálculo do IS no documento |

---

## 4. Comportamento do Cálculo

### Fluxo de Processamento

1. Ao processar cada **Header** (Sales Header), o sistema chama `TaxCalculateFNX.CalculateSalesDoc()` para calcular todos os impostos brasileiros do documento
2. Os resultados são armazenados em registros temporários de `TaxAmountLineFNX`, indexados por número da linha e tipo de imposto (`TaxIdentificationFNX`)
3. Para cada **Line** (Sales Line), os valores de cada imposto são extraídos filtrando por `DocumentLineNo` e `TaxIdentification`
4. Os valores por linha são acumulados nos totais do documento

### Impostos e Identificações

| **Imposto** | **Enum TaxIdentificationFNX** | **Valor** |
| --- | --- | --- |
| IPI | IPI | 1 |
| ICMS-ST | ST | 15 |
| IBS UF | IBSUF | 100 |
| IBS Municipal | IBSMun | 105 |
| CBS | CBS | 110 |
| IS | IS | 115 |

---

## 5. Tratamento de Erros

O cálculo de impostos é executado dentro de uma **TryFunction** (`TryCalculateSalesDoc`). Isso garante que:

- Se ocorrer **qualquer erro** durante o cálculo (configuração fiscal incompleta, dados inconsistentes, etc.), o report **não será interrompido**
- A coluna `BRTaxesCalculatedFNX` será `false`, indicando que os impostos não foram calculados
- Todos os valores de impostos por linha e totais permanecerão **zerados**
- O processamento das linhas do documento pula a extração de impostos quando `BRTaxesCalculated = false`

Esse comportamento permite que o report seja gerado mesmo em cenários onde a configuração fiscal está incompleta, sem impactar a impressão dos demais dados do documento.

---

## 6. Uso em Layouts

As colunas estão disponíveis no dataset do report e podem ser utilizadas em layouts customizados (Word ou RDLC). Para adicionar os impostos ao layout:

1. Abra o layout customizado do report no Word ou Report Builder
2. As colunas de linha (ex: `IPIAmountFNX`) estão disponíveis no dataitem **Line**
3. As colunas de totais (ex: `TotalIPIAmountFNX`) estão disponíveis no dataitem **Totals**
4. Opcionalmente, utilize `BRTaxesCalculatedFNX` para exibir/ocultar a seção de impostos condicionalmente

> **Nota**: As colunas são adicionadas apenas ao dataset. O layout padrão Word/RDLC do report não é modificado. Para que os valores sejam visíveis na impressão, é necessário customizar o layout do report.
