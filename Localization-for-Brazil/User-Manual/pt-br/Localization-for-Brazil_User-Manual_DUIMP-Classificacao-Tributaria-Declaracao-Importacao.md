<!-- PR: #203 -->
# DUIMP: Classificação Tributária (cClassTrib) na Declaração de Importação

**Data**: Setembro/2026

---

## Sumário

Com a Reforma Tributária, toda operação carrega um código de classificação tributária (cClassTrib) que define o tratamento de IBS/CBS. Numa importação, quem informa esse código na DUIMP é o importador ou o despachante — e até agora ele não chegava ao Business Central: a classificação só aparecia na fatura de compra, e era a que a **Matriz Impostos** determinava, não a declarada.

A partir desta versão, cada adição da **Declaração de Importação** mostra a classificação **declarada** na DUIMP, gravada como veio e não editável, ao lado da classificação **adotada** pela empresa, que o analista fiscal confere e corrige e que segue para a linha da fatura de compra. A tela avisa quando o código declarado não está cadastrado, quando a DUIMP declarou mais de um código e quando a classificação adotada não pode ser usada com o NCM da mercadoria — e, neste último caso, a criação do documento de compra fica bloqueada até a correção.

---

## 1. Contexto

### O problema

A conferência fiscal de uma importação acontece na **Declaração de Importação** (`DICardFNX`), antes de a nota de compra existir. A declaração já trazia descrição, NCM, CFOP, quantidades, valores e tributos por adição, mas não a classificação tributária de IBS/CBS. Assim, o que a empresa informou à Receita, na DUIMP, e o que ela escritura, na nota, eram decididos em lugares diferentes, sem que ninguém confrontasse os dois. Quando divergiam, a diferença só aparecia tarde: no cálculo do imposto, na emissão ou numa fiscalização.

### O que mudou

O código declarado passa a ser lido na importação da DUIMP e gravado na adição, separado da classificação que a empresa adota:

| Informação | Papel | Editável |
| --- | --- | --- |
| **Cód. Class. Trib. IBS/CBS Declarado** | Registro do que foi informado à Receita. O sistema nunca altera esse valor nem o valida contra o cadastro | Não |
| **Cód. Class. Trib. IBS/CBS** | A classificação adotada: é a que vai para a linha da fatura de compra e alimenta o cálculo de IBS/CBS | Sim |

Por isso as duas colunas seguem regras diferentes: a declarada aceita qualquer código que a DUIMP trouxer, enquanto a adotada só aceita código existente em **Classificações Tributárias IBS/CBS** (`IBSCBSTaxClassifications`).

### A quem se aplica

- **DUIMPs registradas a partir de 2026**, importadas pela API do Siscomex — são as que trazem o código por item.
- DUIMPs registradas até 2025 e DIs importadas por arquivo XML **não trazem o código**: os campos ficam em branco e o fluxo continua como antes.
- Declarações importadas **antes** desta versão continuam sem classificação declarada; só as novas importações trazem o código.

Para o fluxo completo da declaração — importação, revisão e criação da fatura — consulte [Declaração de Importação (DI/DUIMP)](Localization-for-Brazil_User-Manual_Declaracao-de-Importacao-DI-DUIMP.md).

---

## 2. Pré-requisitos

