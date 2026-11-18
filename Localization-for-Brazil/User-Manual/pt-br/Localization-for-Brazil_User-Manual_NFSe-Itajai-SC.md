<!-- PR: #274, #309 -->
# NFS-e: Emissão em Itajaí - SC

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **Itajaí - SC** emitem a NFS-e pelo Business Central por meio do web service da prefeitura, no padrão ABRASF do manual de integração de Itajaí (versão 7.4, compatível com a NFS-e nacional e com a Reforma Tributária). Na lista de cidades de NFS-e, a integração aparece como **Itajaí - SC** (código `SC-Itajai`); até a próxima atualização do serviço de integração, a mesma opção ainda pode aparecer como **Santa Catarina - SC**, com o mesmo código. A integração cobre transmissão, consulta do processamento, reenvio após rejeição e cancelamento.

A autenticação é feita só pelo **certificado digital** da filial. A autorização não é imediata: a prefeitura recebe o lote de RPS e devolve um protocolo, e o número da NFS-e só chega na **consulta** do lote. Não há ambiente de homologação no Business Central e o DANFSe não é gerado pelo sistema: a nota é consultada e impressa no portal da prefeitura.

---

## 1. Contexto

### O que muda para a filial de Itajaí

| Tema | Como funciona em Itajaí - SC |
| --- | --- |
| **Layout** | RPS no padrão ABRASF da prefeitura de Itajaí, assinado com o certificado digital da filial |
| **Autenticação** | Certificado digital da filial. Não há usuário, senha nem token |
| **Código do serviço** | **Item Lista Serviço** com 6 dígitos, sem pontos (item da lista de serviços do município) |
| **Natureza da operação** | O **Cód. Tributação NFS** do documento vai como natureza da operação: código numérico de até 3 dígitos, da tabela do município |
| **Envio** | Vários RPS por lote. A prefeitura confirma o recebimento com um protocolo, mas a nota ainda não está autorizada |
| **Autorização** | Conhecida na consulta do lote: grava o número da NFS-e, a data e o código de verificação |
| **Rejeição** | A prefeitura valida o lote inteiro: um RPS com erro faz o lote todo ser rejeitado. As mensagens ficam em **Lista Erros** |
| **Cancelamento** | Pelo sistema, com o motivo "Outros" (C999) fixo, para notas emitidas no ano corrente |
| **DANFSe** | Não é gerado no Business Central. Consulte e imprima a nota no portal da prefeitura |
| **Ambientes** | Só produção. A prefeitura tem um ambiente de testes, que esta integração não usa |

### Tributos enviados

- **ISS**: alíquota, valor e base de cálculo (valor dos serviços menos as deduções). Quando o documento tem ISS retido, o RPS sai com ISS retido e o valor retido.
- **Retenções federais**: valores de PIS, COFINS, INSS e IR. O campo de CSLL leva a soma de CSLL, PIS e COFINS retidos. Quando o documento tem PIS ou COFINS retidos com alíquota, segue também o grupo de tributos federais: base, alíquotas, CST do PIS/COFINS e tipo de retenção. Também vão outras retenções, descontos e deduções.
- **Simples Nacional e incentivo cultural**: vêm dos campos **Opção pelo Simples** e **Incentivo Cultural** da **Config. NF-e**. O **Regime Especial de Tributação** não é enviado.
- **IBS/CBS (Reforma Tributária)**: quando a linha do documento tem classificação tributária de IBS/CBS, o RPS leva o município de incidência e as alíquotas de IBS estadual, IBS municipal e CBS, cada uma com o seu percentual de redução. O layout de Itajaí não tem CST, classificação tributária nem código indicador de operação: esses dados não são enviados. O panorama por cidade está em [Reforma Tributária - Suporte à Emissão de Notas de Serviço](Localization-for-Brazil_User-Manual_Reforma-Acompanhamento-Emissao-NotaServico.md).
- **Tomador estrangeiro**: NIF e as informações de comércio exterior.
- **Tributos aproximados (IBPT)**: não são enviados.

