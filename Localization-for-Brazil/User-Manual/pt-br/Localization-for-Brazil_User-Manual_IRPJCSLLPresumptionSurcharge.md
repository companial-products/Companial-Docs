<!-- Commits: 0e67c30e, 61333fb3, fd8181ab, 91273138, 5fa3e1c1, 171f5706, 09a9f7a6, 00762062, bc71daf3, 0cf8d1c0, 079b16ec, 2c683f5f -->
# Apuração IRPJ/CSLL: Acréscimo de Presunção

**Data**: Maio/2026

---

## Sumário

Esta atualização adiciona suporte ao **Acréscimo de Presunção** e às **Receitas Financeiras** na apuração de IRPJ/CSLL para empresas do Lucro Presumido, tanto no Regime de Caixa quanto no Regime de Competência. O cálculo suporta alíquota única (configurada na configuração geral) ou múltiplas alíquotas por tipo de receita (configuradas em tabela própria). Quando a receita bruta trimestral excede o limite configurado, a parcela excedente é tributada com presunção acrescida. Opcionalmente, receitas financeiras podem ser incluídas integralmente na base de cálculo.

**Atualização de maio/2026** (conforme **IN RFB 2.305/2025**, arts. 14 e 15, com redação da IN 2.306/2026):

- **§ 4º — Acumulação de saldo entre trimestres**: o limite não utilizado em um trimestre transfere para os trimestres seguintes, aumentando o limite efetivo isento do acréscimo.
- **§ 5º — Ajuste anual no Q4**: se a receita anual ficar abaixo do limite anual (ou se a soma dos excessos trimestrais superar o excesso anual), o acréscimo cobrado em trimestres anteriores é estornado proporcionalmente, via lançamento automático na tabela **Ajuste Apuração IRPJ/CSLL**.
- **Modo Híbrido de Acréscimo**: opção conservadora. Quando a receita do ano já ultrapassou o limite anual, aplica a alíquota acrescida sobre toda a receita do trimestre, sem distinção da parcela isenta.
- **Correção**: o cálculo por múltiplas alíquotas passa a disparar apenas em Regime de Competência. Linhas remanescentes na tabela **Percentuais de Presunção** não interferem mais em apurações no Regime de Caixa.

## 1. Contexto

### O que é

Empresas tributadas pelo Lucro Presumido calculam o IRPJ e a CSLL aplicando um percentual de presunção sobre a receita bruta. A **LC 224/2025** instituiu acréscimo de **10%** sobre os percentuais de presunção quando a receita bruta total ultrapassa **R$ 5.000.000,00 no ano-calendário**. A **IN RFB 2.305/2025** (arts. 14 e 15, com redação da IN 2.306/2026) regulamentou a apuração trimestral:

- **§ 2º**: cada trimestre verifica a receita bruta contra **R$ 1.250.000,00** (o limite anual dividido por 4).
- **§ 3º**: a parcela do trimestre que exceder o limite é tributada à alíquota acrescida (presunção normal × 1,10).
- **§ 4º**: o limite não utilizado em um trimestre transfere para os trimestres seguintes.
- **§ 5º**: no último trimestre, faz-se o ajuste anual cumulativo. Se a soma das receitas anuais ficar abaixo do limite anual, o acréscimo dos trimestres anteriores é estornado.

**Exemplo típico**: para atividades com presunção de 8%, quando a receita bruta trimestral excede R$ 1.250.000,00, a parcela excedente é tributada a uma alíquota acrescida (8% × 1,10 = 8,80%), o que representa um acréscimo de 10%.

### Quando se aplica

- Apuração de IRPJ ou CSLL pelo regime de **Lucro Presumido**
- Em **Regime de Caixa** (alíquota única) ou **Regime de Competência** (tabela **Percentuais de Presunção**)
- Quando a receita bruta trimestral, somada ao saldo de limite não utilizado em trimestres anteriores do mesmo ano, excede o **Limite Trimestral** configurado

### Cenários suportados

| Cenário | Critério de Escrituração | Origem da Receita | Alíquota de Presunção |
| --- | --- | --- | --- |
| Alíquota única | Regime de Caixa | Recebimentos de caixa | Configuração geral |
| Múltiplas alíquotas | Regime de Competência | Linhas de receita da DRE | Tabela **Percentuais de Presunção** |

No Regime de Competência, cadastre os percentuais na tabela **Percentuais de Presunção** (ao menos uma linha). Com a tabela vazia, o sistema obtém a receita do **Nº Linha Receita DRE** e aplica os campos **% Presunção IRPJ do Reg. Caixa** e **% Presunção CSLL do Reg. Caixa**, que ficam desabilitados fora do Regime de Caixa; sem valor neles, a presunção resulta zero.

