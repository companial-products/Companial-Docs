# Reforma Tributária: Eventos da NF-e

## 1. O que são Eventos da NF-e?

Com a Reforma Tributária do Consumo (EC 132/2023 + LC 214/2025), o governo criou um novo conjunto de obrigações denominado **Eventos**. Esses eventos são registros eletrônicos vinculados a uma NF-e já autorizada, que comunicam à SEFAZ situações ocorridas **após a emissão** do documento fiscal.

Eles são indispensáveis por dois motivos:

- **Rastreabilidade**: permitem acompanhar o que aconteceu com a mercadoria ou operação depois que a NF-e foi emitida (perda, imobilização, consumo pessoal, etc.).
- **Apuração correta de débitos e créditos de IBS/CBS**: o novo modelo tributário utiliza **apuração assistida** (o governo calcula e a empresa confere). Os eventos alimentam essa apuração com informações que não constam na NF-e original — por exemplo, se um item comprado foi destinado a consumo pessoal (sem direito a crédito) ou se houve perda durante o transporte.

## 2. Eventos disponíveis na localização

A localização possui **14 tipos de eventos** implementados, divididos entre fornecedor e adquirente:

#### 2.1. Eventos do Fornecedor (emissor da NF-e)

| Código | Descrição |
|--------|-----------|
| **111500** | Pedido de Prorrogação da suspensão do ICMS na Remessa (1º Prazo) |
| **111501** | Pedido de Prorrogação da suspensão do ICMS na Remessa (2º Prazo) |
| **111502** | Cancelamento de Pedido de Prorrogação da suspensão do ICMS (1º Prazo) |
| **111503** | Cancelamento de Pedido de Prorrogação da suspensão do ICMS (2º Prazo) |
| **112120** | Importação em ALC/ZFM não convertida em isenção |
| **112130** | Perecimento/perda/roubo/furto durante o transporte contratado pelo fornecedor |
| **112140** | Fornecimento não realizado com pagamento antecipado |
| **112150** | Atualização da Data de Previsão de Entrega |

#### 2.2. Eventos do Adquirente (destinatário da NF-e)

| Código | Descrição |
|--------|-----------|
| **211120** | Destinação de item para consumo pessoal |
| **211124** | Perecimento/perda/roubo/furto durante o transporte contratado pelo adquirente |
| **211128** | Aceite de débito na apuração por emissão de nota de crédito |
| **211130** | Imobilização de item |
| **211140** | Apropriação de crédito de combustível |
| **211150** | Apropriação de crédito para bens/serviços que dependem de atividade do adquirente |

## 3. Pré-requisito: Reforma Tributária ativa

Para utilizar os eventos, a Reforma Tributária deve estar ativa no ambiente. Essa ativação é controlada nas **Configurações Fiscais**, no campo de status da Reforma Tributária, que pode assumir três valores:

| Status | Comportamento |
|--------|--------------|
| **Agendado** (padrão) | Ativa automaticamente a partir da data programada (01/05/2026) |
| **Habilitado** | Sempre ativo |
| **Desabilitado** | Sempre inativo |

## 4. Como transmitir um evento — Passo a passo

Todo o processo é feito diretamente no Business Central, pela página **Evento NF-e**.

#### 1 — Criar o evento
Pesquise por "Eventos NF-e" no Business Central e crie um novo registro.

#### 2 — Preencher o cabeçalho

- **Tipo de Evento**: selecione o evento desejado na lista.
- **Nº Documento Fiscal**: informe o documento fiscal relacionado. Ao selecioná-lo, o sistema preenche automaticamente:
  - **Chave de Acesso** (44 dígitos)
  - **Cód. Filial**
  - **Nº Protocolo NF-e**
  - **Nº Participante** e **Nome do Participante**
- Se o documento fiscal não estiver no sistema, é possível informar a **Chave de Acesso** diretamente.

#### 3 — Preencher dados específicos do evento

Dependendo do tipo de evento selecionado, campos adicionais são exibidos e obrigatórios:

| Tipo de Evento | Campo obrigatório | Descrição |
|----------------|-------------------|-----------|
| **112150** (Atualização de data de entrega) | **Data de Previsão de Entrega** | Nova data estimada de entrega da mercadoria |
| **211128** (Aceite de débito) | **Indicador de Aceitação** | Sim ou Não — indica se o adquirente aceita o débito |
| **111500 / 111501** (Prorrogação ICMS) | **Nº Protocolo NF-e** | Protocolo de autorização da NF-e original |
| **111502 / 111503** (Cancelamento de prorrogação) | **Nº Protocolo do Evento** | Protocolo do evento de prorrogação que está sendo cancelado |

#### 4 — Preencher as linhas (itens)

A maioria dos eventos exige detalhamento por item. 

Os dois únicos que **não possuem linhas** são o **112150** (atualização de data de entrega) e o **211128** (aceite de débito).

Para os demais, cada linha contém:
- **Número do item** na NF-e
- **Descrição**
- **Quantidade** e **Unidade de medida**
- **Valor IBS** e **Valor CBS**

> **Facilidade**: a ação **"Carregar Itens do Xml"** permite importar automaticamente os itens a partir do XML da NF-e — seja o XML já armazenado no sistema, seja carregado manualmente de um arquivo. Isso evita preenchimento manual linha a linha.

#### 5 — Transmitir à SEFAZ

Com todos os campos preenchidos, clique na ação **"Registrar"** (Post). O sistema:

1. Valida todos os campos obrigatórios (tipo de evento, chave de acesso, campos específicos e linhas)
2. Exibe uma mensagem de confirmação
3. Gera o XML do evento
4. Transmite diretamente à SEFAZ
5. Atualiza o **Status** para **Registrado** e grava o **Nº do Protocolo** retornado pela SEFAZ

Se houver rejeição, o **Código do Último Status** e a **Mensagem do Último Erro** são atualizados para que o usuário possa diagnosticar o problema.

#### 6 — Consultar o XML transmitido

Após o registro, duas ações ficam disponíveis:
- **"Exibir XML"**: baixa o XML do evento processado pela SEFAZ
- **"Exibir XML de Teste"**: pré-visualiza o XML que seria gerado, útil para conferir antes de transmitir

