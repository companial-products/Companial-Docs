<!-- Commits: 91d35e87 -->
# Balancete Brasil: Filtro por Filial

**Data**: Maio/2025

---

## Sumário

Esta atualização adiciona suporte a **filtro por filial** nos relatórios de **Balancete** e **Balancete Avançado**. Foram criados campos FlowField dedicados na tabela Conta Contábil que incluem o filtro de filial no cálculo dos saldos, permitindo que os relatórios exibam valores segregados por filial sem afetar os campos padrão do Business Central.

## 1. Contexto

### O que é

Empresas com múltiplas filiais frequentemente precisam emitir balancetes por filial para fins de gestão e obrigações fiscais. Os campos padrão do Business Central (`Saldo a Data`, `Variação Líquida`, `Valor Débito`, `Valor Crédito`) não possuem filtro por filial nativo.

Esta atualização cria campos equivalentes na Conta Contábil que incluem o **Filtro Filial** (`BranchFilterFNX`) na fórmula de cálculo. Os relatórios **Balancete** (`TrialBalanceFNX`) e **Balancete Avançado** (`TrialBalanceAdvancedFNX`) passaram a utilizar esses campos, permitindo a geração de balancetes filtrados por filial.

### Quando se aplica

- Empresas que utilizam a funcionalidade de filiais (dimensão de filial configurada)
- Ao emitir relatórios de Balancete ou Balancete Avançado com filtro de filial aplicado

---

## 2. Como Utilizar

1. Abra o relatório **Balancete** ou **Balancete Avançado** via pesquisa do Business Central
2. Na aba de filtros, aplique o **Filtro Filial** desejado
3. Execute o relatório

Os valores de débito, crédito, saldo e variação líquida serão calculados considerando apenas os movimentos da filial selecionada.

### Plano de Contas

Os campos de filial também estão disponíveis na página **Plano de Contas** (ocultos por padrão). Para visualizá-los:

1. Abra o **Plano de Contas**
2. Personalize a página para exibir os campos **Saldo a Data Filial**, **Variação Líquida Filial**, **Valor Débito Filial** e **Valor Crédito Filial**
3. Aplique o **Filtro Filial** para ver os saldos segregados

---

## 3. Campos Disponíveis no Plano de Contas

| Campo | Descrição |
|-------|-----------|
| **Saldo a Data Filial** (`BranchBalanceAtDateFNX`) | Saldo acumulado até a data filtrada, considerando apenas movimentos da filial selecionada |
| **Variação Líquida Filial** (`BranchNetChangeFNX`) | Variação líquida no período filtrado, considerando apenas movimentos da filial selecionada |
| **Valor Débito Filial** (`BranchDebitAmountFNX`) | Total de débitos no período filtrado, considerando apenas movimentos da filial selecionada |
| **Valor Crédito Filial** (`BranchCreditAmountFNX`) | Total de créditos no período filtrado, considerando apenas movimentos da filial selecionada |

---

## 4. Resumo dos Campos Novos

### Tabela Conta Contábil (`G/L Account` — extensão `GLAccountLocFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Saldo a Data Filial** (`BranchBalanceAtDateFNX`) | Decimal (FlowField) | Soma de movimentos até a data, filtrado por filial |
| **Variação Líquida Filial** (`BranchNetChangeFNX`) | Decimal (FlowField) | Soma de movimentos no período, filtrado por filial |
| **Valor Débito Filial** (`BranchDebitAmountFNX`) | Decimal (FlowField) | Soma de débitos no período, filtrado por filial |
| **Valor Crédito Filial** (`BranchCreditAmountFNX`) | Decimal (FlowField) | Soma de créditos no período, filtrado por filial |
| **Filtro Filial** (`BranchFilterFNX`) | Code[20] (FlowFilter) | Filtro para selecionar a filial desejada |
