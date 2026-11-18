<!-- Source: manual geral — Configuração e Emissão de NF-e com Reforma Tributária (IBS/CBS) na Localização do Brasil -->
<!-- PR: #280, #292, #297, #316 -->
# NF-e e Reforma Tributária — Configuração e Emissão

**Data**: Maio/2026

---

## Sumário

A NF-e (Nota Fiscal Eletrônica, modelo 55) é emitida pelo Business Central quando uma venda de mercadoria é registrada. Com a Reforma Tributária (EC 132/2023 + LC 214/2025), a partir de 01/05/2026 os grupos de IBS, CBS e Imposto Seletivo são acrescentados automaticamente ao XML da NF-e, coexistindo com ICMS, IPI, PIS e COFINS durante o período de transição (até 2033). Este manual descreve os pré-requisitos de configuração, os cadastros necessários, os campos essenciais e o passo a passo completo para emissão e transmissão de NF-e com **finalidade Normal**. Também descreve o valor líquido do produto, o total do item e o total da NF-e com IBS/CBS/IS da NT 2026.008, que o XML traz em homologação desde já e em produção nas notas emitidas a partir de 01/01/2027.

---

## 1. Visão Geral

### 1.1 O Caminho de uma NF-e

Antes de entrar nos detalhes, é importante entender o caminho que a informação percorre:

```
CONFIGURAÇÃO (feita uma vez)
       ↓
ITEM  →  NCM + Cód. NCM aponta para as alíquotas e cClassTrib aplicáveis
       ↓
DOCUMENTO FISCAL  →  sistema lê a Matriz ou Área de Impostos e monta a NF-e com todos os dados
       ↓
XML DA NF-e  →  cada campo configurado vira uma tag obrigatória no XML
```

Ou seja: **tudo parte do cadastro fiscal do item e da Matriz Impostos (ou Área de Imposto)**. Eles são os repositórios centrais de informações fiscais para a NF-e.

### 1.2 Fases do Fluxo de Emissão

O fluxo de emissão tem duas fases:

1. **Registro no BC** → criação do Documento Fiscal, cálculo de ICMS, IPI, PIS, COFINS... e agora IBS/CBS/IS pela Reforma Tributária. Além da geração do XML.
2. **Transmissão à SEFAZ** → o XML assinado é enviado ao serviço de transmissão, que o repassa à SEFAZ estadual, retornando o protocolo de autorização e o número definitivo da NF-e.

---

## 2. Pré-Requisitos de Configuração

Antes de emitir qualquer NF-e com os novos tributos, confirme os itens abaixo. Esta verificação precisa ser feita **uma única vez** (ou ao implantar um novo ambiente).

### 2.1 Status da Reforma Tributária

**Caminho**: Diga-me (Lupa) > **Configurações Fiscais** (`FiscalSetupFNX`)

| Campo | Valor esperado |
|-------|----------------|
| **Status Reforma Tributária** | `Agendada` (ativa automaticamente em 01/05/2026) ou `Habilitada` (para testes antecipados) |

> Se o campo estiver como `Desabilitada`, os grupos IBS/CBS/IS **não** serão incluídos no XML.

### 2.2 Config. NF-e da Filial

**Caminho**: Diga-me (Lupa) > **Config. NF-e** (ou no cartão da Filial > ação **Config. NF-e**)

Cada filial emissora precisa ter:

| Configuração | O que verificar |
|---|---|
| **Certificado Digital** | Certificado A1 (.pfx) importado e dentro da validade |
| **Ambiente** | `Produção` para emissão fiscal real; `Homologação` para testes |
| **UF** | UF da filial emissora — determina qual SEFAZ receberá os XMLs |
| **Transmissão Automática** | Define se a NF-e é transmitida automaticamente ao registrar ou se fica pendente para transmissão manual |

> Certificado expirado ou inválido impede qualquer transmissão à SEFAZ.

### 2.3 Áreas de Impostos com Jurisdições IBS/CBS

**Caminho**: Diga-me (Lupa) > **Áreas de Impostos** (`TaxAreasFNX`)

A Área de Impostos usada nas vendas de mercadoria precisa incluir as três novas jurisdições:

| Classificação Imposto | Tributo |
|-----------------------|---------|
| **IBSUF** | IBS Estadual |
| **IBSMun** | IBS Municipal |
| **CBS** | CBS Federal |

> Use a função **Inserir Dados da Reforma Tributária...** na página de **Jurisdições de Impostos** para criar automaticamente as três jurisdições e incluí-las em todas as Áreas de Impostos existentes.

