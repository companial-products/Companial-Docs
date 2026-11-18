# IRPF: Redução de Imposto — Lei 15.270/2025

**Data**: Fevereiro/2026

## Sumário

Esta atualização implementa o **mecanismo de redução do IRRF** instituído pela Lei 15.270/2025, que reduz progressivamente o Imposto de Renda Retido na Fonte para contribuintes pessoa física com renda mais baixa. Além disso, inclui a atualização das **tabelas progressivas do IRRF e do INSS** para os valores vigentes em 2025/2026.

## 1. Mecanismo de Redução do IRPF

### O que é

A Lei 15.270, de 22 de maio de 2025, instituiu um mecanismo de redução do Imposto de Renda Retido na Fonte (IRRF) para contribuintes pessoa física. O objetivo é reduzir a carga tributária efetiva para faixas de renda mais baixas, aplicando uma redução que diminui progressivamente conforme a renda bruta aumenta.

A redução é aplicada automaticamente no cálculo do IRRF sobre pagamentos a pessoas físicas, após o cálculo do imposto pela tabela progressiva e a dedução da parcela a deduzir.

### Como funciona

O sistema calcula o imposto normalmente pela tabela progressiva (incluindo deduções de INSS, dependentes e parcela a deduzir) e, em seguida, aplica a redução com base na renda bruta tributável do beneficiário. A redução é configurada na Jurisdição Fiscal do IRRF através de dois parâmetros: uma constante e um fator.

Para rendas mais baixas, a redução pode zerar o imposto. Conforme a renda aumenta, a redução diminui até se tornar zero para rendas acima de R$ 7.350,01.

## 2. Configuração

### Acessando a Jurisdição Fiscal

1. Navegue até **Jurisdições Impostos** (página TaxJurisdictions) via pesquisa do Business Central
2. Localize e abra o **Cartão Jurisdição Imposto** (página TaxJurisdictionCard) correspondente ao IRRF (ex: IRRF PF, IRRF ALUGUEL)
3. No cartão da jurisdição, localize o grupo **Redução de Imposto (Lei 15.270/2025)**
4. Preencha os campos conforme descrito abaixo

### Campos de Configuração

| Campo | Tipo | Valor Vigente | Descrição |
| --- | --- | --- | --- |
| **Constante de Redução** (TaxReductionConstant) | Decimal | 978,62 | Constante (intercepto) da fórmula de redução. Representa a redução máxima possível, quando a renda é zero |
| **Fator de Redução** (TaxReductionFactor) | Decimal (6 casas) | 0,133145 | Fator (coeficiente angular) da fórmula. Determina a taxa de diminuição da redução conforme a renda aumenta |

**Nota**: Os campos são configuráveis para permitir ajustes caso os parâmetros da lei sejam alterados futuramente, sem necessidade de modificação no código.

## 3. Fórmula de Cálculo

O valor da redução é determinado pela seguinte fórmula:

` Redução = Constante − (Fator × Renda Bruta) `

Onde **Renda Bruta** corresponde ao valor tributável acumulado do beneficiário no mês (soma dos valores tributáveis de todas as notas do mesmo CPF/CNPJ no período).

### Limites

- Se o resultado for **negativo**, a redução é zero (não há aumento de imposto)
- Se o resultado **exceder o imposto calculado**, a redução é limitada ao valor do imposto (IRRF não pode ser negativo)
- O valor da redução é arredondado para 2 casas decimais

## 4. Regras de Aplicação

1. A redução é aplicada **somente** para jurisdições fiscais que utilizam tabela progressiva
2. A redução só é calculada quando **ambos** os parâmetros (Constante e Fator) estão preenchidos na jurisdição de imposto.
3. O imposto deve ser **maior que zero** após o cálculo pela tabela progressiva para que a redução seja aplicada
4. Em cálculos **cumulativos** (múltiplas notas no mês para o mesmo beneficiário), a renda bruta considera o acumulado de todas as notas anteriores do período
5. Para renda bruta acima de **R$ 7.350,01**, a fórmula resulta em zero ou negativo, e nenhuma redução é aplicada

## 5. Exemplo Prático

