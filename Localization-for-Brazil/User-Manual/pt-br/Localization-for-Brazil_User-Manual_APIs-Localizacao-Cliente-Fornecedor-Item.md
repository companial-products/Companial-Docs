<!-- PR: #168 -->
# APIs da Localização Brasileira: Cliente, Fornecedor e Item

**Data**: Agosto/2026

---

## Sumário

A API padrão do Business Central publica a tabela **Cliente**, mas nenhum dos campos que a localização brasileira acrescenta a ela. O mesmo vale para fornecedor e item. Qualquer integração que precisasse ler ou gravar CPF/CNPJ, Inscrição Estadual, Cód. Área Imposto, Código NCM ou Cód. Origem dependia de desenvolvimento sob medida naquele ambiente.

Esta atualização inaugura as **APIs da localização**: 15 entidades publicadas dentro dos próprios apps distribuídos, sendo três de cadastro (cliente, fornecedor e item) e doze listas de códigos fiscais. Qualquer ambiente com a localização instalada passa a ter essas entidades disponíveis, sem instalação nem configuração adicional. Os cadastros são de **leitura e alteração**; as listas de códigos fiscais são de **consulta apenas**.

---

## 1. Contexto

Integrar um Business Central brasileiro sempre esbarrou no mesmo ponto: os dados da operação estão na API padrão da Microsoft, e os dados fiscais não estão em lugar nenhum. Um portal de cadastro de clientes, um fluxo de Power Automate, um middleware que sincroniza itens com o e-commerce, um robô que audita o cadastro antes do fechamento: todos conseguiam ler nome, endereço e condição de pagamento, e nenhum enxergava o CPF/CNPJ, a Inscrição Estadual ou o Código NCM.

A saída até aqui era publicar páginas como web service em cada ambiente, ou desenvolver uma extensão de API específica para aquele cliente. Os dois caminhos têm o mesmo defeito: só alcançam quem os montou. Cada ambiente ficava com um contrato diferente, mantido por quem o criou.

Esta atualização resolve isso da única forma que alcança todo mundo: as páginas de API vão dentro dos apps da localização que o cliente já tem instalados. Não há app novo para instalar, nem página para publicar como web service, nem configuração no ambiente. A partir da versão que traz esta atualização, as entidades existem.

Esta é a **primeira fase**. Ela cobre os três cadastros que sustentam a maior parte das integrações, cliente, fornecedor e item, mais as listas de códigos fiscais para as quais os campos desses cadastros apontam. Documento fiscal, conta contábil e as entidades dos apps de livros fiscais, bancário e pacote avançado ficam para fases seguintes.

---

## 2. Divisão de Trabalho com a API da Microsoft

As entidades de cadastro não substituem as entidades equivalentes da Microsoft: elas as complementam. O desenho é uma divisão explícita de trabalho:

| Etapa | Onde é feita |
| --- | --- |
| Criar o cliente, o fornecedor ou o item | API padrão da Microsoft, que tem a tabela inteira (endereço, condição de pagamento, grupos contábeis) |
| Complementar com os dados fiscais brasileiros | Entidades desta atualização |
| Excluir o registro | API padrão da Microsoft |

É por isso que as entidades de cadastro daqui **recusam a criação e a exclusão**: um cliente criado por elas nasceria sem endereço, sem condição de pagamento e sem grupos contábeis. Uma integração que cria cadastros continua chamando a API da Microsoft para criar o registro, e chama a API da localização em seguida para completá-lo.

Cada entidade traz apenas a identificação mínima, **Nº** e **Nome**, ambos somente leitura, mais os campos da localização. Ela não repete o que a API da Microsoft já publica para a mesma tabela. Há uma única exceção deliberada a essa regra, o par **Possui GTIN** / **GTIN**, explicada na seção 6.

---

## 3. Como Acessar

Todas as entidades ficam sob a mesma rota, com o grupo `brazil` reservado à localização brasileira inteira:

```
/api/arquibr/brazil/v1.0/<entidade>
```

São APIs REST/OData v4 do Business Central, como as da Microsoft: mesma autenticação, mesma paginação, mesmo suporte a `$filter`, `$select` e `$top`, e mesma exigência do cabeçalho de empresa quando o ambiente tem mais de uma. Uma leitura típica fica assim:

