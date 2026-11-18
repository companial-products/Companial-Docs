<!-- Source: manual de referência (Cálculo Imposto Customizado) -->
# Cálculo de Imposto Customizado

**Data**: Maio/2026

---

## Sumário

O **Cálculo Imposto Customizado** (`CustomTaxCalculationFNX`) permite criar fórmulas de cálculo de impostos personalizadas para cenários fiscais que fogem do cálculo padrão da localização. Ele funciona como uma "planilha de cálculo" em que é possível definir, linha a linha, variáveis, operações matemáticas e os campos de destino que receberão os resultados, assumindo controle total sobre os valores finais do imposto.

---

## 1. Conceito

O cálculo customizado é acionado no momento final do cálculo de imposto, onde normalmente seriam aplicadas as fórmulas padrão de ST, MVA ajustada, FCP embutido etc. Quando uma linha de imposto possui um **Cód. Cálculo Imposto Customizado** associado, o motor de cálculo padrão executa normalmente a montagem da base (inclusão de IPI, alterações de base, fator de redução etc.), mas desvia para a fórmula customizada ao calcular os valores finais.

Ao ativar o cálculo customizado para uma linha de imposto, o sistema pula automaticamente os seguintes comportamentos padrão:

- Ajuste automático de MVA
- Inclusão automática de FCP na alíquota do ICMS
- Cálculo padrão de ST (alíquota interna × base − ICMS próprio)
- Cálculo automático de desoneração

Isso dá ao consultor liberdade total para montar a lógica de cálculo exatamente como a legislação exige.

---

## 2. Estrutura da Fórmula

Cada **Cálculo Imposto Customizado** (`CustomTaxCalculationFNX`) é composto por um cabeçalho (código e descrição) e por linhas sequenciais (**Linha de Cálculo Imposto Customizado**, `CustomTaxCalculationLineFNX`). Cada linha possui os seguintes campos:

| Campo | Descrição |
|-------|-----------|
| **Variável** | Nome que armazena o resultado do passo (ex.: `BASE`, `ICMS_ST`, `FCP`). Pode ser referenciado como operando em passos posteriores |
| **Operação** | A operação matemática a executar (ver tabela abaixo) |
| **Operando 1** | Primeiro valor de entrada. Pode ser uma variável definida em linha anterior ou um valor literal |
| **Operando 2** | Segundo valor de entrada. Pode ser uma variável definida em linha anterior ou um valor literal |
| **Nome Campo Associado** | Campo da **Linha Valor Imposto** (`TaxAmountLineFNX`) que receberá o resultado. Quando preenchido, o valor calculado é gravado diretamente nesse campo (ex.: **Valor**, **Valor Base**, **Valor FCP**) |

As linhas são processadas na ordem do número de linha. Uma linha sem **Operação** (em branco) serve para ler o valor atual de um campo da **Linha Valor Imposto** e armazená-lo numa variável. Ela funciona como ponto de partida para capturar valores já calculados pelo motor padrão, como a base de cálculo.

### Operações disponíveis

| Operação | Fórmula |
|----------|---------|
| **Somar** | Operando 1 + Operando 2 |
| **Subtrair** | Operando 1 − Operando 2 |
| **Multiplicar** | Operando 1 × Operando 2 |
| **Dividir** | Operando 1 ÷ Operando 2 |
| **Calcular Percentual** | Operando 1 × Operando 2 ÷ 100 |
| **Embutir Percentual** | Operando 1 ÷ (1 − Operando 2 ÷ 100), o cálculo "por dentro" |
| **Zerar** | Resultado = 0 |
| **Arredondar** | Arredonda o valor da variável para 2 casas decimais |

---

## 3. Onde Configurar

O **Cód. Cálculo Imposto Customizado** pode ser associado em três locais, em ordem crescente de prioridade (o mais específico prevalece):

1. **Jurisdição Imposto** (`TaxJurisdictionFNX`), no campo **Cód. Cálculo Imposto Customizado** do cartão da jurisdição. Aplica-se a todos os documentos que usam essa jurisdição.

2. **Alíquotas Cód. NCM** (`NCMCodeTaxPercentFNX`), no campo **Cód. Cálculo Imposto Customizado** da página **Alíquotas Cód. NCM**. Sobrescreve o valor da jurisdição quando o NCM do item corresponde à configuração.

