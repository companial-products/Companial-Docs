# EFD-Reinf: Informe de Rendimentos PF e PJ

**Data**: Fevereiro/2026

## Sumário

Esta atualização implementa a geração e o envio por e-mail do **Informe de Rendimentos** para Pessoa Física (PF) e Pessoa Jurídica (PJ) dentro do módulo EFD-Reinf. Os informes podem ser visualizados em tela, gerados em PDF ou enviados diretamente por e-mail aos fornecedores. Além disso, inclui a configuração de **Códigos de Retenção** por natureza de rendimento.

## 1. Informe de Rendimentos PJ

### O que é

O Informe de Rendimentos PJ é o **Comprovante Anual de Retenção de Rendimentos Pagos ou Creditados e de Retenção de Imposto de Renda na Fonte — Pessoa Jurídica**. Trata-se de um relatório em PDF gerado a partir dos pagamentos transmitidos com sucesso no EFD-Reinf.

### Seções do Documento

| Seção | Conteúdo |
| --- | --- |
| **Cabeçalho** | Ministério da Economia / Secretaria Especial da Receita Federal do Brasil, ano-calendário |
| **1. Fonte Pagadora** | Nome e CNPJ da empresa (dados da Informação da Empresa) |
| **2. Pessoa Jurídica Beneficiária** | Nome empresarial e CNPJ do fornecedor |
| **3. Rendimento e Imposto Retido na Fonte** | Detalhamento mensal por código de retenção: rendimento e imposto retido (IRPJ, CSLL, COFINS, PIS, Agregado) |
| **4. Informações Complementares** | Espaço para notas adicionais |
| **5. Responsável pelas Informações** | Nome do responsável (configurado na Configuração Reinf), data e assinatura |

### Como Visualizar

1. Navegue até **Reinf - Pagamentos a Pessoas Jurídicas** (página ReinfCompanyPaymentsFNX)
2. Selecione um registro com status **Sucesso**
3. No grupo de ações **Informe de Rendimentos**, clique em **Visualizar**
4. O sistema filtra automaticamente todos os períodos do ano-calendário para o fornecedor selecionado e exibe o relatório

### Pré-requisitos

- O pagamento deve ter status **Sucesso** (transmitido com êxito ao EFD-Reinf)
- O fornecedor deve ter CNPJ preenchido
- A Informação da Empresa deve ter Nome e CNPJ preenchidos
- Os **Códigos de Retenção** devem estar configurados na tabela de Códigos de Natureza de Rendimento (veja seção 3)

## 2. Informe de Rendimentos PF

### O que é

O Informe de Rendimentos PF é o **Comprovante de Rendimentos Pagos e de Imposto sobre a Renda Retido na Fonte — Pessoa Física**. Segue o mesmo modelo da IN/SRF nº 119/2000, adaptado para beneficiários pessoa física.

### Seções do Documento

| Seção | Conteúdo |
| --- | --- |
| **Cabeçalho** | Ministério da Economia / Secretaria Especial da Receita Federal do Brasil, ano-calendário |
| **1. Fonte Pagadora** | Nome e CNPJ da empresa |
| **2. Pessoa Física Beneficiária** | Nome e CPF do beneficiário |
| **3. Rendimento e Imposto Retido na Fonte** | Detalhamento mensal por código de retenção: rendimento bruto e imposto retido (IRPF) |
| **4. Informações Complementares** | Espaço para notas adicionais |
| **5. Responsável pelas Informações** | Nome do responsável, data e assinatura |

### Como Visualizar

1. Navegue até **Reinf - Pagamentos a Pessoas Físicas** (página ReinfPersonPaymentsFNX)
2. Selecione um registro com status **Sucesso**
3. No grupo de ações **Informe de Rendimentos**, clique em **Visualizar**
4. O sistema filtra automaticamente todos os períodos do ano-calendário para o fornecedor/beneficiário selecionado

### Diferenças em relação ao Informe PJ

- O beneficiário é identificado pelo **CPF** (em vez de CNPJ)
- Quando o pagamento possui CPF do beneficiário preenchido, o sistema busca o nome na tabela de beneficiários do fornecedor; caso contrário, utiliza os dados do próprio fornecedor
- O tributo detalhado é o **IRPF** (em vez de IRPJ, CSLL, COFINS, PIS e Agregado)

## 3. Códigos de Retenção por Natureza de Rendimento

### O que mudou

A tabela **Códigos de Natureza de Rendimento Reinf** (ReinfIncomeNatureCodesFNX) agora possui campos para configurar os códigos de receita (códigos de retenção) utilizados no informe de rendimentos.

### Novos Campos

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Código de Retenção (IRPF)** | Code[20] | Código de receita para IRPF — utilizado no informe PF |
| **Código de Retenção (IRPJ)** | Code[20] | Código de receita para IRPJ — utilizado no informe PJ |
| **Código de Retenção (CSLL)** | Code[20] | Código de receita para CSLL — utilizado no informe PJ |
| **Código de Retenção (COFINS)** | Code[20] | Código de receita para COFINS — utilizado no informe PJ |
| **Código de Retenção (PIS)** | Code[20] | Código de receita para PIS — utilizado no informe PJ |
| **Código de Retenção (Agregado)** | Code[20] | Código de receita para retenção agregada — utilizado no informe PJ |