---

## 2. Configuração

### Certificado digital e autorização da prefeitura

O certificado digital é a única credencial de Itajaí: ele assina o lote de RPS, a consulta e o pedido de cancelamento, e essa assinatura é a autenticação perante a prefeitura.

1. Pelo manual de integração da prefeitura, o certificado deve ser emitido por autoridade certificadora da ICP-Brasil, de pessoa física ou jurídica, conter o CNPJ do estabelecimento emissor (ou da matriz) ou o CPF do prestador, e permitir assinatura digital. No Business Central, use o certificado A1 (arquivo `.pfx`) da empresa.
2. Cadastre o certificado em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**) e selecione-o no campo **ID Certificado** da aba **NFS-e**.
3. Antes da primeira emissão, confirme com a prefeitura que o CNPJ da filial está autorizado a emitir NFS-e por web service. Sem a autorização, a prefeitura rejeita o lote com o erro E156 ("CNPJ do prestador não autorizado a emitir NFS-e"). Ela também recusa RPS com data anterior à habilitação do prestador (E17).

O Business Central avisa quando o certificado está perto de vencer e **bloqueia a transmissão** quando ele já venceu, com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação

Não há homologação para Itajaí no Business Central. A prefeitura mantém um ambiente de testes, mas esta integração transmite sempre ao ambiente de produção. Com o campo **Homologação** ligado, a transmissão, a consulta e o cancelamento falham com a mensagem "Não existe ambiente de homologação para a cidade SC-Itajai". Mantenha o campo desligado.