3. **Exceções Imposto** (`TaxExceptionFNX`), no campo **Cód. Cálculo Imposto Customizado** da página **Exceções Imposto**. Sobrescreve os dois anteriores quando a exceção é aplicável.

Para criar e manter as fórmulas, pesquise por **Cálculos de Imposto Customizados** na barra de busca do BC. Ali você cria o cabeçalho e define as linhas de cálculo.

---

## 4. Exemplo Prático — Venda de SP para AL com FCP 1% e ST

**Cenário**: empresa em SP vende mercadoria para AL. O ICMS interestadual é 7%, a alíquota interna de AL é 19%, o MVA é 40%, e há FCP de 1%.

Supondo que o motor padrão já montou a base de cálculo (valor da mercadoria + IPI, se aplicável) e que as alíquotas e o MVA já estão configurados, o cálculo customizado seria estruturado assim:

| Linha | Variável | Operação | Operando 1 | Operando 2 | Nome Campo Associado |
|-------|----------|----------|------------|------------|----------------------|
| 10000 | ALQ_INTER | *(em branco)* | | | `Tax Percent` |
| 20000 | ALQ_INT | *(em branco)* | | | `Internal Percentage` |
| 30000 | ALQ_FCP | *(em branco)* | | | `FCP Percent` |
| 40000 | MARGEM | *(em branco)* | | | `Margin Percent Value` |
| 50000 | BASE | *(em branco)* | | | `Tax Base Amount` |
| 60000 | MVA | Calcular Percentual | `BASE` | `MARGEM` | |
| 70000 | BASE_ST | Somar | `BASE` | `MVA` | `Tax Base Amount` |
| 80000 | ICMS_INT | Calcular Percentual | `BASE_ST` | `ALQ_INT` | |
| 90000 | ICMS_PROP | Calcular Percentual | `BASE` | `ALQ_INTER` | |
| 100000 | ST | Subtrair | `ICMS_INT` | `ICMS_PROP` | `Tax Amount` |
| 110000 | FCP | Calcular Percentual | `BASE_ST` | `ALQ_FCP` | `FCP Amount` |

**O que cada bloco faz:**

- Linhas **10000–50000**: capturam os valores já calculados pelo motor padrão (alíquotas e base de cálculo) e os armazenam em variáveis para uso nas etapas seguintes.
- Linhas **60000–70000**: calculam o MVA sobre a base e obtêm o **Valor Base** de ST. O resultado é gravado no campo `Tax Base Amount` da **Linha Valor Imposto**.
- Linhas **80000–100000**: calculam o ICMS interno sobre a base ST e o ICMS próprio sobre a base original, e subtraem um do outro para obter o ICMS-ST. O resultado é gravado no campo `Tax Amount`.
- Linha **110000**: calcula o FCP (1% sobre a base ST) e grava no campo `FCP Amount`.

> **Nota**: este é um exemplo didático. A fórmula real pode variar conforme o cenário específico (MVA ajustada, reduções de base etc.). Cada variável usada como operando deve ter sido definida em uma linha anterior da fórmula.

---

## 5. Verificar o Cálculo no Documento

É possível inspecionar o log de execução de um cálculo customizado diretamente no documento. O sistema grava automaticamente o valor de cada variável após cada passo da fórmula.

**Para visualizar:**

1. Abra as **Estatísticas** do documento (pedido de venda, fatura de compra etc.).
2. Na aba de impostos, selecione a linha da **Linha Valor Imposto** que utilizou cálculo customizado.
3. Clique na ação **Mostrar Info. Cálculo**.

O sistema exibe uma mensagem com todas as variáveis e seus valores calculados, por exemplo:

```
BASE = 1000,00
MVA = 400,00
BASE_ST = 1400,00
ICMS_INT = 266,00
ICMS_PROP = 70,00
ST = 196,00
FCP = 14,00
```

A ação **Mostrar Info. Cálculo** só fica habilitada quando a linha de imposto efetivamente utilizou um cálculo customizado. Ela está disponível tanto nas **Estatísticas** dos documentos de venda e compra quanto na visualização de impostos do Documento Fiscal.