---

## 2. Configuração - Alíquota Única (Regime de Caixa)

### Passo 1: Acessar a configuração

1. Navegue até a página **Configuração Apuração IRPJ/CSLL** (`IRPJCSLLSettlementSetupFNX`)
2. No campo **Regime de Tributação** (`TributationRegime`), selecione **Lucro Presumido**
3. No campo **Critério de Escrituração do Regime Cumulativo** (`CRegimeBookkeepingType`), selecione **Regime de Caixa**

### Passo 2: Configurar o percentual de presunção

Preencha os campos:

- **% Presunção IRPJ do Reg. Caixa** (`IRPJCashRegAssumptOfProfitPerc`), ex.: 8,00
- **% Presunção CSLL do Reg. Caixa** (`CSLLCashRegAssumptOfProfitPerc`), ex.: 12,00

**Nota**: estes campos ficam habilitados apenas quando o critério de escrituração é **Regime de Caixa**.

### Passo 3: Configurar o acréscimo de presunção

Na seção **Presunção Acrescida IRPJ**:

1. Preencha **% Acréscimo** (`IRPJPresumptSurchargePerc`) com o percentual de acréscimo sobre a presunção normal (ex.: 10 para 10%)
2. Preencha **Limite Trimestral** (`IRPJPresumptQuarterlyExempt`) com o valor da receita bruta trimestral isenta do acréscimo (ex.: 1.250.000,00)

Repita na seção **Presunção Acrescida CSLL**, se aplicável.

**Nota**: se o **% Acréscimo** for zero, o sistema calcula a presunção de forma simples, sem acréscimo. Se o **Limite Trimestral** for zero com o **% Acréscimo** preenchido, toda a receita é tributada pela alíquota acrescida.

### Exemplo de configuração

| Campo | Valor |
| --- | --- |
| **% Presunção IRPJ do Reg. Caixa** | 8,00 |
| **% Acréscimo** (IRPJ) | 10,00 |
| **Limite Trimestral** (IRPJ) | 1.250.000,00 |

---

## 3. Configuração - Múltiplas Alíquotas (Regime de Competência)

### Quando usar

Empresas com múltiplas atividades que possuem alíquotas de presunção diferentes (ex.: serviços a 32% e mercadorias a 8%) devem usar a tabela de percentuais de presunção. Cada tipo de receita é configurado com sua própria linha da DRE e sua alíquota.

### Passo 1: Configuração geral

1. Navegue até a página **Configuração Apuração IRPJ/CSLL** (`IRPJCSLLSettlementSetupFNX`)
2. Defina **Regime de Tributação** como **Lucro Presumido**
3. Defina **Critério de Escrituração do Regime Cumulativo** como **Regime de Competência**
4. Configure a DRE: **Nome Esquema Contas DRE** (`DREAccScheduleName`), **Nome Layout Coluna DRE** (`DREColumnLayoutName`), **Nº Coluna Resultado DRE** (`DREResultColumnNo`), **Nº Linha Resultado DRE IRPJ** (`DREResultLineNo`) e **Nº Linha Resultado DRE CSLL** (`CSLLDREResultLineNo`)

**Nota**: no Regime de Competência, os campos **% Presunção IRPJ/CSLL do Reg. Caixa** ficam desabilitados. As alíquotas são definidas na tabela.

### Passo 2: Configurar a tabela de percentuais de presunção

1. Na página de configuração, clique na ação **Percentuais de Presunção** (`PresumptionPercentages`) na barra de ações
2. Adicione uma linha para cada tipo de receita:

| **Nº Linha DRE** | **Descrição** | **% Presunção IRPJ** | **% Presunção CSLL** |
| --- | --- | --- | --- |
| 100000 | Receita de Serviços | 32,00 | 32,00 |
| 200000 | Receita de Mercadorias | 8,00 | 12,00 |

O campo **Nº Linha DRE** deve apontar para a linha do Esquema de Contas (DRE) que contém a receita bruta daquele tipo de atividade.

### Passo 3: Configurar o acréscimo de presunção

Configure o **% Acréscimo** e o **Limite Trimestral** nas seções de **Presunção Acrescida IRPJ** e **Presunção Acrescida CSLL**, da mesma forma que no Regime de Caixa. O **% Acréscimo** representa o percentual de aumento sobre a alíquota de presunção normal de cada atividade:

- Serviços (32%): alíquota acrescida = 32% × (1 + 10/100) = **35,20%**
- Mercadorias (8%): alíquota acrescida = 8% × (1 + 10/100) = **8,80%**

