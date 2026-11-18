<!-- PR: #296 -->
# NFS-e: Emissão em Aracaju - SE (WebISS)

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **Aracaju - SE** emitem a NFS-e pelo Business Central por meio do web service da plataforma **WebISS**, usada pela prefeitura. A cidade aparece na lista de cidades de NFS-e como **Aracaju - SE (WebISS)** (código `SE-Aracaju-WebISS`). A integração cobre a transmissão, a consulta do lote, o cancelamento e o DANFSe no layout padrão, pelo mesmo fluxo de lotes das demais cidades.

A autenticação é feita só pelo **certificado digital** da filial. A autorização é **posterior**: a transmissão devolve um protocolo, e o número da NFS-e, o código de verificação e a chave de acesso nacional chegam na consulta do lote. A plataforma oferece ambiente de homologação.

---

## 1. Contexto

O WebISS de Aracaju não é o ISSWeb de São João da Barra - RJ: são fornecedores, web services e layouts diferentes. Também não tem relação com a antiga integração WebISS de Ipojuca - PE, que continua desativada.

### O que muda para a filial de Aracaju

| Tema | Como funciona em Aracaju - SE (WebISS) |
| --- | --- |
| **Layout** | RPS no padrão ABRASF 2.02 adaptado pela WebISS ao padrão nacional da NFS-e, com o código NBS, a inscrição estadual e o grupo de IBS/CBS da Reforma Tributária. O RPS e o lote são assinados com o certificado digital da filial |
| **Autenticação** | Certificado digital da filial. Não há usuário, senha nem token |
| **Envio** | Um lote com um ou mais documentos. A prefeitura recebe o lote, devolve um protocolo e o processa depois |
| **Autorização** | Na consulta do lote. Cada documento recebe o número da NFS-e, a data e hora de emissão, o código de verificação e a chave de acesso nacional, e o lote fica com **Status** **Sucesso** |
| **Rejeição** | As mensagens da prefeitura ficam em **Lista Erros** do lote, com o RPS a que se referem, e o documento pode ser corrigido e transmitido de novo |
| **Consulta** | Sempre necessária: é ela que traz as notas geradas ou os erros |
| **Município de incidência do ISS** | O **Município Prestação Serviço** do documento, quando o ISS é exigível |
| **Cancelamento** | Pelo sistema, com o motivo "Erro na emissão" |
| **DANFSe** | Layout padrão, gerado no Business Central a partir da chave de acesso |
| **Ambientes** | Homologação e produção |

### Tributos enviados

- **ISS**: a alíquota e o valor do ISS, a retenção (quando o documento tem ISS retido, com o tomador como responsável pela retenção) e a exigibilidade do ISS (o **Cód. Tributação NFS** do documento, veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)). Para o ISS não retido, o valor enviado é recalculado sobre o valor dos serviços menos as deduções, com duas casas decimais. Quando o ISS é exigível, a nota informa como município de incidência o **Município Prestação Serviço** do documento.
- **Retenções federais**: INSS e IRRF retidos. O **PIS** e a **COFINS** informados na nota são os de **apuração própria**. O **PIS e a COFINS retidos são somados à CSLL retida** e enviados no campo de CSLL, com o tipo de retenção que indica quais deles foram retidos. O CST do PIS/COFINS acompanha a nota. Assim, o valor líquido da nota desconta só o que foi retido, e a base do IBS/CBS calculada pela prefeitura desconta o PIS e a COFINS próprios. Também vão as outras retenções, as deduções e os descontos.
- **Construção civil**: o código da obra e a ART, quando o serviço ou o documento têm obra. Nos serviços de obra (itens 07.02, 07.04 a 07.08, 07.17 e 07.19 da LC 116), a nota leva também o endereço da obra: o **Endereço do Local da Prestação** do documento ou, sem ele, o endereço do cliente.
- **IBS/CBS (Reforma Tributária)**: quando a **primeira linha** do documento tem classificação tributária de IBS/CBS, a nota leva a finalidade "NFS-e regular", o indicador de uso ou consumo pessoal ("não"), o código indicador de operação, o CST e a classificação tributária dessa linha e, quando cadastrados, a tributação regular e o diferimento. O destinatário e o imóvel seguem as mesmas regras do padrão nacional, descritas em [NFS-e: Local da Prestação — Obras, Atividades/Eventos e Imóveis](Localization-for-Brazil_User-Manual_NFSe-Local-Prestacao-Obras-Atividades-Imoveis.md); o imóvel nunca é enviado num serviço de obra. O grupo de evento não existe no layout. A configuração desses cadastros está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md).
- **Valores de IBS/CBS**: a nota **não leva alíquotas nem valores de IBS/CBS**. A prefeitura os calcula e os devolve na NFS-e; o Business Central não os grava nem os compara com o próprio cálculo.
- **Tipo de operação**: segue a regra provisória dos demais provedores com Reforma Tributária, decidida pelo **Item Lista Serviço** (veja [Reforma Tributária - Suporte à Emissão de Notas de Serviço](Localization-for-Brazil_User-Manual_Reforma-Acompanhamento-Emissao-NotaServico.md)).
- **Inscrição estadual**: a **I.E.** da filial e a **Inscrição Estadual** do cliente vão na nota quando estão cadastradas, só com os dígitos. Inscrição "ISENTO" ou em branco não é enviada.
- **Tomador estrangeiro**: o NIF e o código do país.
- **Tributos aproximados**: não são enviados (a integração não usa o IBPT).

