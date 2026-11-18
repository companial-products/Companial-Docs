<!-- PR: #212 -->
# Licenciamento Companial e Funcionalidades Opcionais da Localização

**Data**: Outubro/2026

---

## Sumário

A Localização Brasileira passa a ter um cadastro de **Funcionalidades Opcionais da Localização**: recursos que o administrador liga ou desliga para o ambiente inteiro, sem depender de configuração empresa a empresa. A primeira funcionalidade desse cadastro é o **Licenciamento Companial**. A Localização Brasileira tem dois mecanismos de licença, o **License Monitor** e o **Companial License Management**, e cada ambiente usa um deles conforme o mercado e o contrato do cliente. Com a funcionalidade ligada, os pontos de verificação da licença validam pelo **Companial License Management**, que licencia por usuário. Com ela desligada, a verificação é feita pelo License Monitor, como descrito em [Monitoramento de Licença da Localização Brasileira](Localization-for-Brazil_User-Manual_Monitoramento-de-Licenca.md).

---

## 1. Contexto

A licença da Localização Brasileira pode ser controlada por dois mecanismos, que convivem de forma permanente:

- **License Monitor**: valida a assinatura do ambiente (tenant). Continua sendo o mecanismo de licença em parte dos mercados, sem previsão de ser retirado.
- **Companial License Management**: aplicativo da Companial com um modelo diferente, em que a licença é **por usuário e por assento**. Cada usuário do Business Central precisa estar atribuído e habilitado ao app **Localization for Brazil** para usar as funcionalidades licenciadas.

Qual dos dois vale depende do mercado e do contrato de cada cliente. A escolha é feita por ambiente, ligando ou desligando a funcionalidade **Licenciamento Companial**:

| Situação da funcionalidade | Mecanismo de licença |
|----------------------------|----------------------|
| Desligada (padrão) | License Monitor |
| Ligada | Companial License Management |

A funcionalidade só tem efeito nas versões da Localização distribuídas com o **Companial License Management** como dependência (versões AppSource). Numa versão sem esse aplicativo, ligar a funcionalidade não muda nada: a verificação continua pelo License Monitor e nenhuma operação deixa de funcionar por causa dela.

---

## 2. Funcionalidades Opcionais da Localização

### 2.1. Como Acessar

Abra o recurso **Diga-me** (atalho **Alt+Q**) e pesquise por *Funcionalidades Opcionais*. Selecione **Funcionalidades Opcionais da Localização** (página `OptionalFeatures`).

A página lista todas as funcionalidades opcionais existentes. Elas aparecem automaticamente, sem nenhum cadastro manual, e toda funcionalidade nova começa **desligada**.

### 2.2. Colunas da Lista

| Coluna | Descrição |
|--------|-----------|
| **Funcionalidade** | Nome da funcionalidade opcional. |
| **Descrição** | O que a funcionalidade faz. |
| **Ligada** | Indica se a funcionalidade está ligada. Quando ligada, o valor aparece destacado em verde. |
| **Ligada desde** | Data e hora em que a funcionalidade foi ligada. Fica vazia enquanto a funcionalidade está desligada. Se ela for desligada e ligada de novo, passa a mostrar a data da nova ligação. |
| **Por Empresa** | Indica se a funcionalidade é ligada ou desligada em cada empresa. Quando não está marcado, a funcionalidade vale para todas as empresas do ambiente ao mesmo tempo. |
| **Saiba mais** | Abre uma página com mais informações sobre a funcionalidade. Quando não há página, o sistema informa: *"Não há informações adicionais disponíveis para esta funcionalidade."* |

### 2.3. Ligar e Desligar

Selecione a funcionalidade na lista e use uma das ações:

- **Ligar**: liga a funcionalidade selecionada. Disponível apenas quando ela está desligada.
- **Desligar**: desliga a funcionalidade selecionada. Disponível apenas quando ela está ligada.

As duas ações pedem confirmação. Para uma funcionalidade que vale para o ambiente inteiro, a pergunta avisa que a mudança atinge todas as empresas, por exemplo:

> *Ligar a funcionalidade Licenciamento Companial?*
> *A funcionalidade será ligada para todas as empresas deste ambiente ao mesmo tempo.*

Somente usuários com o conjunto de permissões **SUPER** podem ligar ou desligar uma funcionalidade. Os demais usuários podem abrir a página e consultar a situação, mas ao tentar ligar ou desligar recebem a mensagem:

> *Somente usuários com o conjunto de permissões SUPER podem ligar ou desligar funcionalidades opcionais.*

---

## 3. A Funcionalidade Licenciamento Companial

