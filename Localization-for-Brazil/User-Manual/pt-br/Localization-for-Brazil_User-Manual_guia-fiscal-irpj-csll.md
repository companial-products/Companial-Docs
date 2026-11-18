# IRPJ/CSLL

O IRPJ (Imposto de Renda Pessoa Jurídica) e a CSLL (Contribuição Social sobre o Lucro Líquido) são tributos federais sobre o lucro das empresas. A localização calcula esses tributos a partir dos dados contábeis do BC.

---

## O que é

### IRPJ
- **Alíquota**: 15% sobre o lucro tributável + **adicional de 10%** sobre o que exceder R$ 60.000/trimestre (R$ 20.000/mês)
- **Periodicidade**: trimestral ou anual (com estimativas mensais)

### CSLL
- **Alíquota**: 9% sobre o lucro tributável
- **Periodicidade**: mesma do IRPJ

### Regimes de tributação

| Regime | Como calcula o lucro tributável | Quem usa |
|--------|---------------------------------|----------|
| **Lucro Real** | Lucro contábil + adições - exclusões - compensações | Empresas maiores ou obrigadas |
| **Lucro Presumido** | Receita bruta × percentual de presunção | Empresas menores (receita até R$ 78 mi/ano) |

### Percentuais de presunção (Lucro Presumido)

| Atividade | IRPJ | CSLL |
|-----------|------|------|
| Comércio / Indústria | 8% | 12% |
| Serviços em geral | 32% | 32% |
| Transporte de cargas | 8% | 12% |
| Transporte de passageiros | 16% | 12% |

Exemplo: se a empresa faturou R$ 1.000.000 no trimestre e é comércio:
- Base IRPJ: R$ 1.000.000 × 8% = R$ 80.000
- IRPJ: R$ 80.000 × 15% = R$ 12.000 + adicional sobre excedente de R$ 60.000
- Base CSLL: R$ 1.000.000 × 12% = R$ 120.000
- CSLL: R$ 120.000 × 9% = R$ 10.800

---

## Como a localização implementa

### Lucro Presumido

No Lucro Presumido, a localização suporta dois regimes de apuração da receita:

| Regime | Fonte dos dados | Como funciona |
|--------|----------------|---------------|
| **Regime de Competência** | Esquema de Contas DRE | Lê os movimentos contábeis do período e calcula a receita pelo demonstrativo de resultado |
| **Regime de Caixa** | Recebimentos de Notas Fiscais | Considera apenas as notas fiscais efetivamente recebidas (pagas) no período |

No **Regime de Competência**, a receita é reconhecida no momento do faturamento — o sistema lê o **Esquema de Contas DRE** configurado e calcula o valor agregado do período.

No **Regime de Caixa**, a receita é reconhecida no momento do recebimento — o sistema busca os pagamentos recebidos no período (via movimentos de cliente) e rastreia cada nota fiscal individualmente.

Fluxo de cálculo:

1. **Receita do período**: obtida pelo DRE (competência) ou pelos recebimentos de NFs (caixa)
2. **Presunção**: aplica percentuais de presunção sobre a receita bruta
3. **Adições e exclusões**: ajustes manuais à base de cálculo
4. **Cálculo**: IRPJ (15% + 10% adicional sobre excedente) e CSLL (9%)
5. **Deduções**: valores retidos na fonte, antecipações e compensações

### Lucro Real

No Lucro Real, o ponto de partida é o **lucro contábil** (não a receita). O sistema lê o **Esquema de Contas DRE** para obter o resultado do período, e a partir dele aplica:

1. **Adições**: despesas não dedutíveis, provisões, etc.
2. **Exclusões**: receitas não tributáveis
3. **Compensações**: prejuízos fiscais de períodos anteriores (limitado a 30% do lucro)
4. **Cálculo**: IRPJ (15% + 10% adicional sobre excedente) e CSLL (9%)
5. **Deduções**: valores retidos na fonte, antecipações e compensações

O resultado alimenta a **ECF** (Escrituração Contábil Fiscal) e pode gerar **guias de pagamento** (DARF).

---

## Pontos de atenção

- **Esquema de Contas DRE** (regime de competência) — o cálculo depende da correta configuração do demonstrativo de resultado. Se contas contábeis não estiverem mapeadas, receitas ou despesas podem ficar de fora.
- **Recebimentos** (regime de caixa) — se um pagamento não estiver aplicado à fatura, a receita não será considerada no período. O rastreamento é feito nota a nota.
- **Receitas financeiras** — no lucro presumido, receitas financeiras entram 100% na base (sem aplicar percentual de presunção). Esse é um ponto que costuma gerar erro.
- **Adicional de IRPJ** — o adicional de 10% incide sobre o excedente de R$ 60.000/trimestre, não sobre a base toda.
- **Múltiplas alíquotas de presunção** — empresas que atuam em mais de um segmento podem ter diferentes percentuais de presunção para cada atividade.

---

## Para praticar

1. Localize a configuração de IRPJ/CSLL no BC (Diga-me > "Configuração Apuração IRPJ/CSLL")
2. Verifique o regime de tributação configurado (Lucro Real ou Presumido)
3. Abra uma apuração existente e identifique: receita bruta, base tributável, IRPJ e CSLL calculados
4. Confira se os percentuais de presunção estão corretos para a atividade da empresa

**Referência**: consulte o manual "IRPJ e CSLL" disponível na seção de [Manuais](Localization-for-Brazil_User-Manual_Contents.md).
