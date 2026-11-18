<!-- Source: manual de referência (Matriz Fiscal Avançada) -->
# NOP — Natureza da Operação e Matriz Fiscal

**Data**: Maio/2026

---

## Sumário

A **NOP** (Natureza da Operação) é o eixo central que classifica o tipo de operação comercial em um documento fiscal: venda, remessa para industrialização, devolução, consignação etc. Combinada com a **Matriz Impostos** e a **Matriz Fiscal**, ela determina automaticamente o tipo de documento fiscal, os CFOPs, os CSTs e os demais parâmetros tributários.

Este manual descreve o cadastro dos códigos NOP, a configuração da Matriz Fiscal de Clientes e de Fornecedores, e o procedimento de uso da NOP nos documentos de venda e compra.

---

## 1. Contexto

### O que é

A NOP é um código cadastrado na localização que classifica o **tipo de operação comercial** que está sendo realizada. Exemplos: "Venda de mercadoria", "Remessa para industrialização", "Devolução de compra".

A descrição da NOP é o texto que aparece no campo **Natureza Operação** do documento e da NF-e. Dentro do sistema, ela funciona como o eixo que conecta e determina automaticamente diversas configurações fiscais de um documento.

### Para que serve / Quando usar

A NOP faz parte da funcionalidade de **Matriz Fiscal Avançada**, um módulo complementar da localização. É indicada quando a empresa:

- Trabalha com **múltiplos tipos de operação** (venda, remessa, industrialização, consignação, bonificação etc.) e precisa que cada uma tenha CFOPs, CSTs e tipos de documento fiscal distintos;
- Deseja **automatizar** a atribuição de códigos fiscais nas linhas dos documentos com base no tipo de operação, no cliente ou fornecedor e no item;
- Quer reduzir erros manuais na escolha de CFOP e CST em cenários com muitas combinações.

Sem a NOP, o usuário precisa configurar os códigos fiscais diretamente nos documentos ou depender apenas da **Matriz Impostos** genérica. Com a NOP, o sistema faz a determinação fiscal automaticamente.

---

## 2. Configurações Necessárias

### 2.1. Cadastro dos Códigos NOP

Navegue até a página **Códigos NOP** (`NOPCodesFNX`). Para cada código, preencha:

| Campo | Descrição |
|-------|-----------|
| **Código** | Identificador único (ex.: `VENDA`, `REMIND`, `DEVCO`) |
| **Natureza da Operação (NOP)** | Descrição textual da operação. É esse texto que será levado ao campo **Natureza Operação** do documento e da NF-e |
| **Tipo Documento Fiscal** | Tipo de documento fiscal que será preenchido automaticamente quando essa NOP for selecionada |
| **Filtro Entrada/Saída** | Define se a NOP se aplica a operações de entrada, saída ou ambas |
| **Filtro Compra/Venda** | Define se a NOP se aplica a compras, vendas ou ambas |
| **Cenário** | Permite vincular a NOP a um cenário específico do sistema (ex.: Vendas, Drop Shipping — Fatura, Drop Shipping — Remessa). Cada cenário só pode estar associado a uma única NOP |

### 2.2. Configuração da Matriz Impostos vinculada à NOP

Navegue até a página **Matriz Impostos** (`TaxesMatrixFNX`). Cada registro de Matriz Impostos pode ter um **Cód. NOP** associado. Nessa matriz você define:

- **CFOP - Mesmo Estado**, **CFOP - Fora Estado**, **CFOP - Fora Brasil**
- **CFOP - Usuário Final Mesmo Estado**, **CFOP - Usuário Final Fora Estado**
- **Código CST ICMS**, CSTs de PIS/COFINS, IPI etc.
- **Área Impostos**, **Tipo Documento Fiscal**, **Natureza Operação**
- Indicador de operação para consumidor final

Essa é a matriz que define *quais* códigos fiscais serão atribuídos às linhas do documento.

### 2.3. Configuração da Matriz Fiscal (regra de seleção)

Navegue até a página **Matriz Fiscal de Clientes** (`CustomersFiscalMatrixFNX`) ou **Matriz Fiscal de Fornecedores** (`VendorsFiscalMatrixFNX`). A Matriz Fiscal funciona como uma **regra de seleção** que liga:

> **Cód. NOP** + **Cliente/Fornecedor** (ou categoria de participante) + **Item** (ou categoria de item / NCM) → **Cód. Matriz Impostos**

Ou seja, quando o sistema precisa determinar os códigos fiscais de uma linha, ele busca na Matriz Fiscal qual Matriz Impostos usar, levando em conta a NOP do documento, o participante e o item.

### 2.4. Configurações adicionais em Configurações Fiscais

Nas **Configurações Fiscais** (`FiscalSetupFNX`), existem dois pares de campos que afetam o comportamento da Matriz Fiscal:

- **Tipo Categoria de Item Venda** e **Tipo Categoria de Item Compra**: definem se o campo **Cód. Categoria Item** da Matriz Fiscal referencia a **Categoria de Item** padrão do BC ou o **Gr. Contábil Produto**. A escolha se aplica a todas as linhas da Matriz Fiscal correspondente.

- **Admin. Matriz Fiscal de Vendas** e **Admin. Matriz Fiscal de Compras**: definem qual usuário tem permissão para editar a Matriz Fiscal. Em branco, qualquer usuário pode editar. Preenchido, apenas o usuário indicado consegue modificar as linhas.

---

## 3. Matriz Fiscal de Clientes e Matriz Fiscal de Fornecedores

A **Matriz Fiscal** é a peça que conecta a NOP ao documento. Ela é a **regra de seleção** que determina *qual* Matriz Impostos será aplicada em cada linha, com base na combinação de NOP, participante e item.

Existem duas páginas separadas:

- **Matriz Fiscal de Clientes**, para documentos de venda
- **Matriz Fiscal de Fornecedores**, para documentos de compra

Ambas compartilham a mesma estrutura. A diferença é apenas o filtro de **Tipo de Participante** (cliente ou fornecedor).

### 3.1. Estrutura da Matriz Fiscal

Cada linha da Matriz Fiscal é composta por **critérios de filtro** (do mais genérico ao mais específico) e um **resultado**:

| Campo | Descrição |
|-------|-----------|
| **Cód. NOP** | O código NOP ao qual esta regra se aplica. Em branco, a regra vale para qualquer NOP (ou para documentos sem NOP) |
| **Cód. Categoria Participante** | Categoria do cliente ou fornecedor (ex.: "Contribuinte ICMS", "Simples Nacional", "Órgão Público"). Em branco, vale para qualquer participante |
| **Nº Cliente** / **Nº Fornecedor** | Um cliente ou fornecedor específico. Mutuamente exclusivo com a Categoria: preencher um limpa o outro automaticamente |
| **Cód. Categoria Item** | Dependendo da configuração nas **Configurações Fiscais**, pode ser a **Categoria de Item** ou o **Gr. Contábil Produto** do item. Em branco, vale para qualquer grupo |
| **Cód. NCM** | NCM específico do item. Em branco, vale para qualquer NCM |
| **Nº Item** | Um item específico. Mutuamente exclusivo com **Cód. Categoria Item** e **Cód. NCM**: preencher o item limpa os outros |
| **Cód. Matriz Impostos** | **Este é o resultado da regra**: a Matriz Impostos que será atribuída à linha do documento quando os critérios acima forem atendidos. Só são apresentadas as Matrizes Impostos que tenham o mesmo Cód. NOP da linha |

### 3.2. Como funciona a busca (lógica de correspondência)

Quando o sistema precisa determinar a Matriz Impostos de uma linha do documento, ele faz a busca na Matriz Fiscal usando os seguintes critérios:

1. **NOP**: filtra por linhas que tenham o Cód. NOP igual ao do documento **ou** em branco;
2. **Participante**: filtra pela Categoria do Participante **ou** pelo Nº do participante **ou** em branco;
3. **Item**: filtra pela Categoria de Item, NCM e/ou Nº do Item **ou** em branco;
4. **Seleção do mais específico**: o sistema pega o **último registro** que atenda a todos os filtros. Isso significa que **registros mais específicos prevalecem sobre os genéricos**.

A lógica segue o princípio "do genérico para o específico": uma regra com NOP, Cliente e Item específicos prevalece sobre uma regra que tem apenas NOP.

