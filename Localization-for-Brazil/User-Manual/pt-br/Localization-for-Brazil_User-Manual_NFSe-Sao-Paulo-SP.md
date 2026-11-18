<!-- PR: #274, #215, #309, #323 -->
# NFS-e: Emissão em São Paulo - SP

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **São Paulo - SP** emitem a NFS-e pelo Business Central por meio do web service da Prefeitura de São Paulo, no layout versão 2, que traz os campos da Reforma Tributária 2026. Na lista de cidades de NFS-e, a integração aparece como **São Paulo - SP 2026** (código `SP-SaoPaulo2026`). A integração cobre transmissão, consulta do processamento, reenvio após rejeição, cancelamento e o DANFSe em PDF da prefeitura.

A autenticação é feita só pelo **certificado digital** da filial. A autorização não é imediata: a prefeitura recebe o lote, devolve um protocolo e processa os RPS depois; o número da NFS-e só chega na **consulta**. O campo **Homologação** não aponta para um ambiente separado: ele usa o **envio de teste** da prefeitura, que confere o lote sem gerar NFS-e.

---

## 1. Contexto

### O que muda para a filial de São Paulo

| Tema | Como funciona em São Paulo - SP |
| --- | --- |
| **Layout** | RPS no layout versão 2 do web service da Prefeitura de São Paulo. Cada RPS e a mensagem são assinados com o certificado digital da filial |
| **Autenticação** | Certificado digital da filial, que precisa conter o CNPJ da filial. Não há usuário, senha nem token |
| **Código do serviço** | **Cód. Tributação** do código de serviço = código de serviço municipal, de 5 dígitos |
| **Tributação do ISS** | **Cód. Tributação** de **Códigos Tributação NFS-e** = letra de tributação da prefeitura (T, F, A, ...) |
| **Envio** | Vários RPS por lote, com até 500 KB por mensagem. A prefeitura devolve um protocolo, e a nota ainda não está autorizada |
| **Autorização** | Conhecida na consulta do lote: grava o número da NFS-e, a data e o código de verificação |
| **Rejeição** | Por RPS: os RPS válidos viram NFS-e mesmo que outros do mesmo lote sejam rejeitados |
| **Cancelamento** | Pelo sistema, pelo número e código de verificação da nota |
| **DANFSe** | PDF da prefeitura, baixado pelo Business Central (só em produção) |
| **Ambientes** | Produção. Com **Homologação** ligado, o envio usa o teste da prefeitura, que não gera NFS-e |

### Tributos enviados

