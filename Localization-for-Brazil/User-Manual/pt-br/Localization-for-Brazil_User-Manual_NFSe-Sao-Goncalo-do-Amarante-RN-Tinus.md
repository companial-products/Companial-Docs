<!-- PR: #274, #309 -->
# NFS-e: Emissão em São Gonçalo do Amarante - RN (Tinus)

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **São Gonçalo do Amarante - RN** emitem a NFS-e pelo Business Central por meio do web service da plataforma **Tinus**, usada pela prefeitura. A cidade aparece na lista de cidades de NFS-e como **São Gonçalo do Amarante - RN (Tinus)** (código `RN-SaoGoncaloDoAmarante-TINUS`). A integração cobre a transmissão, a consulta do lote e o DANFSe no layout padrão. O cancelamento da NFS-e é feito no portal da prefeitura.

A autenticação é feita só pelo **certificado digital** da filial. A autorização é **imediata**: a resposta da transmissão já traz a NFS-e autorizada, com número e chave de acesso, ou as mensagens de erro da prefeitura. A plataforma **não oferece ambiente de homologação** para o município: a emissão é sempre em produção.

---

## 1. Contexto

### O que muda para a filial de São Gonçalo do Amarante

| Tema | Como funciona em São Gonçalo do Amarante - RN (Tinus) |
| --- | --- |
| **Layout** | RPS no padrão ABRASF 2.03 da plataforma Tinus, com os grupos de IBS/CBS da Reforma Tributária, assinado com o certificado digital da filial |
| **Autenticação** | Certificado digital da filial. Não há usuário, senha nem token |
| **Envio** | Um lote com um ou mais documentos. A prefeitura processa o lote na hora e responde com as notas geradas ou com os erros |
| **Autorização** | Imediata. A resposta da transmissão grava o número da NFS-e e a chave de acesso no documento, e o lote fica com **Status** **Sucesso** |
| **Código de verificação** | A plataforma não devolve código de verificação: o **Cód. Verificação NFS** do documento fica em branco |
| **Rejeição** | As mensagens da prefeitura aparecem na tela da transmissão. Nada fica gravado e o documento continua pendente |
| **Consulta** | Necessária só quando o lote fica com **Status** **Enviado**, sem a nota na resposta |
| **Município de incidência do ISS** | O **Município Prestação Serviço** do documento, quando o ISS é exigível |
| **Cancelamento** | Não é feito pelo sistema: a NFS-e é cancelada no portal da prefeitura e, no Business Central, só o documento fiscal é cancelado |
| **DANFSe** | Layout padrão, gerado no Business Central a partir da chave de acesso |
| **Ambientes** | Só produção. Não há homologação |

### Tributos enviados

- **ISS**: a alíquota e o valor do ISS, a retenção (quando o documento tem ISS retido) e a exigibilidade do ISS (o **Cód. Tributação NFS** do documento, veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)). Para o ISS não retido, o valor enviado é recalculado sobre o valor dos serviços menos as deduções, com duas casas decimais. Quando o ISS é exigível, a nota informa como município de incidência o **Município Prestação Serviço** do documento.
- **Retenções federais**: INSS e IRRF retidos. O **PIS** e a **COFINS** informados na nota são os de **apuração própria**, com a base e as alíquotas. O **PIS e a COFINS retidos são somados à CSLL retida** e enviados no campo de CSLL, com o tipo de retenção correspondente. Também vão o CST do PIS/COFINS, as outras retenções, as deduções e os descontos.
- **Construção civil**: o código da obra e a ART, quando o serviço ou o documento têm obra.
- **IBS/CBS (Reforma Tributária)**: quando a **primeira linha** do documento tem classificação tributária de IBS/CBS, a nota leva o código indicador de operação, o CST e a classificação tributária dessa linha e, conforme o caso, a tributação regular, o diferimento e os grupos de imóvel e de destinatário. O destinatário só é enviado quando o endereço de entrega é o destinatário da operação e tem CNPJ ou CPF próprio. O grupo de evento não é enviado. O município de incidência do IBS/CBS é o **Município Prestação Serviço**. A configuração desses cadastros está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md).
- **Tomador estrangeiro**: o NIF e o código do país.
- **Tributos aproximados**: não são enviados (a integração não usa o IBPT).

A nota informa a **Opção pelo Simples** e o **Regime Especial de Tributação** da **Config. NF-e**. O campo **Incentivo Cultural** não é enviado: a nota sempre informa que não há incentivo fiscal.

---

## 2. Configuração

