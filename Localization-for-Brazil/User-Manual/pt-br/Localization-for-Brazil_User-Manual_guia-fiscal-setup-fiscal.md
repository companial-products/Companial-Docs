<!-- PR: #343 -->
# Configuração Fiscal — CFOP, NCM, CST, Matrizes

Este capítulo cobre as **tabelas de referência fiscal** que alimentam o cálculo de impostos. São os códigos e as matrizes que determinam, para cada operação, quais alíquotas e regras se aplicam.

Estas tabelas dependem dos conceitos de [Jurisdições, Áreas de Impostos e Contabilizações](Localization-for-Brazil_User-Manual_guia-fiscal-jurisdicoes-areas-impostos.md) — se ainda não leu aquele capítulo, leia antes de continuar.

---

## Configurações Fiscais (Fiscal Setup)

A página **Configurações Fiscais** (Fiscal Setup) é a página central de configuração da localização. Ela contém parâmetros gerais que controlam o comportamento do cálculo de impostos, e serve como ponto de acesso para as tabelas de referência.

**Caminho no BC**: Diga-me > "Configurações Fiscais"

---

## CFOP — Código Fiscal de Operações e Prestações

### O que é

O CFOP é um código no formato **x.xxx** que identifica a **natureza da operação** fiscal. Ele indica a natureza da operação (venda, compra, transferência, remessa, etc.) e como ela será escriturada nos livros fiscais.

### Estrutura

O primeiro dígito indica o grupo:

| Entradas | Saídas | Natureza |
|----------|--------|----------|
| **1**.xxx | **5**.xxx | Operações dentro do estado |
| **2**.xxx | **6**.xxx | Operações interestaduais |
| **3**.xxx | **7**.xxx | Importação / Exportação |

### Exemplos comuns

| CFOP | Operação |
|------|----------|
| 5.101 | Venda de produção — dentro do estado |
| 6.102 | Venda de mercadoria adquirida — interestadual |
| 5.405 | Venda de mercadoria com ST já recolhida |
| 1.102 | Compra para comercialização — dentro do estado |
| 5.901 | Remessa para industrialização |
| 5.949 | Outra saída não especificada |

### No sistema

A localização disponibiliza uma tabela de CFOPs que a empresa deve manter atualizada. O CFOP é:
- **Obrigatório** em cada linha do documento fiscal
- Usado na escrituração fiscal (satisfaz requisitos legais, mas não determina impostos na localização)
- Escriturado nos registros do EFD (campo de todo registro de documento)

**Caminho no BC**: Diga-me > "CFOP"

---

## NCM — Nomenclatura Comum do Mercosul

### O que é

O NCM é um código de **8 dígitos** que classifica a mercadoria conforme padrão internacional. Ele determina alíquotas de IPI, PIS/COFINS (em alguns casos), Imposto de Importação, e é usado para controle estatístico.

### Estrutura

```
XXXX.XX.XX
│    │  │└── Subitem (8º dígito, definido pelo Mercosul)
│    │  └── Item (7º dígito, definido pelo Mercosul)
│    └── Subposição (5º e 6º dígitos, Sistema Harmonizado)
└── Capítulo e posição (4 primeiros dígitos, Sistema Harmonizado)
```

### No sistema

A localização disponibiliza uma tabela de NCMs, incluindo descrição e alíquota IBPT (carga tributária estimada), que a empresa deve manter atualizada. O NCM é:
- Cadastrado no **Item** (produto)
- Copiado para a linha do documento fiscal
- Possui uma tabela de **alíquotas por NCM** que pode alterar impostos (usada mais comumente com IPI e II)
- Usado no cálculo do texto legal de impostos estimados (Lei da Transparência / IBPT)
- Escriturado no registro 0200 do EFD-ICMS/IPI

**Caminho no BC**: Diga-me > "NCM"

---

## CST — Código de Situação Tributária

### O que é

O CST indica o **tratamento tributário** de cada item em uma operação. Cada imposto tem sua própria tabela de CST.

### CST ICMS

Composto por três dígitos: **origem** da mercadoria (1 dígito) + **tributação** (2 dígitos):

