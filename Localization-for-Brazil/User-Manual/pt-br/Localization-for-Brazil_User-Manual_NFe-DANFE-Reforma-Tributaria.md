<!-- PR: #297 -->
# NF-e: DANFE no Leiaute da Reforma Tributária (NT 2026.010)

**Data**: Outubro/2026

---

## Sumário

A Nota Técnica 2026.010 definiu um novo leiaute para o DANFE da NF-e (modelo 55), que passa a mostrar os tributos da Reforma Tributária: CBS, IBS (UF e Município) e Imposto Seletivo (IS). A localização gera o DANFE nesse leiaute para as notas de produção emitidas a partir de **01/12/2026** e para todas as notas de homologação. O novo DANFE traz os tributos de cada item identificados pelo nome, o bloco **Total do IBS/CBS/IS**, o regime tributário do emitente e os totais de FCP e DIFAL. Quadros e linhas sem conteúdo não ocupam espaço. As folhas seguintes à primeira trazem só a identificação da nota e a continuação dos itens. O Setup NF-e ganha a opção **Não imprimir canhoto**.

---

## 1. Contexto

### Qual DANFE vale para cada nota

A escolha do leiaute é automática, pela data de emissão e pelo ambiente da NF-e. Não há nada a configurar.

| Ambiente da nota | Data de emissão | DANFE gerado |
|---|---|---|
| Produção | até 30/11/2026 | Leiaute atual, sem nenhuma alteração |
| Produção | a partir de 01/12/2026 | Novo leiaute da Reforma Tributária |
| Homologação ou Desenvolvimento | qualquer data | Novo leiaute da Reforma Tributária |

O ambiente é o do campo **Ambiente** da **Config. NF-e** da filial.

A regra vale em todos os momentos em que o DANFE é gerado:

- na autorização da NF-e;
- na pré-visualização, pela ação **Visualizar DANFe (Teste)** da **Pré-Visualização de Documento Fiscal**. A pré-visualização mostra o leiaute que vai valer para aquela nota, mesmo sendo uma nota de produção;
- na reimpressão, pela ação **Criar DANFe** da NF-e;
- no cancelamento, com a marca d'água "CANCELADO" em todas as folhas;
- no anexo do e-mail enviado ao cliente.

Assim, uma nota de produção emitida até 30/11/2026 continua saindo no leiaute atual também quando é reimpressa, cancelada ou reenviada por e-mail.

> **Testes antes de 01/12/2026**: notas emitidas em homologação já saem no novo leiaute, com a tarja "Sem Valor Fiscal". Use esse ambiente para conferir o DANFE antes da entrada em produção.

### O que muda em relação ao DANFE atual

| DANFE atual | Novo DANFE |
|---|---|
| Não mostra IBS, CBS nem IS | Mostra base, alíquota e valor de IBS UF, IBS Mun, CBS e IS de cada item, e os totais da nota |
| Não mostra o regime tributário do emitente | Mostra o **Código do Regime Tributário** do emitente |
| Não mostra FCP e DIFAL nos totais | Mostra FCP, FCP retido por ST, DIFAL e FCP na UF de destino, quando a nota os tiver |
| Repete todos os quadros em todas as folhas | Da segunda folha em diante, só a identificação da nota e os itens |
| Canhoto sempre impresso | Canhoto impresso só na primeira folha, e pode ser desligado no Setup NF-e |
| Até 10 duplicatas | Todas as duplicatas da nota |
| Informações complementares longas podem sair cortadas | O texto que não cabe continua nas folhas seguintes |

O DANFE continua sempre em retrato, em folha A4. O leiaute paisagem da NT não é usado.

---

## 2. Configuração

Nenhuma configuração é obrigatória: o novo DANFE é usado automaticamente conforme a tabela acima. As opções de DANFE ficam na **Config. NF-e** da filial.

1. Navegue até **Config. NF-e** (página `NFeSetupFNX`), pela pesquisa do Business Central ou pela ação **Config. NF-e** no cartão da **Informação Filial**.
2. No grupo **DANFe**, ajuste as opções:

| Campo | Efeito no novo DANFE |
|---|---|
| **Não imprimir canhoto** (`DANFeHideReceiptStub`) | Ligado, o DANFE sai sem o canhoto, e o espaço vai para os demais quadros. Desligado (padrão), o canhoto sai na primeira folha. Vale só para o novo leiaute: o DANFE atual sempre imprime o canhoto. |
| **Dados Adicionais Após Itens** (`DANFeAdditDataAfterItems`) | O texto de dados adicionais sai logo depois do último item, na largura total do quadro de produtos. Ver [Dados Adicionais](#7-dados-adicionais). |
| **Observações Sobre Colunas** (`DANFeRemarksOverColumns`) | As informações adicionais de cada item saem numa faixa na largura total, abaixo das colunas do item. Ver [Observações Sobre Colunas](#observações-sobre-colunas). |
| **Mostrar Nº Série**, **Mostrar Nº Lote**, **Mostrar Data Fabricação**, **Mostrar Data Validade**, **Mostrar Desconto do Item**, **Mostrar EAN** | Funcionam como no DANFE atual. |

O campo **Não imprimir canhoto** vem desligado, inclusive nas filiais que já usavam a localização: o novo DANFE sai com canhoto até que a opção seja ligada.

---

## 3. Primeira Folha

A primeira folha traz, de cima para baixo, os quadros que se aplicam à nota, na ordem do modelo retrato da NT. Os quadros marcados como condicionais só aparecem quando há conteúdo, e o espaço que eles ocupariam fica para os itens.

| Quadro | Quando aparece |
|---|---|
| Canhoto | Quando **Não imprimir canhoto** está desligado |
| Identificação do emitente e da NF-e | Sempre |
| Destinatário/Remetente | Sempre |
| Informações do local de retirada | Só quando a nota tem local de retirada |
| Informações do local de entrega | Só quando a nota tem local de entrega |
| Fatura/Duplicata | Só quando a nota tem duplicatas |
| Total dos produtos e total da nota | Sempre |
| Total do ICMS/IPI | Sempre |
| Total do IBS/CBS/IS | Sempre |
| Transportador/Volumes transportados | Omitido quando o frete é "Sem frete" e a nota não tem transportadora, veículo nem volumes |
| Dados do produto/serviço | Sempre, no espaço que sobra |
| Cálculo do ISSQN | Só quando a nota tem serviços, sempre na primeira folha |
| Dados adicionais | Sempre na primeira folha |

### Identificação do emitente

Além dos dados do DANFE atual (emitente e logotipo, número, série, folha, chave de acesso, natureza da operação, protocolo, inscrições estaduais e CNPJ), a identificação traz dois campos novos:

- **Código do Regime Tributário**: o CRT do XML, com código e descrição. Por exemplo, "3 – Regime Normal".
- **Tipo de Regime de Apuração do IBS e da CBS**: o espaço é reservado pela NT, mas sai vazio, porque a informação ainda não foi definida pela Receita.

### Fatura/Duplicata

Todas as duplicatas da nota são impressas, em ordem, 5 por linha (número, valor e vencimento). O quadro cresce uma linha a cada 5 duplicatas e fica inteiro na primeira folha. Com muitas duplicatas, os itens podem começar só na segunda folha.

O quadro é omitido quando a nota não tem duplicatas, como numa venda à vista. Uma customização também pode omiti-lo mesmo quando a nota tem duplicatas. A localização não oferece essa opção na tela; nesse caso, as duplicatas continuam no XML da NF-e e só deixam de ser impressas no DANFE.

---

## 4. Totais

O antigo quadro "Cálculo do imposto" foi dividido em três quadros. Todos os valores são os mesmos do XML autorizado.

### Total dos produtos e total da nota

Valor total dos produtos, valor do frete, valor do seguro, desconto, outras despesas acessórias e valor total da nota.

### Total do ICMS/IPI

| Linha | Campos | Quando aparece |
|---|---|---|
| Principal | Base de cálculo do ICMS, valor do ICMS, base de cálculo do ICMS ST, valor do ICMS ST, valor do IPI | Sempre |
| FCP e DIFAL | Valor do FCP, valor do FCP retido por ST, valor do DIFAL na UF de destino, valor do FCP na UF de destino | Só quando algum deles tem valor |

Numa venda interestadual a consumidor final não contribuinte, por exemplo, a segunda linha mostra o DIFAL e o FCP devidos à UF de destino.

### Total do IBS/CBS/IS

Valor da CBS, valor do IBS UF, valor do IBS Município e valor do Imposto Seletivo. A linha sai sempre, porque a NT não a classifica como facultativa. Quando a nota não tem IBS/CBS, os campos ficam vazios.

---

## 5. Dados do Produto/Serviço

### Colunas

| Coluna | Conteúdo |
|---|---|
| Descrição do produto/serviço | Código e descrição do item ("PN010203 - Descrição do item"), informações adicionais do item e, abaixo, o NCM e a classificação tributária do IBS/CBS (cClassTrib) |
| CST/CFOP | CST e CFOP do item |
| QTD/UN | Quantidade e unidade comercial |
| VLR UNIT | Valor unitário comercial |
| VLR TOTAL | Valor total do item |
| Bases de cálculo | Uma linha por base: ICMS, IPI, IBS/CBS e IS |
| Alíquotas | Uma linha por tributo: ICMS, IPI, IBS UF, IBS Mun, CBS e IS |
| VLR dos tributos | O valor de cada tributo, na mesma linha da alíquota |

Cada base, alíquota e valor sai com o nome do tributo à frente, em letra menor. Assim não é preciso decorar a posição de cada tributo.

### Linhas de tributos

A linha de um tributo só aparece quando o item tem esse tributo (base, alíquota ou valor). Um item sem IPI, por exemplo, não imprime as linhas de IPI, e as linhas seguintes sobem. IBS UF, IBS Mun e CBS formam um conjunto, com uma única base IBS/CBS.

### Alíquotas

| Tributo | Alíquota impressa |
|---|---|
| ICMS e IPI | A alíquota do XML, com 2 casas decimais |
| IBS UF, IBS Mun e CBS | Com redução de alíquota ou em compra governamental, a **alíquota efetiva** aplicada; nos demais casos, a alíquota vigente |
| IS | A alíquota do IS |

As alíquotas de IBS UF, IBS Mun, CBS e IS saem com as casas decimais do XML, entre 2 e 4, sem zeros à direita: 0,10, 0,095, 0,0125.

### Altura do item

Cada item ocupa no mínimo 3 linhas: a descrição, as informações adicionais (ou uma linha em branco) e a linha de NCM e cClassTrib. O item não tem altura máxima:

- a descrição quebra em novas linhas quando passa da largura da coluna;
- as informações adicionais acrescentam linhas;
- o item fica com a altura do maior entre o texto da descrição e as linhas de tributos.

Quando a unidade tributável é diferente da comercial, a quantidade, a unidade e o valor unitário tributáveis saem na segunda linha das colunas QTD/UN e VLR UNIT.

As informações adicionais do item continuam as mesmas do DANFE atual: informação adicional do produto, número da FCI e, conforme o grupo **DANFe** da **Config. NF-e**, lote, série, validade, fabricação, EAN e desconto.

Valores muito altos, que não cabem na largura da coluna, saem em letra menor dentro da própria coluna.

### Observações Sobre Colunas

Com **Observações Sobre Colunas** ligado, o item é impresso em duas partes:

1. as colunas do item, com o código e a descrição, a linha de NCM e cClassTrib e as linhas de tributos;
2. logo abaixo da coluna mais alta, uma faixa na largura total do quadro, com as informações adicionais do item.

Como as colunas de tributos podem ter várias linhas, o item tende a ficar mais alto que no DANFE atual, e cabem menos itens por folha. A linha de NCM e cClassTrib vem antes das informações adicionais. Um item sem informações adicionais não tem faixa.

---

## 6. Folhas Seguintes

Da segunda folha em diante, o DANFE traz só:

- a identificação do emitente e da NF-e, com "Folha x/y";
- a continuação dos itens;
- o quadro de dados adicionais, enquanto houver texto a imprimir.

Canhoto, destinatário, duplicatas, totais e transporte não se repetem. Um item nunca é dividido entre duas folhas, salvo quando ele sozinho é mais alto que uma folha inteira. A numeração "Folha x/y" sai correta em todas as folhas.

---

## 7. Dados Adicionais

O quadro **Dados adicionais** traz as informações complementares da nota e, em seguida, as informações adicionais de interesse do Fisco, como no DANFE atual.

Cada folha comporta 6 linhas de texto no quadro. O texto que não cabe continua no quadro da folha seguinte. Quando os itens já acabaram, o DANFE cria folhas novas só para o restante do texto. A partir da folha em que não sobra texto, o quadro deixa de aparecer. Nenhum trecho é cortado: uma linha longa, que ocupa mais de uma linha impressa, é contada pelas linhas que ocupa.

Com **Dados Adicionais Após Itens** ligado, o texto sai logo depois do último item, na largura total do quadro de produtos, e continua nas folhas seguintes quando necessário.

---

## 8. Campos Reservados

Alguns campos previstos na NT ainda não têm informação no XML gerado pela localização. Eles ficam vazios ou não aparecem até que a tributação correspondente seja emitida:

| Campo | No DANFE |
|---|---|
| Imposto Seletivo (base, alíquota e valor do item; valor total) | As linhas do item não aparecem; o total fica vazio |
| IBS/CBS monofásicos e ICMS monofásico | As linhas não aparecem |
| Tipo de Regime de Apuração do IBS e da CBS | Vazio |
| QR Code da NF-e | Não é impresso, porque depende de NT futura |

---

## 9. Resumo dos Campos Novos

### Tabela Config. NF-e (`NFeSetupFNX`)

| Campo | Tipo | Descrição |
|---|---|---|
| **Não imprimir canhoto** (`DANFeHideReceiptStub`) | Booleano | Imprime o novo DANFE sem o canhoto. Desligado por padrão. Não afeta o DANFE atual. |
