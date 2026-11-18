<!-- PR: #281 -->
# EFD-Reinf: Envio de Eventos em Lotes e Resultado do Consultar Lote

**Data**: Outubro/2026

---

## Sumário

A ação **Enviar Eventos Pendentes** passa a transmitir todas as linhas incluídas no envio, divididas em lotes de até 50 eventos, que é o limite da Receita Federal por lote. Cada linha fica em **Processando** com o lote em que o seu evento realmente foi, ou continua pendente com o motivo informado na mensagem. A ação **Consultar Lote** sempre termina com uma mensagem e tira de **Processando** as linhas cujo evento não consta no lote consultado, para que possam ser reenviadas. O evento aceito pela Receita leva a linha a **Sucesso** com o recibo e não pode ser reenviado.

---

## 1. Contexto

### O problema

Num envio com muitas linhas, o **Enviar Eventos Pendentes** criava um evento para cada linha, mas o lote transmitido à Receita levava só o primeiro e o último. Mesmo assim, todas as linhas recebiam o número desse lote e ficavam em **Processando**. Os eventos que não foram no lote apareciam em **Eventos Enviados** sem **XML de Envio** e sem **Enviado em Data/Hora**.

A partir daí a linha não tinha saída:

- **Consultar Lote** terminava sem mensagem e sem mudar nada, porque a Receita só respondia sobre os eventos que estavam no lote;
- **Enviar Evento** era recusado, porque a linha estava em **Processando**;
- as notas dessas linhas nunca chegavam à Receita.

Quando o lote inteiro não era recebido, todas as linhas iam para **Erro**, mas a coluna **Últimas Mensagens de Erro** ficava vazia, e o histórico da linha acumulava eventos sem status, sem data de envio e sem lote.

### O que mudou

- O envio monta o XML de **todos** os eventos incluídos e divide o envio em lotes de até 50 eventos.
- O número de um lote só é gravado nas linhas cujo evento foi transmitido nele.
- Um lote que não passa interrompe o envio com uma mensagem que explica o motivo, e as linhas desse lote não mudam.
- **Consultar Lote** sempre informa o resultado e resolve as linhas cujo evento não consta no lote.

### Eventos cobertos

O comportamento vale para todas as listas de eventos periódicos da EFD-Reinf:

| Evento | Página |
|--------|--------|
| **R-2010** | **Reinf - Retenção Contribuição Previdenciária - Compras** (`ReinfRetainedINSSPurchasesFNX`) |
| **R-2020** | **Reinf - Retenção Contribuição Previdenciária - Vendas** (`ReinfRetainedINSSSalesFNX`) |
| **R-2050** | **Reinf - Vendas de Produção Rural** (`ReinfRuralProductionSalesFNX`) |
| **R-2055** | **Reinf - Compras de Produção Rural** (`ReinfRuralProductionPurchFNX`) |
| **R-2060** | **Reinf - Retenção Contribuição Previdenciária sobre Receita Bruta** (`ReinfCPRBFNX`) |
| **R-4010** | **Reinf - Pagamentos a Pessoas Físicas** (`ReinfPersonPaymentsFNX`) |
| **R-4020** | **Reinf - Pagamentos a Pessoas Jurídicas** (`ReinfCompanyPaymentsFNX`) |

Os eventos R-1000, R-1070, R-2098, R-2099, R-4099 e R-9000 continuam sendo enviados um a um, como antes.

---

## 2. Enviar Eventos Pendentes

### Como enviar

1. Abra a página do evento (tabela da seção anterior) e filtre o período.
2. Clique na ação **Enviar Eventos Pendentes**.
3. O sistema envia as linhas ainda não transmitidas e as linhas em **Erro**, em lotes de até 50 eventos. Um envio com 120 linhas, por exemplo, vai em três lotes: 50, 50 e 20.

As ações **Enviar Evento** e **Enviar Evento de Retificação** seguem a mesma regra, com um lote de uma linha só.