Todos os campos possuem relação com a tabela **Receitas Tributárias** (TaxRevenueFNX).

### Como Configurar

1. Navegue até **Códigos de Natureza de Rendimento Reinf** (página ReinfIncomeNatureCodesFNX)
2. Para cada código de natureza de rendimento, preencha os códigos de retenção aplicáveis
3. Para informes PF, é obrigatório o preenchimento do campo **Código de Retenção (IRPF)**
4. Para informes PJ, é obrigatório o preenchimento dos campos correspondentes aos tributos que possuem valores nos documentos de pagamento (IRPJ, CSLL, COFINS, PIS ou Agregado)

## 4. Envio por E-Mail

### Como Funciona

O sistema gera o informe de rendimentos em PDF e o envia como anexo por e-mail ao fornecedor, utilizando o módulo de e-mail do Business Central.

### Enviando para um Fornecedor Específico

1. Na página de pagamentos (**Reinf - Pagamentos a Pessoas Jurídicas** ou **Reinf - Pagamentos a Pessoas Físicas**)
2. Selecione o registro do fornecedor desejado
3. No grupo de ações **Informe de Rendimentos**, clique em **Enviar Informe por Email**
4. Confirme o envio na mensagem de confirmação
5. O sistema gera o PDF, envia por e-mail e marca o registro como **E-Mail Enviado**

### Pré-requisitos para Envio

- O fornecedor deve ter o campo **E-Mail** preenchido no cadastro
- A Informação da Empresa deve ter o campo **Nome** preenchido
- O pagamento deve ter status **Sucesso**

### Detalhes do E-Mail

| Item | Informe PJ | Informe PF |
| --- | --- | --- |
| **Assunto** | Informe de Rendimentos Pagos e de Retenção na Fonte | Informe de Rendimentos Pagos e de Imposto sobre a Renda Retido na Fonte |
| **Anexo** | Informe de Rendimentos.pdf | Informe de Rendimentos.pdf |
| **Destinatários** | E-mail(s) do fornecedor (separados por ; ou ,) | E-mail(s) do fornecedor (separados por ; ou ,) |

## 5. Envio em Massa

### Como Funciona

A ação **Enviar Todos Pendentes** permite enviar os informes de rendimentos por e-mail para todos os fornecedores que ainda não receberam, de uma só vez.

### Passo a Passo

1. Na página de pagamentos, selecione qualquer registro que contenha o período desejado
2. No grupo de ações **Informe de Rendimentos**, clique em **Enviar Todos Pendentes**
3. O sistema solicitará confirmação indicando o ano-calendário
4. Após confirmação, o sistema percorre todos os pagamentos com status **Sucesso** e campo **E-Mail Enviado** desmarcado, dentro do ano-calendário
5. Para cada fornecedor (PJ) ou fornecedor/beneficiário (PF), gera o PDF e envia por e-mail
6. Uma barra de progresso exibe o fornecedor sendo processado

### Comportamento

- O sistema agrupa os pagamentos por fornecedor (PJ) ou por fornecedor + CPF do beneficiário (PF), enviando um único e-mail por grupo
- Após o envio bem-sucedido, todos os registros do grupo são marcados como **E-Mail Enviado**
- Se não houver registros pendentes de envio, o sistema exibe a mensagem: *"Não há emails pendentes para enviar."*
- Caso ocorra um erro no envio de um fornecedor, o processamento é interrompido no ponto do erro

## 6. Resumo dos Campos Novos

### Códigos de Natureza de Rendimento Reinf

| Campo | Tipo | Descrição |
| --- | --- | --- |
| Código de Retenção (IRPF) | Code[20] | Código de receita para IRPF (informe PF) |
| Código de Retenção (IRPJ) | Code[20] | Código de receita para IRPJ (informe PJ) |
| Código de Retenção (CSLL) | Code[20] | Código de receita para CSLL (informe PJ) |
| Código de Retenção (COFINS) | Code[20] | Código de receita para COFINS (informe PJ) |
| Código de Retenção (PIS) | Code[20] | Código de receita para PIS (informe PJ) |
| Código de Retenção (Agregado) | Code[20] | Código de receita para retenção agregada (informe PJ) |

### Reinf - Pagamento a Pessoa Jurídica / Pessoa Física

| Campo | Tipo | Descrição |
| --- | --- | --- |
| E-Mail Enviado | Booleano | Indica se o informe de rendimentos foi enviado por e-mail |

### Novas Ações nas Páginas de Pagamentos

| Ação | Grupo | Descrição |
| --- | --- | --- |
| Visualizar | Informe de Rendimentos | Abre o relatório do informe de rendimentos em tela |
| Enviar Informe por Email | Informe de Rendimentos | Gera o PDF e envia por e-mail ao fornecedor selecionado |
| Enviar Todos Pendentes | Informe de Rendimentos | Envia os informes por e-mail para todos os fornecedores pendentes do ano |

### Novos Relatórios

| Relatório | ID | Descrição |
| --- | --- | --- |
| Reinf - Informe de Rendimentos PJ | 52112966 | Comprovante anual de retenção — Pessoa Jurídica |
| Reinf - Informe de Rendimentos PF | 52112965 | Comprovante de rendimentos pagos — Pessoa Física |
