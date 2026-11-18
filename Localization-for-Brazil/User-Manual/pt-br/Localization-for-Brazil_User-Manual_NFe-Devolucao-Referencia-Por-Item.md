<!-- PR: #124, #280, #325 -->
# Referenciamento por Item em Devoluções de NF-e

**Data**: Agosto/2026

---

## Sumário

A NF-e de devolução pode referenciar a nota de origem **item a item** (grupo `DFeReferenciado`) ou no cabeçalho (grupo `NFref`). A NT 2025.002, na versão 1.53, define as regras que valem em produção a partir de **16/11/2026**: a devolução referencia a nota de origem por **uma única chave de NF-e no cabeçalho** ou em **todas as linhas de item**, e a nota complementar, a devolução e as notas de crédito dos tipos 01, 03 e 04 aceitam **um só documento referenciado no cabeçalho**. O sistema valida essas regras no registro da devolução, de venda e de compra, de acordo com a **Data Registro** do documento, e antecipa as rejeições da SEFAZ (255, 321 e 1010) como erros de validação. O **Número Item Ref.** é opcional em qualquer data. O sistema também confere os emitentes das NF-e referenciadas nas linhas pela raiz do CNPJ (rejeições 1193 e 1194) e gera no XML cada documento referenciado no cabeçalho uma única vez. A observação automática de devolução nas compras apenas descreve as referências já informadas, sem criar nenhuma referência.

---

## 1. Contexto

Nas devoluções de mercadoria (campo **Finalidade Emissão** (`ComplementaryInvoiceTypeFNX`) = **Devolução**), as regras de validação da SEFAZ (NT 2025.002, versão 1.53) estabelecem que:

- A NF-e de origem é referenciada **em cada linha de item** da devolução (grupo `DFeReferenciado`) **ou** por **uma única chave de NF-e no cabeçalho** (tag `refNFe` do grupo `NFref`). Sem nenhuma das duas, a nota é rejeitada (rejeição 321).
- Referenciar nos **dois níveis ao mesmo tempo**, cabeçalho e item, é rejeitado com a rejeição 1010, em qualquer data.
- No cabeçalho cabe **um só documento referenciado** (regra B25-40, rejeição 255). A regra vale também para a nota complementar e para as notas de crédito dos tipos 01, 03 e 04 (veja [Uma referência no cabeçalho](#uma-referência-no-cabeçalho)).
- Um documento **não eletrônico** no cabeçalho (NF modelo 1/1A, NF de produtor, ECF) não substitui a referência da devolução. Só a chave de NF-e conta.
- O **número do item** dentro da NF-e de origem (tag `nItem`) é **opcional**. A regra que o exigia (VC03-20, rejeição 1048) foi suspensa pela SEFAZ.

A versão 1.53 entra em produção na SEFAZ em qualquer dia até **16/11/2026**. O sistema usa a **Data Registro** do documento como critério:

| Data Registro | Referência aceita na devolução | Referências no cabeçalho (complementar, devolução e crédito 01/03/04) |
|---|---|---|
| Até 15/11/2026 | Todas as linhas de item referenciadas **ou** qualquer referência no cabeçalho, chave de NF-e ou documento não eletrônico | Sem limite |
| A partir de 16/11/2026 | Todas as linhas de item referenciadas **ou uma** chave de NF-e no cabeçalho | **Uma só** |
| Qualquer data | Cabeçalho e itens ao mesmo tempo são barrados (rejeição 1010). **Número Item Ref.** opcional | — |

Até 15/11/2026, o sistema aplica as regras que a SEFAZ aceita tanto na versão 1.52 quanto na 1.53, para não barrar no registro uma nota que ela autoriza.

> **Calendário da NT 2025.002**: as versões 1.50, 1.51 e 1.52 chegaram a fixar a referência **exclusivamente por item** a partir de 01/09/2026, depois 05/10/2026 e depois 03/11/2026, com o número do item obrigatório. A versão 1.51 também **eliminou a exceção por CFOP** (1.201, 1.202, 1.410, 1.411, 5.921 e 6.921), que antes dispensava a referência e continua eliminada. A versão 1.53 voltou a aceitar a chave no cabeçalho como alternativa e tornou o número do item opcional. Quem estava com a versão anterior da localização tinha a referência por item exigida a partir de 03/11/2026. Com esta versão, devoluções registradas até 15/11/2026 voltam a aceitar a referência no cabeçalho.

---

## 2. Como Referenciar por Item

A referência por item é informada nos dados fiscais da NF-e do documento de devolução:

1. Abra o documento de devolução: **Nota Crédito Venda**, **Devolução Venda**, **Nota Crédito Compra** ou **Devolução de Compra**.
2. No grupo **Dados Fiscais**, clique na ação **NF-e**. A página **Dados NF-e** (`SalesFiscalDataNFeFNX` / `PurchaseFiscalDataNFeFNX`) é aberta.
3. Na seção **Linhas**, preencha para cada linha de item:

| Campo | Descrição |
|-------|-----------|
| **Nº Doc. Fiscal Ref.** (`RefFiscalDocNo`) | Documento fiscal de origem registrado no sistema. A pesquisa mostra apenas documentos que já possuem NF-e autorizada; ao selecionar, a chave de acesso é preenchida automaticamente. |
| **Chave Acesso Ref.** (`RefAccessKey`) | Chave de acesso (44 dígitos) da NF-e de origem. Pode ser digitada manualmente quando a nota de origem não está registrada no sistema. |
| **Número Item Ref.** (`RefItemNumber`) | Posição do item (`nItem`) dentro do XML da NF-e de origem. Use a pesquisa para escolher o item diretamente da relação de itens do XML. **Opcional**: pode ficar em branco, por exemplo quando o XML da nota de origem não está disponível, e nesse caso a NF-e é gerada sem o `nItem`. Quando informado, precisa estar entre 1 e 990. |

> **Importante**: nenhuma referência é criada automaticamente no registro. Todas as referências, de cabeçalho ou de item, são informadas manualmente pelo usuário.

---

## 3. Referência no Cabeçalho (Textos Adicionais)

A referência de cabeçalho é informada pela ação **Texto Adicional**, na página **Linha Texto Adicional NF** (`AdditionalInvoiceTextLineFNX`):

- **Nº Documento Referenciado** (`ReferencedDocumentNo`) seleciona um documento fiscal do sistema e preenche a **Chave NF-e Ref.** (`ReferencedNFeKey`) automaticamente. A chave também pode ser digitada. **A partir de 16/11/2026, uma única chave de NF-e no cabeçalho dispensa a referência por item na devolução.**
- Para documento **não eletrônico** (grupo `refNF`), preencha **Nº Doc. Ref.** (`RefDocumentNo`) junto com **Cód. UF Ref.** (`RefTerritoryCode`), **Data Doc. Ref.** (`RefDocDate`), **CNPJ Emitente Ref.** (`RefIssuerCNPJ`) e **Série Doc. Ref.** (`RefDocumentSeries`). Esse caminho continua disponível para as demais finalidades, mas **em devoluções registradas a partir de 16/11/2026 ele não satisfaz a exigência de referência**. Veja a seção 4.

Cada documento referenciado conta uma vez, mesmo que apareça em mais de uma linha de texto adicional, por exemplo uma observação quebrada em duas linhas com a mesma **Chave NF-e Ref.**. No XML da NF-e, em qualquer finalidade, cada chave e cada documento não eletrônico do cabeçalho é gerado uma única vez, e nenhum documento informado fica de fora. O texto de cada linha continua saindo nas informações complementares da nota.

---

## 4. Comportamento no Registro

A validação ocorre ao registrar a devolução, de venda ou de compra, conforme a **Data Registro**.

### Até 15/11/2026

- A devolução é aceita quando **todas** as linhas de item e de ativo fixo faturáveis estão referenciadas, ou quando há referência no cabeçalho: uma ou mais chaves de NF-e, ou um documento não eletrônico com todos os dados preenchidos.
- Sem nenhuma referência, o registro é interrompido:

> *Não foi informada nenhuma NF-e referenciada para Finalidade Emissão: Devolução.*

### A partir de 16/11/2026

- **Referência por item**: se alguma linha está referenciada, todas as linhas de item e de ativo fixo faturáveis precisam estar. Cada linha sem a **Chave Acesso Ref.** é apontada individualmente:

> *Linha %1: a chave de acesso da NF-e referenciada não foi informada. Um documento de Devolução deve referenciar a NF-e original em cada linha de item ou, alternativamente, uma única chave de acesso de NF-e nos textos adicionais do cabeçalho (regra VC02-14, rejeição 321).*

- **Referência no cabeçalho**: sem nenhuma linha referenciada, uma **Chave NF-e Ref.** nos textos adicionais do cabeçalho basta. Mais de um documento referenciado no cabeçalho é barrado pela regra de [uma referência no cabeçalho](#uma-referência-no-cabeçalho).
- Um documento **não eletrônico** no cabeçalho, sem chave de NF-e, não satisfaz a referência:

> *A partir de 16 de novembro de 2026, um documento não eletrônico (modelo 1/1A) referenciado nos textos adicionais do cabeçalho não satisfaz a referência de um documento de Devolução. Informe a chave de acesso da NF-e do documento original nos textos adicionais do cabeçalho ou referencie-a em cada linha de item (regra VC02-14, rejeição 321).*

- Sem nenhuma referência, nem no cabeçalho nem nas linhas:

> *O documento de Devolução não referencia a NF-e original. Informe uma única chave de acesso de NF-e nos textos adicionais do cabeçalho ou a chave de acesso referenciada em cada linha de item (regra VC02-14, rejeição 321).*

Uma devolução que tenha apenas linhas de conta contábil, encargo ou recurso também precisa de uma chave de NF-e no cabeçalho a partir dessa data.

### Em qualquer período

- Referenciar cabeçalho **e** itens ao mesmo tempo gera erro no registro, antecipando a rejeição 1010:

> *O documento referencia NF-e tanto nos textos adicionais do cabeçalho quanto nas linhas de item. A SEFAZ não permite referenciamento em dois níveis (rejeição 1010). Mantenha apenas um deles.*

- O **Número Item Ref.** não é exigido. A linha com a **Chave Acesso Ref.** preenchida e o número do item em branco é registrada.
- Somente linhas de **item e ativo fixo** faturáveis participam da exigência por item. Linhas de conta contábil, encargo ou recurso não são validadas.

### Uma referência no cabeçalho

A partir de **16/11/2026** (regra B25-40, rejeição 255), o documento só pode referenciar **um** documento nos textos adicionais do cabeçalho quando é:

- nota complementar: **Finalidade Emissão** = **Complemento de Quantidade**, **Complemento de Impostos** ou **Complemento de Preço**;
- devolução: **Finalidade Emissão** = **Devolução**;
- nota de crédito: **Finalidade Emissão** = **Nota de Crédito** com **Tipo Nota de Crédito** (`NFeCreditNoteType`) igual a **Multas e Juros** (01), **Retorno por recusa total na entrega ou por não localização do destinatário na tentativa de entrega** (03) ou **Redução de Valores** (04).

Contam juntas as chaves de NF-e e os documentos não eletrônicos, e a mesma referência repetida em mais de uma linha de texto conta como uma só. A conferência ocorre:

- no registro dos documentos de venda e de compra;
- na ação **Liberar** do documento fiscal avulso, nas páginas **Documento Fiscal de Venda** e **Documento Fiscal de Compra**, quando a **Finalidade de Emissão** (`PurposeOfIssue`) é **Complementar**, **Devolução** ou **Nota de Crédito** de um dos tipos acima.

Na devolução, a mensagem orienta a referenciar por item:

> *Os textos adicionais do cabeçalho referenciam %1 documentos diferentes, mas a partir de 16 de novembro de 2026 uma NF-e de Devolução aceita apenas um documento referenciado no cabeçalho (regra B25-40, rejeição 255). Referencie a NF-e original em cada linha de item.*

Nas demais finalidades, a orientação é emitir uma nota por documento de origem:

> *Os textos adicionais do cabeçalho referenciam %1 documentos diferentes, mas a partir de 16 de novembro de 2026 uma NF-e do tipo %2 aceita apenas um documento referenciado no cabeçalho (regra B25-40, rejeição 255). Emita uma NF-e para cada documento original.*

Até 15/11/2026 não há limite. As demais finalidades e os demais tipos de nota de crédito não são afetados.

> **Atenção**: os textos adicionais de um documento fiscal registrado ainda podem ser alterados até a autorização, e a transmissão não repete essa conferência. Uma segunda referência incluída depois do registro só é apontada pela SEFAZ, com a rejeição 255. Da mesma forma, um documento registrado até 15/11/2026 e transmitido depois que a SEFAZ passar a aplicar a versão 1.53 pode ser rejeitado por ela, mesmo tendo passado pelo registro.

### Emitentes das NF-e referenciadas

Em qualquer data de registro, o sistema confere as chaves informadas em **Chave Acesso Ref.** nas linhas de item da devolução, conforme as regras VC02-40 e VC02-50 da NT 2025.002. Na redação da versão 1.53, as duas regras comparam a **raiz do CNPJ** (os 8 primeiros caracteres, inclusive no CNPJ alfanumérico) ou, quando o emitente é pessoa física, o **CPF** inteiro. Notas de filiais diferentes da mesma empresa são aceitas. O sistema usa essa comparação em qualquer data, porque raízes diferentes também são rejeitadas pela regra anterior, que comparava o CNPJ completo.

- **Mesmo emitente em todas as linhas** (VC02-40): todas as NF-e referenciadas precisam ter sido emitidas pela mesma raiz de CNPJ ou pelo mesmo CPF. Se houver mais de um emitente, o registro é interrompido, com a lista dos emitentes encontrados, um CNPJ/CPF de cada raiz:

> *As NF-e referenciadas nas linhas de item devem ter todas o mesmo CNPJ base (8 primeiros caracteres) ou CPF do emitente, mas foram emitidas por %1 (regra VC02-40, rejeição 1193).*

- **Emitente igual ao destinatário** (VC02-50): vale só para a devolução **de saída** (**Nota Crédito Compra** e **Devolução de Compra**, ou pedido e fatura de venda com **Finalidade Emissão** = **Devolução**) e só quando a NF-e referenciada também é **de saída**. Nesse caso, o emitente da NF-e referenciada precisa ter a mesma raiz de CNPJ, ou o mesmo CPF, do destinatário da devolução, ou seja, do cliente ou do fornecedor do documento. Uma mercadoria comprada da filial de um fornecedor cadastrado pelo CNPJ da matriz pode ser devolvida a ele. Caso contrário:

> *Linha %1: a NF-e referenciada %2 é uma NF-e de saída emitida por %3, que deve ter o mesmo CNPJ base ou CPF do destinatário desta devolução (%4) (regra VC02-50, rejeição 1194).*

O emitente é lido da própria chave de acesso. Na prática:

- A **nota fiscal avulsa** (série 890 a 899), que traz na chave o CNPJ da SEFAZ e não o do emitente, fica de fora das duas conferências. Na versão 1.53 a SEFAZ passa a considerar o emitente real dessa nota, e a validação fica com ela.
- O sentido da NF-e referenciada só é conhecido quando ela está registrada no Business Central: uma NF-e própria de saída ou uma NF-e de terceiro lançada como entrada no sistema. Uma chave digitada de nota que não está no sistema não passa pela regra VC02-50.

### Demais finalidades

As finalidades **Complemento de Quantidade**, **Complemento de Impostos** e **Complemento de Preço** continuam exigindo a referência no cabeçalho, como hoje. A partir de 16/11/2026, essa referência precisa ser única (veja [Uma referência no cabeçalho](#uma-referência-no-cabeçalho)).

---

## 5. Observação Automática de Devolução (Compras)

A flag **Criar Observação Docs. Devolvidos** (`CreateReturnedDocsRemark`), na página **Config. NF-e** (`NFeSetupFNX`), controla a criação automática do texto *"Devolução de mercadorias referente à NF-e: ..."* nas devoluções de compra.

O comportamento mudou:

- A observação agora é apenas texto, montado a partir das referências já informadas no documento, no cabeçalho ou nas linhas. Ela não grava mais a chave de NF-e referenciada na linha de texto, o que causaria o erro de referenciamento em dois níveis.
- Sem nenhuma referência informada no documento, nenhum texto é criado.

> **Mudança de comportamento**: quem dependia da observação automática para referenciar a NF-e de origem sozinha (a observação preenchia a chave referenciada e satisfazia a validação) passa a informar a referência manualmente: por item ou, no cabeçalho, pela **Chave NF-e Ref.** (a partir de 16/11/2026, uma única chave).
