<!-- Branch: feature/Fazendas-IE -->
# Destinatário Fiscal: I.E. por Endereço de Entrega na NF-e

**Data**: Agosto/2026

---

## Sumário

Um mesmo cliente pode ter mais de uma Inscrição Estadual. O produtor rural tem um único CPF e uma I.E. por propriedade, e uma empresa pode ter uma I.E. por estabelecimento. Até agora a NF-e sempre levava a I.E. cadastrada no cliente, independentemente do local que recebia a mercadoria, e por isso a nota saía com a I.E. errada sempre que a venda não era para o endereço principal.

Agora é possível marcar cada Endereço de Entrega como **Destinatário Fiscal**, com I.E. própria. Quando a ordem de venda usa um endereço marcado, a nota é emitida para aquele endereço, com a I.E. e o endereço dele, e o SPED reproduz exatamente a mesma informação.

---

## 1. Contexto

### O problema

Uma pessoa física produtora rural é um único cliente no Business Central, com um único CPF. Mas cada fazenda que ela explora é um estabelecimento distinto perante o fisco estadual e tem sua própria Inscrição Estadual. O mesmo vale para uma empresa com mais de uma inscrição sob o **mesmo CNPJ**, como a inscrição de substituto tributário em outro estado.

Como o cadastro do cliente comporta apenas uma I.E., toda nota emitida para esse cliente levava a mesma inscrição, a do estabelecimento cadastrado no cliente. Vender para o segundo ou o terceiro local produzia uma nota com a I.E. de outro estabelecimento.

### A solução

O Endereço de Entrega passa a poder ser identificado como o **destinatário fiscal** dos documentos entregues nele, com I.E. própria. A partir daí o sistema resolve sozinho, no registro do documento: se a venda é entregue naquele endereço, é ele que figura como destinatário da nota.

### Quando se aplica

Somente quando as **duas condições** abaixo são verdadeiras ao mesmo tempo:

1. O Endereço de Entrega usado na ordem está marcado como **Destinatário Fiscal**;
2. Esse Endereço de Entrega tem **I.E.** preenchida.

Faltando qualquer uma delas, nada muda: a nota continua sendo emitida com os dados do cliente, exatamente como antes. Não há configuração para ligar ou desligar a funcionalidade. Quem não marcar nenhum endereço não é afetado.

---

## 2. Configuração

### Passo 1 — Cadastrar cada local como Endereço de Entrega

Para cada estabelecimento do cliente, crie um Endereço de Entrega em **Envio-a Endereço** (página `Ship-to Address`), acessível pelo Cartão Cliente. Preencha o endereço completo:

| Campo | Observação |
|-------|------------|
| **Envio-a Endereço** | Logradouro do estabelecimento |
| **Número** | Número do logradouro |
| **Bairro** | Bairro ou distrito |
| **Cidade** e **CEP** | Município do estabelecimento |
| **Cód. UF** | Unidade federativa do estabelecimento |
| **I.E.** | Inscrição Estadual **daquele estabelecimento** |

### Passo 2 — Marcar o endereço como destinatário fiscal

Ainda no Endereço de Entrega, marque o campo **Destinatário Fiscal**.

Ao marcar, o sistema valida na hora que **Cód. UF**, **I.E.**, **Envio-a Endereço**, **Cidade** e **CEP** estão preenchidos.

Faltando qualquer um desses dados, a marcação é recusada com uma mensagem indicando o campo pendente. Isso garante que a nota nunca saia com destinatário incompleto.

Depois de marcado, a **I.E.** não pode mais ser apagada enquanto a marcação estiver ativa.

