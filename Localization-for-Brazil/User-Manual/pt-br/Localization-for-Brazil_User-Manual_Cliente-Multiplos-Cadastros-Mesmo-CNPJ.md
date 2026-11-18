<!-- PR: #201, #279 -->
# Cadastro de Clientes: Mais de um Cadastro com o Mesmo CNPJ

**Data**: Setembro/2026

---

## Sumário

Até agora a Localização tratava o CNPJ como identificador exclusivo do cliente: informar um número já usado em outro cadastro da mesma categoria recusava a gravação. A trava protege contra a duplicidade acidental — que é o caso muito mais frequente —, mas impede o cenário legítimo em que uma única pessoa jurídica opera sob mais de uma identidade comercial e a empresa quer acompanhar cada uma como um cadastro próprio.

Agora essa trava pode ser transformada em aviso, de forma controlada e em duas camadas: um interruptor em **Conf. Vendas & Cobranças** habilita a funcionalidade na empresa, e uma marcação no próprio cliente autoriza cadastro a cadastro. Sem as duas, o bloqueio continua exatamente como antes. Em consequência, a EFD-Reinf passa a consolidar num único evento R-2020 os clientes que compartilham o CNPJ, e a importação de XML de NF-e deixa de escolher sozinha o cliente quando o CNPJ do emitente resolve mais de um cadastro e a Inscrição Estadual da nota não identifica um deles.

---

## 1. Contexto

### O problema

O CNPJ identifica a pessoa jurídica, não a marca comercial sob a qual ela opera. Há atividades em que a mesma empresa explora várias identidades comerciais distintas: o exemplo concreto que motivou a funcionalidade é o mercado de apostas, em que a Lei nº 14.790/2023 (art. 12, parágrafo único) e a Portaria SPA/MF nº 827/2024 (art. 5º, I) permitem que um mesmo CNPJ explore até três marcas sob uma única outorga.

Quem precisava faturar e acompanhar o resultado de cada marca separadamente só tinha duas saídas ruins: manter um cadastro único para todas as marcas, perdendo a separação, ou inventar um CNPJ falso para o segundo cadastro.

### A solução

A trava deixa de ser incondicional e passa a depender de **duas autorizações em série**:

1. **Na empresa** — o interruptor **Permitir CNPJ Duplicado em Clientes** em **Conf. Vendas & Cobranças** habilita a funcionalidade. Desligado, que é o padrão, nada muda em relação ao comportamento anterior.
2. **No cadastro** — com o interruptor ligado, o campo **Permitir CNPJ Duplicado** aparece no **Cartão Cliente** e autoriza aquele cadastro, individualmente, a repetir um CNPJ já usado.

A separação acompanha a divisão de responsabilidades habitual do Business Central: quem administra o sistema decide **se** a empresa opera com CNPJ repetido; quem cadastra decide **quais** cadastros o fazem, um a um, e ainda confirma um aviso a cada vez.

### O escopo: somente pessoa jurídica

A flexibilização vale **exclusivamente para cliente com Categoria igual a Pessoa Jurídica**. Nenhum caminho novo se abre para CPF:

- Pessoa física continua sendo recusada sempre, com a exceção de **Produtor Rural**, que já podia repetir o número antes desta funcionalidade e não foi alterada.
- **Fornecedor** está fora desta entrega. A flexibilização é restrita a cliente.
- A marcação não pode ser ligada em cliente que não seja Pessoa Jurídica, e é limpa automaticamente se a **Categoria** do cliente deixar de ser Pessoa Jurídica.

---

## 2. Configuração

### Passo 1 — Habilitar a funcionalidade na empresa

Navegue até **Conf. Vendas & Cobranças** (página `Sales & Receivables Setup`), grupo **Fiscal**, e marque o campo **Permitir CNPJ Duplicado em Clientes**.

Enquanto o interruptor estiver desligado:

- O campo de autorização **não aparece** no **Cartão Cliente**;
- Tentar ligar a autorização por outro caminho (API, pacote de configuração) é recusado com erro;
- A trava de CNPJ duplicado continua bloqueando a gravação, mesmo em cadastros que já estivessem marcados.

