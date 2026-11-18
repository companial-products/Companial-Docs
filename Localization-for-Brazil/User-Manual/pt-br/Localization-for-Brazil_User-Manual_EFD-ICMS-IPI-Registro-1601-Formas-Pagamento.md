<!-- PR: #293 -->
# EFD ICMS/IPI: Registro 1601 por Forma de Pagamento

**Data**: Outubro/2026

---

## Sumário

O Registro 1601 da EFD ICMS/IPI informa o total das vendas recebidas por instrumento de pagamento eletrônico, separado por instituição de pagamento. A apuração de ICMS/IPI gera o 1601 só com os recebimentos das notas cuja **Forma de Pagamento** tem a opção **Incluir no Registro 1601 da EFD** marcada. Cada empresa escolhe quais formas entram, e uma conta bancária só precisa de instituição de pagamento cadastrada quando recebe notas dessas formas.

---

## 1. Contexto

### O que é o Registro 1601

O Registro 1601 traz, por instituição de pagamento (a credenciadora do cartão, por exemplo), o total das vendas recebidas por meio de cartão de crédito, cartão de débito, PIX e outros instrumentos de pagamento eletrônico. O valor é separado em vendas com ICMS, vendas de serviço e outras vendas.

O sistema gera o 1601 na apuração de ICMS/IPI a partir dos recebimentos de clientes lançados no período e ligados a uma nota fiscal. Entram só as notas cuja forma de pagamento está marcada em **Incluir no Registro 1601 da EFD**. Cada recebimento é atribuído à instituição cadastrada na conta bancária que recebeu o dinheiro, no campo **Nº Participante EFD** (`EFDParticipantNoFNX`).

### Quando se aplica

A regra vale para as empresas e filiais com **Gerar Registro 1601** (`Generate1601Record`) marcado na **Configuração EFD ICMS/IPI** (`EFDICMSIPISetupFNX`), grupo **Registros**, ou no **Perfil por Filial** (`EFDProfileByBranchFNX`) da filial. Sem essa opção, o Registro 1601 não é gerado e a marcação das formas de pagamento não tem efeito.

---

## 2. Configuração

1. Navegue até **Configuração EFD ICMS/IPI** (página `EFDICMSIPISetupFNX`) e confirme que **Gerar Registro 1601** está marcado no grupo **Registros**. Quando a empresa usa **Perfil por Filial**, a opção é configurada na linha de cada filial.
2. Cadastre cada instituição de pagamento (a credenciadora do cartão, por exemplo) em **Participantes EFD** (página `EFDParticipants`).
3. Abra o **Cartão da Conta Bancária** de cada conta que recebe vendas por pagamento eletrônico e preencha o **Nº Participante EFD** com a instituição de pagamento correspondente. As contas que recebem só boleto, TED, depósito e outras formas que não entram no 1601 podem ficar sem esse campo.
4. Navegue até **Formas Pagamento** (página `Payment Methods`) e marque **Incluir no Registro 1601 da EFD** (`IncludeInEFD1601FNX`) nas formas de pagamento cujas notas devem entrar no 1601, como cartão de crédito, cartão de débito ou PIX. A coluna fica ao lado de **Código NF-e** (`NFeCodeFNX`).

A escolha é de cada empresa. O sistema não decide pelo **Código NF-e** da forma de pagamento: uma forma com código de cartão de crédito que está desmarcada fica de fora do 1601, e uma forma com qualquer código que está marcada entra.

Toda forma de pagamento nova começa com **Incluir no Registro 1601 da EFD** desmarcado. Marque a opção se as notas emitidas com ela devem entrar no 1601.

---

## 3. Comportamento na Apuração

### Quais recebimentos entram no 1601

Ao executar **Processar** (`Process`) na **Apuração ICMS/IPI** (`ICMSIPISettlementFNX`), o sistema percorre os recebimentos de clientes do período ligados a uma nota fiscal e decide, pela forma de pagamento da nota, se cada um entra no 1601:

| Forma de pagamento da nota | Resultado |
|---|---|
| Marcada em **Incluir no Registro 1601 da EFD** | Gera linha no 1601 |
| Desmarcada | Não gera linha, mesmo que a conta bancária tenha **Nº Participante EFD** |
| Nota sem forma de pagamento | Não gera linha |
| Forma de pagamento excluída depois do registro da nota | Não gera linha |