O campo **CPF/CNPJ** do endereço fica desabilitado no endereço marcado, e a marcação é recusada se ele estiver preenchido. A funcionalidade troca apenas a I.E. e o endereço. O documento continua saindo com o CPF/CNPJ do cadastro do cliente, porque o endereço é outro estabelecimento do **mesmo** contribuinte. Um estabelecimento com CNPJ próprio é outro contribuinte e deve ser cadastrado como **cliente separado**. Ver [Pontos de Atenção](#8-pontos-de-atenção).

O campo também aparece na **Lista de Endereço de Envio** (página `Ship-to Address List`), o que permite conferir rapidamente quais endereços de um cliente são destinatários fiscais.

---

## 3. Emissão da NF-e

Na ordem de venda, basta informar o **Cód. Endereço Envio** correspondente ao local que vai receber a mercadoria. Não há nenhuma ação adicional.

Ao informar um endereço marcado como Destinatário Fiscal, o pedido passa a ser calculado pela UF daquele endereço. A **UF**, o **município** e o **CEP** do destinatário no pedido vêm do endereço de entrega, a **Situação do Cliente** é reavaliada (Mesmo estado / Outro estado) e a situação é repassada às linhas. Assim CFOP, alíquota de ICMS, FCP e os CSTs por NCM já são determinados para o destino real. Se o endereço for desmarcado, ou o **Cód. Endereço Envio** for apagado, a UF do cliente volta a valer.

Ao registrar o documento, o destinatário do Documento Fiscal passa a ser o endereço de entrega:

| Dado do destinatário | Origem |
|----------------------|--------|
| I.E. | **Endereço de entrega** |
| Logradouro, número, complemento, bairro | **Endereço de entrega** |
| Cidade, CEP, UF, país | **Endereço de entrega** |
| Nome | Cliente |
| CPF/CNPJ | Cliente |
| Categoria do participante | Cliente |

O nome e o CPF/CNPJ continuam sendo os do cliente, porque é a mesma pessoa jurídica ou física. Muda o estabelecimento.

Esses dados vão para o XML da NF-e nos campos do grupo de destinatário (`dest`): `destIE`, `destxLgr`, `destnro`, `destxCpl`, `destxBairro`, `destxMun`, `destcMun`, `destCEP` e `destUF`.

### Grupo de entrega

Quando o destinatário já é o endereço de entrega, o grupo de entrega (`entrega`) deixa de ser gerado para esse endereço, mesmo que a opção **Criar Tag de Endereço de Entrega** esteja ligada na configuração da NF-e. O grupo apenas repetiria as mesmas informações que já estão no destinatário.

Endereços de entrega que não são destinatários fiscais continuam gerando o grupo normalmente.

### Endereço em outra UF

Endereço de entrega em UF diferente da do cliente é suportado. Como a UF do endereço passa a reger o cálculo do pedido (seção acima), a venda sai como operação interestadual: CFOP 6xxx, alíquota interestadual e FCP conforme o destino.

Ainda existe uma verificação no registro, para o caso de o documento ter sido calculado para outra UF. Isso acontece, por exemplo, com um pedido criado antes de o endereço ser marcado como Destinatário Fiscal, ou copiado de outro documento:

> *O endereço de entrega `<código>` está no estado `<UF>`, mas este documento foi calculado para o estado `<UF>` (cliente `<nº>`). Reinforme o Cód. Endereço Envio no documento para que o CFOP e os impostos sejam calculados para o estado `<UF>` e registre novamente.*

Reinformar o **Cód. Endereço Envio** no pedido recalcula a UF, o CFOP e os impostos, e o registro passa. A verificação existe porque emitir uma nota com CFOP de operação interna e destinatário em outra UF resulta em rejeição pela SEFAZ e imposto incorreto.

---

## 4. Exemplo Prático

O produtor **João da Silva**, CPF `123.456.789-00`, explora duas fazendas em Goiás:

| Cód. Endereço Envio | Propriedade | Cidade/UF | I.E. |
|---------------------|-------------|-----------|------|
| `FAZ-BVISTA` | Fazenda Boa Vista | Rio Verde/GO | `10.987.654-3` |
| `FAZ-STAFE` | Fazenda Santa Fé | Jataí/GO | `11.223.344-5` |

Ambos os endereços estão marcados como **Destinatário Fiscal**.

Ao registrar uma venda de fertilizante com **Cód. Endereço Envio** = `FAZ-STAFE`, a NF-e é emitida com:

- `destxNome` = João da Silva
- `destCPF` = 12345678900
- `destIE` = **11223344 5** (a I.E. da Fazenda Santa Fé)
- `destxMun` / `destUF` = **Jataí / GO**

Uma segunda venda, agora com `FAZ-BVISTA`, sai com a I.E. `10.987.654-3` e o município de Rio Verde. Mesmo produtor, mesmo CPF, dois estabelecimentos distintos, cada nota com a inscrição correta.

O mesmo raciocínio vale para um cliente pessoa jurídica com mais de um estabelecimento inscrito no mesmo estado.

---

## 5. Reflexo no SPED

Os arquivos do **EFD ICMS/IPI** e do **EFD Contribuições** acompanham automaticamente. Isso não é opcional nem exige nenhuma ação: a mesma regra que decide a I.E. da nota decide o que vai para os livros, de modo que os dois nunca divergem.

### Registro 0150 — Cadastro do Participante

Cada endereço marcado como destinatário fiscal passa a ser um **participante próprio** no registro 0150, com seu próprio código (COD_PART), sua I.E., seu endereço e seu código de município. Um cliente com dois estabelecimentos movimentados no período gera dois registros 0150.

O código do participante do endereço é derivado: é o nº do cliente seguido do cód. do endereço de entrega, separados por hífen. O cliente `C000101` com o endereço `FAZ-STAFE` gera o COD_PART `C000101-FAZ-STAFE`. Não há nada a configurar nem a manter.

O campo COD_PART do layout do EFD aceita 60 caracteres, então o código derivado cabe sem truncar nem o nº do cliente nem o cód. do endereço. Como o hífen é um caractere válido nos dois, em bases que numeram clientes com hífen o código derivado ainda pode coincidir com o nº de outro cliente, com o de um fornecedor ou com o código de uma filial. Se isso acontecer, a geração da apuração é interrompida com uma mensagem que aponta os dois participantes em conflito, para que a numeração seja ajustada antes de o arquivo ir ao Fisco. A alternativa seria os dois compartilharem um único registro 0150.

Clientes sem endereço marcado mantêm exatamente o código de sempre, o próprio nº do cliente, de modo que apurações já transmitidas continuam sendo reproduzidas de forma idêntica.

### Registro C100 e demais registros de documento

Cada documento aponta para o código do participante do endereço que o recebeu, e não mais para o código do cliente. O vínculo entre a nota e o cadastro do participante fica correto para cada estabelecimento.

### Onde conferir

Na página **Participantes EFD** (página `EFDParticipants`), dois campos adicionais podem ser exibidos pela personalização da página:

- **Nº de Origem**: o cliente a partir do qual o participante foi montado;
- **Cód. Endereço de Entrega**: o endereço que aquele participante representa (em branco quando o participante é o próprio cliente).

Eles permitem identificar rapidamente quais participantes do arquivo são endereços de entrega e a qual cliente pertencem.

---

## 6. Documento Fiscal Registrado

A ação **Atualizar Dados do Participante**, no **Documento Fiscal Registrado** (página `PostedFiscalDocumentFNX`), relê o cadastro do cliente para atualizar os dados do participante no documento. Ela preserva o destinatário fiscal: quando o documento foi entregue em um endereço marcado como Destinatário Fiscal, a releitura do participante já traz a I.E. e o endereço daquele endereço de entrega. A **Situação do Cliente/Fornecedor** do documento (Mesmo estado / Outro estado) é recalculada junto, e é ela que vai para o campo `idDest` do XML.

Sem isso, executar a ação em um documento emitido para uma fazenda ou filial substituiria os dados pela sede do cliente.

No **Documento Fiscal** (página `FiscalDocumentFNX`), antes do registro, informar, trocar ou limpar o **Cód. Endereço Envio** aplica ou desfaz o destinatário fiscal na hora. Informar um endereço marcado traz a I.E., o endereço e a UF daquele endereço. Limpar o campo, ou apontá-lo para um endereço que não está marcado, devolve os dados do cliente. Nos dois casos a **Situação do Cliente/Fornecedor** é recalculada e repassada às linhas. O lookup do campo passa a oferecer apenas os endereços de entrega do próprio cliente do documento.

---

## 7. Resumo dos Campos Novos

### Tabela Envio-a Endereço (`Ship-to Address`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Destinatário Fiscal** (`FiscalRecipientFNX`) | Booleano | Identifica o endereço como destinatário fiscal dos documentos entregues nele, com I.E. própria. Exige endereço completo com **I.E.** e desabilita o **CPF/CNPJ** do endereço, que continua vindo do cadastro do cliente. |

### Tabela Participante (`EFDParticipant`)

Substitui a tabela `EFDParticipantFNX`, cujo **Nº** comportava apenas 20 caracteres, insuficiente para o código derivado. A tabela antiga fica marcada como obsoleta, e seus registros são copiados na atualização, um a um, sem alterar nenhum código. Uma apuração antiga reprocessada gera o mesmo arquivo de antes.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Nº** (`ParticipantNo`) | Código (60) | COD_PART do participante. É o nº do cliente ou do fornecedor, ou o nº do cliente seguido do cód. do endereço de entrega. |
| **Nº de Origem** (`SourceNo`) | Código (20) | Cliente ou fornecedor a partir do qual o participante foi montado. |
| **Cód. Endereço de Entrega** (`ShipToCode`) | Código (10) | Endereço de entrega que o participante representa. Em branco quando o participante é o próprio cliente ou fornecedor. |

### Tabela Documento (`SettlementDocumentFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Código do Participante no EFD** (`EFDParticipantCode`) | Código (60) | Código do participante (COD_PART) informado para o documento nos arquivos do EFD. |
| **Cód. Endereço de Entrega do Participante** (`ParticipantShipToCode`) | Código (10) | Endereço de entrega que foi o destinatário fiscal do documento. |

---

## 8. Pontos de Atenção

- **Compras** não estão contempladas. A funcionalidade atende ao ciclo de vendas. Documentos fiscais de compra continuam usando a I.E. do cadastro do fornecedor.
- **Endereço em UF diferente da do cliente** passa a reger o cálculo do pedido (UF, município, CEP e situação do cliente), conforme a seção 3. Pedidos criados antes de o endereço ser marcado precisam ter o **Cód. Endereço Envio** reinformado para recalcular CFOP e impostos.
- **CST por Exceção Fiscal** (`Cód. Exceção Fiscal` na linha) ainda é determinado pela UF do **Endereço de Cobrança**, não pela do Destinatário Fiscal. Se você usa exceção fiscal em vendas para destinatário fiscal em outra UF, confira o CST resultante.
- **Alteração do cadastro após o faturamento**: a informação do SPED é derivada do cadastro atual do Endereço de Entrega. Desmarcar **Destinatário Fiscal** ou alterar a **I.E.** de um endereço depois de faturar faz com que uma reapuração do período gere dados diferentes dos que foram transmitidos na nota. Trate esses cadastros como estáveis após o primeiro faturamento.
- **Contribuinte e isento**: a indicação de contribuinte de ICMS continua vindo do cadastro do cliente. Se estabelecimentos do mesmo cliente tiverem situações diferentes quanto à contribuição de ICMS, avalie o caso com a equipe de suporte.
- **Estabelecimento com CNPJ próprio não é destinatário fiscal**. A funcionalidade cobre um único contribuinte com mais de uma I.E.: o produtor rural com várias fazendas sob um CPF, ou a empresa com mais de uma inscrição sob o mesmo CNPJ. Filial com CNPJ próprio (matriz terminando em `0001`, filial em `0002`) é outro contribuinte. A SEFAZ valida o par CNPJ × I.E. contra o cadastro de contribuintes, e emitir com o CNPJ da matriz e a I.E. da filial tende a ser rejeitado. Cadastre cada CNPJ como um **cliente**, e use o Destinatário Fiscal apenas para as inscrições que compartilham o CNPJ desse cliente. É por isso que o campo **CPF/CNPJ** do endereço fica desabilitado quando ele é marcado.
