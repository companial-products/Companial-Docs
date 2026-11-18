<!-- Commits: a4f74ef8 -->
<!-- PR: #330 -->
# Reforma Tributária: Classificação Tributária IBS/CBS na Exceção de Imposto

**Data**: Outubro/2025

---

## Sumário

Esta atualização adiciona os campos de **Classificação Tributária IBS/CBS** à tabela de **Exceções de Impostos**, permitindo sobrescrever a classificação tributária definida na matriz de impostos para cenários específicos. Quando uma exceção de imposto possui classificação tributária preenchida, ela tem **prioridade sobre a matriz de impostos**, aplicando-se automaticamente em linhas de vendas, compras e documentos fiscais.

## 1. Contexto

### O que é

A Reforma Tributária brasileira introduziu o conceito de **Classificação Tributária** (`cClassTrib`) para os impostos IBS e CBS. Cada operação fiscal recebe um código de classificação que determina o tratamento tributário aplicável (alíquotas, reduções, isenções, etc.).

Normalmente, a classificação tributária é definida na **Matriz de Impostos** e aplicada automaticamente às linhas dos documentos. Porém, existem cenários em que uma operação específica requer uma classificação diferente da configurada na matriz — por exemplo, um produto que em determinada jurisdição ou operação recebe tratamento tributário especial.

Para atender esses casos, a classificação tributária foi incluída na tabela de **Exceções de Impostos** (`TaxExceptionFNX`), cujas linhas de alíquotas já definiam tratamentos especiais (alíquota, redução, CST, texto legal).

### Quando se aplica

Sempre que uma linha de documento (venda, compra ou documento fiscal) possuir um **Cód. Exceção Imposto** (`TaxExceptionCode`) que contenha classificação tributária IBS/CBS preenchida.

---

## 2. Configuração

1. Navegue até **Exceções de Impostos** (página `TaxExceptionsFNX`) via pesquisa do Business Central
2. Abra ou crie um código de exceção
3. Preencha o campo **Cód. Class. Trib. IBS/CBS** (`IBSCBSTaxClassCode`) com a classificação tributária desejada
4. Preencha o campo  **Cód. Class. Trib. Regular IBS/CBS** (`IBSCBSRegularTaxClassCode`) se necessário

> **Nota**: Se o campo **Cód. Class. Trib. IBS/CBS** estiver vazio na exceção, a classificação da matriz de impostos será mantida normalmente.

---

## 3. Comportamento na Aplicação

### Prioridade

A classificação tributária segue a seguinte ordem de prioridade:

1. **Exceção de Imposto** (maior prioridade) — se a exceção possuir classificação tributária preenchida
2. **Matriz de Impostos** (prioridade padrão) — usada quando a exceção não possui classificação ou não há exceção

### Fluxos suportados

A substituição da classificação tributária ocorre automaticamente nos seguintes cenários:

- **Vendas**: ao aplicar a matriz de impostos na linha de venda, ou ao alterar manualmente o **Cód. Exceção Imposto** na linha
- **Compras**: ao aplicar a matriz de impostos na linha de compra, ou ao alterar manualmente o **Cód. Exceção Imposto** na linha
- **Documentos fiscais**: ao aplicar a matriz de impostos na linha do documento fiscal, ou ao alterar manualmente o **Cód. Exceção Imposto** na linha

> **Nota**: se o **Cód. Class. Trib. Regular IBS/CBS** da exceção for igual ao **Cód. Class. Trib. IBS/CBS**, ou estiver preenchido para uma classificação que não exige tributação regular, a substituição não acontece e a linha é bloqueada. O erro cita a exceção e traz a ação **Abrir Exceção Imposto**. Veja como corrigir em [Reforma Tributária — Configurações](Localization-for-Brazil_User-Manual_Reforma-Tributaria-Configuracoes.md#linha-bloqueada-pelo-cód-class-trib-regular-ibscbs).

---

## 4. Resumo dos Campos Novos

### Tabela Exceção de Imposto (`TaxExceptionFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Cód. Class. Trib. IBS/CBS** (`IBSCBSTaxClassCode`) | Code[10] | Código de classificação tributária IBS/CBS. Quando preenchido, sobrescreve a classificação da matriz de impostos. |
| **Cód. Class. Trib. Regular IBS/CBS** (`IBSCBSRegularTaxClassCode`) | Code[10] | Código de classificação tributária regular, preenchido automaticamente a partir da classificação principal. |
