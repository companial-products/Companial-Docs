# Atualizações Setembro/2026








## Localization — 25.5.9.649 <small>29/09/2026</small>

### Novos Recursos

- **NFS-e:** Adicionado campo "Motivo da Não Informação do NIF" na ficha do cliente (aba Fiscal), permitindo registrar por que um cliente do exterior não tem NIF (dispensado ou não exigível). No ambiente nacional (SEFIN), a transmissão da NFS-e passa a ser bloqueada quando o cliente do exterior não tem nem o "Cód. NIF" nem esse motivo preenchidos, apontando diretamente para a ficha do cliente a corrigir. O motivo é enviado à nota juntamente com o NIF quando aplicável.
- **Reforma Tributária / DUIMP:** A Classificação Tributária (cClassTrib) declarada na DUIMP passa a ser trazida automaticamente para a linha da Declaração de Importação. Quando o código declarado já está cadastrado, a classificação é adotada automaticamente (com o respectivo código regular de fallback); quando não está, a linha é sinalizada visualmente para conferência do analista. Foi adicionada a ação "Adotar Classificações Declaradas" na ficha da DI, para reprocessar as adições ainda não classificadas depois de cadastrar um código que faltava. A criação dos documentos de compra agora é bloqueada, com mensagem detalhada por adição, quando a classificação adotada é incompatível com o NCM da linha.
- **Filiais:** O processo de preenchimento retroativo de filial em lançamentos antigos ("Preencher Filial em Movimentos Antigos") passou a incluir também os Movimentos de Item e de Valor do estoque, além das áreas já cobertas anteriormente (contábil, cliente, fornecedor, empregado, conta bancária e ativo fixo).

## Localization — 25.5.9.614 <small>24/09/2026</small>

### Novos Recursos

- Adicionado controle de validade do certificado digital: o cadastro de Certificados Digitais agora exibe a data de vencimento (lida do próprio arquivo do certificado), com destaque visual quando o certificado está vencido ou próximo do vencimento. As telas de transmissão de NF-e, NFS-e, MD-e e CT-e passam a bloquear o envio quando o certificado usado está vencido e exibem um aviso antecipado (uma vez por sessão) quando o vencimento está próximo. O prazo de antecedência do aviso é configurável na Configuração Fiscal, no novo campo "Aviso de Vencimento do Certificado (Dias)" (padrão de 30 dias; 0 desliga o aviso).
- Adicionada opção na Configuração NF-e ("Incidência do ISSQN no Município do Prestador") para que o DANFSe imprima o município da filial como local de incidência do ISSQN, em vez do município onde o serviço foi prestado — útil quando a prefeitura declara a incidência na cidade da filial. A opção afeta apenas a impressão do documento; o que é transmitido à prefeitura não muda.
- Adicionados dois eventos de extensão no rateio de Frete/Siscomex da Declaração de Importação: um antes do recálculo (permitindo que uma extensão assuma o rateio por completo) e outro depois (permitindo ajustes finos, como redistribuir apenas a taxa Siscomex, após o cálculo padrão).
- Adicionado evento de extensão ao final da importação da DUIMP, disparado após todas as linhas e o rateio padrão (Siscomex, ICMS, AFRMM) serem gravados e antes da confirmação da transação — permite que uma extensão sobrescreva valores do cabeçalho ou das linhas da DI antes de o usuário abrir o documento.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.

## Tax Books — 25.5.2.557 <small>21/09/2026</small>

### Novos Recursos

- **EFD-Reinf: cálculo automático do adicional de INSS de aposentadoria especial por Jurisdição Imposto.** A sugestão mensal de eventos agora desmembra, por documento, o adicional de INSS referente à aposentadoria especial (15, 20 e 25 anos) a partir das alíquotas configuradas na nova página **Reinf - Configuração de INSS Adicional**, acessível nos menus de configuração da EFD-Reinf. Os valores do adicional são gravados separadamente da retenção principal nos eventos R-2010 (compras) e R-2020 (vendas), sem necessidade de preenchimento manual. Jurisdições sem configuração não sofrem qualquer alteração. Caso o adicional calculado ultrapasse o valor retido do documento, a sugestão é interrompida com mensagem de erro identificando o documento fiscal e a jurisdição a corrigir.

