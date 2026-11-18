# Visão Geral da Localização

A localização brasileira é um conjunto de apps que adiciona ao Business Central tudo o que é necessário para operar fiscalmente no Brasil: documentos fiscais, cálculo de impostos, emissão de NF-e/NFS-e, escrituração fiscal (SPED), pagamentos eletrônicos e muito mais.

---

## Os 4 apps

A localização é composta por apps instalados como extensões do BC. Cada um tem uma responsabilidade clara:

### 1. Localization for Brazil

O **app da Localização** — o app base. Contém toda a infraestrutura fiscal:

- **Documentos fiscais** — criação, cálculo de impostos, emissão de NF-e e NFS-e
- **Configuração fiscal** — CFOP, NCM, CST, matrizes de impostos, alíquotas, IBPT
- **Configuração de empresa** — filiais, inscrições, certificados digitais
- **Integração com vendas e compras** — ao registrar uma venda ou compra no BC, o documento fiscal é criado automaticamente
- **Retenções na fonte** — IRRF, INSS, PIS, COFINS, CSLL retidos

Todos os outros apps dependem deste. Sem ele, nenhum dos outros funciona.

### 2. Tax Books

O app de **livros fiscais e obrigações acessórias**. Consome os dados do app da Localização para gerar:

- **EFD-ICMS/IPI** (SPED Fiscal) — escrituração de ICMS, IPI, ST
- **EFD-Contribuições** — PIS e COFINS
- **EFD-Reinf** — retenções previdenciárias
- **ECD** — escrituração contábil digital
- **ECF** — escrituração contábil fiscal (IRPJ/CSLL)
- **DCTF** — declaração de débitos tributários federais
- **Apurações** — ICMS, IPI, PIS/COFINS, ISS, IRPJ/CSLL
- **Guias de pagamento** — DARF, DAS, GNRE



---


---

## O que cada app NÃO faz

É igualmente importante saber os limites:

| O que | Quem faz | Quem **não** faz |
|-------|----------|-----------------|
| Calcular ICMS, PIS, COFINS | App da Localização | Livros Fiscais apenas **escritura** o que a Localização calculou |
| Gerar NF-e / NFS-e | App da Localização | Livros Fiscais, Bancário e Pacote Avançado não emitem NF-e |
| Apurar ICMS, gerar EFD | Livros Fiscais | O app da Localização não sabe apurar — ele só grava os dados |

---

## Módulos que o BC padrão tem mas que NÃO possuem integração fiscal

Os seguintes módulos do BC **não geram documentos fiscais** automaticamente. Se um cliente usa algum deles e precisa de NF-e, é necessário um desenvolvimento específico:

| Módulo | Motivo |
|--------|--------|
| **Ordens de Serviço** (Service Management) | Empresas de serviço usam Vendas + NFS-e |
| **Ordens de Produção** (Manufacturing) | Operação interna — sem obrigação fiscal direta |
| **Projetos** (Jobs) | Faturamento é feito via Vendas |
| **Montagem** (Assembly Management) | Operação interna |

Isso é uma limitação conhecida, não um bug. A integração fiscal foi construída para os módulos de Vendas, Compras, Estoque (Transferências) e Financeiro.

---

## Integrações externas

A localização se comunica com sistemas externos:

| Sistema | Para quê | Via |
|---------|----------|-----|
| **SEFAZ** (Secretaria da Fazenda) | Autorizar, cancelar e consultar NF-e | Web Services XML |
| **Prefeituras** | Transmitir NFS-e/RPS | Web Services (varia por município) |
| **IBPT** | Carga tributária estimada (Lei da Transparência) | Importação de tabela |

---

## Para praticar

1. No BC, use o Diga-me para encontrar "Configurações Fiscais" — essa é a porta de entrada do app da Localização
2. Abra a lista de "Filiais" (**Informação Filial** / Branch Information) — cada filial é um estabelecimento com CNPJ
3. Procure "Documentos Fiscais" — essa é a lista central onde todos os documentos ficam
4. Se o app de Livros Fiscais estiver instalado, procure "Apurações" ou "EFD" para ver as telas de escrituração
