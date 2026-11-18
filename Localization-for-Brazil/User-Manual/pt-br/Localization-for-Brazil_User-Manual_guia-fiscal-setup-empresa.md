<!-- PR: #297 -->
# Configuração de Empresa

Antes de usar qualquer funcionalidade fiscal, a empresa precisa estar configurada com seus dados cadastrais, filiais, inscrições e certificados. Este é o primeiro setup que se faz em uma implantação.

---

## O que é

No Brasil, uma empresa pode ter **múltiplos estabelecimentos** (filiais/unidades), cada um com CNPJ próprio (ou raiz compartilhada), inscrição estadual, inscrição municipal e configuração fiscal independente.

A localização modela isso através da entidade **Informação Filial** (Branch Information), que é o ponto central de configuração por estabelecimento.

---

## Como a localização implementa

### Informações da Empresa (Company Information)

A página padrão do BC (Informações da Empresa) contém os dados gerais da empresa: razão social, endereço, CNPJ da matriz. A localização estende essa página com campos adicionais brasileiros, mas o cadastro detalhado de cada estabelecimento fica nas **Filiais**.

### Filiais (Informação Filial)

Cada filial representa um **estabelecimento** — a matriz e cada filial. É aqui que você configura:

| Grupo | Campos principais |
|-------|-------------------|
| **Identificação** | Código, Nome, CNPJ, Inscrição Estadual (IE), Inscrição Municipal (CCM) |
| **Endereço** | Logradouro, Número, Complemento, Bairro, CEP, Cidade, UF |
| **Registros adicionais** | CEI, NIT, SUFRAMA, IE-ST em outros estados |
| **Tributação** | Regime tributário (Simples Nacional / Normal), contribuinte de ICMS, segmento industrial |
| **Dimensão** | Código de dimensão + valor — vincula a filial a uma dimensão contábil para balanceamento |
| **Bancário** | Banco, agência, conta (para operações de pagamento/recebimento) |

**Caminho no BC**: Diga-me > "Informação Filial"

### Dimensão de Filial

Um conceito importante: cada filial é associada a um **valor de dimensão**. Quando um documento fiscal é registrado, essa dimensão é automaticamente vinculada aos movimentos contábeis. Isso permite:

- Balancete por filial
- DRE por estabelecimento
- Rastreabilidade completa de qual filial gerou cada movimento

A dimensão usada é configurada na **Configuração Contabilidade** (General Ledger Setup), campo "Branch Dimension Code".

### IE-ST por Estado (Company IEST)

Quando a empresa realiza operações com **ICMS Substituição Tributária** para estados onde não tem estabelecimento, ela precisa de uma **Inscrição Estadual de Substituto Tributário** naquele estado. O cadastro fica vinculado à filial.

**Caminho**: no cartão da Filial > ação "Territory Inscriptions of the Tributary Subst."

---

## Municípios

A localização mantém uma tabela de **municípios IBGE** — todos os municípios brasileiros com código oficial de 7 dígitos. Essa tabela é usada em:

- Endereços de clientes, fornecedores e filiais
- Determinação de competência do ISS
- Geração de NFS-e (cada município pode ter um sistema diferente)
- Registros do SPED

A tabela é **compartilhada entre todas as empresas** do mesmo ambiente (DataPerCompany = false), já que os municípios são os mesmos para qualquer empresa.

Cada município pode ter:
- **Código SIAFI** — usado em transações governamentais
- **Códigos de Cidade** — mapeamento para códigos específicos de sistemas municipais de NFS-e
- **Configurações de ISS** — dispensa do ISS próprio quando o imposto já foi retido em outro município, tratamento do desconto na base, fornecedor da retenção

**Caminho no BC**: Diga-me > "Municípios"

---

## CEP

A localização possui uma tabela de **CEPs** com preenchimento automático de endereço. Quando você digita um CEP em qualquer tela que tenha campo de código postal (cliente, fornecedor, pedido, filial, etc.), o sistema:

1. Busca o CEP na tabela local
2. Se não encontrar e a consulta automática estiver ativa, consulta um webservice e cadastra o CEP
3. Preenche automaticamente cidade, bairro, UF e logradouro

Isso funciona em mais de **27 tabelas do BC** — clientes, fornecedores, pedidos de venda/compra, localizações, centros de responsabilidade, funcionários, etc.

**Caminho no BC**: Diga-me > "CEPs" ou "Post Codes"

---

## CNAE