- **EFD-Reinf: consolidação de clientes com mesmo CNPJ no evento R-2020.** Ao sugerir os eventos mensais de serviços prestados, clientes pessoa jurídica que compartilham o mesmo CNPJ e possuem retenção no mesmo período e estabelecimento passam a gerar um único evento R-2020 — evitando rejeição pela Receita Federal, que identifica o evento pela combinação período + estabelecimento prestador + CNPJ do tomador. Os documentos de todos os cadastros são agrupados sob o cliente de menor número do grupo. A página **Reinf - Retenção Contribuição Previdenciária - Vendas** passa a exibir o campo **CNPJ Participante** para facilitar a identificação do grupo. Na página **Reinf - Retenção Contribuição Previdenciária - Documentos**, o novo campo **Nº Participante Origem** preserva o cadastro que originou cada nota fiscal, mesmo após a consolidação.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.

## Localization — 25.5.8.557 <small>18/09/2026</small>

### Novos Recursos

- **Cadastro de Clientes:** Incluída opção **Permitir CNPJ Duplicado** no cadastro de clientes, viabilizando o registro de mais de um cliente com o mesmo CNPJ quando necessário (por exemplo, filiais de um mesmo grupo econômico).

- **NF-e / Importação Direta:** Adicionado o campo **Valor Desconto DI** (tag `vDescDI` da NF-e) nas telas de adição da Declaração de Importação. O valor é preenchido manualmente por adição e incluído automaticamente no XML da NF-e quando informado, sem alterar impostos ou totais do documento.

### Correções

- **Documento Fiscal:** Corrigida perda do texto estendido da linha do Documento Fiscal que ocorria quando o registro de estoque era processado de forma concorrente durante o lançamento, garantindo que o texto seja sempre preservado.

- **NF-e / Importação Direta:** Corrigido o processo de cópia do Documento Fiscal para operações com Declaração de Importação: os dados de adição de DI e de exportação eram perdidos em determinadas situações por uso de chave de filtro incorreta, resultando em linhas sem informação fiscal na cópia. Ambos os filtros foram corrigidos.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.

## Tax Books — 25.5.2.466 <small>04/09/2026</small>

### Correções

- **EFD ICMS/IPI:** Documentos com NF-e denegada (COD_SIT 04) e numerações inutilizadas (COD_SIT 05) deixam de ser escriturados no arquivo gerado para períodos a partir de janeiro de 2023, conforme os Ajustes SINIEF 34/2021 e 38/2021, que descontinuaram esses códigos da tabela de situação tributária. Os registros filhos (itens, registros analíticos e observações) também são omitidos, e os contadores de registros do arquivo acompanham apenas as linhas efetivamente gravadas. Períodos anteriores a 2023 continuam sendo gerados exatamente como antes. A apuração interna e os livros impressos não são afetados.

- **Regime de Caixa — PIS/COFINS:** Notas fiscais de exportação e demais operações sem incidência de PIS/COFINS (CST 08 e similares) recebidas no período passam a gerar linha na Apuração PIS/COFINS e, com ela, os registros F500, F525 e 1900 do EFD-Contribuições — comportamento que estava ausente, fazendo essas receitas desaparecerem da escrituração no regime de caixa. A base e o valor das contribuições continuam zerados; o que é registrado é a receita efetivamente recebida. Notas tributadas não são afetadas. Adicionalmente, corrigido o rateio do valor recebido para que incida apenas sobre as linhas de receita do documento: em notas mistas (com linha de remessa junto à venda), a parcela da remessa não inflava mais a receita escriturada nem fazia a receita da venda ser subnotificada.

### Outros

- Documentos internos de desenho conceitual e especificação técnica do módulo de Apuração IBS/CBS (Reforma Tributária) adicionados ao repositório — sem impacto funcional disponível ao usuário nesta versão.

## Localization — 25.5.8.466 <small>04/09/2026</small>

### Novos Recursos

- **NFS-e: suporte a obras, imóveis e eventos (LC 214/2025):** Adicionados os cadastros de **Atividades/Eventos** e **Imóveis** (acessíveis pela configuração manual), com endereço completo e numeração automática por série. No Documento Fiscal e no Documento Fiscal Registrado, novos campos permitem vincular o documento a uma obra, atividade/evento ou imóvel; ao selecionar um desses vínculos, o endereço do local de prestação é preenchido automaticamente. O DANFSe passou a exibir o local de prestação, o identificador e o período do evento nas Informações Complementares, diferenciando imóvel, obra e evento conforme o grupo preenchido no RPS. O cadastro de Construções (obras) também recebeu campo de endereço completo, código ART e suporte a série de numeração. A atualização de instalação classifica automaticamente os códigos de operação de serviço já cadastrados com o local de prestação correspondente (imóvel, adquirente, destinatário, evento ou outro), sem sobrescrever classificações manuais já existentes.

