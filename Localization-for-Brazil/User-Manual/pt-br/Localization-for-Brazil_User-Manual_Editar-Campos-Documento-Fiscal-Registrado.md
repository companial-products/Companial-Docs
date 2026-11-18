<!-- Source: Sessão Horácio 2026-05-08 -->
<!-- PR: #292 -->
# Editar Campos do Documento Fiscal Registrado

**Data**: Maio/2026

---

## Sumário

A ação **Editar Campos** (`EditFields`) permite alterar manualmente informações de um **Documento Fiscal Registrado** mesmo após o registro. Ela é exposta em dois pontos da página `PostedFiscalDocumentFNX`: no cabeçalho (e nas linhas do documento) e na subpágina de impostos. A edição corrige dados fiscais para fins de transmissão da NF-e ou de escrituração, mas não altera os movimentos contábeis já registrados.

---

## 1. Contexto

Depois de registrar um documento fiscal, frequentemente é necessário ajustar uma informação para que a transmissão da NF-e/NFS-e ou a escrituração reflita o que de fato precisa ser declarado ao fisco: uma observação no livro fiscal, um CFOP, um CST, uma alíquota recalculada etc.

A função **Editar Campos** habilita um modo de edição temporário na página `PostedFiscalDocumentFNX`. Enquanto ele está ativo:

- Campos pré-definidos como editáveis ficam disponíveis para alteração e são salvos automaticamente.
- Campos travados por regra fiscal (NF-e autorizada, NFS-e transmitida) permanecem bloqueados, mesmo em modo de edição.
- Alterações em valores de imposto não refazem a contabilização original. Elas apenas atualizam os dados do documento.

Ao ativar o modo de edição, o sistema exibe a mensagem:

> *Cuidado, os dados serão salvos automaticamente.*
>
> *Esteja ciente que a atualização dos valores de impostos não afetará os valores registrados na contabilidade.*

---

## 2. Configuração Necessária

A edição de **valores de impostos** depende de três flags do tipo **Permitir Edição Impostos** (`Allow Tax Changes`), cada uma controlando um contexto distinto:

| Página | Caminho | O que libera |
|--------|---------|--------------|
| **Configurações Fiscais** (`FiscalSetupFNX`) | Configurações Fiscais → **Permitir Edição Impostos** | Edição dos impostos do **Documento Fiscal Registrado** (subpágina de impostos da página `PostedFiscalDocumentFNX`). |
| **Conf. Vendas & Cobranças** | Conf. Vendas & Cobranças → **Permitir Edição Impostos** | Edição dos impostos em **documentos de venda** (pedidos, faturas, notas de crédito) e nas linhas de imposto do **Diário Geral** quando o tipo de conta é Cliente. |
| **Conf. Compras & Pagamentos** | Conf. Compras & Pagamentos → **Permitir Edição Impostos** | Edição dos impostos em **documentos de compra** (pedidos, faturas, notas de crédito) e nas linhas de imposto do **Diário Geral** quando o tipo de conta é Fornecedor. |

Se a flag correspondente estiver desabilitada, a ação **Editar Campos** da subpágina de impostos fica indisponível no respectivo contexto.

> **Importante**: a edição dos campos do **cabeçalho** e das **linhas** do **Documento Fiscal Registrado** não depende dessas flags. Ela é sempre liberada pelo botão **Editar Campos** na página, respeitando apenas as travas de NF-e autorizada e NFS-e transmitida descritas a seguir.

---

## 3. Como Acessar

1. Abra a lista de documentos fiscais registrados e selecione o documento desejado, ou navegue diretamente para a página **Documento Fiscal Registrado** (`PostedFiscalDocumentFNX`).
2. Clique na ação **Editar Campos** no grupo **Documento** da faixa de ações. O modo de edição é ativado para o cabeçalho e as linhas do documento.
3. Para alterar valores de impostos, abra a subpágina de impostos do documento e clique novamente em **Editar Campos**, desta vez sobre o subformulário de impostos. O botão só fica habilitado se a flag **Permitir Edição Impostos** das **Configurações Fiscais** estiver ativa **e** a NF-e/NFS-e não tiver sido autorizada ou transmitida.

