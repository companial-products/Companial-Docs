<!-- Branch: feature/danfse-v2 -->
<!-- Atualização Jul/2026 — Commits: 7cf96c6, 6e7bdbb (NT SE/CGNFS-e nº 008) -->
<!-- Atualização Ago/2026 — PR: #169 (rascunho do DANFSe antes do lançamento) -->
<!-- Atualização Out/2026 — PR: #285 (DANFSe sempre no formato brasileiro) -->
<!-- Atualização Out/2026 — PR: #309 (mensagem do Layout Padrão sem chave de acesso) -->
<!-- Atualização Nov/2026 — PR: #322 (impostos Não Destacar fora do DANFSe e do RPS; classificação sem campo na NFS-e) -->
<!-- Atualização Nov/2026 — PR: #337 (DANFSe montado a partir do XML da NFS-e autorizada; marcação de construção civil na transmissão) -->
# NFS-e: Layout Padrão DANFSe

**Data**: Fevereiro/2026 — **Atualizado**: Agosto/2026 (Rascunho do DANFSe antes do lançamento)

## Sumário

Esta atualização introduz o **layout padrão DANFSe** para impressão de Notas Fiscais de Serviço Eletrônicas (NFS-e), seguindo o modelo nacional publicado pelo governo brasileiro. Além disso, inclui melhorias na tabela IBPT com detalhamento de tributos por esfera (federal, estadual e municipal) e suporte à suspensão de exigibilidade do ISS.

Em **Junho/2026** o layout foi atualizado para a versão **DANFSe v2.0**, conforme a Nota Técnica SE/CGNFS-e nº 008 (05/05/2026): nova seção de **Tributação IBS / CBS** (Reforma Tributária), novos totais, campos **Situação da NFS-e** e **Finalidade** no bloco de identificação, e reorganização dos totais aproximados de tributos.

Em **Agosto/2026**, a conferência do documento de serviço passou a existir também **antes do lançamento**: a Ordem de Venda e a Fatura de Venda ganharam a geração de um **rascunho do DANFSe** a partir do documento ainda não lançado, sempre marcado como **NFS-e SEM VALIDADE JURÍDICA**, sem criar documento fiscal, consumir numeração fiscal ou enviar nada à prefeitura (veja seção 3). O DPS impresso após o lançamento permanece inalterado.

Em **Julho/2026**, ainda no âmbito da NT SE/CGNFS-e nº 008, foi incluído o tratamento do **destinatário da operação e do endereço de entrega** na NFS-e Nacional: o endereço de entrega passa a ser congelado no documento fiscal e roteado conforme o seu papel (destinatário da operação, local da prestação/imóvel ou obra), com um novo quadro **Destinatário da Operação** no DANFSe e os grupos **Imóvel** e **Obra** nas **Informações Complementares**. O DANFSe também passa a permitir a impressão de uma **prévia (DPS)** antes da emissão na prefeitura.

Em **Outubro/2026**, o DANFSe passou a sair **sempre no padrão brasileiro**, qualquer que seja a **Região** de quem gera o PDF: datas em dd/mm/aaaa, hora em 24 horas, valores como R$ 1.234,56 e percentuais como 5,00%. A data e hora de emissão da NFS-e passa a ser impressa sempre no horário de Brasília. Notas cujo PDF foi guardado no formato americano são corrigidas com **Atualizar DANFSe** (veja seções 1 e 2).

Em **Novembro/2026**, o DANFSe, a DPS, o rascunho e o RPS passaram a mostrar **os mesmos impostos enviados na NFS-e**: imposto marcado como **Não Destacar** fica de fora da impressão e dos totais, como já acontecia na transmissão. Um imposto destacado com **Classificação** em branco ou **Outro**, que não tem campo na NFS-e, passa a bloquear a transmissão e as impressões com uma mensagem que aponta a **Jurisdição Imposto** a corrigir, no lugar do erro "Índice fora dos limites" (veja seção 1).

Também em **Novembro/2026**, o DANFSe da NFS-e já emitida passou a ser montado com os dados do **XML da nota autorizada**, trazido do Ambiente de Dados Nacional (ADN) e guardado no documento fiscal. O PDF mostra exatamente o que consta na nota consultada no portal nacional, mesmo que o título, os impostos ou o cadastro do cliente mudem depois da emissão. Quando o XML não está disponível, o sistema avisa e pergunta se o DANFSe deve sair com os valores calculados pelo Business Central. O e-mail do documento não pergunta: fica pendente até o XML chegar (veja seções 1 e 2).

## 1. Layout Padrão DANFSe

### O que é

O DANFSe (Documento Auxiliar da NFS-e) é um relatório em PDF gerado localmente pelo sistema, com layout padronizado similar ao modelo nacional. Ele substitui a necessidade de baixar a imagem da NFS-e diretamente do web service da prefeitura, oferecendo um formato consistente independente do município.

### Origem dos Dados

O DANFSe de uma NFS-e **já emitida** é montado com os dados do **XML da NFS-e autorizada**, guardado no documento fiscal. Assim, o PDF mostra o que consta na nota que o município autorizou, a mesma consultada no portal nacional pela chave de acesso, e não os dados atuais do Business Central.

Vêm do XML: emitente, tomador, destinatário da operação, serviço, local da prestação, município de incidência do ISSQN, tributação do ISSQN, valores, deduções, descontos, retenções, IBS/CBS e informações complementares, inclusive obra, atividade/evento, imóvel e os totais aproximados dos tributos (Lei nº 12.741/2012).

Por isso, o DANFSe continua igual à nota emitida mesmo depois de:

- o título da nota ser baixado, total ou parcialmente: as retenções de PIS, COFINS e CSLL são as da nota, e não são apuradas de novo pelo título;
- um imposto ser marcado como **Não Destacar**: o DANFSe mostra o que foi transmitido;
- o cadastro do cliente mudar (endereço, e-mail, inscrição);
- a opção **Incidência do ISSQN no Município do Prestador** da **Config. NF-e** ser ligada ou desligada: o município de incidência é o da nota (veja [NFS-e: Município de Incidência do ISSQN no DANFSe](Localization-for-Brazil_User-Manual_NFSe-DANFSe-Municipio-Incidencia-ISSQN.md)).

Não vêm do XML e continuam como antes:

- o logotipo da prefeitura (veja seção 4);
- o QR Code de consulta pela chave de acesso;
- a marca d'água "CANCELADA" na NFS-e cancelada;
- a marcação "NFS-e SEM VALIDADE JURÍDICA" na NFS-e de homologação;
- o formato brasileiro de datas, horas e valores;
- a regra do **Valor Líquido da NFS-e + IBS/CBS**, que nas notas emitidas até 31/12/2026 mostra só o valor líquido. A data considerada é a da emissão da NFS-e que consta no XML.

Em comparação com o DANFSe montado pelos dados do Business Central, algumas informações passam a sair como foram transmitidas:

| Informação | DANFSe pelo XML |
| --- | --- |
| **Simples Nacional na Data de Competência**, regime de apuração e regime especial de tributação | Conforme a nota. Pelos dados do Business Central saem sempre "Não optante", "-" e "Nenhum" |
| Tipo de imunidade e benefício municipal | Conforme a nota. Pelos dados do Business Central saem sempre "-" |
| **Data e Hora da emissão da DPS** | Com a hora que consta na DPS. Pelos dados do Business Central sai sempre 00:00:00 |
| Telefone e e-mail do emitente | Só quando constam na nota. Sem eles, saem "-" |
| Inscrição municipal e endereço do tomador | Como foram transmitidos |
| País da prestação, na exportação de serviço | Com o código do país que consta na nota, no lugar do nome |