### Passo 2 — Autorizar o cadastro do cliente

No **Cartão Cliente** (página `Customer Card`), grupo **Fiscal**, entre **Produtor Rural** e **CPF/CNPJ**, marque o campo **Permitir CNPJ Duplicado**.

O campo só fica habilitado quando a **Categoria** do cliente é **Pessoa Jurídica**.

> **Importante**: apenas o cadastro que está sendo gravado precisa da marcação. No cenário das marcas comerciais, o cliente que já existia continua sem marcação alguma — é o cadastro novo que carrega a autorização. Isso significa que desdobrar uma base já existente em marcas **não exige mexer no cadastro antigo**.

---

## 3. Cadastrar um Cliente com CNPJ Repetido

Com as duas autorizações no lugar, informe o **CPF/CNPJ** normalmente. Ao detectar que o número já está em outro cliente, o sistema pergunta:

> *O C.N.P.J. já está cadastrado para o Cliente 10000 - Marca Alfa Ltda. Deseja cadastrá-lo também neste Cliente?*

- **Sim** — o cadastro é gravado com o CNPJ repetido;
- **Não** — o campo volta ao valor anterior e nada é gravado.

### O que acontece em cada situação

| Situação | Comportamento |
| --- | --- |
| **Categoria** diferente de Pessoa Jurídica (inclui toda pessoa física) | Bloqueio: *C.N.P.J./C.P.F. já cadastrado para o Cliente %1 - %2* |
| Pessoa Jurídica, interruptor **desligado** (padrão) | Bloqueio, com a mesma mensagem |
| Pessoa Jurídica, interruptor ligado, **sem** a marcação no cliente | Bloqueio, com a mesma mensagem |
| Pessoa Jurídica, interruptor ligado, **com** a marcação, com interface | Pergunta de confirmação identificando o cliente existente |
| Pessoa Jurídica, interruptor ligado, **com** a marcação, sem interface | Grava sem perguntar |

A verificação do interruptor acontece **na própria trava**, e não apenas na visibilidade do campo. Por isso ela vale também para a API, para o pacote de configuração e para o cadastro que ficou marcado antes de alguém desligar a funcionalidade.

### Desligar o interruptor não desfaz nada

Desligar **Permitir CNPJ Duplicado em Clientes** não apaga as marcações dos clientes nem revalida os cadastros existentes. As duplicidades já criadas continuam como estão; apenas nenhuma duplicidade nova é aceita. Religando o interruptor, as autorizações que estavam lá voltam a valer.

### Desmarcar a autorização em um cliente

Não é possível desmarcar **Permitir CNPJ Duplicado** enquanto existir outro cliente da mesma categoria cadastrado com o mesmo CNPJ. A tentativa é recusada com:

> *Não é possível desmarcar Permitir CNPJ Duplicado enquanto o Cliente 10000 - Marca Alfa Ltda. estiver cadastrado com o mesmo C.N.P.J.*

O motivo é que o número não é revalidado ao desmarcar: sem essa proteção, o cliente ficaria com um CNPJ que a regra já não permite que ele tenha, e a marcação viraria puramente decorativa.

---

## 4. Consultar os Clientes com o Mesmo CNPJ

A partir de qualquer cliente é possível ver quais outros cadastros compartilham aquele número.

No **Cartão Cliente** e na **Lista de Clientes** (página `Customer List`), acesse **Relacionado** → **Cliente** → **Clientes com o Mesmo CNPJ**. A ação abre a **Lista de Clientes** filtrada pelo **CPF/CNPJ** do cliente atual.

A ação fica desabilitada em cadastros sem **CPF/CNPJ** preenchido, e é exibida somente quando a funcionalidade está habilitada em **Conf. Vendas & Cobranças**.

---

## 5. Processos sem Interface: API, Pacotes de Configuração e Integrações

Em processos que rodam sem um usuário para responder — importação de cadastros, integrações, chamadas de API, tarefas em fila —, não há pergunta de confirmação:

- Cliente **autorizado** (interruptor ligado e marcação no registro): gravado sem bloqueio e sem pergunta;
- Cliente **não autorizado**: continua sendo recusado, para que uma carga não crie duplicidades em massa.

### API

A entidade `customers` da API da Localização (página `CustomersAPIv10`) ganhou o campo `allowDuplicateCnpj`, declarado **imediatamente antes** de `cnpjCpf`. A posição não é estética: a API e o pacote de configuração aplicam os campos na ordem em que aparecem, de modo que a autorização precisa chegar antes do número para que o registro seja aceito num único envio.

Gravar `allowDuplicateCnpj` como verdadeiro com o interruptor da empresa desligado retorna erro, exatamente como na tela.

---

## 6. Importação de XML de NF-e: Escolha do Cliente

Ao importar o XML de uma NF-e emitida por um cliente (devolução, retorno), o sistema identificava o cliente pelo CNPJ do emitente e, havendo mais de um cadastro com aquele número, escolhia o primeiro em silêncio.

Agora, quando o CNPJ do emitente corresponde a **mais de um cliente**, o sistema compara a Inscrição Estadual do emitente na nota com a **Inscrição Estadual** de cada um desses clientes, considerando só os dígitos. Se exatamente um deles tiver a mesma IE, ele é o participante.

Se nenhum ou mais de um cliente tiver a mesma IE, ou se a nota vier sem IE ou com IE `ISENTO`, o sistema **não escolhe**: o **Nº Participante** fica em branco e o usuário recebe a mensagem:

> *O CNPJ/CPF 00.000.000/0001-00 do emitente está cadastrado em 2 clientes, portanto o participante ficou em branco. Escolha o cliente no campo Nº Participante antes de criar o documento.*

A escolha é feita na **Manifestação de Destinatário NF-e** (página `NFeMDeFNX`), no campo **Nº Participante**, cuja lista de seleção já vem filtrada pelo CNPJ do emitente — os candidatos são exatamente os cadastros que compartilham aquele número. O documento só pode ser criado depois que o participante estiver preenchido.

O comportamento é o mesmo no XML avulso, no processamento em lote e na tarefa automática, e **não depende do interruptor**: o sistema reage ao dado que encontra na base.

Havendo apenas um cadastro com aquele CNPJ, nada muda — o participante continua sendo preenchido automaticamente.

A mesma regra vale para emitente pessoa física, identificado por CPF. Veja [MD-e: Emitente Pessoa Física (CPF) — Localização do Cliente e do Fornecedor](Localization-for-Brazil_User-Manual_MDe-Emitente-CPF-Pessoa-Fisica.md).

---

## 7. EFD-Reinf: Evento R-2020 Consolidado

Este é o ponto em que a repetição de CNPJ exigiria mais cuidado, e por isso é tratado automaticamente.

O evento **R-2020 (serviços prestados)** é identificado pela Receita pela combinação **período + estabelecimento prestador + CNPJ do tomador**. Como o agrupamento interno era feito pelo **código do cliente**, dois clientes com o mesmo CNPJ e retenção no mesmo mês produziriam dois eventos com a mesma identificação — o segundo seria rejeitado, ou trataria o primeiro como substituído.

### Como funciona agora

Ao executar **Sugerir Eventos Mensais**, os documentos de clientes que compartilham um CNPJ passam a ser agrupados sob **um único cliente do grupo**, chamado aqui de cliente de reporte. O resultado é **um único registro**, e portanto **um único evento R-2020**, com:

- Os valores de retenção **somados**;
- As notas de **todos** os cadastros do grupo relacionadas no evento.

O cliente de reporte é escolhido assim:

1. Se o período já foi sugerido antes e existe um registro do mesmo período, estabelecimento e obra cujo participante é um cliente daquele CNPJ, o evento continua ancorado nesse mesmo registro. Isso mantém um período já transmitido apontando para o evento que a Receita já recebeu;
2. Caso contrário, o **menor código de cliente** entre os que compartilham o CNPJ.

