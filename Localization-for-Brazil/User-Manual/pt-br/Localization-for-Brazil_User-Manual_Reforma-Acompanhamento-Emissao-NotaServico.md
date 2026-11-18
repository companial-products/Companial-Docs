# Reforma Tributária - Suporte à Emissão de Notas de Serviço

> Atualizado em: 2026-11-11
>
> **Legenda:**
> - ✅ Direto — implementado diretamente para esta cidade
> - ✅ Herdado — implementado pelo provedor e aproveitado automaticamente
> - ⚠️ Parcial — implementação incompleta ou condicional (ver nota)
> - ⚠️ Inativo — implementado mas ainda não enviado ao webservice
> - ❌ Não — não implementado
>
> **O que cada coluna representa:**
> - **IBS/CBS**: Preenchimento dos tributos da reforma (Imposto sobre Bens e Serviços e Contribuição sobre Bens e Serviços) com classificação tributária e alíquotas
> - **PIS/COFINS + CSLL**: Inclusão dos valores de retenção federal de PIS, COFINS e CSLL no documento fiscal, conforme as regras da reforma
> - **Grupo Imóvel**: Preenchimento dos dados de imóvel (cadastro imobiliário, inscrição fiscal, endereço) quando a operação envolver imóveis
> - **Grupo Evento**: Preenchimento dos dados do evento (nome, período e identificador ou endereço do local) quando o local de fornecimento da operação é um evento
> - **Destinatário**: Preenchimento do destinatário da operação (bloco `IBSCBS/dest`) quando o documento traz endereço de destinatário próprio (ship-to), com CNPJ/CPF, nome e endereço; sem CNPJ/CPF próprio, repete o do tomador
> - **Tipo Operação**: Indicação do tipo de operação fiscal na nota (`tpOper`); implementação parcial significa que só a regra provisória do Erro E0903 está coberta — `tpOper` = 5 nos itens 10.05, 15.09, 17.12 e 25.05 da LC 116 —, sem a compra governamental (art. 10 §2º da LC 214/2025) nem os valores 1 a 4
> - **Lotes com resposta mista**: Capacidade de `ConsultarLoteRps` reportar corretamente o status individual de cada RPS quando o lote retorna resultados mistos (sucesso e erro/alerta simultâneos), sem duplicar ou perder informação quando o mesmo RPS aparece mais de uma vez na resposta do webservice

