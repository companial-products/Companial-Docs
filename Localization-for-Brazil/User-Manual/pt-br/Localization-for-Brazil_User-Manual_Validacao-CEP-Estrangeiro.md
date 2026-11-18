<!-- Commits: e26a0b66 -->
# Validação de CEP para Endereços Estrangeiros

**Data**: Maio/2026

---

## Sumário

A Localização para o Brasil passa a **identificar automaticamente endereços estrangeiros** e, nesses casos, deixa de validar o CEP contra a tabela brasileira **CEP** (`PostCodeFNX`). A detecção usa o **Cód. BACEN** do **País/Região** vinculado ao registro. Apenas endereços com **Cód. BACEN = 1058** (Brasil) seguem o fluxo de validação de CEP brasileiro. Os demais países, bem como registros com **Cód. País** em branco, são tratados como estrangeiros e pulam o lookup. A mudança elimina erros confusos em cadastros de clientes e fornecedores no exterior e ajusta a geração de NF-e, EFD e dos campos da Reforma Tributária (IBS/CBS) para esses casos.

---

## 1. Contexto

### O problema anterior

Vários cadastros e documentos disparam, no campo **CEP**, uma validação que tenta localizar o CEP informado na tabela **CEP** da Localização (`PostCodeFNX`). Essa tabela contém apenas CEPs brasileiros e seus respectivos códigos IBGE de município. A validação era acionada para qualquer endereço, inclusive em registros sinalizados como estrangeiros, o que resultava em comportamentos indesejados:

- Mensagens de erro confusas, como *"O CEP XYZ não existe"* ou *"As Configurações Fiscais não existem"*, ao cadastrar clientes ou fornecedores no exterior.
- Falhas na geração de NF-e de exportação quando o campo `destcMun` (código do município de destino) tentava ser preenchido a partir de um CEP estrangeiro inexistente.
- Erros nas apurações fiscais (SPED) quando participantes estrangeiros tinham seu CEP editado manualmente.
- Falhas em rotinas automatizadas, como a criação de novas companhias com dados de demonstração da Microsoft, que cadastram registros com endereços de outros países.

### O novo comportamento

A Localização agora reconhece se o endereço é brasileiro antes de tentar validar o CEP. A regra é:

1. Se o **Cód. País** do registro estiver em branco, a validação é pulada.
2. Se o **País/Região** correspondente não existir no cadastro, a validação é pulada.
3. Se o **Cód. BACEN** do País/Região for diferente de `1058` (código do Brasil no BACEN), a validação é pulada, porque o endereço é estrangeiro.
4. Caso contrário (Cód. BACEN = `1058`), a validação de CEP brasileiro é executada normalmente.

### Quando se aplica

A mudança é transparente e automática. Não exige configuração nem ajuste de cadastros. Aplica-se em todos os fluxos que validam o campo CEP, incluindo cadastros (Cliente, Fornecedor, Contato, Funcionário, Depósito, Conta Bancária, Filial, etc.), cabeçalhos de Pedidos de Venda e Compra, Devoluções e Transferências, entre outros.

---

## 2. Pré-Requisito: Cód. BACEN no País/Região

A identificação de "endereço brasileiro" depende do **Cód. BACEN** estar corretamente preenchido no cadastro de **País/Região** correspondente ao Brasil. O valor esperado é **`1058`**, conforme a tabela oficial do Banco Central.

**Como verificar:**

1. Navegue até a página **Países/Regiões**.
2. Abra o registro do Brasil (geralmente `BR`).
3. Confirme o campo **Cód. BACEN** = `1058`.

> Se o campo estiver em branco, a Localização não reconhecerá nenhum endereço como brasileiro, e todos os fluxos pularão a validação de CEP. Esse cadastro normalmente já vem preenchido nas instalações existentes. Confira após a atualização.

---

## 3. Comportamento por Módulo

### 3.1. Validação de CEP em Cadastros

**O que mudou:**

Em qualquer tabela em que a Localização interage com a validação do campo **CEP** (Cliente, Fornecedor, Contato, Funcionário, Depósito, Conta Bancária, Filial, cabeçalhos de Venda/Compra, Devolução, Transferência, etc.), o lookup contra **CEP** (`PostCodeFNX`) é executado apenas quando o Cód. País indica Brasil. Para endereços estrangeiros, City, Endereço, Bairro, Cód. Território e Cód. País são preservados como informados pelo usuário, sem tentativa de preenchimento automático.

**Impacto prático:**

- Cadastrar um cliente nos Estados Unidos com CEP `90210` deixa de gerar erro.
- O campo **Criar CEPs Automaticamente** das **Configurações Fiscais** continua válido apenas para CEPs brasileiros.
- A criação de novas companhias com dados de demonstração (Contoso) que contêm endereços não brasileiros passa a funcionar.

### 3.2. NF-e/NFS-e — Destinatários no Exterior

**O que mudou:**

Na geração do XML da NF-e, o campo `destcMun` (código do município do destinatário) passa a receber o valor fixo **`9999999`** quando o documento fiscal está marcado com **Categoria = Estrangeiro**. Esse é o código de "EXTERIOR" reconhecido pela SEFAZ para destinatários fora do Brasil, conforme o manual de orientação do contribuinte.

