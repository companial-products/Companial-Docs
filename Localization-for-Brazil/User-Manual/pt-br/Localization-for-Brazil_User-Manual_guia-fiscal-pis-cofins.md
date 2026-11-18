# PIS/COFINS e EFD-Contribuições

PIS e COFINS são contribuições federais que incidem sobre a receita das empresas. A localização calcula esses tributos nos documentos fiscais e o app de **Livros Fiscais** (Tax Books) gera a apuração e o arquivo EFD-Contribuições.

---

## O que é

### PIS (Programa de Integração Social)

Contribuição federal sobre a receita bruta. Alíquota varia conforme o regime:
- **Não cumulativo**: 1,65% (permite créditos sobre insumos)
- **Cumulativo**: 0,65% (sem direito a créditos)

### COFINS (Contribuição para Financiamento da Seguridade Social)

Mesma lógica do PIS, com alíquotas diferentes:
- **Não cumulativo**: 7,60%
- **Cumulativo**: 3,00%

### Regime cumulativo vs. não cumulativo

| Aspecto | Não cumulativo | Cumulativo |
|---------|---------------|------------|
| Alíquotas | PIS 1,65% / COFINS 7,60% | PIS 0,65% / COFINS 3,00% |
| Créditos | Sim (insumos, ativos fixos, etc.) | Não |
| Empresas | Lucro Real | Lucro Presumido |
| Base | Receita total | Receita total |

No regime não cumulativo, as compras geram **créditos** que são abatidos dos débitos (vendas). No cumulativo, o imposto é calculado apenas sobre as vendas, sem créditos.

---

## Como a localização implementa

### No app da Localização

O cálculo de PIS e COFINS acontece em cada documento fiscal, da mesma forma que ICMS:
- A **Matriz Impostos** define a Área Impostos e os CSTs
- A **Área Impostos** inclui jurisdições de PIS e COFINS
- As **alíquotas** de PIS e COFINS são fixas por empresa, por isso normalmente são definidas diretamente na **Jurisdição Imposto** (diferente do ICMS, que varia por UF)
- A **Contabilização Imposto** (definida na Área Impostos) determina o efeito contábil — crédito, débito, custo, etc.

### No app de Livros Fiscais

O módulo de apuração PIS/COFINS:

1. **Processa documentos** — lê todos os documentos fiscais do período
2. **Consolida** — separa débitos (saídas) e créditos (entradas)
3. **Créditos especiais**:
   - **F120** — Crédito sobre depreciação de ativo fixo
   - **F130** — Crédito sobre o valor de aquisição do ativo imobilizado, apropriado de forma integral ou em 6, 12, 24 ou 48 parcelas mensais, conforme a configuração do bem
   - **F100** — Demais operações geradoras de contribuição ou de crédito (ex.: receitas financeiras no regime não cumulativo, aluguéis)
4. **Gera demonstrativo** — débitos - créditos - ajustes - retenções = saldo devedor
5. **Alimenta o EFD-Contribuições** com os registros correspondentes

### EFD-Contribuições

Arquivo SPED mensal com a escrituração de PIS e COFINS. Similar ao EFD-ICMS/IPI em formato, mas focado nessas duas contribuições. Contém registros de documentos de entrada (créditos), saída (débitos), operações sem documento (F100/F120/F130) e apuração.

---

## Pontos de atenção

- **CST PIS/COFINS errado** — é o erro mais comum. Um CST de crédito em uma operação que não dá direito gera crédito indevido na apuração.
- **Regime errado** — se a empresa é lucro presumido (cumulativo) mas as alíquotas estão configuradas como não cumulativo (1,65/7,60), os valores estarão errados.
- **Crédito de ativo fixo** — o crédito de PIS/COFINS sobre o ativo imobilizado pode ser apurado sobre os encargos de depreciação (F120) ou sobre o valor de aquisição (F130, integral ou em 6, 12, 24 ou 48 parcelas, conforme o bem). Verifique a configuração usada para cada ativo.
- **Receitas financeiras** — no regime não cumulativo, as receitas financeiras são tributadas com alíquotas específicas (PIS 0,65% e COFINS 4%) e informadas no registro F100.

---

## Para praticar

1. Localize a configuração de EFD-Contribuições no BC
2. Abra uma apuração de PIS/COFINS e identifique: débitos (saídas), créditos (entradas), saldo
3. Confira o CST PIS/COFINS em um documento fiscal — está consistente com o regime da empresa?
4. Se possível, gere o arquivo EFD-Contribuições e compare com o EFD-ICMS/IPI — observe as diferenças de estrutura

**Referência**: consulte o manual "Fiscal Apuração PIS-COFINS" disponível na seção de [Manuais](Localization-for-Brazil_User-Manual_Contents.md).
