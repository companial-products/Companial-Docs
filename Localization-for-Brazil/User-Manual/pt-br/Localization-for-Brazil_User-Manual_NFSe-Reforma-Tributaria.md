<!-- PR: #267 -->
# NFS-e e Reforma Tributária — Configuração e Emissão

**Data**: Maio/2026

---

## Sumário

A NFS-e (Nota Fiscal de Serviços Eletrônica) é emitida pelo Business Central quando uma venda de serviço é registrada. Com a Reforma Tributária (EC 132/2023 + LC 214/2025), a partir de 01/05/2026 os campos de IBS, CBS e Imposto Seletivo são acrescentados automaticamente ao XML da NFS-e, coexistindo com o ISS durante o período de transição (até 2033). Este manual descreve os pré-requisitos de configuração, os cadastros necessários, os campos essenciais e o passo a passo completo para emissão e transmissão.

---

## 1. Visão Geral

### 1.1 O Caminho de uma NFS-e

Antes de entrar nos detalhes, é importante entender o caminho que a informação percorre:

```
CONFIGURAÇÃO (feita uma vez)
       ↓
ITEM  →  campo "Cód. Serviço" aponta para o Código de Serviço NFS
       ↓
DOCUMENTO FISCAL  →  sistema lê o Cód. Serviço e monta o RPS com todos os dados
       ↓
XML DA NFS-e  →  cada campo configurado vira uma tag obrigatória no XML
```

Ou seja: **tudo parte do Código de Serviço NFS**. Ele é o repositório central de informações fiscais do serviço. 

### 1.2 Fases do Fluxo de Emissão

O fluxo de emissão tem duas fases:

1. **Registro no BC** → criação do Documento Fiscal, cálculo de ISS (e agora IBS/CBS/IS pela Reforma Tributária) e geração do RPS.
2. **Transmissão municipal** → o RPS é enviado ao sistema da prefeitura, que retorna o número definitivo da NFS-e.

---

## 2. Pré-Requisitos de Configuração

Antes de emitir qualquer NFS-e com os novos tributos, confirme os itens abaixo. Esta verificação precisa ser feita **uma única vez** (ou ao implantar um novo ambiente).

### 2.1 Status da Reforma Tributária

**Caminho**: Diga-me > **Configurações Fiscais** (`FiscalSetupFNX`)

| Campo | Valor esperado |
|-------|----------------|
| **Status Reforma Tributária** | `Agendada` (ativa automaticamente em 01/05/2026) ou `Habilitada` (para testes antecipados) |

> Se o campo estiver como `Desabilitada`, os grupos IBS/CBS **não** serão incluídos no XML.

### 2.2 Áreas de Impostos com Jurisdições IBS/CBS

**Caminho**: Diga-me > **Áreas de Impostos** (`TaxAreasFNX`)

A Área de Impostos usada nas vendas de serviço precisa incluir as três novas jurisdições:

| Classificação Imposto | Tributo |
|-----------------------|---------|
| **IBSUF** | IBS Estadual |
| **IBSMun** | IBS Municipal |
| **CBS** | CBS Federal |

> Se a Área de Impostos não tiver essas jurisdições cadastradas, os valores de IBS/CBS ficarão zerados no XML.

### 2.3 Classificações Tributárias IBS/CBS

**Caminho**: Diga-me (Lupa) > **Classificações Tributárias IBS/CBS** (`IBSCBSTaxClassifications`)

Mantenha a tabela de **cClassTrib** atualizada. Cada código de 6 dígitos define o tratamento tributário da operação (tributação integral, reduzida, isenta, etc.). Exemplo: `000001` = tributação integral padrão.

> **Atenção**: documentos fiscais de serviço **não** aceitam múltiplas classificações tributárias IBS/CBS por documento. Todas as linhas do mesmo documento fiscal devem usar a mesma classificação.

### 2.4 Alíquotas IBS/CBS

**Caminho**: Diga-me (Lupa) > **Alíquotas IBS/CBS** (`IBSCBSTaxPercentages`)

As alíquotas têm vigência por período (mudam anualmente durante a transição 2026–2033). Verifique se as alíquotas do período atual estão cadastradas para CBS, IBS UF e IBS Municipal.

---

## 3. Configuração dos Cadastros

### 3.1 Cód. Operações de Serviços (indOp)

**Onde fica:** Menu Fiscal → Cód. Operações de Serviços (ou busque "Operações de Serviços")

O `indOp` é um código de 6 dígitos definido pelo **Anexo VII da LC 214/2025**. Ele informa à prefeitura e ao governo federal *qual é a natureza da operação* e, consequentemente, *qual localidade deve ser usada para tributar* com IBS e CBS.

#### Estrutura do código

O código tem sempre 6 dígitos, formados por dois blocos:

```
  0301  01
  ↑↑↑↑  ↑↑
  │      └─ Localidade específica (01 = fornecedor, 02 = adquirente, etc.)
  └──────── Tipo de operação (03 = serviço físico sobre pessoa)
```

#### Tabela completa dos códigos válidos (Anexo VII)