Clientes sem número, pessoa física, estrangeiros e não identificados continuam reportando sob o próprio código, como antes.

Todo o ciclo de vida do evento — envio, consulta de lote, retificação, exclusão e a sinalização de retificação pendente — continua funcionando sem alteração, porque a relação de um registro para um evento foi preservada. Alterar qualquer um dos cadastros que compõem o evento consolidado e sugerir novamente sinaliza a necessidade de retificação, como em qualquer outro evento.

> A consolidação **não lê o interruptor** de **Conf. Vendas & Cobranças**. Ela reage ao dado: se existem dois clientes com o mesmo CNPJ na base — porque a funcionalidade esteve ligada, porque vieram de migração ou porque um aplicativo de terceiros os gravou —, os eventos precisam ser consolidados de qualquer maneira, sob pena de rejeição pela Receita.

### Rastreabilidade: de qual cadastro veio cada nota

Como as notas de vários clientes passam a aparecer sob um só participante, o documento de retenção ganhou o campo **Nº Participante Origem**, que guarda o cliente para o qual o documento foi de fato emitido.

O campo é exibido na página **Reinf - Retenção Contribuição Previdenciária - Documentos** (página `ReinfRetainedINSSDocumentsFNX`), ao lado do **Nº Participante**. Quando não há consolidação, os dois são iguais.

### Conferência na tela de serviços prestados

Na página **Reinf - Retenção Contribuição Previdenciária - Vendas** (página `ReinfRetainedINSSSalesFNX`) foi acrescentada a coluna **C.N.P.J. Participante**, ao lado do **Nº Participante**. É o que permite entender, de relance, por que um evento traz notas de mais de um cadastro: todos eles compartilham o CNPJ exibido.

---

## 8. Efeitos nos Demais Processos Fiscais

### SPED Fiscal e EFD Contribuições

**Cada cadastro continua gerando o seu próprio participante** no registro 0150, com o mesmo CNPJ podendo aparecer em mais de um código de participante. Cada participante leva o nome, o endereço e a Inscrição Estadual do seu cadastro, e cada documento aponta para o participante correspondente ao cadastro que o originou.

Não houve alteração de comportamento: a geração dos arquivos do período já suportava esse cenário.

### IRRF

A acumulação mensal do IRRF e seus limites mínimos já agrupam os movimentos pela **raiz do CNPJ**, ou seja, todos os cadastros que compartilham a raiz entram no mesmo acumulado. Cadastros com o CNPJ inteiro igual já estão contemplados por esse critério.

Na prática: dividir o movimento entre dois cadastros de mesmo CNPJ produz o mesmo valor de IRRF que produziria se tudo estivesse num cadastro só. Não houve alteração.

### Inscrição Estadual

A trava de Inscrição Estadual duplicada já isentava os cadastros que compartilham a raiz do CNPJ. Portanto, dois ou mais clientes com o mesmo CNPJ podem ter a mesma **Inscrição Estadual** sem bloqueio. Não houve alteração.

### Criação de cliente a partir de contato

A criação de cliente a partir de um contato já ligado a um fornecedor com o mesmo CNPJ **continua bloqueada de forma incondicional**, mesmo com o interruptor ligado e o cliente existente autorizado. O bloqueio acontece antes de o registro existir, quando ainda não há onde a autorização pudesse ser informada.

Quem precisa de um segundo cadastro com o mesmo CNPJ deve criá-lo pelo **Cartão Cliente**, que passa a ser o único ponto de entrada da duplicidade autorizada.

Nesse mesmo bloqueio foi corrigido um defeito anterior: quando o fornecedor ligado ao contato estava **sem CPF/CNPJ**, o sistema comparava o número em branco com o de qualquer cliente também sem número e recusava a criação sem haver duplicidade nenhuma, exibindo a mensagem com um número vazio. Agora, fornecedor sem número não dispara bloqueio. Onde há duplicidade real, o bloqueio continua exatamente como antes.

### Faturamento, cobrança e documentos fiscais

