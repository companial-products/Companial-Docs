<!-- PR: #274, #309 -->
# NFS-e: Emissão em Salvador - BA

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **Salvador - BA** emitem NFS-e pelo Business Central por meio do web service da prefeitura de Salvador, no padrão ABRASF. Na lista de cidades de NFS-e, a integração aparece como **Salvador - BA** (código `BA-Salvador`) e cobre a transmissão de lotes de RPS, a consulta do processamento e o reenvio após rejeição.

A autenticação é feita só pelo **certificado digital** da filial. A autorização não é imediata: a prefeitura recebe o lote, e o resultado (NFS-e autorizada ou mensagens de erro) só aparece na **consulta** do lote.

**Não estão disponíveis pelo Business Central**:

- **ambiente de homologação**;
- **cancelamento**: a prefeitura só cancela a NFS-e por processo administrativo;
- **DANFSe**: a nota é consultada e impressa no portal da prefeitura.

---

## 1. Contexto

### O que muda para a filial de Salvador

| Tema | Como funciona em Salvador - BA |
| --- | --- |
| **Layout** | Lote de RPS no padrão ABRASF da prefeitura de Salvador, assinado com o certificado digital da filial |
| **Autenticação** | Certificado digital da filial. Não há usuário, senha nem token |
| **Envio** | Um lote com um ou mais documentos. A prefeitura confirma o recebimento, mas as notas ainda não estão autorizadas |
| **Autorização** | Conhecida na consulta do lote, que grava o número da NFS-e, a data de emissão e o código de verificação |
| **Rejeição** | Na transmissão, a recusa aparece na tela. Na consulta, as mensagens da prefeitura ficam em **Lista Erros** do lote |
| **Cancelamento** | Só por processo administrativo junto à prefeitura. No Business Central, só o documento fiscal é cancelado |
| **DANFSe** | Não é gerado pelo Business Central. Consulte e imprima a nota no portal da prefeitura |
| **Ambientes** | Só produção. Não há homologação |

### Tributos enviados

- **ISS**:
  - alíquota, valor do ISS e indicação de ISS retido;
  - a base de cálculo é o valor dos serviços menos as deduções da base do ISS, e as deduções também são enviadas;
  - descontos condicionado e incondicionado.
- **Retenções federais**: INSS, IRRF e CSLL retidos, sempre informados. PIS e COFINS retidos, quando o documento tem alíquota de retenção desses tributos. Outras retenções.
- **Natureza da operação**: o **Cód. Tributação NFS** do documento (veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)).
- **Simples Nacional e incentivo cultural**: vão os campos **Opção pelo Simples** e **Incentivo Cultural** da **Config. NF-e**. O **Regime Especial de Tributação** não é enviado.
- **Serviço**: código de tributação do município, item da lista de serviços, CNAE, discriminação (até 2000 caracteres) e município da prestação.
- **IBS/CBS (Reforma Tributária)**: o RPS leva o código NBS, a classificação tributária de IBS/CBS da primeira linha do documento e o código indicador de operação. CST, tributação regular, diferimento, destinatário, imóvel e evento não são enviados. A configuração desses cadastros está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md).
- **Tributos aproximados (IBPT)**: não são enviados.
- **Tomador do exterior**: vai com o CEP, o município e a UF do exterior (`EX`). O NIF não é enviado.

---

## 2. Configuração

### Certificado digital

O certificado digital é a única credencial de Salvador: ele autentica a conexão com o web service da prefeitura e assina o lote de RPS.

1. Use o certificado digital A1 (arquivo `.pfx`) da empresa. Antes de configurar a filial, confirme com a prefeitura de Salvador que o contribuinte está habilitado a emitir NFS-e por web service.
2. Cadastre o certificado em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**).
3. Selecione-o no campo **ID Certificado** da aba **NFS-e** (veja a tabela abaixo).

O Business Central avisa quando o certificado está perto de vencer. Quando ele já venceu, a transmissão e a consulta são **bloqueadas** com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação

Salvador **não tem ambiente de homologação** pela integração: toda transmissão vai para produção e gera uma NFS-e válida. Com **Homologação** ligado, a transmissão e a consulta falham com "Não existe ambiente de homologação para a cidade BA-Salvador".

Como a NFS-e de Salvador só pode ser cancelada por processo administrativo, faça a primeira emissão com cuidado:

- confira antes os códigos de serviço, o código de tributação NFS-e, a **Nº Série RPS/Temporário** e os cadastros da filial e do cliente;
- transmita uma única nota real, com **Transmitir Doc. Selecionado**, e consulte o lote;
- confira a nota no portal da prefeitura antes de transmitir as demais.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para Salvador - BA |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Salvador - BA** na lista (código `BA-Salvador`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (`ABRASF`). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial |
| **Configuração Envio** | **Homologação** | **Desligado** |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. A prefeitura não informa o último número de RPS usado; com o campo desligado, toda transmissão falha |
| **Configuração Envio** | **Nome Usuário WS** e **Senha WS** | Não usados por Salvador |
| **Configuração Envio** | **WS NFS-e informa o IM** | Não usado por Salvador: a inscrição municipal da filial é sempre enviada |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão |
| **Configuração Envio** | **ID Relatório p/ TXT** | Em branco. É usado só por filial que não emite por web service |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | `0` (sem limite) ou o número máximo de documentos que deseja em cada lote |
| **Lote** | **Usar Data/Hora como Nº Lote** e **Nº Série Lote** | O número do lote vai à prefeitura e precisa ter **só dígitos**. Ligue **Usar Data/Hora como Nº Lote** ou informe em **Nº Série Lote** uma série de numeração só com dígitos |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `2000`. A prefeitura aceita até 2000 caracteres na discriminação dos serviços; com o limite, o sistema avisa já no registro da venda |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Desligado**. O Layout Padrão depende da chave de acesso, que Salvador não devolve (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Desligado. Afeta só o DANFSe do Layout Padrão, que não é usado em Salvador |
| — | **Nº Série RPS/Temporário** | **Obrigatório para a consulta**: série só com dígitos, com o **Último Nº Usado** preenchido e o mesmo comprimento da numeração fiscal (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente. Em branco, ele precisa ser informado em cada venda |
| — | **NFS-e suporta IBPT** | Desligado automaticamente ao escolher Salvador. Os tributos aproximados não são enviados |
| — | **Opção pelo Simples** | Ligado se a filial é optante pelo Simples Nacional. Vai no RPS |
| — | **Incentivo Cultural** | Ligado se a filial é incentivadora cultural. Vai no RPS |
| — | **Regime Especial de Tributação** | Não enviado a Salvador |

### Numeração e série do RPS

- **Número do RPS**: é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão.
  - A numeração vem da série do tipo de documento fiscal: o **Nº Série Fiscal** da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, o **Nº Série Nota Fiscal** do próprio tipo de documento fiscal.
  - Salvador aceita **só dígitos** no número do RPS. Use, por exemplo, `000031` e não `NFS000031`.
- **Série**: a **Série** do documento vai no RPS como está cadastrada na série por filial ou no tipo de documento fiscal.
- **Nº Série RPS/Temporário**: é usada pela ação **Criar Novo Nº** e é **exigida pela consulta**. Para a consulta funcionar, a série precisa:
  - estar preenchida na **Config. NF-e**;
  - ter o **Último Nº Usado** preenchido na linha da série;
  - gerar números com **o mesmo comprimento** da numeração fiscal. Por exemplo, se as notas são numeradas `000123`, a série RPS também precisa ter 6 dígitos, como `000001`.

  A consulta usa esse comprimento para localizar o documento a partir do número de RPS devolvido pela prefeitura. Com comprimento diferente, o lote fica **Sucesso**, mas o documento não recebe o número da NFS-e.

O número da NFS-e devolvido pela prefeitura é gravado no **Nº NFS** sem os zeros à esquerda.

### Filial e cliente

Estes cadastros são conferidos durante a transmissão. A transmissão para na primeira pendência, com a mensagem de campo obrigatório do Business Central, que indica o campo e o cadastro a corrigir:

- **Informação Filial**: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos. A inscrição municipal vai à prefeitura só com os dígitos.
- **CEPs**: o CEP da filial e o do cliente cadastrados, com **Cód. Municipio IBGE**.
- **Cliente**: categoria, CEP, cidade, bairro, endereço, número, UF e país preenchidos (exceto pessoa física e cliente não identificado).
  - A inscrição municipal do cliente vai no RPS quando o cliente é do mesmo município da filial, ou quando ele tem **Outras Inscrições** do tipo C.C.M. para o município da filial.
  - Cliente pessoa física sem CPF vai com o CPF zerado.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Cód. Tributação** | O código de tributação do município de Salvador para o serviço. Pontos são descartados no envio |
| **Item Lista Serviço** | O item da lista de serviços (LC 116) do serviço. Vai como está cadastrado |
| **Código CNAE** | O CNAE do serviço, **só com dígitos** (por exemplo, `6201501`). Com pontos, traços ou barras, a transmissão é recusada com um erro técnico de formato do serviço de integração |
| **Cód. NBS** | Código NBS do serviço. Vai sem pontos |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS (6 dígitos). Para o NBS `1.0605.40.00`, o RPS leva sempre o código `050201` |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos **Cód. Tributação**, **Item Lista Serviço** e **Código CNAE** desse registro substituem os do código de serviço, **mesmo que estejam em branco**. Preencha o **Código CNAE** também no registro da filial.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item. Sem nenhuma linha assim, a transmissão para com "Nenhuma linha da nota fiscal possui Código de Serviço associado.".

### Códigos Tributação NFS-e

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. Em Salvador, o campo **Cód. Tributação** desse código é a **natureza da operação** do RPS:

- informe o número da natureza da operação conforme a tabela da prefeitura;
- um valor que não seja número vai como natureza da operação zerada.

A lista de valores nacionais e a coluna **Significado nacional** valem só para filiais do Emissor Público Nacional e não se aplicam a Salvador.

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Docs. Pendentes**: transmite os documentos pendentes da filial num único lote, com até **Nº Máx. de Docs. por Lote WS** documentos. Se houver mais pendentes que o máximo, a página avisa que só os primeiros serão enviados.
- **Transmitir Doc. Selecionado**: transmite só o documento selecionado.

Antes do envio, a transmissão para nestas situações:

| Situação | O que corrigir |
| --- | --- |
| "Não existe ambiente de homologação para a cidade BA-Salvador" | Desligar **Homologação** |
| Filial sem nome, CNPJ, **Inscrição Municipal**, cidade ou CEP | **Informação Filial** |
| CEP da filial ou do cliente sem **Cód. Municipio IBGE** | **CEPs** |
| Filial sem **ID Certificado**, ou certificado vencido | **ID Certificado** na **Config. NF-e** |
| Cliente com endereço incompleto | Ficha do cliente |
| "Nenhuma linha da nota fiscal possui Código de Serviço associado." | Ficha do item: tipo Serviço e **Cód. Serviço** |
| Documento sem **Município Prestação Serviço** ou sem **Cód. Tributação NFS** | **Documento Fiscal Registrado**, grupo **NFS**, ação **Editar Campos** (veja [Editar Campos do Documento Fiscal Registrado](Localization-for-Brazil_User-Manual_Editar-Campos-Documento-Fiscal-Registrado.md)) |
| "Discriminação dos serviços excedeu o limite de 2000 caracteres. Total: &lt;n&gt;" | Reduzir a discriminação: revise a descrição das linhas, os textos adicionais do documento e as opções do grupo **Discriminação Serviços** da **Config. NF-e** |
| Erro técnico de formato do serviço de integração | Deixar o **Código CNAE** do código de serviço (e do registro da filial) só com dígitos |
| "Lote &lt;nº&gt; deverá ser consultado antes do envio." | O documento está num lote ainda **Enviado**: consulte o lote (veja [Consultar o processamento](#consultar-o-processamento)) |

Quando a prefeitura aceita o lote, ele fica com **Status** **Enviado**, e os documentos com **Status NFS-e** **Enviado**. As notas **ainda não estão autorizadas**: o número da NFS-e só chega na consulta.

Quando a prefeitura recusa o lote, as mensagens dela aparecem na tela, uma por linha, e nada é gravado: o lote não é criado e os documentos continuam pendentes. Se a prefeitura recusar sem mensagem, aparece "O serviço retornou um status de erro por razões desconhecidas.".

### Consultar o processamento

Depois da transmissão, use **Consultar** em uma destas páginas para obter o resultado:

- **NFS-e - Envio de Lote**: selecione os documentos;
- **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`): selecione o lote.

A consulta exige a **Nº Série RPS/Temporário** configurada (veja [Numeração e série do RPS](#numeração-e-série-do-rps)).

| Resultado da consulta | O que acontece no Business Central |
| --- | --- |
| **Notas autorizadas** | O lote fica com **Status** **Sucesso**. Cada documento recebe o **Nº NFS**, a **Data NFS** e o **Cód. Verificação NFS**, e o **Nº Nota Fiscal** passa a ser o número da NFS-e |
| **Mensagens da prefeitura** | O lote fica com **Status** **Erro**. As mensagens ficam em **Lista Erros**, com o texto de cada mensagem da prefeitura (até 250 caracteres por linha), e os documentos voltam a ficar pendentes |
| **Falha de comunicação** | O lote fica com **Status** **Erro**, com a descrição da falha em **Lista Erros**. Use **Consultar** de novo antes de reenviar os documentos |

### Corrigir e reenviar

Quando o lote fica com **Status** **Erro** por mensagem da prefeitura, corrija o cadastro apontado e transmita os documentos de novo. Eles vão para um lote novo, com o mesmo **Nº RPS**.

Se a prefeitura recusar o número do RPS, a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote** atribui ao documento o próximo número de **Nº Série RPS/Temporário**.

Quando o **Erro** vem de uma falha de comunicação na consulta, não reenvie: consulte o lote de novo. A prefeitura pode já ter autorizado as notas, e o reenvio criaria RPS em duplicidade.

### Cancelar

A NFS-e de Salvador **não é cancelada pelo Business Central**: a prefeitura só cancela a nota por **processo administrativo**.

1. Solicite o cancelamento da NFS-e à prefeitura de Salvador, por processo administrativo.
2. No **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`), use **Cancelar Documento Fiscal** (grupo **Contabilização**). A ação estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central; nada é enviado à prefeitura.

A ação **Cancelar NFS-e** termina com a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:", seguida de "Cancelamento somente por processo administrativo". O campo **NFS Cancelada** do documento não é marcado, nem nesse caso nem quando a nota é cancelada na prefeitura, e o Business Central não tem ação para marcá-lo manualmente.

---

## 4. DANFSe

O Business Central não gera nem baixa o DANFSe das notas de Salvador. **Consulte e imprima a nota no portal da prefeitura**, usando o **Nº NFS** e o **Cód. Verificação NFS** gravados no documento.

- **Abrir DANFSe** e **Atualizar DANFSe**, no **Documento Fiscal Registrado**, terminam com a mensagem "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e".
- Mantenha **Utilizar Layout Padrão** desligado. O Layout Padrão depende da chave de acesso nacional, que Salvador não devolve: com o campo ligado, **Abrir DANFSe** termina com a mensagem que começa por "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe", e **Imprimir** de uma nota autorizada para pedindo a **Chave de Acesso NFS-e**.
- Com o Layout Padrão desligado, a ação **Imprimir** da página **NFS-e - Envio de Lote** imprime o RPS do documento.

---

## 5. Pontos de Atenção

- **Cancelamento só por processo administrativo**: confira os cadastros antes de transmitir; uma nota emitida com erro não pode ser cancelada pelo sistema.
- **Discriminação dos serviços**: com **Tamanho Máx. Discriminação** em `2000`, o registro da venda é barrado com "O tamanho da discriminação (&lt;n&gt;) excede o máximo permitido (2000).". Sem o limite, o documento é registrado e a transmissão é recusada.
- **Código CNAE**: só dígitos, no código de serviço e no registro da filial.
- **Certificado vencido**: a transmissão e a consulta são bloqueadas até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **Nº Série RPS/Temporário com comprimento diferente da numeração fiscal**: o lote consultado fica **Sucesso**, mas os documentos ficam sem **Nº NFS**.
  - Ajuste a série (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) e consulte o lote de novo, pela página **NFS-e - Status de Lote RPS**.
- **E-mail de documento**: como o DANFSe não sai pelo Business Central, o e-mail do documento fiscal fica pendente, aguardando a NFS-e. Para enviar o e-mail sem o DANFSe, ligue **NFS-e sem Anexo** na aba **E-Mails de Documento** da **Config. NF-e**.
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| Mensagens da prefeitura na tela, ao transmitir | Corrigir o cadastro indicado e transmitir de novo |
| "O serviço retornou um status de erro por razões desconhecidas." | A prefeitura recusou o lote sem mensagem. Conferir os cadastros e o certificado e transmitir de novo |
| Mensagens da prefeitura em **Lista Erros**, depois da consulta | Corrigir o cadastro indicado e transmitir os documentos de novo |
| "Discriminação dos serviços excedeu o limite de 2000 caracteres. Total: &lt;n&gt;" | Reduzir a discriminação e usar `2000` em **Tamanho Máx. Discriminação** |
| "O tamanho da discriminação (&lt;n&gt;) excede o máximo permitido (2000)." | Reduzir a discriminação da venda antes de registrá-la |
| Erro técnico de formato do serviço de integração ao transmitir | Deixar o **Código CNAE** só com dígitos |
| "Lote &lt;nº&gt; deverá ser consultado antes do envio." | Consultar o lote antes de reenviar o documento |
| Erro de campo obrigatório em **Nº Série RPS/Temporário** ou no **Último Nº Usado**, ao consultar | Configurar a série (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) |
| "Não existe ambiente de homologação para a cidade BA-Salvador" | Desligar **Homologação** |
| "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. ..." | Cadastrar o certificado renovado e selecioná-lo em **ID Certificado** |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e" | O DANFSe não sai pelo Business Central: consultar e imprimir no portal da prefeitura |
| "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe. ..." | Desligar **Utilizar Layout Padrão** |
| "Os seguintes erros ocorreram no cancelamento da NFS-e: Cancelamento somente por processo administrativo" | Solicitar o cancelamento à prefeitura por processo administrativo e usar só **Cancelar Documento Fiscal** |

- **Log de comunicação**: o envio e o retorno de cada operação com a prefeitura ficam registrados no log do serviço de integração, consultado pelo suporte. No Business Central ficam as mensagens do lote e os dados de retorno do documento.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Salvador - BA** (`BA-Salvador`) |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Homologação** | Desligado |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **Nº Máx. de Docs. por Lote WS** | `0` ou o máximo desejado |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Número de lote só com dígitos |
| **Config. NF-e** | **Tamanho Máx. Discriminação** | `2000` |
| **Config. NF-e** | **Utilizar Layout Padrão** | Desligado |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Série só com dígitos, com **Último Nº Usado** e o comprimento da numeração fiscal |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Config. NF-e** | **Opção pelo Simples**, **Incentivo Cultural** | Conforme a filial |
| **Série Filial Tipo Doc. Fiscal** | **Nº Série Fiscal** | Série de numeração só com dígitos |
| **Informação Filial** | **Inscrição Municipal** | Preenchida |
| **Códigos de Serviço NFS** | **Cód. Tributação** | Código de tributação do município de Salvador |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Item da lista de serviços (LC 116) |
| **Códigos de Serviço NFS** | **Código CNAE** | CNAE só com dígitos (também em **Códigos Serviço NFS por Filial**) |
| **Códigos de Serviço NFS** | **Cód. NBS** | Código NBS do serviço |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | Natureza da operação, conforme a tabela da prefeitura |
