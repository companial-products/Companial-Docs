<!-- Fonte: Orientação Técnica CFC nº 01/2026; loc/FiscalSetup/TaxPosting.Table.al, TaxPostingLine.Table.al, TaxPostingAccounts.Table.al; loc/Fiscal/TaxAmountLine.Table.al; loc/Sales/SalesPostHook.Codeunit.al -->
# Contabilização de IBS/CBS

**Data**: Agosto/2026

---

## Sumário

Esta nota técnica descreve como configurar a Localização para o Brasil para contabilizar o IBS e a CBS, tomando como referência a **Orientação Técnica CFC nº 01/2026**, publicada pelo Conselho Federal de Contabilidade. O documento apresenta o modelo contábil dos novos tributos, o mapeamento de cada situação para as configurações de **Contabilização Imposto** e **Contas Contabilização Impostos** já existentes no Business Central, e um conjunto de casos práticos com os lançamentos resultantes.

Uma seção específica trata do **exercício de 2026**, o período de teste operacional em que os fatos geradores ocorrem e as alíquotas se aplicam, mas o recolhimento é dispensado para o contribuinte que cumpre as obrigações acessórias. A OT do CFC não determina se o passivo deve ou não ser reconhecido nesse período: apresenta os argumentos dos dois lados e remete a decisão ao julgamento do profissional de contabilidade. A Localização suporta as duas políticas, e este documento mostra como configurar cada uma.

> **Aviso**: esta nota técnica descreve a configuração do sistema. Ela não substitui a orientação do contador responsável. A escolha da política contábil, em especial quanto ao exercício de 2026, é de julgamento profissional, conforme a própria OT CFC nº 01/2026 estabelece.

---

## 1. Contexto: o que a OT CFC nº 01/2026 estabelece

A Orientação Técnica CFC nº 01/2026 sistematiza o tratamento contábil do IBS e da CBS à luz das Normas Brasileiras de Contabilidade. Ela não tem caráter normativo vinculante. Seu propósito é subsidiar o julgamento profissional. Cinco definições dela orientam a configuração do sistema:

| # | Definição | Base na OT |
|---|---|---|
| 1 | IBS e CBS são cobrados **"por fora"** e não integram a receita. A receita é reconhecida **líquida** dos tributos | itens 5, 6 e 16.2 |
| 2 | O **passivo tributário** é reconhecido **a cada operação de saída**, no momento do fato gerador | item 8 |
| 3 | O **crédito de entrada é um ativo**, classificado em regra no **ativo circulante**, reconhecido no **período de competência da aquisição** | itens 9, 10, 14.2 e 15.1 |
| 4 | Tributo **recuperável não integra o custo do estoque**; tributo **não recuperável integra** o custo ou a despesa | item 12 (NBC TG 16, item 11) |
| 5 | O débito admite **cinco modalidades de extinção**, cada uma com reflexo contábil próprio | itens 18 a 24 |

As cinco modalidades de extinção do item 5 são: compensação com créditos próprios, pagamento direto pelo contribuinte, recolhimento na liquidação financeira (*split payment*), recolhimento pelo adquirente e pagamento por terceiro responsável.

---

## 2. Princípio de projeto: cálculo fiscal e contabilização são independentes

Na Localização para o Brasil, o **cálculo do imposto** e a **contabilização do imposto** são etapas separadas e configuradas em lugares distintos:

- **O cálculo** é governado pelas **Áreas de Impostos (Brasil)**, pelas **Jurisdições de Impostos (Brasil)**, pelas **Alíquotas IBS/CBS** e pelo **Cód. Class. Trib. IBS/CBS** da linha do documento. O resultado alimenta o Documento Fiscal, o XML da NF-e e os livros fiscais.
- **A contabilização** é governada pela **Contabilização Imposto** (`TaxPostingFNX`) e pelas **Contas Contabilização Impostos** (`TaxPostingAccountsFNX`).

Essa separação é o que torna possível calcular, destacar e escriturar o IBS/CBS sem gerar nenhum lançamento contábil, exatamente o que a política de não reconhecimento do exercício de 2026 requer (seção 6).

> **Estado atual do produto:** a função **Inserir Dados da Reforma Tributária**, disponível nas **Jurisdições de Impostos (Brasil)**, cria as jurisdições `CBS`, `IBS-UF` e `IBS-MUN` e três códigos de **Contabilização Imposto**: `IBS-CBS`, `IBS-CBS-VENDA` e `IBS-CBS-COMPRA`. Ela não preenche os campos de comportamento contábil e não cria contas. Isso significa que, logo após a execução da função, o IBS/CBS é calculado e destacado, mas não afeta a contabilidade nem o valor total dos documentos. O preenchimento descrito na seção 3 é o que ativa a contabilização.

