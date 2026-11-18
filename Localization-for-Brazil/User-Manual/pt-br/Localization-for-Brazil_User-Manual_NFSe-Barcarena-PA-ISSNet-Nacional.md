<!-- PR: #303 -->
# NFS-e: Emissão em Barcarena - PA (ISSNet Nacional)

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **Barcarena - PA** emitem a NFS-e pelo Business Central na plataforma **ISSNet**, usada pela prefeitura de Barcarena, no layout da NFS-e Nacional (DPS). A cidade aparece na lista de cidades de NFS-e como **Barcarena - PA (ISSNet Nacional)** e cobre transmissão, cancelamento e DANFSe.

A autenticação é feita só pelo **certificado digital** da filial. A autorização é **imediata**: a resposta da transmissão já traz a NFS-e autorizada, com número, chave de acesso e data de emissão, ou a lista de erros. Não há consulta posterior. O DANFSe é gerado no próprio Business Central, no layout padrão nacional.

Quando a ISSNet recusa o XML por falha de esquema (`E160`), ele pode ser conferido no validador da ISSNet com o arquivo obtido no **Log de Comunicação da NFS-e** (veja [Validar o XML no validador da ISSNet](#validar-o-xml-no-validador-da-issnet)).

A integração antiga, **Barcarena - PA**, não aparece na lista de cidades e não é tratada neste manual.

---

## 1. Contexto

### O que muda para a filial de Barcarena

| Tema | Como funciona em Barcarena - PA (ISSNet Nacional) |
| --- | --- |
| **Layout** | DPS do layout da NFS-e Nacional, assinada com o certificado digital da filial e enviada à ISSNet |
| **Autenticação** | Certificado digital da filial. Não há usuário, senha nem token |
| **Inscrição municipal** | Vai sempre na DPS, a partir da **Inscrição Municipal** da filial |
| **Código de serviço** | Código de tributação **nacional** (6 dígitos) e código de tributação **municipal** de Barcarena, só com dígitos |
| **Envio** | Vários documentos por lote. Um único erro faz a ISSNet recusar o lote inteiro |
| **Autorização** | Imediata. A resposta da transmissão grava o número da NFS-e, a chave de acesso e a data de emissão |
| **Rejeição** | As mensagens da ISSNet aparecem na tela, no momento da transmissão, e o documento continua pendente |
| **Consulta** | Não existe. O resultado já vem na transmissão |
| **Cancelamento** | Evento de cancelamento do layout nacional, com a chave de acesso da nota |
| **DANFSe** | Layout padrão nacional, gerado no Business Central a partir da chave de acesso |
| **Ambientes** | Homologação e produção |

### Diferenças em relação a Brasília e Goiânia

Barcarena usa a mesma integração ISSNet Nacional de [Brasília](Localization-for-Brazil_User-Manual_NFSe-Brasilia-DF-ISSNet-Nacional.md) e [Goiânia](Localization-for-Brazil_User-Manual_NFSe-Goiania-GO-ISSNet-Nacional.md), com o mesmo layout e o mesmo comportamento: DPS assinada com o certificado da filial, autorização imediata na transmissão, nenhuma consulta, cancelamento por evento com a chave de acesso, DANFSe pelo Layout Padrão e os mesmos tributos enviados e não enviados. A configuração da filial é a mesma, campo a campo. O que muda:

| Tema | Em Barcarena |
| --- | --- |
| **Município e endereço de produção** | Ao escolher **Barcarena - PA (ISSNet Nacional)**, o serviço de integração envia a DPS ao endereço de produção da ISSNet para Barcarena. O município emissor da DPS é o do CEP da filial (`1501303`) |
| **Homologação** | É o mesmo ambiente de homologação de Brasília e Goiânia (veja [Homologação](#homologação)) |
| **Integração anterior** | Não há troca de integração a fazer. Os manuais de Brasília e Goiânia têm uma seção para a filial que ainda usa a integração ISSNet antiga, que não existe em Barcarena |
| **DANFSe com Utilizar Layout Padrão desligado** | Não há busca do DANFSe na prefeitura, como em Brasília. Em Goiânia, essa busca vai ao sistema de notas antigo da prefeitura (veja a [seção 4](#4-danfse)) |

### Tributos enviados

- **ISS**: a DPS leva a tributação do ISSQN (o valor nacional do **Cód. Tributação NFS** do documento, veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)), a retenção (quando o documento tem ISS retido) e o município de prestação. A **alíquota do ISS não é enviada**: a ISSNet aplica a alíquota parametrizada pela prefeitura de Barcarena.
- **Retenções federais**: PIS, COFINS, CSLL, IRRF e INSS retidos. O **PIS e a COFINS de apuração própria não são enviados** nesta integração: o grupo de PIS/COFINS da DPS leva só a indicação de retenção, com CST `00`.
- **Tributos aproximados**: os valores do IBPT vão zerados. A lista de cidades deixa o campo **NFS-e suporta IBPT** desligado para esta integração.
- **IBS/CBS (Reforma Tributária)**: quando a linha do documento tem classificação tributária de IBS/CBS, a DPS leva o código indicador de operação, o CST, a classificação tributária e, conforme o caso, a tributação regular, o diferimento, o destinatário e os grupos de obra, imóvel e evento. A ISSNet **exige o grupo IBS/CBS nas notas com competência a partir de 01/10/2026**. A configuração desses cadastros está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md).
- **Tomador estrangeiro**: NIF (ou o motivo da não informação do NIF), país e informações de comércio exterior. O CEP e a cidade do cliente do exterior vão como `99999999` e `EXTERIOR`.

O prestador é informado na DPS como **não optante** pelo Simples Nacional e sem regime especial de tributação. Os campos **Opção pelo Simples**, **Regime Especial de Tributação** e **Incentivo Cultural** da **Config. NF-e** não são enviados.

---

## 2. Configuração

### Certificado digital

O certificado digital é a única credencial da ISSNet: o acesso ao web service é feito por certificado, sem usuário nem senha. Ele autentica a conexão e assina cada DPS, o lote e o evento de cancelamento.

1. Use o certificado digital A1 (arquivo `.pfx`) da empresa, o mesmo usado na NF-e. Ele precisa ser ICP-Brasil e conter o CNPJ do estabelecimento emissor ou o da matriz.
2. Cadastre o certificado em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**).
3. Selecione-o no campo **ID Certificado** do grupo **Configuração Envio**, na aba **NFS-e** (veja a tabela abaixo).

O Business Central avisa quando o certificado está perto de vencer e **bloqueia a transmissão e o cancelamento** quando ele já venceu, com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação

A ISSNet tem um ambiente de homologação, o mesmo para Barcarena, Brasília e Goiânia. O acesso a ele precisa ser solicitado ao suporte da plataforma, pelo telefone (67) 3041-2075 ou pelo e-mail suporte@notaeletronica.com.br, de segunda a sexta-feira, das 8h30 às 18h30 (horário de Brasília).

Com o acesso liberado, ligue **Homologação** na **Config. NF-e** da filial: a DPS e o cancelamento passam a ser enviados ao ambiente de homologação. As notas emitidas ali não têm validade jurídica, e o DANFSe sai com a marca "NFS-e SEM VALIDADE JURÍDICA". Para emitir em produção, desligue **Homologação**.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para Barcarena - PA (ISSNet Nacional) |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Barcarena - PA (ISSNet Nacional)** na lista (código `PA-Barcarena-ISSNetNacional`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (`Ginfes`). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial |
| **Configuração Envio** | **Homologação** | Ligado para emitir em homologação; desligado para produção |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. A ISSNet não informa o último número usado, então a numeração precisa ser controlada pelo Business Central |
| **Configuração Envio** | **WS NFS-e informa o IM** | Sem efeito em Barcarena: a inscrição municipal da filial vai sempre na DPS, ligado ou desligado |
| **Configuração Envio** | **Nome Usuário WS** e **Senha WS** | Não usados por Barcarena |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão: é o endereço do serviço de integração da Localização, usado por todas as cidades |
| **Configuração Envio** | **ID Relatório p/ TXT** | Em branco. Só é usado por filial que gera a NFS-e por arquivo, sem web service |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | `0` (sem limite) ou um valor baixo. A ISSNet recusa o lote inteiro quando um dos documentos tem erro: com `1`, cada transmissão leva um documento e um erro não segura os demais, mas **Transmitir Docs. Pendentes** passa a enviar um documento por vez |
| **Lote** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Um dos dois é obrigatório: ligue **Usar Data/Hora como Nº Lote** ou informe em **Nº Série Lote** uma série numérica |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Ligado**. É a forma de obter o DANFSe desta integração no Business Central (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Afeta só a impressão do DANFSe, conforme [NFS-e: Município de Incidência do ISSQN no DANFSe](Localization-for-Brazil_User-Manual_NFSe-DANFSe-Municipio-Incidencia-ISSQN.md) |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `1000`. A DPS aceita até 1000 caracteres na descrição do serviço e o texto que passa disso é cortado no envio; com o limite, o sistema avisa já no registro da venda |
| — | **Nº Série RPS/Temporário** | Série de numeração só com dígitos, usada pela ação **Criar Novo Nº** (veja [Numeração e série da DPS](#numeração-e-série-da-dps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente. Em branco, ele precisa ser informado em cada venda |
| — | **NFS-e suporta IBPT** | Desligado automaticamente ao escolher Barcarena. Não é editável |
| — | **Opção pelo Simples**, **Incentivo Cultural** e **Regime Especial de Tributação** | Não enviados. A DPS declara o prestador como não optante pelo Simples Nacional |

### Numeração e série da DPS

O layout nacional aceita **apenas dígitos** no número e na série da DPS:

- O número da DPS é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão. A série de numeração do tipo de documento fiscal usado pela filial, e também a de **Nº Série RPS/Temporário** (usada pela ação **Criar Novo Nº**), precisam gerar números só com dígitos, com até 15 posições (por exemplo, `000031` e não `NFS000031`).
- A **Série** do tipo de documento fiscal precisa estar preenchida, ter só dígitos e até 5 posições. Ela vem da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, do próprio tipo de documento fiscal.

Fora disso, a ISSNet rejeita a DPS por falha no esquema XML.

### Filial e cliente

- **Informação Filial** da filial: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos. Sem eles, a transmissão é barrada com a mensagem de campo obrigatório.
- **CEPs**: o CEP da filial cadastrado com **Cód. Municipio IBGE** de 7 dígitos (`1501303` para Barcarena). O município emissor da DPS é o desse CEP.
- **Cliente**: categoria, CEP, cidade, bairro, endereço, número, UF e país preenchidos (exceto pessoa física e cliente não identificado). Para cliente do exterior, **Cód. NIF** ou **Motivo da não informação do NIF** na aba Fiscal da ficha do cliente.

Diferentemente do Emissor Público Nacional, o Business Central não faz uma conferência completa antes do envio: ele barra a transmissão no primeiro dado obrigatório que faltar na filial ou no cliente, e os demais problemas de cadastro (códigos de serviço, por exemplo) voltam como rejeição da ISSNet, na tela.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Cód. Tributação** | O código de tributação **nacional**, de 6 dígitos (os 4 primeiros são o item da LC 116). Pontos são descartados no envio |
| **Item Lista Serviço** | O código de tributação **municipal** de Barcarena, **só com dígitos** (até 10). A ISSNet exige esse código; pontos e outros sinais não são retirados no envio e fazem a DPS ser rejeitada |
| **Cód. NBS** | Código NBS do serviço. O manual da ISSNet trata o código como obrigatório na DPS: preencha para todos os serviços |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS (6 dígitos), enviado quando a linha tem classificação tributária de IBS/CBS |
| **Código CNAE** | Não enviado a Barcarena |

A correlação entre o código de tributação municipal e o nacional está disponível na área de downloads da página inicial do ISS.net Online do município.

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos **Cód. Tributação** e **Item Lista Serviço** desse registro substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item.

### Obras, imóveis e atividades/eventos

No layout da ISSNet, a obra, o imóvel e a atividade/evento são identificados **pelo endereço**: a DPS não aceita o código da obra no CNO, o CIB do imóvel nem o identificador da atividade/evento. Nos documentos das filiais de Barcarena, deixe em branco o **Nº CNO** da obra (e o **Cód. Obra** do código de serviço), o **CIB** do imóvel e o **ID da Atividade/Evento**, e informe o endereço do local. A **Inscrição Imobiliária Fiscal** do imóvel continua aceita. Os cadastros estão descritos em [NFS-e: Local da Prestação — Obras, Atividades/Eventos e Imóveis](Localization-for-Brazil_User-Manual_NFSe-Local-Prestacao-Obras-Atividades-Imoveis.md).

### Códigos Tributação NFS-e

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. Em Barcarena, o campo **Cód. Tributação** desse código precisa ter um dos valores nacionais da tributação do ISSQN:

| Valor | Significado nacional |
| --- | --- |
| `1` | Operação Tributável |
| `2` | Imunidade |
| `3` | Exportação de Serviço |
| `4` | Não Incidência |

Um valor fora da lista é recusado na transmissão com "Código de Tributação inválido: &lt;valor&gt;". A lista de valores e a coluna **Significado nacional** da página só funcionam para filiais do Emissor Público Nacional: em Barcarena, a página informa que "o valor segue a tabela da prefeitura", mas vale a tabela acima.

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Docs. Pendentes**: transmite os documentos pendentes da filial num único lote, respeitando **Nº Máx. de Docs. por Lote WS**.
- **Transmitir Doc. Selecionado**: transmite só o documento selecionado.

Quando a ISSNet autoriza o lote, cada documento recebe na hora o **Nº NFS**, a **Chave de Acesso NFS-e** e a **Data NFS**, e o lote fica com **Status** **Sucesso**. Num lote com vários documentos, cada um recebe o número e a chave da sua própria DPS. A **Data NFS** é a data e a hora de processamento da nota na ISSNet; quando a resposta não a traz, vale a data de emissão da DPS e, sem nenhuma das duas, a data e a hora da transmissão. O campo **Cód. Verificação NFS** não é preenchido nesta integração: a nota é identificada pela chave de acesso.

A transmissão feita pela rotina automática de NFS-e (página **Movimentos Bot NFS-e**) tem o mesmo resultado: o lote criado pela rotina fica com **Status** **Sucesso** já no envio, sem consulta, e os movimentos dos documentos autorizados passam a **Finalizado**.

Quando a ISSNet rejeita o lote, as mensagens aparecem na tela, uma por linha, no formato `<código>: <mensagem> (<correção>)`. Nada fica gravado no Business Central: nenhum documento do lote é autorizado e todos continuam pendentes.

### Consultar o processamento

Não há consulta: o resultado já vem na transmissão. **Não use Consultar** em lotes de Barcarena, nem na página **NFS-e - Envio de Lote** nem na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`). Na página **NFS-e - Status de Lote RPS**, a consulta coloca o lote com **Status** **Erro**, inclusive um lote já **Sucesso**, com a mensagem "O serviço de PA-Barcarena-ISSNetNacional é síncrono e não necessita de consulta ao lote.".

Como a nota autorizada não tem **Cód. Verificação NFS**, o documento desse lote volta a aparecer como pendente na **NFS-e - Envio de Lote**. **Não o transmita de novo**: a nota já está autorizada. Para devolver o lote ao **Status** **Sucesso**:

1. Abra cada documento do lote no **Documento Fiscal Registrado** e use **Editar Campos**.
2. Preencha **Cód. Verificação NFS** com o próprio **Nº NFS** da nota. O **Nº NFS** e a **Data NFS** já estão preenchidos.
3. Na página **NFS-e - Status de Lote RPS**, selecione o lote e use **Marcar como Sucesso**.

### Corrigir e reenviar

Quando a transmissão é rejeitada, corrija o cadastro apontado pelas mensagens e transmita o documento novamente. O reenvio mantém o número e a série da DPS: como a ISSNet não guarda nada do lote rejeitado, não há duplicidade. Se for preciso usar outro número, a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote** atribui ao documento o próximo número de **Nº Série RPS/Temporário**.

### Validar o XML no validador da ISSNet

Quando a transmissão ou o cancelamento é recusado com a mensagem `E160: Arquivo em desacordo com o XML Schema. (Consulte o Manual da NFS-e para saber quais são as versões de XML Schema suportadas pelo sistema.)`, o XML enviado está fora do esquema da ISSNet, e a mensagem não diz qual campo saiu errado. Para descobrir, confira o XML no validador da ISSNet: [https://nfse.issnetonline.com.br/wsnfsenacional/homologacao/validarxml](https://nfse.issnetonline.com.br/wsnfsenacional/homologacao/validarxml). Ele compara o XML com o esquema do provedor e aponta o campo fora dele. Use-o também na homologação de uma filial nova: se a nota de teste for recusada com `E160`, valide o XML antes de liberar a produção.

Para abrir o validador, basta um certificado digital válido instalado no navegador. Não é preciso pedir ao suporte da ISSNet a liberação descrita em [Homologação](#homologação), que vale para emitir em homologação.

**Cabeçalho**: o validador pede, junto com o XML, o cabeçalho que a integração envia em cada mensagem. Ele é fixo: o mesmo em todas as notas, no envio e no cancelamento, em homologação e em produção, em Brasília, Goiânia e Barcarena. Copie-o como está:

```
<cabecalho versao="1.01" xmlns="http://www.sped.fazenda.gov.br/nfse"><versaoDados>1.01</versaoDados></cabecalho>
```

**XML a validar**: o XML no padrão da ISSNet é obtido no **Log de Comunicação da NFS-e** (veja [NFS-e: Log de Comunicação com a Prefeitura](Localization-for-Brazil_User-Manual_NFSe-Log-Comunicacao-Prefeitura.md)):

1. Na **Config. NF-e** da filial, aba **NFS-e**, grupo **Log**, ligue **Log de Comunicação**.
2. Transmita o documento de novo (no cancelamento, use **Cancelar NFS-e** de novo). Com o log ligado, a tentativa fica registrada mesmo quando é recusada.
3. Na página **NFS-e - Envio de Lote**, use **Log de Comunicação** (grupo **Lote**). A lista traz os arquivos da filial, do mais recente para o mais antigo.
4. Na tentativa recusada, selecione a linha com o **Nome do Arquivo** indicado na tabela abaixo e use **Download**.
5. No validador, cole o cabeçalho no campo do cabeçalho e o conteúdo do arquivo baixado no campo do XML.
6. Corrija o cadastro do campo apontado e transmita de novo. Ao terminar, desligue **Log de Comunicação**.

| **Operação** | Arquivo a validar | Outros arquivos da mesma tentativa |
| --- | --- | --- |
| **Enviar Lote de RPS** | `E_EnviarLoteRps` | `E_RecepcionarLoteDpsSincrono_Soap` (mensagem completa enviada à ISSNet, com o cabeçalho) e `R_EnviarLoteRps` (retorno da ISSNet) |
| **Cancelar NFS-e** | `E_CancelarNfse` | `E_CancelarNfse_Soap` e `R_CancelarNfse` |

### Primeira emissão em homologação

1. Solicite ao suporte da ISSNet o acesso ao ambiente de homologação (veja [Homologação](#homologação)).
2. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** ligado.
3. Confira os códigos de serviço, o código de tributação NFS-e e os cadastros da filial e do cliente.
4. Registre uma fatura de venda de serviço para a filial.
5. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**.
6. Corrija o que a rejeição da ISSNet apontar e transmita de novo. Numa recusa `E160`, valide o XML (veja [Validar o XML no validador da ISSNet](#validar-o-xml-no-validador-da-issnet)) antes de liberar a produção.
7. Com a nota autorizada, abra o **Documento Fiscal Registrado** e use **Abrir DANFSe** (veja a [seção 4](#4-danfse)).

### Cancelar

O cancelamento tem dois passos, no **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`):

1. **Cancelar Documento Fiscal** (grupo **Contabilização**): estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central. Nada é enviado à ISSNet neste passo.
2. **Cancelar NFS-e** (grupo **NFS-e**), habilitado depois do primeiro passo: envia à ISSNet o evento de cancelamento, com a chave de acesso da nota e o motivo "Erro na Emissão".

Quando a ISSNet registra o evento, a mensagem "NFS-e cancelada com sucesso." é exibida e o campo **NFS Cancelada** do documento é marcado. Se ela recusar (por exemplo, nota já cancelada), a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" lista os motivos e a NFS-e continua autorizada; corrija o que for apontado e use **Cancelar NFS-e** de novo.

---

## 4. DANFSe

Com **Utilizar Layout Padrão** ligado, o DANFSe da nota autorizada é gerado no padrão nacional a partir da chave de acesso, pelas ações **Abrir DANFSe** e **Atualizar DANFSe** do **Documento Fiscal Registrado**. O layout, a marca d'água de homologação e a de cancelamento estão descritos em [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md).

O Layout Padrão é a única forma de obter o DANFSe de Barcarena no Business Central: a ISSNet devolve a chave de acesso da nota, mas não um código de verificação nem uma imagem para download. Mantenha **Utilizar Layout Padrão** sempre ligado. Com o campo desligado:

- **Abrir DANFSe** termina com a mensagem de que **Cód. Verificação NFS** deve ter um valor;
- **Atualizar DANFSe** termina com "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e";
- o e-mail do documento fica pendente com o detalhe "Sem Cód. Verificação NFS: use o Layout Padrão", a menos que **NFS-e sem Anexo** (aba **E-Mails de Documento** da **Config. NF-e**) esteja ligado.

Ligue o campo e use **Atualizar DANFSe**.

A data e a hora de emissão impressas no DANFSe vêm da **Data NFS**, gravada na autorização.

---

## 5. Pontos de Atenção

- **Certificado vencido**: a transmissão e o cancelamento são bloqueados até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **Consultar**: não use em lotes desta integração (veja [Consultar o processamento](#consultar-o-processamento)).
- **Mostrar Somente Pendentes**: na página **NFS-e - Envio de Lote**, as notas já autorizadas continuam aparecendo com esse filtro ligado, porque não têm **Cód. Verificação NFS**. O **Status NFS-e** **Sucesso** e o **Nº NFS** indicam as autorizadas, e as ações de transmissão não as reenviam.
- **Lote recusado por inteiro**: um único documento com erro faz a ISSNet recusar todos os documentos do lote (veja **Nº Máx. de Docs. por Lote WS** na [seção 2](#configuração-nf-e-da-filial)).
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| Mensagem de campo obrigatório da filial (**Inscrição Municipal**, CEP, **Cód. Municipio IBGE**) | Completar a **Informação Filial** ou o cadastro de **CEPs** |
| "Nenhuma linha da nota fiscal possui Código de Serviço associado." | Preencher o **Cód. Serviço** na ficha do item de serviço |
| "Código de Tributação inválido: &lt;valor&gt;" | Ajustar o **Cód. Tributação** do código em **Códigos Tributação NFS-e** para `1`, `2`, `3` ou `4` |
| `E160: Arquivo em desacordo com o XML Schema. (Consulte o Manual da NFS-e para saber quais são as versões de XML Schema suportadas pelo sistema.)` | Validar o XML no [validador da ISSNet](#validar-o-xml-no-validador-da-issnet), que aponta o campo fora do esquema. Conferir também **Item Lista Serviço** só com dígitos, **Cód. Tributação** com 6 dígitos, número e série da DPS só com dígitos e, em obra, imóvel e evento, os códigos em branco (veja [Obras, imóveis e atividades/eventos](#obras-imóveis-e-atividadeseventos)) |
| Demais `<código>: <mensagem> (<correção>)` | Rejeição da ISSNet: corrigir o cadastro indicado na correção e transmitir de novo |
| "Resposta vazia do servidor ISSNet." | A ISSNet não respondeu. Transmitir de novo mais tarde |
| "O serviço retornou um status de erro por razões desconhecidas." | A resposta veio sem nota e sem mensagem. Transmitir de novo; se persistir, acionar o suporte |
| "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. ..." | Cadastrar o certificado renovado e selecioná-lo em **ID Certificado** |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "O serviço de PA-Barcarena-ISSNetNacional é síncrono e não necessita de consulta ao lote." | O lote foi consultado. Não retransmitir; seguir os passos de [Consultar o processamento](#consultar-o-processamento) |
| Mensagem de que **Cód. Verificação NFS** deve ter um valor, ao abrir o DANFSe | Ligar **Utilizar Layout Padrão** e usar **Atualizar DANFSe** |

- **Município de emissão**: o município emissor é o do CEP da filial. Um CEP com código IBGE errado envia a DPS para outro município.
- **Log de comunicação**: com **Log de Comunicação** ligado na **Config. NF-e** da filial, os arquivos enviados à ISSNet e os retornos dela ficam na página **Log de Comunicação da NFS-e**, inclusive nas transmissões e nos cancelamentos recusados (veja [Validar o XML no validador da ISSNet](#validar-o-xml-no-validador-da-issnet) e [NFS-e: Log de Comunicação com a Prefeitura](Localization-for-Brazil_User-Manual_NFSe-Log-Comunicacao-Prefeitura.md)). Com o log desligado, o Business Central guarda só os dados de retorno do documento autorizado.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Barcarena - PA (ISSNet Nacional)** (`PA-Barcarena-ISSNetNacional`) |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Homologação** | Conforme o ambiente |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **Nº Máx. de Docs. por Lote WS** | `0` ou um valor baixo |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Um dos dois preenchido |
| **Config. NF-e** | **Utilizar Layout Padrão** | Ligado |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Série de numeração só com dígitos |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Config. NF-e** | **Tamanho Máx. Discriminação** | `1000` |
| **Série Filial Tipo Doc. Fiscal** | **Série** | Preenchida, só com dígitos, até 5 posições |
| **Informação Filial** | **Inscrição Municipal** | Preenchida |
| **Códigos de Serviço NFS** | **Cód. Tributação** | Código de tributação nacional (6 dígitos) |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Código de tributação municipal de Barcarena, só com dígitos |
| **Códigos de Serviço NFS** | **Cód. NBS** | Código NBS |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | `1`, `2`, `3` ou `4` |
