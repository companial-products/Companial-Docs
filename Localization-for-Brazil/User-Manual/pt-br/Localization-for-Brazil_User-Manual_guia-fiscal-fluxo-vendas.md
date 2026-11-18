# Fluxo de Vendas

Este capítulo explica como uma venda no BC se transforma em um documento fiscal com todos os impostos calculados. É o fluxo mais comum da localização — entender ele é a base para diagnosticar a maioria dos problemas fiscais.

---

## O que acontece quando uma venda é registrada

O fluxo completo, da ordem de venda à NF-e:

```
1. ORDEM DE VENDA
   Usuário cria a ordem, escolhe cliente, adiciona itens
   A localização adiciona campos fiscais: CFOP, NCM, Área Impostos

2. DADOS FISCAIS
   A localização preenche automaticamente (ou o usuário ajusta):
   - CFOP de cada linha (pela Matriz Impostos, conforme a Localização Cliente/Fornecedor e o Consumidor Final)
   - NCM de cada linha (do cadastro do item)
   - Área Impostos (pela matriz de impostos)

3. CÁLCULO PRÉVIO
   Antes de registrar, o usuário pode visualizar os impostos
   calculados na página de estatísticas da ordem

4. VALIDAÇÃO DO CLIENTE
   Antes do registro, a localização valida:
   - CNPJ válido
   - Inscrição Estadual (se contribuinte)
   - Dados fiscais obrigatórios preenchidos

5. REGISTRO (POST)
   O BC registra a ordem → cria fatura registrada + movimentos
   Simultaneamente, a localização:
   a) Calcula todos os impostos
   b) Cria o Documento Fiscal
   c) Gera os movimentos contábeis de impostos

6. EMISSÃO NF-e
   Se configurado para transmissão automática:
   → XML gerado → enviado à SEFAZ → autorizado
   Se manual: o usuário transmite depois via lista de documentos fiscais

7. E-MAIL E DANFe
   Se configurado, o sistema envia automaticamente por e-mail
   o XML e o DANFe (PDF) ao cliente
```

---

## Dados fiscais na ordem de venda

A localização estende a ordem de venda com campos fiscais:

### No cabeçalho

| Campo | O que é |
|-------|---------|
| **Cód. Filial** (Branch Code) | Filial emitente |
| **Cód. NOP** (NOP Code) | Natureza da Operação (texto que aparece na NF-e) |
| **Tipo Documento Fiscal** (Fiscal Document Type) | Tipo de documento fiscal |

Estes são apenas alguns exemplos — a localização adiciona diversos outros campos ao cabeçalho.

### Nas linhas

| Campo | O que é |
|-------|---------|
| **Código CFOP** (CFOP Code) | Código fiscal da operação para esta linha |
| **Código NCM** (NCM Code) | Classificação do produto (vem do item) |
| **Cód. Área Impostos** (Tax Area Code) | Área Impostos (quais impostos calcular) |
| **CSTs** | Situação tributária de ICMS, IPI, PIS, COFINS |
| **Exceção Imposto** (Tax Exception) | Exceção fiscal aplicável (se houver) |

Estes são apenas alguns exemplos — a localização adiciona diversos outros campos às linhas. Todos são preenchidos automaticamente com base no item, no cliente e na configuração fiscal. O usuário pode ajustar quando necessário.

---

## Como os impostos são calculados

O cálculo acontece automaticamente ao registrar. Para cada linha:

1. A **Matriz Impostos** define a **Área Impostos** e os CSTs para a operação
2. A **Área Impostos** define quais impostos calcular (jurisdições)
3. Cada jurisdição busca a alíquota conforme sua configuração (Alíquotas ICMS por UF, exceções, etc.)
4. Os **flags da Contabilização Imposto** determinam o efeito contábil
5. Os resultados são gravados no documento fiscal (Mov. Imposto Documento Fiscal)

### O que pode dar errado

| Sintoma | Causa provável |
|---------|---------------|
| Imposto zerado | Matriz Impostos sem configuração para a operação |
| Imposto faltando | Área Impostos não inclui a jurisdição do imposto |
| Alíquota errada | Alíquotas ICMS por UF desatualizada ou exceção fiscal incorreta |
| CST incorreto | CST errado na Matriz Impostos, na exceção, nas Alíquotas NCM ou na jurisdição/área de impostos, ou **Prioridades CST** mal configuradas |
| Contabilização errada | Contabilização Imposto com flags incorretos |

---

## Parcelas (Installments)

A localização estende a tabela **Termos de Pagamento** (Payment Terms) com um sistema de parcelas. Para cada termo de pagamento, é possível configurar múltiplas **Parcelas** (Installments), cada uma com:

- **Percentual** — percentual do valor total da fatura
- **Fórmula de vencimento** — período relativo à data da fatura (ex: 30 dias, 60 dias)

A localização também adiciona ao termo de pagamento:
- **Data Venc. em Dias Não Úteis** (Due Date When Not Work Day) — define se o vencimento em dia não útil deve ser antecipado, adiado ou mantido

Impostos que afetam o financeiro (como IPI "Somar ao Total") alteram o valor das parcelas automaticamente. Retenções ("Subtrair do Total" ou "Subtrair da Primeira Parcela") reduzem o valor.

---

## NF-e Complementar

Quando é necessário complementar valores ou impostos de uma NF-e já emitida (por exemplo, diferença de preço ou imposto incorreto), a localização permite criar uma **NF-e Complementar** — um documento fiscal de saída que referencia a NF-e original e contém apenas a diferença.

---

## Pontos de atenção

- **CFOP inconsistente com a operação** — uma venda interestadual com CFOP de operação interna (5xxx em vez de 6xxx) gera NF-e rejeitada ou escrituração incorreta.
- **Transmissão automática desligada** — o documento fiscal é criado mas a NF-e não é transmitida. O usuário precisa transmitir manualmente.
- **Validação fiscal** — a localização valida dados antes do registro (CNPJ, IE, campos obrigatórios). Se a validação falhar, o registro é bloqueado. Verifique a mensagem de erro.

---

## Para praticar

1. Crie uma **Ordem de Venda** para um cliente dentro do estado. Nas linhas, observe os campos fiscais: CFOP, NCM, Área Impostos
2. Abra a página de **Estatísticas** da ordem e examine os impostos calculados (ICMS, PIS, COFINS, IPI)
3. **Registre** a ordem e depois localize:
   - A Fatura Venda Registrada (Posted Sales Invoice)
   - O Documento Fiscal (Documentos Fiscais, filtrar por Saída)
   - Os Movimentos Contábeis (GL Entries) — identifique os movimentos de impostos
4. Repita com uma venda **interestadual** (cliente de outro estado) e compare as alíquotas
5. Se possível, acompanhe a transmissão da NF-e à SEFAZ e verifique o status

**Referência**: consulte o manual "Vendas" disponível na seção de [Manuais](Localization-for-Brazil_User-Manual_Contents.md).
