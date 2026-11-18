<!-- PR: #140 -->

# ISS Retido: Cálculo em Compras e Fornecedor por Município

**Data**: Agosto/2026

---

## Sumário

O ISS retido é recolhido ao município onde o serviço é prestado. Esta atualização trata dois pontos. Primeiro, o ISS retido de uma compra de serviço passa a ter a alíquota obtida do cadastro de alíquotas por serviço e município. Segundo, cada município passa a ter o seu próprio fornecedor de ISS retido, aplicado automaticamente no cálculo e no registro. Com isso, uma única linha de configuração de contas de impostos atende todos os municípios, e o imposto retido é lançado na prefeitura correta, tanto no registro da nota quanto na retenção feita no pagamento.

## 1. Contexto

Quando a empresa contrata um serviço de um prestador de outro município, ela retém o ISS do prestador e recolhe o valor à prefeitura do município onde o serviço é prestado. Contabilmente, esse valor deixa de ser pago ao prestador e passa a ser uma obrigação com a prefeitura.

Dois problemas existiam nesse fluxo:

1. **A alíquota não era obtida do cadastro em compras.** As alíquotas de ISS por serviço e município já podiam ser cadastradas, mas em uma compra o sistema só usava a alíquota da **Jurisdição Imposto**. Quem deixasse a jurisdição sem **Alíquota**, esperando que o cadastro por serviço prevalecesse, simplesmente não via o imposto ser calculado.
2. **O fornecedor do imposto era fixo.** A **N° Conta a Pagar** do ISS retido vinha da configuração de contas de impostos. Atender vários municípios exigia uma linha de configuração por município, com filtros, ou aceitar que tudo caísse num fornecedor genérico para corrigir depois do registro.

Ambos foram resolvidos: a alíquota do ISS retido de compra passa a vir de **Alíquotas ISS**, e o fornecedor vem do campo **Nº Fornecedor ISS Ret.** do próprio município.

### Quando se aplica

- Empresas que tomam serviços com retenção de ISS de prestadores de outros municípios.
- Casos em que cada prefeitura é cadastrada como um **Fornecedor** distinto.

Se o **Nº Fornecedor ISS Ret.** ficar em branco, o comportamento é o anterior: a conta a pagar continua vindo da configuração de contas de impostos.

---

## 2. Pré-requisitos

| Item | Onde |
| --- | --- |
| Item de serviço com **Cód. Serviço** preenchido | Cartão do item |
| Fornecedor prestador com **CEP** preenchido | Cartão do fornecedor |
| **CEP** do fornecedor com **Cód. Municipio IBGE** preenchido | Página `PostCodeFNX` |
| Filial com **CEP** preenchido e com **Cód. Municipio IBGE** | Informação da filial |
| Municípios envolvidos cadastrados com o **Código IBGE** | Página **Municípios** (`MunicipiosFNX`) |

Sem o **Cód. Municipio IBGE** no CEP do fornecedor, o sistema não consegue identificar o município do prestador e não calcula a retenção.

---

## 3. Configuração

### Passo 1: Cadastrar a prefeitura como fornecedor

Cadastre cada prefeitura destinatária do ISS retido como um **Fornecedor** comum, com os dados fiscais e a forma de pagamento habituais.

### Passo 2: Informar o fornecedor no município

1. Pesquise por **Municípios** (`MunicipiosFNX`).
2. Localize o município pelo **Código IBGE** ou pelo **Nome**.
3. Preencha o campo **Nº Fornecedor ISS Ret.** com o fornecedor que representa a prefeitura daquele município.

| Campo | Conteúdo |
| --- | --- |
| **Código IBGE** | Código IBGE de 7 dígitos do município |
| **Nome** | Nome do município |
| **Nº Fornecedor ISS Ret.** | Fornecedor que recebe o ISS retido do município |

Repita para cada município em que a empresa toma serviços com retenção. Municípios sem retenção podem ficar com o campo em branco.

### Passo 3: Cadastrar as alíquotas por serviço e município

As alíquotas ficam dentro do código de serviço:

1. Pesquise por **Códigos de Serviço NFS** (`NFSServiceCodesFNX`).
2. Selecione o código de serviço do item contratado.
3. Clique na ação **Alíquotas ISS**. A página abre já filtrada por esse código de serviço.
4. Informe uma linha por município e preencha o **% ISS Retido** com a alíquota que o município cobra na retenção. O **% ISS** é usado no ISS próprio, das vendas.

