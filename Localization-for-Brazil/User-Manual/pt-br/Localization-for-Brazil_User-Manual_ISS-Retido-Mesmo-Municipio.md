<!-- PR: #141 -->

# ISS Retido no Mesmo Município

**Data**: Agosto/2026

---

## Sumário

Pela regra geral, o ISS de um serviço prestado no próprio município do prestador é recolhido por ele mesmo. É o ISS próprio, e não há retenção. Alguns municípios, porém, atribuem ao tomador estabelecido neles a responsabilidade de reter o ISS dos prestadores locais. Esta atualização acrescenta ao cadastro de municípios a marca **ISS Retido no Mesmo Município**, que habilita o cálculo do ISS retido na venda de serviço prestado dentro do próprio município da filial.

Veja também: [ISS Retido: Cálculo em Compras e Fornecedor por Município](Localization-for-Brazil_User-Manual_ISS-Retido-Fornecedor-por-Municipio.md).

## 1. Contexto

No cálculo do ISS retido, o sistema compara o município da filial com o município onde o serviço é prestado. Quando são o mesmo município, o entendimento é que não há retenção: o imposto é do prestador, cobrado como ISS próprio.

Essa é a regra na maioria dos municípios, mas não em todos. Onde a legislação municipal define o tomador local como responsável pela retenção, a empresa que presta serviço dentro do próprio município precisa emitir a nota com o ISS retido. O sistema não o calculava, porque o município de prestação era o mesmo da filial.

A única saída era informar a **Alíquota** na **Jurisdição Imposto** de ISS retido. Isso resolve o valor, mas aplica a mesma alíquota a todos os serviços e ignora o cadastro de alíquotas por serviço e município.

### Quando se aplica

- Empresas estabelecidas em município cuja legislação obriga o tomador local a reter o ISS dos prestadores da própria cidade.
- Apenas na **emissão**: a marca vale para o lado de venda.

Nas **compras** nada muda. A obrigação de retenção alcança os tomadores estabelecidos no município, de modo que a compra de serviço de um prestador da mesma cidade continua sem retenção, como antes. Municípios sem a marca também mantêm exatamente o comportamento anterior.

---

## 2. Pré-requisitos

| Item | Onde |
| --- | --- |
| Item de serviço com **Cód. Serviço** preenchido | Cartão do item |
| Filial com **CEP** preenchido e com **Cód. Municipio IBGE** | Informação da filial |
| Município da filial cadastrado com o **Código IBGE** | Página **Municípios** (`MunicipiosFNX`) |
| Alíquota do **% ISS Retido** cadastrada para o serviço e o município | Ação **Alíquotas ISS** dos **Códigos de Serviço NFS** |
| **Jurisdição Imposto** de ISS retido com a **Alíquota** zerada | Jurisdições de imposto da **Cód. Área Impostos** usada na venda |

---

## 3. Configuração

### Passo 1: Marcar o município

1. Pesquise por **Municípios** (`MunicipiosFNX`).
2. Localize o município da filial pelo **Código IBGE** ou pelo **Nome**.
3. Marque o campo **ISS Retido no Mesmo Município**.

| Campo | Conteúdo |
| --- | --- |
| **Código IBGE** | Código IBGE de 7 dígitos do município |
| **Nome** | Nome do município |
| **ISS Retido no Mesmo Município** | Marcado apenas nos municípios que retêm o ISS dos prestadores locais |

Deixe o campo desmarcado em todos os demais municípios. Desmarcado é a regra geral.

### Passo 2: Cadastrar a alíquota da retenção

1. Pesquise por **Códigos de Serviço NFS** (`NFSServiceCodesFNX`).
2. Selecione o código de serviço do item prestado.
3. Clique na ação **Alíquotas ISS**. A página abre já filtrada por esse código de serviço.
4. Informe uma linha com o **Cód. Município** da própria cidade e o **% ISS Retido** que a legislação municipal determina.

| Campo | Conteúdo |
| --- | --- |
| **Cód. Município** | Município da filial, que é também o de prestação |
| **% ISS Retido** | Alíquota da retenção nesse município |

Como o cadastro é por serviço, é possível reter apenas nos serviços que a lei municipal indica. Sem linha cadastrada para o serviço, não há alíquota e não há retenção.

### Passo 3: Deixar a jurisdição sem alíquota

A **Jurisdição Imposto** de ISS retido usada na **Cód. Área Impostos** da venda deve ficar com a **Alíquota** zerada, para que a alíquota do cadastro por serviço prevaleça. Se a jurisdição tiver alíquota própria, é ela que vale, e o cadastro por serviço é ignorado.

---

## 4. Comportamento

### Na venda

Durante o cálculo, para as linhas de **ISS Ret.** o sistema compara:

