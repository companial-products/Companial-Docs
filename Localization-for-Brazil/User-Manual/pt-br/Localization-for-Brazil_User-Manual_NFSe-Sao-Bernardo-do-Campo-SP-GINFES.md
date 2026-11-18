<!-- PR: #312, arquibr/fnx#35, #323 -->
# NFS-e: Emissão em São Bernardo do Campo - SP (GINFES)

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **São Bernardo do Campo - SP** emitem NFS-e pelo Business Central por meio da plataforma GINFES, no layout de 2026, que já traz a Reforma Tributária. Na lista de cidades de NFS-e, a integração aparece como **São Bernardo do Campo - SP** (código `SP-SaoBernardoDoCampo`) e cobre a transmissão de lotes de RPS, a consulta do processamento, o reenvio após rejeição e o cancelamento.

A autenticação é feita só pelo **certificado digital** da filial. A autorização não é imediata: a prefeitura recebe o lote, e o resultado (NFS-e autorizada ou mensagens de erro) só aparece na **consulta** do lote. Toda nota leva o grupo de IBS/CBS e os percentuais aproximados de tributos. O DANFSe sai pelo Layout Padrão quando a GINFES devolve a chave de acesso nacional; sem ela, a nota é impressa no portal da GINFES.

---

## 1. Contexto

### O que muda para a filial de São Bernardo do Campo

São Bernardo do Campo usa a plataforma GINFES, a mesma de Fortaleza, mas com outro layout. A integração de Fortaleza não muda, e as duas não são intercambiáveis: escolha sempre a opção da cidade da filial.

| Tema | Como funciona em São Bernardo do Campo - SP |
| --- | --- |
| **Layout** | Lote de RPS no padrão GINFES de 2026, com o grupo de IBS/CBS e os percentuais aproximados de tributos, assinado com o certificado digital da filial |
| **Autenticação** | Certificado digital da filial. Não há usuário, senha nem token |
| **Envio** | Um lote com um ou mais documentos. A prefeitura confirma o recebimento, mas as notas ainda não estão autorizadas |
| **Autorização** | Conhecida na consulta do lote, que grava o número da NFS-e (15 dígitos: ano + sequencial), a data de emissão, o código de verificação e, quando a GINFES a devolve, a chave de acesso nacional |
| **Rejeição** | Na transmissão, a recusa aparece na tela. Na consulta, as mensagens da prefeitura aparecem na tela, no fim da consulta, e ficam em **Lista Erros** do lote |
| **Cancelamento** | Pedido de cancelamento enviado à prefeitura pelo Business Central, depois do cancelamento do documento fiscal |
| **DANFSe** | Layout Padrão, quando a nota volta com a chave de acesso nacional. Sem ela, consulte e imprima a nota no portal da GINFES (veja a [seção 4](#4-danfse)) |
| **Ambientes** | Homologação e produção |

A integração atende filiais do regime normal (Lucro Real ou Presumido). Filiais do Simples Nacional e MEI de São Bernardo do Campo emitem pelo Emissor Público Nacional (veja [NFS-e: Emissão pelo Emissor Público Nacional (SEFIN)](Localization-for-Brazil_User-Manual_NFSe-Emissor-Publico-Nacional-SEFIN.md)).

### Tributos enviados

- **ISS**: alíquota, valor do ISS e indicação de ISS retido.
  - A base de cálculo é o valor dos serviços menos as deduções e o desconto incondicionado, e o valor do ISS é recalculado no envio como alíquota × base.
  - Com ISS retido, o valor do ISS retido é o próprio valor do ISS, e ele é descontado do valor líquido.
- **Retenções federais**: PIS, COFINS, INSS, IRRF e CSLL retidos, cada um no seu campo, além de outras retenções. O valor líquido da nota é o valor dos serviços menos todas as retenções e o desconto incondicionado.
- **PIS/COFINS da operação**: quando o documento tem CST de PIS/COFINS, a nota leva um grupo próprio com o CST, a base, as alíquotas, os valores e a indicação de retido ou não retido.
  - Com retenção, o grupo leva a base, a alíquota e o valor retidos; sem retenção, os da apuração própria.
  - O layout aceita os CST de `01` a `09`; os demais vão como `00`.
- **Natureza da operação**: o **Cód. Tributação NFS** do documento, de `1` a `6` (veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)).
- **Simples Nacional, incentivo cultural e regime especial**: vão os campos **Opção pelo Simples**, **Incentivo Cultural** e **Regime Especial de Tributação** da **Config. NF-e**. O regime só vai quando é diferente de `00 - Nenhum`.
- **Serviço**: item da lista de serviços, código de tributação do município, código CNAE (quando preenchido), código NBS, discriminação e município da prestação.
- **IBS/CBS (Reforma Tributária)**: vai em **toda** nota, com:
  - a finalidade da emissão (NFS-e regular), o uso ou consumo pessoal e o destinatário, que vão sempre com o valor `0`;
  - o código indicador de operação do código de serviço;
  - o CST e a classificação tributária da **primeira linha** do documento;
  - o município de incidência, que é o município da filial;
  - o percentual de redução da CBS.

  Tipo de operação, imóvel, evento e notas referenciadas não são enviados. A configuração desses cadastros está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md). Os totais de IBS/CBS que a GINFES calcula voltam na nota autorizada, mas não são gravados no Business Central: o documento fiscal mantém os valores calculados pelo Business Central.
