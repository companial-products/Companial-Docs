<!-- PR: #274, #309 -->
# NFS-e: Emissão em Embu das Artes - SP (Giap/Obaratec)

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **Embu das Artes - SP** emitem NFS-e pelo Business Central na plataforma **Giap**, da Obaratec, usada pela prefeitura. A cidade aparece na lista de cidades de NFS-e como **Embu das Artes - SP (Giap / Obaratec)** e cobre transmissão, reenvio após rejeição e cancelamento.

A autenticação é feita pelo **usuário e senha de integração** fornecidos pela prefeitura ou pela Obaratec. O certificado digital não é usado pela prefeitura, mas continua exigido pelo Business Central. A autorização é **imediata**: a resposta da transmissão já traz o número da NFS-e e o código de verificação, ou as mensagens de rejeição. Não há consulta posterior. O DANFSe não é gerado pelo Business Central: a nota é consultada e impressa no portal da prefeitura.

---

## 1. Contexto

### O que muda para a filial de Embu das Artes

| Tema | Como funciona em Embu das Artes - SP (Giap/Obaratec) |
| --- | --- |
| **Layout** | Formato próprio da plataforma Giap/Obaratec, sem assinatura digital |
| **Autenticação** | Usuário e senha de integração, informados em **Nome Usuário WS** e **Senha WS**. O certificado digital é exigido pelo Business Central, mas não é enviado à prefeitura |
| **Código de serviço municipal** | Vai a partir do campo **Item Lista Serviço** do código de serviço NFS |
| **Envio** | Vários documentos no mesmo envio |
| **Autorização** | Imediata. A resposta da transmissão grava o número da NFS-e e o código de verificação |
| **Rejeição** | As mensagens da prefeitura aparecem na transmissão, e o documento continua pendente |
| **Consulta** | Não existe. O resultado já vem na transmissão: não use **Consultar** |
| **Cancelamento** | Pedido de cancelamento enviado à prefeitura com o número da NFS-e, só em produção |
| **DANFSe** | Não é gerado pelo Business Central. A nota é consultada e impressa no portal da prefeitura |
| **Ambientes** | Produção e simulação (homologação), sem cancelamento na simulação |

### Tributos enviados

- **ISS**: alíquota do ISS (em percentual) e valor do ISS retido, quando há retenção.
- **Retenções federais**: os valores retidos de PIS, COFINS, CSLL, INSS e IRRF. O PIS e a COFINS de apuração própria não são enviados.
- **Desconto incondicional** do documento.
- **Tomador estrangeiro**: o documento do tomador vai zerado e a UF como `EX`.

Não são enviados a Embu das Artes: a exigibilidade do ISS (**Cód. Tributação NFS** do documento), o código de tributação municipal (**Cód. Tributação** do código de serviço), o IBS/CBS da Reforma Tributária (inclusive o **Cód. Operação de Serviços**), os tributos aproximados (IBPT) e os campos **Opção pelo Simples**, **Regime Especial de Tributação** e **Incentivo Cultural** da **Config. NF-e**.

---

## 2. Configuração

### Usuário e senha de integração

Embu das Artes identifica a empresa pelo usuário e pela senha de integração por web service e pela inscrição municipal da filial.

1. Solicite à prefeitura de Embu das Artes, ou à Obaratec, responsável pela plataforma Giap, o usuário e a senha de integração por web service da empresa.
2. Informe o usuário em **Nome Usuário WS** (até 30 caracteres) e a senha em **Senha WS** (até 50 caracteres), na aba **NFS-e** da **Config. NF-e** (veja a tabela abaixo).

O Business Central não confere essas credenciais antes do envio. Quando elas estão erradas, vencidas ou foram trocadas na plataforma, a transmissão e o cancelamento são recusados e a mensagem da recusa aparece na tela. Nesse caso, confirme as credenciais com a prefeitura e atualize **Nome Usuário WS** e **Senha WS**.

A **Senha WS** aparece em branco ao reabrir a página, mas o valor continua gravado. Para trocá-la, digite a nova senha; não apague o campo, porque isso grava uma senha vazia.

### Certificado digital

A prefeitura de Embu das Artes não usa o certificado digital, mas o Business Central exige um certificado válido em **ID Certificado** para qualquer operação de NFS-e da filial.

1. Cadastre o certificado digital A1 (arquivo `.pfx`) da empresa em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**).
2. Selecione-o no campo **ID Certificado** da aba **NFS-e**.

Com o certificado vencido, **a transmissão e o cancelamento de Embu das Artes também são bloqueados**, com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Mantenha o certificado da filial renovado e selecione o novo em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação (simulação)

