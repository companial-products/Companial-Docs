# Jurisdições, Áreas de Impostos e Contabilizações

Este é o capítulo mais importante da configuração fiscal. Antes de entender CFOP, NCM e matrizes, é necessário entender o **modelo tributário** que a localização usa para organizar os impostos: **Jurisdições Impostos**, **Áreas de Impostos** e **Contabilizações de Imposto**.

Esses três conceitos trabalham juntos para responder a três perguntas:
1. **Quais impostos existem?** (Jurisdições)
2. **Quais impostos se aplicam a esta operação?** (Áreas de Impostos)
3. **Como cada imposto afeta a contabilidade?** (Contabilizações)

---

## Jurisdição Imposto (Tax Jurisdiction)

### O que é

Uma **Jurisdição Imposto** (Tax Jurisdiction) representa um imposto específico no sistema. Cada imposto que a localização precisa calcular, escriturar e contabilizar tem uma jurisdição própria.

Pense na jurisdição como o "cadastro do imposto" — ela identifica o imposto e define seu comportamento padrão.

### Campos principais

| Campo | O que define |
|-------|-------------|
| **Código** | Identificador da jurisdição (ex: ICMS, IPI, PIS, COFINS, ISS) |
| **Descrição** | Nome descritivo |
| **Classificação Imposto** (Tax Identification) | Qual imposto é — um valor do enum que identifica o tipo (ICMS, IPI, PIS, etc.). Este é o campo mais importante: é ele que conecta a jurisdição a todo o restante do sistema |
| **Cód. Contabilização Imposto** (Tax Posting Code, padrão) | Qual configuração de contabilização usar por padrão para este imposto |

### Classificação Imposto — O eixo central

O campo **Classificação Imposto** (Tax Identification) é o conceito mais fundamental da localização. Ele aparece em praticamente toda tabela de impostos do sistema. É por ele que o sistema sabe:

- Que uma linha de imposto se refere a ICMS (e não a PIS)
- Quais contas contábeis usar (via Contas Impostos)
- Quais registros do EFD gerar
- Qual apuração alimentar

Os principais valores:

| Classificação Imposto | Imposto |
|-------------------|---------|
| ICMS | ICMS próprio |
| ST | ICMS Substituição Tributária |
| Dif. Aliq. ICMS | DIFAL |
| IPI | Imposto sobre Produtos Industrializados |
| PIS | PIS |
| COFINS | COFINS |
| ISS | ISS |
| II | Imposto de Importação |
| IRRF | Imposto de Renda Retido na Fonte |
| INSS | Previdência Social |
| INSS Ret. | INSS Retido na Fonte |
| CSL Ret. | CSLL Retida |
| PIS Ret. | PIS Retido |
| COFINS Ret. | COFINS Retida |
| ISS Ret. | ISS Retido |
| IBSUF | IBS Estadual (Reforma Tributária) |
| IBSMun | IBS Municipal (Reforma Tributária) |
| CBS | Contribuição sobre Bens e Serviços (Reforma Tributária) |
| IS | Imposto Seletivo (Reforma Tributária) |

**Caminho no BC**: Diga-me > "Jurisdições Impostos"

---

## Área Impostos (Tax Area)

### O que é

Uma **Área Impostos** (Tax Area) é um **agrupamento de jurisdições** — ou seja, é a lista de impostos que se aplicam a uma determinada situação.

Se a jurisdição é "qual imposto existe", a Área Impostos é "quais impostos se aplicam aqui".

### Como funciona

Cada Área Impostos contém **linhas**, e cada linha aponta para uma jurisdição. Exemplo:

**Exemplo: Área "VEND" (venda de mercadoria ao mercado interno)**:

| Jurisdição | Ordem Cálculo | Contabilização Imposto |
|------------|:------------:|----------------------|
| IPI | 1 | IPI/ST VENDA |
| PIS | 2 | IMP VENDA |
| COFINS | 3 | IMP VENDA |
| ICMS | 4 | IMP VENDA |
| DIFAL | 5 | IMP VENDA |
| ST | 6 | IPI/ST VENDA |
| CBS | 7 | IBS-CBS-VENDA |
| IBS-MUN | 7 | IBS-CBS-VENDA |
| IBS-UF | 7 | IBS-CBS-VENDA |

### Ordem de Cálculo

Cada linha da Área tem um campo **Ordem Cálculo** que define a sequência em que os impostos são processados. Isso importa quando um imposto afeta a base de cálculo de outro.

No exemplo acima:
- **IPI é calculado primeiro** (ordem 1) — porque o IPI pode compor a base do ICMS
- **PIS e COFINS** vêm em seguida (ordens 2 e 3)
- **ICMS** é calculado depois (ordem 4) — já com o IPI resolvido
- **DIFAL e ST** dependem do ICMS próprio (ordens 5 e 6)
- **IBS/CBS** são calculados por último (ordem 7) — a base pode ser afetada pelos impostos tradicionais (tema ainda em discussão na regulamentação)

Se a ordem estiver errada, um imposto pode ser calculado antes de outro que deveria afetar sua base, gerando valores incorretos.

### Onde a Área Impostos é atribuída

A Área é atribuída nas **linhas dos documentos** (ordens de venda, ordens de compra, faturas). Ela pode vir de:

- Cadastro do **cliente** (vendas) ou do **fornecedor** (compras)
- Configuração via **matriz de impostos** (automaticamente conforme a operação)
- Atribuição manual na linha

### O ponto-chave

A Área Impostos determina **quais impostos serão calculados** para aquela linha e **em que ordem**. Cada linha da Área também define qual **Contabilização Imposto** será usada para aquele imposto — essa é a configuração definitiva que determina o comportamento contábil.

**Caminho no BC**: Diga-me > "Áreas de Impostos"

---

## Contabilização Imposto (Tax Posting)

### O que é

A **Contabilização Imposto** (Tax Posting) define **como o imposto afeta o sistema financeiro e contábil**. É aqui que se configura se o imposto:

- Entra no custo do item
- Altera o valor a pagar/receber
- Gera imposto a recolher
- Gera despesa
- Gera crédito fiscal recuperável

### Os flags — o coração da Contabilização Imposto

Cada Contabilização Imposto tem um conjunto de **flags** (opções) divididos em dois momentos:

#### No faturamento (Registro)

| Flag | O que faz | Exemplo |
|------|-----------|---------|
| **Custo/Preço Item** | Adiciona ou subtrai o imposto do custo/preço do item | IPI na compra → "Somar" (entra no custo) |
| **Financeiro** | Altera o valor financeiro (conta a pagar/receber) | IPI na venda → "Somar ao Total" (aumenta a duplicata) |
| **Imposto a Pagar** | Cria movimento de imposto a recolher | IPI na venda → ativo (gera IPI a pagar) |
| **Despesa** | Cria movimento de despesa | ICMS sem crédito → ativo (vira despesa) |
| **Crédito Imposto** | Cria movimento de crédito fiscal recuperável | ICMS na compra com crédito → ativo |
| **Estorno de Crédito** | Estorna um crédito fiscal | Estorno de crédito de ICMS |

#### No pagamento

Os mesmos conceitos de Financeiro, Imposto a Pagar, Despesa e Crédito Imposto, mas aplicados no momento do pagamento em vez do faturamento. Usado principalmente para **retenções na fonte** — IRRF, INSS Retido, PIS/COFINS/CSLL retidos — onde o efeito contábil ocorre quando o pagamento é feito, não quando a fatura é registrada.

### Exemplos práticos

