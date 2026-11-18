<!-- Commits: ba3400d8 -->
# Nº Item Fiscal na Conta Contábil

**Data**: Dezembro/2025

---

## Sumário

Esta atualização adiciona o campo **Nº Item Fiscal** ao cartão da **Conta Contábil**, permitindo associar uma conta contábil a um item fiscal para fins de classificação tributária em documentos eletrônicos (NF-e/NFS-e). Quando preenchido, o item fiscal é propagado automaticamente para as linhas de compras e vendas ao selecionar uma conta contábil como tipo de linha.

## 1. Contexto

### O que é

Em operações fiscais brasileiras, linhas de documentos com tipo **Conta Contábil** frequentemente precisam de classificação fiscal (NCM, CFOP, unidade de medida, etc.) para emissão de NF-e/NFS-e. Até agora, era necessário preencher manualmente o **Nº Item Fiscal** (`FiscalItemNoFNX`) em cada linha do documento.

Com esta atualização, é possível configurar um item fiscal diretamente no cadastro da conta contábil. Ao inserir uma linha de compra ou venda com tipo Conta Contábil, o sistema preenche automaticamente o **Nº Item Fiscal** a partir da conta, que por sua vez carrega NCM, grupo contábil de produto, unidade de medida e demais dados fiscais do item.

### Quando se aplica

- Empresas que emitem NF-e/NFS-e com linhas do tipo Conta Contábil
- Cenários de prestação de serviços ou despesas onde a conta contábil precisa de classificação fiscal

---

## 2. Configuração

### Habilitando o campo

O campo **Nº Item Fiscal** na conta contábil é controlado por uma configuração global. Para habilitá-lo:

1. Navegue até **Conf. Fiscais** (página `FiscalSetupFNX`) via pesquisa do Business Central
2. No grupo **Documento Fiscal**, localize o campo **Mostrar Nº Item Fiscal na Conta Contábil** (`ShowFiscalItemNoOnGLAccount`)
3. Marque como verdadeiro

> **Nota**: Se este campo estiver desmarcado, a coluna **Nº Item Fiscal** não será visível no cartão da conta contábil.

### Preenchendo o item fiscal na conta contábil

1. Navegue até o **Cartão Conta Contábil** (página `G/L Account Card`)
2. Na seção **Fiscal**, preencha o campo **Nº Item Fiscal** (`FiscalItemNoFNX`)
3. O campo aceita apenas itens existentes na tabela de itens do Business Central

O item fiscal configurado deve possuir as classificações fiscais necessárias (NCM, grupo contábil de produto, unidade de medida) para que a propagação funcione corretamente nos documentos.

---

## 3. Comportamento em Compras e Vendas

Ao criar uma linha de compra ou venda com **Tipo** = **Conta Contábil**:

1. O sistema verifica se a conta contábil selecionada possui **Nº Item Fiscal** preenchido
2. Se sim, o valor é automaticamente propagado para o campo **Nº Item Fiscal** (`FiscalItemNoFNX`) da linha
3. A validação do item fiscal na linha carrega automaticamente:
   - **Descrição** e **Descrição 2**
   - **Grupo Contábil de Produto**
   - **NCM**
   - **Unidade de Medida**

Este comportamento é idêntico em **pedidos de venda**, **faturas de venda**, **pedidos de compra** e **faturas de compra**.

> **Nota**: Se a conta contábil não possuir item fiscal configurado, o comportamento permanece inalterado — o campo deve ser preenchido manualmente na linha, se necessário.

---

## 4. Resumo dos Campos Novos

### Tabela Conf. Fiscais (`FiscalSetupFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Mostrar Nº Item Fiscal na Conta Contábil** (`ShowFiscalItemNoOnGLAccount`) | Boolean | Controla a visibilidade do campo Nº Item Fiscal no cartão da Conta Contábil |

### Tabela Conta Contábil (`G/L Account` — extensão `GLAccountLocFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Nº Item Fiscal** (`FiscalItemNoFNX`) | Code[20] | Item vinculado à conta contábil para fins fiscais. Relacionado à tabela Item. |
