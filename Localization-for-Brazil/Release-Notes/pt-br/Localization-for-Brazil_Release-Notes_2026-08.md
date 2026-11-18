# Atualizações Agosto/2026

## Tax Books — 25.5.2.383 <small>21/08/2026</small>

### Melhorias

- Na **Apuração PIS/COFINS**, as linhas de documento geradas a partir do registro F100 (operações sem documento fiscal informadas manualmente no **EFD Contribuições F100**) passam a exibir o **Cód. CST PIS** e o **Cód. CST COFINS** informados no registro, a conta contábil analítica da operação e os dados do participante (código, nome e CNPJ/CPF do cliente, fornecedor ou filial envolvido), além do CNPJ da filial da apuração. Com isso, o analista fiscal pode revisar e classificar as linhas diretamente nas **Linhas de Documento PIS/COFINS** sem precisar retornar ao registro F100. O reprocessamento do mesmo período reconstrói as linhas sem duplicá-las.

## Localization — 25.5.8.376 <small>20/08/2026</small>

### Melhorias

- Removidas as ações do grupo **Excel Reports** do Role Center Fiscal que deixaram de funcionar a partir do BC 29, pois os componentes do sistema base que as suportavam foram extintos nessa versão. As ações foram ocultadas e desabilitadas, evitando erros ao usuário.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.

## Tax Books — 25.5.2.350 <small>14/08/2026</small>

### Correções

- **EFD Contribuições — NFS-e própria cancelada sem confirmação da prefeitura deixa de ser escriturada:** quando uma NFS-e emitida pela própria empresa é cancelada antes de a prefeitura confirmá-la (sem Chave de Acesso, sem Código de Verificação e sem Nº da NFS-e), a apuração passa a descartá-la de todos os livros fiscais. O documento não recebe marcação no EFD Contribuições, EFD ICMS/IPI, livro fiscal, Sintegra nem livro de ISS, evitando que a Receita e a prefeitura recebam um documento que para elas nunca existiu. NFS-e canceladas que já tenham sido confirmadas (com Chave de Acesso ou Nº da NFS-e) continuam sendo escrituradas normalmente.

### Melhorias

- **Clientes com múltiplas Inscrições Estaduais — suporte ao endereço de entrega como destinatário fiscal:** quando o destinatário fiscal de uma nota é um endereço de entrega com Inscrição Estadual própria (estabelecimento distinto do cliente), a apuração passa a registrar o endereço de entrega como participante independente no registro 0150 do EFD ICMS/IPI e do EFD Contribuições. O código do participante (COD_PART) é derivado do número do cliente combinado com o código do endereço de entrega, e a Inscrição Estadual, o endereço e o CEP escriturados no arquivo são os do endereço de entrega — e não os cadastrais do cliente. Apurações anteriores à mudança não são afetadas: os participantes sem endereço de entrega mantêm exatamente o mesmo COD_PART. Um mecanismo de proteção impede que dois estabelecimentos distintos compartilhem o mesmo código no registro 0150; se isso ocorrer, a geração do arquivo é interrompida com mensagem de erro orientativa.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.

## Localization — 25.5.7.350 <small>14/08/2026</small>

### Novos Recursos

- **Alertas de NCM inválido nos itens:** A ficha do item agora exibe uma notificação quando o Código NCM informado saiu da nomenclatura publicada pelo Siscomex, perdeu a validade ou não está cadastrado na lista de Códigos NCM. A página de Códigos NCM passou a destacar visualmente os códigos inválidos e ganhou a ação "Items with Invalid NCM" para listar diretamente os itens que precisam de correção. A atualização automática dos códigos via Siscomex também passou a registrar as datas de início e fim de vigência e a sinalizar os códigos que não constam mais na nomenclatura vigente.

- **ISS Retido em compras com fornecedor de imposto por município:** O cálculo do ISS Retido em compras agora identifica o município do prestador de serviço (fornecedor) como referência para a retenção, e permite configurar, por município, um fornecedor específico para o qual o ISS retido é recolhido — substituindo a conta contábil genérica da configuração de registro de impostos. Os campos correspondentes foram adicionados à tabela de Municípios.

- **ISS Retido para vendas no mesmo município:** Adicionada configuração por município para que o ISS Retido seja calculado também quando o prestador e o tomador de serviço estão no mesmo município — atendendo às cidades que atribuem ao tomador local a responsabilidade pela retenção.


- **Clientes com múltiplas Inscrições Estaduais no Documento Fiscal:** O Documento Fiscal passou a resolver automaticamente a Inscrição Estadual e o endereço corretos quando o destinatário é um endereço de entrega marcado como destinatário fiscal — contemplando clientes com mais de uma I.E. cadastrada (ex.: produtor rural com um CPF e uma I.E. por propriedade). A geração do XML da NF-e foi ajustada para não duplicar o endereço do destinatário quando ele já coincide com o endereço de entrega fiscal.

### Correções

