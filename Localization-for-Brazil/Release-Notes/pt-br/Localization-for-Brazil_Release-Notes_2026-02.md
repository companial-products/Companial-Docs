# Atualizações Fevereiro/2026

## Localization — 24.0.6.6990 <small>03/03/2026</small>

### Correções

- **Razão Analítico:** corrigido o saldo inicial por filial.

## Tax Books — 24.0.3.6992 <small>03/03/2026</small>

### Novos Recursos

- **Apuração IRPJ/CSLL:** adicionado o [acréscimo de presunção](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_IRPJCSLLPresumptionSurcharge.md) (o percentual de presunção é acrescido de 10%, por exemplo de 8% para 8,8%, sobre a parcela da receita bruta trimestral que exceder R$ 1.250.000).

### Correções

- **CIAP:** corrigido erro ao processar a apuração de ativo com vários documentos no mesmo mês.

## Localization — 24.0.6.6980 <small>02/03/2026</small>

### Novos Recursos

- **NF-e:** suporte à [Unidade de Medida Tributável (exportação) para Ativos Fixos](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_NFe-Unidade-Medida-Tributavel-Ativos.md).

### Melhorias

- **Ordem de Compra Programada:** adicionados os campos da localização.

### Correções

- **Compras/Vendas:** corrigido o erro "As legendas dos campos não devem ser duplicadas ao usar este método. Em vez disso, use UpdatePurchLinesByFieldNo".

## Localization — 24.0.6.6963 <small>26/02/2026</small>

### Correções

- **Razão Analítico Avançado:** corrigido o saldo inicial por filial.
- **MD-e:** corrigida a validação do **Cód. Filial**.
- **Documento Fiscal Simples:** corrigido o preenchimento dos CSTs pela área de impostos.

## Localization — 24.0.6.6923 <small>20/02/2026</small>

### Melhorias

- **Ordem de Venda Programada:** adicionados os campos da localização.

### Correções

- **Compras/Vendas:** corrigidas pequenas inconsistências no preenchimento do **Cód. Filial** em relação à dimensão de filial.

## Tax Books — 24.0.3.6917 <small>20/02/2026</small>

### Novos Recursos

- **EFD-Reinf:** adicionado o [Informe de Rendimentos de pessoas físicas e jurídicas](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_EFD-Reinf-Informe-Rendimentos.md), com emissão manual e envio por e-mail.

## Localization — 24.0.6.6912 <small>20/02/2026</small>

### Melhorias

- **NFS-e:** pequenos ajustes no [layout padrão do DANFSe](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md).

### Correções

- **Vendas:** corrigido o cancelamento de nota fiscal de exportação que falhava com "Já existe o registro na tabela Linha de Documento Fiscal Exportação".

## Localization — 24.0.6.6877 <small>12/02/2026</small>

### Novos Recursos

- **NFS-e:** adicionado o [layout padrão do DANFSe](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md), semelhante ao modelo nacional.

## Localization — 24.0.6.6835 <small>05/02/2026</small>

### Melhorias

- **NFS-e:** a chave de acesso passa a ser gravada quando disponível na consulta.
- **MD-e:** o **Cód. Class. Tributária IBS/CBS** do documento só é sobrescrito quando vem preenchido no XML da nota do fornecedor (vendas e documento fiscal simples).

### Correções

- **Contabilização:** corrigido o uso indevido das contas de compra/venda da **Configuração Registro Geral** quando uma conta contábil é usada como item.

## Tax Books — 24.0.3.6835 <small>03/02/2026</small>

### Novos Recursos

- **EFD ICMS/IPI:** o código GTIN passou a ser informado no registro 0200.

### Correções

- **EFD ICMS/IPI:** corrigida a inserção do código GTIN no registro 0200.
- **Kardex:** corrigido o Nº da nota fiscal no relatório.

## Changelog

