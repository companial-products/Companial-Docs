<!-- Branch: master -->
# FCI — Ficha de Conteúdo de Importação

**Data**: Maio/2026

---

## Sumário

A **Ficha de Conteúdo de Importação (FCI)** é a obrigação acessória, prevista na **Resolução do Senado Federal nº 13/2012** e regulamentada pelo **Ajuste SINIEF 19/2012**, exigida das empresas que praticam **operações interestaduais** com produtos resultantes de **processo de industrialização** que tenham **conteúdo de importação superior a 40%**. Sua função é determinar a alíquota de **4% de ICMS** aplicável a esses bens nas saídas interestaduais. A localização brasileira do Business Central oferece o cálculo automatizado da FCI por filial, mês e item, a geração do arquivo TXT no layout oficial para upload no portal da SEFAZ e a importação do arquivo de retorno com o número da FCI atribuído pelo fisco.

Este documento descreve o que é a FCI dentro da Localização Brasil, lista todas as configurações necessárias e percorre o processo completo, desde a configuração até o processamento mensal e o ajuste do código de origem dos itens.

## 1. Contexto

### O que é a FCI

A FCI é o documento eletrônico em que o industrializador declara, **por item produzido**, o **Valor da parcela Importada (VI)**, o **Valor da Operação de saída interestadual (VO)** e o **Conteúdo de Importação (CI)**, definido como:

> **CI (%) = VI / VO × 100**

A faixa em que esse percentual se enquadra (até 40%, entre 40% e 70%, ou superior a 70%) determina o **código de origem do ICMS** atribuído ao item, que por sua vez controla o CST e a alíquota interestadual aplicada.

### Quando se aplica

- Empresas industrializadoras (item com **Sistema de Reabastecimento** = `Prod. Order` ou `Assembly`).
- Que adquirem **insumos importados** ou produtos com origem importada na cadeia de produção.
- Que praticam **vendas interestaduais** dos produtos resultantes da industrialização.
- A apuração é **mensal e por filial**. Cada estabelecimento (CNPJ) tem o seu próprio cálculo, pois posições de estoque, compras e vendas são distintas por filial.

### O que a localização entrega

- Cálculo automático mensal por filial e por item, consolidando compras (entrada de insumos), consumos de produção e montagem e vendas (saídas interestaduais e venda no mesmo estado).
- Geração do arquivo TXT no layout oficial para upload no portal da SEFAZ.
- Importação do arquivo de retorno com o número da FCI registrado pelo fisco e atualização automática do código de origem do item, das vendas em aberto e dos pedidos.
- Linha do tempo detalhada de VI por item (compras, produção, consumo) para auditoria e justificativa do cálculo.

---

## 2. Configurações Necessárias

Antes do primeiro cálculo, percorra os pontos abaixo. Sem qualquer um deles, a FCI não será gerada, ou será gerada com valores incorretos.

### 2.1. Habilitar o cálculo da FCI por filial

Pesquise por **Config. NF-e** (`NFeSetupFNX`) e abra o registro da filial.

| Campo | O que informar |
| --- | --- |
| **Calcular FCI** (`CalculateFCI`) | Marque para habilitar o cálculo da FCI nesta filial. Sem esta flag o relatório de cálculo não processa nada. |

> Repita a configuração para cada filial industrializadora.

### 2.2. Configurar a numeração dos arquivos FCI

Pesquise por **Configuração Contabilidade** e localize o campo:

| Campo | O que informar |
| --- | --- |
| **Nº Serie Arquivo FCI** (`FCIFileNosFNX`) | Código de uma **Série Numérica** (`No. Series`) que será usada para numerar os arquivos FCI gerados. |

A série numérica precisa estar previamente cadastrada na página **Número de Séries** (`No. Series`).

### 2.3. Cadastrar as Faixas de Conteúdo de Importação

Pesquise por **Faixas de Conteúdo de Importação** (`CIRangesFNX`). Cada linha representa uma faixa percentual de conteúdo de importação prevista pela Resolução SF 13/2012.

