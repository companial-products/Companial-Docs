<!-- PR: #316 -->
# NF-e: Visualizador de XML no Business Central

**Data**: Março/2026

## Sumário

Esta atualização adiciona um **visualizador estruturado de XML da NF-e** diretamente dentro do Business Central. Anteriormente, a única forma de inspecionar o XML era baixar o arquivo e abri-lo em um editor externo. Agora, a ação "Visualizar XML" abre uma interface interativa com abas temáticas, labels legíveis, busca global e formatação automática de campos (CNPJ, CPF, datas, valores monetários). O visualizador reconhece os campos da NT 2026.008: o valor líquido do produto, o total líquido dos produtos, o total do item e o total da NF-e com IBS/CBS/IS.

## 1. Contexto

### O que é

O XML da NF-e contém mais de 400 campos com informações fiscais detalhadas — CST, bases de cálculo, CFOP, classificações IBS/CBS, dados do emitente/destinatário, transporte, cobrança, entre outros. O DANFE (PDF) exibe apenas um subconjunto desses campos, insuficiente para análises fiscais detalhadas.

O Visualizador de XML da NF-e resolve esse problema ao apresentar todos os campos do XML em uma interface organizada por abas, com:

- **Labels legíveis** em português e inglês (baseados no XSD oficial)
- **Formatação automática** de CNPJ, CPF, CEP, telefone, datas e valores monetários
- **Busca global** que filtra campos em todas as abas simultaneamente

### Para quem

Analistas fiscais, contadores e consultores que precisam inspecionar campos específicos do XML sem sair do Business Central.

## 2. Como Acessar

O visualizador pode ser aberto a partir de diversas páginas do Business Central. Em todas elas, a ação se chama **"Visualizar XML"**.

### A partir da lista de NF-e

1. Navegue até **NF-e** (página NFeFNX)
2. Selecione uma NF-e autorizada
3. Clique em Ações > Funções > Outro > **Visualizar XML**

### A partir de Documentos Fiscais Registrados

1. Navegue até qualquer lista de documentos fiscais registrados:
**Documentos Fiscais de Venda Registrados** (PostedSalesFiscalDocuments)
**Documentos Fiscais de Compra Registrados** (PostedPurchaseFiscalDocuments)
**Documentos Fiscais de Transferência Registrados** (PostedTransferFiscalDocs)
**Documento Fiscal Registrado** (PostedFiscalDocument)
2. Selecione o documento.
3. Clique em NF-e > **Visualizar XML**

### A partir do MD-e (Manifestação do Destinatário)

1. Navegue até **Manifestações de Destinatário NF-e** (página NFeMDeFNX) ou abra o cartão **Manifestação de Destinatário NF-e** (NFeMDeCardFNX)
2. Clique na ação **Visualizar XML**

## 3. Interface do Visualizador

Ao abrir o visualizador, o XML é renderizado em uma página com **9 abas temáticas**:

| **Aba** | **Conteúdo** |
| --- | --- |
| **Identificação** | Dados gerais da NF-e: modelo, série, número, data de emissão, natureza da operação, chave de acesso |
| **Emitente** | CNPJ, razão social, inscrição estadual, endereço do emitente |
| **Destinatário** | CNPJ/CPF, razão social, inscrição estadual, endereço do destinatário |
| **Itens** | Detalhamento de cada item: código, descrição, NCM, CFOP, quantidade, valor unitário, valor líquido do produto, valor total do item, impostos (ICMS, IPI, PIS, COFINS, IBS, CBS) |
| **Totais** | Totalizadores da NF-e: base de cálculo, valor do ICMS, valor total dos produtos, total líquido dos produtos, valor total da nota, total da NF-e com IBS/CBS/IS |
| **Transporte** | Modalidade de frete, transportadora, veículo, volumes |
| **Cobrança** | Fatura e duplicatas |
| **Pagamento** | Formas de pagamento e valores |
| **Info. Adicional** | Informações complementares do contribuinte e do fisco |

### Formatação Automática

O visualizador formata automaticamente os seguintes tipos de dados:

| **Tipo** | **Exemplo no XML** | **Exibição no Visualizador** |
| --- | --- | --- |
| CNPJ | 12345678000199 | 12.345.678/0001-99 |
| CPF | 12345678901 | 123.456.789-01 |
| CEP | 01001000 | 01001-000 |
| Data/hora | 2026-03-09T14:30:00 | 09/03/2026 14:30:00 |
| Valor monetário | 1234.56 | 1.234,56 |
| Valor unitário (comercial, tributável e líquido) | 14.4371428571 | 14,4371428571 (de 2 a 10 casas decimais, como no XML) |
| Código UF | 35 | 35 - São Paulo |

### Idioma

O visualizador detecta automaticamente o idioma do Business Central:

- **Português**: labels e descrições em pt-BR
- **Inglês**: labels e descrições em en-US

### Campos da NT 2026.008

Os campos do valor líquido do produto e dos totais com IBS/CBS/IS aparecem com estes rótulos (veja quando cada um sai no XML na seção 7.3 de [NF-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFe-Reforma-Tributaria.md)):

| **Rótulo** | **Tag** | **Aba** | **Descrição** |
| --- | --- | --- | --- |
| Valor Unitário Líquido | `vUnComLiq` | Itens | Valor líquido do produto dividido pela quantidade comercial |
| Valor Líquido do Produto | `vProdLiq` | Itens | Valor do produto antes do desconto, deduzidos o ICMS, o FCP, o PIS e a COFINS nele incluídos |
| Valor Total do Item | `vItem` | Itens | Valor total do item, incluindo todos os tributos; o IBS, a CBS e o IS são somados a partir de 2027 |
| Total Líquido dos Produtos | `vProdLiqTot` | Totais | Soma dos valores líquidos dos produtos dos itens |
| Total da NF-e com IBS/CBS/IS | `vNFTot` | Totais | Soma dos valores totais dos itens (`vItem`), com o IBS, a CBS e o IS |

Os valores unitários (Valor Unitário Comercial, Valor Unitário Tributável e Valor Unitário Líquido) são exibidos com todas as casas decimais do XML, até 10. Assim é possível conferir que o unitário multiplicado pela quantidade reproduz o valor do item.

## 4. Busca Global

O campo de busca no topo do visualizador permite filtrar campos **em todas as abas simultaneamente**.

### Como usar

1. Digite o termo desejado no campo de busca (ex: ICMS, CNPJ, um valor específico como 1234.56)
2. Os campos que contêm o termo buscado são destacados com **highlight**
3. Campos que não correspondem à busca são ocultados
4. Pressione **Esc** ou clique no botão de limpar para remover o filtro

### Exemplos de busca

| **Termo** | **Resultado** |
| --- | --- |
| ICMS | Mostra todos os campos relacionados a ICMS em todas as abas |
| CNPJ | Mostra campos de CNPJ do emitente, destinatário e transportador |
| 7890 | Mostra qualquer campo cujo valor contenha 7890 |
| CST | Mostra os campos de CST de ICMS, IPI, PIS e COFINS dos itens |

## 5. Download do XML

O download do arquivo XML continua disponível dentro do visualizador:

1. Com o visualizador aberto, clique na ação **Baixar XML** na barra de ações
2. O arquivo será baixado com o nome baseado na **chave de acesso** da NF-e:
NF-e autorizada: {ChaveDeAcesso}-procNFe.xml
Evento de cancelamento: {ChaveDeAcesso}-procEventoNFe.xml

## 6. Comportamento por Status da NF-e

A ação **"Visualizar XML"** se comporta de forma diferente dependendo do status da NF-e:

| **Status da NF-e** | **Código** | **Comportamento** |
| --- | --- | --- |
| Autorizada | 100 | Abre o visualizador interativo |
| Autorizada fora do prazo | 150 | Abre o visualizador interativo |
| Qualquer outro status (rejeitada, cancelada, etc.) | — | Download direto do XML (comportamento anterior) |

**Nota**: Para NF-e em status diferente de autorizada, o comportamento permanece o mesmo de antes — o XML é baixado diretamente sem abrir o visualizador.

### Acesso via MD-e

Para registros de MD-e (Manifestação do Destinatário), o visualizador **sempre** abre independentemente do status, pois o XML já está disponível localmente.

### Acesso via Documentos Fiscais Registrados

Nas páginas de documentos fiscais registrados, o sistema localiza a NF-e correspondente pelo número do documento fiscal e aplica a mesma regra de status descrita acima.