#### 7 — Cancelar um evento (quando aplicável)

Eventos já registrados podem ser cancelados pela ação **"Cancelar"**, que transmite uma mensagem de cancelamento à SEFAZ. O status muda para **Cancelado** e o XML de cancelamento fica disponível para consulta.

> **Exceção importante**: os eventos de prorrogação de suspensão do ICMS (111500, 111501, 111502 e 111503) **não podem ser cancelados** pelo cancelamento padrão. Para desfazer uma prorrogação, deve-se usar o evento de cancelamento correspondente:
> - Para cancelar o **111500** (1º prazo), use o evento **111502**
> - Para cancelar o **111501** (2º prazo), use o evento **111503**

---

## Exemplos práticos:

### Evento 111500 – Pedido de Prorrogação da Suspensão do ICMS (1º Prazo)

#### Introdução

Nas remessas interestaduais para **industrialização, conserto ou reparo**, o ICMS tem sua exigência suspensa pelo Convênio AE 15/74, desde que a mercadoria retorne ao estabelecimento de origem em até **180 dias**. Quando esse prazo se mostra insuficiente — por complexidade técnica, atrasos na cadeia de fornecimento ou outros fatores —, a legislação faculta ao contribuinte prorrogar a suspensão por até dois períodos adicionais de igual duração, evitando que a operação seja descaracterizada e o imposto passe a ser exigido com acréscimos.

A formalização desse pedido, antes feita em papel, passou a ser eletrônica com a **Nota Técnica 2015.001**. O **evento 111500 – Pedido de Prorrogação 1º Prazo (EPP1)** é o primeiro dessa série e deve ser emitido pelo remetente quando, próximo do término dos 180 dias iniciais, se verificar a necessidade de estender a suspensão por mais um período. Após o registro, a Secretaria de Fazenda competente analisa o pedido e responde por meio do evento **411500 – Fisco Resposta ao Pedido de Prorrogação 1º Prazo**.

#### Passo a passo

1. **Pesquise por "Eventos NF-e"** no Business Central e abra a página.
2. **Crie um novo evento** e no campo **Tipo de Evento**, selecione `111500`.
3. No campo **Nº Documento Fiscal**, informe o documento fiscal da NF-e de remessa. O sistema preencherá automaticamente a **Chave de Acesso** e o **Nº Protocolo NF-e** (protocolo de autorização da NF-e original).
4. **Verifique o Nº Protocolo NF-e** — ele é obrigatório para a transmissão.
5. Na seção de **Linhas**, use a ação **Carregar Itens do Xml** para importar os itens da NF-e de remessa.
6. Clique em **Registrar** para transmitir o evento à SEFAZ.

#### Resposta da SEFAZ

A resposta do Fisco é recebida automaticamente pelo processo de consulta do **MD-e** (Manifesto do Destinatário Eletrônico). Quando processada, o sistema preenche automaticamente:

- No **cabeçalho**: **Status do Prazo** e **Descrição do Status do Prazo**
- Nas **linhas**: **Status do Pedido** (1 = Deferido, 2 = Indeferido), **Descrição do Status do Pedido**, **Cód. Justificativa do Status** e **Descrição da Justificativa** para cada item

### Evento 111501 – Pedido de Prorrogação da Suspensão do ICMS (2º Prazo)

#### Introdução

Esgotada a primeira prorrogação **(Evento 111500)** sem que o retorno tenha ocorrido, a legislação admite, excepcionalmente, uma **segunda prorrogação** — destinada a situações em que a complexidade técnica, a extensão do serviço contratado ou fatores supervenientes ainda inviabilizem a devolução da mercadoria no novo prazo concedido.

Sua emissão pressupõe a existência de um evento 111500 anterior deferido pelo Fisco e observa as mesmas regras de vinculação à NF-e original, devendo informar, item a item, quantidades nunca superiores às já prorrogadas. Após o registro, a Secretaria de Fazenda competente analisa o pedido e responde por meio do evento **411501 – Fisco Resposta ao Pedido de Prorrogação 2º Prazo**, encerrando o ciclo das prorrogações admitidas pela legislação.

#### Passo a passo

1. **Pesquise por "Eventos NF-e"** no Business Central e abra a página.
2. **Crie um novo evento** e no campo **Tipo de Evento**, selecione `111501`.
3. No campo **Nº Documento Fiscal**, informe o documento fiscal da NF-e de remessa. O sistema preencherá automaticamente a **Chave de Acesso** e o **Nº Protocolo NF-e**.
4. **Verifique o Nº Protocolo NF-e** — ele é obrigatório para a transmissão.
5. Na seção de **Linhas**, use a ação **Carregar Itens do Xml** para importar os itens da NF-e de remessa.
6. Clique em **Registrar** para transmitir o evento à SEFAZ.

> **Diferença em relação ao 111500**
>
> A única diferença é o contexto de uso: o **111501 é a segunda solicitação** de extensão do prazo, utilizada quando o primeiro prazo (concedido via 111500) também não foi suficiente. Os campos e requisitos são os mesmos.

------

### Evento **111502** - Cancelamento de Pedido de Prorrogação da suspensão do ICMS (1º Prazo)

#### Introdução

Após a emissão do **evento 111500 – Pedido de Prorrogação 1º Prazo**, podem ocorrer situações em que o contribuinte precise **desfazer a solicitação registrada**, seja por **retorno antecipado da mercadoria** ao estabelecimento de origem, por **erro no preenchimento** do pedido (itens, quantidades ou chave de acesso incorretos), ou por **alteração das condições da operação** que tornem a prorrogação desnecessária ou inadequada.

Este evento deve ser emitido pelo remetente para tornar sem efeito o pedido anteriormente registrado. O cancelamento é vinculado ao evento 111500. Após o registro, a Secretaria de Fazenda analisa a solicitação e responde por meio do evento **411502 – Fisco Resposta ao Cancelamento de Prorrogação 1º Prazo**, formalizando a decisão sobre o pleito de cancelamento.

#### Passo a passo