---

## 3. As configurações que controlam a contabilização

### 3.1 Contabilização Imposto

**Caminho no BC:** `Lista Contabilizações Imposto` → **Cartão Contabilização Imposto**

Cada código de contabilização define o que acontece contabilmente quando o imposto é lançado. Os campos estão organizados em duas guias: **Faturamento** (lançamento do documento) e **Pagamento/Recebimento** (lançamento na liquidação financeira).

Guia **Faturamento**:

| Campo | O que faz |
|---|---|
| **Custo/Preço Item** | **Adicionar** ou **Subtrair**: inclui o imposto no custo do item (compras) ou no preço (vendas). Deixe **em branco** para tributos por fora recuperáveis |
| **Financeiro** | **Adicionar ao Total** ou **Subtrair do Total**: define se o imposto altera o valor devido pelo cliente ou ao fornecedor |
| **Imposto a Pagar** | Gera o lançamento na conta de passivo tributário |
| **Despesa** | Gera o lançamento na conta de despesa |
| **Crédito Imposto** | Gera o lançamento na conta de ativo (imposto a recuperar) |
| **Estorno de Crédito** | Registra valores de estorno de crédito na conta específica |

Guia **Pagamento/Recebimento**: os mesmos campos **Financeiro**, **Imposto a Pagar**, **Despesa** e **Crédito Imposto**, aplicados no momento da liquidação financeira em vez do faturamento. Hoje esses campos atendem às retenções na fonte. São também o mecanismo previsto para o *split payment* (seção 5.3).

### 3.2 Contas Contabilização Impostos

**Caminho no BC:** ação **Contas Impostos**

Define quais contas serão usadas, por **Classificação Imposto** (`IBS UF`, `IBS Mun`, `CBS`, `IS`):

| Campo | Uso |
|---|---|
| **Classificação Imposto** | O tributo a que a configuração se aplica |
| **Tipo Filtro** / **Cód. Filtro** | Restringe a configuração a uma **Jurisdição**, **Cliente**, **Fornecedor**, **Grupo Contábil Produto** ou **Grupo Contábil Negócio**. Em branco = configuração padrão |
| **Tipo Conta a Pagar** / **N° Conta a Pagar** | Passivo tributário |
| **Tipo Conta a Recuperar** / **N° Conta a Recuperar** | Ativo: crédito do tributo |
| **Tipo Conta Despesa** / **N° Conta Despesa** | Despesa, quando o tributo não é recuperável nem repassado |
| **Tipo Conta Estorno de Crédito** / **Nº Conta Estorno de Crédito** | Contrapartida do estorno de crédito |

A busca da configuração segue uma hierarquia, da mais específica para a mais genérica: **Jurisdição → Fornecedor → Cliente → Grupo Contábil Produto → Grupo Contábil Negócio → configuração padrão** (sem filtro). Se nenhuma for encontrada, o registro do documento é interrompido com erro indicando os filtros testados.

### 3.3 Como o código de contabilização chega ao documento

O vínculo é feito na **Linha de Área de Impostos**, no campo **Cód. Contabilização**. Cada linha da área de impostos associa uma **Cód. Jurisdição Imposto** a um **Cód. Contabilização** e a uma **Ordem Cálculo**. É por esse caminho que a área de impostos aplicada ao documento determina o comportamento contábil de cada tributo.

Como o **Cód. Contabilização** é definido por linha de área de impostos, cenários com tratamento contábil diferente (por exemplo, com e sem direito a crédito) exigem **áreas de impostos distintas**.

### 3.4 Linhas de contabilização adicionais

