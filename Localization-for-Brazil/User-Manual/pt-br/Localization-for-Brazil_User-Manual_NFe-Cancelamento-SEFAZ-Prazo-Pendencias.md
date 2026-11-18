<!-- PR: #228 -->
# NF-e: Cancelamento na SEFAZ no Mesmo Fluxo, Prazo de Cancelamento por Filial e Cancelamentos Pendentes

**Data**: Outubro/2026

---

## Sumário

Agora o cancelamento de um documento com NF-e própria autorizada pode ser concluído na SEFAZ na mesma ação que cancela o documento no sistema. O sistema também passa a bloquear o cancelamento depois do prazo configurado na filial e dá visibilidade aos documentos que ficaram cancelados só no sistema, com a NF-e ainda autorizada na SEFAZ. Essa visibilidade inclui uma lista com as horas restantes, um indicador nos Centros de Função, um aviso no próprio documento e um alerta ao processar a apuração da EFD.

---

## 1. Contexto

### O problema

Até agora, ao **Cancelar** ou **Corrigir** uma fatura registrada, o sistema registrava a nota de crédito de cancelamento e marcava o documento fiscal como **Cancelado**, mas nada era enviado à SEFAZ. Cancelar a NF-e era uma segunda ação manual, feita depois no documento fiscal registrado (**NF-e** → **Cancelar NF-e**).

Quando essa segunda ação era esquecida, o documento ficava inconsistente: cancelado no sistema e autorizado na SEFAZ. Ninguém era avisado, e o prazo legal de cancelamento passava despercebido. Em regra, esse prazo é de 24 horas a partir da autorização. Depois do prazo, a EFD do período era gerada com a nota como cancelada enquanto, para o fisco, ela continuava válida, e a única saída passava a ser uma NF-e de devolução.

### O que mudou

- **Cancelamento na SEFAZ no mesmo fluxo**: quando a filial habilita a transmissão automática, cancelar ou corrigir o documento pede a justificativa e cancela a NF-e na SEFAZ na mesma ação.
- **Bloqueio por prazo**: com o controle de prazo ligado na filial, o sistema não permite cancelar um documento cuja NF-e foi autorizada há mais tempo do que o prazo configurado.
- **Visibilidade das pendências**: os documentos cancelados no sistema com NF-e ainda autorizada aparecem em uma lista própria, em um indicador nos Centros de Função, em um aviso ao abrir o documento e em um alerta ao processar a EFD.

### A quais documentos se aplica

Vale para todo documento fiscal cujo tipo de documento tenha **Emitente Documento** = **Emissão Própria** e **Modelo Documento** = **55** (NF-e), seja qual for a origem:

- **Vendas**: fatura, correção de fatura e nota de crédito de devolução.
- **Compras**: devolução ao fornecedor e NF-e de entrada emitida pela própria empresa.
- **Documento fiscal avulso**.

Não se aplica a documentos de **Terceiros** ou **Sem Valor Fiscal**, nem a NFC-e (modelo 65), NFS-e e CT-e. Esses documentos não são bloqueados e não aparecem na lista, no indicador, no aviso nem no alerta.

### Exceção: alteração manual de status

As funções **Alterar Status para Cancelado** e **Alterar Status do Documento Fiscal** continuam funcionando como antes. Elas são o recurso para corrigir exceções: não transmitem nada à SEFAZ e não são bloqueadas pelo prazo. Se o documento ficar cancelado no sistema com a NF-e ainda autorizada por meio delas, ele aparece normalmente como pendência.

---

## 2. Configuração

Navegue até **Config. NF-e** (página `NFeSetupFNX`) da filial. Na aba **NF-e** há o novo grupo **Cancelamento**:

| Campo | Descrição |
|-------|-----------|
| **Transmitir cancelamento automaticamente** (`CancelTransmission`) | Quando ligado, cancelar ou corrigir um documento com NF-e própria autorizada, feito por um usuário na página, pede a justificativa e também cancela a NF-e na SEFAZ. Quando desligado, o documento é cancelado apenas no sistema e a NF-e deve ser cancelada depois pela ação **Cancelar NF-e**. |
| **Controlar prazo de cancelamento** (`CancelDeadlineControl`) | Quando ligado, o cancelamento de um documento fiscal com NF-e própria autorizada é bloqueado depois que o prazo de cancelamento termina. |
| **Prazo de cancelamento (horas)** (`CancelDeadlineHours`) | Número de horas, contadas a partir da autorização da NF-e, durante as quais ela ainda pode ser cancelada na SEFAZ. Ajuste conforme as regras da UF de emissão. Só fica editável com **Controlar prazo de cancelamento** ligado. Valor mínimo: 1. |