Embu das Artes oferece um serviço de **simulação** da emissão. Para usá-lo, ligue **Homologação** na **Config. NF-e** da filial: a transmissão passa a ir para a simulação, que serve para conferir credenciais e cadastros sem emitir nota com validade. Para emitir em produção, desligue **Homologação**.

- O **cancelamento não existe na simulação**: com **Homologação** ligado, **Cancelar NFS-e** termina com "Os seguintes erros ocorreram no cancelamento da NFS-e:" seguido de "Cancelamento não suportado em homologação.".
- O Business Central trata o retorno da simulação como o de produção: o documento aceito recebe o número e o código de verificação devolvidos e deixa de ficar pendente de envio. Faça a simulação com documentos de teste, de preferência numa empresa de teste.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para Embu das Artes |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Embu das Artes - SP (Giap / Obaratec)** na lista (código `SP-EmbuDasArtes-GIAP_OB`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (`GiapOb`). Não é editável |
| **Configuração Envio** | **Nome Usuário WS** | Usuário de integração fornecido pela prefeitura ou pela Obaratec |
| **Configuração Envio** | **Senha WS** | Senha de integração fornecida pela prefeitura ou pela Obaratec |
| **Configuração Envio** | **ID Certificado** | Certificado digital válido da filial. Exigido pelo Business Central, mas não enviado à prefeitura |
| **Configuração Envio** | **Homologação** | Ligado para usar a simulação; desligado para produção |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. Embu das Artes não informa o último número de RPS usado; com o campo desligado, toda transmissão falha |
| **Configuração Envio** | **WS NFS-e informa o IM** | Não usado por Embu das Artes: a inscrição municipal da filial vai sempre na nota |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o endereço padrão do serviço de integração |
| **Configuração Envio** | **ID Relatório p/ TXT** | Em branco. É usado só por filial que não emite por web service |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | `0` (sem limite) ou a quantidade máxima de documentos por envio que a filial preferir |
| **Lote** | **Usar Data/Hora como Nº Lote** e **Nº Série Lote** | O lote é só um controle do Business Central e seu número não vai para a prefeitura. Ligue **Usar Data/Hora como Nº Lote** ou informe uma série em **Nº Série Lote** |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Desligado**. Embu das Artes não devolve a chave de acesso que o DANFSe padrão exige (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Não usado por Embu das Artes: afeta só o DANFSe do Layout Padrão |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | Opcional. O sistema não aplica limite próprio para Embu das Artes; use o limite indicado pela prefeitura, se houver |
| — | **Nº Série RPS/Temporário** | Série de numeração usada pela ação **Criar Novo Nº** (veja [Numeração](#numeração)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente. Em branco, ele precisa ser informado em cada venda |
| — | **NFS-e suporta IBPT** | Desligado automaticamente ao escolher Embu das Artes |
| — | **Opção pelo Simples**, **Regime Especial de Tributação** e **Incentivo Cultural** | Não usados por Embu das Artes |

### Numeração

- **Número do RPS**: é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão. O **Nº Nota Fiscal** é dado no registro da venda pela série de numeração do tipo de documento fiscal: o **Nº Série Fiscal** da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, ação **Séries por Filial** em **Tipos Documento Fiscal**) ou, sem série por filial, o **Nº Série Nota Fiscal** do próprio tipo de documento fiscal.
- **Série**: a série do documento não vai para Embu das Artes; só o número do RPS é enviado.
- **Nº Série RPS/Temporário**: usada pela ação **Criar Novo Nº**, quando é preciso dar outro número ao documento. Use uma série no mesmo formato da numeração fiscal.

### Filial e cliente

- **Informação Filial** da filial: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos. A **Inscrição Municipal** precisa ser a do cadastro da empresa em Embu das Artes: é por ela, junto com as credenciais, que a prefeitura identifica o prestador.
- **CEPs**: o CEP da filial e o do cliente cadastrados, com **Cód. Municipio IBGE**.
- **Cliente**: categoria, CEP, cidade, bairro, endereço, número, UF e país preenchidos (exceto pessoa física e cliente não identificado). Vão para a nota o CNPJ (pessoa jurídica) ou o CPF (pessoa física), o nome, o endereço e o e-mail do cliente. A inscrição municipal do cliente não é enviada.
- **Documento**: o **Município Prestação Serviço** identifica a cidade e a UF onde o serviço foi prestado; quando o documento tem endereço próprio do local da prestação, ele também vai na nota.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Item Lista Serviço** | O código do serviço na lista da prefeitura de Embu das Artes, no formato indicado por ela. Vai como está cadastrado, sem conferência de formato pelo sistema |
| **Código CNAE** | O CNAE do serviço. Vai na nota |
| **Cód. NBS** | Código NBS do serviço. Enviado quando preenchido, sem os pontos |
| **Cód. Obra** | Para serviço de obra: vai como código da obra o **Nº CNO** da obra informada no documento ou, sem obra no documento, o **Cód. Obra** do código de serviço |
| **Cód. Tributação** | Não enviado a Embu das Artes |
| **Cód. Operação de Serviços** | Não enviado a Embu das Artes |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos desse registro, entre eles **Item Lista Serviço**, **Código CNAE** e **Cód. Obra**, substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item. O código de serviço usado na nota é o da primeira linha com item de serviço.

### Códigos Tributação NFS-e

A exigibilidade do ISS não é enviada a Embu das Artes, mas o Business Central continua exigindo, na transmissão, que o **Cód. Tributação NFS** do documento fiscal aponte para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) cadastrado para a filial e com **Cód. Tributação** preenchido. Cadastre pelo menos um código para a filial e marque-o como **Padrão**, para que ele seja sugerido nas vendas.

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Docs. Pendentes**: transmite os documentos pendentes da filial num mesmo envio, até o limite de **Nº Máx. de Docs. por Lote WS**;
- **Transmitir Doc. Selecionado**: transmite só o documento selecionado.

Quando a prefeitura autoriza a nota, o documento recebe na hora o **Nº NFS** e o **Cód. Verificação NFS**, o **Nº Nota Fiscal** passa a ser o número da NFS-e, e o lote fica com **Status** **Sucesso**. A **Data NFS** não é preenchida: a integração não lê a data de emissão da resposta da prefeitura.

Quando a prefeitura recusa a nota, as mensagens dela aparecem na tela e o documento continua pendente. Se a prefeitura não informar o motivo, a mensagem é "Erro &lt;código&gt; ao emitir RPS &lt;nº&gt;". Falhas de acesso ou de comunicação com a plataforma também aparecem na tela, com o erro devolvido pela conexão.

### Consultar o processamento

Não há consulta: o resultado já vem na transmissão. **Não use Consultar** em lotes de Embu das Artes.

Na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`), **Consultar** coloca o lote com **Status** **Erro**, inclusive um lote já **Sucesso**, e troca as mensagens do lote pela mensagem "Um serviço de consulta de lote não existe.". A nota autorizada não é afetada: o documento continua com **Nº NFS** e **Cód. Verificação NFS** e não volta a ficar pendente. Para devolver o lote ao **Status** **Sucesso**, use **Marcar como Sucesso** na mesma página. Essa ação exige **Data NFS**, **Nº NFS** e **Cód. Verificação NFS** em cada documento do lote; como a **Data NFS** fica em branco em Embu das Artes, preencha-a com a data de emissão da nota, pela ação **Editar Campos** do **Documento Fiscal Registrado** (veja [Editar Campos do Documento Fiscal Registrado](Localization-for-Brazil_User-Manual_Editar-Campos-Documento-Fiscal-Registrado.md)).

### Corrigir e reenviar

Quando a transmissão é recusada, corrija o cadastro apontado pelas mensagens e transmita o documento novamente. O reenvio mantém o número do RPS. Se a prefeitura recusar o número por já ter sido usado, a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote** atribui ao documento o próximo número de **Nº Série RPS/Temporário**.

### Primeira emissão em homologação

1. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** ligado (simulação).
2. Confira as credenciais, o certificado, os códigos de serviço, o código de tributação NFS-e e os cadastros da filial e do cliente.
3. Numa empresa de teste, registre uma fatura de venda de serviço para a filial.
4. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**.
5. Corrija o que a mensagem da simulação apontar e transmita de novo.
6. Com o retorno aceito, confira no documento o **Nº NFS** e o **Cód. Verificação NFS**. O cancelamento não pode ser testado na simulação.

### Cancelar

O cancelamento pelo Business Central só funciona em produção. Ele tem dois passos, no **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`):

1. **Cancelar Documento Fiscal** (grupo **Contabilização**): estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central. Nada é enviado à prefeitura neste passo.
2. **Cancelar NFS-e** (grupo **NFS-e**), habilitado depois do primeiro passo: envia à prefeitura o pedido de cancelamento com o número da NFS-e.

O pedido vai sempre com o motivo "divergência cadastral", definido pela integração com Embu das Artes; o Business Central não permite escolher outro motivo. Se for preciso cancelar com outro motivo, cancele a nota no portal da prefeitura e, no Business Central, faça só o primeiro passo.

Quando a prefeitura aceita o cancelamento, a mensagem "NFS-e cancelada com sucesso." é exibida e o campo **NFS Cancelada** do documento é marcado. Se ela recusar, a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" lista os motivos e a NFS-e continua autorizada; corrija o que for apontado e use **Cancelar NFS-e** de novo.

---

## 4. DANFSe

O DANFSe de Embu das Artes não é gerado pelo Business Central:

- Embu das Artes não devolve chave de acesso, e o Layout Padrão precisa dela. Com **Utilizar Layout Padrão** ligado, **Abrir DANFSe** termina com a mensagem que começa por "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe";
- com **Utilizar Layout Padrão** desligado, como deve ficar, o serviço de integração não baixa a imagem da nota de Embu das Artes, e **Abrir DANFSe** e **Atualizar DANFSe** terminam com "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e".

Consulte e imprima a NFS-e no portal da prefeitura de Embu das Artes, com o **Nº NFS** e o **Cód. Verificação NFS** gravados no documento. A ação **Imprimir** da página **NFS-e - Envio de Lote** imprime o RPS (recibo provisório) do documento, e não a NFS-e.

**E-mail de documento**: como o Business Central não obtém o DANFSe, o e-mail do documento fiscal fica com **Status** **Pendente** e **Detalhes** "Aguardando NFS-e". Para enviar o e-mail sem a nota anexada, ligue **NFS-e sem Anexo** na aba **E-Mails de Documento** da **Config. NF-e**.

---

## 5. Pontos de Atenção

- **Certificado vencido**: mesmo sem ser usado pela prefeitura, o certificado vencido bloqueia a transmissão e o cancelamento até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **Credenciais**: quando a senha de integração é trocada na plataforma, atualize **Senha WS** na **Config. NF-e** da filial.
- **Consultar**: não use em lotes desta integração (veja [Consultar o processamento](#consultar-o-processamento)).
- **Cancelamento**: só em produção e sempre com o motivo "divergência cadastral".
- **E-mail de documento**: fica pendente, aguardando a NFS-e, a menos que **NFS-e sem Anexo** esteja ligado (veja a [seção 4](#4-danfse)).
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| Mensagem da prefeitura na transmissão, ou "Erro &lt;código&gt; ao emitir RPS &lt;nº&gt;" | Corrigir o cadastro indicado e transmitir de novo |
| Erro de acesso ou de comunicação na transmissão ou no cancelamento | Conferir **Nome Usuário WS** e **Senha WS** com a prefeitura e a disponibilidade do serviço; transmitir de novo |
| "Os seguintes erros ocorreram no cancelamento da NFS-e:" seguido de "Cancelamento não suportado em homologação." | O cancelamento só funciona em produção |
| "Um serviço de consulta de lote não existe." em **Lista Erros** | O lote foi consultado. Se o documento tem **Nº NFS**, usar **Marcar como Sucesso**; se não tem, transmitir de novo |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. ..." | Cadastrar o certificado renovado e selecioná-lo em **ID Certificado** |
| Erro apontando o **Cód. Tributação NFS-e** do documento (código inexistente na filial ou sem **Cód. Tributação**) | Cadastrar o código em **Códigos Tributação NFS-e** da filial, com **Cód. Tributação** preenchido, ou escolher outro no documento pela ação **Editar Campos** |
| "Nenhuma linha da nota fiscal possui Código de Serviço associado." | Usar item do tipo Serviço com **Cód. Serviço** na ficha do item |
| "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe. ..." | Desligar **Utilizar Layout Padrão** |
| "A cidade &lt;cidade&gt; não possui automatização de imagem NFS-e" | O DANFSe não sai pelo Business Central: consultar e imprimir a nota no portal da prefeitura |

- **Log de comunicação**: o envio e o retorno de cada operação com a prefeitura ficam registrados no log do serviço de integração, consultado pelo suporte. A senha de integração não aparece no log. No Business Central ficam as mensagens do lote e os dados de retorno do documento.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Embu das Artes - SP (Giap / Obaratec)** (`SP-EmbuDasArtes-GIAP_OB`) |
| **Config. NF-e** | **Nome Usuário WS** | Usuário de integração da prefeitura/Obaratec |
| **Config. NF-e** | **Senha WS** | Senha de integração da prefeitura/Obaratec |
| **Config. NF-e** | **ID Certificado** | Certificado digital válido da filial |
| **Config. NF-e** | **Homologação** | Ligado para simulação; desligado para produção |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Ligado, ou série preenchida |
| **Config. NF-e** | **Utilizar Layout Padrão** | Desligado |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Config. NF-e** | **NFS-e sem Anexo** (aba **E-Mails de Documento**) | Ligado, se o e-mail do documento deve sair sem a nota |
| **Informação Filial** | **Inscrição Municipal** | A do cadastro da empresa em Embu das Artes |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Código do serviço na lista da prefeitura |
| **Códigos de Serviço NFS** | **Código CNAE** | CNAE do serviço |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | Preenchido (não enviado, mas exigido pelo Business Central) |