Há uma regra adicional para NCM: se não encontrar uma combinação exata com o NCM da linha, o sistema relaxa o filtro e tenta novamente aceitando NCM em branco. Isso permite criar regras genéricas por categoria de item sem precisar repetir para cada NCM.

### 3.3. Exemplo prático

Suponha que você tenha as seguintes NOPs cadastradas:

- `VENDA`: Venda de mercadoria
- `REMIND`: Remessa para industrialização

Na **Matriz Fiscal de Clientes**, você configuraria:

| Cód. NOP | Cód. Categoria Participante | Nº Cliente | Cód. Categoria Item | Cód. NCM | Nº Item | Cód. Matriz Impostos |
|----------|-----------------------------|------------|---------------------|----------|---------|----------------------|
| `VENDA` | | | | | | `VENDA-MERC` |
| `VENDA` | `CONTRIB` | | | | | `VENDA-CONTRIB` |
| `VENDA` | | | | | `ITEM-ESP` | `VENDA-ESPECIAL` |
| `REMIND` | | | | | | `REMIND-IND` |

Neste exemplo:

- Uma venda para um cliente contribuinte usará a Matriz Impostos `VENDA-CONTRIB`, mais específica que a regra genérica;
- Uma venda do item `ITEM-ESP` usará `VENDA-ESPECIAL`, mais específica ainda;
- Qualquer operação de remessa para industrialização usará `REMIND-IND`.

### 3.4. Ação Testar Matriz

Ambas as páginas (**Matriz Fiscal de Clientes** e **Matriz Fiscal de Fornecedores**) disponibilizam a ação **Testar Matriz**. Ela simula qual linha da Matriz Fiscal seria selecionada para uma determinada combinação de NOP, cliente ou fornecedor e item, sem precisar criar um documento real. É útil para validar a configuração antes de começar a operar.

---

## 4. Procedimento de Uso

### 4.1. Preenchimento da NOP no documento

No documento de venda ou compra, preencha o campo **Cód. NOP** no cabeçalho do pedido ou da fatura. Ao selecionar a NOP:

- O campo **Natureza Operação** é preenchido automaticamente com a descrição da NOP;
- Se a NOP tiver um **Tipo Documento Fiscal** configurado, esse campo também é preenchido automaticamente;
- Se já existirem linhas no documento e houver Matriz Fiscal configurada, o sistema pergunta se deseja aplicar a matriz fiscal a todas as linhas.

### 4.2. Aplicação automática nas linhas

Quando a Matriz Fiscal é aplicada, o sistema percorre cada linha do documento e:

- Localiza a regra na Matriz Fiscal que corresponde à NOP, ao participante e ao item;
- Atribui à linha o **Cód. Matriz Impostos** e o **Cód. NOP** correspondentes;
- A Matriz Impostos, por sua vez, preenche os CFOPs (conforme a UF do destinatário), os CSTs e os demais dados fiscais.

### 4.3. Hierarquia de NOP (cabeçalho × linha)

A NOP pode ser informada tanto no **cabeçalho** quanto na **linha** do documento. Se a linha tiver uma NOP própria, ela prevalece sobre a do cabeçalho. Isso permite que um mesmo documento tenha linhas com operações de naturezas diferentes.

### 4.4. Resultado final

No documento fiscal gerado e na NF-e, cada linha terá os CFOPs e CSTs corretos para o tipo de operação, sem necessidade de preenchimento manual.

---

## 5. Fluxo Completo

O fluxo completo de configuração e uso funciona assim:

1. **Cadastra-se a NOP** na página **Códigos NOP** (ex.: `VENDA`, `REMIND`);
2. **Cadastra-se a Matriz Impostos** com o **Cód. NOP** correspondente. Essa matriz define os CFOPs, os CSTs e os demais códigos fiscais;
3. **Cadastra-se a Matriz Fiscal** (de Clientes ou de Fornecedores) para definir **quando** cada Matriz Impostos deve ser usada, com base nas combinações de NOP, participante e item;
4. **No documento**, ao selecionar a NOP no cabeçalho, o sistema percorre as linhas e, para cada uma, consulta a Matriz Fiscal para encontrar a Matriz Impostos adequada, aplicando automaticamente os códigos fiscais.