- **NFS-e: prévia do DANFSe antes do lançamento:** Adicionadas as ações **Gerar Rascunho do DANFSe** e **Visualizar DANFSe (Rascunho)** nos pedidos e faturas de venda com Tipo de Documento Fiscal de NFS-e. A ação simula o lançamento internamente, gera o PDF do DANFSe e o descarta sem criar documento fiscal, consumir número ou enviar qualquer dado à prefeitura. O rascunho gerado fica disponível para download imediato pela notificação ou, posteriormente, pela ação de visualização. O documento é marcado como "sem validade jurídica" em todos os ambientes.

- **API para agentes de IA — Fase 1:** Disponibilizadas APIs REST (grupo `brazil`, publisher `arquibr`, versão `v1.0`) que expõem os campos fiscais brasileiros de clientes, fornecedores e itens, complementando as APIs padrão da Microsoft. Também foram publicadas APIs somente leitura de referência para os principais cadastros fiscais: NCM, exceção NCM, CEST, CST, CFOP, origem ICMS, códigos ANP, exceções fiscais, áreas de tributação, territórios, categorias de participante e códigos de serviço NFS-e.

- **NF-e: destino da operação em venda presencial e NIF de cliente estrangeiro:** Na venda presencial com presença do comprador no estabelecimento (`indPres = 1`), o destino da operação passa a ser determinado automaticamente como "mesmo estado", evitando que o sistema classifique erroneamente operações internas como interestaduais ou internacionais. Para vendas presenciais fora do estabelecimento do emitente (`indPres = 5`), o campo **Situação do Cliente/Fornecedor** fica editável para ajuste manual entre "mesmo estado" e "outro estado". Adicionalmente, o XML da NF-e passou a incluir o NIF (Número de Identificação Fiscal) do cliente estrangeiro no nó do destinatário, exceto em importações diretas.

### Melhorias



- **Relatórios Contas a Receber e Contas a Pagar — dimensões e N.º Projeto no dataset:** Os relatórios passaram a incluir no dataset os oito atalhos de dimensão (código, nome e caption) configurados na empresa, permitindo que layouts personalizados exibam dimensões e segmentos por lançamento.

- **Reforma Tributária — validação do Cód. Class. Trib. Regular IBS/CBS:** Adicionada validação ao preencher o campo **Cód. Class. Trib. Regular IBS/CBS** na linha de documento fiscal, na linha de venda, na linha de compra, na exceção fiscal e na matriz de impostos: o sistema bloqueia a gravação se o código regular for igual ao código principal, ou se o código principal não exigir tributação regular. A mesma validação é aplicada ao propagar os códigos a partir da matriz de impostos ou de exceções fiscais.

### Correções

- **NF-e: XML da ação "Criar DANFe" enviava nó de cancelamento:** Corrigido o template XML utilizado pela operação de criação do DANFe junto ao serviço, que estava enviando erroneamente a estrutura do nó de cancelamento em vez da estrutura correta `<criardanfe>`.

## Changelog

