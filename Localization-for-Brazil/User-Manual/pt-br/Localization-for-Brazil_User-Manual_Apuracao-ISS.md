<!-- Branch: master -->

# Apuração de ISS

**Data**: Maio/2026

---

## Sumário

A apuração de ISS é o processo, dentro do app **Livros Fiscais** (Tax Books), que consolida os documentos fiscais de serviço (NFS-e) registrados no período, concilia esses documentos com o arquivo da prefeitura e gera uma **Declaração de Imposto** (`TaxStatementFNX`) com o ISS a recolher, que pode ser revisada, aprovada e registrada na contabilidade. Este documento descreve, passo a passo, todas as configurações necessárias e o fluxo completo, desde a criação da apuração até o registro dos lançamentos contábeis.

## 1. Contexto

A apuração de ISS é normalmente feita em duas etapas:

1. A **Apuração de ISS** (`ISSSettlementFNX`) consolida os documentos fiscais de serviço (NFS-e) registrados no período por filial, importa o arquivo de NFS-e da prefeitura e concilia os dois lados (sistema × prefeitura), permitindo identificar divergências antes de fechar o período.
2. A **Declaração de Imposto** (`TaxStatementFNX`) apresenta o demonstrativo final (total de serviços, débitos, ajustes e imposto a recolher), passa por liberação ou aprovação opcional e, ao ser registrada, gera os lançamentos contábeis de débito do ISS a recolher contra a conta de imposto a pagar.

O sistema suporta:

- **Conciliação automática** entre os documentos do sistema e os documentos da prefeitura, cruzando o número fiscal da NFS-e e o valor do ISS.
- **Conciliação manual** para os casos em que o cruzamento automático não consegue parear o documento.
- **Versionamento** da conciliação. A cada arquivamento, o sistema salva uma foto completa dos dois lados, permitindo rastreabilidade e auditoria.
- **Importação nativa** de arquivos de prefeituras de Rio de Janeiro/RJ, Macaé/RJ, Saquarema/RJ, São Paulo/SP, Cotia/SP e Salvador/BA.
- **Ajustes de débitos e créditos** para acertos identificados em auditoria.

### Quando se aplica

- Empresas prestadoras de serviços com obrigação de apurar e recolher ISS.
- Apuração mensal, por filial.

---

## 2. Pré-requisitos

Antes de criar a primeira apuração, garanta que os pontos abaixo estão configurados. Os tópicos seguintes detalham cada um.

| Item | Onde |
| --- | --- |
| App **Localization for Brazil** instalado e configurado | Documento de instalação geral |
| App **Tax Books** (Livros Fiscais) publicado | Documento de instalação geral |
| **Informação Filial** com **CNPJ**, **Cidade** e **Cód. Postal** preenchidos | Página `BranchInformationFNX` |
| **Cód. Postal** com **Cód. IBGE** preenchido | Página `PostCodeFNX` |
| Documentos fiscais de serviço (NFS-e) do período registrados | Movimentação normal |
| Um **Modelo de Declaração de Imposto** com **Classificação Imposto** = ISS | Página **Modelos de Declaração de Impostos** (`TaxStatementTemplatesFNX`) |
| Configuração de contabilização de impostos para ISS | Configurações fiscais da localização |
| Configuração de NFS-e da filial (web service municipal, certificado, séries de RPS) | Configuração de NF-e da filial |
| Plano de Contas com contas para **imposto a recolher** e **ajustes** | Plano de Contas |

---

## 3. Configuração do Modelo de Declaração ISS

A **Declaração de Imposto** é gerada a partir de um **Modelo de Declaração de Imposto**, que define a numeração, as contas contábeis de débito e crédito do lançamento principal e a conta de ajuste. **Apenas um modelo com Classificação Imposto = ISS é permitido.** O sistema valida e impede a criação de um segundo modelo ISS.

### Passo 1: Acessar a página

1. Pesquise por **Modelos de Declaração de Impostos** (`TaxStatementTemplatesFNX`).

### Passo 2: Criar o modelo ISS

