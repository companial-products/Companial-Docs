<!-- PR: #323 -->
# NFS-e: Mensagens da Prefeitura na Consulta do Lote

**Data**: Novembro/2026

---

## Sumário

Ao consultar um lote de NFS-e, o Business Central mostra na tela, no fim da consulta, o que a prefeitura respondeu: o aviso de lote ainda não processado, as mensagens de rejeição e as mensagens que acompanham uma autorização. A mensagem diz a que lote cada resposta se refere e como o lote ficou. Os status dos lotes e dos documentos continuam os mesmos, e as mensagens continuam gravadas em **Lista Avisos** e **Lista Erros**. A rotina automática de NFS-e segue sem abrir mensagem.

---

## 1. Contexto

Depois de transmitir um lote de NFS-e, o faturista consulta o lote para saber se a prefeitura autorizou as notas. Em muitas prefeituras a autorização não sai na hora, e a consulta pode voltar dizendo que o lote ainda não foi recebido ou processado. É o caso, por exemplo, da GINFES de São Bernardo do Campo, do Emissor Nacional, da WebISS, da Sigcorp e de Barueri.

Até então, a consulta não mostrava nada na tela. O lote ia para o status certo, mas a resposta da prefeitura ficava só nas listas do lote:

- o aviso de lote ainda não processado, em **Lista Avisos**;
- as mensagens de rejeição, em **Lista Erros**.

Quem usava **Consultar** ficava sem saber se a consulta tinha sido feita e o que a prefeitura respondeu.

Agora a resposta aparece na tela no fim da consulta. Isso vale para todas as cidades com integração por web service.

### O que não muda

- A **transmissão** do lote: a recusa no envio já aparecia na tela e continua aparecendo como antes.
- As filiais que trabalham por **arquivo texto**, sem web service: a consulta continua importando o arquivo de retorno, sem mensagem.
- O texto de cada mensagem: ele é o que a prefeitura devolve, como antes.

---

## 2. Como Acessar

A mensagem aparece nas três consultas feitas por um usuário:

| Página | Ação | O que é consultado |
| --- | --- | --- |
| **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`) | **Consultar** | O lote selecionado |
| **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`) | **Consultar** | O lote de cada documento selecionado, um ou vários |
| **Documento Fiscal de Venda** (página `SalesFiscalDocumentFNX`) | **Consultar Status** | O lote do documento, quando o **Status NFS-e** é **Enviado** ou **Erro** |

Em todas elas, a mensagem aparece **depois** que a consulta terminou e o resultado já foi gravado.

---

## 3. A Mensagem na Tela

A mensagem tem **um bloco por lote consultado**, separados por uma linha em branco. Quando a consulta abrange vários lotes, todos aparecem numa única mensagem.

### Primeira linha do bloco

A primeira linha identifica o lote, com o número do lote e, quando há, a filial, e diz como o lote ficou depois da consulta:

| Status do lote depois da consulta | Primeira linha do bloco |
| --- | --- |
| **Erro** | "Lote &lt;nº&gt; (filial &lt;filial&gt;): rejeitado pela prefeitura. O lote está com status Erro." |
| **Enviado** | "Lote &lt;nº&gt; (filial &lt;filial&gt;): continua Enviado, aguardando o processamento da prefeitura." |
| **Sucesso**, com NFS-e recebida | "Lote &lt;nº&gt; (filial &lt;filial&gt;): autorizado." |
| **Sucesso**, sem nenhuma NFS-e recebida | "Lote &lt;nº&gt; (filial &lt;filial&gt;): a prefeitura respondeu sucesso, mas não devolveu nenhuma NFS-e. O lote ficou com status Sucesso e os documentos estão sem número de NFS-e." |

Sem filial no lote, a primeira linha começa só por "Lote &lt;nº&gt;:".

### Linhas seguintes

Abaixo da primeira linha vêm:

1. **As mensagens da prefeitura**, cada uma numa linha, na ordem em que a prefeitura as enviou. O texto aparece **completo**, mesmo quando passa de 250 caracteres, e sem alteração, inclusive com símbolos como `%` e `\`.
2. **Os documentos com erro**, quando a prefeitura rejeita só parte dos RPS do lote e os documentos rejeitados são movidos para um lote novo de erro:
   - a linha "Documentos com erro, movidos para o lote &lt;nº do lote de erro&gt;:";
   - uma linha por documento, no formato `<Nº do documento fiscal>: <erros devolvidos pela prefeitura>`.
3. **Quando a prefeitura não envia texto nenhum**, uma mensagem padrão:
   - lote **Enviado**: "O serviço da prefeitura respondeu, mas não forneceu detalhes. Consulte o lote novamente mais tarde.";
   - lote **Erro** ou **Sucesso** sem NFS-e: "O serviço da prefeitura respondeu, mas não forneceu detalhes.".

### Quando não aparece mensagem

Um lote autorizado, que recebeu NFS-e e não trouxe nenhuma mensagem da prefeitura, **não gera bloco**: a consulta segue como antes, sem mensagem na tela. Se nenhum lote consultado gerar bloco, nenhuma mensagem aparece.

### Exemplo prático

O faturista seleciona, na página **NFS-e - Envio de Lote**, três documentos: dois do lote `261103100512345` e um do lote `261103112030001`, ambos da filial `MATRIZ`. A prefeitura ainda não processou o primeiro lote e rejeitou o segundo. A mensagem tem um bloco para cada lote, e o lote com dois documentos aparece uma só vez:

```text
Lote 261103100512345 (filial MATRIZ): continua Enviado, aguardando o processamento da prefeitura.
Lote ainda não processado pela prefeitura (situação 2 - Não Processado). Consulte novamente em alguns minutos.

