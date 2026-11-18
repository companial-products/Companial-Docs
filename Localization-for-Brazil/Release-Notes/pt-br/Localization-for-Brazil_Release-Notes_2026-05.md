# Atualizações Maio/2026




## Localization — 25.5.3.56 <small>22/05/2026</small>

### Novos Recursos

- Adicionada opção para controlar se lançamentos de adiantamentos brasileiros devem ser incluídos na rotina de ajuste de variação cambial. O novo campo **Ajustar Taxa de Câmbio de Adiantamento** foi disponibilizado nas configurações de Vendas e de Compras, permitindo excluir esses lançamentos da reavaliação de moeda quando desejado. Em ambientes atualizados, o comportamento anterior (ajustar adiantamentos) é preservado automaticamente.

### Correções

- Corrigido o envio de e-mail de NF-e para clientes: o corpo da mensagem passou a utilizar corretamente o campo **Mensagem Adic. de E-Mail NF-e** das configurações, em vez de usar indevidamente a **Mensagem Adic. de E-Mail Transportadora NF-e**, que é destinada exclusivamente ao envio para transportadoras.

- Corrigida a exibição dos valores de PIS/COFINS no DANFSe: o campo de apuração própria agora exibe apenas os valores efetivamente apurados pela própria empresa, sem sobrescrever com valores retidos quando a apuração própria for zero.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.

## Tax Books — 25.5.1.55 <small>22/05/2026</small>

### Novos Recursos

- **EFD Contribuições — Bloco D de saída (CT-e/CT-e OS):** implementada a geração dos registros D200, D201 e D205 no relatório **Gerar EFD Contribuições**, cobrindo documentos de transporte de saída (CT-e e CT-e OS). O sistema agora consolida os documentos fiscais de transporte emitidos por período, calculando automaticamente os valores de PIS e COFINS por CST para preenchimento correto do bloco D.

### Melhorias

- **CNPJ Alfanumérico — suporte ao novo formato da Receita Federal:** todos os relatórios e obrigações fiscais (EFD ICMS/IPI, EFD Contribuições, ECD, DIRF, DCTF, DCTFWeb, GIA-SP e eventos da EFD-Reinf) passaram a utilizar a rotina correta de limpeza de máscara de CNPJ/CPF, garantindo compatibilidade com o novo formato alfanumérico de CNPJ instituído pela Receita Federal. CNPJs com letras no número de inscrição deixam de ser truncados ou gerados incorretamente nos arquivos de exportação.

### Correções

- **EFD Contribuições e EFD ICMS/IPI — código inválido no IND_PGTO do registro C100:** corrigida a geração do indicador de tipo de pagamento (à vista, a prazo, outros) no registro C100. O valor `9` (Sem Pagamento), que não é um código válido para o campo IND_PGTO nestes SPEDs, era gerado indevidamente; agora operações sem esse tipo de pagamento explícito recebem o código correto `2` (Outros).

### Outros

- Diversas melhorias internas de infraestrutura de CI/CD sem impacto funcional visível ao usuário.

## Localization — 25.5.2.186 <small>19/05/2026</small>

### Novos Recursos

- Adicionado suporte ao novo formato alfanumérico de CNPJ da Receita Federal: a validação, a formatação com máscara e o envio nos XMLs da NF-e (destinatário, transportadora, local de retirada, local de entrega e notas fiscais referenciadas) passam a aceitar os novos CNPJs que contêm letras além de dígitos.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.

## Tax Books — 25.5.0.152 <small>14/05/2026</small>

### Melhorias

- **Apuração IRPJ/CSLL — acréscimo de presunção (LC 224/2025):** redesign completo do cálculo do acréscimo de 10% sobre a parcela da receita bruta que excede R$ 1,25 milhões por trimestre (R$ 5 milhões anuais), conforme IN RFB 2.305/2025. Foram implementadas três novas capacidades na **Apuração IRPJ/CSLL**:
  - **Saldo acumulado entre trimestres (§ 4º):** quando a receita de um trimestre fica abaixo do limite trimestral, o saldo não utilizado é carregado para o trimestre seguinte, ampliando seu limite efetivo de isenção do acréscimo.
  - **Ajuste anual no 4º trimestre (§ 5º):** ao calcular o 4º trimestre, o sistema verifica se a receita anual ficou abaixo do limite de R$ 5 milhões ou se os excessos trimestrais acumulados superam o excesso anual real; quando aplicável, gera automaticamente um ajuste de dedução de estorno proporcional do acréscimo já recolhido nos trimestres anteriores.
  - **Modo Híbrido de Acréscimo:** nova opção na **Configuração Apuração IRPJ/CSLL** que, quando ativada, aplica a alíquota acrescida (ex.: 35,20%) sobre toda a receita do trimestre corrente assim que a receita acumulada do ano anterior ao trimestre já ultrapassou R$ 5 milhões, sem distinção de parcela isenta.

### Correções