| Referência | De onde vem |
| --- | --- |
| Município da filial | **CEP** da filial, pelo **Cód. Municipio IBGE** |
| Município de prestação | **Município Prestação Serviço** do documento |

Se os municípios forem diferentes, o ISS retido é calculado como sempre. Se forem o mesmo, o resultado passa a depender da marca:

| Município de prestação | Resultado |
| --- | --- |
| Com **ISS Retido no Mesmo Município** | Calcula o ISS retido com o **% ISS Retido** cadastrado para o serviço e o município |
| Sem a marca (padrão) | Não há retenção. O ISS é cobrado como ISS próprio |

### Interação com o ISS próprio

A marca **ISS Retido no Mesmo Município** habilita apenas o cálculo do ISS retido. O ISS próprio continua vindo da sua própria jurisdição. Se a **Cód. Área Impostos** usada na venda tiver as duas jurisdições, de ISS e de ISS retido, os dois impostos são calculados.

Quem trata da dupla incidência é outro campo do mesmo cadastro, descrito a seguir.

### Imposto Sem Bitributação

O campo **Imposto Sem Bitributação** resolve um cenário distinto do desta atualização: o serviço é prestado em **outro** município, que retém o ISS, e o município da filial dispensa o recolhimento do ISS próprio porque o imposto já foi retido na cidade da prestação.

Um exemplo: uma empresa do Rio de Janeiro presta um serviço entregue em São Paulo. São Paulo retém o ISS, e o Rio permite não recolher o ISS próprio sobre a mesma operação. A marca vai no município **da filial**, o Rio de Janeiro, e não no município que retém.

O efeito acontece durante o cálculo: quando a linha já tem ISS retido apurado, o ISS próprio não recebe a alíquota de **Alíquotas ISS** e fica zerado. Duas condições precisam estar satisfeitas para isso:

| Condição | Onde configurar |
| --- | --- |
| A jurisdição de ISS próprio tem de estar sem **Alíquota**, para que o percentual do ISS próprio venha do cadastro **Alíquotas ISS** | **Jurisdição Imposto** de ISS |
| A jurisdição de ISS retido tem de ser calculada **antes** da de ISS | **Ordem Cálculo** das linhas da **Cód. Área Impostos** |

Fora dessas condições o ISS próprio é calculado normalmente, e os dois impostos aparecem na mesma linha.

### Nas compras

A marca não altera o cálculo das compras. Numa compra de serviço de prestador da mesma cidade em que o serviço é prestado, a retenção continua não sendo calculada. A responsabilidade que a legislação municipal atribui alcança os tomadores estabelecidos no município, e não a empresa quando ela contrata um prestador local.

---

## 5. Exemplo Prático

Uma empresa com filial no Rio de Janeiro/RJ presta uma consultoria de R$ 10.000,00 para um cliente do próprio Rio de Janeiro, e o serviço é prestado no Rio. O município determina que o tomador retenha o ISS à alíquota de 4%.

**Configuração**

| Onde | Campo | Valor |
| --- | --- | --- |
| **Municípios** | **Código IBGE** | `3304557` |
| **Municípios** | **Nome** | Rio de Janeiro |
| **Municípios** | **ISS Retido no Mesmo Município** | Marcado |
| **Alíquotas ISS** | **Cód. Serviço** | `S000001` |
| **Alíquotas ISS** | **Cód. Município** | `3304557` |
| **Alíquotas ISS** | **% ISS Retido** | 4,00 |
| **Jurisdição Imposto** de ISS retido | **Alíquota** | 0,00 |

**Documento**

| Campo | Valor |
| --- | --- |
| Filial | Rio de Janeiro, com **CEP** cujo **Cód. Municipio IBGE** é `3304557` |
| **Município Prestação Serviço** | `3304557` (Rio de Janeiro) |
| Valor do serviço | R$ 10.000,00 |

**Resultado**

Filial e município de prestação são o mesmo, mas o município está marcado: a retenção é calculada com a alíquota cadastrada.

| Item | Valor |
| --- | --- |
| Valor do serviço | R$ 10.000,00 |
| ISS retido (4%) | R$ 400,00 |
| Líquido a receber do cliente | R$ 9.600,00 |

Sem a marca no município, a mesma nota sairia sem ISS retido, com o ISS próprio de R$ 400,00 a recolher pela própria empresa.

---

## 6. Resumo dos Campos Novos

### Tabela Município (`MunicipioFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **ISS Retido no Mesmo Município** (`RetISSSameMunicipality`) | Sim/Não | Indica que o município retém o ISS dos prestadores estabelecidos nele. Marcado, habilita o cálculo do ISS retido na venda de serviço prestado dentro do próprio município da filial. Desmarcado (padrão), mantém a regra geral |