> Se a Área de Impostos não tiver essas jurisdições cadastradas, os valores de IBS/CBS ficarão zerados no XML.

### 2.4 Classificações Tributárias IBS/CBS

**Caminho**: Diga-me > **Classificações Tributárias IBS/CBS** (`IBSCBSTaxClassifications`)

Mantenha a tabela de **cClassTrib** atualizada. Cada código de 6 dígitos define o tratamento tributário da operação (tributação integral, reduzida, isenta, diferida, etc.). Exemplos:

| Código | Significado |
|--------|-------------|
| `000001` | Tributação integral — IBS e CBS pela alíquota cheia |
| `200034` | Alíquota reduzida — fornecimento de alimentos ao consumo humano |
| `510002` | Diferimento — insumos agropecuários para produtor rural |
| `410999` | Imunidade / Não incidência — operações não onerosas |

> Use a função **Baixar Classificações Tributárias...** no menu **Ações** para importar automaticamente todos os códigos do catálogo oficial do governo.

### 2.5 Alíquotas IBS/CBS

**Caminho**: Diga-me (Lupa) > **Alíquotas IBS/CBS** (`IBSCBSTaxPercentages`)

As alíquotas têm vigência por período (mudam anualmente durante a transição 2026–2033). Verifique se as alíquotas do período atual estão cadastradas para CBS, IBS UF e IBS Municipal.

> Use a função **Inserir Alíquotas de Referência** para popular a tabela com as alíquotas de referência vigentes (CBS 0,90%, IBS UF 0,10%, IBS Municipal 0,00%).

### 2.6 Vínculos de Classificação Tributária IBS/CBS (por NCM)

**Caminho**: Diga-me (Lupa) > **Vínculos Class. Trib. IBS/CBS** (`IBSCBSTaxClassificationLinks`)

Esta tabela associa cada NCM ao seu respectivo cClassTrib conforme a legislação, permitindo que a **Matriz Impostos** preencha o código automaticamente nas linhas dos documentos.

> Use a função **Baixar Vínculos de NCM...** no menu **Ações** para importar automaticamente os vínculos NCM → cClassTrib a partir da legislação vigente. Esta tabela depende da tabela de Classificações Tributárias estar populada (seção 2.4).

---

## 3. Configuração dos Cadastros

### 3.1 Filial

**Caminho**: Diga-me (Lupa) > **Informação Filial** (`BranchInformationFNX`)

Cada estabelecimento emissor precisa ter os seguintes dados corretos:

| Campo | Por que é essencial |
|-------|---------------------|
| **CNPJ** | Identifica o emitente no XML da NF-e |
| **Inscrição Estadual (IE)** | Obrigatória para contribuintes de ICMS, com ao menos um algarismo ("ISENTO" não é aceito na transmissão); IE inválida gera rejeição na SEFAZ. O estabelecimento sem IE que é contribuinte apenas do IBS/CBS é marcado como **Contribuinte Exclusivo do IBS/CBS**: veja [NF-e sem Inscrição Estadual: Contribuinte Exclusivo do IBS/CBS](Localization-for-Brazil_User-Manual_NFe-Contribuinte-Exclusivo-IBS-CBS.md) |
| **Endereço completo** (Logradouro, Número, Bairro, CEP, Cidade, UF) | Compõe o endereço do emitente no XML |
| **Cód. CNAE** | Exigido em alguns estados; preencher quando o município ou estado solicitar |

### 3.2 Cadastro do Cliente (Participante)

**Caminho**: Cartão do Cliente → aba Geral / aba Fiscal

O cliente (destinatário da NF-e) precisa ter:

| Campo | Por que é essencial |
|-------|---------------------|
| **CNPJ ou CPF** | Validado antes do registro; CNPJ/CPF inválido bloqueia o registro |
| **Inscrição Estadual (IE)** | Para clientes contribuintes de ICMS — omissão ou valor errado gera rejeição |
| **Endereço completo** (incluindo Município e UF) | Define se a operação é interna ou interestadual, determinando o CFOP e a alíquota de ICMS correta |
| **Categoria Participante** | Define o tipo de participante (Pessoa Física, Empresa, Estrangeiro, Produtor Rural); afeta regras de tributação e validações |

### 3.3 Cadastro do Item (NCM e Dados Fiscais)

**Caminho**: Cartão do Item → aba Fiscal e aba Geral

O item do tipo **Estoque (Inventário)** ou **Não Estocavel** (mercadoria) precisa ter:

| Campo | Por que é essencial |
|-------|---------------------|
| **Cód. NCM** | Classificação da mercadoria — obrigatório no XML; determina alíquota de IPI e é a chave para o vínculo NCM → cClassTrib |
| **Origem ICMS** | Origem da mercadoria (nacional, importada, etc.) — primeiro dígito do CST ICMS |

> O campo **Cód. NCM** é a peça que conecta o item à Reforma Tributária: por meio dos **Vínculos de Classificação Tributária IBS/CBS**, o sistema usa o NCM para preencher automaticamente o **Cód. Class. Trib. IBS/CBS** nas linhas do documento fiscal.

### 3.4 Matriz Impostos

**Caminho**: Diga-me (Lupa) > **Matriz Impostos** (`TaxesMatrixFNX`)

A Matriz Impostos é o repositório central de configuração fiscal para cada tipo de operação. Cada registro reúne:

| Campo | O que configura |
|-------|----------------|
| **Código** | Identificador da configuração (ex.: `VENDA-MERC`, `VENDA-ST`) |
| **Tipo** | Venda ou Compra |
| **Área Impostos** | Qual conjunto de impostos calcular (deve incluir IBS UF, IBS Municipal e CBS — veja seção 2.3) |
| **CFOP - Mesmo Estado** | CFOP para operações dentro do estado (ex.: `5.102`) |
| **CFOP - Fora Estado** | CFOP para operações interestaduais (ex.: `6.102`) |
| **CFOP - Fora Brasil** | CFOP para exportação (ex.: `7.102`) |
| **Código CST ICMS** | Situação tributária do ICMS para esta operação |
| **Cód. CST PIS / COFINS** | Situação tributária do PIS e COFINS |
| **Cód. CST IPI** | Situação tributária do IPI |
| **Cód. Class. Trib. IBS/CBS** | **Reforma Tributária** — cClassTrib padrão para esta operação. Se preenchido, é aplicado automaticamente nas linhas do documento quando não há vínculo por NCM |
| **Cód. Class. Trib. Regular IBS/CBS** | **Reforma Tributária** — usado apenas quando a classificação principal exige tributação regular simultânea (ex.: Zona Franca de Manaus). A flag `Requer Tributação Regular` precisa estar marcada dentro do **Cód. Class. Trib. IBS/CBS** para que esse campo tenha que ser preenchido. |
| **Tipo Documento Fiscal** | Tipo de documento fiscal a ser gerado (deve ser `NF-e`) |
| **Natureza Operação** | Texto descritivo que aparece no campo Natureza de Operação da NF-e |

> Se a empresa utiliza a funcionalidade de **NOP** (Natureza da Operação), a Matriz Impostos é selecionada automaticamente pela combinação NOP + cliente + item via **Matriz Fiscal de Clientes**. Caso contrário, o usuário seleciona a Matriz Impostos diretamente no cabeçalho do documento. Caso a empresa também não use **Matriz de Imposto**, todos os campos citados devem ser preenchidos na linha manualmente.

### 3.5 Exceção Imposto (quando aplicável)

**Caminho**: Diga-me (Lupa) > **Exceções Imposto** (`TaxExceptionsFNX`)

A Exceção Imposto permite sobrescrever configurações específicas da Matriz Impostos ou Área de Imposto em cenários pontuais — por exemplo, um produto que tem alíquota diferente para determinada UF de destino ou para determinado cliente.

Para a Reforma Tributária, dois campos foram adicionados à Exceção Imposto:

| Campo | Quando usar |
|-------|-------------|
| **Cód. Class. Trib. IBS/CBS** | Quando o cClassTrib desta operação específica difere do definido na Matriz Impostos |
| **Cód. Class. Trib. Regular IBS/CBS** | Quando a exceção exige tributação regular simultânea. A flag `Requer Tributação Regular` precisa estar marcada dentro do **Cód. Class. Trib. IBS/CBS** para que esse campo tenha que ser preenchido. |

> **Prioridade:** Exceção Imposto sempre prevalece sobre a Matriz Impostos ou Área de Imposto. Se uma exceção se aplica à operação e possui cClassTrib preenchido, ele é usado em vez do da matriz/área.

---

## 4. Campos Essenciais na Ordem/Fatura de Venda

### 4.1 Cabeçalho da Ordem