### Exemplo de configuração

Configuração geral:

| Campo | Valor |
| --- | --- |
| **% Acréscimo** (IRPJ) | 10,00 |
| **Limite Trimestral** (IRPJ) | 1.250.000,00 |

Tabela **Percentuais de Presunção IRPJ/CSLL** (`IRPJCSLLPresumptionPercentage`):

| **Descrição** | **% Presunção IRPJ** | **% Presunção CSLL** |
| --- | --- | --- |
| Receita de Serviços | 32,00 | 32,00 |
| Receita de Mercadorias | 8,00 | 12,00 |

---

## 4. Configuração - Receitas Financeiras

### O que é

No Lucro Presumido, as **receitas financeiras** (juros, descontos obtidos, rendimentos de aplicações etc.) são adicionadas **integralmente** (100%) à base de cálculo do IRPJ e da CSLL, sem passar pelo percentual de presunção. Ou seja, enquanto a receita operacional é multiplicada pelo percentual de presunção, as receitas financeiras entram diretamente na base.

### Configuração

1. Navegue até a página **Configuração Apuração IRPJ/CSLL** (`IRPJCSLLSettlementSetupFNX`)
2. Na seção **Lucro/Prejuízo**, preencha o campo **Nº Linha Receita Financeira DRE** (`DREFinancialRevenueLineNo`) com a linha do Esquema de Contas (DRE) que contém o total de receitas financeiras do período

**Nota**: este campo é **opcional**. Se não for preenchido, o cálculo não inclui receitas financeiras, e o comportamento anterior é mantido. A DRE (Esquema de Contas, Nome Layout Coluna e Nº Coluna Resultado) deve estar configurada para que o campo funcione.

### Fórmula da base de cálculo

Com receitas financeiras configuradas, a base de cálculo passa a ser:

`Base de Cálculo = Valor da Presunção + Receitas Financeiras`

Sem receitas financeiras (campo não preenchido):

`Base de Cálculo = Valor da Presunção`

### Exemplo

| Componente | Valor |
| --- | --- |
| Receita Operacional | 1.000.000,00 |
| % Presunção IRPJ | 8,00% |
| **Valor da Presunção** | 80.000,00 |
| **Receitas Financeiras** (da DRE) | 15.000,00 |
| **Base de Cálculo** | 80.000 + 15.000 = **95.000,00** |

---

## 5. Configuração - Modo Híbrido de Acréscimo

### O que é

A IN RFB 2.305/2025 prevê apuração trimestral granular, com cálculo separado da parcela do trimestre dentro do limite e da parcela excedente. Algumas empresas, por opção conservadora, preferem aplicar a alíquota acrescida sobre **toda** a receita do trimestre a partir do momento em que o limite anual de R$ 5.000.000,00 for ultrapassado, evitando discussões interpretativas com a Receita. Esse comportamento é o **Modo Híbrido**.

### Configuração

1. Navegue até a página **Configuração Apuração IRPJ/CSLL** (`IRPJCSLLSettlementSetupFNX`)
2. Na seção **Modo de Acréscimo** (`PresumptionSurchargeModeGroup`), marque o campo **Modo Híbrido de Acréscimo** (`HybridSurchargeMode`)

A seção fica habilitada apenas no regime de **Lucro Presumido**. O padrão é **desativado** (cálculo conforme IN RFB 2.305/2025). O Modo Híbrido se aplica apenas ao cálculo com alíquota única; com múltiplas alíquotas (tabela **Percentuais de Presunção**), ele não tem efeito.

### Comportamento

- **Desativado (padrão)**: cálculo granular conforme IN RFB 2.305/2025, com limite trimestral, acumulação de saldo (§ 4º) e ajuste anual no Q4 (§ 5º).
- **Ativado**: quando a **Receita Acumulada do Ano até o trimestre anterior** já ultrapassou o **Limite Anual** (Limite Trimestral × 4 = R$ 5.000.000,00), o limite isento do trimestre atual é zerado, e toda a receita do trimestre é tributada pela **Alíquota Acrescida**. Tipicamente paga mais imposto que o modo padrão.

**Importante**: o **Modo Híbrido não desativa o ajuste anual** do § 5º. Se ao final do ano a receita anual ficar abaixo do limite anual, o estorno proporcional ainda é gerado no Q4.

---

## 6. Comportamento no Cálculo

### Fórmula da alíquota acrescida

A alíquota acrescida de cada atividade é calculada como:

`Alíquota Acrescida = % Presunção Normal × (1 + % Acréscimo / 100)`