| Cidade | Provedor | IBS/CBS | PIS/COFINS + CSLL | Grupo Imóvel | Grupo Evento | Destinatário | Tipo Operação | Lotes com resposta mista |
|---|---|---|---|---|---|---|---|---|
| Nacional (DPS) | Nacional | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ⚠️ Parcial ¹⁷ | ❌ Não |
| Qualquer cidade com provedor BethaCloud | BethaCloud | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ⚠️ Parcial ¹⁷ | ❌ Não |
| Eusébio/CE | EL | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ⚠️ Parcial ¹⁷ | ⚠️ Parcial ¹⁹ |
| Contagem/MG | Governança Brasil | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ⚠️ Parcial ¹⁷ | ❌ Não |
| Saquarema/RJ | Governança Brasil | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ⚠️ Parcial ¹⁷ | ❌ Não |
| São João da Barra/RJ | ISSWeb | ✅ Herdado | ✅ Herdado ¹¹ | ✅ Herdado | ✅ Herdado | ✅ Herdado | ⚠️ Parcial ¹⁷ | ❌ Não |
| Itajaí/SC | | ⚠️ Parcial ⁶ | ⚠️ Parcial ¹ | ❌ Não | ❌ Não | ❌ Não | ❌ Não | ❌ Não |
| Ipojuca/PE | Tinus | ✅ Herdado | ✅ Herdado | ✅ Herdado | ❌ Não | ⚠️ Parcial ²⁰ | ⚠️ Parcial ¹⁷ | ❌ Não |
| Olinda/PE | Tinus | ✅ Herdado | ✅ Herdado | ✅ Herdado | ❌ Não | ⚠️ Parcial ²⁰ | ⚠️ Parcial ¹⁷ | ❌ Não |
| São Gonçalo do Amarante/RN | Tinus | ✅ Herdado | ✅ Herdado | ✅ Herdado | ❌ Não | ⚠️ Parcial ²⁰ | ⚠️ Parcial ¹⁷ | ❌ Não |
| Aracaju/SE | WebISS ²² | ✅ Herdado ²² | ✅ Herdado ²² | ✅ Herdado | ❌ Não ²² | ✅ Herdado | ⚠️ Parcial ²² | ❌ Não |
| São Paulo (Reforma) | | ✅ Direto ⁷ | ✅ Direto ² | ✅ Direto | ⚠️ Parcial ¹⁵ | ✅ Direto | ⚠️ Parcial ¹⁸ | ✅ Direto ¹² |
| PlugNotas | | ✅ Direto | ❌ Não ³ | ✅ Direto | ❌ Não | ✅ Direto | ❌ Não | ❌ Não |
| Rio Grande/RS | Sigcorp | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ✅ Herdado | ⚠️ Parcial ¹⁷ | ❌ Não |
| Fortaleza/CE | | ✅ Direto | ❌ Não ⁵ | ❌ Não | ❌ Não | ❌ Não | ❌ Não | ❌ Não |
| Salvador/BA | | ⚠️ Parcial ⁸ | ❌ Não ⁵ | ❌ Não | ❌ Não | ❌ Não | ❌ Não | ❌ Não |
| Embu das Artes/SP | GIAP/Obaratec | ❌ Não | ❌ Não | ❌ Não | ❌ Não | ❌ Não | ❌ Não | ❌ Não |
| Barueri/SP | | ✅ Direto ⁹ | ❌ Não ⁵ | ❌ Não | ❌ Não | ❌ Não | ❌ Não | ❌ Não |
| Diadema/SP | GISS | ✅ Herdado | ✅ Herdado ¹³ | ❌ Não | ❌ Não | ❌ Não | ❌ Não | ❌ Não |
| Jundiaí/SP | GISS | ✅ Herdado | ✅ Herdado ¹³ | ❌ Não | ❌ Não | ❌ Não | ❌ Não | ❌ Não |
| Santos/SP | GISS | ✅ Herdado | ⚠️ Parcial ¹⁴ | ❌ Não | ❌ Não | ❌ Não | ❌ Não | ❌ Não |
| São Bernardo do Campo/SP | GINFES ²³ | ✅ Herdado ²³ | ✅ Herdado ²³ | ❌ Não | ❌ Não | ❌ Não | ❌ Não | ❌ Não |
| Barcarena/PA (Nacional) | ISSNet Nacional | ✅ Herdado | ⚠️ Parcial ¹⁶ | ✅ Herdado | ✅ Herdado | ✅ Herdado | ⚠️ Parcial ¹⁷ | ❌ Não |
| Brasília/DF (Nacional) | ISSNet Nacional | ✅ Herdado | ⚠️ Parcial ¹⁶ | ✅ Herdado | ✅ Herdado | ✅ Herdado | ⚠️ Parcial ¹⁷ | ❌ Não |
| Goiânia/GO (Nacional) | ISSNet Nacional | ✅ Herdado | ⚠️ Parcial ¹⁶ | ✅ Herdado | ✅ Herdado | ✅ Herdado | ⚠️ Parcial ¹⁷ | ❌ Não |
| Paranaguá/PR | Elotech | ✅ Herdado | ⚠️ Parcial ²⁶ | ✅ Herdado ²⁷ | ❌ Não | ❌ Não | ❌ Não | ❌ Não |
| São Francisco do Sul/SC | Atende.Net | ✅ Herdado | ✅ Herdado ²⁴ | ✅ Herdado | ❌ Não | ❌ Não | ❌ Não | ❌ Não ²⁵ |

**Notas:**

¹ **Itajaí/SC — PIS/COFINS + CSLL** — usa formato próprio do município. A retenção segue a lógica do padrão Nacional: CSLL, PIS e COFINS retidos são somados no campo de CSLL, e o tipo de retenção vem da tabela completa. O grupo de PIS/COFINS, porém, só é montado quando há alíquota de PIS ou COFINS retido e leva a base e as alíquotas da retenção; o PIS/COFINS de apuração própria (faturamento), que o padrão Nacional envia nesse grupo, não é enviado.

² **São Paulo (Reforma)** — usa formato próprio do município de São Paulo: os valores de PIS, COFINS e CSLL são mapeados individualmente e também consolidados no campo de retenção, com mesma lógica de combinação de Itajaí/SC.

³ **PlugNotas** — PIS, COFINS e CSLL são enviados em campos separados conforme o formato do intermediador, sem o campo consolidado de retenção exigido pelo padrão DPS Nacional.

⁵ **Fortaleza/CE, Salvador/BA, Barueri/SP** — possuem o valor de CSLL como campo isolado no padrão do município, sem integração com PIS/COFINS.

