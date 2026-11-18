<!-- PR: #176 -->
# Reforma Tributária - Apuração de IBS/CBS conciliada com a Apuração Assistida

**Data**: Novembro/2026

> **Aviso**: esta funcionalidade foi desenvolvida sem que fosse possível testar por completo as APIs da Apuração Assistida do fisco, e enquanto muitos pontos da Reforma Tributária ainda estão indefinidos, como o serviço do IBS, que ainda não foi publicado. O comportamento, as telas e os valores descritos neste manual estão sujeitos a mudanças nas próximas versões, conforme o fisco publicar os serviços e as regras forem regulamentadas.

---

## Sumário

A **Apuração de IBS/CBS** reúne, por competência mensal e para a empresa inteira, duas apurações do mesmo imposto. A primeira é a **apuração própria**, montada a partir dos documentos fiscais registrados no Business Central. A segunda é a **apuração do fisco**, a Apuração Assistida que a Receita Federal disponibiliza para a CBS. O sistema concilia as duas documento a documento, pela chave de acesso, e mostra o que confere, o que existe só de um lado e o que diverge. Também acompanha cada crédito de aquisição até a apropriação e fecha a competência pelo resultado do fisco, gerando uma **Declaração de Imposto** com a apropriação dos créditos, o título a pagar e o encontro de contas.

---

## 1. Contexto

A partir de 2027, a empresa convive com duas versões da apuração mensal de IBS e CBS:

- **A apuração própria**, calculada a partir dos documentos fiscais que a empresa registrou. É o instrumento de controle interno. Em 2026, período de teste sem recolhimento, é ela que documenta os valores apurados, inclusive para quem adota a política de não reconhecer o passivo na contabilidade.
- **A apuração do fisco (Apuração Assistida)**, montada pelo fisco a partir dos documentos eletrônicos que chegaram até ele. É o valor legal: dela saem o imposto a recolher, o encontro de contas e o saldo credor transportado.

A **Apuração de IBS/CBS** coloca os dois lados na mesma tela e os confronta. Em vez de conferir o portal do fisco à mão, o contador vê, para cada documento, se ele **confere**, se existe **somente no Business Central**, **somente no fisco** ou se é **divergente**. O sistema também acompanha cada crédito de aquisição, desde **Potencial** até **Apropriado**, já que o crédito só se confirma quando o fornecedor extingue o débito dele, um fato que apenas o fisco enxerga.

O recurso fica no app **Livros Fiscais** (Tax Books). Ele apenas **lê** os documentos fiscais registrados. O cálculo de IBS/CBS por documento e a contabilização feita no registro do documento continuam como estão.

### Quando se aplica

- Empresas sujeitas ao IBS e à CBS, a partir das competências de 2027.
- Apuração **mensal** e **consolidada por empresa**: os documentos de todas as filiais entram na mesma competência, como o fisco apresenta a apuração no portal. Diferente do ISS e do ICMS/IPI, não há uma apuração por filial.
- **CBS e IBS são apurados separadamente.** Cada competência tem um cartão por imposto, com resultado, conciliação e fechamento próprios.

### IBS: apenas a apuração própria, por enquanto

Hoje, só a Receita Federal publica a Apuração Assistida, e apenas para a **CBS**. O Comitê Gestor do IBS ainda não disponibilizou o serviço equivalente. Por isso:

- O **IBS** já tem apuração própria, com os movimentos de **IBS UF** e **IBS Mun** consolidados sob o imposto **IBS**. Ele também pode ser fechado, com o próprio modelo de declaração.
- A solicitação da apuração do fisco e as credenciais do IBS ficam ocultas até o serviço existir. Sem dados do fisco, a competência do IBS não tem o que conciliar.

---

## 2. Pré-requisitos

| Item | Onde |
| --- | --- |
| Apps **Localization for Brazil** e **Livros Fiscais** (Tax Books) instalados | Documento de instalação geral |
| Reforma Tributária configurada, com IBS/CBS calculados nos documentos | [Reforma Tributária - Configurações](Localization-for-Brazil_User-Manual_Reforma-Tributaria-Configuracoes.md) |
| Documentos fiscais do período registrados, com IBS/CBS destacados | Movimentação normal de vendas e compras |
| **Contabilização Imposto** de IBS/CBS das compras com o campo **Crédito Imposto** definido. É ele que diz se a aquisição gera crédito | [Contabilização de IBS/CBS](Localization-for-Brazil_User-Manual_Contabilizacao-IBS-CBS.md) |
| Um **Modelo de Declaração de Imposto** para a CBS (e outro para o IBS, se for fechar o IBS) | Página **Modelos de Declaração de Impostos** (`TaxStatementTemplatesFNX`) |
| Plano de contas com as contas **a Apropriar**, **a Recuperar** e **a Recolher** de cada imposto, e um fornecedor que represente o fisco | Plano de Contas e Fornecedores |
| **Client ID** e **Client Secret** da Apuração Assistida da CBS, gerados para a matriz da empresa | Receita Federal |

