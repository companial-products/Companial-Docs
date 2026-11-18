<!-- Source: Sessão Horácio 2026-05-08 -->
# Atualizar Movs. Reg. ao Modificar Doc. Fiscal

**Data**: Maio/2026

---

## Sumário

A flag **Atualizar Movs. Reg. ao Modificar Doc. Fiscal** (`UpdPostEntriesOnFiscDocModify`), na página **Configurações Fiscais** (`FiscalSetupFNX`), controla se alterações em campos do **Documento Fiscal Registrado** são propagadas automaticamente para os movimentos contábeis, de cliente, de fornecedor, de item, de valor, de ativo fixo e de imposto da nota fiscal. Quando ativa, ela elimina a necessidade de ajustar manualmente cada tabela de movimentos após corrigir a **Data Documento** ou o **Nº Nota Fiscal** de um documento já registrado.

---

## 1. Contexto

Depois que um **Documento Fiscal** entra em status **Registrado** (ou **Cancelado**), os movimentos correspondentes já carregam a **Data Documento** e o **Nº Nota Fiscal** originais. São eles: **Mov. Contábil**, **Mov. Cliente**, **Mov. Fornecedor**, **Mov. Item**, **Mov. Valor**, **Mov. AF** e **Mov. Imposto Documento Fiscal**.

Cenários comuns exigem alterar esses dois campos depois do registro:

- O **Nº Nota Fiscal** definitivo só é conhecido depois da autorização da NF-e pela SEFAZ.
- A **Data Documento** precisa ser corrigida para refletir a emissão real ou ajustar a competência fiscal.

Sem essa flag, a correção dos campos no Documento Fiscal não se propaga para os movimentos. O usuário precisaria ajustar manualmente cada tabela de movimentos para manter a consistência entre o documento e a contabilidade. Com a flag ativa, o sistema atualiza todos os movimentos relacionados automaticamente no momento em que o campo é salvo.

> **Observação**: a flag não recalcula valores, não refaz contabilização e não altera lançamentos contábeis em si. Ela apenas sincroniza os campos **Data Documento** e **Nº Nota Fiscal** (e, no caso do número, a descrição dos movimentos) com o novo valor do Documento Fiscal.

---

## 2. Configuração

A flag fica em **Configurações Fiscais** (`FiscalSetupFNX`), na guia geral da página:

1. Abra a página **Configurações Fiscais**.
2. Marque o campo **Atualizar Movs. Reg. ao Modificar Doc. Fiscal** (`UpdPostEntriesOnFiscDocModify`).

A configuração passa a valer imediatamente para qualquer alteração feita em Documentos Fiscais com status **Registrado** ou **Cancelado**.

Se a flag estiver desmarcada, alterações na **Data Documento** ou no **Nº Nota Fiscal** dentro do Documento Fiscal não são repassadas aos movimentos, que permanecem com os valores originais.

---

## 3. Campos que Disparam a Atualização

Com a flag ativa, exatamente **dois campos** do **Documento Fiscal Registrado**, quando alterados, propagam a mudança para os movimentos relacionados:

| Campo | Tipo de propagação |
|-------|--------------------|
| **Data Documento** (`DocumentDate`) | Atualiza o campo **Data Documento** de todos os movimentos vinculados. |
| **Nº Nota Fiscal** (`FiscalNo`) | Atualiza o **Nº Nota Fiscal** e também substitui o número antigo pelo novo dentro do texto da **Descrição** de cada movimento. |

Nenhum outro campo do Documento Fiscal possui propagação automática. Alterações em CFOP, CST, alíquotas, valores de imposto, observações ou demais campos editáveis pelo botão **Editar Campos** afetam somente o Documento Fiscal. Os movimentos contábeis permanecem com os valores registrados originalmente. Para saber quais campos do documento podem ser alterados após o registro, consulte [Editar Campos do Documento Fiscal Registrado](Localization-for-Brazil_User-Manual_Editar-Campos-Documento-Fiscal-Registrado.md).

---

## 4. Movimentos Atualizados

Quando a propagação é disparada, o sistema localiza todos os movimentos vinculados ao Documento Fiscal (pela combinação **Nº Documento** + **Data Registro** + número interno do Documento Fiscal) e atualiza os seguintes:

| Movimento | Tabela AL |
|-----------|-----------|
| **Mov. Contábil** | `G/L Entry` |
| **Mov. Cliente** | `Cust. Ledger Entry` |
| **Mov. Fornecedor** | `Vendor Ledger Entry` |
| **Mov. Item** | `Item Ledger Entry` |
| **Mov. Valor** | `Value Entry` |
| **Mov. AF** | `FA Ledger Entry` |
| **Mov. Imposto Documento Fiscal** | `FiscalDocumentTaxEntryFNX` |

