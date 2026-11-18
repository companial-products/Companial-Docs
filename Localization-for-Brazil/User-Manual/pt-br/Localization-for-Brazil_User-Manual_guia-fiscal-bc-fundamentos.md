# Business Central — Fundamentos

Este capítulo apresenta o Microsoft Dynamics 365 Business Central para quem nunca trabalhou com o sistema. O objetivo não é cobrir tudo — é dar a base necessária para que você consiga navegar, entender a terminologia e acompanhar o restante do guia.

---

## O que é o Business Central

O Business Central (BC) é o ERP da Microsoft para empresas de médio porte. Ele roda na nuvem (Azure) ou on-premises e cobre os processos fundamentais: finanças, compras, vendas, estoque, produção, projetos e ativos fixos.

A localização brasileira da Fenix se conecta ao BC padrão via **apps** (extensões) que adicionam toda a camada fiscal e tributária sem alterar o código-fonte original da Microsoft. Isso é importante: o BC padrão não sabe nada sobre NF-e, ICMS ou SPED — quem adiciona esse conhecimento é a localização.

---

## Navegação básica

### Centro de Função (Role Center)

Ao abrir o BC, você cai no **Centro de Função** — a página inicial configurada para o seu perfil. Ela mostra atalhos, indicadores e atividades pendentes. Cada perfil de usuário pode ter um Centro de Função diferente.

### Diga-me (Tell Me)

O **Diga-me** é a barra de pesquisa do BC (atalho: `Alt+Q`). É a forma mais rápida de encontrar qualquer coisa no sistema — páginas, relatórios, configurações, ações. Em vez de decorar menus, use o Diga-me. Digite o que procura em português (ex: "Configurações Fiscais", "Clientes", "Ordens de Venda") e o sistema encontra.

### Listas e Cartões (Cards)

O BC organiza os dados em dois tipos de página:

- **Listas** — exibem vários registros em formato de tabela. Ex: Lista de Clientes, Lista de Ordens de Venda.
- **Cartões** (Cards) — exibem um único registro em detalhe, com abas/grupos de campos. Ex: Cartão do Cliente, Cartão do Item.

Para abrir um registro, clique nele na lista. Para criar um novo, use o botão **Novo** (+).

### Painéis Informativos (FactBoxes)

Na lateral direita de listas e cartões, o BC mostra **Painéis Informativos** (FactBoxes) — painéis com informações complementares do registro selecionado (saldo, documentos relacionados, imagem, etc.). Nem todas as páginas têm painéis informativos.

### Filtros

Toda lista no BC pode ser filtrada. Clique no ícone de filtro ou use `Shift+F3` para abrir o painel de filtros. Você pode filtrar por qualquer coluna visível. Os filtros suportam operadores: `>1000`, `01/01/2026..31/03/2026`, `@texto` (case-insensitive).

---

## Conceitos fundamentais

### Empresa (Company)

O BC suporta múltiplas **empresas** dentro do mesmo ambiente. Cada empresa tem seus próprios dados — clientes, fornecedores, plano de contas, saldos. Você escolhe a empresa ativa no canto superior direito. Cuidado: verifique sempre em qual empresa você está trabalhando.

### Registro (Posting)

**Registrar** é o ato de transformar um documento em movimentos permanentes no sistema. Quando você registra uma ordem de venda, o BC:

1. Cria os movimentos contábeis (Movimentos Contábeis)
2. Cria movimentos de cliente (Customer Ledger Entries)
3. Cria movimentos de estoque (Item Ledger Entries)
4. Na localização: cria o **Documento Fiscal** com todos os impostos

> **Importante**: o registro é **irreversível**. Não é possível "desregistrar". Para corrigir um erro, é necessário criar um documento de estorno (nota de crédito, devolução, etc.). É exatamente como na contabilidade — movimentos registrados não são apagados, são estornados.

### Nº Série (Number Series)

Todo documento no BC recebe um número automático via **Nº Série** (Number Series). O sistema avança o contador a cada documento criado. As Nº Séries são configuráveis por tipo de documento. Na localização, há séries específicas para documentos fiscais, NF-e, NFS-e, etc.

### Dimensões (Dimensions)

**Dimensões** são eixos analíticos que complementam o plano de contas. Exemplos comuns: Centro de Custo, Filial, Projeto, Departamento. Você pode associar até 8 dimensões a qualquer movimento, permitindo análises cruzadas sem precisar criar contas contábeis separadas.

Na localização, a dimensão de **filial** é especialmente importante — ela vincula movimentos contábeis ao estabelecimento correto.