```
GET /api/arquibr/brazil/v1.0/companies({companyId})/customers?$filter=cnpjCpf eq ''
```

Alguns pontos que valem para todas as entidades:

- cada registro é identificado por um **Id** estável, o mesmo usado pela API padrão da Microsoft para aquele registro. É por ele que se referencia a linha em `PATCH`, e é o identificador que Power Automate e Power Apps exigem;
- as entidades de cadastro suportam **rastreamento de alterações**, o que permite a uma integração consumir apenas o que mudou desde a última leitura, em vez de varrer a tabela inteira;
- os nomes das entidades e dos campos são em inglês, porque fazem parte do endereço. As legendas e descrições saem em pt-BR, com o vocabulário fiscal já usado na tela (**Área Fiscal**, **Categoria do Participante**, **Código NCM**);
- as rotas desta atualização convivem com as de qualquer app de API anterior. Nenhuma desinstalação prévia é necessária.

---

## 4. Entidades de Cadastro

As três entidades abaixo aceitam **leitura** (`GET`) e **alteração** (`PATCH`). Criação (`POST`) e exclusão (`DELETE`) são recusadas.

Toda gravação percorre exatamente as mesmas críticas que a digitação na tela percorre. Uma combinação recusada na ficha é recusada na API, com a mesma mensagem.

### 4.1 Clientes — Dados Fiscais Brasileiros

Entidade `customers` (página `CustomersAPIv10`). Legenda: **Clientes - Dados Fiscais Brasileiros**.

| Campo na API | Campo no cadastro | Observação |
| --- | --- | --- |
| `id` | **Id** | Somente leitura |
| `number` | **Nº** | Somente leitura |
| `displayName` | **Nome** | Somente leitura |
| `territoryCode` | **Cód. UF** | |
| `participantCategoryCode` | **Cód. Categoria Participante** | |
| `category` | **Categoria** | Lista fechada |
| `companyNature` | **Natureza Pessoa Jurídica** | Lista fechada |
| `ruralProducer` | **Produtor Rural** | |
| `allowDuplicateCnpj` | **Permitir CNPJ Duplicado** | Somente com a funcionalidade habilitada em Conf. Vendas & Cobranças |
| `cnpjCpf` | **CPF/CNPJ** | |
| `ie` | **Inscrição Estadual** | |
| `ieValidated` | **I.E. Validado** | Somente leitura |
| `ccm` | **Inscrição Municipal** | |
| `taxAreaCode` | **Cód. Área Imposto** | |
| `district` | **Bairro** | |
| `addressNumber` | **Número** | |
| `suframaRegion` | **Região Suframa** | |
| `suframaCode` | **Cód. Suframa** | |
| `endUser` | **Consumidor Final** | |
| `notIcmsContributor` | **Não Contribuinte ICMS** | |
| `optingForSimples` | **Optante pelo Simples** | |
| `govEntityType` | **Tipo Ente Governamental** | Lista fechada |
| `nifCode` | **Cód. NIF** | |
| `nifNotInformedReason` | **Motivo da não informação do NIF** | Lista fechada. Gravar o `nifCode` apaga o motivo |
| `companyLink` | **Vínculo da Empresa** | Lista fechada |
| `emailNFe` | **E-Mail - NFe** | |
| `dropShippingCustomerType` | **Tipo Cliente Venda Triangular** | Lista fechada |

**Bairro** e **Número** merecem destaque: são obrigatórios na NF-e, e a API padrão da Microsoft não os publica. Ela publica só o logradouro, num campo único.

### 4.2 Fornecedores — Dados Fiscais Brasileiros

Entidade `vendors` (página `VendorsAPIv10`). Legenda: **Fornecedores - Dados Fiscais Brasileiros**.

O fornecedor tem a mesma espinha do cliente, de **Cód. UF** a **E-Mail - NFe**, sem os campos de Suframa e sem **Tipo Cliente Venda Triangular**, que não existem no cadastro de fornecedores. Além dela, expõe:

| Campo na API | Campo no cadastro | Observação |
| --- | --- | --- |
| `rpaRent` | **RPA/Aluguel** | Exige **Categoria** = *Pessoa Física* |
| `numberOfDependents` | **Número de Dependentes** | Exige **Categoria** = *Pessoa Física* |
| `elderly` | **Maior de 65 Anos** | Exige **Categoria** = *Pessoa Física* |
| `directImport` | **Importação Direta** | |
| `otherVendorNo` | **Nº Fornecedor Despachante** | |
| `exporterCode` | **Cód. exportador** | |
| `foreignManufacturerCode` | **Cód. Fabricante Estrangeiro** | |
| `transportMethod` | **Meio Transporte** | |

### 4.3 Itens — Classificação Fiscal Brasileira

Entidade `items` (página `ItemsAPIv10`). Legenda: **Itens - Classificação Fiscal Brasileira**.

| Campo na API | Campo no cadastro | Observação |
| --- | --- | --- |
| `id` | **Id** | Somente leitura |
| `number` | **Nº** | Somente leitura |
| `displayName` | **Nome** | Somente leitura (é a descrição do item) |
| `ncmCode` | **Código NCM** | |
| `ncmExceptionCode` | **Cód. Exceção NCM** | Precisa pertencer ao **Código NCM** do item |
| `cestCode` | **Cód. CEST** | |
| `icmsCstCode` | **Código CST ICMS** | Aceita apenas CST de ICMS |
| `originCode` | **Cód. Origem** | |
| `taxExceptionCode` | **Cód. Exceção Imposto** | |
| `serviceCode` | **Cód. Serviço** | |
| `anpCode` | **Cód. ANP** | |
| `anvisaCode` | **Cód. ANVISA** | Texto livre. Não há lista de códigos no sistema |
| `importCfopCode` | **Código CFOP (Importação)** | Aceita apenas CFOP de compra do exterior |
| `hasGtin` | **Possui GTIN** | Veja a seção 6 |
| `gtin` | **GTIN** | Veja a seção 6 |

Ficam de fora da entidade a **Info. Adicional NF-e** (conteúdo longo, não publicável como campo simples), a **Qtd. Reservada em Requisição** (campo calculado) e o **Nº Ativo Fixo**, que não é classificação fiscal e cuja gravação altera o custo do item.

---

## 5. A Ordem de Preenchimento é Parte da Regra

Os campos das entidades de cadastro são validados na ordem em que aparecem na entidade, que é a mesma ordem da ficha. Isso não é detalhe de apresentação: é o que permite preencher, numa única chamada, campos que dependem uns dos outros, sem obrigar a integração a quebrar a gravação em várias requisições encadeadas.

A cadeia obrigatória em cliente e fornecedor é:

**Cód. UF** → **Cód. Categoria Participante** → **Categoria** → **Produtor Rural** → **CPF/CNPJ** → **Inscrição Estadual** → **Inscrição Municipal**

O motivo de cada elo:

| Elo | Por quê |
| --- | --- |
| **Cód. Categoria Participante** antes de **Categoria** | Gravar a categoria do participante preenche automaticamente a **Categoria**, a **Natureza Pessoa Jurídica**, o **Produtor Rural**, o **Optante pelo Simples** e o **Não Contribuinte ICMS** |
| **Categoria** antes de **CPF/CNPJ** | Informar CPF/CNPJ sem a **Categoria** é recusado; e mudar a **Categoria** limpa **CPF/CNPJ**, **Inscrição Estadual** e **Inscrição Municipal** |
| **Produtor Rural** antes de **CPF/CNPJ** | A crítica de CPF/CNPJ já usado por outro participante da mesma categoria isenta o produtor rural |
| **Cód. UF** antes de **Inscrição Estadual** | A validação da I.E. roda o dígito verificador do estado e exige a UF preenchida |

No item valem duas ordens: **Código NCM** antes de **Cód. Exceção NCM**, porque a exceção só existe dentro de um NCM; e **Possui GTIN** antes de **GTIN**, pelo motivo da seção seguinte.

Enviar os campos fora dessa ordem não corrompe nada. A gravação simplesmente é recusada, com a mesma mensagem que apareceria na tela.

---

## 6. O Par Possui GTIN / GTIN

O **GTIN** é o único campo da API da Microsoft que esta atualização repete de propósito. A razão é que os dois campos não são separáveis:

- **Possui GTIN** é o que liga a exigência do GTIN. Com ele desmarcado, o GTIN não é cobrado no faturamento nem na emissão da NF-e. Marcá-lo sem informar o GTIN transforma o item em algo que passa a falhar na emissão;
- um **GTIN** gravado pela API da Microsoft com **Possui GTIN** desmarcado não vale nada na NF-e.

Como a API da Microsoft não conhece **Possui GTIN**, os dois campos precisam morar na mesma entidade para que uma integração consiga gravá-los na mesma chamada. É a mesma paridade que a ficha do item já pratica: o **GTIN** aparece logo abaixo de **Possui GTIN**, dentro do grupo fiscal, e só fica habilitado quando **Possui GTIN** está marcado.

Na entidade `items`:

- os campos aparecem na ordem **Possui GTIN** → **GTIN**, porque desmarcar **Possui GTIN** limpa o **GTIN**, e o valor novo tem de ser gravado depois dessa limpeza;
- gravar um **GTIN** não vazio com **Possui GTIN** desmarcado é recusado. Esse é o equivalente, num contexto sem tela, do campo desabilitado da ficha. Sem essa recusa, a API aceitaria um GTIN órfão que a tela não deixa digitar.

---

## 7. Listas de Códigos Fiscais

Os campos fiscais dos cadastros apontam para tabelas de código, e uma integração que grava um código inexistente tem a gravação recusada. Sem poder consultar essas tabelas, a integração escreve às cegas: não descobre os valores válidos e não confere o que já existe no ambiente.

As doze listas abaixo são **somente de consulta**, ou seja, criação, alteração e exclusão são recusadas, e trazem código e descrição.

| Entidade | Legenda | Campos |
| --- | --- | --- |
| `ncmCodes` | **Códigos NCM** | **Código**, **Descrição**, **Fora da Nomenclatura Vigente** |
| `ncmExceptionCodes` | **Códigos de Exceção de NCM** | **Cód. NCM**, **Código**, **Descrição** |
| `cestCodes` | **Códigos CEST** | **Código**, **Descrição** |
| `cstCodes` | **Códigos CST** | **Classificação Imposto**, **Código**, **Descrição** |
| `icmsOriginCodes` | **Códigos de Origem do ICMS** | **Código**, **Descrição** |
| `taxExceptions` | **Exceções Fiscais** | **Código**, **Descrição** |
| `nfsServiceCodes` | **Códigos de Serviço NFS-e** | **Código**, **Descrição** |
| `anpCodes` | **Códigos ANP** | **Código**, **Descrição** |
| `cfopCodes` | **Códigos CFOP** | **Nº**, **Descrição** |
| `taxAreas` | **Áreas Fiscais** | **Código**, **Descrição** |
| `participantCategories` | **Categorias do Participante** | **Tipo Participante**, **Código**, **Descrição** |
| `territories` | **Territórios** | **Código**, **Nome** |

Cada lista pode ser filtrada por código e por descrição, e devolve o que está cadastrado naquele ambiente, não uma tabela genérica. A manutenção desses cadastros continua sendo trabalho de tela.

### Três listas em que o código sozinho é ambíguo

Em três casos o código não identifica a linha, e o qualificador vai junto:

- **Códigos CST** trazem a **Classificação Imposto**. O código `00` de ICMS não é o código `00` de IPI, de PIS ou de COFINS. São linhas diferentes com significados diferentes. O campo **Código CST ICMS** do item aceita apenas CST de ICMS, e um CST de outro tributo tem a gravação recusada;
- **Códigos de Exceção de NCM** trazem o **Cód. NCM** a que pertencem. A exceção gravada em um item precisa pertencer ao NCM já gravado nesse mesmo item;
- **Categorias do Participante** trazem o **Tipo Participante**, porque a categoria é cadastrada separadamente para cliente e para fornecedor.

### Por que **Fora da Nomenclatura Vigente** entra na lista de NCM

Um NCM que saiu da nomenclatura ainda existe no cadastro e é aceito na gravação do item. A recusa só acontece semanas depois, na emissão do documento fiscal. Publicar essa marca junto com o código permite conferir a validade antes de classificar, em vez de entregar uma classificação que falha tarde. O comportamento da marca é o descrito em [Vigência dos Códigos NCM e Itens com NCM Inválido](Localization-for-Brazil_User-Manual_NCM-Vigencia-Codigos-Extintos.md).

