<!-- Commits: fa63f09b -->
# Guias de Impostos DCTFWeb: Provisão por Data de Vencimento

**Data**: Outubro/2025

---

## Sumário

Esta atualização adiciona a opção de separar as provisões contábeis da DCTFWeb **por data de vencimento**. Anteriormente, a provisão era gerada em um único lançamento por filial (ou um único lançamento geral). Agora, ao ativar o flag **Provisão por Data Vencimento**, o sistema gera um lançamento contábil para cada data de vencimento distinta encontrada nos documentos da apuração, utilizando a data de vencimento real de cada guia como data de vencimento do movimento do fornecedor.

## 1. Contexto

### O que é

No processo de apuração de guias de impostos (DCTFWeb), o sistema gera provisões contábeis — lançamentos a pagar contra o fornecedor da DCTFWeb. Até esta atualização, todas as guias de uma mesma apuração eram consolidadas em um único lançamento, com a data de vencimento calculada pela fórmula configurada em **Fórmula Data Vencimento da Provisão** (`DCTFWebProvDueDateFormulaFNX`).

O problema é que diferentes tributos da DCTFWeb podem ter datas de vencimento distintas (ex: IRRF vence dia 20, INSS vence dia 15). Ao consolidar tudo em um único lançamento, o controle de contas a pagar por vencimento ficava impreciso.

### Quando se aplica

Esta funcionalidade é útil para empresas que precisam de visibilidade individual das provisões da DCTFWeb por data de vencimento, especialmente para conciliação de contas a pagar e controle de fluxo de caixa.

---

## 2. Configuração

1. Navegue até **Conf. Fiscais** (página `FiscalSetupFNX`) via pesquisa do Business Central
2. Na seção **DCTFWeb**, localize o campo **Provisão por Data Vencimento** (`DCTFWebProvPerDueDateFNX`)
3. Marque como verdadeiro

### Comportamento dos campos relacionados

- Quando **Provisão por Data Vencimento** está ativado, o campo **Fórmula Data Vencimento da Provisão** (`DCTFWebProvDueDateFormulaFNX`) é desabilitado e seu valor ocultado, pois a data de vencimento será obtida diretamente de cada documento da apuração
- Quando desativado, o comportamento permanece o anterior: a data de vencimento é calculada pela fórmula a partir da data final da apuração

---

## 3. Comportamento na Contabilização

### Com Provisão por Data Vencimento desativado (padrão)

O sistema soma todos os documentos DCTFWeb da apuração e gera **um único lançamento contábil** com a data de vencimento calculada pela fórmula.

### Com Provisão por Data Vencimento ativado

1. O sistema percorre os documentos da apuração e identifica todas as **datas de vencimento distintas**
2. Para cada data de vencimento, soma os valores dos documentos correspondentes
3. Gera um **lançamento contábil separado** para cada grupo, com a data de vencimento real da guia
4. A data de vencimento é ajustada para o dia útil anterior caso caia em fim de semana ou feriado

---

## 4. Combinação com Provisão por Filial

Os dois flags podem ser combinados:

| Provisão por Filial | Provisão por Data Vencimento | Resultado |
|---------------------|------------------------------|-----------|
| Desativado | Desativado | Um único lançamento consolidado |
| Ativado | Desativado | Um lançamento por filial |
| Desativado | Ativado | Um lançamento por data de vencimento |
| Ativado | Ativado | Um lançamento por combinação de filial + data de vencimento |

---

## 5. Resumo dos Campos Novos

### Tabela Conf. Fiscais (`FiscalSetupFNX` — extensão `FiscalSetupTaxFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Provisão DCTFWeb por Data Vencimento** (`DCTFWebProvPerDueDateFNX`) | Boolean | Quando ativado, gera provisões separadas por data de vencimento. Desabilita o campo Fórmula Data Vencimento da Provisão. |
