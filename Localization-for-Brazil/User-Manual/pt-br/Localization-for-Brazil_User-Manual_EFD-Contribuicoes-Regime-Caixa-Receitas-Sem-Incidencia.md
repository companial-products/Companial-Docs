<!-- Branch: task/regime-de-caixa-exclui-receitas-sem-inci -->
# EFD Contribuições: Receitas sem Incidência no Regime de Caixa (F500, F525 e 1900)

**Data**: Agosto/2026

---

## Sumário

Na apuração de PIS/COFINS pelo **Regime de Caixa**, as notas de saída recebidas no período cujas bases de PIS e de COFINS fossem ambas zero eram descartadas da apuração e não apareciam nos registros **F500**, **F525** e **1900** da EFD-Contribuições. Isso excluía indevidamente todas as operações **sem incidência**, como exportação (CST 08), isenção (CST 07), alíquota zero (CST 06) e suspensão (CST 09), mesmo quando a receita havia sido efetivamente recebida.

Com esta atualização, o documento passa a ser escriturado sempre que houver receita recebida, com base e contribuição zeradas quando não há incidência. Na mesma correção, o rateio do valor recebido entre as linhas da nota passou a considerar apenas as **linhas de receita**, para que o **Valor Receita** escriturado corresponda ao que de fato foi recebido.

---

## 1. Contexto

### O que mudou

Até esta atualização, o Regime de Caixa aplicava um teste único e genérico ao decidir se um documento recebido entrava na apuração: só entrava se a soma das bases de PIS ou de COFINS do documento fosse maior que zero. Qualquer nota sem incidência tem essas duas bases zeradas. Logo, era descartada por completo e desaparecia dos três registros que a reportam.

O comportamento correto já existia no **Regime de Competência**, onde uma nota sem incidência de PIS/COFINS é escriturada normalmente. Esta correção alinha o Regime de Caixa a esse critério.

### Quando se aplica

- Empresas com **Critério de Escrituração do Regime Cumulativo** (`CRegimeBookkeepingType`) igual a **Regime de Caixa** na **Configuração EFD Contribuições** (`EFDContribSetupFNX`).
- Documentos de **saída** (vendas e prestações de serviço) cujo recebimento tenha sido aplicado dentro do período de apuração.
- Qualquer CST de não incidência de PIS/COFINS na saída:

| CST | Descrição |
| --- | --- |
| **06** | Operação Tributável (alíquota zero) |
| **07** | Operação Isenta da Contribuição |
| **08** | Operação Sem Incidência da Contribuição, o caso típico da **exportação** |
| **09** | Operação com Suspensão da Contribuição |

### O que não muda

