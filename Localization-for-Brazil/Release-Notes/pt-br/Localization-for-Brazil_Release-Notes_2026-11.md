# Atualizações Novembro/2026


## Tax Books — 25.5.3.26111320 <small>13/11/2026</small>

### Novos Recursos

- Adicionada a **Apuração de IBS/CBS**, novo módulo que monta a apuração própria da empresa a partir dos documentos fiscais (débitos, créditos potenciais, devoluções, notas de débito/crédito e estornos de crédito) e concilia automaticamente com os dados da Apuração Assistida da Receita Federal — por download ou por carga de arquivo. A conciliação classifica cada documento como Conferido, Divergente, Só no BC ou Só no fisco, busca pares em competências vizinhas quando a data de lançamento diverge da data do documento, controla o status do crédito (potencial, apropriável, apropriado ou sem direito a crédito), contabiliza a apropriação dos créditos e fecha a competência por imposto, gerando o título a pagar com o valor informado pelo fisco. Inclui as novas páginas de Configuração, Lista/Cartão de Apurações e Solicitações de IBS/CBS.

### Melhorias

- Ampliado de 30 para 60 caracteres o campo Bairro em todo o fluxo de endereços (cliente, fornecedor, filial, EFD), evitando truncamento de endereços longos no cadastro. Os relatórios de EFD ICMS/IPI, EFD-Contribuições e ECF foram ajustados para continuar gerando os registros 0005, 0150 e 0030 corretamente mesmo com o bairro mais longo, respeitando o tamanho de cada campo no leiaute oficial (por exemplo, cortando para 50 caracteres no registro 0030 da ECF quando necessário).

### Correções

- Corrigida a mensagem de erro exibida no processamento do SPED Fiscal quando um Local de localização usado para apurar o estoque está sem o Código Filial ou o Tipo Estoque preenchido: agora a mensagem identifica o Local e o item envolvidos e inclui um atalho para abrir o cartão do Local e corrigir o cadastro, em vez de um erro genérico de campo obrigatório.
- Corrigida a sugestão automática do código de natureza de rendimento na geração dos eventos mensais da EFD-Reinf: quando o IRRF de um documento é agrupado por documento e herda o item da primeira linha, a natureza de rendimento passa a ser buscada também nas demais linhas do documento, evitando que o IRRF fique sem natureza associada quando a primeira linha não tem código de serviço configurado.


## Localization — 25.5.12.26111309 <small>13/11/2026</small>

### Novos Recursos

- Adicionado preenchimento automático do Nº Item Fiscal em linhas de Conta Contábil a partir do cadastro da conta, e o registro de Fatura de Venda/Compra, Nota de Crédito e Devolução com NF-e própria passa a ser barrado quando uma linha de Conta Contábil não tem Nº Item Fiscal preenchido (nem na linha, nem na conta) — hoje essa linha simplesmente some da NF-e, deixando o documento com valor diferente do total faturado. A mensagem de erro orienta onde preencher o campo.
- Adicionada marcação "Mesmo Estado" nas linhas de Alíquotas da Exceção Imposto, permitindo cadastrar regras que valem apenas para operações internas (mesmo estado entre filial e cliente/fornecedor), sem afetar vendas, compras e documentos fiscais interestaduais.

### Melhorias

- O DANFSe do Layout Padrão passou a ser montado a partir do XML recebido do Ambiente Nacional da NFS-e, em vez de outra origem de dados.
- Ampliado o campo Bairro de endereços, de 30 para 60 caracteres, em todo o fluxo de cadastro e documentos.
- Adequado o layout da NF-e à Nota Técnica 2026.008 v1.01, preparando o valor líquido do produto e o total da nota para considerar IBS/CBS a partir de 2027.
- A mensagem de erro sobre Código de Classificação Tributária Regular igual ao Código de Classificação Tributária passou a indicar de qual cadastro vêm os códigos da linha (Exceção Imposto ou Matriz Impostos) e o que corrigir nele, com uma ação para abrir esse cadastro.
- O motivo real de falha de autenticação no Portal Único (DUIMP) passa a ser exibido na mensagem de erro, em vez do genérico "Cabeçalho HTTP inválido".