1. **Pesquise por "Eventos NF-e"** no Business Central e abra a página.
2. **Crie um novo evento** e no campo **Tipo de Evento**, selecione `111502`.
3. No campo **Nº Documento Fiscal**, informe o mesmo documento fiscal da NF-e de remessa utilizado no evento 111500 original.
4. No campo **Nº Protocolo do Evento**, selecione o protocolo do evento 111500 que deseja cancelar. O sistema apresenta apenas os eventos compatíveis (somente protocolos de eventos 111500 vinculados àquela chave de acesso).
5. Na seção de **Linhas**, use a ação **Carregar Itens do Xml** para importar os itens da NF-e de remessa.
6. Clique em **Registrar** para transmitir o cancelamento à SEFAZ.

#### Resposta da SEFAZ

Assim como nos pedidos de prorrogação, a resposta é recebida automaticamente via **MD-e**. No caso do cancelamento, o sistema preenche automaticamente:

- **Status do Prazo** e **Descrição do Status do Prazo**
- **Cód. Justificativa do Prazo** e **Descrição da Justificativa do Prazo**
- Nas linhas: **Status do Pedido**, **Descrição do Status do Pedido**, **Cód. Justificativa do Status** e **Descrição da Justificativa** para cada item

### Evento  **111503** - Cancelamento de Pedido de Prorrogação da suspensão do ICMS (2º Prazo)

#### Introdução

Após a emissão do **evento 111501 – Pedido de Prorrogação 2º Prazo**, podem ocorrer situações em que o contribuinte precise **desfazer a solicitação registrada**, seja por **retorno antecipado da mercadoria** ao estabelecimento de origem, por **erro no preenchimento** do pedido (itens, quantidades ou chave de acesso incorretos), ou por **alteração das condições da operação** que tornem a segunda prorrogação desnecessária ou inadequada.

Este evento deve ser emitido pelo remetente para tornar sem efeito o pedido anteriormente registrado. O cancelamento é vinculado ao evento 111501. Após o registro, a Secretaria de Fazenda analisa a solicitação e responde por meio do evento **411503 – Fisco Resposta ao Cancelamento de Prorrogação 2º Prazo**, formalizando a decisão sobre o pleito de cancelamento.

#### Passo a passo

1. Pesquise por **Eventos NF-e** no Business Central e abra a página.
2. Crie um novo evento e, no campo **Tipo de Evento**, selecione **111503**.
3. No campo **Nº Documento Fiscal**, informe o documento fiscal da NF-e de remessa original. O sistema preencherá automaticamente a **Chave de Acesso**.
4. No campo **Nº Protocolo do Evento**, selecione o protocolo do evento 111501 que deseja cancelar. O sistema filtra automaticamente e apresenta apenas os eventos compatíveis (somente eventos 111501 vinculados àquela chave de acesso).
5. Na seção **Linhas**, carregue os itens da NF-e de remessa utilizando a ação **Carregar Itens do Xml**.
6. Clique em **Registrar** para transmitir o cancelamento à SEFAZ.

#### Resposta da SEFAZ

Após o processamento, a resposta do Fisco é recebida automaticamente via consulta do MD-e (página **Números Sequenciais Únicos (NSU)**). O sistema preenche automaticamente no evento:

- **Status do Prazo** e **Descrição do Status do Prazo** (0 = Após o prazo, 1 = Dentro do prazo)
- **Cód. Justificativa do Prazo** e **Descrição da Justificativa do Prazo**
- Nas linhas: **Status do Pedido** por item (1 = Deferido, 2 = Indeferido) e respectiva justificativa

### Evento **112120** -  Importação em ALC/ZFM não convertida em isenção

#### Introdução

A **Lei Complementar nº 214/2025**, em seus **arts. 443 e 461**, suspende a incidência do **IBS** sobre a importação de bens materiais realizada por **indústrias incentivadas** destinados à utilização na **Zona Franca de Manaus (ZFM)** e nas **Áreas de Livre Comércio (ALC)**, respectivamente. Essa suspensão tem natureza condicional: pressupõe que os bens permaneçam na região incentivada e cumpram os requisitos legais que ensejam sua posterior **conversão em isenção**. O **§ 3º** de ambos os artigos, contudo, determina que, caso os bens importados sejam **remetidos para fora da ZFM ou da ALC antes da conversão em isenção**, os tributos suspensos deverão ser recolhidos.

Para essas hipóteses, está previsto o **evento 112120 – Importação em ALC/ZFM não convertida em isenção**, que deve ser emitido pelo contribuinte da ZFM ou ALC sobre a respectiva **nota fiscal de entrada do bem importado**, indicando o valor do IBS suspenso e não convertido em isenção. O efeito do evento na apuração assistida do IBS é a **geração de um débito na apuração do emitente**, correspondente ao tributo anteriormente suspenso, que — quando extinto — poderá gerar **direito a crédito**, observadas as regras aplicáveis à operação.

#### Passo a passo

1. Pesquise por **Eventos NF-e** no Business Central e abra a página.
2. Crie um novo evento e, no campo **Tipo de Evento**, selecione **112120 - Importação em ALC/ZFM não convertida em isenção**.
3. No campo **Nº Documento Fiscal**, informe o documento fiscal da NF-e de referência. O sistema preencherá automaticamente a **Chave de Acesso**.
4. Na seção **Linhas**, utilize a ação **Carregar Itens do Xml** para importar os itens da NF-e. O sistema carregará automaticamente os dados de cada item, incluindo:
   - **Número do Item** — posição do item na NF-e original
   - **Valor IBS** — valor do IBS (Imposto sobre Bens e Serviços) da linha
   - **Valor CBS** — valor da CBS (Contribuição sobre Bens e Serviços) da linha
   - **Quantidade** e **Cód. Unidade de Medida** — dados de controle de estoque do item
5. Confira os valores carregados e, se necessário, ajuste manualmente.
6. Clique em **Registrar** para transmitir o evento à SEFAZ.

> **Importante:**
>
> - O evento informa por item os valores de **IBS** e **CBS** que seriam devidos, além dos dados de controle de estoque (**Quantidade** e **Unidade de Medida**).

