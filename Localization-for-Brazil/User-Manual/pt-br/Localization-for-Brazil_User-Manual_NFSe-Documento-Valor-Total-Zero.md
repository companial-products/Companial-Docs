<!-- Commits: 0927ae38 -->
# NFS-e: Documento Fiscal com Valor Total Zero

**Data**: Junho/2026

---

## Sumário

Esta atualização permite a emissão de documentos fiscais legítimos cujo valor total é **R$ 0,00**. O cenário típico é a NFS-e de repasse de benefícios: administradoras de vale-refeição e cartões precisam emitir nota de serviço sem valor financeiro. Um novo campo, **Zerar Valor Fiscal**, no **Tipo de Operação** zera automaticamente os valores da linha do documento fiscal e libera o cálculo de impostos sobre base zero, sem exigir o preenchimento do **Valor Unitário**.

---

## 1. Contexto

### O que é

Em algumas operações, a legislação exige a emissão de um documento fiscal de serviço (NFS-e) ainda que o **Valor Total do Serviço** seja igual a **R$ 0,00**. É o caso das administradoras de benefícios (vale-refeição, vale-alimentação, cartões), nas quais o repasse do benefício é registrado por meio de NFS-e sem valor financeiro associado ao serviço.

Até então, o sistema exigia que toda linha do documento fiscal tivesse um **Valor Unitário** preenchido, o que impedia o registro dessas notas. Com esta atualização, é possível marcar a operação como de valor fiscal zerado e registrar o documento fiscal com os valores da linha zerados.

### Base legal

- **Lei Municipal de São Paulo nº 14.097/2005**, que disciplina a emissão da NFS-e no município de São Paulo.
- **Solução de Consulta SC/DEJUG nº 11/2025**, que orienta o tratamento das notas de repasse de benefícios.

### Quando se aplica

- NFS-e de repasse de benefícios emitidas por administradoras de vale-refeição, vale-alimentação e cartões.
- Demais operações de serviço legítimas em que o **Valor Total do Serviço** é R$ 0,00, mas a nota precisa ser emitida e os tributos calculados e registrados.

---

## 2. Configuração no Tipo de Operação

O comportamento é controlado pelo **Tipo de Operação** utilizado na venda ou na compra.

1. Navegue até **Tipos de Operação** (página `OperationTypesFNX`) pela busca do Business Central (**Alt+Q**).
2. Abra (ou crie) o tipo de operação que será usado nas notas de valor zero.
3. Marque o campo **Zerar Valor Fiscal** (`ZeroFiscalAmount`).

Ao marcar **Zerar Valor Fiscal**, o sistema define automaticamente o campo **Mov. Cliente/Fornecedor** (`CustomerVendorEntry`) como vazio e o torna não editável enquanto a marca estiver ativa. Uma operação de valor fiscal zerado não gera movimento financeiro de cliente ou fornecedor.

> **Nota**: Se você tentar alterar o **Mov. Cliente/Fornecedor** com o **Zerar Valor Fiscal** ainda habilitado, o sistema exibe a mensagem: *"O Movimento de Cliente/Fornecedor não pode ser alterado enquanto Zerar Valor Fiscal estiver habilitado."* Para alterá-lo, desmarque primeiro o **Zerar Valor Fiscal**.

---

## 3. Comportamento em Vendas e Compras

Ao registrar um documento fiscal a partir de um documento de venda ou de compra cujo **Tipo de Operação** tenha o **Zerar Valor Fiscal** marcado, o sistema:

1. Propaga a marca para a linha do documento fiscal (campo **Zerar Valor Fiscal** da **Linha de Documento Fiscal**).
2. Zera os valores da linha antes da gravação:
   - **Quantidade** (`Quantity`)
   - **Valor Unitário** (`UnitAmount`)
   - **Valor** (`Amount`)
   - **% Desconto** (`DiscountPercent`)
   - **Valor Desconto** (`DiscountAmount`)

Esse comportamento vale tanto para o fluxo de **Vendas** quanto para o de **Compras**.

---

## 4. Comportamento na Linha do Documento Fiscal

Quando uma linha está marcada com **Zerar Valor Fiscal**:

- A **Quantidade**, o **Valor Unitário**, o **Valor**, o **% Desconto** e o **Valor Desconto** são zerados.
- A **Quantidade** e os campos de valor (**Valor Unitário**, **% Desconto** e **Valor Desconto**) ficam bloqueados para edição nos documentos de venda e de compra.
- O recálculo automático da linha respeita a marca: enquanto ela estiver ativa, esses campos permanecem em zero.

O campo **Zerar Valor Fiscal** da **Linha de Documento Fiscal** vem oculto por padrão. No fluxo de vendas e compras ele é preenchido automaticamente pela configuração do **Tipo de Operação** (ver seção anterior). Para a digitação direta de um documento fiscal, o campo pode ser exibido pela personalização da página de linhas e marcado manualmente quando necessário.

---

## 5. Cálculo de Impostos com Base Zero

Para linhas marcadas com **Zerar Valor Fiscal**, o cálculo de impostos foi adaptado:

- O sistema não ignora a linha durante o cálculo, mesmo que o **Valor Unitário** seja zero.
- Em vez de exigir **Quantidade** e **Valor Unitário** preenchidos, o sistema valida que **Quantidade**, **Valor Unitário**, **Valor**, **% Desconto** e **Valor Desconto** estejam efetivamente zerados.
- Com isso, os impostos são calculados sobre base zero. A NFS-e pode ser emitida e os tributos são registrados e destacados conforme exigido, ainda que sem valor financeiro.

---

## 6. Resumo dos Campos Novos

### Tabela Tipo de Operação (`OperationTypeFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Zerar Valor Fiscal** (`ZeroFiscalAmount`) | Boolean | Quando marcado, os documentos que usam este tipo de operação têm a Quantidade e o valor fiscal (Valor Unitário, Valor, Descontos) zerados na linha do documento fiscal ao registrar a venda ou a compra. Ao marcar, o **Mov. Cliente/Fornecedor** é definido como vazio e fica não editável. |

### Tabela Linha de Documento Fiscal (`FiscalDocumentLineFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Zerar Valor Fiscal** (`ZeroFiscalAmount`) | Boolean | Indica que a linha tem valor fiscal zero. Quando habilitado, os campos Quantidade, Valor Unitário, Valor e Desconto são zerados e bloqueados, e não é obrigatório que tenham valor durante o cálculo de imposto. Oculto por padrão; preenchido automaticamente a partir do Tipo de Operação. |