### Resultado do envio

Ao fim do envio aparece a mensagem:

> Foram enviados 120 evento(s) em 3 lote(s). As linhas ficam em Processando até a consulta do lote com Consultar Lote.

Cada linha enviada fica com **Status** **Processando** (ou **Status Retificação** **Processando**, no envio de retificação). Na ação **Eventos Enviados**, o evento da linha mostra:

| Campo | Conteúdo |
|-------|----------|
| **Status** | **Processando** |
| **Enviado em Data/Hora** | Data e hora em que o lote foi transmitido |
| **Nº Recibo Lote** | Número do lote em que o evento foi, devolvido pela Receita |
| **XML de Envio** | XML do evento, visível pela ação **Visualizar XML de Envio** |

### Quando um lote não passa

O envio para no primeiro lote que não passar. Isso acontece quando a Receita recusa o lote, quando o serviço da EFD-Reinf não responde ou quando o XML de uma das linhas não pode ser montado (por exemplo, um cadastro incompleto). Nesse caso:

- os lotes já recebidos pela Receita ficam como estão, com as linhas em **Processando**;
- as linhas do lote que não passou e as dos lotes seguintes **não mudam** e continuam pendentes, sem evento novo no histórico;
- a mensagem diz em que lote o envio parou, o motivo, quantos eventos já foram e quantas linhas ficaram pendentes.

Se o **primeiro** lote não passar, nada é enviado:

> Nenhum evento foi enviado: *motivo*
> Nenhuma linha foi alterada.

Se um lote seguinte não passar:

> O envio parou no lote 3: *motivo*
> 100 evento(s) foram enviados em 2 lote(s) e estão em Processando. 20 linha(s) não foram enviadas e continuam pendentes; corrija o motivo e use Enviar Eventos Pendentes novamente.

Quando o motivo é uma linha cujo XML não pode ser montado, a mensagem traz "Não foi possível montar o XML do evento de" seguido da identificação da linha. Corrija o cadastro da linha e use **Enviar Eventos Pendentes** de novo: o envio retoma das linhas que ficaram pendentes.

---

## 3. Consultar Lote

### Como consultar

1. Selecione uma linha com **Status** (ou **Status Retificação**) **Processando**.
2. Clique na ação **Consultar Lote**.

A consulta é feita pelo lote do evento da linha, e o resultado é aplicado a **todas** as linhas daquele lote. Não é preciso consultar linha a linha: uma consulta por lote resolve todas as linhas que foram nele.

### Resultados possíveis

A consulta sempre termina com uma mensagem sobre a linha selecionada:

| Situação | O que acontece com a linha | Mensagem |
|----------|----------------------------|----------|
| A Receita aceitou o evento | Passa a **Sucesso** com o **Nº Recibo** do evento e não pode ser reenviada | "A EFD-Reinf aceitou o evento desta linha com o recibo *nº do recibo*. A linha passou a Sucesso e não pode ser reenviada." |
| A Receita rejeitou o evento | Passa a **Erro**, com o motivo em **Últimas Mensagens de Erro** | "A EFD-Reinf rejeitou o evento desta linha: *motivo*" |
| O evento não consta no lote processado | Passa a **Erro** e pode ser reenviada (veja a [seção 4](#4-linhas-cujo-evento-não-consta-no-lote)) | "O evento desta linha não consta no lote *nº do lote* processado pela EFD-Reinf, portanto nunca foi transmitido. A linha passou a Erro e pode ser reenviada com Enviar Evento ou Enviar Eventos Pendentes." |
| O lote ainda está em processamento na Receita | Continua em **Processando** | "O lote *nº do lote* ainda está em processamento na EFD-Reinf: *descrição*. A linha continua em Processando. Consulte o lote novamente mais tarde." |
| O lote não foi encontrado na Receita | Não muda | "Lote *nº do lote* não encontrado na EFD-Reinf." |
| A consulta falhou no serviço da EFD-Reinf | Não muda | "Não foi possível consultar o lote *nº do lote*: *motivo*" |
| A Receita devolveu outro status para o lote | Não muda | "A EFD-Reinf retornou o status *código* para o lote *nº do lote*: *descrição*. A linha não foi alterada." |
| A linha não tem lote em processamento | Não muda | "Esta linha não tem lote em processamento." |

Quando o lote tem outros eventos além do da linha selecionada, a mensagem traz também o resumo do lote:

> Lote 1.202610.1088629926: 2 evento(s) aceito(s), 0 rejeitado(s) e 48 que não constava(m) no lote.

### Lote antigo não desfaz o reenvio

O resultado de um lote só é aplicado à linha se o evento consultado for o **mais recente** da linha. Uma linha que já foi reenviada, ou que tem uma retificação em andamento, não volta para **Erro** nem perde o **Status Retificação** se alguém consultar de novo um lote antigo em que ela constava.

---

## 4. Linhas cujo evento não consta no lote

### Como reconhecer

São linhas presas em **Processando** por envios feitos antes desta correção. Na ação **Eventos Enviados** da linha, o evento mais recente aparece com:

- **Status** **Processando**;
- **Nº Recibo Lote** preenchido;
- **Enviado em Data/Hora** vazio e sem **XML de Envio**.

Esse evento nunca chegou à Receita: as notas da linha não foram transmitidas. Resolva essas linhas antes do fechamento do período com o R-2099 ou o R-4099.

### Como destravar e reenviar

1. Selecione qualquer linha do lote e clique em **Consultar Lote**.
2. As linhas do lote cujo evento a Receita aceitou passam a **Sucesso** com o recibo.
3. As linhas cujo evento não consta no lote passam a **Erro**. A coluna **Últimas Mensagens de Erro** mostra:

   > O evento não consta no lote *nº do lote* processado pela EFD-Reinf e não foi transmitido. Reenvie a linha.

4. Reenvie essas linhas com **Enviar Evento** (uma linha) ou **Enviar Eventos Pendentes** (todas as linhas em **Erro** do período).

### Proteção contra duplicidade

O destravamento só acontece quando a Receita confirma que o evento **não** está no lote. Se o evento da linha foi aceito, a linha vai para **Sucesso** com o recibo e não pode ser reenviada, para não gerar um evento em duplicidade na Receita.

### O histórico é mantido

O reenvio não apaga nada. O evento antigo continua em **Eventos Enviados**, com o status **Erro** e a mensagem de que não constava no lote. A retransmissão aparece como um evento novo, com seu próprio **XML de Envio**, **Enviado em Data/Hora** e **Nº Recibo Lote**.

Os eventos vazios que ficaram no histórico de envios anteriores (sem status, sem data de envio e sem lote) também continuam lá. Eles não impedem o reenvio da linha.

---

## 5. Resumo das Mudanças por Ação

| Ação | Antes | Agora |
|------|-------|-------|
| **Enviar Eventos Pendentes** | Num envio grande, só o primeiro e o último evento iam ao lote, mas todas as linhas recebiam o número do lote | Todas as linhas são transmitidas, em lotes de até 50 eventos, cada uma com o número do seu lote |
| **Enviar Eventos Pendentes** (lote recusado) | Todas as linhas iam para **Erro** sem mensagem | O envio para com uma mensagem que explica o motivo; as linhas do lote recusado não mudam |
| **Consultar Lote** | Podia terminar sem mensagem; eventos fora do lote ficavam em **Processando** para sempre | Sempre termina com mensagem; eventos fora do lote vão para **Erro** e podem ser reenviados |
| **Consultar Lote** (lote antigo) | Podia sobrescrever o status de uma linha já reenviada | Só aplica o resultado se o evento for o mais recente da linha |