- **ISS**: alíquota e indicação de ISS retido. Quando a alíquota informada difere da vigente para o código de serviço, a prefeitura adota a vigente.
- **Valor**: o valor final cobrado (valor dos serviços menos o desconto incondicional). As **deduções da base de cálculo não são enviadas**.
- **Retenções federais**: IR e INSS retidos. O campo de CSLL leva a soma de PIS, COFINS e CSLL retidos, com o tipo de retenção correspondente. Os campos de PIS e COFINS levam os valores de apuração própria do documento.
- **Tributos aproximados (IBPT)**: a carga tributária do IBPT, quando **Texto Legal Valor Aprox. Impostos Automático** está ligado. O campo **NFS-e suporta IBPT** é ligado automaticamente ao escolher São Paulo.
- **IBS/CBS (Reforma Tributária)**: o código NBS, o código indicador de operação, a classificação tributária (e a classificação regular, quando houver), a finalidade "NFS-e regular" e o indicador de uso ou consumo pessoal "Não". Para os itens 10.05, 15.09, 17.12 e 25.05 da LC 116 vai também o tipo de operação "fornecimento e recebimento do pagamento concomitantes". Conforme o caso, seguem os grupos de destinatário, obra ou imóvel e evento. **Alíquotas e valores de IBS/CBS não são enviados.** A configuração desses cadastros está em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md) e em [NFS-e: Local da Prestação — Obras, Atividades e Imóveis](Localization-for-Brazil_User-Manual_NFSe-Local-Prestacao-Obras-Atividades-Imoveis.md).
- **Valores fixos**: o RPS sai sem exigibilidade suspensa, sem pagamento parcelado antecipado e com IPI zero. O **Tipo de Suspensão da ISS** do código de serviço não é enviado: para exigibilidade suspensa, use as letras de tributação **X** ou **V** (veja [Letra de tributação](#letra-de-tributação-códigos-tributação-nfs-e)).
- **Tomador estrangeiro**: NIF (sem ele, a nota informa que o NIF não foi informado), endereço no exterior e, quando o serviço é prestado fora do país, o país da prestação.

**Opção pelo Simples**, **Incentivo Cultural** e **Regime Especial de Tributação** da **Config. NF-e** não são enviados: o RPS da prefeitura não tem esses campos.

---

## 2. Configuração

### Certificado digital e autorização da prefeitura

O certificado digital é a única credencial de São Paulo: ele assina cada RPS, a mensagem do lote e o pedido de cancelamento, e autentica a conexão com a prefeitura.

1. Pelo manual do web service da prefeitura, o certificado deve ser emitido por autoridade certificadora da ICP-Brasil e **conter o CNPJ** do proprietário. O CNPJ do certificado tem de ser o **mesmo CNPJ da filial** que emite: a prefeitura confere a assinatura, a conexão e o remetente da mensagem (erros 1100 e 1107). A prefeitura valida também a validade, a cadeia e a revogação do certificado. No Business Central, use o certificado A1 (arquivo `.pfx`).
2. Cadastre o certificado em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**) e selecione-o no campo **ID Certificado** da aba **NFS-e**.
3. A inscrição municipal (CCM) da filial precisa estar ativa e autorizada a emitir NFS-e. Sem isso, a prefeitura rejeita o lote com 1202 (prestador não encontrado no CCM), 1207 (prestador não autorizado a emitir NF-e) ou 1208 (inscrição cancelada). Confirme a situação com a prefeitura antes da primeira emissão. Microempreendedor individual (MEI) não emite por este web service (erro 359): usa a NFS-e de padrão nacional.

O Business Central avisa quando o certificado está perto de vencer e **bloqueia a transmissão** quando ele já venceu, com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação (envio de teste)

A Prefeitura de São Paulo não tem um ambiente de homologação separado. Ela oferece, no próprio ambiente de produção, um **envio de teste** que faz exatamente as mesmas verificações do envio real, mas **não gera NFS-e**. O manual da prefeitura recomenda usá-lo só na fase de adaptação do sistema.

Com **Homologação** ligado na **Config. NF-e**:

- a **transmissão** usa o envio de teste: o lote recebe um protocolo, como no envio real;
- a **consulta** é a mesma do envio real e mostra os erros encontrados no teste;
- o **cancelamento** continua indo à produção (só se aplica a notas reais já emitidas);
- não há DANFSe, porque nenhuma nota é gerada.

O passo a passo está em [Primeira emissão em homologação](#primeira-emissão-em-homologação). Para emitir de verdade, desligue **Homologação**.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para São Paulo - SP |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **São Paulo - SP 2026** na lista (código `SP-SaoPaulo2026`). O código antigo `SP-SaoPaulo`, do layout anterior, não aparece mais na lista: se a filial ainda o tiver, selecione **São Paulo - SP 2026** e revise a configuração conforme este manual |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (Indefinido). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital com o CNPJ da filial |
| **Configuração Envio** | **Homologação** | Desligado para emitir. Ligado só para usar o envio de teste, que não gera nota |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. Desligado, o sistema pede à prefeitura o último número de RPS, consulta que não funciona com este web service, e a transmissão falha com "Erro ao consultar último número de RPS: ..." |
| **Configuração Envio** | **WS NFS-e informa o IM** | Não usado: a inscrição municipal (CCM) da filial é sempre enviada |
| **Configuração Envio** | **Nome Usuário WS** e **Senha WS** | Não usados por São Paulo |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão: é o endereço do serviço de integração da Localização, usado por todas as cidades |
| **Configuração Envio** | **ID Relatório p/ TXT** | Em branco. Só é usado por filial que gera a NFS-e por arquivo, sem web service |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | Um limite que mantenha a mensagem abaixo de **500 KB**, o máximo aceito pela prefeitura. Com discriminações longas, cada RPS ocupa alguns kilobytes; um valor como `50` deixa folga |
| **Lote** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Um dos dois. O número identifica o lote no Business Central e não é enviado à prefeitura |
| **Lote** | **Permitir Erro Lote Manual** | Ligado, se a filial usar o envio de teste: libera a ação **Marcar como Erro**, necessária para liberar o documento testado |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Desligado**. O DANFSe de São Paulo é o PDF da prefeitura (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Não usado (só afeta o DANFSe do Layout Padrão) |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `2000`, o tamanho que o manual da prefeitura recomenda para a discriminação dos serviços |
| — | **Nº Série RPS/Temporário** | Série com **Último Nº Usado** e números do mesmo comprimento da numeração fiscal (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente |
| — | **NFS-e suporta IBPT** | Ligado automaticamente ao escolher São Paulo |
| — | **Texto Legal Valor Aprox. Impostos Automático** | Ligado para enviar a carga tributária do IBPT |
| — | **Opção pelo Simples**, **Incentivo Cultural** e **Regime Especial de Tributação** | Não enviados |

### Numeração e série do RPS

- O número do RPS é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão. Ele vem da série do tipo de documento fiscal: **Nº Série Fiscal** da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, do próprio tipo de documento fiscal. O número precisa ter **só dígitos**, com até 12 posições; com outro formato, a transmissão é barrada com "Código de Número RPS: &lt;número&gt;".
- A **Série** do tipo de documento fiscal precisa estar preenchida e ter de 1 a 5 caracteres.
- A **Consulta** exige **Nº Série RPS/Temporário** preenchida e com **Último Nº Usado**. O sistema usa o comprimento desse número para localizar o documento devolvido pela prefeitura. Por isso, a série precisa gerar números com o mesmo formato e o mesmo comprimento do **Nº Nota Fiscal** (por exemplo, os dois com 6 dígitos). A mesma série é usada pela ação **Criar Novo Nº**.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Cód. Tributação** | O **código de serviço municipal**, de 5 dígitos, da tabela de serviços da Prefeitura de São Paulo. Pontos, hífens, barras e espaços são descartados no envio; com letras, a transmissão é barrada com "Código de atividade inválido: &lt;código&gt;" |
| **Item Lista Serviço** | O **item da LC 116** do serviço (por exemplo, `7.02` ou `0702`). Ele não vai à prefeitura: decide se a nota leva o grupo de obra (construção civil) e o tipo de operação dos itens 10.05, 15.09, 17.12 e 25.05. Preencha sempre: em branco, o item só é deduzido do código de serviço municipal para os códigos que a prefeitura relaciona a esses casos |
| **Cód. NBS** | Código NBS do serviço. **Obrigatório** no layout 2 |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS. **Obrigatório** |
| **Código CNAE** | Não enviado a São Paulo |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos desse registro substituem os do código de serviço, mesmo que estejam em branco.

As linhas do documento precisam ter a classificação tributária de IBS/CBS, que é obrigatória no layout 2. O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item.

### Letra de tributação (Códigos Tributação NFS-e)

O **Cód. Tributação NFS** do documento fiscal aponta para um código de **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial. Em São Paulo, o campo **Cód. Tributação** desse código é a **letra de tributação** da prefeitura:

| Letra | Tributação |
| --- | --- |
| `T` | Tributado em São Paulo |
| `F` | Tributado fora de São Paulo |
| `A` | Tributado em São Paulo, porém isento |
| `B` | Tributado fora de São Paulo, porém isento |
| `D` | Tributado em São Paulo com isenção parcial |
| `M` | Tributado em São Paulo, porém com indicação de imunidade subjetiva |
| `N` | Tributado fora de São Paulo, porém com indicação de imunidade subjetiva |
| `R` | Tributado em São Paulo, porém com indicação de imunidade objetiva |
| `S` | Tributado fora de São Paulo, porém com indicação de imunidade objetiva |
| `X` | Tributado em São Paulo, porém com exigibilidade suspensa |
| `V` | Tributado fora de São Paulo, porém com exigibilidade suspensa |
| `P` | Exportação de serviços |

Com as letras `F`, `B`, `N`, `S` e `V`, o **Município Prestação Serviço** do documento vai à prefeitura como município onde o serviço foi prestado. Uma letra fora da tabela é rejeitada com o erro 206 ("Tipo de tributação inválido").

Marque como **Padrão** a letra mais usada pela filial: ela preenche automaticamente o **Cód. Tributação NFS-e** do pedido de venda e o **Cód. Tributação NFS** do documento.

### Filial e cliente

- **Informação Filial** da filial: nome, CNPJ, **Inscrição Municipal** (CCM, até 12 dígitos), cidade e CEP preenchidos; o CEP precisa ter **Cód. Municipio IBGE** em **CEPs**. Se a inscrição municipal não tiver nenhum dígito, a transmissão é barrada com "Código de Inscrição Prestador: &lt;valor&gt;".
- **Cliente**: CPF ou CNPJ. Para cliente com CNPJ, a prefeitura exige endereço e cidade/UF (erros 317 e 318). A inscrição municipal do cliente só é enviada quando ele é do mesmo município da filial, como a prefeitura exige (erros 219 e 220).

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
| Inscrição municipal da filial sem dígitos | "Código de Inscrição Prestador: &lt;valor&gt;" |
| Número do RPS com caracteres que não são dígitos | "Código de Número RPS: &lt;número&gt;" |
| **Cód. Tributação** do serviço com letras | "Código de atividade inválido: &lt;código&gt;" |
| Certificado sem chave privada RSA utilizável | "Certificado não possui chave privada RSA válida" |
| Documento em lote ainda **Enviado** | "Lote &lt;nº&gt; deverá ser consultado antes do envio." |
| Nenhuma linha com item de serviço com código | "Nenhuma linha da nota fiscal possui Código de Serviço associado." |

Se a prefeitura recusar o lote na recepção, as mensagens dela aparecem na tela no formato `Erro <código>: <descrição>` e nada fica gravado. É o caso, por exemplo, de mensagem acima de 500 KB (erro 1101), de falha no esquema XML (1001) e de problemas no certificado (1050 a 1057).

Quando a prefeitura recebe o lote, ele fica com **Status** **Enviado** e os documentos ficam com **Status NFS-e** **Enviado**. A nota **ainda não está autorizada**: o número da NFS-e só chega na consulta.

### Consultar o processamento

Na página **NFS-e - Envio de Lote** (selecione o documento) ou na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`, selecione o lote), use **Consultar**.

| Resultado da consulta | O que acontece no Business Central |
| --- | --- |
| **Lote ainda em processamento** | O lote continua **Enviado**. Como a prefeitura não envia mensagem, a tela mostra "O serviço da prefeitura respondeu, mas não forneceu detalhes. Consulte o lote novamente mais tarde.". Consulte novamente em alguns minutos |
| **Todos os RPS autorizados** | O lote fica com **Status** **Sucesso**. Cada documento recebe o **Nº NFS**, a **Data NFS** (data de processamento do lote), o **Cód. Verificação NFS** e a **Chave de Acesso NFS-e** (a chave nacional, quando a prefeitura a devolve), e o **Nº Nota Fiscal** passa a ser o número da NFS-e |
| **Todos os RPS rejeitados** | O lote fica com **Status** **Erro** e as mensagens da prefeitura ficam em **Lista Erros**, no formato `RPS<nº do RPS>: <descrição>` |
| **Parte autorizada, parte rejeitada** | Os documentos autorizados passam para um lote novo com **Status** **Sucesso**, com os dados da nota. Os rejeitados passam para outro lote novo com **Status** **Erro**, e as mensagens em **Lista Erros** começam pelo número do documento fiscal. O lote consultado fica sem documentos e deixa de aparecer na página **NFS-e - Status de Lote RPS** |
| **Retorno sem parte dos RPS do lote** | Os RPS que constam do retorno são tratados um a um: os autorizados passam para um lote novo com **Status** **Sucesso**, com os dados da nota, e os rejeitados para outro lote novo com **Status** **Erro**. Os documentos que não constam do retorno continuam no lote consultado, que fica **Enviado** e pode ser consultado de novo |
| **Lote recusado no processamento** | O lote fica com **Status** **Erro** e a descrição da prefeitura fica em **Lista Erros** (por exemplo, "Não foi possível processar a requisição, por favor, envie novamente" ou "O lote causou muitos erros e foi descartado"). Isso vale também quando a prefeitura recusa o lote inteiro sem identificar nenhum RPS do lote, como no prestador não autorizado ou no total de RPS que não confere: nenhum lote novo é criado. Corrija o que for apontado e transmita os documentos de novo |

No fim da consulta, a resposta da prefeitura aparece na tela, um bloco por lote consultado, com o status em que o lote ficou, as mensagens da prefeitura e, num lote com parte dos RPS rejeitada, os documentos movidos para o lote novo de erro. Um lote autorizado sem mensagem não mostra nada. O formato da mensagem está em [NFS-e: Mensagens da Prefeitura na Consulta do Lote](Localization-for-Brazil_User-Manual_NFSe-Mensagens-Consulta-Lote.md).

### Corrigir e reenviar

Quando o lote (ou o lote novo de erro) fica com **Status** **Erro**, corrija o cadastro apontado pelas mensagens e transmita os documentos novamente. Eles vão para um lote novo, com o mesmo **Nº RPS**: um RPS rejeitado não gerou NFS-e e pode ser enviado de novo com o mesmo número.

Um RPS que já virou NFS-e não deve ser reenviado: a prefeitura cancela a nota anterior e emite outra (alerta 217) ou recusa o reenvio (erros 338, 372 e 373). O Business Central não reenvia documento que já tem **Cód. Verificação NFS**. Se for preciso usar outro número, a ação **Criar Novo Nº** da página **NFS-e - Envio de Lote** atribui ao documento o próximo número de **Nº Série RPS/Temporário**.

### Primeira emissão em homologação

O envio de teste confere o lote sem gerar nota. Para testar a configuração antes da primeira emissão real:

1. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** e **Permitir Erro Lote Manual** ligados.
2. Confira o código de serviço municipal, a letra de tributação, o NBS, o código de operação e a classificação tributária das linhas.
3. Registre uma fatura de venda de serviço para a filial.
4. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**. O lote fica **Enviado**.
5. Depois de alguns minutos, use **Consultar**:
   - com erros, o lote fica com **Status** **Erro** e as mensagens ficam em **Lista Erros**: corrija os cadastros e transmita o documento de novo;
   - sem erros, o lote continua **Enviado**, sem número de NFS-e, porque o teste não gera nota.
6. Para emitir a nota real, abra a página **NFS-e - Status de Lote RPS**, selecione o lote de teste e use **Marcar como Erro**, que libera o documento. Desligue **Homologação** e transmita o documento de novo.

### Cancelar

O cancelamento tem dois passos, no **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`):

1. **Cancelar Documento Fiscal** (grupo **Contabilização**): estorna a contabilização e deixa o documento com **Status** **Cancelado** no Business Central. Nada é enviado à prefeitura neste passo.
2. **Cancelar NFS-e** (grupo **NFS-e**), habilitado depois do primeiro passo: envia à prefeitura o pedido de cancelamento da NFS-e, identificada pela inscrição municipal da filial, pelo **Nº NFS** e pelo **Cód. Verificação NFS**. O layout da prefeitura não tem campo de motivo.

Quando a prefeitura cancela a nota, a mensagem "NFS-e cancelada com sucesso." é exibida e o campo **NFS Cancelada** do documento é marcado. Se ela recusar, a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" traz as mensagens da prefeitura, no formato `Erro <código>: <descrição>` ou `Alerta <código>: <descrição>`, e a NFS-e continua autorizada. Por exemplo: 323 (a nota não pode ser cancelada) e 324 (prazo para a operação por meio eletrônico ultrapassado), casos que são tratados diretamente com a prefeitura.

O cancelamento vai sempre ao ambiente de produção, mesmo com **Homologação** ligado.

---

## 4. DANFSe

Com **Utilizar Layout Padrão** desligado, a ação **Abrir DANFSe** do **Documento Fiscal Registrado** baixa da prefeitura o **PDF da NFS-e**, localizado pela inscrição municipal da filial, pelo **Nº NFS** e pelo **Cód. Verificação NFS**, e o entrega como arquivo. **Atualizar DANFSe** baixa o PDF de novo, por exemplo depois do cancelamento. O PDF também é anexado ao e-mail do documento fiscal.

O PDF só existe para notas de produção. Com **Homologação** ligado, não há nota e a ação termina com erro.

Se a prefeitura não devolver o PDF, **Abrir DANFSe** termina com a mensagem "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e" ou com uma mensagem que começa por "O provedor da NFS-e devolveu uma resposta inválida ao download da imagem". Tente de novo mais tarde com **Atualizar DANFSe** ou consulte a nota no portal da prefeitura.

Mantenha **Utilizar Layout Padrão** desligado nas filiais de São Paulo: o DANFSe desta integração é o PDF da prefeitura. Com o campo ligado, **Abrir DANFSe** de uma nota sem **Chave de Acesso NFS-e** termina com a mensagem que começa por "O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe".

---

## 5. Pontos de Atenção

- **Limite de 500 KB**: a prefeitura recusa a mensagem acima desse tamanho (erro 1101). Ajuste **Nº Máx. de Docs. por Lote WS** para que cada lote fique abaixo do limite.
- **Deduções**: as deduções da base de cálculo não são enviadas. A NFS-e é emitida pelo valor final cobrado.
- **Envio de teste**: um documento testado fica preso ao lote de teste até ser liberado com **Marcar como Erro** (veja [Primeira emissão em homologação](#primeira-emissão-em-homologação)).
- **Lote com Status Sucesso e documento sem Nº NFS**: o número devolvido pela prefeitura não foi localizado porque **Nº Série RPS/Temporário** tem comprimento diferente da numeração fiscal. Ajuste a série e use **Consultar** de novo na página **NFS-e - Status de Lote RPS**.
- **Certificado vencido**: a transmissão, a consulta e o cancelamento são bloqueados até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **Erros mais comuns** (tabela de erros do manual do web service da prefeitura):

| Código | O que fazer |
| --- | --- |
| 1050 a 1054 (certificado inválido, vencido, sem CNPJ, com erro de cadeia ou revogado) | Cadastrar um certificado válido e selecioná-lo em **ID Certificado** |
| 1100 ou 1107 (CNPJ da comunicação ou da assinatura diferente do prestador) | Usar um certificado com o CNPJ da filial |
| 1202, 1207 ou 1208 (prestador sem CCM, não autorizado ou com inscrição cancelada) | Conferir a **Inscrição Municipal** em **Informação Filial** e a autorização com a prefeitura |
| 306, 310, 361 ou 367 (código de serviço inexistente, não informado ou não permitido); alerta 307 (código não cadastrado para o prestador) | Corrigir o **Cód. Tributação** em **Códigos de Serviço NFS** ou **Códigos Serviço NFS por Filial** (código municipal de 5 dígitos) |
| 206 (tipo de tributação inválido) | Corrigir a letra no **Cód. Tributação** de **Códigos Tributação NFS-e** |
| 225 (alíquota fora de 2% a 5%); alerta 208 (alíquota diferente da vigente) | Revisar a alíquota de ISS do serviço |
| 209, 311 ou 321 (retenção de ISS não permitida para o serviço, o tomador ou a tributação) | Revisar o ISS retido do documento |
| 219 ou 220 (inscrição municipal do tomador indevida ou obrigatória); alerta 211 | Conferir a **Inscrição Municipal** do cliente |
| 313 ou 107 (data de emissão posterior a hoje) | Conferir a data de registro do documento |
| 1101 (mensagem acima de 500 KB) | Reduzir **Nº Máx. de Docs. por Lote WS** |
| 268 ou 286 (código NBS inválido ou não informado) | Preencher o **Cód. NBS** do código de serviço |
| 294 ou 630 (código indicador de operação não informado ou inexistente); 273 ou 628 (classificação tributária do IBS/CBS inválida ou não vigente) | Preencher o **Cód. Operação de Serviços** e a classificação tributária das linhas |

- **Log de comunicação**: o envio e o retorno de cada operação com a prefeitura ficam registrados no log do serviço de integração, consultado pelo suporte. No Business Central ficam as mensagens do lote e os dados de retorno do documento.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **São Paulo - SP 2026** (`SP-SaoPaulo2026`) |
| **Config. NF-e** | **ID Certificado** | Certificado digital com o CNPJ da filial |
| **Config. NF-e** | **Homologação** | Desligado (ligado só para o envio de teste) |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **Nº Máx. de Docs. por Lote WS** | Valor que mantenha o lote abaixo de 500 KB (por exemplo, `50`) |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | Um dos dois |
| **Config. NF-e** | **Permitir Erro Lote Manual** | Ligado, se usar o envio de teste |
| **Config. NF-e** | **Utilizar Layout Padrão** | Desligado |
| **Config. NF-e** | **Tamanho Máx. Discriminação** | `2000` |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Com **Último Nº Usado**, mesmo comprimento da numeração fiscal |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Config. NF-e** | **Texto Legal Valor Aprox. Impostos Automático** | Ligado, para enviar a carga tributária do IBPT |
| **Série Filial Tipo Doc. Fiscal** | **Série** | Preenchida, 1 a 5 caracteres; numeração só com dígitos |
| **Informação Filial** | **Inscrição Municipal** | CCM da filial |
| **Códigos de Serviço NFS** | **Cód. Tributação** | Código de serviço municipal (5 dígitos) |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Item da LC 116 |
| **Códigos de Serviço NFS** | **Cód. NBS** | Código NBS (obrigatório) |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS (obrigatório) |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | Letra de tributação (`T`, `F`, `A`, `B`, `D`, `M`, `N`, `R`, `S`, `X`, `V` ou `P`) |
