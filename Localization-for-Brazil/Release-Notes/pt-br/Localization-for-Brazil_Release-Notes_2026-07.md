# Atualizações Julho/2026






## Localization — 25.5.6.270 <small>21/07/2026</small>

### Novos Recursos

- Adicionada importação automática dos valores de ICMS e AFRMM (Adicional de Frete para Renovação da Marinha Mercante) durante a importação via API SISCOMEX (DUIMP). Os valores são obtidos diretamente da aduana e rateados proporcionalmente entre as linhas da Declaração de Importação, com base nos valores de mercadoria, frete e seguro de cada linha. O campo **Valor AFRMM** passa a ser exibido nas linhas da Declaração de Importação e gerado como encargo de item no documento de compra criado.
- Adicionada ação **Consultar ICMS** na Declaração de Importação, permitindo consultar ou reconsultar manualmente os valores de ICMS e AFRMM junto ao sistema aduaneiro e distribuí-los pelas linhas, a qualquer momento após a importação.
- Adicionado campo **Cód. Encargo Item AFRMM** na **Conf. Importação NF-e**, para configurar o encargo de item a ser utilizado na geração dos custos de AFRMM nos documentos de compra originados de Declarações de Importação.

### Correções

- Corrigido problema na geração de documentos de compra a partir da Declaração de Importação quando as linhas eram divididas por número máximo: as linhas do lote anterior eram carregadas indevidamente nos documentos seguintes, causando duplicação de itens nos documentos criados após o primeiro.

## Tax Books — 25.5.2.245 <small>10/07/2026</small>

### Melhorias

- Adicionada opção **"CIAP Usar Valor Contábil"** na **Configuração EFD ICMS/IPI** para incluir o IPI (e demais encargos da operação) no Total das Saídas utilizado no cálculo do CIAP. Quando habilitada, as saídas tributadas e o total de saídas passam a considerar o valor contábil do documento (que já inclui o IPI), em vez da base de cálculo do ICMS. A opção permanece desativada por padrão — ative-a apenas quando a legislação estadual exigir que o IPI componha o Total das Saídas para fins de apropriação do crédito do CIAP.

## Localization — 25.5.6.245 <small>10/07/2026</small>

### Novos Recursos

- **Envio de e-mail de documentos fiscais para o cliente Faturar-a:** adicionado novo campo **Destinatário de E-Mail do Documento** nas Configurações NF-e, permitindo escolher se o e-mail da NF-e/NFS-e será enviado ao **Cliente Venda-a**, ao **Cliente Faturar-a** ou a **Ambos**. Quando o cliente Faturar-a difere do Venda-a, o sistema cria registros de envio separados para cada destinatário configurado.

- **Endereço de entrega e destinatário da operação na NFS-e Nacional (NT SE/CGNFS-e nº 008):** o Documento Fiscal agora armazena o endereço de entrega completo (logradouro, número, complemento, bairro, cidade, CEP, UF, CNPJ/CPF e I.E.) e o papel que esse endereço exerce na operação — destinatário, local do imóvel ou obra. A DANFSe passa a exibir essas informações nos grupos corretos do layout nacional: o destinatário da operação é preenchido na seção correspondente quando o endereço de entrega assume esse papel; para imóvel e obra, os dados do endereço são incluídos nas Informações Complementares.

- **Impressão da DANFSe antes da emissão municipal (DPS):** a DANFSe pode agora ser impressa como Declaração de Prestação de Serviços (DPS) para documentos ainda não emitidos pela prefeitura, substituindo o relatório de RPS legado nesse fluxo. O número de RPS é gerado automaticamente na primeira impressão.

### Melhorias

- **Informações de imóvel e obra nas Informações Complementares da DANFSe (NT SE/CGNFS-e nº 008):** o endereço do local de prestação de serviço (imóvel) e da obra de construção civil — incluindo o código da obra quando disponível — passam a ser exibidos na seção de Informações Complementares da DANFSe, conforme exigido pela nota técnica.

### Correções

- **Cancelamento por nota de crédito em dois passos marca a fatura como cancelada:** ao cancelar uma fatura de venda registrada utilizando o fluxo de nota de crédito em dois passos, o sistema agora registra corretamente o vínculo de cancelamento, fazendo com que a fatura apareça como cancelada na interface — comportamento que estava ausente nesse fluxo específico.

- **NF-e: corrigido estouro de limite ao processar muitos dados de Exportação Indireta:** aumentada a capacidade interna de armazenamento de dados de exportação, eliminando o erro que ocorria ao gerar o XML de NF-e com grande volume de registros de Dados de Exportação.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.

## Localization — 25.5.5.224 <small>06/07/2026</small>

### Novos Recursos

- **DUIMP:** Adicionada a possibilidade de gerar múltiplos documentos de compra a partir de uma única Declaração de Importação. A tela de **Declarações de Importação** passa a exibir uma aba **Documentos** listando todos os documentos gerados, com drill-down para abertura direta de cada fatura (pendente ou registrada). Nas **Configurações de Importação NF-e** foram adicionadas as opções de número máximo de linhas por documento gerado e de divisão dos documentos por CFOP, permitindo controle fino do agrupamento das linhas da DI na criação das faturas de compra.

