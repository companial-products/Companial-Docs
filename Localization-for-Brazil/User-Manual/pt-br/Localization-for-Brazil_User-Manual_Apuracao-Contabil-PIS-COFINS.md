<!-- Branch: master -->
# Apuração Contábil de PIS/COFINS

**Data**: Maio/2026

---

## Sumário

A apuração contábil de PIS/COFINS é o processo, dentro do app **Livros Fiscais** (Tax Books), que consolida débitos (saídas), créditos (entradas), créditos sobre ativo fixo, ajustes, retenções e per/dcomp em uma **Declaração de Imposto** (`TaxStatementFNX`), que pode ser revisada, aprovada e registrada na contabilidade. Este documento descreve, passo a passo, todas as configurações necessárias e o fluxo completo, desde a criação da apuração até o registro dos lançamentos contábeis.

## 1. Contexto

A apuração de PIS/COFINS é normalmente feita em duas etapas:

1. A **Apuração de Impostos** (`TaxSettlementFNX`) concentra os documentos do período, calcula bases e impostos, gera os registros do EFD-Contribuições (F100, F120, F130, F600, 1100, 1500 etc.) e produz dois números de **Declaração de Imposto**: um para PIS e outro para COFINS.
2. A **Declaração de Imposto** (`TaxStatementFNX`) apresenta o demonstrativo final (débitos − créditos − retenções − saldo credor anterior − per/dcomp = imposto a recolher), passa por liberação ou aprovação opcional e, ao ser registrada, gera os lançamentos contábeis de débito do imposto a recolher contra a conta de imposto a pagar (e eventuais compensações de créditos e retenções).

A apuração suporta:

- **Regime Não Cumulativo** (Lucro Real): apura débitos e créditos.
- **Regime Cumulativo** (Lucro Presumido): apura apenas débitos. Dois critérios de escrituração: **Regime de Caixa** (a apuração calcula sobre os recebimentos) ou **Regime de Competência - Detalhado**.
- **Compensação automática** com saldo credor de período anterior, retenções na fonte (registro F600) e créditos contábeis específicos.
- **Per/Dcomp** como dedução do imposto devido.
- **Crédito sobre depreciação de ativo fixo** (registro F120) e **crédito sobre aquisição de ativo fixo** (registro F130, parcelado em até 48 meses).
- **Receitas financeiras** no regime cumulativo de caixa (registro F100).
- **Reoneração de PIS/COFINS** (LC 224/2025), opcional, configurada por matriz de CST.

### Quando se aplica

- Empresas com obrigação acessória do EFD-Contribuições.
- Apuração mensal, por filial (ou consolidada via matriz, conforme o cenário fiscal).

---

## 2. Pré-requisitos

Antes de criar a primeira apuração, garanta que os pontos abaixo estão configurados. Os tópicos seguintes detalham cada um.

| Item | Onde |
| --- | --- |
| App **Localization for Brazil** instalado e configurado | Documento de instalação geral |
| App **Tax Books** (Livros Fiscais) publicado | Documento de instalação geral |
| **Informação Filial** com **CNPJ** preenchido | Página `BranchInformationFNX` |
| **CST PIS/COFINS** corretamente atribuído nos itens, matrizes e jurisdições | Apps **Localization** e **Tax Books** |
| **Áreas de Impostos / Jurisdição Imposto** com alíquotas de PIS/COFINS | App **Localization** |
| Documentos fiscais do período registrados | Movimentação normal |
| **Configuração EFD Contribuições** | Página **Configuração EFD Contribuições** (`EFDContribSetupFNX`) |
| Pelo menos um **Modelo de Declaração de Imposto** para PIS e outro para COFINS | Página **Modelos de Declaração de Impostos** (`TaxStatementTemplatesFNX`) |
| Plano de Contas com contas para **imposto a recolher**, **créditos** e **retenções** | Plano de Contas |

---

## 3. Configuração EFD Contribuições

Esta é a configuração central. Ela define o regime tributário, mapeia as contas usadas na apuração e referencia os modelos de declaração.

### Passo 1: Acessar a configuração

1. Pesquise por **Configuração EFD Contribuições** (`EFDContribSetupFNX`) na barra de busca do BC.
2. Na primeira abertura, o BC cria automaticamente o registro vazio.

### Passo 2: Preencher a seção **Geral**

