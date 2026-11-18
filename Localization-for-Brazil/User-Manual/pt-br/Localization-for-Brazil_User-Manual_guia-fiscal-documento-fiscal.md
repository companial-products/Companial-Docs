# O Documento Fiscal

O Documento Fiscal é a **entidade central** de toda a localização. Tudo converge para ele: a configuração fiscal determina como ele será preenchido, o cálculo de impostos atua sobre ele, a NF-e é gerada a partir dele, e os livros fiscais o escrituram.

Entender o Documento Fiscal é entender como a localização funciona.

---

## O que é

No contexto fiscal brasileiro, toda operação que envolva circulação de mercadorias ou prestação de serviços deve ser documentada por um documento fiscal. A versão eletrônica é obrigatória:

| Tipo | Modelo | Uso |
|------|--------|-----|
| **NF-e** | 55 | Mercadorias (venda, compra, transferência, devolução) |
| **NFC-e** | 65 | Venda ao consumidor final (varejo) |
| **NFS-e** | — | Serviços |
| **CT-e** | 57 | Transporte |

O documento fiscal não é apenas uma "nota" — ele é a **unidade básica** de toda a cadeia tributária: cálculo de impostos, escrituração fiscal (SPED), apuração de tributos e obrigações acessórias.

---

## Como a localização implementa

### O Documento Fiscal no BC

A localização cria uma entidade própria — o **Documento Fiscal** — que armazena todos os dados fiscais de cada operação. Ela é **separada** dos documentos padrão do BC (Fatura Venda Registrada, Fatura Compra Registrada, etc.).

Quando você registra uma venda ou compra no BC, dois conjuntos de registros são criados:
1. Os registros padrão do BC (fatura registrada, movimentos contábeis, estoque)
2. O **Documento Fiscal** da localização (com todos os dados tributários)

Eles são vinculados, mas independentes. Você pode consultar ambos.

### Estrutura

| Entidade | O que contém |
|----------|-------------|
| **Documento Fiscal** (cabeçalho) | Dados gerais: partes (emitente/destinatário), filial, tipo, direção (entrada/saída), status, datas |
| **Linha de Documento Fiscal** (linhas) | Itens/serviços: quantidade, valor, CFOP, NCM, CST, impostos calculados |
| **Mov. Imposto Documento Fiscal** (Fiscal Document Tax Entry) | Impostos calculados e registrados por Classificação Imposto |

### Campos-chave do cabeçalho

| Campo | Significado |
|-------|------------|
| **Nº** (No.) | Número do documento fiscal |
| **Tipo** (Type) | Entrada (compra) ou Saída (venda) |
| **Tipo Documento Fiscal** (Fiscal Document Type) | Define o tipo do documento (NF-e, NFS-e, CT-e, etc.) — determina quais registros do SPED serão alimentados e como a transmissão eletrônica será feita |
| **Tipo Origem** (Source Type) | Origem (Ordem de Venda, Ordem de Compra, Transferência, etc.) |
| **Cód. Filial** (Branch Code) | Filial que emitiu/recebeu |
| **Participante** (Participant) | Cliente ou fornecedor |
| **Data Registro** (Posting Date) | Data de registro |
| **Chave de Acesso** (Access Key) | Chave de acesso de 44 dígitos (se NF-e) |
| **Código Status** (Status Code) | Código do status da NF-e retornado pela SEFAZ (ver [Ciclo de Vida da NF-e](Localization-for-Brazil_User-Manual_guia-fiscal-nfe-ciclo.md)) |
| **Resultado** (Result) | Texto da resposta da SEFAZ, como o motivo de uma rejeição |

### Campos-chave da linha

| Campo | Significado |
|-------|------------|
| **Nº Item** (Item No.) | Código do item/serviço |
| **Código CFOP** (CFOP Code) | Código Fiscal da Operação |
| **Código NCM** (NCM Code) | Classificação do produto |
| **Cód. Área Impostos** (Tax Area Code) | Área Impostos (quais impostos se aplicam) |
| **CSTs** | Situação tributária (ICMS, IPI, PIS, COFINS) |
| **Valores** | Quantidade, Preço Unitário, Valor Total, Descontos |

**Caminho no BC**: Diga-me > "Documentos Fiscais"

---

## Ciclo de vida

```
1. CRIAÇÃO E REGISTRO
   O documento fiscal é criado automaticamente ao registrar
   uma venda, compra ou transferência no BC.
   Nesse momento, os impostos são calculados com base na configuração:
   - Matriz Impostos → Área Impostos, CSTs, CFOPs
   - Área Impostos → quais impostos calcular (jurisdições)
   - Contabilização Imposto → como contabilizar
   O registro gera: movimentos contábeis de impostos, movimentos fiscais.
   (Exceção: NF-e complementar pode ser criada manualmente.)

4. EMISSÃO ELETRÔNICA
   Se for NF-e: geração do XML → transmissão à SEFAZ → autorização
   Se for NFS-e: geração do RPS → transmissão à prefeitura

5. ESCRITURAÇÃO
   O app de Livros Fiscais lê os documentos fiscais registrados e gera:
   - Registros do EFD-ICMS/IPI
   - Registros do EFD-Contribuições
   - Dados para apurações

6. EVENTOS (pós-emissão)
   - Cancelamento
   - Carta de Correção (CCe)
   - Manifestação do Destinatário (MD-e)
   - Inutilização de numeração
```

---

## Tipos de documento fiscal