| Campo | Conteúdo típico |
| --- | --- |
| **Faixa** (`Range`) | Limite superior do percentual de CI. Tipicamente: `40`, `70` e `100`. |
| **Descrição** (`Description`) | Texto descritivo da faixa (ex.: `Até 40%`, `Entre 40% e 70%`, `Acima de 70%`). |
| **VI (%)** (`VIPercentage`) | Percentual aplicado ao valor da nota de compra para estimar o VI quando ainda não há FCI declarada para o item. |
| **Cód. Origem** (`ICMSOriginCode`) | Código de origem do ICMS associado a esta faixa (ex.: `3` para bens com CI entre 40% e 70%; `5` para bens com CI até 40%; `8` para bens com CI superior a 70%). |

> A faixa é o que determina o código de origem que será atribuído ao item após a importação do retorno da SEFAZ.

#### Exemplo de configuração

Configuração típica das três faixas previstas pela Resolução SF 13/2012:

| **Faixa** | **Descrição** | **Cód. Origem** | **VI (%)** |
| ---: | --- | :---: | ---: |
| 40,00 | Conteúdo importação até 40% | 5 | 0,00 |
| 70,00 | Conteúdo importação maior que 40% e até 70% | 3 | 50,00 |
| 100,00 | Conteúdo importação superior a 70% | 8 | 100,00 |

### 2.4. Cadastrar os Códigos de Origem ICMS

Pesquise por **Códigos Origem ICMS** (`ICMSOriginCodesFNX`).

Para os códigos que representam bens com conteúdo de importação (tipicamente `3`, `5`, `8`):

| Campo | O que informar |
| --- | --- |
| **Código** (`Code`) | Código numérico de origem (ex.: `3`, `5`, `8`). |
| **Descrição** (`Description`) | Descrição completa do código de origem. |
| **Nº FCI Obrigatório** (`FCINoMandatory`) | Marque para os códigos em que o número da FCI é exigido pelo fisco no XML da NF-e. |
| **Faixa CI** (`CIRange`) | Faixa de conteúdo de importação correspondente. Deve apontar para um registro de **Faixas de Conteúdo de Importação**. |

### 2.5. Atribuir Código de Origem aos itens importados

No cadastro do **Item**, no campo **Código Origem ICMS** (`OriginCodeFNX`), informe o código de origem inicial do item. Após o primeiro cálculo de FCI seguido pela importação do retorno da SEFAZ, este campo passa a ser atualizado automaticamente pela rotina de FCI, conforme a faixa de CI calculada.

### 2.6. Informação Filial / Informação Empresa completa

Pesquise por **Informação Filial** (`BranchInformationFNX`) ou, em empresas que operam apenas com um único estabelecimento e não usam filiais, por **Informação Empresa** (`Company Information`). Confira os dados que entram no arquivo TXT da FCI:

- **CNPJ**
- **IE** (Inscrição Estadual)
- **Nome**
- **Endereço** e **CEP**
- **Cidade**
- **Código Território** (UF)

Estes dados são gravados nos registros `0010` do arquivo TXT enviado à SEFAZ.

---

## 3. Dados Relevantes para o Cálculo

Para que a FCI seja gerada corretamente, os seguintes dados de movimentação precisam estar consistentes:

### 3.1. Itens elegíveis

A FCI só é calculada para itens cujo **Sistema de Reabastecimento** seja **`Prod. Order`** ou **`Assembly`**. Itens comprados e revendidos sem industrialização ficam fora do cálculo.

### 3.2. Compras (entrada de insumos)

Documentos fiscais de **entrada** registrados no período, com:

- **Item** com **Código Origem ICMS** (`OriginCodeFNX`) preenchido, o que indica que o insumo tem componente importado.
- O **Código Origem ICMS** do item associado a uma **Faixa CI** com **VI (%)** maior que zero (caso o insumo não seja de importação direta), ou o documento marcado como **Importação Direta** (DI). Nesse caso, o valor do item é considerado integralmente como VI.
- **Crédito de ICMS** apurado no documento. Quando o insumo não é de importação direta, o ICMS creditado é subtraído do valor antes do cálculo do VI.
- **Frete e Seguro** lançados como acessórios na linha do documento. Eles são somados ao valor base do VI.
- **Quantidade não devolvida**. Devoluções de compra reduzem proporcionalmente a quantidade e o valor considerados.

