<!-- PR: #284 -->
# Reforma Tributária - Notas de Crédito NF-e

A Nota de Crédito é uma nova finalidade de emissão da NF-e (finNFe = 5), criada pela Nota Técnica 2025.002 para atender à Reforma Tributária. Ela serve para documentar uma **redução no imposto devido pelo emissor** em operações já registradas, sendo um instrumento essencial para ajustes na apuração dos novos tributos IBS e CBS.

> O sentido de "crédito" é sempre **sob a ótica do emissor** do documento. A utilização se restringe a ajustes dos novos tributos (IBS/CBS) — **não se aplica ainda** a tributos como ICMS e IPI.

Alguns tipos de Nota de Crédito exigem um cClassTrib específico nos itens, e alguns cClassTrib só podem ser usados num tipo de nota. O sistema confere esses pares antes do registro, pela matriz baixada da Localização (veja [Conferência do cClassTrib por tipo de nota](#conferência-do-cclasstrib-por-tipo-de-nota)).

---

## Tipos de Nota de Crédito

| Código | Tipo | Descrição |
|--------|------|-----------|
| T01 | Multas e Juros | Redução de imposto referente a multas e juros cobrados indevidamente |
| T02 | Apropriação de crédito presumido de IBS sobre o saldo devedor na ZFM | Crédito presumido específico para operações na Zona Franca de Manaus |
| T03 | Retorno por recusa total na entrega ou não localização do destinatário | Mercadoria devolvida integralmente por recusa ou impossibilidade de entrega |
| T04 | Redução de Valores | Ajuste para reduzir valores de uma operação já documentada |
| T05 | Transferência de crédito de sucessão | Transferência de créditos tributários em processo de sucessão empresarial |
| T06 | Retorno por recusa parcial na entrega | Mercadoria devolvida parcialmente por recusa do destinatário |

---

## Onde criar e como preencher

| Módulo | Documentos disponíveis | Como preencher |
|--------|----------------------|----------------|
| **Vendas** | Nota de Crédito de Venda | 1. Na aba **Fiscal**, no campo **Finalidade de Emissão**, selecione **"Nota de Crédito"**. 2. Acesse **Dados NF-e** (menu lateral > Dados Fiscais > Linha NF-e > Abrir) e preencha o campo **Tipo Nota de Crédito** com o tipo desejado. |
| **Compras** | Ordem de Compra  ou Fatura de Compra | Mesmo procedimento: **Finalidade de Emissão** na aba Fiscal + **Tipo Nota de Crédito** nos Dados NF-e. |
| **Documento Fiscal** | Documento Fiscal de Venda ou Documento Fiscal de Compra (manual) | Na aba **NF-e**, no campo **Finalidade de Emissão**, selecione **"Nota de Crédito"**. O campo **Tipo Nota de Crédito** aparece logo abaixo; preencha-o com o tipo desejado. |

## Conferência do cClassTrib por tipo de nota

A NT 2025.002 (regras UB14-60 e UB14-80) liga alguns tipos de Nota de Crédito a um cClassTrib específico. Se a nota não respeitar esses pares, a SEFAZ a rejeita (rejeição 1201 ou 1202). Para evitar que a nota seja registrada e só depois rejeitada, o sistema confere os pares da matriz **Tipos de Nota Class. Trib. IBS/CBS** antes de registrar documentos de venda e de compra e antes de liberar o documento fiscal manual. A matriz, o download e a desativação de linhas estão descritos em [Reforma Tributária — Configurações](Localization-for-Brazil_User-Manual_Reforma-Tributaria-Configuracoes.md#47-tipos-de-nota-class-trib-ibscbs).

> **Importante:** a matriz começa vazia. Enquanto a função **Baixar Matriz da Localização...** não for executada, nenhuma nota é barrada por esta conferência.

Com o arquivo publicado pela Localização, os pares de Nota de Crédito são:

| Tipo Nota de Crédito | cClassTrib exigido nos itens |
|---|---|
| 02 – Apropriação de crédito presumido de IBS sobre o saldo devedor na ZFM | `810001` – Crédito presumido IBS ZFM |
| 05 – Transferência de Crédito na Sucessão | `800001` – Fusão, cisão ou incorporação |
| 01 – Multas e Juros, 03 e 06 (retorno por recusa) e 04 – Redução de Valores | Livre, desde que não seja um cClassTrib exclusivo de outro tipo de nota |

A conferência vale nos dois sentidos:

- **Tipo de nota que exige um cClassTrib (UB14-80):** numa Nota de Crédito 02 ou 05, todos os itens precisam usar o cClassTrib pareado com o tipo. Por exemplo, uma Nota de Crédito 02 com um item de cClassTrib diferente de `810001` é barrada.
- **cClassTrib exclusivo (UB14-60):** um cClassTrib da tabela só pode ser usado no tipo de nota com que está pareado. Por exemplo, o `810001` numa Nota de Crédito 04 é barrado. O `800001` é aceito tanto na Nota de Crédito 05 quanto na Nota de Débito 05.

Quando a nota é barrada, o erro aparece na lista **Mensagens de Erro**, com a linha, o item, o cClassTrib usado, a finalidade e o tipo da nota, o que é aceito e a regra da NT. Corrija o **Cód. Class. Trib. IBS/CBS** da linha ou o **Tipo Nota de Crédito** e registre (ou libere) o documento de novo. Se uma linha da matriz estiver barrando uma nota que deveria passar, o responsável fiscal pode desativá-la.

---

## Cenários para cada tipo de Nota de Crédito

### Tipo 01 - Multas e Juros:

Meu fornecedor recebeu valores de multa ou juros por atraso no pagamento que efetuei. Ele deve emitir uma nota fiscal de débito do tipo "04 – Multa e Juros". Essa nota serve para complementar a base de cálculo do IBS (Imposto sobre Bens e Serviços) referente ao fornecimento. <u>Se o fornecedor não emitir a nota, o comprador (minha empresa) pode corrigir essa falha emitindo uma nota fiscal de crédito do tipo "01 – Multa e Juros".</u> 

Neste cenário devo emitir uma **Fatura de compra** usando o tipo de nota de crédito **"01 – Multas e Juros"**. Como essa operação não representa uma compra efetiva de mercadoria ou serviço, é necessário utilizar um **Tipo de Operação:**

1. **Crie uma Fatura de Compra** para o fornecedor em questão.
2. Preencha as linhas da fatura com os valores de multa/juros pagos.
3. **Na aba Fiscal**, preencha o campo **Tipo de Operação** com uma opção que **não gere movimentação de estoque** (campo **Mov. Item** desmarcado). 
4. **Na aba Fiscal**, no campo **Finalidade de Emissão**, selecione **"Nota de Crédito"**.
5. **Nos Dados Fiscais do cabeçalho** (menu lateral > **Dados Fiscais**), preencha o campo **Tipo Nota de Crédito** com **"Multas e Juros"** (código 01).
6. **Abra os Textos Adicionais** do documento (ação **Texto Adicional** disponível no cabeçalho da Fatura de Compra). Nessa tela, informe a NF-e original do fornecedor:
   - **Nº Documento Referenciado** — selecione o documento fiscal original (se já existir no sistema) ou preencha manualmente os dados da nota do fornecedor.
   - **Chave Acesso NF-e** — será preenchida automaticamente ao selecionar o documento, ou informe manualmente a chave de 44 dígitos da NF-e original.

#### Resumo dos campos

| Onde              | Campo                                         | Valor                                |
| ----------------- | --------------------------------------------- | ------------------------------------ |
| Aba Fiscal        | Tipo de Operação                              | Código sem movimentação de estoque   |
| Aba Fiscal        | Finalidade de Emissão                         | Nota de Crédito                      |
| Dados Fiscais     | Tipo Nota de Crédito                          | Multas e Juros (01)                  |
| Textos Adicionais | Nº Documento Referenciado / Chave Acesso NF-e | Dados da NF-e original do fornecedor |

> **Importante:** O Tipo de Operação é essencial neste cenário porque como a Nota de Crédito de multas e juros não representa aquisição de mercadoria, a movimentação de estoque deve ser desativada.

---

### Tipo 02 -   Apropriação de crédito presumido de IBS sobre o saldo devedor na ZFM:

Cenário específico de empresas que fazem parte da Zona Franca de Manaus. 

> Por não se tratar de um contexto comum, não há indicação de procedimento. Caso sua empresa se enquadre nesta situação, pedimos que reporte através de nossa plataforma de suporte: https://network.arquiconsult.com/

### Tipo 03 - Retorno por recusa total na entrega ou não localização do destinatário:

A empresa vende a mercadoria e o cliente recusa, ou endereço não é localizado. 

Nessa situação, são necessários **dois documentos** no Business Central: um para o acerto financeiro/fiscal e outro para o retorno do item ao estoque.

---

#### Documento 1 — Nota de Crédito de Venda (acerto financeiro e fiscal)

Este documento registra que o cliente devolveu o item e que a empresa precisa estornar o valor e os impostos correspondentes. **Ele não movimenta estoque** — apenas ajusta o financeiro.

| Passo | O que fazer                                                 | Onde                                                         | O que preencher                                              |
| ----- | ----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1     | Criar uma **Nota de Crédito de Venda** com o item devolvido | Menu: Notas de Crédito de Venda                              | Tipo de documento fiscal: **NF-e**                           |
| 2     | Usar um tipo de operação **que não movimente item**         | Campo **Tipo de Operação** no cabeçalho                      | Escolher uma opção configurada para não gerar movimento de estoque |
| 3     | Definir a finalidade de emissão                             | Aba **Fiscal** → campo **Finalidade de Emissão**             | Selecionar **Nota de Crédito**                               |
| 4     | Informar o motivo da devolução                              | **Dados Fiscais > Linha NF-e > Abrir** → campo **Tipo Nota de Crédito** | Selecionar **Retorno por recusa total na entrega ou não localização do destinatário (03)** |
| 5     | Referenciar a nota fiscal do item que está sendo devolvido  | Ação **Texto Adicional** no documento                        | Informar a referência à nota fiscal original (número e chave de acesso) para que conste no documento fiscal emitido |

#### Documento 2 — Devolução de Venda (retorno do item ao estoque)

A Nota de Crédito acima cuidou do acerto financeiro, mas o item voltou fisicamente para a empresa. Para que o estoque reflita essa entrada, é preciso criar um segundo documento:

| Passo | O que fazer                                                | Onde                                    | O que preencher                                              |
| ----- | ---------------------------------------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| 1     | Criar uma **Devolução de Venda** com o item devolvido      | Menu: Devoluções de Venda               | Aba **Fiscal** → campo **Finalidade de Emissão**: selecionar **Devolução** |
| 2     | Usar um tipo de operação **que movimente somente item**    | Campo **Tipo de Operação** no cabeçalho | Escolher uma opção configurada para gerar apenas movimento de estoque, sem efeito financeiro |
| 3     | Referenciar a nota fiscal do item que está sendo devolvido | Ação **Texto Adicional** no documento   | Informar a referência à nota fiscal original (número e chave de acesso) para que conste no documento fiscal emitido |

Ao registrar esse documento, o item retorna ao estoque e fica disponível novamente.

#### Resumo dos dois documentos

| Documento                    | Finalidade                  | Movimenta estoque? | Movimenta financeiro? | Referencia a nota original? |
| ---------------------------- | --------------------------- | ------------------ | --------------------- | --------------------------- |
| **Nota de Crédito de Venda** | Estornar valor e impostos   | Não                | Sim                   | Sim — via Texto Adicional   |
| **Devolução de Venda**       | Devolver o cinto ao estoque | Sim                | Não                   | Sim — via Texto Adicional   |

### Tipo 04 - Ajuste para reduzir valores de uma operação já documentada:

Essa modalidade da nota de crédito é utilizada para diminuir o valor do imposto destacado em uma nota fiscal original emitida com erro a maior (valor, alíquota, base de cálculo). Neste caso:

1. Crie uma **Nota Crédito Venda** com o cliente da NF-e original.

2. Na aba **Fiscal**, preencha:

   | Campo                  | Valor                                                        |
   | ---------------------- | ------------------------------------------------------------ |
   | **Finalidade Emissão** | **Nota de Crédito**                                          |
   | **NF Referenciada**    | Nº da fatura de venda registrada original (a NF-e emitida com erro) |
   | **Tipo de Operação**   | Tipo de operação configurado **sem Mov. Item** (somente financeiro) |

3. Acesse os **Dados Fiscais** (menu lateral direito > Linha NF-e > Abrir) e no campo **Tipo Nota de Crédito** selecione **T04 — Redução de Valores**.

4. Inclua o nota referenciada em **Texto Adicional NF** (disponível no menu de ações do documento) 

5. Nas **linhas**, altere o valor do item, ou a quantidade, de forma que o valor da redução que precisa ser demonstrado para SEFAZ fique correspondente. 

   > #### Exemplo simples:
   >
   > ##### NF original
   >
   > Venda:
   >
   > - 100 unidades - R$ 10.000 - IBS R$ 2.500. Depois: percebe-se que somente 80 unidades deveriam ter sido cobradas.
   >
   > Então, 
   >
   > ##### NF Crédito Tipo 04, reduz:
   >
   > - 20 unidades;
   > - R$ 2.000;
   > - IBS proporcional.



### Tipo 05 - Transferência de crédito de sucessão:

Quando uma empresa é **fundida, cindida ou incorporada**, ela pode possuir créditos de IBS/CBS ainda não utilizados. O procedimento normal seria a própria empresa **sucedida** transferir esses créditos para a **sucessora** por meio de uma Nota de Débito. Porém, se o CNPJ da sucedida já estiver **baixado ou inapto**, ela não consegue mais emitir documentos fiscais.

Nesse caso, a **empresa sucessora** emite a **Nota de Crédito Tipo 05**, indicando a sucedida como destinatária. 

> Por não se tratar de um contexto comum, não há indicação de procedimento. Caso sua empresa se enquadre nesta situação, pedimos que reporte através de nossa plataforma de suporte: https://network.arquiconsult.com/
>
> **Importante:** O crédito transferido só passa a valer na apuração após a manifestação favorável das demais sucessoras (se houver) e do fisco. Esses eventos de manifestação ainda não estão implementados no sistema.

### Tipo 06 - Retorno por recusa parcial na entrega:

A empresa vende um sapato e uma mochila para um cliente. No momento da entrega, o cliente percebe que recebeu um sapato e um cinto — o cinto foi enviado por engano no lugar da mochila. O cliente recusa o cinto e fica apenas com o sapato, devolvendo o item incorreto.

Nessa situação, são necessários **dois documentos** no Business Central: um para o acerto financeiro/fiscal e outro para o retorno do item ao estoque.

---

#### Documento 1 — Nota de Crédito de Venda (acerto financeiro e fiscal)

Este documento registra que o cliente devolveu o cinto e que a empresa precisa estornar o valor e os impostos correspondentes. **Ele não movimenta estoque** — apenas ajusta o financeiro.

| Passo | O que fazer                                                  | Onde                                                         | O que preencher                                              |
| ----- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1     | Criar uma **Nota de Crédito de Venda** com o item devolvido (o cinto) | Menu: Notas de Crédito de Venda                              | Tipo de documento fiscal: **NF-e**                           |
| 2     | Usar um tipo de operação **que não movimente item**          | Campo **Tipo de Operação** no cabeçalho                      | Escolher uma opção configurada para não gerar movimento de estoque |
| 3     | Definir a finalidade de emissão                              | Aba **Fiscal** → campo **Finalidade de Emissão**             | Selecionar **Nota de Crédito**                               |
| 4     | Informar o motivo da devolução                               | **Dados Fiscais > Linha NF-e > Abrir** → campo **Tipo Nota de Crédito** | Selecionar **Retorno por recusa parcial na entrega (06)**    |
| 5     | Referenciar a nota fiscal original **por item**              | **Dados Fiscais > Linha NF-e > Abrir** → campos **Nº Doc. Fiscal Ref.**, **Chave Acesso Ref.** e **Número Item Ref.** | No campo **Nº Doc. Fiscal Ref.**, selecione o documento fiscal da venda original. A **Chave Acesso Ref.** será preenchida automaticamente. Em seguida, no campo **Número Item Ref.**, selecione o item correspondente da nota original (o cinto). Isso deve ser feito **em cada linha** da nota de crédito |

> **Por que "recusa parcial"?** Porque o cliente aceitou parte dos itens (o sapato) e recusou apenas o cinto. Se o cliente tivesse recusado **todos** os itens, a opção correta seria *"Retorno por recusa total na entrega ou por não localização do destinatário na tentativa de entrega"*.

---

#### Documento 2 — Devolução de Venda (retorno do item ao estoque)

A Nota de Crédito acima cuidou do acerto financeiro, mas o cinto voltou fisicamente para a empresa. Para que o estoque reflita essa entrada, é preciso criar um segundo documento:

| Passo | O que fazer                                                  | Onde                                    | O que preencher                                              |
| ----- | ------------------------------------------------------------ | --------------------------------------- | ------------------------------------------------------------ |
| 1     | Criar uma **Devolução de Venda** com o item devolvido (o cinto) | Menu: Devoluções de Venda               | Aba **Fiscal** → campo **Finalidade de Emissão**: selecionar **Devolução** |
| 2     | Usar um tipo de operação **que movimente somente item**      | Campo **Tipo de Operação** no cabeçalho | Escolher uma opção configurada para gerar apenas movimento de estoque, sem efeito financeiro |
| 3     | Referenciar a nota fiscal do item que está sendo devolvido   | Ação **Texto Adicional** no documento   | Informar a referência à nota fiscal original (número e chave de acesso) para que conste no documento fiscal emitido |

Ao registrar esse documento, o cinto retorna ao estoque e fica disponível novamente.

#### Resumo dos dois documentos

| Documento                    | Finalidade                         | Movimenta estoque? | Movimenta financeiro? | Referencia a nota original?                |
| ---------------------------- | ---------------------------------- | ------------------ | --------------------- | ------------------------------------------ |
| **Nota de Crédito de Venda** | Estornar valor e impostos do cinto | Não                | Sim                   | Sim — por item, nos Dados Fiscais da linha |
| **Devolução de Venda**       | Devolver o cinto ao estoque        | Sim                | Não                   | Sim — via Texto Adicional                  |

---