| Coluna | Valor |
|--------|-------|
| **Funcionalidade** | Licenciamento Companial |
| **Descrição** | Valida a licença da localização pelo Companial License Management em vez do License Monitor. A licença é por usuário: cada usuário precisa estar atribuído e habilitado ao app Localização para Brasil. |
| **Por Empresa** | Não marcado: vale para todas as empresas do ambiente |

Uma licença não pode estar ativa numa empresa e inativa em outra. Por isso, o **Licenciamento Companial** é ligado e desligado uma única vez para o ambiente inteiro, e todas as empresas enxergam sempre a mesma situação.

### 3.1. Pontos Verificados

Ligar a funcionalidade não acrescenta nem retira pontos de verificação. As mesmas operações descritas em [Monitoramento de Licença da Localização Brasileira](Localization-for-Brazil_User-Manual_Monitoramento-de-Licenca.md#3-funcionalidades-sujeitas-à-verificação) são validadas pela Companial quando a funcionalidade está ligada:

| Área | Operação |
|------|----------|
| Fiscal | Cálculo de impostos (todas as operações que recalculam tributos) |
| NF-e | Transmissão à SEFAZ (síncrona e assíncrona) |
| MD-e | Consulta de documentos na SEFAZ |
| NFS-e | Transmissão de lote |
| Cadastros | Consulta de CNPJ |

A verificação usa a identidade do app **Localization for Brazil** e o usuário que está executando a operação. Os apps **Tax Books**, **Banking** e **Advanced Pack** não têm verificação própria de licença.

### 3.2. Quando a Verificação Não Acontece

As mesmas isenções do License Monitor continuam valendo com o **Licenciamento Companial** ligado:

- **Processamento sem interface de usuário**: fila de trabalhos, tarefas agendadas e integrações via API não passam pela verificação.
- **Verificar Contabilização**: a simulação de lançamentos não dispara a verificação.
- **Ambientes On-Premises**: as instalações locais não recebem nenhum dos aplicativos de licenciamento e não passam por verificação, nem da Companial nem do License Monitor.

---

## 4. Comportamento da Verificação pela Companial

### 4.1. Uma Verificação por Sessão

A licença é verificada na **primeira** operação licenciada de cada sessão do usuário. Quando a verificação passa, ela não é repetida até o usuário sair e entrar de novo no Business Central. Quando a verificação falha, nada fica guardado e a próxima tentativa verifica de novo. Assim que o administrador regulariza a situação do usuário, a próxima tentativa já é liberada, sem precisar sair e entrar.

Em consequência, se o administrador **desabilitar** um usuário no Companial License Management, a mudança vale para esse usuário a partir da próxima sessão. Na sessão que já estiver aberta, ele continua operando.

### 4.2. Regras da Licença Companial

As regras abaixo pertencem ao aplicativo da Companial. A Localização apenas aciona a verificação para o seu próprio app.

| Situação | O que acontece |
|----------|----------------|
| Usuário atribuído e habilitado ao app, licença válida | A operação segue normalmente. |
| Ambiente sem licença Companial | Período de teste de **30 dias**, contados a partir da primeira verificação feita com a funcionalidade ligada. Durante o teste, qualquer usuário opera, mesmo sem estar atribuído ao app, e recebe um aviso dos dias restantes. Terminado o prazo sem licença, a operação é bloqueada. |
| Usuário não atribuído ao app (com licença emitida) | A operação é bloqueada. |
| Usuário atribuído mas não habilitado (com licença emitida) | A operação é bloqueada. |
| Licença sem validação há mais de **30 dias** | Se os termos de uso da Companial já foram aceitos, o aplicativo da Companial tenta revalidar a licença automaticamente junto à Companial. |
| Licença sem validação há **60 dias** ou mais | A operação segue, com um aviso de que a validação vai expirar. |
| Licença sem validação há **90 dias** ou mais | A operação é bloqueada até a licença ser revalidada. |
| Mais usuários habilitados do que assentos comprados | A operação segue com um aviso durante **30 dias**. Depois desse prazo, é bloqueada até o excesso ser resolvido. |

### 4.3. Mensagens Exibidas

As mensagens de aviso e de bloqueio são exibidas pelo aplicativo da Companial. Como esse aplicativo não tem tradução para o português, elas aparecem em inglês e indicam que o usuário deve procurar o administrador ou escrever para o suporte da Companial. As mais comuns são:

| Mensagem (em inglês) | Significado |
|----------------------|-------------|
| *Warning: No valid license found for ... You can continue using the app for N more day(s) during the trial period.* | Período de teste: a operação segue e o aviso informa os dias restantes. |
| *There is no valid license for ...* | Período de teste encerrado sem licença: a operação é bloqueada. |
| *There is no App User Assignment record for user ... and app ...* | O usuário não está atribuído ao app: a operação é bloqueada. |
| *User ... does not have an active license for ...* | O usuário está atribuído mas não habilitado: a operação é bloqueada. |
| *Warning: License validation is expiring soon for ...* | A licença não é validada há 60 dias ou mais. |
| *License validation has expired for ...* | A licença não é validada há 90 dias ou mais: a operação é bloqueada. |
| *Warning: License over-allocated for ...* | Há mais usuários habilitados do que assentos comprados. |

Como a verificação acontece uma vez por sessão, os avisos também aparecem no máximo uma vez por sessão.

---

## 5. Configuração

### 5.1. Ligar o Licenciamento Companial

1. Confirme que o aplicativo **Companial License Management** está instalado no ambiente, na versão da Localização que traz esse aplicativo como dependência.
2. Pesquise por *Companial Licenses* no **Diga-me** (**Alt+Q**) para consultar a licença do app **Localization for Brazil** no ambiente. Licenças, atribuição de usuários, sincronização e aceite dos termos de uso são feitos nas páginas do próprio aplicativo da Companial, cujos nomes aparecem em inglês.
3. Atribua e habilite ao app **Localization for Brazil** os usuários que vão usar as funcionalidades licenciadas, pela ação **User Assignments** da página da Companial.
4. Com um usuário **SUPER**, abra **Funcionalidades Opcionais da Localização**, selecione **Licenciamento Companial** e clique em **Ligar**. Confirme a pergunta.
5. A partir desse momento, todas as empresas do ambiente verificam a licença pela Companial.

Um cliente novo pode ligar a funcionalidade antes mesmo de ter a licença definitiva: enquanto não houver licença, vale o período de teste de 30 dias da Companial. Desligar e ligar de novo a funcionalidade não reinicia esse prazo.

### 5.2. Voltar ao License Monitor

Se o ambiente precisar voltar a ser licenciado pelo License Monitor:

1. Com um usuário **SUPER**, abra **Funcionalidades Opcionais da Localização**.
2. Selecione **Licenciamento Companial** e clique em **Desligar**. Confirme a pergunta.

A verificação volta imediatamente para o License Monitor em todas as empresas. Nenhum dado da Companial é apagado: licenças e atribuições de usuários continuam no ambiente, e a funcionalidade pode ser ligada de novo a qualquer momento.

---

## 6. Permissões

| Ação | Permissão necessária |
|------|----------------------|
| Abrir **Funcionalidades Opcionais da Localização** e consultar a situação | Qualquer usuário |
| **Ligar** ou **Desligar** uma funcionalidade | Conjunto de permissões **SUPER** |
| Passar pela verificação de licença da Companial | Já incluída no **D365 BASIC** |

O conjunto de permissões **D365 BASIC** passa a incluir o conjunto **BR - Companial License**, que dá ao usuário comum o acesso de que a verificação da Companial precisa. Nenhuma permissão adicional precisa ser atribuída, da mesma forma que já acontecia com o License Monitor. O conjunto **BR - Companial License** também pode ser atribuído diretamente a um usuário que não tenha o **D365 BASIC**.

---

## 7. Mudanças nas Mensagens do License Monitor

Mesmo com o **Licenciamento Companial** desligado, as mensagens de licença exibidas pelo License Monitor para a Localização Brasileira passam a indicar como contato **Arquiconsult Brasil ou Parceiro responsável**. Quando disponíveis, também trazem o identificador do app (*AppId*), o nome do app (*AppName*) e a funcionalidade verificada (*Feature*), para facilitar o atendimento.

Com o **Licenciamento Companial** ligado, as mensagens que o License Monitor exibe por conta própria para a Localização deixam de aparecer, já que nesse ambiente a licença é controlada pela Companial.

---

## 8. Resumo dos Campos Novos

### Tabela Funcionalidade Opcional da Localização (`OptionalFeature`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Funcionalidade** (`Feature`) | Opção | Funcionalidade opcional. |
| **Descrição** (`Description`) | Texto | O que a funcionalidade faz. |
| **Ligada** (`Enabled`) | Sim/Não | Indica se a funcionalidade está ligada. |
| **Ligada desde** (`EnabledAt`) | Data/Hora | Quando a funcionalidade foi ligada; vazio enquanto desligada. |
| **URL Saiba Mais** (`LearnMoreUrl`) | Texto | Página com mais informações sobre a funcionalidade. |
| **Por Empresa** (`PerCompany`) | Sim/Não | Indica se a funcionalidade é ligada ou desligada em cada empresa. |
