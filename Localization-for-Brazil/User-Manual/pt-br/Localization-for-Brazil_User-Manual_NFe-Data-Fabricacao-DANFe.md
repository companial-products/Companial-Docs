<!-- Commits: 7a3c18d5 -->
# NF-e: Data de Fabricação no DANFe

**Data**: Julho/2025

---

## Sumário

Esta atualização adiciona a opção de exibir a **Data de Fabricação** dos lotes no DANFe, nas informações adicionais do produto. Anteriormente, apenas a Data de Validade podia ser exibida. Agora, o campo **Mostrar Data Fabricação** permite incluir a data de fabricação de cada lote junto às informações do produto no DANFe.

## 1. Contexto

### O que é

Em determinados segmentos (alimentos, medicamentos, cosméticos), é necessário que o DANFe contenha a **data de fabricação** dos lotes associados aos itens da NF-e, além da data de validade. Esta informação aparece nas informações adicionais do produto (campo `infAdProd` da NF-e) no formato `Fab: DD/MM/AAAA`.

### Quando se aplica

- Empresas que trabalham com rastreamento de lotes e precisam informar a data de fabricação no DANFe
- Requer que o campo **Mostrar Nº Lote** (`ShowLotNo`) esteja ativado na configuração da NF-e

---

## 2. Configuração

1. Navegue até **Configuração NF-e** (página `NFeSetupFNX`) via pesquisa do Business Central
2. Na seção **DANFe**, localize os campos:

| Campo | Descrição |
|-------|-----------|
| **Mostrar Nº Série** (`DANFeShowSerialNo`) | Exibe números de série no DANFe |
| **Mostrar Nº Lote** (`ShowLotNo`) | Exibe números de lote no DANFe. **Pré-requisito** para os campos de data abaixo. |
| **Mostrar Data Fabricação** (`DANFeShowProductionDate`) | Exibe a data de fabricação de cada lote no DANFe. Habilitado somente quando **Mostrar Nº Lote** está ativo. |
| **Mostrar Data de Validade** (`DANFeShowExpirationDate`) | Exibe a data de validade de cada lote no DANFe. Habilitado somente quando **Mostrar Nº Lote** está ativo. |

> **Nota**: Ao desmarcar **Mostrar Nº Lote**, os campos **Mostrar Data Fabricação** e **Mostrar Data de Validade** são automaticamente desmarcados.

---

## 3. Comportamento na Geração do DANFe

Quando **Mostrar Data Fabricação** está ativado, para cada lote encontrado nas linhas do documento fiscal, o sistema:

1. Obtém a data de fabricação conforme a configuração do campo **Campo Data Fabricação Lote** (`LotProductionDateField`):
   - **Primeiro Movimento de Entrada**: usa a data de registro do primeiro movimento positivo do lote
   - **Informações do Lote**: usa o campo **Data Fabricação** (`ProductionDateFNX`) do cadastro de informações do lote
2. Adiciona a informação no formato `Fab: DD/MM/AAAA` nas informações adicionais do produto no DANFe

### Exemplo de informações adicionais no DANFe

```
Lote: ABC123 Qtd: 100 Fab: 15/03/2025 Val: 15/03/2026
```