---

## 3. Configuração

### 3.1 Modelo de Declaração de Imposto

O fechamento da competência gera uma **Declaração de Imposto** (`TaxStatementFNX`) a partir do modelo informado na configuração da apuração. O modelo define a numeração, a regra de aprovação, a descrição dos lançamentos e as contas do título a pagar.

1. Pesquise por **Modelos de Declaração de Impostos** (`TaxStatementTemplatesFNX`).
2. Crie um modelo para a CBS:

| Campo | Valor |
| --- | --- |
| **Classificação Imposto** (`TaxIdentification`) | **CBS**. Para o modelo do IBS, use **IBS UF** ou **IBS Mun** |
| **Nº Série Declaração** (`StatementNoSeries`) | Série que numera as declarações do modelo |
| **Tipo de Conta Débito** / **Nº Conta Débito** (`DebitAccountType` / `DebitAccountNo`) | Conta contábil do **imposto a recolher** (passivo). É a conta debitada no título a pagar e no encontro de contas |
| **Tipo de Conta Crédito** / **Nº Conta Crédito** (`CreditAccountType` / `CreditAccountNo`) | Fornecedor que representa o fisco. O título a pagar é gerado contra ele |

O campo do modelo na configuração da apuração só aceita um modelo do mesmo imposto: informar um modelo de CBS no campo do IBS, ou o contrário, gera erro.

### 3.2 Configuração da Apuração de IBS/CBS

Pesquise por **Configuração da Apuração de IBS/CBS** (página `IBSCBSSettlementSetup`). A página também aparece na lista de configurações manuais, na categoria dos Livros Fiscais.

| Campo | Descrição |
| --- | --- |
| **Cód. Filial** (`BranchCode`) | Filial em que as declarações da apuração são lançadas. Só aparece quando a empresa usa filiais e já vem preenchido com a matriz (a filial cujo CNPJ tem a ordem `0001`). Como o IBS e a CBS são apurados pela empresa inteira, normalmente fica a matriz |
| **Conta CBS a Apropriar** (`CBSAppropriationAccountNo`) | Conta que guarda o crédito de CBS das aquisições enquanto ele ainda é potencial |
| **Conta CBS a Recuperar** (`CBSRecoverableAccountNo`) | Conta definitiva de CBS a recuperar, para onde o crédito vai quando o fisco confirma a apropriação |
| **Modelo de Declaração de Imposto CBS** (`CBSStatementTemplateCode`) | Modelo da seção 3.1 usado no fechamento da CBS |
| **Client ID da CBS** / **Client Secret da CBS** | Credenciais da Apuração Assistida da CBS, emitidas pela Receita Federal |
| **Conta IBS a Apropriar**, **Conta IBS a Recuperar**, **Modelo de Declaração de Imposto IBS** | Os mesmos campos, para o IBS |
| **URL do Serviço** (`ServiceURL`), na guia **Serviço** | Endereço do serviço de integração que conversa com o fisco. Deixe em branco para usar o serviço padrão |

> **Conta a Apropriar e Contabilização Imposto**: a contabilização feita no registro do documento não muda. Quem quiser segregar o crédito ainda não confirmado aponta a conta de imposto a recuperar da **Contabilização Imposto** para a **Conta CBS a Apropriar** (ou **Conta IBS a Apropriar**). É uma decisão de plano de contas, não uma configuração adicional. No fechamento, a apuração transfere o crédito que o fisco apropriou da conta a apropriar para a conta a recuperar.

### 3.3 Credenciais

O **Client ID** e o **Client Secret** ficam guardados no armazenamento isolado da empresa, criptografados quando a criptografia está ativa no servidor. Depois de informado, o **Client Secret** aparece mascarado. Para trocá-lo, digite o novo valor. Para removê-lo, limpe o campo.

A credencial deve ser gerada para a **matriz** da empresa e ter permissão para usar a API da Apuração Assistida da CBS. Se o fisco aceitar a credencial mas negar o acesso ao serviço, a mensagem de erro informa isso.

As credenciais do IBS só aparecem quando o Comitê Gestor publicar o serviço do IBS.

### 3.4 Atualização automática

Cada consulta ao fisco traz apenas o que foi incluído ou alterado desde a consulta anterior, dentro de uma janela de **oito dias**. O que fica fora dessa janela não volta em consultas posteriores. Por isso, as consultas precisam rodar com regularidade.

Na **Configuração da Apuração de IBS/CBS**, clique na ação **Agendar Atualização Automática**. O sistema cria uma entrada na fila de tarefas que, uma vez por dia, baixa o que estiver disponível e faz a próxima solicitação. Se a entrada já existir, o sistema apenas avisa. Impostos sem Apuração Assistida ou sem credenciais são ignorados.