A tabela de **Códigos CNAE** (Classificação Nacional de Atividades Econômicas) é usada no cadastro de filiais e em obrigações acessórias. Cada código tem formato XX.XX.X-XX e pode ter associada uma alíquota de CPRB (Contribuição Previdenciária sobre Receita Bruta).

**Caminho no BC**: Diga-me > "CNAE"

---

## Categorias de Participantes

A localização classifica clientes e fornecedores em **categorias fiscais** que determinam regras de tributação:

| Categoria | Quem é | Impacto fiscal |
|-----------|--------|---------------|
| **Pessoa Física** | CPF | Diferentes regras de retenção, sem IE |
| **Empresa** | CNPJ | Regras padrão de ICMS, PIS/COFINS |
| **Estrangeiro** | Sem CPF/CNPJ | Importação/exportação, sem impostos estaduais |
| **Produtor Rural** | CPF com inscrição de produtor | Regras especiais de ICMS, Funrural |
| **Não identificado** | Sem identificação (só cliente) | NFC-e para consumidor final |

Cada categoria também define: natureza da empresa (privada, órgão público, etc.), se é optante pelo Simples Nacional, se é contribuinte de ICMS.

**Caminho no BC**: Diga-me > "Categorias de Participantes"

---

## Certificado Digital

Para emitir NF-e e NFS-e, a empresa precisa de um **certificado digital A1** (arquivo .pfx). A localização gerencia certificados por filial:

- O certificado é **importado** na configuração de NF-e da filial
- Armazenado de forma segura (criptografado, não acessível como arquivo)
- Usado automaticamente na assinatura do XML da NF-e/NFS-e

Cada filial pode ter seu próprio certificado, permitindo que empresas com múltiplos CNPJs emitam NF-e por cada estabelecimento.

**Caminho**: no cartão de Config. NF-e da filial > ação "Certificados Digitais"

---

## Numeração de Documentos Fiscais

A localização configura Nº Séries para documentos fiscais, organizadas por:

- **Tipo de origem** — Ordem de Venda, Ordem de Compra, Transferência, etc.
- **Direção** — Entrada, Saída ou Único

A página gera automaticamente todas as combinações possíveis. Cada combinação aponta para uma Nº Série do BC.

**Caminho no BC**: Diga-me > "Numeração Documentos Fiscais"

---

## Config. NF-e (por Filial)

Cada filial tem uma **configuração completa de NF-e e NFS-e**, que é a maior página de setup da localização. Os principais grupos:

| Grupo | O que configura |
|-------|----------------|
| **NF-e General** | Cálculo de FCI, inutilização de numeração |
| **Transmissão** | Transmissão automática ao registrar, lote |
| **DANFe** | Campos a exibir no DANFe (lote, série, desconto, EAN) e a opção **Não imprimir canhoto**, que vale para o [DANFE da Reforma Tributária](Localization-for-Brazil_User-Manual_NFe-DANFE-Reforma-Tributaria.md) |
| **Web Services** | Ambiente (teste/produção), certificado, URLs SEFAZ, UF, contingência |
| **E-Mail** | Envio automático de NF-e por e-mail ao cliente |
| **NFS-e** | Tipo de webservice municipal, usuário/senha, séries de RPS |
| **NFS-e Bot** | Automação de transmissão de NFS-e |

**Caminho no BC**: Diga-me > "Config. NF-e" ou no cartão da Filial > ação "Config. NF-e"

---

## Pontos de atenção

- **Filial sem dimensão** = movimentos contábeis sem identificação de estabelecimento. Configure a dimensão antes de registrar qualquer documento.
- **IE (Inscrição Estadual) inválida** = NF-e rejeitada pela SEFAZ. Valide no cadastro.
- **Certificado expirado** = NF-e não pode ser assinada. Monitore a validade.
- **Regime tributário errado** = cálculo de impostos incorreto (Simples vs. Normal).
- **Município sem city code** = NFS-e não transmite. Verifique o mapeamento na tabela de municípios.

---

## Para praticar

1. Abra a lista de Filiais e examine a configuração da filial principal
2. Verifique se a dimensão de filial está preenchida — abra a Configuração Contabilidade e confira o campo "Branch Dimension Code"
3. Abra a tabela de Municípios e localize a cidade da filial
4. Abra a Config. NF-e da filial e identifique: qual certificado está configurado? Qual ambiente (teste/produção)? Qual UF?
5. Abra as **Categorias de Participantes** e veja quais categorias estão cadastradas

**Referência**: consulte o manual "Configurações de Filiais" disponível nos documentos internos.