### Valores iniciais

| Situação | Transmitir cancelamento automaticamente | Controlar prazo de cancelamento | Prazo de cancelamento (horas) |
|----------|:---:|:---:|:---:|
| **Config. NF-e já existentes** antes da atualização | Desligado | Desligado | 24 |
| **Config. NF-e criadas** depois da atualização | Ligado | Ligado | 24 |

Nas filiais que já existiam, a atualização mantém o comportamento anterior: o documento continua sendo cancelado só no sistema e nada é bloqueado. **Recomendamos ligar os dois controles em cada filial existente** depois da atualização.

> **Atenção**: com **Transmitir cancelamento automaticamente** ligado e **Controlar prazo de cancelamento** desligado, o sistema não impede o cancelamento de uma NF-e fora do prazo. A SEFAZ rejeitará o cancelamento, e o documento ficará cancelado só no sistema, como pendência (ver [Quando a SEFAZ rejeita ou a transmissão falha](#quando-a-sefaz-rejeita-ou-a-transmissão-falha)).

---

## 3. Cancelamento na SEFAZ no Mesmo Fluxo

### Onde se aplica

Com **Transmitir cancelamento automaticamente** ligado na filial do documento, as ações abaixo cancelam a NF-e na SEFAZ na mesma operação que cancela o documento no sistema:

| Origem | Ação |
|--------|------|
| Fatura de venda registrada | **Cancelar** e **Corrigir** |
| Fatura de compra registrada | **Cancelar** e **Corrigir** |
| Nota de crédito de venda ou de compra marcada como **Cancelamento** (por exemplo, a criada por **Criar Nota de Crédito Corretiva**) | **Registrar** |
| Fatura que cancela uma nota de crédito de devolução (venda ou compra) | **Registrar** |
| Documento fiscal avulso | **Cancelar Documento Fiscal** |

### Passo a passo

1. Execute a ação de cancelamento e confirme a pergunta do sistema, como antes.
2. O sistema abre a janela **Justificativa do Cancelamento da NF-e**. Informe a justificativa que será enviada à SEFAZ (mínimo de 15 caracteres).
   - Fechar a janela sem confirmar interrompe a operação: **nada é registrado**.
   - Quando a operação cancela mais de uma NF-e (por exemplo, a fatura e o documento fiscal vinculado de uma operação triangular), a mesma justificativa vale para todas.
3. O sistema registra o cancelamento e, em seguida, transmite o cancelamento da NF-e à SEFAZ.
4. Quando a SEFAZ homologa o cancelamento, aparece a notificação: *"O cancelamento da NF-e foi registrado na SEFAZ para o documento fiscal [nº]."*

O prazo de cancelamento é verificado antes de a justificativa ser pedida. Se o prazo já terminou, a operação é bloqueada (ver [Bloqueio por Prazo de Cancelamento](#4-bloqueio-por-prazo-de-cancelamento)).

### Quando a SEFAZ rejeita ou a transmissão falha

O cancelamento na SEFAZ só é enviado depois que o lançamento de cancelamento é registrado. Se a SEFAZ rejeitar o cancelamento ou a transmissão falhar:

- o lançamento de cancelamento **permanece registrado** e o documento fiscal **permanece Cancelado**;
- a NF-e continua autorizada na SEFAZ, e o documento passa a constar como pendência;
- o usuário recebe uma mensagem com o motivo retornado e o prazo restante:

> O documento fiscal [nº] foi cancelado no sistema, mas o cancelamento da NF-e [nº] não foi concluído na SEFAZ.
>
> Motivo: [mensagem retornada]
>
> Prazo de cancelamento: [data/hora] ([n] horas restantes).
>
> A NF-e continua autorizada. Para concluir o cancelamento, abra o documento fiscal e use a ação NF-e > Cancelar NF-e.

Corrija a causa (por exemplo, a justificativa ou a disponibilidade do serviço) e conclua o cancelamento pela ação **Cancelar NF-e** no documento fiscal registrado, na lista **Cancelamentos de NF-e pendentes** ou na própria notificação do documento.

### Quando a NF-e não é transmitida

Nos casos abaixo, o documento é cancelado só no sistema, sem transmissão à SEFAZ, e fica visível como pendência para ser concluído depois pela ação **Cancelar NF-e**:

- **Transmitir cancelamento automaticamente** desligado na filial;
- registro pela fila de tarefas, registro em lote, **Registrar e Imprimir** e demais rotinas sem usuário na tela;
- NF-e que ainda não estava autorizada no momento do cancelamento (por exemplo, com lote em processamento na SEFAZ). Se ela for autorizada depois, passa a aparecer como pendência.

### Desfazer o cancelamento

Depois que o cancelamento é homologado na SEFAZ, ele não pode mais ser desfeito no sistema: a ação **Cancelar** da nota de crédito corretiva é recusada, porque a NF-e não está mais autorizada. Isso é esperado, já que o cancelamento na SEFAZ é irreversível.

---

## 4. Bloqueio por Prazo de Cancelamento

### Como o prazo é calculado

Com **Controlar prazo de cancelamento** ligado na filial do documento:

> **Prazo** = data/hora de autorização da NF-e + **Prazo de cancelamento (horas)** da filial

Cada documento é avaliado pelo prazo da **sua** filial. Dois documentos de filiais com prazos diferentes (por exemplo, 24 e 48 horas) recebem prazos diferentes.

A data/hora de autorização é obtida, nesta ordem:

1. da data/hora de recebimento registrada pela SEFAZ no XML autorizado (`dhRecbto`), que passa a ser gravada na autorização;
2. para NF-e autorizadas antes desta atualização, do XML autorizado já guardado no sistema;
3. na falta do XML, da data de envio registrada na NF-e, que nunca é anterior à autorização real.

Se nenhuma data estiver disponível, o documento não é bloqueado e aparece na lista sem contagem de prazo.

### O que acontece depois do prazo

Tentar cancelar um documento fora do prazo é bloqueado **antes de qualquer lançamento** e nada é enviado à SEFAZ. A mensagem tem o título **Prazo de cancelamento expirado (configuração do sistema)**:

> Cancelamento bloqueado pelo prazo configurado no sistema — nada foi enviado à SEFAZ.
>
> A NF-e [nº] foi autorizada em [data/hora]. O prazo de cancelamento definido na Config. NF-e da filial [filial] é de [n] horas e terminou em [data/hora]. O documento não foi cancelado e nenhum evento foi transmitido.
>
> Para desfazer a operação, emita uma NF-e de devolução. Se o prazo legal da UF da filial for maior, peça ao responsável que ajuste o Prazo de cancelamento (horas) nas configurações do sistema.

O bloqueio vale para **todos os caminhos de cancelamento** (venda, compra e documento fiscal avulso), inclusive registro pela fila de tarefas e em lote. Com o controle desligado, o cancelamento é permitido.

Não são bloqueados:

- documentos de **Terceiros**, **Sem Valor Fiscal** ou de outro modelo que não 55;
- documentos cuja NF-e não está autorizada;
- as funções **Alterar Status para Cancelado** e **Alterar Status do Documento Fiscal**.

### Ação Cancelar NF-e depois do prazo

A ação manual **Cancelar NF-e** do documento fiscal registrado **não é bloqueada** pelo prazo, porque algumas UFs aceitam o cancelamento extemporâneo. Quando o prazo da filial já terminou, a confirmação avisa:

> O prazo de cancelamento da NF-e [nº] terminou em [data/hora], e a SEFAZ pode rejeitar o cancelamento.
>
> Tem certeza que deseja cancelar o documento [nº] na SEFAZ?

---

## 5. Cancelamentos Pendentes

Um documento fiscal está **pendente** quando está com Status **Cancelado** no sistema, é NF-e própria de modelo 55 e a NF-e continua autorizada na SEFAZ. A regra vale também para os documentos que já estavam nessa situação antes da atualização, sem nenhum ajuste. Quando o cancelamento é homologado na SEFAZ, no fluxo ou depois pela ação **Cancelar NF-e**, o documento deixa de ser pendente e sai da lista, do indicador, do aviso e do alerta.

### Lista Cancelamentos de NF-e pendentes

A página **Cancelamentos de NF-e pendentes** (`NFeCancellationsPending`) pode ser encontrada pela pesquisa (**Alt+Q**), pelo indicador dos Centros de Função ou pela notificação do documento.

| Coluna | Descrição |
|--------|-----------|
| **Nº** | Nº do documento fiscal. Clicar no número abre o documento. |
| **Cód. Filial** | Filial do documento. |
| **Nº Nota Fiscal** | Número da NF-e. |
| **Série** | Série da NF-e. |
| **Nº Participante** / **Nome** | Cliente ou fornecedor do documento. |
| **Data Registro** | Data de registro do documento fiscal. |
| **Canc. por ID Usuário** | Usuário que cancelou o documento no sistema. |
| **Nº Doc. Origem Cancelamento** | Documento que originou o cancelamento. |
| **Data/Hora de Autorização** | Data e hora em que a SEFAZ autorizou a NF-e. |
| **Prazo de Cancelamento** | Data e hora até as quais a NF-e pode ser cancelada na SEFAZ, de acordo com a Config. NF-e da filial. |
| **Horas Restantes** | Horas que faltam para o fim do prazo. Um valor negativo mostra há quantas horas o prazo terminou. |

**Prazo de Cancelamento** e **Horas Restantes** ficam em vermelho quando o prazo já terminou e em branco quando a filial não controla o prazo de cancelamento.

Ações disponíveis:

| Ação | Descrição |
|------|-----------|
| **Cancelar NF-e** | Cancela na SEFAZ a NF-e do documento fiscal selecionado e atualiza a lista. |
| **Abrir Documento** | Abre o documento fiscal selecionado. |
| **Atualizar** | Atualiza a lista com os documentos fiscais cujo cancelamento na SEFAZ continua pendente. |

### Indicador nos Centros de Função

O quadro **Atividades de NF-e**, com o indicador **Cancelamentos de NF-e pendentes** no grupo **Cancelamentos de NF-e**, mostra a quantidade de documentos pendentes. Ele fica verde quando não há pendências e vermelho quando há. Clicar no número abre a lista.

O indicador aparece nos Centros de Função:

| Centro de Função | Posição |
|-------------|---------|
| **Fiscal** | Junto às atividades do Centro de Função |
| **Gerente de Negócios** | Depois das atividades principais |
| **Contador** | Depois das atividades do contador |
| **Processador de Ordens de Venda** | Depois das atividades de vendas |

Para usuários sem permissão de leitura dos documentos fiscais e das NF-e, o indicador fica oculto, sem gerar erro na página inicial.

### Aviso no documento fiscal

Ao abrir um **Documento Fiscal Registrado** pendente, uma notificação é exibida no topo da página:

> Este documento foi cancelado no sistema, mas a NF-e [nº] continua autorizada na SEFAZ. Prazo de cancelamento: [data/hora] ([n] horas restantes).

Quando o prazo já terminou, o texto informa *"O prazo de cancelamento terminou em [data/hora]."*. Quando a filial não controla o prazo, informa *"Nenhum prazo de cancelamento é controlado para esta NF-e."*.

A notificação oferece as ações **Cancelar NF-e**, que conclui o cancelamento na SEFAZ, e **Mostrar Cancelamentos Pendentes**, que abre a lista.

---

## 6. Alerta no Processamento da EFD

Ao executar **Processar** na **Apuração ICMS/IPI** ou na **Apuração PIS/COFINS** (na ficha ou nas listas **Apurações de ICMS/IPI** e **Apurações de PIS/COFINS**), o sistema verifica se há documentos pendentes no período. Entram na verificação os documentos:

- da mesma filial da apuração;
- com **Data Registro** dentro do período;
- cujo cancelamento foi registrado até a data final do período.

Se houver algum, o sistema pergunta:

> Os documentos fiscais a seguir foram cancelados no sistema, mas a NF-e deles continua autorizada na SEFAZ:
> [nº do documento fiscal] - NF-e [nº]
> ...
>
> Eles serão informados como cancelados, enquanto a NF-e continua válida para o fisco.
>
> Deseja continuar?

A mensagem lista até 10 documentos e, se houver mais, informa *"... e mais [n]."*.

O alerta **não impede** a geração: respondendo **Sim**, o processamento continua normalmente; respondendo **Não**, ele é interrompido para que as pendências sejam tratadas antes. O processamento da apuração de ISS não exibe o alerta.

---

## 7. Resumo dos Campos Novos

### Tabela Config. NF-e (`NFeSetupFNX`)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Transmitir cancelamento automaticamente** (`CancelTransmission`) | Booleano | Cancela a NF-e na SEFAZ no mesmo fluxo interativo que cancela o documento. Ligado nas Config. NF-e novas; desligado nas existentes. |
| **Controlar prazo de cancelamento** (`CancelDeadlineControl`) | Booleano | Bloqueia o cancelamento depois do prazo. Ligado nas Config. NF-e novas; desligado nas existentes. |
| **Prazo de cancelamento (horas)** (`CancelDeadlineHours`) | Inteiro | Horas após a autorização durante as quais a NF-e pode ser cancelada. Padrão 24; mínimo 1. |

### Novas páginas

| Página | Descrição |
|--------|-----------|
| **Cancelamentos de NF-e pendentes** (`NFeCancellationsPending`) | Lista dos documentos cancelados no sistema com NF-e ainda autorizada, com prazo e horas restantes. |
| **Atividades de NF-e** (`NFeCancellationActivities`) | Quadro dos Centros de Função com o indicador **Cancelamentos de NF-e pendentes**. |