---

## 4. Como Acessar

- Pesquise por **Apurações de IBS/CBS** (página `IBSCBSSettlementList`).
- Nos Centros de Funções do contador, do gestor de negócios, do fiscal e do processador de pedidos, use as ações **Apurações de IBS/CBS** e **Configuração da Apuração de IBS/CBS**.

---

## 5. A lista e o cartão da apuração

### 5.1 Lista Apurações de IBS/CBS

A lista mostra uma linha por **Competência** e **Imposto**, da mais recente para a mais antiga. Cada linha traz o **Status do Calendário**, o **Status**, o **Saldo Devedor**, o **Saldo Credor** e o **Resultado da Apuração** do fisco, além da contagem de documentos por veredito. A lista é somente leitura. As competências não são criadas nem excluídas à mão: elas nascem da consulta ao fisco ou da ação **Gerar Apuração Própria de um Mês**.

Ações da lista:

| Ação | O que faz |
| --- | --- |
| **Solicitar Apuração do Fisco** | Pede ao fisco a apuração do imposto (seção 7.1) |
| **Consultar Apuração do Fisco** | Verifica se a apuração solicitada já está pronta e, se estiver, baixa e aplica o resultado em todas as competências que ele cobre |
| **Gerar Apuração Própria de um Mês** | Pede o **Ano**, o **Mês** e o **Imposto**, cria a competência quando ela ainda não existe, gera a apuração própria e abre o cartão. Serve para um mês que o fisco ainda não trouxe |
| **Solicitações** | Abre as **Solicitações de Apuração de IBS/CBS** (seção 7.2) |

### 5.2 Status do Calendário

O **Status do Calendário** é calculado pela data e diz em que fase a competência está:

| Status do Calendário | Período |
| --- | --- |
| **Em Andamento** | Durante o próprio mês da competência |
| **Período de Ajuste** | Do dia 1º até o dia 25 do mês seguinte |
| **Concluída** | A partir do dia 26 do mês seguinte. Só nesta fase a competência pode ser fechada |

O **Status** (**Aberta** / **Fechada**) é outra informação: diz se a competência já foi fechada para aquele imposto.

### 5.3 Cartão Apuração de IBS/CBS

O cartão (página `IBSCBSSettlementCard`) é onde todo o trabalho acontece. Ele tem quatro grupos de campos e, abaixo deles, as duas listas de linhas, uma em cima da outra.

| Grupo | Campos | O que mostra |
| --- | --- | --- |
| **Geral** | Competência, Imposto, Status do Calendário, Status, Apuração Própria Gerada Em, Dados do Fisco Aplicados Em | Identificação da competência e quando cada lado foi atualizado pela última vez |
| **Apuração do Fisco** | Saldo Devedor, Saldo Credor, Outros Ajustes, Resultado da Apuração. Ao mostrar mais campos: Redutores de Débitos, Redutores de Créditos, Créditos de Apuração Anterior, Pagamentos Utilizados | O resultado do fisco no formato do portal (seção 7.3). Clicar em um valor abre as linhas do fisco daquela conta |
| **Conciliação** | Documentos Conferidos, Somente no Business Central, Somente no Fisco, Documentos Divergentes, Diferença de Crédito, Movimentos Pós-Fechamento | Os totais da conciliação (seções 8 e 9.5) |
| **Fechamento** | Nº Declaração de Imposto, Status da Declaração de Imposto, Crédito Transportado. Ao mostrar mais campos: Fechada Em, Fechada Por, Resultado no Fechamento | A declaração gerada no fechamento e o saldo credor transportado (seção 9) |
| **Linhas da Apuração Própria** e **Linhas da Apuração do Fisco** | — | As linhas de cada lado, já filtradas pela competência e pelo imposto do cartão |

Ações do cartão, na ordem em que o trabalho é feito:

| Grupo | Ação | O que faz |
| --- | --- | --- |
| Processo | **Gerar Apuração Própria** | Refaz as linhas próprias do imposto (seção 6) |
| Processo | **Conciliar** | Confronta os dois lados de novo (seção 8) |
| Conciliação | **Linhas de Documentos** | Mostra as linhas dos documentos fiscais da competência com IBS e CBS lado a lado (seção 6.3) |
| Conciliação | **Somente no Fisco** | Lista os documentos que o fisco tem e a empresa ainda não registrou |
| Conciliação | **Somente no Business Central** | Lista os documentos que só existem no Business Central |
| Conciliação | **Divergente** | Lista os documentos cujo valor difere entre os dois lados |
| Fechamento | **Fechar Apuração** | Fecha a competência para o imposto (seção 9) |
| Fechamento | **Reabrir Apuração** | Reabre a competência para o imposto (seção 9.4) |
| Fechamento | **Declaração de Imposto** | Abre a declaração gerada no fechamento |

---

## 6. Apuração própria