| Origem (1o dígito) | Tributação (2 dígitos) |
|--------------------|----------------------|
| 0 = Nacional | 00 = Tributada integralmente |
| 1 = Estrangeira (importação direta) | 10 = Tributada + ICMS-ST |
| 2 = Estrangeira (mercado interno) | 20 = Com redução de base |
| 3-8 = Demais origens (conteúdo de importação, PPB, estrangeira sem similar nacional) | 40 = Isenta |
| | 41 = Não tributada |
| | 51 = Diferimento |
| | 60 = ICMS cobrado por ST anterior |
| | 70 = Redução de base + ST |
| | 90 = Outros |

**Exemplo**: CST 010 = Nacional (0) + Tributada com ST (10)

### CSOSN (Simples Nacional)

Para empresas do Simples Nacional, usa-se o **CSOSN** em vez do CST ICMS:
- 101 = Tributada com permissão de crédito
- 102 = Tributada sem permissão de crédito
- 500 = ICMS cobrado anteriormente por ST
- 900 = Outros

### CST PIS/COFINS

Código de 2 dígitos: 01-05 (saídas tributáveis), 06 (alíquota zero), 07 (isenta), 08 (sem incidência), 09 (suspensão), 49 (outras saídas), 50-56 (entradas com direito a crédito), 60-67 (crédito presumido), 70-75 (entradas sem direito a crédito), 98-99 (outras operações).

### CST IPI

Código de 2 dígitos: 00-05 e 49 (entradas), 50-55 e 99 (saídas).

### No sistema

A localização disponibiliza uma tabela de CSTs por imposto que a empresa deve manter atualizada. O CST é:
- Sugerido pela jurisdição/área de impostos, pelas **Alíquotas NCM**, pela exceção e pela **matriz de impostos**, nessa ordem; cada um substitui o anterior, salvo quando **Prioridades CST** dá prioridade ao já aplicado. Um CST preenchido na matriz prevalece sobre o da exceção
- Obrigatório na linha do documento fiscal
- Escriturado nos registros do EFD

**Caminho no BC**: Diga-me > "CST"

---

## Matriz de Impostos (Taxes Matrix)

### O que é

A **Matriz Impostos** (Taxes Matrix) é a tabela central de configuração fiscal. Cada registro da matriz, identificado por um **Código**, reúne toda a parametrização fiscal de uma operação — quais impostos se aplicam, quais CFOPs usar, quais CSTs atribuir.

### Como funciona

Cada registro da matriz define:

| Campo | O que configura |
|-------|----------------|
| **Código** | Identificador da configuração (ex: VENDA-MERC, COMPRA-SERV) |
| **Tipo** | Compra ou Venda |
| **Área Impostos** | Quais impostos serão calculados (referência ao [capítulo anterior](Localization-for-Brazil_User-Manual_guia-fiscal-jurisdicoes-areas-impostos.md)) |
| **CFOPs** | Um CFOP para cada situação: mesmo estado, interestadual, exterior, consumidor final |
| **CSTs** | Códigos de situação tributária por imposto (ICMS, IPI, PIS, COFINS) |
| **Exceção Imposto** | Exceção fiscal aplicável, se houver |
| **Texto Legal** | Texto legal padrão para a NF-e |

Quando o usuário seleciona uma Matriz Impostos na linha de compra ou venda, o sistema aplica automaticamente todos esses campos na linha — incluindo o CFOP correto conforme a localização do parceiro (mesmo estado, interestadual ou exterior).

### Exemplo simplificado

| Código | Tipo | Área Impostos | CFOP Mesmo Estado | CFOP Interestadual | CST ICMS |
|--------|------|---------------|-------------------|-------------------|----------|
| VENDA-MERC | Venda | VEND | 5.102 | 6.102 | 00 |
| VENDA-ST | Venda | VEND-ST | 5.401 | 6.401 | 10 |

Na matriz, o CST ICMS tem os dois dígitos da tributação; a origem é informada no campo **Código Origem ICMS** da matriz.

A partir da **Área Impostos** definida na matriz, o sistema sabe quais impostos calcular. As alíquotas são determinadas pelas jurisdições dentro da Área (Alíquotas ICMS por UF, exceções, etc.) — a matriz em si não contém alíquotas.

