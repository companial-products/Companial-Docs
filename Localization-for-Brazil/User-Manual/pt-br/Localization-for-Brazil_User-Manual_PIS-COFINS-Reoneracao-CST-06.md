<!-- PR: #29 -->
# EFD Contribuições: Reoneração PIS/COFINS (LC 224/2025)

**Data**: Abril/2026

---

## Sumário

Esta atualização implementa o cálculo automático da **reoneração de PIS/COFINS**, conforme determinado pela **Lei Complementar 224/2025** e orientado pela **NT 012/2026** da Receita Federal. A nota fiscal continua sendo emitida normalmente com o CST original — o sistema calcula um valor de "recomposição" aplicando um percentual sobre a alíquota padrão e gera os ajustes nos registros **M210/M220/M225** (PIS) e **M610/M620/M625** (COFINS) do arquivo da EFD-Contribuições.

## 1. Contexto

### O que é

A **Lei Complementar 224/2025** introduziu a redução linear de incentivos e benefícios fiscais de PIS/COFINS. Operações que antes eram tributadas à alíquota zero (CST 06) ou isentas (CST 07) passam a ter parte da carga tributária restabelecida (reoneração), de forma progressiva e linear.

A NT 012/2026 da Receita Federal orienta como escriturar essas operações na EFD-Contribuições:

1. A nota fiscal continua sendo emitida com o CST original (06 ou 07), sem alteração.
2. O contribuinte calcula o valor da reoneração aplicando um percentual sobre a alíquota padrão de PIS/COFINS.
3. Os valores são escriturados como **ajustes de acréscimo** nos registros M220 (PIS) e M620 (COFINS).
4. Cada documento é detalhado nos registros **M225** (PIS) e **M625** (COFINS) com a chave da NF-e.

### Quando se aplica

- Operações de **saída** com CST listado na **Matriz de Reoneração PIS/COFINS** (atualmente CST 06; eventualmente CST 07 conforme novas vigências da legislação)
- Empresas que entregam **EFD-Contribuições**, em **Regime de Competência** ou **Regime de Caixa**
- A partir da data de vigência configurada na matriz (inicialmente, a NT 012/2026 prevê 10% de reoneração a partir de 01/04/2026)

### Cenários suportados

| Cenário | Base de Cálculo |
| --- | --- |
| Operação sem ICMS | **Valor da Linha** |
| Operação com ICMS, matriz com **Deduzir ICMS da Base** marcado | **Valor da Linha − Valor ICMS** |
| Operação com ICMS, matriz com **Deduzir ICMS da Base** desmarcado | **Valor da Linha** |
| Múltiplos documentos no período | Cálculo por linha; um único ajuste consolidado por tributo |
| Múltiplos CSTs com vigências distintas | Cada linha usa a configuração da matriz vigente para seu CST e data |

---

## 2. Configuração

### Passo 1: Habilitar a reoneração

1. Navegue até a página **Configuração EFD Contribuições** (`EFDContribSetupFNX`).
2. Na seção **Reoneração PIS/COFINS**, marque o campo **Habilitar Reoneração PIS/COFINS** (`EnableReoneration`).

Sem essa marcação, o cálculo não é executado durante a apuração — o comportamento permanece idêntico ao das versões anteriores.

### Passo 2: Cadastrar a Matriz de Reoneração

A matriz define, para cada CST e data de início de vigência, o percentual de reoneração e as alíquotas padrão de PIS/COFINS aplicáveis.

1. Na **Configuração EFD Contribuições**, clique na ação **Matriz de Reoneração** (`ReonerationMatrix`).
2. Adicione uma linha para cada combinação de CST e data de vigência.

Campos da matriz:

| Campo | Descrição |
| --- | --- |
| **Cód. CST** (`CSTCode`) | Código CST de PIS/COFINS sujeito à reoneração (ex: 06) |
| **Data Início** (`StartingDate`) | Data a partir da qual esta configuração é aplicada |
| **% Reoneração** (`ReonerationPercent`) | Percentual de reoneração sobre a alíquota padrão (ex: 10,00 para 10%) |
| **Alíquota Padrão PIS** (`StandardPISRate`) | Alíquota padrão de PIS de referência (ex: 1,65 para Não-Cumulativo, 0,65 para Cumulativo) |
| **Alíquota Padrão COFINS** (`StandardCOFINSRate`) | Alíquota padrão de COFINS de referência (ex: 7,60 para Não-Cumulativo, 3,00 para Cumulativo) |
| **Deduzir ICMS da Base** (`DeductICMSFromBase`) | Se marcado, o **Valor ICMS** da linha é subtraído da base de cálculo da reoneração |
| **Descrição** (`Description`) | Descrição livre da configuração (ex: "LC 224/25 — Reoneração 10%") |

