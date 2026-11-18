<!-- Branch: feature/Eventos -->
<!-- PR: #337 -->

# NFS-e: Local da Prestação — Obras, Atividades/Eventos e Imóveis

**Data**: Setembro/2026

---

## Sumário

A NFS-e de padrão nacional informa o **local da prestação** do serviço separadamente do **destinatário** da operação. Até agora a localização usava um único endereço para os dois papéis: o endereço de entrega do documento era reaproveitado como local da prestação, e a obra vinha do cadastro do código de serviço. Esta funcionalidade separa os dois blocos e introduz três cadastros próprios — **Obras NFS-e**, **Atividades/Eventos NFS-e** e **Imóveis NFS-e** —, além de um bloco de **Endereço do Local da Prestação** editável no documento. Qual dos três cadastros se aplica deixa de ser adivinhado: passa a ser determinado pelo **Local de Fornecimento** registrado no código de operação de serviços, conforme o Anexo VII da LC 214/2025.

---

## 1. Contexto

Um documento de serviço pode envolver até três endereços distintos:

| Papel | Exemplo |
| --- | --- |
| **Destinatário** da operação | A construtora que contratou a reforma e recebe a nota |
| **Local da prestação** | O canteiro de obras onde o serviço foi executado |
| Município de tributação | O município que recebe o ISS |

Antes desta funcionalidade, o **Endereço de Entrega** do documento cobria o primeiro *ou* o segundo papel, nunca os dois ao mesmo tempo: o sistema decidia o papel no momento do registro (campo **Papel do Endereço de Entrega**, `ShipToAddressRole`) e o endereço alimentava apenas um dos grupos da NFS-e. Quando o serviço era prestado em uma obra, em um imóvel de terceiro ou no pavilhão de uma feira, não havia onde informar o endereço do destinatário sem perder o local da prestação.

Agora o documento carrega um bloco de local da prestação próprio. Quando ele está preenchido, os dois grupos são informados lado a lado na NFS-e: o endereço de entrega volta a significar o destinatário, e o local da prestação vem do bloco novo.

### Quando se aplica

- Emissão de NFS-e de padrão nacional para serviços prestados fora do endereço do destinatário.
- **Construção civil**: obras com CNO e ART.
- **Serviços sobre imóveis** (locação, administração, corretagem, obras em imóvel de terceiro): imóveis com CIB ou inscrição imobiliária fiscal.
- **Serviços prestados em atividades ou eventos** (feiras, exposições, congressos, shows): atividades com identificador e período.

---

## 2. O que decide o local da prestação

O **Local de Fornecimento** (`SupplyLocation`) é um campo novo do **Cód. Operação de Serviços** (`ServiceOperationCode`), classificado conforme o Anexo VII da LC 214/2025. Ele diz qual endereço é o local da prestação da operação e, por consequência, qual cadastro o documento precisa referenciar.

| Local de Fornecimento | Significado | Cadastro exigido no documento |
| --- | --- | --- |
| **Tomador/Adquirente** | Endereço ou domicílio principal do adquirente | Nenhum |
| **Fornecedor** | Estabelecimento do prestador | Nenhum |
| **Destinatário** | Endereço ou domicílio do destinatário | Nenhum |
| **Imóvel** | Localidade do imóvel | **Imóvel** |
| **Atividade/Evento** | Local do evento | **Atividade/Evento** |
| **Outros** | Endereço diverso (portuário, transporte, entrega de carga, via explorada) | Nenhum — endereço digitado direto |
| *(em branco)* | Código não classificado | Nenhum |

A marca **Construção Civil** (`CivilConstruction`) do **Código de Serviço NFS** tem precedência sobre o Local de Fornecimento: um serviço de construção civil sempre reporta o canteiro de obras, qualquer que seja o código de operação.

### Classificação automática na atualização

Ao atualizar o aplicativo, os códigos de operação já existentes recebem a classificação do Anexo VII automaticamente:

| Local de Fornecimento | Códigos de operação classificados |
| --- | --- |
| **Imóvel** | `020101`, `020201`, `020301` |
| **Fornecedor** | `030101`, `050101` |
| **Tomador/Adquirente** | `030102`, `050102`, `100101`, `100301`, `100501` |
| **Destinatário** | `030103`, `050103`, `100102`, `100201`, `100302`, `100401`, `100502`, `100601` |
| **Atividade/Evento** | `040101` |
| **Outros** | `030104`, `050104`, `050201`, `060101`, `070101`, `070102`, `080101` |

A atualização **só classifica códigos que ainda estão em branco** — uma classificação feita manualmente pela empresa nunca é sobrescrita. Códigos de operação criados depois da atualização precisam ser classificados à mão.

Para revisar ou ajustar a classificação, abra **Códs. Operação de Serviços** (`ServiceOperationCodesFNX`) e preencha a coluna **Local de Fornecimento**.

---

## 3. Cadastros

Os três cadastros compartilham a mesma estrutura: um código, uma descrição, os dados fiscais específicos do tipo e um endereço completo. O endereço é o que será copiado para o documento e informado na NFS-e.

Em todos eles, preencher o **CEP** popula automaticamente **Endereço**, **Bairro**, **Cidade**, **Cód. Território** e **Cód. País/Região** a partir da tabela de CEPs.

### 3.1 Obras NFS-e

Página **Obras NFS-e** (`ConstructionsFNX`), tabela **Obra** (`ConstructionFNX`).

Cadastro já existente, agora com endereço próprio e o número da ART.

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Código** (`Code`) | Código (10) | Identificação da obra. Atribuído pela numeração, se configurada |
| **Descrição** (`Description`) | Texto (100) | Descrição da obra |
| **Nº CNO** (`CNONo`) | — | Cadastro Nacional de Obras (CNO/INSS), informado na NFS-e |
| **Tipo de Empreitada** (`ConstructionWorkType`) | Opção | Total ou Parcial |
| **Cód. ART** (`ARTCode`) | Texto (30) | **Novo.** Anotação de Responsabilidade Técnica, exigida em serviços de construção e engenharia |
| **CEP** (`PostCode`) | Código (20) | **Novo.** CEP do canteiro de obras |
| **Endereço** (`Address`) | Texto (100) | **Novo.** Logradouro do canteiro |
| **Número** (`Number`) | Texto (12) | **Novo.** Número do logradouro |
| **Complemento** (`Address2`) | Texto (50) | **Novo.** Quadra, lote |
| **Bairro** (`District`) | Texto (30) | **Novo.** |
| **Cidade** (`City`) | Texto (30) | **Novo.** Determina o município de tributação em serviços de construção civil |
| **Cód. Território** (`TerritoryCode`) | Código (10) | **Novo.** UF do canteiro |
| **Cód. País/Região** (`CountryCode`) | Código (10) | **Novo.** |

### 3.2 Atividades/Eventos NFS-e

Página **Atividades/Eventos NFS-e** (`NFSeActivities`), tabela **Atividade/Evento** (`NFSeActivity`). Cadastro novo.

Registra a feira, exposição, congresso ou show em que o serviço é prestado. O identificador e o período viajam na NFS-e.

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Código** (`Code`) | Código (10) | Identificação da atividade ou evento |
| **Descrição** (`Description`) | Texto (100) | Nome da feira, exposição, congresso ou show |
| **ID da Atividade/Evento** (`ActivityId`) | Texto (30) | Identificador informado na NFS-e. Deixe em branco quando não houver identificador atribuído |
| **Data Início** (`StartDate`) | Data | Data de início |
| **Data Fim** (`EndDate`) | Data | Data de término. Não pode ser anterior à data de início |
| **CEP** (`PostCode`) | Código (20) | CEP do local |
| **Endereço** (`Address`) | Texto (100) | Logradouro do local |
| **Número** (`Number`) | Texto (12) | |
| **Complemento** (`Address2`) | Texto (50) | Pavilhão, andar, sala |
| **Bairro** (`District`) | Texto (30) | |
| **Cidade** (`City`) | Texto (30) | Determina o município de tributação quando o Local de Fornecimento é uma atividade ou evento |
| **Cód. Território** (`TerritoryCode`) | Código (10) | UF do local |
| **Cód. País/Região** (`CountryCode`) | Código (10) | |