### Evento **112130 **-  Perecimento/perda/roubo/furto durante o transporte contratado pelo fornecedor

#### Introdução

Nos termos do **art. 10 da Lei Complementar nº 214/2025**, o fato gerador do **IBS** ocorre no momento do **fornecimento**, caracterizado pela **entrega ou disponibilização do bem material** ao destinatário. Nas operações realizadas sob **cláusula CIF**, em que o transporte é de responsabilidade do próprio fornecedor, o fato gerador somente se consolida com a efetiva entrega ao adquirente. Assim, quando ocorrem **perecimento, perda, roubo ou furto durante o trajeto** — total ou parcialmente, antes da entrega —, o débito de IBS originalmente destacado na nota fiscal de fornecimento perde seu fundamento e deve ser desfeito, conforme autoriza o **art. 12, § 1º, II**, da mesma lei.

Este evento deve ser emitido pelo **fornecedor (emitente da NF-e)** sobre o documento fiscal de fornecimento afetado. O evento informa, de forma proporcional à quantidade de bens atingidos, o **valor do IBS a ser estornado** do débito originalmente destacado e, no grupo próprio do leiaute, o **IBS apropriado como crédito nas aquisições anteriores** dos bens ou insumos que serão objeto de estorno. O uso deste evento é **exclusivo para bens materiais em trânsito na modalidade CIF**, não se aplicando às perdas ocorridas em estoque, hipótese em que se observa procedimento distinto previsto para a nota fiscal de débito do tipo 07 – Perda em Estoque.

#### Passo a passo

1. Pesquise por **Eventos NF-e** no Business Central e abra a página.
2. Crie um novo evento e, no campo **Tipo de Evento**, selecione **112130 - Perecimento/perda/roubo/furto durante o transporte contratado pelo fornecedor**.
3. No campo **Nº Documento Fiscal**, informe o documento fiscal da NF-e de referência. O sistema preencherá automaticamente a **Chave de Acesso**.
4. Na seção **Linhas**, utilize a ação **Carregar Itens do Xml** para importar os itens da NF-e. O sistema carregará automaticamente os dados de cada item.
5. Confira e ajuste, se necessário, os seguintes campos em cada linha:
   - **Número do Item** — posição do item na NF-e original
   - **Valor IBS** — valor do IBS do item
   - **Valor CBS** — valor da CBS do item
   - **Quantidade** — quantidade de itens que sofreram perecimento/perda/roubo/furto
   - **Cód. Unidade de Medida** — unidade de medida do item
   - **Valor IBS Créd. Compra** — valor do crédito de IBS de compra a ser estornado referente ao item perdido
   - **Valor CBS Créd. Compra** — valor do crédito de CBS de compra a ser estornado referente ao item perdido
6. Clique em **Registrar** para transmitir o evento à SEFAZ.

> **Importante:**
>
> - Diferente de outros eventos similares, o 112130 inclui um grupo de **controle de estoque** com a quantidade e unidade de medida dos itens afetados, além dos valores de crédito de compra (IBS e CBS) que devem ser informados.
> - Existe um evento análogo, o **211124**, que é utilizado quando o transporte foi contratado pelo **adquirente** (destinatário). A diferença principal é quem contratou o frete e, portanto, quem registra o evento.

### Evento 112140 -  Fornecimento não realizado com pagamento antecipado

#### Introdução

Nos termos do **art. 10, § 4º, inciso I, da Lei Complementar nº 214/2025**, o recebimento de **pagamento antecipado** relativo a fornecimento futuro enseja a emissão de **nota fiscal de débito do tipo 06 – Pagamento Antecipado**, com o destaque do IBS correspondente ao valor recebido. O débito é registrado no período de apuração do recebimento, antecipando a tributação em momento anterior ao da efetiva entrega do bem ou da prestação do serviço. Ocorre, porém, que nem sempre o fornecimento contratado se concretiza: podem haver **distratos totais ou parciais**, devolução de valores ao cliente e outras situações em que a operação que ensejou a antecipação deixa de existir, tornando indevido o débito anteriormente destacado.

Para essas hipóteses, está previsto este evento, que deve ser emitido pelo **fornecedor** sobre a respectiva **nota fiscal de débito de antecipação** cujo valor tenha sido objeto de distrato, informando o **valor do IBS proporcional ao montante efetivamente devolvido ao cliente**. O evento somente deve ser utilizado quando for possível afirmar que **não haverá mais nenhum fornecimento** decorrente do pagamento que originou a nota de antecipação, hipótese em que o débito anteriormente lançado é desfeito na apuração assistida do emitente, preservando a coerência entre os fatos econômicos e os efeitos tributários registrados.

#### Passo a passo

1. **Abra o Cartão de Evento NF-e** — acesse a página **Evento NF-e**.

2. **Preencha os dados do cabeçalho:**

   - **Tipo de Evento**: selecione **112140 - Fornecimento não realizado com pagamento antecipado**
   - **Nº Documento Fiscal**: informe o documento fiscal relacionado
   - **Cód. Filial**: filial emissora
   - **Participante**: o participante da operação
   - **Chave de Acesso**: a chave de acesso da NF-e autorizada (campo obrigatório)

3. **Preencha as linhas do evento:**
   Este evento **exige o preenchimento de linhas** (diferentemente do evento 112150, que só tem cabeçalho). As linhas contêm os itens da NF-e que não foram fornecidos. Para cada linha, informe:

   - **Número do Item** (obrigatório) — posição do item na NF-e original
   - **Descrição** — descrição do item
   - **Quantidade** — quantidade do item
   - **Cód. Unidade de Medida** — unidade de medida
   - **Valor IBS** — valor do IBS referente ao item
   - **Valor CBS** — valor da CBS referente ao item

   **Dica:** Utilize a ação **Carregar Itens do Xml** disponível nas linhas para importar automaticamente os itens a partir do XML da NF-e. Isso preenche descrição, quantidade, unidade de medida e valores de impostos, evitando digitação manual.