| Cód. indOp | Tipo de Operação | Característica do Fornecimento | Local de Tributação no DFe |
|---|---|---|---|
| **020101** | Bem imóvel / bem imaterial relacionado a imóvel | Op. com bem imóvel, bem imaterial, inclusive direito, relacionado a bem imóvel | Localidade do imóvel |
| **020201** | Serviço físico sobre bem imóvel | Execução de serviços sobre bem imóvel | Localidade do imóvel |
| **020301** | Administração / intermediação de bem imóvel | Serviços de administração e intermediação de bens imóveis | Localidade do imóvel |
| **030101** | Serviço físico sobre a pessoa ou fruído presencialmente | Serviços prestados fisicamente sobre a pessoa ou fruídos presencialmente | Estabelecimento do fornecedor |
| **030102** | idem | idem | Endereço do adquirente |
| **030103** | idem | idem | Endereço do destinatário |
| **030104** | idem | idem | Endereço diverso do fornecedor, adquirente ou destinatário |
| **040101** | Feiras, exposições, congressos, espetáculos | Planejamento, organização e administração de feiras, exposições, congressos, espetáculos | Local do Evento |
| **050101** | Serviço físico sobre bem móvel material | Serviços fisicamente prestados sobre bem móvel material | Estabelecimento do fornecedor |
| **050102** | idem | idem | Endereço do adquirente |
| **050103** | idem | idem | Endereço do destinatário |
| **050104** | idem | idem | Endereço diverso |
| **050201** | Serviços portuários | Execução de serviços portuários | Local da prestação |
| **060101** | Transporte de passageiros | Execução de serviços de transporte de passageiros | Local de início do transporte |
| **070101** | Transporte de carga | Execução de serviços de transporte de carga | Endereço fornecido para entrega |
| **070102** | idem | idem | Local da retirada |
| **080101** | Exploração de via | Execução de serviços de exploração de via | Local proporcional à extensão da via explorada |
| **100101** | Cessão de espaço publicitário — onerosa | Cessão de espaço para prestação de serviços publicitários | Domicílio principal do adquirente |
| **100102** | idem | idem | Domicílio do destinatário (adquirente no exterior) |
| **100201** | Cessão de espaço publicitário — não onerosa | Cessão de espaço para prestação de serviços publicitários | Domicílio principal do destinatário |
| **100301** | Demais serviços — onerosos | Demais serviços não enquadrados nos incisos anteriores (ou Inc. III prestado à distância) | Domicílio principal do adquirente |
| **100302** | idem | idem | Domicílio do destinatário (adquirente no exterior) |
| **100401** | Demais serviços — não onerosos | Demais serviços não enquadrados nos incisos anteriores | Domicílio principal do destinatário |
| **100501** | Demais bens imateriais / direitos — onerosos | Demais operações com bens móveis imateriais, inclusive direitos | Domicílio principal do adquirente |
| **100502** | idem | idem | Domicílio do destinatário (adquirente no exterior) |
| **100601** | Demais bens imateriais / direitos — não onerosos | Demais operações com bens móveis imateriais, inclusive direitos | Domicílio principal do destinatário |

> **Atenção (Art. 11, §4º):** Serviços do Inc. III (código que começam com 03) que forem prestados *parcialmente à distância* migram automaticamente para o Inc. X (códigos que começam com 10)— ou seja, devem usar `100301` ou em vez de `030101`.

#### Como carregar esses códigos no BC

Acesse **Cód. Operações de Serviços** e cadastre cada linha com:
- **Código:** o valor de 6 dígitos da coluna `indOp` (ex: `100301`)
- **Característica Fornecimento:** a descrição da coluna correspondente da tabela acima
- **Local de Fornecimento:** a classificação da coluna *Local de Tributação no DFe*, conforme o quadro abaixo

Depois, no **Código de Serviço NFS** (detalhado mais adiante na seção 4) de cada serviço, preencha o campo **Cód. Operação de Serviços** com o `indOp` correto para aquele tipo de serviço.

#### Local de Fornecimento — o que ele controla

O **Local de Fornecimento** traduz a coluna *Local de Tributação no DFe* para uma classificação que o sistema entende, e é ele que decide **qual cadastro de local da prestação** o documento precisa informar:

| Local de Fornecimento | Corresponde a | O que o documento precisa informar |
|---|---|---|
| **Imóvel** | Localidade do imóvel | O **Cód. Imóvel** — sai no DANFSe como `Imóvel: <endereço>` e leva CIB e inscrição imobiliária na NFS-e |
| **Atividade/Evento** | Local do evento | O **Cód. Atividade/Evento** — sai no DANFSe como `Local da atividade/evento: <endereço>`, com nome, ID e período |
| **Outros** | Endereço diverso, local da prestação, local da retirada, endereço de entrega, etc. | O endereço do local da prestação, digitado direto — sai no DANFSe como `Local da prestação: <endereço>` |
| **Destinatário** | Endereço / domicílio do destinatário | Nada — o endereço de envio já é o do destinatário |
| **Tomador/Adquirente** | Endereço / domicílio principal do adquirente | Nada — a tributação usa o endereço do próprio tomador, já identificado na nota |
| **Fornecedor** | Estabelecimento do fornecedor | Nada — a tributação usa o endereço do próprio prestador |
| *(em branco)* | — | Nada |

O **endereço de envio** do documento é sempre o endereço do **destinatário** — ele não é mais reaproveitado como local da prestação. Os dois vão juntos na mesma nota.

> **A obra tem precedência.** Se o **Código de Serviço NFS** estiver marcado como **Construção Civil**, o local da prestação é sempre a obra, qualquer que seja o Local de Fornecimento do `indOp`.

Ao atualizar o aplicativo, os códigos do Anexo VII já cadastrados são classificados automaticamente. A classificação só é aplicada onde o campo ainda estiver em branco — se você já tiver ajustado algum código manualmente, esse ajuste é preservado.