| Campo | O que informar |
| --- | --- |
| **Indicador Natureza Pessoa Jurídica** (`PresentationProfile`) | Perfil de apresentação no EFD-Contribuições (PJ em geral, cooperativa, SCP etc.) |
| **Tipo Atividade** (`ActivityType`) | **0-Industrial ou equiparado a industrial**, **1-Prestador Serviço**, **2-Atividade de Comércio**, **3-Atividade Financeira**, **4-Atividade Imobiliária**, **9-Outros** |
| **Incidência Tributária** (`TaxIncidenceCode`) | **Não-Cumulativo**, **Cumulativo** ou **Não-Cumulativo e Cumulativo** |
| **Critério de Escrituração do Regime Cumulativo** (`CRegimeBookkeepingType`) | Habilitado apenas no regime cumulativo. **Regime de Caixa** ou **Regime de Competência - Detalhado** |
| **Tipo de Receita do Regime Cumulativo** (`CRegimeIncomeType`) | Tipo de origem da receita no regime cumulativo |

### Passo 3: Preencher a seção **Registros EFD**

| Campo | Descrição |
| --- | --- |
| **Incluir Conta/Centro Custo** (`IncludeAccountCostCenter`) | Incluir o código da conta contábil e o centro de custo nas linhas geradas |
| **Cód. Dimensão Centro Custo** (`CostCenterDimensionCode`) | Dimensão usada como centro de custo |
| **Criar F100 de Frete/Seguro/Outras Despesas** (`EFDCreateF100FromFIOE`) | Quando marcado, gera registros F100 a partir de frete, seguro e outras despesas acessórias |
| **Criar 0035 (Identificação SCP)** (`EFDCreate0035`) | Disponível apenas para perfis de SCP |

### Passo 4: Preencher a seção **Declarações** → **Não-Cumulativo**

