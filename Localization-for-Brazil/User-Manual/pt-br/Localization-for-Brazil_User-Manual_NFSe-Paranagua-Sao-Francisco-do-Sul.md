<!-- Branch: task/nfs-e-emissao-direta-no-web-service-de-p -->
<!-- Outubro/2026 — fnx: provedores Elotech (NFSEElotech203) e Atende.Net (NFSEAtendeNet) -->
# NFS-e: Emissão Direta em Paranaguá-PR (Elotech) e São Francisco do Sul-SC (Atende.Net)

**Data**: Outubro/2026

## Sumário

A emissão de NFS-e passa a contar com dois novos web services municipais, já com as informações da Reforma Tributária (IBS e CBS):

- **Paranaguá-PR**, no provedor **Elotech**, que passou a ser o sistema de NFS-e do município;
- **São Francisco do Sul-SC**, no provedor **Atende.Net (IPM)**.

As duas cidades usam o fluxo padrão de NFS-e da Localização — lote, RPS, transmissão, retorno, impressão e arquivamento do XML — sem nenhuma rotina específica. Basta selecionar a cidade na configuração de NFS-e da filial. Nada muda para as demais cidades, e quem emite Paranaguá pelo **PlugNotas** continua emitindo normalmente até decidir migrar.

> As duas cidades aparecem na lista de **Cód. Cidade WS** a partir da atualização do serviço de NFS-e da Localização.

## 1. Paranaguá-PR (Elotech)

### Antes de começar

O web service da Elotech não usa usuário: cada envio é identificado pelo CNPJ, pela inscrição municipal e por uma **senha de acesso** gerada pelo próprio contribuinte.

1. Procure o setor de tributação da prefeitura e faça o credenciamento para uso do web service.
2. Já credenciado, acesse a área restrita do portal da NFS-e de Paranaguá e clique em **Gera Senha WS**. A senha é enviada para o e-mail cadastrado no portal.

### Configuração

Em **Config. NF-e** da filial, na aba **NFS-e**:

| Grupo | Campo | Valor |
| --- | --- | --- |
| Configuração Envio | **Cód. Cidade WS** | `PR-Paranagua-Elotech` — *Paranaguá - PR (Elotech)* |
| Configuração Envio | **Tipo de WS** | Preenchido automaticamente com `Elotech` |
| Configuração Envio | **ID Certificado** | Certificado digital da empresa |
| Configuração Envio | **Senha WS** | A senha gerada no portal (opção **Gera Senha WS**) |
| Configuração Envio | **Nome Usuário WS** | Não é usado por este provedor |
| Configuração Envio | **Homologação** | Marcado para testes, desmarcado para emissão real |
| Configuração Envio | **Controlar Nº RPS** | Marcado — o web service não informa o último RPS emitido |
| Lote | **Nº Máx. de Docs. por Lote WS** | Até **50** |

A **inscrição municipal (CCM)** da filial é obrigatória: ela identifica o contribuinte em todo envio.

### Homologação

Homologação e produção usam o mesmo endereço; o ambiente é definido pelo campo **Homologação**. Notas emitidas em homologação não têm validade jurídica.

### Transmissão e retorno

- A transmissão é síncrona: a prefeitura processa o lote na hora e devolve, para cada RPS, o **número da NFS-e**, o **código de verificação**, a **data de emissão** e a **chave de acesso da NFS-e nacional**.
- Se qualquer RPS do lote for rejeitado, a prefeitura rejeita o lote inteiro e nenhuma nota é emitida. A mensagem de erro traz o código, a descrição e a orientação de correção da prefeitura.

### Cancelamento

O cancelamento está disponível pelo sistema, pela ação de cancelamento habitual do documento fiscal.

### Informações enviadas na nota

- **ISS**: alíquota, valor, retenção e a exigibilidade do ISS, que vem do **código de tributação NFS-e** da filial.
- **Códigos de serviço**: item da lista de serviços, CNAE, código de tributação municipal e NBS, do cadastro de códigos de serviço.
- **Retenções federais**: IR, INSS e CSLL retidos. Para PIS e COFINS, vai o valor **retido** quando houver; sem retenção, vai o de **apuração própria**, com CST e base de cálculo.
- **IBS/CBS**: município de incidência (definido pelo local de fornecimento do código de operação do serviço), base de cálculo, alíquotas e valores de IBS e CBS, indicador de operação, CST e classificação tributária — incluindo tributação regular e diferimento quando houver.
- **Imóvel**: inscrição imobiliária, CIB ou endereço do imóvel, quando o indicador de operação exigir (020101, 020201 e 020301).
- **Tomador no exterior**: NIF e endereço no exterior (país, cidade e estado/província).
- **Inscrição estadual do tomador**: a do cadastro do cliente; sem inscrição (não inscrito, isento ou exterior), o campo não é enviado.

