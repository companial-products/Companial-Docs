# Atualizações Outubro/2026







## Localization — 25.5.10.976 <small>30/10/2026</small>

### Novos Recursos

- Adicionada emissão de NFS-e Nacional para serviço prestado no exterior, com indicação do país da prestação e preenchimento dos dados de comércio exterior exigidos pelo layout nacional.
- Adicionada emissão de NFS-e em Aracaju-SE através do novo provedor WebISS, já no padrão ABRASF 2.02 com suporte a IBS/CBS.
- Adicionado envio da DPS de Rio Grande-RS no padrão nacional, agora através da nova API da Sigcorp.
- Possibilitada a emissão de NF-e sem Inscrição Estadual para contribuintes exclusivos do IBS/CBS, conforme NT 2026.007.

### Melhorias

- Ajustada a devolução de NF-e para seguir a NT 2025.002 v1.53, permitindo informar a referência do documento original no cabeçalho ou nos itens.
- A alíquota da Jurisdição Imposto agora aceita quatro casas decimais, permitindo maior precisão no cálculo.
- Atualizado o DANFE para o novo leiaute previsto pela Reforma Tributária (NT 2026.010).
- Reforçada a validação do cClassTrib na NF-e para impedir classificações incompatíveis com a finalidade e o tipo da nota de débito/crédito, conforme NT 2025.002 v1.40.

### Correções

- Corrigida a mensagem de erro "UF inválida = 0" ao emitir NFS-e de exportação de serviço, que agora indica qual cadastro (país/região do cliente) precisa ser corrigido.
- Corrigida a visualização do XML do lote de NFS-e, que falhava por exigir indevidamente o certificado e as senhas da filial.
- Corrigida a mensagem de erro ao abrir o DANFSe para orientar o usuário quando o município possui layout próprio e a opção Layout Padrão está marcada incorretamente na Configuração NF-e.
- Corrigida a NF-e Automática para não travar o documento nem gerar reenvio duplicado quando ocorre falha na confirmação logo após a autorização.
- Corrigido o cálculo do ICMS em vendas para cliente Consumidor Final: o valor do IPI agora é somado automaticamente à base de cálculo, quando aplicável.
- Corrigida a transmissão automática de NF-e para não desfazer o registro do documento sem gravar a chave de acesso e o recibo quando a postagem falha, evitando saltos de numeração e perda de rastreabilidade.
- Corrigida a gravação da chave de acesso retornada pelo serviço de NFS-e de Goiânia, que não estava sendo salva no documento fiscal.

### Outros

- Diversas melhorias internas de qualidade de código e reorganização de permissões sem impacto funcional visível ao usuário.

## Tax Books — 25.5.2.976 <small>30/10/2026</small>

### Novos Recursos

- Adicionados eventos de integração na Apuração IRPJ/CSLL e no ECF, permitindo que extensões personalizem o cálculo da apuração e completem os Blocos M e N do ECF (incluindo o registro N990), substituindo o processamento padrão quando necessário.

### Melhorias

- O Registro 1601 da EFD-ICMS/IPI agora considera apenas as notas fiscais cujo método de pagamento esteja marcado como "Incluir no Registro 1601 da EFD" — nova opção disponível no cadastro de Formas de Pagamento. Recebimentos de notas sem forma de pagamento ou com forma de pagamento não marcada deixam de gerar linha no registro. Em empresas que já utilizavam o Registro 1601, a atualização marca automaticamente todas as formas de pagamento existentes para preservar o comportamento anterior, cabendo ao usuário desmarcar as que não correspondem a instrumentos eletrônicos de pagamento. Reprocessar a apuração após alterar essa marcação atualiza corretamente as linhas geradas pelo sistema, sem duplicar nem remover linhas incluídas manualmente. Caso uma conta bancária sem instituição de pagamento vinculada receba uma nota com forma de pagamento marcada, o processamento é interrompido com mensagem indicando o documento, a forma de pagamento e a conta bancária envolvidos.

## Localization — 25.5.10.837 <small>16/10/2026</small>

### Novos Recursos

- NFS-e: adicionado registro no Business Central dos arquivos enviados à prefeitura e dos retornos recebidos, permitindo consultar o histórico de transmissão de cada nota.
- NFS-e: no ambiente nacional, a transmissão agora confere previamente os cadastros necessários e lista, numa única mensagem, tudo o que está faltando antes de tentar enviar.
- Documentos Fiscais: nova ação "Participante" para abrir diretamente o cartão do cliente, fornecedor ou filial relacionado ao documento.
- Número Item Ref.: ao preencher a referência, o sistema agora busca automaticamente o XML da nota do fornecedor já recebido via MD-e, ou solicita o arquivo ao usuário quando não encontrado.

