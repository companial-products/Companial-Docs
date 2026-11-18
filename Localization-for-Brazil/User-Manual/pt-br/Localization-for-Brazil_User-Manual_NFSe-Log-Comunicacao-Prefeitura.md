<!-- PR: #195 -->
# NFS-e: Log de Comunicação com a Prefeitura

**Data**: Outubro/2026

---

## Sumário

O Business Central passa a guardar os arquivos trocados com a prefeitura nas operações de NFS-e: o arquivo **exatamente como foi enviado** e o **retorno bruto** que a prefeitura devolveu. Os arquivos ficam na nova tela **Log de Comunicação da NFS-e**, aberta a partir das telas de lote de NFS-e, e podem ser baixados um a um. Isso vale também quando a transmissão termina em erro, que é justamente quando o arquivo mais faz falta.

O log é ligado **por filial** na **Config. NF-e**, nasce **desligado** e tem um prazo de retenção configurável, cumprido por uma tarefa de expurgo que o sistema cria e ativa sozinho quando o log é ligado. A recomendação é manter o log desligado e ligá-lo apenas durante testes e verificações, para não popular o banco de dados desnecessariamente (veja a seção 2).

---

## 1. Contexto

O serviço de integração da Fenix monta o arquivo no padrão exigido por cada cidade, assina, envia à prefeitura e recebe a resposta. Até agora esses arquivos ficavam registrados apenas no log de nuvem do serviço, ao qual poucas pessoas têm acesso. Quando uma nota era rejeitada, ou voltava com um erro que a mensagem resumida na **Lista Erros** do lote não explicava, a análise dependia de acionar quem tinha esse acesso.

Com o log de comunicação, o próprio usuário, o consultor funcional ou o contador do cliente consegue, sem sair do Business Central:

- baixar o arquivo enviado de um lote rejeitado para conferir qual informação saiu errada;
- ler a mensagem de rejeição completa no retorno da prefeitura, que muitas vezes chega resumida ou cortada na **Lista Erros**;
- anexar o par enviado/retornado a um chamado aberto na prefeitura ou no provedor da cidade;
- acompanhar, num lote que ficou aguardando retorno, cada consulta enviada e cada resposta recebida;
- diagnosticar um **cancelamento recusado** pela prefeitura;
- recuperar, dentro do prazo de retenção, o que foi transmitido no passado.

### Diferença em relação a "Visualizar XML"

A tela **NFS-e - Status de Lote RPS** já tinha a ação **Visualizar XML**. Ela continua existindo e funcionando como antes, mas mostra outra coisa:

| | **Visualizar XML** | **Log de Comunicação** |
| --- | --- | --- |
| O que mostra | O XML montado pelo Business Central, que é o insumo enviado ao serviço de integração | O arquivo que o serviço de integração efetivamente enviou à prefeitura, já no padrão da cidade e assinado, e o retorno da prefeitura |
| Quando está disponível | Apenas em parte dos casos | Em toda operação feita com o log ligado |
| Retorno da prefeitura | Não | Sim |
| Histórico de tentativas | Não | Sim, um conjunto de registros por tentativa |

O arquivo que a prefeitura valida e critica é o do **Log de Comunicação**. Um não substitui o outro.

---

## 2. Configuração

### Ligar o log

O log é configurado por filial.

1. Navegue até **Config. NF-e** (página `NFeSetupFNX`) da filial desejada.
2. No grupo **NFS-e**, localize o subgrupo **Log**.
3. Ative o campo **Log de Comunicação** (`NFSLogEnabled`).
4. Se quiser, ajuste o campo **Retenção do Log (Dias)** (`NFSLogRetentionDays`).

Ao ligar o log, o sistema exibe a mensagem *"O log de comunicação da NFS-e está ligado e a tarefa recorrente de expurgo está em execução."* A partir desse momento, toda transmissão de NFS-e daquela filial passa a ser registrada, e a tarefa de expurgo já está ativa (veja a seção 5). Não há nenhum passo adicional.