4. **Transmita o evento:**

   - Clique em **Registrar**. O sistema validará os campos obrigatórios (Chave de Acesso e Número do Item em cada linha) e pedirá confirmação antes de transmitir à SEFAZ.
   - Após a transmissão bem-sucedida, o **Status** passará para "Registrado" e o **Nº Protocolo** será gravado.

> **Importante:**
>
> - Caso precise **cancelar** um evento já transmitido, use a ação **Cancelar** (disponível quando o status estiver "Registrado").

### Evento **112150** -  Atualização da Data de Previsão de Entrega

#### Introdução

Nos termos do **art. 10 da Lei Complementar nº 214/2025**, o fato gerador do **IBS** ocorre no momento do **fornecimento**, definido pelo **art. 3º, II, "a"** como a **entrega ou disponibilização do bem material** ao destinatário. Por essa razão, o débito de IBS destacado nas notas fiscais eletrônicas de fornecimento de bens móveis materiais deve ser considerado no **período de apuração correspondente ao mês e ano em que se verificar a entrega**, utilizando-se, em ordem de prioridade, os campos **dPrevEntrega** (Data Prevista de Entrega), **dhSaiEnt** (Data de Saída) ou **dhEmi** (Data de Emissão) da NF-e. Situações como **atrasos logísticos, alterações de rota, indisponibilidade do destinatário** ou simples **omissão do campo dPrevEntrega** na emissão original podem, contudo, tornar a data inicialmente informada incompatível com o momento real de ocorrência do fato gerador.

Para essas hipóteses, está previsto este evento que deve ser emitido pelo contribuinte para informar a **nova data prevista de entrega** vinculada à NF-e original. O efeito do evento na apuração assistida do IBS é a **alteração da ordem cronológica do documento fiscal** e, quando a nova data implicar mudança de mês ou ano, o **deslocamento do débito para o período de competência correspondente**, assegurando que o lançamento ocorra no período em que efetivamente se verifica o fornecimento.

#### Passo a passo

1. **Abra o Cartão de Evento NF-e** — acesse a página **Evento NF-e**.
2. **Preencha os dados do cabeçalho:**
   - **Tipo de Evento**: selecione **112150 - Atualização da Data de Previsão de Entrega**
   - **Nº Documento Fiscal**: informe o documento fiscal relacionado
   - **Cód. Filial**: filial emissora
   - **Participante**: o participante da operação
   - **Chave de Acesso**: a chave de acesso da NF-e autorizada
3. **Preencha o dado específico do evento:**
   - **Data de Previsão de Entrega** — campo obrigatório. Informe a nova data estimada de entrega.
4. **Transmita o evento:**
   - Clique em **Registrar**. O sistema validará os campos obrigatórios (Chave de Acesso e Data de Previsão de Entrega) e pedirá confirmação antes de transmitir à SEFAZ.
   - Após a transmissão bem-sucedida, o **Status** passará para "Registrado" e o **Nº Protocolo** será gravado.

> **Importante**:
>
> - Este evento **não exige linhas** (subformulário de itens) — apenas o preenchimento do cabeçalho e da data de previsão.
> - Caso precise **cancelar** um evento já transmitido, use a ação **Cancelar** disponível quando o status estiver "Registrado".

### Evento 211120 – Destinação de item para consumo pessoal

#### Introdução

Nos termos do **art. 57, § 5º, da Lei Complementar nº 214/2025**, os créditos de **IBS** apropriados — ou passíveis de apropriação — sobre aquisições de bens ou serviços que venham a ser **destinados ao consumo pessoal** do contribuinte, de seus sócios, administradores, empregados ou de seus familiares **não geram direito a crédito** e, quando já apropriados, devem ser **estornados** da apuração. A regra preserva a neutralidade do IBS, que se destina a onerar o consumo final, impedindo que pessoas físicas se beneficiem indiretamente, por meio de créditos empresariais, de aquisições que, na essência, configuram consumo particular.

Este evento deve ser emitido pelo **adquirente enquadrado no regime regular de apuração** sobre o documento fiscal correspondente à aquisição, identificando o **item destinado ao consumo pessoal** e o **valor do IBS a ser estornado**, calculado de forma proporcional à parcela efetivamente consumida. O efeito do evento na apuração assistida do IBS é o **lançamento a débito** do valor informado, regularizando o saldo de créditos do contribuinte conforme determina a legislação.

#### Passo a passo

1. **Abra o Cartão de Evento NF-e** — acesse a página **Evento NF-e**.

2. **Preencha os dados do cabeçalho:**

   - **Tipo de Evento**: selecione **211120 - Destinação de item para consumo pessoal**
   - **Nº Documento Fiscal**: informe o documento fiscal relacionado (obrigatório — o sistema utiliza este campo para determinar se a NF-e é de importação ou não)
   - **Cód. Filial**: filial do adquirente
   - **Participante**: o participante da operação
   - **Chave de Acesso**: a chave de acesso da NF-e autorizada (obrigatório)

3. **Preencha as linhas do evento:**
   Este evento **exige o preenchimento de linhas** com os itens destinados a consumo pessoal. Para cada linha, informe:

   - **Número do Item** (obrigatório) — posição do item na NF-e
   - **Descrição** — descrição do item
   - **Quantidade** — quantidade destinada a consumo pessoal
   - **Cód. Unidade de Medida** — unidade de medida
   - **Valor IBS** — valor do IBS referente ao item
   - **Valor CBS** — valor da CBS referente ao item

   **Campos exclusivos deste evento** (visíveis apenas para o 211120):

   - **Nº Doc. Fiscal Ref.** — documento fiscal de referência. Ao selecionar, a chave de acesso de referência é preenchida automaticamente (restringe a documentos com chave NF-e autorizada)
   - **Chave Acesso Ref.** (obrigatório) — chave de acesso da NF-e referenciada
   - **Número Item Ref.** (obrigatório) — posição do item na NF-e referenciada. Possui lookup que permite selecionar diretamente a partir dos itens do XML da NF-e de origem

   **Dica:** Utilize a ação **Carregar Itens do Xml** disponível nas linhas para importar automaticamente os itens a partir do XML da NF-e, preenchendo descrição, quantidade, unidade de medida e valores de impostos.

