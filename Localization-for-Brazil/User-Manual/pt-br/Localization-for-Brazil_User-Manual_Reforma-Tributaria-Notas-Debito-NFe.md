<!-- Source: NT 2025.002 -->
<!-- PR: #284, #340 -->
# Reforma Tributária - Notas de Débito NF-e

**Data**: Maio/2026

---

## Sumário

A Nota de Débito é uma nova finalidade de emissão da NF-e (finNFe = 6), criada pela Nota Técnica 2025.002 para atender à Reforma Tributária. Ela serve para documentar um **aumento no imposto devido pelo emissor** em operações já registradas, sendo utilizada para ajustes na apuração dos novos tributos IBS e CBS.

> O sentido de "débito" é sempre **sob a ótica do emissor** do documento. A utilização se restringe a ajustes dos novos tributos (IBS/CBS) — **não se aplica** a tributos como ICMS e IPI.

Alguns tipos de Nota de Débito exigem um cClassTrib específico nos itens, e alguns cClassTrib só podem ser usados num tipo de Nota de Débito. O sistema confere esses pares antes do registro, pela matriz baixada da Localização (veja [Conferência do cClassTrib por tipo de nota](#conferência-do-cclasstrib-por-tipo-de-nota)).

---

## Tipos de Nota de Débito

| Código | Tipo | Descrição |
|--------|------|-----------|
| T01 | Transferência de créditos para Cooperativas | Transferência de créditos de IBS/CBS do cooperado para a cooperativa |
| T02 | Anulação de Crédito por Saídas Imunes/Isentas | Estorno de crédito apropriado em entradas quando a saída é imune ou isenta |
| T03 | Débitos de notas fiscais não processadas na apuração | Regularização de débitos de NF-e que não foram incluídas na apuração original |
| T04 | Multas e Juros | Aumento de imposto referente a multas e juros devidos |
| T05 | Transferência de crédito de sucessão | Transferência de créditos tributários em processo de sucessão empresarial |
| T06 | Pagamento antecipado | Débito gerado por pagamento antecipado de tributo |
| T07 | Perda em Estoque | Débito gerado por perda, perecimento ou roubo de mercadoria em estoque |
| T08 | Desenquadramento do SN | Débito decorrente do desenquadramento da empresa do Simples Nacional |

---

## Onde criar e como preencher

| Módulo | Documentos disponíveis | Como preencher |
|--------|----------------------|----------------|
| **Vendas** | Ordem de Venda ou Fatura de Venda | 1. Na aba **Fiscal**, no campo **Finalidade de Emissão**, selecione **"Nota de Débito"**. 2. Acesse **Dados NF-e** (menu lateral > Dados Fiscais > Linha NF-e > Abrir) e preencha o campo **Tipo Nota de Débito** com o tipo desejado. |
| **Compras** | Nota de Crédito de Compra | Mesmo procedimento: **Finalidade de Emissão** na aba Fiscal + **Tipo Nota de Débito** nos Dados NF-e. |
| **Documento Fiscal** | Documento Fiscal de Venda ou Documento Fiscal de Compra (manual) | Na aba **NF-e**, no campo **Finalidade de Emissão**, selecione **"Nota de Débito"**. O campo **Tipo Nota de Débito** aparece logo abaixo; preencha-o com o tipo desejado. |

---

## Conferência do cClassTrib por tipo de nota

A NT 2025.002 (regras UB14-60 e UB14-70) liga alguns tipos de Nota de Débito a um cClassTrib específico. Se a nota não respeitar esses pares, a SEFAZ a rejeita (rejeição 1200 ou 1202). Para evitar que a nota seja registrada e só depois rejeitada, o sistema confere os pares da matriz **Tipos de Nota Class. Trib. IBS/CBS** antes de registrar documentos de venda e de compra e antes de liberar o documento fiscal manual. A matriz, o download e a desativação de linhas estão descritos em [Reforma Tributária — Configurações](Localization-for-Brazil_User-Manual_Reforma-Tributaria-Configuracoes.md#47-tipos-de-nota-class-trib-ibscbs).

> **Importante:** a matriz começa vazia. Enquanto a função **Baixar Matriz da Localização...** não for executada, nenhuma nota é barrada por esta conferência.

Com o arquivo publicado pela Localização, os pares de Nota de Débito são:

| Tipo Nota de Débito | cClassTrib exigido nos itens |
|---|---|
| 01 – Transferência de créditos para Cooperativas | `800002` – Transferência de crédito do associado |
| 02 – Anulação de Crédito por Saídas Imunes/Isentas | `811001` – Anulação de crédito por saída imune/isenta |
| 03 – Débitos de notas fiscais não processadas na apuração | `811002` – Débito de NF não processada na apuração |
| 05 – Transferência de crédito de sucessão | `800001` – Fusão, cisão ou incorporação |
| 07 – Perda em Estoque | `410030` – Estorno de crédito por perda |
| 08 – Desenquadramento do SN | `811003` – Desenquadramento do Simples Nacional |
| 04 – Multas e Juros e 06 – Pagamento antecipado | Livre, desde que não seja um cClassTrib exclusivo de outro tipo de nota |

A conferência vale nos dois sentidos:

- **Tipo de nota que exige um cClassTrib (UB14-70):** numa Nota de Débito de um tipo da tabela, todos os itens precisam usar o cClassTrib pareado com ele. Por exemplo, uma Nota de Débito 07 com um item de cClassTrib `000001` é barrada.
- **cClassTrib exclusivo (UB14-60):** um cClassTrib da tabela só pode ser usado no tipo de nota com que está pareado. Por exemplo, o `811002` numa venda normal, ou numa Nota de Débito 04, é barrado. O `800001` é aceito tanto na Nota de Débito 05 quanto na Nota de Crédito 05.

Quando a nota é barrada, o erro aparece na lista **Mensagens de Erro**, com a linha, o item, o cClassTrib usado, a finalidade e o tipo da nota, o que é aceito e a regra da NT. Corrija o **Cód. Class. Trib. IBS/CBS** da linha ou o **Tipo Nota de Débito** e registre (ou libere) o documento de novo. Se uma linha da matriz estiver barrando uma nota que deveria passar, o responsável fiscal pode desativá-la.

---

## Cenários para cada tipo de Nota de Débito

### Tipo 01 - Transferência de créditos para Cooperativas

O **art. 271 da Lei Complementar nº 214/2025** estabelece um **regime específico de tributação** aplicável às **sociedades cooperativas** que optem por sua adoção, prevendo, em seu **inciso I**, a **redução de 100% da alíquota do IBS** sobre os fornecimentos realizados entre o cooperado e a respectiva cooperativa. Em decorrência dessa desoneração, podem permanecer, na escrituração do cooperado sujeito ao **regime regular de apuração**, **créditos de IBS apropriados e não utilizados**, originados de aquisições de bens e serviços vinculados às operações destinadas à cooperativa. Para que esses créditos não se acumulem indefinidamente no cooperado — beneficiando, em última instância, apenas uma das pontas da cadeia produtiva —, a legislação faculta a **transferência desses créditos à cooperativa destinatária**.

Para essas hipóteses deve ser emitida pelo **cooperado (associado)** sujeito ao regime regular de apuração, com a cooperativa optante pelo regime específico do art. 271 figurando como **destinatária**. O efeito da nota na apuração assistida é o **lançamento a débito** na apuração do cooperado emitente e o **lançamento a crédito** correspondente na apuração da cooperativa destinatária, formalizando a migração dos créditos entre as duas pessoas jurídicas. O preenchimento do **Grupo de Informações de Transferências de Crédito** (`gTransfCred`) é **obrigatório** nesse tipo de nota, devendo ser utilizado o código de classificação tributária `cClassTrib = 800002`.

Cabe destacar que o valor a ser transferido fica **limitado ao montante de créditos efetivamente apropriados e não utilizados** pelo cooperado no período de apuração. Caso o valor destacado na nota fiscal **exceda o limite de créditos disponíveis**, a parcela excedente **não produzirá efeitos** na apuração assistida, sendo automaticamente desconsiderada pelo sistema do CG-IBS.

> **Importante**
>
> Por se tratar de um contexto que depende do **Sistema de Apuração Assistida do IBS**, que ainda está em fase piloto, não há indicação de procedimento. Caso sua empresa se enquadre nesta situação, pedimos que reporte através de nossa plataforma de suporte: https://network.arquiconsult.com/

### Tipo 02 - Anulação de Crédito por Saídas Imunes/Isentas

Os créditos de IBS vinculados a aquisições que sejam utilizadas em **operações subsequentes imunes ou isentas** devem ser **anulados proporcionalmente**, ressalvadas as hipóteses excepcionadas. A regra decorre da lógica da não cumulatividade aplicada ao IBS: o crédito apropriado na entrada pressupõe que o bem ou serviço seja utilizado em operação tributada subsequente; quando essa operação resulta imune ou isenta, o crédito perde seu fundamento e deve ser revertido, evitando que o desonero da saída seja indevidamente potencializado pela manutenção do crédito da entrada.

Para auxiliar o contribuinte, o **sistema de apuração assistida do IBS** apresenta, ao final de cada período de apuração, **um valor sugerido de anulação**, calculado automaticamente a partir dos valores e dos códigos de classificação tributária (`cClassTrib`) informados nas notas fiscais de fornecimento emitidas pelo contribuinte. Esse valor tem caráter **indicativo**: a efetivação do estorno depende de ação do próprio contribuinte, que deverá emitir a **nota fiscal de débito do tipo 02 – Anulação de Crédito por Saídas Imunes/Isentas** durante o **período de ajustes definido em regulamento**. Essa nota deve ser criada com **emitente e destinatário coincidentes**, ou seja, mesmo CNPJ da empresa, e com preenchimento obrigatório do grupo **"Ajuste de Competência"** (`gAjusteCompet`), indicando o período a que o ajuste se refere.

> **Importante**
>
> Por se tratar de um contexto que depende do **Sistema de Apuração Assistida do IBS**, que ainda está em fase piloto, não há indicação de procedimento. Caso sua empresa se enquadre nesta situação, pedimos que reporte através de nossa plataforma de suporte: https://network.arquiconsult.com/

### Tipo 03 - Débitos de notas fiscais não processadas na apuração

No modelo de **apuração assistida do IBS**, o sistema do CG-IBS consolida automaticamente, ao final de cada período de apuração, os débitos e créditos do contribuinte com base nos documentos fiscais eletrônicos efetivamente processados. Pode ocorrer, contudo, que **uma ou mais notas fiscais de fornecimento legitimamente emitidas não tenham sido reconhecidas** pelo sistema dentro do período — em razão de falhas técnicas, contingências, problemas de transmissão ou outras inconsistências de processamento. Nesses casos, o saldo apresentado na prévia da apuração estará **incompleto**, deixando de refletir débitos efetivamente devidos pelo contribuinte.

Esta nota deve ser emitida pelo contribuinte do **regime regular**, durante o **período de ajustes definido em regulamento**, para garantir que o saldo final reflita corretamente o total de débitos devidos. A nota é emitida com **emitente e destinatário coincidentes**, ou seja, o CNPJ base do próprio contribuinte. Cada item corresponde a um documento fiscal não processado, **referenciando sua chave de acesso** no grupo `<DFeReferenciado>` e informando, no campo `<vIBS>`, o **valor total do débito** do documento faltante. Deve-se, ainda, preencher o grupo **"Ajuste de Competência"** (`gAjusteCompet`), indicando o período em que o débito deve ser lançado.

> **Importante**
>
> Por se tratar de um contexto que depende do **Sistema de Apuração Assistida do IBS**, que ainda está em fase piloto, não há indicação de procedimento. Caso sua empresa se enquadre nesta situação, pedimos que reporte através de nossa plataforma de suporte: https://network.arquiconsult.com/

### Tipo 04 - Multas e Juros

Meu fornecedor recebeu valores de multa ou juros por atraso no pagamento que efetuei. Ele deve emitir uma nota fiscal de débito do tipo "04 – Multa e Juros". Essa nota serve para complementar a base de cálculo do IBS (Imposto sobre Bens e Serviços) referente ao fornecimento.

Deve ser emitida pelo **fornecedor** no **momento do efetivo recebimento** dos acréscimos moratórios, lançando-se o débito correspondente no período de apuração do recebimento. Cada item da nota deve **referenciar o item da nota fiscal original** a que o acréscimo se relaciona, aplicando-se **a mesma classificação tributária (cClassTrib) e a mesma alíquota efetiva** incidente sobre o fornecimento original — assegurando, dessa forma, coerência entre o tratamento tributário da operação principal e o de seu acréscimo.


Neste cenário, o fornecedor deve emitir uma **Fatura de Venda** para o cliente. Como essa operação não representa venda efetiva de mercadoria ou serviço, é necessário utilizar um **Tipo de Operação** que não gere movimentação de estoque:

1. **Crie uma Fatura de Venda** para o cliente em questão.
2. Preencha as linhas da fatura com os valores de multa/juros pagos.
3. **Na aba Fiscal**, preencha o campo **Tipo de Operação** com uma opção que **não gere movimentação de estoque** (campo **Mov. Item** desmarcado).
4. **Na aba Fiscal**, no campo **Finalidade de Emissão**, selecione **"Nota de Débito"**.
5. **Nos Dados Fiscais do cabeçalho** (menu lateral > **Dados Fiscais**), preencha o campo **Tipo Nota de Débito** com **"Multas e Juros"** (código 04).
6. **Abra os Textos Adicionais** do documento (ação **Texto Adicional** disponível no cabeçalho da Fatura de Venda). Nessa tela, informe a NF-e original do fornecedor:
   - **Nº Documento Referenciado** — selecione o documento fiscal original (se já existir no sistema) ou preencha manualmente os dados da nota do fornecedor.
   - **Chave Acesso NF-e** — será preenchida automaticamente ao selecionar o documento, ou informe manualmente a chave de 44 dígitos da NF-e original.

#### Resumo dos campos

| Onde              | Campo                                         | Valor                                |
| ----------------- | --------------------------------------------- | ------------------------------------ |
| Aba Fiscal        | Tipo de Operação                              | Código sem movimentação de estoque   |
| Aba Fiscal        | Finalidade de Emissão                         | Nota de Débito                       |
| Dados Fiscais     | Tipo Nota de Débito                           | Multas e Juros (04)                  |
| Textos Adicionais | Nº Documento Referenciado / Chave Acesso NF-e | Dados da NF-e original do fornecedor |

> **Importante:** O Tipo de Operação é essencial neste cenário porque como a Nota de Débito de multas e juros não representa venda de mercadoria, a movimentação de estoque deve ser desativada.

### Tipo 05 - Transferência de crédito de sucessão

A empresa que for objeto de **fusão, cisão ou incorporação** e que possua **créditos de IBS apropriados e não utilizados** ao tempo do evento societário poderá **transferi-los à(s) empresa(s) sucessora(s)**.

Esta nota deve ser emitida **pela empresa sucedida** — a fundida, cindida ou incorporada — informando, no campo destinatário, a **empresa sucessora** que receberá o crédito, e o **valor do IBS** correspondente ao montante transferido. Havendo **mais de uma empresa sucessora**, deverão ser emitidas **notas fiscais distintas**, uma para cada destinatária, com a respectiva parcela do crédito a ser transferida.

Cabe destacar que esta é a **modalidade preferencial** de formalização da transferência. Caso, contudo, o **CNPJ da empresa sucedida se torne inapto para emissão de documentos fiscais eletrônicos** antes que a transferência seja efetivada, a operação poderá ser realizada por via alternativa — mediante a emissão, pela(s) sucessora(s), da **nota fiscal de crédito tipo 05 – Transferência de Crédito na Sucessão**.

> **Importante**
>
> Por não se tratar de um contexto comum, não há indicação de procedimento. Caso sua empresa se enquadre nesta situação, pedimos que reporte através de nossa plataforma de suporte: https://network.arquiconsult.com/

### Tipo 06 - Pagamento antecipado

Nos termos do **art. 10, § 4º, inciso I, da Lei Complementar nº 214/2025**, o recebimento de **pagamento antecipado** relativo a fornecimento futuro de bens ou serviços constitui hipótese de **antecipação do fato gerador do IBS**, devendo o tributo ser destacado e lançado a débito no período de apuração em que o valor for efetivamente recebido pelo fornecedor — ainda que a entrega ou disponibilização do bem, ou a conclusão do serviço, venha a ocorrer em momento posterior.

Esta nota deve ser emitida pelo fornecedor a cada recebimento antecipado, aplicando-se a **mesma classificação tributária (cClassTrib) e a mesma alíquota** incidente sobre o bem ou serviço a ser fornecido. Quando houver direito à apropriação de crédito pelo adquirente, este será reconhecido **no momento da extinção do débito do fornecedor**, observadas as regras gerais aplicáveis às notas fiscais de fornecimento.

#### Passo a passo

1. **Crie uma Fatura de Venda** — preencha o cliente e demais informações.
2. **Defina a Finalidade de Emissão** — na **aba Fiscal** do documento, localize o campo **Finalidade de Emissão** e selecione a opção **"Nota de Débito"**.
3. **Preencha as linhas do documento** — insira os itens/serviços e valores conforme a operação.
4. **Informe o Tipo Nota de Débito** — no menu lateral direito, acesse **Dados Fiscais > Linha NF-e > Abrir**. No campo **Tipo Nota de Débito**, selecione a opção **"Pagamento antecipado"** (que corresponde ao tipo 06).
5. Preencha o campo **Tipo de Operação** com uma opção que movimente somente financeiro (sem movimento de item).
6. **Informe a Área de Imposto** adequada.
7. **Registre o documento** — após preencher todos os dados.

> **Importante**
>
> Posteriormente, por ocasião do **efetivo fornecimento**, o contribuinte deverá emitir a respectiva nota fiscal com o **valor total da operação**, referenciando as chaves de acesso das notas de antecipação previamente emitidas, nos **Textos Adicionais**, usando um tipo de operação que movimente somente o item, visto que o financeiro foi efetivado anteriormente.

### Tipo 07 - Perda em Estoque

Nos termos do **art. 47, § 6º, da Lei Complementar nº 214/2025**, os créditos de **IBS** apropriados sobre **bens materiais em estoque** que venham a ser objeto de **perda** — bem como os créditos vinculados a **serviços relacionados** a esses bens — devem ser **estornados** da apuração do contribuinte.

Deve ser emitida quando o bem perdido tiver sido adquirido de terceiros; a nota deve **referenciar o documento fiscal de aquisição** e, quando aplicável, o **documento fiscal dos serviços vinculados** a essa aquisição, permitindo ao sistema vincular o estorno aos créditos originalmente apropriados. O efeito da nota na apuração assistida é o **lançamento a débito** do valor de IBS correspondente aos créditos a serem estornados, regularizando o saldo do contribuinte conforme determina a legislação.

Cabe destacar que este procedimento **não se aplica às perdas ocorridas durante o transporte de bens**, hipóteses em que devem ser utilizados os eventos próprios de **perecimento, perda, roubo ou furto durante o transporte contratado pelo fornecedor (CIF)** ou **pelo adquirente (FOB)**, conforme o caso.

#### Passo a passo

1. **Crie uma Fatura de Venda** — crie documento normalmente, com o cliente sendo a própria empresa, pois a nota deve ter como emitente e destinatário o próprio estabelecimento.
2. **Defina a Finalidade de Emissão** — na aba **Fiscal** do documento, localize o campo **Finalidade de Emissão** e selecione a opção **Nota de Débito**.
3. **Informe o Tipo de Nota de Débito** — após selecionar a finalidade, acesse o menu lateral **Dados Fiscais > Linha NF-e > Abrir**. No campo **Tipo Nota de Débito**, selecione a opção **Perda em Estoque** (que corresponde ao tipo 07). Nas linhas, o **Cód. Class. Trib. IBS/CBS** deve ser `410030`: com a matriz baixada, a Nota de Débito 07 com outro cClassTrib não é registrada (veja [Conferência do cClassTrib por tipo de nota](#conferência-do-cclasstrib-por-tipo-de-nota)).
4. Preencha o campo **Tipo de Operação** com uma opção que movimente somente item (sem movimento financeiro).
5. **Informe a Área de Imposto** adequada.
6. **Preencha nos Textos Adicionais** a nota original nos casos em que o bem perdido foi adquirido de terceiros — tanto da nota de **aquisição do próprio bem** quanto, quando houver, das **notas dos serviços associados** a essa aquisição (frete, armazenagem, seguro do transporte, etc., quando geraram crédito de IBS apropriado). Isto não se aplica para itens produzidos pela própria empresa.
7. **Verifique a contabilização** para confirmar se estão sendo gerados somente movimentos contábeis corretos, movimentos de item e imposto.

> **Importante**
>
> Se a perda ocorrer **durante o transporte** (e não em estoque), a regularização deve ser feita por **evento específico** — `112130` (transporte CIF, pelo fornecedor) ou `211124` (transporte FOB, pelo adquirente) — e **não** pela nota fiscal de débito tipo 07.

### Tipo 08 - Desenquadramento do SN

Este tipo destina-se à regularização de débitos de IBS originados em períodos de enquadramento no Simples Nacional. Como o **escopo da Loc BR** contempla exclusivamente contribuintes do regime regular de apuração (Lucro Real e Lucro Presumido), o cenário não é aplicável. O tipo 08 está desabilitado e não deve ser utilizado.