### Correções

- O DANFSe e o RPS da NFS-e passam a desconsiderar os impostos marcados como "Não Destacar", alinhando a exibição ao que já acontece na transmissão.
- Corrigido o preenchimento do campo de e-mail na geração de NFS-e pelo ambiente nacional.
- Corrigida a validação do registro de NF-e própria: uma linha de Conta Contábil sem Nº Item Fiscal, que hoje desaparece silenciosamente da nota, agora é barrada no registro (ver também em Novos Recursos).
- Corrigido o estorno de documento fiscal (HNFV), que indevidamente validava período financeiro fechado.

### Outros

- Diversas melhorias internas de qualidade de código e de infraestrutura de versionamento/CI sem impacto funcional visível ao usuário.

## Changelog

| Hash | Data | Descrição | Aplicativo | Versão |
|---|---|---|---|---|
| e718efd | 13/11 | Reforma Tributária: apuração de IBS/CBS conciliada com a Apuração Assistida L10 (#176) | Tax Books | 25.5.3.26111320 |
| b83f2ca | 13/11 | Mensagem de erro de depósito sem Cód. Filial/Tipo Estoque no processamento do SPED não orienta o que preencher L989 (#350) | Tax Books | 25.5.3.26111320 |
| c54e831 | 13/11 | Sugestão automática do código de natureza de rendimento falha para alguns registros na geração da EFD-Reinf L1002 (#354) | Tax Books | 25.5.3.26111320 |
| df24e90 | 04/11 | Endereços: ampliar o Bairro de 30 para 60 caracteres em todo o fluxo L761 (#336) | Tax Books | 25.5.3.26111320 |
| c590fd2 | 12/11 | Campo de e-mail não é preenchido na geração de NFS-e pelo ambiente nacional L919 (#346) | Localization | 25.5.12.26111309 |
| b8f598a | 11/11 | NFS-e: DANFSe do Layout Padrão montado a partir do XML do Ambiente Nacional L764 (#337) | Localization | 25.5.12.26111309 |
| 2534443 | 10/11 | NF-e própria: barrar no registro a linha de Conta Contábil sem Nº Item Fiscal, que hoje some da nota L874 (#345) | Localization | 25.5.12.26111309 |
| f94c2ac | 10/11 | Exceção Imposto: marcação Mesmo Estado nas linhas de alíquotas para regras só de operações internas L841 (#343) | Localization | 25.5.12.26111309 |
| a1d76dd | 06/11 | NFS-e: DANFSe e RPS devem desconsiderar os impostos marcados como Não Destacar, como a transmissão já faz L637 (#322) | Localization | 25.5.12.26111309 |
| df24e90 | 04/11 | Endereços: ampliar o Bairro de 30 para 60 caracteres em todo o fluxo L761 (#336) | Localization | 25.5.12.26111309 |
| 26c5489 | 04/11 | NF-e: adequar à NT 2026.008 v1.01 (valor líquido do produto e vItem/vNFTot com IBS/CBS a partir de 2027) L607 (#316) | Localization | 25.5.12.26111309 |
| c1f1c22 | 04/11 | NFS-e: mostrar na tela as mensagens da prefeitura ao consultar o lote L657 (#323) | Localization | 25.5.12.26111309 |
| 9e93fac | 03/11 | Estorno de documento fiscal (HNFV) não valida período financeiro fechado L718 (#332) | Localization | 25.5.12.26111309 |
| 16e233c | 03/11 | DUIMP: informar o motivo real da falha de autenticação no Portal Único, em vez de "Cabeçalho HTTP inválido" L487 (#320) | Localization | 25.5.12.26111309 |
| 3942b77 | 03/11 | Mensagem de erro sobre Cód. Class. Trib. Regular igual ao Cód. Class. Trib. não orienta qual campo ajustar no cadastro do item L709 (#330) | Localization | 25.5.12.26111309 |