Exemplos:
- 32% × (1 + 10/100) = **35,20%**
- 8% × (1 + 10/100) = **8,80%**

### § 4º — Acumulação de saldo de limite entre trimestres

O **Limite Trimestral** é aplicado individualmente a cada trimestre. Quando a receita de um trimestre fica **abaixo** do limite, a **diferença não utilizada é acumulada** e aumenta o limite efetivo do trimestre seguinte (§ 4º art. 15 IN RFB 2.305/2025). O saldo é aplicado apenas no cálculo com alíquota única; com múltiplas alíquotas, ele não é considerado.

`Saldo Acumulado de Limite = Σ Max(0, Limite Trimestral - Receita do Trimestre_i) para Trimestre_i < Trimestre Atual`

Trimestres em que a receita **excedeu** o limite não geram saldo negativo. Eles contribuem com zero ao Saldo Acumulado.

O cálculo da receita de cada trimestre passado depende do cenário:

- **Regime de Caixa**: soma dos recebimentos de caixa do trimestre
- **Regime de Competência (alíquota única)**: receita via **Nº Linha Receita DRE** do trimestre

### Limite remanescente do trimestre atual (apuração mensal antecipada)

Se a empresa apura mensalmente dentro do mesmo trimestre, o sistema também desconta a receita dos meses anteriores do trimestre atual:

`Limite Remanescente = Limite Trimestral + Saldo Acumulado - Receita dos Meses Anteriores do Trimestre Atual`

(O Saldo Acumulado refere-se aos **trimestres** anteriores; a subtração dos **meses** anteriores cobre o avanço mensal dentro do trimestre.)

### Cálculo com alíquota única

1. O sistema verifica se o **% Acréscimo** está configurado (> 0)
2. Calcula o **Saldo Acumulado de Limite** dos trimestres anteriores do mesmo ano (§ 4º)
3. Subtrai a receita dos meses anteriores do trimestre atual e obtém o **Limite Remanescente**
4. Se o **Modo Híbrido** estiver ativo, calcula a **Receita Acumulada do Ano** e, se ela já ultrapassou o limite anual, zera o **Limite Remanescente** (toda a receita do trimestre vai para a alíquota acrescida)
5. Se for Q4 com período fechado e o ajuste do § 5º se aplicar, iguala o **Limite Remanescente** à receita do trimestre (o Q4 não cobra acréscimo; o estorno proporcional é gerado em lançamento separado)
6. **Se a receita excede o Limite Remanescente**:
   - **Presunção (Normal)** = Limite Remanescente × % Presunção / 100
   - **Presunção (Acrescida)** = (Receita - Limite Remanescente) × Alíquota Acrescida / 100
   - **Valor da Presunção** = Presunção Normal + Presunção Acrescida
7. **Se a receita NÃO excede**:
   - Cálculo normal: Receita × % Presunção / 100
   - **Presunção (Acrescida)** é zero

### Cálculo com múltiplas alíquotas

Aplicado **apenas em Regime de Competência** com pelo menos uma linha cadastrada na tabela **Percentuais de Presunção**. No Regime de Caixa, linhas eventualmente cadastradas são ignoradas (ver "Correção" abaixo). Neste cálculo não se aplicam o saldo acumulado do § 4º nem o **Modo Híbrido de Acréscimo**, e o Q4 não é calculado sem acréscimo.

1. Para cada linha da tabela **Percentuais de Presunção**, o sistema lê a receita da respectiva linha da DRE
2. Calcula a presunção a alíquotas normais para toda a receita:
   - `Presunção Normal = Σ(Receita_i × % Presunção_i / 100)`
3. Calcula a presunção a alíquotas acrescidas para toda a receita:
   - `Presunção Acrescida = Σ(Receita_i × Alíquota Acrescida_i / 100)`
4. Soma a receita total de todas as linhas
5. Determina o **Limite Remanescente** (Limite Trimestral - Receita dos meses anteriores do trimestre atual, mínimo 0)
6. Calcula o excesso: `Excesso = Receita Total - Limite Remanescente` (mínimo 0)
7. Se houver excesso, calcula o acréscimo proporcionalmente:
   - `Acréscimo = (Excesso / Receita Total) × (Presunção Acrescida - Presunção Normal)`
8. **Valor da Presunção** = Presunção Normal + Acréscimo

### Correção de cálculo multi-alíquota em Regime de Caixa (maio/2026)