⁶ **Itajaí/SC — IBS/CBS** — usa formato próprio do município (grupo `IBSCBS` do layout 7.4), e não o formato DPS Nacional. Envia a localidade de incidência e as alíquotas de IBS estadual, IBS municipal e CBS, cada uma com seu percentual de redução quando houver. Não envia a base de cálculo, as alíquotas efetivas nem o grupo de totalizadores (`totCIBS`), onde o layout traz os valores de IBS/CBS, o diferimento e a tributação regular. O layout de Itajaí não tem CST nem classificação tributária nesse grupo. O grupo só é enviado quando o documento traz classificação tributária.

⁷ **São Paulo (Reforma) — IBS/CBS** — usa formato próprio do município de São Paulo com suporte a imóvel/obra e tipo de operação; não usa o formato DPS Nacional.

⁸ **Salvador/BA — IBS/CBS** — o layout de Salvador traz, no grupo do serviço, apenas a classificação tributária (`cClassTrib`), o indicador de operação e o código NBS, e os três são enviados; o indicador de operação vai como 050201 quando o NBS é 1.0605.40.00. Não há CST nem alíquotas de IBS/CBS, e o bloco completo de IBS/CBS está pendente de implementação.

⁹ **Barueri/SP — IBS/CBS** — o padrão de Barueri (layout PMB004) é um arquivo posicional, enviado à prefeitura em Base64; não há XML da nota. Os dados de IBS/CBS vão no registro 4 desse arquivo, gerado sempre que o documento traz classificação tributária: indicador de operação, classificação tributária e CST, com uso/consumo pessoal e indicador de destinatário fixos em 0. O registro 5 (crédito presumido, ente governamental e destinatário) não é gerado.

¹¹ **São João da Barra/RJ (ISSWeb)** — PIS/COFINS herdado do padrão Nacional. Um ajuste provisório que forçava um tipo de retenção específico foi desativado para conformar ao padrão nacional; será reativado se o servidor Fiorilli exigir.

¹³ **Diadema/SP e Jundiaí/SP (GISS) — PIS/COFINS + CSLL** — no grupo `trib`/`piscofins` os campos de valor `vPis`/`vCofins` enviam o valor **retido** (`ValorPis`/`ValorCofins`); quando o valor retido não é informado (zero), envia-se o valor **faturado** (`ValorPisFat`/`ValorCofinsFat`) nesses campos. O indicador `tpRetPisCofins` permanece calculado apenas sobre os valores retidos, refletindo `Item0` (sem retenção) quando existe somente valor faturado. A base de cálculo (`vBCPisCofins`) e as alíquotas seguem a mesma escolha entre retido e faturado, e o grupo `trib`/`piscofins` também é montado quando a nota tem apenas PIS/COFINS de apuração própria.

¹⁴ **Santos/SP (GISS) — PIS/COFINS + CSLL** — herda o comportamento padrão do provedor GISS (ver nota ¹³), **exceto** quando há, na mesma nota, PIS/COFINS **retido** e PIS/COFINS de **apuração própria** (faturado). Nesse cenário específico (`NFSeGISSSPSantos.CalcularValoresRetencao`): indica-se a retenção **apenas do CSLL** (`tpRetPisCofins = Item8`); o PIS e COFINS **retidos** são **somados ao valor do CSLL** (`vCsll = ValorCsll + ValorPis + ValorCofins`); o PIS e COFINS de **apuração própria** (`ValorPisFat`/`ValorCofinsFat`) são enviados em `vPis`/`vCofins`; e os valores **retidos originais** de PIS, COFINS e CSLL são descritos no campo de **discriminação dos serviços** para preservar a decomposição embutida no CSLL.

¹⁵ **São Paulo (Reforma) — Grupo Evento** — o grupo `atvEvento` fica em `tpRPS` (e não dentro de `serv`, como no padrão Nacional) e é preenchido com nome e período do evento mais o endereço do local. Parcial porque o layout de São Paulo **não possui o identificador de evento** (`idAtvEvt`): o schema oferece apenas o sub-elemento `end`, então `EventoId` é usado somente como nome alternativo quando `EventoNome` não vem preenchido. As datas, tipadas como `xs:date` obrigatório (`dtIniEvt`/`dtFimEvt`) em vez de texto, são convertidas a partir do ISO recebido; se qualquer uma faltar ou não converter, o grupo inteiro é omitido para não gerar `0001-01-01` e invalidar o RPS.

