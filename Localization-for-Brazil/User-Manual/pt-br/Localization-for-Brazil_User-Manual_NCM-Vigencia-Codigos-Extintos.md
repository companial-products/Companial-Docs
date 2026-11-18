<!-- PR: #148 -->

# Vigência dos Códigos NCM e Itens com NCM Inválido

**Data**: Agosto/2026

---

## Sumário

O NCM de um item não é permanente: a cada resolução do Gecex, códigos são criados e extintos. Até esta atualização, nada no Business Central indicava que um código cadastrado havia saído da nomenclatura. O problema só aparecia na rejeição da NF-e pela SEFAZ ou na validação da EFD.

Esta atualização acrescenta ao cadastro de NCM os campos **Data Inicial**, **Data Final** e **Fora da Nomenclatura Vigente**, alimentados pelo próprio download do Siscomex. Acrescenta também um aviso no cartão do item cujo NCM não vale mais e a ação **Itens com NCM Inválido**, que reúne numa única lista todos os itens que precisam ser reclassificados.

---

## 1. Contexto

O download de códigos NCM já existente busca a nomenclatura publicada pelo Portal Único Siscomex. Até agora ele aproveitava apenas o código e a descrição de cada registro, embora o arquivo oficial também informe o período de vigência e o ato normativo que criou ou encerrou cada código.

Isso deixava uma lacuna no cadastro: um item classificado com um NCM extinto continuava aparentemente correto no Business Central. Não havia campo, aviso ou relatório que apontasse o problema, e ele se manifestava tarde, em dois pontos igualmente inconvenientes:

- na **transmissão da NF-e**, com a rejeição da SEFAZ por NCM inexistente na tabela vigente;
- na **validação da EFD**, ao conferir o registro de identificação dos itens.

Uma característica do arquivo do Siscomex orienta todo o comportamento descrito neste manual: **ele publica somente a nomenclatura em vigor**. Quando um código é extinto, o arquivo simplesmente deixa de trazê-lo. Não existe registro dizendo que aquele código foi encerrado, nem em que data. Por isso o sistema identifica o código extinto pela **ausência** dele no arquivo, e não apenas pelas datas de vigência.

---

## 2. Como Acessar

| O que | Onde |
| --- | --- |
| Cadastro de NCM | **Códigos NCM** (página `NCMCodesFNX`), por pesquisa (`Alt+Q`) ou pela **Configuração Manual**, categoria Fiscal |
| Atualização a partir do Siscomex | **Download de Dados Mestres** (página `MasterDataDownload`), link **Baixar Códigos NCM** |
| Itens a reclassificar | Ação **Itens com NCM Inválido**, na página **Códigos NCM** |

---

## 3. Como o Download do Siscomex Preenche a Vigência

Em **Download de Dados Mestres**, o link **Baixar Códigos NCM** pede a confirmação *Tem certeza de que deseja substituir a tabela de Códigos NCM pelos códigos baixados do Siscomex?* e, ao final, informa *Atualização de códigos NCM concluída. %1 registros processados.*

A partir desta atualização, cada execução do download faz três coisas:

1. **Grava a vigência de cada código.** A data de início publicada vai para **Data Inicial** e a data de fim para **Data Final**. Quando o Siscomex informa a vigência como aberta (`31/12/9999`), o campo **Data Final** fica em branco. É a mesma convenção já usada nas alíquotas por NCM, em que uma data final vazia significa "sem prazo".

2. **Marca os códigos que saíram da nomenclatura.** Todo código do cadastro que não veio no arquivo recebe a marca **Fora da Nomenclatura Vigente**. Se um código voltar a constar em um download posterior, a marca é retirada automaticamente. Ela nunca fica permanente. Quando algum código é marcado, o sistema informa ao final: *%1 código(s) NCM cadastrado(s) não fazem parte da nomenclatura publicada atualmente pelo Siscomex e foram marcados como tal.*

3. **Compara desconsiderando a pontuação.** O Siscomex publica os códigos com pontos (`3913.90.50`), mas um código digitado manualmente pode ter sido cadastrado sem eles (`39139050`). A comparação usa apenas os dígitos, de modo que a diferença de formatação não faz um código válido ser marcado como extinto.

Duas proteções acompanham esse processo:

- se o download falhar, por indisponibilidade do portal ou resposta inválida, nenhuma marca é alterada;
- se o arquivo vier sem nenhum código, o cadastro também permanece intacto. Uma resposta vazia nunca invalida a tabela inteira.

---

## 4. Quando um NCM é Considerado Inválido

O sistema considera um código inválido em duas situações:

| Situação | Como identificar no cadastro |
| --- | --- |
| O código não faz mais parte da nomenclatura publicada | **Fora da Nomenclatura Vigente** marcado |
| A vigência do código terminou | **Data Final** preenchida com uma data já passada |

No último dia de vigência o código ainda é válido. A invalidez começa no dia seguinte à data informada em **Data Final**.

A marca **Fora da Nomenclatura Vigente** vale para qualquer data, inclusive retroativa. Isso é intencional: como o arquivo do Siscomex não informa quando o código foi extinto, o sistema sabe que ele não vale mais, mas não sabe desde quando. Se essa data for relevante para a empresa, ela pode ser informada manualmente em **Data Final** (veja a seção 7).

Na página **Códigos NCM**, os códigos inválidos aparecem destacados, o que permite conferir o estado do cadastro logo após o download.

---

## 5. Aviso no Cartão do Item