### Melhorias

- NF-e: leiaute atualizado conforme a NT 2025.002 v1.40 (compra governamental e local da operação).
- Reforma Tributária: tabela de códigos de classificação tributária (cClassTrib) atualizada com os novos códigos da NT 2025.002 (versão de 13/10/2026).
- NF-e: adicionada validação do Nº do Item Referenciado (nItem) antes do envio à SEFAZ, evitando rejeição pelo órgão fiscal.
- E-mails de Documento: tratamento específico para documento fiscal de transferência entre filiais (participante Filial), com mensagem mais clara quando não há destinatário configurado.

### Correções

- NFS-e: o DANFSe passa a exibir sempre datas, horas e valores no formato brasileiro, independentemente da configuração regional de quem gera o PDF.
- MD-e: corrigido caso em que o emitente identificado por CPF não localizava o cliente ou fornecedor pessoa física correspondente.
- NFS-e: corrigido o DANFSe para mostrar apenas o valor líquido no campo "Valor Líquido + IBS/CBS" em notas emitidas até o fim de 2026.
- Diário de Pagamentos: corrigido processo de "Sugerir Pagamentos de Fornecedor" que consumia indevidamente o número da série, fazendo o registro recusar as linhas geradas.
- NFS-e: o envio de e-mail do documento agora aguarda a geração do Cód. Verificação NFS e trata corretamente respostas inválidas no download da imagem.
- MD-e: corrigida falha na criação do documento quando a linha da NF-e estava marcada como Ativo Fixo.
- Venda Triangular: a remessa ao cliente final deixou de gerar parcela e pagamento indevidos na NF-e.
- NF-e: corrigido o lookup de Número Item Ref. em Dados NF-e, que gravava o item mesmo quando a operação era cancelada.
- CT-e e NF-e: a importação de XML agora tolera chave de NF-e vinculada a um Documento Fiscal já excluído, evitando erro no processo.
- Documento Fiscal de Transferência: a dimensão de filial agora segue corretamente o Cód. Filial do documento, tanto na saída quanto na entrada.
- Documento Fiscal de Transferência: linhas geradas automaticamente passam a manter o Cód. Exceção Imposto definido na Matriz de Impostos e a derivar corretamente o cClassTrib a partir dela.

### Outros

- Diversas melhorias internas de qualidade de código e de convenções de documentação técnica, sem impacto funcional visível ao usuário.

## Tax Books — 25.5.2.837 <small>16/10/2026</small>

### Correções

- Corrigida a transmissão de eventos periódicos da EFD-Reinf (R-2010/R-2020, R-2050, R-2055, R-2060, R-4010 e R-4020, incluindo Pagamentos a Pessoa Jurídica): a ação "Enviar Eventos Pendentes" agora divide o envio em lotes de até 50 eventos (limite da EFD-Reinf) e grava corretamente o XML de envio, a data de envio e o número do lote em cada linha efetivamente transmitida. Antes, ao enviar mais de 50 linhas de uma vez, a maioria delas ficava presa em "Processando" com o número de um lote que não chegou a transmiti-las, sem XML de envio nem data de envio. Agora, se um lote for recusado ou falhar na comunicação, o envio para naquele ponto com uma mensagem explicando o motivo, preservando os lotes já recebidos e deixando as linhas restantes pendentes para reenvio. A ação "Consultar Lote" também foi corrigida: quando a EFD-Reinf não lista o evento de uma linha no retorno do lote consultado, a linha sai de "Processando" e vai para "Erro" com a mensagem de que o evento não consta no lote, permitindo reenvio sem duplicidade; eventos aceitos ficam em "Sucesso" com o número de recibo, e a consulta sempre termina com uma mensagem informando o resultado.
- Corrigida a geração do arquivo EFD-ICMS/IPI: o registro 0400 (Natureza da Operação/Prestação) passa a listar somente as naturezas efetivamente citadas pelos itens das notas fiscais (registro C170, modelos 01, 1B, 04 e 55). Antes, naturezas usadas apenas em documentos escriturados sem itens — como contas de energia, água, gás, frete e comunicação — também apareciam no 0400, o que causava rejeição do arquivo no validador oficial (PVA) por natureza não referenciada em nenhum outro bloco.

## Localization — 25.5.10.703 <small>02/10/2026</small>

### Novos Recursos

