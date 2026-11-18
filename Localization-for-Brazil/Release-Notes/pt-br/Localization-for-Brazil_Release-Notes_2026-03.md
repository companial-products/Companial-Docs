# Atualizações Março/2026







## Localization for Brazil — 24.0.6.7179 <small>30/03/2026</small>

### Novos Recursos

- **NF-e:** Adicionado suporte à [Nota Técnica NT 2024.003 — Guia de Trânsito Agropecuário](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_NFe-Guia-Transito-Agropecuario.md)

### Melhorias

- **NF-e:** Implementada a [Prorrogação da Suspensão do ICMS na Remessa para Industrialização, conforme NT 2015.001](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_NFe-Prorrogacao-Suspensao-ICMS.md)

## Localization for Brazil - Tax Books — 24.0.3.7138 <small>23/03/2026</small>

### Correções

- **Apuração IRPJ/CSLL:** corrigido erro no recálculo quando um dos impostos já estava registrado
- **EFD-ICMS/IPI:** corrigido problema no registro 0200 que poderia não ser gerado quando o item estava presente apenas no registro C197

## Localization for Brazil - Tax Books — 24.0.3.7119 <small>20/03/2026</small>

### Correções

- **Apuração IRPJ/CSLL:** o cálculo do limite isento do acréscimo de presunção agora considera corretamente a receita dos meses anteriores do mesmo trimestre

## Localization for Brazil - Tax Books — 24.0.3.7110 <small>18/03/2026</small>

### Correções

- **EFD ICMS/IPI:** corrigido erro de divisão por zero no registro de Inventário

## Localization for Brazil — 24.0.6.7092 <small>17/03/2026</small>

### Novos Recursos

- **DUIMP:** Adicionada confirmação ao usuário antes de inserir itens provenientes da importação

### Correções

- **DUIMP:** Corrigidos os valores apresentados durante a importação
- **Rateio de Dimensões:** Corrigido problema que substituía a conta de desconto pela conta do grupo contábil de produto do rateio

## Localization for Brazil — 24.0.6.7078 <small>16/03/2026</small>

### Novos Recursos

- [Implementação de estorno de crédito IBS/CBS na Reforma Tributária](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_EstornoCreditoIBSCBS.md), com suporte ao campo gEstornoCred para controle e processamento de créditos tributários

## Localization for Brazil — 24.0.6.7051 <small>10/03/2026</small>

### Novos Recursos
- [IRPF: implementação de mecanismo de redução do imposto - Lei 15.270/2025](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_irpf-lei-15270.md)
- [Visualizador de XML da NF-e diretamente no Documento Fiscal](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_NFeXmlViewer.md)
- Reforma Tributária: estrutura inicial para Grupo Estorno de Crédito (gEstornoCred) na NF-e

### Melhorias
- Preparação para BC 28.0
- [Reports Standard Sales Quote/Order: impostos brasileiros (IPI, ST, CBS, IBS UF, IBS Mun, IS)](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_SalesQuoteOrderReportExtensions.md)
- Compras/Vendas: inclusão de impostos IPI/ST nos subpages de Cotações
- Visualizar XML da NF-e: incluído em funções de XML de teste, inclusão de dados de protocolo para notas autorizadas

### Correções
- Balancete Avançado: correção de flag para Excluir movimentos de encerramento do saldo anterior

## Changelog

| Hash | Data | Descrição | Aplicativo | Versão |
|---|---|---|---|---|
| 98ce2640 | 04/03 | Preparação para BC 28.0 | Localization | 24.0.6.7041 |
| [9aa89483](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_SalesQuoteOrderReportExtensions.md) | 04/03 | Reports Standard Sales Quote/Order: impostos brasileiros (IPI, ST, CBS, IBS UF, IBS Mun, IS) | Localization | 24.0.6.7041 |
| [41e409aa](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_irpf-lei-15270.md) | 05/03 | IRPF: implementação de mecanismo de redução do imposto - Lei 15.270/2025 | Localization | 24.0.6.7041 |
| dae22923 | 05/03 | Compras/Vendas: inclusão de impostos IPI/ST nos subpages de Cotações | Localization | 24.0.6.7041 |
| 39c9847c | 09/03 | Traduções | Localization | 24.0.6.7041 |
| [2ad7c03d](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_NFeXmlViewer.md) | 09/03 | Visualizar XML da NF-e diretamente no Documento Fiscal | Localization | 24.0.6.7041 |
| 93164df5 | 09/03 | Balancete Avançado: correção de flag para Excluir movimentos de encerramento do saldo anterior | Localization | 24.0.6.7041 |
| cd9684bb | 10/03 | Visualizar XML da NF-e: incluído em funções de XML de teste, inclusão de dados de protocolo para notas autorizadas | Localization | 24.0.6.7041 |
| [5d44ffd](../../User-Manual/pt-br/Localization-for-Brazil_User-Manual_EstornoCreditoIBSCBS.md) | 16/03 | Reforma Tributária: Estorno de Crédito IBS/CBS (gEstornoCred) | Localization | 24.0.6.7078 |
| 0826a7c | 17/03 | DUIMP: adição de confirmação para inserção de itens vindo da importação | Localization | 24.0.6.7092 |
| f3d1969 | 17/03 | DUIMP: Correção nos valores apresentados na importação | Localization | 24.0.6.7092 |
| 8fcc770 | 17/03 | Rateio de Dimensões: correção para não substituir a conta de Desconto pela conta do Grupo Contábil de Produto do Rateio | Localization | 24.0.6.7092 |
| bddd686 | 18/03 | EFD ICMS/IPI: correção de erro de divisão por zero no registro de Inventário | Tax Books | 24.0.3.7110 |
| 0e67c30 | 19/03 | Apuração IRPJ/CSLL: acréscimo de presunção - o limite isento do acréscimo de presunção agora considera a receita dos meses anteriores do mesmo trimestre | Tax Books | 24.0.3.7119 |
| 61333fb | 23/03 | Apuração IRPJ/CSLL: correção de recálculo quando um dos impostos já está registrado | Tax Books | 24.0.3.7138 |
| ec47e5f | 23/03 | EFD-ICMS/IPI: correção de registro 0200 que poderia não ser criado quando o item está apenas no registro C197 | Tax Books | 24.0.3.7138 |
| b72646a | 23/03 | NF-e: Prorrogação da Suspensão do ICMS na Remessa para Industrialização (NT 2015.001) | Localization | 24.0.6.7179 |
| 6bc89b7 | 19/03 | NF-e: Guia de Trânsito Agropecuário (NT 2024.003) | Localization | 24.0.6.7179 |