- **EFD Contribuições — consolidação multifilial dos registros M220/M620 de reoneração (NT 012/2026):** no modo multifilial consolidado, os registros M220 (PIS) e M620 (COFINS) referentes ao ajuste de reoneração (código 11) passam a ser emitidos como um único registro consolidado com o valor somado de todas as filiais, evitando que o PVA rejeite o arquivo por registros duplicados com indicadores e códigos de ajuste idênticos por filial.

- **EFD Contribuições — detalhamento M225/M625 de todas as filiais sob o M220/M620 consolidado de reoneração:** em modo multifilial, os registros de detalhe M225 (PIS) e M625 (COFINS) passam a listar os lançamentos individuais de todas as filiais vinculadas ao M220/M620 consolidado, garantindo rastreabilidade completa dos valores por filial no arquivo gerado.

- Corrigido erro no cálculo do acréscimo de presunção da **Apuração IRPJ/CSLL** em regime de caixa: quando havia linhas cadastradas na tabela de percentuais de presunção por múltiplas alíquotas (utilizada exclusivamente no regime de competência), o cálculo as utilizava indevidamente, sobrescrevendo a base de receita já apurada pelo regime de caixa e produzindo valores incorretos de presunção.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.

## Tax Books — 25.5.0.96 <small>08/05/2026</small>

### Novos Recursos

- **EFD Contribuições: reoneração PIS/COFINS (LC 224/2025 / NT 012/2026).** Implementado o cálculo automático dos ajustes de reoneração de PIS e COFINS exigidos pela Lei Complementar 224/2025. Durante a apuração, o sistema identifica os documentos de saída com CST sujeito à reoneração, calcula o valor do ajuste por documento (aplicando o percentual de reoneração sobre a alíquota padrão) e gera os registros de detalhe M225 (PIS) e M625 (COFINS) no arquivo EFD-Contribuições, com a chave de acesso da NF-e preenchida conforme orientação da NT 012/2026. Os totais de ajuste de acréscimo e a contribuição do período (campos VL_AJUS_ACRES e VL_CONT_PER) nos registros M210 e M610 passam a ser preenchidos automaticamente. Uma nova tela de consulta permite visualizar os lançamentos de reoneração calculados por período. A funcionalidade é habilitada na **Configuração EFD Contribuições**, onde também é acessada a **Matriz de Reoneração PIS/COFINS** — tabela parametrizável por CST, data de vigência, percentual de reoneração e alíquotas padrão de PIS e COFINS.

- **EFD Contribuições: suporte aos blocos I, P e registro 0120.** O arquivo EFD-Contribuições passa a incluir os blocos I e P (emitidos sempre, com indicador de sem movimento, conforme obrigatoriedade do leiaute). O registro 0120 — motivo de ausência de dados — também é gerado automaticamente quando o campo **EFD Contribuições - Motivo Sem Dados** estiver preenchido na **Apuração PIS/COFINS**. O bloco M passa a emitir sempre o indicador de movimento ativo (conforme exigência do Guia Prático v1.35), e os contadores de linhas dos blocos 1 e 9 foram ajustados para refletir os novos blocos.

### Melhorias

- **NF-e e Escrituração Fiscal: tipo de pagamento lido diretamente das parcelas do XML.** O indicador de tipo de pagamento (à vista, a prazo, sem pagamento, etc.) nos registros C100 do SPED Fiscal e D100 da EFD-Contribuições passa a ser determinado a partir do campo de pagamento informado no documento fiscal, com compatibilidade retroativa para documentos que ainda usavam o tipo definido nas condições de pagamento. O valor "Sem Pagamento" (código 9 no SPED) também passou a ser reconhecido corretamente.

- **EFD Contribuições: CT-e OS (modelo 67) incluído no filtro de documentos de transporte.** O modelo de documento 67 (CT-e de Outros Serviços) foi adicionado ao conjunto de modelos classificados como aquisição de serviços de transporte, passando a ser escriturado no registro D100 da EFD-Contribuições e do SPED Fiscal.

### Correções

- **Apuração PIS/COFINS: corrigido agrupamento indevido de documentos no cálculo da reoneração.** Em períodos com múltiplos documentos do mesmo participante, o sistema estava agrupando lançamentos de notas fiscais diferentes em uma única entrada de reoneração, distorcendo a base e o valor do ajuste. O número do documento foi adicionado como critério de agrupamento, garantindo que cada NF-e gere seu próprio registro de detalhe com base e valor corretos.

## Localization — 25.5.1.96 <small>08/05/2026</small>

### Novos Recursos

- **MD-e:** Adicionada ação **Exportar XMLs** nas listas de MD-e e MD-e Arquivado, permitindo selecionar múltiplos documentos e baixar os arquivos XML em lote num único arquivo ZIP.

- **NF-e + Escrituração Fiscal:** Adicionado preenchimento automático do indicador de tipo de pagamento (à vista, a prazo, sem pagamento, etc.) no Documento Fiscal, derivado das parcelas e datas de vencimento no momento do registro — alimentando o campo **Tipo de Pagamento** utilizado pela Escrituração Fiscal (indicador IND_PGTO do SPED). Também removida a restrição que impedia a importação de parcelas do XML quando a quantidade excedia o número de parcelas definido nas condições de pagamento.