### 3.3 Imóveis NFS-e

Página **Imóveis NFS-e** (`NFSeProperties`), tabela **Imóvel** (`NFSeProperty`). Cadastro novo.

Registra a edificação ou o terreno em que o serviço é prestado. A NFS-e aceita identificar o imóvel pelo CIB, pela inscrição imobiliária fiscal do município ou pelo endereço.

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Código** (`Code`) | Código (10) | Identificação do imóvel |
| **Descrição** (`Description`) | Texto (100) | Descrição da edificação ou do terreno |
| **CIB** (`CIB`) | Código (20) | Cadastro Imobiliário Brasileiro, identifica o imóvel nacionalmente |
| **Inscrição Imobiliária Fiscal** (`MunicipalPropertyReg`) | Texto (30) | Cadastro próprio do município, usado quando ele não identifica o imóvel pelo CIB |
| **Cód. Município** (`MunicipalityCode`) | Código (20) | Município IBGE do imóvel. Em branco, o município é obtido a partir do CEP |
| **CEP** (`PostCode`) | Código (20) | CEP do imóvel |
| **Endereço** (`Address`) | Texto (100) | Logradouro do imóvel |
| **Número** (`Number`) | Texto (12) | |
| **Complemento** (`Address2`) | Texto (50) | Bloco, unidade, andar |
| **Bairro** (`District`) | Texto (30) | |
| **Cidade** (`City`) | Texto (30) | Determina o município de tributação quando o Local de Fornecimento é um imóvel |
| **Cód. Território** (`TerritoryCode`) | Código (10) | UF do imóvel |
| **Cód. País/Região** (`CountryCode`) | Código (10) | Um país diferente do Brasil faz o imóvel ser informado como estrangeiro na NFS-e |

### Exclusão protegida

Nos três cadastros, um registro **utilizado em um documento fiscal não pode ser excluído**. O CNO, a ART, o CIB, a inscrição imobiliária e o período do evento são lidos do cadastro tanto na montagem do RPS quanto a cada impressão do DANFSe, então a exclusão deixaria documentos já emitidos sem esses dados. A mensagem indica o documento fiscal que impede a exclusão.

---

## 4. Configuração

### Numerações automáticas dos códigos

Em **Configurações Fiscais** (`FiscalSetupFNX`), guia **NFS-e**, três campos novos definem a numeração dos códigos dos cadastros:

| Campo | Descrição |
| --- | --- |
| **Nos. Obra** (`ConstructionNos`) | Numeração dos códigos de novas obras |
| **Nos. Atividade/Evento** (`ActivityNos`) | Numeração dos códigos de novas atividades e eventos |
| **Nos. Imóvel** (`PropertyNos`) | Numeração dos códigos de novos imóveis |

Os três são **opcionais**: deixe em branco para continuar digitando os códigos manualmente. Quem já digitava os códigos das obras à mão não precisa mudar nada.

> **Atenção**: o código dos três cadastros aceita **10 caracteres**. Configure uma numeração cujo formato gere números de até 10 caracteres — se a numeração gerar um número mais longo, o sistema recusa a inclusão com uma mensagem explícita, em vez de truncar o código.

Quando a numeração está configurada e não é manual, o código é atribuído ao incluir o registro e não pode ser digitado. Sem numeração configurada, o código é obrigatório na inclusão.

### Classificação dos códigos de operação