### 3.3. Produção / Montagem

Movimentos de estoque do tipo **`Output`** (Produção) e **`Assembly Output`** (Montagem), com seus respectivos **`Consumption`** e **`Assembly Consumption`** vinculados pelo número da ordem. Esses movimentos transferem o VI dos insumos consumidos para o produto produzido, na proporção da quantidade consumida.

### 3.4. Vendas (saídas)

Documentos fiscais de **saída registrados** do período, com:

- **Item** com **Código Origem ICMS** preenchido.
- **Situação Cliente/Fornecedor** classificada como **`Same state`** (mesmo estado) ou **`Other state`** (outro estado).
- **Valor da linha** maior que zero e **ICMS** apurado (que é subtraído para chegar ao VO).

> Quando o item ainda não teve vendas no mês, o sistema busca **Pedidos de Venda Liberados** com **Quantidade a Enviar** maior que zero como base estimada para o VO. Nesses casos, a alíquota interestadual cadastrada em **Percentual de Imposto por Território** (`TerritoryTaxPercentageFNX`) é usada para deduzir o ICMS estimado.

### 3.5. Saldo do mês anterior

A rotina sempre traz como **saldo inicial** o resultado do mês imediatamente anterior (mesmo `BranchCode` e mesmo `ItemNo`), para preservar o estoque acumulado de VI entre períodos.

---

## 4. Procedimento Mensal de FCI

A operação mensal segue cinco passos.

### Passo 1 — Calcular a FCI do período

1. Pesquise por **Lista de FCIs** (`FCIListFNX`).
2. Clique na ação **Calcular FCI**.
3. No painel de filtros do relatório:
   - **Mês** (`Month`): informe o mês a calcular (1 a 12).
   - **Ano** (`Year`): informe o ano (igual ou superior a 2013, quando a Resolução SF 13/2012 entrou em vigor).
   - **Código Filial** (`BranchCode`): informe a filial a processar.
4. Clique em **OK** para executar.

Por padrão, o sistema sugere o mês imediatamente anterior ao corrente (ex.: em maio sugere março, `<CM-2M>`), conforme a janela típica de geração da FCI.

Ao final, se houver itens com **Nova FCI Pendente** (`NewFCIPending`), o sistema avisa que existem novas FCIs pendentes para o período. São os itens que mudaram de **Faixa CI** em relação ao último cálculo e exigem novo número de FCI declarado à SEFAZ.

### Passo 2 — Conferir os resultados na Lista de FCIs

Na **Lista de FCIs** (`FCIListFNX`) é exibida uma linha por **Data de Cálculo** × **Código Filial** × **Nº Item**. As colunas relevantes são:

| Coluna | Significado |
| --- | --- |
| **Data Cálculo** (`CalculationDate`) | Último dia do mês calculado. |
| **Código Filial** (`BranchCode`) | Filial. |
| **Nº FCI** (`FCINo`) | Número da FCI atribuído pela SEFAZ. Em branco até a importação do retorno. |
| **Nº Item** (`ItemNo`) | Item produzido. |
| **Quantidade** (`Quantity`) | Quantidade total considerada no cálculo de VI. |
| **VI - Valor Parcela Importada** (`VIAmount`) | VI **por unidade** (Valor / Quantidade). |
| **VO - Valor Operação Saída Interestadual** (`VOAmount`) | VO por unidade, denominador do CI. |
| **CI - Conteúdo Importação** (`CIPercent`) | Percentual de conteúdo de importação. |
| **Faixa CI** (`CIRange`) | Faixa em que o CI se enquadra (40, 70, 100). |
| **Nova FCI Pendente** (`NewFCIPending`) | Marcado quando o item exige nova FCI a ser enviada à SEFAZ. |

