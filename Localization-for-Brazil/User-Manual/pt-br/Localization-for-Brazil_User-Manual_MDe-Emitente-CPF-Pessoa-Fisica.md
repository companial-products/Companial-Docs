<!-- PR: #279 -->
# MD-e: Emitente Pessoa Física (CPF) — Localização do Cliente e do Fornecedor

**Data**: Outubro/2026

---

## Sumário

A **Manifestação de Destinatário NF-e** e a importação de XML de NF-e passam a encontrar o cliente ou o fornecedor pessoa física pelo CPF do emitente, como já acontecia com o CNPJ. Antes, uma nota emitida por pessoa física chegava com o **Nº Participante** em branco, e informar o cadastro à mão dava erro, embora o CPF fosse o mesmo. Também mudaram a ação **Criar Fornecedor**, que passa a criar o fornecedor como **Pessoa Física** e com um modelo próprio, e a escolha entre cadastros com o mesmo documento, que agora usa a Inscrição Estadual da nota. As MD-e já gravadas são acertadas pela atualização da versão.

---

## 1. Contexto

### O problema

Uma NF-e pode ser emitida por pessoa física, identificada por CPF em vez de CNPJ. Isso acontece, por exemplo, na devolução de venda feita por um cliente pessoa física ou na compra de um produtor rural.

O **Cartão Cliente** e o **Cartão Fornecedor** guardam o **CPF/CNPJ** sempre com máscara (`369.752.330-98`). Até esta versão, a MD-e guardava o CPF do emitente só com dígitos (`36975233098`). O sistema compara o texto exato, então os dois nunca coincidiam:

- o **Nº Participante** da MD-e ficava em branco, mesmo com o cliente ou o fornecedor já cadastrado;
- informar o cadastro à mão no **Nº Participante** era recusado, como se o cliente não existisse, e a lista do campo vinha vazia;
- a importação do XML para documento de compra, venda ou fiscal também não encontrava o participante;
- a ação **Criar Fornecedor** tratava todo emitente como **Pessoa Jurídica**: não encontrava o fornecedor pessoa física existente e criava outro, com a categoria errada.

Com emitente CNPJ, os dois lados já ficavam formatados e o problema não acontecia.

### A solução

O documento do emitente passa a ser gravado e procurado **no mesmo formato do cadastro**, seja CPF ou CNPJ (numérico ou alfanumérico, conforme a [NT 2026.004](Localization-for-Brazil_User-Manual_CNPJ-Alfanumerico.md)). O formato do cadastro de clientes e fornecedores não mudou.

---

## 2. Configuração

A localização do participante pelo CPF não exige configuração.

Para usar a ação **Criar Fornecedor** com emitente pessoa física, informe o modelo do fornecedor:

1. Navegue até **Conf. Importação NF-e** (página `NFeImportationSetupFNX`), grupo **Cadastros**.
2. No campo **Cód. Modelo Fornecedor Pessoa Física**, escolha o modelo de configuração usado para criar fornecedores pessoa física a partir da MD-e.
3. O campo **Cód. Modelo Fornecedor**, logo acima, continua valendo para emitente identificado por CNPJ.

Os dois modelos existem separados porque o fornecedor pessoa física costuma ter grupos contábeis, retenções e marcação de **Produtor Rural** diferentes dos de uma pessoa jurídica. Tudo isso vem do modelo escolhido.

