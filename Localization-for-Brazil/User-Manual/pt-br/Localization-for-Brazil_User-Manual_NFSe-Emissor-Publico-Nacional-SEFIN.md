<!-- PR: #274, #322, #346 -->
# NFS-e: Emissão pelo Emissor Público Nacional (SEFIN)

**Data**: Outubro/2026

---

## Sumário

As filiais de municípios que emitem a NFS-e pelo **Emissor Público Nacional** transmitem a nota direto ao ambiente nacional da NFS-e (Sefin Nacional), sem passar por um sistema da prefeitura. Na lista de cidades de NFS-e, essa integração aparece como **Emissor Público Nacional** (código `SEFIN`) e vale para qualquer município que emita pelo ambiente nacional. A relação dos municípios atendidos será publicada em [NFS-e: Manuais de Emissão por Município e Plataforma](Localization-for-Brazil_User-Manual_NFSe-Manuais-por-Municipio.md).

A autenticação é feita só pelo **certificado digital** da filial. A autorização é **imediata**: a resposta da transmissão já traz a NFS-e autorizada, com número e chave de acesso, ou a lista de erros. Não há consulta posterior. O DANFSe é gerado no próprio Business Central, no layout padrão nacional. Os dados do prestador, inclusive o telefone com DDD e o e-mail, vêm da **Informação Filial** da filial emitente.

---

## 1. Contexto

### O que muda para a filial que emite pelo Emissor Público Nacional

