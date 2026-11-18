<!-- PR: #274 -->
# NFS-e: Emissão em Barueri - SP

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **Barueri - SP** emitem a NFS-e pelo Business Central por meio do web service da Prefeitura de Barueri. O sistema monta o arquivo de RPS no layout da prefeitura (PMB004), envia-o pelo web service e, na consulta, lê o arquivo de retorno com as notas geradas. Na lista de cidades de NFS-e, a integração aparece como **Barueri - SP** (código `SP-Barueri`). A integração cobre transmissão, consulta do processamento, reenvio após rejeição e o DANFSe em PDF da prefeitura.

A autenticação é feita só pelo **certificado digital** da filial. A autorização não é imediata: a prefeitura recebe o arquivo, devolve um protocolo de remessa e processa os RPS depois; o número da NFS-e só chega na **consulta**. Há ambiente de homologação. O **cancelamento não é feito pelo sistema**: a nota é cancelada no portal da prefeitura.

---

## 1. Contexto

### O que muda para a filial de Barueri

| Tema | Como funciona em Barueri - SP |
| --- | --- |
| **Layout** | Arquivo de RPS em texto, no layout PMB004 da Prefeitura de Barueri, enviado pelo web service |
| **Autenticação** | Certificado digital da filial, usado na conexão com a prefeitura. Não há usuário, senha nem token |
| **Código do serviço** | **Item Lista Serviço** = código de atividade de Barueri |
| **Envio** | Um arquivo por lote. A prefeitura devolve um protocolo de remessa, e a nota ainda não está autorizada |
| **Autorização** | Conhecida na consulta do lote: o sistema acompanha o processamento do arquivo e, quando a prefeitura o importa, lê o arquivo de retorno e grava o número da NFS-e, a data e o código de verificação |
| **Rejeição** | Os erros do arquivo voltam com código de 3 dígitos e ficam em **Lista Erros** |
| **Cancelamento** | Não é feito pelo sistema. A NFS-e é cancelada no portal da prefeitura |
| **DANFSe** | PDF da prefeitura, baixado pelo Business Central |
| **Ambientes** | Homologação (ambiente de testes da prefeitura) e produção |

### Tributos enviados

O arquivo de Barueri leva:

- **Valor do serviço**: o valor dos serviços já descontado do desconto incondicional, porque o layout não tem campo de desconto.
- **Retenções**: IR, PIS, COFINS, CSLL e INSS retidos, cada um em seu registro, e o total das retenções. As **deduções** da base de cálculo também vão como "outros valores" (veja [Pontos de Atenção](#5-pontos-de-atenção)).
- **IBS/CBS (Reforma Tributária)**: quando a linha do documento tem classificação tributária de IBS/CBS, o arquivo leva o código indicador de operação, a classificação tributária, o CST, o código NBS e, para cliente do exterior, o NIF, o vínculo entre as partes e o endereço no exterior. O prestador é declarado como **não optante** pelo Simples Nacional. Os indicadores de uso ou consumo pessoal e de destinatário diferente do tomador vão sempre como "não"; crédito presumido e compra governamental não são enviados.

**Não vão no arquivo**:

- o ISS: nem a alíquota nem a indicação de ISS retido;
- o **Cód. Tributação** e o **Código CNAE** do código de serviço;
- o valor do **Cód. Tributação NFS** do documento;
- **Opção pelo Simples**, **Incentivo Cultural** e **Regime Especial de Tributação** da **Config. NF-e**;
- outras retenções;
- os tributos aproximados (IBPT);
- as alíquotas e os valores de IBS/CBS.

---

## 2. Configuração

### Certificado digital e cadastro na prefeitura

O certificado digital é a única credencial de Barueri: ele autentica a conexão do Business Central com o web service da prefeitura.

1. Use o certificado digital A1 (arquivo `.pfx`) da empresa.
2. Cadastre o certificado em **Certificados Digitais** (ação de mesmo nome na **Config. NF-e**) e selecione-o no campo **ID Certificado** da aba **NFS-e**.
3. A inscrição municipal da filial precisa estar cadastrada e ativa na prefeitura. Sem isso, a prefeitura rejeita o arquivo com os erros 101 ("Inscricao do Prestador de Servicos nao encontrada na base de dados") ou 256 ("Contribuinte com condicao diferente de ativo"). Confirme com a prefeitura o cadastro da filial para a emissão por web service e o acesso ao ambiente de testes.

O Business Central avisa quando o certificado está perto de vencer e **bloqueia a transmissão** quando ele já venceu, com a mensagem "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir.". Nesse caso, cadastre o certificado renovado e selecione-o em **ID Certificado**. O aviso e o bloqueio estão descritos em [Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão](Localization-for-Brazil_User-Manual_Certificado-Digital-Validade-Aviso-Vencimento.md).

### Homologação

Barueri tem um ambiente de testes. Para usá-lo, ligue **Homologação** na **Config. NF-e** da filial: o envio, a consulta e o download do DANFSe passam a usar o ambiente de testes da prefeitura. Para emitir em produção, desligue **Homologação**.

Uma nota gerada no ambiente de testes fica gravada no documento (**Nº NFS** e **Cód. Verificação NFS**), e o documento não volta a ficar pendente. Faça os testes com documentos que não precisarão ser emitidos em produção.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para Barueri - SP |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Barueri - SP** na lista (código `SP-Barueri`) |
| **Configuração Envio** | **Tipo de WS** | Preenchido automaticamente (Indefinido). Não é editável |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial |
| **Configuração Envio** | **Homologação** | Ligado para emitir no ambiente de testes; desligado para produção |
| **Configuração Envio** | **Controlar Nº RPS** | **Ligado**. Barueri não informa o último número usado; desligado, toda transmissão falha com "Erro ao consultar último número de RPS: Barueri nao possui servico de consulta do ultimo numero de RPS." |
| **Configuração Envio** | **WS NFS-e informa o IM** | Não usado: a inscrição municipal da filial é sempre enviada |
| **Configuração Envio** | **Nome Usuário WS** e **Senha WS** | Não usados por Barueri |
| **Configuração Envio** | **Endereço de WS NFS** | Mantenha o valor padrão: é o endereço do serviço de integração da Localização, usado por todas as cidades |
| **Configuração Envio** | **ID Relatório p/ TXT** | Em branco. Só é usado por filial que gera a NFS-e por arquivo, sem web service |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | Opcional (`0` = sem limite). A prefeitura recusa arquivo com quantidade de RPS acima do padrão dela (erro 700); nesse caso, defina um limite menor |
| **Lote** | **Usar Data/Hora como Nº Lote** ou **Nº Série Lote** | O número do lote vai à prefeitura como identificação da remessa, com até 11 posições, e não pode se repetir (erro 102). Ligue **Usar Data/Hora como Nº Lote** ou use uma série só com dígitos |
| **Lote** | **Permitir Erro Lote Manual** | Opcional. Libera a ação **Marcar como Erro** na página **NFS-e - Status de Lote RPS** |
| **Layout Padrão** | **Utilizar Layout Padrão** | **Desligado**. O DANFSe de Barueri é o PDF da prefeitura (veja a [seção 4](#4-danfse)) |
| **Layout Padrão** | **Incidência do ISSQN no Município do Prestador** | Não usado (só afeta o DANFSe do Layout Padrão) |
| **Discriminação Serviços** | **Tamanho Máx. Discriminação** | `1000`. O arquivo de Barueri leva até 1000 caracteres de discriminação e o que passa disso é cortado no envio; com o limite, o sistema avisa já no registro da venda |
| — | **Nº Série RPS/Temporário** | Série com **Último Nº Usado** e números do mesmo comprimento da numeração fiscal (veja [Numeração e série do RPS](#numeração-e-série-do-rps)) |
| — | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente**, para que o **Município Prestação Serviço** do documento seja preenchido automaticamente |
| — | **NFS-e suporta IBPT** | Desligado automaticamente ao escolher Barueri. Os tributos aproximados não são enviados |
| — | **Opção pelo Simples**, **Incentivo Cultural** e **Regime Especial de Tributação** | Não enviados |

### Numeração e série do RPS

- O número do RPS é o **Nº RPS** do documento, que recebe o **Nº Nota Fiscal** na primeira transmissão. Ele vem da série do tipo de documento fiscal: **Nº Série Fiscal** da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, do próprio tipo de documento fiscal. Use números **só com dígitos**: a prefeitura aceita até `0009999999` (erro 204).
- A **Série** do tipo de documento fiscal pode ter até 4 caracteres; o que passa disso é cortado. Série recusada pela prefeitura gera o erro 202.
- A **Consulta** exige **Nº Série RPS/Temporário** preenchida e com **Último Nº Usado**. O sistema usa o comprimento desse número para localizar o documento no arquivo de retorno. Por isso, a série precisa gerar números com o mesmo formato e o mesmo comprimento do **Nº Nota Fiscal** (por exemplo, os dois com 6 dígitos). A mesma série é usada pela ação **Criar Novo Nº**.

### Códigos de serviço

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), para cada serviço emitido pela filial:

| Campo | O que informar |
| --- | --- |
| **Item Lista Serviço** | O **código de atividade** de Barueri. Só os dígitos são enviados, completados com zeros à esquerda até 9 posições. Código inexistente, não permitido ou bloqueado é rejeitado (erros 216, 260 e 261) |
| **Cód. NBS** | Código NBS do serviço. Vai no grupo de IBS/CBS, quando a linha tem classificação tributária (erro 807 se faltar ou estiver fora do padrão) |
| **Cód. Operação de Serviços** | O código indicador de operação do IBS/CBS. Vai no mesmo grupo (erro 817) |
| **Cód. Tributação** | Não enviado a Barueri |
| **Código CNAE** | Não enviado a Barueri |

Quando a filial tem registro próprio em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`, ação **Códigos por Filial**), os campos desse registro substituem os do código de serviço, mesmo que estejam em branco.

O item de cada linha precisa ser do tipo Serviço e ter o **Cód. Serviço** preenchido na ficha do item.

### Códigos Tributação NFS-e

O valor do **Cód. Tributação NFS** não vai à prefeitura de Barueri, mas o sistema exige que o documento tenha um **Cód. Tributação NFS** que exista em **Códigos Tributação NFS-e** (página `NFSeTributationCodesFNX`) da filial e tenha o campo **Cód. Tributação** preenchido. Cadastre um código para a filial e marque-o como **Padrão**: ele preenche automaticamente o **Cód. Tributação NFS-e** do pedido de venda e o **Cód. Tributação NFS** do documento.

### Filial e cliente

- **Informação Filial** da filial: nome, CNPJ, **Inscrição Municipal**, cidade e CEP preenchidos; o CEP precisa ter **Cód. Municipio IBGE** em **CEPs**. No cabeçalho do arquivo, a inscrição municipal vai com até 7 posições.
- **Cliente**: CPF ou CNPJ, nome, endereço, número, bairro, cidade, UF e CEP (erros 237 a 245). O arquivo tem tamanho fixo por campo: nome até 60 caracteres, endereço até 75, complemento até 30, bairro e cidade até 40. O que passa disso é cortado.
- **Cliente do exterior**: o campo CEP da ficha precisa ter o código postal do país do cliente, só com números, e o campo UF, o estado, província ou região. O país é identificado pelo **Cód. BACEN** do país/região do cliente (veja [Validação de CEP para Endereços Estrangeiros](Localization-for-Brazil_User-Manual_Validacao-CEP-Estrangeiro.md)). Informe também o **Cód. NIF**.
- **Município Prestação Serviço** do documento: indica se o serviço foi prestado em Barueri ou fora. Para serviço exportado, use EXTERIOR; o código postal e a província do local no exterior vêm do endereço de prestação do documento ou, sem ele, do endereço do cliente.

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), informe o **Cód. Filial** e use:

- **Transmitir Docs. Pendentes**: transmite os documentos pendentes da filial num único arquivo, com até **Nº Máx. de Docs. por Lote WS** documentos. Se houver mais pendentes que o máximo, o sistema pergunta se deseja enviar só os primeiros; os demais ficam para a próxima transmissão.
- **Transmitir Doc. Selecionado**: transmite só o documento selecionado.

A transmissão é barrada, com a mensagem na tela, quando:

| Situação | Mensagem |
| --- | --- |
| Certificado vencido | "O certificado digital &lt;nome&gt; venceu em &lt;data&gt;. Importe um certificado válido antes de transmitir." |
| Cliente do exterior sem código postal ou sem estado/província, ou local de prestação no exterior sem esses dados | "RPS &lt;nº&gt; (&lt;cliente&gt;): a NFS-e de Barueri exige o endereço no exterior. Preencha &lt;o que falta&gt;.", indicando o campo: o CEP do cliente (código postal do país, só números), a UF do cliente (estado/província/região), ou o CEP e a UF do endereço do local da prestação do documento |
| Documento em lote ainda **Enviado** | "Lote &lt;nº&gt; deverá ser consultado antes do envio." |
| Nenhuma linha com item de serviço com código | "Nenhuma linha da nota fiscal possui Código de Serviço associado." |

Se a prefeitura recusar o arquivo na recepção, a mensagem dela aparece na tela no formato `<código>: <mensagem>` e nada fica gravado: corrija e transmita de novo.

Quando a prefeitura recebe o arquivo, o lote fica com **Status** **Enviado** e os documentos ficam com **Status NFS-e** **Enviado**. A nota **ainda não está autorizada**: o número da NFS-e só chega na consulta.

### Consultar o processamento

Na página **NFS-e - Envio de Lote** (selecione o documento) ou na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`, selecione o lote), use **Consultar**. O sistema verifica a situação do arquivo na prefeitura e, quando ele já foi processado, baixa e lê o arquivo de retorno.

| Resultado da consulta | O que acontece no Business Central |
| --- | --- |
| **Arquivo ainda não processado** | O lote continua **Enviado** e a situação fica em **Lista Avisos**: "Arquivo aguardando processamento", "Arquivo em processamento", "Arquivo validado" ou "Situacao: &lt;n&gt; - Arquivo de retorno ainda nao disponivel". Consulte novamente em alguns minutos |
| **Arquivo importado** | O lote fica com **Status** **Sucesso**. Cada documento recebe o **Nº NFS**, a **Data NFS** e o **Cód. Verificação NFS** (código de autenticidade da nota), e o **Nº Nota Fiscal** passa a ser o número da NFS-e |
| **Arquivo com erros** | O lote fica com **Status** **Erro** e os erros ficam em **Lista Erros**, no formato `Linha <n> - <código>: <descrição>`, em que a linha é a do arquivo em que a prefeitura achou o erro |

### Corrigir e reenviar

Quando o lote fica com **Status** **Erro**, corrija o cadastro apontado pelas mensagens e transmita os documentos novamente. Eles vão num arquivo novo, com novo número de remessa e o mesmo **Nº RPS**.

Se a prefeitura informar que o RPS já foi enviado (erro 205), confira no portal da prefeitura se a nota existe. Se ela não existir, use **Criar Novo Nº** na página **NFS-e - Envio de Lote** para dar ao documento o próximo número de **Nº Série RPS/Temporário** e transmita de novo.

### Primeira emissão em homologação

1. Confirme com a prefeitura o acesso da filial ao ambiente de testes.
2. Na **Config. NF-e** da filial, configure a aba **NFS-e** conforme a [seção 2](#2-configuração), com **Homologação** ligado.
3. Confira o código de atividade, a série do RPS, a numeração do lote e os cadastros da filial e do cliente.
4. Registre uma fatura de venda de serviço para a filial, que não precisará ser emitida em produção.
5. Na página **NFS-e - Envio de Lote**, informe a filial, selecione o documento e use **Transmitir Doc. Selecionado**. O lote fica **Enviado**.
6. Use **Consultar** até o arquivo ser processado. Corrija o que **Lista Erros** apontar e transmita de novo.
7. Com a nota gerada, abra o **Documento Fiscal Registrado** e use **Abrir DANFSe** (veja a [seção 4](#4-danfse)).
8. Para emitir em produção, desligue **Homologação**.

### Cancelar

O cancelamento da NFS-e de Barueri **não é feito pelo sistema**. Para cancelar uma nota:

1. Cancele a NFS-e no **portal da prefeitura de Barueri**, conforme as regras da prefeitura.
2. No Business Central, abra o **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`) e use **Cancelar Documento Fiscal** (grupo **Contabilização**). Essa ação estorna a contabilização e deixa o documento com **Status** **Cancelado**. Nada é enviado à prefeitura.

Não use **Cancelar NFS-e**. Em Barueri, essa ação termina com a mensagem "Os seguintes erros ocorreram no cancelamento da NFS-e:" seguida de "Cancelamento em Barueri e feito via RPS com situacao 'C'. Use EnviarLoteRps com o campo de cancelamento.". A mensagem significa que o sistema não envia cancelamento para Barueri: nenhum pedido chegou à prefeitura e a nota continua válida até ser cancelada no portal. O texto técnico não pede nenhuma ação no Business Central.

A nota cancelada no portal continua com **NFS Cancelada** desmarcado no Business Central; o documento fica com **Status** **Cancelado** pelo passo 2.

A página **NFS-e - Exportar Arquivo de Cancelamentos** também não se aplica: ela atende só filiais que emitem por arquivo TXT, e termina com "Essa funcionalidade atende somente notas fiscais eletrônicas via exportação de arquivo TXT.".

---

## 4. DANFSe

Com **Utilizar Layout Padrão** desligado, a ação **Abrir DANFSe** do **Documento Fiscal Registrado** baixa do site da prefeitura o **PDF da NFS-e**, localizado pelo **Cód. Verificação NFS** e pelo CPF/CNPJ do cliente, e o entrega como arquivo. Com **Homologação** ligado, o PDF vem do ambiente de testes. **Atualizar DANFSe** baixa o PDF de novo. O PDF também é anexado ao e-mail do documento fiscal.

Se a prefeitura não devolver o PDF, **Abrir DANFSe** termina com a mensagem "A cidade &lt;cidade da filial&gt; não possui automatização de imagem NFS-e" ou com uma mensagem que começa por "O provedor da NFS-e devolveu uma resposta inválida ao download da imagem". Tente de novo mais tarde com **Atualizar DANFSe** ou consulte a nota no portal da prefeitura.

Mantenha **Utilizar Layout Padrão** desligado nas filiais de Barueri: o DANFSe desta integração é o PDF da prefeitura.

---

## 5. Pontos de Atenção

- **Cancelamento**: só pelo portal da prefeitura (veja [Cancelar](#cancelar)).
- **Deduções**: as deduções da base de cálculo vão no arquivo como "outros valores" do tipo VN, que a prefeitura recusa para RPS com data a partir de 29/12/2017 (erro 305). Revise o documento com dedução antes de transmitir.
- **ISS**: a alíquota e o ISS retido não vão no arquivo.
- **Textos cortados**: discriminação acima de 1000 caracteres e dados do cliente acima do tamanho do campo são cortados no envio (veja [Filial e cliente](#filial-e-cliente)).
- **Lote com Status Sucesso e documento sem Nº NFS**: o número devolvido pela prefeitura não foi localizado porque **Nº Série RPS/Temporário** tem comprimento diferente da numeração fiscal. Ajuste a série e use **Consultar** de novo na página **NFS-e - Status de Lote RPS**.
- **Certificado vencido**: a transmissão e a consulta são bloqueadas até um certificado válido ser selecionado em **ID Certificado**. O sistema avisa antes do vencimento.
- **Erros mais comuns** (mensagens da prefeitura em **Lista Erros**):

| Código | O que fazer |
| --- | --- |
| 101 (inscrição do prestador não encontrada) ou 256 (contribuinte não ativo) | Conferir a **Inscrição Municipal** em **Informação Filial** e o cadastro na prefeitura |
| 102 (identificação da remessa inválida ou já informada) | Ligar **Usar Data/Hora como Nº Lote** ou usar uma **Nº Série Lote** só com dígitos, sem repetir número |
| 202 (série do RPS inválida) | Ajustar a **Série** do tipo de documento fiscal (até 4 caracteres) |
| 204 (número do RPS não informado ou inválido) | Numeração fiscal só com dígitos, até `0009999999` |
| 205 ou 206 (RPS já enviado ou repetido no arquivo) | Conferir a nota no portal; se não existir, **Criar Novo Nº** |
| 210 (data de emissão posterior a hoje) ou 259 (competência já encerrada) | Conferir a data de registro do documento |
| 216, 260 ou 261 (código de atividade não encontrado, não permitido ou bloqueado) | Corrigir o **Item Lista Serviço** com o código de atividade de Barueri |
| 237 a 245 (CNPJ, nome, endereço, número, bairro, cidade, UF ou CEP do cliente) | Completar ou corrigir a ficha do cliente |
| 233 ou 234 (tomador estrangeiro ou país de nacionalidade inválido) | Conferir o país/região do cliente e o **Cód. BACEN** dele |
| 305 (outros valores do tipo VN não permitidos) | Documento com dedução da base de cálculo: revisar antes de transmitir |
| 807 (código NBS) | Preencher o **Cód. NBS** do código de serviço |
| 817, 818 ou 819 (indicador de operação, classificação tributária ou CST do IBS/CBS) | Preencher o **Cód. Operação de Serviços** e a classificação tributária das linhas |
| 700 (quantidade de RPS superior ao padrão) | Definir um **Nº Máx. de Docs. por Lote WS** menor |

- **Log de comunicação**: o envio e o retorno de cada operação com a prefeitura ficam registrados no log do serviço de integração, consultado pelo suporte. No Business Central ficam as mensagens do lote e os dados de retorno do documento.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Barueri - SP** (`SP-Barueri`) |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Homologação** | Conforme o ambiente |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **Usar Data/Hora como Nº Lote** | Ligado (ou **Nº Série Lote** só com dígitos) |
| **Config. NF-e** | **Utilizar Layout Padrão** | Desligado |
| **Config. NF-e** | **Tamanho Máx. Discriminação** | `1000` |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Com **Último Nº Usado**, mesmo comprimento da numeração fiscal |
| **Config. NF-e** | **Padrão Município Prestação Serviço** | **Empresa** ou **Cliente** |
| **Série Filial Tipo Doc. Fiscal** | **Série** | Até 4 caracteres; numeração só com dígitos |
| **Informação Filial** | **Inscrição Municipal** | Inscrição da filial em Barueri |
| **Códigos de Serviço NFS** | **Item Lista Serviço** | Código de atividade de Barueri |
| **Códigos de Serviço NFS** | **Cód. NBS** | Código NBS |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS |
| **Códigos Tributação NFS-e** | **Cód. Tributação** | Preenchido (exigido pelo sistema; o valor não vai à prefeitura) |