- **Posicionamento do campo "Aplicado por" nos Movimentos de Cliente e Fornecedor:** Corrigido o posicionamento dos campos "Applies-to Doc. Type" nas páginas de Movimentos de Cliente e Movimentos de Fornecedor, que estavam sendo adicionados em um grupo de conteúdo separado ao final da página em vez de permanecerem junto aos demais campos do grupo original.

- **Encargos de Item: dedução do crédito de impostos no custo aplicado a recebimentos de transferência:** O crédito de impostos recuperáveis (ex.: ICMS, PIS/COFINS) agora é corretamente deduzido do custo do encargo de item quando este é aplicado a um recebimento de transferência, alinhando o tratamento ao que já ocorria em compras diretas.

- **Importação: campos Siscomex descontinuados:** Campos e marcadores que nunca foram lidos pelo cálculo de impostos desde a remoção do cálculo legado de importação foram marcados como obsoletos, evitando que continuem aparecendo em integrações ou relatórios como se ainda tivessem efeito.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário (inclusão de quebra de linha final e remoção do BOM UTF-8 em arquivos AL, atualização de baselines do AppSourceCop).

## Tax Books — 25.5.2.295 <small>07/08/2026</small>

### Melhorias

- **EFD-Reinf — Configurações Reinf agora sempre editáveis:** removida a exigência de clicar em um botão de desbloqueio para editar a página de **Configurações Reinf**. Todos os campos (dados do contribuinte, contato, responsável, web service e certificado) passam a estar diretamente editáveis ao abrir a página, eliminando a etapa extra de habilitação.

### Correções

- **EFD Contribuições — Registro 0035 com CNPJ alfanumérico:** corrigida a geração do campo COD_SCP no registro 0035 do arquivo EFD Contribuições para suportar corretamente CNPJs alfanuméricos de Sociedades em Conta de Participação (SCP). Anteriormente, apenas dígitos numéricos eram extraídos, o que resultava em um código inválido quando o CNPJ continha letras.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.


## Localization — 25.5.6.295 <small>07/08/2026</small>

### Novos Recursos

- **NF-e — Devoluções:** Adicionado suporte ao referenciamento da NF-e de origem **por item** nas devoluções de venda e de compra (grupo `gDFeReferenciado`), conforme exigência da SEFAZ (NT 2025.002 v1.51, regra VC02-14). A referência pode ser feita por item antes do prazo obrigatório (05/10/2026 em produção) e é aceita como alternativa à referência de cabeçalho a partir de 01/07/2026 em homologação. O sistema passa a validar no registro: bloqueia referenciamento simultâneo em dois níveis (cabeçalho + item), exige chave de acesso e número do item em cada linha após o prazo, e impede referência eletrônica no cabeçalho após a virada. O texto automático de devolução nas compras passa a ser gerado a partir das referências já preenchidas pelo usuário (cabeçalho ou linhas), sem criar referências automaticamente.

- **Termos de Pagamento:** Adicionado totalizador de percentuais das parcelas na tela de parcelas dos termos de pagamento. O total é exibido em tempo real com destaque visual (verde = 100%, vermelho = fora) e o sistema bloqueia o fechamento da tela quando os percentuais não somam exatamente 100%.

### Melhorias

- **NF-e — Visualizador de XML NF-e:** Diversas melhorias na exibição do XML:
  - Grupos ausentes do `infNFe` que não estavam cobertos pelas abas fixas (como autorizados a obter o XML, solicitação NFF, intermediador, entre outros) agora são exibidos automaticamente, sem que nenhum campo seja silenciosamente omitido.
  - Exibição correta dos grupos do item: IPI Devolução (`impostoDevol`) e DF-e Referenciado (`DFeReferenciado`) aparecem na aba de detalhes do item, com rótulos e campos devidamente identificados.
  - Campos dos grupos da Reforma Tributária `vItem` (Valor Total do Item) e `vNFTot` (Total da NF-e com IBS/CBS/IS) passam a ser exibidos.
  - Corrigido desalinhamento visual de campos que apareciam fora do subgrupo correto após a renderização de subgrupos aninhados.
  - Novas abas de grupos opcionais da NF-e: Exportação, Compra, Cana-de-Açúcar, Emissão Avulsa pelo Fisco, Autorizados a Obter o XML e Solicitação NFF.

- **NF-e — Evento 211120 (Destinação para Consumo Pessoal):** O evento foi eliminado pela SEFAZ (NT 2025.002) e marcado como obsoleto. Tentativas de transmiti-lo resultam em erro, preservando eventos já existentes para consulta.

- **Documentos Fiscais — Verificar Contabilização:** Os textos adicionais do Documento Fiscal agora são capturados corretamente na pré-visualização de registro, refletindo o conteúdo que será gerado no registro definitivo.

- **DUIMP:** A importação de declarações com mais de 100 itens agora funciona corretamente: a busca de itens é paginada automaticamente (100 itens por chamada) até que todos sejam carregados.

### Correções

