<!-- Branch: master -->

# Variação Cambial — Ajuste das Taxas de Câmbio

**Data**: Maio/2026

---

## Sumário

A variação cambial é o efeito financeiro causado pela oscilação das taxas de câmbio sobre saldos em aberto denominados em moeda estrangeira. No Business Central, esses saldos são reavaliados pelo processo **Ajustar Taxas de Câmbio**, que recalcula o valor em moeda local de movimentos de clientes, fornecedores e contas bancárias e gera lançamentos automáticos de ganho ou perda. Este manual explica o conceito, as configurações necessárias, o fluxo de execução, os lançamentos gerados, as particularidades da localização brasileira para adiantamentos, uma lista de verificação para diagnosticar variações fora do esperado e o procedimento de estorno.

## 1. Contexto

Quando uma empresa registra uma transação (venda, compra, pagamento ou recebimento) em moeda estrangeira, o valor é convertido para a moeda local na data da operação, usando a taxa de câmbio vigente. Se a taxa muda entre a data da operação e a data do fechamento (ou da liquidação), surge uma diferença, a **variação cambial**, que pode representar um **ganho** ou uma **perda** para a empresa.

No Business Central, o processo que recalcula esses saldos com base na taxa de câmbio atualizada é chamado **Ajustar Taxas de Câmbio**.

A lógica fundamental:

| Tipo de saldo | Moeda estrangeira valoriza | Moeda estrangeira desvaloriza |
|---------------|---------------------------|-------------------------------|
| **Ativo** (Clientes, Bancos) | **Ganho** (ativo vale mais em ML) | **Perda** (ativo vale menos em ML) |
| **Passivo** (Fornecedores) | **Perda** (passivo aumenta) | **Ganho** (passivo diminui) |

> ML = Moeda Local

---

## 2. Configurações Necessárias

### 2.1. Cadastro de Moedas

Acesse a página **Moedas** (`Currencies`) e abra o **Cartão Moeda** (`Currency Card`) de cada moeda estrangeira utilizada. Para cada moeda, é necessário configurar **seis contas contábeis** que receberão os lançamentos gerados pelo ajuste:

| Campo | Finalidade |
|-------|-----------|
| **Conta Ganhos Não Realiz.** | Ganhos sobre saldos ainda em aberto (não liquidados) |
| **Conta Perdas Não Realiz.** | Perdas sobre saldos ainda em aberto |
| **Conta Ganhos Realiz.** | Ganhos apurados no momento da liquidação |
| **Conta Perdas Realiz.** | Perdas apuradas no momento da liquidação |
| **Conta Aj. Residual Ganho** | Ganhos residuais de arredondamento |
| **Conta Aj. Residual Perda** | Perdas residuais de arredondamento |

> Sem essas contas configuradas, o processo de ajuste não consegue gerar os lançamentos contábeis e retorna erro.

### 2.2. Grupos Contábeis

As contrapartidas dos lançamentos de ajuste cambial vêm dos grupos contábeis dos parceiros e das contas bancárias:

- **Grupo Contábil Cliente**: campo **Conta Clientes** (`Receivables Account`)
- **Grupo Contábil Fornecedor**: campo **Conta Fornecedor** (`Payables Account`)
- **Grupo Contábil de Conta Bancária**: conta vinculada à conta bancária

Esses são os mesmos campos usados no registro normal de faturas e pagamentos. O processo de ajuste reutiliza essas contas para manter o saldo do parceiro ou do banco contabilmente coerente com o saldo recalculado pela nova taxa.

### 2.3. Taxas de Câmbio

A tabela **Valor da Taxa de Conversão** (`Currency Exchange Rate`) precisa estar atualizada com a taxa vigente na **Data Registro** que será usada no processo. Se não houver uma taxa cadastrada exatamente nessa data, o sistema utiliza a última taxa disponível antes dela.

> Cadastrar a taxa correta da data antes de rodar o ajuste é a primeira validação a fazer quando a variação calculada não bate com o esperado.

---

## 3. Execução do Processo

O processo é executado pelo relatório **Ajustar Taxas de Câmbio**, acessível via Diga-me (`Alt+Q`). Ao executar, o usuário define os parâmetros:

| Parâmetro | Descrição |
|-----------|-----------|
| **Moeda** | Filtro para selecionar quais moedas serão ajustadas |
| **Data Inicial / Data Final** | Período dos registros que serão considerados |
| **Data Registro** | Data em que os lançamentos de ajuste serão registrados |
| **Nº Documento** | Número do documento aplicado aos lançamentos gerados |
| **Ajustar Clientes** | Se os saldos de clientes serão ajustados |
| **Ajustar Fornecedores** | Se os saldos de fornecedores serão ajustados |
| **Ajustar Contas Bancárias** | Se os saldos de contas bancárias serão ajustados |
| **Ajustar por Movimento** | Se marcado, gera um lançamento individual por registro em aberto; caso contrário, agrupa por grupo contábil e moeda |

O sistema percorre todos os movimentos em aberto (clientes, fornecedores, contas bancárias) que possuem saldo em moeda estrangeira. Para cada movimento:

1. Calcula o **Valor Pendente (ML)** atual, ou seja, quanto o saldo em aberto vale na moeda local com a taxa originalmente registrada (incluindo ajustes anteriores).
2. Recalcula o mesmo saldo usando a **nova taxa de câmbio** da Data Registro.
3. A diferença entre o valor recalculado e o valor atual é o **valor do ajuste**.

> O processo calcula apenas a **diferença incremental** desde o último ajuste. Cada execução não repete o ajuste total. Ela só registra o que mudou.

---

## 4. Lançamentos Contábeis

Os lançamentos gerados seguem lógicas distintas conforme o tipo de saldo ajustado.

### 4.1. Clientes (Contas a Receber)

Um saldo a receber em moeda estrangeira é um **ativo**. Se a moeda estrangeira se valoriza, o ativo passa a valer mais em moeda local, e há **ganho**. Se desvaloriza, o ativo vale menos, e há **perda**.

**Ganho não realizado** (moeda estrangeira valorizou):

| Conta | Débito | Crédito |
|-------|--------|---------|
| Conta Clientes (do **Grupo Contábil Cliente**) | X | |
| Conta Ganhos Não Realiz. (da **Moeda**) | | X |

**Perda não realizada** (moeda estrangeira desvalorizou):

| Conta | Débito | Crédito |
|-------|--------|---------|
| Conta Perdas Não Realiz. (da **Moeda**) | X | |
| Conta Clientes (do **Grupo Contábil Cliente**) | | X |

### 4.2. Fornecedores (Contas a Pagar)

Um saldo a pagar em moeda estrangeira é um **passivo**. A lógica é inversa em relação aos clientes: se a moeda estrangeira se valoriza, o passivo aumenta, e há **perda**. Se desvaloriza, o passivo diminui, e há **ganho**.

**Perda não realizada** (moeda estrangeira valorizou e o passivo aumentou):

| Conta | Débito | Crédito |
|-------|--------|---------|
| Conta Perdas Não Realiz. (da **Moeda**) | X | |
| Conta Fornecedor (do **Grupo Contábil Fornecedor**) | | X |

**Ganho não realizado** (moeda estrangeira desvalorizou e o passivo diminuiu):

| Conta | Débito | Crédito |
|-------|--------|---------|
| Conta Fornecedor (do **Grupo Contábil Fornecedor**) | X | |
| Conta Ganhos Não Realiz. (da **Moeda**) | | X |

### 4.3. Contas Bancárias

O saldo bancário em moeda estrangeira é um ativo, e a lógica segue o mesmo raciocínio dos clientes. A contrapartida é a conta configurada no **Grupo Contábil de Conta Bancária**. As contas de ganho e perda usadas são as **realizadas** (Conta Ganhos Realiz. / Conta Perdas Realiz.), diferentemente de clientes e fornecedores, onde se usam as não realizadas.

---

## 5. Não Realizado vs. Realizado

O Business Central separa o reconhecimento contábil em duas camadas:

- **Não realizado**: o ajuste é sobre saldos **em aberto**; a transação ainda não foi liquidada. Os valores são provisórios e podem mudar a cada nova execução do processo de ajuste, conforme a taxa varia.
- **Realizado**: o ganho ou perda é apurado no momento do **pagamento ou recebimento**, quando a diferença se concretiza. Esses lançamentos são gerados automaticamente pelo sistema ao registrar a liquidação, e não pelo processo de ajuste.