Além dos lançamentos automáticos acima, a **Linha Contabilização Imposto** permite acrescentar pares livres de lançamento a um código de contabilização, informando **Tipo** (**Débito**/**Crédito**), **Tipo Conta** e **N° Conta**. Use quando a política contábil exigir contas de controle além das quatro contas padrão.

---

## 4. Casos práticos — operações do dia a dia

Os exemplos usam as alíquotas de referência de 2026: **CBS 0,90%**, **IBS UF 0,10%** e **IBS Municipal 0,00%**, num total de **1,00%**.

### Caso 1 — Venda de mercadoria (tributo por fora)

Venda de R$ 10.000,00. CBS R$ 90,00 + IBS UF R$ 10,00.

| Conta | Débito | Crédito |
|---|---:|---:|
| Clientes | 10.100,00 | |
| Receita de Vendas | | 10.000,00 |
| CBS a Recolher | | 90,00 |
| IBS a Recolher | | 10,00 |

**Configuração** do código `IBS-CBS-VENDA`:

| Campo | Valor |
|---|---|
| **Custo/Preço Item** | *(em branco)* |
| **Financeiro** | **Adicionar ao Total** |
| **Imposto a Pagar** | Sim |
| **Despesa** | Não |
| **Crédito Imposto** | Não |

A receita fica líquida dos tributos sem necessidade de conta redutora, atendendo aos itens 5, 6 e 16.2 da OT. O comportamento é o mesmo já usado para o IPI, e o oposto do ICMS, que é calculado "por dentro".

> **Devoluções e notas de crédito**: o sistema inverte automaticamente os lançamentos quando o documento é do tipo Nota de Crédito. Não é necessário um código de contabilização separado.

### Caso 2 — Compra com direito a crédito

Compra de mercadoria por R$ 5.000,00. CBS R$ 45,00 + IBS UF R$ 5,00.

| Conta | Débito | Crédito |
|---|---:|---:|
| Estoques | 5.000,00 | |
| CBS a Recuperar | 45,00 | |
| IBS a Recuperar | 5,00 | |
| Fornecedores | | 5.050,00 |

**Configuração** do código `IBS-CBS-COMPRA`:

| Campo | Valor |
|---|---|
| **Custo/Preço Item** | *(em branco)* |
| **Financeiro** | **Adicionar ao Total** |
| **Imposto a Pagar** | Não |
| **Despesa** | Não |
| **Crédito Imposto** | Sim |

O estoque é registrado líquido dos tributos recuperáveis, conforme o item 12 da OT e o item 11 da NBC TG 16.

> **Erro comum**: deixar **Custo/Preço Item** como **Adicionar** nesse cenário. O estoque fica superavaliado, e o crédito deixa de ser reconhecido como ativo.

### Caso 3 — Compra sem direito a crédito

Mesma compra, em operação com vedação à apropriação do crédito.

| Conta | Débito | Crédito |
|---|---:|---:|
| Estoques (ou Despesa) | 5.050,00 | |
| Fornecedores | | 5.050,00 |

**Configuração**: código de contabilização próprio, vinculado a uma **Área de Impostos** distinta:

| Campo | Valor |
|---|---|
| **Custo/Preço Item** | **Adicionar** |
| **Financeiro** | **Adicionar ao Total** |
| **Imposto a Pagar** | Não |
| **Despesa** | Não |
| **Crédito Imposto** | Não |

O tributo não recuperável integra o custo de aquisição ou a despesa, conforme a parte final do item 12 da OT.

### Caso 4 — Brindes e bonificações

Saída sem contraprestação financeira: não há contas a receber para absorver o tributo, que então transita pelo resultado. Valor da operação R$ 10.000,00, tributos R$ 100,00.

| Conta | Débito | Crédito |
|---|---:|---:|
| Despesa com Brindes — IBS/CBS | 100,00 | |
| IBS/CBS a Recolher | | 100,00 |

**Configuração**: código de contabilização próprio:

| Campo | Valor |
|---|---|
| **Custo/Preço Item** | *(em branco)* |
| **Financeiro** | *(em branco)* |
| **Imposto a Pagar** | Sim |
| **Despesa** | Sim |
| **Crédito Imposto** | Não |

O item 16.2 da OT ressalva justamente esse caso: em regra o IBS/CBS não impacta o resultado, "ressalvadas situações como brindes e determinadas bonificações, em que o tributo pode transitar pelo resultado".

### Caso 5 — Estorno de crédito

Quando a legislação exige o estorno de crédito apropriado anteriormente (grupo `gEstornoCred` da NF-e), o valor é revertido do ativo contra a conta configurada. Estorno de R$ 50,00:

| Conta | Débito | Crédito |
|---|---:|---:|
| Despesa — Estorno de Crédito IBS/CBS | 50,00 | |
| IBS/CBS a Recuperar | | 50,00 |

**Configuração**: marque **Estorno de Crédito** no código de contabilização e informe **Tipo Conta Estorno de Crédito** e **Nº Conta Estorno de Crédito** nas **Contas Contabilização Impostos**. Ao marcar **Estorno de Crédito**, o sistema ajusta automaticamente os demais campos do código de contabilização, pois esse tratamento é exclusivo.

---

## 5. Casos práticos — extinção do débito

### 5.1 Compensação com créditos (apuração do período)

A não cumulatividade se materializa no encontro de contas do período. Débitos de R$ 100,00 e créditos de R$ 45,00:

| Conta | Débito | Crédito |
|---|---:|---:|
| CBS a Recolher | 45,00 | |
| CBS a Recuperar | | 45,00 |

O saldo remanescente de R$ 55,00 permanece em **CBS a Recolher**, no passivo circulante.

Quando os créditos superam os débitos, o excedente permanece no ativo, para uso em períodos subsequentes ou ressarcimento (item 20 da OT). Se a realização for esperada para além de doze meses, a classificação passa ao ativo não circulante (item 16.1). Hoje essa reclassificação é feita manualmente.

### 5.2 Pagamento direto pelo contribuinte

É a modalidade mais simples: baixa do passivo contra as disponibilidades, no momento do recolhimento.

| Conta | Débito | Crédito |
|---|---:|---:|
| IBS/CBS a Recolher | 55,00 | |
| Bancos | | 55,00 |

### 5.3 *Split payment* (previsto no Roadmap)

No *split payment*, a instituição financeira segrega e recolhe o tributo no momento da liquidação. Recebimento da venda do Caso 1 (R$ 10.100,00), com R$ 100,00 retidos:

| Conta | Débito | Crédito |
|---|---:|---:|
| Bancos | 10.000,00 | |
| CBS a Recolher | 90,00 | |
| IBS a Recolher | 10,00 | |
| Clientes | | 10.100,00 |

Três consequências que a OT deixa explícitas (item 22):

- o passivo é reconhecido **integralmente na emissão do documento**. O *split payment* é apenas a forma de liquidação e não altera o reconhecimento inicial (item 22.3);
- o adquirente reconhece o crédito **com base no documento fiscal recebido**, sendo irrelevante o fracionamento do pagamento (item 22.2);
- **não há fluxo de caixa de tributo** a apresentar na Demonstração dos Fluxos de Caixa do vendedor. Apenas o valor líquido transita pelo caixa (item 16.3).

O *split payment* está previsto no Roadmap da Localização para produção gradual a partir de 2027, com impacto principal nos fluxos de CNAB Pagamentos e Recebimentos.

### 5.4 Recolhimento pelo adquirente (previsto no Roadmap)

Quando o adquirente é o responsável pelo recolhimento, o efeito contábil para o fornecedor é idêntico ao do *split payment* (item 23 da OT): baixa do passivo pelo valor recolhido pelo adquirente e ingresso apenas do valor líquido nas disponibilidades.

### 5.5 Pagamento por terceiro responsável

Quando a lei atribui a responsabilidade a terceiro distinto do adquirente, o item 24 da OT orienta, por analogia aos itens 55 e 56 da NBC TG 25, que o contribuinte originário mantém o passivo reconhecido enquanto a obrigação subsistir. O ativo de reembolso só é registrado quando o recebimento futuro for praticamente certo. Não há automação específica no sistema para essa modalidade.

---

## 6. O exercício de 2026 — período de teste operacional

Em 2026 ocorrem os fatos geradores e aplicam-se as alíquotas legais, mas o recolhimento é dispensado para o contribuinte que cumprir as obrigações acessórias. O PIS/Pasep e a Cofins permanecem integralmente exigíveis.

O Capítulo IV da OT registra que a natureza jurídica dessa dispensa é controvertida, apresenta os argumentos favoráveis e contrários ao reconhecimento do passivo, e não adota posição prescritiva. A escolha cabe ao profissional de contabilidade.

| Posição | Argumentos centrais (OT) |
|---|---|
| **Reconhecer** | O fato gerador ocorreu; a dispensa é **condicionada** ao cumprimento das obrigações acessórias, atuando como condição resolutiva; o regime de competência independe do desembolso; o reconhecimento produz informação mais completa sobre a situação tributária (itens 28.1 a 28.4) |
| **Não reconhecer** | Para o contribuinte adimplente não há obrigação presente exigível; a essência econômica não comporta passivo sem desembolso; a entidade tem capacidade prática de evitar a saída; a saída é improvável, afastando os critérios da NBC TG 25 (itens 29.1 a 29.7) |

### Como configurar cada política

**Política de não reconhecimento**: o IBS/CBS é calculado, destacado no documento fiscal e no XML da NF-e e escriturado nos livros fiscais, sem qualquer lançamento contábil:

| Campo do código de contabilização | Valor |
|---|---|
| **Custo/Preço Item** | *(em branco)* |
| **Financeiro** | *(em branco)* |
| **Imposto a Pagar** | Não |
| **Despesa** | Não |
| **Crédito Imposto** | Não |

Este é o comportamento resultante da execução da função **Inserir Dados da Reforma Tributária** sem configuração adicional. Ainda assim, recomenda-se registrar a escolha explicitamente, para que a política contábil da empresa esteja documentada e não pareça uma configuração pendente.

**Política de reconhecimento**: configure os códigos de contabilização conforme os Casos 1 a 5 da seção 4, com as contas correspondentes.

### Obrigações que independem da política adotada

O item 30 da OT orienta o profissional a, em qualquer das duas posições:

- **avaliar individualmente** as circunstâncias da entidade, considerando o grau de conformidade com as obrigações acessórias;
- **documentar o julgamento** exercido em papéis de trabalho;
- **divulgar em nota explicativa** a política adotada, a natureza do período de teste e os valores envolvidos, ainda que não reconhecidos;
- **manter controle interno adequado** dos valores apurados de IBS e CBS em 2026, com rastreabilidade das obrigações acessórias cumpridas.

Os dois últimos pontos são atendidos pela base fiscal da Localização independentemente da política contábil. Os valores de IBS/CBS calculados ficam registrados nos movimentos de imposto do Documento Fiscal e nos livros fiscais, disponíveis para consulta e extração, mesmo quando não há lançamento contábil correspondente.

---

## 7. O que ainda não é automático

Os pontos abaixo decorrem da OT e não possuem automação no produto no momento. Foram levantados na análise da Orientação Técnica e estão em avaliação:

| Tema | Situação |
|---|---|
| **Conta de "IBS/CBS a apropriar"**: os itens 11.1 e 15.1 da OT admitem reconhecer o crédito na competência da aquisição em **conta segregada**, reclassificando-o para "a recuperar" quando a condição legal se implementa (extinção do débito pelo fornecedor). Hoje há uma única conta de crédito por classificação de imposto | Sem automação |
| **Reclassificação do saldo credor** entre ativo circulante e não circulante conforme o prazo esperado de realização (item 16.1) | Manual |
| **Ajuste a valor recuperável** do crédito (item 11.2) | Manual |
| **IBS/CBS sobre adiantamentos**: o item 8 da OT afirma que o passivo nasce inclusive no caso de adiantamento de recursos financeiros | Sem tratamento específico |
| **Contabilização da apuração** de IBS/CBS (encontro de contas do período) | Lançamentos manuais; automação em avaliação junto à Apuração Assistida |
| ***Split payment*** e **recolhimento pelo adquirente** | Previstos no Roadmap da Localização |

---

## 8. Referências

- **Orientação Técnica CFC nº 01/2026**: Aspectos contábeis relacionados ao IBS, à CBS e ao período de teste operacional do exercício de 2026. Conselho Federal de Contabilidade.
- **Lei Complementar nº 214, de 16 de janeiro de 2025**, alterada pela **Lei Complementar nº 227, de 2026**: institui a CBS e o IBS.
- **Decreto nº 12.955, de 29 de abril de 2026**: regulamenta a CBS.
- **NBC TG Estrutura Conceitual**: definições de ativo e passivo (itens 4.14, 4.36, 4.37, 4.43, 4.45 e 4.68).
- **NBC TG 16 (R2)**: Estoques (item 11: tributos recuperáveis não integram o custo).
- **NBC TG 25 (R2)**: Provisões, Passivos Contingentes e Ativos Contingentes.
- **NBC TG 26 (R5)**: Apresentação das Demonstrações Contábeis (item 66: circulante e não circulante).
- **NBC TG 47**: Receita de Contrato com Cliente (item 47: exclusão dos montantes cobrados em nome de terceiros).
- **NBC TG 03 (R3)**: Demonstração dos Fluxos de Caixa.

### Documentos relacionados neste portal

- [Reforma Tributária - Configurações](Localization-for-Brazil_User-Manual_Reforma-Tributaria-Configuracoes.md): alíquotas, classificações tributárias e ativação da reforma
- [Reforma Tributária - NF-e: Configuração e Emissão](Localization-for-Brazil_User-Manual_NFe-Reforma-Tributaria.md)
- [Jurisdições, Áreas e Contabilizações](Localization-for-Brazil_User-Manual_guia-fiscal-jurisdicoes-areas-impostos.md): fundamentos do motor de contabilização de impostos
