<!-- PR: #209, #240 -->
# Certificado Digital: Validade no Cadastro, Aviso de Vencimento e Bloqueio da Transmissão

**Data**: Setembro/2026

---

## Sumário

A validade do certificado digital agora é lida do próprio arquivo e exibida nos cadastros de certificado (Certificados Digitais, Configurações Reinf, APIs bancárias e CT-e). O sistema avisa com antecedência configurável (padrão: 30 dias) que o certificado está para vencer e bloqueia a transmissão com mensagem clara quando ele já venceu — na NF-e, NFS-e, MD-e, CT-e, EFD-Reinf e nas APIs bancárias. Ao abrir as telas, o aviso considera apenas os certificados apontados em alguma configuração: certificados antigos que ficaram no cadastro após uma renovação não geram aviso. Além disso, as listas de eventos da EFD-Reinf passam a mostrar o motivo do erro devolvido pela Receita na própria lista, com a coluna **Últimas Mensagens de Erro** e a ação **Visualizar Erros**.

---

## 1. Contexto

### O problema

Até agora, a data de validade do certificado digital não ficava registrada em nenhum cadastro — apenas o nome do arquivo e a data de importação. O usuário só descobria que o certificado havia vencido quando uma transmissão falhava, e mesmo assim a falha aparecia como erro genérico. Nas listas de eventos da EFD-Reinf, por exemplo, as linhas recusadas ficavam apenas com Status **Erro**, e o motivo real só era visível navegando até o histórico de eventos enviados.

### O que mudou

- A **data de validade** do certificado é lida do próprio arquivo (`.pfx`/`.p12`) e exibida nos cadastros.
- Um **aviso não bloqueante** aparece com antecedência configurável quando o certificado está para vencer — ao abrir as telas de trabalho e ao transmitir.
- Com certificado **já vencido**, a transmissão é **bloqueada** imediatamente, com mensagem indicando a data do vencimento, sem enviar nada e sem marcar as linhas com erro genérico.
- Nas listas de eventos da EFD-Reinf, o **motivo do erro** devolvido pela Receita fica visível na própria lista.

### Certificados já importados

Não é preciso reimportar nada. Para certificados importados antes da atualização, a validade é lida do certificado armazenado na primeira vez em que for necessária (ao abrir a tela de cadastro ou na primeira transmissão) e gravada no cadastro. Se o certificado ou a senha não puderem ser lidos, o sistema segue funcionando como antes — sem aviso e sem bloqueio.

---

## 2. Validade Visível nos Cadastros

### Certificados Digitais (módulo principal)

