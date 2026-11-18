<!-- PR: #198, #229 -->
# Filiais: Preencher Filial em Movimentos Antigos

**Data**: Setembro/2026

---

## Sumário

Empresas que já operam há algum tempo no Business Central com um único estabelecimento e depois abrem a primeira filial precisam informar, na **Configuração Contabilidade** (página `General Ledger Setup`), qual dimensão identifica a filial. A partir desse momento o sistema passa a exigir a filial em praticamente todos os cadastros que geram movimento, e os movimentos anteriores — lançados antes da configuração — ficam sem filial. Na prática a rotina financeira trava no dia seguinte: baixar um título antigo contra um pagamento novo (que já nasce com filial) é recusado por falta de filial ou pelo bloqueio de aplicação entre filiais.

Esta atualização entrega o processo **Preencher Filial em Movimentos Antigos** (relatório `FillBranchInOldEntries`, ID 52112451), que completa retroativamente a filial nos movimentos que ficaram em branco — inclusive nos movimentos de estoque (**Movimentos Item** e **Movimentos Valor**), para que o histórico de estoque fique alinhado com a sua contrapartida contábil e o Bloco K da EFD ICMS/IPI enxergue a produção anterior à virada. O processo não é pesquisável: é executado pelo seu ID ou agendado na fila de tarefas.

---

## 1. Contexto

### O que é

Um processo, executado sob demanda, que grava a filial escolhida nos movimentos que **nunca tiveram filial**. A filial de um movimento não é um campo solto: ela é derivada da dimensão de filial gravada no próprio movimento. Por isso o processo preenche as três coisas de uma vez:

- a dimensão de filial dentro do conjunto de dimensões do movimento;
- a dimensão global correspondente, quando a dimensão de filial é uma das duas globais;
- o campo **Cód. Filial** (`BranchCodeFNX`) do movimento, que é por onde os relatórios da Localização filtram filial.

### Quando se aplica

- Empresa que acabou de informar o **Cód. Dimensão Filial** (`BranchDimensionCodeFNX`) na **Configuração Contabilidade** e tem histórico anterior a essa configuração.
- O processo pode ser executado quantas vezes for necessário: como ele só alcança o que está em branco, uma segunda execução não altera o que já ficou correto. Isso permite retomar com segurança uma execução interrompida.
- Empresa que **já executou** o processo antes de ele incluir o estoque: basta executá-lo de novo, com a mesma filial. As áreas já preenchidas não têm mais o que atualizar e aparecem zeradas no resumo; só os **Movimentos Item** e os **Movimentos Valor** que ficaram em branco são preenchidos.

### O que NÃO é

O processo **completa, nunca troca**. Só é alterado o movimento cuja filial está **totalmente em branco** — sem a dimensão de filial no conjunto de dimensões e sem o **Cód. Filial**. Movimento que já tenha qualquer um dos dois fica intacto. A proteção que impede trocar a filial de um movimento originado de documento fiscal continua valendo para trocas; ela não se aplica aqui, porque o processo apenas preenche o que nunca existiu.

---

## 2. Ordem de Execução

A ordem importa. Gravar a dimensão de filial na **Configuração Contabilidade** desfaz o vínculo entre cada filial cadastrada e o seu valor de dimensão — nesse instante nenhuma filial tem valor associado, e não haveria o que gravar nos movimentos. Por isso o preenchimento é um processo separado, e o próprio sistema avisa disso ao gravar a dimensão:

> Nenhuma filial tem ainda um valor de dimensão associado, e os movimentos lançados até agora não têm filial. Associe cada filial ao seu valor de dimensão na ficha da filial. Em seguida, preencha a filial nos movimentos lançados antes desta configuração com o relatório 52112451, Preencher Filial em Movimentos Antigos, que não é pesquisável e é executado pelo seu ID ou agendado na fila de tarefas.

1. Na **Configuração Contabilidade**, informe o **Cód. Dimensão Filial**. O sistema confirma a mudança e, ao final, exibe o aviso acima. **Nenhum movimento é alterado nesse momento.**
2. Na **Lista Filiais** (página `BranchListFNX`), abra cada filial e, na ficha **Informação Filial** (página `BranchInformationFNX`), informe o **Cód. Valor Dimensão** (`DimensionValueCode`) correspondente àquela filial.
3. Execute **Preencher Filial em Movimentos Antigos**.

---

## 3. Como Executar o Processo