### 6.1 Gerar Apuração Própria

No cartão, clique em **Gerar Apuração Própria**. O sistema percorre os documentos fiscais **registrados** cuja **data de registro** cai na competência, de todas as filiais, e cria uma linha por documento, por classificação de imposto (**CBS**, **IBS UF**, **IBS Mun**) e por conta. Documentos cancelados não entram.

A competência do documento próprio é a **data de registro**, o mesmo critério usado no ICMS/IPI e no PIS/COFINS. O fisco usa a data do documento eletrônico. Os dois critérios divergem na virada do mês, e a conciliação trata isso (seção 8.2).

A ação pode ser repetida quantas vezes for necessário. Cada execução atualiza as linhas e preserva o veredito, os vínculos manuais e o status do crédito. Linhas de documentos que foram cancelados, estornados ou saíram da competência são removidas. A ação refaz apenas o imposto do cartão e não mexe no outro imposto do mês. Uma competência já fechada para o imposto não pode ser refeita.

### 6.2 Natureza de cada linha

A apuração própria segue a forma da Apuração Assistida da CBS, que trabalha com débitos e créditos e identifica devoluções e notas de crédito pela origem do movimento:

| Situação do documento | Conta da linha própria | Observação |
| --- | --- | --- |
| Saída (venda, nota de débito, devolução de compra) | **Débitos** | A **Finalidade** e o **Tipo Nota de Débito** ficam registrados na linha |
| Entrada com direito a crédito (compra, devolução de venda, nota de crédito recebida) | **Créditos** | **Status Crédito** inicial **Potencial**. O **Tipo Nota de Crédito** fica registrado na linha |
| Entrada sem direito a crédito | **Créditos** | **Valor Apropriável** zero, **Sem Direito a Crédito** marcado e **Status Crédito** **Sem Direito a Crédito**. A linha existe para que o documento não apareça como falso **Somente no Fisco** |
| Documento com estorno de crédito (`gEstornoCred`) | **Débitos** | O valor do estorno entra como débito do documento. Veja [Estorno de Crédito IBS/CBS](Localization-for-Brazil_User-Manual_EstornoCreditoIBSCBS.md) |
| Documento cancelado | — | Não entra na apuração |

O direito a crédito vem do campo **Crédito Imposto** da **Contabilização Imposto** usada no movimento de IBS/CBS da compra.

Nas **Linhas da Apuração Própria**, a ação **Documento Fiscal** abre o documento de origem da linha.

### 6.3 Linhas de Documentos

A ação **Linhas de Documentos** abre a página **Linhas de Documentos IBS/CBS** (`IBSCBSDocumentLines`). Ela mostra uma linha por linha de documento fiscal registrado da competência que tenha IBS ou CBS, com o **Nº Nota Fiscal**, o item, o CFOP, as classificações tributárias e os valores de imposto lado a lado: **Base IBS/CBS**, **Valor CBS**, **Valor IBS UF**, **Valor IBS Mun** e **Valor Estorno de Crédito IBS/CBS**.

Os valores são lidos dos movimentos de imposto dos documentos fiscais e nada é copiado. A **Base IBS/CBS** mostra a maior base entre os movimentos de IBS e CBS da linha, e não a soma delas.

---

## 7. Apuração do fisco

### 7.1 Solicitar e consultar

A Apuração Assistida é pedida por imposto e cobre todas as competências que tiverem movimento. A resposta do fisco demora horas.

1. Em **Apurações de IBS/CBS**, clique em **Solicitar Apuração do Fisco**. Quando os dois impostos tiverem Apuração Assistida, o sistema pergunta **Qual imposto?**. Hoje, com apenas a CBS disponível, a pergunta não aparece.
2. O sistema envia duas consultas, uma de **Débitos** e outra de **Créditos**, e informa o número de cada solicitação e a previsão do fisco. Continue trabalhando: a competência segue utilizável com os dados próprios.
3. Mais tarde, clique em **Consultar Apuração do Fisco**. Quando o resultado está pronto, o sistema baixa e aplica o conteúdo em todas as competências que ele cobre, criando as que ainda não existem. A mensagem informa, competência por competência, quantos débitos e créditos foram gravados.

Com a **atualização automática** agendada (seção 3.4), os dois passos rodam sozinhos uma vez por dia.

Regras da solicitação:

- **Uma solicitação por imposto de cada vez.** Uma nova solicitação de CBS, com outra de CBS ainda em andamento, é recusada com uma mensagem que informa a solicitação pendente. Solicitações de CBS e de IBS não interferem uma na outra.
- **Sem credenciais**, a solicitação é recusada com uma mensagem que diz o que falta preencher na **Configuração da Apuração de IBS/CBS**, sem gastar nenhuma tentativa.
- Cada movimento devolvido pelo fisco vai para a competência a que pertence, pela data de apuração do fisco. Uma mesma resposta pode alimentar várias competências.
- Uma nova consulta substitui somente os dados do fisco do imposto consultado. A apuração própria e o outro imposto não mudam.