¹² **São Paulo (Reforma) — Lotes com resposta mista** — única implementação que localiza um `RespostaRps` já existente pelo `NumeroRps` antes de adicionar um novo (mescla o erro/alerta ao campo `Erros` do item existente em vez de duplicar), evitando entradas duplicadas quando o mesmo RPS aparece mais de uma vez na resposta de `ConsultarLoteRps` (ex.: erro + alerta). Marca `resposta.PartialSuccessBatch = true` e calcula `Sucesso`/`Erro` agregados de forma que ambos fiquem `false` quando o lote tiver resultado misto (nem todos sucesso, nem todos erro). Nenhuma outra cidade/provedor mescla as respostas por `NumeroRps`: a maioria não suporta consulta de lote (lança exceção) ou constrói `ListaRps` sempre adicionando um novo item por nó, sem verificar duplicidade por `NumeroRps`. O provedor EL (Eusébio/CE) também marca `PartialSuccessBatch`, mas com um único RPS por lote (ver nota ¹⁹).

¹⁶ **Barcarena/PA, Brasília/DF e Goiânia/GO (ISSNet Nacional) — PIS/COFINS + CSLL** — a retenção federal (`vRetCSLL`, `tpRetPisCofins`) segue o padrão Nacional; os campos de PIS/COFINS de apuração própria (`vBCPisCofins`, `pAliqPis`, `pAliqCofins`, `vPis`, `vCofins`) são enviados ausentes, provisoriamente, até o provedor confirmar a validação desses campos.

¹⁷ **Padrão Nacional, provedores que herdam dele e Tinus — Tipo Operação** — regra provisória do Erro E0903: `tpOper` = 5 (fornecimento e recebimento do pagamento concomitantes) quando os 4 primeiros dígitos do código de tributação nacional (`cTribNac`), que correspondem ao item da LC 116, são `1005` (intermediação), `1509` (arrendamento), `1712` (administração) ou `2505` (cessão onerosa de imóvel). A lista é única para todos os layouts (`AbstractNFSE.ItensLC116TipoOperacao`). A compra governamental (art. 10 §2º da LC 214/2025), os valores 1 a 4 e o tipo de ente governamental (`tpEnteGov`) não são tratados.

¹⁸ **São Paulo (Reforma) — Tipo Operação** — mesma regra provisória e mesma lista de itens da nota ¹⁷, aplicada ao item da LC 116 do serviço (`NFSESPRef.ResolverItemLC116`): o Código do Item de Serviço (`ItemListaServico`, com ou sem ponto) quando preenchido — e, se ele não for reconhecido, nem `tpOper` nem o grupo de obra são enviados —; em branco, uma correlação embutida código de serviço municipal → item da LC 116 (Anexo 1 da IN SF/SUREM 08/2011, atualizado até a IN 06/2024), restrita aos códigos de obra e dos quatro itens do tipo de operação. O item segue a numeração da LC 116, e não a da Lei municipal 13.701/03, que a tabela da prefeitura usa: nos grupos 7 e 17 elas diferem (administração de imóveis é 17.12 na LC 116 e 17.11 na tabela municipal; leilão é 17.13 na LC 116 e fica fora da regra). Ficam fora da correlação os códigos de autônomo ligados a vários itens ao mesmo tempo (01503, 02232, 03980) e os de "Habite-se" (01024, 01032, 01059); para eles, e para códigos criados depois da IN 06/2024, decide o Código do Item de Serviço. O mesmo item decide o grupo de obra (`imovelobra`).

¹⁹ **Eusébio/CE (EL) — Lotes com resposta mista** — o lote tem sempre um único RPS (o envio recusa lotes com mais de um), e a consulta é feita pelo identificador dessa DPS (`idDPS`), então não há resultado misto possível. Quando a DPS está autorizada, `ConsultarLoteRps` marca `PartialSuccessBatch = true` e devolve o RPS individualmente (número e série do RPS, número e chave de acesso da NFS-e), sem `Sucesso` no lote, para o ERP gravar a chave de acesso localizando o documento pelo número do RPS. A rejeição volta apenas no nível do lote (erro e mensagens, sem item em `ListaRps`), e não há a mescla por `NumeroRps` da nota ¹², que é desnecessária com um RPS por lote.

²⁰ **Ipojuca/PE, Olinda/PE e São Gonçalo do Amarante/RN (Tinus) — Destinatário** — o provedor Tinus monta o bloco IBS/CBS por conta própria e envia o destinatário (`IBSCBS/dest`) só quando o endereço de entrega está no papel de destinatário **e** o destinatário tem CNPJ/CPF próprio. Sem CNPJ/CPF próprio o grupo não é enviado, ao contrário do padrão Nacional, que nesse caso repete o documento e o nome do tomador.