Ao abrir o **Cartão Item** de um item cujo **Código NCM** não é mais válido, o sistema exibe uma notificação no alto da tela. A mensagem identifica o motivo:

| Situação do NCM do item | Mensagem exibida |
| --- | --- |
| Fora da nomenclatura publicada | *O código NCM %1 deste item não faz mais parte da nomenclatura publicada pelo Siscomex e é rejeitado em documentos fiscais.* |
| Vigência encerrada | *O código NCM %1 deste item era válido até %2 e é rejeitado em documentos fiscais emitidos após essa data.* |
| Não cadastrado em **Códigos NCM** | *O código NCM %1 deste item não está cadastrado na lista de Códigos NCM, portanto sua validade não pode ser verificada.* |

O aviso é apenas informativo: **o registro de documentos e a emissão da NF-e não são bloqueados**. A intenção é dar ao usuário a chance de corrigir a classificação antes de faturar, sem travar operações em bases que tenham cadastro antigo.

A notificação é substituída conforme o usuário navega entre itens, de modo que os avisos não se acumulam na tela.

---

## 6. Lista de Itens com NCM Inválido

O aviso no cartão resolve o caso individual, mas não mostra o tamanho do problema. Para isso, a página **Códigos NCM** ganhou a ação **Itens com NCM Inválido**.

A ação varre os itens que têm **Código NCM** preenchido e abre a lista de **Itens** contendo apenas os que precisam de atenção:

- itens classificados com um código inválido, seja por estar fora da nomenclatura, seja por ter a vigência encerrada;
- itens cujo **Código NCM** não existe no cadastro de NCM. Esse caso costuma vir de importação de itens por pacote de configuração, que grava o campo sem passar pela validação da tela, e é igualmente rejeitado pela SEFAZ.

Se não houver nenhum item a corrigir, o sistema informa *Todos os itens estão classificados com códigos NCM válidos.* e não abre a lista.

A lista aberta é a lista padrão de **Itens**, com a coluna **Código NCM** visível, e trabalha sobre os registros reais. É possível abrir o **Cartão Item** a partir dela e corrigir a classificação normalmente, com a alteração sendo gravada. Depois de corrigir os itens, execute a ação novamente para ver a lista atualizada.

---

## 7. Ajuste Manual da Vigência

Os três campos são editáveis na página **Códigos NCM**, e há dois usos legítimos para isso:

- **Informar a data real da extinção.** Como o arquivo do Siscomex não a publica, um código marcado como **Fora da Nomenclatura Vigente** pode receber manualmente a **Data Final** correta (por exemplo, `31/03/2022`, para os códigos extintos na reestruturação daquele ano). O download seguinte não sobrescreve essa data, porque o código não consta no arquivo.
- **Cadastrar um código que o Siscomex não publica**, informando sua vigência à mão.

Vale conhecer a regra de prevalência:

| Código | O que o download faz |
| --- | --- |
| Publicado no arquivo do Siscomex | O Siscomex é a fonte oficial: descrição, **Data Inicial** e **Data Final** são atualizadas conforme o arquivo, sobrescrevendo edições manuais |
| Ausente do arquivo | Apenas a marca **Fora da Nomenclatura Vigente** é aplicada; as datas informadas manualmente são preservadas |

Desmarcar **Fora da Nomenclatura Vigente** à mão em um código ausente do arquivo não é definitivo. O próximo download aplica a marca novamente, porque o código continua fora da nomenclatura publicada.

---

## 8. Exemplo Prático

O arquivo do Siscomex consultado em agosto de 2026 traz um caso real de extinção programada: o código `3913.90.50` (Quitosan), encerrado em **30/09/2026** pela Resolução Gecex nº 926/2026.

Após o download, o cadastro fica assim:

| Código | Data Inicial | Data Final | Fora da Nomenclatura Vigente |
| --- | --- | --- | --- |
| `3913.90.50` | 01/04/2022 | 30/09/2026 | Não |
| `8712.00.10` | 01/04/2022 | *(em branco)* | Não |
| `8473.30.11` (extinto em anos anteriores) | *(preservada)* | *(em branco, se não informada)* | **Sim** |

Consequências para um item classificado com `3913.90.50`:

- **até 30/09/2026**: nenhum aviso. O código é válido, e a **Data Final** preenchida serve como alerta antecipado de que a reclassificação precisa ser planejada;
- **a partir de 01/10/2026**: o cartão do item passa a exibir a notificação de vigência encerrada, e o item aparece na lista de **Itens com NCM Inválido**.

É essa antecipação que dá tempo de trabalhar a reclassificação antes da primeira rejeição da SEFAZ.

---

## 9. Resumo dos Campos Novos

### Tabela Códigos NCM (`NCMCodeFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Data Inicial** (`StartDate`) | Data | Data a partir da qual o código passou a integrar a nomenclatura do Mercosul, conforme publicado pelo Siscomex. |
| **Data Final** (`EndDate`) | Data | Data até a qual o código é válido. Fica em branco quando não há extinção publicada para o código. |
| **Fora da Nomenclatura Vigente** (`NotInCurrentNomenclature`) | Sim/Não | Indica que o código não foi encontrado na nomenclatura baixada mais recentemente do Siscomex, ou seja, não pertence mais à tabela NCM vigente e é rejeitado em documentos fiscais. |

Os três campos são preenchidos pelo download a partir do Siscomex e podem ser ajustados manualmente na página **Códigos NCM**.