O processo **não aparece na pesquisa** do Business Central e não tem ação em nenhuma página. Isso é deliberado: trata-se de um processo de virada, de execução única e em janela de manutenção, e não de uma rotina de uso corrente. Há dois caminhos para executá-lo.

> **Quem pode executar.** O processo **não está incluído em nenhum conjunto de permissões da Localização**, pelo mesmo motivo das demais rotinas de correção em massa. Só o executa quem tem **SUPER** ou quem recebeu permissão explícita sobre o relatório 52112451 e o codeunit `FillBranchInOldEntriesMgt`. Isso vale também para a execução pela fila de tarefas, que roda com as permissões do usuário informado no movimento da fila.

### 3.1. Pelo ID do relatório

Acrescente o parâmetro `report=52112451` ao endereço do Business Central, na forma `.../?company=<Empresa>&report=52112451`. A página de solicitação do processo é aberta diretamente.

### 3.2. Agendado na fila de tarefas

1. Abra **Movs. de Fila de Tarefas** (página `Job Queue Entries`) e crie um novo movimento.
2. Informe **Tipo Objeto a Executar** = `Relatório` e **ID do Objeto para Executar** = `52112451`.
3. Na ação **Página de Requisição de Relatório**, informe o **Código Filial**. Sem isso o processo termina com erro, porque a filial é obrigatória.
4. Agende o movimento para a janela de manutenção.

> **Atenção**: na execução agendada não há ninguém para responder à confirmação, e por isso ela não é apresentada — o processo roda direto. O resumo final por área também não é exibido; confira o resultado pelos próprios movimentos ou pelo log da fila de tarefas.

### 3.3. Página de solicitação

Na página de solicitação, no grupo **Opções**:

| Campo | Descrição |
|-------|-----------|
| **Código Filial** | A filial que corresponde ao estabelecimento que a empresa já era antes de abrir a nova. Selecionável pela **Lista Filiais**, onde a filial é identificada também pelo nome e pelo CNPJ |
| **Nome Filial** | Exibido automaticamente ao informar o código, para conferência. Não editável |

Validações feitas ao informar a filial:

- Filial que ainda não tem **Cód. Valor Dimensão** associado é recusada, com a indicação de associá-la na ficha da filial antes de executar o processo.
- Filial associada a uma dimensão que não é a dimensão de filial da **Configuração Contabilidade** é recusada.
- Se o **Cód. Dimensão Filial** não estiver informado na **Configuração Contabilidade**, o processo não é executado.

Em seguida o sistema pede confirmação, listando as áreas que serão atualizadas — movimentos contábeis, de cliente, de fornecedor, de empregado, de conta bancária, de ativo fixo, de item e de valor —, informando que os movimentos com filial em branco são atualizados em aberto e liquidados, em todo o histórico, e que o processo deve ser executado sem outros usuários trabalhando na empresa. **Se você recusar, nada é alterado.**

Ao final, em execução interativa, o sistema informa quantos movimentos foram preenchidos em cada área, nesta ordem: **Movimentos Contábeis**, **Movimentos Cliente**, **Movimentos Fornecedor**, **Movimentos Empregado**, **Movimentos Conta Bancária**, **Movimentos Ativo Fixo**, **Movimentos Item**, **Movimentos Valor** e **Movimentos Detalhados**.

---

## 4. O que é Preenchido

| Área | Movimento |
|------|-----------|
| Contabilidade | **Movimentos Contábeis**, inclusive os originados de documento fiscal |
| Contas a receber | **Movimentos Cliente** |
| Contas a pagar | **Movimentos Fornecedor** |
| Funcionários | **Movimentos Empregado** |
| Tesouraria | **Movimentos Conta Bancária** |
| Ativo Fixo | **Movimentos Ativo Fixo** |
| Estoque | **Movimentos Item** e **Movimentos Valor**, inclusive os de produção e montagem |
| Saldos por dimensão global | **Movimentos Detalhados** de cliente, fornecedor e empregado, quando a dimensão de filial é uma das duas globais |

**Não há recorte de "em aberto"**: títulos já baixados e quitados entram junto com os em aberto. Isso é necessário, entre outras razões, para que a EFD Contribuições enxergue os impostos já pagos do período de virada, que por definição são títulos baixados.

**Não há filtro por data de lançamento**: o processo alcança todo o histórico da empresa.

Os **Movimentos Detalhados** entram porque é de lá — e não do título — que o Business Central calcula o saldo do cliente, do fornecedor e do funcionário quando há filtro por dimensão global. Sem eles, a ficha e a lista de clientes filtradas por filial continuariam sem enxergar o histórico. Apenas o campo da dimensão global **da filial** é tocado; o outro campo global (Departamento ou Centro de Custo, por exemplo) fica como está, preenchido ou em branco.