Sem o XML, o DANFSe é montado com os dados do Business Central, como antes desta versão. Isso acontece:

- na prévia (DPS) e no rascunho antes do lançamento, que ainda não têm NFS-e emitida (veja seções 2 e 3);
- quando o XML não está disponível e o usuário aceita imprimir com os valores calculados pelo Business Central (veja [Quando o XML Não Está Disponível](#quando-o-xml-não-está-disponível));
- no PDF guardado na nota antes desta versão, até ser gerado de novo com **Atualizar DANFSe**.

### Seções do Documento

O layout impresso contém as seguintes seções:

| Seção | Conteúdo |
| --- | --- |
| **Cabeçalho** | Chave de acesso, número da NFS-e, número e série da DPS, data de competência, data/hora de emissão, situação da NFS-e e finalidade |
| **Emitente** | CNPJ, inscrição municipal (CCM), razão social, telefone, e-mail, endereço completo com CEP |
| **Tomador / Adquirente** | Nome/razão social, CNPJ/CPF/NIF, inscrição municipal, telefone, e-mail, endereço completo |
| **Destinatário da Operação** | Exibido apenas quando o endereço de entrega representa o destinatário da operação: nome/razão social, CNPJ/CPF, município com UF, CEP e endereço completo (veja seção 7) |
| **Serviço Prestado** | Códigos de tributação nacional/municipal, código da NBS, local da prestação com UF e país, descrição do código de tributação e descrição do serviço |
| **Tributação Municipal** | Tipo de tributação ISSQN, município de incidência, regime especial, imunidade, suspensão de exigibilidade |
| **Cálculo do Tributo** | Deduções/reduções, base de cálculo ISSQN, alíquota aplicada, retenção ISSQN, ISSQN apurado |
| **Tributação Federal (Exceto CBS)** | IRRF, INSS retido, contribuições sociais retidas (CSLL/PIS/COFINS), PIS e COFINS débito apuração própria |
| **Tributação IBS / CBS** | CST/cClassTrib, indicador de operação e município de incidência, base de cálculo, reduções de alíquota, alíquotas e valores apurados de IBS (estadual e municipal) e CBS |
| **Valor Total da NFS-e** | Valor do serviço, descontos condicionados e incondicionados, ISSQN retido, retenções federais, valor líquido, total do IBS/CBS e valor líquido da NFS-e + IBS/CBS |
| **Informações Complementares** | Notas adicionais e totais aproximados dos tributos (Lei nº 12.741/2012), estimados por esfera federal, estadual e municipal (IBPT) |

### Impostos Impressos

O DANFSe traz os mesmos impostos que a NFS-e envia à prefeitura ou ao ambiente nacional. Um imposto da nota marcado como **Não Destacar** não é enviado na NFS-e e, por isso, também não aparece no DANFSe. A marcação vem do campo **Não Destacar** da **Jurisdição Imposto** usada no cálculo e é gravada nos impostos da nota no lançamento.

O imposto Não Destacar fica de fora de todos os quadros e totais do documento:

- **Cálculo do Tributo**: ISSQN apurado e retenção do ISSQN;
- **Tributação Federal (Exceto CBS)**: IRRF, INSS retido, CSLL, PIS e COFINS retidos, e PIS e COFINS débito apuração própria;
- **Tributação IBS / CBS**: valores de IBS e CBS;
- **Valor Total da NFS-e**: ISSQN retido, retenções federais, valor líquido, total do IBS/CBS e valor líquido da NFS-e + IBS/CBS.

A regra vale também para as retenções que o sistema apura a partir do título a receber do cliente. Assim, para a mesma nota, os impostos, as retenções e o valor líquido impressos são iguais aos transmitidos, e o valor líquido não desconta uma retenção que não foi enviada.

No DANFSe montado pelo XML da NFS-e autorizada (veja [Origem dos Dados](#origem-dos-dados)), os impostos, as retenções e o valor líquido são lidos da própria nota, e por isso são sempre os transmitidos. As regras desta seção valem para o DANFSe montado com os dados do Business Central: a DPS, o rascunho, o RPS e o DANFSe impresso quando o XML não está disponível.

Isso vale para todas as impressões da NFS-e:

| Impressão | Onde |
| --- | --- |
| DANFSe da NFS-e autorizada | **Abrir DANFSe** e **Atualizar DANFSe**, no **Documento Fiscal Registrado**, e o anexo dos **E-Mails de Documento** |
| DPS, antes da emissão na prefeitura | **Imprimir**, na **NFS-e - Envio de Lote**, e **Imprimir RPS**, no **Documento Fiscal de Venda** (veja seção 2) |
| Rascunho antes do lançamento | **Gerar Rascunho do DANFSe**, na **Ordem de Venda** e na **Fatura de Venda** (veja seção 3) |
| RPS das filiais sem **Utilizar Layout Padrão** | **Imprimir**, na **NFS-e - Envio de Lote**, e **Imprimir RPS**, no **Documento Fiscal de Venda** |

Um imposto Não Destacar não impede a impressão, qualquer que seja a sua **Classificação**, inclusive em branco ou **Outro**. Notas sem imposto Não Destacar saem com os mesmos valores de antes.

Um DANFSe guardado na nota antes de Novembro/2026 pode ainda mostrar os impostos Não Destacar: veja [Corrigir um DANFSe Guardado com Impostos Não Destacar](#corrigir-um-danfse-guardado-com-impostos-não-destacar).

### Imposto com Classificação sem Campo na NFS-e

Cada imposto vai para um campo da NFS-e conforme a **Classificação** da **Jurisdição Imposto**. As classificações em branco e **Outro** não têm campo correspondente na NFS-e. Quando um imposto **destacado** (com **Não Destacar** desligado) tem uma dessas classificações, o sistema não gera a NFS-e e exibe a mensagem:

*"A NFS-e do documento fiscal &lt;nº&gt; não pode ser gerada: o imposto da Jurisdição Imposto &lt;código&gt; tem a Classificação &lt;classificação&gt;, que não tem campo correspondente na NFS-e. Corrija a Classificação na Jurisdição Imposto &lt;código&gt;; se o documento já foi lançado, cancele-o e lance-o de novo."*

A classificação em branco aparece na mensagem como "(em branco)".

O bloqueio acontece na transmissão da NFS-e e em todas as impressões da tabela acima, inclusive no rascunho antes do lançamento. A nota bloqueada não recebe número de RPS nem é enviada. Nas filiais do Emissor Público Nacional, a pendência aparece também na conferência de cadastros que antecede a transmissão, na página **Mensagens de Erro**, junto das demais (veja [NFS-e: Emissão pelo Emissor Público Nacional (SEFIN)](Localization-for-Brazil_User-Manual_NFSe-Emissor-Publico-Nacional-SEFIN.md#transmitir)).

O lançamento do documento de venda não é bloqueado por esse motivo. A **Classificação** fica gravada nos impostos da nota no lançamento e não muda depois. Por isso, para resolver:

1. Abra **Jurisdições de Impostos (Brasil)**, selecione a jurisdição indicada na mensagem e, no **Cartão Jurisdição Imposto**, corrija a **Classificação**.
2. Se a nota ainda não foi lançada, por exemplo no rascunho, gere de novo.
3. Se a nota já foi lançada, use **Cancelar Documento Fiscal** no **Documento Fiscal Registrado** e lance o documento de venda de novo. A nova nota é calculada com a classificação corrigida.

### Elementos Visuais

- **Logotipo da Prefeitura**: Exibido no cabeçalho, a partir do cadastro do município (veja seção 4)
- **QR Code**: Link para consulta pública da NFS-e em nfse.gov.br
- **Marca d'água de Homologação**: Quando o ambiente for de homologação, o documento exibe "NFS-e SEM VALIDADE JURÍDICA". O rascunho gerado antes do lançamento exibe essa marcação em qualquer ambiente, inclusive produção (veja seção 3)
- **Marca d'água de Cancelamento**: Quando a NFS-e está cancelada, o documento exibe a marca d'água diagonal "CANCELADA" (item 2.5.1 da NT SE/CGNFS-e nº 008)

### Formato de Datas, Horas e Valores

O DANFSe sai sempre no padrão brasileiro, independentemente da **Região** configurada para o usuário que o gera. O resultado é o mesmo quando o PDF é montado pela fila de **E-Mails de Documento**, por um usuário com Região Português (Brasil) ou por um usuário com Região em inglês ou em outro idioma:

| Informação | Formato | Exemplo |
| --- | --- | --- |
| **Competência da NFS-e** | dd/mm/aaaa | 15/10/2026 |
| **Data e Hora da emissão da NFS-e** | dd/mm/aaaa, hora em 24 horas, no horário de Brasília | 15/10/2026 21:21:49 |
| **Data e Hora da emissão da DPS** | dd/mm/aaaa, hora em 24 horas | 15/10/2026 00:00:00 |
| Valores em R$ (serviço, descontos, deduções, bases de cálculo, ISSQN, retenções federais, IBS/CBS e valor líquido) | Ponto no milhar e vírgula nos centavos | R$ 1.234,56 |
| Percentuais (alíquota do ISSQN, alíquotas, reduções e alíquotas efetivas de IBS/CBS) | Vírgula decimal | 5,00% |
| Totais aproximados dos tributos em **Informações Complementares** (Federais, Estaduais, Municipais) | Ponto no milhar e vírgula nos centavos | 1.234,56 |

A data e hora de emissão da NFS-e é sempre impressa no **horário de Brasília**. Ela não varia conforme o fuso horário do usuário ou da sessão que gera o PDF, e não segue o **Fuso Horário** da filial na **Configuração NF-e**. Assim, uma nota emitida às 21h30 em Brasília sai com 21:30 mesmo quando o PDF é gerado por um usuário em outro fuso.

Isso vale para todas as formas de geração do DANFSe: **Abrir DANFSe**, **Atualizar DANFSe**, **Imprimir RPS** (DPS), a impressão no **NFS-e - Envio de Lote**, **Gerar Rascunho do DANFSe** e o anexo enviado pelos **E-Mails de Documento**. Vale também para a NFS-e cancelada (com a marca d'água "CANCELADA") e para a NFS-e de homologação.

**Exceção — Descrição do Serviço com Lista Itens Detalhada.** Quando a **Lista Itens Detalhada** está ativada na **Configuração NF-e** da filial, as quantidades e os valores dessa lista, dentro da descrição do serviço, seguem o formato da Região de quem gera o documento. Esse texto é o mesmo enviado à prefeitura na DPS e não foi alterado.

**Layouts personalizados.** Se a empresa usa uma cópia própria do layout do DANFSe, criada antes desta versão, ela não recebe o formato brasileiro fixo e continua mostrando a data e hora de emissão da NFS-e no fuso de quem gera o PDF. Para obter o mesmo resultado, refaça a cópia a partir do layout padrão atual.

### DANFSe v2.0 (NT SE/CGNFS-e nº 008)

A Nota Técnica SE/CGNFS-e nº 008, de 05/05/2026, especifica o layout técnico do **DANFSe v2.0**, a representação impressa da NFS-e nacional. A API oficial de geração do DANFSe será **suspensa (sobrestada) a partir de 1º de julho de 2026**, e a geração local pelo sistema já segue o novo modelo.

Mudanças em relação à versão anterior:

- O cabeçalho passa a identificar o documento como **DANFSe v2.0**.
- Novos campos no bloco de identificação: **Situação da NFS-e** (exibido como "NFS-e Gerada") e **Finalidade** (exibido como "Regular").
- A seção do tomador passa a se chamar **TOMADOR / ADQUIRENTE**.
- A seção de tributos federais passa a se chamar **TRIBUTAÇÃO FEDERAL (EXCETO CBS)**.
- Nova seção **TRIBUTAÇÃO IBS / CBS** (detalhada abaixo).
- O bloco **VALOR TOTAL DA NFS-E** ganha os totais **Total do IBS/CBS** e **Valor Líquido da NFS-e + IBS/CBS**.
- A seção **SERVIÇO PRESTADO** foi reorganizada: códigos de tributação nacional/municipal lado a lado, novo campo **Código da NBS** (do cadastro de códigos de serviço NFS-e) e **Local da Prestação / Sigla UF / País**, seguidos da descrição do código de tributação e da descrição do serviço.
- Os totais aproximados de tributos (Lei nº 12.741/2012) deixam de ter seção própria e passam a ser exibidos dentro de **INFORMAÇÕES COMPLEMENTARES** (apenas quando há valores).
- NFS-e cancelada passa a ser impressa com a marca d'água **CANCELADA** (item 2.5.1).
- Novo quadro **DESTINATÁRIO DA OPERAÇÃO** (item 2.1.5), exibido apenas quando o endereço de entrega representa o destinatário da operação (veja seção 7).
- Os grupos **Imóvel** e **Obra** passam a ser informados dentro de **INFORMAÇÕES COMPLEMENTARES** quando o endereço de entrega representa o local da prestação (imóvel) ou a obra (veja seção 7).
- Possibilidade de imprimir uma **prévia (DPS)** do documento antes da emissão na prefeitura (veja seção 2).

#### Seção Tributação IBS / CBS

Os valores desta seção são preenchidos automaticamente a partir da apuração de impostos do documento fiscal (impostos IBS UF, IBS Mun e CBS) e da classificação tributária IBS/CBS das linhas do documento. Nenhuma configuração adicional é necessária além da configuração de impostos da Reforma Tributária.

| Campo | Conteúdo |
| --- | --- |
| **CST / cClassTrib** | Código de Situação Tributária e código de classificação tributária IBS/CBS do serviço |
| **Indicador de Operação / Código IBGE Incidência / Município Incidência / Sigla UF** | Indicador da operação e município de incidência do imposto |
| **Exclusões e Reduções da Base de Cálculo** | Valores excluídos/reduzidos da base de cálculo (exibido como "-" quando não aplicável) |
| **Base de Cálculo Após Exclusões e Reduções** | Base de cálculo do IBS/CBS |
| **Red. Alíquota IBS / Red. Alíquota CBS** | Percentuais de redução de alíquota (IBS estadual, IBS municipal e CBS) |
| **Alíquota – IBS UF / IBS Mun** e **Alíquota - CBS** | Alíquotas originais (sem benefício); quando não informadas, exibem a alíquota efetiva |
| **Alíq. Efetiva Estadual – IBS**, **Alíq. Efetiva Municipal – IBS** e **Alíquota Efetiva – CBS** | Alíquotas efetivamente aplicadas no cálculo |
| **Valor Apurado Estadual – IBS** e **Valor Apurado Municipal – IBS** | Valores apurados de IBS por esfera |
| **Valor Total Apurado – IBS** e **Valor Total Apurado – CBS** | Totais apurados de IBS (estadual + municipal) e de CBS |

## 2. Configuração

### Ativando o Layout Padrão

1. Navegue até **Configuração NF-e** (página NFeSetup)
2. Na aba **NFS-e**, localize o grupo **Layout Padrão**
3. Ative o campo **Utilizar Layout Padrão** (NFSUseStandardLayout)

### Comportamento

| Configuração | Comportamento ao visualizar/baixar imagem da NFS-e |
| --- | --- |
| **Desativado** (padrão) | O sistema baixa a imagem da NFS-e do web service da prefeitura municipal |
| **Ativado** | O sistema gera um PDF localmente usando o relatório DANFSe com layout padronizado, com os dados do XML da NFS-e autorizada (veja [Origem dos Dados](#origem-dos-dados)) |

### Pré-requisitos

Quando o layout padrão está ativado, o sistema exige que:

- A **Chave de Acesso** da NFS-e esteja preenchida. Sem ela, **Abrir DANFSe** e **Atualizar DANFSe** terminam com o erro *"O documento fiscal &lt;nº&gt; não tem Chave de Acesso NFS-e, que o Layout Padrão usa para montar o DANFSe. Se o município da filial &lt;filial&gt; emite a NFS-e em layout próprio, desmarque Utilizar Layout Padrão na Config. NF-e dessa filial."* Prefeituras com layout próprio, como São Paulo, não garantem a chave de acesso nacional: nas filiais delas, deixe **Utilizar Layout Padrão** desligado e obtenha o DANFSe como indica o manual do município (veja [NFS-e: Manuais de Emissão por Município e Plataforma](Localization-for-Brazil_User-Manual_NFSe-Manuais-por-Municipio.md)).
- O certificado da NFS-e da filial esteja válido. É com ele que o sistema busca o XML da nota no Ambiente de Dados Nacional (ADN), no ambiente da filial (produção ou homologação), quando o XML ainda não está guardado no documento fiscal. O município da filial precisa compartilhar as notas com o ADN; quando não compartilha, o XML não chega, e cada DANFSe depende da confirmação descrita em [Quando o XML Não Está Disponível](#quando-o-xml-não-está-disponível).

### Abrir e Atualizar o DANFSe

Na página **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`), grupo NFS-e, estão disponíveis as ações:

| Ação | Descrição |
| --- | --- |
| **Abrir DANFSe** | Abre o DANFSe (PDF) armazenado; se ainda não existir, ele é gerado e armazenado no documento |
| **Atualizar DANFSe** | Gera novamente o DANFSe, substituindo o arquivo armazenado — útil após atualizações de layout ou correções no documento |
| **Baixar XML da NFS-e** | Baixa o arquivo XML da NFS-e autorizada. O primeiro download consulta o XML no ADN pela chave de acesso e o guarda no documento fiscal; os seguintes usam o arquivo guardado. Fica habilitada quando o documento tem **Chave de Acesso NFS-e** |

Na página **Documento Fiscal** (página `FiscalDocumentFNX`), a ação **Abrir Imagem** abre o DANFSe da mesma forma que **Abrir DANFSe**, e a ação **Baixar XML da NFS-e** também está disponível.

Ao gerar o DANFSe de uma NFS-e emitida, o sistema segue esta ordem:

1. **PDF guardado**: se a nota já tem DANFSe guardado, **Abrir DANFSe** e **Abrir Imagem** abrem esse PDF, sem consultar o ADN e sem perguntar nada. Vale para os PDFs gerados antes desta versão e para os impressos com os valores do Business Central depois de um "Sim" no aviso. **Atualizar DANFSe** pula este passo.
2. **XML guardado**: se a nota já tem o XML, o DANFSe é montado com ele.
3. **XML do ADN**: se a nota não tem o XML, o sistema busca a nota no ADN pela chave de acesso, guarda o XML no documento fiscal e monta o DANFSe. Nas próximas vezes, usa o XML guardado, sem nova consulta.
4. **Aviso**: se o ADN não devolve a nota, o sistema avisa e pergunta o que fazer (veja a seção seguinte).

O PDF gerado fica armazenado junto à NFS-e e é reutilizado nas próximas visualizações e no envio por e-mail. Quando a NFS-e é cancelada, o arquivo armazenado é descartado automaticamente, e a próxima impressão é gerada com a marca d'água "CANCELADA". O XML continua guardado: o DANFSe da nota cancelada é montado com ele, sem nova consulta ao ADN.

Para passar ao DANFSe montado pelo XML uma nota que tem PDF guardado antes desta versão, ou impresso com os valores do Business Central quando o XML ainda não estava disponível, use **Atualizar DANFSe** na nota. Não há regeração em lote: cada nota muda quando alguém usa a ação.

### Quando o XML Não Está Disponível

Se a nota não tem o XML guardado e o ADN não o devolve, **Abrir DANFSe**, **Abrir Imagem**, **Atualizar DANFSe** e **Imprimir RPS** de nota emitida mostram a mensagem:

*"Os dados da NFS-e com chave de acesso &lt;chave&gt; não estão disponíveis: &lt;motivo&gt;. Deseja imprimir o DANFSe com os valores calculados pelo Business Central? Ele ficará guardado como o DANFSe desta NFS-e."*

O motivo é um destes, seguido do detalhe técnico devolvido pelo ADN ou pela conexão:

| Motivo | Quando acontece |
| --- | --- |
| *a NFS-e não foi encontrada no Ambiente Nacional (…)* | O ADN não tem a nota, por exemplo porque o município ainda não a compartilhou |
| *o Ambiente Nacional não respondeu (…)* | O ADN está fora do ar, demorou a responder ou devolveu uma resposta que não pôde ser lida |
| *o certificado da NFS-e da filial &lt;filial&gt; não pôde ser utilizado (…)* | A filial não tem certificado da NFS-e, ou ele está vencido ou foi recusado pelo ADN |

- **Sim**: o DANFSe é montado com os dados do Business Central, com o mesmo conteúdo que tinha antes desta versão, e fica guardado como o DANFSe da nota. É esse PDF que **Abrir DANFSe** e o e-mail do documento passam a usar.
- **Não**: nada é impresso e nada muda na nota. Em **Atualizar DANFSe**, o PDF que já estava guardado continua o mesmo. O sistema não abre o link de consulta da prefeitura no lugar do DANFSe.

Depois que o XML estiver disponível no ADN, **Atualizar DANFSe** gera o PDF montado pelo XML no lugar do impresso com os valores do Business Central.

### E-mail do Documento

O e-mail do documento, tanto pela fila de **E-Mails de Documento** quanto pela ação **Enviar Agora** da página **E-Mails de Documento** (página `DocumentEmailsFNX`), anexa o DANFSe guardado na nota. Se a nota ainda não tem PDF guardado, o DANFSe é gerado pelo XML: primeiro o guardado, depois o do ADN.

O e-mail nunca mostra o aviso e nunca usa os valores do Business Central por conta própria. Se a nota não tem XML e o ADN não o devolve:

- o e-mail fica com **Status** **Pendente** e **Detalhes** "Aguardando XML da NFS-e no Ambiente Nacional";
- o motivo, com a chave de acesso, fica na **Última Mensagem de Erro**;
- a cada execução, a fila tenta de novo buscar o XML no ADN. Quando consegue, o e-mail sai com o DANFSe montado pelo XML.

Não há prazo: o e-mail espera até o XML chegar. Para liberá-lo antes, abra a nota no **Documento Fiscal Registrado**, clique em **Abrir DANFSe** e responda **Sim** ao aviso. O PDF fica guardado na nota, e a próxima execução da fila envia o e-mail com ele.

> **Atenção**: se o município de uma filial com **Utilizar Layout Padrão** não compartilha as notas com o ADN, todo e-mail de NFS-e dessa filial fica **Pendente** até alguém aceitar, nota a nota, a impressão com os valores do Business Central. Não há liberação em lote.

Nas filiais sem **Utilizar Layout Padrão** nada muda: o e-mail continua com o PDF ou o link do município.

### Corrigir um DANFSe Guardado no Formato Americano

Antes da versão de Outubro/2026, o DANFSe gerado pela fila de **E-Mails de Documento** ou por um usuário com Região em inglês saía no padrão americano: datas em mês/dia/ano, hora com AM/PM, valores como R$ 1,234.56 e percentuais como 5.00%. Como o PDF fica guardado na nota, esse formato continua aparecendo em **Abrir DANFSe** e no e-mail até que o arquivo seja gerado de novo.

Para corrigir uma nota afetada:

1. Abra a nota na página **Documento Fiscal Registrado**.
2. Clique em **Atualizar DANFSe**. O PDF guardado é substituído por um novo, no padrão brasileiro, qualquer que seja a Região de quem executa a ação.
3. Se o cliente precisa receber o documento corrigido, abra **E-Mails de Documento** (página `DocumentEmailsFNX`), selecione o e-mail da nota e clique em **Colocar como Aguardando**. Em seguida, clique em **Enviar Agora**. O anexo vai com o PDF novo.

A correção é feita nota a nota. O reenvio dos e-mails já enviados não é automático e fica a critério do usuário.

### Corrigir um DANFSe Guardado com Impostos Não Destacar

Antes da versão de Novembro/2026, o DANFSe mostrava também os impostos marcados como **Não Destacar**, que não foram enviados na NFS-e (veja [Impostos Impressos](#impostos-impressos)). Nesses documentos, as retenções e o valor líquido impressos podiam ser diferentes dos da NFS-e transmitida. O PDF guardado na nota não é alterado pela atualização e continua aparecendo em **Abrir DANFSe** e no e-mail até ser gerado de novo.

Para corrigir uma nota afetada, siga os mesmos passos da seção anterior: **Atualizar DANFSe** no **Documento Fiscal Registrado** e, se o cliente precisa receber o documento corrigido, reenvio pelos **E-Mails de Documento**. O novo PDF sai com os impostos, as retenções e o valor líquido enviados na NFS-e.

A DPS e o RPS não ficam guardados: cada impressão é gerada de novo e já sai com os valores transmitidos.

### Impressão de Prévia (DPS)

Quando o documento ainda **não foi emitido na prefeitura** (sem número de NFS-e), o DANFSe é impresso em **modo DPS** (Declaração de Prestação de Serviços). Nesse modo:

- O título e o subtítulo do documento passam a **DPS** e **Documento Auxiliar da DPS**.
- São omitidos os dados que só existem após a emissão: chave de acesso, número da NFS-e, data de competência, situação da NFS-e, data/hora de emissão e QR Code.
- O número fiscal do documento é criado na primeira impressão, da mesma forma que o antigo relatório RPS.

A impressão da prévia está disponível nos seguintes pontos, com o **Utilizar Layout Padrão** ativado:

| Página | Ação |
| --- | --- |
| **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`) | **Imprimir** |
| **Documento Fiscal de Venda** (página `SalesFiscalDocumentFNX`) | **Imprimir RPS** |

Quando a nota **já foi emitida** (tem **Chave de Acesso NFS-e**), essas mesmas ações abrem o DANFSe da NFS-e autorizada, como **Abrir DANFSe**: o PDF guardado ou, se não houver, o DANFSe montado pelo XML, com o aviso de [Quando o XML Não Está Disponível](#quando-o-xml-não-está-disponível) quando o XML não chega. A prévia com os dados do Business Central fica para a nota ainda não emitida.

Nos municípios em que o layout padrão **não** está ativado, essas ações continuam gerando o documento RPS legado. O RPS segue a mesma regra de impostos do DANFSe: os impostos Não Destacar ficam de fora, e um imposto destacado sem campo na NFS-e bloqueia a impressão (veja seção 1).

Para conferir o documento **antes** do lançamento, a partir da Ordem de Venda ou da Fatura de Venda, veja a seção 3.

## 3. Rascunho do DANFSe Antes do Lançamento

### O que é

A prévia descrita na seção anterior só existe **depois** do lançamento, quando o documento fiscal já foi criado. Se algo estivesse errado, era preciso cancelar o documento lançado e refazer todo o processo.

Para evitar esse retrabalho, a **Ordem de Venda** e a **Fatura de Venda** passam a permitir gerar um **rascunho do DANFSe a partir do documento ainda não lançado**, conferindo tomador, serviço, código de tributação, base de cálculo, ISS, retenções e totais antes de gerar qualquer documento fiscal — a mesma conferência que a NF-e de mercadorias já oferece com **Validar** e **Visualizar DANFe (Teste)**.

O rascunho é um documento **interno de conferência**. Ele não é o DPS, que continua sendo impresso após o lançamento e mantém o seu valor, nem a NFS-e definitiva autorizada pela prefeitura. Para que não seja confundido com nenhum dos dois, o rascunho carrega a marcação **NFS-e SEM VALIDADE JURÍDICA** em **qualquer ambiente**, inclusive produção — enquanto no DPS e na NFS-e autorizada essa marcação continua aparecendo apenas em homologação.

Gerar o rascunho **não** produz efeito fiscal algum:

- nenhum documento fiscal é criado;
- nenhuma numeração fiscal é consumida;
- nada é enviado à prefeitura nem ao Ambiente Nacional;
- nenhum movimento contábil, de estoque ou de cliente é registrado.

Internamente o sistema simula o lançamento do documento para montar os valores, gera o PDF e **desfaz tudo** em seguida. Por isso o rascunho pode ser gerado quantas vezes for necessário durante a montagem do documento, sem deixar rastro.

### Pré-requisitos

- O **Tipo Documento Fiscal** do documento de venda deve emitir NFS-e própria. Caso contrário, a ação recusa o documento com a mensagem *"A prévia do documento de serviço só está disponível quando o Tipo Documento Fiscal emite NFS-e própria."*
- A filial deve ter o **Utilizar Layout Padrão** ativado (veja seção 2). Nos municípios que ainda usam o layout municipal antigo o rascunho não se aplica — eles continuam emitindo o RPS legado como hoje, sem configuração adicional.
- O usuário precisa das mesmas permissões exigidas para lançar o documento de venda.

### Como Acessar

As ações estão na **Ordem de Venda** (página `Sales Order`) e na **Fatura de Venda** (página `Sales Invoice`), em dois pontos: no menu **Ações**, grupo **NFS-e**, logo após o grupo **NF-e**; e na aba **Fiscal** da barra de ações, grupo promovido **NFS-e**.

| Ação | Descrição |
| --- | --- |
| **Gerar Rascunho do DANFSe** | Monta o rascunho do DANFSe a partir do documento ainda não lançado e o disponibiliza para download |
| **Ver DANFSe (Rascunho)** | Abre novamente o rascunho gerado pela ação anterior |

As ações ficam sempre visíveis, mas só são **habilitadas** quando o documento é de serviço — ou seja, quando o **Tipo Documento Fiscal** informado emite NFS-e própria (campo **NF-e de Serviço** do documento de venda). Em documentos de mercadoria elas permanecem desabilitadas e nada muda.

Elas existem apenas nos documentos de saída: Nota de Crédito e Ordem de Devolução de Venda não emitem NFS-e e não recebem o rascunho.

### Gerando o Rascunho

1. Abra a **Ordem de Venda** ou a **Fatura de Venda** que deseja conferir, ainda **não lançada**.
2. Clique em **Gerar Rascunho do DANFSe**.
3. O sistema simula o lançamento, monta o DANFSe e desfaz a simulação.
4. É exibida a confirmação **Rascunho pronto**, com a mensagem *"O rascunho do DANFSe do documento &lt;nº&gt; está pronto. O documento ainda não foi registrado: o rascunho é um RPS montado a partir dos dados do Business Central, sem NFS-e emitida."* e o botão **Abrir DANFSe**.
5. Clique em **Abrir DANFSe** para baixar o PDF imediatamente, ou feche a mensagem e use **Ver DANFSe (Rascunho)** depois.

O arquivo é entregue como PDF, com o nome do documento de venda de origem. Se você acionar **Ver DANFSe (Rascunho)** sem ter gerado nenhum rascunho para aquele documento, o sistema informa: *"Use a ação Gerar Rascunho do DANFSe para montar o rascunho deste documento primeiro."*

Cada geração substitui o rascunho anterior. Depois de corrigir o documento, gere novamente para obter uma conferência atualizada — o rascunho guardado **não** é atualizado sozinho quando o documento é alterado.

### O Que Sai Impresso

O rascunho usa o mesmo relatório DANFSe em **modo DPS**, com as omissões descritas na seção 2 (sem chave de acesso, número da NFS-e, data de competência, situação da NFS-e, data/hora de emissão e QR Code), porque o documento ainda não foi emitido na prefeitura. Sobre isso, acrescenta a marcação **NFS-e SEM VALIDADE JURÍDICA** em qualquer ambiente.

As demais seções — emitente, tomador, serviço prestado, tributação municipal e federal, cálculo do tributo, retenções (IR, INSS, PIS, COFINS, CSLL) e totais — trazem os valores que seriam emitidos caso o documento fosse lançado naquele momento. Os impostos são os mesmos que seriam transmitidos na NFS-e: os marcados como **Não Destacar** ficam de fora (veja [Impostos Impressos](#impostos-impressos)).

Se um imposto destacado tem **Classificação** em branco ou **Outro**, a geração é interrompida com a mensagem descrita em [Imposto com Classificação sem Campo na NFS-e](#imposto-com-classificação-sem-campo-na-nfs-e), e nenhum rascunho é guardado. Corrija a **Classificação** na **Jurisdição Imposto** e gere novamente.

### Comportamento

**Documento com erros de lançamento.** Como o rascunho simula o lançamento completo, qualquer condição que impediria o registro — dimensão obrigatória, período contábil fechado, cliente bloqueado, item sem custo — também impede a geração. Nesse caso o sistema apresenta os erros normalmente, na página **Mensagens de Erro**, e nenhum rascunho é entregue: o rascunho anterior é descartado antes da simulação, de forma que **Ver DANFSe (Rascunho)** nunca devolva um arquivo desatualizado como se fosse a conferência atual. Corrija os problemas apontados e gere novamente.

**Documento ainda não aprovado.** O rascunho não exige que o documento já tenha passado pelo fluxo de aprovação — é justamente durante a montagem que a conferência é útil. A verificação de aprovação prévia ao lançamento é ignorada enquanto o rascunho é montado.

**Movimentação de estoque.** O campo **Ignorar Registro de Itens na Validação** (`ValidationSkipItemsPost`), da página **Config. NF-e**, também se aplica ao rascunho da NFS-e: quando ativado, a simulação não processa o registro de itens, o que a torna mais rápida. Em qualquer dos casos nada é efetivamente gravado, porque a simulação inteira é desfeita.

**Numeração fiscal.** O número do DPS impresso no rascunho é o que o próprio lançamento simulado atribui, e ele é devolvido à série quando a simulação é desfeita — a numeração não avança. A exceção é a série configurada com **Permitir lacunas em números** ativado: nesse modo o Business Central usa uma sequência de banco de dados, que não é revertida, e cada rascunho consome um número. É o mesmo comportamento que a prévia da NF-e de mercadorias já possui. Se a numeração fiscal não pode ter lacunas, mantenha essa opção desativada na série usada como **Nº Série Fiscal**.

**Desempenho.** Cada rascunho equivale a um lançamento completo seguido da geração de um PDF. Em documentos de serviço, com poucas linhas, o tempo de resposta é próximo ao do lançamento do próprio documento.

### O Que Não Muda

O DPS impresso após o lançamento (**NFS-e - Envio de Lote** e **Imprimir RPS**), o DANFSe da NFS-e já autorizada, as regras e o fluxo de emissão e transmissão da NFS-e e o RPS legado dos municípios fora do layout nacional permanecem exatamente como antes — o rascunho apenas mostra o que já seria emitido, e a marcação de "sem validade jurídica" não é acrescentada a nenhum deles.

## 4. Logotipo do Município

O layout DANFSe exibe o logotipo da prefeitura no cabeçalho do documento. Para isso, é necessário configurar o logotipo no cadastro do município.

### Campos Novos na Tabela de Municípios

| Campo | Descrição |
| --- | --- |
| **Nome da Entidade Federal** | Nome da prefeitura/entidade (ex: "Prefeitura Municipal de São Paulo"). Exibido no cabeçalho do DANFSe |
| **Logotipo** | Imagem da prefeitura em formato BMP |
| **Logotipo Carregado** | Indica se um logotipo foi enviado |

### Como Enviar o Logotipo

1. Navegue até **Municípios** (página Municipios)
2. Selecione o município desejado
3. Preencha o campo **Nome da Entidade Federal**
4. Utilize uma das opções:
Clique no campo **Logotipo** na linha do município, ou
No menu **Ações > Processamento > Imagem**, selecione **Enviar Logotipo**
5. Selecione um arquivo de imagem **BMP**

### Gerenciamento do Logotipo

As seguintes ações estão disponíveis no menu **Ações > Processamento > Imagem**:

| Ação | Descrição |
| --- | --- |
| **Enviar Logotipo** | Faz upload de uma nova imagem BMP |
| **Remover Logotipo** | Remove o logotipo existente |
| **Visualizar Logotipo** | Baixa/visualiza o logotipo atual |

## 5. Detalhamento IBPT por Esfera Tributária

### O que mudou

A tabela de tributos IBPT agora armazena as alíquotas detalhadas por esfera governamental, em vez de apenas os totais agregados.

### Novos Campos na Tabela IBPT

| Campo | Descrição |
| --- | --- |
| **Alíquota Federal Nacional** | Percentual de tributos federais para produtos/serviços nacionais |
| **Alíquota Federal Importação** | Percentual de tributos federais para produtos/serviços importados |
| **Alíquota Estadual** | Percentual de tributos estaduais |
| **Alíquota Municipal** | Percentual de tributos municipais |

**Nota**: Os campos existentes *Alíquota Nacional* e *Alíquota Importação* passam a ser calculados automaticamente como a soma das alíquotas federal + estadual + municipal, mantendo compatibilidade com processos anteriores.

### Importação do Arquivo IBPT

Ao importar o arquivo CSV da tabela IBPT, o sistema agora lê automaticamente as colunas de alíquota estadual (coluna 7) e municipal (coluna 8) do arquivo. Nenhuma ação adicional do usuário é necessária — basta importar o arquivo normalmente.

### Onde os Detalhes Aparecem

O detalhamento por esfera tributária flui por todo o sistema:

1. **Página Tributos IBPT**: Novos campos visíveis na lista
2. **Subformulário do Documento Fiscal de Venda**: Campos ocultos por padrão (podem ser exibidos via personalização)
3. **Página Tributos Estimados Detalhados (IBPT)**: Campos visíveis por padrão
4. **Transmissão NFS-e via Web Service**: As alíquotas detalhadas são enviadas no XML do lote RPS
5. **Relatório DANFSe**: os totais aproximados dos tributos, calculados por esfera (Federais, Estaduais, Municipais), são exibidos em **Informações Complementares**

## 6. Suspensão de Exigibilidade do ISS

### O que é

Permite configurar a suspensão da exigibilidade do ISS por código de serviço NFS-e, indicando se o ISS está suspenso por decisão judicial ou processo administrativo.

### Novos Campos nos Códigos de Serviço NFS-e

Os campos foram adicionados em dois níveis:

**Códigos de Serviço NFS-e** (página NFSServiceCodes):

| Campo | Opções | Descrição |
| --- | --- | --- |
| **Tipo de Suspensão da ISS** | *(em branco)*, Decisão Judicial, Processo Administrativo | Tipo de suspensão da exigibilidade do ISS |
| **Nº Processo de Suspensão** | Texto livre (até 30 caracteres) | Número do processo judicial ou administrativo |

**Códigos de Serviço NFS-e por Filial** (página BranchNFSServiceCodes):

Os mesmos campos estão disponíveis no nível da filial, permitindo configurações específicas por estabelecimento.

## 7. Destinatário da Operação e Local da Prestação

### O que é

A NT SE/CGNFS-e nº 008 prevê a identificação do **destinatário da operação** e do **local da prestação** na NFS-e Nacional. São duas informações **independentes**: quem recebe o serviço e onde ele é prestado podem estar em endereços diferentes, e os dois vão na mesma nota.

Na localização isso se traduz assim:

- O **endereço de entrega** do documento de venda é o **endereço do destinatário**.
- O **local da prestação** tem campos próprios, informados pela ação **NFS-e** do documento.

### Onde informar o local da prestação

Na factbox **Dados Fiscais** do documento de venda, logo abaixo da entrada de NF-e, há a entrada **NFS-e**. Ela fica disponível quando o **Tipo de Documento Fiscal** está marcado como serviço. Clique nela para abrir a tela **Dados da NFS-e**, com dois grupos:

- **Local da Prestação** — os campos **Cód. Obra**, **Cód. Atividade/Evento** e **Cód. Imóvel**.
- **Endereço do Local da Prestação** — CEP, logradouro, número, complemento, bairro, cidade, UF e país.

Selecionar uma obra, um evento ou um imóvel que tenha endereço cadastrado **preenche automaticamente o endereço abaixo**. O endereço também pode ser digitado direto, o que atende as operações cujo local de fornecimento não tem cadastro próprio.

Os três códigos são **mutuamente exclusivos**: preencher um limpa os outros dois, porque uma operação informa um único local de prestação.

### Quais campos ficam habilitados

Só o cadastro que a operação realmente reporta fica habilitado, seguindo o código de serviço da primeira linha de serviço do documento:

| Situação do código de serviço | Campo habilitado |
| --- | --- |
| **Construção Civil** marcado | **Cód. Obra** — tem precedência sobre o local de fornecimento |
| Local de Fornecimento do `indOp` = **Imóvel** | **Cód. Imóvel** |
| Local de Fornecimento do `indOp` = **Atividade/Evento** | **Cód. Atividade/Evento** |
| Tomador/Adquirente, Fornecedor, Destinatário, Outros ou em branco | Nenhum — informe o endereço direto, se for o caso |

O **Cód. Atividade/Evento** é o único obrigatório: uma operação com local de fornecimento **Atividade/Evento** não é registrada sem atividade ou evento vinculado, e o registro vinculado precisa ter data de início e de fim.

### Cadastros de Obra, Atividade/Evento e Imóvel

Três cadastros com a mesma estrutura — código, descrição e endereço completo — mais o que é específico de cada um:

| Cadastro | Campos específicos |
| --- | --- |
| **Obras** | Nº CNO, Tipo de Obra, Cód. ART |
| **Atividades/Eventos** | ID da Atividade/Evento, Data de Início, Data de Término |
| **Imóveis** | CIB, Inscrição Imobiliária Fiscal, Cód. Município |

Os três estão na **Configuração Manual** (categoria Fiscal) e nos Centros de Função. Um cadastro **não pode ser excluído** enquanto houver documento fiscal que o referencie — seus dados são lidos toda vez que a NFS-e é montada e o DANFSe é impresso.

O **CIB** e a **Inscrição Imobiliária Fiscal** do imóvel são informados na NFS-e. Pelo menos um entre CIB, inscrição imobiliária e endereço precisa estar preenchido no imóvel vinculado, senão o registro é bloqueado. Um imóvel com país diferente do Brasil é informado como imóvel estrangeiro.

#### Numeração automática dos códigos

Os códigos de Obra, Atividade/Evento e Imóvel podem ser atribuídos por numeração automática. Configure em **Configuração Fiscal**, grupo NFS-e:

| Campo | Cadastro |
| --- | --- |
| **Nos. Obra** | Obras |
| **Nos. Atividade/Evento** | Atividades/Eventos |
| **Nos. Imóvel** | Imóveis |

A numeração é **opcional** — deixe em branco para continuar digitando os códigos. Como o código aceita 10 caracteres, configure uma numeração que gere números de até 10 caracteres; se ela gerar algo maior, o sistema avisa em vez de cortar o código.

### Papel do Endereço de Entrega

O campo **Papel do Endereço de Entrega** continua sendo determinado no registro e indica o que o endereço de entrega representa naquele documento:

| Papel | Quando se aplica |
| --- | --- |
| **Nenhum** | Não há endereço de entrega distinto do endereço de cobrança |
| **Destinatário** | O documento tem local da prestação próprio — o endereço de entrega é o do destinatário |

Documentos criados **antes** desta funcionalidade, que não têm local da prestação próprio, continuam usando o comportamento anterior, em que o endereço de entrega acumulava os dois papéis e podia assumir também **Local da Prestação**, **Obra** ou **Não Reportado**. Isso garante que documentos já registrados sejam reimpressos e retransmitidos exatamente como antes.

> **Atenção nos documentos em aberto criados antes da atualização.** Neles, selecionar a obra ou o evento copiava o endereço do cadastro por cima do **endereço de entrega**. Antes de registrar, informe o local da prestação pela ação NFS-e e confira se o endereço de entrega voltou a ser o do destinatário.

### Campo Construção Civil no Código de Serviço

O campo **Construção Civil** (`CivilConstruction`) no cadastro de **Códigos de Serviço NFS-e** indica que o serviço se refere a uma obra.

Quando marcado, o grupo **Obra** das Informações Complementares apresenta o **Código da Obra** (CNO) quando ele está preenchido; caso contrário, apresenta o endereço da obra. **A obra tem precedência**: num serviço de construção civil, o DANFSe imprime a obra mesmo que o `indOp` indique imóvel ou evento.

A marcação também vai na transmissão da NFS-e e, nos layouts que informam a obra, é ela que decide se a nota leva o grupo da obra. Antes, essa decisão seguia uma lista de códigos de serviço de construção civil mantida para cada layout de prefeitura. Por isso, marque **Construção Civil** em todo código de serviço de obra: sem a marcação, a NFS-e é enviada sem os dados da obra e pode ser rejeitada quando o código de operação exige a obra.

### Como Aparece no DANFSe

Com um local da prestação informado, o **quadro do destinatário e a linha do local da prestação aparecem juntos** — é justamente o que a separação das duas informações permite.

- **Destinatário da Operação**: o quadro **DESTINATÁRIO DA OPERAÇÃO** traz nome/razão social, CNPJ/CPF, município com UF, CEP e endereço completo, vindos do endereço de entrega. Inscrição municipal, telefone e e-mail não têm origem no endereço de entrega e aparecem como "-".
- **Imóvel**: `Imóvel: <endereço>`.
- **Local da atividade/evento**: `Local da atividade/evento: <endereço>`, com uma segunda linha trazendo nome, identificação e período — `Atividade/Evento: <nome> (<ID>) - Período: <início> a <fim>`. Quando só um dos dois está cadastrado, aparece o que houver.
- **Obra**: `Obra - Código da Obra: <CNO>` quando há CNO, senão `Obra: <endereço>`.
- **Local da prestação**: `Local da prestação: <endereço>` nos demais casos.

### Dados congelados no Documento Fiscal

O endereço de entrega e o local da prestação são gravados no documento fiscal no registro. No documento fiscal os dois blocos permanecem **editáveis** — informar obra, evento ou imóvel ali também preenche o endereço do local da prestação, do mesmo jeito que no documento de venda.

O CPF/CNPJ e a Inscrição Estadual do destinatário só são preenchidos quando o endereço de entrega vem de um cadastro de endereço de entrega; endereços digitados manualmente ficam sem esses dados. Os campos estão listados na seção 8.

## 8. Resumo dos Campos Novos

### Configuração NF-e

| Campo | Localização | Tipo | Descrição |
| --- | --- | --- | --- |
| Utilizar Layout Padrão | Aba NFS-e > Layout Padrão | Booleano | Ativa geração local do PDF via DANFSe |

### Municípios

| Campo | Tipo | Descrição |
| --- | --- | --- |
| Nome da Entidade Federal | Texto (100) | Nome da prefeitura para cabeçalho do DANFSe |
| Logotipo | Imagem (BMP) | Brasão/logo da prefeitura |
| Logotipo Carregado | Booleano | Indica presença do logotipo |

### Tributos IBPT

| Campo | Tipo | Descrição |
| --- | --- | --- |
| Alíquota Federal Nacional | Decimal | % tributos federais (nacional) |
| Alíquota Federal Importação | Decimal | % tributos federais (importado) |
| Alíquota Estadual | Decimal | % tributos estaduais |
| Alíquota Municipal | Decimal | % tributos municipais |

### Códigos de Serviço NFS-e

| Campo | Tipo | Descrição |
| --- | --- | --- |
| Tipo de Suspensão da ISS | Opção | Tipo de suspensão (Judicial/Administrativo) |
| Nº Processo de Suspensão | Texto (30) | Número do processo |
| Construção Civil | Booleano | Serviço referente a obra de construção civil. Decide se a NFS-e é transmitida com o grupo da obra |

### Documento Fiscal (Endereço de Entrega)

Campos congelados automaticamente no registro do documento, usados para informar o destinatário/endereço de entrega na NFS-e:

| Campo | Tipo | Descrição |
| --- | --- | --- |
| Papel do Endereço de Entrega | Opção | Destinatário ou Nenhum. Documentos anteriores à separação entre destinatário e local da prestação também podem ter Local da Prestação, Obra ou Não Reportado |
| Nome de Entrega | Texto (100) | Nome do destinatário/endereço de entrega |
| CPF/CNPJ de Entrega | Texto (20) | CPF ou CNPJ do endereço de entrega |
| I.E. de Entrega | Texto (30) | Inscrição Estadual do endereço de entrega |
| Endereço de Entrega | Texto (100) | Logradouro |
| Número de Entrega | Texto (12) | Número |
| Endereço de Entrega 2 | Texto (50) | Complemento |
| Bairro de Entrega | Texto (30) | Bairro |
| Cidade de Entrega | Texto (30) | Cidade |
| CEP de Entrega | Código (20) | CEP |
| Cód. UF de Entrega | Código (10) | Estado/UF |
| Cód. País de Entrega | Código (10) | País |

### Documento Fiscal (Local da Prestação)

Gravados no registro a partir da tela **Dados da NFS-e** do documento de venda, e editáveis no documento fiscal:

| Campo | Tipo | Descrição |
| --- | --- | --- |
| Cód. Obra | Código (10) | Obra vinculada — informa CNO e ART na NFS-e |
| Cód. Atividade/Evento | Código (10) | Atividade ou evento vinculado — informa nome, ID e período na NFS-e |
| Cód. Imóvel | Código (10) | Imóvel vinculado — informa CIB e inscrição imobiliária na NFS-e |
| Endereço do Local da Prestação | Texto (100) | Logradouro |
| Número do Local da Prestação | Texto (12) | Número |
| Complemento do Local da Prestação | Texto (50) | Complemento |
| Bairro do Local da Prestação | Texto (30) | Bairro |
| Cidade do Local da Prestação | Texto (30) | Cidade |
| CEP do Local da Prestação | Código (20) | CEP |
| Cód. UF do Local da Prestação | Código (10) | Estado/UF |
| Cód. País do Local da Prestação | Código (10) | País |

### Linha do Documento Fiscal

| Campo | Tipo | Descrição |
| --- | --- | --- |
| % Tributos Federais Estimados (IBPT) | Decimal | Alíquota federal estimada por linha |
| % Tributos Estaduais Estimados (IBPT) | Decimal | Alíquota estadual estimada por linha |
| % Tributos Municipais Estimados (IBPT) | Decimal | Alíquota municipal estimada por linha |