Revise o **Local de Fornecimento** dos códigos em **Códs. Operação de Serviços** (`ServiceOperationCodesFNX`), conforme a seção [2](#2-o-que-decide-o-local-da-prestação). Um código sem classificação faz o documento seguir o comportamento anterior, em que o endereço de entrega representa o destinatário.

### Construção civil

Em **Códigos de Serviço NFS** (`NFSServiceCodesFNX`), a marca **Construção Civil** continua sendo o que caracteriza um serviço de obra. Ela também vai na transmissão da NFS-e e decide se a nota leva o grupo da obra: marque-a em todo código de serviço de obra (veja [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md#campo-construção-civil-no-código-de-serviço)). Os campos **Cód. Obra** (`ConstructionCode`) e **Cód. ART** (`ARTCode`) do código de serviço permanecem válidos como configuração de fallback: eles são usados quando o documento não tem obra vinculada.

---

## 5. Uso no documento de venda

### Como acessar

Nos documentos de venda — **Cotação Vendas**, **Ordem de Venda**, **Fatura de Venda** e **Ordem Venda Programado** —, o factbox de dados fiscais ganhou a linha **NFS-e**. Clique nela para abrir a página **Dados da NFS-e** (`SalesFiscalDataNFSe`).

A linha só fica ativa quando o **Tipo Documento Fiscal** do documento está marcado como serviço.

### A página Dados da NFS-e

A página tem dois grupos:

**Local da Prestação** — os três vínculos, mutuamente exclusivos:

| Campo | Descrição |
| --- | --- |
| **Cód. Obra** (`ConstructionCode`) | Obra em que o serviço é prestado |
| **Cód. Atividade/Evento** (`ActivityCode`) | Atividade ou evento em que o serviço é prestado |
| **Cód. Imóvel** (`PropertyCode`) | Imóvel em que o serviço é prestado |

**Endereço do Local da Prestação** — o endereço efetivamente informado na NFS-e: **CEP**, **Endereço**, **Número**, **Complemento**, **Bairro**, **Cidade**, **Cód. UF** e **Cód. País/Região**.

### Comportamento dos campos

**Apenas o campo que a operação realmente informa fica habilitado.** A página aplica a mesma precedência que a montagem do RPS:

1. Se o código de serviço do documento está marcado como **Construção Civil**, só **Cód. Obra** fica habilitado.
2. Caso contrário, o **Local de Fornecimento** do código de operação decide: **Imóvel** habilita **Cód. Imóvel**, **Atividade/Evento** habilita **Cód. Atividade/Evento** (marcado como obrigatório).
3. Locais de fornecimento sem cadastro próprio — **Tomador/Adquirente**, **Fornecedor**, **Destinatário** e **Outros** — deixam os três desabilitados, e o endereço é digitado direto no segundo grupo.

O código de serviço considerado é o da **primeira linha de serviço com quantidade positiva** do documento. É a mesma regra usada no registro e na montagem do RPS, então a página nunca exige um local da prestação diferente do que o registro vai cobrar.

**Selecionar um dos três copia o endereço do cadastro** para o grupo de endereço abaixo e **limpa os outros dois códigos** — uma operação informa um único local da prestação. O endereço copiado permanece editável: se o canteiro daquela obra específica fica em um endereço diferente do cadastrado, basta ajustar no documento.

Se o cadastro selecionado não tiver endereço nenhum, o grupo de endereço é preservado como está — nada é apagado.

### Campo Cód. Obra do cabeçalho: obsoleto

O campo **Cód. Obra** (`ConstructionCodeFNX`) do cabeçalho de venda foi marcado como obsoleto e não é mais exibido nos documentos. A obra passa a ser informada pela ação **NFS-e**, no campo **Cód. Obra** dos dados fiscais. Documentos abertos que já tinham o campo antigo preenchido precisam ter a obra reinformada pela nova página.

---

## 6. Validações no registro

Ao registrar um documento de serviço, além das validações já existentes de **Município Prestação Serviço** e **Cód. Tributação NFS-e**, o sistema valida o local da prestação conforme o **Local de Fornecimento** do código de operação:

| Local de Fornecimento | Validação |
| --- | --- |
| **Atividade/Evento** | **Cód. Atividade/Evento** é obrigatório no documento, e a atividade referenciada precisa ter **Data Início** e **Data Fim** preenchidas — o período é informado na NFS-e |
| **Imóvel** | Se um imóvel estiver vinculado, ele precisa ter ao menos um entre **CIB**, **Inscrição Imobiliária Fiscal** e endereço. O vínculo em si não é obrigatório |
| Demais | Nenhuma validação adicional |

As mensagens aparecem no painel de erros do documento e permitem navegar direto ao campo: um erro nos campos de local da prestação abre a página **Dados da NFS-e** posicionada no campo em questão.

---

## 7. O que a NFS-e informa

Com um **Endereço do Local da Prestação** preenchido no documento, o RPS transmitido passa a informar os dois blocos:

| Grupo da NFS-e | Origem |
| --- | --- |
| Local da prestação / imóvel | Bloco **Endereço do Local da Prestação** do documento |
| Destinatário | Endereço de entrega do documento, quando houver |
| Construção civil (CNO e ART) | Obra vinculada; na falta dela, o cadastro do código de serviço |
| Atividade/evento (id, nome e período) | Atividade vinculada |
| Imóvel (CIB, inscrição imobiliária, indicador de estrangeiro) | Imóvel vinculado |
| Município de tributação | **Município Prestação Serviço** do documento — **não muda** |

O grupo que recebe o endereço segue a precedência da seção [5](#5-uso-no-documento-de-venda): construção civil informa o local da prestação com o CNO e a ART; Local de Fornecimento **Imóvel** informa o grupo do imóvel; os demais informam o local da prestação.

> **Importante**: o **Município Prestação Serviço** continua sendo o que determina onde a operação é tributada. A **Cidade do Local da Prestação** é parte do endereço informado na NFS-e e **não substitui** o município de tributação.

Para o grupo do imóvel, o município IBGE vem do **Cód. Município** do cadastro do imóvel; em branco, é derivado do CEP do local da prestação.

---

## 8. DANFSe

O DANFSe acompanha o mesmo modelo:

- Quando o documento tem local da prestação próprio, o **quadro do destinatário** e a **linha do local da prestação** coexistem — antes, um excluía o outro.
- O rótulo da linha segue o grupo que foi efetivamente transmitido, de modo que a impressão não pode divergir do que foi enviado: *imóvel*, *obra* (código do CNO ou endereço do canteiro, nunca os dois), *atividade/evento* (endereço mais id e período) ou *local da prestação*.
- Documentos sem local da prestação próprio continuam impressos como antes.

---

## 9. Documento fiscal registrado

Os campos de local da prestação também existem no **Documento Fiscal Registrado** (`PostedFiscalDocumentFNX`) e no **Documento Fiscal de Venda** (`SalesFiscalDocumentFNX`): **Cód. Obra**, **Cód. Atividade/Evento**, **Cód. Imóvel** e os oito campos de endereço. No documento registrado eles ficam editáveis no modo de edição de documento fiscal, permitindo corrigir o local da prestação antes de uma retransmissão.

O campo **Papel do Endereço de Entrega** ganhou o valor **Não Reportado**, para os casos em que a operação é tributada em um endereço que a NFS-e já identifica de outra forma (o do adquirente ou o do fornecedor) e o endereço de entrega, portanto, não é informado de novo.

---

## 10. Compatibilidade e transição

Nada precisa ser reconfigurado para que os documentos existentes continuem funcionando:

| Situação | Comportamento |
| --- | --- |
| Documento **sem** endereço de local da prestação | Segue o modelo anterior: o endereço de entrega alimenta o local da prestação ou o quadro do destinatário conforme o **Papel do Endereço de Entrega** calculado no registro |
| Documento **com** endereço de local da prestação | Novo modelo: os dois blocos são informados lado a lado |
| Documento sem obra vinculada | O CNO e a ART vêm do cadastro do código de serviço, como antes |
| Código de operação não classificado | O endereço de entrega representa o destinatário |

Documentos já registrados continuam imprimindo e retransmitindo exatamente como antes — a decisão é tomada a partir dos dados do próprio documento, não de uma configuração global.

---

## 11. Resumo dos Campos Novos

### Tabela Obra (`ConstructionFNX`)

| Campo | Tipo |
| --- | --- |
| **Cód. ART** (`ARTCode`) | Texto (30) |
| **Endereço** (`Address`) | Texto (100) |
| **Número** (`Number`) | Texto (12) |
| **Complemento** (`Address2`) | Texto (50) |
| **Bairro** (`District`) | Texto (30) |
| **Cidade** (`City`) | Texto (30) |
| **CEP** (`PostCode`) | Código (20) |
| **Cód. Território** (`TerritoryCode`) | Código (10) |
| **Cód. País/Região** (`CountryCode`) | Código (10) |

### Tabela Atividade/Evento (`NFSeActivity`) — nova

| Campo | Tipo |
| --- | --- |
| **Código** (`Code`) | Código (10) |
| **Descrição** (`Description`) | Texto (100) |
| **ID da Atividade/Evento** (`ActivityId`) | Texto (30) |
| **Data Início** (`StartDate`) | Data |
| **Data Fim** (`EndDate`) | Data |
| **Endereço** (`Address`) | Texto (100) |
| **Número** (`Number`) | Texto (12) |
| **Complemento** (`Address2`) | Texto (50) |
| **Bairro** (`District`) | Texto (30) |
| **Cidade** (`City`) | Texto (30) |
| **CEP** (`PostCode`) | Código (20) |
| **Cód. Território** (`TerritoryCode`) | Código (10) |
| **Cód. País/Região** (`CountryCode`) | Código (10) |

### Tabela Imóvel (`NFSeProperty`) — nova

| Campo | Tipo |
| --- | --- |
| **Código** (`Code`) | Código (10) |
| **Descrição** (`Description`) | Texto (100) |
| **CIB** (`CIB`) | Código (20) |
| **Inscrição Imobiliária Fiscal** (`MunicipalPropertyReg`) | Texto (30) |
| **Cód. Município** (`MunicipalityCode`) | Código (20) |
| **Endereço** (`Address`) | Texto (100) |
| **Número** (`Number`) | Texto (12) |
| **Complemento** (`Address2`) | Texto (50) |
| **Bairro** (`District`) | Texto (30) |
| **Cidade** (`City`) | Texto (30) |
| **CEP** (`PostCode`) | Código (20) |
| **Cód. Território** (`TerritoryCode`) | Código (10) |
| **Cód. País/Região** (`CountryCode`) | Código (10) |

### Tabela Cód. Operação de Serviços (`ServiceOperationCode`)

| Campo | Tipo |
| --- | --- |
| **Local de Fornecimento** (`SupplyLocation`) | Enum **Local de Fornecimento do Serviço** |

### Tabela Configurações Fiscais (`FiscalSetupFNX`)

| Campo | Tipo |
| --- | --- |
| **Nos. Obra** (`ConstructionNos`) | Código (20) |
| **Nos. Atividade/Evento** (`ActivityNos`) | Código (20) |
| **Nos. Imóvel** (`PropertyNos`) | Código (20) |

### Dados fiscais do cabeçalho de venda (`SalesHeaderFiscalDataFNX`) e Documento Fiscal (`FiscalDocumentFNX`)

| Campo | Tipo |
| --- | --- |
| **Cód. Obra** (`ConstructionCode`) | Código (10) |
| **Cód. Atividade/Evento** (`ActivityCode`) | Código (10) |
| **Cód. Imóvel** (`PropertyCode`) | Código (10) |
| **Endereço do Local da Prestação** (`SupplyAddress`) | Texto (100) |
| **Número do Local da Prestação** (`SupplyNumber`) | Texto (12) |
| **Complemento do Local da Prestação** (`SupplyAddress2`) | Texto (50) |
| **Bairro do Local da Prestação** (`SupplyDistrict`) | Texto (30) |
| **Cidade do Local da Prestação** (`SupplyCity`) | Texto (30) |
| **CEP do Local da Prestação** (`SupplyPostCode`) | Código (20) |
| **Cód. UF do Local da Prestação** (`SupplyTerritoryCode`) | Código (10) |
| **Cód. País/Região do Local da Prestação** (`SupplyCountryCode`) | Código (10) |

### Campo obsoleto

| Campo | Substituto |
| --- | --- |
| **Cód. Obra** (`ConstructionCodeFNX`) do cabeçalho de venda | **Cód. Obra** (`ConstructionCode`) dos dados fiscais, pela ação **NFS-e** |