| Campo | Descrição |
| --- | --- |
| **Cód. Modelo Declaração PIS Não-Cumulativo** (`NCPISStatmntTemplCode`) | Modelo de declaração que será usado para a apuração de PIS. Veja a [seção 4](#4-configuração-dos-modelos-de-declaração-de-imposto) |
| **Cód. Modelo Declaração COFINS Não-Cumulativo** (`NCCOFINSStatmntTemplCode`) | Modelo de declaração que será usado para a apuração de COFINS |

### Passo 5: Preencher a seção **Compensações** → **Filtros de Contas**

São filtros do plano de contas que dizem ao sistema quais contas representam crédito de PIS/COFINS recuperável e quais representam retenções na fonte recuperáveis. O sistema soma o saldo dessas contas até a data da apuração e usa esse saldo para abater o imposto devido, gerando lançamentos automáticos na declaração.

| Campo | Descrição |
| --- | --- |
| **Créditos PIS** (`PISCreditGLAcctFilter`) | Filtro de contas de crédito de PIS (ex.: `21300|21305`). Aceita o operador `\|` para múltiplas contas |
| **Retenções de PIS** (`PISRetCompGLAcctFilter`) | Filtro de contas de PIS retido na fonte a recuperar |
| **Créditos COFINS** (`COFINSCreditGLAcctFilter`) | Filtro de contas de crédito de COFINS |
| **Retenções de COFINS** (`COFRetCompGLAcctFilter`) | Filtro de contas de COFINS retido na fonte a recuperar |

### Passo 6: Preencher a seção **Compensações** → **Descrições de Lançamentos Personalizadas**

Estes campos são opcionais. Quando preenchidos, substituem a descrição padrão dos lançamentos de compensação:

| Campo | Descrição |
| --- | --- |
| **Compensação Créditos** (`CreditCompCustomDescription`) | Descrição personalizada para o lançamento de compensação de créditos. Aceita variáveis `%1` (Classificação Imposto), `%2` (Nº Declaração), `%3` (Data Inicial), `%4` (Data Final) |
| **Compensação Retenções** (`RetCompCustomDescription`) | Descrição personalizada para o lançamento de compensação de retenções |

### Passo 7: Preencher a seção **Compensações** → **Per/Dcomp**

| Campo | Descrição |
| --- | --- |
| **Nº Conta Crédito Per/Dcomp PIS** (`PerDcompCreditPISAccountNo`) | Conta contábil de contrapartida para os lançamentos de Per/Dcomp PIS |
| **Nº Conta Crédito Per/Dcomp COFINS** (`PerDcompCreditCOFINSAccountNo`) | Conta contábil de contrapartida para os lançamentos de Per/Dcomp COFINS |

### Passo 8: Preencher a seção **Reoneração PIS/COFINS** (opcional)

| Campo | Descrição |
| --- | --- |
| **Habilitar Reoneração de PIS/COFINS** (`EnableReoneration`) | Ativa o cálculo automático dos ajustes de reoneração (LC 224/2025) durante o processamento da apuração |

Ao habilitar, mantenha a **Matriz de Reoneração** (ação **Matriz de Reoneração** no topo da página) com as alíquotas e datas de vigência por CST. Veja o documento dedicado: [EFD Contribuições: Reoneração PIS/COFINS (LC 224/2025)](Localization-for-Brazil_User-Manual_PIS-COFINS-Reoneracao-CST-06.md).

---

## 4. Configuração dos Modelos de Declaração de Imposto

A **Declaração de Imposto** é gerada a partir de um **Modelo de Declaração de Imposto**, que define a numeração, as contas contábeis de débito e crédito do lançamento principal e a conta de ajuste. É necessário criar **um modelo para PIS** e **um modelo para COFINS**.

### Passo 1: Acessar a página

1. Pesquise por **Modelos de Declaração de Impostos** (`TaxStatementTemplatesFNX`).

### Passo 2: Criar o modelo PIS

| Campo | Valor |
| --- | --- |
| **Código** (`Code`) | Ex.: `PIS` |
| **Classificação Imposto** (`TaxIdentification`) | **PIS** |
| **Descrição** (`Description`) | Ex.: "Apuração PIS" |
| **Cód. Jurisdição Guia Pagamento** (`PaymentBillJurisdictionCode`) | Jurisdição usada para gerar a Guia de Pagamento de Imposto (DARF). Apenas jurisdições com **Tipo Guia Pagamento** preenchido aparecem na lista |
| **Nº Série Declaração** (`StatementNoSeries`) | Nº série usado para numerar as Declarações de Imposto desse modelo |
| **Modelo de Descrição do Registro** (`PostingDescriptionTemplate`) | Modelo da descrição usada nos lançamentos contábeis. Padrão: `REF. %1 Nº %2 - %3..%4` (`%1` = imposto, `%2` = nº declaração, `%3` = início, `%4` = fim) |
| **Tipo de Conta Débito** (`DebitAccountType`) | **G/L Account** ou **Vendor**. Em geral, **G/L Account** |
| **Nº Conta Débito** (`DebitAccountNo`) | Conta de despesa/débito do imposto a recolher |
| **Tipo de Conta Crédito** (`CreditAccountType`) | **G/L Account** ou **Vendor**. Para apuração contra fornecedor "Receita Federal", use **Vendor** |
| **Nº Conta Crédito** (`CreditAccountNo`) | Conta de imposto a recolher (passivo) |
| **Nº Conta Ajuste** (`AdjustmentAccountNo`) | Conta para lançamento de ajustes opcionais. Só usada quando há valor de ajuste |

### Passo 3: Criar o modelo COFINS

Repita o **Passo 2** para COFINS, alterando o **Código** (ex.: `COFINS`), a **Classificação Imposto** (**COFINS**) e as contas contábeis correspondentes.

### Passo 4: Vincular os modelos à configuração

Volte à página **Configuração EFD Contribuições** e preencha **Cód. Modelo Declaração PIS Não-Cumulativo** e **Cód. Modelo Declaração COFINS Não-Cumulativo** com os códigos criados.

---

## 5. Configuração de Crédito de Ativo Fixo (opcional)

Empresas no regime não cumulativo podem aproveitar crédito de PIS/COFINS sobre depreciação e sobre a aquisição de ativos fixos. As tabelas abaixo só são necessárias se a empresa quer aproveitar esse crédito.

### Crédito sobre depreciação (registro F120)

A configuração base fica em **Conf. Crédito Depreciação AF** (`FADepreciationCreditSetupFNX`, do app **Localization**). O app **Tax Books** estende essa tabela com os campos específicos do EFD-Contribuições:

| Campo (extensão) | Descrição |
| --- | --- |
| **Cód. Base Cálculo Crédito** (`CalcBaseCreditCodeFNX`) | **09** (Crédito com Base nos Encargos de Depreciação) ou **11** (Crédito com Base nos Encargos de Amortização) |
| **Identificação dos Bens** (`AssetsIdentificationFNX`) | Edificações e Benfeitorias em Imóveis Próprios, Edificações e Benfeitorias em Imóveis de Terceiros, Instalações, Máquinas, Equipamentos, Veículos, Outros |
| **Tipo de Utilização** (`TypeOfUseFNX`) | Produção de Bens Destinados a Venda, Prestação de Serviços, Locação a Terceiros, Outros |
| **CST PIS** (`PISCSTCodeFNX`) | CST do crédito de PIS |
| **CST COFINS** (`COFINSCSTCodeFNX`) | CST do crédito de COFINS |

No cadastro do **Ativo Fixo**, preencha o campo **Código de Crédito de Depreciação** (`DepreciationCreditCodeFNX`) apontando para o código configurado.

### Crédito sobre aquisição (registro F130)

A configuração fica em **Conf. Crédito Aquisição AF** (`FAAcquisitionCreditSetupTaxFNX`):

| Campo | Descrição |
| --- | --- |
| **Código** (`Code`) | Código identificador |
| **Descrição** (`Description`) | Descrição |
| **CST PIS** / **CST COFINS** | CSTs do crédito |
| **% Crédito PIS** (`PISCreditPercent`) / **% Crédito COFINS** (`COFINSCreditPercent`) | Alíquotas de crédito |
| **Identificação dos Bens** (`AssetsIdentification`) | Mesma lógica do registro F120 |
| **Origem do Crédito** (`CreditSource`) | Aquisição no Mercado Interno ou Externo |
| **Tipo de Utilização** (`TypeOfUse`) | Mesma lógica do registro F120 |
| **Parcelas a Apropriar** (`InstallmentsToAppropriate`) | Integral, 6, 12, 24 ou 48 meses (parcelamento do crédito) |

No cadastro do **Ativo Fixo**, preencha **Código de Crédito de Aquisição** (`AcquisitionCreditCodeFNX`) apontando para o código.

A cada apuração, o sistema calcula automaticamente quantas parcelas já foram apropriadas e gera o crédito proporcional do mês, até completar o número de parcelas configurado.

---

## 6. Configuração do Fluxo de Aprovação (opcional)

A Declaração de Imposto pode passar por um fluxo de aprovação antes do registro contábil. A configuração é feita em **Conf. Aprovação Declaração de Impostos** (`TaxStatementApprovalSetupFNX`):

1. Pesquise por **Conf. Aprovação Declaração de Impostos**.
2. Crie uma linha por **Modelo de Declaração de Imposto** que deve ter aprovação:
   - **Cód. Modelo Declaração de Impostos** (`TaxStatementTemplateCode`): modelo PIS, COFINS etc.
   - **Cód. Regra de Fluxo de Aprovação** (`ApprovalFlowRuleCode`): regra do fluxo de aprovação aplicável.
3. Use a ação **Observações de Aprovação** para registrar observações padrão usadas no fluxo.

Quando configurado, a declaração fica em **Pendente Aprovação** após a ação **Enviar Requisição Aprovação**, e só pode ser registrada após **Liberada**.

> **Nota:** se a aprovação não for configurada para o modelo, a declaração segue o fluxo simplificado (Aberta → Liberada → Registrada).

---

## 7. Criar a Apuração de Impostos

Toda a apuração de PIS/COFINS gira em torno do registro **Apuração de Impostos** (`TaxSettlementFNX`). Pode haver uma apuração por filial e por mês.

### Passo 1: Abrir a lista de apurações

1. Pesquise por **Apurações de Impostos** (`TaxSettlementsFNX`).

### Passo 2: Criar nova apuração

1. Clique em **+ Novo**.
2. Preencha os campos principais:
   - **Nº**: número sequencial (manual ou via série).
   - **Cód. Filial** (`BranchCode`): filial da apuração.
   - **Data Inicial** (`StartDate`): primeiro dia do período (geralmente o primeiro dia do mês).
   - **Data Final** (`EndDate`): último dia do período (último dia do mês).

> **Nota:** há um relatório auxiliar **Criar Apurações Imposto** (`CreateTaxSettlementsFNX`) que gera as apurações em lote para todas as filiais de um período.

### Passo 3: Abrir o cartão de apuração PIS/COFINS

A partir da Apuração de Impostos, clique na ação que abre o cartão **Apuração PIS/COFINS** (`PISCOFINSSettlementFNX`). Esse é o cartão dedicado a tudo que é PIS/COFINS:

- Resumo de débitos, créditos e estornos.
- Subpáginas com os registros do EFD: **Controle Créditos PIS (1100)**, **Controle Créditos COFINS (1500)**, **F100**, **F120**, **F130**, **F600**, **F700**, **1300**, **1700**.
- Bloco **Receita Bruta** (**Não Cumulativa Trib.**, **Não Cumulativa Não Trib.**, **Não Cumulativa Exportação**, **Cumulativa**, **Receita Total**).
- Per/Dcomp PIS e Per/Dcomp COFINS (drill-down até **Apurações de Impostos** com filtro de tipo PER/DCOMP).

---

## 8. Processar a Apuração PIS/COFINS

### Passo 1: Executar o cálculo

1. No cartão **Apuração PIS/COFINS**, clique em **Processar** (ícone livro fiscal).

O processamento executa, nesta ordem:

1. **Limpa os registros gerados pelo sistema** das tabelas F100, F120, F130 do período (mantém os registros inseridos manualmente).
2. **Processa os documentos do período** (entradas e saídas) e gera as linhas de **Linhas de Documento PIS/COFINS** (`PISCOFINSDocumentLinesFNX`), usadas como base do demonstrativo.
3. Se o regime for **Cumulativo / Caixa**, calcula sobre os recebimentos do período.
4. **Gera registros F100 a partir de Frete/Seguro/Outras Despesas** (se a flag estiver ligada na configuração).
5. **Gera registros F120** (crédito sobre depreciação) com base nos lançamentos de **Movimentos de Crédito AF** (`FixedAssetCreditEntriesFNX`).
6. **Gera registros F130** (crédito sobre aquisição), calculando a parcela do mês conforme o número de meses configurado em **Parcelas a Apropriar**.
7. **Gera as linhas de Receita Financeira** (registro F100, no regime cumulativo de caixa).
8. **Atualiza o saldo credor anterior** dos registros 1100 (PIS) e 1500 (COFINS) com base no saldo das contas configuradas em **Filtros de Contas**.
9. **Calcula a reoneração** (se habilitada).
10. Em seguida, ainda na mesma ação, o sistema chama o `PISCOFINSSettlementFNX`, que monta as **Declarações de Imposto** PIS e COFINS:
    - Cria a declaração PIS (a partir do **Cód. Modelo Declaração PIS Não-Cumulativo**) e a COFINS (a partir do **Cód. Modelo Declaração COFINS Não-Cumulativo**), caso ainda não existam.
    - Calcula débitos, créditos, ajustes, retenções, saldo credor anterior e per/dcomp.
    - Preenche as linhas do detalhamento da declaração (visualizadas em [Revisar a Declaração](#9-revisar-a-declaração)).
    - Cria os lançamentos pendentes (`TaxStatementEntryFNX`) que efetivarão a contabilização.

### Passo 2: Conferir as linhas de documento

1. Após o processamento, clique em **Linhas dos Documentos** para abrir **Linhas de Documento PIS/COFINS** (`PISCOFINSDocumentLinesFNX`).
2. Confira: documento, série, participante, CFOP, CST, base e valor do PIS/COFINS por linha.
3. Use a ação **Navegar** para investigar a origem de uma linha específica.

### Passo 3: Sugerir Valores de Retenções (opcional)

A ação **Sugerir Valores Retenções** (re)cria os registros F600, 1300 e 1700 a partir dos pagamentos com PIS/COFINS retidos. Use quando houver divergência ou for preciso atualizar após acertos. **Atenção:** ela sobrepõe os registros existentes desses blocos.

### Passo 4: Reprocessar quando necessário

Se você corrigir documentos do período, volte ao cartão e execute **Processar** novamente para recalcular tudo. O sistema limpa os registros gerados antes de regerar. Em geral, reprocesse a apuração depois de qualquer alteração que afete bases ou impostos do período.

---

## 9. Revisar a Declaração

Cada apuração resulta em **duas declarações**: uma para PIS e outra para COFINS. Os números das declarações são exibidos no cartão **Apuração PIS/COFINS**, nos campos **Nº Declaração** das seções **Apuração PIS** e **Apuração COFINS**, com **Status Declaração** ao lado.

### Abrir uma declaração

1. No cartão **Apuração PIS/COFINS**, dê drill-down em **Nº Declaração** (PIS ou COFINS), ou clique em **Declaração PIS** / **Declaração COFINS** na barra de ações.
2. A página **Declaração de Imposto** (`TaxStatementFNX`) abre com:
   - Bloco **Geral**: Nº, Classificação (PIS/COFINS), Data de Início, Data Final, Status.
   - Bloco de detalhes (`TaxStatementDetailsFNX`), com o demonstrativo na ordem:

     1. **Base de Cálculo**
     2. **Débitos do Imposto** (Total de Débitos, Acréscimos de Contribuição, Reduções de Contribuição) → **Subtotal**
     3. **Créditos do Imposto** (Total de Créditos, Acréscimos de Créditos, Reduções de Créditos, Retenções na Fonte, Saldo Credor Anterior) → **Subtotal**
     4. **Apuração do Saldo**: **Saldo Devedor**, **(-) Per/Dcomp** (quando houver) → **Imposto a Recolher** ou **Saldo Credor**

### Validar os lançamentos pendentes

1. Na barra de ações, clique em **Movimentos** para abrir **Movimentos de Declaração de Imposto** (`TaxStatementEntriesFNX`).
2. Cada linha representa um futuro lançamento contábil:
   - **Imposto a Recolher**: debita a conta de despesa/débito do modelo, credita a conta de imposto a recolher (ou fornecedor "Receita Federal").
   - **Compensações de Retenções**: uma linha por conta listada em **Filtros de Contas — Retenções**, até zerar o débito.
   - **Compensações de Créditos**: idem para créditos.
   - **Per/Dcomp**: debita a conta de crédito do modelo, credita a conta configurada em **Nº Conta Crédito Per/Dcomp PIS/COFINS**.

### Visualizar contabilização sem registrar

Use a ação **Verificar Contabilização** para abrir a tela padrão do BC com os lançamentos contábeis simulados. É útil para conferir contas, valores e dimensões antes de efetivamente registrar.

### Imprimir a declaração

A ação **Imprimir** dispara o relatório `PISCOFINSStatementFNX`, que produz o demonstrativo em PDF ou Excel para arquivo ou conferência.

### Dimensões

Use a ação **Dimensões** para definir ou alterar o conjunto de dimensões aplicado a todos os lançamentos pendentes. Pelo padrão da plataforma, a apuração já recebe a dimensão da filial (configurada em **Informação Filial**).

---

## 10. Liberar e Aprovar (se aplicável)

A declaração nasce com status **Aberta** (`Open`).

### Sem aprovação configurada

1. Clique em **Liberar** (atalho `Ctrl+F9`).
2. O status passa para **Liberada** (`Released`).
3. Para voltar a editar, use **Reabrir**.

### Com aprovação configurada (ver [seção 6](#6-configuração-do-fluxo-de-aprovação-opcional))

1. Clique em **Enviar Requisição Aprovação**.
2. O status passa para **Pendente Aprovação** (`PendingApproval`).
3. O fluxo de aprovação cria os registros em **Aprovações** e notifica os aprovadores.
4. Cancelar é possível via **Cancelar Requisição Aprovação**.
5. Após o último aprovador, o status passa para **Liberada**.

> **Nota:** com aprovação habilitada, o botão **Liberar** fica desabilitado. A liberação acontece pelo fluxo.

---

## 11. Registrar os Lançamentos Contábeis

Com a declaração **Liberada**, o registro contábil pode ser feito.

### Passo 1: Registrar

1. Na **Declaração de Imposto**, clique em **Registrar** (atalho `F9`).
2. Confirme a mensagem "Tem certeza que deseja registrar a declaração XXX?".
3. O sistema percorre cada linha de **Movimento de Declaração de Imposto** (`TaxStatementEntryFNX`) e cria um lançamento de diário geral com:
   - **Data Registro** = data final da declaração (`EndingDate`).
   - **Nº Documento** = número da declaração.
   - **Descrição** = descrição da linha (template configurado).
   - **Tipo de Conta / Tipo de Conta Contrapartida** = conforme a linha.
   - **Dimensões** = dimension set ID da declaração (incluindo a dimensão da filial).
   - **Cód. Jurisdição** = jurisdição da Guia de Pagamento (quando aplicável). Quando preenchido, o BC calcula a **Data Vencimento** com base na configuração de `TaxDueDateCalcFNX`.
4. Quando o lançamento é gravado:
   - Cada linha é marcada como registrada (`Posted = true`).
   - O status da declaração passa para **Registrada** (`Posted`).

### Passo 2: Conferir os lançamentos

1. Na barra de ações, use **Navegar** ou **Movimentos** para inspecionar:
   - **Movimentos de Declaração de Imposto**: todas as linhas registradas e estornadas.
   - **Movimentos Contábeis**: através da ação **Navegar** padrão do BC, pelo número da declaração.

### Passo 3 (opcional): Gerar a Guia de Pagamento

Se o **Cód. Jurisdição Guia Pagamento** estiver preenchido no modelo, o lançamento já carrega a jurisdição correta e o cálculo do vencimento. A geração da DARF ou Guia em si é feita pelo módulo de **Guias de Pagamento de Imposto** (apuração separada via `TaxBillsSettlementFNX`).

---

## 12. Estornar a Apuração

Após registrar, é possível estornar a declaração para corrigir e refazer.

1. Na **Declaração de Imposto** (status **Registrada**), clique em **Estornar**.
2. Confirme a mensagem "Tem certeza que deseja estornar a declaração XXX?".
3. O sistema:
   - Cria lançamentos de estorno (com sinal trocado) e descrição prefixada por "Estorno: …".
   - Marca os movimentos originais como estornados (`Reversed = true`).
   - Cria cópias dos movimentos para serem registrados novamente.
   - Muda o status para **Estornada** (`Reversed`).
4. A partir do status **Estornada**, é possível **Reabrir** a declaração para edição manual, ou voltar à Apuração de Impostos, ajustar os documentos e **Processar** novamente.

> **Nota:** a reversão é feita exclusivamente pelo botão **Estornar**. Não tente apagar movimentos contábeis manualmente. Isso quebra a rastreabilidade da declaração.

---

## 13. Resumo de Status

A **Declaração de Imposto** segue o ciclo abaixo. O botão **Registrar** só fica habilitado nos status que permitem registro.

| Status (`TaxStatementStatusFNX`) | Significado | Ações habilitadas |
| --- | --- | --- |
| **Aberta** (`Open`) | Recém-criada / após reabertura. Detalhes editáveis | **Liberar**, **Enviar Requisição Aprovação**, reprocessar (via apuração), **Verificar Contabilização** |
| **Pendente Aprovação** (`PendingApproval`) | Aguardando aprovador | **Cancelar Requisição Aprovação** |
| **Liberada** (`Released`) | Pronta para registrar | **Reabrir**, **Registrar**, **Verificar Contabilização** |
| **Registrada** (`Posted`) | Lançamentos contábeis efetivados | **Estornar** |
| **Estornada** (`Reversed`) | Estorno gerado, com cópias dos movimentos pendentes | **Reabrir**, **Registrar** (novamente) |

---

## Referências

- Página **Apurações de Impostos** (`TaxSettlementsFNX`): lista mestra das apurações.
- Página **Apuração PIS/COFINS** (`PISCOFINSSettlementFNX`): cartão dedicado da apuração.
- Página **Configuração EFD Contribuições** (`EFDContribSetupFNX`): configuração central.
- Página **Modelos de Declaração de Impostos** (`TaxStatementTemplatesFNX`): modelos PIS/COFINS.
- Página **Declaração de Imposto** (`TaxStatementFNX`): cartão da declaração.
- Página **Linhas de Documento PIS/COFINS** (`PISCOFINSDocumentLinesFNX`): base de cálculo detalhada.
- Página **Conf. Aprovação Declaração de Impostos** (`TaxStatementApprovalSetupFNX`): fluxo de aprovação.
- Documento relacionado: [PIS/COFINS — Reoneração CST 06 (LC 224/2025)](Localization-for-Brazil_User-Manual_PIS-COFINS-Reoneracao-CST-06.md)
- Documento de fundamentos: [Guia Fiscal — PIS/COFINS](Localization-for-Brazil_User-Manual_guia-fiscal-pis-cofins.md)
