<!-- Commits: 057d1b94 -->
# IBPT: Tabela de Tributos por UF

**Data**: Dezembro/2022

---

## Sumário

Esta atualização adiciona suporte à importação de tabelas IBPT **por UF** (Unidade Federativa), permitindo que cada estado possua percentuais de tributos diferenciados. Anteriormente, a tabela IBPT era única para todo o país. Agora, ao importar um arquivo CSV por UF, os códigos recebem um sufixo `-UF` (ex: `84713012-SP`), e o cálculo de impostos estimados prioriza a tabela da UF da filial antes de usar a tabela nacional. Também foi corrigida a exibição do percentual de tributos estimados nas estatísticas de vendas.

## 1. Contexto

### O que é

A tabela IBPT (Instituto Brasileiro de Planejamento e Tributação) contém os percentuais estimados de tributos para cada NCM ou código de serviço, exibidos no DANFe e NFS-e conforme a Lei da Transparência Fiscal (Lei 12.741/2012). O IBPT disponibiliza tabelas tanto nacionais quanto por UF, com percentuais que podem variar significativamente entre estados.

### Quando se aplica

- Empresas que operam em múltiplas UFs e precisam de percentuais de tributos estimados mais precisos
- O cálculo afeta o campo **Valor Aprox. Tributos** exibido no DANFe, NFS-e e nas estatísticas de vendas

---

## 2. Importação por UF

### Como importar

1. Navegue até **Impostos IBPT** (página `IBPTTaxesFNX`) via pesquisa do Business Central
2. Clique na ação **Importação CSV (por UF)** (`TerritoryImport`)
3. Selecione a **UF** desejada na lista de territórios
4. Selecione o arquivo CSV da tabela IBPT correspondente àquela UF
5. Confirme a importação

### Convenção de nomenclatura

Ao importar por UF, cada código NCM/serviço recebe o sufixo `-UF`. Por exemplo:

| Código Original | Código com UF |
|----------------|---------------|
| `84713012` | `84713012-SP` |
| `84713012` | `84713012-RJ` |
| `84713012` | `84713012` (nacional, sem sufixo) |

A importação por UF **remove apenas os registros daquela UF** (filtro `*-UF`), mantendo as demais UFs e a tabela nacional intactas.

### Visualização por NCM

Ao consultar os impostos IBPT a partir da página **Códigos NCM** (`NCMCodesFNX`), o sistema exibe tanto o registro nacional quanto os registros por UF do NCM selecionado.

---

## 3. Comportamento no Cálculo

### Prioridade de tabela

Ao calcular o percentual de tributos estimados, o sistema segue a seguinte ordem de prioridade:

1. **Tabela da UF da filial** — busca o código com sufixo da UF do documento (ex: `84713012-SP`)
2. **Tabela nacional** — se não encontrar registro por UF, usa o código sem sufixo (ex: `84713012`)

A UF é obtida automaticamente a partir do **Cód. UF** (`TerritoryCodeFNX`) da filial associada ao documento de venda.

### Códigos de serviço

Para itens de serviço, o campo **Cód. IBPT** do **Código de Serviço NFS-e** (`NFSServiceCodeFNX`) é utilizado. O sufixo `-UF` é removido automaticamente ao preencher o campo, pois a busca por UF é feita dinamicamente no momento do cálculo.

---

## 4. Estatísticas de Vendas

Na página **Estatísticas de Vendas** (`Sales Statistics`), o campo **Valor Aprox. Tributos** agora exibe o percentual entre parênteses: **Valor Aprox. Tributos (X,XX%)**. O percentual é calculado como a proporção do valor estimado de tributos sobre o valor total das linhas.