A nota informa a **Opção pelo Simples** e o **Regime Especial de Tributação** da **Config. NF-e**. O campo **Incentivo Cultural** não é enviado: a nota sempre informa que não há incentivo fiscal.

---

## 2. Configuração

### Certificado digital e habilitação na prefeitura

O certificado digital é a única credencial da integração: ele autentica a conexão com o web service da WebISS e assina o RPS, o lote e o pedido de cancelamento.

1. Confirme com a prefeitura de Aracaju que a filial está inscrita no município e habilitada a emitir NFS-e por web service. Não há usuário, senha ou token a gerar: a filial é identificada pelo certificado, pelo CNPJ e pela **Inscrição Municipal**.
2. Use o certificado digital A1 (arquivo `.pfx`) da empresa e cadastre-o em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**).
3. Selecione-o no campo **ID Certificado** da aba **NFS-e** (veja a tabela abaixo).

O Business Central avisa quando o certificado está perto de vencer e **bloqueia a transmissão e a consulta** quando ele já venceu, com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação

A WebISS tem um ambiente de homologação. Ligue **Homologação** na **Config. NF-e** da filial para emitir em homologação: a transmissão, a consulta e o cancelamento passam a ser enviados ao ambiente de testes da WebISS, e o DANFSe sai com a marca "NFS-e SEM VALIDADE JURÍDICA". As notas emitidas em homologação não têm validade jurídica. Para emitir em produção, no ambiente de Aracaju, desligue **Homologação**.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para Aracaju - SE (WebISS) |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Aracaju - SE (WebISS)** na lista (código `SE-Aracaju-WebISS`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (`WebISS`). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial |
| **Configuração Envio** | **Homologação** | Ligado para emitir em homologação; desligado para produção |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. A WebISS não informa o último número de RPS usado; desligado, toda transmissão falha |
| **Configuração Envio** | **WS NFS-e informa o IM** | Não usado por Aracaju. A **Inscrição Municipal** da filial vai sempre na nota |
| **Configuração Envio** | **Nome Usuário WS** e **Senha WS** | Não usados por Aracaju |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão |
| **Configuração Envio** | **ID Relatório p/ TXT** | Não usado. Deixe em branco |
| **Lote** | **Usar Data/Hora como Nº Lote** e **Nº Série Lote** | A WebISS recebe o número do lote só com os dígitos. Ligue **Usar Data/Hora como Nº Lote** ou informe em **Nº Série Lote** uma série só com dígitos |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | Opcional. Com `0`, **Transmitir Docs. Pendentes** envia todos os documentos pendentes num único lote; com outro valor, envia no máximo essa quantidade |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** (veja a [seção 5](#5-pontos-de-atenção)) |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Ligado**. É a única forma de obter o DANFSe no Business Central (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Desligado. Em Aracaju, a nota declara como município de incidência o local da prestação, e o DANFSe deve mostrar o mesmo |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `2000`. A WebISS aceita até 2000 caracteres na discriminação; com o limite, o sistema avisa já no registro da venda |
| — | **Nº Série RPS/Temporário** | Série de numeração só com dígitos, com o **Último Nº Usado** preenchido (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente. Em branco, ele precisa ser informado em cada venda |
| — | **NFS-e suporta IBPT** | Desligado automaticamente ao escolher Aracaju. Os tributos aproximados não são enviados |
| — | **Opção pelo Simples** | Conforme a filial. Enviado na nota |
| — | **Regime Especial de Tributação** | Conforme a filial. Enviado quando diferente de **00 - Nenhum** |
| — | **Incentivo Cultural** | Não enviado |

### Numeração e série do RPS

O layout da WebISS aceita **apenas dígitos** no número do RPS e no número do lote:

- O número do RPS é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão. A série de numeração do tipo de documento fiscal usado pela filial (**Nº Série Fiscal** da **Série Filial Tipo Doc. Fiscal** ou, sem série por filial, **Nº Série Nota Fiscal** do tipo de documento fiscal) precisa gerar números só com dígitos, com até 15 posições (por exemplo, `000031` e não `NFS000031`).
- A **Série** do tipo de documento fiscal precisa estar preenchida, com até 5 caracteres. Ela vem da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, do próprio tipo de documento fiscal.
- A série de **Nº Série RPS/Temporário** é usada:
  - pela ação **Criar Novo Nº**;
  - pela consulta do lote, que exige essa série preenchida e com **Último Nº Usado**, e a usa para localizar o documento de cada RPS devolvido.

  Ela precisa ter só dígitos e o mesmo número de posições da numeração do tipo de documento fiscal.

### Filial e cliente

Estes cadastros são exigidos na transmissão. O sistema não os confere de uma só vez: a transmissão é interrompida na primeira falta, com a mensagem do campo correspondente.

- **Informação Filial** da filial: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos. A **I.E.** da filial vai na nota quando está preenchida.
- **CEPs**: o CEP da filial e o do cliente cadastrados, com **Cód. Municipio IBGE**.
- **Cliente**: categoria, CEP, cidade, bairro, endereço, número, UF e país preenchidos (exceto pessoa física e cliente não identificado).
- **Categoria do cliente**: o tomador precisa ser **Pessoa Jurídica** com CNPJ, **Pessoa Física** (sem CPF, a nota vai com o CPF zerado) ou **Estrangeiro**. Nas demais categorias a nota segue sem o CPF ou CNPJ do tomador, e a prefeitura pode recusá-la.
- **Inscrição Estadual** do cliente: vai na nota quando está preenchida, para pessoa jurídica e pessoa física. Não vai para cliente do exterior.
- **Cliente do exterior**: o **Cód. NIF** na aba Fiscal da ficha do cliente e o **Cód. BACEN** do país em **Países/Regiões**. O cliente do exterior é identificado na nota pelo NIF e pelo país, sem CPF nem CNPJ.
- A **Inscrição Municipal** do cliente pessoa jurídica vai na nota quando ele é do mesmo município da filial.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Cód. Tributação** | O código de tributação do município de Aracaju. Vai na nota exatamente como está no campo |
| **Item Lista Serviço** | O item da lista de serviços da LC 116, com até 5 caracteres (por exemplo, `01.07`). **Obrigatório** no layout da WebISS. Vai na nota exatamente como está no campo. O sistema também usa esse item, sem pontos, para identificar os serviços de obra e o tipo de operação do IBS/CBS |
| **Código CNAE** | O CNAE do serviço, com 7 dígitos. Pontos, traços e barras são descartados; com menos de 7 dígitos, o CNAE não é enviado |
| **Cód. NBS** | O código NBS do serviço, com 9 dígitos. Vai na nota só com os dígitos |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS (6 dígitos), enviado quando a linha tem classificação tributária de IBS/CBS |
| **Cód. Obra** e **Cód. ART** | Enviados como dados da obra quando preenchidos e o documento não traz obra própria |
| **Processo de Suspensão Nº** | Enviado como número do processo quando a exigibilidade do ISS é `6` ou `7` (veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)) |
| **Tipo de Suspensão da ISS** | Não enviado |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos **Cód. Tributação**, **Item Lista Serviço**, **Código CNAE**, **Cód. Obra** e **Cód. ART** desse registro substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item.

### Códigos Tributação NFS-e

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. Na WebISS, o campo **Cód. Tributação** desse código é a **exigibilidade do ISS** do padrão ABRASF, e precisa ter um destes valores:

| Valor | Exigibilidade do ISS |
| --- | --- |
| `1` | Exigível |
| `2` | Não incidência |
| `3` | Isenção |
| `4` | Exportação |
| `5` | Imunidade |
| `6` | Exigibilidade suspensa por decisão judicial |
| `7` | Exigibilidade suspensa por processo administrativo |

A pesquisa do campo não oferece lista nesta filial: digite o valor. Com outro valor, a transmissão é recusada pelo serviço de integração com a mensagem "Exigibilidade do ISS '&lt;valor&gt;' do RPS &lt;número&gt; não é aceita pelo WebISS, que admite apenas os códigos 1 a 7 do padrão ABRASF 2.02.". Com o valor `1`, a nota informa o município de incidência do ISS (veja [Tributos enviados](#tributos-enviados)).

O **Município Prestação Serviço** e o **Cód. Tributação NFS** do documento podem ser corrigidos no **Documento Fiscal Registrado**, grupo NFS, ação **Editar Campos**.

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Docs. Pendentes**: envia num único lote os documentos pendentes da filial, até o limite de **Nº Máx. de Docs. por Lote WS**.
- **Transmitir Doc. Selecionado**: envia só o documento selecionado.

Antes de chamar a prefeitura, a transmissão é barrada com mensagem quando:

| Situação | Mensagem |
| --- | --- |
| Certificado vencido | "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir." |
| Nenhuma linha com item de serviço com código de serviço | "Nenhuma linha da nota fiscal possui Código de Serviço associado." |
| Discriminação com mais de 2000 caracteres | "Discriminação dos serviços excedeu o limite de 2000 caracteres. Total: &lt;quantidade&gt;" |
| Documento num lote ainda **Enviado** | "Lote &lt;nº&gt; deverá ser consultado antes do envio." |
| **Controlar Nº RPS** desligado | "Erro ao consultar último número de RPS: ..." |

Quando a prefeitura recebe o lote, ele fica com **Status** **Enviado** e o **Protocolo** recebe o protocolo da WebISS. A nota **ainda não está autorizada**: o número da NFS-e, o código de verificação e a chave de acesso só chegam na consulta.

Quando a prefeitura recusa o próprio lote (por exemplo, por falha na assinatura, no certificado ou na estrutura do arquivo), as mensagens aparecem na tela, uma por linha, no formato `<código> - <mensagem>`, seguidas de `<código> - Correção: <correção>` quando a prefeitura sugere a correção. Nada fica gravado: o documento continua pendente, sem lote. Anote a mensagem antes de fechar a tela.

### Consultar o processamento

Na página **NFS-e - Envio de Lote** ou na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`), selecione o lote e use **Consultar**. A consulta usa o protocolo do lote e não gera nota nova: pode ser repetida quantas vezes for preciso.

| Resultado da consulta | O que acontece no Business Central |
| --- | --- |
| **Em processamento** | O lote continua **Enviado**, com o aviso "Lote &lt;nº&gt; aguardando processamento no WebISS (protocolo &lt;protocolo&gt;); consulte novamente em alguns minutos." em **Lista Avisos** |
| **Nota gerada** | Cada documento recebe o **Nº NFS**, a **Data NFS** (data e hora de emissão), o **Cód. Verificação NFS** e a **Chave de Acesso NFS-e**, o **Nº Nota Fiscal** passa a ser o número da NFS-e e o lote fica com **Status** **Sucesso** |
| **Rejeitada** | O lote fica com **Status** **Erro** e as mensagens da prefeitura ficam em **Lista Erros**, no formato `RPS <número>/<série>: <código> - <mensagem>` (sem o prefixo `RPS`, quando a mensagem é do lote e não de um RPS). Quando a prefeitura sugere a correção, ela vem na linha seguinte, como `RPS <número>/<série>: <código> - Correção: <correção>` |
| **Falha de comunicação** | O lote continua **Enviado**, com a mensagem "Não foi possível consultar o lote &lt;nº&gt; no WebISS (protocolo &lt;protocolo&gt;): ..." em **Lista Avisos**. O reenvio não é liberado, porque as notas podem já ter sido geradas |

A consulta exige a série **Nº Série RPS/Temporário** preenchida (veja [Numeração e série do RPS](#numeração-e-série-do-rps)).

### Corrigir e reenviar

Quando o lote fica com **Status** **Erro** depois da consulta, corrija o cadastro apontado pelas mensagens de **Lista Erros** e transmita o documento novamente. O reenvio mantém o número e a série do RPS. Se a prefeitura recusar o número, a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote** atribui ao documento o próximo número de **Nº Série RPS/Temporário**.

### Primeira emissão em homologação

1. Confirme com a prefeitura que a filial está habilitada a emitir NFS-e por web service.
2. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** ligado.
3. Confira os códigos de serviço, os códigos de tributação NFS-e e os cadastros da filial e do cliente.
4. Registre uma fatura de venda de serviço para a filial.
5. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**.
6. Com o lote **Enviado**, use **Consultar** até o lote ficar com **Status** **Sucesso** ou **Erro**.
7. Corrija o que as mensagens da prefeitura apontarem e transmita de novo.
8. Com a nota autorizada, abra o **Documento Fiscal Registrado** e use **Abrir DANFSe** (veja a [seção 4](#4-danfse)).

### Cancelar

O cancelamento tem dois passos, no **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`):

1. **Cancelar Documento Fiscal** (grupo **Contabilização**): estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central. Nada é enviado à prefeitura neste passo.
2. **Cancelar NFS-e** (grupo **NFS-e**), habilitado depois do primeiro passo: envia à WebISS o pedido de cancelamento, assinado com o certificado da filial, com o número da NFS-e e o motivo "Erro na emissão". O motivo não pode ser alterado pelo Business Central.

Quando a prefeitura confirma o cancelamento, a mensagem "NFS-e cancelada com sucesso." é exibida e o campo **NFS Cancelada** do documento é marcado. Se ela recusar, a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" lista os motivos e a NFS-e continua ativa; corrija o que for apontado e use **Cancelar NFS-e** de novo.

---

## 4. DANFSe

Com **Utilizar Layout Padrão** ligado, o DANFSe da nota autorizada é gerado no padrão nacional a partir da chave de acesso devolvida na consulta, pelas ações **Abrir DANFSe** e **Atualizar DANFSe** do **Documento Fiscal Registrado**. Antes da transmissão, a ação **Imprimir** da página **NFS-e - Envio de Lote** gera o documento em modo DPS, para conferência. O layout, a marca d'água de homologação e a de cancelamento estão descritos em [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md).

O Layout Padrão é a única forma de obter o DANFSe no Business Central nesta integração: mantenha **Utilizar Layout Padrão** sempre ligado. Com o campo desligado, **Abrir DANFSe** e **Atualizar DANFSe** terminam com a mensagem "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e". Ligue o campo e use **Atualizar DANFSe**.

O DANFSe do Layout Padrão é montado com os dados do documento no Business Central. Os valores de IBS/CBS que ele mostra são os calculados pelo Business Central, e não os calculados pela prefeitura; o documento de referência para esses valores é a NFS-e da prefeitura, disponível no portal da WebISS.

---

## 5. Pontos de Atenção

- **Consulte sempre**: a transmissão só deixa o lote **Enviado**. Sem a consulta, o documento não recebe o número da NFS-e nem a chave de acesso, e o DANFSe não pode ser gerado.
- **Lote preso em Enviado**: consulte o lote antes de qualquer outra ação. **Marcar como Erro** (com **Permitir Erro Lote Manual** ligado) libera o documento para nova transmissão; use-a só depois de confirmar no portal da WebISS que a nota não foi gerada, para não emitir a mesma nota duas vezes.
- **Rejeição do lote na transmissão**: as mensagens de uma transmissão recusada não ficam gravadas no lote. Copie a mensagem antes de fechar a tela. As rejeições de conteúdo das notas chegam pela consulta e ficam em **Lista Erros**.
- **Mensagens longas**: cada mensagem é gravada em **Lista Erros** com até 250 caracteres. A mensagem completa da prefeitura fica no retorno da consulta, no [Log de Comunicação](Localization-for-Brazil_User-Manual_NFSe-Log-Comunicacao-Prefeitura.md).
- **Certificado vencido**: a transmissão e a consulta são bloqueadas até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **E-mail do documento**: com **Utilizar Layout Padrão** ligado, o DANFSe é anexado ao e-mail do documento.
- **Log de comunicação**: com **Log de Comunicação** ligado na **Config. NF-e**, o arquivo enviado à WebISS e o retorno de cada transmissão, consulta e cancelamento ficam disponíveis na ação **Log de Comunicação** das telas de lote (veja [NFS-e: Log de Comunicação com a Prefeitura](Localization-for-Brazil_User-Manual_NFSe-Log-Comunicacao-Prefeitura.md)).
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| `RPS <número>/<série>: <código> - <mensagem>` em **Lista Erros** | Recusa da prefeitura: corrigir o cadastro indicado na mensagem (e na correção sugerida, na linha seguinte) e transmitir de novo |
| "Lote &lt;nº&gt; aguardando processamento no WebISS (protocolo &lt;protocolo&gt;); consulte novamente em alguns minutos." | Aguardar e usar **Consultar** de novo |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "Exigibilidade do ISS '&lt;valor&gt;' do RPS &lt;número&gt; não é aceita pelo WebISS, que admite apenas os códigos 1 a 7 do padrão ABRASF 2.02." | Corrigir o **Cód. Tributação** do código de **Códigos Tributação NFS-e** usado no documento (veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)) |
| "Discriminação dos serviços excedeu o limite de 2000 caracteres. Total: &lt;quantidade&gt;" | Reduzir a discriminação e usar **Tamanho Máx. Discriminação** = `2000` |
| "O tamanho da discriminação (&lt;quantidade&gt;) excede o máximo permitido (2000)." | Aparece no registro da venda: reduzir as descrições das linhas ou os textos adicionais |
| "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. ..." | Cadastrar o certificado renovado e selecioná-lo em **ID Certificado** |
| "Lote &lt;nº&gt; deverá ser consultado antes do envio." | Consultar o lote do documento (veja [Consultar o processamento](#consultar-o-processamento)) |
| "Os seguintes erros ocorreram no cancelamento da NFS-e:" | Recusa do cancelamento pela prefeitura: conferir o motivo listado; a NFS-e continua ativa |
| "A cidade &lt;cidade&gt; não possui automatização de imagem NFS-e" | Ligar **Utilizar Layout Padrão** e usar **Atualizar DANFSe** |

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Aracaju - SE (WebISS)** (`SE-Aracaju-WebISS`) |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Homologação** | Conforme o ambiente |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** | Ligado, ou **Nº Série Lote** só com dígitos |
| **Config. NF-e** | **Utilizar Layout Padrão** | Ligado |
| **Config. NF-e** | **Incidência do ISSQN no Município do Prestador** | Desligado |
| **Config. NF-e** | **Tamanho Máx. Discriminação** | `2000` |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Série só com dígitos, com **Último Nº Usado**, no formato da numeração fiscal |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Série Filial Tipo Doc. Fiscal** | **Nº Série Fiscal** | Série que gera números só com dígitos, até 15 posições |
| **Série Filial Tipo Doc. Fiscal** | **Série** | Preenchida, com até 5 caracteres |
| **Informação Filial** | **Inscrição Municipal** | Preenchida |
| **Informação Filial** | **I.E.** | Preenchida, quando a filial tem inscrição estadual |
| **Cliente** | **Inscrição Estadual** | Preenchida, quando o cliente tem inscrição estadual |
| **Códigos de Serviço NFS** | **Cód. Tributação** | Código de tributação do município de Aracaju |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Item da lista de serviços da LC 116 (obrigatório) |
| **Códigos de Serviço NFS** | **Código CNAE** | CNAE de 7 dígitos |
| **Códigos de Serviço NFS** | **Cód. NBS** | Código NBS de 9 dígitos |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | Exigibilidade do ISS, de `1` a `7` |