| Campo | Valor |
| --- | --- |
| **Código** (`Code`) | Ex.: `ISS` |
| **Classificação Imposto** (`TaxIdentification`) | **ISS** |
| **Descrição** (`Description`) | Ex.: "Apuração ISS" |
| **Cód. Jurisdição Guia Pagamento** (`PaymentBillJurisdictionCode`) | Jurisdição usada para gerar a Guia de Pagamento de Imposto. Apenas jurisdições com **Tipo Guia Pagamento** preenchido aparecem na lista |
| **Nº Série Declaração** (`StatementNoSeries`) | Nº série usado para numerar as Declarações de Imposto desse modelo |
| **Modelo de Descrição do Registro** (`PostingDescriptionTemplate`) | Modelo da descrição usada nos lançamentos contábeis. Aceita as variáveis `%1` (Classificação Imposto), `%2` (Nº Declaração), `%3` (Data Inicial) e `%4` (Data Final) |
| **Tipo de Conta Débito** (`DebitAccountType`) | **G/L Account** ou **Vendor**. Em geral, **G/L Account** |
| **Nº Conta Débito** (`DebitAccountNo`) | Conta de despesa/débito do imposto a recolher |
| **Tipo de Conta Crédito** (`CreditAccountType`) | **G/L Account** ou **Vendor**. Para apuração contra fornecedor "Prefeitura", use **Vendor** |
| **Nº Conta Crédito** (`CreditAccountNo`) | Conta de imposto a recolher (passivo) |
| **Nº Conta Ajuste** (`AdjustmentAccountNo`) | Conta para lançamento de ajustes. Só usada quando há valor de ajuste |

---

## 4. Configuração de Contabilização de Impostos

Na tabela de contas de contabilização de impostos, deve haver registros para a **Classificação Imposto** = ISS, definindo as contas de imposto a pagar, despesa e crédito fiscal, conforme a necessidade da empresa. Essa configuração é compartilhada com os demais impostos da localização e está detalhada no documento de fundamentos do **Localization for Brazil**.

---

## 5. Configuração de NFS-e por Filial

Na configuração de NF-e da filial, a seção **NFS-e** deve estar configurada com:

- Dados do **web service** da prefeitura.
- **Certificado digital** vinculado à filial.
- **Séries de RPS** que serão usadas na emissão.

Sem esses dados, os documentos de serviço não são emitidos nem registrados corretamente, e a apuração fica vazia para o período.

---

## 6. Configuração do Fluxo de Aprovação (opcional)

A Declaração de Imposto pode passar por um fluxo de aprovação antes do registro contábil. A configuração é feita em **Conf. Aprovação Declaração de Impostos** (`TaxStatementApprovalSetupFNX`):