| Campo | Por que é essencial |
|-------|---------------------|
| **Cód. Filial** | Define qual filial emite a NF-e e qual Config. NF-e (certificado, ambiente, UF SEFAZ) será usada |
| **Tipo Documento Fiscal** | Deve ser `NF-e` — determina que o documento gerado será transmitido à SEFAZ |
| **Cliente** | CNPJ/CPF válido e dados cadastrais completos (a localização valida antes de registrar) |
| **Matriz de Impostos** ou **Área de Imposto** | Preenchimento dos dados fiscais e impostos constantes na nota |
| **Finalidade de Emissão** | Deve ser `Normal` (padrão) para emissão comum de venda |

> O campo **Finalidade de Emissão** (`Normal`) corresponde ao `finNFe = 1` no XML da NF-e. Este manual cobre **exclusivamente** a finalidade Normal.

### 4.2 Linhas da Ordem

| Campo | Por que é essencial |
|-------|---------------------|
| **Nº Item** | Deve ser do tipo mercadoria (Estoque ou Não Estocável); a localização identifica o tipo de NF pela natureza do item |
| **Cód. NCM** | Vem do cadastro do item; copiado para a linha do documento fiscal e obrigatório no XML |
| **Cód. Área Impostos** | Define quais impostos serão calculados; deve incluir as jurisdições IBS UF, IBS Municipal e CBS |
| **Código CFOP** | Preenchido automaticamente pela Matriz Impostos com base na localização do cliente (mesmo estado, interestadual ou exterior). Se a Matriz de Imposto não for utilizada, o campo deve ser preenchido manualmente |
| **Cód. CST ICMS** | Situação tributária do ICMS — vem da Matriz Impostos (se utilizada). Se não, o campo deve ser preenchido manualmente. |
| **Cód. CST IPI** | Situação tributária do IPI — vem da Matriz Impostos (se utilizada). Se não, o campo deve ser preenchido manualmente. |
| **Cód. CST PIS / COFINS** | Situação tributária do PIS e COFINS — vem da Matriz Impostos (se utilizada). Se não, o campo deve ser preenchido manualmente. |
| **Cód. Class. Trib. IBS/CBS** | **Reforma Tributária** — código cClassTrib; preenchido automaticamente pela Matriz Impostos (via vínculo NCM ou diretamente). Deve estar corretamente parametrizado para cada linha. Se a Matriz de Imposto não for utilizada, o campo deve ser preenchido manualmente |

### 4.3 Cód. Indicador do Local da Operação (quando a operação exige)

A NT 2025.002 v1.40 criou na identificação da NF-e o **código indicador do local da operação de fornecimento** (tag `cIndOp`). Ele é obrigatório em situações específicas, como:

| Código | Situação |
|--------|----------|
| `010104` | Leilão judicial ou licitação promovida pelo poder público |
| `010105` | Constatação de irregularidade pela falta de documentação fiscal ou pelo acobertamento por documentação inidônea |

Para informar o código:

1. No documento de venda ou de compra, abra a página **Dados NF-e** (ação **NF-e** do grupo **Dados Fiscais**).
2. No grupo geral, clique em **Mostrar mais** e preencha o campo **Cód. Indicador do Local da Operação** (`OperationLocationCode`). O campo só aparece quando o tipo de documento fiscal é NF-e.
3. Escolha o código na tabela **Códs. Operação de Serviços**, a mesma usada na NFS-e.

No registro, o código é copiado para o **Documento Fiscal Registrado**, onde também pode ser corrigido pela ação **Editar Campos** enquanto a NF-e não estiver autorizada. Na geração do XML, a tag `cIndOp` sai na identificação da NF-e modelo 55 quando o campo está preenchido; em branco, a tag não é gerada. A NFC-e nunca leva a tag.