| Tema | Como funciona no Emissor Público Nacional |
| --- | --- |
| **Layout** | DPS do layout da NFS-e Nacional, assinada com o certificado digital da filial |
| **Autenticação** | Certificado digital da filial. Não há usuário, senha nem token |
| **Município emissor** | Identificado pelo código IBGE do CEP da filial |
| **Contato do prestador** | Telefone e e-mail da **Informação Filial**, impressos no quadro do emitente da NFS-e nacional (veja [Filial e cliente](#filial-e-cliente)) |
| **Conferência antes do envio** | O sistema confere os cadastros da filial, do cliente e do documento e lista numa única mensagem tudo o que falta |
| **Envio** | Um documento por vez, mesmo quando vários documentos são transmitidos juntos |
| **Autorização** | Imediata. A resposta da transmissão grava o número da NFS-e e a chave de acesso nacional |
| **Rejeição** | As mensagens do ambiente nacional ficam no lote do documento ou aparecem na tela (veja a [seção 3](#3-emissão-consulta-e-cancelamento)) |
| **Consulta** | Não existe. O resultado já vem na transmissão |
| **Cancelamento** | Evento de cancelamento do layout nacional, com a chave de acesso da nota |
| **DANFSe** | Layout padrão nacional, gerado no Business Central a partir da chave de acesso |
| **Ambientes** | Homologação (produção restrita do ambiente nacional) e produção |

### Tributos na DPS

- **ISS**: a DPS leva a tributação do ISSQN (o valor nacional do **Cód. Tributação NFS** do documento, veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)), a retenção (quando o documento tem ISS retido) e o município de prestação. A **alíquota do ISS não é enviada**: o ambiente nacional usa a alíquota parametrizada pelo município.
- **Retenções federais**: PIS, COFINS, CSLL, IRRF e INSS retidos, além do PIS e da COFINS de apuração própria, com o CST do PIS/COFINS da linha.
- **Tributos aproximados**: os valores federal, estadual e municipal do IBPT. Ao escolher o Emissor Público Nacional, o campo **NFS-e suporta IBPT** é ligado automaticamente.
- **IBS/CBS (Reforma Tributária)**: quando a linha do documento tem classificação tributária de IBS/CBS, a DPS leva o código indicador de operação, o CST, a classificação tributária e, conforme o caso, a tributação regular, o diferimento, o destinatário e os grupos de obra, imóvel e evento. A configuração desses cadastros está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md).
- **Tomador estrangeiro**: NIF (ou o motivo da não informação do NIF), endereço no exterior e as informações de comércio exterior.

O prestador é informado na DPS como **não optante** pelo Simples Nacional e sem regime especial de tributação. Os campos **Opção pelo Simples**, **Regime Especial de Tributação** e **Incentivo Cultural** da **Config. NF-e** não são enviados.

---

## 2. Configuração

### Certificado digital

O certificado digital é a única credencial do Emissor Público Nacional: ele autentica a conexão com o ambiente nacional e assina a DPS e o evento de cancelamento.

1. Use o certificado digital A1 (arquivo `.pfx`) da empresa, o mesmo usado na NF-e. O município da filial precisa emitir pelo ambiente nacional: confirme com a prefeitura antes de configurar a filial.
2. Cadastre o certificado em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**).
3. Selecione-o no campo **ID Certificado** da aba **NFS-e** (veja a tabela abaixo).

O Business Central avisa quando o certificado está perto de vencer e **bloqueia a transmissão** quando ele já venceu, com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação (produção restrita)

O ambiente nacional tem um ambiente de testes, a **produção restrita**. Para usá-lo, ligue **Homologação** na **Config. NF-e** da filial: a DPS e o cancelamento passam a ser enviados à produção restrita. As notas emitidas ali não têm validade jurídica, e o DANFSe sai com a marca "NFS-e SEM VALIDADE JURÍDICA". Para emitir em produção, desligue **Homologação**.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para o Emissor Público Nacional |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Emissor Público Nacional** na lista (código `SEFIN`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (Indefinido). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial |
| **Configuração Envio** | **Homologação** | Ligado para emitir na produção restrita; desligado para produção |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. O ambiente nacional não informa o último número usado, então a numeração precisa ser controlada pelo Business Central |
| **Configuração Envio** | **WS NFS-e informa o IM** | Ligue se o município exige a inscrição municipal do prestador na DPS. Desligado, a inscrição municipal não é enviada |
| **Configuração Envio** | **Nome Usuário WS** e **Senha WS** | Não usados pelo Emissor Público Nacional |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão: é o endereço do serviço de integração da Localização, usado por todas as cidades |
| **Configuração Envio** | **ID Relatório p/ TXT** | Em branco. Só é usado por filial que gera a NFS-e por arquivo, sem web service |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | Não usado: o sistema transmite um documento por vez. Deixe `0`; com outro valor, a página de envio avisa que só parte dos documentos será enviada, mas todos são transmitidos |
| **Lote** | **Usar Data/Hora como Nº Lote** e **Nº Série Lote** | Não usados: o lote recebe o número do documento fiscal |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Ligado**. É a forma de obter o DANFSe no Business Central (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Afeta só a impressão do DANFSe, conforme [NFS-e: Município de Incidência do ISSQN no DANFSe](Localization-for-Brazil_User-Manual_NFSe-DANFSe-Municipio-Incidencia-ISSQN.md) |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `1000`. O ambiente nacional aceita até 1000 caracteres na descrição do serviço e o texto que passa disso é cortado no envio; com o limite, o sistema avisa já no registro da venda |
| — | **Nº Série RPS/Temporário** | Série de numeração só com dígitos (veja [Numeração e série da DPS](#numeração-e-série-da-dps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente. Em branco, ele precisa ser informado em cada venda |
| — | **NFS-e suporta IBPT** | Ligado automaticamente ao escolher o Emissor Público Nacional |
| — | **Opção pelo Simples**, **Incentivo Cultural** e **Regime Especial de Tributação** | Não enviados. A DPS declara o prestador como não optante pelo Simples Nacional |

Ao trocar o **Cód. Cidade WS** de uma filial para **Emissor Público Nacional**, o sistema avisa quantos códigos de **Códigos Tributação NFS-e** da filial estão com valor não reconhecido pelo padrão nacional (veja [Códigos Tributação NFS-e](#códigos-tributação-nfs-e)).

### Numeração e série da DPS

O layout nacional aceita **apenas dígitos** no número e na série da DPS:

- O número da DPS é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão. A série de numeração do tipo de documento fiscal usado pela filial, e também a de **Nº Série RPS/Temporário** (usada pela ação **Criar Novo Nº**), precisam gerar números só com dígitos, com até 15 posições (por exemplo, `000031` e não `NFS000031`).
- A **Série** do tipo de documento fiscal precisa estar preenchida, ter só dígitos e até 5 posições. Ela vem da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, do próprio tipo de documento fiscal.

Fora disso, o ambiente nacional rejeita a DPS por falha no esquema XML.

### Filial e cliente

O sistema confere estes cadastros antes de transmitir (veja [Transmitir](#transmitir)):

- **Informação Filial** da filial: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos.
- **CEPs**: o CEP da filial e o do cliente cadastrados, com **Cód. Municipio IBGE** de 7 dígitos.
- **Cliente**: categoria, CEP, cidade, bairro, endereço, número, UF e país preenchidos (exceto pessoa física e cliente não identificado). Para cliente do exterior, **Cód. NIF** ou **Motivo da não informação do NIF** na aba Fiscal da ficha do cliente, e o país com o código ISO de duas letras.

A DPS leva também o contato da filial como prestador, impresso no quadro do emitente da NFS-e autorizada pelo ambiente nacional. Os dois campos ficam na **Informação Filial** (página `BranchInformationFNX`), grupo **Comunicação**, e não são conferidos antes do envio:

| Campo | O que vai na NFS-e |
| --- | --- |
| **Telefone** | Todos os dígitos do campo, com o DDD: `(21) 99876-5432` é enviado como `21998765432`. Informe o telefone com DDD e sem o código do país |
| **E-Mail** | O e-mail do prestador. Quando o campo tem mais de um endereço, separados por `;` ou `,`, só o primeiro é enviado: com `nfse@filial.com.br;financeiro@filial.com.br`, a nota leva `nfse@filial.com.br`. Com o campo em branco, a NFS-e sai sem e-mail do prestador |

A NFS-e guarda o telefone e o e-mail da data da transmissão: corrigir a **Informação Filial** depois não altera as notas já autorizadas. O DANFSe da nota emitida, gerado no Business Central a partir do XML da NFS-e autorizada (veja a [seção 4](#4-danfse)), mostra o telefone e o e-mail que constam na nota; quando a nota não tem e-mail do prestador, o campo sai com "-".

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Cód. Tributação** | O código de tributação **nacional**, de 6 dígitos (os 4 primeiros são o item da LC 116). Pontos são descartados no envio |
| **Item Lista Serviço** | O código de tributação **municipal**, quando o município tem um: **exatamente 3 dígitos**, ou em branco. Com outro formato, a transmissão é barrada |
| **Cód. NBS** | Código NBS do serviço. Obrigatório para cliente do exterior; quando preenchido, vai em todas as notas |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS (6 dígitos), enviado quando a linha tem classificação tributária de IBS/CBS |
| **Código CNAE** | Não enviado ao ambiente nacional |

Para escolher o **Cód. Operação de Serviços** de cada serviço, use as tabelas oficiais da NFS-e Nacional:

- [Anexo VII — Tabela de códigos indicadores de operação (indOp) do IBS/CBS](https://view.officeapps.live.com/op/view.aspx?src=https%3A%2F%2Fwww.gov.br%2Fnfse%2Fpt-br%2Fbiblioteca%2Fdocumentacao-tecnica%2Frtc%2Fanexovii-indop_ibscbs_v1-03-00-nt009.xlsx&wdOrigin=BROWSELINK);
- [Anexo VIII — Correlação entre item da LC 116, NBS, indOp e classificação tributária do IBS/CBS](https://view.officeapps.live.com/op/view.aspx?src=https%3A%2F%2Fwww.gov.br%2Fnfse%2Fpt-br%2Fbiblioteca%2Fdocumentacao-tecnica%2Frtc%2Fanexoviii-correlacaoitemnbsindopcclasstrib_ibscbs_v1-01-00.xlsx&wdOrigin=BROWSELINK), que indica, para cada serviço, o código indicador de operação e a classificação tributária a usar.

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos **Cód. Tributação** e **Item Lista Serviço** desse registro substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item.

### Códigos Tributação NFS-e

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. No Emissor Público Nacional, o campo **Cód. Tributação** desse código precisa ter um dos valores nacionais da tributação do ISSQN:

| Valor | Significado nacional |
| --- | --- |
| `1` | Operação Tributável |
| `2` | Imunidade |
| `3` | Exportação de Serviço |
| `4` | Não Incidência |

A coluna **Significado nacional** mostra o significado de cada valor e indica "Não reconhecido" quando o valor não é um dos quatro. Um valor fora da lista é recusado na transmissão com "Código de Tributação inválido: &lt;valor&gt;".

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Docs. Pendentes**: transmite todos os documentos pendentes da filial, um de cada vez. Um documento rejeitado não impede o envio dos demais.
- **Transmitir Doc. Selecionado**: transmite só o documento selecionado.

Antes de enviar, o sistema confere os cadastros da filial, do cliente e do documento, e o documento com pendência não é enviado:

- em **Transmitir Doc. Selecionado**, a página **Mensagens de Erro** lista todas as pendências de uma vez, cada uma com o cadastro a corrigir;
- em **Transmitir Docs. Pendentes**, as pendências da filial aparecem uma única vez na página **Mensagens de Erro** e nada é enviado; as pendências de cada documento ficam em **Lista Erros** do lote dele, e os demais documentos seguem.

Entre as conferências:

| Situação | O que corrigir |
| --- | --- |
| Filial sem nome, CNPJ, **Inscrição Municipal**, cidade ou CEP | **Informação Filial** |
| CEP da filial ou do cliente não cadastrado, ou sem **Cód. Municipio IBGE** de 7 dígitos | **CEPs** |
| Filial sem certificado ou com certificado inexistente | **ID Certificado** na **Config. NF-e** |
| Cliente com endereço incompleto | Ficha do cliente |
| Cliente do exterior sem **Cód. NIF** nem **Motivo da não informação do NIF** | Ficha do cliente, aba Fiscal |
| Cliente do exterior e código de serviço sem **Cód. NBS** | **Códigos de Serviço NFS** |
| **Item Lista Serviço** preenchido sem ter 3 dígitos | **Códigos de Serviço NFS** ou **Códigos Serviço NFS por Filial** |
| Documento sem **Município Prestação Serviço** ou sem **Cód. Tributação NFS** | **Documento Fiscal Registrado**, grupo NFS, ação **Editar Campos** |
| Linha sem item do tipo Serviço com código de serviço | Ficha do item |
| Imposto da nota sem **Não Destacar** e com **Classificação** em branco ou **Outro**, que não tem campo na NFS-e | **Classificação** na **Jurisdição Imposto**; depois cancelar o documento fiscal e lançá-lo de novo (veja [Imposto com Classificação sem Campo na NFS-e](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md#imposto-com-classificação-sem-campo-na-nfs-e)) |

Quando o ambiente nacional autoriza a nota, o documento recebe na hora o **Nº NFS**, a **Data NFS** e a chave de acesso nacional, e o lote fica com **Status** **Sucesso**. O campo **Cód. Verificação NFS** recebe o próprio número da NFS-e.

Quando o ambiente nacional rejeita a DPS:

- em **Transmitir Docs. Pendentes**, o lote do documento fica com **Status** **Erro** e a rejeição fica em **Lista Erros**, no formato `Erro <código>: <descrição> - <complemento>`. A linha guarda até 250 caracteres: se a mensagem aparecer cortada, transmita o documento com **Transmitir Doc. Selecionado** para vê-la inteira;
- em **Transmitir Doc. Selecionado**, as mensagens aparecem na tela e o documento continua pendente.

### Consultar o processamento

Não há consulta: o resultado já vem na transmissão. **Não use Consultar** em lotes do Emissor Público Nacional, nem na página **NFS-e - Envio de Lote** nem na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`). Na página **NFS-e - Envio de Lote**, um lote já **Sucesso** não é consultado. Na página **NFS-e - Status de Lote RPS**, a consulta coloca o lote com **Status** **Erro**, inclusive um lote já **Sucesso**, e troca as mensagens originais pela mensagem "O serviço de SEFIN é síncrono e não necessita de consulta ao lote.". A nota autorizada não é afetada: o documento continua com **Nº NFS** e não volta a ficar pendente. Para devolver o lote ao **Status** **Sucesso**, use **Marcar como Sucesso** na página **NFS-e - Status de Lote RPS**.

### Corrigir e reenviar

Quando a transmissão é rejeitada, corrija o cadastro apontado pelas mensagens e transmita o documento novamente. O reenvio mantém o número e a série da DPS. Se for preciso usar outro número, a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote** atribui ao documento o próximo número de **Nº Série RPS/Temporário**.

### Primeira emissão em homologação

1. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** ligado.
2. Confira os códigos de serviço, o código de tributação NFS-e e os cadastros da filial e do cliente.
3. Registre uma fatura de venda de serviço para a filial.
4. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**.
5. Corrija o que a página **Mensagens de Erro** ou a rejeição do ambiente nacional apontar e transmita de novo.
6. Com a nota autorizada, abra o **Documento Fiscal Registrado** e use **Abrir DANFSe** (veja a [seção 4](#4-danfse)).

### Cancelar

O cancelamento tem dois passos, no **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`):

1. **Cancelar Documento Fiscal** (grupo **Contabilização**): estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central. Nada é enviado ao ambiente nacional neste passo.
2. **Cancelar NFS-e** (grupo **NFS-e**), habilitado depois do primeiro passo: envia ao ambiente nacional o evento de cancelamento, com a chave de acesso da nota e o motivo "Erro na Emissão".

Quando o ambiente nacional registra o evento, a mensagem "NFS-e cancelada com sucesso." é exibida e o campo **NFS Cancelada** do documento é marcado. Se ele recusar, a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" lista os motivos e a NFS-e continua autorizada; corrija o que for apontado e use **Cancelar NFS-e** de novo.

---

## 4. DANFSe

Com **Utilizar Layout Padrão** ligado, o DANFSe da nota autorizada é gerado no padrão nacional a partir da chave de acesso, pelas ações **Abrir DANFSe** e **Atualizar DANFSe** do **Documento Fiscal Registrado**. O layout, a marca d'água de homologação e a de cancelamento estão descritos em [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md).

O Layout Padrão é a forma de obter o DANFSe do Emissor Público Nacional no Business Central: mantenha **Utilizar Layout Padrão** sempre ligado nas filiais desta integração. Com o campo desligado, **Abrir DANFSe** não gera o documento e termina com a mensagem "Erro ao baixar o DANFSe: &lt;motivo&gt; - &lt;resposta&gt;"; ligue o campo e use **Atualizar DANFSe**.

---

## 5. Pontos de Atenção

- **Certificado vencido**: a transmissão e o cancelamento são bloqueados até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **Consultar**: não use em lotes desta integração (veja [Consultar o processamento](#consultar-o-processamento)).
- **Erros mais comuns**:

| Mensagem | O que fazer |
| --- | --- |
| Pendências listadas em **Mensagens de Erro** antes do envio | Corrigir cada cadastro apontado e transmitir de novo |
| "O imposto da Jurisdição Imposto &lt;código&gt; tem a Classificação &lt;classificação&gt;, que não tem campo correspondente na NFS-e: ..." | Corrigir a **Classificação** na **Jurisdição Imposto**, cancelar o documento fiscal e lançá-lo de novo: a classificação fica gravada na nota no lançamento |
| "Código de Tributação inválido: &lt;valor&gt;" | Ajustar o **Cód. Tributação** do código em **Códigos Tributação NFS-e** para `1`, `2`, `3` ou `4` |
| "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. ..." | Cadastrar o certificado renovado e selecioná-lo em **ID Certificado** |
| Rejeição por falha no esquema XML apontando número ou série da DPS | Ajustar as séries para só dígitos (veja [Numeração e série da DPS](#numeração-e-série-da-dps)) |
| "Erro ao consultar último número de RPS: ..." | Ligar **Controlar Nº RPS** |
| "O serviço de SEFIN é síncrono e não necessita de consulta ao lote." | O lote foi consultado. Se o documento tem **Nº NFS**, usar **Marcar como Sucesso**; se não tem, transmitir de novo |
| "Erro ao baixar o DANFSe: ..." | Ligar **Utilizar Layout Padrão** e usar **Atualizar DANFSe** |
| Demais `Erro <código>: <descrição>` | Rejeição do ambiente nacional: corrigir o cadastro indicado na descrição e transmitir de novo |

- **Município de emissão**: o município emissor é o do CEP da filial. Um CEP com código IBGE errado envia a DPS para outro município.
- **Log de comunicação**: o envio e o retorno de cada operação ficam registrados no log do serviço de integração, consultado pelo suporte. No Business Central ficam as mensagens do lote e os dados de retorno do documento.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Emissor Público Nacional** (`SEFIN`) |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Homologação** | Conforme o ambiente |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **WS NFS-e informa o IM** | Ligado se o município exige a inscrição municipal |
| **Config. NF-e** | **Nº Máx. de Docs. por Lote WS** | `0` |
| **Config. NF-e** | **Utilizar Layout Padrão** | Ligado |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Série de numeração só com dígitos |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Config. NF-e** | **Tamanho Máx. Discriminação** | `1000` |
| **Série Filial Tipo Doc. Fiscal** | **Série** | Preenchida, só com dígitos, até 5 posições |
| **Informação Filial** | **Inscrição Municipal** | Preenchida |
| **Informação Filial** | **Telefone** | Com DDD, sem o código do país |
| **Informação Filial** | **E-Mail** | E-mail do prestador; com mais de um endereço, o primeiro vai na NFS-e |
| **Códigos de Serviço NFS** | **Cód. Tributação** | Código de tributação nacional (6 dígitos) |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Código de tributação municipal com 3 dígitos, ou em branco |
| **Códigos de Serviço NFS** | **Cód. NBS** | Código NBS (obrigatório para cliente do exterior) |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | `1`, `2`, `3` ou `4` |