Como a primeira nota já é real, confira os cadastros antes de transmitir e faça o primeiro envio com um único documento (veja [Primeira emissão](#primeira-emissão)).

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para Itajaí - SC |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Itajaí - SC** na lista (código `SC-Itajai`). Enquanto a lista ainda mostrar **Santa Catarina - SC**, escolha essa opção: o código é o mesmo |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (ABRASF). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial |
| **Configuração Envio** | **Homologação** | **Desligado**. Itajaí não tem homologação no sistema |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. A prefeitura não informa o último número usado; desligado, toda transmissão falha com "Erro ao consultar último número de RPS: ..." |
| **Configuração Envio** | **WS NFS-e informa o IM** | Não usado: a inscrição municipal da filial é sempre enviada |
| **Configuração Envio** | **Nome Usuário WS** e **Senha WS** | Não usados por Itajaí |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão: é o endereço do serviço de integração da Localização, usado por todas as cidades |
| **Configuração Envio** | **ID Relatório p/ TXT** | Em branco. Só é usado por filial que gera a NFS-e por arquivo, sem web service |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | Opcional (`0` = sem limite). Como a prefeitura rejeita o lote inteiro quando um RPS tem erro, lotes menores reduzem o retrabalho |
| **Lote** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Um dos dois. O número do lote é enviado à prefeitura, que recusa número de lote repetido (E163) |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Desligado**. A prefeitura não devolve chave de acesso, e o Layout Padrão não tem como gerar o DANFSe (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Não usado (só afeta o DANFSe do Layout Padrão) |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `2000`. A discriminação acima de 2000 caracteres é recusada na transmissão; com o limite, o sistema avisa já no registro da venda |
| — | **Nº Série RPS/Temporário** | Série com **Último Nº Usado** e números do mesmo comprimento da numeração fiscal (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente |
| — | **NFS-e suporta IBPT** | Desligado automaticamente ao escolher Itajaí. Os tributos aproximados não são enviados |
| — | **Opção pelo Simples** | Conforme o cadastro da filial na prefeitura. Divergência com o cadastro municipal é rejeitada (E328) |
| — | **Incentivo Cultural** | Conforme a situação da filial |
| — | **Regime Especial de Tributação** | Não enviado |

### Numeração e série do RPS

- O número do RPS é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão. Ele vem da série do tipo de documento fiscal: **Nº Série Fiscal** da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, do próprio tipo de documento fiscal. Use números só com dígitos, com até 15 posições.
- A **Série** do tipo de documento fiscal precisa estar preenchida e ter até 5 caracteres. A prefeitura só aceita as séries permitidas para o prestador: confirme com ela a série a usar (erros E97 e E150).
- A **Consulta** exige **Nº Série RPS/Temporário** preenchida e com **Último Nº Usado**. O sistema usa o comprimento desse número para localizar o documento devolvido pela prefeitura. Por isso, a série precisa gerar números com o mesmo formato e o mesmo comprimento do **Nº Nota Fiscal** (por exemplo, os dois com 6 dígitos). A mesma série é usada pela ação **Criar Novo Nº**.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Item Lista Serviço** | O item da lista de serviços do município, com **6 dígitos e sem pontos**, como o manual da prefeitura pede (por exemplo, `140101` para o item 14.01.01). O item `990101` é o de serviço sem incidência de ISS. O valor vai à prefeitura exatamente como está no cadastro |
| **Cód. NBS** | Código NBS do serviço, só com dígitos e pontos. Enviado quando preenchido |
| **Cód. Tributação** | Não enviado a Itajaí |
| **Código CNAE** | Não enviado a Itajaí |
| **Cód. Operação de Serviços** | Não enviado: o layout de Itajaí não tem o código indicador de operação |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos desse registro substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item.

### Natureza da operação (Códigos Tributação NFS-e)

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. Em Itajaí, o campo **Cód. Tributação** desse código vai à prefeitura como a **natureza da operação**: informe o código numérico, de até 3 dígitos, da natureza de operação vigente no município. Peça à prefeitura a tabela de naturezas de operação.

Um valor que não seja numérico vai como zero e a prefeitura rejeita o RPS ("Natureza da operação não informada", E3, ou "Natureza da operação inválida", E144). A natureza também precisa ser coerente com o ISS retido (E169, E173, E174) e com o município de prestação (E110, E170 a E172).

Marque como **Padrão** a natureza mais usada pela filial: ela preenche automaticamente o **Cód. Tributação NFS-e** do pedido de venda e o **Cód. Tributação NFS** do documento.

### Filial e cliente

- **Informação Filial** da filial: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos; o CEP precisa ter **Cód. Municipio IBGE** em **CEPs**. A inscrição municipal é sempre enviada e precisa estar vinculada ao CNPJ na prefeitura (E43, E142).
- **Cliente**: CPF ou CNPJ. Para pessoa jurídica, a prefeitura exige endereço, número, bairro, cidade e UF (E56, E59, E113 a E115). A inscrição municipal do cliente só é enviada quando ele é do mesmo município da filial, como a prefeitura exige (E53).
- **Município Prestação Serviço** do documento: é o município de incidência do ISS enviado à prefeitura.

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Docs. Pendentes**: transmite os documentos pendentes da filial num único lote, com até **Nº Máx. de Docs. por Lote WS** documentos. Se houver mais pendentes que o máximo, o sistema pergunta se deseja enviar só os primeiros; os demais ficam para a próxima transmissão.
- **Transmitir Doc. Selecionado**: transmite só o documento selecionado.

A transmissão é barrada, com a mensagem na tela, quando:

| Situação | Mensagem |
| --- | --- |
| Certificado vencido | "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir." |
| **Homologação** ligado | "Não existe ambiente de homologação para a cidade SC-Itajai" |
| Discriminação com mais de 2000 caracteres | "Discriminação dos serviços excedeu o limite de 2000 caracteres. Total: &lt;quantidade&gt;" |
| Documento em lote ainda **Enviado** | "Lote &lt;nº&gt; deverá ser consultado antes do envio." |
| Nenhuma linha com item de serviço com código | "Nenhuma linha da nota fiscal possui Código de Serviço associado." |

Quando a prefeitura recebe o lote, ele fica com **Status** **Enviado** e os documentos ficam com **Status NFS-e** **Enviado**. A nota **ainda não está autorizada**: o número da NFS-e só chega na consulta.

Se a prefeitura recusar o lote na recepção, as mensagens dela aparecem na tela e nada fica gravado: corrija e transmita de novo.

### Consultar o processamento

Na página **NFS-e - Envio de Lote** (selecione o documento) ou na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`, selecione o lote), use **Consultar**.

| Resultado da consulta | O que acontece no Business Central |
| --- | --- |
| **Autorizado** | O lote fica com **Status** **Sucesso**. Cada documento recebe o **Nº NFS**, a **Data NFS** e o **Cód. Verificação NFS**, e o **Nº Nota Fiscal** passa a ser o número da NFS-e. O número é gravado sem o ano: a prefeitura numera a nota com o ano mais 11 dígitos e orienta usar só a parte sequencial na impressão |
| **Rejeitado** | O lote fica com **Status** **Erro** e as mensagens da prefeitura ficam em **Lista Erros**, só com o texto da mensagem. Como a prefeitura rejeita o lote inteiro, todos os documentos do lote voltam a ficar pendentes |

### Corrigir e reenviar

Quando o lote fica com **Status** **Erro**, corrija o cadastro apontado pelas mensagens e transmita os documentos novamente. Eles vão para um lote novo, com o mesmo **Nº RPS**.

Se a prefeitura informar que o RPS já existe ("RPS já informado", E10), use **Criar Novo Nº** na página **NFS-e - Envio de Lote** para dar ao documento o próximo número de **Nº Série RPS/Temporário** e transmita de novo.

### Primeira emissão

Como não há homologação, a primeira emissão já é uma nota real:

1. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** desligado.
2. Confira o item da lista de serviço, a natureza da operação, a série do RPS e os cadastros da filial e do cliente.
3. Registre uma fatura de venda de serviço para a filial.
4. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**.
5. Depois que a prefeitura processar o lote, use **Consultar** e confira o resultado.
6. Com a nota autorizada, confira-a no portal da prefeitura.

### Cancelar

O cancelamento tem dois passos, no **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`):

1. **Cancelar Documento Fiscal** (grupo **Contabilização**): estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central. Nada é enviado à prefeitura neste passo.
2. **Cancelar NFS-e** (grupo **NFS-e**), habilitado depois do primeiro passo: envia à prefeitura o pedido de cancelamento da NFS-e, com o código de cancelamento **C999** ("Outros") e a descrição "Erro de escrituracao". Os demais motivos da tabela da prefeitura (C001 a C005) não são usados.

Quando a prefeitura cancela a nota, a mensagem "NFS-e cancelada com sucesso." é exibida e o campo **NFS Cancelada** do documento é marcado. Se ela recusar, a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" traz as mensagens da prefeitura e a NFS-e continua autorizada.

O pedido de cancelamento identifica a NFS-e pelo **ano corrente**. Uma nota emitida em ano anterior não é localizada pela prefeitura: cancele-a no portal da prefeitura e, no Business Central, faça só o primeiro passo. O mesmo vale quando a prefeitura só aceita o cancelamento por processo administrativo (prazo de cancelamento excedido, E165, ou nota com crédito, E6). A nota cancelada no portal continua com **NFS Cancelada** desmarcado no Business Central.

---

## 4. DANFSe

O DANFSe de Itajaí não é gerado nem baixado pelo Business Central: **consulte e imprima a nota no portal da prefeitura**. No **Documento Fiscal Registrado**, a ação **Abrir DANFSe** termina com a mensagem "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e".

Mantenha **Utilizar Layout Padrão** desligado. Como a prefeitura não devolve chave de acesso, com o campo ligado **Abrir DANFSe** termina com a mensagem que começa por "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe".

Se a filial envia o documento fiscal por e-mail (aba **E-Mails de Documento** da **Config. NF-e**), ligue **NFS-e sem Anexo**. Desligado, o e-mail fica pendente esperando a imagem da NFS-e (**Detalhes** "Aguardando NFS-e").

---

## 5. Pontos de Atenção

- **Certificado vencido**: a transmissão, a consulta e o cancelamento são bloqueados até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **Lote rejeitado por inteiro**: um RPS com erro faz a prefeitura rejeitar todo o lote. Se a filial transmite muitos documentos de uma vez, um **Nº Máx. de Docs. por Lote WS** menor limita o reenvio.
- **Lote com Status Sucesso e documento sem Nº NFS**: o número devolvido pela prefeitura não foi localizado porque **Nº Série RPS/Temporário** tem comprimento diferente da numeração fiscal. Ajuste a série e use **Consultar** de novo na página **NFS-e - Status de Lote RPS**.
- **Nota de ano anterior**: cancele no portal da prefeitura (veja [Cancelar](#cancelar)).
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| "Não existe ambiente de homologação para a cidade SC-Itajai" | Desligar **Homologação** |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "Discriminação dos serviços excedeu o limite de 2000 caracteres. ..." | Reduzir a discriminação; usar **Tamanho Máx. Discriminação** = `2000` |
| E43, E141 ou E142 (inscrição municipal do prestador não encontrada, não informada ou não vinculada ao CNPJ) | Conferir **Inscrição Municipal** e CNPJ em **Informação Filial** |
| E156 (CNPJ do prestador não autorizado a emitir NFS-e) | Pedir à prefeitura a autorização de emissão para o CNPJ da filial |
| E1 (assinatura não confere) | Conferir o certificado selecionado em **ID Certificado** |
| E3, E144 ou E175 (natureza da operação não informada, inválida ou não autorizada) | Corrigir o **Cód. Tributação** do código em **Códigos Tributação NFS-e** (natureza numérica do município) |
| E30 ou E31 (item da lista de serviço inexistente ou não informado) | **Item Lista Serviço** com 6 dígitos, sem pontos |
| E8, E146, E328 ou E34 (opção pelo Simples Nacional inválida, fora do cadastro ou incompatível com a natureza) | Conferir **Opção pelo Simples** e o cadastro da filial na prefeitura |
| E29, E37, E39, E173 ou E174 (retenção de ISS não permitida ou obrigatória) | Revisar o ISS retido do documento e a natureza da operação |
| E10 (RPS já informado) ou E97/E150 (série do RPS incorreta ou não informada) | **Criar Novo Nº** e transmitir de novo; ajustar a **Série** com a prefeitura |
| E56, E59, E113, E114 ou E115 (endereço, cidade, número, bairro ou UF do cliente pessoa jurídica não informado) | Completar o endereço na ficha do cliente |

- **Log de comunicação**: o envio e o retorno de cada operação com a prefeitura ficam registrados no log do serviço de integração, consultado pelo suporte. No Business Central ficam as mensagens do lote e os dados de retorno do documento.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Itajaí - SC** (`SC-Itajai`), ainda exibido como **Santa Catarina - SC** até a atualização do serviço |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Homologação** | Desligado |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Um dos dois, sem repetir número de lote |
| **Config. NF-e** | **Utilizar Layout Padrão** | Desligado |
| **Config. NF-e** | **Tamanho Máx. Discriminação** | `2000` |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Com **Último Nº Usado**, mesmo comprimento da numeração fiscal |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Config. NF-e** | **Opção pelo Simples** e **Incentivo Cultural** | Conforme o cadastro na prefeitura |
| **Config. NF-e** | **NFS-e sem Anexo** (aba **E-Mails de Documento**) | Ligado, se a filial envia o documento por e-mail |
| **Série Filial Tipo Doc. Fiscal** | **Série** | Série aceita pela prefeitura, até 5 caracteres |
| **Informação Filial** | **Inscrição Municipal** | Preenchida |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Item da lista de serviços com 6 dígitos, sem pontos |
| **Códigos de Serviço NFS** | **Cód. NBS** | Código NBS, quando houver |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | Natureza da operação do município (numérica, até 3 dígitos) |