Selecionando uma linha, o subformulário **Detalhes** apresenta a **Linha do Tempo de VI do Item** com cada movimento que compôs o cálculo: saldo inicial, compras, produções, montagens e consumos, com `Tipo de Mov.`, `Data Mov.`, `Nº Doc.`, `Quantidade Mov.`, `Quantidade Acum.`, `VI Acum.`, `Nº Subitem`, `Quantidade Subitem`, `VI Subitem` e `Valor`. Use a ação **Detalhes Subitem** para auditar a composição do VI dos insumos consumidos.

### Passo 3 — Criar o Arquivo FCI e inserir os itens pendentes

1. Pesquise por **Arquivos FCI** (`FCIFilesFNX`).
2. Crie um novo arquivo (a numeração é atribuída automaticamente pela série configurada em **Nº Serie Arquivo FCI**).
3. Informe **Código Filial** (`BranchCode`).
4. Clique em **Inserir Itens Pendentes** (`InsertPendingItems`). O sistema inclui no arquivo todos os itens da filial que estejam marcados como **Nova FCI Pendente**.

O subformulário **Itens** lista cada **FCI do Item** (`ItemFCIFNX`) atribuída ao arquivo: **Nº Item**, **Data Cálculo**, **VI - Valor Parcela Importada**, **VO - Valor Operação Saída Interestadual**, **CI - Conteúdo Importação** e **Nº FCI**.

### Passo 4 — Exportar o arquivo TXT e enviar à SEFAZ

1. Ainda na ficha **Arquivo FCI**, clique em **Exportar Arquivo** (`ExportFile`).
2. O sistema gera o TXT (UTF-8) com o nome `FCI.txt`, contendo os registros padronizados pela SEFAZ:
   - `0000` (abertura), `0001`, `0010` (dados do contribuinte), `0990`, `5001` (abertura do bloco de itens), `5020` (uma linha por item, com descrição, NCM, código do item, unidade, VO, VI e CI), `5990` (totalização), `9001`, `9900` (totais por registro), `9990` e `9999` (encerramento).
3. Faça o upload do arquivo no portal da SEFAZ e aguarde o processamento.
4. Após o processamento, registre o **Nº Protocolo** (`ProtocolNo`) devolvido pela SEFAZ na ficha **Arquivo FCI**.

### Passo 5 — Importar o arquivo de retorno

1. De volta à ficha **Arquivo FCI**, clique em **Importar Arquivo de Retorno** (`ImportReturnFile`).
2. Selecione o arquivo de retorno fornecido pela SEFAZ.
3. O sistema lê os registros `5020` do arquivo, identifica o item pelo código (campo 4) e o **Nº FCI** atribuído (campo 10), e atualiza:
   - O **Nº FCI** e a **Data Importação Arquivo FCI** (`FCIFileImportDateTime`) na **FCI do Item**.
   - O **Nova FCI Pendente** é desmarcado.
   - O **Código Origem ICMS** do **Item** é atualizado conforme o **Código Origem ICMS** definido na **Faixa CI** correspondente, e o **Código Origem Anterior** (`PreviousOriginCode`) é gravado para fins de auditoria.
   - Todas as **Linhas de Pedido de Venda** (`Sales Line`) e **Linhas de Fatura de Venda** abertas para o item são atualizadas com o novo **Código Origem ICMS** e com o **Nº FCI** recém-atribuído.
   - Todas as **FCI do Item** posteriores ao cálculo atualizado também recebem o novo **Nº FCI** e perdem o status de pendente, evitando reenvio desnecessário.

> A partir desse ponto, todas as novas saídas do item passarão a usar o código de origem ICMS recém-atualizado e a referenciar o número da FCI nas NF-e (campo `nFCI`).

---

## 5. Como o Cálculo é Feito

Esta seção explica, em alto nível, a lógica aplicada pela rotina de cálculo (codeunit `FCIMgtFNX`). Compreender o passo a passo é fundamental para diagnosticar valores aparentemente incorretos.

