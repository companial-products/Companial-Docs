<!-- Source: manual geral — Configuração da Reforma Tributária (IBS/CBS) na Localização do Brasil -->
<!-- PR: #275, #284, #330 -->
<!-- Branch: feature/rtc-oracle -->
# Reforma Tributária — Configurações

**Data**: Maio/2026

---

## Sumário

A Reforma Tributária do Consumo (EC 132/2023 + LC 214/2025) substitui gradualmente cinco tributos — ICMS, ISS, PIS, COFINS e IPI — por IBS, CBS e IS, em uma transição de 2026 a 2033. A Localização do Brasil já contempla as adaptações necessárias para a primeira fase: novas tabelas de alíquotas e classificações tributárias (cClassTrib), estas importadas e mantidas atualizadas a partir da tabela vigente do portal SVRS DFE, configuração de jurisdições e áreas de impostos, geração automática dos grupos `<IBSCBS>` no XML da NF-e e da NFS-e, e a conferência, antes do registro, do cClassTrib de cada item com a finalidade e o tipo da nota de débito ou de crédito, pela matriz de **Tipos de Nota Class. Trib. IBS/CBS** baixada da Localização. Este manual descreve os conceitos fundamentais, a cadeia de configuração e o impacto na operação diária.

---

## 1. Introdução

### 1.1 O que é a Reforma Tributária e por que ela afeta o Business Central

A Reforma Tributária do Consumo, instituída pela Emenda Constitucional 132/2023 e regulamentada pela Lei Complementar 214/2025, representa a maior reestruturação do sistema tributário brasileiro em décadas. Seu objetivo central é simplificar a tributação sobre o consumo, substituindo cinco tributos diferentes — cada um com suas próprias regras, bases de cálculo e obrigações acessórias — por um modelo unificado, com menos exceções e maior transparência.

Para quem opera com o Business Central, isso significa que o sistema precisa ser capaz de calcular e declarar corretamente os novos tributos **ao mesmo tempo** em que continua apurando os tributos atuais, já que a transição é gradual e os dois modelos coexistirão até 2033.

A Localização do Brasil já contempla as adaptações necessárias para esta primeira fase da reforma, incluindo novas tabelas de dados, novos campos nos documentos fiscais e a geração dos grupos de informação exigidos no XML da NF-e e NFS-e.

### 1.2 O que muda: de cinco tributos para três

O modelo atual distribui a tributação sobre o consumo entre cinco tributos de esferas diferentes. A reforma consolida esse modelo em três novos tributos:

| Tributo Atual | Esfera | Substituído por |
|---|---|---|
| ICMS | Estadual | IBS — parte estadual |
| ISS | Municipal | IBS — parte municipal |
| PIS | Federal | CBS |
| COFINS | Federal | CBS |
| IPI | Federal | IS (Imposto Seletivo — apenas para produtos específicos nocivos à saúde ou ao meio ambiente) |

Os novos tributos são:

| Sigla | Nome completo | Quem arrecada |
|---|---|---|
| **IBS** | Imposto sobre Bens e Serviços | Estados e Municípios (subnacional) |
| **CBS** | Contribuição sobre Bens e Serviços | União Federal |
| **IS** | Imposto Seletivo | União Federal |

O IBS funciona de forma integrada entre estados e municípios: cada esfera define sua própria alíquota, mas a cobrança é unificada. Isso explica por que, no Business Central, o IBS aparece dividido em duas jurisdições distintas — **IBS UF** (estadual) e **IBS Municipal** — cada uma com sua alíquota independente.

### 1.3 A transição gradual (2026–2033)

A extinção dos tributos antigos não acontece de uma vez. O cronograma é gradual e exige que a empresa opere com os dois sistemas em paralelo durante anos:

- **2026:** Início da cobrança de IBS e CBS com alíquotas de teste (muito baixas). PIS, COFINS, ICMS e ISS continuam normalmente.
- **2027–2028:** Alíquotas de IBS e CBS aumentam progressivamente.
- **2029–2032:** ICMS, ISS, PIS e COFINS começam a ser reduzidos gradualmente.
- **2033:** Extinção completa dos tributos antigos. Apenas IBS, CBS e IS permanecem.

Na prática, durante todo esse período, **cada nota fiscal emitida precisará calcular e declarar tanto os tributos do modelo antigo quanto os do modelo novo**. O Business Central trata esse cenário de coexistência automaticamente, desde que as configurações estejam corretas.

### 1.4 O que foi implementado na Localização do Brasil

As adaptações para a Reforma Tributária concentram-se em duas frentes:

**Estrutura de dados:** foram criadas novas tabelas para suportar os conceitos que não existiam antes — as alíquotas específicas de IBS e CBS, e os códigos de classificação tributária (cClassTrib) exigidos pelo Fisco para identificar o enquadramento fiscal de cada operação.

**Documentos fiscais eletrônicos:** o XML da NF-e passou a incluir novos grupos de informação para declarar os valores de IBS e CBS por item, conforme o layout técnico definido pelos órgãos fiscais. Esses grupos são gerados automaticamente a partir das configurações feitas nas novas tabelas. 

> **Importante:** O Governo ainda não definiu tecnicamente como IBS e CBS devem aparecer na DANFE (o documento impresso). Por isso, a DANFE ainda não reflete a Reforma Tributária — apenas o XML é afetado neste momento.

---

## 2. Conceitos Fundamentais

Antes de configurar qualquer coisa no sistema, é importante entender alguns conceitos que são a base de todo o funcionamento da reforma no Business Central. O mais crítico deles é o **cClassTrib** — ele é a peça central que conecta todas as configurações.

### 2.1 O cClassTrib: o código que governa tudo

O **cClassTrib** (Código de Classificação Tributária) é um código de 6 dígitos criado pela Reforma Tributária para identificar o enquadramento fiscal de cada operação em relação ao IBS e à CBS. Ele substitui a lógica que antes era controlada pelo CST isoladamente e passa a determinar, de forma única e padronizada:

- Se a operação é **tributada integralmente**, com **alíquota reduzida**, **isenta**, **diferida** ou **imune**
- Qual o **percentual de redução** aplicável sobre a alíquota padrão
- Se a operação exige a declaração de **tributação regular** em paralelo (casos como Zona Franca de Manaus ou suspensão de tributo)

#### Anatomia do código

O código tem 6 dígitos divididos em dois blocos de três:

```
             cClassTrib
      ┌─────────────────────────
      │  [ X  X  X   X  X  X ]
      │    ↑  ↑  ↑   ↑  ↑  ↑
      │    1  2  3   4  5  6
      └─────┬─────────────┬─────    
        Dígitos       Dígitos
          1–3           4–6
        ─────────     ──────────
        CST           Tipo de
        IBS/CBS       benefício
        Situação      ou trata-
        tributária    mento
        principal     específico
```