### 7.2 Solicitações de Apuração de IBS/CBS

A página **Solicitações de Apuração de IBS/CBS** (`IBSCBSSettlementRequests`) registra cada pedido feito ao fisco, com **Imposto**, **Consulta** (**Débitos** ou **Créditos**), **Status**, **Status no Fisco**, **ID de Correlação**, **Previsto Em**, **Resultado Disponível Em**, **Disponível Até**, **Baixado Em** e **Mensagem de Erro**.

O **Status** de uma solicitação pode ser:

| Status | Significado |
| --- | --- |
| **Solicitado** | Aguardando o fisco |
| **Disponível** | O resultado está pronto para ser baixado |
| **Baixado** | O resultado foi baixado e aplicado às competências |
| **Erro** | O fisco não conseguiu gerar o resultado, ou o conteúdo baixado não pôde ser aplicado |
| **Abandonado** | O usuário desistiu da solicitação |
| **Expirado** | O prazo de download passou. É preciso solicitar de novo |

Ações:

| Ação | O que faz |
| --- | --- |
| **Consultar** | Pergunta ao fisco se o resultado da solicitação está pronto |
| **Baixar** | Baixa o resultado da solicitação e aplica nas competências que ele cobre |
| **Aplicar Novamente** | Reaplica um conteúdo que foi baixado mas não pôde ser aplicado, depois de corrigida a causa, sem consultar o fisco de novo |
| **Abandonar Solicitação** | Desiste de uma solicitação que o fisco não responde, para liberar uma nova. O resultado dela nunca mais será consultado nem baixado, mesmo que chegue depois |
| **Conteúdo Original** | Baixa o conteúdo original devolvido pelo fisco, que comprova a origem de cada valor apresentado |

### 7.3 O resultado no formato do portal

As linhas do fisco são classificadas nas contas do portal da Apuração Assistida, com os mesmos nomes e a mesma ordem:

| Conta | Efeito no resultado |
| --- | --- |
| **Débitos** | (+) o saldo ainda não extinto de cada débito |
| **Redutores de Débitos** | (−) |
| **Créditos** | (−) o saldo credor de cada crédito: a parte apropriada e ainda não utilizada |
| **Redutores de Créditos** | (+) |
| **Créditos de Apuração Anterior** | (−) |
| **Pagamentos Utilizados** | (−) |

No cartão:

- **Saldo Devedor**: o que ainda é devido nos débitos, depois de descontados o que foi extinto (compensação com créditos e pagamentos, inclusive o recolhimento pelo adquirente), o pagamento antecipado e os valores suspensos ou inexigíveis.
- **Saldo Credor**: a parte apropriada dos créditos que ainda não foi utilizada. O crédito ainda a apropriar e o crédito já utilizado não entram.
- **Outros Ajustes**: o que as demais contas (redutores, créditos de apuração anterior e pagamentos utilizados) somam ao resultado. Clicar no valor abre essas linhas.
- **Resultado da Apuração**: a soma das contas com seus sinais. Valor positivo é saldo **devedor**, e valor negativo é saldo **credor**. É o resultado que confere com o portal do fisco.

As contas que o arquivo do fisco não trouxer aparecem zeradas. Movimentos sem chave de acesso, como créditos de apuração anterior e pagamentos utilizados, entram apenas no resultado e não participam da conciliação por documento.

Em cada linha do fisco, os campos **Valor Extinto**, **Valor Pagamento Antecipado**, **Valor Suspenso**, **Valor Inexigível**, **Valor Não Extinto** e **Situação** (**Não Extinto**, **Extinto Parcial**, **Extinto Total**, **Suspenso**, **Inexigível**) detalham os débitos. Nos créditos, a **Situação do Crédito** e os campos **Valor a Apropriar**, **Valor Apropriado**, **Valor Utilizado** e **Saldo Credor** mostram a fase em que o crédito está. O CNPJ/CPF do emitente, a UF, a série e o número do documento são lidos da chave de acesso.

### 7.4 Linhas do fisco digitadas manualmente

A consulta ao fisco só traz o que mudou nos últimos oito dias. Um documento que ficou fora dessa janela não pode ser recuperado pela consulta. Para esses casos, as linhas do fisco podem ser digitadas:

1. Em **Linhas da Apuração do Fisco**, no cartão, clique em **Editar Lista**.
2. Inclua a linha com a **Conta** **Débitos** ou **Créditos**, a **Chave de Acesso do DF-e** e os valores do documento como aparecem no portal. A linha fica marcada como **Digitada Manualmente**.

Regras:

- Só as linhas digitadas podem ser alteradas ou excluídas. As linhas baixadas do fisco mudam apenas com uma nova consulta.
- Uma linha digitada aceita somente as contas **Débitos** e **Créditos**.
- A competência e o imposto de uma linha não podem ser alterados. Para mudar, exclua a linha e digite de novo na competência certa.
- Uma competência fechada não aceita linhas novas.
- Quando uma consulta posterior traz o mesmo documento, com a mesma consulta e a mesma origem, a linha baixada substitui a digitada.

---

## 8. Conciliação

### 8.1 Os quatro vereditos

A conciliação roda sozinha a cada consulta aplicada e também pela ação **Conciliar** do cartão, que pode ser repetida quantas vezes for necessário. O resultado sempre reflete o último download. O par é identificado pela **chave de acesso do DF-e** e pelo **imposto**, em débitos e em créditos. Cada linha, dos dois lados, recebe um **Status Conciliação**:

| Status Conciliação | Significado | O que fazer |
| --- | --- | --- |
| **Conferido** | O documento existe nos dois lados com o mesmo valor | Nada |
| **Somente no Business Central** | O documento foi registrado no Business Central e o fisco não o tem | Verificar se o documento ainda está em processamento no fisco ou se houve falha de transmissão |
| **Somente no Fisco** | O fisco tem o documento e a empresa não o registrou | Registrar o documento pelo fluxo normal de entrada e gerar a apuração própria de novo |
| **Divergente** | O documento existe nos dois lados com valores diferentes | Conferir o **Valor do Fisco** e o **Valor da Diferença** da linha própria e tratar no período de ajuste |

O cartão mostra a quantidade de documentos de cada veredito. Os campos **Somente no Business Central**, **Somente no Fisco** e **Documentos Divergentes**, assim como as ações de mesmo nome, abrem as linhas correspondentes.

**Pendências de registro.** A apuração não cria nem registra documentos. Um documento **Somente no Fisco** é uma pendência: registre-o pelo fluxo normal de entrada e use **Gerar Apuração Própria**. O par passa a **Conferido** sem nenhuma conciliação manual.

### 8.2 Competência vizinha

Como o Business Central data o documento pela data de registro e o fisco pela data do documento eletrônico, uma NF-e emitida em 28/02 e registrada em 05/03 fica em competências diferentes nos dois lados. Quando o par não é encontrado na competência, a conciliação procura nas competências vizinhas (mês anterior e mês seguinte). Encontrando, o par fica **Conferido**, as duas linhas ganham a marca **Competência Vizinha** e o campo **Cod. Apuração Vinculada** mostra onde está a contraparte.

Nenhum movimento muda de competência, então o resultado de cada competência continua igual ao do portal. Uma competência vizinha já fechada nunca é alterada.

### 8.3 Vínculo manual

Quando o par não é encontrado pela chave de acesso, ele pode ser feito à mão nas **Linhas da Apuração Própria**:

- **Vincular Manualmente**: liga a linha própria a uma linha do fisco do mesmo imposto. O **Tipo Vínculo** passa a indicar o vínculo manual.
- **Remover Vínculo**: desfaz o vínculo.

O vínculo manual é lembrado pela chave de acesso da contraparte e sobrevive a uma nova consulta, que substitui as linhas do fisco. Se a contraparte deixar de existir, a linha volta ao veredito automático.

### 8.4 Status do crédito e Diferença de Crédito

Nas linhas próprias de crédito, o **Status Crédito** começa como **Potencial**, e o que vem depois é informado pelo fisco: **Pendente**, **Apropriado**, **Inapropriável**, **Suspenso** ou **Prescrito**. A conciliação atualiza o status a partir da **Situação do Crédito** da linha do fisco correspondente. Um crédito **Inapropriável** indica, por exemplo, um fornecedor que não extinguiu o débito dele.

O campo **Diferença de Crédito** do cartão mostra quanto do crédito esperado pelas aquisições da competência o fisco ainda não reconheceu como apropriado. É o valor a cobrar dos fornecedores ou a reconhecer como perda.

---

## 9. Fechamento

### 9.1 Condições

O fechamento é **por imposto** e usa o **resultado do fisco**, nunca o cálculo próprio. Para fechar:

- o **Status do Calendário** deve estar **Concluída**;
- deve haver dados do fisco aplicados à competência para o imposto;
- o **Modelo de Declaração de Imposto** do imposto deve estar informado na configuração.

**Divergências não bloqueiam.** Se houver documentos **Divergentes**, **Somente no Business Central** ou **Somente no Fisco**, o sistema mostra quantos são de cada tipo e pergunta se deseja fechar mesmo assim. Confirmando, a competência é fechada e as divergências continuam registradas nas linhas.

### 9.2 Fechar Apuração

No cartão, clique em **Fechar Apuração**. O sistema:

1. Gera a **Declaração de Imposto** a partir do modelo do imposto, com as datas da competência e a filial da configuração. No primeiro fechamento a declaração é criada, e nos seguintes é refeita enquanto estiver aberta.
2. Monta o demonstrativo da declaração com a apuração do fisco:

   | Bloco | Linhas |
   | --- | --- |
   | Débitos | Débito apurado, (-) Extinto, (-) Pagamento antecipado, (-) Suspenso, (-) Inexigível, **Saldo devedor** |
   | Créditos | Crédito apurado, Crédito apropriado, (-) Utilizado na extinção de débitos, **Saldo credor** |
   | Resultado | Saldo devedor, (-) Saldo credor, Outros ajustes do fisco, **Imposto a recolher** ou **Saldo credor a transportar** |

   As linhas sem valor são omitidas, exceto os totais.
3. Cria os lançamentos da declaração, cada um quando houver valor:

   | Lançamento | Débito | Crédito | Valor |
   | --- | --- | --- | --- |
   | Apropriação de créditos | **Conta a Recuperar** do imposto | **Conta a Apropriar** do imposto | Créditos apropriados pelo fisco |
   | Título a pagar | **Nº Conta Débito** do modelo (imposto a recolher) | **Nº Conta Crédito** do modelo (fornecedor do fisco) | Resultado devedor, com vencimento no último dia útil do mês seguinte à competência |
   | Compensação de créditos (encontro de contas) | **Nº Conta Débito** do modelo (imposto a recolher) | **Conta a Recuperar** do imposto | Crédito usado na extinção de débitos da competência |

4. Marca a competência como **Fechada** para o imposto, grava o **Resultado no Fechamento** e abre a declaração.

O **Valor Aprovação** da declaração é o imposto a recolher, e não a soma dos lançamentos.

A guia de recolhimento é emitida fora do Business Central, nos sistemas do fisco. A baixa do título acontece na liquidação, pelo fluxo normal de pagamentos.

### 9.3 Declaração de Imposto: aprovar e registrar

Os lançamentos ainda não estão na contabilidade quando a competência é fechada. Na **Declaração de Imposto**:

- ajuste as **Dimensões**, se necessário;
- use **Liberar** ou **Enviar Requisição Aprovação**, conforme a regra de aprovação do modelo;
- clique em **Registrar** para gerar os lançamentos contábeis e o título no fornecedor do fisco;
- use **Estornar** para desfazer uma declaração já registrada.

### 9.4 Saldo credor e reabertura

- **Saldo credor**: quando o resultado é credor, não há título a pagar. O valor fica no ativo, na conta a recuperar, e aparece no campo **Crédito Transportado** do cartão.
- **Reabrir Apuração**: devolve a competência ao status **Aberta** para que os dados do fisco sejam aplicados de novo e a competência seja fechada outra vez. A declaração é mantida e refeita no próximo fechamento. A reabertura é recusada se a declaração estiver **Registrado** (estorne-a antes) ou **Pendente Aprovação** (cancele a requisição antes).

### 9.5 Fatos comunicados depois do fechamento

Uma competência fechada não muda com uma nova consulta. Um fato novo que o fisco comunicar depois do fechamento fica na competência de origem, marcado como **Pós-Fechamento**, separado do resultado já fechado. Os campos **Movimentos Pós-Fechamento** e **Valor Pós-Fechamento** do cartão mostram quantos são e quanto somam. O **Resultado da Apuração** e o título da competência fechada não mudam.

---

## 10. Exemplo Prático

Competência **2026-12** da CBS, com três vendas e três compras com direito a crédito.

**Apuração própria** (após **Gerar Apuração Própria**):

| Documento | Conta | Valor CBS | Status Crédito |
| --- | --- | ---: | --- |
| NF-e de venda 1 | **Débitos** | 1.000,00 | |
| NF-e de venda 2 | **Débitos** | 600,00 | |
| NF-e de venda 3 | **Débitos** | 400,00 | |
| NF-e de compra 1 | **Créditos** | 100,00 | **Potencial** |
| NF-e de compra 2 | **Créditos** | 80,00 | **Potencial** |
| NF-e de compra 3 | **Créditos** | 50,00 | **Potencial** |

**Apuração do fisco** (após **Consultar Apuração do Fisco**):

| Documento | Situação no fisco | Efeito |
| --- | --- | --- |
| Venda 1 | **Extinto Total**: recolhido pelo adquirente | Não soma ao saldo devedor |
| Venda 2 | **Extinto Parcial**: 180,00 compensados com créditos | Soma 420,00 |
| Venda 3 | **Não Extinto** | Soma 400,00 |
| Compra 1 | Apropriado e utilizado | Não soma ao saldo credor |
| Compra 2 | Apropriado e utilizado | Não soma ao saldo credor |
| Compra 3 | A apropriar | Não soma ao saldo credor |

**Resultado no cartão** (após a conciliação):

