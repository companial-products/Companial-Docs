<!-- PR: #267 -->
# NFS-e: Exportação de Serviço — País da Prestação e Dados de Comércio Exterior

**Data**: Outubro/2026

---

## Sumário

Um serviço executado fisicamente fora do Brasil não tem município brasileiro de prestação: o layout nacional da NFS-e pede o **país da prestação** no lugar do município e, na exportação de serviço, o grupo de **comércio exterior** (`comExt`). Os dados da NFS-e do documento de venda e o documento fiscal ganharam a marca **Exportação de Serviço**, com o país da prestação, o país do resultado do serviço e os dados de comércio exterior. Com a marca, o **Município Prestação Serviço** deixa de ser exigido, a NFS-e do ambiente nacional sai com o país da prestação e sem código de município, e o IBS/CBS é apurado no município EXTERIOR (`9999999`). Documentos sem a marca continuam exatamente como antes.

---

## 1. Contexto

Até esta versão, o local da prestação de uma NFS-e era sempre o **Município Prestação Serviço**, um município brasileiro da tabela IBGE. Para emitir um serviço prestado no exterior, a saída era cadastrar o município EXTERIOR (`9999999`) e informá-lo no documento. O ambiente nacional recusa esse código com o erro `E0302` (município inexistente na tabela IBGE), e a nota acabava sendo emitida no portal do governo e registrada à mão no documento fiscal.

Agora o documento de venda informa que o serviço foi prestado no exterior, e em qual país. O sistema monta a NFS-e com o país da prestação, envia os dados de comércio exterior e não exige mais um município brasileiro.

### Quando se aplica

