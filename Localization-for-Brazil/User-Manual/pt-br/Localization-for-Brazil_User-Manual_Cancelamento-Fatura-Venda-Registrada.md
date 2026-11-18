# Vendas: Cancelamento de Fatura de Venda Registrada

**Data**: Maio/2022

---

## Sumário

Uma fatura de venda registrada é cancelada no sistema pelas ações do grupo **Corrigir**: **Corrigir**, **Cancelar** e **Criar Nota de Crédito Corretiva**. As três registram uma nota de crédito que estorna os movimentos da fatura e mudam o status do documento fiscal da fatura para **Cancelado**. Quando a nota foi transmitida, o cancelamento também precisa chegar à SEFAZ ou à prefeitura, o que pode acontecer na mesma ação ou depois, pelo documento fiscal.

## 1. Contexto

### Quando cancelar

O cancelamento é usado quando um documento registrado precisa ser desfeito no sistema, por exemplo por:

- erro de contabilização, de impostos ou de dados preenchidos;
- rejeição na SEFAZ;
- erro de envio para a prefeitura.

### As três formas de cancelar

| Ação | O que faz |
|------|-----------|
| **Corrigir** | Cancela a fatura registrada, registrando uma nota de crédito que a estorna, e cria uma nova fatura com os mesmos dados para ser corrigida e registrada de novo. Quando a fatura veio de uma ordem de venda que ainda existe, a ordem é aberta para a correção em vez de uma nova fatura. Quando veio de várias ordens, ou a ordem já foi excluída, a fatura é apenas cancelada. |
| **Cancelar** | Cancela a fatura registrada, criando e registrando automaticamente uma nota de crédito que estorna os lançamentos. |
| **Criar Nota de Crédito Corretiva** | Cria uma nota de crédito em aberto, copiada da fatura, para ser conferida e registrada manualmente. O cancelamento acontece quando essa nota de crédito é registrada. |

O cancelamento acontece em três etapas:

1. Cancelar a fatura de venda registrada por uma das ações acima.
2. Conferir, no **Documento Fiscal Registrado** da fatura, que o **Status** passou a **Cancelado**.
3. Cancelar a NF-e na SEFAZ ou a NFS-e na prefeitura, quando a nota foi transmitida.

> **Nota**: **Corrigir** e **Cancelar** só aparecem em faturas que ainda não estão canceladas e são recusadas quando a fatura está total ou parcialmente paga, quando algum item da fatura já foi devolvido ou quando a própria fatura é o cancelamento de uma nota de crédito. Para estornar uma fatura paga, crie manualmente uma nota de crédito.

---

## 2. Configuração

As três formas de cancelamento dependem de um tipo de documento fiscal de cancelamento, que é atribuído à nota de crédito de estorno.

1. Pesquise por **Tipos Documento Fiscal** ou, no Centro de Funções, abra **Configurações** > **Fiscal** > **Documentos** > **Tipos Documento Fiscal**.
2. Marque o campo **Cancelamento** no tipo de documento fiscal usado para os cancelamentos.

![Lista Tipos Documento Fiscal com o tipo NC marcado como Cancelamento](images/Cancelamento-Fatura-Venda-Registrada/fiscal-doc-types.png)

> **Nota**: apenas um tipo de documento fiscal pode ter o campo **Cancelamento** marcado. Ao marcá-lo, o tipo passa a ter **Emitente Documento** = **Sem Valor Fiscal**, e os campos que não se aplicam a um documento de cancelamento, como **Modelo Documento**, série e as obrigatoriedades de CFOP, NCM e CST, são limpos.

---

## 3. Cancelar a fatura de venda registrada

Pesquise por **Faturas Venda Registradas** ou, no Centro de Funções, abra **Vendas** > **Faturas Venda Registradas**. Selecione a fatura na lista, ou abra a fatura, e clique em **Corrigir**. O grupo tem as três ações.

![Lista Faturas Venda Registradas com o grupo Corrigir aberto e as ações Corrigir, Cancelar e Criar Nota de Crédito Corretiva](images/Cancelamento-Fatura-Venda-Registrada/correct-menu.png)

### Corrigir

1. Clique em **Corrigir** > **Corrigir**.
2. Confirme a pergunta. O texto varia conforme a origem da fatura: avulsa, de uma ordem ou de várias ordens.

   ![Confirmação A fatura de venda registrada será cancelada e uma nova versão da fatura será criada](images/Cancelamento-Fatura-Venda-Registrada/correct-confirmation.png)

3. O sistema registra a nota de crédito que cancela a fatura e abre a nova fatura de venda, ou a ordem de venda de origem, para a correção. Corrija os dados e registre o documento normalmente.

