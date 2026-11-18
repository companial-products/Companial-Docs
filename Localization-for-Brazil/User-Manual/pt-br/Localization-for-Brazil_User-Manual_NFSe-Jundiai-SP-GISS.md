<!-- PR: #274, #309 -->
# NFS-e: Emissão em Jundiaí - SP (GISS)

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **Jundiaí - SP** emitem NFS-e pelo Business Central na plataforma **GISS**, usada pela prefeitura, no padrão GISS 2.04. A cidade aparece na lista de cidades de NFS-e como **Jundiaí - SP (GISS)** e cobre transmissão dos RPS em lote, consulta do processamento, reenvio após rejeição e cancelamento.

A autenticação é feita só pelo **certificado digital** da filial. A autorização não é imediata: a GISS recebe o lote e confirma o recebimento, e o número da NFS-e e o código de verificação só chegam na **consulta** do lote. O DANFSe não é gerado pelo Business Central: a nota é consultada e impressa no portal da prefeitura.

---

## 1. Contexto

### O que muda para a filial de Jundiaí

| Tema | Como funciona em Jundiaí - SP (GISS) |
| --- | --- |
| **Layout** | RPS do padrão GISS 2.04, assinado com o certificado digital da filial |
| **Autenticação** | Certificado digital da filial. **Nome Usuário WS** e **Senha WS** não são usados |
| **Código de serviço municipal** | Obrigatório, no formato `00.00`. Vai no RPS a partir do campo **Item Lista Serviço** do código de serviço NFS |
| **Envio** | Vários RPS no mesmo lote. A GISS confirma o recebimento do lote, mas as notas ainda não estão autorizadas |
| **Autorização** | Conhecida na consulta do lote: grava o número da NFS-e, a data de emissão e o código de verificação |
| **Rejeição** | Se a GISS recusa o lote no recebimento, as mensagens aparecem na transmissão. Se recusa no processamento, as mensagens ficam em **Lista Erros** do lote, depois da consulta |
| **Cancelamento** | Pedido de cancelamento enviado à GISS com o número da NFS-e e o motivo "Erro na emissão" |
| **DANFSe** | Não é gerado pelo Business Central. A nota é consultada e impressa no portal da prefeitura |
| **Ambientes** | Homologação (ambiente de testes da GISS) e produção |

### Tributos enviados

- **ISS**: alíquota, indicação de ISS retido e município de incidência, que é o **Município Prestação Serviço** do documento. O valor do ISS enviado é recalculado pela alíquota sobre o valor dos serviços menos as deduções, com arredondamento em duas casas. A exigibilidade do ISS sai do **Cód. Tributação NFS** do documento (veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)).
- **Retenções federais**: PIS, COFINS, INSS, IRRF, CSLL e outras retenções. Nos campos de PIS e de COFINS vão os valores **retidos**; quando o PIS ou a COFINS não tem retenção, vai no lugar o valor de **apuração própria**, com a base e a alíquota correspondentes. O indicador de retenção de PIS, COFINS e CSLL considera só os valores retidos, e o CST do PIS/COFINS é o do PIS da primeira linha do documento.
- **IBS/CBS (Reforma Tributária)**: o código indicador de operação (**Cód. Operação de Serviços** do código de serviço), o CST e a classificação tributária da primeira linha do documento e o percentual de redução da CBS. Os valores de IBS/CBS, a tributação regular, o diferimento, o destinatário e os grupos de imóvel e de evento não são enviados. A configuração desses cadastros está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md).
- **Tributos aproximados (IBPT)**: não são enviados. A GISS não está marcada como compatível com IBPT na lista de cidades, e o campo **NFS-e suporta IBPT** fica desligado.
- **Tomador estrangeiro**: **Cód. NIF** do cliente e endereço no exterior, com o país identificado pelo **Cód. BACEN** do país.

O prestador é informado com a **Opção pelo Simples** e o **Regime Especial de Tributação** da **Config. NF-e**. O **Incentivo Cultural** não é usado: o RPS informa sempre que não há incentivo fiscal. A inscrição municipal da filial vai em todos os RPS.

---

## 2. Configuração

### Certificado digital

O certificado digital é a única credencial da GISS: ele identifica a empresa na conexão e assina o lote, cada RPS, a consulta e o pedido de cancelamento.

