<!-- Commits: 15784d46 -->
# NF-e: Receituário Agronômico (Grupo ZF — Defensivos Agrícolas)

**Data**: Maio/2026

---

## Sumário

Esta atualização implementa o suporte ao **Grupo ZF02 — Defensivos Agrícolas** na NF-e modelo 55, conforme a Nota Técnica 2024.003 da SEFAZ. Foi criada uma tabela para registrar os receituários agronômicos (tags `nReceituario` e `CPFRespTec`) e novas ações nos documentos de venda, compra, transferência e fiscais para gerenciar até 20 receituários por documento. Os receituários são incluídos automaticamente no XML da NF-e ao transmitir.

## 1. Contexto

### O que é

A Nota Técnica 2024.003 da SEFAZ introduz o grupo **agropecuário** no XML da NF-e, que inclui informações de defensivos agrícolas (Grupo ZF02). Para cada defensivo, o emitente deve informar o número do receituário (tag `nReceituario`) e o CPF do responsável técnico pela emissão do receituário (tag `CPFRespTec`).

### Quando se aplica

- Operações de **venda, compra ou transferência** de defensivos agrícolas que exijam receituário agronômico
- NF-e modelo 55 cujo XML deva conter o grupo `agropecuario/Defensivo` conforme a NT 2024.003
- Até **20 receituários por documento**, limite definido pela NT

## 2. Cadastro de Receituários Agronômicos

### Nova Tabela

Foi criada a tabela **Receituário Agronômico** (`PesticidePrescription`) para armazenar os receituários vinculados a cada documento. A tabela é alimentada diretamente pela página de **Receituários Agronômicos**, acessada a partir do documento.

| **Campo** | **Tipo** | **Descrição** |
| --- | --- | --- |
| **Tipo de Documento** | Opção | Tipo do documento de origem (venda, compra, transferência ou documento fiscal) |
| **Nº Documento** | Código (20) | Número do documento de origem |
| **Nº Linha** | Inteiro | Identificador da linha (gerado automaticamente pelo Business Central) |
| **Nº do Receituário** | Código (30) | Número do receituário ou recibo da aplicação do defensivo (tag `nReceituario`, ZF03) |
| **CPF Resp. Tec.** | Texto (20) | CPF do responsável técnico pela emissão do receituário (tag `CPFRespTec`, ZF03a). O sistema valida o CPF ao informar o valor |

### Validações

- O **Nº do Receituário** é obrigatório
- O **CPF Resp. Tec.**, quando preenchido, é validado pelo sistema (formato e dígito verificador)
- O sistema impede a inclusão de mais de **20 receituários** por documento

## 3. Como Acessar

A página **Receituários Agronômicos** está disponível como ação em diversas páginas, conforme o tipo de documento. Todas as ações abrem a mesma página, filtrada pelo documento de origem.

### Documentos de Venda

| **Página**                  | **Objeto**                | **Localização da Ação**                                            |
| --------------------------- | ------------------------- | ------------------------------------------------------------------ |
| Dados NF-e (Venda)          | `SalesFiscalDataNFeFNX`   | Grupo **Agrícola** (promovida) e ação **Receituários Agronômicos** |
| Documento Fiscal de Venda   | `SalesFiscalDocumentFNX`  | Grupo **NF-e** > **Receituários Agronômicos**                      |
| Documento Fiscal Registrado | `PostedFiscalDocumentFNX` | Grupo **NF-e** > **Receituários Agronômicos**                      |

### Documentos de Compra

| **Página** | **Objeto** | **Localização da Ação** |
| --- | --- | --- |
| Dados NF-e (Compra) | `PurchaseFiscalDataNFeFNX` | Grupo **Agrícola** (promovida) e ação **Receituários Agronômicos** |
| Documento Fiscal de Compra | `PurchaseFiscalDocumentFNX` | Grupo **NF-e** > **Receituários Agronômicos** |
| Documento Fiscal Registrado | `PostedFiscalDocumentFNX` | Grupo **NF-e** > **Receituários Agronômicos** |

### Documento Fiscal e Transferência

| **Página** | **Objeto** | **Localização da Ação** |
| --- | --- | --- |
| Documento Fiscal | `FiscalDocumentFNX` | Grupo **Agrícola** > **Receituários Agronômicos** |
| Documento Fiscal de Transferência | `TransferFiscalDocumentFNX` | Grupo **NF-e** > **Receituários Agronômicos** |
| Documento Fiscal Registrado | `PostedFiscalDocumentFNX` | Grupo **NF-e** > **Receituários Agronômicos** |

## 4. Comportamento na Geração da NF-e

Ao transmitir a NF-e, se houver receituários cadastrados no documento fiscal, o sistema inclui automaticamente no XML um grupo `agropecuario/Defensivo` para cada receituário, com os seguintes dados:

| **Tag XML** | **Origem (Campo)** | **Descrição** |
| --- | --- | --- |
| `nReceituario` | Nº do Receituário | Número do receituário ou recibo do defensivo |
| `CPFRespTec` | CPF Resp. Tec. | CPF do responsável técnico pela emissão do receituário |

Se não houver receituários cadastrados, nenhum grupo `agropecuario/Defensivo` é incluído no XML.

## 5. Transferência no Registro do Documento

Quando o documento de origem (pedido, fatura, nota de crédito ou devolução de venda/compra) é registrado, os receituários cadastrados são transferidos automaticamente para o documento fiscal correspondente. Após a transferência, os registros vinculados ao documento de origem são removidos, e a edição passa a ser feita diretamente no documento fiscal.

### Comportamento na Edição após Registro

Na página **Documento Fiscal Registrado**, a edição dos receituários só é permitida enquanto a NF-e não foi autorizada, não foi cancelada e não está em processamento. Em documentos com NF-e própria já autorizada, a página exibe os registros em modo somente leitura.

### Exclusão em Cascata

Ao excluir um documento de venda ou compra (pedido, fatura, nota de crédito ou devolução), os receituários vinculados são automaticamente removidos.

## 6. Resumo dos Campos Novos

### Tabela Receituário Agronômico (`PesticidePrescription`)

| **Campo**                                | **Tipo**    | **Descrição**                                    |
| ---------------------------------------- | ----------- | ------------------------------------------------ |
| **Tipo de Documento** (`DocumentType`)   | Opção       | Venda, compra, transferência ou documento fiscal |
| **Nº Documento** (`DocumentNo`)          | Código (20) | Documento de origem                              |
| **Nº Linha** (`LineNo`)                  | Inteiro     | Identificador da linha (auto-gerado)             |
| **Nº do Receituário** (`PrescriptionNo`) | Código (30) | Tag `nReceituario` (ZF03)                        |
| **CPF Resp. Tec.** (`CPFRespTec`)        | Texto (20)  | Tag `CPFRespTec` (ZF03a), CPF validado           |