Antes desta versão, se houvesse qualquer linha cadastrada na tabela **Percentuais de Presunção**, o cálculo multi-alíquota disparava mesmo em Regime de Caixa. Ele sobrescrevia a **Receita** com a soma das linhas da DRE e ignorava a alíquota única do Setup. A correção aplicada limita o multi-alíquota ao Regime de Competência. Clientes em Regime de Caixa com linhas remanescentes na tabela voltam a calcular corretamente com a alíquota única, sem necessidade de limpar a tabela.

### § 5º — Ajuste anual no Q4

No último trimestre do ano-calendário (Quarter = 4 e **Fim do Período** marcado), o sistema verifica se o acréscimo cobrado ao longo do ano foi maior que o devido pelo total anual e, se for o caso, gera um estorno proporcional:

Este ajuste segue o cálculo com alíquota única. Com múltiplas alíquotas, o Q4 não é calculado sem acréscimo, e o estorno usa os campos **% Presunção IRPJ do Reg. Caixa** e **% Presunção CSLL do Reg. Caixa**, que ficam desabilitados no Regime de Competência; sem valor neles, nenhum estorno é gerado.

**Cenário I — Receita anual < Limite anual**:
- Nenhum acréscimo deveria ter sido cobrado no ano
- O Q4 é calculado sem acréscimo
- O acréscimo cobrado em trimestres anteriores é estornado integralmente

**Cenário II — Receita anual > Limite anual, mas Σ(Excessos Trimestrais Q1-Q3) > Excesso Anual**:
- Parte do acréscimo cobrado foi maior que o devido, porque excessos pontuais em alguns trimestres foram compensados por receitas baixas em outros
- O Q4 é calculado sem acréscimo
- A parcela excedente do acréscimo é estornada proporcionalmente

O estorno é gerado automaticamente como um lançamento na página **Ajustes Apuração IRPJ/CSLL** (`IRPJCSLLSettlAdjustFNX`):

| Campo | Valor |
| --- | --- |
| **Tributo** | IRPJ ou CSLL |
| **Tipo** | Dedução |
| **Descrição** | Ajuste Anual - Estorno de Acréscimo (Art. 15 § 5º) |
| **Valor** | Valor do estorno |
| **Lançamento Automático** | Marcado |

Cálculo do valor do estorno:

```
Base a Estornar         = Σ(Excessos Trimestrais Q1-Q3) - Excesso Anual
Estorno na Presunção    = Base a Estornar × (% Presunção Normal × % Acréscimo / 100)
Valor do Estorno        = Estorno na Presunção × Alíquota do Tributo (15% IRPJ ou 9% CSLL)
```

Os lançamentos marcados como **Lançamento Automático** com descrição **Ajuste Anual...** são **idempotentes**: são recriados a cada execução da apuração do Q4. Não é necessário, nem recomendado, editá-los manualmente.

### Exemplo numérico - § 4º (Acumulação de saldo)

Empresa com presunção IRPJ 32%, **% Acréscimo** 10%, **Limite Trimestral** R$ 1.250.000,00.

| Trimestre | Receita | Saldo Acumulado | Limite Efetivo | À 32% | À 35,20% | Valor Presunção |
| --- | --- | --- | --- | --- | --- | --- |
| Q1 | 800.000,00 | 0 | 1.250.000,00 | 800.000,00 | 0 | 256.000,00 |
| Q2 | 1.700.000,00 | 450.000,00 | 1.700.000,00 | 1.700.000,00 | 0 | 544.000,00 |
| Q3 | 1.500.000,00 | 0 | 1.250.000,00 | 1.250.000,00 | 250.000,00 | 488.000,00 |

Sem o § 4º, o Q2 teria pago acréscimo sobre R$ 450.000,00 indevidamente (R$ 450.000 × 3,20% = R$ 14.400 a mais na presunção).

### Exemplo numérico - § 5º Cenário I (Estorno total no Q4)

Q1 = Q2 = Q3 = R$ 1.500.000,00 cada (cada um excedeu o limite em R$ 250.000,00). Q4 = R$ 200.000,00. Receita anual = R$ 4.700.000,00 (< R$ 5.000.000,00).

| Componente | Cálculo | Valor |
| --- | --- | --- |
| Σ Excessos Q1-Q3 | 3 × 250.000 | 750.000,00 |
| Excesso Anual | 4.700.000 - 5.000.000 (mín. 0) | 0 |
| Base a Estornar | 750.000 - 0 | 750.000,00 |
| Estorno na Presunção | 750.000 × 32% × 10% / 100 | 24.000,00 |
| **Valor do Estorno IRPJ** | 24.000 × 15% | **3.600,00** |