4. **Transmita o evento:**

   - Clique em **Registrar**. O sistema validará os campos obrigatórios — no cabeçalho: Chave de Acesso; nas linhas: Número do Item, Chave Acesso Ref. e Número Item Ref.
   - Após confirmação, o evento será transmitido à SEFAZ. Se bem-sucedido, o **Status** passará para "Registrado" e o **Nº Protocolo** será gravado.

> **Importante**:
>
> - Este é o **único evento** que exibe os campos de documento referenciado (Nº Doc. Fiscal Ref., Chave Acesso Ref. e Número Item Ref.) nas linhas — eles são específicos para vincular os itens à NF-e de origem.
> - Para **cancelar** um evento já transmitido, use a ação **Cancelar** (disponível quando o status estiver "Registrado").

### Evento 211124 – Perecimento, perda, roubo ou furto durante o transporte contratado pelo adquirente

#### Introdução

Nos termos do **art. 10 da Lei Complementar nº 214/2025**, o fato gerador do **IBS** ocorre no momento do **fornecimento**, caracterizado pela **entrega ou disponibilização do bem material** ao destinatário. Nas operações realizadas sob **cláusula FOB**, em que o transporte é de responsabilidade do **adquirente ou destinatário**, considera-se que o bem foi efetivamente entregue pelo fornecedor no momento da retirada, consolidando-se ali o fato gerador do imposto e o consequente direito à apropriação do crédito pelo adquirente. Quando, durante esse transporte, ocorrem **perecimento, perda, roubo ou furto** — total ou parcialmente —, o **art. 12, § 1º, II**, da mesma lei impõe ao adquirente o **estorno proporcional do crédito** correspondente aos bens afetados, uma vez que estes não serão utilizados em operações tributadas subsequentes.

Este evento deve ser emitido pelo **adquirente** sobre a respectiva NF-e de aquisição, informando o **valor do IBS a ser estornado** proporcionalmente à quantidade objeto da ocorrência. O uso deste evento é **exclusivo para bens materiais em trânsito na modalidade FOB** e, por refletir um fato gerador já consolidado na esfera do fornecedor, **não produz efeitos na apuração assistida deste último** — atinge apenas o crédito apropriado pelo adquirente. Não se aplica, ainda, às perdas ocorridas em estoque, hipótese em que se observa procedimento distinto previsto para a nota fiscal de débito do tipo 07 – Perda em Estoque.

#### Passo a passo

1. **Abra o Cartão de Evento NF-e** — acesse a página **Evento NF-e**.

2. **Preencha os dados do cabeçalho:**

   - **Tipo de Evento**: selecione **211124 - Perecimento/perda/roubo/furto durante o transporte contratado pelo adquirente**
   - **Nº Documento Fiscal**: informe o documento fiscal relacionado
   - **Cód. Filial**: filial do adquirente
   - **Participante**: o participante da operação
   - **Chave de Acesso**: a chave de acesso da NF-e autorizada (obrigatório)

3. **Preencha as linhas do evento:**
   Este evento **exige o preenchimento de linhas** com os itens afetados pelo perecimento, perda, roubo ou furto. Para cada linha, informe:

   - **Número do Item** (obrigatório) — posição do item na NF-e original
   - **Descrição** — descrição do item
   - **Quantidade** — quantidade afetada
   - **Cód. Unidade de Medida** — unidade de medida
   - **Valor IBS** — valor do IBS referente ao item
   - **Valor CBS** — valor da CBS referente ao item

   **Dica:** Utilize a ação **Carregar Itens do Xml** disponível nas linhas para importar automaticamente os itens a partir do XML da NF-e, preenchendo descrição, quantidade, unidade de medida e valores de impostos.

4. **Transmita o evento:**

   - Clique em **Registrar**. O sistema validará os campos obrigatórios (Chave de Acesso no cabeçalho e Número do Item em cada linha) e pedirá confirmação antes de transmitir à SEFAZ.
   - Após a transmissão bem-sucedida, o **Status** passará para "Registrado" e o **Nº Protocolo** será gravado.

> #### Diferença em relação ao evento 112130
>
> Existe um evento semelhante — o **112130 - Perecimento/perda/roubo/furto durante o transporte contratado pelo fornecedor** — que se aplica quando o transporte é contratado pelo fornecedor (modalidade FOB). A diferença está em quem contratou o transporte:
>
> - **211124**: transporte contratado pelo **adquirente** (evento do adquirente)
> - **112130**: transporte contratado pelo **fornecedor** (evento do emitente)

> **Importante:**
>
> - Para **cancelar** um evento já transmitido, use a ação **Cancelar** (disponível quando o status estiver "Registrado").

### Evento 211128 – Aceite de débito na apuração por emissão de nota de crédito

#### Introdução

Em determinadas hipóteses, a emissão de **notas fiscais de crédito** pelo fornecedor — como nos casos de **redução de valores por destaque a maior** ou **entrega parcial da quantidade consignada no documento original** — implica, simultaneamente, a **redução do débito de IBS** na apuração do emitente e a necessidade de **contrapartida de igual valor, a débito, na apuração do adquirente**, sempre que este já tiver apropriado o crédito decorrente da operação original. Sem essa contrapartida, o ajuste promovido pelo fornecedor geraria uma assimetria entre débitos e créditos vinculados a uma mesma operação, comprometendo a integridade do modelo de apuração assistida do IBS.

Para essas hipóteses, está previsto este evento, que deve ser emitido pelo **adquirente** sobre a respectiva **nota fiscal de crédito** emitida pelo fornecedor, formalizando sua **anuência quanto ao lançamento do débito correspondente** em sua própria apuração. O evento é **condição necessária para que o fornecedor possa efetivar a apropriação do crédito** pleiteado na nota fiscal de crédito, assegurando o equilíbrio entre as apurações das partes envolvidas. **Não é exigido**, contudo, nas situações em que o desfazimento da operação ocorre **antes do efetivo recebimento do bem pelo destinatário**, hipótese em que o crédito do adquirente sequer chegou a se consolidar.

#### Passo a passo

