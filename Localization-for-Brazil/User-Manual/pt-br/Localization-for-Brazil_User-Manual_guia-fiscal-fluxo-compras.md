# Fluxo de Compras

O fluxo de compras tem uma diferença fundamental em relação ao de vendas: a NF-e **já existe** — foi emitida pelo fornecedor. O papel da localização na compra é **escriturar** o documento fiscal, calcular os impostos (incluindo retenções) e gerar os movimentos contábeis corretos.

---

## O que acontece quando uma compra é registrada

```
1. FATURA DE COMPRA
   Usuário cria a fatura (ou registra uma ordem de compra)
   Informa os dados do fornecedor e os itens recebidos

2. DADOS FISCAIS
   A localização preenche automaticamente:
   - CFOP de entrada, pela Matriz Impostos da operação de compra
   - NCM de cada item
   - Área Impostos
   - Dados de retenção (se aplicável ao fornecedor)

3. VALIDAÇÃO DO FORNECEDOR
   Antes do registro, a localização valida:
   - CNPJ válido
   - Inscrição Estadual (se contribuinte)
   - Dados fiscais obrigatórios preenchidos

4. REGISTRO (POST)
   O BC registra a fatura → movimentos contábeis + estoque
   A localização:
   a) Calcula impostos (ICMS, IPI, PIS, COFINS)
   b) Calcula retenções na fonte (se aplicável)
   c) Cria o Documento Fiscal de entrada
   d) Gera movimentos contábeis:
      - Impostos recuperáveis (créditos fiscais)
      - Impostos no custo
      - Retenções
   e) Cria parcelas (installments)

5. ESCRITURAÇÃO
   O documento fiscal fica disponível para o app de Livros Fiscais
   gerar os registros de entrada no EFD
```

---

## Retenções na fonte

A grande particularidade da compra. Quando a empresa (tomador) contrata serviços de determinados fornecedores, ela é obrigada a **reter e recolher** impostos que seriam devidos pelo fornecedor.

### Quais impostos podem ser retidos

| Imposto | Alíquota típica | Quando se aplica |
|---------|-----------------|-----------------|
| **IRRF** | 1% a 2,5% | Serviços profissionais, comissões, publicidade |
| **CSLL Retida** | 1% | Serviços listados na Lei 10.833 |
| **PIS Retido** | 0,65% | Junto com CSLL e COFINS ("PCC" ou "4,65%") |
| **COFINS Retida** | 3% | Junto com CSLL e PIS |
| **INSS Retido** | 11% | Cessão de mão de obra |
| **ISS Retido** | Varia por município | Quando o ISS é retido pelo tomador |

### Como as retenções funcionam no sistema

1. A **Área Impostos** da linha deve incluir as jurisdições de retenção (IRRF, PCC, INSS, etc.)
2. No registro da fatura, a localização calcula o valor de cada retenção
3. O valor retido é **subtraído** do valor a pagar ao fornecedor:
   - Se configurado no **Registro** (Posting): a retenção é descontada no registro da fatura
   - Se configurado no **Pagamento** (Payment): a retenção é descontada no momento do pagamento
4. A empresa gera os DARFs de retenção e recolhe os valores

### Efeito financeiro

| Sem retenção | Com retenção (IRRF 1,5%) |
|-------------|--------------------------|
| Fatura: R$ 10.000 | Fatura: R$ 10.000 |
| Paga ao fornecedor: R$ 10.000 | Paga ao fornecedor: R$ 9.850 |
| | DARF IRRF: R$ 150 |

O fornecedor recebe menos, mas o IR já foi recolhido em nome dele.

---

## Créditos fiscais na compra

Outro aspecto importante da compra é o **crédito fiscal**. Alguns impostos pagos na compra podem ser recuperados (creditados) na apuração:

| Imposto | Crédito quando |
|---------|---------------|
| **ICMS** | Compra de mercadoria para revenda ou insumo de produção (operação tributada) |
| **IPI** | Compra de matéria-prima por indústria |
| **PIS/COFINS** | Regime não cumulativo — compras geram crédito |

Se o imposto **não** gera crédito (ex: compra para uso e consumo), ele entra no **custo do item** ou como **despesa**. Quem controla isso é a **Contabilização Imposto** — os flags `Crédito Imposto` vs. `Custo/Preço Item` + `Despesa`.

---

## Importação de XML

Uma funcionalidade importante da compra é a **importação de XML** da NF-e do fornecedor. Em vez de digitar os dados manualmente, o usuário pode:

1. Importar o arquivo XML da NF-e recebida
2. O sistema parseia o XML e preenche automaticamente os dados da fatura
3. O usuário confere e ajusta se necessário
4. Registra normalmente

Isso também pode acontecer automaticamente via **MD-e** (Manifestação do Destinatário), onde o sistema consulta periodicamente a SEFAZ por NF-e destinadas à empresa.

---

## Pontos de atenção

- **Retenção não configurada** — se a matriz de impostos ou a categoria do fornecedor não indicam retenção, o sistema não retém. Isso pode gerar problemas com o fisco.
- **Crédito fiscal indevido** — se a Contabilização Imposto está com `Crédito Imposto = Sim` em uma operação sem direito a crédito (uso e consumo, por exemplo), a apuração ficará errada.
- **IPI no custo** — o IPI na compra geralmente **entra no custo** do item (Contabilização Imposto com `Custo/Preço Item = Somar`). Se não estiver configurado, o custo do item ficará menor que o real.
- **ICMS-ST na compra** — quando a mercadoria vem com ST, o ICMS-ST pago é custo (não gera crédito de ICMS próprio). Verifique se a Contabilização Imposto está correta.
- **CFOP de entrada** — o CFOP de entrada (1xxx, 2xxx, 3xxx) depende da operação e do uso da mercadoria por quem compra (revenda, industrialização, uso e consumo, ativo), e não do CFOP de saída do fornecedor. Ele vem da Matriz Impostos: revise a matriz de cada operação de compra. CFOP errado = escrituração incorreta.

---

## Para praticar

1. Crie uma **Fatura de Compra** para um fornecedor de mercadoria. Observe os campos fiscais nas linhas
2. Registre e localize o **Documento Fiscal** de entrada. Compare os impostos: quais geraram crédito? Quais entraram no custo?
3. Crie uma fatura de compra de **serviço** com fornecedor sujeito a retenção. Após registrar, verifique:
   - O valor do Mov. Fornecedor (Vendor Ledger Entry) — está líquido das retenções?
   - Os movimentos contábeis de retenção
4. Se disponível, **importe um XML** de NF-e e compare os dados preenchidos automaticamente com o XML original
5. Abra a **Contabilização Imposto** usada na compra e confira os flags: `Crédito Imposto` está ativo? `Custo/Preço Item` está com `Somar`?

**Referência**: consulte o manual "Compras" disponível na seção de [Manuais](Localization-for-Brazil_User-Manual_Contents.md).