- O **Regime de Competência** não sofre nenhuma alteração. Ele já tratava o cenário corretamente.
- Documentos de **entrada** (compras) no Regime de Caixa seguem fluxo próprio e não são afetados.
- O **valor devido** de PIS/COFINS apurado não muda por conta da escrituração dessas receitas, porque elas somam zero de base e zero de contribuição. A única exceção está descrita em [7. Efeito na Reoneração PIS/COFINS](#7-efeito-na-reoneração-piscofins-lc-2242025).

---

## 2. Critério de Inclusão do Documento na Apuração

A decisão de incluir o documento passou a ser tomada uma única vez, no nível do documento, com um critério **aditivo**: nenhum documento que já entrava deixou de entrar.

Um documento de saída recebido no período é escriturado quando **qualquer** das condições abaixo for verdadeira:

1. O documento possui base ou valor de PIS ou de COFINS diferente de zero (critério anterior, mantido); **ou**
2. O documento possui ao menos uma linha de **receita** com valor maior que zero (critério novo).

Uma vez incluído o documento, todas as suas linhas de receita com valor são escrituradas, com ou sem incidência de PIS/COFINS.

### O que é uma "linha de receita"

Uma linha do documento fiscal é receita quando o campo **Receita** (`Income`) da **Linha de Documento Fiscal** (`FiscalDocumentLineFNX`) está marcado. Esse campo é preenchido no registro do documento a partir do **Tipo de Operação** (`OperationTypeFNX`) da linha: quando o campo **Mov. Cliente/Fornecedor** (`CustomerVendorEntry`) do tipo de operação é **Valor Bruto**, a linha é receita.

Linhas que não são receita, como remessas, transferências e demais movimentos que viajam dentro da nota sem representar faturamento, continuam fora da receita escriturada. O mesmo vale para linhas de valor zero. Ambas continuam sujeitas à regra anterior: só geram linha de apuração se carregarem PIS ou COFINS.

> **Nota fiscal complementar de imposto**: por não ter natureza de receita, continua fora do F500. O comportamento é o mesmo de antes da correção.

---

## 3. Rateio do Valor Recebido entre as Linhas

O Regime de Caixa distribui o valor recebido entre as linhas do documento, proporcionalmente ao valor de cada uma, para preencher o **Valor Receita** (`IncomeAmount`) de cada linha da apuração.

Esse rateio passou a considerar apenas as linhas de receita do documento, tanto no divisor quanto na distribuição. Antes, uma linha sem natureza de receita entrava na conta do divisor e retinha uma parcela do valor recebido. Como essa linha não é escriturada, a parcela simplesmente se perdia, e a receita escriturada ficava menor que a recebida.

- Documento com **todas** as linhas de receita, ou com **uma única** linha: nenhuma alteração de valor.
- Documento com linhas de receita **e** linhas que não são receita: o **Valor Receita** das linhas de receita passa a corresponder integralmente ao valor recebido.
- Documento **sem nenhuma** linha de receita: mantém o comportamento histórico, rateando entre todas as linhas.

A sobra de arredondamento continua sendo atribuída à última linha, de modo que a soma escriturada corresponda exatamente ao valor recebido.

---

## 4. Exemplos Práticos

### Exemplo 1 — Exportação (CST 08) recebida integralmente

Nota fiscal de saída de exportação, linha única, CFOP 7.101, sem incidência de PIS/COFINS. Valor da nota R$ 50.000,00, recebido integralmente dentro do período de apuração.

| Campo da linha de apuração | Antes | Depois |
| --- | --- | --- |
| Linha gerada na apuração | *nenhuma* | 1 linha |
| **Tipo Fiscal** (`FiscalType`) | — | Saída |
| **Valor Bruto** (`GLAmount`) | — | 50.000,00 |
| **Valor Receita** (`IncomeAmount`) | — | 50.000,00 |
| **Cód. CST PIS** (`PISCSTCode`) | — | 08 |
| **PIS Base** (`PISBase`) / **Valor PIS** (`PISAmount`) | — | 0,00 / 0,00 |
| **Cód. CST COFINS** (`COFINSCSTCode`) | — | 08 |
| **COFINS Base** (`COFINSBase`) / **Valor COFINS** (`COFINSAmount`) | — | 0,00 / 0,00 |

A nota passa a constar do **F500**, do **F525** e do **1900**.

### Exemplo 2 — Recebimento parcial

A mesma nota do exemplo anterior, com apenas 50% do valor recebido dentro do período:

| Campo | Valor |
| --- | --- |
| **Valor Bruto** | 50.000,00 |
| **Valor Receita** | 25.000,00 |
| **PIS Base** / **COFINS Base** | 0,00 / 0,00 |

O **Valor Receita** acompanha o percentual efetivamente recebido, e não o total da nota, exatamente como já acontecia com as notas tributadas.

### Exemplo 3 — Nota com linha que não é receita

Nota fiscal de saída com duas linhas: uma venda de R$ 2.000,00 (linha de receita) e uma remessa de R$ 4.000,00 (linha sem natureza de receita, cujo tipo de operação não gera movimento de cliente). O valor de venda de R$ 2.000,00 é recebido integralmente no período.

| Linha do documento | Valor | É receita? | **Valor Receita** antes | **Valor Receita** depois |
| --- | --- | --- | --- | --- |
| Venda | 2.000,00 | Sim | 666,67 | **2.000,00** |
| Remessa | 4.000,00 | Não | *não escriturada* | *não escriturada* |
| **Total escriturado** | — | — | **666,67** | **2.000,00** |

Antes, o divisor do rateio era R$ 6.000,00 (as duas linhas somadas), e a parcela retida pela remessa se perdia. Depois, o divisor é R$ 2.000,00 (só a linha de receita), e a receita escriturada corresponde ao valor recebido.

### Exemplo 4 — Período com notas tributadas e sem incidência

Apuração com uma nota tributada (CST 01) e uma nota de exportação (CST 08), ambas recebidas integralmente no período:

| Documento | **Valor Receita** | **PIS Base** | **Valor PIS** | **COFINS Base** | **Valor COFINS** |
| --- | --- | --- | --- | --- | --- |
| NF 001 — CST 01 | 10.000,00 | 10.000,00 | 65,00 | 10.000,00 | 300,00 |
| NF 002 — CST 08 (exportação) | 50.000,00 | 0,00 | 0,00 | 0,00 | 0,00 |

Cada nota gera seu próprio grupo de registros F500/F525 com o seu CST. A nota tributada mantém exatamente os mesmos valores de antes da correção.

---

## 5. Reflexo no Arquivo SPED

A geração dos registros abaixo é acionada quando a empresa está em **Regime de Caixa**. Nenhum filtro foi alterado nos registros. Eles apenas passam a receber as linhas que antes não eram criadas.

| Registro | Conteúdo | Campo alimentado pelo **Valor Receita** |
| --- | --- | --- |
| **F500** | Consolidação da receita recebida no período, por CST, CFOP, alíquota, modelo de documento e conta contábil | `VL_REC_CAIXA` |
| **F525** | Detalhamento da composição da receita escriturada no F500, por documento, participante e item | `VL_REC` e `VL_REC_DET` |
| **1900** | Consolidação dos documentos do período por modelo, série, situação, CST e CFOP | `VL_TOT_REC` (e a nota passa a somar em `QUANT_DOC`) |

Nas notas sem incidência, os campos de base e de contribuição (`VL_BC_PIS`, `VL_PIS`, `VL_BC_COFINS`, `VL_COFINS`) do F500 saem zerados, com o CST original preservado em `CST_PIS` e `CST_COFINS`.

A conta contábil de receita da nota sem incidência também passa a ser declarada no **bloco 0500**, o que garante que o `COD_CTA` informado no F500 e no F525 referencie uma conta existente no arquivo.

---

## 6. Como Conferir

1. Abra a **Apuração PIS/COFINS** (`PISCOFINSSettlementFNX`) do período.
2. Clique na ação **Processar**.
3. Clique na ação **Linhas dos Documentos** para abrir **Linhas de Documento PIS/COFINS** (`PISCOFINSDocumentLinesFNX`).
4. Localize a nota sem incidência pelo **Nº Nota Fiscal** (`FiscalNo`). Confira:
   - **Tipo Fiscal** (`FiscalType`) igual a **Saída**;
   - **Valor Receita** (`IncomeAmount`) igual ao valor recebido no período;
   - **Cód. CST PIS** e **Cód. CST COFINS** com o CST original da nota (06, 07, 08 ou 09);
   - **PIS Base**, **Valor PIS**, **COFINS Base** e **Valor COFINS** zerados;
   - **Nº Conta Contábil** (`GLAccountNo`) preenchido com a conta de receita da nota.
5. Gere o arquivo pela ação **Exportar** e confirme a presença da nota nos registros F500, F525 e 1900.

> Se a nota não aparecer, verifique se o recebimento foi de fato **aplicado** ao documento dentro do período de apuração e se o documento fiscal está com o **Status** igual a **Registrado**. O Regime de Caixa parte dos recebimentos aplicados, não das notas emitidas.

---

## 7. Efeito na Reoneração PIS/COFINS (LC 224/2025)

Este é o único efeito da correção sobre o valor devido, e é esperado.

O cálculo da reoneração varre as linhas da apuração PIS/COFINS filtrando pelo CST cadastrado na **Matriz de Reoneração PIS/COFINS** e, no Regime de Caixa, usa o **Valor Receita** como base. Como as receitas sem incidência voltaram a ser escrituradas, elas passam a ser alcançadas por esse cálculo.

- Em empresa com **Habilitar Reoneração PIS/COFINS** marcado e um CST de não incidência cadastrado na matriz, **o valor devido pode subir em relação ao mês anterior**. Nenhuma regra de cálculo da reoneração foi alterada. O efeito vem das linhas que voltaram a existir.
- O efeito se concentra em **CST 06** e **CST 07**, que são os benefícios fiscais alcançados pela LC 224/2025. A exportação (**CST 08**) é imunidade constitucional, não benefício fiscal, e não deve constar na matriz.
- Notas com linhas que não são receita também têm o ajuste de reoneração aumentado na mesma proporção do **Valor Receita**, pelo motivo descrito em [3. Rateio do Valor Recebido entre as Linhas](#3-rateio-do-valor-recebido-entre-as-linhas).

Detalhes do cálculo e da matriz: [EFD Contribuições: Reoneração PIS/COFINS (LC 224/2025)](Localization-for-Brazil_User-Manual_PIS-COFINS-Reoneracao-CST-06.md).

---

## 8. Compatibilidade

- **Nenhuma configuração nova**: a correção não introduz campos, tabelas ou páginas. Não há nada a habilitar. O novo comportamento vale automaticamente.
- **Apurações geradas a partir da atualização**: a correção vale para as apurações processadas depois dela. Períodos já fechados ou já transmitidos à Receita Federal não são reprocessados nem retificados automaticamente.
- **Reprocessamento voluntário**: reprocessar uma apuração antiga (ação **Processar**) recria as linhas de documento e faz a apuração passar a incluir as receitas sem incidência do período. Faça isso apenas de forma deliberada, ciente do impacto sobre um arquivo já transmitido.
- **Notas canceladas**: continuam excluídas da apuração.
- **Notas tributadas**: escrituradas exatamente como antes, sem alteração de valores.