> **Atenção nos códigos de transporte e via explorada.** `060101` (local de início do transporte) e `080101` (extensão da via explorada) são classificados como **Outros**, mas nesses casos o endereço informado pode não ser o local de tributação. Se não for, deixe o Local de Fornecimento em branco nesses códigos.

### 3.2 Atividades/Eventos

**Onde fica:** Menu Fiscal → Atividades/Eventos (ou busque "Atividades/Eventos")

Quando o `indOp` do serviço tem **Local de Fornecimento = Atividade/Evento** (é o caso do `040101` — feiras, exposições, congressos e espetáculos), a NFS-e precisa informar o **período do evento**. Esses dados ficam num cadastro próprio, preenchido uma vez por atividade ou evento:

| Campo | Para que serve |
|---|---|
| **Código** | Identificação interna do evento no BC, usada para vincular o evento ao documento |
| **Descrição** | Nome do evento, para facilitar a identificação no cadastro |
| **ID da Atividade/Evento** | Identificador da atividade ou evento reportado na NFS-e. Opcional — deixe em branco se não houver um |
| **Data Início** / **Data Fim** | Período do evento, obrigatório no lançamento. O sistema impede uma data de fim anterior à de início |
| **CEP, Endereço, Número, Complemento, Bairro, Cidade, Cód. Território, Cód. País/Região** | Endereço do local do evento. Ao digitar o CEP, o restante é preenchido automaticamente |

No pedido ou na fatura de venda, abra a ação **NFS-e** na factbox **Dados Fiscais** e informe o **Cód. Atividade/Evento**. Se o registro vinculado tiver endereço cadastrado, ele preenche automaticamente o **Endereço do Local da Prestação** na mesma tela. O endereço de envio do documento não é tocado — ele continua sendo o do destinatário.

> **As datas ficam só no cadastro.** O documento guarda apenas o vínculo com o evento. Se você corrigir o período de um evento, todas as notas daquele evento passam a imprimir e transmitir o período corrigido — inclusive as já lançadas. Por isso o sistema **não permite excluir** um evento que já esteja usado em algum documento fiscal.

> **O lançamento exige a atividade ou evento.** Se o serviço tiver `indOp` com Local de Fornecimento igual a Atividade/Evento, o BC não deixa lançar sem o Cód. Atividade/Evento preenchido e sem as duas datas informadas no cadastro.

### 3.3 Obras

**Onde fica:** Menu Fiscal → Cód. Obras (ou busque "Constructions")

O cadastro de obras passou a ser a origem dos dados de construção civil informados na NFS-e, e ganhou os mesmos campos de endereço do cadastro de eventos, além do **Cód. ART**:

| Campo | Para que serve |
|---|---|
| **Código** | Identificação interna da obra no BC |
| **Descrição** | Nome da obra |
| **Nº CNO** | Cadastro Nacional de Obras — é o valor transmitido como código da obra na NFS-e |
| **Tipo de Obra** | Total ou parcial |
| **Cód. ART** | Anotação de Responsabilidade Técnica da obra, transmitida na NFS-e |
| **CEP, Endereço, Número, Complemento, Bairro, Cidade, Cód. Território, Cód. País/Região** | Endereço do canteiro de obras, copiado para o Endereço do Local da Prestação ao selecionar a obra |

Antes desta versão, o código da obra e a ART transmitidos vinham do **Código de Serviço NFS**, o que obrigava a criar um código de serviço por obra. Agora eles vêm da obra vinculada ao documento.

> **Compatibilidade.** Se o documento não tiver obra vinculada, o sistema continua transmitindo o **Código de Obra** e o **Cód. ART** preenchidos no Código de Serviço NFS. Nada deixa de funcionar — mas recomendamos migrar para o cadastro de obras, que permite usar o mesmo código de serviço em várias obras.

> Assim como no evento, **não é possível excluir** uma obra já usada em documento fiscal.

### 3.4 Imóveis

**Onde fica:** Menu Fiscal → Imóveis (ou busque "Imóveis")

Quando o `indOp` do serviço tem **Local de Fornecimento = Imóvel** (os códigos `020101`, `020201` e `020301` — locação, administração e corretagem de imóveis), a NFS-e precisa identificar o imóvel. Esses dados ficam num cadastro próprio:

| Campo | Para que serve |
|---|---|
| **Código** | Identificação interna do imóvel no BC, usada para vincular o imóvel ao documento |
| **Descrição** | Nome do imóvel, para facilitar a identificação no cadastro |
| **CIB** | Cadastro Imobiliário Brasileiro — identifica o imóvel nacionalmente na NFS-e |
| **Inscrição Imobiliária Fiscal** | Cadastro do imóvel na prefeitura, usado quando o município identifica o imóvel pelo registro próprio em vez do CIB |
| **Cód. Município** | Município IBGE do imóvel, informado na NFS-e. Em branco, o município é obtido a partir do CEP |
| **CEP, Endereço, Número, Complemento, Bairro, Cidade, Cód. Território, Cód. País/Região** | Endereço do imóvel, copiado para o Endereço do Local da Prestação ao selecionar o imóvel |

No pedido ou na fatura de venda, abra a ação **NFS-e** na factbox **Dados Fiscais** e informe o **Cód. Imóvel**.

> **Pelo menos uma identificação.** O imóvel vinculado precisa ter CIB, inscrição imobiliária fiscal **ou** endereço preenchido — o layout nacional aceita qualquer um dos três. Sem nenhum deles, o registro é bloqueado.

