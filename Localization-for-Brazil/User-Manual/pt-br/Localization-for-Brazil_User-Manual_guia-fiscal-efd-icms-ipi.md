# EFD-ICMS/IPI (SPED Fiscal)

O EFD-ICMS/IPI é a maior e mais complexa obrigação acessória gerada pelo app de **Livros Fiscais** (Tax Books). Ele substitui os antigos livros fiscais de entrada, saída e apuração por um arquivo digital transmitido mensalmente ao SPED.

---

## O que é

A Escrituração Fiscal Digital do ICMS e do IPI consolida **todos os documentos fiscais** do período (entradas e saídas), calcula os débitos e créditos de ICMS e IPI, e gera um arquivo texto no formato definido pela Receita Federal e pelo CONFAZ.

O arquivo é obrigatório para todas as empresas que recolhem ICMS e IPI.

### Formato

- Arquivo texto com separador pipe (`|`)
- Codificação Windows/ANSI
- Cada linha é um registro, iniciando e terminando com `|`
- Organizado em **blocos** (0, B, C, D, E, G, H, K, 1, 9)

---

## Estrutura de blocos

| Bloco | Nome | O que contém |
|-------|------|-------------|
| **0** | Abertura e Identificação | Dados da empresa, participantes, itens, unidades de medida, plano de contas, naturezas de operação |
| **C** | Documentos Fiscais I | NF-e de mercadorias — registros detalhados de cada documento de entrada e saída |
| **D** | Documentos Fiscais II | Documentos de serviço e transporte (CT-e, contas de energia, telecomunicações) |
| **E** | Apuração | Apuração do ICMS e IPI — débitos, créditos, ajustes, saldo a pagar/restituir |
| **G** | CIAP | Crédito de ICMS sobre ativo permanente (bens do imobilizado) |
| **H** | Inventário Físico | Estoque físico no final do período |
| **K** | Produção e Estoque | Movimentação de produção (para industriais) |
| **1** | Outras Informações | Exportações, créditos fiscais, GIA |

### Registros mais importantes

Para o dia a dia, os registros que você mais vai conferir:

| Registro | O que é | Por que importa |
|----------|---------|----------------|
| **0200** | Cadastro de itens | Se o item não aparece aqui, algo está errado no documento fiscal |
| **C100** | Cabeçalho do documento fiscal (NF-e) | Um registro para cada NF-e do período |
| **C170** | Linhas do documento | Detalhamento: item, quantidade, CFOP, CST, impostos |
| **C190** | Consolidação por CST, CFOP e alíquota do ICMS | Totalização — é o que a SEFAZ mais valida. Linhas com o mesmo CST e CFOP e alíquotas diferentes geram registros separados |
| **E110** | Apuração do ICMS | Débitos, créditos, ajustes, saldo final |
| **E510** | Consolidação do IPI por CFOP+CST | Totalização dos valores de IPI dos documentos |
| **E520** | Apuração do IPI | Débitos, créditos e saldo final do IPI |

---

## Como a localização gera o EFD

1. O usuário cria uma **Apuração Impostos** (Tax Settlement) para o período (mês)
2. Executa o report de geração do EFD-ICMS/IPI
3. O sistema lê todos os documentos fiscais do período
4. Para cada documento, gera os registros correspondentes (C100, C170, C190, etc.)
5. Gera os registros de apuração (bloco E) a partir dos valores calculados
6. Gera o bloco 0 com os cadastros referenciados
7. Exporta o arquivo texto

### Versões de layout

O layout muda anualmente. A localização suporta as versões de V014 (2020) até V020 (2026). A versão é determinada automaticamente pela data do período, mas pode ser sobrescrita na configuração.

---

## Conferência e troubleshooting

### O que conferir no arquivo

| Verificação | Onde olhar | Problema se errado |
|------------|-----------|-------------------|
| Todos os documentos estão no arquivo? | Blocos C e D — contar documentos | Documentos fiscais fora do período ou sem registro |
| CFOP e CST estão corretos? | C170, C190 | Escrituração incorreta, apuração com valores errados |
| Item cadastrado? | 0200 | Item presente no documento mas não no registro 0200 |
| Apuração fecha? | E110 | Débitos - créditos = saldo — confere com a apuração no BC |
| Participante cadastrado? | 0150 | CNPJ/IE do participante não aparece |

### Rastreando um registro de volta ao BC

Se o PVA acusar erro em um registro (ex: C170 com valor inconsistente):

1. Identifique o **número do documento** no registro C100
2. No BC, abra os **Documentos Fiscais** e localize pelo número
3. Abra o documento fiscal e confira a linha correspondente
4. Verifique: CFOP, NCM, CST, valores de impostos
5. Se necessário, corrija a origem (configuração fiscal, matriz, exceção)

---

## Pontos de atenção

- **Registro C197/D197** — ajustes de ICMS que não estão no documento fiscal (benefícios, estornos). Estes registros são configurados separadamente.
- **Inventário (bloco H)** — obrigatório no encerramento do exercício. O BC precisa ter o inventário físico registrado.
- **CIAP (bloco G)** — crédito de ICMS sobre bens do ativo imobilizado é fracionado em 48 parcelas. A localização calcula automaticamente.
- **Versão de layout** — se a versão estiver errada, o PVA rejeita o arquivo na importação.

---

## Para praticar

1. Localize no BC a configuração de EFD-ICMS/IPI (Diga-me > "Configuração EFD ICMS/IPI")
2. Gere um arquivo EFD de teste para um mês
3. Abra o arquivo em um editor de texto e identifique: bloco 0 (cadastros), bloco C (documentos), bloco E (apuração)
4. Pegue um documento fiscal do período e localize o registro C100 correspondente no arquivo
5. Compare os valores do registro C170 com os valores da linha do documento fiscal no BC

**Referência**: consulte o manual "Fiscal Apuração ICMS-IPI" e a planilha "Lista de Registros EFD ICMS IPI" disponíveis nos documentos internos.