> **Atenção**: a tabela **Códs. Operação de Serviços** é cadastrada manualmente e, em geral, só tem os códigos de serviço da NFS-e. Cadastre os códigos de bens que você usa (como `010104` e `010105`), com 6 dígitos, antes de informá-los no documento — veja como cadastrar em [NFS-e e Reforma Tributária](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md#31-cód-operações-de-serviços-indop). O sistema não filtra nem valida o código escolhido: confira na tabela oficial "Código Indicador do Local de Operação" se o código é permitido na NF-e. Nos códigos `010104` e `010105`, a SEFAZ também exige o local de retirada na NF-e.

---

## 5. Passo a Passo — Emissão da NF-e

### Passo 1 — Criar a Ordem/Fatura de Venda

Atenção especial aos campos mostrados na **seção 4**, acima.

### Passo 2 — Verificar os Impostos Calculados

Antes de registrar, abra a página de **Estatísticas** da ordem. Confira:

| Imposto | O que verificar |
|---------|----------------|
| **ICMS** | Alíquota e valor correspondentes à operação (interna ou interestadual) |
| **IPI** | Valor calculado, quando aplicável |
| **PIS** | Valor calculado conforme CST e alíquota |
| **COFINS** | Valor calculado conforme CST e alíquota |
| **CBS** | Valor calculado com alíquota de referência 2026 (0,90%) |
| **IBS UF** | Valor calculado com alíquota estadual de referência 2026 (0,10%) |
| **IBS Municipal** | Valor calculado com alíquota municipal de referência 2026 (0,00%) |

> Durante 2026–2032, **ICMS/IPI/PIS/COFINS e IBS/CBS coexistem** no mesmo documento. Valores zerados para IBS/CBS indicam problema de configuração — consulte o Diagnóstico Rápido na seção 7.4.

### Passo 3 — Registrar a Ordem/Fatura

1. Na faixa de opções, clique em **Registrar** (ou **Registrar e Enviar**)
2. A localização realiza as validações fiscais automáticas:
   - CNPJ/CPF do cliente válido
   - IE do cliente preenchida quando obrigatória
   - Campos fiscais obrigatórios preenchidos (NCM, CFOP, CSTs)
   - Cód. Class. Trib. IBS/CBS presente nas linhas (quando a Reforma Tributária está ativa)
3. Se a validação passar, o BC cria:
   - A fatura registrada padrão
   - O **Documento Fiscal** com todos os impostos detalhados

---

## 6. Transmissão à SEFAZ

### 6.1 Transmissão Automática

Se o campo **Transmissão Automática** estiver habilitado na **Config. NF-e** da filial, a NF-e é transmitida à SEFAZ imediatamente ao registrar o documento, sem etapa manual adicional.

### 6.2 Transmissão pelo Documento Fiscal

1. Diga-me > **Documentos Fiscais** (`FiscalDocumentsFNX`) — filtre por Direção = Saída e Tipo = NF-e
2. Selecione o documento desejado e abra o cartão
3. Clique em **Transmitir** na faixa de opções
4. Acompanhe o retorno: status **100** (Autorizada) ou mensagem de rejeição com código e motivo

### 6.3 Retransmissão após Rejeição

Quando a SEFAZ rejeita a NF-e (status de 156 a 188, ou ≥ 201), leia o **motivo da rejeição** no campo Resultado do Documento Fiscal, corrija o dado apontado no próprio documento fiscal e transmita novamente. A maioria das rejeições envolve:

- IE inválida ou ausente no cadastro do cliente
- NCM inexistente ou inativo na TIPI
- CFOP inconsistente com a operação (ex.: CFOP interno para operação interestadual)
- CST incompatível com a alíquota (ex.: CST 00 com alíquota zero)
- cClassTrib ausente ou inválido (a partir de 01/05/2026)

> **Importante**
> Algumas rejeições não podem ser corrigidas com o documento já registrado. Sendo necessário cancelar do documento e criar um novo.

### 6.4 Status da NF-e

| Status | Significado |
|--------|------------|
| *(vazio)* | Criada, não transmitida |
| **100** | **Autorizada** — emitir DANFe, enviar e-mail ao cliente |
| **150** | Autorizada fora de prazo — equivalente a 100 |
| **101** | Cancelada com sucesso |
| **110** | **Denegada** — rejeição definitiva; não pode ser retransmitida |
| **156 a 188** | **Rejeitada** — rejeições criadas pela NT 2026.007; corrigir e retransmitir |
| **≥ 201** | **Rejeitada** — corrigir e retransmitir |

---

## 7. Referência Técnica

### 7.1 Resumo: O que Configurar e em Qual Ordem

```
PASSO 1 — Status Reforma Tributária
   Confirmar que está como "Agendada" ou "Habilitada".
   (Diga-me > Configurações Fiscais)

PASSO 2 — Alíquotas IBS/CBS
   Inserir as alíquotas de referência vigentes.
   (Diga-me > Alíquotas IBS/CBS → Inserir Alíquotas de Referência)

PASSO 3 — Classificações Tributárias IBS/CBS
   Importar todos os códigos cClassTrib do catálogo oficial.
   (Diga-me > Classificações Tributárias IBS/CBS → Baixar Classificações Tributárias...)

PASSO 4 — Jurisdições e Áreas de Impostos
   Criar as jurisdições CBS, IBS UF e IBS Municipal e incluí-las
   nas Áreas de Impostos existentes.
   (Diga-me > Jurisdições de Impostos (Brasil) → Inserir Dados da Reforma Tributária...)

PASSO 5 — Vínculos de Classificação Tributária IBS/CBS por NCM
   Importar os vínculos NCM → cClassTrib conforme a legislação.
   (Diga-me > Vínculos Class. Trib. IBS/CBS → Baixar Vínculos de NCM...)

PASSO 6 — Filial
   Verificar CNPJ, IE, endereço, Regime Tributário e Config. NF-e
   (certificado digital, ambiente, UF SEFAZ).
   (Diga-me > Informação Filial → Config. NF-e)

PASSO 7 — Cadastro do Cliente
   Verificar CNPJ/CPF, IE, endereço completo e Categoria Participante.
   (Cartão do Cliente → aba Geral / aba Fiscal)

PASSO 8 — Cadastro do Item
   Preencher Cód. NCM e Origem ICMS em cada item mercadoria.
   (Cartão do Item → aba Fiscal)

PASSO 9 — Matriz Impostos
   Para cada tipo de operação, configurar:
   ✔ Área Impostos (com IBS UF, IBS Municipal, CBS)
   ✔ CFOPs por situação (mesmo estado, interestadual, exterior)
   ✔ CSTs (ICMS, IPI, PIS, COFINS)
   ✔ Cód. Class. Trib. IBS/CBS (cClassTrib padrão)
   ✔ Tipo Documento Fiscal = NF-e
   (Diga-me > Matriz Impostos)

PASSO 10 — Exceção Imposto (se necessário)
   Para operações com tratamento diferenciado, preencher
   Cód. Class. Trib. IBS/CBS na exceção correspondente.
   (Diga-me > Exceções Imposto)
```

### 7.2 Mapeamento: XML da NF-e × Campos no BC

| Tag XML | De onde vem no BC | Obrigatório |
|---|---|---|
| `<CNPJ>` emitente | Informação Filial → CNPJ | Sempre |
| `<IE>` emitente | Informação Filial → Inscrição Estadual | Sempre (contribuinte ICMS) |
| `<CNPJ>` / `<CPF>` destinatário | Cadastro do Cliente | Sempre |
| `<IE>` destinatário | Cadastro do Cliente → Inscrição Estadual | Contribuintes ICMS |
| `<CFOP>` | Linha do documento fiscal → Código CFOP (via Matriz Impostos) | Sempre |
| `<NCM>` | Linha do documento fiscal → Cód. NCM (do item) | Sempre |
| `<CST>` ICMS | Linha do documento fiscal → Cód. CST ICMS (via Matriz Impostos) | Sempre |
| `<CST>` IPI | Linha do documento fiscal → Cód. CST IPI (via Matriz Impostos) | Quando aplicável |
| `<CST>` PIS / COFINS | Linha do documento fiscal → Cód. CST PIS/COFINS (via Matriz Impostos) | Sempre |
| `<natOp>` | Documento fiscal → Natureza Operação (via NOP ou Matriz Impostos) | Sempre |
| `<finNFe>` | Documento fiscal → Finalidade de Emissão (`1` = Normal) | Sempre |
| `<cIndOp>` | Documento fiscal → Cód. Indicador do Local da Operação | Leilão judicial, licitação pública, irregularidade fiscal (veja [4.3](#43-cód-indicador-do-local-da-operação-quando-a-operação-exige)) |
| `<IS>` | Calculado — Imposto Seletivo, quando aplicável ao produto | Produtos seletivos |
| `<IBSCBS><CST>` | Calculado — primeiros 3 dígitos do cClassTrib | Reforma Tributária |
| `<IBSCBS><cClassTrib>` | Linha do doc. fiscal → Cód. Class. Trib. IBS/CBS | Reforma Tributária |
| `<gIBSUF>` | Calculado — IBS estadual (alíquota, redução, diferimento) | Reforma Tributária |
| `<gIBSMun>` | Calculado — IBS municipal (alíquota, redução) | Reforma Tributária |
| `<gCBS>` | Calculado — CBS federal (alíquota, redução, diferimento) | Reforma Tributária |
| `<vUnComLiq>` / `<vProdLiq>` | Calculado — valor líquido do produto (veja 7.3) | Homologação; produção a partir de 01/01/2027 |
| `<vProdLiqTot>` | Calculado — soma dos valores líquidos dos itens | Homologação; produção a partir de 01/01/2027 |
| `<vItem>` | Calculado — total do item, com IBS/CBS a partir de 2027 (veja 7.3) | Homologação; produção a partir de 01/01/2027 |
| `<vNFTot>` | Calculado — soma dos `<vItem>` | Homologação; produção a partir de 01/01/2027 |

### 7.3 Valor Líquido do Produto e Total do Item (NT 2026.008)

A NT 2026.008 acrescenta ao XML da NF-e e da NFC-e o **valor líquido do produto**, que é o valor do produto sem os tributos, e passa a somar o IBS, a CBS e o IS no **total do item**. Os valores são calculados pelo serviço de transmissão da NF-e a partir do documento fiscal: não há campo nem configuração no Business Central, e todas as versões da Localização passam a gerá-los assim que o serviço é atualizado.

**Quando os campos saem no XML**

A regra segue a data de emissão da NF-e (`dhEmi`) e o **Ambiente** da **Config. NF-e** da filial:

| Ambiente | Valor líquido (`vUnComLiq`, `vProdLiq` e `vProdLiqTot`) | Total do item (`vItem`) e total com IBS/CBS/IS (`vNFTot`) |
|---|---|---|
| `Homologação` | Em todas as notas, qualquer que seja a data de emissão | Em todas as notas. O IBS/CBS só é somado ao total do item nas notas com data de emissão a partir de 01/01/2027 |
| `Produção` | Nas notas com data de emissão a partir de 01/01/2027 | Nas notas com data de emissão a partir de 01/01/2027, já somando o IBS/CBS |

As notas de produção emitidas até 31/12/2026 saem como antes, sem nenhum desses campos. Em homologação, é possível testar desde já notas com data de 2026 e notas que simulam 2027.

**Valor líquido do produto**

- **Valor líquido do item** (`vProdLiq`): valor do produto (`vProd`), antes do desconto, menos o ICMS (com o FCP), o PIS e a COFINS do item.
- O ICMS descontado é o do grupo de ICMS do item, como sai na nota, mesmo quando a base dele inclui frete, seguro, outras despesas ou IPI. No CST 51, é o ICMS depois do diferimento. Nos CSTs sem valor de ICMS destacado (40, 41, 50 e 60, e o Simples Nacional sem destaque), nenhum ICMS é descontado.
- O ICMS-ST, o FCP-ST, o ICMS desonerado, o IPI, o IBS, a CBS e o IS não são descontados: não estão no valor do produto.
- Na importação (CFOP iniciado por 3), o valor líquido é o próprio valor do produto.
- O valor líquido nunca é negativo.
- **Valor unitário líquido** (`vUnComLiq`): valor líquido dividido pela quantidade comercial, com as casas decimais necessárias (até 10) para que o unitário multiplicado pela quantidade reproduza o valor líquido, como a SEFAZ confere.
- Em nota sem quantidade, como a complementar de preço ou de imposto, o unitário líquido é zero. Se a nota for de finalidade Normal e o líquido for maior que zero, o item sai sem os dois campos.
- **Total líquido da nota** (`vProdLiqTot`): soma dos valores líquidos dos itens.

O valor líquido não é a base de cálculo do IBS/CBS, que desconta também outros valores, como o DIFAL e o ISS.

**Total do item e total da NF-e com IBS/CBS/IS**

- **Total do item** (`vItem`): a parcela do item no total da NF-e (valor do produto menos o desconto, mais frete, seguro, outras despesas, IPI e ICMS-ST). Nas notas com data de emissão a partir de 01/01/2027, soma também o IBS UF, o IBS Municipal e a CBS do item, inclusive na importação. Nas notas de 2026, não soma.
- O total do item só é informado nos itens que têm o grupo de IBS/CBS.
- **Total da NF-e com IBS/CBS/IS** (`vNFTot`): soma dos totais dos itens.
- O Imposto Seletivo e a tributação monofásica ainda não são calculados pela Localização. Quando forem, entram no total do item pela mesma regra.

**O que não muda**

- O valor do produto (`vProd`), os valores unitários (`vUnCom` e `vUnTrib`), o total dos produtos e o total da NF-e (`vNF`) continuam iguais, em qualquer data, ambiente e operação. O IBS, a CBS e o IS não entram no valor do produto.
- O preço das linhas do documento não é alterado, e o cálculo e os grupos de IBS/CBS continuam com os mesmos valores e a mesma base.
- O DANFE não ganha campos novos.
- O título e o boleto continuam com o valor de hoje.

**Exemplo**

Venda de um item, quantidade 1, sem PIS/COFINS, emitida em 2027: linha de R$ 115,27, ICMS de 12% (R$ 13,83), IBS UF de 0,1% (R$ 0,10) e CBS de 11,9% (R$ 12,07) sobre a base de R$ 101,44.

| Tag | Valor no XML |
|---|---|
| `vProd` e `vUnCom` | 115,27 |
| `vProdLiq` e `vUnComLiq` | 101,44 (115,27 − 13,83) |
| `vProdLiqTot` | 101,44 |
| `vNF` | 115,27 |
| `vItem` e `vNFTot` | 127,44 (115,27 + 0,10 + 12,07) |

A mesma nota com data de emissão em 2026, em homologação, sai com o mesmo valor líquido e com `vItem` e `vNFTot` de 115,27. Em produção, sai sem nenhum desses campos.

Os campos novos podem ser conferidos no [Visualizador de XML](Localization-for-Brazil_User-Manual_NFeXmlViewer.md).

### 7.4 Diagnóstico Rápido

| Sintoma | Causa mais provável | Onde verificar |
|---------|---------------------|----------------|
| IBS/CBS zerado ou ausente | Área de Impostos sem as jurisdições IBSUF/IBSMun/CBS | Diga-me > **Áreas de Impostos** |
| IBS/CBS zerado | Alíquota IBS/CBS não cadastrada para o período | Diga-me > **Alíquotas IBS/CBS** |
| cClassTrib vazio no XML | Linha sem o campo **Cód. Class. Trib. IBS/CBS** preenchido | Verificar o campo na linha do documento; verificar se o NCM tem vínculo em **Vínculos Class. Trib. IBS/CBS** |
| Rejeição SEFAZ por cClassTrib inválido | Código não consta na tabela de Classificações Tributárias | Reimportar com **Baixar Classificações Tributárias...** |
| ICMS zerado ou alíquota errada | Alíquotas ICMS por UF não configuradas para a combinação UF origem/destino | Diga-me > **Alíquotas ICMS por UF** |
| IPI zerado (produto industrializado) | CST IPI zerado ou alíquota IPI não configurada no NCM | Verificar CST IPI na Matriz Impostos e alíquota na tabela de NCMs |
| CFOP errado (intra × interestadual) | Matriz Impostos com CFOP de mesmo estado aplicado a operação interestadual | Verificar campos **CFOP - Mesmo Estado** e **CFOP - Fora Estado** na Matriz Impostos |
| NF-e rejeitada — IE inválida | Inscrição Estadual do cliente incorreta ou ausente | Cartão do Cliente → campo IE |
| NF-e rejeitada — NCM inválido | NCM desatualizado ou não constante na TIPI vigente | Cartão do Item → Cód. NCM |
| Certificado expirado | Certificado A1 vencido | Config. NF-e da Filial → Certificados Digitais |
| Ambiente errado | NF-e transmitida em homologação em vez de produção | Config. NF-e da Filial → campo Ambiente |

### 7.5 Pontos de Atenção

- **Período de transição 2026–2032**: ICMS, IPI, PIS e COFINS e IBS/CBS coexistem obrigatoriamente no mesmo documento. Não é necessário escolher entre um e outro — todos são calculados e enviados no XML.
- **Alíquotas provisórias**: as alíquotas de referência de 2026 (CBS 0,90%, IBS UF 0,10%) são de teste/transição e serão alteradas anualmente. Mantenha a tabela de **Alíquotas IBS/CBS** atualizada a cada publicação do governo.
- **Imposto Seletivo (IS)**: aplica-se exclusivamente a produtos específicos definidos em lei como nocivos à saúde ou ao meio ambiente (cigarros, bebidas alcoólicas, veículos de alto consumo, etc.). Para a grande maioria dos produtos, o grupo `<IS>` não é gerado no XML. Verifique com a equipe jurídica/contábil se seus produtos se enquadram.
- **DANFE da Reforma Tributária**: as notas de produção emitidas a partir de 01/12/2026, e todas as notas de homologação, saem no leiaute de DANFE da NT 2026.010, com base, alíquota e valor de IBS, CBS e IS por item e o quadro de totais do IBS/CBS/IS. Notas de produção emitidas até 30/11/2026 continuam no DANFE atual, que não exibe IBS/CBS. Veja [NF-e: DANFE no Leiaute da Reforma Tributária](Localization-for-Brazil_User-Manual_NFe-DANFE-Reforma-Tributaria.md).
- **NCM atualizado**: a TIPI é revisada periodicamente. NCMs desativados ou inexistentes geram rejeição na SEFAZ. Mantenha o cadastro de NCMs atualizado.
- **Legislação em evolução**: Notas Técnicas e regulamentações estão sendo publicadas continuamente. A localização é atualizada a cada nova NT. Acompanhe as atualizações mensais no portal.