1. Use o certificado digital A1 (arquivo `.pfx`) da empresa, o mesmo usado na NF-e. Confirme com a prefeitura de Jundiaí se a empresa precisa de algum credenciamento prévio para emitir NFS-e por web service.
2. Cadastre o certificado em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**).
3. Selecione-o no campo **ID Certificado** da aba **NFS-e** (veja a tabela abaixo).

O Business Central avisa quando o certificado está perto de vencer e **bloqueia a transmissão, a consulta e o cancelamento** quando ele já venceu, com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação

A GISS tem um ambiente de homologação. Para usá-lo, ligue **Homologação** na **Config. NF-e** da filial: a transmissão, a consulta e o cancelamento passam a ser feitos no ambiente de homologação da GISS, e as notas emitidas ali não têm validade jurídica. Para emitir em produção, desligue **Homologação**.

O Business Central trata o retorno da homologação como o de produção: o documento autorizado recebe o número da nota de homologação e deixa de ficar pendente de envio. Faça os testes com documentos de teste, de preferência numa empresa de teste.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para Jundiaí - SP (GISS) |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Jundiaí - SP (GISS)** na lista (código `SP-Jundiai-GISS`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (`Ginfes`). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial |
| **Configuração Envio** | **Nome Usuário WS** e **Senha WS** | Não usados por Jundiaí |
| **Configuração Envio** | **Homologação** | Ligado para emitir no ambiente de homologação da GISS; desligado para produção |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. A GISS não informa o último número de RPS usado; com o campo desligado, toda transmissão falha |
| **Configuração Envio** | **WS NFS-e informa o IM** | Não usado por Jundiaí: a inscrição municipal da filial vai sempre no RPS |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o endereço padrão do serviço de integração |
| **Configuração Envio** | **ID Relatório p/ TXT** | Em branco. É usado só por filial que não emite por web service |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | `0` (sem limite) ou a quantidade máxima de documentos por lote que a filial preferir. A GISS recebe vários RPS no mesmo lote |
| **Lote** | **Usar Data/Hora como Nº Lote** e **Nº Série Lote** | O número do lote vai para a GISS e precisa ter só dígitos: ligue **Usar Data/Hora como Nº Lote** ou informe em **Nº Série Lote** uma série de numeração só com dígitos |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Desligado**. A GISS não devolve a chave de acesso que o DANFSe padrão exige (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Não usado por Jundiaí: afeta só o DANFSe do Layout Padrão |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `2000`. A GISS aceita até 2000 caracteres na discriminação do serviço e recusa a transmissão acima disso; com o limite, o sistema avisa já no registro da venda |
| — | **Nº Série RPS/Temporário** | Obrigatório para a consulta: série só com dígitos, já usada (com **Último Nº Usado** preenchido) e com o mesmo número de dígitos do **Nº Nota Fiscal** (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente. Em branco, ele precisa ser informado em cada venda |
| — | **NFS-e suporta IBPT** | Desligado automaticamente ao escolher Jundiaí |
| — | **Opção pelo Simples** | Ligado se a filial é optante pelo Simples Nacional. Vai no RPS |
| — | **Regime Especial de Tributação** | O regime especial da filial, quando houver. **00 - Nenhum** não é enviado |
| — | **Incentivo Cultural** | Não usado por Jundiaí |

Se a filial ainda estiver configurada com o código antigo `SP-Jundiai`, que não aparece mais na lista de cidades, selecione **Jundiaí - SP (GISS)** em **Cód. Cidade WS** e revise a configuração conforme este manual.

### Numeração e série do RPS

- **Número do RPS**: é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão. O **Nº Nota Fiscal** é dado no registro da venda pela série de numeração do tipo de documento fiscal: o **Nº Série Fiscal** da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, ação **Séries por Filial** em **Tipos Documento Fiscal**) ou, sem série por filial, o **Nº Série Nota Fiscal** do próprio tipo de documento fiscal. A GISS aceita **apenas dígitos** no número do RPS: a série precisa gerar números como `000031`, e não `NFS000031`.
- **Série do RPS**: é a **Série** do tipo de documento fiscal usado pela filial, enviada como está.
- **Nº Série RPS/Temporário**: a consulta exige essa série preenchida e com **Último Nº Usado**, e usa o número de dígitos desse último número para localizar o documento de cada RPS que a GISS devolve. Ela precisa ter o mesmo formato e o mesmo número de dígitos do **Nº Nota Fiscal** (por exemplo, a mesma série de numeração do tipo de documento fiscal). Com outro comprimento, a consulta não encontra o documento e ele fica sem o número da NFS-e. Essa série também é usada pela ação **Criar Novo Nº**.
- **Número do lote**: só dígitos (veja **Usar Data/Hora como Nº Lote** e **Nº Série Lote** na tabela acima).

### Filial e cliente

- **Informação Filial** da filial: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos.
- **CEPs**: o CEP da filial e o do cliente cadastrados, com **Cód. Municipio IBGE**.
- **Cliente**: categoria, CEP, cidade, bairro, endereço, número, UF e país preenchidos (exceto pessoa física e cliente não identificado). A **Inscrição Municipal** do cliente vai no RPS quando o cliente é do mesmo município da filial, ou quando ele tem em **Outras Inscrições** uma inscrição do tipo **C.C.M.** para o município da filial.
- **Cliente do exterior**: **Cód. NIF** na ficha do cliente e país com **Cód. BACEN** em **Países/Regiões**. O Business Central informa CEP `99999999` e cidade `EXTERIOR` no endereço do tomador.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Item Lista Serviço** | **Obrigatório.** O código do serviço na lista usada pela GISS, no formato `00.00`: dois dígitos, ponto e dois dígitos (por exemplo, `01.07` ou `17.01`). `1.07`, `0107` e o campo em branco são recusados na transmissão |
| **Cód. Tributação** | O código de tributação municipal do serviço, quando a prefeitura o exige. Vai como está cadastrado; em branco, não é enviado |
| **Cód. NBS** | Código NBS do serviço. Enviado quando preenchido, sem os pontos |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS (6 dígitos). Vai em todas as notas, no grupo do IBS/CBS |
| **Código CNAE** | Não enviado à GISS |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos desse registro, entre eles **Item Lista Serviço** e **Cód. Tributação**, substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item. O código de serviço usado no RPS é o da primeira linha com item de serviço.

### Códigos Tributação NFS-e

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. Em Jundiaí, o campo **Cód. Tributação** desse código informa a **exigibilidade do ISS** do RPS: use o número da exigibilidade conforme a tabela da GISS. O sistema envia valores de `1` a `10`; qualquer outro conteúdo é enviado como `1`.

O documento precisa ter um **Cód. Tributação NFS** cadastrado para a filial e com **Cód. Tributação** preenchido. O código marcado como **Padrão** é sugerido nas vendas da filial.

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Docs. Pendentes**: transmite num único lote os documentos pendentes da filial, até o limite de **Nº Máx. de Docs. por Lote WS**;
- **Transmitir Doc. Selecionado**: transmite só o documento selecionado.

Antes de chegar à GISS, a transmissão é barrada, com o documento ainda pendente, quando:

| Situação | Mensagem |
| --- | --- |
| **Item Lista Serviço** fora do formato `00.00` ou em branco | "Item Lista de Serviço inválido: &lt;valor&gt;, Formato permitido: 00.00" |
| Discriminação do serviço com mais de 2000 caracteres | "Discriminação dos serviços excedeu o limite de 2000 caracteres. Total: &lt;quantidade&gt;" |
| Cliente do exterior com país sem **Cód. BACEN** reconhecido | "Código de país BACEN '&lt;código&gt;' não encontrado na tabela de conversão BACEN→ISO." |
| Nenhuma linha com item do tipo Serviço e **Cód. Serviço** | "Nenhuma linha da nota fiscal possui Código de Serviço associado." |
| Certificado vencido | "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir." |

Quando a GISS aceita o lote, ele fica com **Status** **Enviado** e o documento com **Status NFS-e** **Enviado**. O lote guarda a identificação do recebimento usada na consulta. A nota **ainda não está autorizada**: o número da NFS-e e o código de verificação só chegam na consulta.

Se a GISS recusar o lote já no recebimento, as mensagens dela aparecem na tela, no formato `<mensagem> - <detalhe técnico>` (o detalhe aparece quando a GISS o informa), e o documento continua pendente. Corrija o que for apontado e transmita de novo.

### Consultar o processamento

Depois da transmissão, consulte o lote:

- na página **NFS-e - Envio de Lote**, selecione os documentos e use **Consultar**; ou
- na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`), selecione o lote e use **Consultar**.

A consulta exige o **Nº Série RPS/Temporário** preenchido e com **Último Nº Usado** (veja [Numeração e série do RPS](#numeração-e-série-do-rps)).

| Resultado da consulta | O que acontece no Business Central |
| --- | --- |
| **Autorizada** | O lote fica com **Status** **Sucesso**. Cada documento recebe o **Nº NFS**, a **Data NFS** e o **Cód. Verificação NFS**, e o **Nº Nota Fiscal** passa a ser o número da NFS-e |
| **Rejeitada** | O lote fica com **Status** **Erro** e as mensagens da GISS ficam em **Lista Erros**, no formato `<mensagem> - <detalhe técnico>`. O código do erro da GISS não aparece. Cada linha guarda até 250 caracteres |
| **Falha de comunicação** | O lote fica com **Status** **Erro** e **Lista Erros** traz a falha de comunicação (por exemplo, o status HTTP e a resposta do servidor). Antes de transmitir o documento de novo, use **Consultar** outra vez: a identificação do recebimento do lote continua valendo |

### Corrigir e reenviar

Quando o lote fica com **Status** **Erro** por rejeição da GISS, corrija o cadastro apontado em **Lista Erros** e transmita o documento novamente. Ele vai para um lote novo, com o mesmo número e a mesma série de RPS. Se a GISS recusar o número do RPS por já ter sido usado, a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote** atribui ao documento o próximo número de **Nº Série RPS/Temporário**.

Um documento cujo lote está com **Status** **Enviado** não pode ser transmitido de novo antes da consulta: a transmissão termina com "Lote &lt;nº&gt; deverá ser consultado antes do envio.".

### Primeira emissão em homologação

1. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** ligado.
2. Confira os códigos de serviço (em especial o **Item Lista Serviço** no formato `00.00`), o código de tributação NFS-e, a série **Nº Série RPS/Temporário** e os cadastros da filial e do cliente.
3. Registre uma fatura de venda de serviço para a filial.
4. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**.
5. Com o lote **Enviado**, use **Consultar**.
6. Corrija o que a transmissão ou a **Lista Erros** apontar e transmita de novo.
7. Com a nota autorizada, confira no documento o **Nº NFS** e o **Cód. Verificação NFS**.

### Cancelar

O cancelamento tem dois passos, no **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`):

1. **Cancelar Documento Fiscal** (grupo **Contabilização**): estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central. Nada é enviado à GISS neste passo.
2. **Cancelar NFS-e** (grupo **NFS-e**), habilitado depois do primeiro passo: envia à GISS o pedido de cancelamento, com o número da NFS-e e o motivo "Erro na emissão". O motivo não pode ser alterado pelo Business Central.

Quando a GISS aceita o cancelamento, a mensagem "NFS-e cancelada com sucesso." é exibida e o campo **NFS Cancelada** do documento é marcado. Se ela recusar, a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" lista os motivos e a NFS-e continua autorizada; corrija o que for apontado e use **Cancelar NFS-e** de novo.

---

## 4. DANFSe

O DANFSe de Jundiaí não é gerado pelo Business Central:

- a GISS não devolve chave de acesso, e o Layout Padrão precisa dela. Com **Utilizar Layout Padrão** ligado, **Abrir DANFSe** termina com a mensagem que começa por "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe";
- com **Utilizar Layout Padrão** desligado, como deve ficar, o serviço de integração não baixa a imagem da nota de Jundiaí, e **Abrir DANFSe** e **Atualizar DANFSe** terminam com "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e".

Consulte e imprima a NFS-e no portal da prefeitura de Jundiaí, com o **Nº NFS** e o **Cód. Verificação NFS** gravados no documento. A ação **Imprimir** da página **NFS-e - Envio de Lote** imprime o RPS (recibo provisório) do documento, e não a NFS-e.

**E-mail de documento**: como o Business Central não obtém o DANFSe, o e-mail do documento fiscal fica com **Status** **Pendente** e **Detalhes** "Aguardando NFS-e". Para enviar o e-mail sem a nota anexada, ligue **NFS-e sem Anexo** na aba **E-Mails de Documento** da **Config. NF-e**.

---

## 5. Pontos de Atenção

- **Autorização na consulta**: a transmissão só registra o recebimento do lote. Consulte o lote para obter o número da NFS-e; até lá, o documento fica com **Status NFS-e** **Enviado**.
- **Nº Série RPS/Temporário**: sem ela, a consulta não roda; com número de dígitos diferente do **Nº Nota Fiscal**, a consulta não grava o número da NFS-e no documento. Depois de ajustar a série, use **Consultar** na página **NFS-e - Status de Lote RPS** (a página **NFS-e - Envio de Lote** não consulta documentos de lote com **Status** **Sucesso**).
- **Certificado vencido**: a transmissão, a consulta e o cancelamento são bloqueados até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **E-mail de documento**: fica pendente, aguardando a NFS-e, a menos que **NFS-e sem Anexo** esteja ligado (veja a [seção 4](#4-danfse)).
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| "Item Lista de Serviço inválido: &lt;valor&gt;, Formato permitido: 00.00" | Ajustar o **Item Lista Serviço** para `00.00` em **Códigos de Serviço NFS** e, se houver registro da filial, em **Códigos Serviço NFS por Filial** |
| "Discriminação dos serviços excedeu o limite de 2000 caracteres. Total: &lt;quantidade&gt;" | Reduzir as descrições e os textos adicionais do documento. Para barrar já no registro da venda, use **Tamanho Máx. Discriminação** |
| "Código de país BACEN '&lt;código&gt;' não encontrado na tabela de conversão BACEN→ISO." | Conferir o **Cód. BACEN** do país do cliente em **Países/Regiões** |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "Lote &lt;nº&gt; deverá ser consultado antes do envio." | Consultar o lote do documento antes de transmitir de novo |
| Erro de campo em **Nº Série RPS/Temporário** ou no **Último Nº Usado** da série, ao consultar | Preencher **Nº Série RPS/Temporário** com uma série já usada, no mesmo formato do **Nº Nota Fiscal** |
| "O serviço retornou um status de erro por razões desconhecidas." | A GISS recusou o envio sem mensagem. Transmitir de novo e, persistindo, acionar o suporte, que vê a resposta da GISS no log do serviço de integração |
| "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. ..." | Cadastrar o certificado renovado e selecioná-lo em **ID Certificado** |
| "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe. ..." | Desligar **Utilizar Layout Padrão** |
| "A cidade &lt;cidade&gt; não possui automatização de imagem NFS-e" | O DANFSe não sai pelo Business Central: consultar e imprimir a nota no portal da prefeitura |
| Mensagens da GISS em **Lista Erros** | Corrigir o cadastro indicado e transmitir de novo |

- **Mensagens longas**: **Lista Erros** guarda até 250 caracteres por mensagem. O texto completo fica no log do serviço de integração.
- **Log de comunicação**: o envio e o retorno de cada operação com a GISS ficam registrados no log do serviço de integração, consultado pelo suporte. No Business Central ficam as mensagens do lote e os dados de retorno do documento.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Jundiaí - SP (GISS)** (`SP-Jundiai-GISS`) |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Homologação** | Conforme o ambiente |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Série só com dígitos, já usada, com o mesmo número de dígitos do **Nº Nota Fiscal** |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Ligado, ou série só com dígitos |
| **Config. NF-e** | **Utilizar Layout Padrão** | Desligado |
| **Config. NF-e** | **Tamanho Máx. Discriminação** | `2000` |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Config. NF-e** | **Opção pelo Simples** e **Regime Especial de Tributação** | Conforme a filial |
| **Config. NF-e** | **NFS-e sem Anexo** (aba **E-Mails de Documento**) | Ligado, se o e-mail do documento deve sair sem a nota |
| **Série Filial Tipo Doc. Fiscal** | **Nº Série Fiscal** | Série de numeração só com dígitos |
| **Informação Filial** | **Inscrição Municipal** | Preenchida |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Código do serviço no formato `00.00` |
| **Códigos de Serviço NFS** | **Cód. Tributação** | Código de tributação municipal, quando exigido |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | Exigibilidade do ISS conforme a tabela da GISS |
