<!-- Commits: b72646a -->
# NF-e: Prorrogação da Suspensão do ICMS na Remessa para Industrialização

**Data**: Março/2026

---

## Sumário

Esta atualização implementa os eventos de **Prorrogação e Cancelamento de Prorrogação da Suspensão do ICMS** em operações de remessa para industrialização por encomenda, conforme a Nota Técnica 2015.001. Foram adicionados quatro novos tipos de eventos NF-e (111500 a 111503), com suporte completo à transmissão, cancelamento e processamento automático da resposta do Fisco via MD-e.

## 1. Contexto

### O que é

Nas operações de remessa para industrialização por encomenda, o ICMS é suspenso na saída da mercadoria. O contribuinte possui um prazo para retornar os produtos industrializados. Caso necessite de prazo adicional, deve solicitar à SEFAZ uma **prorrogação da suspensão do ICMS**, utilizando eventos específicos da NF-e.

### Quando se aplica

- Quando o contribuinte precisa estender o prazo de retorno de mercadorias enviadas para industrialização por encomenda
- Quando é necessário cancelar um pedido de prorrogação já solicitado

### Base legal

Nota Técnica 2015.001 da NF-e, que define os seguintes eventos:

| Código | Descrição |
|--------|-----------|
| 111500 | Pedido de Prorrogação da suspensão do ICMS na Remessa (1º Prazo) |
| 111501 | Pedido de Prorrogação da suspensão do ICMS na Remessa (2º Prazo) |
| 111502 | Cancelamento de Pedido de Prorrogação da suspensão do ICMS (1º Prazo) |
| 111503 | Cancelamento de Pedido de Prorrogação da suspensão do ICMS (2º Prazo) |

---

## 2. Tipos de Eventos

Os quatro novos tipos de evento estão disponíveis no campo **Tipo de Evento** (`EventType`) ao criar um novo **Evento NF-e** (`NFeEvent`):

### Eventos de prorrogação

- **111500 — Pedido de Prorrogação (1º Prazo)**: primeira solicitação de extensão do prazo de retorno. Requer a chave de acesso da NF-e de remessa e o protocolo de autorização da NF-e.
- **111501 — Pedido de Prorrogação (2º Prazo)**: segunda solicitação, quando o primeiro prazo também não foi suficiente. Mesmos requisitos do evento 111500.

### Eventos de cancelamento

- **111502 — Cancelamento de Prorrogação (1º Prazo)**: cancela um pedido 111500 previamente autorizado. Requer o protocolo do evento 111500 a ser cancelado.
- **111503 — Cancelamento de Prorrogação (2º Prazo)**: cancela um pedido 111501 previamente autorizado. Requer o protocolo do evento 111501 a ser cancelado.

> **Importante**: esses eventos possuem cancelamento próprio (111502/111503). O cancelamento padrão (110001) **não** pode ser utilizado — o sistema bloqueia essa tentativa e orienta o uso do evento de cancelamento correspondente.

---

## 3. Criando um Pedido de Prorrogação

### Passo a passo

1. Navegue até **Eventos NF-e** (`NFeEvents`) via pesquisa do Business Central
2. Crie um novo evento e selecione o **Tipo de Evento** (`EventType`):
   - `111500` para o 1º prazo
   - `111501` para o 2º prazo
3. No campo **Nº Documento Fiscal** (`FiscalDocumentNo`), informe o documento fiscal da NF-e de remessa. O sistema preencherá automaticamente a **Chave de Acesso** (`AccessKey`) e o **Nº Protocolo NF-e** (`NFeProtocolNo`)
4. Verifique o **Nº Protocolo NF-e** — este é o protocolo de autorização da NF-e original de remessa, obrigatório para a transmissão
5. Na seção **Linhas** (`NFeEventSubform`), utilize a ação **Carregar Itens do Xml** para importar os itens da NF-e de remessa
6. Clique em **Registrar** para transmitir o evento à SEFAZ

### Campos do cabeçalho (eventos 111500/111501)

| Campo | Descrição |
|-------|-----------|
| **Nº Protocolo NF-e** (`NFeProtocolNo`) | Protocolo de autorização da NF-e de remessa. Obrigatório. Preenchido automaticamente ao informar o documento fiscal. |
| **Status do Prazo** (`DeadlineStatus`) | Retorno da SEFAZ indicando se o pedido está dentro ou após o prazo. Somente leitura. |
| **Descrição do Status do Prazo** (`DeadlineStatusDescription`) | Descrição textual do status do prazo. Somente leitura. |

### Campos das linhas (resposta da SEFAZ)

Após o processamento pelo Fisco, os seguintes campos são preenchidos automaticamente em cada linha do evento:

| Campo | Descrição |
|-------|-----------|
| **Status do Pedido** (`RequestStatus`) | Código do status do pedido por item (1 = Deferido, 2 = Indeferido). Somente leitura. |
| **Descrição do Status do Pedido** (`RequestStatusDescription`) | Descrição do status. Somente leitura. |
| **Cód. Justificativa do Status** (`JustificationCode`) | Código da justificativa da SEFAZ. Somente leitura. |
| **Descrição da Justificativa** (`JustificationDescription`) | Descrição da justificativa. Somente leitura. |

---

## 4. Cancelamento de Pedido de Prorrogação

### Passo a passo

1. Navegue até **Eventos NF-e** (`NFeEvents`)
2. Crie um novo evento e selecione o **Tipo de Evento**:
   - `111502` para cancelar um pedido do 1º prazo (111500)
   - `111503` para cancelar um pedido do 2º prazo (111501)
