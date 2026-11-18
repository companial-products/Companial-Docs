<!-- PR: #274 -->
# NFS-e: Emissão em Saquarema - RJ (Governança Brasil)

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **Saquarema - RJ** emitem a NFS-e pelo Business Central na plataforma da **Governança Brasil**, usada pela prefeitura, no layout da NFS-e Nacional (DPS). A cidade aparece na lista de cidades de NFS-e como **Saquarema - RJ (Governança Brasil)** e cobre transmissão, cancelamento e DANFSe.

A autenticação usa o **certificado digital** da filial e um **usuário e senha** do web service, fornecidos pela prefeitura ou pela Governança Brasil. A autorização é **imediata**: a resposta da transmissão já traz a NFS-e autorizada, com número e chave de acesso, ou a lista de erros. Não há consulta posterior. O DANFSe é gerado no próprio Business Central, no layout padrão nacional.

---

## 1. Contexto

### O que muda para a filial de Saquarema

| Tema | Como funciona em Saquarema - RJ (Governança Brasil) |
| --- | --- |
| **Layout** | DPS do layout da NFS-e Nacional, assinada com o certificado digital da filial e enviada à Governança Brasil |
| **Autenticação** | Certificado digital da filial, mais o usuário e a senha do web service, informados em **Nome Usuário WS** e **Senha WS** |
| **Inscrição municipal** | Vai na DPS quando **WS NFS-e informa o IM** está ligado |
| **Código de serviço** | Código de tributação **nacional** (6 dígitos); o código municipal, quando a prefeitura tiver um; e o **Código CNAE** do serviço |
| **Envio** | Até 50 documentos por lote |
| **Autorização** | Imediata. A resposta da transmissão grava o número da NFS-e e a chave de acesso |
| **Rejeição** | As mensagens da prefeitura aparecem na tela, no momento da transmissão, e o documento continua pendente |
| **Consulta** | Não existe. O resultado já vem na transmissão |
| **Cancelamento** | Evento de cancelamento do layout nacional, com a chave de acesso da nota |
| **DANFSe** | Layout padrão nacional, gerado no Business Central a partir da chave de acesso |
| **Ambientes** | Homologação e produção |

### Tributos enviados

- **ISS**: a DPS leva a tributação do ISSQN (o valor nacional do **Cód. Tributação NFS** do documento, veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)), a retenção (quando o documento tem ISS retido) e o município de prestação. A **alíquota do ISS não é enviada**: vale a alíquota parametrizada pela prefeitura.
- **Retenções federais**: PIS, COFINS, CSLL, IRRF e INSS retidos, além do PIS e da COFINS de apuração própria, com o CST do PIS/COFINS da linha.
- **Tributos aproximados**: os valores do IBPT vão zerados. A lista de cidades deixa o campo **NFS-e suporta IBPT** desligado para esta integração.
- **IBS/CBS (Reforma Tributária)**: quando a linha do documento tem classificação tributária de IBS/CBS, a DPS leva o código indicador de operação, o CST, a classificação tributária e, conforme o caso, a tributação regular, o diferimento, o destinatário e os grupos de obra, imóvel e evento. A configuração desses cadastros está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md).
- **Tomador estrangeiro**: NIF (ou o motivo da não informação do NIF), endereço no exterior e as informações de comércio exterior.

O prestador é informado na DPS como **não optante** pelo Simples Nacional e sem regime especial de tributação. Os campos **Opção pelo Simples**, **Regime Especial de Tributação** e **Incentivo Cultural** da **Config. NF-e** não são enviados.

---

## 2. Configuração

### Certificado digital

O certificado digital autentica a conexão com a Governança Brasil e assina a DPS e o evento de cancelamento. Em Saquarema ele é exigido junto com o usuário e a senha do web service.

1. Use o certificado digital A1 (arquivo `.pfx`) da empresa, o mesmo usado na NF-e. Confirme com a prefeitura de Saquarema que a empresa está habilitada a emitir NFS-e por web service na plataforma da Governança Brasil.
2. Cadastre o certificado em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**).
3. Selecione-o no campo **ID Certificado** do grupo **Configuração Envio**, na aba **NFS-e** (veja a tabela abaixo).