O escopo da atualização varia conforme o tipo do documento:

- Vendas geram **Mov. Cliente**, **Mov. Item** / **Mov. Valor** e **Mov. Contábil**.
- Compras geram **Mov. Fornecedor**, **Mov. Item** / **Mov. Valor** e **Mov. Contábil**.
- Operações com ativo fixo geram **Mov. AF**.
- Em todos os casos, há **Mov. Imposto Documento Fiscal** se a apuração de impostos gerou linhas.

A flag age sobre todos os movimentos encontrados. Não há filtro por tipo de operação ou de imposto.

---

## 5. Comportamento Detalhado

### Alteração da Data Documento

Quando o usuário altera a **Data Documento** no cabeçalho do **Documento Fiscal Registrado** e a flag está ativa:

1. O sistema valida que o documento está em status **Registrado** ou **Cancelado**.
2. Localiza os movimentos relacionados em todas as tabelas listadas na seção anterior.
3. Atualiza o campo **Data Documento** de cada movimento encontrado para o novo valor.
4. Não altera a **Data Registro** dos movimentos, apenas a **Data Documento**.

### Alteração do Nº Nota Fiscal

Quando o usuário altera o **Nº Nota Fiscal** e a flag está ativa:

1. O sistema valida o status do documento (**Registrado** ou **Cancelado**).
2. Localiza os movimentos relacionados.
3. Para cada movimento, executa duas atualizações:
   - O campo **Nº Nota Fiscal** (`FiscalNoFNX`) é atualizado para o novo valor.
   - A **Descrição** do movimento é varrida para substituir a ocorrência do número antigo pelo novo. Por exemplo, uma descrição original `NF 000123 - Cliente Alfa` passa a `NF 000456 - Cliente Alfa` após a alteração de `000123` para `000456`.

### Status que Habilitam a Propagação

A propagação ocorre somente quando o **Status** do Documento Fiscal é **Registrado** ou **Cancelado**. Em outros status (rascunho, aberto), a alteração desses campos nem aciona a lógica de propagação, porque ainda não existem movimentos contábeis registrados.

### Quando a Flag Está Desmarcada

Se a flag **Atualizar Movs. Reg. ao Modificar Doc. Fiscal** está desabilitada nas **Configurações Fiscais**, a edição da **Data Documento** ou do **Nº Nota Fiscal** no Documento Fiscal afeta apenas o registro do documento. Os movimentos permanecem com os valores antigos, e qualquer ajuste tem de ser feito manualmente.

---

## 6. Cenários de Uso Típicos

### Nº definitivo recebido após autorização

Em ambientes em que o **Nº Nota Fiscal** definitivo é definido pela autorização da SEFAZ, e não pela numeração interna do BC, o documento é registrado com um número provisório e atualizado quando a autorização retorna. Com a flag ativa, a atualização do número provisório para o definitivo já reposiciona o **Nº Nota Fiscal** em todos os movimentos da NF-e, sem retrabalho manual.

### Correção de data após registro indevido

Quando um documento é registrado por engano com a **Data Documento** errada (por exemplo, data de emissão divergente da entrada física), corrigir o campo no Documento Fiscal Registrado já reflete a nova data em todos os movimentos contábeis, de cliente ou fornecedor e de estoque.

### Conciliação fiscal versus contábil

Em conciliações entre o livro fiscal e o razão contábil, a flag garante que filtros por **Data Documento** ou por **Nº Nota Fiscal** nos movimentos retornem o mesmo conjunto que o Documento Fiscal Registrado, sem divergências de número ou de data entre as duas visões.

---

## 7. Limitações e Cuidados

- A propagação não recalcula valores, não reabre lançamentos contábeis e não altera **Data Registro** ou **Data Postagem**. Ela apenas sincroniza dois campos específicos.
- Para alteração de outros campos (CFOP, CST, alíquotas, valores de imposto), use a ação **Editar Campos** do Documento Fiscal Registrado. Essas alterações não propagam para movimentos.
- Em ambientes com integração contábil externa (BI, ERP corporativo, consolidações), avalie se o sistema downstream consome a **Data Documento** ou o **Nº Nota Fiscal** dos movimentos. Se sim, a propagação automática pode disparar reprocessamentos.
- O histórico de alterações nos movimentos pode ser consultado pela ação **Log de Alterações** do cabeçalho do Documento Fiscal Registrado, descrita em [Editar Campos do Documento Fiscal Registrado](Localization-for-Brazil_User-Manual_Editar-Campos-Documento-Fiscal-Registrado.md#9-rastreabilidade-e-auditoria).