| Campo | Valor |
| --- | ---: |
| **Documentos Conferidos** | 6 |
| **Saldo Devedor** | 820,00 |
| **Saldo Credor** | 0,00 |
| **Resultado da Apuração** | 820,00 (devedor) |
| **Diferença de Crédito** | 50,00 (crédito da compra 3, ainda a apropriar) |

O **Status Crédito** das compras 1 e 2 passa a **Apropriado**, e o da compra 3 a **Pendente**.

**Fechamento** (a partir de 26/01/2027, com **Fechar Apuração**): a **Declaração de Imposto** traz o demonstrativo e três lançamentos:

| Lançamento | Valor |
| --- | ---: |
| Apropriação de créditos CBS 2026-12 | 180,00 |
| Título a pagar, com vencimento em 29/01/2027 | 820,00 |
| Compensação de créditos CBS 2026-12 | 180,00 |

A CBS de 2026-12 fica **Fechada**, com **Resultado no Fechamento** de 820,00 e **Crédito Transportado** zero.

---

## 11. O que a apuração não faz

- **Não emite a guia de recolhimento**, que é gerada nos sistemas do fisco, e não faz o pagamento, que segue o fluxo normal de baixa.
- **Não cria nem registra documentos**: um documento **Somente no Fisco** é apenas apontado.
- **Não altera o cálculo** de IBS/CBS por documento nem a contabilização feita no registro do documento.
- **Não trata** *split payment*, recolhimento pelo adquirente como lançamento próprio, pedido de ressarcimento de saldo credor nem o Imposto Seletivo.
- **Não apura ICMS, IPI, PIS e COFINS**, que continuam nas apurações atuais.
- **IBS estadual × municipal**: a forma de apresentar as duas parcelas na apuração do fisco será definida quando o Comitê Gestor do IBS publicar o serviço.
- **Carga de arquivo**: o resultado do fisco é obtido pela consulta online. Não há ação na tela para carregar um arquivo de apuração salvo em disco.

---

## 12. Resumo dos Campos Novos

### Tabela Configuração da Apuração de IBS/CBS (`IBSCBSSettlementSetup`)

| Campo | Descrição |
| --- | --- |
| **Cód. Filial** (`BranchCode`) | Filial em que as declarações da apuração são lançadas |
| **Conta CBS a Apropriar** (`CBSAppropriationAccountNo`) | Conta do crédito de CBS ainda potencial |
| **Conta CBS a Recuperar** (`CBSRecoverableAccountNo`) | Conta definitiva de CBS a recuperar |
| **Modelo de Declaração de Imposto CBS** (`CBSStatementTemplateCode`) | Modelo da declaração gerada no fechamento da CBS |
| **Conta IBS a Apropriar** (`IBSAppropriationAccountNo`) | Conta do crédito de IBS ainda potencial |
| **Conta IBS a Recuperar** (`IBSRecoverableAccountNo`) | Conta definitiva de IBS a recuperar |
| **Modelo de Declaração de Imposto IBS** (`IBSStatementTemplateCode`) | Modelo da declaração gerada no fechamento do IBS |
| **URL do Serviço** (`ServiceURL`) | Endereço do serviço de integração com o fisco |

### Tabela Linha de Documento Fiscal (`FiscalDocumentLineFNX`)

Campos calculados, mostrados na página **Linhas de Documentos IBS/CBS**:

| Campo | Descrição |
| --- | --- |
| **Nº Nota Fiscal** (`FiscalNoFNX`) | Número fiscal do documento |
| **Base IBS/CBS** (`IBSCBSBaseAmountFNX`) | Maior base entre os movimentos de IBS e CBS da linha |
| **Valor CBS** (`CBSAmountFNX`) | CBS da linha |
| **Valor IBS UF** (`IBSUFAmountFNX`) | IBS estadual da linha |
| **Valor IBS Mun** (`IBSMunAmountFNX`) | IBS municipal da linha |
| **Valor Estorno de Crédito IBS/CBS** (`IBSCBSCreditReversalAmtFNX`) | Estorno de crédito de IBS/CBS da linha |

### Tabela Movimento de Declaração de Imposto (`TaxStatementEntryFNX`)

| Campo | Descrição |
| --- | --- |
| **Data Vencimento** (`DueDate`) | Vencimento do título gerado pelo lançamento. Em branco, o vencimento vem da jurisdição do imposto, quando houver |

---

## Documentos relacionados

- [Reforma Tributária - Configurações](Localization-for-Brazil_User-Manual_Reforma-Tributaria-Configuracoes.md)
- [Contabilização de IBS/CBS](Localization-for-Brazil_User-Manual_Contabilizacao-IBS-CBS.md)
- [Estorno de Crédito IBS/CBS (gEstornoCred)](Localization-for-Brazil_User-Manual_EstornoCreditoIBSCBS.md)
- [Apuração de ISS](Localization-for-Brazil_User-Manual_Apuracao-ISS.md), que usa a mesma **Declaração de Imposto**