| Tipo | Quando é criado | Direção |
|------|----------------|---------|
| **Venda (fatura)** | Ao registrar ordem/fatura de venda | Saída |
| **Compra (escrituração)** | Ao registrar fatura de compra | Entrada |
| **Devolução de venda** | Ao registrar nota de crédito de venda | Entrada |
| **Devolução de compra** | Ao registrar nota de crédito de compra | Saída |
| **Transferência** | Ao registrar transferência entre localizações | Saída + Entrada |
| **NF-e Complementar** | Criada manualmente para complementar valores/impostos | Saída |
| **NFS-e** | Ao registrar venda de serviço | Saída |

### Modelos de documento fiscal

Além dos tipos acima, o BC suporta diversos **modelos** de documento fiscal (códigos usados no SPED e na SEFAZ):

| Modelo | Descrição | Tipo |
|--------|-----------|------|
| **55** | NF-e — Nota Fiscal Eletrônica | Eletrônico |
| **65** | NFC-e — Nota Fiscal ao Consumidor | Eletrônico |
| **57** | CT-e — Conhecimento de Transporte | Eletrônico |
| **66** | NF3-e — Energia Elétrica | Eletrônico |
| **67** | CT-e OS — Outros Serviços de Transporte | Eletrônico |
| 01 | Nota Fiscal (papel) | NF |
| 04 | Nota Fiscal de Produtor | NF |
| 06, 28, 29 | NF de concessionárias (energia, gás, água) | Concessionária |

---

## Documento Fiscal vs. Documento do BC

É importante entender a diferença:

| Aspecto | Documento BC (Fatura Venda Registrada) | Documento Fiscal |
|---------|-------------------------------------|-----------------|
| **Quem cria** | BC padrão | Localização |
| **O que contém** | Valores comerciais, movimentos contábeis | Dados tributários completos |
| **Numeração** | Nº Série do BC | Nº Série fiscal (pode ser a mesma) |
| **NF-e** | Não sabe o que é NF-e | Contém chave de acesso, status SEFAZ |
| **Impostos** | Não tem ICMS, IPI, PIS individual | Tem cada imposto detalhado por linha |
| **Escrituração** | Não aparece no EFD | Base de toda a escrituração SPED |

Na prática, quando alguém diz "abra a nota fiscal", pode estar se referindo a qualquer um dos dois. No contexto fiscal, é sempre o **Documento Fiscal** da localização.

---

## Cancelamento e eventos pós-emissão

Após o registro e a emissão da NF-e, ainda é possível realizar operações sobre o documento fiscal:

| Operação | O que faz | Quando usar |
|----------|-----------|-------------|
| **Cancelamento** | Anula o documento fiscal junto à SEFAZ (até 24h após autorização) | Erro no documento, operação não realizada |
| **Cancelamento com estorno** | Cancela e gera lançamentos contábeis de estorno | Quando os movimentos contábeis também precisam ser revertidos |
| **Carta de Correção (CC-e)** | Corrige dados textuais da NF-e sem alterar valores | Erros de digitação em dados complementares; CFOP só quando a correção não altera o imposto |
| **Mudança manual de status** | Toggle entre Registrado↔Cancelado com justificativa | Regularização fiscal com observação obrigatória |

**Caminho no BC**: abra o Documento Fiscal → use as ações disponíveis na faixa de opções (ribbon).

> **Atenção**: o cancelamento direto na SEFAZ tem prazo (normalmente 24 horas). Após esse período, a alternativa é emitir uma NF-e de devolução/entrada referenciando o documento original. O passo a passo do cancelamento está em [Ciclo de Vida da NF-e](Localization-for-Brazil_User-Manual_guia-fiscal-nfe-ciclo.md#cancelamento) e em [NF-e: Cancelamento na SEFAZ no Mesmo Fluxo, Prazo de Cancelamento por Filial e Cancelamentos Pendentes](Localization-for-Brazil_User-Manual_NFe-Cancelamento-SEFAZ-Prazo-Pendencias.md).

---

## Pontos de atenção

- **Documento fiscal sem NF-e** — o documento pode existir sem ter sido transmitido à SEFAZ. Verifique o **Código Status** e, se necessário, use **NF-e** → **Consultar Status**.
- **Impostos zerados** — se a Área Impostos não inclui a jurisdição do imposto, ou se a Matriz Impostos não está configurada para a operação, os impostos podem ficar zerados sem gerar erro.
- **Direção errada** — devoluções invertem a direção. Uma devolução de venda é um documento de Entrada (e não de Saída).
- **Documento fiscal de compra** — na compra, o documento fiscal é de escrituração. A NF-e já existe (emitida pelo fornecedor). O que a localização faz é registrar os dados fiscais para escrituração.

---

## Para praticar

1. Abra a lista de **Documentos Fiscais** e explore os filtros: filtre por Direção (Entrada/Saída), por Filial, por período
2. Abra um documento fiscal de venda e identifique: CFOP, NCM, impostos calculados, status da NF-e
3. A partir do documento fiscal, navegue até a **fatura registrada** do BC (campo Tipo Origem) e compare os dados
4. Abra um documento fiscal de compra e compare: quais impostos aparecem? Há retenções?
5. Localize um documento fiscal com NF-e autorizada (**Código Status** 100) e um com NF-e rejeitada — o que difere no **Resultado**?

**Referência**: consulte os manuais de "Vendas" e "Compras" disponíveis na seção de [Manuais](Localization-for-Brazil_User-Manual_Contents.md).