Documentos fiscais, faturamento e cobrança funcionam normalmente para cada um dos cadastros que dividem o CNPJ, cada um com seus próprios movimentos e saldos. A diferenciação entre as marcas se dá pelo **Nome** do cliente, como em qualquer outro cadastro — não há campo novo para identificar a marca comercial.

---

## 9. Resumo dos Campos Novos

### Tabela Conf. Vendas & Cobranças (`Sales & Receivables Setup`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Permitir CNPJ Duplicado em Clientes** (`AllowDuplicateCNPJFNX`) | Booleano | Habilita a funcionalidade na empresa. Desmarcado por padrão; enquanto estiver desmarcado, o mesmo CNPJ nunca pode ser cadastrado em dois clientes |

### Tabela Cliente (`Customer`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Permitir CNPJ Duplicado** (`AllowDuplicateCNPJFNX`) | Booleano | Autoriza este cliente a ser cadastrado com um CNPJ já usado por outro cliente. Disponível somente para **Categoria** igual a Pessoa Jurídica e apenas com a funcionalidade habilitada em **Conf. Vendas & Cobranças** |

### Tabela Reinf - Documento Retenção Contribuição Previdenciária (`ReinfRetainedINSSDocumentFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Nº Participante Origem** (`SourceParticipantNo`) | Código (20) | Cliente para o qual o documento foi de fato emitido. Difere do **Nº Participante** quando vários clientes compartilham um CNPJ e são reportados num único evento. Somente leitura |

### Demais inclusões

| Onde | Inclusão |
| --- | --- |
| **Cartão Cliente** e **Lista de Clientes** | Ação **Clientes com o Mesmo CNPJ**, em **Relacionado** → **Cliente** |
| **Reinf - Retenção Contribuição Previdenciária - Vendas** | Coluna **C.N.P.J. Participante**, apenas para conferência |
| API `customers` (`CustomersAPIv10`) | Campo `allowDuplicateCnpj`, declarado antes de `cnpjCpf` |

---

## 10. Pontos de Atenção

- **A funcionalidade nasce desligada.** Quem atualizar e não fizer nada tem um sistema indistinguível do anterior: os dois campos novos nascem desmarcados e o campo do cliente nem aparece na tela. Nenhum dado precisa ser convertido.
- **A proteção vale o que valer a separação de permissões.** As duas camadas só fazem sentido se o perfil que cadastra clientes não tiver acesso livre a **Conf. Vendas & Cobranças**. Vale conferir isso nas empresas em que a funcionalidade for habilitada.
- **Bases com duplicidade herdada.** A trava anterior só era avaliada ao informar o número, então um cliente pode ter chegado à base com CNPJ repetido por migração, carga antiga ou aplicativo de terceiros. Nessas bases, a primeira sugestão de eventos da EFD-Reinf após a atualização passa a consolidar esses cadastros — **mesmo com o interruptor desligado**, porque a apuração reage ao dado. Antes de atualizar, verifique se existem CNPJs repetidos entre clientes pessoa jurídica e, havendo, confirme com o time fiscal se algum deles tem R-2020 já transmitido.
- **A ação de consulta segue o interruptor.** Numa base com duplicidade herdada e a funcionalidade desligada, a ação **Clientes com o Mesmo CNPJ** não aparece. Para usá-la como ferramenta de investigação, habilite a funcionalidade em **Conf. Vendas & Cobranças**.
- **A escolha do participante na NF-e é cega ao tipo de número.** O sistema deixa o **Nº Participante** em branco sempre que o número do emitente corresponde a mais de um cadastro e a Inscrição Estadual da nota não identifica um deles, inclusive quando se trata de CPF de produtores rurais — que já podiam repetir o número antes desta funcionalidade. O ganho é não escolher em silêncio o primeiro cadastro encontrado.
- **Validações que exigem ambiente real.** A aceitação do R-2020 consolidado depende da transmissão ao ambiente da Receita, e a leitura do arquivo do SPED com o mesmo CNPJ em dois códigos de participante depende da validação pelo PVA. Recomenda-se exercitar os dois antes do primeiro fechamento com cadastros duplicados em produção.
