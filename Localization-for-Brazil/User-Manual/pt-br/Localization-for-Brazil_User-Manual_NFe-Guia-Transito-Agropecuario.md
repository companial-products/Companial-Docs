<!-- Commits: 6bc89b7 -->
# NF-e: Guia de Trânsito Agropecuário (NT 2024.003)

**Data**: Março/2026

---

## Sumário

Esta atualização implementa o suporte à **Guia de Trânsito Agropecuário** na NF-e, conforme a Nota Técnica 2024.003 da SEFAZ. Foi criado um cadastro de documentos de trânsito agropecuário e um novo campo nos documentos fiscais para vincular a guia, que é incluída automaticamente no XML da NF-e ao transmitir.

## 1. Contexto

### O que é

A Nota Técnica 2024.003 da SEFAZ introduz o grupo **agropecuário** no XML da NF-e, que inclui informações sobre defensivos agrícolas e guias de trânsito agropecuário. Essas guias são documentos obrigatórios para o transporte de animais e vegetais entre estados, emitidos por órgãos de defesa agropecuária.

### Tipos de documento suportados

| Sigla | Descrição |
| --- | --- |
| GTA | Guia de Trânsito Animal |
| TTA | Termo de Trânsito Animal |
| DTA | Documento de Transferência Animal |
| ATV | Autorização de Trânsito Vegetal |
| PTV | Permissão de Trânsito Vegetal |
| GTV | Guia de Trânsito Vegetal |
| DOF | Documento Florestal (DOF/SisFlora/SIAM) |

### Quando se aplica

- Operações de **venda, compra ou transferência** que envolvam produtos agropecuários sujeitos a controle sanitário ou ambiental
- Transporte interestadual de animais vivos, vegetais, produtos florestais ou defensivos agrícolas
- NF-e que devam informar o grupo `agropecuario/guiaTransito` conforme a NT 2024.003

## 2. Cadastro de Documentos de Trânsito Agropecuário

### Nova Tabela

Foi criada a tabela **Documento de Trânsito Agropecuário** (`AgriculturalTransitDoc`) para cadastrar as guias de trânsito. Cada registro representa um documento emitido por um órgão de defesa agropecuária.

| **Campo** | **Tipo** | **Descrição** |
| --- | --- | --- |
| **Nº** | Código (20) | Identificador único do registro. Ao ser preenchido, o **Nº Documento** é inicializado automaticamente com o mesmo valor |
| **Descrição** | Texto (100) | Descrição livre do documento para identificação |
| **Tipo de Documento** | Opção | Tipo da guia: GTA, TTA, DTA, ATV, PTV, GTV ou Documento Florestal |
| **Cód. Território** | Código (10) | UF de emissão da guia. Relacionamento com a tabela de Territórios |
| **Série** | Código (9) | Série da guia de trânsito, quando aplicável |
| **Nº Documento** | Código (9) | Número da guia de trânsito |

### Como Acessar

Navegue até **Documentos de Trânsito Agropecuário** (página `AgriculturalTransitDocs`). A página está disponível na pesquisa do Business Central.

## 3. Vinculação no Documento Fiscal

### Novo Campo

O campo **Cód. Documento Trânsito Agropecuário** (`AgriTransitDocCode`) foi adicionado aos seguintes locais:

| **Página** | **Objeto** | **Seção** |
| --- | --- | --- |
| Documento Fiscal | `FiscalDocumentFNX` | Aba NF-e |
| Documento Fiscal Registrado | `PostedFiscalDocumentFNX` | Aba NF-e |
| Documento Fiscal de Venda | `SalesFiscalDocumentFNX` | Aba NF-e |
| Documento Fiscal de Compra | `PurchaseFiscalDocumentFNX` | Aba NF-e |
| Documento Fiscal de Transferência | `TransferFiscalDocumentFNX` | Aba NF-e |
| Dados Fiscais NF-e (Venda) | `SalesFiscalDataNFeFNX` | Dados da NF-e |
| Dados Fiscais NF-e (Compra) | `PurchaseFiscalDataNFeFNX` | Dados da NF-e |

O campo possui lookup para a tabela **Documentos de Trânsito Agropecuário**, permitindo selecionar uma guia previamente cadastrada.

### Configuração

1. Cadastre o documento de trânsito em **Documentos de Trânsito Agropecuário**
2. No documento fiscal (venda, compra ou transferência), preencha o campo **Cód. Documento Trânsito Agropecuário** na aba NF-e

Para pedidos de venda e compra, o campo também está disponível na página **Dados Fiscais NF-e** do pedido. O valor é transferido automaticamente para o documento fiscal no momento do registro.

## 4. Comportamento na Geração da NF-e

Ao transmitir a NF-e, se o campo **Cód. Documento Trânsito Agropecuário** estiver preenchido, o sistema inclui automaticamente o grupo `agropecuario/guiaTransito` no XML com os seguintes dados:

| **Tag XML** | **Origem** | **Descrição** |
| --- | --- | --- |
| `tpGuia` | Tipo de Documento | Código do tipo da guia (1=GTA, 2=TTA, 3=DTA, 4=ATV, 5=PTV, 6=GTV, 7=Florestal) |
| `UFGuia` | Cód. Território | UF de emissão da guia |
| `serieGuia` | Série | Série da guia (incluída somente quando preenchida) |
| `nGuia` | Nº Documento | Número da guia de trânsito |

Se o campo estiver vazio, o grupo `agropecuario/guiaTransito` não é incluído no XML.

## 5. Resumo dos Campos Novos

### Documento de Trânsito Agropecuário (nova tabela)

| **Campo** | **Tipo** | **Descrição** |
| --- | --- | --- |
| Nº | Código (20) | Identificador do registro |
| Descrição | Texto (100) | Descrição do documento |
| Tipo de Documento | Opção | GTA, TTA, DTA, ATV, PTV, GTV ou Florestal |
| Cód. Território | Código (10) | UF de emissão |
| Série | Código (9) | Série da guia |
| Nº Documento | Código (9) | Número da guia |

### Documento Fiscal e Dados Fiscais (novo campo)

| **Campo** | **Tabela** | **Tipo** | **Descrição** |
| --- | --- | --- | --- |
| Cód. Documento Trânsito Agropecuário | `FiscalDocumentFNX` | Código (20) | Referência ao documento de trânsito agropecuário |
| Cód. Documento Trânsito Agropecuário | `SalesHeaderFiscalDataFNX` | Código (20) | Guia vinculada ao pedido de venda |
| Cód. Documento Trânsito Agropecuário | `PurchaseHeaderFiscalDataFNX` | Código (20) | Guia vinculada ao pedido de compra |