3. No campo **Nº Documento Fiscal** (`FiscalDocumentNo`), informe o mesmo documento fiscal da NF-e de remessa
4. No campo **Nº Protocolo do Evento** (`EventProtocolNo`), selecione o protocolo do evento de prorrogação que deseja cancelar. O sistema apresenta apenas os eventos compatíveis (111500 para cancelamento via 111502, ou 111501 para cancelamento via 111503)
5. Na seção **Linhas**, carregue os itens da NF-e de remessa
6. Clique em **Registrar** para transmitir o cancelamento à SEFAZ

### Campos específicos do cancelamento

| Campo | Descrição |
|-------|-----------|
| **Nº Protocolo do Evento** (`EventProtocolNo`) | Protocolo do evento de prorrogação a ser cancelado. Obrigatório. Filtrado automaticamente pelo tipo de evento e chave de acesso. |
| **Status do Prazo** (`DeadlineStatus`) | Retorno da SEFAZ sobre o cancelamento. Somente leitura. |
| **Descrição do Status do Prazo** (`DeadlineStatusDescription`) | Descrição do status. Somente leitura. |
| **Cód. Justificativa do Prazo** (`DeadlineJustificationCode`) | Código da justificativa do Fisco sobre o cancelamento. Somente leitura. |
| **Descrição da Justificativa do Prazo** (`DeadlineJustifDescription`) | Descrição da justificativa. Somente leitura. |

---

## 5. Resposta Automática do Fisco via MD-e

A resposta do Fisco aos pedidos de prorrogação e cancelamento é recebida automaticamente pelo processo de consulta do **MD-e** (Manifesto do Destinatário Eletrônico).

### Como funciona

1. A SEFAZ processa o pedido de prorrogação e gera um evento de resposta
2. Na consulta MD-e (**Números Sequenciais Únicos (NSU)** — página `NFeMDeNSUsFNX`), o sistema identifica automaticamente as respostas do tipo "Fisco — Prorrogação ICMS remessa para industrialização"
3. O XML da resposta é armazenado no registro NSU para consulta futura
4. O sistema localiza o evento NF-e original correspondente e preenche automaticamente:
   - No **cabeçalho**: **Status do Prazo** e sua descrição
   - Nas **linhas**: **Status do Pedido**, **Descrição do Status do Pedido**, **Cód. Justificativa do Status** e **Descrição da Justificativa** para cada item
5. Para eventos de cancelamento, o sistema também preenche o **Cód. Justificativa do Prazo** e sua descrição

### Valores possíveis da resposta

**Status do Prazo** (`DeadlineStatus`):

| Código | Descrição |
|--------|-----------|
| 0 | Após o prazo |
| 1 | Dentro do prazo |

**Status do Pedido** (`RequestStatus`) — por item:

| Código | Descrição |
|--------|-----------|
| 1 | Deferido |
| 2 | Indeferido |

**Justificativa** (`JustificationCode`):

| Código | Descrição |
|--------|-----------|
| 1 | Autorizado pelo Fisco |
| 2 | Manifestação do Destinatário — desconhece a operação |
| 3 | Manifestação do Destinatário — operação não realizada |
| 4 | O item não consta na NF-e |
| 5 | O item não consta no pedido de prorrogação do 1º prazo |
| 6 | CFOP não autorizado |
| 7 | Quantidade inconsistente com a quantidade do item |
| 8 | Solicitação de pedido fora do prazo |
| 9 | Pedido de prorrogação cancelado pelo contribuinte |
| 10 | Outra |

---

## 6. Download do XML do Evento no MD-e

Uma nova ação foi adicionada à página **Números Sequenciais Únicos (NSU)** (`NFeMDeNSUsFNX`) para permitir o download do XML de eventos recebidos via MD-e.

### Como utilizar

1. Navegue até **Números Sequenciais Únicos (NSU)** (`NFeMDeNSUsFNX`)
2. Selecione um registro do tipo **Evento**
3. Clique na ação **Download XML do Evento** (`DownloadEventXml`)
4. O sistema baixará o arquivo XML com o nome `{ChaveAcesso}-{NSU}-Evento.xml`

> **Nota**: a ação fica habilitada somente para registros do tipo Evento que possuem XML armazenado.

---

## 7. Resumo dos Campos Novos

### Tabela Evento NF-e (`NFeEvent`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Nº Protocolo NF-e** (`NFeProtocolNo`) | Text[50] | Protocolo de autorização da NF-e de remessa. Obrigatório para eventos 111500/111501. |
| **Nº Protocolo do Evento** (`EventProtocolNo`) | Text[50] | Protocolo do evento de prorrogação a cancelar. Obrigatório para eventos 111502/111503. |
| **Status do Prazo** (`DeadlineStatus`) | Integer | Código do status do prazo retornado pela SEFAZ. |
| **Descrição do Status do Prazo** (`DeadlineStatusDescription`) | Text[50] | Descrição do status do prazo. |
| **Cód. Justificativa do Prazo** (`DeadlineJustificationCode`) | Integer | Código da justificativa retornado pela SEFAZ (cancelamento). |
| **Descrição da Justificativa do Prazo** (`DeadlineJustifDescription`) | Text[250] | Descrição da justificativa do prazo. |

### Tabela Linha Evento NF-e (`NFeEventLine`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Status do Pedido** (`RequestStatus`) | Integer | Código do status do pedido por item (1 = Deferido, 2 = Indeferido). |
| **Descrição do Status do Pedido** (`RequestStatusDescription`) | Text[50] | Descrição do status do pedido. |
| **Cód. Justificativa do Status** (`JustificationCode`) | Integer | Código da justificativa por item. |
| **Descrição da Justificativa** (`JustificationDescription`) | Text[250] | Descrição da justificativa por item. |

### Tabela NSU de Manifestação de Destinatário NF-e (`NFeMDeNSUFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **XML do Evento** (`EventXml`) | Blob | Armazena o XML do evento recebido via MD-e para consulta e download. |