Ao acionar o botão **Editar Campos** novamente, o modo de edição é desligado, e os campos voltam ao estado somente leitura.

---

## 4. Campos Editáveis no Cabeçalho

Os campos abaixo ficam editáveis em modo de edição. Os marcados com **NF-e autorizada bloqueia** ficam indisponíveis quando a NF-e do documento já está autorizada pelo fisco, mesmo com o modo de edição ativo.

### Geral

| Campo | NF-e autorizada bloqueia |
|-------|--------------------------|
| **Data Documento** (`DocumentDate`) | Não |

### Fiscal

| Campo | NF-e autorizada bloqueia |
|-------|--------------------------|
| **Tipo Documento Fiscal** (`FiscalDocumentType`) | Sim |
| **Natureza Operação** (`OperationNature`) | Sim |
| **Nº Nota Fiscal** (`FiscalNo`) | Sim |
| **Série** (`PrintSerie`) | Sim |
| **Modelo Documento** (`DocumentModel`) | — (sempre somente leitura) |
| **Emitida sem I.E.** (`IssuedWithoutIE`) | Sim — veja [NF-e sem Inscrição Estadual: Contribuinte Exclusivo do IBS/CBS](Localization-for-Brazil_User-Manual_NFe-Contribuinte-Exclusivo-IBS-CBS.md) |
| **Observ. Livro Fiscal** (`FiscalBookRemarks`) | Não |

### NF-e

| Campo | NF-e autorizada bloqueia |
|-------|--------------------------|
| **Finalidade de Emissão** (`PurposeOfIssue`) | Sim |
| **Tipo Nota de Débito** (`NFeDebitNoteType`) | Sim |
| **Tipo Nota de Crédito** (`NFeCreditNoteType`) | Sim |
| **Presença Comprador** (`BuyerPresence`) | Sim |

### Transportador

| Campo | NF-e autorizada bloqueia |
|-------|--------------------------|
| **Cód. Endereço Envio** (`ShipToCode`) | Sim |
| **Cod. Método de Envio** (`ShipmentMethodCode`) | Sim |
| **Frete por Conta** (`FreightBilledTo`) | Sim |
| **Contratante do Frete** (`FreightContractor`) | Não |
| **Cód. Transportador** (`ShippingAgentCode`) | Sim |
| **Quantidade Transportada** (`TransportedQuantity`) | Sim |
| **Especie** (`TransportSpecies`) | Sim |
| **Marca** (`TransportMarks`) | Sim |
| **Número** (`TransportNumber`) | Sim |
| **Peso Bruto** (`TransportGrossWeight`) | Sim |
| **Peso Líquido** (`TransportNetWeight`) | Sim |
| **Placa do Veículo** (`TransportLicensePlate`) | Sim |
| **UF Veiculo** (`TransportUFVehicle`) | Sim |

### Endereço de Retirada

Todos os campos do grupo **Endereço de Retirada** (`PickUpAddressGroup`) ficam editáveis em modo de edição e são bloqueados quando a NF-e está autorizada:

- **Cód. Depósito** (`PickUpLocationCode`), **Nome** (`PickUpName`), **CPF/CNPJ** (`PickUpCPFCNPJ`), **I.E.** (`PickUpIE`)
- **Endereço** (`PickUpAddress`), **Número** (`PickUpNumber`), **Complemento** (`PickUpAddress2`), **Bairro** (`PickUpDistrict`)
- **CEP** (`PickUpPostCode`), **Cidade** (`PickUpCity`), **Cód. Unidade Federal** (`PickUpTerritoryCode`), **Cód. País** (`PickUpCountryCode`)

### Exportação

