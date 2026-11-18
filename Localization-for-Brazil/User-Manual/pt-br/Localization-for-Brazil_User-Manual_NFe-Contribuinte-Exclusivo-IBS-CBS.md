<!-- PR: #292 -->
# NF-e sem Inscrição Estadual: Contribuinte Exclusivo do IBS/CBS (NT 2026.007)

**Data**: Outubro/2026

---

## Sumário

O estabelecimento que é contribuinte apenas do IBS/CBS e não tem Inscrição Estadual passa a emitir NF-e modelo 55 pelo Business Central. É o caso do prestador de serviço que vende um bem do ativo, transfere bens de uso e consumo ou devolve uma compra. Basta marcar a filial (ou a empresa sem filiais) como **Contribuinte Exclusivo do IBS/CBS**. A partir daí, a NF-e desse estabelecimento sai sem a I.E. do emitente e é autorizada pela SVRS, qualquer que seja a UF. A nota não pode ter ICMS e precisa de IBS/CBS em todos os itens. O sistema confere essas regras no registro, antes da transmissão.

---

## 1. Contexto

A Nota Técnica 2026.007 (v1.10), da Reforma Tributária (LC 214/2025), define como fica a NF-e do contribuinte exclusivo do IBS/CBS, que não tem Inscrição Estadual:

| Regra da NT | Como fica no Business Central |
|---|---|
| A NF-e sai **sem a I.E. do emitente** | A nota não leva a I.E. do emitente nem a I.E. de substituto tributário, mesmo que haja I.E.-ST cadastrada para a UF de destino |
| A NF-e é autorizada **sempre pela SVRS** | Fora da contingência, a nota vai à SVRS em homologação e em produção. Os endereços da SVRS já vêm no sistema, sem cadastro. A chave de acesso e a UF do emitente continuam sendo as da UF da filial |
| A NF-e **não pode ter ICMS** (regra N01-10, rejeição 161) | Nenhum item leva ICMS, ICMS-ST, FCP nem a partilha do DIFAL. Se algum desses impostos for calculado, o registro é bloqueado. As exceções estão em [Exceções: devolução e nota de crédito tipo 03](#4-exceções-devolução-e-nota-de-crédito-tipo-03) |
| Todo item **tem de ter IBS/CBS** (rejeição 162) | O **Cód. Class. Trib. IBS/CBS** passa a ser obrigatório nos itens desse estabelecimento, mesmo com a Reforma Tributária desabilitada |
| **NFC-e (modelo 65) é proibida** (regra C17-42, rejeição 156) | O registro de NFC-e pelo estabelecimento exclusivo é bloqueado |

A regra entra em **produção em 03/11/2026**. A homologação já está disponível.

Vale para todos os fluxos que emitem NF-e própria:

- venda e devolução de venda;
- compra com emissão própria e devolução de compra;
- transferência entre estabelecimentos;
- documento fiscal simples de venda, de compra e de transferência.

Para os estabelecimentos com I.E., nada muda: a emissão, os eventos, a inutilização, a contingência, a NFC-e e o envio à SEFAZ da UF continuam como antes.

---

## 2. Configuração

### 2.1 Classificar o estabelecimento

A classificação fica no cadastro da filial ou, em empresa sem filiais, no cadastro da empresa.

1. Navegue até **Informação Filial** (página `BranchInformationFNX`) ou **Informação Empresa** (página `Company Information`).
2. Na guia **Impostos**, marque **Contribuinte Exclusivo do IBS/CBS** (`ExclusiveIBSCBSTaxpayer` na filial, `ExclusiveIBSCBSTaxpayerFNX` na empresa).
3. Deixe o campo **I.E.** em branco. Um texto sem algarismos, como "ISENTO", também é aceito, porque a nota do estabelecimento exclusivo nunca leva a I.E. do emitente.

O estabelecimento não pode ser exclusivo e ter uma I.E. com número ao mesmo tempo. Ao marcar a classificação com a I.E. preenchida, ou ao preencher a I.E. com a classificação marcada, o sistema mostra:

> O estabelecimento não pode ter I.E. e ser contribuinte exclusivo do IBS/CBS ao mesmo tempo. Limpe a I.E. ou o campo Contribuinte Exclusivo do IBS/CBS.

> **Importante**
> A marcação **Não Contribuinte ICMS** (`NotICMSContributorFNX`), que já existia no mesmo cadastro, continua com o mesmo papel e **não** identifica o contribuinte exclusivo. Use apenas **Contribuinte Exclusivo do IBS/CBS**.

### 2.2 Classificação tributária de IBS/CBS nos itens

Todo item de mercadoria, de ativo imobilizado ou com **Nº Item Fiscal** precisa do **Cód. Class. Trib. IBS/CBS** nas linhas do documento. A exigência vale mesmo com o **Status Reforma Tributária** das **Configurações Fiscais** como `Desabilitada`. Sem o código, o registro é bloqueado, indicando a linha.

A configuração das classificações e dos vínculos por NCM está em [Reforma Tributária - NF-e: Configuração e Emissão](Localization-for-Brazil_User-Manual_NFe-Reforma-Tributaria.md).

### 2.3 Matriz e áreas de impostos sem ICMS

As operações do estabelecimento exclusivo não podem calcular ICMS, ICMS-ST nem DIFAL. Ajuste a **Matriz Impostos** ou a **Área Impostos** usada nos documentos dessa filial para que esses impostos não sejam calculados, exceto nas devoluções e nas notas de crédito tipo 03.

O **CST ICMS** e o **Cód. Origem** deixam de ser obrigatórios nas linhas de mercadoria e de ativo do estabelecimento exclusivo, fora das exceções.

### 2.4 Config. NF-e

Nada muda na **Config. NF-e** da filial. Mantenha o **Ambiente** e a **UF** da filial. Com **Tipo Emissão** em `Normal`, a nota vai à SVRS automaticamente. Não é preciso cadastrar endereços da SVRS.

---

## 3. Comportamento na Emissão

### 3.1 Campo Emitida sem I.E. no documento fiscal

Cada documento fiscal tem o campo **Emitida sem I.E.** (`IssuedWithoutIE`). É ele que decide se a NF-e sai sem I.E. e pela SVRS.

- **Quando é preenchido:** o sistema o marca na criação do documento e sempre que a filial ou o **Tipo Documento Fiscal** do documento muda. Ele fica marcado quando o documento é NF-e própria modelo 55 de um estabelecimento **Contribuinte Exclusivo do IBS/CBS**.
- **Onde aparece:**
  - nos cartões **Documento Fiscal de Venda**, **Documento Fiscal de Compra** e **Documento Fiscal de Transferência**, na guia **Fiscal**, somente leitura;
  - no **Documento Fiscal Registrado**, na guia **Fiscal**;
  - como coluna oculta nas listas de documentos fiscais, que pode ser exibida pela personalização da página.
- **Como corrigir:** no **Documento Fiscal Registrado**, o campo pode ser alterado com a ação **Editar Campos**, enquanto a NF-e não estiver autorizada. Veja [Editar Campos do Documento Fiscal Registrado](Localization-for-Brazil_User-Manual_Editar-Campos-Documento-Fiscal-Registrado.md). A correção não recalcula os impostos: a transmissão envia a nota como ela está.

Reclassificar a filial não altera os documentos fiscais já registrados. Eles mantêm o **Emitida sem I.E.** com que foram registrados.

### 3.2 Conferências no registro

Ao registrar uma venda, uma compra com emissão própria ou um documento fiscal simples (ação **Liberar**), o sistema confere, para a NF-e do estabelecimento exclusivo:

| Conferência | Quando bloqueia |
|---|---|
| ICMS | Uma linha tem ICMS, ICMS-ST ou DIFAL com base, valor ou FCP diferente de zero. Uma base sem valor também bloqueia |
| IBS/CBS | Uma linha de mercadoria, de ativo ou com **Nº Item Fiscal** está sem **Cód. Class. Trib. IBS/CBS** |
| NFC-e | O **Tipo Documento Fiscal** é de NFC-e (modelo 65) |

A conferência de ICMS indica a linha e os valores:

> Linha 10000: ICMS está calculado (base 1.000,00, valor 180,00, FCP 0,00), mas a NF-e de um contribuinte exclusivo do IBS/CBS não pode conter ICMS (regra N01-10, rejeição 161). Somente a NF-e de devolução e a nota de crédito do tipo 03 aceitam ICMS. Revise os impostos da linha.

A conferência de NFC-e mostra:

> Um contribuinte exclusivo do IBS/CBS não pode emitir NFC-e, modelo 65 (regra C17-42, rejeição 156). Use um tipo de documento fiscal de NF-e, modelo 55.

As conferências rodam no registro e na liberação, não na transmissão. Uma nota registrada antes da reclassificação da filial, ou alterada depois do registro, vai à SEFAZ como está e pode voltar com as rejeições 161 ou 162.

### 3.3 O que vai na NF-e

| Grupo da NF-e | Estabelecimento exclusivo | Estabelecimento com I.E. |
|---|---|---|
| I.E. do emitente (`emit/IE`) | Não enviada | Enviada |
| I.E. de substituto tributário (`emit/IEST`) | Não enviada | Enviada quando cadastrada para a UF de destino |
| Grupo `ICMS` dos itens | Não enviado, exceto nas exceções | Enviado |
| Partilha do DIFAL (`ICMSUFDest`) | Não enviada, exceto nas exceções | Enviada quando calculada |
| Grupo `IBSCBS` dos itens | Enviado em todos os itens | Conforme o **Status Reforma Tributária** |
| Autorizador | SVRS, fora da contingência | SEFAZ da UF da filial |

O CFOP não é conferido antes do envio. Se o CFOP não for permitido ao contribuinte exclusivo, a SEFAZ rejeita a nota (rejeição 159), e o motivo aparece no documento fiscal como em qualquer outra rejeição.

O DANFE sai com o campo de I.E. do emitente em branco.

### 3.4 Transferência para uma filial exclusiva

Quando a filial de destino de uma transferência é **Contribuinte Exclusivo do IBS/CBS**, ela é tratada como não contribuinte do ICMS no cálculo de impostos e no documento fiscal, com a I.E. em branco ou "ISENTO".

---

## 4. Exceções: Devolução e Nota de Crédito tipo 03

A NT permite ICMS em dois casos. Neles, o sistema não bloqueia o ICMS calculado, envia o grupo `ICMS` normalmente e continua exigindo **CST ICMS** e **Cód. Origem** nas linhas:

| Caso | Como o documento é identificado |
|---|---|
| NF-e de devolução | **Finalidade Emissão** = **Devolução** |
| Nota de crédito tipo 03 | **Finalidade Emissão** = **Nota de Crédito** e **Tipo Nota de Crédito** = **Retorno por recusa total na entrega ou por não localização do destinatário na tentativa de entrega** |

Os demais tipos de nota de crédito e de débito seguem a regra geral, sem ICMS. O preenchimento da nota de crédito está em [Reforma Tributária - Notas de Crédito NF-e](Localization-for-Brazil_User-Manual_Reforma-Tributaria-Notas-Credito-NFe.md).

O **Cód. Class. Trib. IBS/CBS** continua obrigatório nas exceções.

---

## 5. Eventos, Consulta e Inutilização

| Operação | Para onde vai |
|---|---|
| Retorno da autorização, consulta de situação, cancelamento e carta de correção | Ao autorizador da própria nota: SVRS quando o documento tem **Emitida sem I.E.** marcado; SEFAZ da UF quando não tem |
| Inutilização do número de uma nota | Segue o **Emitida sem I.E.** do documento |
| Inutilização de uma faixa de numeração de NF-e | Segue a classificação **atual** do estabelecimento: SVRS para o exclusivo |

Assim, depois de reclassificar uma filial, as notas antigas emitidas com I.E. continuam sendo canceladas e corrigidas na SEFAZ da UF, e as emitidas sem I.E., na SVRS.

A inutilização de faixa de NFC-e (modelo 65) pelo estabelecimento exclusivo é bloqueada. Números de NFC-e do período em que a filial tinha I.E. não podem ser inutilizados depois da reclassificação.

---

## 6. Contingência

Em contingência (**Tipo Emissão** diferente de `Normal` na **Config. NF-e**), o estabelecimento exclusivo segue as mesmas regras de contingência de hoje e usa os serviços de contingência da UF. A nota continua sem I.E., sem ICMS e com IBS/CBS.

> **Atenção**
> Um serviço de contingência (SVC) pode rejeitar a nota sem I.E. do emitente (rejeição 166). Nesse caso, aguarde o fim da contingência e transmita a nota com **Tipo Emissão** em `Normal`.

---

## 7. Estabelecimentos com I.E.: o que muda

- **I.E. sem algarismos:** a filial ou a empresa que **não** é contribuinte exclusiva precisa de uma I.E. com número para transmitir. Uma I.E. como "ISENTO" saía em branco na NF-e e era rejeitada pela SEFAZ (rejeição 166). Agora a transmissão é barrada antes do envio, com a mensagem:

  > O campo I.E. "ISENTO" de *nome da filial* deve conter ao menos um algarismo. Um estabelecimento sem I.E. que seja contribuinte exclusivo do IBS/CBS deve ter o campo Contribuinte Exclusivo do IBS/CBS marcado.

  Se o estabelecimento de fato não tem I.E., classifique-o como exclusivo ([Configuração](#2-configuração)).

- **Rejeições 156 a 188:** a NT criou rejeições nessa faixa, que antes só tinha códigos de sucesso. O sistema as trata como rejeição, para qualquer emitente. A nota rejeitada pode ser corrigida e transmitida de novo, e a consulta de situação reabre o documento, como nas rejeições a partir de 201.

---

## 8. Pontos de Atenção

- **Prazo:** a regra entra em produção em 03/11/2026. Teste a emissão em homologação antes dessa data.
- **Cadastro na Receita Federal:** a mesma NT traz validações do cadastro do emitente, do destinatário e dos locais de retirada e de entrega na Receita Federal e no Cadastro Centralizado de Contribuintes. Elas valem para todos os emitentes e são feitas pela SEFAZ, sem mudança no sistema. A partir de 03/11/2026 podem surgir rejeições por cadastro desatualizado. Em homologação, a SEFAZ consulta a base de produção da Receita.
- **Fora do escopo:**
  - a escrituração fiscal (SPED) das notas do estabelecimento exclusivo;
  - a NF-e avulsa (séries 890 a 919);
  - o emitente MEI (CRT 4).

---

## 9. Resumo dos Campos Novos

### Tabela Informação Filial (`BranchInformationFNX`)

| Campo | Tipo | Descrição |
|---|---|---|
| **Contribuinte Exclusivo do IBS/CBS** (`ExclusiveIBSCBSTaxpayer`) | Booleano | A filial não tem I.E. e emite NF-e sem I.E., pela SVRS, sem ICMS e com IBS/CBS em todos os itens. Não pode emitir NFC-e |

### Tabela Informação Empresa (`Company Information`)

| Campo | Tipo | Descrição |
|---|---|---|
| **Contribuinte Exclusivo do IBS/CBS** (`ExclusiveIBSCBSTaxpayerFNX`) | Booleano | O mesmo, para a empresa sem filiais |

### Tabela Documento Fiscal (`FiscalDocumentFNX`)

| Campo | Tipo | Descrição |
|---|---|---|
| **Emitida sem I.E.** (`IssuedWithoutIE`) | Booleano | A NF-e sai sem a I.E. do emitente. A nota, os eventos e a inutilização do número vão à SVRS. É preenchido pela classificação da filial e editável no **Documento Fiscal Registrado** com **Editar Campos**, até a autorização |