### Limitações atuais

- As informações de **comércio exterior** (modo de prestação, vínculo entre as partes, moeda e mecanismos de fomento) ainda não são enviadas.
- O **destinatário da operação**, quando diferente do tomador, não é enviado.

## 2. São Francisco do Sul-SC (Atende.Net)

### Antes de começar

1. No **Portal do Cidadão/Autoatendimento** do município, localize o serviço **Emissão de NFS-e por WebService**, selecione **Acesso ao Usuário** e confirme a solicitação.
2. No módulo de Nota Fiscal do Atende.Net, em **Personalização do Prestador**, aba **Webservice**, habilite **Utiliza Retorno Completo na Importação de XML**. Sem essa opção a prefeitura não devolve as informações de IBS e CBS no retorno.

### Configuração

Em **Config. NF-e** da filial, na aba **NFS-e**:

| Grupo | Campo | Valor |
| --- | --- | --- |
| Configuração Envio | **Cód. Cidade WS** | `SC-SaoFranciscoDoSul-AtendeNet` — *São Francisco do Sul - SC (Atende.Net)* |
| Configuração Envio | **Tipo de WS** | Preenchido automaticamente com `AtendeNet` |
| Configuração Envio | **ID Certificado** | Certificado digital da empresa — a nota é assinada digitalmente |
| Configuração Envio | **Nome Usuário WS** / **Senha WS** | Somente se a prefeitura exigir usuário e senha no acesso ao web service |
| Configuração Envio | **Homologação** | Marcado para envio de teste (ver abaixo) |
| Configuração Envio | **Controlar Nº RPS** | Marcado — o web service não informa o último RPS emitido |
| Lote | **Nº Máx. de Docs. por Lote WS** | **1** (recomendado — ver abaixo) |

O município do prestador e do tomador é enviado no **código TOM da Receita Federal**, convertido automaticamente a partir do código IBGE do cadastro. Não é preciso cadastrar nada a mais.

### Uma nota por arquivo

A prefeitura recebe **uma nota por arquivo**, e o sistema envia cada documento do lote separadamente. Se uma das notas for rejeitada, a transmissão termina com erro e nenhum documento é atualizado — mesmo os que a prefeitura já tenha emitido. A mensagem de erro informa quais RPS já foram emitidos.

Essas notas **não se perdem nem são emitidas em duplicidade**: cada documento é enviado com um identificador único (o nº do documento fiscal), e quando ele é reenviado a prefeitura devolve a nota já gerada, que é então registrada no documento.

Por isso a configuração recomendada é **um documento por lote** (**Nº Máx. de Docs. por Lote WS** = 1): assim uma nota rejeitada nunca segura as demais.

### Envio de teste

Com **Homologação** marcado, a prefeitura faz todas as validações mas **não emite** a nota. Se o arquivo estiver correto, a transmissão termina com a mensagem **"NFS-e válida para emissão."**, apresentada como erro, e o documento continua pendente. Para emitir de fato, desmarque **Homologação** e transmita novamente.

### Cancelamento

O web service do Atende.Net não oferece cancelamento. Ao tentar cancelar pelo sistema, a Localização informa que a operação não está disponível para o município; o cancelamento deve ser feito no portal da prefeitura.

### Informações enviadas na nota

- **ISS**: alíquota, valor tributável, dedução, ISS retido e a situação tributária, que vem do **código de tributação NFS-e** da filial.
- **Códigos de serviço**: o código de tributação nacional alimenta o subitem da lista de serviços e o código de tributação municipal alimenta o código de atividade do município.
- **Retenções federais**: IR, INSS e contribuição social. PIS e COFINS **próprios** vão no grupo próprio (CST, base e alíquotas), que reduz a base do IBS/CBS; os **retidos** vão nos campos de retenção, que reduzem o valor líquido da nota.
- **IBS/CBS**: o município de incidência junto aos valores da nota e o bloco da operação com finalidade, indicador de operação, CST e classificação tributária.
- **Imóvel**: inscrição imobiliária, CIB ou endereço, quando o indicador de operação exigir; omitido nos demais casos.
- **Exterior**: local de prestação, país e código NBS para serviço prestado fora do Brasil ou tomador estrangeiro.

## 3. Migração a partir do PlugNotas

Quem emite Paranaguá pelo PlugNotas não é afetado. Para passar a emitir direto na prefeitura, basta trocar o **Cód. Cidade WS** da filial para `PR-Paranagua-Elotech` e preencher a **Senha WS**, conforme a seção 1.