1. Pesquise por **Conf. Aprovação Declaração de Impostos**.
2. Crie uma linha vinculando o modelo ISS ao fluxo:
   - **Cód. Modelo Declaração de Impostos** (`TaxStatementTemplateCode`): modelo ISS criado no [passo 3](#3-configuração-do-modelo-de-declaração-iss).
   - **Cód. Regra de Fluxo de Aprovação** (`ApprovalFlowRuleCode`): regra do fluxo de aprovação aplicável.
3. Use a ação **Observações de Aprovação** para registrar observações padrão usadas no fluxo.

Quando configurado, a declaração fica em **Pendente Aprovação** após a ação **Enviar Requisição Aprovação**, e só pode ser registrada após **Liberada**.

> **Nota:** se a aprovação não for configurada para o modelo, a declaração segue o fluxo simplificado (Aberto → Liberado → Registrado).

---

## 7. Criar a Apuração

Toda a apuração de ISS gira em torno do registro **Apuração de ISS** (`ISSSettlementFNX`). Pode haver uma apuração por filial e por mês.

### Passo 1: Acessar a lista

1. Pesquise por **Apurações de ISS** (`ISSSettlementsFNX`).

### Passo 2: Criar as apurações em lote

1. Clique na ação **Criar Apurações Imposto**.
2. Informe o **Ano** e o **Mês** do período de apuração.
3. O sistema cria automaticamente uma **Apuração de ISS** para cada filial cadastrada (ou uma única apuração se não houver filiais), preenchendo:
   - **Cód. Filial** (`BranchCode`).
   - **Data Inicial** (`StartDate`): primeiro dia do mês.
   - **Data Final** (`EndDate`): último dia do mês.

### Passo 3: Abrir o cartão de apuração

A partir da lista, clique na linha da apuração para abrir o cartão **Apuração de ISS**, que reúne:

- Bloco **Geral**: **Nº**, **Cód. Filial**, **Nº Versão**, **Data Inicial**, **Data Final**, **Status**.
- Bloco **Apuração**: **Saldo Devedor no Período**, **Estorno de Notas Fiscais**, **Diferença Débito e Crédito**.
- Bloco **Reconciliação**: saldos do **Sistema** (**Saldo Mov. Contábil**, **Saldo Pendente**, **Ajustes de Débitos**) e da **Prefeitura** (**Saldo Prefeitura**, **Saldo Pendente Prefeitura**, **Ajustes de Créditos**).
- Subpáginas **Sistema** (documentos do sistema) e **Prefeitura** (documentos da prefeitura), lado a lado.

---

## 8. Processar os Documentos

### Passo 1: Executar o processamento

1. No cartão **Apuração de ISS**, clique na ação **Processar**.

O processamento consolida todos os documentos fiscais do período que possuem ISS, populando o lado esquerdo da tela (**Sistema**) com os dados de cada NFS-e registrada.

### Passo 2: Conferir o bloco **Apuração**

Após o processamento, o bloco **Apuração** exibe:

- **Saldo Devedor no Período**: total de ISS apurado nos documentos.
- **Estorno de Notas Fiscais**: ISS de faturas canceladas no período.
- **Diferença Débito e Crédito**: diferença entre saldo e cancelamentos.

### Passo 3: Reprocessar quando necessário

Se você corrigir documentos do período, volte ao cartão e execute **Processar** novamente para recalcular tudo. Em geral, reprocesse a apuração depois de qualquer alteração que afete bases ou impostos do período.

---

## 9. Importar o Arquivo da Prefeitura

### Passo 1: Importar

1. No cartão da apuração, clique em **Importar Arquivo**.
2. Selecione o arquivo fornecido pela prefeitura (CSV ou Excel, conforme o município).
3. O sistema importa os documentos da prefeitura para o lado direito da tela (**Prefeitura**).

### Municípios suportados nativamente

| Município | UF | Formato |
| --- | --- | --- |
| Rio de Janeiro | RJ | CSV |
| Macaé | RJ | CSV |
| Saquarema | RJ | Excel |
| São Paulo | SP | CSV |
| Cotia | SP | CSV |
| Salvador | BA | CSV |

> **Nota:** o município é detectado automaticamente a partir do **Cód. IBGE** vinculado ao **Cód. Postal** da filial. Se o município não estiver na lista acima, o sistema retorna o erro "City *X* unsupported." e a importação não é feita.

---

## 10. Conciliar Documentos

### Passo 1: Conciliação automática

1. Clique na ação **Corresponder Auto.**
2. O sistema cruza os documentos do **Sistema** com os da **Prefeitura**, usando como critérios o **número fiscal da NFS-e** e o **valor do ISS**.
3. Os documentos pareados ficam com o status **ISS Reconciliado** = **Automático**.

### Passo 2: Conciliação manual

Para documentos que não foram conciliados automaticamente:

1. No lado esquerdo (**Sistema**), selecione o documento desejado.
2. No lado direito (**Prefeitura**), selecione o documento correspondente.
3. Clique em **Corresponder Manualmente**.
4. O par fica vinculado com status **ISS Reconciliado** = **Manualmente**.

### Passo 3: Desfazer uma conciliação manual

1. Selecione os documentos vinculados.
2. Clique em **Remover Correspondência**.

---

## 11. Ajustes e Versionamento

### Ajustes de Débitos e Créditos

A seção **Reconciliação** do cartão exibe os saldos de ambos os lados (Sistema e Prefeitura) e permite informar:

- **Ajustes de Débitos**: aumentam o valor de ISS a recolher (ex.: diferenças identificadas em auditoria).
- **Ajustes de Créditos**: reduzem o valor de ISS a recolher (ex.: créditos não considerados).

### Arquivar uma versão

1. Clique na ação **Arquivo** para salvar um snapshot da conciliação atual.
2. O sistema incrementa o **Nº Versão** e arquiva tanto os documentos do sistema quanto os da prefeitura.
3. As versões anteriores podem ser consultadas pela ação **Versões**, que abre a página **Arquivo de Reconciliação ISS** (`ISSReconciliationArchiveFNX`).
4. Dentro de uma versão arquivada, a ação **Restaurar Versão** restaura a conciliação para o estado salvo.

> **Recomendação:** arquive sempre que quiser preservar uma conciliação antes de aplicar correções ou recriar a apuração.

---

## 12. Gerar a Declaração ISS

### Passo 1: Gerar

1. No cartão da apuração, clique na ação **Declaração ISS**.
2. O sistema gera (ou atualiza) uma **Declaração de Imposto** (`TaxStatementFNX`) com os seguintes itens:
   - **Total de Serviços**: soma dos valores de serviço dos documentos do período.
   - **Débitos**: soma do ISS dos documentos.
   - **Ajustes de Débitos**, se houver.
   - **Ajustes de Créditos**, se houver.
   - **Imposto a Recolher**: valor final (Débitos + Ajustes de Débitos − Ajustes de Créditos).
3. A declaração é criada com status **Aberto**.

### Passo 2: Revisar a declaração

A partir do cartão da declaração, é possível:

- Inspecionar o bloco de detalhes (`TaxStatementDetailsFNX`) com a base, os débitos, os ajustes e o imposto a recolher.
- Acessar **Movimentos** para abrir os lançamentos contábeis pendentes (`TaxStatementEntryFNX`).
- Usar **Verificar Contabilização** para abrir a tela padrão do BC com os lançamentos simulados.
- Clicar em **Dimensões** para definir ou alterar o conjunto de dimensões aplicado aos lançamentos.

> **Importante:** o demonstrativo só é gerado se a declaração estiver em **Aberto**. Se já estiver liberada ou registrada, é necessário reabrir antes de reprocessar.

---

## 13. Liberar e Aprovar (se aplicável)

A declaração nasce com status **Aberto**.

### Sem aprovação configurada

1. Clique em **Liberar**.
2. O status passa para **Liberado**.
3. Para voltar a editar, use **Reabrir**.

### Com aprovação configurada (ver [seção 6](#6-configuração-do-fluxo-de-aprovação-opcional))

1. Clique em **Enviar Requisição Aprovação**.
2. O status passa para **Pendente Aprovação**.
3. O fluxo cria os registros em **Aprovações** e notifica os aprovadores por e-mail, com os detalhes do demonstrativo.
4. Cancelar é possível via **Cancelar Requisição Aprovação**.
5. Após o último aprovador, o status passa para **Liberado**.

> **Nota:** com aprovação habilitada, o botão **Liberar** fica desabilitado. A liberação acontece pelo fluxo.

---

## 14. Registrar a Declaração

Com a declaração **Liberada**, o registro contábil pode ser feito.

### Passo 1: Registrar

1. Na **Declaração de Imposto**, clique em **Registrar**.
2. Confirme a mensagem de confirmação.
3. O sistema percorre cada linha de **Movimento de Declaração de Imposto** e cria um lançamento de diário geral usando as contas de débito e crédito definidas no [modelo ISS](#3-configuração-do-modelo-de-declaração-iss).
4. O status da declaração passa para **Registrado**.

### Passo 2: Conferir os lançamentos

Use **Navegar** ou **Movimentos** para inspecionar:

- **Movimentos de Declaração de Imposto**: todas as linhas registradas.
- **Movimentos Contábeis**: através da ação **Navegar** padrão do BC, pelo número da declaração.

### Passo 3 (opcional): Gerar a Guia de Pagamento

Se o **Cód. Jurisdição Guia Pagamento** estiver preenchido no modelo, o lançamento já carrega a jurisdição correta e o cálculo do vencimento. A geração da Guia em si é feita pelo módulo de **Guias de Pagamento de Imposto**.

---

## 15. Estornar a Apuração

Após registrar, é possível estornar a declaração para corrigir e refazer.

1. Na **Declaração de Imposto** (status **Registrado**), clique em **Reverter**.
2. Confirme a mensagem de confirmação.
3. O sistema cria lançamentos de estorno (com sinal trocado), marca os movimentos originais como estornados e cria cópias dos movimentos para serem registrados novamente.
4. O status passa para **Estornado**.
5. A partir daí, é possível **Reabrir** a declaração para edição manual, ou voltar à Apuração de ISS, ajustar os documentos e **Processar** novamente.

> **Nota:** a reversão é feita exclusivamente pelo botão **Reverter**. Não tente apagar movimentos contábeis manualmente. Isso quebra a rastreabilidade da declaração.

---

## 16. Relatórios Disponíveis

Na **Apuração de ISS**, dois relatórios podem ser emitidos pelo grupo **Imprimir** da barra de ações:

- **Registro de ISS** (`ISSRegistryBookFNX`): Livro Registro de ISS com os documentos fiscais de saída de serviço, agrupados por apuração, contendo data, série, número fiscal, alíquota ISS, base, valor ISS, retenção, isento, remessa/devolução e cancelados.
- **Livro Registro de Saídas ISS** (`ISSOutDocRegisterFNX`): inclui documentos do sistema e ajustes de débito e crédito da apuração, com opção de visualização em tela.

---

## 17. Resumo de Status

A **Declaração de Imposto** segue o ciclo abaixo. O botão **Registrar** só fica habilitado nos status que permitem registro.

| Status (`TaxStatementStatusFNX`) | Significado | Ações habilitadas |
| --- | --- | --- |
| **Aberto** (`Open`) | Recém-criada / após reabertura. Detalhes editáveis | **Liberar**, **Enviar Requisição Aprovação**, reprocessar (via apuração), **Verificar Contabilização** |
| **Pendente Aprovação** (`PendingApproval`) | Aguardando aprovador | **Cancelar Requisição Aprovação** |
| **Liberado** (`Released`) | Pronta para registrar | **Reabrir**, **Registrar**, **Verificar Contabilização** |
| **Registrado** (`Posted`) | Lançamentos contábeis efetivados | **Reverter** |
| **Estornado** (`Reversed`) | Estorno gerado, com cópias dos movimentos pendentes | **Reabrir**, **Registrar** (novamente) |

---

## 18. Particularidades

- **Apenas um modelo ISS é permitido.** O sistema valida e impede a criação de um segundo Modelo de Declaração de Imposto com **Classificação Imposto** = ISS.
- **Versionamento da conciliação.** Cada vez que você arquiva, o sistema salva uma foto completa dos dois lados (sistema e prefeitura). É possível restaurar uma versão anterior a qualquer momento pela ação **Versões** → **Restaurar Versão**.
- **A conciliação automática cruza por número fiscal + valor do ISS.** Se o número ou o valor divergirem entre sistema e prefeitura, será necessário conciliar manualmente.
- **O demonstrativo só é gerado se a declaração estiver em Aberto.** Se já estiver liberada ou registrada, é necessário reabrir antes de reprocessar.
- **O valor de imposto a recolher só aparece no demonstrativo se for positivo.** Se o resultado for zero ou negativo (créditos maiores que débitos), a linha de imposto a recolher não é exibida.
- **Identificação do município pelo Cód. IBGE da filial.** O leiaute do arquivo importado é selecionado a partir do **Cód. IBGE** do **Cód. Postal** da filial. Sem **Cód. IBGE** preenchido, a importação falha.

---

## Referências

- Página **Apurações de ISS** (`ISSSettlementsFNX`): lista mestra das apurações.
- Página **Apuração de ISS** (`ISSSettlementFNX`): cartão dedicado da apuração.
- Página **Modelos de Declaração de Impostos** (`TaxStatementTemplatesFNX`): modelo ISS.
- Página **Declaração de Imposto** (`TaxStatementFNX`): cartão da declaração.
- Página **Conf. Aprovação Declaração de Impostos** (`TaxStatementApprovalSetupFNX`): fluxo de aprovação.
- Página **Arquivo de Reconciliação ISS** (`ISSReconciliationArchiveFNX`): versões arquivadas da conciliação.
- Documento de fundamentos: [Guia Fiscal — NFS-e e ISS](Localization-for-Brazil_User-Manual_guia-fiscal-nfse.md)