| Campo | Conteúdo |
| --- | --- |
| **Cód. Município** | Município onde o serviço é prestado |
| **% ISS Retido** | Alíquota da retenção nesse município |

> A coluna **Alíquotas ISS** da lista de códigos de serviço mostra quantas alíquotas já foram cadastradas para cada código.

### Passo 4: Deixar a jurisdição sem alíquota

Para que a alíquota do cadastro acima prevaleça, a **Jurisdição Imposto** de ISS retido usada na **Cód. Área Impostos** da compra deve ficar com a **Alíquota** zerada. Se a jurisdição tiver alíquota própria, é ela que vale, e o cadastro por serviço é ignorado.

### Passo 5: Ajustar as contas de contabilização do ISS Retido

1. Pesquise por **Contas Impostos** (`TaxPostingAccountsFNX`).
2. Localize ou crie a linha com **Classificação Imposto** = **ISS Ret.**
3. Informe **Tipo Conta a Pagar** = **Fornecedor**.
4. Deixe a **N° Conta a Pagar** em branco. É o município que passa a determinar o fornecedor.

| Campo | Conteúdo |
| --- | --- |
| **Classificação Imposto** | **ISS Ret.** |
| **Tipo Filtro** | Em branco (uma única linha atende todos os municípios) |
| **Cód. Filtro** | Em branco |
| **Tipo Conta a Pagar** | **Fornecedor** |
| **N° Conta a Pagar** | Em branco, resolvido pelo município |

> Uma única linha de configuração passa a atender todos os municípios. Não é mais necessário criar uma linha por município usando **Tipo Filtro** e **Cód. Filtro**.

Se preferir manter um fornecedor padrão para os municípios ainda não configurados, informe-o na **N° Conta a Pagar**. Ele será usado sempre que o município não tiver o **Nº Fornecedor ISS Ret.** preenchido.

---

## 4. Comportamento

### Quando há retenção numa compra

Numa compra, quem toma o serviço é a empresa. A retenção é devida quando o prestador está em **outro município**, por isso a comparação é feita entre:

| Referência | De onde vem |
| --- | --- |
| Município do prestador | **CEP** do cadastro do fornecedor, pelo **Cód. Municipio IBGE** |
| Município de prestação | **Município Prestação Serviço** do documento; em branco, assume o município da filial |

Se os dois forem o mesmo município, não há retenção. Se forem diferentes, o ISS retido é calculado com a alíquota de **Alíquotas ISS** para o **Cód. Serviço** do item e o município de prestação.

O ISS próprio nunca é obtido do cadastro de serviço numa compra. Ele pertence ao prestador, não à empresa.

> Na **venda** há uma exceção a essa comparação: municípios que atribuem ao tomador local a responsabilidade pela retenção podem reter o ISS mesmo quando o serviço é prestado no próprio município da filial. Veja [ISS Retido no Mesmo Município](Localization-for-Brazil_User-Manual_ISS-Retido-Mesmo-Municipio.md).

### Como o fornecedor do imposto é definido

Durante o cálculo, para as linhas de **ISS Ret.** o sistema:

1. Lê o **Município Prestação Serviço** da linha de imposto.
2. Busca o município correspondente no cadastro.
3. Se o município tiver **Nº Fornecedor ISS Ret.** preenchido, grava esse fornecedor na linha, no campo **Nº Fornecedor Imposto a Pagar**.

No registro, esse fornecedor substitui a **N° Conta a Pagar** da configuração de contas de impostos, desde que o **Tipo Conta a Pagar** seja **Fornecedor**. Qualquer outra combinação, seja outro imposto, outro tipo de conta ou município sem fornecedor, mantém a conta configurada.

### No registro da nota fiscal de compra

Ao registrar a **Fatura de Compra** ou a **Ordem de Compra**, o lançamento do ISS retido a pagar é feito no fornecedor do município de prestação, e não no fornecedor fixo da configuração.

### Na retenção feita no pagamento

Quando o ISS retido é configurado para ser retido no pagamento, o valor só é lançado no momento em que a fatura é paga, pelo **Diário Pagamento** ou pela baixa do movimento de fornecedor. Nesse fluxo, a linha de imposto é remontada a partir do **Mov. Imposto Documento Fiscal** gravado no registro do documento.