No Q4, a apuração é feita sem acréscimo e, no demonstrativo de **Ajustes**, aparece um lançamento de Dedução de R$ 3.600,00 (IRPJ). Para CSLL com presunção normal de 12% e alíquota 9%, o estorno seria 750.000 × 12% × 10% / 100 × 9% = R$ 810,00.

### Exemplo numérico - § 5º Cenário II (Estorno proporcional)

Q1 = R$ 8.000.000,00, Q2 = R$ 2.200.000,00, Q3 = R$ 2.200.000,00, Q4 = R$ 500.000,00. Receita anual = R$ 12.900.000,00. Excesso Anual = R$ 7.900.000,00.

| Componente | Cálculo | Valor |
| --- | --- | --- |
| Σ Excessos Q1-Q3 | 6.750.000 + 950.000 + 950.000 | 8.650.000,00 |
| Excesso Anual | 12.900.000 - 5.000.000 | 7.900.000,00 |
| Base a Estornar | 8.650.000 - 7.900.000 | 750.000,00 |
| Estorno na Presunção | 750.000 × 32% × 10% / 100 | 24.000,00 |
| **Valor do Estorno IRPJ** | 24.000 × 15% | **3.600,00** |

### Exemplo numérico - Alíquota única (Q3 com excesso)

Apuração do 3º mês de um trimestre. Presunção de 8%, **% Acréscimo** de 10%, **Limite Trimestral** de R$ 1.250.000,00. Sem saldo acumulado de trimestres anteriores.

| Componente | Cálculo | Valor |
| --- | --- | --- |
| Receita do Mês | -- | 900.000,00 |
| **Limite Trimestral** | -- | 1.250.000,00 |
| Receita Acumulada Anterior | -- | 1.100.000,00 |
| Limite Remanescente | 1.250.000 - 1.100.000 | 150.000,00 |
| Parcela Excedente | 900.000 - 150.000 | 750.000,00 |
| % Presunção Normal | -- | 8,00% |
| Alíquota Acrescida | 8% × (1 + 10/100) | 8,80% |
| **Presunção (Normal)** | 150.000 × 8% / 100 | 12.000,00 |
| **Presunção (Acrescida)** | 750.000 × 8,80% / 100 | 66.000,00 |
| **Valor da Presunção** | 12.000 + 66.000 | **78.000,00** |

Sem o acréscimo, a presunção seria 900.000 × 8% = R$ 72.000,00. Com o acréscimo, R$ 78.000,00 (R$ 6.000,00 a mais).

### Exemplo numérico - Múltiplas alíquotas

Receita total: R$ 1.500.000,00 (Serviços: R$ 1.000.000, Mercadorias: R$ 500.000). **Limite Trimestral**: R$ 1.250.000,00. **% Acréscimo**: 10%. Sem saldo acumulado.

| Componente | Cálculo | Valor |
| --- | --- | --- |
| Receita Total | 1.000.000 + 500.000 | 1.500.000,00 |
| Presunção Normal Total | 1.000.000 × 32% + 500.000 × 8% | 360.000,00 |
| Presunção Acrescida Total | 1.000.000 × 35,2% + 500.000 × 8,8% | 396.000,00 |
| **Limite Trimestral** | -- | 1.250.000,00 |
| Excesso | 1.500.000 - 1.250.000 | 250.000,00 |
| Proporção do excesso | 250.000 / 1.500.000 | 16,67% |
| Acréscimo | 16,67% × (396.000 - 360.000) | 6.000,00 |
| **Valor da Presunção** | 360.000 + 6.000 | **366.000,00** |

Sem o acréscimo, a presunção seria R$ 360.000,00. Com o acréscimo, R$ 366.000,00 (R$ 6.000,00 a mais).

### Exemplo numérico - Modo Híbrido (cliente conservador)

Empresa com Modo Híbrido ativado, presunção IRPJ 32%, **% Acréscimo** 10%, **Limite Trimestral** R$ 1.250.000,00. Q1 = R$ 6.024.172,55 (já ultrapassou sozinho o limite anual de R$ 5.000.000). Apuração do Q2 com receita R$ 2.233.249,23.

| Componente | Cálculo | Valor |
| --- | --- | --- |
| Receita Acumulada do Ano (Q1) | -- | 6.024.172,55 |
| Limite Anual | 1.250.000 × 4 | 5.000.000,00 |
| Anual já ultrapassado? | 6.024.172,55 ≥ 5.000.000 | Sim |
| Limite Remanescente (Q2) | Override Híbrido | 0,00 |
| **Presunção (Acrescida)** | 2.233.249,23 × 35,20% | 786.103,73 |
| **Valor da Presunção** | -- | **786.103,73** |