Quando a dimensão de filial **não** é uma das duas dimensões globais, o processo preenche o conjunto de dimensões e o **Cód. Filial** dos movimentos, e nenhum movimento detalhado é alterado — os detalhados só guardam as duas dimensões globais.

### Movimentos de estoque

Os **Movimentos Item** e os **Movimentos Valor** seguem as mesmas regras das demais áreas: só é alterado o movimento com a filial totalmente em branco, sem recorte de data nem de situação — movimentos de item já totalmente consumidos ou fechados entram junto. Quantidades, custos, aplicações e a situação de aberto ou fechado do movimento não são alterados; o processo grava apenas a dimensão de filial, a dimensão global correspondente e o **Cód. Filial**.

Dois pontos específicos do estoque:

- **Cada movimento é avaliado pela sua própria filial.** Um **Mov. Valor** que já tenha filial fica intacto, mesmo que o **Mov. Item** a que pertence estivesse em branco e tenha sido preenchido agora — e o inverso também vale. O processo não copia a filial de um para o outro.
- **O depósito não é consultado.** O cadastro de depósito pode estar associado a uma filial (campo **Cód. Filial** no **Cartão Depósito**). Mesmo quando um movimento em branco está num depósito associado a outra filial, ele recebe a **filial escolhida no processo**, como nas demais áreas. Assim o estoque fica alinhado com a sua contrapartida contábil, que também recebe a filial escolhida.

Depois do processo, os movimentos de estoque antigos passam a aparecer ao filtrar por filial nas páginas **Movimentos Item** e **Movimentos Valor**, nos relatórios de estoque e nas apurações que filtram o movimento de item por filial (veja *Bloco K, Bloco H e FCI*, abaixo).

### Bloco K, Bloco H e FCI

O preenchimento dos movimentos de estoque muda o resultado de parte das apurações fiscais **de períodos anteriores à virada**, quando elas passam a ser feitas por filial:

| Apuração | Como identifica a filial | Efeito do processo |
|----------|--------------------------|--------------------|
| EFD ICMS/IPI — Bloco K, registros K230/K235 (produção e consumo) | Pelo **Cód. Filial** do **Mov. Item** | Passa a trazer a produção do período anterior à virada para a filial escolhida. Antes do processo, não trazia nada |
| FCI — Ficha de Conteúdo de Importação | Pelo **Cód. Filial** do **Mov. Item** de produção e de consumo | Passa a enxergar a produção e o consumo anteriores à virada na filial escolhida |
| EFD ICMS/IPI — Bloco K, registro K200 (estoque escriturado) | Pelo **Cód. Filial** do **Cartão Depósito** | Não muda: o saldo continua atribuído à filial do depósito |
| EFD ICMS/IPI — Bloco H (inventário) | Pelos depósitos associados à filial da apuração | Não muda |

Consequências práticas:

- **Associe os depósitos antigos à filial escolhida.** O K200 e o Bloco H não olham a filial do movimento, e sim a do depósito. Quando a apuração é feita por filial, um depósito com saldo e sem **Cód. Filial** interrompe a geração do K200 com erro. Revise o **Cód. Filial** de cada depósito no **Cartão Depósito** antes de gerar a EFD por filial.
- **Depósito associado a outra filial.** Nesse caso o movimento de item antigo recebe a filial escolhida, mas o saldo daquele depósito no K200 e no Bloco H continua na filial do depósito. É o comportamento esperado: cada registro segue o seu próprio critério.
- **Períodos já transmitidos.** Assim como na ECD e na EFD Contribuições (veja a seção 5), regerar os registros K230/K235 ou a FCI de um período já entregue passa a produzir resultado diferente do transmitido.

### Fora do escopo

- Movimentos de recurso, de capacidade e de inventário físico. Os de capacidade e de inventário físico não têm **Cód. Filial**.
- Documentos ainda não lançados: pedidos de compra e venda em aberto, faturas em digitação e diários não lançados também passarão a exigir a filial no momento do lançamento e continuam a ser corrigidos manualmente.

---

## 5. Cuidados Antes de Executar

**Rode com a empresa parada.** O processo grava diretamente nos movimentos contábeis e nos auxiliares, sobre todo o histórico. É execução de janela de manutenção. Não há trava programática impedindo outros usuários de trabalharem — é orientação de operação, reforçada no texto da confirmação.

