# NF-e: Unidade de Medida Tributável para Ativos Fixos

**Data**: Fevereiro/2026

## Sumário

Esta atualização adiciona suporte à **Unidade de Medida Tributável (Comércio Exterior)** para Ativos Fixos na geração de NF-e. Anteriormente, a conversão de unidades de medida para exportação era suportada apenas para Itens. Agora, Ativos Fixos possuem sua própria tabela de unidades de medida com fatores de conversão específicos para operações de comércio exterior.

## 1. Contexto

### O que é

Ao emitir uma NF-e de exportação, a legislação exige que determinados produtos informem uma **Unidade de Medida Tributável** diferente da unidade comercial utilizada na linha do documento. Essa unidade é definida pelo código NCM (campo *Unid. Medida Comex NF-e* na tabela de NCM) e representa a unidade padronizada para fins de comércio exterior.

Para **Itens**, o Business Central já possui a tabela padrão *Unidades de Medida do Item* (Item Unit of Measure), onde o fator de conversão é configurado. Para **Ativos Fixos**, no entanto, essa tabela padrão não existe. Esta atualização resolve essa limitação criando uma tabela dedicada de unidades de medida para Ativos Fixos.

### Quando se aplica

A conversão de unidade tributável é utilizada nas seguintes situações:

- NF-e de **saída para o exterior** (exportação)
- **Devolução de exportação** (quando configurado na Configuração NF-e)
- Operações com CFOPs de exportação/importação: 1501, 2501, 5501, 5502, 5504, 5505, 6501, 6502, 6504, 6505

## 2. Unidades de Medida do Ativo Fixo

### Nova Tabela

Foi criada a tabela **Unidades de Medida do Ativo Fixo** (FAUnitOfMeasureFNX), com a seguinte estrutura:

| **Campo** | **Tipo** | **Descrição** |
| --- | --- | --- |
| **Nº Ativo Fixo** | Código (20) | Número do ativo fixo. Relacionamento com a tabela de Ativos Fixos |
| **Código** | Código (10) | Código da unidade de medida. Relacionamento com a tabela de Unidades de Medida |
| **Qtd. por Unidade de Medida (Comex)** | Decimal | Fator de conversão entre a unidade base do ativo fixo e a unidade de comércio exterior |

### Regras de Validação

- O campo **Qtd. por Unidade de Medida (Comex)** não pode ser zero
- Quando a unidade de medida informada é a mesma unidade base do ativo fixo, o fator de conversão deve ser obrigatoriamente **1**
- O valor inicial padrão do fator de conversão é **1**

### Página de Lista

A página **Unidades de Medida do Ativo Fixo** (FAUnitsOfMeasureFNX) exibe as unidades cadastradas para um ativo fixo.

## 3. Configuração

### Passo 1: Cadastrar a Unidade de Medida no NCM

1. Navegue até **Códigos NCM** (página NCMCodes)
2. No código NCM desejado, preencha o campo **Unid. Medida Comex NF-e** (ForeignCommNFeUOM) com a unidade de medida exigida para comércio exterior (ex: KG, UN, M)

### Passo 2: Configurar o Fator de Conversão no Ativo Fixo

1. Navegue até **Cartão do Ativo Fixo** (página Fixed Asset Card)
2. Certifique-se de que o campo **Cód. Unidade de Medida** está preenchido (unidade base do ativo)
3. No menu **Ativo Fixo**, clique na ação **Unidades de Medida**
4. Adicione a unidade de medida de comércio exterior (deve ser a mesma configurada no NCM)
5. Informe o fator de conversão no campo **Qtd. por Unidade de Medida (Comex)**

### Exemplo Prático

Um ativo fixo é comercializado em unidades (UN), mas o NCM exige que a exportação informe a quantidade em quilogramas (KG). Cada unidade pesa 2,5 kg:

| **Ativo Fixo** | **Unidade Base** | **Unid. Medida Comex (NCM)** | **Fator de Conversão (Comex)** |
| --- | --- | --- | --- |
| AF-001 | UN | KG | 0,4 |

**Nota**: O fator de conversão segue a mesma semântica do campo padrão *Qtd. por Unidade de Medida* do BC: quantas unidades base cabem em 1 unidade de destino. Como 1 KG = 0,4 UN (ou seja, 1 UN = 2,5 KG), o fator é **0,4**.

Na NF-e de exportação com 10 unidades, o sistema calculará:

- **Quantidade Comercial**: 10 UN
- **Quantidade Tributável**: 25 KG (10 ÷ 0,4)
- **Unidade Tributável**: KG

## 4. Comportamento na Geração da NF-e

### Fluxo de Processamento

Ao gerar a NF-e para uma linha com Ativo Fixo e operação de comércio exterior:

1. O sistema verifica se o NCM da linha possui uma unidade de medida de comércio exterior configurada
2. Se a unidade de comércio exterior for diferente da unidade da linha, o sistema busca o fator de conversão na tabela **Unidades de Medida do Ativo Fixo**
3. A quantidade tributável é calculada: Quantidade da Linha ÷ Fator de Conversão
4. Os campos qTrib (quantidade tributável) e uTrib (unidade tributável) são preenchidos no XML da NF-e

### Diferenças em Relação a Itens

| **Aspecto** | **Item** | **Ativo Fixo** |
| --- | --- | --- |
| Tabela de conversão | Unidades de Medida do Item (padrão BC) | Unidades de Medida do Ativo Fixo (nova) |
| Campo do fator de conversão | Qtd. por Unidade de Medida (Comex) do Item | Qtd. por Unidade de Medida (Comex) do AF |
| Pesquisa de GTIN | Busca GTIN por unidade de medida tributável | Não se aplica (ignorado para Ativos Fixos) |
| Cálculo da quantidade base | Converte pela unidade da linha, se necessário | Usa diretamente a quantidade da linha |

## 5. Resumo dos Campos Novos

### Unidades de Medida do Ativo Fixo (nova tabela)

| **Campo** | **Tipo** | **Descrição** |
| --- | --- | --- |
| Nº Ativo Fixo | Código (20) | Referência ao ativo fixo |
| Código | Código (10) | Código da unidade de medida |
| Qtd. por Unidade de Medida (Comex) | Decimal | Fator de conversão para comércio exterior na NF-e |

### Cartão do Ativo Fixo (nova ação)

| **Ação** | **Localização** | **Descrição** |
| --- | --- | --- |
| Unidades de Medida | Menu Ativo Fixo | Abre a lista de unidades de medida do ativo fixo com fatores de conversão Comex |
