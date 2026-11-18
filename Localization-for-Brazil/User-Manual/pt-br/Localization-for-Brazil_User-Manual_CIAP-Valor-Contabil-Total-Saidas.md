<!-- PR: #96 -->
# CIAP: Inclusão do IPI no Total das Saídas via Valor Contábil

**Data**: Julho/2026

---

## Sumário

Uma nova opção na **Configuração EFD ICMS/IPI** permite que o Total das Saídas do CIAP seja composto pelo valor contábil dos documentos fiscais, que inclui o IPI e as demais despesas da operação, em vez da base de cálculo do ICMS. A opção atende às legislações estaduais que exigem que o IPI faça parte do valor total das saídas no cálculo do coeficiente de creditamento.

---

## 1. Contexto

### O que é o coeficiente de creditamento do CIAP

O CIAP (Controle de Crédito de ICMS do Ativo Permanente) apropria o crédito de ICMS das aquisições de ativo imobilizado em 48 parcelas mensais. Cada parcela é multiplicada por um coeficiente de creditamento, calculado pela razão entre as saídas tributadas (somadas às exportações) e o total das saídas do período:

```
Coeficiente = (Saídas Tributadas + Exportações) / Total das Saídas
```

Esses valores são informados no registro **G110** do arquivo EFD ICMS/IPI, nos campos `VL_TRIB_EXP` (valor das saídas tributadas e exportações), `VL_TOTAL` (valor total das saídas) e `IND_PER_SAI` (índice de participação das saídas tributadas).

### Por que incluir o IPI

Por padrão, o sistema compõe as saídas tributadas pela base de cálculo do ICMS e as saídas isentas pelos valores de ICMS isento e outros. Como o IPI não integra a base de cálculo do ICMS nas operações entre contribuintes com produtos destinados à industrialização ou à comercialização, ele fica de fora do total das saídas.

Algumas legislações estaduais, porém, exigem que o total das saídas corresponda ao valor da operação, ou seja, ao valor contábil do documento, incluindo o IPI e as demais despesas acessórias. A nova opção permite adotar esse critério.

---

## 2. Configuração

1. Navegue até **Configuração EFD ICMS/IPI** (página `EFDICMSIPISetupFNX`).
2. No grupo **CIAP**, marque o campo **Usar Valor Contábil para Saídas** (`CIAPUseGLAmount`).

Com a opção desmarcada (padrão), o comportamento anterior é mantido. Nada muda para quem não habilitar o campo.

---

## 3. Comportamento no Cálculo

A opção altera a composição dos valores de saída acumulados para o CIAP durante a geração da apuração:

| Tipo de saída | Opção desmarcada (padrão) | Opção marcada |
|---------------|---------------------------|---------------|
| Exportação (CFOP iniciado em 7) | Valor contábil do documento | Valor contábil do documento (sem alteração) |
| Saída com base de ICMS | Base de cálculo do ICMS | Valor contábil do documento |
| Saída isenta / outras | ICMS Isento + ICMS Outros | Valor contábil do documento |

O valor contábil corresponde ao valor total da operação escriturado no documento fiscal (campo `VL_CONT` dos registros de apuração), incluindo IPI e demais despesas acessórias.

Com a opção marcada:

- Linhas de saída **com base de ICMS** entram nas saídas tributadas pelo valor contábil.
- Linhas de saída **sem base de ICMS** (isentas, não tributadas ou outras) entram nas saídas isentas pelo valor contábil.
- As exportações não sofrem alteração, pois já eram consideradas pelo valor contábil.

---

## 4. Exemplo Prático

Considere um período de apuração com duas notas fiscais de saída:

| Documento | Operação | Valor Mercadoria | IPI | Valor Contábil | Base ICMS |
|-----------|----------|------------------|-----|----------------|-----------|
| NF 1001 | Venda tributada | R$ 10.000,00 | R$ 1.000,00 | R$ 11.000,00 | R$ 10.000,00 |
| NF 1002 | Venda isenta | R$ 5.000,00 | — | R$ 5.000,00 | — |

Cálculo do coeficiente em cada cenário:

| | Opção desmarcada | Opção marcada |
|---|------------------|---------------|
| Saídas tributadas (`VL_TRIB_EXP`) | R$ 10.000,00 | R$ 11.000,00 |
| Total das saídas (`VL_TOTAL`) | R$ 15.000,00 | R$ 16.000,00 |
| Coeficiente (`IND_PER_SAI`) | 0,66666667 | 0,68750000 |

Com a opção marcada, o IPI da NF 1001 passa a compor tanto as saídas tributadas quanto o total das saídas, o que altera o coeficiente aplicado às parcelas de crédito do CIAP no período.

---

## 5. Onde os Valores Aparecem

Os totais calculados com o novo critério refletem em todos os pontos que utilizam o coeficiente do CIAP:

- **Registro G110** do arquivo EFD ICMS/IPI, nos campos `VL_TRIB_EXP`, `VL_TOTAL` e `IND_PER_SAI`.
- Relatório **CIAP** (`CIAPFNX`), emitido a partir da **Apuração ICMS/IPI** (`ICMSIPISettlementFNX`), nos campos **Valor Total Vendas Tributadas/Exportação** (`TotalTaxedSalesAmount`), **Valor Total Vendas** (`TotalSalesAmount`) e **Coeficiente** (`Coefficient`).

---

## 6. Resumo dos Campos Novos

### Tabela Configuração EFD ICMS/IPI (`EFDICMSIPISetupFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Usar Valor Contábil para Saídas** (`CIAPUseGLAmount`) | Booleano | Especifica se as saídas totais do CIAP usam o valor contábil do documento, que inclui o IPI e outras despesas da operação, em vez da base do ICMS. Habilite quando a legislação estadual exigir que o IPI faça parte do valor total das saídas. |