### Por que **Territórios** entra

A API padrão da Microsoft não publica o **Cód. UF** do cliente e do fornecedor, e sem ele a validação da Inscrição Estadual não roda. Nenhuma I.E. seria gravável pela API. O **Cód. UF** passou a integrar as entidades de cliente e de fornecedor, e a lista de **Territórios** existe para que a UF possa ser conferida antes da gravação.

---

## 8. Valores das Listas Fechadas

Os campos de lista fechada expõem os valores válidos no documento de metadados da API, de modo que a integração possa escolher entre eles em vez de tratar o campo como texto livre. Para **Categoria**, os valores são:

| Valor na API | Como aparece na tela |
| --- | --- |
| `1.- Person` | **Pessoa Física** |
| `2.- Company` | **Pessoa Jurídica** |
| `3.- Foreign` | **Estrangeiro** |
| `4.- Produtor Rural` | **Produtor Rural** |
| `Unidentified` | **Não Identificado** |

O fornecedor tem ainda `5.- Foreign Person` (**Pessoa Estrangeira**).

Os nomes técnicos com prefixo numérico vêm do cadastro original e não podem ser renomeados sem quebrar integrações existentes.

O campo **Motivo da não informação do NIF** do cliente aceita `_x0020_` (em branco), `WaivedFromNIF` (**Dispensado do NIF**) e `NIFNotRequired` (**Não exigência do NIF**). Ele informa na NFS-e por que o cliente do exterior não tem NIF; ver [NFS-e e Reforma Tributária — Tomador estrangeiro](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md#62-tomador-estrangeiro).

O campo **Tipo Cliente Venda Triangular** passou a ter lista fechada nesta atualização, com os mesmos valores de antes: **Revendedor**, **Cliente Final** e **Ambos**. Nada muda na tela nem nos dados já gravados. A diferença é que os valores válidos agora aparecem nos metadados.

---

## 9. Segurança e Permissões

| Garantia | Como é obtida |
| --- | --- |
| Disponível sem passo manual | As entidades entram no conjunto de permissão básico da localização, que já faz parte do **D365 BASIC**. Nenhuma configuração é necessária no cliente |
| Cada usuário vê o que já podia ver | O acesso concedido é ao objeto de API. Os dados continuam governados pelos conjuntos de permissão do usuário que autentica a chamada: quem não pode ver ou alterar o cadastro pela tela também não consegue pela API |
| Ninguém cria nem apaga cadastros por aqui | A recusa está gravada na própria entidade, não numa configuração que possa ser afrouxada depois |
| As listas de códigos são imutáveis | Além de não permitirem criação e exclusão, elas também recusam alteração |

---

## 10. Mudanças no Comportamento dos Cadastros

Duas mudanças acompanham a publicação das entidades e valem também para a tela, para pacotes de configuração e para importações.

### 10.1 Críticas passam a rodar apenas quando o campo muda de fato

A API valida todos os campos enviados, mesmo aqueles cujo valor não mudou. Como algumas críticas de cliente e de fornecedor rodavam de forma incondicional, uma gravação que apenas devolvia o registro lido podia ser recusada sem que nada tivesse sido alterado. É exatamente isso que a ação *Atualizar registro* do Power Automate e do Power Apps faz. No fornecedor, revalidar **Produtor Rural** com o mesmo valor chegava a limpar a Inscrição Estadual silenciosamente.

A partir desta atualização, essas críticas só rodam quando o valor realmente muda. Os campos afetados em cliente e fornecedor são **Produtor Rural**, **Inscrição Municipal**, **Natureza Pessoa Jurídica**, **Não Contribuinte ICMS**, **Cód. Suframa** (cliente) e **RPA/Aluguel**, **Número de Dependentes**, **Maior de 65 Anos** (fornecedor).

Na tela nada muda, porque ali o valor sempre muda ao ser editado. Uma alteração real para uma combinação inválida continua sendo recusada, com a mesma mensagem de sempre.

### 10.2 Validação da I.E. em sessão sem tela

Ao informar a **Inscrição Estadual**, a tela pergunta se o sistema deve validá-la. Numa sessão sem interface, como API, fila de tarefas ou importação, não há quem responda. A partir desta atualização, nesse contexto a resposta assumida é **Sim**: a I.E. é sempre validada, o que exige a **Cód. UF** preenchida, roda o dígito verificador do estado, recusa duplicidade e marca **I.E. Validado**.

A consequência a registrar: uma I.E. que a tela deixaria gravar respondendo *Não*, como uma base legada ou um layout atípico, é recusada pela API. Para esses casos, o caminho continua sendo a tela.

---

## 11. Limitações Conhecidas

| Limitação | Detalhe |
| --- | --- |
| **Cód. ANVISA** e **Meio Transporte** ficam como texto livre | O código da ANVISA não tem tabela de códigos no sistema, e o meio de transporte aponta para uma tabela padrão da Microsoft que não está publicada como API. Os dois são graváveis, mas não conferíveis por API |
| A lista de **Códigos CFOP** sai completa | Ela mistura os CFOP de compra e os de venda. O campo **Código CFOP (Importação)** do item aceita apenas CFOP de compra do exterior, então um código escolhido fora desse recorte tem a gravação recusada |
| Escopo desta fase | Estão publicados cliente, fornecedor e item. Conta contábil, documento fiscal e as entidades dos apps de livros fiscais, bancário e pacote avançado ficam para fases seguintes |

O conjunto de campos de cada entidade e o nome de cada entidade são estáveis a partir da publicação da `v1.0`: acrescentar campo a uma entidade publicada é possível, retirar não. É por isso que as listas saíram enxutas, com código e descrição, mais o qualificador onde ele faz parte da identidade da linha.

---

## 12. Bônus: Consumo por Agentes de IA (MCP)

Há um ganho que vem de graça com estas entidades, e que não exige nenhuma linha de integração.

O ambiente em nuvem do Business Central oferece um servidor MCP, que é o canal por onde clientes de IA, como Copilot Studio, VS Code, Claude e ChatGPT, consultam e alteram os dados da empresa. O que esse canal expõe é exatamente o catálogo de páginas de API publicadas no ambiente: cada entidade publicada vira uma ferramenta que o agente pode chamar. Como a localização não publicava nenhuma, ela era invisível ao agente. Agora aparece junto com as entidades da Microsoft.

Na prática, o consultor funcional que configura a parte fiscal de um ambiente passa a poder pedir ao agente:

- *"quais clientes estão sem CPF/CNPJ ou sem Cód. Área Imposto?"*, antes da primeira emissão de NF-e;
- *"preenche a Categoria e o CPF/CNPJ deste cliente"*, depois de criá-lo pela API padrão;
- *"classifica estes itens com Código NCM e Código CST ICMS"*, sem abrir a ficha de cada um;
- *"quais CSTs de ICMS existem?"*, e recebe a lista cadastrada naquele ambiente, não uma lista genérica.

Três características do desenho descrito acima tornam isso viável:

- **as legendas em pt-BR** são o que o agente lê para escolher a ferramenta certa. Cada entidade traz também uma descrição dizendo que é a extensão fiscal brasileira do mesmo registro e que o cadastro-base nasce pela API da Microsoft. É isso que evita o agente confundir a entidade `customers` daqui com a `customers` da Microsoft;
- **as listas de códigos fiscais** deixam o agente conferir um código antes de gravá-lo, em vez de inventar um valor plausível e ter a gravação recusada;
- **a ordem dos campos** deixa o agente preencher categoria e CPF/CNPJ numa operação só, como faria na tela.

### Governança da escrita

A leitura pelo agente funciona assim que as entidades existem. A **gravação** é uma chave separada: ela só funciona se o administrador do ambiente habilitá-la explicitamente na configuração do servidor MCP, e a habilitação é feita por ambiente. Sandbox e produção são decisões independentes. Sem ela, o agente lê e não altera nada.

Criação e exclusão não são habilitáveis nem por engano. Como a recusa está gravada na própria entidade, a opção correspondente nem fica disponível para o administrador marcar. O mesmo vale para a alteração das listas de códigos fiscais.
