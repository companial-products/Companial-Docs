<!-- PR: #16 -->
# NF-e e Escrituração Fiscal: Parcelas do XML e Tipo de Pagamento

**Data**: Abril/2026

---

## Sumário

Dois ajustes relacionados à importação de NF-e e à classificação do indicador de pagamento na escrituração fiscal. As parcelas do XML passam a ser importadas independentemente do **Termos de Pagamento**, e o novo campo **Tipo de Pagamento** no **Documento Fiscal** alimenta o IND_PGTO da EFD a partir das parcelas reais do documento.

## 1. Contexto

A importação de NF-e via XML e o cálculo do indicador de pagamento (IND_PGTO) usado nas escriturações EFD-ICMS/IPI e EFD-Contribuições apresentavam duas limitações:

- As **Parcelas** (`<cobr>/<dup>`) do XML só eram criadas quando o **Termos de Pagamento** do documento de compra também tinha parcelas configuradas em quantidade compatível.
- O **Tipo de Pagamento** registrado na escrituração era derivado do **Tipo** (campo obsoleto) do **Termos de Pagamento**, um cadastro manual que não refletia necessariamente as parcelas reais do documento.

---

## 2. Importação de Parcelas do XML — Antes e Depois

### Antes

A criação das parcelas em **Parcelas Documento Compra** (`PurchaseDocumentInstallmentFNX`) dependia da quantidade de parcelas configurada no **Termos de Pagamento**:

- Se o **Termos de Pagamento** tinha **N** parcelas e o XML trazia mais que **N** → as parcelas do XML **não** eram criadas.
- Se o **Termos de Pagamento** estava sem parcelas configuradas (`NumberOfInstallmentsFNX = 0`) → também não eram criadas (caso reportado por Flavio Santos: 1 parcela no XML, prazo sem parcelas, sistema exibia "O número de parcelas do XML excede o máximo definido no sistema").

### Depois

Quando a opção **Importar Parcelas do XML** (`ImportInstallmentsFromXML`) está ativa em **Conf. Importação NF-e** (`NFeImportationSetupFNX`), todas as parcelas presentes no XML (`<cobr>/<dup>`) são criadas em **Parcelas Documento Compra** independentemente do que o **Termos de Pagamento** do documento tenha configurado.

| Cenário | Antes | Depois |
|---------|-------|--------|
| XML com 3 parcelas, **Termos de Pagamento** com 3 parcelas | Criadas | Criadas |
| XML com 3 parcelas, **Termos de Pagamento** com 1 parcela | **Não** criadas | Criadas |
| XML com 1 parcela, **Termos de Pagamento** sem parcelas | **Não** criadas (com erro) | Criadas |
| XML sem `<cobr>/<dup>` | Nada a criar | Nada a criar |

### Reflexo no registro do documento

Ao **Registrar** o documento de compra ou venda, as parcelas presentes em **Parcelas Documento Compra** ou **Parcelas Documento Venda** são respeitadas mesmo quando o **Termos de Pagamento** está sem parcelas configuradas. Antes, o registro nesse cenário gerava um único movimento de fornecedor/cliente (descartando as parcelas manuais ou importadas).

---

## 3. Tipo de Pagamento no Documento Fiscal — Antes e Depois

### Antes

O indicador IND_PGTO no registro C100 da EFD-ICMS/IPI (e equivalentes em EFD-Contribuições) era derivado do campo **Tipo** do **Termos de Pagamento** (`Payment Terms.TypeFNX`). Esse campo era preenchido manualmente no cadastro e não refletia necessariamente as parcelas reais do documento — uma compra à vista lançada com um termo cadastrado como "A Prazo" gerava IND_PGTO incorreto.

### Depois

O **Documento Fiscal** (`FiscalDocumentFNX`) ganha o novo campo **Tipo de Pagamento** (`PaymentType`), preenchido no momento do **Registro** a partir dos movimentos de cliente ou fornecedor efetivamente registrados.

#### Regra de derivação

| Movimentos registrados | Vencimento da 1ª parcela | Tipo de Pagamento |
|------------------------|--------------------------|-------------------|
| 0 | — | **Sem Pagamento** |
| 1 | ≤ Data Registro | **À Vista** |
| 1 | > Data Registro | **A Prazo** |
| 2 ou mais | qualquer | **A Prazo** |

#### Mapeamento para a EFD

| Tipo de Pagamento | IND_PGTO |
|-------------------|----------|
| À Vista | 0 |
| A Prazo | 1 |
| Outros | 2 |
| Sem Pagamento | 2 |

#### Documentos pré-existentes

Documentos registrados antes da atualização permanecem com **Tipo de Pagamento** vazio. Nesses casos, a escrituração utiliza um *fallback* automático: consulta o campo **Tipo** (obsoleto) do **Termos de Pagamento** do documento e mapeia para o novo enum (`A Vista → À Vista`, `A Prazo → A Prazo`, `Outros → Outros`). Não há necessidade de reprocessar documentos antigos.

#### Campo obsoleto

O campo **Tipo** do **Termos de Pagamento** continua existindo no banco de dados (mantido por compatibilidade AppSource) mas foi marcado como obsoleto e ocultado da página **Termos de Pagamento**. Não é mais necessário mantê-lo atualizado no cadastro — todas as derivações novas vêm do registro do documento.

---

## 4. Resumo dos Campos Novos

### Tabela Documento Fiscal (`FiscalDocumentFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tipo de Pagamento** (`PaymentType`) | Enum `FiscalDocumentPaymentTypeFNX` | Classificação do pagamento derivada das parcelas reais do documento no momento do Registro. Valores: **À Vista**, **A Prazo**, **Outros**, **Sem Pagamento**. |
