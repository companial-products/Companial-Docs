# ECF — Escrituração Contábil Fiscal

A ECF é a declaração anual que demonstra o cálculo do IRPJ e da CSLL. Ela parte dos dados da ECD (escrituração contábil) e detalha como a empresa chegou ao lucro tributável.

---

## O que é

A ECF substituiu a antiga DIPJ e é obrigatória para todas as empresas tributadas pelo lucro real, presumido ou arbitrado. Ela contém:

- **Plano de contas** — mapeado para o plano referencial da Receita Federal
- **Balanço patrimonial** — referencial
- **e-LALUR** (Livro de Apuração do Lucro Real) — adições, exclusões e compensações do IRPJ
- **e-LACS** (Livro de Apuração da CSLL) — mesma lógica para CSLL
- **Cálculo final** — deduções de IRPJ e CSLL
- **Informações complementares** — sócios, dividendos, impostos retidos

### Formato

Arquivo texto SPED (tipo `LECF`), separador pipe, transmitido anualmente.

---

## Estrutura de blocos

| Bloco | Nome | Conteúdo principal |
|-------|------|-------------------|
| **0** | Abertura | Parâmetros gerais, regime de tributação, indicadores da empresa |
| **J** | Plano de Contas | Plano de contas da ECD com mapeamento referencial |
| **L** | Lucro Líquido | Balanço patrimonial, avaliação de estoque, composição de custos |
| **M** | e-LALUR / e-LACS | O bloco mais complexo — Parte A (ajustes ao lucro) e Parte B (contas de controle) |
| **N** | Cálculo | Deduções do IRPJ e CSLL apurados |
| **Y** | Informações Gerais | Sócios, impostos retidos, receitas |

### Dependência da ECD

Para quem é obrigado à ECD, a ECF **depende dela**: o programa da ECF recupera da ECD transmitida o plano de contas e os saldos usados nos blocos J, L e M. Quem não é obrigado à ECD, como parte das empresas do Lucro Presumido, entrega a ECF sem essa recuperação.

---

## O e-LALUR e o e-LACS (Bloco M)

O bloco M é o coração da ECF para empresas no lucro real. Ele implementa digitalmente os livros de apuração:

### Parte A — Ajustes ao lucro

Para cada período (trimestral ou anual), lista:
- **Adições** — despesas contabilizadas mas não dedutíveis (multas, provisões, etc.)
- **Exclusões** — receitas contabilizadas mas não tributáveis
- **Compensações** — prejuízos fiscais de períodos anteriores

Cada ajuste pode ser vinculado a contas contábeis da ECD e a contas da Parte B.

### Parte B — Contas de controle

Contas que acumulam valores ao longo do tempo — por exemplo, prejuízos fiscais a compensar, depreciação acelerada, etc. Cada conta tem saldo inicial e movimentações no período.

---

## Pontos de atenção

- **Plano de contas referencial** — cada conta contábil deve ser mapeada para um código do plano referencial da Receita Federal. Se o mapeamento estiver incompleto, a ECF terá inconsistências.
- **Consistência ECD × ECF** — os saldos da ECF devem bater com os da ECD. Divergências geram rejeição.
- **Códigos de ajuste** — as adições e exclusões (Parte A) usam códigos padronizados pela Receita Federal. O código errado pode gerar questionamento fiscal.

---

## Para praticar

1. Localize a configuração da ECF no BC (Diga-me > "Configuração ECF")
2. Verifique o tipo de tributação e a forma de apuração configurados
3. Explore os cadastros de sócios (associados) e as contas da Parte B
4. Se possível, gere um arquivo ECF de teste e observe a estrutura dos blocos

**Referência**: consulte o manual "Fiscal ECF" disponível na seção de [Manuais](Localization-for-Brazil_User-Manual_Contents.md).