### Exemplo de configuração — NT 012/2026

| **Cód. CST** | **Data Início** | **% Reoneração** | **Alíquota Padrão PIS** | **Alíquota Padrão COFINS** | **Deduzir ICMS da Base** |
| --- | --- | --- | --- | --- | --- |
| 06 | 01/04/2026 | 10,00 | 1,65 | 7,60 | Sim |

Quando o cálculo é executado, o sistema busca, para cada documento, o registro mais recente da matriz com **Data Início ≤ Data Registro do documento** e **Cód. CST** correspondente. Isso permite cadastrar, no futuro, novas vigências (ex: 20% a partir de 01/01/2027) sem alterar as configurações anteriores.

---

## 3. Cálculo da Reoneração

### Quando é executado

O cálculo é executado automaticamente durante o **Processamento da Apuração PIS/COFINS** (na criação das linhas de PIS/COFINS do período). Não é necessário rodar uma ação separada — basta processar a apuração normalmente.

O cálculo é **idempotente**: se a apuração for reprocessada, os movimentos e o ajuste consolidado são recriados sem duplicação.

### Fórmulas

Para cada linha de documento de saída com CST elegível na matriz:

```
Alíquota Efetiva = Alíquota Padrão × % Reoneração / 100
Base de Cálculo  = Base Inicial − Valor ICMS  (quando "Deduzir ICMS da Base" estiver marcado)
Valor do Ajuste  = Base de Cálculo × Alíquota Efetiva / 100
```

A **Base Inicial** depende do critério de escrituração configurado em **Critério de Escrituração do Regime Cumulativo**:

- **Regime de Competência — Detalhado**: usa o **Valor da Linha** (`LineAmount`) do documento, sem outros impostos somados.
- **Regime de Caixa**: usa o valor recebido no período (campo já existente que considera os recebimentos parciais).

### Geração dos movimentos

Os movimentos são **agregados por chave única** `(Chave NF-e, Conta Contábil, CST, Tributo)`. Linhas do mesmo documento que apontam para a mesma conta contábil e o mesmo CST somam-se em um único movimento (um para PIS e outro para COFINS).

Essa agregação é necessária porque os registros **M225/M625** do SPED têm chave única `(CST, DET_ALIQ, DESC_AJ, COD_CTA, INFO_COMPL)`. Sem o agrupamento, o validador da Receita Federal recusa o arquivo por duplicidade quando há mais de uma linha do mesmo documento na mesma conta de receita.

### Ajuste consolidado

Após processar todas as linhas, o sistema gera um único **ajuste de acréscimo** em **Apuração — Valores Adicionais** (tabela `TaxSettlementAmountsFNX`) por tributo:

- **Tipo de Ajuste**: Outros Débitos (IND_AJ = 1)
- **Cód. Ajuste**: `11` (redução linear de benefícios fiscais — tabela 4.3.8 SPED)
- **Descrição**: "Redução linear de benefícios fiscais — LC 224/2025"
- **Valor**: soma de todos os movimentos do tributo
- **Valor Base**: soma das bases de cálculo

O ajuste é marcado como **System Created Entry**, o que permite distingui-lo de ajustes manuais e garante a recriação a cada reprocessamento.

---

## 4. Movimentos de Reoneração

A página **Movimentos de Reoneração PIS/COFINS** (`PISCOFINSReonerationEntries`) lista, para cada apuração, todos os movimentos agregados que geraram reoneração — separadamente por tributo (PIS e COFINS).

Cada movimento contém:

- Informações do documento de origem (**Nº Documento**, **Nº Fiscal**, **Chave de Acesso NF-e**, **Data Registro**)
- Identificação do participante (**Nº Participante**, **Nome do Participante**, **CNPJ/CPF**)
- Conta contábil (**Nº Conta**) reportada no campo `COD_CTA` do M225/M625
- Cálculo (**Valor da Linha**, **Valor ICMS**, **Valor Base**, **Alíquota Padrão**, **% Reoneração**, **Alíquota Efetiva**, **Valor**)
- Indicação se é PIS ou COFINS (**Identificação do Tributo**)