1. **Saldo inicial.** Para cada item que tinha FCI no mês anterior, copia para o mês corrente: `Quantity`, `Amount` (valor base de VI), `VIAmount`, `VOAmount`, `CIPercent`, `CIRange` e `FCINo`. Adicionalmente, lança uma linha de **Tipo de Mov.** = `Balance` na **Linha do Tempo de VI do Item**.

2. **Compras (entrada de insumos).** Para cada linha de documento fiscal de entrada do período:
   - Calcula a **Quantidade base** = `Quantity × QtyPerUnitOfMeasure`.
   - Subtrai a **quantidade já devolvida** (devoluções de compra registradas).
   - Se o documento não é Importação Direta, subtrai do valor da linha o **ICMS creditado** (consultado em `FiscalDocumentTaxEntryFNX` com `PostingTaxCredit = TRUE`).
   - Soma **Frete** e **Seguro** lançados na linha (registros de `FiscalDocumentLineChargeFNX`).
   - Aplica a **VI (%)** da **Faixa CI** associada ao **Código Origem ICMS** do item. Em **Importação Direta**, o valor é integral (100%).
   - Acumula no item: `Quantity += LineQty` e `Amount += LineAmount`.
   - Recalcula `VIAmount = Amount / Quantity` (arredondado a 0,01).

3. **Produção e Montagem.** Para cada par **Output / Consumption** (ou **Assembly Output / Assembly Consumption**) do período, vinculado por `OrderType + OrderNo + SourceNo`:
   - O **insumo consumido** tem seu `Quantity` e `Amount` reduzidos no item (sai do estoque de VI), pelo `VIAmount` corrente do insumo × quantidade consumida.
   - O **produto produzido** recebe acréscimo no `Quantity` (quantidade produzida) e no `Amount` (valor de VI proveniente dos insumos), elevando indiretamente o seu `VIAmount`.
   - Cada movimento é registrado na linha do tempo, com cruzamento subitem/produto, para que a origem do VI seja rastreável.

4. **Cálculo do VO (Valor de Operação de Saída).** Considera **vendas registradas** do período em que a **Situação Cliente/Fornecedor** seja `Same state` ou `Other state`:
   - Soma `LineAmount − ICMSAmount` em `VOOtherStateAmt` ou `VOSameStateAmt`, conforme a situação.
   - Soma a quantidade base em `VOOtherStateQuantity` ou `VOSameStateQuantity`.
   - Quando o item não tem vendas registradas no mês, busca **Pedidos de Venda Liberados** com `Qty. to Ship > 0` e usa esses valores como base estimada, deduzindo o ICMS interestadual cadastrado em **Percentual de Imposto por Território**.
   - **VO unitário** = preferencialmente `VOOtherStateAmt / VOOtherStateQuantity`; caso não haja saída interestadual, `VOSameStateAmt / VOSameStateQuantity`.

5. **CI e Faixa.** `CIPercent = VIAmount / VOAmount × 100`. A **Faixa CI** é o menor `Range` da tabela **Faixas de Conteúdo de Importação** maior ou igual ao `CIPercent`. Esta faixa determina o **Código Origem ICMS** do item após o retorno da SEFAZ.

6. **Detecção de FCI pendente.** O item é marcado como **Nova FCI Pendente** quando:
   - Já existe FCI declarada num cálculo anterior, **e**
   - A **Faixa CI** atual difere da última faixa declarada, **ou**
   - O item ainda não teve nenhuma FCI registrada e é elegível (Sistema de Reabastecimento = `Prod. Order` ou `Assembly`).

---

## 6. Conferência: o que verificar quando a FCI parecer incorreta

Quando os números da FCI não baterem com o esperado, percorra os pontos abaixo na ordem. A maioria dos desvios cai em uma destas categorias.

### 6.1. Configuração