- NF-e: adicionado controle de prazo para cancelamento de NF-e própria. Agora é possível configurar, por filial na Config. NF-e, se o cancelamento é bloqueado após um prazo (padrão 24 horas) e se a transmissão do cancelamento à SEFAZ ocorre automaticamente junto com o cancelamento do documento. Foram criadas a lista "Cancelamentos de NF-e pendentes" e um indicador correspondente no Role Center Fiscal, além de avisos no documento fiscal e na ação "Cancelar NF-e" para notificar quando uma NF-e própria continua autorizada na SEFAZ apesar de o documento estar cancelado apenas no sistema. O alerta de pendências também passou a ser exibido antes do processamento da EFD-ICMS/IPI e da EFD-Contribuições.
- NFS-e: adicionado o significado nacional dos Códigos Tributação NFS-e para filiais que emitem pelo ambiente nacional (SEFIN). A lista "Códigos Tributação NFS-e" agora mostra, para essas filiais, o nome do código conforme o padrão nacional (Operação Tributável, Imunidade, Exportação de Serviço, Não Incidência) ou sinaliza "Não reconhecido" para valores fora do padrão, com confirmação ao digitar um valor não reconhecido e uma lista auxiliar de seleção. Também foi adicionado aviso para revisar os Códigos Tributação ao migrar uma filial para o ambiente nacional.
- Licenciamento: adicionada infraestrutura de "Funcionalidades Opcionais da Localização", permitindo a usuários SUPER ligar/desligar recursos opcionais do ambiente. A primeira funcionalidade disponível valida a licença da localização via Companial License Management, em paralelo ao mecanismo existente do License Monitor.

### Melhorias

- Certificado Digital: o aviso de vencimento exibido ao abrir as telas de trabalho (NF-e, MD-e, NFS-e, CT-e) agora é mostrado apenas para certificados efetivamente em uso por alguma configuração, evitando alertas sobre certificados antigos que ficaram no cadastro após uma renovação.
- Compras: o "Desfazer Recebimento" agora permite reverter linhas de recebimento geradas por subcontratação (Planilha de Subcontratação) e por envio direto (drop shipment), seguindo o mesmo comportamento padrão do Business Central, incluindo o estorno da saída e capacidade da ordem de produção e o desfazimento vinculado entre remessa de venda e recebimento de compra.

### Outros

- Diversas melhorias internas de qualidade de código e de infraestrutura de licenciamento/build sem impacto funcional visível ao usuário.



## Tax Books — 25.5.2.703 <small>02/10/2026</small>

### Novos Recursos

- Adicionado alerta de certificado digital vencido na EFD-Reinf: o sistema agora bloqueia o envio de eventos e a consulta de lotes quando o certificado está vencido, com mensagem clara citando o arquivo do certificado, em vez de deixar as linhas marcadas com erro genérico. A validade do certificado passou a ser exibida no cadastro (Configuração EFD-Reinf) e um aviso antecipado é mostrado quando o vencimento está próximo.
- Adicionada coluna "Motivo do Erro" (Last Error Messages) nas listas de eventos da EFD-Reinf (Pagamentos a PJ, Pagamentos a PF, INSS Retido, Produção Rural de Compras e de Vendas e CPRB), mostrando direto na lista o motivo da rejeição devolvido pela Receita, com nova ação "Visualizar Erros" para abrir o detalhe completo — sem precisar navegar até o histórico de eventos enviados.
- Adicionado alerta ao processar a EFD-ICMS/IPI ou a EFD-Contribuições: quando há documentos cancelados apenas no sistema cuja NF-e própria ainda está autorizada na SEFAZ, o usuário é avisado antes do processamento e decide se continua, evitando entregar a escrituração com uma nota tratada como cancelada enquanto ela ainda vale para o fisco. O processamento da EFD de ISS não é afetado por esse alerta.
- Adicionada seção de "lançamentos pendentes de fatura na data de corte" ao Registro de Inventário: o relatório agora identifica e lista, ao final da impressão, os documentos (recebimentos/expedições) cujo custo ainda é o custo esperado por falta de faturamento, já que o registro valoriza o estoque apenas pelo custo real. Uma mensagem é exibida ao gerar o relatório informando a quantidade de lançamentos pendentes encontrados.

### Correções

- Corrigido o cálculo do custo unitário no Registro de Inventário: itens com remessa de venda não faturada agora apresentam o custo em conformidade com o cartão do item, eliminando a divergência anteriormente observada no relatório.

### Outros

- Diversas melhorias internas de qualidade de código e de manutenção, incluindo ajustes de nomenclatura de codeunits de assinatura de eventos, sem impacto funcional visível ao usuário.

## Changelog