Esta tabela é a fonte para os registros **M225** e **M625** do arquivo SPED — cada movimento gera uma linha de detalhamento.

---

## 5. Reflexo no Arquivo SPED

Quando o arquivo da EFD-Contribuições é gerado, os seguintes registros são afetados:

### Registros novos

| Registro | Conteúdo | Origem |
| --- | --- | --- |
| **M225** | Detalhamento dos ajustes da contribuição PIS, com a chave NF-e em `DESC_AJ` | Movimentos de Reoneração (filtro: PIS) |
| **M625** | Detalhamento dos ajustes da contribuição COFINS, com a chave NF-e em `DESC_AJ` | Movimentos de Reoneração (filtro: COFINS) |

Cada M225/M625 traz: valor do ajuste (`DET_VALOR_AJ`), CST, base de cálculo (`DET_BC_CRED`), alíquota efetiva (`DET_ALIQ`), data de operação (`DT_OPER`), chave de acesso da NF-e (`DESC_AJ`), conta contábil (`COD_CTA`) e descrição complementar (`INFO_COMPL`).

### Registros alterados

| Registro | Campo | Comportamento Anterior | Novo Comportamento |
| --- | --- | --- | --- |
| **M210** (PIS) | `VL_AJUS_ACRES` | Zero | Soma dos ajustes de acréscimo da apuração (Outros Débitos + Estornos de Crédito) |
| **M210** (PIS) | `VL_AJUS_REDUC` | Zero | Soma dos ajustes de redução (Outros Créditos + Estornos de Débito) |
| **M210** (PIS) | `VL_CONT_PER` | Zero | `VL_AJUS_ACRES − VL_AJUS_REDUC` (mínimo zero) |
| **M610** (COFINS) | `VL_AJUS_ACRES` / `VL_AJUS_REDUC` / `VL_CONT_PER` | Zero | Idem ao M210, considerando apenas COFINS |
| **M220** (PIS) | — | Já existente | Passa a incluir o ajuste consolidado da reoneração com `IND_AJ = 1` e `COD_AJ = 11` |
| **M620** (COFINS) | — | Já existente | Idem M220, para COFINS |

### Registros não alterados

Os registros **M400/M410** (PIS) e **M800/M810** (COFINS), que detalham receitas por CST, **continuam exibindo o CST original normalmente**. A reoneração não altera o CST da operação — apenas adiciona o ajuste de acréscimo nos registros M210/M220/M225 (PIS) e M610/M620/M625 (COFINS).

---

## 6. Exemplos Numéricos

### Exemplo 1 — CST 06 sem ICMS

Nota fiscal de saída com CST 06, valor de R$ 250,00, sem ICMS. Matriz: 10% de reoneração, PIS 1,65%, COFINS 7,60%.

| Tributo | Cálculo | Valor |
| --- | --- | --- |
| Base de Cálculo | 250,00 (sem dedução de ICMS) | 250,00 |
| Alíquota Efetiva PIS | 1,65% × 10% | 0,1650% |
| **PIS — Valor do Ajuste** | 250,00 × 0,1650% | **R$ 0,41** |
| Alíquota Efetiva COFINS | 7,60% × 10% | 0,7600% |
| **COFINS — Valor do Ajuste** | 250,00 × 0,7600% | **R$ 1,90** |

### Exemplo 2 — CST 06 com ICMS

Nota fiscal de saída com CST 06, **Valor da Linha** R$ 33.520,50, **Valor ICMS** R$ 1.608,98. Matriz com **Deduzir ICMS da Base** marcado.

| Componente | Cálculo | Valor |
| --- | --- | --- |
| Valor da Linha | — | 33.520,50 |
| Valor ICMS | — | 1.608,98 |
| Base de Cálculo | 33.520,50 − 1.608,98 | 31.911,52 |
| **PIS — Valor do Ajuste** | 31.911,52 × 0,1650% | **R$ 52,65** |
| **COFINS — Valor do Ajuste** | 31.911,52 × 0,7600% | **R$ 242,53** |

### Exemplo 3 — Múltiplos documentos no período