1. **Abra o Cartão de Evento NF-e** — acesse a página **Evento NF-e**.
2. **Preencha os dados do cabeçalho:**
   - **Tipo de Evento**: selecione **211128 - Aceite de débito na apuração por emissão de nota de crédito**
   - **Nº Documento Fiscal**: informe o documento fiscal relacionado
   - **Cód. Filial**: filial do adquirente
   - **Participante**: o participante da operação
   - **Chave de Acesso**: a chave de acesso da NF-e autorizada (obrigatório)
3. **Preencha o dado específico do evento:**
   - **Indicador de Aceitação** (obrigatório) — informe **Sim** para aceitar o débito ou **Não** para rejeitá-lo. Este campo é exclusivo deste evento e determina a posição do adquirente frente ao débito lançado na apuração.
4. **Transmita o evento:**
   - Clique em **Registrar**. O sistema validará os campos obrigatórios (Chave de Acesso e Indicador de Aceitação) e pedirá confirmação antes de transmitir à SEFAZ.
   - Após a transmissão bem-sucedida, o **Status** passará para "Registrado" e o **Nº Protocolo** será gravado.

> **Importante**
>
> - Este evento **não exige linhas** — assim como o evento 112150 (Atualização da Data de Previsão de Entrega), o preenchimento é feito exclusivamente no cabeçalho.
> - Para **cancelar** um evento já transmitido, use a ação **Cancelar** (disponível quando o status estiver "Registrado").

### Evento 211130 – Imobilização de Item

#### Introdução

Nos termos dos **arts. 39 e 40 da Lei Complementar nº 214/2025**, os créditos de **IBS** apropriados pelos contribuintes podem ser objeto de **pedido de ressarcimento**, observados prazos diferenciados de análise conforme a natureza da aquisição. O **art. 40, inciso I**, estabelece **prazo específico** para a análise dos pedidos relativos a créditos decorrentes de **aquisições de bens incorporados ao ativo imobilizado**, reconhecendo a particularidade dessas operações, em que o crédito está vinculado a um bem destinado ao uso continuado na atividade econômica do contribuinte, e não à revenda ou ao consumo no ciclo produtivo imediato.

Nestes casos, o evento deve ser emitido pelo **contribuinte adquirente** sobre a respectiva NF-e quando o bem adquirido for **integrado ao ativo imobilizado**. O efeito do evento é **sinalizar ao sistema do CG-IBS** que o crédito correspondente, em eventual pedido de ressarcimento futuro, deverá observar o prazo de análise previsto no art. 40, I, da referida lei. **Na ausência do evento**, o prazo de análise será determinado exclusivamente com base nas demais regras dos arts. 39 e 40, sem o benefício do tratamento específico aplicável às imobilizações.

#### Passo a passo

1. **Abra o Cartão de Evento NF-e** — acesse a página **Evento NF-e**.

2. **Preencha os dados do cabeçalho:**

   - **Tipo de Evento**: selecione **211130 - Imobilização de Item**
   - **Nº Documento Fiscal**: informe o documento fiscal relacionado
   - **Cód. Filial**: filial do adquirente
   - **Participante**: o participante da operação
   - **Chave de Acesso**: a chave de acesso da NF-e autorizada (obrigatório)

3. **Preencha as linhas do evento:**
   Este evento **exige o preenchimento de linhas** com os itens destinados à imobilização. Para cada linha, informe:

   - **Número do Item** (obrigatório) — posição do item na NF-e original
   - **Descrição** — descrição do item
   - **Quantidade** — quantidade a ser imobilizada
   - **Cód. Unidade de Medida** — unidade de medida
   - **Valor IBS** — valor do IBS referente ao item
   - **Valor CBS** — valor da CBS referente ao item

   **Dica:** Utilize a ação **Carregar Itens do Xml** disponível nas linhas para importar automaticamente os itens a partir do XML da NF-e, preenchendo descrição, quantidade, unidade de medida e valores de impostos.

4. **Transmita o evento:**

   - Clique em **Registrar**. O sistema validará os campos obrigatórios (Chave de Acesso no cabeçalho e Número do Item em cada linha) e pedirá confirmação antes de transmitir à SEFAZ.
   - Após a transmissão bem-sucedida, o **Status** passará para "Registrado" e o **Nº Protocolo** será gravado.

> **Importante**
>
> - Para **cancelar** um evento já transmitido, use a ação **Cancelar** (disponível quando o status estiver "Registrado").

### Evento 211140 – Apropriação de Crédito de Combustível

#### Introdução

Nos termos do **art. 180 da Lei Complementar nº 214/2025**, é **vedada a apropriação de créditos de IBS** nas aquisições de **combustíveis sujeitos à incidência única (monofásica)** do IBS e da CBS, quando destinados à **distribuição, comercialização ou revenda**. Em razão dessa regra, o tratamento padrão da apuração assistida, nas operações de fornecimento de combustíveis para contribuintes do regime regular **pertencentes à cadeia de produção e distribuição desse tipo de bem** — como refinarias, distribuidoras e postos —, é **não permitir a apropriação automática do crédito**. O **§ 1º** do mesmo artigo, contudo, ressalva que a **parcela do combustível efetivamente consumida pelo próprio contribuinte do setor em sua operação** — por exemplo, no abastecimento de sua frota — gera direito ao crédito correspondente.

Este evento deve ser emitido pelo **contribuinte do setor de combustíveis** sobre a respectiva NF-e de aquisição, indicando a **parcela do produto destinada ao consumo próprio** e o **valor do IBS a ser apropriado como crédito**. **Para contribuintes que não pertencem ao setor de combustíveis**, a apropriação dos créditos de aquisição ocorre de forma **automática** na apuração assistida, **não sendo necessária a emissão deste evento**.

#### Passo a passo

1. **Abra o Cartão de Evento NF-e** — acesse a página **Evento NF-e**.

2. **Preencha os dados do cabeçalho:**

   - **Tipo de Evento**: selecione **211140 - Solicitação de Apropriação de Crédito de Combustível**
   - **Nº Documento Fiscal**: informe o documento fiscal relacionado
   - **Cód. Filial**: filial do adquirente
   - **Participante**: o participante da operação
   - **Chave de Acesso**: a chave de acesso da NF-e autorizada (obrigatório)