Na página **Certificados Digitais** (`DigitalCertificatesFNX`), usada pela NF-e, NFS-e, MD-e e CT-e, a lista ganhou a coluna **Data/Hora de Vencimento**, preenchida automaticamente na importação do certificado. A coluna é destacada em vermelho quando o certificado já venceu e em amarelo quando está dentro do prazo de aviso. O destaque vale para todas as linhas da lista, inclusive certificados antigos que nenhuma configuração usa mais — mas só os certificados em uso geram aviso ao abrir a tela (veja [Quais certificados geram aviso ao abrir as telas](#quais-certificados-geram-aviso-ao-abrir-as-telas)).

### Configurações Reinf

Na página **Configurações Reinf** (`ReinfSetupFNX`), o novo campo **Data/Hora de Vencimento do Certificado** aparece abaixo do campo **Certificado** e é preenchido ao importar o certificado — inclusive imediatamente após a importação, sem precisar reabrir a página.



---

## 3. Configuração do Aviso Antecipado

Navegue até **Configurações Fiscais** (`FiscalSetupFNX`). No grupo Geral há o novo campo:

| Campo | Descrição |
|-------|-----------|
| **Aviso de Vencimento do Certificado (Dias)** (`CertExpiryWarningDays`) | Com quantos dias de antecedência do vencimento o sistema passa a avisar o usuário. Padrão: **30 dias**. Informe **0** para desligar o aviso antecipado. |

Pontos importantes:

- O padrão de 30 dias vale também para **clientes existentes**: após a atualização, todos passam a receber o aviso automaticamente, sem configuração adicional.
- O aviso antecipado **não bloqueia** nenhuma operação — ele apenas informa a data de vencimento para que haja tempo de renovar o certificado.
- Um certificado **já vencido** sempre bloqueia a transmissão, independentemente do valor configurado.

---

## 4. Comportamento: Aviso e Bloqueio

### Aviso antecipado (certificado a vencer)

Quando o certificado vence dentro do prazo configurado, o usuário recebe um aviso não bloqueante:

> O certificado digital [nome] vence em [data]. Renove-o para evitar a interrupção das suas transmissões.

O aviso aparece:

- **Ao abrir as telas de trabalho** de cada módulo: **Certificados Digitais**, **NF-e - Transmitir Pendentes**, **NFS-e - Envio de Lote**, **Manifestações de Destinatário NF-e**, **Gerenciamento de CT-e**, as sete listas de eventos da EFD-Reinf, **Pagamentos Eletrônicos** e **Boletos**, além das telas de configuração (**Configurações Reinf** e **Configurações CT-e**).
- **Ao transmitir** com o certificado a vencer.

Ao abrir as telas, um certificado que **já venceu** e ainda está em uso também gera aviso, para que a renovação seja feita antes da próxima transmissão:

> O certificado digital [nome] venceu em [data]. As transmissões que o utilizam serão bloqueadas até que um certificado válido seja importado.

### Quais certificados geram aviso ao abrir as telas

Cada importação de certificado cria uma linha nova na página **Certificados Digitais**, e a linha do certificado anterior permanece no cadastro. Por isso, ao abrir as telas que usam esse cadastro — **Certificados Digitais**, **NF-e - Transmitir Pendentes**, **NFS-e - Envio de Lote**, **Manifestações de Destinatário NF-e** e **Gerenciamento de CT-e** —, o aviso considera apenas os certificados **em uso**, isto é, apontados em alguma configuração que os utiliza na transmissão:

| Configuração | Campo |
|--------------|-------|
| **Config. NF-e** (`NFeSetupFNX`), de **todas as filiais**, inclusive a linha sem filial | **ID Certificado** do grupo NF-e (usado na NF-e e na MD-e) e **ID Certificado** do grupo NFS-e |
| **Configurações CT-e** (`CTeSetupFNX`), quando o Advanced Pack está instalado | **ID Certificado** |

Na prática:

- Depois de importar o certificado novo e apontá-lo na **Config. NF-e**, o certificado antigo vencido **deixa de gerar aviso**. Na lista **Certificados Digitais** ele continua com a data de vencimento destacada em vermelho, para identificação.
- O aviso cobre o certificado de **qualquer filial**, mesmo que a filial selecionada na tela seja outra.
- O certificado apontado na **Configurações CT-e** conta como em uso também nas telas de NF-e, MD-e e NFS-e.
- Se alguma configuração **ainda aponta para o certificado antigo** — por exemplo, o **ID Certificado** do grupo NFS-e de uma filial, ou a **Configurações CT-e** —, o aviso continua aparecendo. Isso é intencional: as transmissões daquela configuração serão bloqueadas. Para eliminar o aviso, atualize o certificado nessa configuração.
- A mensagem, o prazo de antecedência e a regra de exibir o aviso uma vez por sessão para cada certificado não mudam.

As listas de eventos da EFD-Reinf, as telas de **Pagamentos Eletrônicos** e **Boletos** e as telas de configuração (**Configurações Reinf** e **Configurações CT-e**) já avisavam apenas sobre o certificado da própria configuração, e continuam assim.

Para não incomodar, o aviso de um mesmo certificado aparece **no máximo uma vez por sessão** — quem viu a notificação ao abrir uma lista não recebe a mesma mensagem de novo ao transmitir.

### Bloqueio (certificado vencido)

Com o certificado já vencido, qualquer operação que o utilize — envio, consulta de retorno, distribuição, cancelamento, inutilização, download — é bloqueada na hora, antes de qualquer comunicação com o serviço externo:

> O certificado digital [nome] venceu em [data]. Importe um certificado válido antes de transmitir.

O bloqueio vale para todos os módulos que usam certificado digital:

| Módulo | Operações cobertas |
|--------|--------------------|
| NF-e | Transmissão, consulta, cancelamento, CC-e, inutilização, eventos |
| NFS-e | Transmissão de lote e download (Emissor Nacional) |
| MD-e | Consulta de documentos e envio de manifestações |
| CT-e | Consulta e distribuição de documentos |
| EFD-Reinf | Envio e consulta de lotes de eventos |
| APIs bancárias | Pagamentos e recebimentos via API (Banco do Brasil, Itaú, Santander) |

Na EFD-Reinf, o bloqueio acontece **antes** de o lote sair: as linhas não são mais marcadas com Status **Erro** por causa de certificado vencido.

### Rotinas automáticas (fila de tarefas)

Em rotinas que rodam sem interface de usuário — consulta automática de CT-e, transmissão automática de NF-e, rotinas de cobrança —, o aviso antecipado não é exibido, mas o bloqueio por certificado vencido registra a mensagem clara no log da fila de tarefas, no lugar da falha genérica de antes.

---

## 5. Motivo do Erro nas Listas de Eventos da EFD-Reinf

### O que mudou

As sete listas de eventos que mostravam apenas o Status ganharam a coluna **Últimas Mensagens de Erro** e a ação **Visualizar Erros**:

| Página | Evento |
|--------|--------|
| **Reinf - Pagamentos a Pessoas Jurídicas** (`ReinfCompanyPaymentsFNX`) | R-4020 |
| **Reinf - Pagamentos a Pessoas Físicas** (`ReinfPersonPaymentsFNX`) | R-4010 |
| **Reinf - Retenção Contribuição Previdenciária - Compras** (`ReinfRetainedINSSPurchasesFNX`) | R-2010 |
| **Reinf - Retenção Contribuição Previdenciária - Vendas** (`ReinfRetainedINSSSalesFNX`) | R-2020 |
| **Reinf - Compras de Produção Rural** (`ReinfRuralProductionPurchFNX`) | R-2055 |
| **Reinf - Vendas de Produção Rural** (`ReinfRuralProdSalesFNX`) | R-2050 |
| **Reinf - Retenção Contribuição Previdenciária sobre Receita Bruta** (`ReinfCPRBFNX`) | R-2060 |

### Como funciona

- A coluna **Últimas Mensagens de Erro** mostra as mensagens devolvidas pela Receita para o último evento enviado da linha. A coluna pode ser usada para **filtrar e ordenar** a lista.
- A ação **Visualizar Erros** abre o texto completo dos erros de validação do registro selecionado — o mesmo detalhe que antes só era alcançável pelo histórico de eventos enviados.
- Quando um novo envio é **aceito**, a coluna é limpa automaticamente.
- Na atualização, as linhas que **já estavam com erro** são preenchidas com o motivo do último evento enviado — não é preciso reenviar nada para ver o motivo da recusa.

---

## 6. Resumo dos Campos Novos

### Tabela Certificado Digital (`DigitalCertificateFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Data/Hora de Vencimento** (`ExpirationDateTime`) | Data/Hora | Validade lida do próprio arquivo do certificado. Preenchida na importação e, para certificados já importados, no primeiro uso. |

### Tabela Configurações Fiscais (`FiscalSetupFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Aviso de Vencimento do Certificado (Dias)** (`CertExpiryWarningDays`) | Inteiro | Antecedência do aviso de vencimento. Padrão 30; 0 desliga o aviso antecipado. |

### Tabela Configurações Reinf (`ReinfSetupFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Data/Hora de Vencimento do Certificado** (`CertificateExpiration`) | Data/Hora | Validade do certificado usado no EFD-Reinf, lida do próprio arquivo. |

### Tabelas de eventos da EFD-Reinf

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Últimas Mensagens de Erro** (`LastErrorMessages`) | Texto | Mensagens de erro devolvidas pelo EFD-Reinf para o último evento enviado da linha. Presente nas seis tabelas por trás das sete listas de eventos. |