Três notas com CST 06 no mesmo período de apuração, cada uma com uma única conta de receita. O sistema gera três pares de movimentos (PIS + COFINS) e um único ajuste consolidado por tributo.

| Documento | Base | PIS | COFINS |
| --- | --- | --- | --- |
| NF 001 | 10.000,00 | 16,50 | 76,00 |
| NF 002 | 25.000,00 | 41,25 | 190,00 |
| NF 003 | 15.000,00 | 24,75 | 114,00 |
| **Total (M220 / M620)** | **50.000,00** | **R$ 82,50** | **R$ 380,00** |

No arquivo SPED:
- 1 registro **M220** (PIS) com `VL_AJ = 82,50`, `COD_AJ = 11`
- 3 registros **M225** (um por NF, com a chave NF-e em `DESC_AJ`)
- 1 registro **M620** (COFINS) com `VL_AJ = 380,00`, `COD_AJ = 11`
- 3 registros **M625** (um por NF)

### Exemplo 4 — Documento com múltiplas linhas na mesma conta

Documento com 2 linhas, ambas com CST 06 e mesma conta de receita. **Valor da Linha** R$ 2.000 / R$ 4.000, **Valor ICMS** R$ 400 / R$ 800. Matriz com **Deduzir ICMS da Base** marcado.

| Linha | Valor da Linha | Valor ICMS | Base de Cálculo |
| --- | --- | --- | --- |
| 1 | 2.000,00 | 400,00 | 1.600,00 |
| 2 | 4.000,00 | 800,00 | 3.200,00 |

O sistema **agrega** as linhas em um único movimento PIS e um único movimento COFINS:

| Tributo | Base Agregada | Cálculo | Valor |
| --- | --- | --- | --- |
| **PIS** | 4.800,00 | 4.800,00 × 0,1650% | **R$ 7,92** |
| **COFINS** | 4.800,00 | 4.800,00 × 0,7600% | **R$ 36,48** |

No arquivo SPED são gerados **2 registros M225 e 2 registros M625** no total para este documento (não 4 de cada). Se as linhas estivessem em contas contábeis diferentes, seriam gerados movimentos separados por conta.

---

## 7. Resumo dos Campos Novos

### Configuração EFD Contribuições (`EFDContribSetupFNX`)

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Habilitar Reoneração PIS/COFINS** (`EnableReoneration`) | Booleano | Ativa o cálculo automático da reoneração durante o processamento da apuração |

### Matriz de Reoneração PIS/COFINS (`PISCOFINSReonerationMatrix`) — tabela nova

Acessível via ação **Matriz de Reoneração** na Configuração EFD Contribuições.

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Cód. CST** (`CSTCode`) | Código | Código CST de PIS/COFINS (ex: 06) |
| **Data Início** (`StartingDate`) | Data | Data de início de vigência da configuração |
| **% Reoneração** (`ReonerationPercent`) | Decimal | Percentual de reoneração aplicado sobre a alíquota padrão |
| **Alíquota Padrão PIS** (`StandardPISRate`) | Decimal | Alíquota padrão de PIS de referência |
| **Alíquota Padrão COFINS** (`StandardCOFINSRate`) | Decimal | Alíquota padrão de COFINS de referência |
| **Deduzir ICMS da Base** (`DeductICMSFromBase`) | Booleano | Indica se o **Valor ICMS** deve ser subtraído da base de cálculo |
| **Descrição** (`Description`) | Texto | Descrição livre |

### Movimento de Reoneração PIS/COFINS (`PISCOFINSReonerationEntry`) — tabela nova

Visualizável na página **Movimentos de Reoneração PIS/COFINS** (`PISCOFINSReonerationEntries`). Tabela alimentada automaticamente pelo cálculo a cada processamento de apuração — não é editável.