| Item | Onde configurar |
| --- | --- |
| Cadastro de classificações tributárias carregado | Página **Classificações Tributárias IBS/CBS** (`IBSCBSTaxClassifications`) — veja [Reforma Tributária - Configurações](Localization-for-Brazil_User-Manual_Reforma-Tributaria-Configuracoes.md) |
| Vínculos entre classificação e NCM mantidos, para que o alerta de NCM incompatível funcione | Página **Vínculos Class. Trib. IBS/CBS** (`IBSCBSTaxClassificationLinks`), acessível pela ação **NCMs Vinculados** do cadastro de classificações |
| Importação da DUIMP pela API do Siscomex configurada | **Config. DUIMP Siscomex** (`DUIMPSiscomexSetup`) |
| Opção **Priorizar NCM XML** revisada | **Conf. Importação NF-e** (`NFeImportationSetupFNX`) — define qual NCM vai para a linha da nota e, portanto, qual NCM é conferido (veja [Qual NCM é comparado](#qual-ncm-é-comparado)) |

> **Nota:** se uma classificação não tiver nenhum NCM vinculado, ela é aceita com qualquer NCM. Com o cadastro de vínculos vazio ou incompleto, o alerta de incompatibilidade simplesmente não aparece — não há falso alerta nem bloqueio indevido, mas também não há proteção.

### Permissões

| Conjunto de permissões | O que permite neste recurso |
| --- | --- |
| **BR - NF-e** | Consultar o cadastro de **Classificações Tributárias IBS/CBS** (inclusive pela pesquisa do campo), conferir e corrigir a classificação adotada nas adições e executar a ação **Adotar Classificações Declaradas** |
| **BR - Configurações Fiscais** | Cadastrar e corrigir códigos no cadastro de **Classificações Tributárias IBS/CBS** |

---

## 3. Novas Colunas nas Linhas da Declaração

Na parte **Linhas** (`DISubformFNX`) do cartão da **Declaração de Importação**, ao lado de **Código NCM**, aparecem as colunas:

| Campo | Descrição |
| --- | --- |
| **Cód. Class. Trib. IBS/CBS Declarado** | A classificação tributária (cClassTrib) exatamente como veio da DUIMP. Não editável |
| **Cód. Class. Trib. IBS/CBS** | A classificação que a empresa adota para a adição. Editável, com pesquisa no cadastro de classificações |
| **Aviso de Class. Trib.** | Reúne, numa única coluna, tudo o que precisa ser conferido na classificação da adição antes de o documento ser criado. Não editável |

Duas colunas de apoio ficam ocultas e podem ser exibidas pela personalização da página:

| Campo | Descrição |
| --- | --- |
| **Outros Cód. Class. Trib. Declarados** | Os demais códigos declarados na DUIMP para o item, quando houver mais de um, separados por ponto e vírgula. Não editável |
| **Cód. Class. Trib. Regular IBS/CBS** | A classificação regular, que vale quando o tratamento diferenciado da classificação adotada não se confirma |

> Como os demais campos das linhas, as novas colunas deixam de ser editáveis depois que a declaração gera documento de compra.

---

## 4. Como a Classificação Declarada Chega

Ao importar uma DUIMP pela API do Siscomex (ação **Importar DUIMP por número e versão**, veja [Declaração de Importação (DI/DUIMP)](Localization-for-Brazil_User-Manual_Declaracao-de-Importacao-DI-DUIMP.md#5-importar-a-duimp-pela-api-do-siscomex)), o sistema lê a classificação tributária declarada em cada item e a grava na adição correspondente:

1. O **primeiro** código declarado vai para **Cód. Class. Trib. IBS/CBS Declarado**, exatamente como veio — esteja ele cadastrado ou não.
2. Quando a DUIMP declara **mais de um código** para o mesmo item, os demais ficam em **Outros Cód. Class. Trib. Declarados**, na ordem em que foram declarados. O conjunto inteiro fica registrado como prova do que foi informado.
3. A classificação adotada, **Cód. Class. Trib. IBS/CBS**, nasce igual à declarada **quando o código declarado existe no cadastro**. Quando não existe, ela fica em branco e a adição exibe um aviso.
4. Quando a adotada é preenchida, **Cód. Class. Trib. Regular IBS/CBS** é preenchido junto: se a classificação estiver marcada com **Requer Tributação Regular** no cadastro, a regular recebe o código de tributação integral (`000001`); caso contrário, fica em branco.

> **A importação nunca falha por causa da classificação.** Um código desconhecido ou fora do padrão, ou um item sem a informação, não interrompem a importação: no pior caso a adição fica sem classificação adotada, e o restante da declaração é importado normalmente.

---

## 5. Avisos na Conferência

Cada situação destaca uma coluna, e mais de uma pode acontecer na mesma adição. As mensagens ficam reunidas na coluna **Aviso de Class. Trib.**

| Situação | Destaque | Mensagem em **Aviso de Class. Trib.** |
| --- | --- | --- |
| O código declarado não está cadastrado | **Cód. Class. Trib. IBS/CBS Declarado** em amarelo | *A classificação tributária %1 declarada na DUIMP não está cadastrada. Atualize as classificações tributárias ou escolha um código válido.* |
| A DUIMP declarou mais de um código para o item | — | *A DUIMP também declarou as classificações %1 para este item.* |
| A adotada é diferente da declarada (adição corrigida pelo analista) | **Cód. Class. Trib. IBS/CBS** em vermelho, itálico | — |
| A adotada não pode ser usada com o NCM da mercadoria | **Código NCM** e **Aviso de Class. Trib.** em vermelho, negrito | *A classificação tributária %1 não pode ser usada com o NCM %2.* |

Pontos de atenção:

- A **adição corrigida** é medida contra o **conjunto** declarado. Se a DUIMP declarou mais de um código e o analista adota qualquer um deles, isso não conta como correção e a linha não é destacada. Adotar um código de fora do conjunto, sim.
- A classificação declarada continua exibindo o valor original depois de o analista corrigir a adotada. É esse par — declarada × adotada — que registra a divergência e permite levar o caso ao despachante para pedir a retificação da DUIMP.
- Os avisos são recalculados a cada vez que a linha é exibida: corrigida a classificação, o aviso desaparece.

---

## 6. Código Declarado Não Cadastrado

Quando a DUIMP traz um código que a empresa ainda não tem cadastrado, a adotada fica em branco e a coluna declarada é destacada. É o sinal de que o cadastro precisa ser atualizado — o que é esperado sempre que a Receita publica códigos novos.

A resolução tem dois passos:

1. **Cadastre o código que falta** na página **Classificações Tributárias IBS/CBS** (`IBSCBSTaxClassifications`). Esta parte é o cadastro fiscal de sempre e exige o conjunto de permissões **BR - Configurações Fiscais**.
2. De volta ao cartão da **Declaração de Importação**, clique na ação **Adotar Classificações Declaradas**. O sistema preenche a classificação adotada (e a regular) nas adições cujo código declarado passou a existir no cadastro, e o aviso desaparece. Ao final, uma mensagem informa quantas adições foram reclassificadas: *%1 adições foram reclassificadas.*

Regras da ação **Adotar Classificações Declaradas**:

- Só são reclassificadas adições que ainda **não** geraram documento e que estão com a classificação adotada **em branco**. Adição já classificada pelo analista não é alterada.
- A ação não altera o cadastro de classificações — ela apenas adota, nas adições, códigos que já existem nele.
- Se preferir, o analista pode informar a classificação adotada diretamente na linha, escolhendo um código válido pela pesquisa do campo **Cód. Class. Trib. IBS/CBS**.

---

## 7. Alerta de NCM Incompatível

No cadastro de vínculos (**Vínculos Class. Trib. IBS/CBS**), cada classificação pode ser restrita a determinados NCMs. Quando a classificação adotada na adição tem NCMs vinculados e o NCM da mercadoria não está entre eles, a linha exibe o alerta em vermelho.

### Qual NCM é comparado

O NCM conferido é o que vai **efetivamente** para a linha da nota de compra, pela mesma regra usada na criação do documento:

| **Priorizar NCM XML** em **Conf. Importação NF-e** | NCM comparado |
| --- | --- |
| Ativado | O **Código NCM** da linha da declaração |
| Desativado | O NCM do cadastro do **Item** ou do **Ativo Fixo**; se estiver em branco no cadastro, o **Código NCM** da declaração |

Como é o mesmo NCM que o cálculo do imposto confere mais tarde, o alerta antecipa o problema real, e não uma aproximação.

### Quando o alerta não aparece

- Quando a classificação adotada não tem nenhum NCM vinculado — ela aceita qualquer NCM.
- Quando a classificação adotada está em branco — sem classificação, a nota é classificada pelas regras da empresa, como antes.

Durante a conferência o alerta é apenas um aviso: ele não impede editar a linha e some assim que a classificação é corrigida. O bloqueio acontece na criação do documento, descrita a seguir.

---

## 8. Bloqueio da Criação do Documento

A criação do documento de compra é **bloqueada** enquanto houver qualquer adição com classificação incompatível com o NCM.

1. No cartão da **Declaração de Importação**, clique em **Criar Documento**.
2. Antes de criar qualquer documento, o sistema confere todas as adições que ainda não geraram documento.
3. Havendo incompatibilidade, **nenhum** documento é criado — nem o das adições corretas — e a mensagem nomeia **todas** as adições reprovadas, uma por linha:

   > *Nenhum documento foi criado porque as adições a seguir da Declaração de Importação %1 têm classificação tributária incompatível com o NCM:*
   > *Adição %1, linha %2: a classificação tributária %3 não pode ser usada com o NCM %4.*
   > *Corrija a classificação tributária dessas adições e acione a criação dos documentos novamente.*

4. O analista corrige as adições apontadas e clica em **Criar Documento** novamente.

Pontos de atenção:

- É **tudo ou nada**: não há criação parcial das adições corretas com retenção das incompatíveis.
- Adição com a classificação adotada **em branco não bloqueia**: valem as regras da empresa.
- A restrição que já existia no **cálculo do imposto** continua valendo. O novo bloqueio acontece antes e é mais claro: em vez de um erro de cálculo com a nota já criada, a mensagem nomeia as adições a corrigir antes de qualquer documento existir.

---

## 9. Da Adição para a Fatura de Compra

Ao criar a fatura de compra, cada linha nasce com a **classificação adotada** na adição correspondente, **prevalecendo** sobre o que a **Matriz Impostos** ou a **Exceção Imposto** determinariam. A classificação regular acompanha.

| Situação na adição | Linha da fatura de compra |
| --- | --- |
| **Cód. Class. Trib. IBS/CBS** preenchido | Nasce com a classificação adotada na adição, independentemente da **Matriz Impostos** e da **Exceção Imposto** |
| **Cód. Class. Trib. IBS/CBS** em branco | É classificada pelas regras da empresa (**Matriz Impostos** e **Exceção Imposto**), como antes |

A classificação acompanha todas as linhas geradas:

- linhas de **Item** e de **Ativo Fixo**;
- todas as faturas, quando uma mesma declaração gera mais de uma.

> **Alcance da prevalência.** Ela vale no **momento da criação** da fatura. Se depois o usuário alterar a classificação na linha da nota ou reaplicar a **Matriz Impostos**, vale o que ele fizer — como em qualquer outro documento. O registro do que foi conferido permanece na declaração, no par declarada × adotada.

---

## 10. Exemplo Prático

Uma DUIMP de 2026 com quatro adições de equipamentos é importada pela API do Siscomex.

| Adição | Declarado na DUIMP | Adotado após a importação | Situação |
| --- | --- | --- | --- |
| 1 | `000001` | `000001` | Código declarado existe no cadastro: nada a fazer |
| 2 | `410001` | *(em branco)* | Código não cadastrado: coluna declarada em amarelo e aviso na linha |
| 3 | `000001` | `200002` | O analista corrigiu a classificação: adotada destacada; a declarada continua exibindo `000001` |
| 4 | `000001` e `410001` | `000001` | Dois códigos declarados: o primeiro vai para a coluna, o segundo para **Outros Cód. Class. Trib. Declarados**, e o aviso informa o excedente. Adotar qualquer um dos dois não destaca a linha como corrigida |

Fluxo de trabalho:

1. O responsável pelo cadastro fiscal cadastra o código `410001` em **Classificações Tributárias IBS/CBS**. De volta à declaração, o analista clica em **Adotar Classificações Declaradas**: a adição 2 passa a adotar `410001`, e a mensagem informa que uma adição foi reclassificada.
2. A adição 3, corrigida para `200002`, exibe o alerta de NCM incompatível — essa classificação tem NCMs vinculados que não incluem o do equipamento.
3. O analista clica em **Criar Documento** e recebe o bloqueio nomeando a adição 3. Nenhuma fatura é criada.
4. Ele corrige a classificação da adição 3, clica em **Criar Documento** novamente, e a fatura de compra nasce com a classificação adotada em cada linha.

---

## 11. Resumo dos Campos Novos

### Tabela Linha DI (`DILineFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Cód. Class. Trib. IBS/CBS Declarado** (`DeclaredIBSCBSTaxClassCode`) | Código (20) | A classificação tributária (cClassTrib) exatamente como declarada na DUIMP. Não editável e sem validação contra o cadastro |
| **Outros Cód. Class. Trib. Declarados** (`OtherDeclaredTaxClassCodes`) | Texto (250) | Os demais códigos declarados para o item, separados por ponto e vírgula, quando mais de um foi informado. Não editável |
| **Cód. Class. Trib. IBS/CBS** (`IBSCBSTaxClassCode`) | Código (10) | A classificação adotada pela empresa. Só aceita código existente em **Classificações Tributárias IBS/CBS**; ao ser informada, preenche a classificação regular |
| **Cód. Class. Trib. Regular IBS/CBS** (`IBSCBSRegularTaxClassCode`) | Código (10) | A classificação regular, usada quando o tratamento diferenciado da adotada não se confirma |

### Página Linhas (`DISubformFNX`)

| Coluna | Descrição |
| --- | --- |
| **Aviso de Class. Trib.** (`TaxClassWarning`) | Reúne os avisos de conferência da classificação na adição. Coluna apenas de exibição, não editável |

### Página Declaração de Importação (`DICardFNX`)

| Ação | Descrição |
| --- | --- |
| **Adotar Classificações Declaradas** (`UpdateTaxClassifications`) | Adota o código declarado nas adições ainda sem classificação e sem documento, quando o código existe em **Classificações Tributárias IBS/CBS**. Não altera o cadastro |