Lote 261103112030001 (filial MATRIZ): rejeitado pela prefeitura. O lote está com status Erro.
<mensagem de rejeição da prefeitura>
<outra mensagem de rejeição da prefeitura>
```

As mensagens de rejeição dependem da prefeitura e do motivo da recusa. O texto da segunda linha do primeiro bloco é o aviso devolvido pela GINFES de São Bernardo do Campo.

---

## 4. Comportamento

### Status e dados gravados

Mostrar a mensagem não altera o resultado da consulta. Os status continuam os de antes:

| Resposta da prefeitura | Lote | **Status NFS-e** dos documentos | Onde as mensagens ficam gravadas |
| --- | --- | --- | --- |
| Sem erro e sem nota autorizada (lote ainda não processado) | **Enviado** | **Enviado** | **Lista Avisos** |
| Rejeição | **Erro** | **Erro**, liberados para correção e nova transmissão | **Lista Erros** |
| Autorização | **Sucesso** | **Sucesso**, com o **Nº NFS** e os demais dados da nota | Não são gravadas: aparecem só na tela |

Em todos os casos ficam gravados a **Data/Hora Consulta**, o **Nº de Consultas** do lote e os contadores de avisos e de erros. A cada nova consulta, as mensagens anteriores do lote são substituídas pelas da resposta nova. Assim, quando a prefeitura autoriza um lote que antes estava pendente, o aviso anterior sai de **Lista Avisos**.

### Texto completo na tela, 250 caracteres nas listas

Em **Lista Avisos** e **Lista Erros**, cada mensagem continua gravada com até 250 caracteres. A tela mostra o texto inteiro. Para guardar uma mensagem longa, copie-a da tela ou use o [Log de Comunicação](Localization-for-Brazil_User-Manual_NFSe-Log-Comunicacao-Prefeitura.md), que guarda o retorno completo da prefeitura.

### Vários documentos em NFS-e - Envio de Lote

Na página **NFS-e - Envio de Lote**, **Consultar** consulta **cada lote uma única vez**, por mais documentos dele que estejam selecionados. Antes, o lote era consultado uma vez para cada documento selecionado. Com isso:

- o **Nº de Consultas** do lote sobe 1 por clique em **Consultar**, e não 1 por documento;
- o mesmo lote não se repete na mensagem;
- a prefeitura recebe uma só consulta por lote.

Os documentos com **Status NFS-e** **Sucesso** continuam fora da consulta, como antes.

### Rotina automática

A consulta feita pela rotina automática de NFS-e (**Movimentos Bot NFS-e**, executada pela fila de tarefas) **não abre mensagem**. Ela não para por causa das mensagens da prefeitura, e as mensagens ficam gravadas no lote, como antes. Isso vale também quando a tarefa é executada pela ação **Executar uma vez (primeiro plano)**, na sessão do usuário.

---

## 5. Pontos de Atenção

- **Falha de comunicação durante a consulta**: um erro de conexão, de certificado ou do serviço de integração interrompe a consulta com a mensagem de erro, como antes, e não aparece o resumo. Numa consulta de vários lotes em **NFS-e - Envio de Lote**, consulte os lotes de novo depois de resolver o problema.
- **Lote Sucesso sem NFS-e**: algumas integrações respondem sucesso sem devolver a nota. O lote fica **Sucesso** e os documentos continuam sem **Nº NFS**, como antes, mas agora a mensagem avisa. Confira o lote no portal da prefeitura e, se for o caso, consulte de novo ou fale com o suporte.
- **Mensagens de uma autorização**: as mensagens que a prefeitura envia junto com a autorização aparecem só na tela e não ficam em **Lista Avisos**. Anote-as antes de fechar a mensagem, se precisar delas.
- **Lote de erro criado pela consulta**: quando a prefeitura rejeita só parte dos RPS, os documentos rejeitados passam para um lote novo com **Status** **Erro**. As mensagens desses documentos ficam em **Lista Erros** do lote novo, cujo número aparece na mensagem.

Os resultados da consulta em cada cidade estão nos manuais de emissão por município (veja [NFS-e: Manuais de Emissão por Município e Plataforma](Localization-for-Brazil_User-Manual_NFSe-Manuais-por-Municipio.md)).