**Caminho no BC**: Diga-me > "Matriz Impostos"

---

## Alíquotas ICMS por UF (ICMS Percent by Territory)

### O que é

Tabela de **alíquotas de ICMS** por UF. Define a alíquota de ICMS, DIFAL e ST para cada combinação de UF origem e UF destino.

### Como funciona

| UF Origem | UF Destino | Alíquota ICMS |
|-----------|-----------|---------------|
| SP | SP | 18% |
| SP | MG | 12% |
| SP | BA | 7% |
| MG | SP | 12% |

As alíquotas interestaduais são fixadas por Resolução do Senado (4% para importados, 7% ou 12% conforme origem/destino), enquanto as internas variam por estado.

Existe também a tabela **Territory FCP Percentage** para o Fundo de Combate à Pobreza (adicional estadual).

**Caminho no BC**: Diga-me > "Alíquotas ICMS por UF"

---

## Exceções de Imposto (Tax Exception)

### O que é

A **Exceção Imposto** (Tax Exception) permite definir regras fiscais específicas — como isenções, alíquotas diferenciadas ou diferimentos — que se aplicam em determinadas condições. A própria matriz de impostos pode referenciar um código de exceção, de modo que a exceção faz parte da configuração, não a "sobrescreve".

Para redução de base de cálculo por NCM, existe a tela de **Alíquotas NCM** (mencionada na seção de NCM acima), que funciona de forma semelhante à exceção. A diferença é que a exceção pode ser configurada independentemente da NCM.

### Quando usar

- Isenção para determinada operação
- Alíquota diferenciada por estado
- Diferimento parcial
- Benefícios fiscais estaduais (PRODEPE, PROEDI, etc.)

Há um caso específico em que a exceção pode sobrescrever um campo da matriz: os campos **Classificação Tributária IBS/CBS** (IBS/CBS Tax Classification) e **Classificação Tributária Regular IBS/CBS** (IBS/CBS Regular Tax Classification), que existem em ambas as tabelas. Se a exceção tiver esses campos preenchidos, eles prevalecem sobre os da matriz.

As exceções permitem modelar a complexidade das legislações estaduais de forma organizada.

**Caminho no BC**: Diga-me > "Exceções Imposto"

### Alíquotas da exceção

Os tratamentos de cada exceção ficam nas linhas de **Alíquotas Exceção Imposto** (página `TaxExceptionPercentagesFNX`), abertas pela ação **Alíquotas** na página **Exceções Imposto** (`TaxExceptionsFNX`). Cada linha vale para:

- um imposto (**Classificação Imposto**);
- um estado (**Cód. Território**), ou para qualquer estado, com o campo em branco;
- um período (**Data Início** e **Data Fim**).

A linha traz o tratamento daquele imposto: alíquota, **Fator de Redução de Base**, diferimento, CST, **Cód. Enquadramento Legal** e **Cód. Texto Legal**.

Quando o usuário informa o item ou o **Cód. Exceção Imposto** na linha do documento, o CST da linha de Alíquotas escolhida é sugerido no documento, mas um CST preenchido na matriz de impostos prevalece sobre ele, salvo quando **Prioridades CST** dá prioridade ao da exceção. Nas saídas (vendas e devoluções de compra) o sistema usa o **CST (Saída)**, e nas entradas (compras e devoluções de venda) usa o **CST (Entrada)**.

### Linhas só para operações internas (Mesmo Estado)

Alguns benefícios estaduais valem só "nas operações internas", como uma redução de base de ICMS nas saídas internas de um produto. Para esses casos, marque **Mesmo Estado** (`SameState`) na linha de Alíquotas. A linha marcada só é considerada quando a **Localização Cliente/Fornecedor** do documento é **Mesmo estado**. É o mesmo critério que o sistema usa para escolher o CFOP de operação interna. Todos os efeitos da linha seguem essa regra: alíquota, redução de base, base isenta/outras, diferimento, desoneração, MVA, CST, enquadramento legal e texto legal.