| Hash | Data | Descrição | Aplicativo | Versão |
|---|---|---|---|---|
| c15c6031 | 14/01/2026 | NFSe: Gravar chave de acesso se disponível na consulta | Localization | 24.0.6.6835 |
| 98dfa2d4 | 03/02/2026 | Correção de uso indevido das contas de Compra/Venda da Configuração Registro Geral ao utilizar uma conta contábil como item | Localization | 24.0.6.6835 |
| 97d4b7d6 | 03/02/2026 | Correção Nº Nota Fiscal Relatorio Kardex | Tax Books | 24.0.3.6835 |
| fc9c4d5e | 03/02/2026 | EFD ICMS/IPI: Correção na inserção do GTINCode no registro 0200 | Tax Books | 24.0.3.6835 |
| e384b238 | 03/02/2026 | EFD ICMS/IPI: Inserção do Código GTIN no registro 0200 | Tax Books | 24.0.3.6835 |
| 5cb94e2c | 05/02/2026 | MD-e: somente sobrescreve o Cód. Class. Tributária IBS/CBS do documento se ele estiver preenchido no XML da nota fiscal do fornecedor (Vendas e Documento Fiscal Simples) | Localization | 24.0.6.6835 |
| [d373b6ef](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md) | 11/02/2026 | NFS-e: inclusão de layout padrão similar ao modelo nacional | Localization | 24.0.6.6877 |
| e618cbbe | 13/02/2026 | NFS-e: inclusão de layout padrão similar ao modelo nacional - pequenos ajustes | Localization | 24.0.6.6912 |
| aab12ac8 | 20/02/2026 | Vendas: correção de cancelamento de nota fiscal de exportação "Já existe o registro na tabela Linha de Documento Fiscal Exportação" | Localization | 24.0.6.6912 |
| [b202c485](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_EFD-Reinf-Informe-Rendimentos.md) | 20/02/2026 | EFD-Reinf: implementação de Informe de Rendimentos PF e PJ (manual e e-mail) | Tax Books | 24.0.3.6917 |
| 75d244a4 | 20/02/2026 | Compras/Vendas: correções de pequenas inconsistências no preenchimento de Cód. Filial vs Dimensão Filial | Localization | 24.0.6.6923 |
| 0cc37c6f | 20/02/2026 | Ordem Venda Programada: adição de campos da Localização | Localization | 24.0.6.6923 |
| 38f41927 | 25/02/2026 | Razão Analítico Avançado: correção de Saldo Inicial por Filial | Localization | 24.0.6.6963 |
| 50423406 | 25/02/2026 | MD-e: correção de validação do Cód. Filial | Localization | 24.0.6.6963 |
| a5ba9d32 | 26/02/2026 | Documento Fiscal Simples: correção de preenchimento de CSTs por Área de Imposto | Localization | 24.0.6.6963 |
| [5fb716b1](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_NFe-Unidade-Medida-Tributavel-Ativos.md) | 26/02/2026 | NF-e: suporte a Unidade de Medida Tributável (exportação) para Ativos Fixos | Localization | 24.0.6.6980 |
| e75fc6bb | 02/03/2026 | Compras/Vendas: correção de erro: As legendas dos campos não devem ser duplicadas ao usar este método. Em vez disso, use UpdatePurchLinesByFieldNo | Localization | 24.0.6.6980 |
| f1052834 | 02/03/2026 | Ordem Compra Programada: adição de campos da Localização | Localization | 24.0.6.6980 |
| b48d0e9b | 03/03/2026 | Razão Analítico: correção de Saldo Inicial por Filial | Localization | 24.0.6.6990 |
| dcf2828f | 25/02/2026 | CIAP: correção de erro ao processar apuração de ativo com vários documentos no mesmo mês | Tax Books | 24.0.3.6992 |
| [5f1b9850](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_IRPJCSLLPresumptionSurcharge.md) | 03/03/2026 | Apuração de IRPJ/CSLL: funcionalidade de acréscimo de presunção (ex: 10% sobre excesso de R$ 1.250.000) | Tax Books | 24.0.3.6992 |