| Hash | Data | Descrição | Aplicativo | Versão |
|---|---|---|---|---|
| e43bfee | 30/10 | NF-e: adequar a devolução à NT 2025.002 v1.53 (referência no cabeçalho ou nos itens) L691 (#325) | Localization | 25.5.10.976 |
| 77cbdec | 30/10 | NFS-e Nacional: serviço prestado no exterior com país da prestação e dados de comércio exterior #BRL-SUP55-58 L214 (#267) | Localization | 25.5.10.976 |
| 82042e2 | 30/10 | Jurisdição de Imposto: alíquota com quatro casas decimais para reter ISS de fornecedor do Simples Nacional L690 (#324) | Localization | 25.5.10.976 |
| c83e4aa | 28/10 | Mensagem "UF inválida = 0" ao emitir NFS-e de exportação de serviço não indica qual cadastro corrigir L620 (#314) | Localization | 25.5.10.976 |
| cdb211b | 28/10 | NFS-e: Visualizar XML do lote sem o certificado e as senhas da filial L614 (#315) | Localization | 25.5.10.976 |
| 4d45b20 | 26/10 | Mensagem de erro ao abrir DANFSe não orienta o usuário quando o município tem layout próprio e a flag Layout Padrão está marcada incorretamente L573 (#309) | Localization | 25.5.10.976 |
| 7ec11d7 | 26/10 | Falha de confirmação pós-autorização na NF-e Automática trava documento e leva a reenvio com duplicidade L571 (#307) | Localization | 25.5.10.976 |
| 49b7e61 | 26/10 | IPI não é somado automaticamente à base do ICMS em venda para cliente Consumidor Final L579 (#310) | Localization | 25.5.10.976 |
| 241d2a5 | 22/10 | Transmissão automática de NF-e (NF-e Automática) desfaz registro sem gravar chave/recibo quando a postagem falha, causando saltos de numeração e perda de rastreabilidade L540 (#302) | Localization | 25.5.10.976 |
| 3cec3dc | 22/10 | NF-e: adequar o DANFE ao leiaute da Reforma Tributária (NT 2026.010) L483 (#297) | Localization | 25.5.10.976 |
| e6403a8 | 21/10 | NFS-e: emissão em Aracaju-SE pelo novo provedor WebISS (ABRASF 2.02 com IBS/CBS) L481 (#296) | Localization | 25.5.10.976 |
| 877838e | 21/10 | NFS-e: chave de acesso retornada pelo serviço não fica gravada no documento fiscal (Goiânia) L123 (#215) | Localization | 25.5.10.976 |
| d7f1fad | 20/10 | NFS-e: Rio Grande/RS passa a enviar a DPS no padrão nacional pela nova API da Sigcorp L397 (#282) | Localization | 25.5.10.976 |
| 2ad2d7f | 20/10 | NF-e: emissão sem Inscrição Estadual por contribuinte exclusivo do IBS/CBS (NT 2026.007) L449 (#292) | Localization | 25.5.10.976 |
| 2c236ce | 19/10 | NF-e: impedir cClassTrib incompatível com a finalidade e o tipo de nota de débito/crédito (NT 2025.002 v1.40) L419 (#284) | Localization | 25.5.10.976 |
| 8a79726 | 21/10 | Apuração IRPJ/CSLL e ECF: eventos para extensões calcularem a apuração e completarem os Blocos M e N L406 (#294) | Tax Books | 25.5.2.976 |
| be1bf5d | 20/10 | EFD ICMS/IPI: Registro 1601 deve considerar só notas com forma de pagamento marcada para o registro L411 (#293) | Tax Books | 25.5.2.976 |
| 418ccbc | 16/10 | NFS-e: DANFSe padrão sempre no formato brasileiro (datas, horas e valores), independente da Região de quem gera o PDF L431 (#285) | Localization | 25.5.10.837 |
| 3c40461 | 16/10 | MD-e: emitente com CPF não encontra cliente nem fornecedor pessoa física L399 (#279) | Localization | 25.5.10.837 |
| be24e2e | 15/10 | NF-e: adequar o leiaute à NT 2025.002 v1.40 (compra governamental, local da operação) L389 (#280) | Localization | 25.5.10.837 |
| ff4daee | 14/10 | NFS-e: registrar no Business Central os arquivos enviados à prefeitura e os retornos recebidos L39 (#195) | Localization | 25.5.10.837 |
| bcd6e31 | 14/10 | NFS-e: DANFSe deve mostrar só o valor líquido no campo "Valor Líquido + IBS/CBS" para notas emitidas até o fim de 2026 L398 (#277) | Localization | 25.5.10.837 |
| 2587b3e | 14/10 | Diário de Pagamentos: Sugerir Pagamentos de Fornecedor consome o nº da série e o registro recusa as linhas L401 (#278) | Localization | 25.5.10.837 |
| 0257676 | 14/10 | Reforma Tributária: atualizar a tabela cClassTrib com os códigos novos da NT 2025.002 (arquivo de 13/10/2026) L390 (#275) | Localization | 25.5.10.837 |
| 96775ac | 09/10 | Documentos Fiscais: ação "Participante" para abrir o cartão do cliente, fornecedor ou filial L372 (#269) | Localization | 25.5.10.837 |
| 45045a0 | 09/10 | NFS-e: e-mail de documento deve aguardar o Cód. Verificação NFS e tratar resposta inválida do download da imagem L360 (#268) | Localization | 25.5.10.837 |
| 752a612 | 09/10 | Número Item Ref.: buscar o XML da nota de fornecedor no MD-e ou pedir o arquivo ao usuário L361 (#266) | Localization | 25.5.10.837 |
| b703b7a | 08/10 | MD-e: criar documento falha quando a linha da NF-e está marcada como Ativo Fixo L358 (#265) | Localization | 25.5.10.837 |
| afa3113 | 08/10 | Venda Triangular: remessa ao cliente final não deve gerar parcela nem pagamento na NF-e L304 (#260) | Localization | 25.5.10.837 |
| 8965d4c | 07/10 | NF-e: validar o Nº do Item Referenciado (nItem) antes do envio à SEFAZ L326 (#262) | Localization | 25.5.10.837 |
| c2720bb | 07/10 | NF-e: lookup de Número Item Ref. em Dados NF-e grava o item mesmo ao cancelar L328 (#263) | Localization | 25.5.10.837 |
| b7c9c7d | 07/10 | CT-e e NF-e: importação de XML deve tolerar chave de NF-e ligada a Documento Fiscal excluído L282 (#256) | Localization | 25.5.10.837 |
| 11d940f | 06/10 | E-mails de Documento: tratar documento fiscal de transferência entre filiais (participante Filial) e dar mensagem clara quando não há destinatário L270 (#254) | Localization | 25.5.10.837 |
| e1ea030 | 05/10 | Documento Fiscal de Transferência: dimensão de filial deve seguir o Cód. Filial do documento (saída e entrada) L235 (#252) | Localization | 25.5.10.837 |
| 9ab1c2d | 05/10 | NFS-e: no ambiente nacional, conferir os cadastros antes de transmitir e listar tudo o que falta numa única mensagem L211 (#247) | Localization | 25.5.10.837 |
| a0b8a47 | 05/10 | Documento Fiscal de Transferência: linhas geradas automaticamente devem manter o Cód. Exceção Imposto da Matriz Impostos e derivar o cClassTrib dela L236 (#238) | Localization | 25.5.10.837 |
| 81b5c45 | 16/10 | EFD-Reinf: R-2020 fica em Processando com recibo de lote sem ter sido transmitido L403 (#281) | Tax Books | 25.5.2.837 |
| 22a49ba | 16/10 | EFD-ICMS/IPI: registro 0400 deve listar só as naturezas citadas nos itens das notas (C170) L413 (#283) | Tax Books | 25.5.2.837 |
| 26745f0 | 01/10 | NF-e: impedir que NF-e própria autorizada fique cancelada só no sistema L166 (#228) | Tax Books | 25.5.2.703 |
| 722d2dd | 28/09 | Registro de Inventário: custo unitário diverge do cartão do item quando há remessa de venda não faturada L160 (#227) | Tax Books | 25.5.2.703 |
| ba1af27 | 21/09 | Certificado digital: validade no cadastro, aviso antecipado de vencimento, bloqueio da transmissão e erro visível nas telas L106 (#209) | Tax Books | 25.5.2.703 |
| d0fd0b7 | 02/10 | Certificado Digital: aviso de vencimento nas telas de trabalho só para certificados em uso L242 (#240) | Localization | 25.5.10.703 |
| 4e21b16 | 02/10 | Licenciamento: Feature Management e verificação de licença Companial em paralelo ao License Monitor L113 (#212) | Localization | 25.5.10.703 |
| 26745f0 | 01/10 | NF-e: impedir que NF-e própria autorizada fique cancelada só no sistema L166 (#228) | Localization | 25.5.10.703 |
| 283dc53 | 01/10 | NFS-e: significado nacional e validação dos Códigos de Tributação nas filiais do ambiente nacional L213 (#235) | Localization | 25.5.10.703 |
| b784753 | 01/10 | Compras: permitir Desfazer Recebimento de linhas de subcontratação e envio direto, como no padrão L204 (#234) | Localization | 25.5.10.703 |