### Cancelar

1. Clique em **Corrigir** > **Cancelar**.
2. Confirme a pergunta.

   ![Confirmação A fatura de venda registrada será cancelada e uma nota de crédito de venda será criada e registrada](images/Cancelamento-Fatura-Venda-Registrada/cancel-confirmation.png)

3. O sistema registra a nota de crédito e pergunta se você deseja abri-la. Clique em **Sim** para ver a nota de crédito registrada.

   ![Pergunta Uma nota de crédito foi criada com sucesso. Deseja abrir a nota de crédito registrada?](images/Cancelamento-Fatura-Venda-Registrada/open-credit-memo-question.png)

### Criar Nota de Crédito Corretiva

1. Clique em **Corrigir** > **Criar Nota de Crédito Corretiva**.
2. O sistema cria a nota de crédito em aberto, copiada da fatura e já com o tipo de documento fiscal de cancelamento. Uma notificação informa qual documento fiscal será cancelado por ela.
3. Confira os dados e registre a nota de crédito.

![Nota Crédito Venda corretiva com a notificação O documento fiscal será cancelado por essa nota de crédito](images/Cancelamento-Fatura-Venda-Registrada/corrective-credit-memo.png)

### Conferir a fatura cancelada

Na lista **Faturas Venda Registradas**, a fatura cancelada fica com a coluna **Cancelada** = **Sim**. Isso vale para as três formas de cancelamento: na **Criar Nota de Crédito Corretiva**, a marca aparece quando a nota de crédito é registrada.

![Lista Faturas Venda Registradas com a fatura cancelada marcada na coluna Cancelada](images/Cancelamento-Fatura-Venda-Registrada/posted-invoices-canceled.png)

---

## 4. Conferir o status do documento fiscal

Cada fatura registrada está associada a um **Documento Fiscal Registrado**, cujo **Status** fica **Registrado** enquanto a fatura não é cancelada. O cancelamento por uma das ações acima muda esse status para **Cancelado** automaticamente.

1. Na fatura de venda registrada, seção **Fiscal**, clique no **Nº Documento Fiscal** para abrir o documento fiscal.

   ![Fatura Venda Registrada com o campo Nº Documento Fiscal na seção Fiscal](images/Cancelamento-Fatura-Venda-Registrada/invoice-fiscal-doc-no.png)

2. Confira, na seção **Geral**, o **Status** **Cancelado**.

   ![Documento Fiscal Registrado com o Status Cancelado](images/Cancelamento-Fatura-Venda-Registrada/fiscal-doc-canceled.png)

---

## 5. Cancelar na SEFAZ ou na prefeitura

Esta etapa só se aplica quando a nota cancelada foi transmitida.

### NF-e

Quando a filial tem **Transmitir cancelamento automaticamente** ligado na **Config. NF-e**, **Corrigir** e **Cancelar** pedem a justificativa e cancelam a NF-e na SEFAZ na mesma ação, e o registro da nota de crédito corretiva faz o mesmo. Com o controle de prazo ligado, o cancelamento é bloqueado depois do prazo configurado na filial. Os detalhes estão em [NF-e: Cancelamento na SEFAZ no Mesmo Fluxo, Prazo de Cancelamento por Filial e Cancelamentos Pendentes](Localization-for-Brazil_User-Manual_NFe-Cancelamento-SEFAZ-Prazo-Pendencias.md).

Quando a NF-e não foi cancelada junto com a fatura:

1. No documento fiscal cancelado, clique em **NF-e** > **Cancelar NF-e**. A ação só fica habilitada com o **Status** **Cancelado** e a NF-e autorizada.
2. Confirme a pergunta e informe a **Justificativa**, com pelo menos 15 caracteres.

![Documento Fiscal Registrado com a ação NF-e > Cancelar NF-e](images/Cancelamento-Fatura-Venda-Registrada/cancel-nfe-action.png)

### NFS-e

O cancelamento da NFS-e não é transmitido automaticamente. No documento fiscal cancelado, clique em **NFS-e** > **Cancelar NFS-e**. A ação só fica habilitada com o **Status** **Cancelado** e a NFS-e autorizada.

![Documento Fiscal Registrado com a ação NFS-e > Cancelar NFS-e](images/Cancelamento-Fatura-Venda-Registrada/cancel-nfse-action.png)

> **Nota**: algumas prefeituras não disponibilizam o cancelamento pelo web service, e nesse caso a NFS-e não pode ser cancelada pelo Business Central. Confirme antes se a prefeitura da filial aceita o cancelamento.
