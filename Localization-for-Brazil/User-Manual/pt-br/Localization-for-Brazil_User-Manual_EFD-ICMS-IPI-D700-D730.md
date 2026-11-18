<!-- Commits: f7dd2ee7 -->
# EFD ICMS/IPI: Registros D700 e D730 (Modelo 62 — NFCom)

**Data**: Dezembro/2025

---

## Sumário

Esta atualização adiciona os registros **D700**, **D730** e **D731** na geração do arquivo EFD ICMS/IPI, contemplando documentos fiscais do modelo 62 (NFCom — Nota Fiscal Faturamento de Comunicação). Anteriormente, documentos do modelo 62 não eram incluídos no bloco D da EFD. Com a inclusão, o bloco D passa a considerar automaticamente esses documentos quando marcados como **EFD Fiscal** na apuração.

## 1. Contexto

### O que é

O registro D700 é utilizado na EFD ICMS/IPI para informar documentos fiscais de serviços de comunicação emitidos sob o modelo 62 (NFCom). O registro D730 detalha as informações analíticas por combinação de CST, CFOP e alíquota de ICMS. O registro D731 informa o valor do Fundo de Combate à Pobreza (FCP), quando aplicável.

### Quando se aplica

A geração desses registros ocorre quando existem documentos do modelo 62 vinculados à apuração fiscal e marcados com o campo **EFD Fiscal** (`EFDFiscal`) ativado na tabela **Documento da Apuração** (`SettlementDocumentFNX`).

---

## 2. Registros Incluídos

| Registro | Nível | Descrição |
|----------|-------|-----------|
| D700 | 2 | Nota Fiscal Faturamento de Comunicação (modelo 62) |
| D730 | 3 | Registro analítico por CST, CFOP e alíquota de ICMS |
| D731 | 4 | Informações do FCP (Fundo de Combate à Pobreza) |

---

## 3. Configuração

Não há configuração adicional necessária. Os documentos do modelo 62 são automaticamente incluídos na geração da EFD quando:

1. O documento está vinculado a uma apuração fiscal
2. O campo **EFD Fiscal** (`EFDFiscal`) está marcado como verdadeiro
3. O campo **Modelo Documento Fiscal** (`DocumentModel`) contém o valor `62`

O modelo 62 foi adicionado ao filtro do bloco D (registro D001), junto aos demais modelos já suportados (07, 08, 8B, 09, 10, 11, 26, 27, 57, 67, 21, 22).

---

## 4. Comportamento na Geração da EFD

Ao gerar o arquivo **EFD ICMS/IPI** (`EFDICMSIPIFNX`), o sistema:

1. Identifica os documentos do modelo 62 vinculados à apuração
2. Se existirem, o registro D001 é gerado com indicador `0` (bloco com dados)
3. Para cada documento do modelo 62, gera um registro **D700** com os dados do cabeçalho
4. Para cada combinação de CST/CFOP/alíquota, gera um registro **D730** com os valores analíticos
5. Se houver valor de FCP, gera um registro **D731** vinculado ao D730

Documentos com **Situação Tributária** (`TributarySituation`) igual a `02` (cancelado) não geram registros D730/D731.

---

## 5. Campos do Registro D700

| Campo EFD | Descrição | Origem |
|-----------|-----------|--------|
| `IND_OPER` | Indicador de operação (0=Entrada, 1=Saída) | Tipo fiscal do documento |
| `IND_EMIT` | Indicador de emissão (0=Próprio, 1=Terceiros) | Emitente do documento |
| `COD_PART` | Código do participante | **Nº Participante** (`ParticipantNo`) |
| `COD_MOD` | Modelo do documento fiscal | `62` |
| `COD_SIT` | Situação tributária | **Situação Tributária** (`TributarySituation`) |
| `SER` | Série do documento | Série do documento fiscal |
| `NUM_DOC` | Número do documento | **Nº Nota Fiscal** (`FiscalNo`) |
| `DT_DOC` | Data de emissão | Data do documento |
| `DT_E_S` | Data de entrada/saída | Data de registro |
| `VL_DOC` | Valor total do documento | Valor contábil calculado |
| `VL_DESC` | Valor do desconto | Valor de desconto calculado |
| `VL_SERV` | Valor dos serviços | Valor contábil calculado |
| `VL_SERV_NT` | Valor de serviços não tributados | ICMS Isento + ICMS Outros |
| `VL_DA` | Valor de despesas acessórias | Outras despesas calculadas |
| `VL_BC_ICMS` | Base de cálculo do ICMS | Base ICMS calculada |
| `VL_ICMS` | Valor do ICMS | Valor ICMS calculado |
| `VL_PIS` | Valor do PIS | Valor PIS calculado |
| `VL_COFINS` | Valor do COFINS | Valor COFINS calculado |
| `CHV_DOCe` | Chave de acesso da NFCom | Chave de acesso do documento |
| `COD_MUN_DEST` | Código IBGE do município destino | Código IBGE da empresa |

---

## 6. Campos do Registro D730

| Campo EFD | Descrição | Origem |
|-----------|-----------|--------|
| `CST_ICMS` | Código da Situação Tributária | Cód. CST do registro analítico |
| `CFOP` | Código Fiscal de Operações e Prestações | CFOP do registro analítico |
| `ALIQ_ICMS` | Alíquota do ICMS | Percentual ICMS do registro analítico |
| `VL_OPR` | Valor da operação | Valor contábil |
| `VL_BC_ICMS` | Base de cálculo do ICMS | **Valor Base ICMS** (`ICMSBaseAmount`) |
| `VL_ICMS` | Valor do ICMS | Valor ICMS |
| `VL_RED_BC` | Valor da redução da base de cálculo | Valor da redução da base ICMS |

---

## 7. Registro D731 — FCP

O registro D731 é gerado apenas quando o campo **FCP** (`FCPAmount`) do registro analítico possui valor maior que zero.

| Campo EFD | Descrição | Origem |
|-----------|-----------|--------|
| `VL_FCP_OP` | Valor do FCP | Valor FCP do registro analítico |