### Grupos de Registro (Posting Groups)

Os **Grupos de Registro** (Posting Groups) definem para quais contas contábeis os movimentos vão. Existem vários tipos:

| Tipo | Determina |
|------|-----------|
| **Grupo Registro Negócios** (Gen. Business Posting Group) | Conta por tipo de parceiro (doméstico, interestadual, exterior) |
| **Grupo Registro Produto** (Gen. Product Posting Group) | Conta por tipo de produto (mercadoria, serviço, matéria-prima) |
| **Grupo Registro Cliente/Fornecedor** (Customer/Vendor Posting Group) | Conta de clientes/fornecedores a receber/pagar |
| **Grupo Registro Estoque** (Inventory Posting Group) | Conta de estoque |

A combinação de Grupo Registro Negócios + Grupo Registro Produto (chamada **Configuração Registro Geral** / General Posting Setup) determina as contas de receita, custo, ajuste de estoque, etc. É fundamental para a contabilização correta.

---

## Fluxo básico de um documento

Para entender o BC, é útil acompanhar o ciclo de vida de um documento simples:

```
1. CRIAR o documento (ex: Ordem de Venda)
   - Escolher cliente, adicionar linhas com itens e quantidades

2. LIBERAR (Release)
   - O documento fica "travado" para edição

3. REGISTRAR (Post)
   - O sistema gera todos os movimentos (contábeis, estoque, fiscal)
   - O documento original some da lista de pendentes
   - Aparece nas listas de "Registrados" (Posted)
   - Na localização: os impostos são calculados, o Documento Fiscal é criado e a NF-e pode ser emitida

4. CONSULTAR
   - Documentos registrados ficam acessíveis para consulta
   - Movimentos contábeis podem ser navegados (drill-down)
```

---

## Páginas de configuração (Setup)

Muitos comportamentos do BC são controlados por **páginas de Configuração**. Algumas das mais importantes:

| Configuração | O que controla |
|-------|---------------|
| **Configuração Contabilidade** (General Ledger Setup) | Moeda, dimensões obrigatórias, períodos contábeis |
| **Configuração de Vendas** (Sales & Receivables Setup) | Numeração, políticas de desconto, avisos de crédito |
| **Configuração de Compras** (Purchases & Payables Setup) | Numeração, recebimento obrigatório, políticas de pagamento |
| **Configuração de Estoque** (Inventory Setup) | Método de custeio, numeração de itens |
| **Configurações Fiscais** (Fiscal Setup) | Configurações da localização — imprescindível |

> Dica: use o Diga-me para encontrar qualquer uma delas. Digite "Configuração" e explore.

---

## Terminologia essencial

| Termo no BC (pt-BR) | Significado |
|---------------------|-------------|
| **Registrar** (Post) | Confirmar um documento e gerar movimentos permanentes |
| **Data Registro** (Posting Date) | Data contábil do movimento — pode diferir da data do documento |
| **Movimento** (Entry/Ledger Entry) | Registro permanente gerado pelo posting (contábil, estoque, cliente, etc.) |
| **Nota de Crédito** (Credit Memo) | Documento de estorno/devolução |
| **Remessa** (Shipment) | Documento de expedição de mercadoria |
| **Recebimento** (Receipt) | Documento de recebimento de mercadoria |
| **Fatura** (Invoice) | Documento de cobrança/pagamento |
| **Diário** (Journal) | Registro manual (contábil, estoque, etc.) |
| **Nº Série** (Number Series) | Sequência automática de numeração |
| **Dimensão** (Dimension) | Eixo analítico (centro de custo, filial, etc.) |

---

## Para praticar

1. **Navegação**: abra o BC, explore o Centro de Função, use o Diga-me para encontrar "Clientes", "Itens", "Ordens de Venda"
2. **Listas e filtros**: abra a lista de Itens, aplique um filtro por tipo ou grupo, remova o filtro
3. **Documento completo**: crie uma Ordem de Venda simples (escolha um cliente, adicione um item), registre e depois encontre o documento registrado na lista de "Faturas Venda Registradas"
4. **Movimentos**: a partir da fatura registrada, navegue até os Movimentos Contábeis usando o menu "Relacionado" ou "Navegar"
5. **Configurações**: use o Diga-me para abrir a "Configuração Contabilidade", "Configuração de Vendas" e "Configurações Fiscais" — apenas observe, sem alterar

**Referência**: para um guia completo de navegação, consulte o manual "Introdução e Navegação" disponível na seção de [Manuais](Localization-for-Brazil_User-Manual_Contents.md).