Os **três primeiros dígitos** definem a situação tributária principal — a "categoria" da operação. Os **três últimos** especificam o tratamento dentro dessa categoria: o tipo de benefício fiscal, o anexo da lei aplicável ou a natureza específica da operação.

#### Os grupos de CST (primeiros 3 dígitos)

| CST | Situação tributária |
|---|---|
| **000** | Tributação integral — IBS e CBS cobrados pela alíquota cheia |
| **010 / 011** | Tributação com alíquotas uniformes (com ou sem redução) |
| **200** | Alíquota reduzida — aplica um percentual de desconto sobre a alíquota padrão |
| **300** | Isenção |
| **410** | Imunidade ou não incidência |
| **510** | Diferimento — o tributo existe, mas o recolhimento é postergado |

#### Lendo um código na prática

```
  ┌─────────────────────────────────────────────────────────────────┐
  │  000001  →  Tributação integral                                 
  ├─────────────────────────────────────────────────────────────────┤
  │                                                                 
  │         0  0  0         0  0  1                                 
  │         ───────         ───────                                   
  │           CST             Trat.                                 
  │                                                                 
  │   000  =  Tributação integral                                   
  │   001  =  Situações tributadas integralmente pelo IBS e CBS     
  │                                                                 
  └─────────────────────────────────────────────────────────────────┘


  ┌─────────────────────────────────────────────────────────────────┐
  │  200034  →  Alíquota reduzida (alimentos)                       
  ├─────────────────────────────────────────────────────────────────┤
  │                                                                 
  │         2  0  0         0  3  4                                 
  │         ───────         ───────                                 
  │           CST             Trat.                                 
  │                                                                 
  │   200  =  Alíquota reduzida                                     
  │   034  =  Fornecimento de alimentos ao consumo humano           
  │            (Anexo VII) — redução de 60% sobre a alíquota        
  │                                                                 
  └─────────────────────────────────────────────────────────────────┘


  ┌─────────────────────────────────────────────────────────────────┐
  │  510002  →  Diferimento total (insumos agropecuários)           
  ├─────────────────────────────────────────────────────────────────┤
  │                                                                
  │         5  1  0         0  0  2                                 
  │         ───────         ───────                                 
  │           CST             Trat.                                 
  │                                                                 
  │   510  =  Diferimento                                           
  │   002  =  Insumos agropecuários para produtor rural             
  │            contribuinte (Anexo IX) — diferimento total          
  │                                                                 
  └─────────────────────────────────────────────────────────────────┘


  ┌─────────────────────────────────────────────────────────────────┐
  │  410999  →  Imunidade / Não incidência                          
  ├─────────────────────────────────────────────────────────────────┤
  │                                                                 
  │         4  1  0         9  9  9                                 
  │         ───────         ───────                                 
  │           CST             Trat.                                 
  │                                                                 
  │   410  =  Imunidade ou não incidência                           
  │   999  =  Operações não onerosas sem previsão de tributação     
  │                                                                 
  └─────────────────────────────────────────────────────────────────┘
```

**Por que esse código é tão importante no BC?** Porque é o preenchimento do cClassTrib em cada linha do documento fiscal que aciona o cálculo dos novos tributos e a geração das tags correspondentes no XML. Sem ele, o sistema não calcula IBS nem CBS, e a NF-e fica fora do padrão exigido a partir de 2026.

### 2.2 Não cumulatividade plena

No sistema atual, os créditos de PIS e COFINS têm diversas restrições — nem toda aquisição gera crédito, e as regras variam conforme o regime tributário e o tipo de gasto. O IBS/CBS adota o princípio da **não cumulatividade plena**: praticamente toda aquisição tributada pelo IBS/CBS gera crédito correspondente, independente de sua natureza. As exceções são poucas e explicitamente listadas na legislação.

### 2.3 Alíquota no destino

Diferente do ICMS — que tem uma parcela cobrada no estado de origem da mercadoria — o IBS/CBS é cobrado integralmente no **destino**, ou seja, no estado e município onde o bem é consumido ou o serviço é prestado. Isso tem impacto nas alíquotas aplicadas: elas correspondem à UF e ao município do destinatário, não do emitente.

### 2.4 Split Payment (futuro)

O Split Payment é um mecanismo previsto na legislação em que o próprio sistema financeiro reterá e repassará automaticamente o valor do IBS e CBS no momento do pagamento da nota, sem que a empresa precise recolher depois. 

> Este recurso **ainda esta sendo regulamentado pelo governo** e, portanto, **não está implementado na localização**.

### 2.5 Cashback (futuro)

O Cashback é a devolução de parte do IBS/CBS a consumidores de baixa renda, como forma de progressividade do novo sistema. 

> Este recurso **ainda esta sendo regulamentado pelo governo** e, portanto, **não está implementado na localização**.

---

## 3. Como Tudo se Conecta no Business Central

Antes de entrar nas configurações individuais, é fundamental entender o fluxo completo — como cada tabela alimenta a próxima e como o resultado final aparece no documento fiscal e no XML.

### 3.1 A cadeia de configuração

O Business Central opera com IBS e CBS a partir de uma cadeia de dependências que funciona assim:

```
Alíquotas IBS/CBS
       ↓
Jurisdições de Impostos  →  Áreas de Impostos
       ↓
Classificações Tributárias (cClassTrib)
       ↓
Vínculos Class. Trib. (por NCM / Natureza de Operação)
       ↓
Documento Fiscal (linha do item)
       ↓
Cálculo automático → XML da NF-e
```

Cada elo desta cadeia tem uma função específica:

- **Alíquotas IBS/CBS** armazena as alíquotas vigentes por tributo e por período. O sistema busca aqui o percentual a aplicar quando calcula o imposto.
- **Jurisdições e Áreas de Impostos** registram IBS UF, IBS Municipal e CBS como jurisdições reconhecidas pelo sistema e as associam às áreas de impostos que já existem para vendas e compras.
- **Classificações Tributárias (cClassTrib)** definem o enquadramento de cada operação: se há redução, diferimento, isenção, e qual alíquota se aplica.
- **Vínculos por NCM** relacionam automaticamente cada código NCM (ou Natureza de Operação) ao cClassTrib correspondente, evitando que o usuário precise preencher esse dado manualmente em cada item.
- **Documento Fiscal** é onde tudo converge: com o cClassTrib presente na linha do item, o sistema calcula os valores de IBS UF, IBS Municipal e CBS, e ao gerar o XML da NF-e, inclui os grupos `<IBSCBS>` com todos os subgrupos exigidos pelo Fisco.

### 3.2 O papel de cada função auxiliar

A localização fornece funções que automatizam o preenchimento dessas tabelas a partir de fontes oficiais do governo. A utilização das funções não é obrigatória, mas eliminam trabalho manual e reduzem o risco de erros de digitação. São cinco funções principais:

| Função | Onde acessar | O que faz |
|---|---|---|
| **Inserir Alíquotas de Referência** | Alíquotas IBS/CBS | Importa automaticamente as alíquotas de referência vigentes (CBS 0,90%, IBS UF 0,10%, IBS Mun 0,00%) |
| **Baixar Classificações Tributárias...** | Classificações Tributárias IBS/CBS | Importa todos os códigos cClassTrib com CST, descrição, tipo de alíquota e percentuais de redução; numa tabela já carregada, atualiza os códigos existentes, inclui os novos e marca os que saíram da tabela oficial |
| **Inserir Dados da Reforma Tributária...** | Jurisdições de Impostos (Brasil) | Cria as jurisdições CBS, IBS UF e IBS Mun e as insere nas áreas de impostos existentes |
| **Baixar Vínculos de NCM...** | Vínculos Class. Trib. IBS/CBS | Importa os vínculos entre NCMs e seus respectivos cClassTrib conforme a legislação |
| **Baixar Matriz da Localização...** | Tipos de Nota Class. Trib. IBS/CBS | Importa, do arquivo publicado pela Localização, os pares de cClassTrib e tipo de nota de débito ou de crédito exigidos pela NT 2025.002 (regras UB14) e ativa a conferência desses pares no registro da NF-e (seção 4.7) |

A sequência correta de execução é justamente a ordem em que as tabelas aparecem na cadeia acima: primeiro alíquotas, depois jurisdições, depois classificações, e por fim vínculos. A matriz de tipos de nota não depende das demais e pode ser baixada a qualquer momento.

### 3.3 Ativação da Reforma Tributária

Toda essa estrutura só entra em operação quando o **Status Reforma Tributária** estiver ativo nas **Configurações Fiscais**. Esse campo funciona como um interruptor global:

| Status | Comportamento |
|---|---|
| **Agendada** (padrão) | O sistema ativa a reforma automaticamente em 01/05/2026 |
| **Habilitada** | Ativo imediatamente — recomendado para testes e homologação |
| **Desabilitada** | Nenhuma funcionalidade da reforma está ativa |

> Com o status como **Agendada** ou **Habilitada**, o sistema passa a exigir o preenchimento do campo **Cód. Class. Trib. IBS/CBS** nas linhas do documento fiscal e os cálculos de IBS/CBS são ativados.
>

### 3.4 Novas Classificações de Imposto no BC

Para suportar os novos tributos, a localização adiciona quatro novas classificações de imposto ao sistema. Elas funcionam da mesma forma que as classificações já existentes (como ICMS, PIS, COFINS) e são usadas nas jurisdições, áreas de impostos e contabilizações:

| Classificação | Tributo correspondente |
|---|---|
| **IBSUF** | IBS — parcela estadual |
| **IBSMun** | IBS — parcela municipal |
| **CBS** | Contribuição sobre Bens e Serviços (Federal) |
| **IS** | Imposto Seletivo (Federal) |

### 3.5 O que aparece no XML da NF-e

Quando o cClassTrib está preenchido na linha do documento e a reforma está ativa, a localização gera automaticamente os seguintes grupos no XML da NF-e para cada item:

```xml
<imposto>
  <IS>                    <!-- Imposto Seletivo, quando aplicável -->
  <IBSCBS>
    <CST>                 <!-- Situação tributária principal (3 primeiros dígitos do cClassTrib) -->
    <cClassTrib>          <!-- Código de classificação tributária completo (6 dígitos) -->
    <gIBSCBS>
      <gIBSUF>            <!-- IBS estadual: alíquota, redução percentual, diferimento -->
      <gIBSMun>           <!-- IBS municipal: alíquota, redução percentual -->
      <gCBS>              <!-- CBS federal: alíquota, redução percentual, diferimento -->
      <gEstornoCred>      <!-- Estorno de crédito, quando aplicável -->
    </gIBSCBS>
  </IBSCBS>
</imposto>
```

O conteúdo desses grupos — alíquotas, percentuais de redução, indicadores de diferimento — é preenchido automaticamente com base nas configurações feitas nas tabelas de Alíquotas e Classificações Tributárias.

---

## 4. Configurações

Esta seção detalha cada tabela e configuração necessária, explicando a função de cada campo e como eles se relacionam. Siga a ordem indicada — cada etapa depende da anterior.

> **Recomendação:** Use as funções auxiliares disponíveis em cada página. Elas importam dados oficiais do governo e eliminam o risco de erro no preenchimento manual.

### 4.1 Alíquotas IBS/CBS

**Caminho no BC:** `Alíquotas IBS/CBS`

Esta é a tabela que guarda as alíquotas vigentes para IBS e CBS. O sistema a consulta no momento do cálculo dos impostos em cada documento fiscal. Como as alíquotas mudam ao longo do período de transição (2026–2033), a tabela permite múltiplos registros para o mesmo tributo com datas de validade diferentes — o sistema aplica sempre a alíquota vigente na data do documento.

| Campo | O que representa |
|---|---|
| **Data Inicial** | A partir de quando esta alíquota passa a valer |
| **Tipo de Alíquota** | **Referencial** (alíquota única nacional, definida pelo Comitê Gestor) ou **Padrão** (alíquota definida por cada UF ou Município) |
| **Tipo Imposto** | Qual tributo esta alíquota representa: CBS, IBS UF ou IBS Municipal |
| **Código** | Código da UF ou Município a que esta alíquota se refere (apenas para o tipo Padrão) |
| **Nome** | Nome da UF ou Município |
| **Alíquota** | O percentual em si |

**Referencial vs. Padrão:** A alíquota Referencial é uma alíquota única nacional, definida pelo Comitê Gestor do IBS, usada como base para todas as operações. A alíquota Padrão é definida individualmente por cada estado ou município. Em 2026, apenas as alíquotas referenciais serão aplicadas — as alíquotas padrão específicas por UF/Município entrarão em vigor em etapas posteriores.

#### Como popular esta tabela: Inserir Alíquotas de Referência

Em vez de digitar manualmente cada registro, use a função **Inserir Alíquotas de Referência**, disponível no menu **Ações** da página. Com um clique, a tabela é populada com as alíquotas de referência vigentes conforme a legislação:

| Tributo | Tipo | Alíquota |
|---|---|---|
| CBS | Referencial | 0,90% |
| IBS UF | Referencial | 0,10% |
| IBS Municipal | Referencial | 0,00% |

> As alíquotas de referência de 2026 são de transição e intencionalmente baixas. Elas serão atualizadas pela legislação ao longo dos anos seguintes.

---

### 4.2 Classificação Tributária IBS/CBS (cClassTrib)

**Caminho no BC:** `Classificações Tributárias IBS/CBS`

Esta é a tabela central da Reforma Tributária no Business Central. Ela armazena todos os códigos cClassTrib — um catálogo oficial disponibilizado pelo governo com centenas de classificações, cada uma representando um enquadramento fiscal diferente para operações com IBS e CBS.

