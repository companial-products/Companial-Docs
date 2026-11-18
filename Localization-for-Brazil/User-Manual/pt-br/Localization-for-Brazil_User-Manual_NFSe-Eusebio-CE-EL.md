<!-- PR: #243 -->
# NFS-e: Emissão em Eusébio - CE (Provedor EL, Layout Nacional)

**Data**: Outubro/2026

---

## Sumário

As filiais localizadas em **Eusébio - CE** passam a emitir NFS-e pelo Business Central. A prefeitura usa a plataforma da **EL**, que trabalha sobre o layout da NFS-e Nacional (DPS). A cidade aparece na lista de cidades de NFS-e como **Eusébio - CE (EL)** e cobre transmissão, consulta do processamento, reenvio após rejeição, cancelamento e DANFSe.

A autenticação é feita por um **token de integração** gerado pelo próprio contribuinte no portal da prefeitura. A autorização não é imediata: a prefeitura recebe a DPS, o ambiente nacional a processa e o resultado (NFS-e autorizada ou lista de erros) só aparece na consulta do lote.

---

## 1. Contexto

### O que muda para a filial de Eusébio

| Tema | Como funciona em Eusébio - CE (EL) |
| --- | --- |
| **Layout** | DPS do layout da NFS-e Nacional, assinada com o certificado digital da filial |
| **Autenticação** | Token de integração gerado no portal da prefeitura, informado no campo **Senha WS** |
| **Código de serviço municipal** | Obrigatório. Vai na DPS a partir do campo **Item Lista Serviço** do código de serviço NFS da filial |
| **Envio** | Um documento por lote. A prefeitura confirma o recebimento, mas a nota ainda não está autorizada |
| **Autorização** | Conhecida na consulta do lote: grava o número da NFS-e e a chave de acesso nacional |
| **Rejeição** | Os erros da prefeitura ficam no lote. Depois da correção, o reenvio usa a mesma identificação da DPS e a prefeitura o trata como correção, não como nova declaração |
| **Cancelamento** | Evento de cancelamento do layout nacional, com a chave de acesso da nota |
| **Substituição** | Não há evento separado: a EL trata a substituição a partir da própria DPS |
| **DANFSe** | Layout padrão nacional, gerado localmente a partir da chave de acesso |
| **Ambientes** | Homologação (produção restrita do ambiente nacional) e produção |

### Tributos na DPS

As retenções federais (PIS, COFINS, CSLL, IRRF e INSS), os grupos de IBS/CBS da Reforma Tributária, obra, imóvel e evento, o NIF do tomador estrangeiro e as informações de comércio exterior seguem na DPS como nas demais cidades do layout nacional. Quando a operação é tributável e há alíquota de ISS, a alíquota do ISSQN também vai na DPS.

O prestador é informado na DPS como **não optante** pelo Simples Nacional, como nas demais cidades do layout nacional.

---

## 2. Configuração

### Gerar o token de integração no portal da prefeitura

1. Acesse o portal de NFS-e da prefeitura de Eusébio com o usuário do contribuinte.
2. Abra o menu **Configurações → Integração**.
3. Clique em **Gerar Token** e copie o valor gerado.