| Cenário | Custo/Preço Item | Financeiro | Imposto a Pagar | Despesa | Crédito Imposto |
|---------|-----------------|------------|----------------|---------|----------------|
| **ICMS/PIS/COFINS compra com crédito** | Subtrair | — | — | — | Sim |
| **ICMS/PIS/COFINS compra sem crédito** | — | — | — | — | — |
| **ICMS/PIS/COFINS na venda** | — | — | Sim | Sim | — |
| **IPI com crédito na compra** | — | Somar ao Total | — | — | Sim |
| **IPI/ST sem crédito na compra** | Somar | Somar ao Total | — | — | — |
| **IPI/ST na venda** | Somar | Somar ao Total | Sim | Sim | — |
| **DIFAL** | Somar | — | Sim | — | — |
| **Retenção na compra** (faturamento) | — | Subtrair da 1a Parcela | Sim | — | — |
| **Retenção na venda** (faturamento) | — | Subtrair da 1a Parcela | — | — | Sim |
| **Retenção no pagamento** | — | Subtrair do Total* | Sim* | — | — |

*Os flags marcados com asterisco ficam na seção "Payment" da Contabilização Imposto — o efeito contábil ocorre no pagamento, não no faturamento.*

> **Compra sem crédito de ICMS/PIS/COFINS** — todos os flags ficam vazios. Como esses impostos já estão **embutidos no preço** da nota (não há destaque a recuperar nem ajuste a fazer no custo), o sistema só precisa reconhecer o valor total da nota como custo do item e como obrigação financeira a pagar — comportamento que já é o padrão do BC, sem nenhum efeito adicional da Contabilização Imposto. A jurisdição existe apenas para que o imposto seja escriturado nos registros do EFD-ICMS/IPI ou EFD-Contribuições.

A lógica contábil é de **partidas dobradas** — cada flag representa um lado do movimento contábil. Por exemplo, na compra com crédito de ICMS:
- **Crédito Imposto = Sim** → débito na conta de ICMS a recuperar (ativo)
- **Custo/Preço Item = Subtrair** → crédito que reduz o custo do item (o ICMS será recuperado, então não é custo)

### Por que isso importa

A configuração da Contabilização Imposto é o que determina como os impostos são contabilizados. Uma configuração errada aqui gera:

- Custo de item incorreto
- Valor de duplicata errado
- Crédito fiscal que não deveria existir (ou que está faltando)
- Contas contábeis com saldo incorreto
- Apurações com valores errados

**Caminho no BC**: Diga-me > "Contabilizações de Imposto"

---

## Contas Impostos (Tax Posting Accounts)

### O que é

A tabela de **Contas Impostos** (Tax Posting Accounts) define **para quais contas contábeis** os valores de impostos vão. É a ligação entre o imposto e o plano de contas.

### Como funciona

Cada registro associa uma **Classificação Imposto** (qual imposto) a um conjunto de contas:

| Conta | Quando é usada |
|-------|---------------|
| **Imposto a Pagar** (Payable) | Quando o flag "Imposto a Pagar" está ativo |
| **Despesa** (Expense) | Quando o flag "Despesa" está ativo |
| **Crédito Imposto** (Tax Credit) | Quando o flag "Crédito Imposto" está ativo |
| **Estorno de Crédito** (Credit Reversal) | Quando o flag "Estorno de Crédito" está ativo |

### Filtros — contas diferentes por contexto

O mesmo imposto pode usar contas diferentes dependendo do contexto. A tabela suporta filtros:

| Filtro | Permite | Exemplo |
|--------|---------|---------|
| **Jurisdição** | Conta diferente por jurisdição | ICMS de SP usa uma conta, ICMS de MG usa outra |
| **Fornecedor** | Conta diferente por fornecedor | Fornecedor específico tem conta de retenção separada |
| **Cliente** | Conta diferente por cliente | — |
| **Grupo Registro Produto** | Conta por tipo de produto | Mercadoria vs. serviço |
| **Grupo Registro Negócios** | Conta por tipo de parceiro | — |