Cenário: três pagamentos de R$ 3.000,00 a uma mesma pessoa física no mês, com área de imposto que possua INSS e IRRF.

### NF 1 — R$ 3.000,00

| Etapa | Valor (R$) |
| --- | --- |
| Valor bruto | 3.000,00 |
| INSS retido (tabela progressiva) | 248,60 |
| Base IRRF (3.000 − 248,60) | 2.751,40 |
| Faixa IRRF: 7,5% | 24,20 |
| Redução: 978,62 − (0,133145 × 3.000) = 579,19 | 24,20 (limitada ao imposto) |
| **IRRF final** | **0,00** |

### NF 2 — R$ 3.000,00 (acumulada)

| Etapa | Valor (R$) |
| --- | --- |
| Valor bruto | 3.000,00 |
| INSS retido (acumulado) | 392,91 |
| Base IRRF acumulada | 5.358,49 |
| Faixa IRRF: 27,5% | 564,85 |
| Redução: 978,62 − (0,133145 × 6.000) = 179,75 | 179,75 |
| IRRF final (564,85 − 179,75) | **385,10** |

### NF 3 — R$ 3.000,00 (acumulada)

| Etapa | Valor (R$) |
| --- | --- |
| Valor bruto | 3.000,00 |
| INSS retido (teto 988,09) | 346,58 |
| Base IRRF acumulada | 8.011,91 |
| Faixa IRRF: 27,5% | 909,45 |
| Redução: 978,62 − (0,133145 × 9.000) = −219,69 | 0,00 (renda > 7.350) |
| **IRRF final** | **909,45** |

## 6. Atualização das Tabelas Progressivas

### Tabela Progressiva do IRRF (2026)

| Base de Cálculo (R$) | Alíquota | Parcela a Deduzir (R$) | Ded. Dependente (R$) | Faixa Isenta (R$) |
| --- | --- | --- | --- | --- |
| Até 2.428,80 | 0% | 0,00 | 0,00 | 0,00 |
| De 2.428,81 até 2.826,65 | 7,5% | 182,16 | 189,59 | 2.428,80 |
| De 2.826,66 até 3.751,05 | 15% | 394,16 | 189,59 | 2.428,80 |
| De 3.751,06 até 4.664,68 | 22,5% | 675,49 | 189,59 | 2.428,80 |
| Acima de 4.664,68 | 27,5% | 908,73 | 189,59 | 2.428,80 |

### Tabela Progressiva do INSS (2026)

| Faixa Salarial (R$) | Alíquota | Parcela a Deduzir (R$) | Teto (R$) |
| --- | --- | --- | --- |
| Até 1.621,00 | 7,5% | 0,00 |  |
| De 1.621,01 até 2.902,84 | 9% | 24,32 |  |
| De 2.902,85 até 4.354,27 | 12% | 111,40 |  |
| Acima de 4.354,28 | 14% | 198,49 | 988,09 |

## 7. Resumo dos Campos Novos

### Jurisdição Fiscal (Tax Jurisdiction)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| Constante de Redução | Decimal | Constante da fórmula de redução do IRRF (Lei 15.270/2025) |
| Fator de Redução | Decimal (6 casas) | Fator da fórmula de redução do IRRF (Lei 15.270/2025) |

### Linha de Impostos (Tax Amount Line / Fiscal Document Tax Entry)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| Valor da Redução | Decimal | Valor da redução efetivamente aplicada ao IRRF na operação |
| Constante de Redução | Decimal | Constante utilizada no cálculo (copiada da jurisdição fiscal) |
| Fator de Redução | Decimal | Fator utilizado no cálculo (copiado da jurisdição fiscal) |

### Onde os campos aparecem

1. **Cartão Jurisdição de Imposto** (TaxJurisdictionCard): Grupo "Redução de Imposto (Lei 15.270/2025)" com os campos Constante e Fator
2. **Subformulário Linha de Impostos** (TaxAmountLineSubform): Campos de redução ocultos por padrão (podem ser exibidos via personalização)
3. **Movimento de Imposto do Documento Fiscal** (FiscalDocumentTaxEntry): Campos gravados para rastreabilidade do benefício aplicado