| Campo | NF-e autorizada bloqueia |
|-------|--------------------------|
| **UF Embarque** (`ExportUFBoarding`) | Sim |
| **Local Embarque** (`ExportBoardingLocate`) | Sim |

---

## 5. Campos Editáveis nas Linhas

A subpágina de linhas (`PostedFiscalDocSubformFNX`) também entra em modo de edição quando o botão **Editar Campos** do cabeçalho é acionado.

### Sempre editáveis em modo de edição

| Campo | Observação |
|-------|------------|
| **Descrição** (`Description`) | Bloqueado se NF-e autorizada **ou** NFS-e própria. |
| **Código CFOP** (`CFOPCode`) | — |
| **Cód. Origem** (`OriginCode`) | — |
| **Código CST ICMS** (`ICMSCSTCode`) | — |
| **Cód. CST PIS** (`PISCSTCode`) | — |
| **Cód. CST COFINS** (`COFINSCSTCode`) | — |
| **Cód. CST IPI** (`IPICSTCode`) | — |
| **Cód. Nat. Receita PIS** (`PISCSTIncomeNature`) | — |
| **Cód. Nat. Receita COFINS** (`COFINSCSTIncomeNature`) | — |
| **Nº Ordem Cliente** (`CustomerOrderNo`) | — |
| **Nº Item Ordem Cliente** (`CustomerOrderLineNo`) | — |

### Bloqueados se NF-e autorizada

| Campo | |
|-------|---|
| **Código NCM** (`NCMCode`) | |
| **Cód. Exceção NCM** (`NCMExceptionCode`) | |
| **Nº Doc. Fiscal Ref.** (`RefFiscalDocNo`) | |
| **Chave Acesso Ref.** (`RefAccessKey`) | |
| **Número Item Ref.** (`RefItemNumber`) | |
| **Nº FCI** (`FCINo`) | |
| **Cód. Class. Trib. IBS/CBS** (`IBSCBSTaxClassCode`) | |

---

## 6. Campos Editáveis na Subpágina de Impostos

A subpágina de impostos (`TaxEntriesSubformFNX`) tem o seu próprio botão **Editar Campos**. Para acioná‑lo é necessário:

1. A flag **Permitir Edição Impostos** das **Configurações Fiscais** estar ativa.
2. O documento não ter NF-e autorizada ou cancelada, nem estar em processo de transmissão.
3. Para NFS-e própria, não ter sido autorizada nem transmitida.

Com o modo de edição ativo, os seguintes campos podem ser alterados:

| Campo | |
|-------|---|
| **Valor Base** (`TaxBaseAmount`) | |
| **Alíquota** (`TaxPercent`) | |
| **Valor do Imposto** (`TaxAmount`) | |
| **Valor Base Isenta** (`ExemptBaseAmount`) | |
| **Valor Base Outras** (`OthersBaseAmount`) | |
| **Fator de Redução de Base** (`ReductionFactor`) | |
| **Cód. Enquadramento Legal** (`TaxStatuteCode`) | |
| **Código CST** (`CSTCode`) | |
| **Margem Valor Agregado** (`MarginPercentValue`) | |
| **% Diferido** (`DeferredPercent`) | |
| **Valor Diferido** (`DeferredAmount`) | |
| **% Desonerado** (`DiscountedPercent`) | |
| **Valor Descontado** (`DiscountedAmount`) | |
| **% FCP** (`FCPPercent`) | |
| **Valor FCP** (`FCPAmount`) | |
| **Valor UF Origem** (`SourceTerritoryDifAmount`) | |
| **Valor UF Destino** (`DestTerritoryDifAmount`) | |
| **% Partilha Dif.** (`DifPartitionPercent`) | |
| **Data Pagamento** (`PaymentDate`) | |
| **CPF/CNPJ Beneficiário** (`BeneficiaryCPFCNPJ`) | |
| **Base Pagamento/Recebimento** (`PaymentReceiptBase`) | |