O sistema busca a conta na ordem mais específica (jurisdição) para a mais genérica (default). Se não encontrar em nenhum nível, dá erro.

**Caminho no BC**: Diga-me > "Contas Impostos" ou dentro do cartão da Contabilização Imposto

---

## Como tudo se conecta

O fluxo completo, da configuração até o movimento contábil:

```
1. CONFIGURAÇÃO
   Jurisdição define: qual imposto + Contabilização padrão
   Área Impostos agrupa: quais jurisdições se aplicam
   Contabilização Imposto define: como o imposto se comporta (flags)
   Contas Impostos define: para quais contas contábeis

2. DOCUMENTO
   Linha do pedido recebe uma Área Impostos
   → Sistema sabe quais impostos calcular
   → Cada imposto usa a Contabilização definida na Área Impostos
   → Impostos são calculados

3. REGISTRO
   Ao registrar, para cada imposto calculado:
   → Lê os flags da Contabilização Imposto
   → Aplica efeito no custo, financeiro, crédito, despesa
   → Busca contas nas Contas Impostos
   → Gera movimentos contábeis

4. ESCRITURAÇÃO
   Livros fiscais leem os impostos pela Classificação Imposto
   → EFD-ICMS/IPI busca ICMS, IPI, ST
   → EFD-Contribuições busca PIS, COFINS
   → Apurações agrupam débitos e créditos
```

---

## Impacto na escrituração fiscal

A **Classificação Imposto** (Tax Identification) é o eixo que conecta a configuração fiscal à escrituração. Cada módulo de livros fiscais filtra os dados pela classificação:

| Obrigação | Classificações relevantes |
|-----------|--------------------------|
| **EFD-ICMS/IPI** | ICMS, IPI, ST, DIFAL |
| **EFD-Contribuições** | PIS, COFINS |
| **EFD-Reinf** | INSS, INSS Ret., IRRF, CSL Ret. |
| **Apuração PIS/COFINS** | PIS, COFINS |
| **Apuração ISS** | ISS |
| **DCTF** | Todos os federais |

Se a Classificação Imposto de uma jurisdição estiver errada, os valores podem aparecer na obrigação acessória errada — ou simplesmente não aparecer.

---

## Pontos de atenção

- **Contabilização Imposto errada** = contabilização incorreta. Este é o erro mais comum e mais difícil de detectar — os valores parecem corretos, mas as contas estão erradas.
- **Área Impostos incompleta** = imposto não calculado. Se ISS não está na área, o sistema simplesmente não calcula ISS.
- **Contas Impostos sem registro** = erro ao registrar. O sistema exige que as contas estejam configuradas para cada Classificação Imposto usada.
- **Flags de Registro vs Pagamento** — retenções na fonte geralmente usam flags de Pagamento. Se configurar em Registro, o efeito será no faturamento em vez de no pagamento.
- **Crédito fiscal indevido** — se o flag "Crédito Imposto" está ativo em uma operação sem direito a crédito, a apuração ficará errada.

---

## Para praticar

1. Abra a lista de **Jurisdições Impostos** e identifique quais impostos estão cadastrados. Confira a Classificação Imposto de cada uma.
2. Abra uma **Área Impostos** e veja quais jurisdições ela agrupa. Tente encontrar a área usada para vendas de mercadorias e a de serviços.
3. Abra uma **Contabilização Imposto** e leia os flags. Tente entender: este imposto entra no custo? Altera o financeiro? Gera crédito?
4. Abra as **Contas Impostos** de ICMS e veja quais contas estão configuradas.
5. **Exercício mental**: na compra de uma mercadoria com direito a crédito de ICMS — quais flags devem estar ativos na Contabilização Imposto? (Resposta: Crédito Imposto = Sim, os demais vazios/não — o ICMS não entra no custo porque é recuperável)

**Referência**: consulte o manual "Guia de Impostos" disponível na pasta Outros Manuais.