Quando uma transação é finalmente liquidada, o sistema zera automaticamente os ajustes não realizados acumulados sobre aquele saldo, porque o saldo pendente passa a ser zero, e mantém apenas a diferença realizada na conta de ganhos ou perdas realizadas.

## 6. Como o ajuste é calculado

Para cada registro aberto de cliente ou fornecedor em moeda estrangeira (Movimento Cliente / Mov. Fornecedor), o sistema:

1. **Consulta o ajuste anterior acumulado**: soma todos os Movimentos Detalhados (Mov. Detalhado Cliente / Mov. Detalhado Fornecedor) do tipo "Ganho Não Realizado" e "Perda Não Realizada" já existentes para aquele registro.
2. **Calcula o novo valor em moeda local (R$)**: converte o Valor Pendente em moeda estrangeira pela taxa de câmbio vigente na data informada na execução.
3. **Compara o novo valor com o saldo atual em moeda local** (já incluindo ajustes anteriores). A diferença é o ajuste a ser contabilizado.

------

### O que acontece com o lançamento do mês anterior

O sistema não estorna o lançamento do mês 1. Em vez disso, ele calcula a diferença **incremental (diferencial)**. Exemplo:

| Situação                               | Valor                                                        |
| :------------------------------------- | :----------------------------------------------------------- |
| Fatura original: USD 1.000 à taxa 5,00 | R$ 5.000                                                     |
| **Mês 1** — taxa sobe para 5,20        | Ajuste = R$ 200 (ganho ou perda não realizado)               |
| **Mês 2** — taxa sobe para 5,50        | Novo ajuste total necessário = R$ 500, já existe R$ 200 → lança apenas **R$ 300** |

Ou seja:

- O processo consulta **quanto já foi ajustado** anteriormente (os R$ 200 do mês 1)
- Calcula **quanto deveria ser** o ajuste total pela nova taxa (R$ 500)
- Lança **apenas a diferença** (R$ 300)

Se no mês 2 a taxa tivesse **caído** (ex.: para 5,10), o sistema geraria um ajuste negativo, invertendo parte do ganho ou perda anterior, mas ainda sem estornar o lançamento original. Ele apenas complementaria com a diferença.

## 7. Cenário Prático — Ajustes Sucessivos

Suponha uma fatura de venda de **USD 1.000** emitida em **01/03**, quando a taxa era **R$ 5,00**. O valor registrado em moeda local foi **R$ 5.000**.

Cenário mês a mês, com a fatura permanecendo em aberto:

| Data | Evento | Taxa | Valor em R$ do saldo | Ajuste gerado |
|------|--------|------|----------------------|---------------|
| 01/03 | Fatura registrada | 5,00 | 5.000 | — |
| 31/03 | 1º Ajuste de câmbio | 5,20 | 5.200 | **+200** (Ganho não realizado) |
| 30/04 | 2º Ajuste de câmbio | 5,10 | 5.100 | **−100** (Perda não realizada) |
| 31/05 | 3º Ajuste de câmbio | 5,35 | 5.350 | **+250** (Ganho não realizado) |

Observe a leitura do segundo ajuste, em 30/04:

- O saldo já estava ajustado para R$ 5.200 pela taxa de 5,20 do mês anterior.
- A nova taxa de 5,10 indica que o saldo deveria valer R$ 5.100.
- Portanto, o ajuste é de **−100**, uma perda que reverte parcialmente o ganho do mês anterior.

Enquanto a fatura permanecer em aberto, cada execução do processo com taxa diferente gera um novo lançamento incremental. Esses valores são provisórios e mudam até a liquidação.

### O que acontece quando o não realizado supera o realizado

Continuando o exemplo: em 31/05 o saldo acumula ajustes não realizados líquidos de **+R$ 350** (200 − 100 + 250). A contabilidade tem o saldo do cliente em **R$ 5.350**.

Em **15/06**, o cliente paga os USD 1.000, e a taxa do dia é **5,15**, totalizando **R$ 5.150** recebidos.

O ganho **realizado** efetivo é apenas **R$ 150** (5.150 − 5.000). Mas a contabilidade já carrega **R$ 350** de ganho não realizado acumulado. O sistema resolve assim:

1. Na liquidação (aplicação do pagamento), o sistema registra o ganho realizado de R$ 150 com base na diferença entre a taxa original da fatura e a taxa do pagamento.
2. Como o saldo pendente da fatura passa a ser zero, os R$ 350 de ajustes não realizados acumulados são revertidos automaticamente. O sistema gera um lançamento inverso que zera o efeito não realizado.

Efeito líquido final na contabilidade:

| Conta | Efeito líquido |
|-------|----------------|
| Conta Ganhos Não Realiz. | Zero (os +350 são revertidos com −350) |
| Conta Ganhos Realiz. | +150 (o ganho efetivo apurado no pagamento) |
| Conta Clientes | Zerada (saldo pago integralmente) |

Ou seja: ganhos e perdas não realizados são temporários por natureza. Quando a transação é liquidada, eles desaparecem da contabilidade, e apenas o resultado **realizado** permanece.

---

## 8. Particularidades da Localização Brasileira

A localização brasileira adiciona dois comportamentos específicos para **adiantamentos (pré-pagamentos)** brasileiros, isto é, movimentos cujo lançamento original foi feito com a marcação de adiantamento no Diário Geral, propagada para o **Movimento Cliente** ou **Movimento Fornecedor** correspondente.

### 8.1. Conta de Contrapartida Específica

Quando a rotina ajusta um movimento de adiantamento brasileiro, a **conta de contrapartida** utilizada não é a Conta Clientes / Conta Fornecedor padrão do grupo contábil. Em vez disso:

- O sistema utiliza a **conta de adiantamento** configurada no respectivo grupo contábil (por exemplo, o campo **Conta de Adiantamento** do Grupo Contábil Cliente / Fornecedor adicionado pela localização).
- Isso garante que a variação cambial sobre adiantamentos fique segregada contabilmente da variação sobre títulos normais, em coerência com o tratamento contábil brasileiro de pagamentos antecipados.

> Se o registro for um adiantamento e a conta de adiantamento não estiver configurada no grupo contábil correspondente, o processo gera um erro durante a execução. Ajuste a configuração antes de rodar novamente.

A localização possuía anteriormente um relatório próprio de ajuste cambial, que foi descontinuado e substituído pelo processo padrão da Microsoft com as adaptações descritas acima.

### 8.2. Opção para Desabilitar o Ajuste em Adiantamentos Brasileiros

Quando uma empresa registra um adiantamento em moeda estrangeira, o valor em moeda local é fixado pela taxa do dia do pagamento. Contabilmente, esse valor não deve oscilar depois: o adiantamento já está liquidado financeiramente, e o que falta é apenas a baixa contra um título futuro. Empresas que adotam esse entendimento podem desejar que a rotina **Ajustar Taxas de Câmbio** ignore adiantamentos brasileiros.

A localização oferece um controle por módulo (Vendas e Compras), independentes:

**Configuração no módulo de Vendas**

1. Abra **Conf. Vendas & Cobranças** (`Sales & Receivables Setup`) via Diga-me (`Alt+Q`).
2. Localize o agrupamento **Fiscal**.
3. Marque ou desmarque o campo **Ajustar Taxa de Câmbio de Adiantamento** (`AdjustPrepmtExchangeRateFNX`).

**Configuração no módulo de Compras**

1. Abra **Conf. Compras & Pagamentos** (`Purchases & Payables Setup`) via Diga-me.
2. Localize o agrupamento **Fiscal** → subgrupo **Pagamentos**.
3. Marque ou desmarque o campo **Ajustar Taxa de Câmbio de Adiantamento** (`AdjustPrepmtExchangeRateFNX`).

**Comportamento**

| Estado da opção | Efeito sobre adiantamentos brasileiros |
|-----------------|-----------------------------------------|
| **Desativada** | Movimentos marcados como adiantamento são ignorados pela rotina; o valor em moeda local permanece igual ao originalmente registrado |
| **Ativada** | Adiantamentos brasileiros são processados como qualquer outro movimento em aberto, gerando ganho/perda não realizada conforme a variação da taxa (comportamento histórico do Business Central padrão) |

Demais movimentos (faturas em aberto, devoluções, contas bancárias, adiantamentos padrão da Microsoft) continuam sendo ajustados normalmente, independentemente do estado da opção.

**Valores iniciais**

Para evitar mudança silenciosa de comportamento em bases já em produção, a opção tem valores iniciais distintos conforme o cenário:

| Cenário | Valor inicial | Efeito |
|---------|---------------|--------|
| **Instalação nova** da localização | Desativada (`false`) | Adiantamentos brasileiros não recebem ajuste cambial desde o primeiro uso |
| **Upgrade** de versão anterior | Ativada (`true`) | Comportamento histórico preservado; nenhuma mudança automática nos próximos fechamentos |

Empresas que estavam em produção antes desta atualização e desejam aderir ao novo comportamento contábil devem desmarcar manualmente o campo nas duas páginas de configuração após o upgrade. A mudança passa a valer a partir da próxima execução do **Ajustar Taxas de Câmbio**.

> Atenção ao timing: desmarcar a opção no meio de um período fiscal pode deixar o saldo contábil incoerente entre adiantamentos já ajustados em meses anteriores e adiantamentos não ajustados nos meses seguintes. Recomenda-se desmarcar no início de um exercício ou após reverter ajustes anteriores feitos sobre adiantamentos.

---

## 9. Lista de Verificação — Quando a Variação Não Está Correta

Quando os valores apurados pelo processo não correspondem ao esperado, percorra os seguintes pontos antes de abrir um chamado:

1. **Taxa de câmbio na data**: confirme que a taxa cadastrada na **Valor da Taxa de Conversão** para a Data Registro está correta e atualizada.
2. **Data Registro e período**: verifique se a Data Registro e o intervalo Data Inicial / Data Final do relatório cobrem os movimentos esperados. Movimentos fora do período não são ajustados.
3. **Contas contábeis na Moeda**: confirme que as seis contas (Ganhos/Perdas Não Realiz., Ganhos/Perdas Realiz., Aj. Residual Ganho/Perda) estão configuradas no Cartão Moeda.
4. **Grupos Contábeis**: verifique se as contas de Conta Clientes, Conta Fornecedor e a conta do Grupo Contábil de Conta Bancária estão corretamente preenchidas.
5. **Saldo pendente**: confira se o movimento em aberto realmente possui saldo pendente em moeda estrangeira (Valor Pendente e Valor Pendente (ML) no movimento de cliente ou fornecedor).
6. **Fator de ajuste de moeda**: no movimento do cliente ou fornecedor, confira o fator de ajuste já aplicado em ajustes anteriores. Ele indica a última taxa utilizada e ajuda a entender por que a diferença incremental do ajuste atual é o que é.
7. **Histórico de ajustes anteriores**: consulte o histórico de ajustes de moedas para descartar ajuste duplicado ou um ajuste anterior que já cobriu a mesma data.
8. **Opção Ajustar por Movimento**: se marcada, cada registro gera lançamento individual; se desmarcada, os valores são agrupados por grupo contábil e moeda. Isso afeta a forma de ler os lançamentos gerados.
9. **Movimentos detalhados**: abra **Movs. Detalhados Clientes** ou Movs. Detalhados Fornecedores e filtre pelos tipos "Ganho não Realizado" e "Perda não Realizada" para identificar exatamente os ajustes gerados em cada movimento.
10. **Adiantamentos (Loc BR)**: se o registro é um adiantamento, verifique se a conta de adiantamento do grupo contábil está configurada. Sem isso, o processo falha.

---

## 10. Como Estornar uma Variação Cambial

O Business Central não possui um processo dedicado de "desfazer ajuste cambial". Para anular uma variação já registrada, o procedimento é:

1. Acesse a página **Registros Contábeis** (G/L Registers) e localize o registro gerado pelo ajuste de câmbio. Identifique-o pelo **Nº Documento** usado no relatório e pelo código-fonte de ajuste de câmbio.
2. Use a ação **Reverter Registro** (ou Reverter Transação, se quiser reverter uma transação específica do registro). Isso gera lançamentos contábeis inversos (espelhados) que anulam o efeito do ajuste original.
3. Após a reversão, se necessário, corrija a taxa de câmbio cadastrada e execute novamente o relatório **Ajustar Taxas de Câmbio** com os parâmetros corretos.

> A reversão pelo Registro Contábil reverte os lançamentos do razão geral. Os movimentos detalhados de cliente e fornecedor do tipo Ganho/Perda Não Realizada também são revertidos automaticamente pelo processo de reversão, mantendo os saldos consistentes.
