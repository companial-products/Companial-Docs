# Atualizações Junho/2026



## Localization — 25.5.5.184 <small>26/06/2026</small>

### Correções

- **MD-e — Ciência automática:** Corrigido problema em que a execução automática da ação de Ciência deixava eventos de Manifestação de Destinatário NF-e com situação pendente. A correção garante que todos os registros sem manifestação sejam marcados corretamente antes do envio e que lotes pendentes de execuções anteriores (interrompidas por limite de lote, erro de transmissão ou chaves de acesso não devolvidas pela SEFAZ) também sejam transmitidos na mesma execução, eliminando acúmulo de eventos em aberto.

- **DUIMP — Rateio do Valor do Siscomex:** Corrigido o cálculo de rateio da Taxa de Utilização do Siscomex entre as linhas da DI. O valor passa a ser distribuído proporcionalmente conforme o método de atribuição de encargo configurado (por valor de linha, peso líquido/bruto ou quantidade), com arredondamento adequado e ajuste do resíduo na última linha para que o total rateado corresponda exatamente ao valor cobrado.

## Tax Books — 25.5.2.158 <small>26/06/2026</small>

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário, incluindo: migração de campos de seleção para o formato de enumeração tipada em todos os módulos do app (ISS, DCTF, ECF, Apuração IRPJ/CSLL, ECD, EFD-ICMS/IPI, EFD-Contribuições, EFD-Reinf, PIS/COFINS e Apuração de Impostos).

## Localization — 25.5.5.127 <small>16/06/2026</small>

### Novos Recursos

- **NFS-e — DANFSe v2.0:** Impressão da NFS-e atualizada para o novo layout nacional (NT SE/CGNFS-e nº 008), passando a exibir os campos de tributação IBS/CBS (alíquotas, base de cálculo, valores apurados por esfera estadual e municipal), situação e finalidade da NFS-e, seções de destinatário e intermediário da operação, código NBS, local de prestação, e marca d'água de cancelamento. A seção de tributação federal passa a ser identificada como "TRIBUTAÇÃO FEDERAL (EXCETO CBS)" e o tomador do serviço como "TOMADOR / ADQUIRENTE". Adicionada também a ação **Atualizar DANFSe** no Documento Fiscal Registrado, permitindo forçar a regeneração do arquivo armazenado.

- **NFS-e — Documentos com valor total zero (Lei Municipal SP nº 14.097/2005):** Adicionado suporte ao registro e emissão de NFS-e com valor total zero, conforme exigência da legislação municipal de São Paulo. O novo campo **Valor Fiscal Zero** pode ser ativado no Tipo de Operação para zeragem automática de quantidade e valores fiscais nas linhas do documento, ou preenchido manualmente na entrada direta do documento fiscal. O cálculo de impostos passa a reconhecer e validar adequadamente linhas com esse indicador.

- **Declaração de Importação — Evento de extensão ao criar documento:** Adicionado evento de integração que permite alterar o tipo de documento gerado (por exemplo, de Fatura para Ordem de Compra) ao criar um documento de compra a partir de uma Declaração de Importação. A criação de Ordens de Compra passou a ser suportada nesse fluxo.

- **NF-e — Geração do grupo de rastreabilidade estendida a novos segmentos:** A geração do grupo rastro (lote/série) na NF-e, antes restrita a medicamentos, foi estendida a defensivos agrícolas, produtos veterinários, bebidas, água mineral envasada e embalagens, conforme a NT 2016.002. Adicionado também evento de integração para que parceiros possam customizar essa lógica por categoria de item.

### Melhorias

- **NF-e/NFS-e — Suporte a CNPJ alfanumérico na validação de chave de acesso (NT 2026.004):** A validação do dígito verificador das chaves de acesso de NF-e (44 dígitos) e NFS-e (50 dígitos) foi atualizada para aceitar letras nas posições correspondentes ao CNPJ do emitente, conforme a NT 2026.004. As chaves informadas manualmente são normalizadas para maiúsculas automaticamente.

- **NF-e — Tratamento do status 120 "Autorizado com Alerta" (NT 2026.002):** O status 120 retornado pela SEFAZ passa a ser tratado como autorizado em todos os fluxos relevantes — verificação de autorização, cancelamento, MDF-e e consulta — eliminando bloqueios indevidos em documentos com esse retorno.

- **CEP — País em branco tratado como doméstico:** Na validação de CEP em cadastros de endereço, um campo País não preenchido passa a ser interpretado como Brasil (comportamento doméstico), evitando que a validação seja ignorada indevidamente em registros sem país informado.

- **NFS-e — Ação "Abrir DANFSe" renomeada no Documento Fiscal Registrado:** A ação de visualização da NFS-e foi renomeada de "Open Image" para "Open DANFSe", tornando o rótulo mais claro e consistente com a terminologia do documento oficial.