- **Aplicação de Adiantamento:** Implementado cálculo e contabilização automática de impostos retidos no momento da aplicação de adiantamentos a faturas, com o correspondente estorno na desaplicação. A nova opção **Calcular Impostos na Aplicação**, disponível na configuração fiscal, habilita o recurso. As páginas de aplicação de clientes e fornecedores passam a exibir o valor dos impostos considerados na aplicação.

### Melhorias

- **MD-e:** Passagem do Número de Inscrição Estadual (IE) do emitente do XML da NF-e para o registro do MD-e durante a importação.

- **NF-e – Reforma Tributária:** Adicionada opção **06 – Retorno por recusa parcial na entrega** ao campo **Tipo Nota de Crédito**, conforme Ajuste SINIEF 49/2025. Os valores das opções existentes foram revisados para refletir a nomenclatura atualizada.

### Correções

- **Cálculo IR:** Corrigido o cálculo da redução de imposto (Lei 15.270/2025) em jurisdição de pagamento — a base tributável agora é corretamente preenchida no fluxo de pagamento, garantindo que a progressão de tabela e a dedução simplificada do IRPF sejam aplicadas sobre o valor correto.


### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário (reorganização de declarações de variáveis, adição de comentários em construções de código, proteção contra truncamento de campos de texto e atualização de diretrizes de desenvolvimento).

## Changelog

| Hash | Data | Descrição | Aplicativo | Versão |
|---|---|---|---|---|
| 5d44ab7 | 22/05 | Variação Cambial: opção para desabilitar ajuste em adiantamentos brasileiros | Localization | 25.5.3.56 |
| dfd2c30 | 20/05 | E-Mail de Documentos: e-mail de nf-e estava utilizando mensagem adicional de Transportadora no envio para clientes | Localization | 25.5.3.56 |
| c6d6275 | 20/05 | DANFSe: Ajuste no campo de PISCOFINS Apuração Própria Deve apenas mostrar os valores de apuração própria, nunca os valores retidos | Localization | 25.5.3.56 |
| 95541f8 | 22/05 | EFD-Contribuições / EFD-ICMS/IPI: correção de código inválido no IND_PGTO do registro C100 | Tax Books | 25.5.1.55 |
| 350e8cf | 20/05 | EFD Contribuições: implementar bloco D de saída (D200/D201/D205) para CT-e/CT-e OS (#37) | Tax Books | 25.5.1.55 |
| a903ad3 | 15/05 | CNPJ Alfanumérico: suporte ao novo formato da Receita Federal (#17) | Tax Books | 25.5.1.55 |
| a903ad3 | 15/05 | CNPJ Alfanumérico: suporte ao novo formato da Receita Federal (#17) | Localization | 25.5.2.186 |
| 496d3c8 | 14/05 | Apuração IRPJ/CSLL: redesign do acréscimo de presunção (LC 224/2025) (#50) | Tax Books | 25.5.0.152 |
| c2a77f8 | 13/05 | EFD-Contribuições: M225/M625 lista detalhes de todas filiais sob M220/M620 consolidado da reoneração | Tax Books | 25.5.0.152 |
| a79fb12 | 13/05 | EFD-Contribuições: consolidação de M220/M620 da reoneração em modo multifilial (NT 012/2026) | Tax Books | 25.5.0.152 |
| 3909fcb | 06/05 | Apuração PIS/COFINS: correção de agrupamento indevido de documentos na reoneração (#38) | Tax Books | 25.5.0.96 |
| dcd0a2a | 30/04 | EFD Contribuições: incluir modelo 67 (CT-e OS) no filtro D100 (#34) | Tax Books | 25.5.0.96 |
| d1f8081 | 30/04 | NF-e + Escrituração Fiscal: parcelas do XML e classificação de pagamento (#16) | Tax Books | 25.5.0.96 |
| 6ab2f39 | 29/04 | EFD Contribuições: reoneração PIS/COFINS (LC 224/25, NT 012/2026) (#29) | Tax Books | 25.5.0.96 |
| a29d5b2 | 27/04 | EFD Contribuições: suporte aos blocos I/P vazios e registro 0120 (#20) | Tax Books | 25.5.0.96 |
| a22f979 | 08/05 | Revisão inicial Companial (#36) | Localization | 25.5.1.96 |
| 08ce08e | 06/05 | MD-e: exportação de XMLs em lote nas listas de MD-e e MD-e Arquivado (#39) | Localization | 25.5.1.96 |
| d1f8081 | 30/04 | NF-e + Escrituração Fiscal: parcelas do XML e classificação de pagamento (#16) | Localization | 25.5.1.96 |
| c0a8f67 | 29/04 | Aplicação de Adiantamento: cálculo e contabilização de impostos retidos (#32) | Localization | 25.5.1.96 |
| 70505b5 | 29/04 | Cálculo IR: correção da redução de imposto (Lei 15.270/2025) em jurisdição de pagamento (#28) | Localization | 25.5.1.96 |
| ddcac43 | 28/04 | NF-e: adicionada opção 06 ao campo Tipo Nota de Crédito (Ajuste SINIEF 49/2025) (#27) | Localization | 25.5.1.96 |