O fornecedor do município é gravado nesse movimento e recuperado na hora do pagamento. Assim a retenção chega ao mesmo fornecedor que teria sido usado no registro da nota. O mesmo vale para o ajuste de adiantamento.

### Quando o município não tem fornecedor

Se a linha de **ISS Ret.** estiver configurada com **Tipo Conta a Pagar** = **Fornecedor** e **N° Conta a Pagar** em branco, e o município de prestação também não tiver o **Nº Fornecedor ISS Ret.** preenchido, o registro é interrompido. A mensagem informa que o fornecedor de ISS retido não foi encontrado e indica a **Classificação Imposto**, o **Tipo Filtro** e o **Cód. Filtro** da configuração usada.

A mensagem substitui o aviso genérico de campo obrigatório e aponta diretamente o que falta configurar.

---

## 5. Exemplo Prático

Uma empresa com filial no Rio de Janeiro/RJ contrata uma consultoria de um prestador de São Paulo/SP, no valor de R$ 10.000,00. O serviço é prestado no Rio, onde a empresa está. Como o prestador é de outro município, a retenção é devida ao Rio, à alíquota de 5%.

**Configuração**

| Onde | Campo | Valor |
| --- | --- | --- |
| **Municípios** | **Código IBGE** | `3304557` |
| **Municípios** | **Nome** | Rio de Janeiro |
| **Municípios** | **Nº Fornecedor ISS Ret.** | `PREF-RJ` |
| **Alíquotas ISS** | **Cód. Serviço** | `S000001` |
| **Alíquotas ISS** | **Cód. Município** | `3304557` |
| **Alíquotas ISS** | **% ISS Retido** | 5,00 |
| **Jurisdição Imposto** de ISS retido | **Alíquota** | 0,00 |
| **Contas Impostos** | **Classificação Imposto** | **ISS Ret.** |
| **Contas Impostos** | **Tipo Conta a Pagar** | **Fornecedor** |
| **Contas Impostos** | **N° Conta a Pagar** | Em branco |

**Documento**

| Campo | Valor |
| --- | --- |
| Fornecedor do serviço | `F-CONSULT`, com **CEP** de São Paulo |
| **Município Prestação Serviço** | `3304557` (Rio de Janeiro), ou em branco, assumindo a filial |
| Valor do serviço | R$ 10.000,00 |

**Resultado**

O prestador está em São Paulo e o serviço é prestado no Rio de Janeiro. Municípios diferentes, logo há retenção, com a alíquota cadastrada para o Rio.

| Item | Valor | Destino |
| --- | --- | --- |
| Valor do serviço | R$ 10.000,00 | Despesa |
| ISS retido (5%) | R$ 500,00 | Fornecedor `PREF-RJ` |
| Líquido a pagar ao prestador | R$ 9.500,00 | Fornecedor `F-CONSULT` |

Se o prestador também fosse do Rio de Janeiro, não haveria retenção. E se o serviço fosse prestado em outro município, bastaria cadastrar a alíquota e o **Nº Fornecedor ISS Ret.** daquele município. Nenhuma alteração seria necessária em **Contas Impostos**.

---

## 6. Resumo dos Campos Novos

### Tabela Município (`MunicipioFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Nº Fornecedor ISS Ret.** (`RetISSVendorNo`) | Código (20) | Fornecedor usado para registrar o ISS retido a pagar quando o serviço é prestado neste município. Relacionado a **Fornecedor**. Em branco, mantém o comportamento anterior |

### Tabela Linha Valor Imposto (`TaxAmountLineFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Nº Fornecedor Imposto a Pagar** (`PayableTaxVendorNo`) | Código (20) | Fornecedor que substitui a conta a pagar da configuração de impostos, preenchido no cálculo a partir do município de prestação |
| **CEP do Participante** (`ParticipantPostCode`) | Código (20) | CEP do cliente, fornecedor ou filial da operação, usado para identificar o município do prestador no cálculo do ISS retido de uma compra |

### Tabela Mov. Imposto Documento Fiscal (`FiscalDocumentTaxEntryFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Nº Fornecedor Imposto a Pagar** (`PayableTaxVendorNo`) | Código (20) | Mesmo conteúdo da linha de valor de imposto, gravado no registro do documento para que a retenção feita no pagamento alcance o mesmo fornecedor |