**Impacto prático:**

- NF-es de exportação para clientes Estrangeiros são geradas sem necessidade de cadastrar um CEP fictício no cliente.
- Para documentos com categoria diferente de Estrangeiro, o comportamento permanece inalterado: o `destcMun` é obtido a partir do CEP do cliente na tabela **CEP**.

Para NFS-e, tratamentos similares são feitos em cada interface de prefeitura, conforme a necessidade.

### 3.3. Reforma Tributária — IBS/CBS Município de Ocorrência

**O que mudou:**

Para o preenchimento do **Município de Ocorrência IBS/CBS** (`cMunIncid`), campo introduzido pela Reforma Tributária (LC 214/2024) na geração da NF-e, a Localização adota a seguinte regra quando o cliente ou fornecedor é **Estrangeiro**:

- **Venda para cliente Estrangeiro** (exportação): em vez de tentar usar o CEP do Ship-to ou do Sell-to, usa o **CEP da Filial emitente** (campo CEP em **Informações da Empresa** da filial correspondente). O município de ocorrência da operação é interpretado como o local da empresa que emite o documento, e não o endereço do cliente no exterior.
- **Compra de fornecedor Estrangeiro** (importação): usa o **CEP da Filial recebedora** (Informações da Empresa). Esse já era o fallback quando o Ship-to estava em branco. Agora vale também quando o Ship-to estiver indevidamente preenchido com endereço estrangeiro.
- **Recálculo a partir de Documento Fiscal Registrado**: quando o documento fiscal está marcado com Categoria = Estrangeiro, usa o CEP da Filial do documento.

Esse comportamento se aplica somente quando o tipo de documento não é de Serviço nem de Frete (CT-e), pois esses já possuem campos próprios para o município (Cidade de Prestação de Serviços e Cidade de Destino do CT-e).

**Impacto prático:**

- Documentos fiscais de exportação e importação geram o campo `cMunIncid` automaticamente, sem erro de "CEP não encontrado".
- Mantém aderência ao espírito da norma: o IBS/CBS é apurado no município de origem da operação no Brasil, não no exterior.
- Para operações entre estabelecimentos no Brasil, o comportamento anterior (Ship-to, depois Sell-to, depois CompInfo) permanece intacto.

### 3.4. SPED — Participante Estrangeiro

**O que mudou:**

Na edição manual do campo **CEP** de um **Participante de EFD** (tabela auxiliar utilizada nas apurações de EFD-ICMS/IPI, EFD-Contribuições e EFD-Reinf), o lookup contra a tabela **CEP** é executado apenas para participantes com **Cód. País** correspondente ao Brasil (Cód. BACEN = `1058`). Para participantes estrangeiros, o **Cód. Município IBGE** permanece como `9999999`, valor que já era atribuído automaticamente nas rotinas que carregam participantes a partir de Cliente ou Fornecedor cuja Categoria é Estrangeiro.

**Impacto prático:**

- Edições no participante estrangeiro deixam de gerar o erro "CEP não encontrado".
- O código IBGE de município `9999999` (exterior) continua sendo o esperado pelos blocos de identificação de participantes nos SPEDs.

---

## 4. O que NÃO Mudou

Para endereços e participantes **brasileiros**, todos os comportamentos anteriores são preservados:

- O CEP continua sendo validado contra a tabela **CEP** (`PostCodeFNX`).
- Se o CEP não estiver cadastrado e a opção **Criar CEPs Automaticamente** estiver habilitada nas **Configurações Fiscais**, a Localização tenta criar o CEP automaticamente via consulta de serviço externo.
- Erros legítimos (CEP brasileiro digitado errado, CEP que não retornou da consulta) continuam sendo sinalizados ao usuário.
- O Cód. IBGE do município segue sendo populado a partir da tabela **CEP** em todos os fluxos fiscais (NF-e, EFD, NFS-e, CT-e, etc.).

---

## 5. Pontos de Atenção

### Cód. País em branco

Registros (Cliente, Fornecedor, Conta Bancária, etc.) que tenham o **Cód. País** em branco passam a não disparar mais a validação de CEP. Se a empresa deixa o País em branco em clientes brasileiros, assumindo "Brasil por padrão", considere preencher explicitamente o **Cód. País** com o código do Brasil cadastrado nesse ambiente. Isso garante que a validação continue ativa.

### Categoria do Cliente/Fornecedor

A categoria **Estrangeiro** no cadastro de **Cliente** e **Fornecedor** (campo **Categoria**) é o sinalizador usado pelos fluxos de NF-e (`destcMun`) e Reforma Tributária (`cMunIncid`). Garanta que clientes e fornecedores no exterior estejam corretamente classificados como **3.- Estrangeiro**.

### Cód. BACEN ausente no País/Região

Como descrito na seção 2, o **Cód. BACEN** = `1058` no País/Região do Brasil é o que permite à Localização identificar o endereço como brasileiro. Se o campo estiver em branco, a validação de CEP não será executada nem para endereços brasileiros. Verifique este cadastro após instalar a atualização.