> **Imóvel no exterior.** Um imóvel com Cód. País/Região diferente do Brasil é informado na NFS-e como imóvel estrangeiro.

> Assim como no evento e na obra, **não é possível excluir** um imóvel já usado em documento fiscal.

### 3.5 Numeração automática dos códigos

Os códigos de Obra, Atividade/Evento e Imóvel podem ser atribuídos automaticamente por numeração, em vez de digitados.

**Onde fica:** Configuração Fiscal → grupo **NFS-e**

| Campo | Cadastro |
|---|---|
| **Nos. Obra** | Obras |
| **Nos. Atividade/Evento** | Atividades/Eventos |
| **Nos. Imóvel** | Imóveis |

Deixe em branco para continuar digitando os códigos — a numeração é opcional e nada muda para quem já cadastra manualmente.

> **Use um formato curto.** O código aceita 10 caracteres. Se a numeração gerar um número maior, o sistema avisa e não cria o registro, em vez de cortar o código e correr o risco de gerar códigos repetidos.

### 3.6 Classificações Tributárias IBS/CBS (cClassTrib)

**Onde fica:** Menu Fiscal → Reforma Tributária → Classificações (ou em Diga-me (Lupa) > **Classificações Tributárias IBS/CBS**)

Enquanto o `indOp` define *onde* o serviço é tributado, o `cClassTrib` define *como* ele é tributado pelo IBS e CBS — ou seja, qual alíquota se aplica, se há redução de base, se há diferimento, etc.

É um código de 6 dígitos cadastrado nesta tabela. Cada código representa um regime tributário específico definido pela LC 214/2025.

Exemplos de uso:
- `000001` — tributação integral (alíquota cheia de IBS + CBS)
- `200025` — redução de base (IBS 60%, CBS 100% isentos)
- `410999` — operação não tributada (sem IBS nem CBS)
- `550001` — suspensão com tributação regular simultânea

O campo aparece na **linha** do documento fiscal. O sistema calcula os valores de IBS e CBS com base nesse código, nas alíquotas configuradas em **Alíquotas IBS/CBS** e na data da operação.

---

## 4. Código de Serviço NFS

**Onde fica:** Menu Fiscal → Códigos de Serviço NFS (ou busque "Códigos de Serviço")

Esta é a tabela mais importante para a NFS-e. Cada tipo de serviço que a empresa presta deve ter um registro aqui. Pense nela como uma "ficha completa do serviço" — ela reúne o código municipal, a classificação nacional, o enquadramento tributário da Reforma e o tipo de operação.

### 4.1 Campos da tabela

| Campo | O que é | Por que existe |
|---|---|---|
| **Código** | Identificador interno (ex: `CONSUL`, `DESEN`) | Chave da ficha, referenciada no cadastro do item |
| **Descrição** | Nome do serviço | Informativo |
| **Código de Tributação** | Código municipal (ex: `1701`) | Exigido pelo web service da prefeitura |
| **Código do Item de Serviço** | Item da lista LC 116/2003 (ex: `17.06`) | Identifica o tipo de serviço para o município |
| **Cód. CNAE** | Classificação Nacional de Atividades Econômicas | Exigido por alguns municípios |
| **Cód. NBS** | Nomenclatura Brasileira de Serviços (ex: `1.0107.51.00`) | **Reforma Tributária** — classificação nacional obrigatória no padrão federal SEFIN. Também obrigatório quando o tomador é estrangeiro |
| **Cód. Operação de Serviços** | Código `indOp` do Anexo VII (ex: `100301`) | **Reforma Tributária** — define *onde* o serviço é tributado (município do fornecedor, do adquirente, do local do evento, etc.) |
| **Tipo de Suspensão ISS** | Se há suspensão de ISS (judicial, administrativa, etc.) | Continua relevante durante a transição 2026–2032 |
| **Nº do Processo de Suspensão** | Número do processo que fundamenta a suspensão | Complementa o campo anterior |
| **Cód. ART** | Anotação de Responsabilidade Técnica | Para serviços de engenharia/arquitetura |
| **Código de Obra** | Código da obra (construção civil) | Para serviços de construção civil |
| **Cód. IBPT** / **Tipo Cód. IBPT** | Carga tributária aproximada (Lei do SAT) | Informação ao consumidor |
| **Cód. Terceirização Reinf** | Código EFD-Reinf para serviços terceirizados | Para obrigações acessórias do EFD-Reinf |

### 4.2 Configuração por Filial

Algumas informações do Código de Serviço NFS podem variar por filial. Para isso existe a página **Cód. Serviços NFS por Filial**, acessível dentro de cada Código de Serviço NFS pelo botão **Cód. por Filial**.

Os campos que podem ser sobrescritos por filial são: Código de Tributação, Código do Item de Serviço, Cód. CNAE, Código de Obra, Cód. ART e Tipo de Suspensão ISS.

Quando o sistema monta a NFS-e, ele primeiro verifica se existe configuração específica para a filial emissora. Se existir, usa os valores da filial. Se não existir, usa os valores globais do Código de Serviço NFS.

---

## 5. Cadastro do Item

**Onde fica:** Cartão do Item → aba Fiscal (ou aba Localização)

O item do tipo **Serviço** precisa ter o campo **Cód. Serviço** preenchido. Esse campo aponta para o **Código de Serviço NFS** descrito na seção 4.