No modo padrão (IN), o Q2 mantém o **Limite Trimestral** de R$ 1.250.000,00 (o Saldo Acumulado de Q1 é zero, porque o Q1 estourou): aplica 32% sobre R$ 1.250.000,00 (R$ 400.000,00) e 35,20% sobre R$ 983.249,23 (R$ 346.103,73), totalizando R$ 746.103,73. O Modo Híbrido gera R$ 40.000,00 a mais de presunção.

---

## 7. Demonstrativo

Quando o acréscimo de presunção está configurado (**Limite Trimestral** > 0), a página **Apuração IRPJ/CSLL** (`IRPJCSLLSettlementFNX`) exibe uma seção adicional com o detalhamento:

| Linha | Descrição |
| --- | --- |
| **Limite Trimestral Isento** | Valor do limite de receita bruta trimestral isenta do acréscimo |
| **Saldo Acumulado de Limite** | Saldo do § 4º: soma do limite não utilizado em trimestres anteriores do ano. Visível apenas quando > 0 |
| **Receita Acumulada** | Receita bruta total acumulada do ano até o trimestre anterior. Visível apenas quando > 0 (no Modo Híbrido, usada para apurar se o limite anual já foi ultrapassado) |
| **Presunção (Normal)** | Valor da presunção à alíquota normal |
| **% Alíquota Acrescida** | Percentual efetivo da presunção acrescida (apenas alíquota única) |
| **Presunção (Acrescida)** | Valor da presunção sobre a parcela excedente |

Esta seção aparece dentro de um grupo **Acréscimo de Presunção**, visível apenas quando o **Limite Trimestral Isento** é maior que zero.

O campo **% Presunção do Lucro** é exibido apenas no Regime de Caixa (alíquota única). No Regime de Competência com múltiplas alíquotas, as alíquotas individuais estão na tabela **Percentuais de Presunção**.

Quando configurado, o campo **Valor Receita Financeira** também é exibido na seção **Resultado**, logo acima do **Valor Base**. Este campo mostra as receitas financeiras do período, que são somadas integralmente à base de cálculo.

Quando o ajuste do § 5º é gerado, a página **Ajustes Apuração IRPJ/CSLL** mostra um ou dois lançamentos do tipo **Dedução** com descrição **Ajuste Anual - Estorno de Acréscimo (Art. 15 § 5º)**, marcados como **Lançamento Automático**. Esses lançamentos são recalculados automaticamente a cada execução da apuração do Q4.

---

## 8. Resumo dos Campos Novos

### Configuração Apuração IRPJ/CSLL (`IRPJCSLLSettlementSetupFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **% Presunção IRPJ do Reg. Caixa** (`IRPJCashRegAssumptOfProfitPerc`) | Decimal | Percentual de presunção de lucro para IRPJ. Habilitado apenas no Regime de Caixa |
| **% Presunção CSLL do Reg. Caixa** (`CSLLCashRegAssumptOfProfitPerc`) | Decimal | Percentual de presunção de lucro para CSLL. Habilitado apenas no Regime de Caixa |
| **Nº Linha Receita DRE** (`DRERevenueLineNo`) | Inteiro | Linha da DRE com a receita bruta total. Usado no Regime de Competência quando a tabela **Percentuais de Presunção** está vazia |
| **Nº Linha Receita Financeira DRE** (`DREFinancialRevenueLineNo`) | Inteiro | Linha da DRE com receitas financeiras. Opcional. Se preenchido, o valor é somado integralmente à base de cálculo |
| **% Acréscimo** (`IRPJPresumptSurchargePerc` / `CSLLPresumptSurchargePerc`) | Decimal | Percentual de acréscimo sobre a presunção normal (0 a 100) |
| **Limite Trimestral** (`IRPJPresumptQuarterlyExempt` / `CSLLPresumptQuarterlyExempt`) | Decimal | Receita bruta trimestral isenta do acréscimo de presunção |
| **Modo Híbrido de Acréscimo** (`HybridSurchargeMode`) | Booleano | Quando ativo, aplica a alíquota acrescida sobre toda a receita do trimestre se a receita acumulada do ano já ultrapassou o limite anual. Disponível apenas no Lucro Presumido e aplicado apenas no cálculo com alíquota única |

### Percentual de Presunção IRPJ/CSLL (`IRPJCSLLPresumptionPercentage`) - Tabela nova