| Operação | Linha marcada **Mesmo Estado** |
|----------|--------------------------------|
| Cliente ou fornecedor no mesmo estado da filial | Aplicada |
| Venda presencial a consumidor de outro estado, ou venda a consumidor não identificado (**Localização Cliente/Fornecedor** = **Mesmo estado**) | Aplicada |
| Cliente ou fornecedor de outro estado (**Outro estado**) | Desconsiderada |
| Importação e exportação (**Fora do Brasil**) | Desconsiderada |
| Transferência entre filiais do mesmo estado | Aplicada |
| Transferência entre filiais de estados diferentes | Desconsiderada |

Alguns pontos sobre a marcação:

- **Vale por linha.** As demais linhas da mesma exceção não mudam. Uma exceção pode ter a linha de ICMS marcada e as linhas de PIS/COFINS sem marcação, que continuam valendo em qualquer operação.
- **Estado da linha marcada.** O **Cód. Território** de uma linha marcada é o estado da filial. Uma linha marcada com o **Cód. Território** em branco vale para as operações internas de qualquer filial, seja qual for o estado.
- **Linha marcada desconsiderada.** O sistema procura outra linha do mesmo imposto na exceção que sirva para a operação: uma linha sem a marcação, do estado da operação ou sem estado. Se nenhuma servir, o imposto segue a tributação normal da área de imposto e da matriz.
- **Exceções sem linha marcada** funcionam como antes.

Quando mais de uma linha serve para a operação, a escolha segue esta ordem:

1. Linha marcada do estado da filial (só em operação interna).
2. Linha sem a marcação do estado da operação: nas vendas, o estado do cliente; nas compras, o da filial.
3. Linha marcada sem estado (só em operação interna).
4. Linha sem a marcação e sem estado.

Ou seja, a linha de um estado específico prevalece sobre a linha sem estado. Entre linhas do mesmo estado, numa operação interna, a linha marcada prevalece sobre a não marcada.

A mesma exceção não aceita duas linhas com a mesma **Classificação Imposto**, **Cód. Classificação Adicional**, **Cód. Território** e **Data Início**, estejam marcadas ou não. Para dar a um mesmo estado um tratamento interno e outro interestadual, há duas formas:

- cadastrar o tratamento interestadual numa linha sem estado; ou
- cadastrar as duas linhas do estado com **Data Início** diferentes, ambas vigentes.

**Exemplo** — exceção de um produto com redução de base de ICMS nas saídas internas de SP:

| Classificação Imposto | Cód. Território | Mesmo Estado | Tratamento |
|-----------------------|-----------------|--------------|------------|
| ICMS | SP | Sim | Redução de base, CST e texto legal do benefício |
| ICMS | (em branco) | Não | Tratamento das demais operações |
| PIS | (em branco) | Não | Tratamento de PIS |
| COFINS | (em branco) | Não | Tratamento de COFINS |

- Filial de SP vendendo para cliente de SP: vale a primeira linha de ICMS.
- A mesma filial vendendo para cliente do RJ: vale a segunda linha de ICMS.
- Compra da filial de SP de fornecedor de SP: vale a primeira linha de ICMS.
- Compra da filial de SP de fornecedor do PR: vale a segunda linha de ICMS.
- Em todas essas operações, as linhas de PIS e COFINS são aplicadas.

O resultado é o mesmo em vendas, compras, documentos fiscais e transferências, e aparece igual nas estatísticas do documento, no documento registrado e na NF-e. Se a **Localização Cliente/Fornecedor** do documento mudar (troca de cliente, de endereço ou do indicador de presença), o cálculo dos impostos acompanha a nova situação. O CST é sugerido de novo nas linhas cuja exceção tem linha marcada.

---

## IBPT (Carga Tributária Estimada)

### O que é

O **IBPT** (Instituto Brasileiro de Planejamento e Tributação) fornece uma base de dados com a carga tributária estimada. A Lei da Transparência (12.741/2013) exige que as notas fiscais informem ao consumidor o valor aproximado dos tributos.

### No sistema