**O processo alcança todo o histórico, inclusive exercícios já transmitidos.** Como não há filtro por data de lançamento, **regerar** uma ECD de exercício já entregue, ou reabrir uma apuração de PIS/COFINS já fechada, passa a produzir resultado diferente do que foi transmitido, porque esses processos filtram os movimentos contábeis por filial. Avalie e comunique isso ao cliente antes de executar.

**Aplicações históricas entre filiais diferentes.** Se a empresa tinha o bloqueio de aplicação entre filiais desligado, pode existir título antigo em branco quitado por um pagamento que já nasceu com outra filial. Depois do processo esse título recebe a filial escolhida, e a aplicação passa a ligar duas filiais no histórico. Nada quebra — já está lançado —, mas o processo não reconcilia esses casos: a premissa é que a filial escolhida é o estabelecimento que a empresa já era.

**Pause o ajuste de custo.** Se houver uma tarefa **Ajustar Custo - Mov. Itens** agendada em **Movs. de Fila de Tarefas**, use a ação **Definir status Em Espera** nela antes de executar o processo e **Definir status Pronto** depois. As duas rotinas gravam nos mesmos **Movimentos Item** e **Movimentos Valor** e disputariam os mesmos registros.

**Ajustes de custo posteriores.** O ajuste de custo copia as dimensões do **Mov. Valor** original. Depois do processo, os ajustes de custo que ainda vierem a ser feitos sobre movimentos antigos passam a sair com a filial escolhida, tanto no **Mov. Valor** do ajuste quanto na contabilidade — inclusive quando o movimento está num depósito associado a outra filial.

**Vistas de análise contábeis.** Ao final, as vistas de análise não bloqueadas que acompanham a dimensão de filial são marcadas com **Atualização de dados necessária**, para que não sigam mostrando o rateio por dimensão anterior ao preenchimento. Atualize essas vistas em **Dados Análises** (página `Analysis View List`) depois do processo.

**Vistas de análise de item.** As vistas de análise de estoque, de venda e de compra (**Ver Lista Análises Inventário**, **Ver Lista Análises Venda** e **Ver Lista Análises Compra**) **não** são tratadas pelo processo. Uma vista dessas que acompanhe a dimensão de filial continua mostrando o histórico sem filial até ser reconstruída. Como a ação **Atualizar** só acrescenta os movimentos novos, a reconstrução é feita assim, na ficha da vista (por exemplo, **Cartão Visualização Análise Inv.**):

1. Altere o campo **Compressão por Datas** (ou a **Data Inicial**) para outro valor. O sistema avisa que os movimentos dos dados de análise serão eliminados e que será preciso atualizar novamente; confirme.
2. Volte o campo ao valor original.
3. Execute a ação **Atualizar**. A vista é reconstruída a partir dos movimentos, agora com a filial preenchida.

Uma vista bloqueada não aceita essa alteração: desbloqueie-a antes.

**Volume.** O processo percorre oito tabelas de movimento mais as três de movimentos detalhados, sobre todo o histórico. Os **Movimentos Valor** costumam ser a maior tabela da base e, em empresas com estoque, tendem a dominar o tempo do processo. Em bases com anos de movimentação, dimensione a janela de manutenção de acordo e prefira agendar na fila de tarefas.

---

## 6. Resumo dos Objetos Novos

### Localization for Brazil

| Objeto | Descrição |
|--------|-----------|
| **Preencher Filial em Movimentos Antigos** (Relatório `FillBranchInOldEntries`, 52112451) | Processo com página de solicitação, escolha da filial e confirmação. Não pesquisável: executado pelo ID do relatório ou agendado na fila de tarefas |
| `FillBranchInOldEntriesMgt` (Codeunit, 52112613) | Motor do processo: resolve o conjunto de dimensões novo para cada conjunto antigo e grava as oito áreas de movimento (contabilidade, cliente, fornecedor, empregado, conta bancária, ativo fixo, item e valor), os movimentos detalhados e a marcação das vistas de análise |

### Alteração em objeto existente

| Objeto | Alteração |
|--------|-----------|
| **Configuração Contabilidade** (extensão `GeneralLedgerSetupLocFNX`) | Ao informar o **Cód. Dimensão Filial**, o sistema avisa que nenhuma filial tem valor de dimensão associado, que é preciso associá-lo em cada ficha de filial e que o preenchimento retroativo é feito pelo relatório 52112451. Nenhum movimento é alterado nesse momento |
