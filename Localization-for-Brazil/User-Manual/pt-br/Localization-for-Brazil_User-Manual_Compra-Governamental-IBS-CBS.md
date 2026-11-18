<!-- Commits: b0bc359f, 78076b70, cc4ca035, 68eb43a2, dafb6e77, a25e6ff2, d997a865 -->
<!-- PR: #280 -->
<!-- Branch: feature/rtc-oracle -->
# Compra Governamental — Redistribuição IBS/CBS

**Data**: Março/2026

---

## Sumário

Esta funcionalidade implementa o tratamento de **Compra Governamental** conforme a Lei Complementar 214/2025, permitindo a correta redistribuição dos impostos IBS e CBS em operações com entes governamentais. Quando o cliente ou fornecedor é um ente governamental, o sistema destina o IBS ao componente correspondente ao ente (federal, estadual ou municipal) e a CBS na proporção que a lei fixa para cada ano da transição, gerando a NF-e com os campos exigidos pela legislação.

Conforme a NT 2025.002 v1.40, o tipo de ente aceita também **Consórcio Público** e **Comitê Gestor do IBS**, e o tipo de operação passa a ter quatro situações. Nas operações de recebimento do pagamento com fornecimento já realizado e de fornecimento com pagamento já realizado, a NF-e referencia a nota anterior da mesma operação, informada na lista **Documentos Fiscais Anteriores** dos **Dados NF-e**, e o sistema confere essa referência no registro.

## 1. Contexto

### O que é

A Lei Complementar 214/2025 (Arts. 370, 372, 472 e 473) estabelece regras específicas para operações de venda ou prestação de serviço a entes governamentais. Nessas operações, os impostos IBS e CBS devem ser **redistribuídos**: ao invés de serem divididos entre CBS (federal), IBS-UF (estadual) e IBS-Município (municipal), o produto da arrecadação é direcionado ao componente tributário correspondente ao ente governamental comprador.

A redistribuição segue a transição do Art. 372:

- o **IBS** vai integralmente para o ente comprador desde 01/01/2027;
- a **CBS** só passa a acompanhar o IBS de Estados, Distrito Federal e Municípios a partir de 2029, em parcelas crescentes, e integralmente a partir de 2033 (veja a tabela em [Comportamento no Cálculo de Impostos](#5-comportamento-no-cálculo-de-impostos)).

Por exemplo, em uma venda de 2027 para um órgão municipal, todo o IBS calculado é concentrado no componente IBS-Município, o IBS-UF é zerado e a CBS continua como calculada. Na mesma venda em 2033, a CBS também é zerada e o IBS-Município recebe a soma dos três.

O **Consórcio Público** é tributado como o Município da sua sede, e o **Comitê Gestor do IBS** recebe o mesmo tratamento do consórcio (Art. 473, §§ 4º e 6º).

### Quando se aplica

- Operações com **clientes** ou **fornecedores** cadastrados como entes governamentais
- Somente para impostos da **Reforma Tributária** (IBS e CBS)
- Vigência a partir de **01/01/2027**

### Tipos de ente governamental

| Valor | Código na NF-e | Descrição |
|-------|----------------|-----------|
| **Federal** | 1 | Órgão da administração federal (União) |
| **Estado** | 2 | Órgão da administração estadual |
| **Distrito Federal** | 3 | Órgão do Distrito Federal |
| **Município** | 4 | Órgão da administração municipal |
| **Consórcio Público** | 5 | Consórcio público formado por entes federativos |
| **Comitê Gestor do IBS** | 6 | Comitê Gestor do Imposto sobre Bens e Serviços |

### Tipos de operação

| Valor | Código na NF-e | Quando usar | Documento anterior |
|-------|----------------|-------------|--------------------|
| **Fornecimento com pagamento posterior** | 1 | Nota da entrega dos bens ou da prestação do serviço, com o pagamento a receber depois | Não se informa |
| **Recebimento do pagamento com fornecimento já realizado** | 2 | Nota do recebimento do pagamento de um fornecimento que já teve nota emitida (tipo 1) | **Obrigatório**: exatamente uma NF-e de tipo 1 |
| **Fornecimento com pagamento já realizado** | 3 | Nota da entrega dos bens ou da prestação do serviço já pagos antecipadamente (pagamento com nota de tipo 4) | **Obrigatório**: uma ou mais NF-e de tipo 4 |
| **Recebimento do pagamento com fornecimento posterior** | 4 | Nota do recebimento antecipado do pagamento, antes do fornecimento | Não se informa |

As operações formam pares: a nota de tipo 2 referencia a nota de tipo 1 da mesma operação, e a nota de tipo 3 referencia as notas de tipo 4 que a anteciparam.

---

## 2. Cadastro do Ente Governamental

Para que o sistema reconheça um parceiro como ente governamental, é necessário configurar o campo **Tipo Ente Governamental** (`GovEntityTypeFNX`) diretamente no cadastro do cliente ou fornecedor.

### No cadastro do cliente

1. Navegue até **Cartão do Cliente** (página `Customer Card`)
2. Localize o campo **Tipo Ente Governamental** (`GovEntityTypeFNX`)
3. Selecione o tipo do ente: **Federal**, **Estado**, **Distrito Federal**, **Município**, **Consórcio Público** ou **Comitê Gestor do IBS**

### No cadastro do fornecedor

1. Navegue até **Cartão do Fornecedor** (página `Vendor Card`)
2. Localize o campo **Tipo Ente Governamental** (`GovEntityTypeFNX`)
3. Selecione o tipo do ente

> **Nota**: Se o campo estiver em branco, o sistema trata o parceiro como não-governamental e nenhuma redistribuição é aplicada.

---

## 3. Configuração do Percentual de Redução

A LC 214/2025 (Arts. 370 e 472) prevê um **percentual redutor** uniforme para CBS e IBS em compras governamentais, atualizado anualmente.

1. Navegue até **Configuração Fiscal** (página `FiscalSetup`)
2. Preencha o campo **% Redução Compra Gov.** (`GovPurchaseReductionPercent`) com o percentual vigente

> **Nota**: Este percentual é aplicado uniformemente a CBS e IBS, conforme a legislação.

O redutor é aplicado sobre a alíquota já reduzida pela classificação tributária do item: a alíquota efetiva é a alíquota do tributo × (1 − redução do código) × (1 − redutor). Com alíquota de 0,90%, redução de 60% no código e redutor de 5%, a alíquota efetiva é 0,90% × 0,40 × 0,95 = 0,342%. Na NF-e, a redução do código vai em `pRedAliq` e o redutor em `gCompraGov/pRedutor`.

---

## 4. Operação de Compra ou Venda

Ao criar um pedido de compra ou venda para um parceiro cadastrado como ente governamental, o campo **Tipo Operação Compra Gov.** (`GovPurchaseOperationType`) será exibido automaticamente na aba de dados fiscais da NF-e.

### Em compras

1. Crie um pedido de compra para um fornecedor com **Tipo Ente Governamental** preenchido
2. Na aba **Dados Fiscais NF-e** (página `PurchaseFiscalDataNFe`), o campo **Tipo Operação Compra Gov.** será exibido
3. Selecione um dos quatro tipos de operação (veja a tabela [Tipos de operação](#tipos-de-operação))

### Em vendas

1. Crie um pedido de venda para um cliente com **Tipo Ente Governamental** preenchido
2. Na aba **Dados Fiscais NF-e** (página `SalesFiscalDataNFe`), o campo **Tipo Operação Compra Gov.** será exibido
3. Selecione um dos quatro tipos de operação (veja a tabela [Tipos de operação](#tipos-de-operação))

### Documentos fiscais anteriores (tipos 2 e 3)

Quando o **Tipo Operação Compra Gov.** é **Recebimento do pagamento com fornecimento já realizado** ou **Fornecimento com pagamento já realizado**, a página **Dados NF-e** do pedido, da fatura, da nota de crédito ou da devolução, de venda ou de compra, exibe a lista **Documentos Fiscais Anteriores** logo abaixo do grupo geral. Nos tipos 1 e 4 a lista não aparece.

Para cada NF-e anterior, inclua uma linha:

| Campo | Descrição |
|-------|-----------|
| **Chave de Acesso NF-e** (`NFeAccessKey`) | Chave de acesso de 44 posições da NF-e anterior. Use a pesquisa do campo para escolher uma nota registrada no sistema ou digite a chave de uma nota emitida fora do Business Central. Espaços digitados são removidos, e o sistema confere o tamanho e o dígito verificador ao sair do campo. |
| **Nº do Documento Fiscal** (`FiscalDocumentNo`) | Preenchido automaticamente quando a chave pertence a um documento fiscal registrado no sistema. Fica em branco para notas emitidas fora do Business Central. |

A pesquisa da **Chave de Acesso NF-e** mostra apenas os documentos fiscais que atendem a todas as condições:

- registrados e com NF-e própria autorizada;
- do mesmo cliente (em vendas) ou fornecedor (em compras) do documento;
- de compra governamental, com o tipo de operação par do documento atual: na nota de tipo 2, as notas de tipo 1; na nota de tipo 3, as notas de tipo 4.

Ao escolher um documento, a chave e o **Nº do Documento Fiscal** são preenchidos juntos. A lista aceita até 99 chaves por documento.

No registro, a lista é copiada para o documento fiscal. Um pedido faturado em partes repete as mesmas referências em cada nota.

> **Nota**: a lista não é levada pela função de copiar documento nem pelo arquivamento do pedido. Ao copiar um documento de compra governamental de tipo 2 ou 3, informe as referências de novo.

### Tipo 2: somente IBS/CBS

A NF-e de **Recebimento do pagamento com fornecimento já realizado** leva somente IBS e CBS. ICMS, ICMS-ST, IPI, PIS, COFINS, II, ISS e diferencial de alíquota de ICMS não podem ser informados nela (regra B25-80, rejeição 1001). Configure a Matriz Impostos ou a Área de Imposto desse documento sem esses tributos.

Se, depois do cálculo dos impostos, alguma linha tiver valor de um desses tributos (ou base de ICMS ou ICMS-ST), o registro é interrompido com a mensagem:

> *Linha %1: %2 está informado (base %3, valor %4), mas a NF-e do tipo de operação %5 contém somente IBS/CBS (regra B25-80, rejeição 1001). Revise os tributos da linha.*

### Validação no registro

O sistema impede o registro do documento se o parceiro possuir **Tipo Ente Governamental** preenchido, mas o **Tipo Operação Compra Gov.** estiver em branco.

Nos tipos 2 e 3, o sistema também confere a lista **Documentos Fiscais Anteriores** e apresenta todos os problemas encontrados de uma vez, antecipando as rejeições da SEFAZ:

| Situação | Mensagem |
|----------|----------|
| Tipo 2 sem nenhuma chave | *O tipo de operação %1 deve referenciar a NF-e de fornecimento na lista de documentos fiscais anteriores (rejeição 1196).* |
| Tipo 2 com mais de uma chave | *O tipo de operação %1 deve referenciar apenas uma NF-e anterior, mas a lista de documentos fiscais anteriores possui %2 (rejeição 1197).* |
| Tipo 3 sem nenhuma chave | *O tipo de operação %1 deve referenciar ao menos uma NF-e de recebimento do pagamento na lista de documentos fiscais anteriores (rejeição 1198).* |
| Chave com tamanho diferente de 44 posições | *NF-e anterior %1: a chave de acesso deve ter 44 caracteres (rejeição 1203).* |
| Dígito verificador inválido | *NF-e anterior %1: o dígito verificador da chave de acesso não é válido (rejeição 1204).* |
| Chave repetida na lista | *NF-e anterior %1: a chave de acesso está repetida na lista de documentos fiscais anteriores (rejeição 1211).* |
| Chave de outro modelo que não NF-e 55 | *NF-e anterior %1: a chave de acesso não é de uma NF-e, modelo 55 (rejeição 1209).* |
| Chave emitida por outra empresa (CNPJ base diferente do da empresa ou filial emitente) | *NF-e anterior %1: a chave de acesso foi emitida por %2, que não é a empresa (%3) (rejeição 1215).* |
| Nota do sistema que não é de compra governamental com o tipo par | *NF-e anterior %1: o documento fiscal %2 não é uma NF-e de compra governamental do tipo %3 (rejeição %4).* (1216 no tipo 2, 1217 no tipo 3) |
| Nota do sistema não autorizada (por exemplo, cancelada) | *NF-e anterior %1: a NF-e do documento fiscal %2 não está autorizada (status %3) (rejeição 1214).* |

O pareamento e a autorização só são conferidos quando a chave pertence a um documento fiscal registrado no sistema. Uma chave digitada de nota emitida fora do Business Central passa apenas pelas conferências da própria chave (tamanho, dígito verificador, repetição, modelo e emitente). A regra de que a chave não pode ser a da própria nota (rejeição 1212) fica a cargo da SEFAZ.

---

## 5. Comportamento no Cálculo de Impostos

Ao calcular os impostos IBS/CBS em uma operação com ente governamental, o sistema executa as seguintes etapas:

1. **Calcula normalmente** os impostos CBS, IBS-UF e IBS-Município conforme as alíquotas configuradas
2. **Armazena os valores originais** nos campos **% Alíquota Compra Gov.** e **Valor Imposto Compra Gov.** de cada linha de imposto
3. **Redistribui os impostos** conforme o ente governamental:

| Tipo Ente Governamental | CBS | IBS-UF | IBS-Município |
|-------------------------|-----|--------|---------------|
| Federal | Recebe a CBS e todo o IBS | Zerado | Zerado |
| Estado / Distrito Federal | Mantém a parcela que não vai para o ente | Recebe todo o IBS e a parcela da CBS | Zerado |
| Município / Consórcio Público / Comitê Gestor do IBS | Mantém a parcela que não vai para o ente | Zerado | Recebe todo o IBS e a parcela da CBS |

A parcela da CBS que vai para o IBS do Estado, do Distrito Federal ou do Município depende da data do documento (Art. 372 da LC 214/2025):

| Data do documento | Parcela da CBS que vai para o ente | Parcela que continua como CBS |
|-------------------|------------------------------------|-------------------------------|
| 2027 e 2028 | 0% | 100% |
| 2029 | 10% | 90% |
| 2030 | 20% | 80% |
| 2031 | 30% | 70% |
| 2032 | 40% | 60% |
| A partir de 2033 | 100% | 0% |

Exemplo, com alíquotas de CBS 8,40%, IBS-UF 0,05% e IBS-Município 0,05% sobre uma base de R$ 200,00:

| Venda | CBS | IBS-UF | IBS-Município |
|-------|-----|--------|---------------|
| Sem ente governamental | 8,40% — R$ 16,80 | 0,05% — R$ 0,10 | 0,05% — R$ 0,10 |
| Para a União em 2027 | 8,50% — R$ 17,00 | Zerado | Zerado |
| Para um Estado em 2027 | 8,40% — R$ 16,80 | 0,10% — R$ 0,20 | Zerado |
| Para um Município ou Consórcio Público em 2027 | 8,40% — R$ 16,80 | Zerado | 0,10% — R$ 0,20 |

Cada valor redistribuído é a base de cálculo multiplicada pela alíquota redistribuída.

Quando o código do item tem redução de alíquota, ou há redutor configurado, a NF-e informa a alíquota de cada tributo antes da redução (`pIBSUF`, `pIBSMun`, `pCBS`) e a alíquota efetiva (`pAliqEfet`). Essa alíquota antes da redução acompanha a redistribuição: os tributos zerados saem sem alíquota, e o tributo que recebe os demais leva a soma das alíquotas, de modo que a alíquota efetiva continue sendo a alíquota informada menos a redução. Os valores originais continuam gravados nos campos **% Alíquota Compra Gov.** e **Valor Imposto Compra Gov.**, e o percentual redutor continua aplicado.

A subpágina **Linhas Valor de Imposto** (página `TaxAmountLineSubform`) exibe os campos **% Alíquota Compra Gov.** e **Valor Imposto Compra Gov.** para conferência dos valores originais antes da redistribuição.

---

## 6. Documento Fiscal Registrado

Após o registro, o **Documento Fiscal Registrado** (página `PostedFiscalDocument`) exibe os campos:

- **Tipo Ente Governamental** (`GovEntityType`) — o nível do ente governamental
- **Tipo Operação Compra Gov.** (`GovPurchaseOperationType`) — o tipo de operação selecionado

Esses campos ficam disponíveis no cabeçalho do documento fiscal para consulta e auditoria.

Nos tipos de operação 2 e 3, a página também exibe a lista **Documentos Fiscais Anteriores** copiada do documento de venda ou de compra. Se a NF-e for rejeitada, a lista e o **Tipo Operação Compra Gov.** podem ser corrigidos pela ação **Editar Campos**, enquanto a NF-e própria não estiver autorizada (veja [Editar Campos do Documento Fiscal Registrado](Localization-for-Brazil_User-Manual_Editar-Campos-Documento-Fiscal-Registrado.md)). Uma chave incluída ou alterada nesse momento passa pela conferência de tamanho e dígito verificador; as demais conferências da lista acontecem só no registro do documento de venda ou de compra.

---

## 7. Geração da NF-e

Na geração do XML da NF-e, os seguintes campos são incluídos automaticamente:

### Cabeçalho

| Tag XML | Descrição | Origem |
|---------|-----------|--------|
| `gCompraGovTpEnteGov` | Tipo de ente governamental | Documento fiscal |
| `gCompraGovTpOperGov` | Tipo de operação governamental | Documento fiscal |
| `gCompraGovPRedutor` | Percentual redutor | Movimentos de imposto (se diferente de zero) |
| `gCompraGov/refDFeAnt` | Chave de acesso de cada NF-e anterior, uma tag por chave | Lista **Documentos Fiscais Anteriores** (somente nos tipos de operação 2 e 3) |

O tipo de ente sai com o código de 1 a 6 e o tipo de operação com o código de 1 a 4, conforme as tabelas da seção [Contexto](#1-contexto). Na NF-e de tipo 2, o XML não traz os grupos de ICMS, IPI, PIS, COFINS, II, ISSQN, ICMS da UF de destino nem de devolução de IPI, apenas IBS/CBS.

### Linhas (por componente IBS/CBS)

| Tag XML | Descrição |
|---------|-----------|
| `detimpostoIBSCBSCompraGovAliqCBS` | Alíquota CBS para compra governamental |
| `detimpostoIBSCBSCompraGovValorCBS` | Valor CBS para compra governamental |
| `detimpostoIBSCBSCompraGovAliqIBSUF` | Alíquota IBS-UF para compra governamental |
| `detimpostoIBSCBSCompraGovValorIBSUF` | Valor IBS-UF para compra governamental |
| `detimpostoIBSCBSCompraGovAliqIBSMun` | Alíquota IBS-Município para compra governamental |
| `detimpostoIBSCBSCompraGovValorIBSMun` | Valor IBS-Município para compra governamental |

---

## 8. Resumo dos Campos Novos

### Configuração Fiscal (`FiscalSetup`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **% Redução Compra Gov.** (`GovPurchaseReductionPercent`) | Decimal | Percentual de redução para compras governamentais (Arts. 370/472, LC 214/2025). Uniforme para CBS e IBS, atualizado anualmente. |

### Cartão do Cliente (`Customer`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tipo Ente Governamental** (`GovEntityTypeFNX`) | Enum | Tipo de ente governamental do cliente. Determina a redistribuição IBS/CBS (Art. 473, LC 214/2025). |

### Cartão do Fornecedor (`Vendor`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tipo Ente Governamental** (`GovEntityTypeFNX`) | Enum | Tipo de ente governamental do fornecedor. Determina a redistribuição IBS/CBS (Art. 473, LC 214/2025). |

### Dados Fiscais NF-e de Compra (`PurchaseHeaderFiscalData`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tipo Operação Compra Gov.** (`GovPurchaseOperationType`) | Enum | Tipo de operação de compra governamental, de 1 a 4. Nos tipos 2 e 3 exibe a lista **Documentos Fiscais Anteriores**. |

### Dados Fiscais NF-e de Venda (`SalesHeaderFiscalData`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tipo Operação Compra Gov.** (`GovPurchaseOperationType`) | Enum | Tipo de operação de compra governamental, de 1 a 4. Nos tipos 2 e 3 exibe a lista **Documentos Fiscais Anteriores**. |

### Documento Fiscal (`FiscalDocument`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tipo Ente Governamental** (`GovEntityType`) | Enum | Tipo de ente governamental (Federal, Estado, Distrito Federal, Município, Consórcio Público, Comitê Gestor do IBS). |
| **Tipo Operação Compra Gov.** (`GovPurchaseOperationType`) | Enum | Tipo de operação de compra governamental. |

### Documento Anterior de Compra Gov. (`GovPurchasePrevDoc`)

Tabela nova. Guarda a lista **Documentos Fiscais Anteriores** do documento de venda, de compra ou do documento fiscal, uma linha por chave.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Chave de Acesso NF-e** (`NFeAccessKey`) | Code[44] | Chave de acesso da NF-e anterior da compra governamental: a NF-e de fornecimento, no tipo 2, ou a NF-e de recebimento do pagamento, no tipo 3. |
| **Nº do Documento Fiscal** (`FiscalDocumentNo`) | Code[20] | Documento fiscal registrado que tem a chave, quando a NF-e anterior foi emitida no Business Central. Não editável. |

### Movimento de Imposto de Documento Fiscal (`FiscalDocumentTaxEntry`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **% Alíquota Compra Gov.** (`GovPurchaseTaxPercent`) | Decimal | Alíquota original do imposto antes da redistribuição governamental. |
| **Valor Imposto Compra Gov.** (`GovPurchaseTaxAmount`) | Decimal | Valor original do imposto antes da redistribuição governamental. |
| **% Redução Compra Gov.** (`GovPurchaseReductionPercent`) | Decimal | Percentual de redução aplicado à compra governamental. |

### Linhas Valor de Imposto (`TaxAmountLine`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **% Alíquota Compra Gov.** (`GovPurchaseTaxPercent`) | Decimal | Alíquota original do imposto antes da redistribuição governamental. |
| **Valor Imposto Compra Gov.** (`GovPurchaseTaxAmount`) | Decimal | Valor original do imposto antes da redistribuição governamental. |
| **% Redução Compra Gov.** (`GovPurchaseReductionPercent`) | Decimal | Percentual de redução aplicado à compra governamental. |