| Hash | Data | Descrição | Aplicativo | Versão |
|---|---|---|---|---|
| 900af58 | 29/09 | NFS-e: exigir o NIF ou o motivo da falta dele para cliente do exterior no ambiente nacional L181 (#230) | Localization | 25.5.9.649 |
| b19619a | 29/09 | Reforma Tributária: rotinas de download das classificações tributárias e dos vínculos de NCM tornadas públicas para apps de carga de dados | Localization | 25.5.9.649 |
| 9045264 | 29/09 | Filiais: incluir movimentos de item e de valor no preenchimento da filial nos lançamentos antigos L185 (#229) | Localization | 25.5.9.649 |
| 3ad8a68 | 28/09 | DUIMP: trazer a Classificação Tributária (cClassTrib) para a linha da Declaração de Importação L83 (#203) | Localization | 25.5.9.649 |
| 9256845 | 23/09 | DUIMP: eventos para customização do rateio Siscomex na importação e no recálculo da DI L119 (#214) | Localization | 25.5.9.614 |
| 500e06b | 22/09 | NFS-e: DANFSe deve imprimir o município de incidência do ISSQN, e não o de prestação L77 (#210) | Localization | 25.5.9.614 |
| ba1af27 | 21/09 | Certificado digital: validade no cadastro, aviso antecipado de vencimento, bloqueio da transmissão e erro visível nas telas L106 (#209) | Localization | 25.5.9.614 |
| 4687b15 | 18/09 | Qualidade de Código: correção geral de avisos CodeCop nos apps AppSource L99 (#206) | Tax Books | 25.5.2.557 |
| e4f1efa | 16/09 | Cadastro de Clientes: permitir mais de um cadastro com o mesmo CNPJ L74 (#201) | Tax Books | 25.5.2.557 |
| 4e65c3d | 10/09 | EFD-Reinf: calcular o adicional de INSS de aposentadoria especial por Jurisdição Imposto L40 (#191) | Tax Books | 25.5.2.557 |
| 4687b15 | 18/09 | Qualidade de Código: correção geral de avisos CodeCop nos apps AppSource L99 (#206) | Localization | 25.5.8.557 |
| 10343b3 | 17/09 | NF-e: desconto da adição da DI (vDescDI) na tela de importação direta e no XML L89 (#205) | Localization | 25.5.8.557 |
| e4f1efa | 16/09 | Cadastro de Clientes: permitir mais de um cadastro com o mesmo CNPJ L74 (#201) | Localization | 25.5.8.557 |
| b9cd923 | 15/09 | Filiais: preencher a filial nos lançamentos antigos ao configurar a dimensão de filial L48 (#198) | Localization | 25.5.8.557 |
| 63a0f78 | 14/09 | Documento Fiscal: corrigir a perda do texto estendido com o registro concorrente de estoque ativo L60 (#197) | Localization | 25.5.8.557 |
| 27bbb7d | 14/09 | Filial: dimensão deve prevalecer sobre o Cód. Filial em branco ao lançar L47 (#194) | Localization | 25.5.8.557 |
| cea8944 | 10/09 | Processo de entrega: desfazer o merge prematuro do DUIMP (#190) e voltar a PR aberto L46 (#193) | Localization | 25.5.8.557 |
| d5b0bf8 | 10/09 | DUIMP: trazer a Classificação Tributária (cClassTrib) para a linha da Declaração de Importação L23 (#190) | Localization | 25.5.8.557 |
| 06a1d8b | 03/09 | EFD ICMS/IPI: parar de escriturar documentos denegados e com numeração inutilizada (04 e 05) L16 (#181) | Tax Books | 25.5.2.466 |
| 6fadc36 | 01/09 | Reforma Tributária: spec da apuração IBS/CBS — vínculo da conciliação por chaves explícitas em vez de RecordID | Tax Books | 25.5.2.466 |
| ccec2f9 | 01/09 | Reforma Tributária: especificação técnica da apuração IBS/CBS (módulo IBSCBSSettlement no taxbooks) | Tax Books | 25.5.2.466 |
| 4b5a05b | 01/09 | Reforma Tributária: desenho conceitual da apuração IBS/CBS (apuração própria + conciliação com a assistida) | Tax Books | 25.5.2.466 |
| f0f9f90 | 25/08 | Regime de Caixa exclui receitas sem incidência (CST 08 exportação) do F500/F525/1900 L4 | Tax Books | 25.5.2.466 |
| 1f0d777 | 03/09 | NFSe: Adicionadas lógicas de eventos obras, imóveis e destinatários (#127) | Localization | 25.5.8.466 |
| 8d7bca4 | 03/09 | NF-e: destino da operação na venda presencial e envio do NIF do cliente estrangeiro L12 (#179) | Localization | 25.5.8.466 |
| 4b1ba44 | 31/08 | NFS-e: prévia do DANFSe antes do lançamento do pedido/fatura de venda L7 (#169) | Localization | 25.5.8.466 |
| feee685 | 28/08 | API para agentes de IA: expor campos brasileiros de cliente, fornecedor e item (Fase 1) L5 (#168) | Localization | 25.5.8.466 |
| 39d4dbd | 24/08 | Relatório Contas a Receber não expõe Nº Projeto nem Dimensões/Segmentos no dataset T#3 | Localization | 25.5.8.466 |
| 87b8ae0 | 21/08 | NF-e: correção do XML de Criar DANFe, que enviava o nó de cancelamento (#158) | Localization | 25.5.8.466 |
| 6bb71a2 | 20/08 | Reforma Tributária: validação do Cód. Class. Trib. Regular IBS/CBS | Localization | 25.5.8.466 |