> **Importante**: o log **nasce desligado**, tanto em instalações novas quanto em ambientes que receberam a atualização. Nenhuma filial passa a registrar arquivos automaticamente.

### Recomendação de uso

**Recomendamos manter o log desligado no dia a dia** e ligá-lo apenas durante testes e verificações, como a homologação de uma cidade nova, a análise de um lote rejeitado ou de um cancelamento recusado, ou o acompanhamento de um chamado junto à prefeitura. Concluída a análise, desligue o log novamente.

Com o log ligado, toda transmissão da filial grava os arquivos enviados e recebidos, e cada tentativa gera registros novos. Em operação contínua, isso popula o banco de dados desnecessariamente: um lote de 50 RPS gera arquivos de algumas centenas de KB por tentativa, e nas cidades atendidas via PlugNotas a consulta do lote faz uma chamada por documento. O expurgo limita por quanto tempo os arquivos ficam guardados, mas não evita que eles sejam gravados.

Os arquivos registrados durante a verificação continuam disponíveis depois que o log é desligado, até o prazo de retenção removê-los (veja [Desligar o log](#desligar-o-log)).

### Prazo de retenção

| Regra | Valor |
| --- | --- |
| Valor inicial | 30 dias (também em configurações já existentes antes da atualização) |
| Mínimo | 15 dias |
| Máximo | 90 dias |

Um valor fora da faixa é recusado com a mensagem *"A retenção do log de comunicação da NFS-e deve ficar entre 15 e 90 dias."* O campo nunca fica vazio.

O prazo pode ser alterado mesmo com o log desligado, porque continua valendo para o histórico que a filial já tem gravado.

### Desligar o log

Ao desligar o campo **Log de Comunicação**, o sistema pede confirmação: *"As novas transmissões de NFS-e deixarão de ser registradas no log de comunicação. O que já está registrado continua disponível até o prazo de retenção removê-lo. Deseja continuar?"*

Desligar o log **não apaga** o que já está registrado e **não para** a tarefa de expurgo. O histórico continua disponível para consulta e download, e vai sendo removido conforme o prazo de retenção da filial.

### Pré-requisito: serviço de integração atualizado

Os arquivos são devolvidos ao Business Central pelo serviço de integração da Fenix. Se o log estiver ligado e nenhuma transmissão aparecer na tela, mesmo depois de enviar ou consultar lotes, entre em contato com o suporte da Fenix para confirmar que o serviço de integração usado pelo ambiente já está na versão que devolve os arquivos.

---

## 3. Como Acessar

A tela **Log de Comunicação da NFS-e** (página `NFSeCommunicationLogs`) pode ser aberta de quatro formas:

| Origem | Ação | O que é exibido |
| --- | --- | --- |
| **NFS-e - Status de Lote RPS** (página `CheckNFSeBatchFNX`) | **Log de Comunicação**, ao lado de **Visualizar XML** | Os arquivos do lote selecionado |
| **NFS-e - Envio de Lote** (página `TransmitNFSeBatchFNX`) | Grupo **Lote** → **Log de Comunicação**, junto de **Lista Erros** e **Lista Avisos** | Todos os arquivos da filial da tela, do mais recente para o mais antigo |
| **Config. NF-e** (página `NFeSetupFNX`) | **Log de Comunicação da NFS-e** | Todos os arquivos da filial da configuração |
| Busca do Business Central (**Alt+Q**) | Pesquise por **Log de Comunicação da NFS-e** | Todos os arquivos, de todas as filiais |

Na tela **NFS-e - Envio de Lote**, o log é filtrado apenas pela filial, e não pelo lote do documento selecionado. Quando uma transmissão falha, o documento fiscal não fica ligado ao lote em que a tentativa foi registrada. Filtrar pelo lote do documento esconderia justamente as tentativas com erro, que são as que o usuário procura nessa tela.

---

## 4. A tela Log de Comunicação da NFS-e

A tela é somente leitura e lista um registro por arquivo trocado, do mais recente para o mais antigo.

### Colunas

| Coluna | Descrição |
| --- | --- |
| **Data e Hora** | Quando o arquivo foi registrado no Business Central |
| **Nº Lote** | Lote de RPS ao qual o arquivo pertence. Fica vazio nas operações que não estão ligadas a um lote, como o cancelamento e a consulta do último número de RPS |
| **Cód. Filial** | Filial cuja configuração de NFS-e foi usada na transmissão |
| **Nº Documento** | Documento fiscal ao qual o arquivo se refere. É preenchido nas operações de cancelamento |
| **Operação** | Operação de NFS-e que gerou o arquivo (veja a seção 6) |
| **Direção** | **Envio** para o arquivo enviado à prefeitura, **Retorno** para a resposta recebida |
| **Nº Sequencial** | Ordem do arquivo entre os arquivos trocados na mesma tentativa |
| **Nome do Arquivo** | Nome dado ao arquivo pelo serviço de integração |
| **Tipo de Conteúdo** | **XML**, **JSON** ou **Texto**. Define a extensão sugerida no download |
| **Tamanho do Conteúdo** | Tamanho do arquivo, em caracteres |
| **ID Usuário** | Usuário da sessão que fez a transmissão. Nas transmissões feitas pela rotina automática (bot) de NFS-e, é o usuário com que a fila de tarefas executou |

As colunas **Nº Mov.** e **ID da Tentativa** ficam ocultas por padrão e podem ser exibidas pela personalização da página. O **ID da Tentativa** agrupa todos os arquivos de uma mesma tentativa: filtrar por ele mostra exatamente o que foi enviado e o que voltou naquela tentativa.

### Ações

| Ação | Descrição |
| --- | --- |
| **Download** | Baixa o arquivo da linha selecionada exatamente como foi trocado com a prefeitura. O nome sugerido é o **Nome do Arquivo**, seguido do **Nº Mov.** e da extensão correspondente ao **Tipo de Conteúdo** (`.xml`, `.json` ou `.txt`), de modo que dois downloads nunca têm o mesmo nome |
| **Expurgar Agora** | Executa o expurgo imediatamente, após confirmação, removendo os registros mais antigos que o prazo de retenção de cada filial (veja a seção 5) |

---

## 5. Expurgo do Log

### Tarefa recorrente

No momento em que o log é ligado em qualquer filial, o sistema cria em **Movs. de Fila de Tarefas** a tarefa **Expurgar Log de Comunicação da NFS-e** e a deixa com status **Pronto**. A tarefa roda uma vez por dia, todos os dias da semana.

- Se a tarefa já existir, ligar o log em outra filial não cria uma segunda.
- Se alguém tiver colocado a tarefa em espera, ligar o log de novo em qualquer filial a reativa.
- Se não for possível criar ou ativar a tarefa, o log não fica ligado: a filial nunca fica registrando arquivos sem o expurgo correspondente.

### O que é removido

A cada execução, o expurgo percorre **todas as filiais** e remove de cada uma os registros mais antigos que a sua **Retenção do Log (Dias)**. Isso inclui as filiais que já tiveram o log ligado e depois o desligaram: o histórico delas também sai pelo prazo.

**Exemplo**:

| Filial | Log de Comunicação | Retenção do Log (Dias) | Registros removidos |
| --- | --- | --- | --- |
| MATRIZ | Ligado | 30 | Os registrados há mais de 30 dias |
| FILIAL SP | Ligado | 90 | Os registrados há mais de 90 dias |
| FILIAL RJ | Desligado | 15 | Os registrados há mais de 15 dias, enquanto ainda houver histórico gravado |

A ação **Expurgar Agora**, na tela do log, aplica exatamente as mesmas regras, sem esperar a próxima execução da tarefa.

---

## 6. Comportamento

### Operações registradas

| Operação | Quando ocorre |
| --- | --- |
| **Enviar Lote de RPS** | Transmissão do lote à prefeitura, inclusive o reenvio de um lote já montado e a recuperação de documento PlugNotas |
| **Consultar Lote de RPS** | Consulta do retorno do lote, nas cidades em que a prefeitura só devolve o resultado numa consulta posterior |
| **Cancelar NFS-e** | Pedido de cancelamento de uma NFS-e |
| **Consultar Cancelamento** | Consulta da situação de um pedido de cancelamento |
| **Consultar Último Nº de RPS** | Consulta do último número de RPS usado na prefeitura |

A operação **Consultar Último Nº de RPS** só é usada pelos padrões de cidade que a oferecem (DSF, São Paulo e São Luís). Nas demais cidades não há essa chamada à prefeitura e, portanto, não há registro.

### Cada tentativa tem seus próprios registros

Toda tentativa gera registros novos e nenhum registro sobrescreve o anterior. Um lote consultado cinco vezes até a prefeitura concluir o processamento deixa cinco conjuntos de registros de **Consultar Lote de RPS**, o que permite ver em que etapa o processo parou e como ele evoluiu.

Quando uma mesma operação troca várias mensagens com a prefeitura, todas são registradas, na ordem indicada pelo **Nº Sequencial**. Nas cidades atendidas via PlugNotas, por exemplo, a consulta do lote faz uma chamada por documento, e cada uma delas aparece no log.

### Transmissões com erro

O registro é gravado assim que a resposta do serviço de integração chega ao Business Central, **antes** de ser interpretada. Assim, um erro que ocorra depois, durante o processamento da resposta, não desfaz o registro: se o arquivo chegou a ser montado, ele aparece; se a resposta da prefeitura chegou a ser recebida, ela também aparece.

Com o log ligado, uma transmissão que falha pode consumir um número de lote da numeração interna de lotes sem que o lote apareça na tela de lotes. Esse número é um controle interno e não tem efeito fiscal.

### Quando nada é registrado

Só é registrado o que foi efetivamente trocado com a prefeitura. Não há registro quando:

- o log da filial está desligado;
- o serviço de integração não respondeu (tempo esgotado, falha de rede, erro de comunicação);
- o serviço de integração usado pelo ambiente ainda não está na versão que devolve os arquivos.

### Conteúdo dos arquivos

- **Retorno sem tratamento**: o retorno é gravado exatamente como foi recebido, sem corte, reformatação ou reindentação. Pode ser XML, JSON, HTML de erro ou texto simples, conforme o que a prefeitura devolveu.
- **Mesmo conteúdo que o suporte consulta**: nas cidades com web service SOAP, o arquivo registrado é o conteúdo devolvido pelo método do web service da prefeitura, sem o envelope SOAP, que é o mesmo conteúdo que a equipe de suporte da Fenix consulta no log de nuvem. No ISSNet Nacional, o log guarda também a mensagem SOAP completa enviada, num arquivo cujo nome termina em `_Soap`.
- **PlugNotas**: nas cidades atendidas via PlugNotas, o log registra o **JSON enviado ao PlugNotas** e o **JSON recebido do PlugNotas**, no lugar do arquivo da prefeitura. Nas consultas, em que não há corpo enviado, o arquivo de envio traz o endereço consultado.
- **Senhas mascaradas**: alguns padrões de cidade (por exemplo, GISS e ISSNet) enviam a senha do web service da prefeitura dentro do próprio XML. Nesses arquivos, o conteúdo das tags de senha, password e token aparece como `*****`. Nada além disso é alterado.
- **Certificado digital**: o certificado e a senha do certificado nunca são gravados no log.

### Transmissões manuais e automáticas

O registro funciona da mesma forma nas transmissões feitas pelo usuário, nas telas de lote e de documento fiscal, e nas feitas pela rotina automática (bot) de NFS-e.

### O lote continua exibindo o que já exibia

O log é informação adicional. As telas de lote continuam mostrando status, protocolo, **Lista Erros** e **Lista Avisos** exatamente como antes.

---

## 7. Permissões

Quem já tem acesso às telas de lote de NFS-e tem acesso ao log. A tela, a tabela e as rotinas do log fazem parte do conjunto de permissões **BR - NFS-e** (`NFSeLocFNX`), o mesmo que dá acesso às telas **NFS-e - Status de Lote RPS** e **NFS-e - Envio de Lote**. Não há permissão adicional a conceder.

---

## 8. Exemplo Prático

**Cenário**: um lote da filial MATRIZ foi rejeitado pela prefeitura, e a **Lista Erros** mostra apenas *"Erro de esquema"*.

1. Abra **NFS-e - Status de Lote RPS** e selecione o lote rejeitado.
2. Clique em **Log de Comunicação**. A tela abre filtrada pelo lote:

| Data e Hora | Operação | Direção | Nº Sequencial | Nome do Arquivo | Tipo de Conteúdo |
| --- | --- | --- | --- | --- | --- |
| 14/10/2026 10:02 | Consultar Lote de RPS | Retorno | 2 | R_ConsultarLoteRps | XML |
| 14/10/2026 10:02 | Consultar Lote de RPS | Envio | 1 | E_ConsultarLoteRps | XML |
| 14/10/2026 10:00 | Enviar Lote de RPS | Retorno | 2 | R_EnviarLoteRps | XML |
| 14/10/2026 10:00 | Enviar Lote de RPS | Envio | 1 | E_EnviarLoteRps | XML |

3. Selecione a linha de **Retorno** da operação **Consultar Lote de RPS** e clique em **Download** para ler a mensagem de rejeição completa, com o campo e o valor criticados.
4. Selecione a linha de **Envio** da operação **Enviar Lote de RPS** e clique em **Download** para conferir o valor que saiu no arquivo transmitido.
5. Corrija o cadastro e reenvie. A nova tentativa gera registros novos, e os anteriores continuam disponíveis para comparação.

---

## 9. Resumo dos Campos Novos

### Tabela Config. NF-e (`NFeSetupFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Log de Comunicação** (`NFSLogEnabled`) | Booleano | Liga o registro dos arquivos trocados com a prefeitura para a filial. Nasce desligado. Ao ligar, cria e ativa a tarefa de expurgo |
| **Retenção do Log (Dias)** (`NFSLogRetentionDays`) | Inteiro | Por quantos dias os arquivos ficam no log. Valor inicial 30, entre 15 e 90 |

### Nova Tabela Log de Comunicação da NFS-e (`NFSeCommunicationLog`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Nº Mov.** (`EntryNo`) | Inteiro | Número sequencial do registro |
| **Cód. Filial** (`BranchCode`) | Código | Filial da transmissão |
| **Nº Lote** (`BatchNo`) | Código | Lote de RPS do arquivo |
| **Nº Documento** (`DocumentNo`) | Código | Documento fiscal, nas operações de cancelamento |
| **Operação** (`Operation`) | Enum | **Enviar Lote de RPS**, **Consultar Lote de RPS**, **Cancelar NFS-e**, **Consultar Cancelamento** ou **Consultar Último Nº de RPS** |
| **Direção** (`Direction`) | Opção | **Envio** ou **Retorno** |
| **Nº Sequencial** (`SequenceNo`) | Inteiro | Ordem do arquivo dentro da tentativa |
| **ID da Tentativa** (`AttemptID`) | GUID | Agrupa os arquivos de uma mesma tentativa |
| **Nome do Arquivo** (`FileName`) | Texto | Nome dado ao arquivo pelo serviço de integração |
| **Tipo de Conteúdo** (`ContentType`) | Opção | **XML**, **JSON** ou **Texto** |
| **Tamanho do Conteúdo** (`ContentLength`) | Inteiro | Tamanho do arquivo, em caracteres |
| **Data e Hora** (`LoggedDateTime`) | Data/Hora | Quando o arquivo foi registrado |
| **ID Usuário** (`UserID`) | Código | Usuário da sessão que fez a transmissão |
| **Conteúdo** (`Content`) | BLOB | O arquivo, exatamente como foi trocado |