O token tem data de validade, definida no portal. O Business Central não controla essa validade: quando o token vence ou é inativado, a prefeitura recusa a operação e a mensagem dela aparece para o usuário (veja [seção 5](#5-pontos-de-atenção)). Nesse caso, gere um novo token e informe-o na filial.

### Configuração NF-e da filial

Navegue até **Config. NF-e** da filial (página `NFeSetupFNX`) e, na aba **NFS-e**, preencha:

| Grupo | Campo | Valor para Eusébio - CE |
| --- | --- | --- |
| **Configuração Envio** | **Cód. Cidade WS** | Selecione **Eusébio - CE (EL)** na lista (código `CE-Eusebio-EL`) |
| **Configuração Envio** | **ID Certificado** | Certificado digital da filial, usado para assinar a DPS e o evento de cancelamento |
| **Configuração Envio** | **Senha WS** | O token de integração gerado no portal da prefeitura |
| **Configuração Envio** | **Homologação** | Ligado para emitir em homologação; desligado para produção |
| **Configuração Envio** | **Controlar Nº RPS** | Ligado. A numeração da DPS é controlada pelo Business Central |
| **Lote** | **Nº Máx. de Docs. por Lote WS** | `1`. A prefeitura aceita um documento por envio |
| **Layout Padrão** | **Utilizar Layout Padrão** | Ligado, para que o DANFSe seja gerado no padrão nacional (veja [seção 4](#4-danfse)) |

O campo **Nome Usuário WS** não é usado por Eusébio.

O campo **WS NFS-e informa o IM** controla se a inscrição municipal da filial vai na DPS. Se a prefeitura rejeitar a DPS por falta da inscrição municipal, ligue esse campo.

### Numeração e série da DPS

O layout nacional aceita **apenas dígitos** no número e na série da DPS:

- A série de numeração informada em **Nº Série RPS/Temporário** precisa gerar números só com dígitos, sem prefixo de letras (por exemplo, `000031` e não `NFS000031`).
- A **Série** do tipo de documento fiscal usado pela filial precisa estar preenchida e conter só dígitos. Ela vem da **Série Filial Tipo Doc. Fiscal** (página `FiscalDocumentTypeBranchSeriesFNX`, aberta a partir de **Tipos Documento Fiscal**) ou, sem série por filial, do próprio tipo de documento fiscal.

Fora disso, a prefeitura rejeita a DPS com o erro `E1235` (falha no esquema XML), apontando o número ou a série.

### Códigos de serviço

Em **Códigos Serviço NFS por Filial** (página `BranchNFSServiceCodesFNX`), para cada serviço emitido pela filial de Eusébio:

| Campo | O que informar |
| --- | --- |
| **Cód. Tributação** | O código de tributação **nacional**, de 6 dígitos |
| **Item Lista Serviço** | O código de serviço **municipal** de Eusébio. Pontos e outros sinais são descartados no envio: vão só letras e números, até 20 caracteres |

Em **Códigos de Serviço NFS** (página `NFSServiceCodesFNX`), confira também:

| Campo | O que informar |
| --- | --- |
| **Cód. Operação de Serviços** | O código indicador de operação (6 dígitos) do IBS/CBS. Sem ele, a prefeitura rejeita a DPS |
| **Cód. NBS** | Código NBS do serviço, exigido para tomador estrangeiro |

---

## 3. Emissão, Consulta e Cancelamento

### Transmitir

Na página **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`), use **Transmitir Docs. Pendentes** ou **Transmitir Doc. Selecionado**.

Antes de qualquer envio, a transmissão é barrada com mensagem que indica o que corrigir quando:

| Situação | Mensagem |
| --- | --- |
| O token não foi informado | Pede para gerar o token no portal da prefeitura e informá-lo em **Senha WS** |
| O serviço não tem código municipal | Nomeia o código de serviço NFS e pede o **Item Lista Serviço** em **Códigos Serviço NFS por Filial** |
| O lote tem mais de um documento | Pede `1` em **Nº Máx. de Docs. por Lote WS** |

Quando a prefeitura aceita o envio, o lote fica com **Status** **Enviado** e o **Protocolo** recebe a identificação da DPS. A nota **ainda não está autorizada**: o número da NFS-e e a chave de acesso só chegam na consulta.

Se a prefeitura recusar o envio (por exemplo, token vencido), a mensagem dela é exibida na transmissão.

### Consultar o processamento

Na página **NFS-e - Envio de Lote** ou na página **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`), selecione o lote e use **Consultar**.

| Resultado da consulta | O que acontece no Business Central |
| --- | --- |
| **Em processamento** | O lote continua **Enviado**, com um aviso em **Lista Avisos** para consultar novamente em alguns minutos |
| **Autorizada** | O documento recebe o número da NFS-e, a data de emissão e a chave de acesso nacional. Ele passa para um novo lote com **Status** **Sucesso**, que traz o lote enviado em **Nº Lote de Origem** |
| **Rejeitada** | O lote fica com **Status** **Erro** e as mensagens da prefeitura ficam em **Lista Erros**, no formato `Erro <código>: <descrição> <complemento>` |
| **Falha de comunicação** | O lote continua **Enviado**, com a falha em **Lista Avisos**. O reenvio não é liberado, porque a DPS pode já estar autorizada |

Depois que o documento foi autorizado e passou para o lote **Sucesso**, não é preciso consultar de novo o lote enviado, que continua com **Status** **Enviado**.

### Corrigir e reenviar

Quando o lote fica com **Status** **Erro**, corrija o cadastro apontado pelas mensagens e transmita o documento novamente. O reenvio mantém o número do documento e o número da DPS, e por isso a mesma identificação da DPS: a prefeitura trata o reenvio como correção da declaração anterior, sem duplicidade.

### Cancelar

No **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`), grupo NFS-e, use **Cancelar NFS-e**.

O cancelamento é enviado como evento do layout nacional, com a chave de acesso gravada na consulta que autorizou a nota e o motivo "Erro na Emissão", como nas demais cidades do layout nacional. O documento só fica cancelado no Business Central quando a prefeitura confirma o registro do evento; se ela devolver erros, eles são exibidos e o documento continua autorizado.

---

## 4. DANFSe

Com **Utilizar Layout Padrão** ligado, o DANFSe da nota autorizada é gerado no padrão nacional a partir da chave de acesso, pelas ações **Abrir DANFSe** e **Atualizar DANFSe** do **Documento Fiscal Registrado**. O layout, a marca d'água de homologação e a de cancelamento estão descritos em [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md).

---

## 5. Pontos de Atenção

- **Token vencido ou inativado**: o sistema não avisa o vencimento. A prefeitura recusa a operação e a mensagem dela aparece ao usuário; gere um novo token no portal e atualize **Senha WS**.
- **Erro `E0037` em homologação**: a prefeitura ainda não habilitou a produção restrita no ambiente nacional. O contribuinte precisa solicitar essa habilitação à prefeitura.
- **Validações do ambiente nacional**: as conferências que o sistema faz antes do envio para o emissor nacional (SEFIN) — código NBS e NIF (ou motivo da falta dele) para tomador estrangeiro e significado nacional dos códigos de tributação — não são feitas antes do envio para Eusébio. Falhas nesses cadastros aparecem como rejeição da prefeitura, em **Lista Erros**, após a consulta.
- **Log de comunicação**: o envio e o retorno de cada operação com a prefeitura ficam registrados no log do web service, como nas demais cidades. O token não aparece no log.
- **Ambientes**: o campo **Homologação** define se a filial usa a homologação ou a produção da EL. Notas emitidas em homologação não têm validade jurídica.

---

## 6. Resumo da Configuração

| Página | Campo | Valor |
| --- | --- | --- |
| **Config. NF-e** | **Cód. Cidade WS** | **Eusébio - CE (EL)** |
| **Config. NF-e** | **Senha WS** | Token de integração do portal da prefeitura |
| **Config. NF-e** | **ID Certificado** | Certificado digital da filial |
| **Config. NF-e** | **Homologação** | Conforme o ambiente |
| **Config. NF-e** | **Controlar Nº RPS** | Ligado |
| **Config. NF-e** | **Nº Máx. de Docs. por Lote WS** | `1` |
| **Config. NF-e** | **Utilizar Layout Padrão** | Ligado |
| **Config. NF-e** | **Nº Série RPS/Temporário** | Série de numeração só com dígitos |
| **Série Filial Tipo Doc. Fiscal** | **Série** | Preenchida, só com dígitos |
| **Códigos Serviço NFS por Filial** | **Cód. Tributação** | Código de tributação nacional (6 dígitos) |
| **Códigos Serviço NFS por Filial** | **Item Lista Serviço** | Código de serviço municipal de Eusébio |
| **Códigos de Serviço NFS** | **Cód. Operação de Serviços** | Código indicador de operação do IBS/CBS |