O Business Central avisa quando o certificado está perto de vencer e **bloqueia a transmissão e o cancelamento** quando ele já venceu, com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Usuário e senha do web service

Além do certificado, Saquarema exige um usuário e uma senha de acesso ao web service da Governança Brasil. Solicite-os à prefeitura de Saquarema ou à Governança Brasil e informe-os no grupo **Configuração Envio** da aba **NFS-e**:

- **Nome Usuário WS**: o usuário, com até 30 caracteres;
- **Senha WS**: a senha, com até 50 caracteres. O campo aparece vazio quando a página é reaberta, mas a senha continua gravada. Para trocá-la, digite a nova senha; não digite e apague o conteúdo do campo, porque isso grava uma senha vazia.

O Business Central não confere o usuário e a senha antes do envio: se estiverem errados, em branco ou bloqueados, a plataforma recusa a transmissão (veja os erros na [seção 5](#5-pontos-de-atenção)). Quando a prefeitura trocar a senha, atualize **Senha WS**.

### Homologação

A Governança Brasil tem um ambiente de homologação. Antes do primeiro teste, confirme com a prefeitura ou com a Governança Brasil que a empresa está habilitada nesse ambiente.

Para usá-lo, ligue **Homologação** na **Config. NF-e** da filial: a DPS e o cancelamento passam a ser enviados ao ambiente de homologação. As notas emitidas ali não têm validade jurídica, e o DANFSe sai com a marca "NFS-e SEM VALIDADE JURÍDICA". Para emitir em produção, desligue **Homologação**.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para Saquarema - RJ (Governança Brasil) |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Saquarema - RJ (Governança Brasil)** na lista (código `RJ-Saquarema-GovBr`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (Indefinido). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial |
| **Configuração Envio** | **Homologação** | Ligado para emitir em homologação; desligado para produção |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. A Governança Brasil não informa o último número usado, então a numeração precisa ser controlada pelo Business Central |
| **Configuração Envio** | **WS NFS-e informa o IM** | Ligado, a **Inscrição Municipal** da filial vai na DPS; desligado, não vai. Ligue se a prefeitura exigir a inscrição municipal do prestador. A dica do campo diz que ele só é usado no Emissor Nacional, mas em Saquarema ele é usado |
| **Configuração Envio** | **Nome Usuário WS** | O usuário do web service fornecido pela prefeitura ou pela Governança Brasil (até 30 caracteres) |
| **Configuração Envio** | **Senha WS** | A senha desse usuário (até 50 caracteres). Aparece vazia ao reabrir a página, mas continua gravada (veja [Usuário e senha do web service](#usuário-e-senha-do-web-service)) |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão: é o endereço do serviço de integração da Localização, usado por todas as cidades |
| **Configuração Envio** | **ID Relatório p/ TXT** | Em branco. Só é usado por filial que gera a NFS-e por arquivo, sem web service |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | Um valor de `1` a `50`. A plataforma aceita até 50 documentos por lote, e com `0` (sem limite) a transmissão de mais de 50 documentos pendentes é recusada |
| **Lote** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Um dos dois é obrigatório: ligue **Usar Data/Hora como Nº Lote** ou informe uma série em **Nº Série Lote**. O número do lote fica só no Business Central |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Ligado**. É a forma de obter o DANFSe desta integração no Business Central (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Afeta só a impressão do DANFSe, conforme [NFS-e: Município de Incidência do ISSQN no DANFSe](Localization-for-Brazil_User-Manual_NFSe-DANFSe-Municipio-Incidencia-ISSQN.md) |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `1000`. A DPS aceita até 1000 caracteres na descrição do serviço e o texto que passa disso é cortado no envio; com o limite, o sistema avisa já no registro da venda |
| — | **Nº Série RPS/Temporário** | Série de numeração só com dígitos, usada pela ação **Criar Novo Nº** (veja [Numeração e série da DPS](#numeração-e-série-da-dps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente. Em branco, ele precisa ser informado em cada venda |
| — | **NFS-e suporta IBPT** | Desligado automaticamente ao escolher Saquarema. Não é editável |
| — | **Opção pelo Simples**, **Incentivo Cultural** e **Regime Especial de Tributação** | Não enviados. A DPS declara o prestador como não optante pelo Simples Nacional |

### Numeração e série da DPS

O layout nacional aceita **apenas dígitos** no número e na série da DPS:

- O número da DPS é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão. A série de numeração do tipo de documento fiscal usado pela filial, e também a de **Nº Série RPS/Temporário** (usada pela ação **Criar Novo Nº**), precisam gerar números só com dígitos, com até 15 posições (por exemplo, `000031` e não `NFS000031`).
- A **Série** do tipo de documento fiscal precisa estar preenchida, ter só dígitos e até 5 posições. Ela vem da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, do próprio tipo de documento fiscal.

Fora disso, a DPS é rejeitada por falha no esquema XML.

### Filial e cliente

- **Informação Filial** da filial: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos. Sem eles, a transmissão é barrada com a mensagem de campo obrigatório.
- **CEPs**: o CEP da filial cadastrado com **Cód. Municipio IBGE** de 7 dígitos (`3305505` para Saquarema). O município emissor da DPS é o desse CEP.
- **Cliente**: categoria, CEP, cidade, bairro, endereço, número, UF e país preenchidos (exceto pessoa física e cliente não identificado). Para cliente do exterior, **Cód. NIF** ou **Motivo da não informação do NIF** na aba Fiscal da ficha do cliente.

Diferentemente do Emissor Público Nacional, o Business Central não faz uma conferência completa antes do envio: ele barra a transmissão no primeiro dado obrigatório que faltar na filial ou no cliente, e os demais problemas de cadastro (códigos de serviço, por exemplo) voltam como rejeição da prefeitura, na tela.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Cód. Tributação** | O código de tributação **nacional**, de 6 dígitos (os 4 primeiros são o item da LC 116). Pontos são descartados no envio |
| **Item Lista Serviço** | O código de tributação **municipal**, quando a prefeitura tiver um. Em branco, não é enviado. Pontos e outros sinais não são retirados no envio |
| **Código CNAE** | O CNAE da atividade. Vai na DPS como código interno do contribuinte, com até 20 caracteres: mantenha-o preenchido |
| **Cód. NBS** | Código NBS do serviço. Obrigatório para cliente do exterior; quando preenchido, vai em todas as notas |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS (6 dígitos), enviado quando a linha tem classificação tributária de IBS/CBS |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos **Cód. Tributação**, **Item Lista Serviço** e **Código CNAE** desse registro substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item.

Obras, imóveis e atividades/eventos seguem o layout nacional, conforme [NFS-e: Local da Prestação — Obras, Atividades/Eventos e Imóveis](Localization-for-Brazil_User-Manual_NFSe-Local-Prestacao-Obras-Atividades-Imoveis.md).

### Códigos Tributação NFS-e

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. Em Saquarema, o campo **Cód. Tributação** desse código precisa ter um dos valores nacionais da tributação do ISSQN:

| Valor | Significado nacional |
| --- | --- |
| `1` | Operação Tributável |
| `2` | Imunidade |
| `3` | Exportação de Serviço |
| `4` | Não Incidência |

Um valor fora da lista é recusado na transmissão com "Código de Tributação inválido: &lt;valor&gt;". A lista de valores e a coluna **Significado nacional** da página só funcionam para filiais do Emissor Público Nacional: em Saquarema, a página informa que "o valor segue a tabela da prefeitura", mas vale a tabela acima.

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Docs. Pendentes**: transmite os documentos pendentes da filial num único lote, respeitando **Nº Máx. de Docs. por Lote WS**.
- **Transmitir Doc. Selecionado**: transmite só o documento selecionado.

Quando a prefeitura autoriza a nota, o documento recebe na hora o **Nº NFS** e a chave de acesso, e o lote fica com **Status** **Sucesso**. Os campos **Cód. Verificação NFS** e **Data NFS** não são preenchidos nesta integração: a nota é identificada pela chave de acesso.

Quando a prefeitura rejeita a DPS, as mensagens aparecem na tela, uma por linha, no formato `<código>: <descrição>`, e o documento continua pendente.

### Consultar o processamento

Não há consulta: o resultado já vem na transmissão. **Não use Consultar** em lotes de Saquarema, nem na página **NFS-e - Envio de Lote** nem na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`). Na página **NFS-e - Status de Lote RPS**, a consulta coloca o lote com **Status** **Erro**, inclusive um lote já **Sucesso**, com a mensagem "O serviço de RJ-Saquarema-GovBr é síncrono e não necessita de consulta ao lote.".

Como a nota autorizada não tem **Cód. Verificação NFS**, o documento desse lote volta a aparecer como pendente na **NFS-e - Envio de Lote**. **Não o transmita de novo**: a nota já está autorizada. Para devolver o lote ao **Status** **Sucesso**:

1. Abra cada documento do lote no **Documento Fiscal Registrado** e use **Editar Campos**.
2. Preencha **Data NFS** com a data e a hora de emissão da NFS-e e **Cód. Verificação NFS** com o próprio **Nº NFS** da nota (o **Nº NFS** já está preenchido).
3. Na página **NFS-e - Status de Lote RPS**, selecione o lote e use **Marcar como Sucesso**.

### Corrigir e reenviar

Quando a transmissão é rejeitada, corrija o cadastro apontado pelas mensagens e transmita o documento novamente. O reenvio mantém o número e a série da DPS. Se for preciso usar outro número, a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote** atribui ao documento o próximo número de **Nº Série RPS/Temporário**.

### Primeira emissão em homologação

1. Confirme com a prefeitura ou com a Governança Brasil que a empresa está habilitada no ambiente de homologação e qual usuário e senha usar nesse ambiente.
2. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** ligado.
3. Confira os códigos de serviço, o código de tributação NFS-e e os cadastros da filial e do cliente.
4. Registre uma fatura de venda de serviço para a filial.
5. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**.
6. Corrija o que a rejeição da prefeitura apontar e transmita de novo.
7. Com a nota autorizada, abra o **Documento Fiscal Registrado** e use **Abrir DANFSe** (veja a [seção 4](#4-danfse)).

### Cancelar

O cancelamento tem dois passos, no **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`):

1. **Cancelar Documento Fiscal** (grupo **Contabilização**): estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central. Nada é enviado à prefeitura neste passo.
2. **Cancelar NFS-e** (grupo **NFS-e**), habilitado depois do primeiro passo: envia à Governança Brasil o evento de cancelamento, com a chave de acesso da nota e o motivo "Erro na Emissão".

Quando a prefeitura registra o evento, a mensagem "NFS-e cancelada com sucesso." é exibida e o campo **NFS Cancelada** do documento é marcado. Se ela recusar, a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" lista os motivos e a NFS-e continua autorizada; corrija o que for apontado e use **Cancelar NFS-e** de novo.

---

## 4. DANFSe

Com **Utilizar Layout Padrão** ligado, o DANFSe da nota autorizada é gerado no padrão nacional a partir da chave de acesso, pelas ações **Abrir DANFSe** e **Atualizar DANFSe** do **Documento Fiscal Registrado**. O layout, a marca d'água de homologação e a de cancelamento estão descritos em [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md).

O Layout Padrão é a única forma de obter o DANFSe de Saquarema no Business Central: a Governança Brasil devolve a chave de acesso da nota, mas não um código de verificação nem uma imagem para download. Mantenha **Utilizar Layout Padrão** sempre ligado. Com o campo desligado:

- **Abrir DANFSe** termina com a mensagem de que **Cód. Verificação NFS** deve ter um valor;
- **Atualizar DANFSe** termina com "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e";
- o e-mail do documento fica pendente com o detalhe "Sem Cód. Verificação NFS: use o Layout Padrão", a menos que **NFS-e sem Anexo** (aba **E-Mails de Documento** da **Config. NF-e**) esteja ligado.

Ligue o campo e use **Atualizar DANFSe**.

A data e a hora de emissão da NFS-e não são gravadas na autorização, e o DANFSe sai com esse dado em branco. Para que ele apareça, preencha **Data NFS** no **Documento Fiscal Registrado** pela ação **Editar Campos** e use **Atualizar DANFSe**.

---

## 5. Pontos de Atenção

- **Certificado vencido**: a transmissão e o cancelamento são bloqueados até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **Consultar**: não use em lotes desta integração (veja [Consultar o processamento](#consultar-o-processamento)).
- **Mostrar Somente Pendentes**: na página **NFS-e - Envio de Lote**, as notas já autorizadas continuam aparecendo com esse filtro ligado, porque não têm **Cód. Verificação NFS**. O **Status NFS-e** **Sucesso** e o **Nº NFS** indicam as autorizadas, e as ações de transmissão não as reenviam.
- **Discriminação do serviço**: as quebras de linha da discriminação viram espaços na DPS.
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| Mensagem de campo obrigatório da filial (**Inscrição Municipal**, CEP, **Cód. Municipio IBGE**) | Completar a **Informação Filial** ou o cadastro de **CEPs** |
| "Nenhuma linha da nota fiscal possui Código de Serviço associado." | Preencher o **Cód. Serviço** na ficha do item de serviço |
| "Código de Tributação inválido: &lt;valor&gt;" | Ajustar o **Cód. Tributação** do código em **Códigos Tributação NFS-e** para `1`, `2`, `3` ou `4` |
| "RJ-Saquarema-GovBr permite no máximo 50 RPS por lote. Foram informados &lt;n&gt;." | Informar de `1` a `50` em **Nº Máx. de Docs. por Lote WS** e transmitir de novo |
| `<código>: <descrição>` | Rejeição da prefeitura: corrigir o cadastro indicado na descrição e transmitir de novo |
| "Ocorreu um erro desconhecido ao enviar o lote: &lt;resposta&gt;" | A plataforma respondeu fora do padrão, por indisponibilidade ou por recusa de acesso. Conferir **Nome Usuário WS** e **Senha WS** e transmitir de novo; se persistir, acionar o suporte |
| Mensagem curta, sem código, com o status da comunicação | A plataforma recusou a conexão. Conferir **Nome Usuário WS**, **Senha WS** e o certificado em **ID Certificado**; se estiverem corretos, confirmar a habilitação da empresa junto à prefeitura |
| "O serviço retornou um status de erro por razões desconhecidas." | A resposta veio sem nota e sem mensagem. Transmitir de novo; se persistir, acionar o suporte |
| "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. ..." | Cadastrar o certificado renovado e selecioná-lo em **ID Certificado** |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "O serviço de RJ-Saquarema-GovBr é síncrono e não necessita de consulta ao lote." | O lote foi consultado. Não retransmitir; seguir os passos de [Consultar o processamento](#consultar-o-processamento) |
| "Os seguintes erros ocorreram no cancelamento da NFS-e:" seguida de "Ocorreu um erro ao cancelar a NFS-e: &lt;resposta&gt;" | A plataforma respondeu fora do padrão. Usar **Cancelar NFS-e** de novo mais tarde |
| Mensagem de que **Cód. Verificação NFS** deve ter um valor, ao abrir o DANFSe | Ligar **Utilizar Layout Padrão** e usar **Atualizar DANFSe** |

- **Município de emissão**: o município emissor é o do CEP da filial. Um CEP com código IBGE errado envia a DPS para outro município.
- **Log de comunicação**: o envio e o retorno de cada operação ficam registrados no log do serviço de integração, consultado pelo suporte. No Business Central ficam as mensagens do lote e os dados de retorno do documento.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Saquarema - RJ (Governança Brasil)** (`RJ-Saquarema-GovBr`) |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Nome Usuário WS** | Usuário do web service |
| **Config. NF-e** | **Senha WS** | Senha do web service |
| **Config. NF-e** | **Homologação** | Conforme o ambiente |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **WS NFS-e informa o IM** | Ligado se a prefeitura exige a inscrição municipal |
| **Config. NF-e** | **Nº Máx. de Docs. por Lote WS** | De `1` a `50` |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Um dos dois preenchido |
| **Config. NF-e** | **Utilizar Layout Padrão** | Ligado |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Série de numeração só com dígitos |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Config. NF-e** | **Tamanho Máx. Discriminação** | `1000` |
| **Série Filial Tipo Doc. Fiscal** | **Série** | Preenchida, só com dígitos, até 5 posições |
| **Informação Filial** | **Inscrição Municipal** | Preenchida |
| **Códigos de Serviço NFS** | **Cód. Tributação** | Código de tributação nacional (6 dígitos) |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Código de tributação municipal, quando houver |
| **Códigos de Serviço NFS** | **Código CNAE** | CNAE da atividade |
| **Códigos de Serviço NFS** | **Cód. NBS** | Código NBS (obrigatório para cliente do exterior) |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | `1`, `2`, `3` ou `4` |