- **NFS-e (São Paulo):** Adicionado tratamento de retorno de lote com resultados mistos — quando a prefeitura retorna simultaneamente notas aprovadas e notas com erro no mesmo lote, o sistema agora separa e processa cada RPS individualmente, atualizando as aprovadas com número, data e chave de acesso da NFS-e e vinculando as rejeitadas ao lote de erro, sem bloquear o processamento das demais.

### Melhorias

- **NFSe:** A **Chave de Acesso NFS-e** no **Documento Fiscal Registrado** passou a habilitar edição manual somente quando a nota não está autorizada pela prefeitura, evitando alteração indevida em notas já transmitidas com sucesso.

- **DUIMP:** Na importação via DUIMP, o número de adição e a unidade de medida comercial da mercadoria agora são lidos diretamente da resposta da API e gravados corretamente nas linhas da Declaração de Importação.

- **DUIMP:** Corrigido o rateio de frete e das taxas Siscomex quando o peso líquido total ou o valor total das linhas é zero, evitando divisão por zero e adotando rateio por valor como alternativa.

### Correções

- **NF-e:** O campo **Usar Proxy Web** foi removido da exibição nas **Configurações NF-e**, pois o suporte a proxy estava inativo. O campo permanece na base de dados em modo obsoleto para compatibilidade.


- **Documento Fiscal:** Corrigida a limpeza do código CFOP ao copiar documentos de compra em modo de estorno — o código era indevidamente apagado no cabeçalho em vez de na linha correspondente.

- **Transferências:** Corrigida a leitura do estabelecimento (filial) no registro de ordens de transferência, que usava sempre o local de origem em vez do local efetivamente determinado pela lógica de envio/recebimento.

- **Cotação de Compra:** Corrigida a validação de valor mínimo no campo de preço por fornecedor na linha de cotação, que estava configurada incorretamente como valor máximo zero.

- **Diário Geral:** Corrigida a filtragem de lançamentos de clientes em aberto ao determinar o documento de aplicação, que usava o campo errado para o tipo de documento.

## Changelog

| Hash | Data | Descrição | Aplicativo | Versão |
|---|---|---|---|---|
| 0744c25 | 21/07 | DUIMP: correção nas linhas que vão para os documentos criados ao serem divididos por número máximo de linhas | Localization | 25.5.6.270 |
| ec6abdf | 16/07 | DUIMP: introduzida a importação dos valores de ICMS e AFRMM na importação via API SISCOMEX (#108) | Localization | 25.5.6.270 |
| 0c3263c | 06/07 | CIAP: opção para incluir o IPI no Total das Saídas via valor contábil (#96) | Tax Books | 25.5.2.245 |
| 2ec6b19 | 09/07 | NF-e/NFS-e: envio de e-mail para Venda-a/Fatura-a/Ambos (#107) | Localization | 25.5.6.245 |
| b77e507 | 09/07 | Marcar fatura registrada como cancelada no cancelamento por nota de crédito em 2 passos (#100) | Localization | 25.5.6.245 |
| 6e7bdbb | 09/07 | NT SE/CGNFS-e nº 008: inclusas as informações dos grupos imóvel e Obra nas informações Complementares da DANFSe | Localization | 25.5.6.245 |
| 7cf96c6 | 09/07 | NT SE/CGNFS-e nº 008: incluir o destinatário da operação e o endereço de entrega na NFS-e Nacional (#104) | Localization | 25.5.6.245 |
| 9c4b836 | 07/07 | Revert "NF-e: força SelectLatestVersion ao gerar o XML para evitar cBenef incorreto por cache obsoleto do NST" | Localization | 25.5.6.245 |
| 9a46b34 | 07/07 | NF-e: correção de estouro de array quando há muitos dados de Exportação Indireta | Localization | 25.5.6.245 |
| 109386e | 07/07 | NF-e: força SelectLatestVersion ao gerar o XML para evitar cBenef incorreto por cache obsoleto do NST | Localization | 25.5.6.245 |
| 80c4660 | 07/07 | Revert "NF-e: força SelectLatestVersion ao gerar o XML para evitar cBenef incorreto por cache obsoleto do NST" | Localization | 25.5.6.245 |
| a089105 | 07/07 | NF-e: força SelectLatestVersion ao gerar o XML para evitar cBenef incorreto por cache obsoleto do NST | Localization | 25.5.6.245 |
| a19cb57 | 06/07 | NFSe: Habilitada inserção manual da chave de acesso NFSe em casos específicos | Localization | 25.5.5.224 |
| 0fe160c | 03/07 | DUIMP: Adicionada possibilidade da criação de múltiplos Documentos a partir de um único documento de Declaração de Importação (#101) | Localization | 25.5.5.224 |
| 4073902 | 03/07 | NFS-e: Possibilidade de Retornos de Lotes com erros e Sucessos misturados para São Paulo (#99) | Localization | 25.5.5.224 |
| c1a254b | 02/07 | NF-e: flag de Proxy (inativo) removido de conf. NF-e | Localization | 25.5.5.224 |
| d4611f9 | 30/06 | DUIMP: Correção da gravação dos campos de numero de adição e Unidade de medida DI na importação da DUIMP | Localization | 25.5.5.224 |
| 553c448 | 29/06 | Novos testes automatizados e correções de pequenos bugs (#95) | Localization | 25.5.5.224 |