> **Importante**: com o **Cód. Modelo Fornecedor Pessoa Física** em branco, o sistema **não usa** o modelo de pessoa jurídica. Ele abre a lista de modelos de fornecedor para você escolher na hora (veja a [seção 5](#5-criar-fornecedor-com-emitente-pessoa-física)).

---

## 3. Localização do Participante pelo CPF

### Na MD-e

O CPF do emitente é gravado no campo **CNPJ** da **Manifestação de Destinatário NF-e** já com a máscara, nos dois momentos em que a nota chega:

- **no resumo da nota**, recebido na consulta de documentos à SEFAZ (por exemplo, pela ação **Consultar Documentos**);
- **no XML completo da nota**, recebido depois da manifestação ou por **Importar XML Manualmente**.

Com o documento no formato do cadastro, a MD-e se comporta com emitente CPF do mesmo jeito que com emitente CNPJ:

- o **Nº Participante** é preenchido automaticamente quando o cliente (em nota de devolução) ou o fornecedor (em nota de compra) está cadastrado com aquele CPF;
- informar o cadastro à mão no **Nº Participante** é aceito;
- a lista do campo **Nº Participante** mostra os cadastros com o mesmo CPF do emitente.

Se o resumo da nota chegar sem o documento do emitente, o CPF é gravado quando o XML completo for recebido.

Quando a SEFAZ envia um evento de cancelamento sem o CNPJ do autor, o que acontece quando ele é pessoa física, o documento já gravado na MD-e é mantido. Antes, esse evento apagava o CPF.

### Na importação de XML para documentos

A importação do XML de NF-e para documento de compra, documento de venda ou documento fiscal, por exemplo pela ação **Importar Arquivo XML da NF-e**, também passa a encontrar o cliente ou o fornecedor pessoa física pelo CPF do emitente.

### Exemplo prático

| Onde | Antes | Agora |
| --- | --- | --- |
| **CPF/CNPJ** no **Cartão Cliente** | `369.752.330-98` | `369.752.330-98` (sem alteração) |
| **CNPJ** na MD-e | `36975233098` | `369.752.330-98` |
| **Nº Participante** na MD-e de devolução | em branco | cliente encontrado pelo CPF |
| Informar o cliente à mão | erro: o cliente não existe | aceito |

---

## 4. Mais de um Cadastro com o Mesmo Documento

Um mesmo CPF ou CNPJ pode estar em mais de um cadastro. O caso mais comum é o do produtor rural com várias propriedades, cada uma com a sua **Inscrição Estadual**. Outro caso são os clientes que compartilham o CNPJ (veja [Cadastro de Clientes: Mais de um Cadastro com o Mesmo CNPJ](Localization-for-Brazil_User-Manual_Cliente-Multiplos-Cadastros-Mesmo-CNPJ.md)).

Nesses casos, o sistema compara a Inscrição Estadual do emitente na nota com a **Inscrição Estadual** de cada cadastro encontrado, considerando só os dígitos:

| Situação | Cliente | Fornecedor |
| --- | --- | --- |
| Exatamente um cadastro com a mesma IE | Esse cliente é o participante | Esse fornecedor é o participante |
| Nenhum ou mais de um cadastro com a mesma IE | **Nº Participante** em branco, com aviso | O primeiro fornecedor encontrado, como antes |

Quando o cliente fica em branco, a mensagem é:

> *O CNPJ/CPF 369.752.330-98 do emitente está cadastrado em 2 clientes, portanto o participante ficou em branco. Escolha o cliente no campo Nº Participante antes de criar o documento.*

Nota sem Inscrição Estadual, ou com IE `ISENTO`, não serve para desempatar.

A mesma regra vale com emitente CNPJ. O resultado muda apenas quando há mais de um cadastro com o mesmo documento e a IE da nota identifica um deles.

---

## 5. Criar Fornecedor com Emitente Pessoa Física

Na **Manifestação de Destinatário NF-e** (página `NFeMDeCardFNX`), a ação **Criar Fornecedor** fica disponível enquanto o **Nº Participante** está em branco. Ela identifica o tipo do emitente pelo documento: CPF é **Pessoa Física**, CNPJ é **Pessoa Jurídica**.

### Fornecedor já cadastrado

O sistema procura um fornecedor com a **Categoria** e o **CPF/CNPJ** do emitente. Se encontrar, pergunta:

> *Já existe um fornecedor cadastrado com o CNPJ/CPF 369.752.330-98. Nº: F000001 Nome: … Deseja utilizar esse fornecedor?*

- **Sim**: o fornecedor existente vira o **Nº Participante** da MD-e e nenhum fornecedor é criado.
- **Não**: nada é criado.

Se houver mais de um fornecedor com o mesmo documento, o sistema oferece o que tem a mesma Inscrição Estadual da nota, como na [seção 4](#4-mais-de-um-cadastro-com-o-mesmo-documento).

### Fornecedor novo

Se nenhum fornecedor for encontrado, o sistema cria um novo:

| | Emitente CPF | Emitente CNPJ |
| --- | --- | --- |
| Modelo | **Cód. Modelo Fornecedor Pessoa Física** | **Cód. Modelo Fornecedor** |
| Modelo em branco | Abre a lista de modelos de fornecedor | Abre a lista de modelos de fornecedor |
| **Categoria** | **Pessoa Física** | **Pessoa Jurídica** |
| **CPF/CNPJ** | CPF com máscara | CNPJ com máscara |

O nome, o endereço e a **Inscrição Estadual** vêm da nota. Para emitente pessoa física, a **Categoria** é aplicada depois do modelo. Assim, mesmo que se escolha um modelo de pessoa jurídica na lista, o fornecedor fica como **Pessoa Física** e os campos que só valem para pessoa jurídica são limpos.

### MD-e sem documento do emitente

Se o campo **CNPJ** da MD-e estiver em branco, por exemplo num resumo que chegou sem o documento, a ação **Criar Fornecedor** para com um erro que pede o **CNPJ**. Nenhum fornecedor é procurado nem criado. Isso evita casar a nota com qualquer fornecedor que também esteja sem documento.

### Processos sem interface

Quando a criação do fornecedor roda sem um usuário para responder, por exemplo numa fila de tarefas, numa API ou numa customização:

- se já existe fornecedor com o documento do emitente, ele é usado sem pergunta;
- se o modelo do tipo do emitente estiver em branco, o processo para com a mensagem *Preencha o campo Cód. Modelo Fornecedor Pessoa Física em Conf. Importação NF-e para criar o fornecedor de uma MD-e sem interação do usuário.* (ou **Cód. Modelo Fornecedor**, para emitente CNPJ), e nenhum fornecedor é criado.

---

## 6. MD-e Já Gravadas

Na atualização para esta versão, o sistema formata o CPF das MD-e já gravadas que tinham o documento só com dígitos, **inclusive as arquivadas**. A partir daí, elas aceitam o cliente ou o fornecedor pessoa física no **Nº Participante**, sem reimportar nota por nota.

- A atualização roda uma única vez por empresa.
- MD-e de emitente CNPJ não são alteradas.
- O **Nº Participante** dessas MD-e não é preenchido pela atualização. Informe o participante na MD-e: a lista do campo já mostra os cadastros com aquele CPF.

---

## 7. Resumo dos Campos Novos

### Tabela Conf. Importação NF-e (`NFeImportationSetupFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Cód. Modelo Fornecedor Pessoa Física** (`MDePersonVendorTemplateCode`) | Código (10) | Modelo de configuração usado pela ação **Criar Fornecedor** da MD-e quando o emitente é pessoa física, identificado por CPF. Em branco, o modelo é escolhido na hora da criação |

O ToolTip do campo **Cód. Modelo Fornecedor** (`MDeVendorTemplateCode`) agora informa que ele vale para emitente identificado por CNPJ.

---

## 8. Pontos de Atenção

- **Revise o Nº Participante das MD-e antigas de pessoa física.** A atualização acerta o CPF, mas não preenche o participante. As notas de pessoa física que estavam paradas por esse motivo precisam do participante informado antes de criar o documento.
- **Customizações que comparam o documento do emitente.** Depois da atualização, o campo **CNPJ** das MD-e de pessoa física passa a ter máscara. Uma customização que compare esse campo com o CPF só em dígitos precisa ser revista.
- **Produtor rural com uma propriedade nova.** Se já existe fornecedor com o CPF, mas nenhum tem a IE da nota, **Criar Fornecedor** oferece o fornecedor existente. Ao responder **Não**, nada é criado: cadastre a propriedade nova pelo **Cartão Fornecedor**.
- **Fora desta entrega: CT-e.** A criação de fornecedor a partir do CT-e continua tratando o emitente como pessoa jurídica.