É por ele que o sistema sabe, ao registrar a venda, qual o código de tributação, qual o `indOp`, qual o NBS e todos os demais dados fiscais do serviço.

> O mesmo campo existe no **Cartão do Ativo Imobilizado**, para casos em que a nota é gerada a partir de um ativo.

---

## 6. Casos Especiais: Exportação e Tomador Estrangeiro

### 6.1 Exportação de serviço

Quando o documento fiscal tem **Categoria = Exportação (Foreign)**:

- O sistema procura a **moeda estrangeira** informada no documento
- Preenche o XML com o **Código ISO Numérico** da moeda (ex: `840` para USD)
- Se o documento for marcado como exportação mas não houver moeda estrangeira, usa `986` (BRL)

**O que configurar:** No cadastro de **Moedas** do BC, certifique-se de que o campo **Cód. ISO Numérico** está preenchido para cada moeda usada em exportações.

**Serviço prestado no exterior.** Quando o serviço é executado fora do Brasil, marque **Exportação de Serviço** na página **Dados da NFS-e** do documento de venda e informe o **Cód. País/Região da Prestação**. O **Município Prestação Serviço** deixa de ser exigido, a NFS-e do ambiente nacional sai com o país da prestação no lugar do município e com os dados de comércio exterior do documento, e o IBS/CBS é apurado no município EXTERIOR (`9999999`). Não use o município EXTERIOR como local da prestação no ambiente nacional: ele é recusado com o erro `E0302`. Ver [NFS-e: Exportação de Serviço — País da Prestação e Dados de Comércio Exterior](Localization-for-Brazil_User-Manual_NFSe-Exportacao-Servico-Exterior.md).

### 6.2 Tomador estrangeiro

Quando o **cliente** tem **Categoria = 3.- Estrangeiro**:

- O CEP e o código IBGE de município deixam de ser obrigatórios
- O **Cód. ISO** do país do cliente é obrigatório no cadastro do país/região
- O campo **Cód. NBS** no Código de Serviço NFS torna-se **obrigatório** (validado antes da transmissão para sistemas SEFIN)
- O código BACEN do país também é enviado no XML
- O cliente precisa ter o **Cód. NIF** ou o **Motivo da não informação do NIF** preenchido (validado antes da transmissão para sistemas SEFIN — ver [NIF ou motivo da não informação do NIF](#nif-ou-motivo-da-não-informação-do-nif))

**O que configurar:**
1. No cadastro do **cliente**: Categoria = `3.- Estrangeiro`, País/Região preenchido
2. No endereço do **cliente** e no **Endereço de Envio** do exterior: **Cód. Território** = `EX`, o território do exterior, cadastrado com **Código IBGE** = `99` (ver [Endereço do exterior: UF EX com Código IBGE 99](#endereço-do-exterior-uf-ex-com-código-ibge-99))
3. No **Cartão Cliente**, grupo **Fiscal**: **Cód. NIF** preenchido ou, se o cliente não tiver NIF, **Motivo da não informação do NIF** escolhido
4. No cadastro do **País/Região**: campo **Cód. ISO** preenchido (ex: `US`, `DE`, `AR`)
5. No **Código de Serviço NFS**: campo **Cód. NBS** obrigatoriamente preenchido

#### Endereço do exterior: UF EX com Código IBGE 99

Endereço fora do Brasil usa sempre a UF do exterior, **`EX`**, no campo **Cód. Território**: não deixe o campo em branco nem informe a UF de um estado brasileiro. O território `EX` precisa existir na página **Territórios** com o **Código IBGE** = **`99`**, o mesmo `99` que abre o código do município EXTERIOR, `9999999`.

| Onde | Campo | Valor |
|------|-------|-------|
| **Cartão Cliente** do cliente do exterior | **Cód. Território** | `EX` |
| **Endereço de Envio** no exterior | **Cód. Território** | `EX` |
| **Territórios**, registro `EX` | **Código IBGE** | `99` |
| **Municípios**, município EXTERIOR | **Código IBGE** / **Cód. Território** | `9999999` / `EX` |

O território e o município do exterior trabalham juntos na exportação de serviço:

- O IBS/CBS de um serviço prestado no exterior é apurado no município EXTERIOR (`9999999`). A UF desse município sai do código IBGE: `99` corresponde a `EX`, e é para a UF `EX` que o sistema procura a alíquota do IBS estadual.
- Nas filiais de provedor municipal, a exportação de serviço envia o município `9999999` como local da prestação. Por isso o município EXTERIOR precisa estar cadastrado em **Municípios**, no território `EX`. No padrão nacional (`SEFIN`), a nota leva o país da prestação no lugar do município.

#### NIF ou motivo da não informação do NIF

O layout nacional identifica o tomador do exterior pelo NIF (Número de Identificação Fiscal), o documento fiscal do país de origem, lido do campo **Cód. NIF** do **Cartão Cliente**. Há países que não emitem NIF e operações em que ele é dispensado. Nesses casos, o layout aceita o tomador sem NIF, desde que a nota traga o motivo (grupo `cNaoNIF`), informado no campo **Motivo da não informação do NIF**, ao lado do **Cód. NIF**:

| Opção | Código na NFS-e | Quando usar |
|-------|:---------------:|-------------|
| **Dispensado do NIF** | 1 | O tomador é dispensado de apresentar o NIF |
| **Não exigência do NIF** | 2 | O país do tomador não exige o NIF |

O layout prevê também o código 0 (*não informado na nota de origem*). Ele não é oferecido no cadastro porque é reservado às notas que as prefeituras repassam ao ambiente nacional.

Regras do cadastro:

- Com o **Cód. NIF** preenchido, o **Motivo da não informação do NIF** fica desabilitado, porque não se aplica.
- Preencher o **Cód. NIF** apaga o motivo que estava gravado. Ao apagar o **Cód. NIF**, o motivo volta a ficar editável, em branco.
- Os dois campos continuam **opcionais** no cadastro: o mesmo cliente pode receber NF-e de exportação, que não exige o NIF.
- O motivo também pode ser gravado pela API de clientes da localização, no campo `nifNotInformedReason` — ver [APIs da Localização Brasileira: Cliente, Fornecedor e Item](Localization-for-Brazil_User-Manual_APIs-Localizacao-Cliente-Fornecedor-Item.md).

Nas filiais que transmitem pelo padrão nacional (**Cód. Cidade WS** = `SEFIN` na **Config. NF-e**), a NFS-e de tomador estrangeiro sai assim:

| Cadastro do cliente | Na transmissão |
|---------------------|----------------|
| **Cód. NIF** preenchido | A nota sai com o NIF (`<NIF>`) |
| **Cód. NIF** em branco e **Motivo da não informação do NIF** preenchido | A nota sai com o motivo (`<cNaoNIF>`) |
| Nenhum dos dois | A transmissão é **bloqueada** antes de qualquer envio (ver abaixo) |

Os provedores municipais que também usam o layout nacional (ISSNet Nacional, Betha Cloud, Governa Brasil e ISSWeb 2.04 Reforma) passam a enviar o motivo quando o cliente não tem NIF, mas não bloqueiam a nota sem nenhum dos dois. Os demais provedores e a NF-e, inclusive de exportação, não mudam.

#### Bloqueio da transmissão sem NIF e sem motivo

Antes, a nota de tomador estrangeiro sem NIF seguia para o ambiente nacional com o NIF vazio e voltava rejeitada com o erro `E1235` (falha de esquema, `NIF ... MinLength`). Agora a transmissão é interrompida antes de o RPS ser numerado e de qualquer comunicação com o governo, com a mensagem:

> *"O documento fiscal &lt;Nº do documento&gt; não pode ser transmitido: o cliente &lt;Nº do cliente&gt; é do exterior e não tem Cód. NIF nem Motivo da não informação do NIF. Preencha um dos dois na ficha do cliente, aba Fiscal."*

Quando a mensagem aparece em tela, a ação **Mostrar Registro** abre o **Cartão Cliente** indicado. A verificação vale em todos os caminhos de transmissão (seção [9](#9-transmissão-do-rps)):

| Caminho | O que o usuário vê |
|---------|--------------------|
| **Transmitir** no documento fiscal, ou **Transmitir Doc. Selecionado** na **NFS-e - Envio de Lote** | A mensagem em tela; nada é gravado |
| **Transmitir Docs. Pendentes** na **NFS-e - Envio de Lote** | As demais notas seguem; a barrada fica com erro, e a mensagem aparece em **Lista Erros** |
| Rotina agendada de geração de lote (**Movimentos Bot NFS-e**) | O lote não é criado, e a mensagem fica na **Última Mensagem de Erro** das entradas |

Se um lote reúne várias notas e uma delas cai no bloqueio, o lote inteiro é barrado, como já acontece com as outras pendências de cadastro do cliente. Para resolver, preencha o **Cód. NIF** ou o **Motivo da não informação do NIF** no **Cartão Cliente** e transmita a mesma nota de novo — nenhum outro passo é necessário. A impressão do DANFSe e do RPS não passa por essa verificação: nota já emitida continua podendo ser reimpressa.

---

## 7. Campos Essenciais na Ordem/Fatura de Venda

### 7.1 Cabeçalho da Ordem

| Campo | Por que é essencial |
|-------|---------------------|
| **Cód. Filial** | Define qual filial emite a NFS-e e qual configuração de web service será usada |
| **Tipo Documento Fiscal** | Deve ser `NFS-e` — determina que o documento gerado será transmitido à prefeitura |
| **Cliente** | CNPJ/CPF válido e dados cadastrais completos (a localização valida antes de registrar) |
| **Cód. Área Imposto** | Define quais impostos serão calculados |
| **Endereço de Envio** | É o endereço do **destinatário da operação** na NFS-e. Informe-o quando quem recebe o serviço está num endereço diferente do cliente |

O local da prestação não fica no cabeçalho: informe-o pela ação **NFS-e** da factbox **Dados Fiscais**, descrita na seção 3.

### 7.2 Linhas da Ordem

| Campo | Por que é essencial |
|-------|---------------------|
| **Nº Item** | Deve ser do tipo **Serviço** (não produto); a localização identifica o tipo de NF pela natureza do item |
| **Cód. Área Imposto** | Define quais impostos serão calculados |
| **Cód. Serviço** | Código do serviço, informado dentro do cartão do Item. Este código deve estar configurado com todos os dados pertinentes já detalhados neste documento, conforme a instrução do município. |
| **Cód. Class. Trib. IBS/CBS** | Código cClassTrib para IBS/CBS — deve estar corretamente parametrizado |

---

## 8. Passo a Passo — Emissão da NFS-e

### Passo 1 — Criar a Ordem/Fatura de Venda

Atenção especial aos campos mostrados na **seção 7**, acima.

### Passo 2 — Verificar os Impostos Calculados

Antes de registrar, abra a página de **Estatísticas** da ordem. Confira:

| Imposto | O que verificar |
|---------|----------------|
| **ISS** | Alíquota e valor correspondentes à regra municipal |
| **CBS** | Valor calculado com alíquota 2026 (alíquota-teste do período de transição) |
| **IBS UF** | Valor calculado com alíquota estadual 2026 |
| **IBS Municipal** | Valor calculado com alíquota municipal 2026 |

> Durante 2026–2032, **ISS e IBS/CBS coexistem** no mesmo documento.

### Passo 3 — Registrar a Ordem

1. Na faixa de opções, clique em **Registrar** (ou **Registrar e Enviar**)
2. A localização realiza as validações fiscais automáticas:
   - CNPJ/CPF do cliente válido
   - Campos fiscais obrigatórios preenchidos
   - Classificação Tributária IBS/CBS única por documento
3. Se a validação passar, o BC cria:
   - A fatura registrada padrão
   - O **Documento Fiscal** com todos os impostos detalhados

---

## 9. Transmissão do RPS

### 9.1 Transmissão pelo Documento Fiscal

1. Diga-me > **Documentos Fiscais** (`FiscalDocumentsFNX`) — filtre por Direção = Saída e Tipo = NFS-e
2. Selecione o documento desejado e abra o cartão
3. Clique em **Transmitir** na faixa de opções
4. Acompanhe o retorno: número definitivo da NFS-e ou mensagem de erro

### 9.2 Transmissão pela Tela de Lote: NFS-e - Envio de Lote

A tela **NFS-e - Envio de Lote** (`TransmitNFSeBatchFNX`) é utilizada para transmitir à prefeitura múltiplas NFS-e de uma só vez. A lista exibe os Documentos Fiscais do tipo NFS-e já registrados que ainda não foram autorizados pelo município.

- Pesquise por **NFS-e - Envio de Lote** na barra de busca do Business Central.

- Informe o **Cód. Filial** no campo de filtro no topo da página. A lista será carregada automaticamente com os Documentos Fiscais NFS-e daquela filial a partir da data de início NFS configurada.

Há duas opções de transmissão:

- **Opção A — Transmitir todos os pendentes:**
  Clique em **Transmitir Docs. Pendentes**. O sistema enviará todos os documentos que ainda não foram transmitidos com sucesso. Será solicitada confirmação com a quantidade de documentos a enviar.
- **Opção B — Transmitir um documento específico:**
  Selecione a linha desejada e clique em **Transmitir Doc. Selecionado**. O sistema transmite somente aquele documento, desde que esteja pendente.

Após a transmissão, cada documento recebe um **Nº Lote NFS-e** que identifica o lote em que foi incluído.

> **Importante:**
> Grande parte das prefeituras, com a reforma tributária, estão com retorno síncrono, o que significa que imediatamente o status é atualizado para <u>sucesso</u> ou <u>erro</u>.
>
> Se o web service do município for assíncrono (retorno não imediato), selecione os documentos transmitidos e clique em **Consultar**. O sistema consultará a prefeitura e atualizará o status dos documentos.

### 9.3 Verificando Erros e Avisos

Após a transmissão ou consulta, se houver retornos de erro da prefeitura:

- Clique em **Lista Erros** para visualizar as mensagens de erro do lote.

O botão fica habilitado automaticamente quando há status de erro para o lote da linha selecionada.

### 9.4 Ações Auxiliares

| Ação | Quando usar |
|------|-------------|
| **Criar Novo Nº** | Gera um novo número de sequência RPS para o documento selecionado, permitindo retransmissão. Útil quando o número original foi rejeitado ou expirou. |
| **Criar Novos Nº (Seleção)** | Mesmo que o anterior, mas para múltiplos documentos selecionados ao mesmo tempo. |
| **Recuperar Documento** | Visível apenas em ambientes que usam o integrador PlugNotas. Tenta recuperar as informações do documento diretamente do provedor externo. |

---

## 10. Referência Técnica

### 10.1 Resumo: O que Configurar e em Qual Ordem

```
PASSO 1 — Cód. Operações de Serviços
   Cadastrar os 26 códigos indOp da tabela do Anexo VII.
   (Menu Fiscal → Cód. Operações de Serviços)

PASSO 1.1 — Atividades/Eventos, Obras e Imóveis (quando aplicável)
   Cadastrar as atividades e eventos (com período e endereço), as obras
   (com CNO, ART e endereço) e os imóveis (com CIB, inscrição
   imobiliária e endereço) que serão vinculados aos documentos.
   (Menu Fiscal → Atividades/Eventos / Cód. Obras / Imóveis)

PASSO 2 — Classificações Tributárias IBS/CBS
   Cadastrar os cClassTrib que se aplicam aos serviços da empresa.
   (Menu Fiscal → Reforma Tributária → Classificações)

PASSO 3 — Alíquotas IBS/CBS
   Configurar as alíquotas por período e jurisdição.
   (Menu Fiscal → Reforma Tributária → Alíquotas IBS/CBS)

PASSO 4 — Código de Serviço NFS
   Para cada tipo de serviço prestado, criar uma ficha com:
   ✔ Código de Tributação (municipal)
   ✔ Item da Lista (LC 116/2003)
   ✔ Cód. NBS (padrão federal / Reforma)
   ✔ Cód. Operação de Serviços (indOp — Reforma)
   ✔ Tipo de Suspensão ISS (se aplicável)
   (Menu Fiscal → Códigos de Serviço NFS)

PASSO 5 — Cadastro do Item
   Em cada item do tipo Serviço, preencher o campo Cód. Serviço
   apontando para o registro criado no Passo 4.
   (Cartão do Item → aba Fiscal)

PASSO 6 — Documento Fiscal
   Na linha do documento fiscal, verificar/preencher:
   ✔ Cód. Class. Trib. IBS/CBS (cClassTrib)
   (Vem do item mas pode ser ajustado manualmente na linha)
```

### 10.2 Mapeamento: XML da NFS-e × Campos no BC

| Tag XML | De onde vem no BC | Obrigatório |
|---|---|---|
| `<ItemListaServico>` | Código de Serviço NFS → Código do Item de Serviço | Sempre |
| `<CodTributacaoMunicipio>` | Código de Serviço NFS → Código de Tributação | Sempre |
| `<CodNBS>` | Código de Serviço NFS → Cód. NBS | Padrão nacional e tomador estrangeiro |
| `<CodIndOperacao>` | Código de Serviço NFS → Cód. Operação de Serviços (indOp) | Reforma Tributária |
| `<IBSCBS><CST>` | Calculado — CST de IBS/CBS | Reforma Tributária |
| `<IBSCBS><cClassTrib>` | Linha do doc. fiscal → Cód. Class. Trib. IBS/CBS | Reforma Tributária |
| `<TomadorEstrangeiro>` | Cliente → Categoria = Estrangeiro | Quando aplicável |
| `<TomadorPaisISO>` | Cadastro de País/Região → Cód. ISO | Tomador estrangeiro |
| `<TomadorNIF>` | Cliente → Cód. NIF | Tomador estrangeiro com NIF |
| `<TomadorMotivoNaoNIF>` | Cliente → Motivo da não informação do NIF | Tomador estrangeiro sem NIF (padrão nacional) |
| `<MoedaServicos>` | Moeda do documento (Cód. ISO Numérico) | Exportação de serviço |
| `<ISSSuspensionType>` | Código de Serviço NFS → Tipo de Suspensão ISS | Quando há suspensão |
| `<SuspensionProcessNo>` | Código de Serviço NFS → Nº do Processo | Quando há suspensão |
| `<CodCNAE>` | Código de Serviço NFS → Cód. CNAE | Municípios que exigem |
| `<ConstrCivilCodigoObra>` | Documento → Cód. Obra → Nº CNO da obra (se não houver obra vinculada: Código de Serviço NFS → Código de Obra) | Construção civil |
| `<ConstrCivilART>` | Documento → Cód. Obra → Cód. ART da obra (se não houver obra vinculada: Código de Serviço NFS → Cód. ART) | Construção civil |
| `<EventoId>` | Documento → Cód. Atividade/Evento → ID da Atividade/Evento | Local de Fornecimento = Atividade/Evento |
| `<EventoDataInicio>` | Documento → Cód. Atividade/Evento → Data Início | Local de Fornecimento = Atividade/Evento |
| `<EventoDataFim>` | Documento → Cód. Atividade/Evento → Data Fim | Local de Fornecimento = Atividade/Evento |

### 10.3 Diagnóstico Rápido

| Sintoma | Causa mais provável | Onde verificar |
|---------|---------------------|----------------|
| IBS/CBS zerado ou ausente | Área de Impostos sem as jurisdições IBSUF/IBSMun/CBS | Diga-me > **Áreas de Impostos** |
| IBS/CBS zerado | Alíquota IBS/CBS não cadastrada para o período | Diga-me > **Alíquotas IBS/CBS** |
| Erro "múltiplas classificações IBS/CBS" | Linhas do documento com **Cód. Class. Trib. IBS/CBS** diferentes | Ajuste as linhas para o mesmo código |
| cClassTrib vazio no XML | Linha sem o campo **Cód. Class. Trib. IBS/CBS** preenchido | Verificar o campo na linha do documento |
| RPS rejeitado pela prefeitura | **Cód. Serviço** inválido para o município | Analisar o **Cód. Serviço** cadastrado no cartão do item |
| ISS zerado | Área de Impostos sem jurisdição de ISS ou sem alíquota | Analisar as **Áreas de Impostos** usadas no documento |
| Transmissão bloqueada: cliente do exterior sem **Cód. NIF** nem **Motivo da não informação do NIF** | Tomador estrangeiro sem NIF e sem motivo, em filial SEFIN | **Cartão Cliente**, grupo **Fiscal** (seção [6.2](#62-tomador-estrangeiro)) |

### 10.4 Pontos de Atenção

- **Período de transição 2026–2032**: ISS e IBS/CBS coexistem obrigatoriamente no mesmo documento. Não é necessário escolher entre um e outro — ambos são calculados e enviados no XML.
- **Alíquotas provisórias**: as alíquotas de 2026 são de teste/transição e serão alteradas anualmente. Mantenha a tabela de **Alíquotas IBS/CBS** atualizada a cada publicação do governo.
- **Classificação única por documento**: todos os itens de um documento de serviço precisam ter o mesmo **Cód. Class. Trib. IBS/CBS**. Se a operação envolver serviços com classificações diferentes, emita documentos separados.
- **Legislação em evolução**: regulamentações e Notas Técnicas estão sendo publicadas continuamente. A localização é atualizada a cada nova NT. Acompanhe as atualizações mensais no portal.