²² **Aracaju/SE (WebISS)** — provedor próprio (`NFSEWebISS`), ABRASF 2.02 com o grupo `IBSCBS` do manual de integração da WebISS, independente do Tinus. **IBS/CBS**: o grupo leva a finalidade (NFS-e regular), o uso ou consumo pessoal, o indicador de operação, o destinatário, o imóvel, o CST e a classificação tributária e, quando cadastrados, a tributação regular e o diferimento. O layout **não tem alíquotas nem valores de IBS/CBS**: a prefeitura os calcula e os devolve na NFS-e, com a chave de acesso da nota nacional. O envio é assíncrono: `RecepcionarLoteRps` devolve o protocolo, e as notas ou os erros vêm em `ConsultarLoteRps`; a consulta do último RPS não existe no WebISS. **PIS/COFINS + CSLL** no padrão Nacional, adaptado aos campos ABRASF: `ValorPis`/`ValorCofins` levam a apuração própria (faturamento), `ValorCsll` soma CSLL, PIS e COFINS retidos, e o grupo `InfoPisCofins` sai na mesma condição do Nacional, com CST 01 a 09 (os demais como 00) e `tpRetPisCofins` pela tabela completa. **Grupo Evento**: o layout não tem grupo de evento. **Tipo Operação** e obra: mesma regra provisória e lista da nota ¹⁷, mas decididas pelo item da lista de serviço (`ItemListaServico`), porque o código de tributação do município não é o `cTribNac`; nos itens de obra (07.02, 07.04 a 07.08, 07.17, 07.19) vai o endereço da obra e nunca o grupo imóvel. Envia também a inscrição estadual do prestador e do tomador quando informadas.

²³ **São Bernardo do Campo/SP (GINFES)** — layout GINFES de 2026, independente da integração de Fortaleza. O envio é assíncrono: a transmissão devolve o protocolo, e a consulta primeiro verifica a situação do lote e só busca as notas ou os erros quando o lote já foi processado; enquanto não foi, o lote continua enviado. A natureza da operação aceita só os valores de 1 a 6, e fora deles o lote é recusado antes do envio. **IBS/CBS**: o grupo vai em toda nota, com a finalidade (NFS-e regular), o uso ou consumo pessoal e o destinatário sempre `0` (o layout não tem grupo de destinatário), o indicador de operação, o CST e a classificação tributária da primeira linha, o município de incidência igual ao da filial e o percentual de redução da CBS; tipo de operação, notas referenciadas, ente governamental e base de cálculo não são enviados. Os percentuais aproximados de tributos federal, estadual e municipal (IBPT) vão em toda nota. Os totais de IBS/CBS que a GINFES devolve não são gravados. **PIS/COFINS + CSLL**: o grupo de PIS/COFINS segue o comportamento da nota ¹³ e só é enviado quando o documento tem CST de PIS/COFINS (01 a 09; os demais como 00); CSLL e as demais retenções vão nos campos próprios dos valores da nota.

²⁴ **São Francisco do Sul/SC (Atende.Net) — PIS/COFINS + CSLL** — o layout da NT 122/2025 separa os dois casos: o PIS/COFINS de **apuração própria** (CST, base e alíquotas) vai no grupo `pis_cofins`, que reduz a base de cálculo do IBS/CBS, e os valores **retidos** vão em `valor_pis`/`valor_cofins`, que não reduzem essa base mas reduzem o valor líquido da nota. IR, INSS e contribuição social (CSLL) seguem em `valor_ir`, `valor_inss` e `valor_contribuicao_social`.

²⁵ **São Francisco do Sul/SC (Atende.Net) — Lotes com resposta mista** — a prefeitura exige uma nota por arquivo e não oferece consulta de lote: cada RPS do lote é transmitido em uma requisição própria. Se uma nota for rejeitada, o lote inteiro volta como erro (o ERP não grava nada) e a mensagem lista os RPS que a prefeitura já emitiu; como cada RPS leva o nº do documento em `<identificador>`, essas notas são devolvidas, sem nova emissão, na próxima transmissão. A configuração recomendada é um documento por lote.

²⁶ **Paranaguá/PR (Elotech) — PIS/COFINS + CSLL** — o layout ABRASF 2.03 da Elotech tem um único conjunto de campos por tributo, com o indicador de retenção. Quando há PIS/COFINS **retido**, ele é o informado; sem retenção, informa-se o de **apuração própria**. Parcial porque um documento com retenção e apuração própria do mesmo tributo envia só a retenção. IR, INSS e CSLL retidos vão nos respectivos campos.

²⁷ **Paranaguá/PR (Elotech) — Grupo Imóvel** — no web service da Elotech o endereço do imóvel é um texto simples, e não o grupo estruturado do padrão nacional; por isso logradouro, número, complemento e bairro seguem concatenados. Inscrição imobiliária e CIB vão nos campos próprios.