Acessível via ação **Percentuais de Presunção** na página de configuração. Habilitada quando o critério de escrituração **não** é Regime de Caixa.

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Nº Linha** (`LineNo`) | Inteiro | Número sequencial da linha |
| **Nº Linha DRE** (`DRELineNo`) | Inteiro | Linha do Esquema de Contas (DRE) com a receita bruta desta atividade |
| **Descrição** (`Description`) | Texto | Descrição do tipo de receita (ex.: "Receita de Serviços") |
| **% Presunção IRPJ** (`IRPJPresumptionPerc`) | Decimal | Percentual de presunção de lucro para IRPJ desta atividade |
| **% Presunção CSLL** (`CSLLPresumptionPerc`) | Decimal | Percentual de presunção de lucro para CSLL desta atividade |

### Apuração IRPJ/CSLL (`IRPJCSLLSettlementFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Lucro antes da Presunção** (`IRPJProfitBeforeAssumption` / `CSLLProfitBeforeAssumption`) | Decimal | Receita bruta do período (caixa ou DRE) |
| **% Presunção do Lucro** (`IRPJProfitAssumptPerc` / `CSLLProfitAssumptPerc`) | Decimal | Percentual de presunção aplicado. Visível apenas no Regime de Caixa |
| **Valor da Presunção** (`IRPJPresumptionAmount` / `CSLLPresumptionAmount`) | Decimal | Presunção total calculada (Normal + Acrescida) |
| **Limite Trimestral Isento** (`IRPJPresumptQuarterlyExempt` / `CSLLPresumptQuarterlyExempt`) | Decimal | Limite trimestral copiado da configuração no momento do cálculo |
| **Saldo Acumulado de Limite IRPJ/CSLL** (`IRPJUnusedExemptCarried` / `CSLLUnusedExemptCarried`) | Decimal | Saldo do § 4º: soma do limite não utilizado em trimestres anteriores do ano. Visível apenas quando > 0 |
| **Receita Acumulada IRPJ/CSLL** (`IRPJPriorYearRevenue` / `CSLLPriorYearRevenue`) | Decimal | Receita bruta acumulada do ano até o trimestre anterior. Utilizada pelo Modo Híbrido para apurar se o limite anual foi ultrapassado. Visível apenas quando > 0 |
| **Presunção (Normal)** (`IRPJPresumptNormalAmt` / `CSLLPresumptNormalAmt`) | Decimal | Valor da presunção à alíquota normal |
| **% Alíquota Acrescida** (`IRPJPresumptSurchargePerc` / `CSLLPresumptSurchargePerc`) | Decimal | Alíquota efetiva de presunção acrescida |
| **Presunção (Acrescida)** (`IRPJPresumptSurchargeAmt` / `CSLLPresumptSurchargeAmt`) | Decimal | Valor da presunção sobre a parcela excedente |
| **Valor Receita Financeira** (`IRPJFinancialRevenueAmount` / `CSLLFinancialRevenueAmount`) | Decimal | Receitas financeiras do período, somadas integralmente à base de cálculo. Visível apenas no Lucro Presumido |

### Ajuste Apuração IRPJ/CSLL (`IRPJCSLLSettlAdjustFNX`)

Lançamentos gerados automaticamente pelo § 5º no Q4, quando aplicável:

| Característica | Valor |
| --- | --- |
| **Tipo** | Dedução |
| **Descrição** | Ajuste Anual - Estorno de Acréscimo (Art. 15 § 5º) |
| **Lançamento Automático** | Marcado |
| **Idempotência** | Recriado a cada recálculo da apuração do Q4 |

### Compatibilidade

- **Clientes existentes (versão de abril/2026)**: continuam com o cálculo trimestral granular. O **§ 4º** (acumulação) e o **§ 5º** (ajuste Q4) passam a ser aplicados automaticamente no cálculo com alíquota única, sem necessidade de configuração adicional.
- **Modo Híbrido**: campo novo `HybridSurchargeMode`, default `false`. Quem não ativar mantém o cálculo da IN.
- **Correção multi-alíquota**: clientes em Regime de Caixa com linhas remanescentes na tabela **Percentuais de Presunção** voltam a calcular corretamente com a alíquota única do Setup. Não é necessário limpar a tabela.
- **Tabela `Percentuais de Presunção` vazia**: o sistema utiliza o fluxo de alíquota única (configuração geral).
- **Regime de Caixa**: continua funcionando com a alíquota única da configuração.
- **Regime de Competência sem tabela**: a receita vem do **Nº Linha Receita DRE** e o percentual, dos campos **% Presunção IRPJ do Reg. Caixa** e **% Presunção CSLL do Reg. Caixa**, que ficam desabilitados fora do Regime de Caixa; sem valor neles, a presunção resulta zero. Cadastre os percentuais na tabela **Percentuais de Presunção**.