- **Calcular FCI** marcado na **Config. NF-e** da filial em questão.
- **Faixas de Conteúdo de Importação** cadastradas com os percentuais corretos (40, 70, 100) e cada faixa apontando para um **Código Origem ICMS** existente.
- **Código Origem ICMS** dos códigos de origem importados com **Faixa CI** preenchida e, quando aplicável, **Nº FCI Obrigatório** marcado.
- **Nº Serie Arquivo FCI** preenchido na **Configuração Contabilidade** e a série numérica existindo em **Número de Séries**.
- **Informação Filial** (ou **Informação Empresa**, se a empresa não usa filiais) com CNPJ, IE, endereço e UF preenchidos corretamente. Esses dados vão para o registro `0010` do TXT.

### 6.2. Cadastro de Itens

- **Sistema de Reabastecimento** do item é **`Prod. Order`** ou **`Assembly`**. Itens fora dessas categorias não entram no cálculo.
- **Código Origem ICMS** (`OriginCodeFNX`) preenchido nos itens importados ou com componente importado.
- **NCM** preenchido (entra no registro `5020` do TXT).
- **Unidade Base** preenchida (também entra no `5020`).

### 6.3. Movimentação de Compras

- Linhas de documento fiscal de entrada com **Item**, **Quantidade** > 0, **Valor** > 0 e **Código Origem ICMS** preenchido na linha. Sem isso, o documento não compõe VI.
- **Importação Direta** marcada nos documentos de DI. Em compras nacionais com componente importado, a **Faixa CI** do **Código Origem ICMS** precisa ter **VI (%)** > 0; caso contrário, a parcela importada será zero.
- **ICMS creditado** lançado nas entradas nacionais. Quando o crédito não foi registrado, a localização não tem como subtraí-lo do VI, e o conteúdo importado fica superestimado.
- **Frete e Seguro** lançados como acessórios da linha (e não somente do cabeçalho), para que entrem no cálculo.
- **Devoluções de compra** registradas no período. Devoluções não registradas inflam a quantidade e o valor do VI.

### 6.4. Movimentação de Produção/Montagem

- **Movimentos de saída de produção** (`Output`) e **consumo** (`Consumption`), ou **Assembly Output / Assembly Consumption**, vinculados pelo número da ordem. Movimentos órfãos não são considerados.
- **Item produzido** com **Sistema de Reabastecimento** correspondente. Mudanças de Sistema de Reabastecimento entre períodos podem fazer o item desaparecer do cálculo.
- **Insumo consumido** com **Código Origem ICMS** importado. Insumos sem origem informada não transferem VI ao produto.

### 6.5. Movimentação de Vendas

- Documentos fiscais de saída **registrados** no período. Faturas pendentes ainda não registradas não compõem VO de vendas registradas.
- **Situação Cliente/Fornecedor** preenchida (`Same state` ou `Other state`). Vendas marcadas como exterior ou outras situações ficam fora do VO da FCI.
- **Pedidos de Venda Liberados** com **Quantidade a Enviar** > 0 quando o item ainda não tem vendas registradas no mês. Sem pedidos abertos e sem vendas registradas, o item fica sem VO e não tem CI calculado.
- **Percentual de Imposto por Território** cadastrado para a UF de destino, no caso de itens cujo VO foi estimado a partir de pedidos abertos.

### 6.6. Mês Anterior e Continuidade

- Cálculo do mês anterior executado para a mesma filial. Sem ele, o saldo inicial fica zerado, e o VI se torna apenas o do próprio mês.
- Após importar o retorno da SEFAZ, conferir se o **Código Origem ICMS** do item foi atualizado para o esperado e se as **Linhas de Pedido de Venda** e **Linhas de Fatura de Venda** abertas receberam o novo **Nº FCI**.

### 6.7. Auditoria via Linha do Tempo de VI

Sempre que um valor de VI parecer estranho, abra a **Lista de FCIs**, posicione na linha do item e use o subformulário **Detalhes** (e a ação **Detalhes Subitem**). A linha do tempo mostra, em ordem cronológica, cada movimento que compôs o `VIAmount` (saldo, compras, produções, consumos), permitindo localizar o documento ou a ordem responsável pelo desvio.