- **Filiais e Municípios — Logotipo migrado para formato Media:** As imagens de logotipo cadastradas em Informação Filial e em Municípios foram migradas do formato legado Blob/Bitmap para o tipo Media, compatível com os requisitos atuais da plataforma. A migração ocorre automaticamente na atualização. A exibição da imagem passa a ser feita por meio de um painel dedicado na página.

### Correções

- **NFS-e — Cancelamento via chave de acesso (sem código de verificação):** Corrigida falha que impedia o cancelamento de NFS-e quando o documento possuía chave de acesso mas não tinha código de verificação. O botão **Cancelar NFS-e** agora é habilitado corretamente nessa situação.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.
## Tax Books — 25.5.2.127 <small>16/06/2026</small>

### Correções

- Corrigida a geração do registro I157 da ECD (Transferência de Saldos de Plano de Contas Anterior): contas com saldo zero agora são suprimidas do registro somente quando a opção **Suprimir Contas Inativas** está habilitada na configuração da ECD, evitando que contas com saldo zero, mas ainda ativas, fossem incorretamente omitidas do arquivo.

## Localization — 25.5.5.103 <small>08/06/2026</small>

### Melhorias

- Adicionada a leitura dos valores retidos de CSLL (CSL Ret.), IRRF e INSS (INSS Ret.) no cabeçalho do Documento Fiscal durante a importação de NF-e. Esses tributos retidos agora são corretamente capturados do XML e somados às entradas de imposto do documento, passando a compor o cabeçalho da NF-e de forma equivalente ao COFINS Ret. e demais retenções já tratadas anteriormente.

## Tax Books — 25.5.2.102 <small>08/06/2026</small>

### Correções

- Corrigida a validação do campo CEP nos **Participantes EFD**: ao informar um participante estrangeiro, o sistema não tentará mais buscar o código de município IBGE a partir do CEP, evitando erros de validação para endereços de países fora do Brasil.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário, incluindo remoção de objetos e elementos de interface marcados como obsoletos em versões anteriores.

## Localization — 25.5.5.97 <small>03/06/2026</small>

### Novos Recursos

- **NF-e – Defensivos Agrícolas (Grupo ZF):** Adicionado suporte ao cadastro de receituários agronômicos no Documento Fiscal. É possível informar o número do receituário e o CPF do responsável técnico diretamente nos documentos de vendas, compras e no Documento Fiscal. As tags `nReceituario` e `CPFRespTec` são geradas automaticamente no XML da NF-e durante a transmissão, atendendo à exigência do Grupo ZF para operações com defensivos agrícolas.

### Melhorias

- **Cópia de Documentos:** Ao copiar um documento de vendas ou compras com recálculo de linhas, o vínculo com o Documento Fiscal de origem é agora preservado nas linhas copiadas, evitando perda da referência fiscal.

- **MD-e – Desarquivar em lote:** Na lista de Arquivo de Manifestações de Destinatário NF-e, passou a ser possível selecionar múltiplos documentos e desarquivá-los de uma só vez, com confirmação prévia. Anteriormente, o desarquivamento era feito registro a registro.

- **License Monitor:** A verificação de licença do License Monitor foi adicionada ao fluxo de cálculo de impostos, transmissão de NF-e e transmissão de NFS-e, garantindo que o monitoramento seja acionado nessas operações críticas.

- **CEP – Endereços Estrangeiros:** A validação de CEP passou a ignorar endereços cujo país não seja o Brasil, evitando erros ao preencher dados de clientes ou fornecedores estrangeiros que não possuem CEP brasileiro.

- **IBS/CBS – Município de incidência em operações com exterior:** O cálculo do município de incidência (cMunIncid) para IBS/CBS em documentos com clientes ou fornecedores estrangeiros passou a utilizar o município da filial da empresa emissora, que é a origem ou destino efetivo da operação, em vez de tentar buscar um CEP estrangeiro inexistente na base brasileira.

### Outros

- Diversas melhorias internas de qualidade de código sem impacto funcional visível ao usuário.

## Changelog