| Campo | Tipo | Descrição |
| --- | --- | --- |
| **Nº Apuração Fiscal** (`TaxSettlementNo`) | Código | Apuração à qual o movimento pertence |
| **Nº Movimento** (`EntryNo`) | Inteiro | Número sequencial |
| **Identificação do Tributo** (`TaxIdentification`) | Enum | PIS ou COFINS |
| **Nº Documento** (`DocumentNo`) | Código | Número interno do documento de origem |
| **Nº Fiscal** (`FiscalNo`) | Código | Número fiscal do documento de origem |
| **Chave de Acesso NF-e** (`NFeAccessKey`) | Código | Chave reportada em `DESC_AJ` do M225/M625 |
| **Data Registro** (`PostingDate`) | Data | Data de operação do documento |
| **Cód. CST** (`CSTCode`) | Código | CST que originou a reoneração |
| **Valor da Linha** (`LineAmount`) | Decimal | Valor original da linha do documento |
| **Valor ICMS** (`ICMSAmount`) | Decimal | Valor do ICMS deduzido da base, quando aplicável |
| **Valor Base** (`BaseAmount`) | Decimal | Base de cálculo efetiva |
| **Alíquota Padrão** (`StandardRate`) | Decimal | 1,65/0,65 para PIS, 7,60/3,00 para COFINS |
| **% Reoneração** (`ReonerationPercent`) | Decimal | Percentual aplicado |
| **Alíquota Efetiva** (`EffectiveRate`) | Decimal | Alíquota Padrão × % Reoneração / 100 |
| **Valor** (`Amount`) | Decimal | Valor calculado da reoneração |
| **Nº Participante** (`ParticipantNo`) | Código | Cliente/fornecedor da operação |
| **Nome do Participante** (`ParticipantName`) | Texto | Nome do participante |
| **CNPJ/CPF** (`CNPJCPF`) | Código | CNPJ/CPF do participante |
| **Modelo do Documento Fiscal** (`FiscalDocumentModel`) | Texto | Modelo do documento fiscal |
| **Nº Conta** (`GLAccountNo`) | Código | Conta contábil reportada em `COD_CTA` do M225/M625 |
| **Descrição** (`Description`) | Texto | Descrição do ajuste |

### Compatibilidade

- **Clientes existentes**: enquanto a flag **Habilitar Reoneração PIS/COFINS** estiver desmarcada, o comportamento é idêntico ao anterior — nenhum cálculo, nenhum registro M225/M625, nenhum ajuste em M210/M610.
- **Reprocessamento**: o cálculo limpa os movimentos e o ajuste consolidado da apuração antes de recalcular. Ajustes manuais com outros códigos de ajuste (≠ 11) são preservados.
- **Apurações já fechadas**: para apurações criadas antes da atualização, será necessário reprocessar a apuração para que os campos **Valor da Linha** e **Valor ICMS** sejam populados nas linhas de PIS/COFINS e os movimentos de reoneração sejam gerados.
- **Regime de Caixa — aumento esperado do valor devido a partir da correção das receitas sem incidência**: até esta atualização, o Regime de Caixa descartava da apuração os documentos de saída recebidos no período cuja base de PIS e de COFINS fossem ambas zero, ou seja, todas as operações sem contribuição devida (CST 06 alíquota zero, 07 isenta, 08 sem incidência e 09 suspensão). Com a correção, essas receitas voltam a ser escrituradas nos registros F500, F525 e 1900 — e, por consequência, passam a ser varridas pelo cálculo da reoneração. **Em empresa com a reoneração habilitada e um CST de não incidência cadastrado na Matriz de Reoneração, o valor devido pode subir em relação ao mês anterior.** Isso é consequência esperada da receita voltar a ser escriturada, e não um erro novo: nenhuma regra de cálculo da reoneração foi alterada. O efeito se concentra em **CST 06** e **CST 07**, que são os benefícios fiscais alcançados pela LC 224/2025; a exportação (**CST 08**) é imunidade constitucional, não benefício fiscal, e não deve constar na matriz. A correção vale para as apurações geradas a partir dela — períodos já fechados ou já transmitidos à Receita Federal não são reprocessados nem retificados automaticamente.
- **Regime de Caixa — notas com linhas que não são receita passam a escriturar o valor recebido integralmente**: o rateio do valor recebido entre as linhas da nota passou a considerar apenas as linhas de receita. Numa nota que traz, ao lado da venda, uma linha sem natureza de receita (uma remessa, por exemplo), o **Valor Receita** da linha de venda deixa de ser diluído pelo valor da outra linha e passa a corresponder ao valor efetivamente recebido. Como o **Valor Receita** é a base da reoneração no Regime de Caixa, o ajuste dessas notas sobe na mesma proporção. Notas com uma única linha, ou cujas linhas sejam todas de receita, não têm nenhuma alteração de valor.