As linhas alteradas manualmente passam a ser exibidas em **vermelho**, conforme indicado no texto explicativo do subformulário: *"As linhas em vermelho indicam que houve uma atualização manual dos valores dos impostos"*.

---

## 7. Comportamento com NF-e Autorizada

Quando a NF-e do documento já está autorizada, o sistema verifica se o documento está autorizado ou cancelado e bloqueia a maioria dos campos fiscais, independentemente de a flag **Permitir Edição Impostos** estar ativa.

### O que continua editável

Mesmo com a NF-e autorizada, o botão **Editar Campos** do cabeçalho permanece funcional, mas limitado a um conjunto restrito de campos:

- **Data Documento** (`DocumentDate`)
- **Observ. Livro Fiscal** (`FiscalBookRemarks`)
- **Contratante do Frete** (`FreightContractor`)
- **Código CFOP** (`CFOPCode`)
- **Cód. Origem** (`OriginCode`)
- **Código CST ICMS** (`ICMSCSTCode`)
- **Cód. CST PIS** (`PISCSTCode`)
- **Cód. CST COFINS** (`COFINSCSTCode`)
- **Cód. CST IPI** (`IPICSTCode`)
- **Cód. Nat. Receita PIS** (`PISCSTIncomeNature`)
- **Cód. Nat. Receita COFINS** (`COFINSCSTIncomeNature`)
- **Nº Ordem Cliente** (`CustomerOrderNo`)
- **Nº Item Ordem Cliente** (`CustomerOrderLineNo`)

### O que fica bloqueado

- Demais campos fiscais do cabeçalho (Tipo Documento Fiscal, Natureza Operação, Nº Nota Fiscal, Série, Finalidade de Emissão, Tipos de Nota de Débito/Crédito, Presença Comprador).
- Todos os campos de transporte e do grupo **Endereço de Retirada**.
- Campos de exportação (**UF Embarque**, **Local Embarque**).
- Campos de referência das linhas (**Nº Doc. Fiscal Ref.**, **Chave Acesso Ref.**, **Número Item Ref.**, **Nº FCI**, **Código NCM**, **Cód. Exceção NCM**, **Cód. Class. Trib. IBS/CBS**).
- A subpágina de impostos, cujo botão **Editar Campos** fica desabilitado.

O mesmo critério vale para **NFS-e**: quando a NFS-e própria já foi autorizada ou transmitida, a edição de impostos é bloqueada, e a edição do cabeçalho e das linhas fica limitada ao mesmo subconjunto.

---

## 8. Reflexo Contábil das Alterações de Imposto

Esta é a regra crítica para evitar surpresa no fechamento contábil:

- Alterar valores de imposto pelo **Editar Campos** atualiza somente os dados fiscais do documento.
- Os **movimentos contábeis** (razão geral) gerados no momento do **Registro** original não são tocados.
- Se a contabilização precisar refletir os novos valores, o ajuste tem que ser feito manualmente, tipicamente via **Diário Geral**.

A mensagem exibida ao ativar o modo de edição na subpágina de impostos reforça esse ponto: *"a atualização dos valores de impostos não afetará os valores registrados na contabilidade"*.

---

## 9. Rastreabilidade e Auditoria

Todas as alterações feitas na subpágina de impostos com **Editar Campos** ficam registradas em um log de alterações específico do módulo. Para consultá-lo:

1. Abra a subpágina de impostos do **Documento Fiscal Registrado**.
2. Clique na ação **Movs. Registro Alterações** (`ChangeLogEntries`).
3. A página `RecordChangeLogEntriesFNX` exibe os movimentos com **Valor Antigo (Local)** e **Valor Novo (Local)**, permitindo identificar quem alterou o quê e quando, para fins de auditoria.

O cabeçalho do documento também tem a ação **Log de Alterações** (`ChangeLogEntries`), que dá acesso ao mesmo histórico para os campos do documento.