Pense nela como um "dicionário de tributação": para cada tipo de operação possível, existe um código que diz ao sistema como tratar aquela operação — se tributa integralmente, se reduz a alíquota, se há diferimento, se é isenta.

| Campo | O que representa |
|---|---|
| **Código** | O código cClassTrib de 6 dígitos (ex: `000001`, `200034`, `510002`) |
| **Descrição CST** | A descrição da situação tributária principal (os 3 primeiros dígitos) |
| **Descrição cClassTrib** | A descrição completa e específica do enquadramento (ex: "Fornecimento de alimentos destinados ao consumo humano — Anexo VII") |
| **Tipo de Alíquota** | Como a alíquota será aplicada: **Padrão** (alíquota cheia ou com redução), **Referência** (usa a alíquota de referência, como nos códigos de alíquota uniforme nacional), **Sem Alíquota** (isenção, imunidade, não incidência) ou em branco, quando o tipo de alíquota publicado para o código ainda não é calculado pela localização (veja [Tipo de Alíquota](#tipo-de-alíquota)) |
| **Redução IBS e CBS** | Percentual de desconto sobre a alíquota padrão. Se `60,00`, significa que o IBS e CBS são cobrados sobre 40% da base (redução de 60%) |
| **Requer Tributação Regular** | Quando marcado, indica que a operação também exige o preenchimento do grupo `gTribRegular` no XML — necessário em casos como Zona Franca de Manaus, suspensão de tributo ou operações com condições suspensivas/resolutórias |
| **Fora da Tabela Vigente** | Marcado quando o código não constava na última tabela baixada do portal. O código continua podendo ser usado; a lista apenas o oculta por padrão (veja [Códigos fora da tabela vigente](#códigos-fora-da-tabela-vigente)) |

#### Como popular esta tabela: Baixar Classificações Tributárias

Use a função **Baixar Classificações Tributárias...**, disponível no menu **Ações**, para importar automaticamente todos os códigos cClassTrib a partir da tabela vigente publicada no portal SVRS DFE (`https://dfe-portal.svrs.rs.gov.br/DFE/classificacaotributaria`). O preenchimento manual não é viável dada a quantidade de códigos existentes.

A mesma função serve tanto para a primeira carga quanto para atualizar uma tabela já carregada, e pode ser executada sempre que o governo publicar uma nova versão da tabela:

| Situação do código | O que a importação faz |
|---|---|
| Código do arquivo que ainda não existe no BC | É incluído |
| Código do arquivo que já existe no BC | Os dados vindos do portal são atualizados: **Descrição CST**, **Descrição cClassTrib**, **Tipo de Alíquota**, **% Redução IBS**, **% Redução do CBS**, **Requer Tributação Regular**, **Estorno de Crédito**, **% Diferido IBS** e **% Diferido CBS** |
| Código que existe no BC, mas não consta no arquivo | Não é apagado: recebe a marca **Fora da Tabela Vigente** |
| Código marcado que volta a constar no arquivo | A marca **Fora da Tabela Vigente** é retirada |

Nenhum código é apagado pela importação. Por isso, os **Vínculos** de NCM já cadastrados (seção 4.5) e os documentos que usam os códigos permanecem como estão.

Os códigos com diferimento no portal são gravados com 100% em **% Diferido IBS** e **% Diferido CBS**; os demais, com zero.

> **Atenção:** os campos vindos do portal são sempre sobrescritos na importação, inclusive um **Tipo de Alíquota** alterado manualmente (por exemplo, para **Referência**). Se a empresa ajustou algum desses campos à mão, confira-os depois de cada importação.

> **Após atualizar a localização:** a atualização do app não altera os códigos já cadastrados. Os códigos novos, as descrições revisadas e o tipo de alíquota lido do portal só passam a valer depois que a função **Baixar Classificações Tributárias...** é executada novamente.

Se o arquivo baixado estiver vazio, nenhum código é marcado. Se o arquivo for inválido, a importação é interrompida com erro e nenhuma alteração é gravada.

#### Tipo de Alíquota

O **Tipo de Alíquota** de cada código é gravado conforme a coluna "Tipo de Alíquota" publicada no portal:

| Tipo de alíquota no portal | **Tipo de Alíquota** no BC | Efeito no cálculo |
|---|---|---|
| 2 - Padrão | **Padrão** | IBS e CBS calculados pela alíquota vigente, com a redução do código |
| 3 - Sem alíquota | **Sem Alíquota** | Sem IBS e CBS |
| 4 - Uniforme Nacional | **Referência** | IBS e CBS calculados pela alíquota de referência, com a redução do código, qualquer que seja a alíquota própria da UF ou do Município |
| 1 - Fixa, 5 - Uniforme Setorial | Em branco | Sem IBS e CBS, porque esses tipos de alíquota ainda não são calculados pela localização (veja abaixo) |

Os códigos de isenção (CST 400) e de imunidade e não incidência (CST 410) são sempre gravados como **Sem Alíquota**, qualquer que seja o tipo publicado no portal.

**Tipo de Alíquota em branco.** Alíquota fixa e uniforme setorial (CSTs 010, 220 e 221) ainda não são calculadas pela localização. Para não gerar um valor errado, a linha do documento com um desses códigos fica com a base de cálculo, mas sem alíquota e sem valor de IBS e CBS — mesmo que uma alíquota tenha sido definida para o NCM ou na **Exceção Imposto**.

**Tipo de Alíquota Referência.** É o tipo gravado para os códigos de alíquota uniforme nacional (CST 011: planos de assistência à saúde, planos de assistência funerária e concursos de prognósticos). Nesses códigos a LC 214/2025 fixa a mesma alíquota no país inteiro, a de referência de cada esfera, com a redução do código quando houver. Por isso o cálculo usa somente as alíquotas de referência cadastradas e desconsidera a alíquota própria que a UF ou o Município de destino tenha. O tipo também pode ser escolhido manualmente em outro código, lembrando que a próxima importação grava novamente o tipo do portal.

#### Códigos fora da tabela vigente

Quando a importação marca algum código com **Fora da Tabela Vigente**, uma mensagem informa quantos códigos estão nessa situação e que a lista os oculta por padrão. Na página **Classificações Tributárias IBS/CBS**:

- A lista abre mostrando somente os códigos vigentes. Isso vale também quando a página é aberta para escolher um código a partir de um campo, como o **Cód. Class. Trib. IBS/CBS** na linha do documento ou na **Matriz Imposto**.
- Os códigos marcados aparecem com o **Código** e a **Descrição cClassTrib** destacados em vermelho.
- A ação **Mostrar Todos os Códigos** (grupo **Processo**) exibe também os códigos marcados; a ação **Mostrar Somente Códigos Vigentes** volta ao filtro inicial. O filtro também pode ser removido diretamente no painel de filtros da lista.
- Se a página for aberta a partir de um campo que já contém um código marcado, ela abre mostrando esse código, sem o filtro.
- O campo **Fora da Tabela Vigente** pode ser marcado ou desmarcado manualmente. Na próxima importação, ele volta a refletir a tabela do portal.

A marca não bloqueia nada: um código fora da tabela vigente continua podendo ser digitado nos documentos e cadastros, e os documentos e vínculos que já o usam não são afetados. Códigos criados manualmente pela empresa, que não existem no portal, também recebem a marca na próxima importação.

Após a importação, a tabela ficará assim (exemplos):

| Código | Descrição CST | Enquadramento | Tipo Alíquota | % Red. IBS | % Red. CBS |
|---|---|---|---|---|---|
| 000001 | Tributação integral | Situações tributadas integralmente | Padrão | — | — |
| 011001 | Tributação c/ alíquotas uniformes reduzidas | Planos de assistência funerária | Referência | 60,00 | 60,00 |
| 200034 | Alíquota reduzida | Fornecimento de alimentos — consumo humano (Anexo VII) | Padrão | 60,00 | 60,00 |
| 510001 | Diferimento | Operações, sujeitas a diferimento, com energia elétrica | Sem Alíquota | — | — |
| 410999 | Imunidade / Não incidência | Operações não onerosas sem previsão de tributação | Sem Alíquota | — | — |

---

### 4.3 Jurisdições de Impostos (Brasil) e Áreas de Impostos (Brasil)

**Caminho no BC:** `Jurisdições de Impostos (Brasil)` / `Áreas de Impostos (Brasil)`

Para que o Business Central reconheça o IBS e a CBS como tributos válidos e os inclua nos cálculos de documentos fiscais, eles precisam estar registrados como **Jurisdições de Imposto** e associados às **Áreas de Impostos** que controlam quais tributos incidem em cada tipo de operação.

Pense nas **Jurisdições** como o cadastro de cada tributo no sistema, e como cada um é calculado (base de cálculo, tipo de cálculo, etc.).

As **Áreas de Impostos** agrupam jurisdições. Cada empresa tem áreas configuradas para vendas, compras, etc. A reforma exige que as novas jurisdições sejam adicionadas a essas áreas existentes — sem isso, o IBS e a CBS simplesmente não aparecem nos documentos.

#### Como configurar: Inserir Dados da Reforma Tributária

A função **Inserir Dados da Reforma Tributária...**, disponível na página de Jurisdições, realiza automaticamente três ações em sequência:

1. **Cria as Jurisdições** de CBS, IBS UF e IBS Municipal com os parâmetros corretos de cálculo
2. **Cria as Contabilizações de Imposto** associadas a cada jurisdição, para que os valores de IBS/CBS sejam contabilizados nas contas corretas
3. **Insere as novas jurisdições nas Áreas de Impostos existentes** — inclui as jurisdições dos novos impostos em áreas já configuradas no sistema. Tanto para compras, quanto para vendas.

> **Detalhe técnico:** As jurisdições CBS e IBS são criadas **sem alíquotas fixas**. Isso é intencional: o sistema vai buscar as alíquotas na tabela de Alíquotas IBS/CBS (configurada na etapa 4.1), aplicando sempre a alíquota vigente na data do documento. Esse mecanismo garante que as atualizações de alíquota ao longo da transição sejam automáticas — basta inserir a nova alíquota com a nova data de validade na tabela, sem precisar alterar cada jurisdição manualmente.

> **Coexistência durante a transição:** Como IBS/CBS entram em vigor enquanto ICMS, PIS e COFINS ainda existem, a função não cria áreas de impostos novas — ela adiciona as novas jurisdições nas áreas existentes. Isso preserva toda a configuração atual e simplesmente acrescenta os novos tributos.

---

### 4.4 Preenchimento Automático do cClassTrib (Matriz Imposto e Exceção Imposto)

**Caminho no BC:** `Matriz Imposto` / `Exceção Imposto`

Com as tabelas base configuradas, o próximo passo é garantir que o campo **Cód. Class. Trib. IBS/CBS** seja preenchido automaticamente nas linhas dos documentos fiscais — sem que o usuário precise digitar o código em cada item manualmente.

É aqui que entram a **Matriz Imposto** e a **Exceção Imposto**, que já existiam no sistema para controlar CST, alíquotas de ICMS, PIS e COFINS. Para a Reforma Tributária, dois novos campos foram adicionados a essas tabelas.

#### Matriz Imposto

A Matriz Imposto é a regra geral: para cada combinação de fatores fiscais (tipo de operação, NCM, cliente/fornecedor, estado, etc.), ela define qual CST usar, qual alíquota aplicar — e agora, qual cClassTrib associar.

Os dois novos campos são:

| Campo | Quando usar |
|---|---|
| **Cód. Class. Trib. IBS/CBS** | O cClassTrib padrão para esta combinação fiscal. É o código que aparecerá automaticamente nas linhas de documentos que se enquadrem nesta regra |
| **Cód. Class. Trib. Regular IBS/CBS** | Usado apenas em operações que, além do tratamento normal (redução, diferimento, etc.), também precisam declarar a tributação regular em paralelo — como ocorre em operações com Zona Franca de Manaus ou em cenários de suspensão de tributo. Este campo deve ser preenchido somente quando o **Cód. Class. Trib. IBS/CBS** tiver a flag  `Requer Tributação Regular` marcada, e nunca com o mesmo código do **Cód. Class. Trib. IBS/CBS** (veja [Linha bloqueada pelo Cód. Class. Trib. Regular IBS/CBS](#linha-bloqueada-pelo-cód-class-trib-regular-ibscbs)). A alíquota da tributação regular informada na nota é a deste código: a **% Redução IBS** e a **% Redução do CBS** dele são aplicadas sobre a alíquota do período, e um código com **Tipo de Alíquota** **Sem Alíquota** resulta em tributação regular zero. |

#### Exceção Imposto

A Exceção Imposto é o mecanismo para situações específicas que precisam de tratamento diferente do padrão. Funciona como uma sobreposição: quando uma linha de documento se enquadra em uma exceção, o cClassTrib definido aqui prevalece sobre o da Matriz Imposto.

Isso é útil, por exemplo, quando um mesmo produto tem classificação tributária diferente dependendo do cliente (pessoa física vs. pessoa jurídica, consumidor final vs. revendedor) ou da natureza da operação.

> **Regra de prioridade:** Exceção Imposto > Matriz Imposto. Se um registro da Exceção se aplica à operação, ele é usado. Caso contrário, o sistema cai para a Matriz.

> **Importante**
> Se Matriz de Imposto não é utilizada, o código **Cód. Class. Trib. IBS/CBS** deve ser preenchido manualmente nas linhas do documento.

#### Linha bloqueada pelo Cód. Class. Trib. Regular IBS/CBS

O **Cód. Class. Trib. Regular IBS/CBS** segue as mesmas regras na **Matriz Impostos**, na **Exceção Imposto** e na linha do documento:

| Situação | Aceito? |
|---|---|
| Campo em branco | Sim |
| Igual ao **Cód. Class. Trib. IBS/CBS** | Não. A classificação regular é a que se aplica quando o tratamento diferenciado não se aplica, por isso precisa ser outro código |
| Preenchido, e o **Cód. Class. Trib. IBS/CBS** não tem **Requer Tributação Regular** marcado na página **Classificações Tributárias IBS/CBS** | Não. O campo deve ficar em branco |
| Preenchido com outro código, e o **Cód. Class. Trib. IBS/CBS** tem **Requer Tributação Regular** marcado | Sim |

Quem digita o código direto no campo recebe o erro na hora, no próprio campo.

Uma **Matriz Impostos** ou **Exceção Imposto** gravada antes dessas regras pode ter um par que elas não aceitam, como o regular igual ao principal. Esse cadastro só é recusado quando os códigos dele são copiados para uma linha de venda, de compra ou de documento fiscal: ao informar o item na linha, quando a Matriz Impostos é aplicada, ou ao informar o **Cód. Exceção Imposto** na linha. O item não tem campo de classificação IBS/CBS. Os códigos vêm da Exceção Imposto informada no **Cód. Exceção Imposto** do item ou da matriz, ou então da própria Matriz Impostos.

Nesses casos a linha não é gravada, e o erro diz de qual cadastro vêm os códigos e o que corrigir. Exemplos com a Exceção Imposto `TESTE-REG`:

| Problema no cadastro | Mensagem |
|---|---|
| Regular igual ao principal | *Os códigos de classificação IBS/CBS da linha vêm da Exceção Imposto TESTE-REG, onde o Cód. Class. Trib. Regular IBS/CBS é igual ao Cód. Class. Trib. IBS/CBS 000001. Abra a Exceção Imposto e altere ou limpe o Cód. Class. Trib. Regular IBS/CBS: ele deve ser diferente do Cód. Class. Trib. IBS/CBS.* |
| Regular preenchido para uma classificação que não exige tributação regular | *Os códigos de classificação IBS/CBS da linha vêm da Exceção Imposto TESTE-REG, onde o Cód. Class. Trib. IBS/CBS 000001 não exige tributação regular, mas o Cód. Class. Trib. Regular IBS/CBS é 200022. Abra a Exceção Imposto e limpe o Cód. Class. Trib. Regular IBS/CBS.* |

Quando o par vem da matriz, a mensagem cita a **Matriz Impostos** e o código dela no lugar da Exceção Imposto.

O erro traz a ação **Abrir Exceção Imposto** ou **Abrir Matriz Impostos**, que abre a página **Exceções Imposto** (`TaxExceptionsFNX`) ou **Matriz Impostos** (`TaxesMatrixFNX`) já no registro citado. Para liberar a linha:

1. Clique na ação do erro. Outra opção é abrir a página e localizar o código citado na mensagem.
2. No registro, limpe o **Cód. Class. Trib. Regular IBS/CBS**. Se o **Cód. Class. Trib. IBS/CBS** tiver **Requer Tributação Regular** marcado, você também pode informar no regular um código diferente do principal.
3. Volte ao documento e informe de novo o item ou o **Cód. Exceção Imposto** na linha.

> **Nota:** nenhum cadastro é corrigido automaticamente. Decidir qual código está certo cabe a quem configura os impostos. Se a Exceção Imposto e a Matriz Impostos tiverem o problema, o erro aponta um cadastro de cada vez: corrija o indicado e repita a operação. Quando o campo **Admin. Matriz de Impostos** da página **Configurações Fiscais** está preenchido, só o usuário informado nele pode editar a **Matriz Impostos**.

---

### 4.5 Vínculos de Classificação Tributária IBS/CBS (por NCM)

**Caminho no BC:** `Vínculos Class. Trib. IBS/CBS`

Esta tabela é um complemento da tabela de Classificações Tributárias (seção 4.2). Enquanto aquela define o que cada cClassTrib significa, esta define **a qual NCM ou Natureza de Operação cada cClassTrib se aplica**.

Na prática, ela responde à pergunta: "para o cClassTrib 200003 (Alíquota reduzida - Vendas de produtos destinados à alimentação humana relacionados no Anexo I...), quais NCMs estão abrangidos pela legislação?" A resposta está nos vínculos — e com a função de importação, esses vínculos são preenchidos automaticamente a partir da legislação vigente.

> **Dependência:** Esta tabela depende da tabela de Classificações Tributárias estar populada. Não é possível criar um vínculo para um cClassTrib que ainda não foi importado.

| Campo | O que representa |
|---|---|
| **Cód. Class. Trib. IBS/CBS** | O código cClassTrib ao qual este vínculo se refere |
| **Descrição** | Descrição do CST associado |
| **Tipo** | O tipo do vínculo: **Código NCM** (produto específico) ou **Natureza da Operação** |
| **Código** | O NCM ou código de Natureza de Operação vinculado |

#### Como popular esta tabela: Baixar Vínculos de NCM

Use a função **Baixar Vínculos de NCM...** no menu **Ações** para importar automaticamente todos os vínculos de NCM conforme a legislação. Após a importação, cada código de cClassTrib exibirá na coluna **Vínculos** (na tabela de Classificações) a quantidade de NCMs associados.

Esses vínculos são a base para que a Matriz Imposto consiga pré-preencher o cClassTrib correto em cada linha de documento com base no NCM do produto, sem intervenção manual do usuário.

---

### 4.6 O campo Cód. Class. Trib. IBS/CBS nos Documentos Fiscais

Toda a configuração feita até aqui converge para um único ponto no documento fiscal: o campo **Cód. Class. Trib. IBS/CBS** presente em cada linha do documento.

Quando este campo está preenchido em uma linha, o sistema executa automaticamente:

1. Busca o cClassTrib na tabela de Classificações para saber o tipo de tributação (integral, reduzida, diferida, isenta). Se o **Tipo de Alíquota** do código estiver em branco, a linha fica sem alíquota e sem valor de IBS e CBS (veja [Tipo de Alíquota](#tipo-de-alíquota))
2. Busca as alíquotas vigentes na tabela de Alíquotas IBS/CBS para CBS, IBS UF e IBS Municipal
3. Aplica os percentuais de redução ou diferimento conforme o cClassTrib
4. Calcula os valores de cada tributo e os registra nas linhas de imposto do documento
5. Ao gerar o XML da NF-e, inclui os grupos `<IBSCBS>` com todos os subgrupos e valores calculados

---

### 4.7 Tipos de Nota Class. Trib. IBS/CBS

**Caminho no BC:** `Tipos de Nota Class. Trib. IBS/CBS` (página `IBSCBSTaxClassNoteTypes`)

A NT 2025.002 (a partir da versão 1.40) exige que o cClassTrib de cada item seja compatível com a finalidade da NF-e e com o tipo da nota de débito ou de crédito. A exigência vale nos dois sentidos:

- **Classificações exclusivas:** alguns cClassTrib só podem ser usados num tipo de nota específico. Por exemplo, o `410030` só pode ser usado na Nota de Débito 07 – Perda em Estoque; numa venda normal, a SEFAZ rejeita a nota (rejeição 1202, regra UB14-60).
- **Tipos de nota que exigem uma classificação:** alguns tipos de nota exigem um cClassTrib específico em todos os itens. Por exemplo, a Nota de Débito 07 exige o `410030` (rejeição 1200, regra UB14-70); a Nota de Crédito 02 exige o `810001` (rejeição 1201, regra UB14-80).

Esta tabela guarda os pares de cClassTrib e tipo de nota definidos pela NT. Com ela preenchida, o sistema confere as notas antes do registro e impede que uma nota que a SEFAZ rejeitaria seja registrada.

| Campo | O que representa |
|---|---|
| **Cód. Class. Trib. IBS/CBS** | O cClassTrib do par |
| **Descrição Class. Trib. IBS/CBS** | A descrição do cClassTrib, lida da tabela de Classificações Tributárias (seção 4.2). Fica em branco se o código ainda não foi baixado na empresa |
| **Finalidade de Emissão** | **Nota de Débito** ou **Nota de Crédito** |
| **Tipo de Nota** | O código e a descrição do tipo de nota de débito (`tpNFDebito`) ou de crédito (`tpNFCredito`), por exemplo `07 – Perda em Estoque` |
| **Desativada** | Marcado quando a empresa desativou a linha (veja abaixo) |
| **Fora do Arquivo Atual** | Marcado quando o par não consta no último arquivo baixado. A linha aparece destacada em vermelho e deixa de ser conferida |

Os pares vêm somente do arquivo publicado pela Localização. A empresa não inclui, não altera e não exclui linhas: pode apenas desativar e reativar uma linha, no campo **Desativada**. Só os usuários com o conjunto de permissões **BR - Configurações Fiscais** desativam linhas e baixam a matriz; os demais usuários apenas consultam a tabela.

O arquivo publicado contém os oito pares da NT 2025.002 (versão 1.52, sem alteração desde a versão 1.40):

| cClassTrib | Finalidade de Emissão | Tipo de Nota |
|---|---|---|
| `410030` – Estorno de crédito por perda | Nota de Débito | 07 – Perda em Estoque |
| `800001` – Fusão, cisão ou incorporação | Nota de Débito | 05 – Transferência de crédito de sucessão |
| `800001` – Fusão, cisão ou incorporação | Nota de Crédito | 05 – Transferência de Crédito na Sucessão |
| `800002` – Transferência de crédito do associado | Nota de Débito | 01 – Transferência de créditos para Cooperativas |
| `810001` – Crédito presumido IBS ZFM | Nota de Crédito | 02 – Apropriação de crédito presumido de IBS sobre o saldo devedor na ZFM |
| `811001` – Anulação de crédito por saída imune/isenta | Nota de Débito | 02 – Anulação de Crédito por Saídas Imunes/Isentas |
| `811002` – Débito de NF não processada na apuração | Nota de Débito | 03 – Débitos de notas fiscais não processadas na apuração |
| `811003` – Desenquadramento do Simples Nacional | Nota de Débito | 08 – Desenquadramento do SN |

#### Como popular esta tabela: Baixar Matriz da Localização

> **Após atualizar a localização:** a tabela começa vazia, e a atualização ou instalação do app não a preenche. Enquanto ela estiver vazia, nenhuma nota é barrada por esta conferência. Para ativá-la, execute a função **Baixar Matriz da Localização...** depois da atualização.

Use a função **Baixar Matriz da Localização...**, no menu **Processo** da página. O sistema pede confirmação, baixa o arquivo publicado pela Localização e, ao final, informa quantas linhas foram incluídas, quantas foram mantidas (e quantas delas estão desativadas) e quantas estão fora do arquivo atual.

O arquivo é publicado num endereço fixo: quando a NT alterar a tabela de pares, basta executar a função de novo para receber a atualização, sem esperar uma nova versão do app. A mesma função serve para a primeira carga e para as atualizações:

| Situação da linha | O que a importação faz |
|---|---|
| Par do arquivo que ainda não existe no BC | É incluído, ativo |
| Par do arquivo que já existe no BC | É mantido como está. Uma linha desativada pela empresa **continua desativada** — a importação nunca a reativa |
| Par que existe no BC, mas não consta no arquivo | Não é apagado: recebe a marca **Fora do Arquivo Atual** e deixa de ser conferido |
| Par marcado que volta a constar no arquivo | A marca **Fora do Arquivo Atual** é retirada |

Repetir a importação com o mesmo arquivo não altera nada. Se o arquivo estiver indisponível, vazio ou inválido, a importação é interrompida com uma mensagem que informa o motivo, e a tabela não é alterada.

#### Como a conferência funciona

A conferência roda nas notas de emissão própria, modelo 55 (NF-e):

- no registro de documentos de venda e de compra que geram NF-e;
- na liberação (ação **Liberar**) do **Documento Fiscal de Venda** e do **Documento Fiscal de Compra** lançados manualmente;
- na ação **Validar NF-e** dos documentos de venda e de compra, que passa pela mesma validação do registro.

São conferidas as linhas que geram item na NF-e e que têm o **Cód. Class. Trib. IBS/CBS** preenchido. Linhas sem cClassTrib não são conferidas; por isso, empresas que ainda não emitem com IBS/CBS não são afetadas. Valem apenas as linhas da tabela ativas e que não estão fora do arquivo atual. O sistema impede o registro ou a liberação quando:

| Regra da NT | Situação barrada | Rejeição evitada |
|---|---|---|
| **UB14-60** | Um item usa um cClassTrib da tabela, e a nota não é de débito ou de crédito de um tipo pareado com ele | 1202 |
| **UB14-70** | A nota é de débito de um tipo da tabela, e algum item usa um cClassTrib que não está pareado com esse tipo | 1200 |
| **UB14-80** | A nota é de crédito de um tipo da tabela, e algum item usa um cClassTrib que não está pareado com esse tipo | 1201 |

Tipos de nota e cClassTrib que não estão na tabela não sofrem restrição. Com o arquivo atual, isso vale para as Notas de Débito 04 e 06 e para as Notas de Crédito 01, 03, 04 e 06: nelas o cClassTrib é escolhido livremente, desde que não seja um dos exclusivos de outro tipo.

A conferência sempre bloqueia — não existe opção de apenas avisar. O erro aparece na lista **Mensagens de Erro**, junto com as demais pendências do documento, e informa a linha, o item, o cClassTrib usado, a finalidade e o tipo da nota, o cClassTrib ou os tipos de nota aceitos e a regra da NT. Exemplo:

> Linha 10000, item 1000: a Nota de Débito 07 – Perda em Estoque só aceita a classificação tributária IBS/CBS 410030, e a linha usa 000001. A SEFAZ rejeita a NF-e (NT 2025.002, regra UB14-70, rejeição 1200).

#### Desativar uma linha

Se uma linha estiver barrando notas indevidamente, o responsável fiscal pode marcar **Desativada** nela. Uma linha desativada deixa de impor restrição: o tipo de nota deixa de exigir aquele cClassTrib, e o cClassTrib deixa de ficar restrito por causa daquela linha. A linha continua, porém, aceitando o próprio par. Por isso, desativar uma linha nunca faz uma nota que antes era aceita passar a ser barrada.

Exemplo: com a linha `410030` × Nota de Débito 07 desativada, a Nota de Débito 07 passa a aceitar outros cClassTrib, e o `410030` pode ser usado numa venda normal. A SEFAZ continua aplicando a regra; desative uma linha apenas quando tiver certeza de que ela não se aplica à operação.

Uma linha desativada continua desativada nas importações seguintes. Para voltar a conferi-la, desmarque **Desativada**.

---

## 5. Impacto na Operação e Pontos de Atenção

### 5.1 O que muda no dia a dia de quem configura

A Reforma Tributária introduz responsabilidades novas na gestão da configuração fiscal do sistema. As principais são:

- **Manter a tabela de Alíquotas IBS/CBS atualizada:** as alíquotas mudam a cada ano da transição. A cada atualização legislativa, um novo registro precisa ser inserido com a data de início de vigência correta.
- **Manter a tabela de Classificações Tributárias atualizada:** o governo está publicando e revisando os cClassTrib continuamente. Novos códigos, alterações de percentuais de redução e novas situações tributárias precisarão ser incorporados à tabela.
- **Revisar as Áreas de Impostos:** as áreas existentes precisam incluir as novas jurisdições (IBS UF, IBS Municipal, CBS). A função de importação cuida disso para as áreas já configuradas, mas novas áreas criadas futuramente precisarão incluir as novas jurisdições manualmente.
- **Configurar Matriz de Imposto e Exceção Imposto com os cClassTrib:** opcional. Usar a matriz de imposto é um facilitador, pois realiza o preenchimento automático nas linhas. Caso a empresa não use Matriz de Imposto, o preenchimento do **cClassTrib** precisa ser feito manualmente.

### 5.2 O que muda nos documentos fiscais

- O XML da NF-e e da NFS-e passa a incluir os grupos `<IBSCBS>` por item, com CST, cClassTrib, alíquotas de IBS UF, IBS Municipal e CBS, e eventuais reduções ou diferimentos.
- Os documentos fiscais exibirão campos de IBS/CBS nas linhas de imposto, juntamente com os campos já existentes de ICMS, PIS e COFINS.
- O campo **Cód. Class. Trib. IBS/CBS** aparecerá em cada linha do documento — e será obrigatório a partir de 2026.

### 5.3 O que muda na escrituração fiscal

- O **EFD-ICMS/IPI** e o **EFD-Contribuições** continuam sendo obrigações durante o período de transição — a extinção do ICMS, PIS e COFINS é gradual.
- O governo está definindo as novas obrigações acessórias para IBS e CBS. Quando publicadas, serão incorporadas à localização.
- Está previsto um modelo de **apuração assistida**, no qual o próprio governo calcula os valores de IBS/CBS a serem recolhidos com base nos XMLs transmitidos, cabendo à empresa apenas a conferência. Este modelo ainda não está regulamentado.

### 5.4 Alertas importantes

> **Dois sistemas ao mesmo tempo (2026–2032):** durante todo o período de transição, a empresa calcula ICMS, PIS e COFINS normalmente, e também calcula IBS e CBS. As configurações de Áreas de Impostos e Matriz Imposto precisam contemplar todos os tributos sem conflito.

> **Legislação em construção:** a Reforma Tributária ainda está sendo regulamentada em detalhes. Novas Notas Técnicas (NTs) da SEFAZ e novos regulamentos serão publicados ao longo dos anos. A localização é atualizada a cada nova publicação relevante.

> **Tabela de cClassTrib extensa e dinâmica:** o catálogo oficial de cClassTrib tem centenas de códigos e está sujeito a revisões. Use com periodicidade a função de importação para garantir que a tabela esteja sincronizada com a versão mais recente publicada pelo governo. A reimportação atualiza a tabela sem apagar códigos nem vínculos de NCM; os códigos que saírem da tabela oficial ficam marcados como **Fora da Tabela Vigente**.

> **Matriz de tipos de nota começa vazia:** a conferência do cClassTrib com o tipo da nota de débito ou de crédito (seção 4.7) só passa a funcionar depois que a função **Baixar Matriz da Localização...** é executada. Até lá, a SEFAZ continua sendo a única barreira, e uma nota incompatível só é rejeitada depois de registrada.

> **Alíquotas de transição:** as alíquotas de 2026 (CBS 0,90%, IBS UF 0,10%) são provisórias e deliberadamente baixas — fazem parte da fase de teste do novo sistema. Elas aumentarão progressivamente até 2033.

> **DANFE sem reflexo da reforma:** o Governo ainda não definiu como IBS e CBS devem aparecer no documento impresso (DANFE). Portanto, a DANFE atual não exibe informações de IBS/CBS — apenas o XML é afetado.

---

## 6. Checklist de Configuração

Execute os passos abaixo na ordem indicada. Cada etapa é pré-requisito da seguinte.

| # | O que fazer | Onde no BC | Função que ajuda |
|---|---|---|---|
| 1 | Definir o Status Reforma Tributária (Habilitada ou Agendada) | Configurações Fiscais | — |
| 2 | Inserir as alíquotas de referência vigentes | Alíquotas IBS/CBS | **Inserir Alíquotas de Referência** |
| 3 | Importar todos os códigos cClassTrib do catálogo oficial | Classificações Tributárias IBS/CBS | **Baixar Classificações Tributárias...** |
| 4 | Criar as jurisdições CBS, IBS UF e IBS Mun e incluí-las nas áreas de impostos existentes | Jurisdições de Impostos (Brasil) | **Inserir Dados da Reforma Tributária...** |
| 5 | Importar os vínculos entre NCMs e seus respectivos cClassTrib | Vínculos Class. Trib. IBS/CBS | **Baixar Vínculos de NCM...** |
| 6 | Preencher os campos cClassTrib nas matrizes de impostos (se utilizada) | Matriz Imposto | — |
| 7 | Preencher cClassTrib para casos com tratamento diferenciado | Exceção Imposto | — |
| 8 | Baixar a matriz de cClassTrib por tipo de nota de débito/crédito (ela começa vazia, mesmo depois da atualização da localização) | Tipos de Nota Class. Trib. IBS/CBS | **Baixar Matriz da Localização...** |