A forma de pagamento considerada é a da nota fiscal: a mesma que vai no XML da NF-e, informada no **Cód. Forma de Pagamento** do documento de venda. A forma como o cliente efetivamente pagou não é considerada. Uma nota emitida com boleto e paga com cartão segue a marcação da forma de pagamento do boleto.

### Como a linha é preenchida

| Campo da linha | Origem |
|---|---|
| **Nº Conta Bancária** (`BankAccountNo`) | Conta bancária que recebeu o pagamento |
| **Nº Participante** (`ParticipantNo`) | **Nº Participante EFD** cadastrado nessa conta bancária |
| **Data Registro** (`PostingDate`) | Data do recebimento no banco |
| **Vendas ICMS** (`ICMSSales`) | Valor recebido no banco, quando a nota é de modelo de ICMS (NF-e, por exemplo) |
| **Vendas Serviço** (`ServiceSales`) | Valor recebido no banco, quando a nota é de serviço |
| **Outras Vendas** (`OtherSales`) | Valor recebido no banco, nos demais casos |

No arquivo, as linhas são somadas por instituição de pagamento: cada instituição gera um Registro 1601. Uma conta que recebe de mais de uma instituição (duas credenciadoras na mesma conta, por exemplo) tem todos os valores atribuídos à instituição cadastrada na conta.

### Conta bancária sem instituição de pagamento

Uma conta bancária sem **Nº Participante EFD** só é checada quando recebe uma nota de forma marcada:

- **Recebeu só notas de formas desmarcadas, ou sem forma de pagamento**: a conta não gera erro e a apuração é processada até o fim. Não é preciso cadastrar uma instituição para essa conta.
- **Recebeu uma nota de forma marcada**: a apuração para com a mensagem:

  > O documento fiscal *&lt;nº do documento&gt;* (nº da nota *&lt;nº da nota&gt;*) tem a forma de pagamento *&lt;forma&gt;*, que entra no Registro 1601, e foi recebido na conta bancária *&lt;conta&gt;*, que não tem Nº Participante EFD. Preencha Nº Participante EFD na conta bancária ou desmarque Incluir no Registro 1601 da EFD na forma de pagamento *&lt;forma&gt;*.

  A mensagem traz a ação **Mostrar Conta Bancária**, que abre o **Cartão da Conta Bancária** da conta citada. Corrija um dos dois cadastros e processe a apuração de novo:
  - se a conta recebe vendas por pagamento eletrônico, preencha o **Nº Participante EFD** com a instituição de pagamento;
  - se a forma de pagamento não é de pagamento eletrônico, desmarque **Incluir no Registro 1601 da EFD** na forma.

A mesma verificação acontece quando o 1601 é refeito pelo **Processar** da **Apuração PIS/COFINS** (`PISCOFINSSettlementFNX`).

### Reprocessamento e linhas incluídas manualmente

- A marcação vale a partir do próximo processamento da apuração, inclusive quando um mês anterior é reprocessado. Depois de marcar ou desmarcar uma forma de pagamento, processe a apuração de novo para o 1601 refletir a mudança.
- Ao reprocessar, só as linhas criadas pelo sistema são refeitas. As linhas incluídas manualmente na página **Transações com Instrumentos de Pagamentos Eletrônicos (1601)** continuam, qualquer que seja a forma de pagamento. Uma linha manual precisa do **Nº Participante** preenchido.

### Indicador de operações com cartão no Registro 1010

O indicador `IND_CART` do Registro 1010 segue as linhas do 1601 da apuração:

- sem nenhuma linha (nenhuma nota de forma marcada recebida no período e nenhuma linha manual), o 1601 não sai no arquivo e o `IND_CART` sai `N`;
- basta uma linha, criada pelo sistema ou incluída manualmente, para o `IND_CART` sair `S`.

### Outras apurações

A marcação vale só para o Registro 1601. As apurações de IRPJ/CSLL no Regime de Caixa e de PIS/COFINS no Regime de Caixa consideram todos os recebimentos, qualquer que seja a forma de pagamento da nota.

---

## 4. Como Consultar as Linhas do 1601

1. Abra a **Apuração ICMS/IPI** do período.
2. Clique na ação **Transações com Instrumentos de Pagamentos Eletrônicos (1601)** (página `EFD1601DocumentsFNX`). A ação aparece quando **Gerar Registro 1601** está marcado.
3. A coluna **Cod. Forma Pagamento** (`PaymentMethodCode`), depois de **Data Registro**, mostra a forma de pagamento da nota de cada linha. A coluna é só de consulta e pode ser filtrada.

