<!-- PR: #274, #282, #215, #309 -->
# NFS-e: Emissão em Rio Grande - RS (Sigcorp, Layout Nacional)

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **Rio Grande - RS** emitem NFS-e pelo Business Central na API da **Sigcorp**, a plataforma de NFS-e da prefeitura de Rio Grande, no layout da NFS-e Nacional (DPS). Na lista de cidades de NFS-e, a integração aparece como **Rio Grande - RS (Sigcorp)** (código `RS-RioGrande`) e cobre transmissão, consulta do processamento, reenvio após rejeição e DANFSe.

A autenticação é feita pela **inscrição municipal** da filial e pela **senha** do prestador na Sigcorp, e a DPS vai assinada com o **certificado digital** da filial. A autorização não é imediata: o envio devolve um protocolo, e o resultado (NFS-e aprovada ou motivos da rejeição) só aparece na consulta do lote. O número da NFS-e é o próprio número do documento, tirado da série de numeração já configurada no Business Central.

Esta integração substitui o antigo web service do SIGISS, que deixou de ser usado: as filiais que emitiam por ele precisam revisar a configuração (veja [Filial que emitia pelo SIGISS](#filial-que-emitia-pelo-sigiss)).

---

## 1. Contexto

### O que muda para a filial de Rio Grande

| Tema | Como funciona em Rio Grande - RS (Sigcorp) |
| --- | --- |
| **Layout** | DPS do layout da NFS-e Nacional, assinada com o certificado digital da filial |
| **Autenticação** | **Inscrição Municipal** da filial e **Senha WS** (a senha do prestador na Sigcorp) |
| **Prestador** | Nome, inscrição municipal e endereço da filial vão sempre na DPS |
| **Número da NFS-e** | Informado pelo Business Central: é o **Nº RPS** do documento, da série de numeração da filial |
| **Envio** | Um documento por lote. A prefeitura devolve um protocolo, mas a nota ainda não está autorizada |
| **Autorização** | Conhecida na consulta do lote: grava o número e a data da NFS-e no documento |
| **Rejeição** | Os motivos da prefeitura ficam no lote, em **Lista Erros**. Depois da correção, o documento é transmitido de novo |
| **Cancelamento** | Não é feito pelo sistema. A NFS-e é cancelada no portal da prefeitura |
| **DANFSe** | Layout padrão nacional, gerado no Business Central a partir da chave de acesso |
| **Simples Nacional** | Não atendido: a transmissão de filial com **Opção pelo Simples** ligado é bloqueada |
| **Ambientes** | Homologação (ambiente de testes da Sigcorp) e produção |

### Tributos na DPS

As retenções federais (PIS, COFINS, CSLL, IRRF e INSS), o ISS retido, os tributos aproximados (IBPT), os grupos de IBS/CBS da Reforma Tributária, obra, imóvel e evento, o NIF do tomador estrangeiro e as informações de comércio exterior seguem na DPS como nas demais cidades do layout nacional. A configuração dos cadastros da Reforma Tributária está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md).

Em Rio Grande, a DPS tem ainda estas regras:

- **Alíquota do ISSQN**: vai na DPS sempre que a operação é tributável (código de tributação `1`) e o documento tem alíquota de ISS. A prefeitura a exige quando o ISSQN é devido a outro município e a ignora nos demais casos; nesse caso, ela precisa estar entre 2% e 5%.
- **IBS/CBS**: quando a DPS leva o grupo de IBS/CBS, o **Cód. NBS** do código de serviço é obrigatório. Sem ele, a transmissão é bloqueada antes do envio.
- **Simples Nacional**: a DPS declara o prestador como **não optante** pelo Simples Nacional, e a integração não atende prestador optante (veja [Configuração NF-e da filial](#configuração-nf-e-da-filial)).

Os campos **Incentivo Cultural** e **Regime Especial de Tributação** da **Config. NF-e** não são enviados.

---

## 2. Configuração

### Senha e inscrição municipal

A Sigcorp identifica a filial pela **inscrição municipal** e pela **senha** do prestador:

1. Obtenha com a prefeitura de Rio Grande a senha de acesso do prestador à Sigcorp para a inscrição municipal da filial.
2. Informe a senha no campo **Senha WS** da **Config. NF-e** da filial.
   - O campo aceita até 50 caracteres.
   - Ao reabrir a página, ele aparece vazio, mas a senha continua gravada: deixe-o em branco para manter a senha atual, ou digite a nova senha para trocá-la.
3. Confira a **Inscrição Municipal** da filial em **Informação Filial**: ela precisa ser a mesma do cadastro do prestador na prefeitura. Só os dígitos são enviados, na autenticação e na DPS.

O Business Central não controla a validade da senha. Quando ela é trocada ou deixa de valer, a prefeitura recusa a transmissão com "Falha de autenticação na prefeitura (HTTP 401): confira a Inscrição Municipal da filial, o CNPJ e a Senha WS da Configuração NF-e."; obtenha a nova senha e atualize **Senha WS**.

### Certificado digital

O certificado digital assina a DPS:

1. Cadastre o certificado digital A1 da empresa em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**).
2. Selecione-o no campo **ID Certificado** da aba **NFS-e**.

Com o certificado vencido, a transmissão é **bloqueada** com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação

A Sigcorp tem um ambiente de testes. Antes do primeiro teste, confirme com a prefeitura ou com a Sigcorp que a inscrição municipal da filial está habilitada nesse ambiente e qual senha usar nele.

Para usá-lo, ligue **Homologação** na **Config. NF-e** da filial: a DPS e a consulta passam a ser enviadas ao ambiente de testes. As notas emitidas ali não têm validade jurídica. Para emitir em produção, desligue **Homologação** e informe em **Senha WS** a senha de produção.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para Rio Grande - RS (Sigcorp) |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Rio Grande - RS (Sigcorp)** na lista (código `RS-RioGrande`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (`Indefinido`). Não é editável |
| **Configuração Envio** | **Senha WS** | A senha do prestador na Sigcorp |
| **Configuração Envio** | **Nome Usuário WS** | Não usado por Rio Grande: o usuário é a inscrição municipal da filial |
| **Configuração Envio** | **ID Certificado** | Certificado digital válido da filial, usado para assinar a DPS |
| **Configuração Envio** | **Homologação** | Ligado para emitir no ambiente de testes; desligado para produção |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. A Sigcorp não informa o último número usado: o número da NFS-e vem da série de numeração do documento |
| **Configuração Envio** | **WS NFS-e informa o IM** | Não usado por Rio Grande: a inscrição municipal da filial vai sempre na DPS |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão: é o endereço do serviço de integração da Localização, usado por todas as cidades |
| **Configuração Envio** | **ID Relatório p/ TXT** | Em branco. Só é usado por filial que gera a NFS-e por arquivo, sem web service |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | **`1`**. A Sigcorp aceita uma nota por envio |
| **Lote** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Um dos dois é obrigatório: ligue **Usar Data/Hora como Nº Lote** ou informe uma série em **Nº Série Lote**. O número do lote fica só no Business Central |
| **Lote** | **Permitir Erro Lote Manual** | Recomendado. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS**, para liberar o reenvio de um lote que ficou **Enviado** sem resposta (veja [Pontos de Atenção](#5-pontos-de-atenção)) |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Ligado**. É a forma de obter o DANFSe desta integração no Business Central (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Afeta só a impressão do DANFSe, conforme [NFS-e: Município de Incidência do ISSQN no DANFSe](Localization-for-Brazil_User-Manual_NFSe-DANFSe-Municipio-Incidencia-ISSQN.md) |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `1000`. A DPS aceita até 1000 caracteres na descrição do serviço e o texto que passa disso é cortado no envio |
| — | **Nº Série RPS/Temporário** | A mesma série de numeração que numera as notas de serviço da filial (veja [Numeração da NFS-e e série da DPS](#numeração-da-nfs-e-e-série-da-dps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente. Em branco, ele precisa ser informado em cada venda |
| — | **NFS-e suporta IBPT** | Ligado automaticamente ao escolher **Rio Grande - RS (Sigcorp)** em **Cód. Cidade WS** |
| — | **Opção pelo Simples** | **Desligado**. Com o campo ligado, a transmissão é bloqueada: a integração não atende prestador optante pelo Simples Nacional, que deve emitir no Emissor Nacional |
| — | **Incentivo Cultural** e **Regime Especial de Tributação** | Não enviados |

### Filial que emitia pelo SIGISS

As filiais de Rio Grande configuradas para o antigo web service do SIGISS continuam com o código `RS-RioGrande` e passam a emitir pela Sigcorp sem trocar de cidade. A configuração do SIGISS, porém, não serve para o layout nacional. Antes da primeira transmissão, revise:

1. **Cód. Cidade WS**: selecione de novo **Rio Grande - RS (Sigcorp)** na lista, para que **NFS-e suporta IBPT** seja ligado. Sem isso, os tributos aproximados vão zerados na DPS.
2. **Senha WS**: informe a senha do prestador na Sigcorp, se ela for diferente da senha do SIGISS.
3. **Utilizar Layout Padrão**: ligue, para o DANFSe.
4. **Nº Série RPS/Temporário**: preencha conforme [Numeração da NFS-e e série da DPS](#numeração-da-nfs-e-e-série-da-dps).
5. **Opção pelo Simples**: confira que está desligado. O SIGISS não usava o campo; a Sigcorp bloqueia a transmissão com ele ligado.
6. **Códigos Tributação NFS-e**: troque as situações tributárias do SIGISS pelos valores nacionais (veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)).
7. **Códigos de serviço**: troque o código de serviço do SIGISS em **Item Lista Serviço** pelo código de tributação municipal de 3 dígitos, ou deixe o campo em branco, e confira o **Cód. NBS** (veja [Códigos de serviço](#códigos-de-serviço)).
8. **Informação Filial**: confira o endereço da filial (veja [Filial e cliente](#filial-e-cliente)).

### Numeração da NFS-e e série da DPS

Em Rio Grande, quem numera a NFS-e é o prestador: o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão, vai à prefeitura como número da NFS-e e como número da DPS. Por isso:

- A série de numeração do tipo de documento fiscal usado pela filial precisa gerar números **só com dígitos**, com até 13 dígitos sem contar os zeros à esquerda (por exemplo, `000031` e não `NFS000031`). Fora disso, a transmissão é bloqueada antes do envio.
- Cada número só pode ser usado uma vez: a prefeitura recusa número de NFS-e já utilizado.
- A **Série** do tipo de documento fiscal precisa estar preenchida, ter só dígitos e até 5 posições. Ela vem da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, do próprio tipo de documento fiscal.
- A série de **Nº Série RPS/Temporário** é usada:
  - pela consulta do lote, que exige essa série preenchida e com **Último Nº Usado**, e usa o número de posições desse número para localizar o documento na resposta da prefeitura;
  - pela ação **Criar Novo Nº**.

  Informe nela **a mesma série** que numera as notas de serviço da filial. Assim o documento é localizado na consulta, e o número dado por **Criar Novo Nº** não repete um número já enviado à prefeitura.

### Filial e cliente

- **Informação Filial**: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos. A Sigcorp exige também o endereço do prestador na DPS: **Endereço**, **Número**, **Bairro** e **CEP** precisam estar preenchidos, e o **Endereço Complementar**, quando preenchido, vai como complemento. Sem eles, a transmissão é bloqueada com "Rio Grande - RS (Sigcorp) exige o nome e o endereço do prestador na DPS: preencha &lt;campos&gt; da filial. ...".
- **CEPs**: o CEP da filial e o do cliente cadastrados com **Cód. Municipio IBGE**.
- **Cliente**: categoria, CEP, cidade, bairro, endereço, número, UF e país preenchidos (exceto pessoa física e cliente não identificado). Para cliente do exterior, **Cód. NIF** ou **Motivo da não informação do NIF** na aba Fiscal da ficha do cliente. Cliente pessoa jurídica de Rio Grande precisa estar cadastrado na prefeitura; caso contrário, a nota é rejeitada.

O Business Central barra a transmissão no primeiro dado obrigatório que faltar na filial ou no cliente, com a mensagem de campo obrigatório. Os demais problemas de cadastro voltam como rejeição da prefeitura, na consulta.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Cód. Tributação** | O código de tributação **nacional**, de 6 dígitos (os 4 primeiros são o item da LC 116). Pontos são descartados no envio |
| **Item Lista Serviço** | O código de tributação **municipal**, com exatamente 3 dígitos, ou em branco. Fora disso, a transmissão é bloqueada antes do envio. Não use mais o código de serviço do SIGISS |
| **Cód. NBS** | Código NBS do serviço. **Obrigatório** quando a nota leva IBS/CBS e para cliente do exterior; quando preenchido, vai em todas as notas |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS (6 dígitos), enviado quando a linha tem classificação tributária de IBS/CBS |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos **Cód. Tributação** e **Item Lista Serviço** desse registro substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item. Sem nenhuma linha assim, a transmissão para com "Nenhuma linha da nota fiscal possui Código de Serviço associado.".

A prefeitura confere se o prestador está autorizado a emitir nota para o código de serviço informado; caso contrário, a nota é rejeitada.

Obras, imóveis e atividades/eventos seguem o layout nacional, conforme [NFS-e: Local da Prestação — Obras, Atividades/Eventos e Imóveis](Localization-for-Brazil_User-Manual_NFSe-Local-Prestacao-Obras-Atividades-Imoveis.md).

### Códigos Tributação NFS-e

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. Em Rio Grande, o campo **Cód. Tributação** desse código precisa ter um dos valores nacionais da tributação do ISSQN:

| Valor | Significado nacional |
| --- | --- |
| `1` | Operação Tributável |
| `2` | Imunidade |
| `3` | Exportação de Serviço |
| `4` | Não Incidência |

Um valor fora da lista é recusado na transmissão com "Código de Tributação inválido: &lt;valor&gt;". A lista de valores e a coluna **Significado nacional** da página só funcionam para filiais do Emissor Público Nacional: em Rio Grande, a página informa que "o valor segue a tabela da prefeitura", mas vale a tabela acima.

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Doc. Selecionado**: transmite o documento selecionado.
- **Transmitir Docs. Pendentes**: com **Nº Máx. de Docs. por Lote WS** igual a `1`, transmite só o primeiro documento pendente. Repita a ação para os demais documentos.

Antes do envio, a transmissão é bloqueada, com uma mensagem que indica o que corrigir, quando:

| Situação | O que corrigir |
| --- | --- |
| O lote tem mais de um documento | `1` em **Nº Máx. de Docs. por Lote WS** |
| A senha não foi informada | **Senha WS** na **Config. NF-e** |
| A filial não tem **Inscrição Municipal** | **Informação Filial** |
| **Opção pelo Simples** está ligado | Desligar o campo, se a filial não for optante. Optante pelo Simples Nacional deve emitir no Emissor Nacional |
| O **Nº RPS** do documento não é numérico ou tem mais de 13 dígitos | A série de numeração do tipo de documento fiscal (veja [Numeração da NFS-e e série da DPS](#numeração-da-nfs-e-e-série-da-dps)) |
| Falta o endereço da filial | **Endereço**, **Número**, **Bairro** e **CEP** em **Informação Filial** |
| A nota leva IBS/CBS e o código de serviço não tem NBS | **Cód. NBS** do código de serviço indicado na mensagem |
| O **Item Lista Serviço** não tem 3 dígitos | **Item Lista Serviço** do código de serviço indicado na mensagem |
| Filial sem **ID Certificado**, ou certificado vencido | **ID Certificado** na **Config. NF-e** |

Quando a prefeitura recebe a DPS, o lote fica com **Status** **Enviado** e o **Protocolo** recebe o protocolo devolvido. A nota **ainda não está autorizada**: o número da NFS-e só é confirmado na consulta.

Quando a prefeitura recusa o envio (por exemplo, senha errada ou DPS fora do esquema), a mensagem aparece na tela e nada é gravado: o lote não é criado e o documento continua pendente.

### Consultar o processamento

Na página **NFS-e - Envio de Lote** ou na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`), selecione o lote e use **Consultar**. A consulta pede à prefeitura o resultado pelo protocolo do lote.

| Resultado da consulta | O que acontece no Business Central |
| --- | --- |
| **Aprovada** | O lote fica com **Status** **Sucesso**. O documento recebe o **Nº NFS**, o **Nº Nota Fiscal** passa a ser o número da NFS-e, a **Data NFS** recebe a data e a hora de processamento da prefeitura, o **Cód. Verificação NFS** recebe o próprio número da NFS-e, e a **Chave de Acesso NFS-e** recebe a chave de acesso devolvida pela prefeitura |
| **Rejeitada pela prefeitura** ou **recusada pelo Ambiente Nacional** | O lote fica com **Status** **Erro**, e os motivos ficam em **Lista Erros**, um por linha, no formato "Rejeitada pela prefeitura: &lt;motivo&gt;" ou "Recusada pelo Ambiente Nacional: &lt;motivo&gt;" |
| **Em processamento** | O lote continua **Enviado**, com o aviso "Protocolo &lt;protocolo&gt; em processamento na prefeitura; consulte novamente em alguns minutos." em **Lista Avisos** |
| **Erro interno da prefeitura** | O lote continua **Enviado**, com o protocolo, e **Lista Avisos** traz "Erro de comunicação: a prefeitura teve um erro interno ao processar o protocolo &lt;protocolo&gt;; consulte novamente mais tarde." |
| **Falha de comunicação** | O lote continua **Enviado**, com a falha em **Lista Avisos**. O reenvio não é liberado, porque a nota pode já estar aprovada |

Enquanto o lote está **Enviado**, uma nova transmissão do documento é recusada: o documento não é enviado duas vezes. Consulte o lote até ele sair desse status.

A consulta exige **Nº Série RPS/Temporário** preenchido e com **Último Nº Usado** (veja [Numeração da NFS-e e série da DPS](#numeração-da-nfs-e-e-série-da-dps)).

Dois motivos de rejeição da prefeitura são mostrados com texto próprio:

- "Alíquota do ISSQN ausente ou fora da faixa de 2% a 5%, obrigatória para prestador optante do Simples Nacional."
- "Alíquota do ISSQN ausente ou fora da faixa de 2% a 5%, obrigatória quando o ISSQN é devido a outro município: confira a alíquota de ISS do documento."

Os demais motivos (prestador não autorizado para o código de serviço, tomador pessoa jurídica do município sem cadastro, número de NFS-e já utilizado, e-mail inválido, entre outros) aparecem como a prefeitura os descreveu.

### Corrigir e reenviar

Quando o lote fica com **Status** **Erro**, corrija o cadastro apontado pelas mensagens e transmita o documento novamente. O reenvio mantém o **Nº RPS** e, portanto, o número da NFS-e. Se a prefeitura recusar o reenvio por número de NFS-e já utilizado, use a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote**, que atribui ao documento o próximo número de **Nº Série RPS/Temporário**, e transmita de novo.

### Primeira emissão em homologação

1. Confirme com a prefeitura ou com a Sigcorp que a inscrição municipal da filial está habilitada no ambiente de testes e obtenha a senha desse ambiente.
2. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** ligado.
3. Confira os códigos de serviço, o código de tributação NFS-e e os cadastros da filial e do cliente.
4. Registre uma fatura de venda de serviço para a filial.
5. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**.
6. Use **Consultar** até o lote sair de **Enviado**. Corrija o que a rejeição apontar e transmita de novo.
7. Com a nota aprovada, abra o **Documento Fiscal Registrado** e use **Abrir DANFSe** (veja a [seção 4](#4-danfse)).

### Cancelar

A NFS-e de Rio Grande **não é cancelada pelo Business Central**:

1. Cancele a NFS-e no portal da prefeitura.
2. No **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`), use **Cancelar Documento Fiscal** (grupo **Contabilização**). A ação estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central; nada é enviado à prefeitura.

Não use **Cancelar NFS-e**: a ação termina com "Os seguintes erros ocorreram no cancelamento da NFS-e:", seguida da orientação de cancelar a nota no portal da prefeitura, e o campo **NFS Cancelada** do documento não é marcado.

---

## 4. DANFSe

Com **Utilizar Layout Padrão** ligado, o DANFSe da nota aprovada é gerado no padrão nacional a partir da chave de acesso, pelas ações **Abrir DANFSe** e **Atualizar DANFSe** do **Documento Fiscal Registrado**. O layout, a marca d'água de homologação e a de cancelamento estão descritos em [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md). O e-mail do documento leva esse DANFSe em anexo.

O Layout Padrão é a forma de obter o DANFSe de Rio Grande no Business Central: o sistema não baixa o PDF da Sigcorp. Mantenha **Utilizar Layout Padrão** sempre ligado nas filiais desta integração.

O DANFSe depende da **Chave de Acesso NFS-e** do documento, que a consulta grava quando a nota é aprovada. Quando ela está em branco, **Abrir DANFSe** termina com a mensagem que começa por "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe" (veja [Pontos de Atenção](#5-pontos-de-atenção)). A mensagem sugere desmarcar **Utilizar Layout Padrão** quando o município emite a NFS-e em layout próprio, o que não é o caso aqui: mantenha o campo ligado.

---

## 5. Pontos de Atenção

- **Chave de acesso em notas antigas**: a consulta que aprova a nota grava a **Chave de Acesso NFS-e** no documento. Nas versões anteriores ela gravava só número, data e código de verificação, então documentos aprovados antes da atualização podem estar com a chave em branco. Sem ela, o DANFSe não sai pelo Business Central e o e-mail do documento fica pendente, aguardando o DANFSe. Para esses documentos, consulte e imprima a nota no portal da prefeitura. Para enviar o e-mail sem o DANFSe, ligue **NFS-e sem Anexo** na aba **E-Mails de Documento** da **Config. NF-e**.
- **Lote que fica Enviado**: se a consulta continuar sem resultado (erro interno ou falha de comunicação persistentes), confira a nota no portal da prefeitura pelo número do documento. Se ela não foi emitida, use **Marcar como Erro** na página **NFS-e - Status de Lote RPS** (exige **Permitir Erro Lote Manual** ligado) e transmita o documento de novo.
- **Falha de comunicação na transmissão**: a mensagem "A nota pode ter sido recebida pela prefeitura: confira no portal antes de transmitir de novo." indica que a conexão caiu depois do envio. O documento continua pendente, mas a nota pode ter sido recebida. Confira no portal antes de transmitir: um reenvio com o mesmo número é recusado como número já utilizado.
- **Optante pelo Simples Nacional**: a integração não atende. A filial optante deve emitir no Emissor Nacional.
- **Certificado vencido**: a transmissão é bloqueada até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| "Senha WS não informada: ..." | Informar a senha do prestador na Sigcorp em **Senha WS** |
| "Falha de autenticação na prefeitura (HTTP 401): ..." | Conferir a **Inscrição Municipal** e o CNPJ da filial e a **Senha WS** |
| "A prefeitura recusou o acesso (HTTP 403): a inscrição municipal da autenticação difere da do prestador na DPS." | Conferir a **Inscrição Municipal** da filial com a prefeitura |
| "Rio Grande - RS (Sigcorp) somente permite o envio de um RPS por vez: ..." | Usar `1` em **Nº Máx. de Docs. por Lote WS** |
| "Rio Grande - RS (Sigcorp): a integração não atende prestador optante do Simples Nacional. ..." | Desligar **Opção pelo Simples**, se a filial não for optante; se for, emitir no Emissor Nacional |
| "Rio Grande - RS (Sigcorp): o nº do RPS '&lt;número&gt;' não pode ser o número da NFS-e, ..." | Ajustar a série de numeração do tipo de documento fiscal para gerar só dígitos, até 13 |
| "Rio Grande - RS (Sigcorp) exige o nome e o endereço do prestador na DPS: ..." | Completar **Endereço**, **Número**, **Bairro** e **CEP** em **Informação Filial**. Se já estiverem preenchidos, atualizar a extensão Localization for Brazil |
| "Rio Grande - RS (Sigcorp) exige o código NBS quando a nota leva IBS/CBS: ..." | Preencher o **Cód. NBS** do código de serviço indicado |
| "Rio Grande - RS (Sigcorp): o Item Lista Serviço do código de serviço NFS '&lt;serviço&gt;' vai como código de tributação municipal, que tem 3 dígitos: ..." | Informar 3 dígitos em **Item Lista Serviço**, ou deixá-lo em branco |
| "Código de Tributação inválido: &lt;valor&gt;" | Ajustar o **Cód. Tributação** do código em **Códigos Tributação NFS-e** para `1`, `2`, `3` ou `4` |
| "A prefeitura recusou a requisição (HTTP 400): ..." | A DPS foi recusada no recebimento. Corrigir o que a mensagem indica e transmitir de novo |
| "Rejeitada pela prefeitura: ..." ou "Recusada pelo Ambiente Nacional: ..." | Corrigir o cadastro indicado e transmitir de novo |
| "Alíquota do ISSQN ausente ou fora da faixa de 2% a 5%, obrigatória quando o ISSQN é devido a outro município: ..." | Conferir a alíquota de ISS do documento, que vem da área de imposto do serviço |
| "Protocolo &lt;protocolo&gt; em processamento na prefeitura; ..." | Aguardar e usar **Consultar** de novo |
| "Erro de comunicação: a prefeitura teve um erro interno ao processar o protocolo &lt;protocolo&gt;; ..." | Usar **Consultar** de novo mais tarde |
| "Protocolo &lt;protocolo&gt; não encontrado na prefeitura (HTTP 404): ..." | Conferir se **Homologação** está no mesmo ambiente em que a nota foi transmitida |
| "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. ..." | Cadastrar o certificado renovado e selecioná-lo em **ID Certificado** |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "Os seguintes erros ocorreram no cancelamento da NFS-e: ..." | O cancelamento não é feito pelo sistema: cancelar no portal da prefeitura e usar só **Cancelar Documento Fiscal** |
| "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe. ..." | Documento aprovado antes da atualização, sem a chave gravada: consultar e imprimir a nota no portal da prefeitura (veja o primeiro item desta seção) |

- **Log de comunicação**: o envio e o retorno de cada operação ficam registrados no log do serviço de integração, consultado pelo suporte, inclusive o XML da NFS-e aprovada e o link de impressão devolvidos pela prefeitura. A senha não aparece no log. No Business Central ficam as mensagens do lote e os dados de retorno do documento.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Rio Grande - RS (Sigcorp)** (`RS-RioGrande`) |
| **Config. NF-e** | **Senha WS** | Senha do prestador na Sigcorp |
| **Config. NF-e** | **ID Certificado** | Certificado digital válido da filial |
| **Config. NF-e** | **Homologação** | Conforme o ambiente |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **Nº Máx. de Docs. por Lote WS** | `1` |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Um dos dois configurado |
| **Config. NF-e** | **Permitir Erro Lote Manual** | Ligado (recomendado) |
| **Config. NF-e** | **Utilizar Layout Padrão** | Ligado |
| **Config. NF-e** | **Tamanho Máx. Discriminação** | `1000` |
| **Config. NF-e** | **Nº Série RPS/Temporário** | A mesma série que numera as notas de serviço da filial |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Config. NF-e** | **NFS-e suporta IBPT** | Ligado (automático ao escolher a cidade) |
| **Config. NF-e** | **Opção pelo Simples** | Desligado |
| **Série Filial Tipo Doc. Fiscal** | **Série** | Preenchida, só com dígitos, até 5 posições |
| **Informação Filial** | **Inscrição Municipal** | A mesma do cadastro do prestador na prefeitura |
| **Informação Filial** | **Endereço**, **Número**, **Bairro** e **CEP** | Preenchidos |
| **Códigos de Serviço NFS** | **Cód. Tributação** | Código de tributação nacional (6 dígitos) |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Código de tributação municipal de 3 dígitos, ou em branco |
| **Códigos de Serviço NFS** | **Cód. NBS** | Código NBS do serviço (obrigatório com IBS/CBS e para cliente do exterior) |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | `1`, `2`, `3` ou `4` |