### Certificado digital e habilitação na prefeitura

O certificado digital é a única credencial da integração: ele autentica a conexão com o web service da plataforma Tinus e assina o RPS e o lote.

1. Confirme com a prefeitura de São Gonçalo do Amarante que a filial está inscrita no município e habilitada a emitir NFS-e por web service. Não há usuário, senha ou token a gerar: a filial é identificada pelo certificado, pelo CNPJ e pela **Inscrição Municipal**.
2. Use o certificado digital A1 (arquivo `.pfx`) da empresa e cadastre-o em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**).
3. Selecione-o no campo **ID Certificado** da aba **NFS-e** (veja a tabela abaixo).

Quando o CNPJ da filial não está cadastrado no município, a prefeitura recusa a transmissão com "E45 - CNPJ não encontrado na base de dados. Confira o numero do CNPJ informado. Caso esteja correto, o prestador não está inscrito no município.".

O Business Central avisa quando o certificado está perto de vencer e **bloqueia a transmissão e a consulta** quando ele já venceu, com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação

A plataforma Tinus **não tem ambiente de homologação** para São Gonçalo do Amarante: toda nota transmitida é emitida em produção, com validade jurídica. Mantenha **Homologação** desligado na **Config. NF-e** da filial. Ligado, toda operação é recusada antes de chegar à prefeitura com a mensagem "Não existe ambiente de homologação para a cidade RN-SaoGoncaloDoAmarante-TINUS":

- na transmissão, a mensagem aparece na tela e nada é enviado;
- na consulta, o lote fica com **Status** **Erro** e a mensagem fica em **Lista Erros**.

Para a primeira emissão, faça o teste com uma nota real, de baixo valor:

1. Confirme com a prefeitura que a filial está habilitada a emitir NFS-e por web service.
2. Configure a aba **NFS-e** da **Config. NF-e** conforme a tabela abaixo, com **Homologação** desligado.
3. Confira os códigos de serviço, os códigos de tributação NFS-e e os cadastros da filial e do cliente antes de registrar a venda.
4. Registre uma fatura de venda de serviço para a filial.
5. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**.
6. Corrija o que a mensagem da prefeitura apontar e transmita de novo.
7. Com a nota autorizada, confira os dados no portal de NFS-e da prefeitura e use **Abrir DANFSe** no **Documento Fiscal Registrado** (veja a [seção 4](#4-danfse)). Se a nota tiver sido emitida com algum dado errado, cancele-a conforme [Cancelar](#cancelar).

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para São Gonçalo do Amarante - RN (Tinus) |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **São Gonçalo do Amarante - RN (Tinus)** na lista (código `RN-SaoGoncaloDoAmarante-TINUS`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (`Tinus`). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial |
| **Configuração Envio** | **Homologação** | **Desligado**. Não há homologação para São Gonçalo do Amarante (veja [Homologação](#homologação)) |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. A plataforma não informa o último número de RPS usado; desligado, toda transmissão falha |
| **Configuração Envio** | **WS NFS-e informa o IM** | Não usado por São Gonçalo do Amarante. A **Inscrição Municipal** da filial vai sempre na nota |
| **Configuração Envio** | **Nome Usuário WS** e **Senha WS** | Não usados por São Gonçalo do Amarante |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão |
| **Configuração Envio** | **ID Relatório p/ TXT** | Não usado. Deixe em branco |
| **Lote** | **Usar Data/Hora como Nº Lote** e **Nº Série Lote** | O número do lote precisa ter só dígitos. Ligue **Usar Data/Hora como Nº Lote** ou informe em **Nº Série Lote** uma série só com dígitos |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | Opcional. Com `0`, **Transmitir Docs. Pendentes** envia todos os documentos pendentes num único lote; com outro valor, envia no máximo essa quantidade |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** (veja a [seção 5](#5-pontos-de-atenção)) |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Ligado**. É a única forma de obter o DANFSe no Business Central (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Desligado. Em São Gonçalo do Amarante, a nota declara como município de incidência o local da prestação, e o DANFSe deve mostrar o mesmo |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `2000`. A plataforma aceita até 2000 caracteres na discriminação; com o limite, o sistema avisa já no registro da venda |
| — | **Nº Série RPS/Temporário** | Série de numeração só com dígitos, com o **Último Nº Usado** preenchido (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente. Em branco, ele precisa ser informado em cada venda |
| — | **NFS-e suporta IBPT** | Desligado automaticamente ao escolher São Gonçalo do Amarante. Os tributos aproximados não são enviados |
| — | **Opção pelo Simples** | Conforme a filial. Enviado na nota |
| — | **Regime Especial de Tributação** | Conforme a filial. Enviado quando diferente de **00 - Nenhum** |
| — | **Incentivo Cultural** | Não enviado |

### Numeração e série do RPS

O layout do Tinus aceita **apenas dígitos** no número do RPS e no número do lote:

- O número do RPS é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão. A série de numeração do tipo de documento fiscal usado pela filial (**Nº Série Fiscal** da **Série Filial Tipo Doc. Fiscal** ou, sem série por filial, **Nº Série Nota Fiscal** do tipo de documento fiscal) precisa gerar números só com dígitos, com até 15 posições (por exemplo, `000031` e não `NFS000031`).
- A **Série** do tipo de documento fiscal precisa estar preenchida, com até 5 caracteres. Ela vem da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, do próprio tipo de documento fiscal.
- A série de **Nº Série RPS/Temporário** é usada:
  - pela ação **Criar Novo Nº**;
  - pela consulta do lote, que exige essa série preenchida e com **Último Nº Usado**, e a usa para localizar o documento de cada RPS devolvido.

  Ela precisa ter só dígitos e o mesmo número de posições da numeração do tipo de documento fiscal.

### Filial e cliente

Estes cadastros são exigidos na transmissão. O sistema não os confere de uma só vez: a transmissão é interrompida na primeira falta, com a mensagem do campo correspondente.

- **Informação Filial** da filial: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos.
- **CEPs**: o CEP da filial e o do cliente cadastrados, com **Cód. Municipio IBGE**.
- **Cliente**: categoria, CEP, cidade, bairro, endereço, número, UF e país preenchidos (exceto pessoa física e cliente não identificado).
- **Categoria do cliente**: o tomador precisa ser **Pessoa Jurídica** com CNPJ, **Pessoa Física** (sem CPF, a nota vai com o CPF zerado) ou **Estrangeiro**. Clientes das categorias **Produtor Rural** e **Não Identificado**, e pessoa jurídica sem CNPJ, não podem ser tomadores nesta integração: a transmissão é recusada pelo serviço de integração com uma mensagem técnica.
- **Cliente do exterior**: o **Cód. NIF** na aba Fiscal da ficha do cliente e o **Cód. BACEN** do país em **Países/Regiões**.
- A **Inscrição Municipal** do cliente pessoa jurídica vai na nota quando ele é do mesmo município da filial.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Cód. Tributação** | O código de tributação **nacional**, de 6 dígitos (os 4 primeiros são o item da LC 116). Vai na nota como código de tributação do município, exatamente como está no campo. O sistema também usa esse código, sem pontos, para identificar os serviços de obra e o tipo de operação do IBS/CBS nas operações com bens imóveis |
| **Item Lista Serviço** | O item da lista de serviços da LC 116. **Obrigatório** no layout do Tinus. Vai na nota exatamente como está no campo |
| **Código CNAE** | O CNAE do serviço, com 7 dígitos. Pontos, traços e barras são descartados; com menos de 7 dígitos, o CNAE não é enviado |
| **Cód. NBS** | O código NBS do serviço. Quando preenchido, vai na nota só com os dígitos |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS (6 dígitos), enviado quando a linha tem classificação tributária de IBS/CBS |
| **Cód. Obra** e **Cód. ART** | Enviados como dados da obra quando preenchidos e o documento não traz obra própria |
| **Tipo de Suspensão da ISS** e **Processo de Suspensão Nº** | Não enviados |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos **Cód. Tributação**, **Item Lista Serviço**, **Código CNAE**, **Cód. Obra** e **Cód. ART** desse registro substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item.

### Códigos Tributação NFS-e

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. No Tinus, o campo **Cód. Tributação** desse código é a **exigibilidade do ISS** do padrão ABRASF, e precisa ter um destes valores:

| Valor | Exigibilidade do ISS |
| --- | --- |
| `1` | Exigível |
| `2` | Não incidência |
| `3` | Isenção |
| `4` | Exportação |
| `5` | Imunidade |
| `6` | Exigibilidade suspensa por decisão judicial |
| `7` | Exigibilidade suspensa por processo administrativo |

A pesquisa do campo não oferece lista nesta filial: digite o valor. Com o valor `1`, a nota informa o município de incidência do ISS (veja [Tributos enviados](#tributos-enviados)). Nos valores `6` e `7`, o número do processo de suspensão não é enviado.

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

Quando a prefeitura autoriza a nota, o documento recebe na hora o **Nº NFS**, a **Data NFS** e a **Chave de Acesso NFS-e**, o **Nº Nota Fiscal** passa a ser o número da NFS-e e o lote fica com **Status** **Sucesso**. O **Cód. Verificação NFS** fica em branco, porque a plataforma não devolve esse código.

Quando a prefeitura recusa o envio, as mensagens aparecem na tela, uma por linha, no formato `<código> - <descrição> <correção>`, precedidas de `RPS <número>/<série>:` quando a prefeitura identifica o RPS. Nada fica gravado: o documento continua pendente, sem lote, e as mensagens não ficam em **Lista Erros**. Anote a mensagem antes de fechar a tela.

### Consultar o processamento

Na transmissão normal, a nota já volta autorizada e não é preciso consultar: **não consulte** um lote com **Status** **Sucesso**.

Se a prefeitura aceitar o lote sem devolver a nota, o lote fica com **Status** **Enviado**. Nesse caso, selecione o documento na página **NFS-e - Envio de Lote** e use **Consultar**:

| Resultado da consulta | O que acontece no Business Central |
| --- | --- |
| **Nota gerada** | O documento recebe o **Nº NFS** e a **Data NFS**, e o lote fica com **Status** **Sucesso** |
| **Rejeitada** | O lote fica com **Status** **Erro** e as mensagens da prefeitura ficam em **Lista Erros** |
| **Sem resultado** | O lote continua **Enviado**. Consulte de novo mais tarde |

A consulta exige a série **Nº Série RPS/Temporário** preenchida (veja [Numeração e série do RPS](#numeração-e-série-do-rps)).

### Corrigir e reenviar

Quando a transmissão é recusada, ou o lote fica com **Status** **Erro** depois da consulta, corrija o cadastro apontado pelas mensagens e transmita o documento novamente. O reenvio mantém o número e a série do RPS. Se a prefeitura recusar o número, a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote** atribui ao documento o próximo número de **Nº Série RPS/Temporário**.

### Cancelar

A integração **não cancela a NFS-e** pelo sistema. Para cancelar uma nota:

1. Cancele a NFS-e no portal de NFS-e da prefeitura de São Gonçalo do Amarante.
2. No **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`), use **Cancelar Documento Fiscal** (grupo **Contabilização**): a contabilização é estornada e o documento fica com **Status** **Cancelado** no Business Central. Nada é enviado à prefeitura neste passo.

Não use **Cancelar NFS-e** nesta integração: a ação termina com a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" seguida de "Cancelamento não implementado.".

O campo **NFS Cancelada** do documento continua desmarcado, porque não há como marcá-lo sem o cancelamento pelo sistema. Por isso, o DANFSe gerado pelo Business Central não sai com a marca de cancelada: o comprovante do cancelamento é o da prefeitura.

---

## 4. DANFSe

Com **Utilizar Layout Padrão** ligado, o DANFSe da nota autorizada na transmissão é gerado no padrão nacional a partir da chave de acesso, pelas ações **Abrir DANFSe** e **Atualizar DANFSe** do **Documento Fiscal Registrado**. Antes da transmissão, a ação **Imprimir** da página **NFS-e - Envio de Lote** gera o documento em modo DPS, para conferência. O layout, a marca d'água de cancelamento e as demais opções estão descritos em [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md).

O Layout Padrão é a única forma de obter o DANFSe no Business Central nesta integração: mantenha **Utilizar Layout Padrão** sempre ligado. Com o campo desligado, não há DANFSe no sistema:

- **Abrir DANFSe** exige o **Cód. Verificação NFS**, que a plataforma não devolve;
- **Atualizar DANFSe** termina com a mensagem "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e".

Ligue o campo e use **Atualizar DANFSe**.

O DANFSe do Layout Padrão segue o modelo nacional e é montado com os dados do documento no Business Central. Alguns quadros usam a tabela nacional:

- a situação do Simples Nacional sai como "Não optante" e o regime especial de tributação como "Nenhum";
- a tributação do ISSQN usa os nomes nacionais, e só o valor `1` ("Operação Tributável") corresponde ao do Tinus.

Quando a filial é optante pelo Simples, tem regime especial ou emite com **Cód. Tributação** diferente de `1`, o documento de referência é o DANFSe da prefeitura, disponível no portal de NFS-e.

---

## 5. Pontos de Atenção

- **Mostrar Somente Pendentes**: o filtro da página **NFS-e - Envio de Lote** usa o **Cód. Verificação NFS**, que a plataforma não devolve. Por isso, as notas já autorizadas **continuam aparecendo** na lista com o filtro ligado. Confira a coluna **Status NFS-e** (**Sucesso**) ou o **Nº NFS** do documento. **Transmitir Docs. Pendentes** não reenvia essas notas: ele ignora os documentos com **Status NFS-e** **Enviado** ou **Sucesso**.
- **Rejeição só na tela**: as mensagens de uma transmissão recusada não ficam gravadas no lote. Copie a mensagem antes de fechar a tela.
- **Lote preso em Enviado**: consulte o lote antes de qualquer outra ação. **Marcar como Erro** (com **Permitir Erro Lote Manual** ligado) libera o documento para nova transmissão; use-a só depois de confirmar no portal da prefeitura que a nota não foi gerada, para não emitir a mesma nota duas vezes.
- **Cancelamento**: feito no portal da prefeitura, seguido de **Cancelar Documento Fiscal** no Business Central (veja [Cancelar](#cancelar)).
- **Certificado vencido**: a transmissão e a consulta são bloqueadas até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **E-mail do documento**: com **Utilizar Layout Padrão** ligado, o DANFSe é anexado ao e-mail do documento. Com o campo desligado, o e-mail fica pendente com "Sem Cód. Verificação NFS: use o Layout Padrão", a menos que **NFS-e sem Anexo** (aba **E-Mails de Documento** da **Config. NF-e**) esteja ligado.
- **Sem homologação**: toda nota transmitida é emitida em produção. Mantenha **Homologação** desligado e confira os cadastros antes da primeira transmissão (veja [Homologação](#homologação)).
- **Log de comunicação**: o envio e o retorno de cada operação ficam registrados no log do serviço de integração, consultado pelo suporte. No Business Central ficam as mensagens gravadas no lote e os dados de retorno do documento.
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| "E45 - CNPJ não encontrado na base de dados. Confira o numero do CNPJ informado. Caso esteja correto, o prestador não está inscrito no município." | Conferir o CNPJ na **Informação Filial** e a inscrição da filial com a prefeitura de São Gonçalo do Amarante |
| `<código> - <descrição> <correção>` | Recusa da prefeitura: corrigir o cadastro indicado na descrição e transmitir de novo |
| "Não existe ambiente de homologação para a cidade RN-SaoGoncaloDoAmarante-TINUS" | Desligar **Homologação** na **Config. NF-e** da filial |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "Discriminação dos serviços excedeu o limite de 2000 caracteres. Total: &lt;quantidade&gt;" | Reduzir a discriminação e usar **Tamanho Máx. Discriminação** = `2000` |
| "O tamanho da discriminação (&lt;quantidade&gt;) excede o máximo permitido (2000)." | Aparece no registro da venda: reduzir as descrições das linhas ou os textos adicionais |
| "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. ..." | Cadastrar o certificado renovado e selecioná-lo em **ID Certificado** |
| "Lote &lt;nº&gt; deverá ser consultado antes do envio." | Consultar o lote do documento (veja [Consultar o processamento](#consultar-o-processamento)) |
| Mensagem técnica do serviço de integração ao transmitir para cliente **Produtor Rural**, **Não Identificado** ou pessoa jurídica sem CNPJ | Ajustar a categoria ou o CNPJ do cliente (veja [Filial e cliente](#filial-e-cliente)) |
| "Os seguintes erros ocorreram no cancelamento da NFS-e:" e "Cancelamento não implementado." | Cancelar a NFS-e no portal da prefeitura e usar **Cancelar Documento Fiscal** (veja [Cancelar](#cancelar)) |
| "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe. ..." | O documento não tem **Chave de Acesso NFS-e**: obter o DANFSe no portal de NFS-e da prefeitura, sem desligar **Utilizar Layout Padrão** |
| "A cidade &lt;cidade&gt; não possui automatização de imagem NFS-e" | Ligar **Utilizar Layout Padrão** e usar **Atualizar DANFSe** |

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **São Gonçalo do Amarante - RN (Tinus)** (`RN-SaoGoncaloDoAmarante-TINUS`) |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Homologação** | Desligado |
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
| **Códigos de Serviço NFS** | **Cód. Tributação** | Código de tributação nacional (6 dígitos) |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Item da lista de serviços da LC 116 (obrigatório) |
| **Códigos de Serviço NFS** | **Código CNAE** | CNAE de 7 dígitos |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | Exigibilidade do ISS, de `1` a `7` |