A coluna também aparece nas linhas incluídas manualmente que têm o **Nº Documento** (`DocumentNo`) preenchido.

---

## 5. Exemplo Prático

A empresa recebe boletos e vendas no cartão de crédito na mesma conta bancária. A conta tem o **Nº Participante EFD** da credenciadora do cartão. Em **Formas Pagamento**, CARTAO está marcada em **Incluir no Registro 1601 da EFD** e BOLETO está desmarcada.

No mês, a conta recebe:

| Nota | Forma de pagamento da nota | Valor recebido | Entra no 1601? |
|---|---|---|---|
| 1001 | CARTAO (marcada) | R$ 109,56 | Sim: R$ 109,56 em **Vendas ICMS** |
| 1002 | BOLETO (desmarcada) | R$ 2.500,00 | Não |
| 1003 | BOLETO (desmarcada) | R$ 1.800,00 | Não |

O 1601 do mês sai com uma linha para a credenciadora, com R$ 109,56 em vendas com ICMS. Se BOLETO também estivesse marcada, as três notas entrariam, somando R$ 4.409,56.

---

## 6. Campos do Registro 1601

### Tabela Configuração EFD ICMS/IPI (`EFDICMSIPISetupFNX`) e Perfil por Filial (`EFDProfileByBranchFNX`)

| Campo | Tipo | Descrição |
|---|---|---|
| **Gerar Registro 1601** (`Generate1601Record`) | Booleano | Liga a geração do Registro 1601 na apuração e no arquivo. No **Perfil por Filial**, vale para a filial da linha. |

### Tabela Conta Bancária (`Bank Account`)

| Campo | Tipo | Descrição |
|---|---|---|
| **Nº Participante EFD** (`EFDParticipantNoFNX`) | Código | Instituição de pagamento da conta, escolhida em **Participantes EFD**. Os recebimentos da conta que entram no 1601 são atribuídos a ela. Obrigatório só nas contas que recebem notas de forma de pagamento marcada em **Incluir no Registro 1601 da EFD**. |

### Tabela Forma de Pagamento (`Payment Method`)

| Campo | Tipo | Descrição |
|---|---|---|
| **Incluir no Registro 1601 da EFD** (`IncludeInEFD1601FNX`) | Booleano | Indica se os recebimentos das notas emitidas com esta forma de pagamento entram no Registro 1601, na instituição de pagamento da conta bancária que os recebeu. Começa desmarcado em toda forma de pagamento nova. Editável na página **Formas Pagamento**. |

### Tabela Documento EFD 1601 (`EFD1601DocumentFNX`)

São as colunas da página **Transações com Instrumentos de Pagamentos Eletrônicos (1601)**. A apuração preenche as linhas criadas pelo sistema; nas linhas incluídas manualmente, o usuário preenche os campos.

| Campo | Tipo | Descrição |
|---|---|---|
| **Nº Conta Bancária** (`BankAccountNo`) | Código | Conta bancária que recebeu o pagamento. |
| **Nº Participante** (`ParticipantNo`) | Código | Instituição de pagamento da linha, informada no `COD_PART_IP` do Registro 1601. Obrigatório. |
| **Nº Documento** (`DocumentNo`) | Código | Documento fiscal de origem da linha. |
| **Nº Nota Fiscal** (`FiscalNo`) | Código | Número da nota. |
| **Nº Cliente** (`CustomerNo`) | Código | Cliente da nota. |
| **Nome Cliente** (`CustomerName`) | Texto (somente leitura) | Nome do cliente, exibido a partir da ficha do cliente. |
| **Data Registro** (`PostingDate`) | Data | Data do recebimento no banco. |
| **Cod. Forma Pagamento** (`PaymentMethodCode`) | Código (somente leitura) | Forma de pagamento da nota. A opção **Incluir no Registro 1601 da EFD** dessa forma decide se a apuração cria a linha. |
| **Vendas ICMS** (`ICMSSales`) | Decimal | Valor das vendas com ICMS recebidas pela instituição, somado no `TOT_VS`. |
| **Vendas Serviço** (`ServiceSales`) | Decimal | Valor das vendas de serviço recebidas pela instituição, somado no `TOT_ISS`. |
| **Outras Vendas** (`OtherSales`) | Decimal | Valor das demais vendas recebidas pela instituição, somado no `TOT_OUTROS`. |