3. **Preencha as linhas do evento:**
   Este evento **exige o preenchimento de linhas** com os itens de combustível para os quais se solicita a apropriação de crédito. Para cada linha, informe:

   - **Número do Item** (obrigatório) — posição do item na NF-e original
   - **Descrição** — descrição do item
   - **Quantidade** — quantidade de combustível
   - **Cód. Unidade de Medida** — unidade de medida do combustível
   - **Valor IBS** — valor do IBS referente ao item
   - **Valor CBS** — valor da CBS referente ao item

   **Dica:** Utilize a ação **Carregar Itens do Xml** disponível nas linhas para importar automaticamente os itens a partir do XML da NF-e, preenchendo descrição, quantidade, unidade de medida e valores de impostos.

4. **Transmita o evento:**

   - Clique em **Registrar**. O sistema validará os campos obrigatórios (Chave de Acesso no cabeçalho e Número do Item em cada linha) e pedirá confirmação antes de transmitir à SEFAZ.
   - Após a transmissão bem-sucedida, o **Status** passará para "Registrado" e o **Nº Protocolo** será gravado.

> #### Diferença em relação ao evento 211150
>
> Existe um evento semelhante — o **211150 - Solicitação de Apropriação de Crédito para bens e serviços que dependem de atividade do adquirente**. A diferença é o escopo:
>
> - **211140**: específico para **combustíveis**
> - **211150**: para **bens e serviços em geral** que dependem de atividade do adquirente

> **Importante**
>
> - O XML gerado para este evento inclui, para cada linha, os dados de controle de estoque de combustível (quantidade e unidade de medida) além dos valores de IBS e CBS.
> - Para **cancelar** um evento já transmitido, use a ação **Cancelar** (disponível quando o status estiver "Registrado").

### Evento 211150 – Apropriação de Crédito para bens e serviços que dependem de atividade do adquirente

#### Introdução

Nos termos do **art. 57, inciso I, da Lei Complementar nº 214/2025**, haverá uma **lista taxativa de bens e serviços** cujo tratamento padrão na apuração assistida do IBS será a **não apropriação automática de crédito** pelo adquirente, ainda que este se encontre no regime regular de apuração. Trata-se de itens cuja natureza — em regra associada a usos de **caráter pessoal, recreativo ou não diretamente vinculado à atividade econômica** — gera presunção de que não integram o ciclo produtivo do adquirente. O **§ 3º** do mesmo artigo, contudo, prevê **hipóteses de exceção** (incisos I a III) nas quais o direito ao crédito é preservado, especialmente quando os bens ou serviços são utilizados em **operações subsequentes tributadas** pelo próprio adquirente, configurando insumo legítimo de sua atividade.

Para essas hipóteses, está previsto este evento que deve ser emitido pelo **adquirente** sobre a respectiva NF-e quando, por **falha técnica no processo de identificação automática do direito ao crédito**, o sistema do CG-IBS não tiver reconhecido o enquadramento da operação em uma das exceções previstas no art. 57, § 3º. O efeito do evento é **autorizar a apropriação do crédito** correspondente aos itens que efetivamente se enquadrem nas exceções legais, complementando o processamento automático e assegurando a fruição do direito creditório nas operações legítimas.

#### Passo a passo

1. **Abra o Cartão de Evento NF-e** — acesse a página **Evento NF-e**.

2. **Preencha os dados do cabeçalho:**

   - **Tipo de Evento**: selecione **211150 - Solicitação de Apropriação de Crédito para bens e serviços que dependem de atividade do adquirente**
   - **Nº Documento Fiscal**: informe o documento fiscal relacionado
   - **Cód. Filial**: filial do adquirente
   - **Participante**: o participante da operação
   - **Chave de Acesso**: a chave de acesso da NF-e autorizada (obrigatório)

3. **Preencha as linhas do evento:**
   Este evento **exige o preenchimento de linhas** com os itens para os quais se solicita a apropriação de crédito. Para cada linha, informe:

   - **Número do Item** (obrigatório) — posição do item na NF-e original
   - **Descrição** — descrição do item
   - **Quantidade** — quantidade do item
   - **Cód. Unidade de Medida** — unidade de medida
   - **Valor IBS** — valor do IBS referente ao item
   - **Valor CBS** — valor da CBS referente ao item

   O XML gerado para este evento transmite, para cada linha, o Número do Item e os valores de crédito IBS e CBS.

   **Dica:** Utilize a ação **Carregar Itens do Xml** disponível nas linhas para importar automaticamente os itens a partir do XML da NF-e, preenchendo descrição, quantidade, unidade de medida e valores de impostos.

4. **Transmita o evento:**

   - Clique em **Registrar**. O sistema validará os campos obrigatórios (Chave de Acesso no cabeçalho e Número do Item em cada linha) e pedirá confirmação antes de transmitir à SEFAZ.
   - Após a transmissão bem-sucedida, o **Status** passará para "Registrado" e o **Nº Protocolo** será gravado.

> #### Diferença em relação ao evento 211140
>
> - **211150**: apropriação de crédito para **bens e serviços em geral** que dependem de atividade do adquirente — o XML transmite os valores de crédito IBS/CBS por item
> - **211140**: específico para **combustíveis** — o XML transmite adicionalmente dados de controle de estoque (quantidade e unidade de medida do combustível)

> **Importante:**
>
> - Para **cancelar** um evento já transmitido, use a ação **Cancelar** (disponível quando o status estiver "Registrado").

## Eventos ainda não implementados

Alguns eventos previstos na Nota Técnica ainda não estão disponíveis na localização:

- Informação de pagamento integral para crédito presumido
- Solicitação de apropriação de crédito presumido
- Manifestação sobre transferência de crédito IBS na sucessão
- Manifestação sobre transferência de crédito CBS na sucessão
- Manifestação do Fisco sobre transferência IBS na sucessão
- Manifestação do Fisco sobre transferência CBS na sucessão
- Cancelamento de evento (genérico)

> **Importante:**
>
> Esses serão implementados conforme evolução da legislação e publicação de novas Notas Técnicas.

---
