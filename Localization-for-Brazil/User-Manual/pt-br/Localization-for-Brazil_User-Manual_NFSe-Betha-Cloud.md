<!-- PR: #274 -->
# NFS-e: Emissão pela plataforma Betha Cloud

**Data**: Outubro/2026

---

## Sumário

Os municípios atendidos pela plataforma **Betha Cloud** recebem a NFS-e no layout da NFS-e Nacional (DPS), pelo web service da própria Betha. Na lista de cidades de NFS-e, essa integração aparece como **Betha Cloud (Reforma Tributária)** (código `BethaCloud`) e vale para qualquer município atendido pela plataforma: o município é identificado pelo código IBGE do CEP da filial. A relação dos municípios atendidos será publicada em [NFS-e: Manuais de Emissão por Município e Plataforma](Localization-for-Brazil_User-Manual_NFSe-Manuais-por-Municipio.md).

A autenticação é feita só pelo **certificado digital** da filial. A autorização **não é imediata**: a Betha recebe a DPS e devolve um protocolo, e o resultado (NFS-e autorizada ou lista de erros) é obtido na **consulta** do lote. A plataforma aceita **um documento por envio**. O DANFSe é gerado no próprio Business Central, no layout padrão nacional.

---

## 1. Contexto

### O que muda para a filial atendida pelo Betha Cloud

| Tema | Como funciona no Betha Cloud |
| --- | --- |
| **Layout** | DPS do layout da NFS-e Nacional, assinada com o certificado digital da filial |
| **Autenticação** | Certificado digital da filial. Não há usuário, senha nem token |
| **Município emissor** | Identificado pelo código IBGE do CEP da filial |
| **Conferência antes do envio** | Não há conferência prévia dos cadastros. Um cadastro incompleto interrompe a transmissão com uma mensagem por vez, e as exigências da plataforma aparecem como rejeição da Betha |
| **Envio** | Um documento por lote. A Betha confirma o recebimento com um protocolo, mas a nota ainda não está autorizada |
| **Autorização** | Conhecida na consulta do lote, feita pelo protocolo do envio |
| **Rejeição** | A recusa no envio aparece na tela. A recusa no processamento fica em **Lista Erros** do lote, depois da consulta |
| **Cancelamento** | Evento de cancelamento do layout nacional, com a chave de acesso da nota |
| **DANFSe** | Layout padrão nacional, gerado no Business Central a partir da chave de acesso |
| **Ambientes** | Homologação e produção, pelo mesmo endereço da Betha |

### Tributos na DPS

- **ISS**: a DPS leva a tributação do ISSQN (o valor do **Cód. Tributação NFS** do documento, veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)), a retenção (quando o documento tem ISS retido) e o município de prestação. A **alíquota do ISS** também vai na DPS sempre que o documento tem **% ISS** maior que zero.
- **Retenções federais**: PIS, COFINS, CSLL, IRRF e INSS retidos, além do PIS e da COFINS de apuração própria, com o CST do PIS/COFINS da linha.
- **Tributos aproximados**: a integração não usa o IBPT. Ao escolher o Betha Cloud, o campo **NFS-e suporta IBPT** é desligado automaticamente, e os valores aproximados vão zerados na DPS.
- **IBS/CBS (Reforma Tributária)**: quando a linha do documento tem classificação tributária de IBS/CBS, a DPS leva:
  - o código indicador de operação, o CST e a classificação tributária;
  - conforme o caso, a tributação regular, o diferimento, o destinatário e os grupos de obra, imóvel e evento.

  A configuração desses cadastros está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md).
- **Tomador estrangeiro**: NIF (ou o motivo da não informação do NIF), endereço no exterior e as informações de comércio exterior.

O prestador é informado na DPS como **não optante** pelo Simples Nacional e sem regime especial de tributação. Os campos **Opção pelo Simples**, **Regime Especial de Tributação** e **Incentivo Cultural** da **Config. NF-e** não são enviados.

---

## 2. Configuração

### Certificado digital

O certificado digital é a única credencial do Betha Cloud: ele autentica a conexão com o web service da Betha e assina a DPS e o evento de cancelamento.

1. Confirme com a prefeitura que o município da filial emite a NFS-e pela plataforma Betha Cloud e que a empresa está habilitada a emitir por ela.
2. Use o certificado digital A1 (arquivo `.pfx`) da empresa, o mesmo usado na NF-e, e cadastre-o em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**).
3. Selecione-o no campo **ID Certificado** da aba **NFS-e** (veja a tabela abaixo).

O Business Central avisa quando o certificado está perto de vencer e **bloqueia a transmissão, a consulta e o cancelamento** quando ele já venceu, com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação

O Betha Cloud usa o **mesmo endereço** para homologação e produção. O ambiente é indicado em cada operação:

- Ligue **Homologação** na **Config. NF-e** da filial para emitir em homologação. A DPS, a consulta e o cancelamento passam a ser enviados como ambiente de homologação, e o DANFSe sai com a marca "NFS-e SEM VALIDADE JURÍDICA".
- Desligue **Homologação** para emitir em produção.

As notas emitidas em homologação não têm validade jurídica. Confirme com a prefeitura se o município oferece o ambiente de homologação na plataforma.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para o Betha Cloud |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Betha Cloud (Reforma Tributária)** na lista (código `BethaCloud`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (Indefinido). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial |
| **Configuração Envio** | **Homologação** | Ligado para emitir em homologação; desligado para produção |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. A Betha não informa o último número usado, então a numeração precisa ser controlada pelo Business Central |
| **Configuração Envio** | **WS NFS-e informa o IM** | Ligue se o município exige a inscrição municipal do prestador na DPS. Desligado, a inscrição municipal não é enviada |
| **Configuração Envio** | **Nome Usuário WS** e **Senha WS** | Não usados pelo Betha Cloud |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão |
| **Configuração Envio** | **ID Relatório p/ TXT** | Não usado |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | **`1`**. Obrigatório: a Betha aceita um documento por envio, e um lote com mais documentos é recusado antes do envio |
| **Lote** | **Usar Data/Hora como Nº Lote** e **Nº Série Lote** | Ligue **Usar Data/Hora como Nº Lote** ou informe uma série em **Nº Série Lote**: o lote precisa de um número |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Ligado**. É a forma de obter o DANFSe no Business Central (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Afeta só a impressão do DANFSe, conforme [NFS-e: Município de Incidência do ISSQN no DANFSe](Localization-for-Brazil_User-Manual_NFSe-DANFSe-Municipio-Incidencia-ISSQN.md) |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `1000`. A DPS aceita até 1000 caracteres na descrição do serviço e o texto que passa disso é cortado no envio; com o limite, o sistema avisa já no registro da venda |
| — | **Nº Série RPS/Temporário** | Série de numeração só com dígitos, com o **Último Nº Usado** preenchido (veja [Numeração e série da DPS](#numeração-e-série-da-dps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente. Em branco, ele precisa ser informado em cada venda |
| — | **NFS-e suporta IBPT** | Desligado automaticamente ao escolher o Betha Cloud |
| — | **Opção pelo Simples**, **Incentivo Cultural** e **Regime Especial de Tributação** | Não enviados. A DPS declara o prestador como não optante pelo Simples Nacional |

### Numeração e série da DPS

O layout aceita **apenas dígitos** no número da DPS, e o número e a série compõem a identificação da DPS:

- O número da DPS é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão. A série de numeração do tipo de documento fiscal usado pela filial precisa gerar números só com dígitos, com até 15 posições (por exemplo, `000031` e não `NFS000031`).
- A **Série** do tipo de documento fiscal precisa estar preenchida, ter só dígitos e até 5 posições. Ela vem da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, do próprio tipo de documento fiscal.
- A série de **Nº Série RPS/Temporário** é usada:
  - pela ação **Criar Novo Nº**;
  - pela consulta do lote, que exige essa série preenchida e com **Último Nº Usado** quando **Controlar Nº RPS** está ligado.

  Ela precisa ter só dígitos e o mesmo formato e número de posições da numeração do tipo de documento fiscal.

### Filial e cliente

Estes cadastros são exigidos na transmissão. Diferentemente do Emissor Público Nacional, o sistema não os confere de uma só vez: a transmissão é interrompida na primeira falta, com a mensagem do campo correspondente.

- **Informação Filial** da filial: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos.
- **CEPs**: o CEP da filial e o do cliente cadastrados, com **Cód. Municipio IBGE**.
- **Cliente**: categoria, CEP, cidade, bairro, endereço, número, UF e país preenchidos (exceto pessoa física e cliente não identificado).
- **Cliente do exterior**: **Cód. NIF** ou **Motivo da não informação do NIF** na aba Fiscal da ficha do cliente, e o país com o código ISO de duas letras. Sem nenhum dos dois, o sistema não bloqueia a transmissão e a DPS sai com o NIF em branco.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Cód. Tributação** | O código de tributação **nacional**, de 6 dígitos (os 4 primeiros são o item da LC 116). Pontos são descartados no envio |
| **Item Lista Serviço** | O código de tributação **municipal**, quando o município tem um. Ele vai na DPS como código de tributação municipal (`cTribMun`) exatamente como está no campo, sem conferência de formato pelo sistema: informe-o como o município o define na plataforma. Em branco, não é enviado |
| **Cód. NBS** | O código NBS do serviço. **Preencha em todos os serviços**: o layout do Betha Cloud exige o código NBS na DPS. Só os dígitos são enviados |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS (6 dígitos), enviado quando a linha tem classificação tributária de IBS/CBS |
| **Código CNAE** | Não enviado |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos **Cód. Tributação** e **Item Lista Serviço** desse registro substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item.

### Códigos Tributação NFS-e

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. No Betha Cloud, o campo **Cód. Tributação** desse código precisa ter um dos valores nacionais da tributação do ISSQN:

| Valor | Significado nacional |
| --- | --- |
| `1` | Operação Tributável |
| `2` | Imunidade |
| `3` | Exportação de Serviço |
| `4` | Não Incidência |

A lista de valores do campo e a coluna **Significado nacional** só funcionam para filiais do Emissor Público Nacional. Numa filial do Betha Cloud, a lista mostra a mensagem "A lista de valores nacionais só está disponível para filiais que emitem NFS-e pelo ambiente nacional (Cód. Cidade WS = SEFIN). Na filial &lt;filial&gt;, o valor segue a tabela da prefeitura.". Mesmo assim, digite um dos quatro valores acima. Um valor fora da lista é recusado na transmissão com "Código de Tributação inválido: &lt;valor&gt;".

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Doc. Selecionado**: transmite o documento selecionado.
- **Transmitir Docs. Pendentes**: com **Nº Máx. de Docs. por Lote WS** igual a `1`, transmite só o primeiro documento pendente. Quando há mais de um pendente, o sistema pergunta "O número máximo de documentos permitidos por lote foi excedido. Somente os primeiros 1 documentos serão enviados. Deseja continuar?". Repita a ação para os demais documentos.

Quando a Betha aceita o envio, o lote fica com **Status** **Enviado**. A nota **ainda não está autorizada**: o resultado é obtido na consulta (veja [Consultar o processamento](#consultar-o-processamento)).

Quando a transmissão é recusada, as mensagens aparecem na tela e nada é gravado: o documento continua pendente. Isso vale para:

- a falta de um cadastro (por exemplo, a **Inscrição Municipal** da filial);
- a recusa da Betha no recebimento, com mensagens no formato `<código>: <mensagem>`.

Corrija o que for apontado e transmita de novo.

### Consultar o processamento

Na página **NFS-e - Envio de Lote**, selecione o documento e use **Consultar**, ou use **Consultar** na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`). A consulta pede à Betha o resultado do processamento pelo protocolo recebido no envio.

| Resultado da consulta | O que acontece no Business Central |
| --- | --- |
| **Autorizada** | O lote fica com **Status** **Sucesso**, e o documento recebe o número da NFS-e e a chave de acesso nacional |
| **Rejeitada** | O lote fica com **Status** **Erro**. A **Lista Erros** traz a situação devolvida pela Betha e as mensagens dela, no formato `<código>: <mensagem>` |
| **Falha de comunicação** | O lote fica com **Status** **Erro**, com o texto da falha em **Lista Erros**. Use **Consultar** de novo antes de corrigir ou reenviar o documento |

Com **Controlar Nº RPS** ligado, a consulta exige **Nº Série RPS/Temporário** preenchido e com **Último Nº Usado** (veja [Numeração e série da DPS](#numeração-e-série-da-dps)).

### Corrigir e reenviar

Quando o lote fica com **Status** **Erro** por rejeição da Betha, o documento volta a ficar pendente:

1. Corrija o cadastro apontado pelas mensagens.
2. Transmita o documento novamente. Ele vai em um novo lote, com o mesmo número e a mesma série da DPS.
3. Se a Betha recusar o número da DPS, use a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote**, que atribui ao documento o próximo número de **Nº Série RPS/Temporário**, e transmita de novo.

Um documento cujo lote está **Enviado** não pode ser transmitido de novo: a transmissão termina com "Lote &lt;nº&gt; deverá ser consultado antes do envio.". Consulte o lote primeiro.

### Primeira emissão em homologação

1. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** ligado.
2. Confira os códigos de serviço (inclusive o **Cód. NBS**), o código de tributação NFS-e e os cadastros da filial e do cliente.
3. Registre uma fatura de venda de serviço para a filial.
4. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**.
5. Com o lote **Enviado**, aguarde alguns minutos e use **Consultar**.
6. Se o lote ficar com **Status** **Erro**, corrija o que a **Lista Erros** apontar e transmita de novo.
7. Com a nota autorizada, abra o **Documento Fiscal Registrado** e use **Abrir DANFSe** (veja a [seção 4](#4-danfse)).

### Cancelar

O cancelamento tem dois passos, no **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`):

1. **Cancelar Documento Fiscal** (grupo **Contabilização**): estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central. Nada é enviado à Betha neste passo.
2. **Cancelar NFS-e** (grupo **NFS-e**), habilitado depois do primeiro passo: envia à Betha o evento de cancelamento, com a chave de acesso da nota e o código de motivo `1` (erro na emissão).

Quando a Betha registra o evento, a mensagem "NFS-e cancelada com sucesso." é exibida e o campo **NFS Cancelada** do documento é marcado. Se ela recusar, a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" lista os motivos e a NFS-e continua autorizada; corrija o que for apontado e use **Cancelar NFS-e** de novo.

---

## 4. DANFSe

Com **Utilizar Layout Padrão** ligado, o DANFSe da nota autorizada é gerado no padrão nacional a partir da chave de acesso, pelas ações **Abrir DANFSe** e **Atualizar DANFSe** do **Documento Fiscal Registrado**. O layout, a marca d'água de homologação e a de cancelamento estão descritos em [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md).

O Layout Padrão é a forma de obter o DANFSe do Betha Cloud no Business Central: o sistema não baixa o PDF da Betha. Mantenha **Utilizar Layout Padrão** sempre ligado nas filiais desta integração; com o campo desligado, **Abrir DANFSe** termina em erro.

---

## 5. Pontos de Atenção

- **Um documento por envio**: mantenha **Nº Máx. de Docs. por Lote WS** igual a `1`. Com `0` (sem limite), **Transmitir Docs. Pendentes** monta um lote com todos os pendentes e, havendo mais de um, a transmissão é recusada.
- **Sem conferência prévia**: as conferências que o sistema faz antes do envio para o Emissor Público Nacional (SEFIN) não valem para o Betha Cloud. Por exemplo:
  - o **Item Lista Serviço** com 3 dígitos;
  - o **Cód. NBS** e o NIF para tomador estrangeiro.

  Falhas nesses cadastros aparecem como rejeição da Betha.
- **Cód. NBS**: preencha em todos os códigos de serviço usados pela filial.
- **Consultar**: o lote enviado só tem resultado depois da consulta. Consulte os lotes **Enviado** até eles ficarem **Sucesso** ou **Erro**.
- **Certificado vencido**: a transmissão, a consulta e o cancelamento são bloqueados até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| "Betha Cloud somente permite o envio de um RPS por vez." | Informar `1` em **Nº Máx. de Docs. por Lote WS** e transmitir de novo |
| "Código de Tributação inválido: &lt;valor&gt;" | Ajustar o **Cód. Tributação** do código em **Códigos Tributação NFS-e** para `1`, `2`, `3` ou `4` |
| "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. ..." | Cadastrar o certificado renovado e selecioná-lo em **ID Certificado** |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "Lote &lt;nº&gt; deverá ser consultado antes do envio." | Consultar o lote do documento antes de transmiti-lo de novo |
| Consulta interrompida pedindo **Nº Série RPS/Temporário** ou o **Último Nº Usado** da série | Preencher a série na **Config. NF-e** (veja [Numeração e série da DPS](#numeração-e-série-da-dps)) |
| "Nenhuma linha da nota fiscal possui Código de Serviço associado." | Preencher o **Cód. Serviço** na ficha do item de serviço |
| "A discriminação dos serviços não foi informada no documento &lt;nº&gt;." | Revisar a descrição das linhas, os textos adicionais do documento e as opções do grupo **Discriminação Serviços** |
| Mensagens `<código>: <mensagem>` da Betha | Rejeição da plataforma: corrigir o cadastro indicado e transmitir de novo |
| "Erro desconhecido" | A Betha não devolveu o motivo. Repetir a operação; se persistir, procurar o suporte |

- **Município de emissão**: o município é identificado pelo CEP da filial. Um CEP com código IBGE errado envia a DPS para outro município.
- **Log de comunicação**: o envio e o retorno de cada operação ficam registrados no log do serviço de integração, consultado pelo suporte. No Business Central ficam as mensagens do lote e os dados de retorno do documento.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Betha Cloud (Reforma Tributária)** (`BethaCloud`) |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Homologação** | Conforme o ambiente |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **WS NFS-e informa o IM** | Ligado se o município exige a inscrição municipal |
| **Config. NF-e** | **Nº Máx. de Docs. por Lote WS** | `1` |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Um dos dois preenchido |
| **Config. NF-e** | **Utilizar Layout Padrão** | Ligado |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Série só com dígitos, com **Último Nº Usado** |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Config. NF-e** | **Tamanho Máx. Discriminação** | `1000` |
| **Série Filial Tipo Doc. Fiscal** | **Série** | Preenchida, só com dígitos, até 5 posições |
| **Informação Filial** | **Inscrição Municipal** | Preenchida |
| **Códigos de Serviço NFS** | **Cód. Tributação** | Código de tributação nacional (6 dígitos) |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Código de tributação municipal, ou em branco |
| **Códigos de Serviço NFS** | **Cód. NBS** | Código NBS, em todos os serviços |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | `1`, `2`, `3` ou `4` |