- **DUIMP:** Corrigida a busca do valor da taxa Siscomex em declarações onde o valor não estava informado no nó `tributosCalculados`: o sistema passa a buscar o valor alternativo no nó de pagamentos (`pagamentos`), evitando que a taxa fosse ignorada em versões de correção da declaração.

- **Compras — Copiar Documento:** Corrigido o preenchimento do **Código CFOP** nas linhas geradas por **Copiar Documento** a partir de uma nota fiscal de entrada registrada. O sentido da operação era avaliado de forma invertida: nas cópias para **Devolução de Compra** e **Nota Crédito Compra** o CFOP da nota de origem era mantido, em vez de ser limpo para receber o CFOP da devolução; e nas cópias para pedidos e faturas de compra o CFOP de origem era apagado sem necessidade. Documentos de cancelamento continuam preservando o CFOP de origem, e as linhas de venda já se comportavam corretamente.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário, incluindo reorganização de convenções de codificação e atualização de conjuntos de permissões.

## Changelog

| Hash | Data | Descrição | Aplicativo | Versão |
|---|---|---|---|---|
| 78f3712 | 17/08 | Apuração PIS/COFINS: copiar os CSTs do registro F100 para as linhas da apuração | Tax Books | 25.5.2.383 |
| 15835e4 | 17/08 | Role Center Fiscal: obsolescência das ações de Excel Reports removidas no BC 29 | Localization | 25.5.8.376 |
| f16215a | 14/08 | EFD-Contribuições: NFS-e própria cancelada sem confirmação da prefeitura deixa de ser escriturada T159 (#145) | Tax Books | 25.5.2.350 |
| a90c685 | 12/08 | Fiscal: Possibilidade de Clientes com múltiplas Inscrições Estaduais (#129) | Tax Books | 25.5.2.350 |
| 66a927d | 14/08 | Avisar quando o NCM do item saiu da nomenclatura ou perdeu a validade T162 (#148) | Localization | 25.5.7.350 |
| 6961623 | 14/08 | Encargos de Item: dedução do crédito de impostos no custo do encargo aplicado a recebimento de transferência (#143) | Localization | 25.5.7.350 |
| 43efbcd | 14/08 | Importação: obsolescência dos campos Siscomex inertes no cálculo de impostos T161 (#144) | Localization | 25.5.7.350 |
| ce5e930 | 13/08 | ISS Retido: cálculo no mesmo município para venda (#141) | Localization | 25.5.7.350 |
| be49d3c | 12/08 | ISS Retido: cálculo em compras e fornecedor de imposto por município (#140) | Localization | 25.5.7.350 |
| a90c685 | 12/08 | Fiscal: Possibilidade de Clientes com múltiplas Inscrições Estaduais (#129) | Localization | 25.5.7.350 |
| 50e1008 | 12/08 | Correção de posicionamento de campos "Aplicado por" em páginas de Movimentos Cliente/Fornecedor | Localization | 25.5.7.350 |
| 5b7c6b9 | 10/08 | Eventos de extensao para pontos sem gancho, e remocao da validacao de telefone | Localization | 25.5.7.350 |
| b86f099 | 03/08 | EFD-Contribuições: correção do COD_SCP no registro 0035 para suportar CNPJ alfanumérico | Tax Books | 25.5.2.295 |
| 31f8f0d | 10/07 | EFD-Reinf: proteções de gravação removidas da Conf. Reinf | Tax Books | 25.5.2.295 |
| eaf5e7a | 05/08 | NF-e: adequação das devoluções à NT 2025.002 v1.51 e obsolescência do evento 211120 (#128) | Localization | 25.5.6.295 |
| d632a14 | 04/08 | Termos de Pagamento: total dos percentuais das parcelas e bloqueio quando não fecha 100% (#126) | Localization | 25.5.6.295 |
| 60de544 | 04/08 | Referenciar NF-e de origem por item nas devoluções de venda e compra T143 (#124) | Localization | 25.5.6.295 |
| c58c2cd | 04/08 | Documentos Fiscais: textos adicionais do documento fiscal capturados na Pré-visualização de Registro | Localization | 25.5.6.295 |
| 699ff8f | 04/08 | NF-e: visualizador de XML exibe grupos ausentes do infNFe e corrige campos desalinhados após subgrupos | Localization | 25.5.6.295 |
| 821c66a | 04/08 | NF-e: visualizador de XML exibe os grupos do item (IPI devolvido e DF-e referenciado) no detalhe (#123) | Localization | 25.5.6.295 |
| 8effbe7 | 03/08 | NF-e: visualizador de XML exibe as tags vItem e vNFTot da Reforma Tributária | Localization | 25.5.6.295 |
| 81d0687 | 03/08 | DUIMP: Correção para Declarações com mais de 100 itens nas adições #BRL-SUP35-187 | Localization | 25.5.6.295 |
| bfe386b | 30/07 | DUIMP: Correção na busca do valor da taxa siscomex em casos de versões de correção #BRL-SUP35-186 | Localization | 25.5.6.295 |