- A tabela de IBPT é importada periodicamente (o IBPT publica atualizações a cada semestre)
- Para **mercadorias**: busca pela NCM do item
- Para **serviços**: busca pelo código de serviço (NBS ou LC 116, conforme configurado no cadastro de serviços NFS-e)
- As alíquotas são separadas por origem (nacional/importada) e esfera (federal, estadual, municipal)
- O valor é incluído automaticamente como texto legal na NF-e, desde que o flag **Texto Legal Valor Aprox. Impostos Automático** esteja ativo na Config. NF-e

**Caminho no BC**: Diga-me > "IBPT"

---

## Outros códigos importantes

| Código | O que é | Uso |
|--------|---------|-----|
| **NOP** (Natureza da Operação) | Descrição textual da operação na NF-e. Utilizado nas matrizes fiscais — conceito mais avançado que não será abordado neste momento | "Venda de mercadoria", "Remessa para industrialização" |
| **ANP** | Código da Agência Nacional do Petróleo | Obrigatório para combustíveis |
| **CEST** | Código Especificador da Substituição Tributária | Identifica a mercadoria na ST |
| **NBS** | Nomenclatura Brasileira de Serviços | Equivalente ao NCM para serviços |
| **Origem ICMS** | Origem da mercadoria (nacional, importada, etc.) | Primeiro dígito do CST ICMS |

---

## Como tudo se encaixa

O fluxo de configuração completo:

```
1. Cadastrar códigos base: CFOP, NCM, CST
2. Configurar Jurisdições (impostos) e Contabilizações de Imposto
3. Criar Áreas de Impostos (agrupamento de jurisdições)
4. Montar Matrizes de Impostos (configuração fiscal por operação → Área Impostos + CSTs)
5. Configurar Exceções (casos especiais)
6. Configurar Alíquotas ICMS por UF (ICMS interestadual)
7. Importar IBPT (carga tributária estimada)

Ao calcular impostos de uma linha de documento:
→ A Matriz Impostos define a Área Impostos e os CSTs
→ A Área Impostos define quais impostos calcular (jurisdições)
→ Cada jurisdição busca a alíquota conforme sua configuração (Alíquotas ICMS por UF, exceções, etc.)
→ Os flags da Contabilização Imposto determinam o efeito contábil
→ O sistema calcula os valores
```

---

## Pontos de atenção

- **Matriz Impostos sem configuração para a operação** = impostos não calculados ou zerados. Verifique se existe um registro na matriz para o tipo de operação.
- **NCM errado no item** = alíquotas de IPI e valores de IBPT incorretos. O NCM deve ser sempre conferido com a TIPI vigente.
- **CST inconsistente** = pode gerar rejeição da NF-e na SEFAZ (ex: CST 00 com alíquota zero).
- **Exceção fiscal vencida** = muitos benefícios estaduais têm prazo de validade. Exceções expiradas devem ser removidas ou atualizadas.
- **Linha de exceção marcada Mesmo Estado sem alternativa** = nas operações interestaduais, de importação e de exportação, o imposto segue a tributação normal da área de imposto e da matriz. Se a operação interestadual também tem tratamento especial, cadastre-o em outra linha da mesma exceção.
- **IBPT desatualizado** = o texto legal na NF-e pode mostrar valores defasados. Atualize a cada publicação do IBPT.

---

## Para praticar

1. Abra a tabela de **CFOP** e localize os CFOPs de venda dentro do estado (5.xxx) e interestadual (6.xxx)
2. Abra a tabela de **NCM** e localize o NCM de um item do ambiente de teste
3. Abra a **Matriz Impostos** e explore um registro — identifique qual Área Impostos ele define e quais CFOPs por situação (mesmo estado, interestadual, exterior)
4. Abra a **Alíquotas ICMS por UF** e confira a alíquota ICMS de SP para MG
5. **Exercício de rastreamento**: a partir de um registro da Matriz Impostos, rastreie o caminho completo: Matriz → Área Impostos → Jurisdições → Alíquotas ICMS por UF → Contabilização Imposto → Contas Impostos

**Referência**: consulte o manual [Configuração Fiscal: Cadastros Fiscais e Impostos](Localization-for-Brazil_User-Manual_Configuracao-Fiscal.md).