- **Tributos aproximados (IBPT)**: os percentuais federal, estadual e municipal da primeira linha do documento, em toda nota (veja [Tributos aproximados (IBPT)](#tributos-aproximados-ibpt)).
- **Tomador**: CNPJ ou CPF, inscrição municipal (quando houver), razão social, endereço e e-mail. O tomador pessoa física vai com o CPF.
- **Tomador do exterior**: vai com o endereço internacional, que tem o país (pelo **Cód. BACEN** do país), o endereço completo (endereço, número, complemento e bairro), o código postal, a cidade e o estado/província (veja [Cliente do exterior](#cliente-do-exterior)).

---

## 2. Configuração

### Certificado digital

O certificado digital é a única credencial de São Bernardo do Campo: ele autentica a conexão com a GINFES e assina o lote, a consulta e o pedido de cancelamento.

1. Use o certificado digital A1 (arquivo `.pfx`) da empresa. Antes de configurar a filial, confirme com a prefeitura de São Bernardo do Campo que o contribuinte está habilitado a emitir NFS-e por web service na GINFES.
2. Cadastre o certificado em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**).
3. Selecione-o no campo **ID Certificado** da aba **NFS-e** (veja a tabela abaixo).

O Business Central avisa quando o certificado está perto de vencer. Quando ele já venceu, a transmissão, a consulta e o cancelamento são **bloqueados** com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação

A GINFES tem um ambiente de homologação. Com **Homologação** ligado na **Config. NF-e** da filial, a transmissão, a consulta e o cancelamento passam a ser enviados a esse ambiente. As notas emitidas em homologação não têm validade jurídica. Antes de testar, confirme com a prefeitura que o contribuinte tem acesso ao ambiente de homologação. Para emitir em produção, desligue **Homologação**.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para São Bernardo do Campo - SP |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **São Bernardo do Campo - SP** na lista (código `SP-SaoBernardoDoCampo`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (`Ginfes`). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial |
| **Configuração Envio** | **Homologação** | Ligado para emitir em homologação; desligado para produção |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. A GINFES não informa o último número de RPS usado; com o campo desligado, toda transmissão falha |
| **Configuração Envio** | **Nome Usuário WS** e **Senha WS** | Não usados por São Bernardo do Campo |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão |
| **Configuração Envio** | **ID Relatório p/ TXT** | Em branco. É usado só por filial que não emite por web service |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | `0` (sem limite) ou o número máximo de documentos que deseja em cada lote |
| **Lote** | **Usar Data/Hora como Nº Lote** e **Nº Série Lote** | O número do lote vai à prefeitura e precisa ter **só dígitos**. Ligue **Usar Data/Hora como Nº Lote** ou informe em **Nº Série Lote** uma série de numeração só com dígitos |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** |
| **Layout Padrão** | **Utilizar Layout Padrão** | Conforme a chave de acesso nacional volte ou não na nota autorizada (veja a [seção 4](#4-danfse)) |
| — | **Nº Série RPS/Temporário** | **Obrigatório para a consulta**: série só com dígitos, com o **Último Nº Usado** preenchido e o mesmo comprimento da numeração fiscal (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente. Em branco, ele precisa ser informado em cada venda |
| — | **NFS-e suporta IBPT** | Ligado automaticamente ao escolher São Bernardo do Campo. Mantenha ligado |
| — | **Texto Legal Valor Aprox. Impostos Automático** | **Ligado**, para enviar os percentuais aproximados de tributos (veja [Tributos aproximados (IBPT)](#tributos-aproximados-ibpt)) |
| — | **Opção pelo Simples** | Desligado: a integração atende filiais do regime normal |
| — | **Incentivo Cultural** | Ligado se a filial é incentivadora cultural. Vai no RPS |
| — | **Regime Especial de Tributação** | O regime especial da filial, ou `00 - Nenhum`. Vai no RPS quando é diferente de `00 - Nenhum` |

### Numeração e série do RPS

- **Número do RPS**: é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão.
  - A numeração vem da série do tipo de documento fiscal: o **Nº Série Fiscal** da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, o **Nº Série Nota Fiscal** do próprio tipo de documento fiscal.
  - A GINFES aceita **só dígitos** no número do RPS, com até 15 dígitos. Use, por exemplo, `000031` e não `NFS000031`.
- **Série**: a **Série** do documento vai no RPS como está cadastrada na série por filial ou no tipo de documento fiscal, com até 5 caracteres.
- **Nº Série RPS/Temporário**: é usada pela ação **Criar Novo Nº** e é **exigida pela consulta**. Para a consulta funcionar, a série precisa:
  - estar preenchida na **Config. NF-e**;
  - ter o **Último Nº Usado** preenchido na linha da série;
  - gerar números com **o mesmo comprimento** da numeração fiscal. Por exemplo, se as notas são numeradas `000123`, a série RPS também precisa ter 6 dígitos, como `000001`.

  A consulta usa esse comprimento para localizar o documento a partir do número de RPS devolvido pela prefeitura. Com comprimento diferente, o lote fica **Sucesso**, mas o documento não recebe o número da NFS-e.

### Filial e cliente

Estes cadastros são conferidos durante a transmissão. A transmissão para na primeira pendência, com a mensagem de campo obrigatório do Business Central, que indica o campo e o cadastro a corrigir:

- **Informação Filial**: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos. A inscrição municipal vai à prefeitura só com os dígitos.
- **CEPs**: o CEP da filial e o do cliente cadastrados, com **Cód. Municipio IBGE**.
- **Cliente**: categoria, CEP, cidade, bairro, endereço, número, UF e país preenchidos (exceto pessoa física e cliente não identificado). A inscrição municipal do cliente vai no RPS quando o cliente é do mesmo município da filial, ou quando ele tem **Outras Inscrições** do tipo C.C.M. para o município da filial.

### Cliente do exterior

O tomador do exterior é o cliente com **Categoria** `3.- Estrangeiro`. A GINFES exige os cinco dados do endereço internacional, e o lote é recusado pela prefeitura se algum faltar:

| Dado do endereço internacional | De onde vem |
| --- | --- |
| País | **Cód. BACEN** do país do cliente, em **Países** |
| Endereço completo | Endereço, número, complemento e bairro da ficha do cliente, até 255 caracteres |
| Código postal | `99999999` |
| Cidade | `EXTERIOR` |
| Estado/província | **Cód. Unidade Federal** da ficha do cliente |

Na ficha do cliente do exterior, preencha o **Cód. Unidade Federal** com a sigla do estado ou da província do cliente: sem ele, o endereço internacional vai incompleto. Confira também o **Cód. BACEN** do país em **Países**.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Item Lista Serviço** | O item da lista de serviços (LC 116) do serviço. Vai como está cadastrado |
| **Cód. Tributação** | O código de tributação do município de São Bernardo do Campo para o serviço. Vai como está cadastrado |
| **Cód. NBS** | Código NBS do serviço. **Obrigatório** no layout de São Bernardo do Campo. Vai sem pontos |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS (6 dígitos). **Obrigatório**: o grupo de IBS/CBS vai em toda nota |
| **Código IBPT** | O código do serviço na tabela IBPT, de onde vêm os percentuais aproximados de tributos |
| **Código CNAE** | Opcional. Vai só com os dígitos, quando preenchido |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos **Cód. Tributação** e **Item Lista Serviço** desse registro substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item. Sem nenhuma linha assim, a transmissão para com "Nenhuma linha da nota fiscal possui Código de Serviço associado.".

### Classificação tributária de IBS/CBS

O CST e a classificação tributária de IBS/CBS vêm do **Cód. Class. Trib. IBS/CBS** da **primeira linha** do documento fiscal, preenchido pela matriz fiscal ou pela exceção ao registrar a venda. Como o grupo de IBS/CBS vai em toda nota, confira que a primeira linha do documento tem a classificação: sem ela, a prefeitura recusa o lote. A configuração está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md).

### Códigos Tributação NFS-e

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. Em São Bernardo do Campo, o campo **Cód. Tributação** desse código é a **natureza da operação** do RPS, com um destes seis valores:

| Cód. Tributação | Natureza da operação |
| --- | --- |
| `1` | Tributação no município |
| `2` | Tributação fora do município |
| `3` | Isenção |
| `4` | Imune |
| `5` | Exigibilidade suspensa por decisão judicial |
| `6` | Exigibilidade suspensa por procedimento administrativo |

Qualquer outro valor é recusado antes do envio: o lote não é criado, e a mensagem "RPS &lt;nº&gt;: a natureza da operação "&lt;valor&gt;" não é aceita pelo GINFES, que admite os valores de 1 a 6." aparece na tela. A natureza nunca vai zerada.

A lista de valores nacionais e a coluna **Significado nacional** valem só para filiais do Emissor Público Nacional e não se aplicam a São Bernardo do Campo.

### Tributos aproximados (IBPT)

O layout de São Bernardo do Campo leva em toda nota os percentuais aproximados de tributos federais, estaduais e municipais (Lei da Transparência). Os percentuais vêm da **primeira linha** do documento fiscal, calculados pela tabela IBPT ao registrar a venda. Para que eles sejam enviados:

1. Mantenha **NFS-e suporta IBPT** e **Texto Legal Valor Aprox. Impostos Automático** ligados na **Config. NF-e** da filial.
2. Preencha o **Código IBPT** de cada código de serviço.
3. Mantenha a tabela **Tributos IBPT** atualizada (veja [IBPT: Tabela de Tributos por UF](Localization-for-Brazil_User-Manual_IBPT-Tabela-Por-UF.md)).

Com **NFS-e suporta IBPT** ligado, o texto legal dos tributos aproximados não entra na discriminação do serviço: os percentuais vão nos campos próprios do layout.

Com qualquer um dos dois campos desligado, ou sem tributos aproximados calculados no documento, os percentuais vão zerados (`0.00`).

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Docs. Pendentes**: transmite os documentos pendentes da filial num único lote, com até **Nº Máx. de Docs. por Lote WS** documentos. Se houver mais pendentes que o máximo, a página avisa que só os primeiros serão enviados.
- **Transmitir Doc. Selecionado**: transmite só o documento selecionado.

Antes do envio, a transmissão para nestas situações:

| Situação | O que corrigir |
| --- | --- |
| Filial sem nome, CNPJ, **Inscrição Municipal**, cidade ou CEP | **Informação Filial** |
| CEP da filial ou do cliente sem **Cód. Municipio IBGE** | **CEPs** |
| Filial sem **ID Certificado**, ou certificado vencido | **ID Certificado** na **Config. NF-e** |
| Cliente com endereço incompleto | Ficha do cliente |
| "Nenhuma linha da nota fiscal possui Código de Serviço associado." | Ficha do item: tipo Serviço e **Cód. Serviço** |
| Documento sem **Município Prestação Serviço** ou sem **Cód. Tributação NFS** | **Documento Fiscal Registrado**, grupo **NFS**, ação **Editar Campos** (veja [Editar Campos do Documento Fiscal Registrado](Localization-for-Brazil_User-Manual_Editar-Campos-Documento-Fiscal-Registrado.md)) |
| "RPS &lt;nº&gt;: a natureza da operação "&lt;valor&gt;" não é aceita pelo GINFES, que admite os valores de 1 a 6." | **Cód. Tributação** do código de **Códigos Tributação NFS-e** do documento (veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)) |
| "Lote &lt;nº&gt; deverá ser consultado antes do envio." | O documento está num lote ainda **Enviado**: consulte o lote (veja [Consultar o processamento](#consultar-o-processamento)) |

O Business Central não confere antes do envio a classificação de IBS/CBS, o código indicador de operação, o código NBS, os percentuais aproximados nem o endereço do cliente do exterior. Quando falta algum deles, quem recusa o lote é a prefeitura, com as mensagens dela.

Quando a prefeitura aceita o lote, ele fica com **Status** **Enviado**, e os documentos com **Status NFS-e** **Enviado**. As notas **ainda não estão autorizadas**: o número da NFS-e só chega na consulta.

Quando a prefeitura recusa o lote, as mensagens dela aparecem na tela, uma por linha, com o código da prefeitura antes do texto e, quando a prefeitura a informa, a correção sugerida numa linha própria. Nada é gravado: o lote não é criado e os documentos continuam pendentes.

### Consultar o processamento

Consulte o lote alguns minutos depois da transmissão, quando a prefeitura já o processou. Use **Consultar** em uma destas páginas:

- **NFS-e - Envio de Lote**: selecione os documentos;
- **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`): selecione o lote.

A consulta exige a **Nº Série RPS/Temporário** configurada (veja [Numeração e série do RPS](#numeração-e-série-do-rps)). Ela primeiro pergunta à GINFES a situação do lote e só busca as notas quando o lote já foi processado:

| Resultado da consulta | O que acontece no Business Central |
| --- | --- |
| **Lote ainda não processado** | O lote continua com **Status** **Enviado**, com o aviso "Lote ainda não processado pela prefeitura (situação &lt;n&gt; - &lt;descrição&gt;). Consulte novamente em alguns minutos." em **Lista Avisos**. Consulte de novo mais tarde |
| **Notas autorizadas** | O lote fica com **Status** **Sucesso**. Cada documento recebe o **Nº NFS** (15 dígitos), a **Data NFS**, o **Cód. Verificação NFS** e, quando a GINFES a devolve, a **Chave de Acesso NFS-e**. O **Nº Nota Fiscal** passa a ser o número da NFS-e |
| **Mensagens da prefeitura** | O lote fica com **Status** **Erro**. As mensagens ficam em **Lista Erros**, com o código e o texto de cada mensagem da prefeitura (até 250 caracteres por linha) e a correção sugerida numa linha própria, e os documentos voltam a ficar pendentes |
| **Falha de comunicação** | O lote fica com **Status** **Erro**, com a descrição da falha em **Lista Erros**. Use **Consultar** de novo antes de reenviar os documentos |

No fim da consulta, a resposta da GINFES aparece na tela, com o número do lote e o status em que ele ficou: o aviso de lote ainda não processado, as mensagens de rejeição ou as mensagens que acompanham a autorização. Uma autorização sem mensagem não mostra nada. O formato da mensagem está em [NFS-e: Mensagens da Prefeitura na Consulta do Lote](Localization-for-Brazil_User-Manual_NFSe-Mensagens-Consulta-Lote.md).

### Corrigir e reenviar

Quando o lote fica com **Status** **Erro** por mensagem da prefeitura, corrija o cadastro apontado e transmita os documentos de novo. Eles vão para um lote novo, com o mesmo **Nº RPS**.

Se a prefeitura recusar o número do RPS, a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote** atribui ao documento o próximo número de **Nº Série RPS/Temporário**.

Quando o **Erro** vem de uma falha de comunicação na consulta, não reenvie: consulte o lote de novo. A prefeitura pode já ter autorizado as notas, e o reenvio criaria RPS em duplicidade.

### Primeira emissão em homologação

Faça a primeira emissão em homologação, com o certificado da filial e o acesso ao ambiente de homologação da GINFES, antes de liberar a filial para produção.

1. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** ligado.
2. Confira a **Nº Série RPS/Temporário**, os códigos de serviço (com **Cód. NBS**, **Cód. Operação de Serviços** e **Código IBPT**), o código de tributação NFS-e e os cadastros da filial e do cliente.
3. Registre uma fatura de venda de serviço para a filial e confira, no documento fiscal, o **Cód. Class. Trib. IBS/CBS** da primeira linha e os percentuais aproximados de tributos.
4. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**.
5. Com o lote **Enviado**, aguarde alguns minutos e use **Consultar**.
6. Se o lote ficar com **Erro**, corrija o que **Lista Erros** apontar e transmita de novo.
7. Com a nota autorizada, confira **Nº NFS**, **Data NFS** e **Cód. Verificação NFS** no **Documento Fiscal Registrado**, e veja se a **Chave de Acesso NFS-e** veio preenchida: é ela que define a configuração do DANFSe (veja a [seção 4](#4-danfse)).
8. Cancele a nota de teste (veja [Cancelar](#cancelar)).
9. Repita com as variantes da filial: ISS retido, retenções federais, PIS/COFINS da operação, cliente do exterior e cliente pessoa física. Confira se os valores da nota no portal da GINFES batem com os do documento fiscal.

### Cancelar

O cancelamento tem dois passos, no **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`):

1. **Cancelar Documento Fiscal** (grupo **Contabilização**): estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central. Nada é enviado à prefeitura neste passo.
2. **Cancelar NFS-e** (grupo **NFS-e**), habilitado depois do primeiro passo: envia à prefeitura o pedido de cancelamento da NFS-e, identificada pelo **Nº NFS**, com o código de cancelamento `1` (erro na emissão). A pergunta de confirmação fala em "SEFAZ", mas o pedido vai à GINFES.

Quando a prefeitura aceita o pedido, a mensagem "NFS-e cancelada com sucesso." é exibida e o campo **NFS Cancelada** do documento é marcado. Se ela recusar, a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" lista os motivos e a NFS-e continua autorizada, enquanto o documento fiscal continua cancelado no Business Central. Corrija o que for apontado e use **Cancelar NFS-e** de novo.

Com **Homologação** ligado, o pedido de cancelamento vai ao ambiente de homologação.

---

## 4. DANFSe

A GINFES pode devolver, na nota autorizada, a chave de acesso nacional, gravada na **Chave de Acesso NFS-e** do documento. Ela decide como o DANFSe sai. Confira o resultado na primeira emissão em homologação e configure a filial conforme a tabela:

| Nota autorizada | **Utilizar Layout Padrão** | **NFS-e sem Anexo** (aba **E-Mails de Documento**) | Resultado |
| --- | --- | --- | --- |
| **Com** a **Chave de Acesso NFS-e** | Ligado | Desligado | **Abrir DANFSe** mostra o DANFSe no Layout Padrão, e o e-mail do documento fiscal sai com ele anexo (veja [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md)) |
| **Sem** a **Chave de Acesso NFS-e** | Desligado | Ligado | O Business Central não gera a imagem da nota. Consulte e imprima a nota no portal da GINFES, com o **Nº NFS** e o **Cód. Verificação NFS**. O e-mail do documento fiscal sai sem o DANFSe |

Sem a chave de acesso nacional:

- **Abrir DANFSe** e **Atualizar DANFSe**, no **Documento Fiscal Registrado**, terminam com a mensagem "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e". Imprima a nota no portal da GINFES.
- Com **Utilizar Layout Padrão** ligado, **Abrir DANFSe** termina com a mensagem que começa por "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe", e **Imprimir** de uma nota autorizada para pedindo a **Chave de Acesso NFS-e**. Desligue **Utilizar Layout Padrão**.
- Com **NFS-e sem Anexo** desligado, o e-mail do documento fiscal fica pendente, aguardando a NFS-e.
- Com o Layout Padrão desligado, a ação **Imprimir** da página **NFS-e - Envio de Lote** imprime o RPS do documento.

---

## 5. Pontos de Atenção

- **Certificado vencido**: a transmissão, a consulta e o cancelamento são bloqueados até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **Nº Série RPS/Temporário com comprimento diferente da numeração fiscal**: o lote consultado fica **Sucesso**, mas os documentos ficam sem **Nº NFS**.
  - Ajuste a série (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) e consulte o lote de novo, pela página **NFS-e - Status de Lote RPS**.
- **Tabela IBPT desatualizada**: os tributos aproximados são calculados ao registrar a venda. Sem a tabela **Tributos IBPT** atualizada ou sem o **Código IBPT** do serviço, o documento fica sem tributos aproximados e os percentuais vão zerados. Mantenha a tabela atualizada antes de registrar as vendas da filial.
- **Município de incidência**: vai sempre o município da filial, também na natureza `2` (tributação fora do município).
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| Mensagens da prefeitura na tela, ao transmitir | Corrigir o cadastro indicado e transmitir de novo |
| "O GINFES não devolveu protocolo nem mensagem para o lote enviado." | A prefeitura recusou o lote sem mensagem. Conferir os cadastros e o certificado e transmitir de novo |
| "RPS &lt;nº&gt;: a natureza da operação "&lt;valor&gt;" não é aceita pelo GINFES, que admite os valores de 1 a 6." | Corrigir o **Cód. Tributação** do código de **Códigos Tributação NFS-e** |
| "Lote ainda não processado pela prefeitura (situação &lt;n&gt; - &lt;descrição&gt;). Consulte novamente em alguns minutos." | Consultar o lote de novo mais tarde |
| Mensagens da prefeitura na tela e em **Lista Erros**, depois da consulta | Corrigir o cadastro indicado e transmitir os documentos de novo |
| Mensagem da prefeitura sobre o grupo de IBS/CBS | Conferir o **Cód. Operação de Serviços** do código de serviço e o **Cód. Class. Trib. IBS/CBS** da primeira linha do documento |
| Mensagem da prefeitura sobre o endereço do exterior | Conferir o **Cód. Unidade Federal** do cliente e o **Cód. BACEN** do país |
| "Lote &lt;nº&gt; deverá ser consultado antes do envio." | Consultar o lote antes de reenviar o documento |
| Erro de campo obrigatório em **Nº Série RPS/Temporário** ou no **Último Nº Usado**, ao consultar | Configurar a série (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) |
| "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. ..." | Cadastrar o certificado renovado e selecioná-lo em **ID Certificado** |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e" | A nota não tem chave de acesso nacional: consultar e imprimir no portal da GINFES |
| "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe. ..." | Desligar **Utilizar Layout Padrão** |
| "Os seguintes erros ocorreram no cancelamento da NFS-e: ..." | Corrigir o que a prefeitura apontar e usar **Cancelar NFS-e** de novo |

- **Log de comunicação**: o envio e o retorno de cada operação com a prefeitura ficam registrados no log do serviço de integração, consultado pelo suporte. No Business Central ficam as mensagens do lote e os dados de retorno do documento.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **São Bernardo do Campo - SP** (`SP-SaoBernardoDoCampo`) |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Homologação** | Conforme o ambiente |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **Nº Máx. de Docs. por Lote WS** | `0` ou o máximo desejado |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Número de lote só com dígitos |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Série só com dígitos, com **Último Nº Usado** e o comprimento da numeração fiscal |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Config. NF-e** | **NFS-e suporta IBPT** | Ligado (automático) |
| **Config. NF-e** | **Texto Legal Valor Aprox. Impostos Automático** | Ligado |
| **Config. NF-e** | **Utilizar Layout Padrão** | Ligado se a nota volta com a chave de acesso nacional; desligado se não volta |
| **Config. NF-e** | **NFS-e sem Anexo** | Desligado se a nota volta com a chave de acesso nacional; ligado se não volta |
| **Config. NF-e** | **Incentivo Cultural**, **Regime Especial de Tributação** | Conforme a filial |
| **Série Filial Tipo Doc. Fiscal** | **Nº Série Fiscal** | Série de numeração só com dígitos |
| **Informação Filial** | **Inscrição Municipal** | Preenchida |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Item da lista de serviços (LC 116) |
| **Códigos de Serviço NFS** | **Cód. Tributação** | Código de tributação do município de São Bernardo do Campo |
| **Códigos de Serviço NFS** | **Cód. NBS** | Código NBS do serviço (obrigatório) |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS (obrigatório) |
| **Códigos de Serviço NFS** | **Código IBPT** | Código do serviço na tabela IBPT |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | Natureza da operação, de `1` a `6` |
| **Cliente** (exterior) | **Cód. Unidade Federal** | Estado ou província do cliente |
| **Países** | **Cód. BACEN** | Código BACEN do país do cliente do exterior |