| Hash | Data | Descrição | Aplicativo | Versão |
|---|---|---|---|---|
| 393aac6 | 22/06 | MD-e: correção da ciência automática que deixava eventos de manifestação pendentes | Localization | 25.5.5.184 |
| 5ce24a3 | 02/06 | DUIMP: Ajuste no Rateio do Valor Siscomex | Localization | 25.5.5.184 |
| 6ac7b9d | 25/06 | ISS: migração de campos Option para Enum (CM0029) | Tax Books | 25.5.2.158 |
| 9eaa1b4 | 25/06 | DCTF: migração de campos Option para Enum (CM0029) | Tax Books | 25.5.2.158 |
| 2ccf867 | 25/06 | ECF e Apuração IRPJ/CSLL: migração de campos Option para Enum (CM0029) | Tax Books | 25.5.2.158 |
| 4aa806c | 25/06 | ECD: migração de campos Option para Enum (CM0029) | Tax Books | 25.5.2.158 |
| 56b4b16 | 25/06 | EFD-ICMS/IPI e EFD-Contribuições: migração de campos Option para Enum (CM0029) | Tax Books | 25.5.2.158 |
| 84f2e15 | 25/06 | EFD-Reinf: migração de campos Option para Enum (CM0029) | Tax Books | 25.5.2.158 |
| f547ecf | 25/06 | Relatórios Fiscais: reuso do enum FiscalDocumentStatus em vez de Option (CM0029) | Tax Books | 25.5.2.158 |
| 756a226 | 25/06 | PIS/COFINS: migração de campos Option para Enum (CM0029) | Tax Books | 25.5.2.158 |
| 0b0cfc5 | 25/06 | Apuração: enums compartilhados e migração de campos Option para Enum (CM0029) | Tax Books | 25.5.2.158 |
| 254a68f | 25/06 | Tax Books: conformidade Companial (CM0027/CM0028) e migração de SettlementAmountType para enum | Tax Books | 25.5.2.158 |
| 61cbe99 | 24/06 | Conformidade com analisadores CodeCop/AppSourceCop (validação MS/Companial) (#87) | Tax Books | 25.5.2.158 |
| d3838d0 | 15/06 | Declaração de Importação: criação de evento OnBeforeCreateDocumentAfterSetDocumentType para permitir alterar o tipo de documento para Ordem de Compra | Localization | 25.5.5.127 |
| 3fc52cd | 12/06 | Correção de avisos na build | Localization | 25.5.5.127 |
| c47a56e | 12/06 | NFS-e: zeramento da Quantidade em documento fiscal com valor total zero (Lei Municipal SP nº 14.097/2005) | Localization | 25.5.5.127 |
| 0927ae3 | 12/06 | NFS-e: registro de documento com valor total zero (Lei Municipal SP nº 14.097/2005) (#82) | Localization | 25.5.5.127 |
| d67e987 | 11/06 | DANFSe v2.0: impressão da NFS-e conforme o novo layout nacional (NT SE/CGNFS-e nº 008) (#81) | Localization | 25.5.5.127 |
| c26c939 | 11/06 | NF-e: geração do grupo rastro estendida a defensivos agrícolas e demais segmentos da NT 2016.002 | Localization | 25.5.5.127 |
| af558a9 | 11/06 | NF-e: tratamento do status 120, de Autorizado com alerta (NT 2026.002) | Localization | 25.5.5.127 |
| 63608ea | 10/06 | CEP: país em branco tratado como doméstico na validação de CEP | Localization | 25.5.5.127 |
| 565a0b8 | 10/06 | NF-e/NFS-e: suporte a CNPJ alfanumérico na validação de chave de acesso (NT 2026.004) | Localization | 25.5.5.127 |
| 92865a2 | 08/06 | NFS-e: correção de cancelamento via chave de acesso (sem código de verificação) | Localization | 25.5.5.127 |
| 846a98e | 08/06 | NFS-e: botão Cancelar NFS-e não estava habilitado quando há Chave de Acesso e não há Cód. Verificação | Localization | 25.5.5.127 |
| c9fb280 | 08/06 | Filiais/Municípios: logo migrado de Blob/Bitmap para Media (AW0009) | Localization | 25.5.5.127 |
| 25d8cab | 12/06 | ECD: registro I157 passa a respeitar a flag "Suprimir Contas Inativas" para contas com saldo zero na transferência de plano de contas | Tax Books | 25.5.2.127 |
| da85309 | 08/06 | NF-e: leitura de CSLL, IRRF e INSS retidos no cabeçalho | Localization | 25.5.5.103 |
| aedda0f | 26/05 | CEP: nova forma de validação para endereços estrangeiros | Tax Books | 25.5.2.102 |
| 5beb601 | 02/06 | MD-e: permitir desarquivar múltiplos documentos de uma vez na lista de arquivados (#72) | Localization | 25.5.5.97 |
| 9979011 | 02/06 | Cópia de Documentos: vínculo ao Documento Fiscal de origem mantido ao copiar documento com recálculo de linhas | Localization | 25.5.5.97 |
| 3df73d1 | 28/05 | PermissionSet LicenseMonitor adicionado ao grupo D365 BASIC | Localization | 25.5.5.97 |
| 5c6cfd6 | 28/05 | Novos caminhos de checagem adicionados ao License Monitor (Cálculo de Impostos, Transmissão de NF-e e NFS-e) | Localization | 25.5.5.97 |
| 15784d4 | 26/05 | NF-e: inclusão das tags nReceituario e CPFRespTec (Grupo ZF - Defensivos Agrícolas) (#70) | Localization | 25.5.5.97 |
| aedda0f | 26/05 | CEP: nova forma de validação para endereços estrangeiros | Localization | 25.5.5.97 |