- Serviço executado fora do Brasil, para **qualquer tomador**: cliente estrangeiro ou cliente brasileiro.
- Documentos de venda de serviço — **Ordem de Venda**, **Fatura de Venda** e **Nota Crédito Venda** — e o documento fiscal gerado no registro.
- Todas as filiais podem marcar a exportação. O que vai para a nota depende do provedor da filial (seção [5](#5-o-que-a-nfs-e-informa)).

### Quando não se aplica

- Serviço prestado no Brasil, mesmo para cliente estrangeiro (exportação de serviço clássica, com resultado no exterior): o documento continua sem a marca, com o **Município Prestação Serviço** preenchido, como hoje.
- NFS-e de compra (recebidas).

---

## 2. Configuração

Antes da primeira emissão, confira os cadastros abaixo.

| Onde | O que conferir |
| --- | --- |
| **Países** | O país da prestação precisa ter o **Código ISO** de duas letras preenchido (ex.: `US`, `DE`). É esse código que vai na NFS-e |
| **Moedas** | Se a nota for informar uma moeda estrangeira, a moeda precisa ter o **Código Numérico ISO** preenchido (ex.: `840` para USD) |
| **Códigos de Serviço NFS** | O código de serviço do item precisa ter o **Cód. NBS** preenchido. Na exportação de serviço ele é exigido para qualquer tomador, não só para cliente estrangeiro |
| **Códigos Tributação NFS-e** da filial | Um código de tributação cujo **Cód. Tributação** seja `3` — no ambiente nacional, `3` é *Exportação de Serviço*. É esse código que o documento usa no **Cód. Tributação NFS-e** |
| **Municípios** | Nas filiais de provedor municipal, o município EXTERIOR precisa existir com o código `9999999`, na UF `EX` |
| **Unidades Federais** | A UF `EX` com **Código IBGE** = `99`, usada pelo IBS/CBS do município EXTERIOR |

A configuração do território `EX` e do município EXTERIOR está detalhada em [Endereço do exterior: UF EX com Código IBGE 99](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md#endereço-do-exterior-uf-ex-com-código-ibge-99). O cadastro do tomador estrangeiro (NIF ou motivo da não informação do NIF) segue as regras de [Tomador estrangeiro](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md#62-tomador-estrangeiro).

### Recomendações para a exportação de serviço

Validadas em emissões de homologação:

| Dado | Valor recomendado |
| --- | --- |
| Código de operação de serviços (`indOp`) | `100302`, o recomendado para exportação |
| Classificação tributária do IBS/CBS (`cClassTrib`) | `410004` |
| **Cód. País/Região do Resultado do Serviço** | Em branco, a não ser nos cenários de exportação que o exigem (seção [3.3](#33-comportamento-dos-campos)) |

O código de operação e a classificação tributária são configurados como em qualquer NFS-e: ver [Cód. Operações de Serviços (indOp)](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md#31-cód-operações-de-serviços-indop) e [Classificações Tributárias IBS/CBS](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md#36-classificações-tributárias-ibscbs-cclasstrib).

---

## 3. Uso no documento de venda

### 3.1 Como acessar

No documento de venda, abra a página **Dados da NFS-e** (`SalesFiscalDataNFSe`) pela linha **NFS-e** do factbox de dados fiscais — a mesma página do local da prestação, descrita em [NFS-e: Local da Prestação](Localization-for-Brazil_User-Manual_NFSe-Local-Prestacao-Obras-Atividades-Imoveis.md#a-página-dados-da-nfs-e). Os campos novos ficam no grupo **Exportação de Serviço**.

### 3.2 Campos do grupo Exportação de Serviço

| Campo | Descrição |
| --- | --- |
| **Exportação de Serviço** | Indica que o serviço foi prestado no exterior. Todos os demais campos do grupo só ficam habilitados com a marca |
| **Cód. País/Região da Prestação** | País onde o serviço foi prestado. Obrigatório com a marca. Precisa ter **Código ISO** de duas letras e não pode ser o Brasil |
| **Cód. País/Região do Resultado do Serviço** | País onde ocorre o resultado do serviço. Opcional; enviado só pelo ambiente nacional |
| **Modo de Prestação** | Como o serviço é prestado no comércio exterior. Obrigatório nas filiais do ambiente nacional |
| **Vínculo Prestador/Tomador** | Vínculo entre o prestador e o tomador do serviço |
| **Cód. Moeda do Comércio Exterior** | Moeda em que o serviço foi negociado. Em branco, vale a moeda do documento, ou o real quando o documento não tem moeda |
| **Valor do Serviço na Moeda** | Valor do serviço na moeda do comércio exterior. Obrigatório quando a moeda está preenchida |
| **Mecanismo de Apoio/Fomento do Prestador** | Mecanismo de apoio ou fomento ao comércio exterior usado pelo prestador |
| **Mecanismo de Apoio/Fomento do Tomador** | Mecanismo de apoio ou fomento ao comércio exterior usado pelo tomador |
| **Movimentação Temporária de Bens** | Se o serviço está vinculado a uma movimentação temporária de bens e a qual declaração |
| **Nº Declaração de Importação** | Número da DI. Habilitado e obrigatório quando a movimentação é **Vinculada - Declaração de Importação** |
| **Nº Registro de Exportação** | Número do RE. Habilitado e obrigatório quando a movimentação é **Vinculada - Declaração de Exportação** |
| **Enviar ao MDIC** | Indica que o ambiente nacional deve compartilhar a NFS-e com o Ministério do Desenvolvimento, Indústria, Comércio e Serviços (MDIC) |

### 3.3 Comportamento dos campos

**Ao marcar a Exportação de Serviço**, o sistema preenche sugestões, todas editáveis:

| Campo | Sugestão |
| --- | --- |
| **Cód. País/Região da Prestação** | O país do cliente, quando a **Categoria** do cliente é **Estrangeiro** e o país tem **Código ISO** válido e diferente do Brasil. Para cliente brasileiro o campo fica em branco, para ser escolhido |
| **Vínculo Prestador/Tomador** | O **Vínculo da Empresa** do cliente |
| **Mecanismo de Apoio/Fomento do Prestador** e **do Tomador** | **Nenhum** |
| **Movimentação Temporária de Bens** | **Não** |
| **Enviar ao MDIC** | Desmarcado |

O **Modo de Prestação** fica em branco, para ser escolhido.

**Ao desmarcar a Exportação de Serviço**, todos os campos do grupo são limpos.

**País da prestação igual ao Brasil** é recusado com a mensagem:

> *"O serviço prestado no exterior não pode ter o Brasil como Cód. País/Região da Prestação. Escolha o país onde o serviço foi prestado, ou desmarque a exportação de serviço."*

Um país sem **Código ISO** de duas letras também é recusado, com a indicação de preenchê-lo em **Países**.

**País do resultado do serviço.** O ambiente nacional só aceita esse país em alguns cenários de exportação; fora deles recusa a nota com o erro `E0591`. Deixe o campo em branco, a não ser nos cenários 2, 30, 58, 62, 72 e 76. O Brasil é aceito como país do resultado.

**Movimentação temporária de bens.** Trocar a opção limpa o número que deixou de se aplicar: o **Nº Declaração de Importação** só é mantido com **Vinculada - Declaração de Importação**, e o **Nº Registro de Exportação** só com **Vinculada - Declaração de Exportação**.

**Moeda do comércio exterior.** Apagar a moeda zera o **Valor do Serviço na Moeda**.

### 3.4 Opções dos campos de comércio exterior

As opções seguem o layout nacional da NFS-e, e o código ao lado é o que vai na nota. O código `0` (*desconhecido*) do layout não é oferecido: ele é reservado às notas que as prefeituras repassam ao ambiente nacional.

**Modo de Prestação**

| Código | Opção |
| :---: | --- |
| 1 | **Transfronteiriço** |
| 2 | **Consumo no Brasil** |
| 3 | **Presença Comercial no Exterior** |
| 4 | **Movimento Temporário de Pessoas Físicas** |

**Vínculo Prestador/Tomador**

| Código | Opção |
| :---: | --- |
| 0 | **Nenhum** |
| 1 | **Controlada** |
| 2 | **Controladora** |
| 3 | **Coligada** |
| 4 | **Empresa Matriz** |
| 5 | **Filial ou Subsidiária** |
| 6 | **Outro** |

**Mecanismo de Apoio/Fomento do Prestador**

| Código | Opção |
| :---: | --- |
| 01 | **Nenhum** |
| 02 | **ACC - Adiantamento sobre Contrato de Câmbio - Redução a zero do IR e do IOF** |
| 03 | **ACE - Adiantamento sobre Cambiais Entregues - Redução a zero do IR e do IOF** |
| 04 | **BNDES-Exim Pós-Embarque - Serviços** |
| 05 | **BNDES-Exim Pré-Embarque - Serviços** |
| 06 | **FGE - Fundo de Garantia à Exportação** |
| 07 | **PROEX - Equalização** |
| 08 | **PROEX - Financiamento** |

**Mecanismo de Apoio/Fomento do Tomador**

| Código | Opção |
| :---: | --- |
| 01 | **Nenhum** |
| 02 | **Adm. Pública e Repr. Internacional** |
| 03 | **Aluguéis e Arrend. Mercantil de máquinas/equip./embarc. e aeronaves** |
| 04 | **Arrend. Mercantil de aeronave para empresa de transporte aéreo público** |
| 05 | **Comissão a agentes externos na exportação** |
| 06 | **Despesas de armazenagem/mov. e transporte de carga no exterior** |
| 07 | **Eventos FIFA (subsidiária)** |
| 08 | **Eventos FIFA** |
| 09 | **Fretes/arrendamentos de embarcações ou aeronaves e outros** |
| 10 | **Material Aeronáutico** |
| 11 | **Promoção de Bens no Exterior** |
| 12 | **Promoção de Dest. Turísticos Brasileiros** |
| 13 | **Promoção do Brasil no Exterior** |
| 14 | **Promoção Serviços no Exterior** |
| 15 | **RECINE** |
| 16 | **RECOPA** |
| 17 | **Registro e Manutenção de marcas/patentes e cultivares** |
| 18 | **REICOMP** |
| 19 | **REIDI** |
| 20 | **REPENEC** |
| 21 | **REPES** |
| 22 | **RETAERO** |
| 23 | **RETID** |
| 24 | **Royalties/Assistência Técnica/Científica e Assemelhados** |
| 25 | **Serviços de avaliação da conformidade vinculados aos Acordos da OMC** |
| 26 | **ZPE** |

**Movimentação Temporária de Bens**

| Código | Opção |
| :---: | --- |
| 1 | **Não** |
| 2 | **Vinculada - Declaração de Importação** |
| 3 | **Vinculada - Declaração de Exportação** |

### 3.5 Cópia de documentos

A exportação, os países e os dados de comércio exterior acompanham a cópia de documentos: tanto de um documento de venda para outro quanto de um documento registrado para um novo documento (por exemplo, de uma fatura registrada para uma nota de crédito de estorno).

---

## 4. Validações no registro

Com a **Exportação de Serviço** marcada:

- O **Município Prestação Serviço** deixa de ser exigido. Se estiver preenchido, é ignorado: o documento marcado nunca envia um município brasileiro como local da prestação.
- O **Cód. Tributação NFS-e** continua obrigatório, como em qualquer NFS-e.
- O **Cód. País/Região da Prestação** é obrigatório, com **Código ISO** de duas letras e diferente do Brasil.
- **Somente nas filiais do ambiente nacional**, também são obrigatórios:
  - **Modo de Prestação**;
  - **Mecanismo de Apoio/Fomento do Prestador** e **Mecanismo de Apoio/Fomento do Tomador**;
  - **Movimentação Temporária de Bens**, com o **Nº Declaração de Importação** ou o **Nº Registro de Exportação** quando a opção os exige;
  - **Valor do Serviço na Moeda**, quando o **Cód. Moeda do Comércio Exterior** está preenchido.

O erro de registro abre a página **Dados da NFS-e**, onde os campos são corrigidos. Sem a marca, as validações são as de antes: o **Município Prestação Serviço** e o **Cód. Tributação NFS-e** continuam obrigatórios, e nenhum campo novo é exigido.

### Conferência antes da transmissão (ambiente nacional)

Nas filiais do ambiente nacional (**Cód. Cidade WS** = `SEFIN` na **Config. NF-e**), a conferência que roda antes da transmissão e lista todas as pendências numa única mensagem passou a considerar a exportação:

- No documento marcado, o município de prestação não é conferido; no lugar, são conferidos o país da prestação, o país do resultado (se preenchido), o modo de prestação, os mecanismos, a movimentação temporária de bens com o nº da DI ou do RE, e a moeda e o valor do comércio exterior.
- O **Cód. NBS** do código de serviço é exigido também para tomador brasileiro.
- **Documento sem a marca com o município EXTERIOR (`9999999`)** passa a ser uma pendência, para qualquer cliente, em vez de seguir para o governo e voltar rejeitado com o `E0302`:

  > *"O Município Prestação Serviço EXTERIOR (9999999) não é aceito pelo ambiente nacional: para serviço prestado no exterior, marque Exportação de Serviço e informe o país da prestação no Documento Fiscal Registrado, grupo NFS (ação Editar Campos)."*

As pendências do documento fiscal apontam para o **Documento Fiscal Registrado**, grupo **NFS**, ação **Editar Campos**.

---

## 5. O que a NFS-e informa

O que vai para a nota depende do provedor da filial que emite.

| Provedor da filial | Local da prestação | Comércio exterior (`comExt`) | País do resultado (`cPaisResult`) |
| --- | --- | --- | --- |
| **Ambiente nacional** (`SEFIN`) | País da prestação (`cPaisPrestacao`), sem código de município | Montado com os dados do documento | Enviado só quando o **Cód. País/Região do Resultado do Serviço** está preenchido |
| **São Paulo** | País da prestação (`cPaisPrestacao`) | Não existe no layout de São Paulo | — |
| **Demais provedores municipais** | Município EXTERIOR (`9999999`) | Não enviado a partir do documento | — |

Em todos os provedores, a tributação do ISSQN da nota vem do **Cód. Tributação NFS-e** do documento, como em qualquer NFS-e. No ambiente nacional, use o código de valor `3` (*Exportação de Serviço*); a exportação também admite outros valores, como imunidade ou não incidência, se for o caso da operação.

### Moeda e valor em moeda estrangeira

No ambiente nacional:

- Com o **Cód. Moeda do Comércio Exterior** preenchido, vão o **Código Numérico ISO** dessa moeda e o **Valor do Serviço na Moeda**.
- Com a moeda em branco, vale a regra de antes para cliente estrangeiro — a moeda do documento com o valor original dos movimentos do cliente, ou `986` (real) com o valor do documento —, agora também para o tomador brasileiro de serviço prestado no exterior.

Nos provedores municipais, a moeda continua seguindo a regra de antes.

### Tomador brasileiro e tomador estrangeiro

A marca não depende da **Categoria** do cliente: um cliente brasileiro com serviço executado no exterior emite a nota da mesma forma. Para cliente estrangeiro continuam valendo as exigências de [Tomador estrangeiro](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md#62-tomador-estrangeiro) (NIF ou motivo, **Código ISO** do país, **Cód. NBS**).

### Destinatário no exterior

Quando o documento tem um **Cód. Endereço Envio** de um endereço fora do Brasil, a NFS-e informa o destinatário com o endereço no exterior e o NIF, no lugar do CEP e do CPF/CNPJ. O NIF fica no cadastro do endereço de envio do cliente (página **Envio-a Endereço**), nos campos **Cód. NIF** e **Motivo da não informação do NIF**, com as mesmas regras do **Cartão Cliente**:

- Com o **Cód. NIF** preenchido, o motivo fica desabilitado.
- Sem NIF, a nota leva o motivo escolhido; sem nenhum dos dois, vai o código `0`, e quem decide se aceita é o ambiente.

O NIF e o motivo são gravados no documento fiscal no registro, junto com o CPF/CNPJ e a I.E. do endereço de envio. Um endereço de entrega digitado direto no documento, sem **Cód. Endereço Envio**, sai sem NIF.

**São Paulo** recusa a exportação de serviço com código de operação `100302` sem o destinatário no exterior. Nas filiais de São Paulo, informe na venda o endereço de envio do destinatário no exterior, com endereço, país e NIF ou motivo. O ambiente nacional aceita a nota sem destinatário.

### Código postal do exterior

O código postal de um endereço no exterior — do tomador e do destinatário — passa a ir para a nota como está no cadastro, com letras e espaços (ex.: `SW1A 1AA`). Antes ele era convertido para número, e códigos com letras chegavam como `0`. Vale para o ambiente nacional, São Paulo e Barueri.

---

## 6. IBS/CBS

Um serviço prestado no exterior não tem município brasileiro de incidência. No documento marcado, o IBS/CBS é calculado no município EXTERIOR (`9999999`, UF `EX`) — o mesmo "Exterior" que o governo registra como localidade de incidência da nota. O **Código Município IBS/CBS** do documento fiscal fica com `9999999`.

Com a UF `EX`, só se aplicam as alíquotas de IBS/CBS cadastradas sem UF ou município específico (alíquotas genéricas). Uma empresa que tenha cadastrado alíquotas de IBS apenas por UF ou por município fica com o IBS zerado no documento marcado.

Sem a marca, o município de incidência do IBS/CBS continua sendo o **Município Prestação Serviço**.

---

## 7. Documento fiscal

O grupo **NFS** do documento fiscal traz os mesmos campos da página **Dados da NFS-e**, copiados do documento de venda no registro:

| Página | Quando pode ser editado |
| --- | --- |
| **Documento Fiscal** e **Documento Fiscal de Venda** | Enquanto o arquivo da NFS-e não foi gerado |
| **Documento Fiscal Registrado** | Pela ação **Editar Campos**, enquanto a NFS-e não foi autorizada |
| **Pré-Visualização de Documento Fiscal** | Somente leitura |

Com a marca, o **Município Prestação Serviço** deixa de aparecer como obrigatório. As mesmas validações do registro (seção [4](#4-validações-no-registro)) valem para o documento fiscal digitado diretamente.

---

## 8. DANFSe

A prévia do DANFSe e o DANFSe da nota emitida mostram:

- **Local da prestação**: o país da prestação, no lugar de município/UF.
- **Município de incidência do IBS/CBS**: código `9999999`, município "Exterior" e UF "-".
- **Município de incidência do ISSQN**: "-".
- **Destinatário**: o NIF do endereço de envio, quando o destinatário não tem CPF/CNPJ.

O campo *País Resultado da Prestação do Serviço* do DANFSe mostra o país da prestação, e não o **Cód. País/Região do Resultado do Serviço**.

---

## 9. Compatibilidade

- **Documentos e filiais existentes**: sem a marca, nada muda — o município de prestação continua obrigatório e o país e os dados de comércio exterior do documento não são enviados. Os campos novos nascem em branco em todos os documentos.
- **Tomador estrangeiro sem a marca**: a nota continua com os dados de comércio exterior de antes; os dados do documento só vão com a marca.
- **Município EXTERIOR**: deixa de ser necessário como contorno no ambiente nacional. Nas filiais de provedor municipal, ele continua sendo o município enviado na exportação de serviço, por isso precisa estar cadastrado.

---

## 10. Resumo dos Campos Novos

### Dados fiscais do cabeçalho de venda (`SalesHeaderFiscalDataFNX`) e Documento Fiscal (`FiscalDocumentFNX`)

Os mesmos campos nas duas tabelas, na página **Dados da NFS-e** e no grupo **NFS** do documento fiscal.

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Exportação de Serviço** (`NFSServiceExport`) | Sim/Não | Serviço prestado no exterior |
| **Cód. País/Região da Prestação** (`NFSServiceCountryCode`) | Código | País da prestação (`cPaisPrestacao`) |
| **Cód. País/Região do Resultado do Serviço** (`NFSServiceResultCountryCode`) | Código | País do resultado do serviço (`cPaisResult`), opcional |
| **Modo de Prestação** (`NFSForeignTradeServiceMode`) | Opção | `mdPrestacao` |
| **Vínculo Prestador/Tomador** (`NFSForeignTradeLink`) | Opção | `vincPrest` |
| **Cód. Moeda do Comércio Exterior** (`NFSForeignTradeCurrencyCode`) | Código | Moeda do serviço (`tpMoeda`) |
| **Valor do Serviço na Moeda** (`NFSForeignTradeAmount`) | Decimal | `vServMoeda` |
| **Mecanismo de Apoio/Fomento do Prestador** (`NFSProviderSupportMechanism`) | Opção | `mecAFComexP` |
| **Mecanismo de Apoio/Fomento do Tomador** (`NFSCustomerSupportMechanism`) | Opção | `mecAFComexT` |
| **Movimentação Temporária de Bens** (`NFSTempGoodsMovement`) | Opção | `movTempBens` |
| **Nº Declaração de Importação** (`NFSImportDeclarationNo`) | Código | `nDI` |
| **Nº Registro de Exportação** (`NFSExportRegistrationNo`) | Código | `nRE` |
| **Enviar ao MDIC** (`NFSSendToMDIC`) | Sim/Não | `mdic` |

### Documento Fiscal (`FiscalDocumentFNX`) — destinatário

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Envio-a Cód. NIF** (`NFSShipToNIF`) | Código | NIF do endereço de envio, gravado no registro |
| **Envio-a Motivo da não informação do NIF** (`NFSShipToNIFNotInformedReason`) | Opção | Motivo da falta de NIF do endereço de envio, gravado no registro |

### Endereço de envio (`Ship-to Address`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Cód. NIF** (`NIFCodeFNX`) | Código | NIF do endereço de envio no exterior |
| **Motivo da não informação do NIF** (`NIFNotInformedReasonFNX`) | Opção | Por que o endereço no exterior não tem NIF; editável só com o **Cód. NIF** em branco |
