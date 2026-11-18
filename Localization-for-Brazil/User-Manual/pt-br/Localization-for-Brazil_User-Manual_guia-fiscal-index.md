# Guia Fiscal da Localização

Guia conceitual da Localização Brasileira para Business Central. Voltado para profissionais de fiscal e tributário que precisam entender como a localização funciona — não apenas operar as telas, mas compreender a lógica por trás das configurações, dos cálculos e dos fluxos.

---

## Para quem é este guia

- Profissionais de fiscal/tributário que vão trabalhar com a localização
- Consultores funcionais que precisam entender o "porquê" além do "como"
- Quem vem de outro ERP e precisa aprender Business Central e a localização simultaneamente

## Como usar

Cada página explica um tema seguindo a mesma estrutura:

1. **O que é** — o conceito em linguagem fiscal
2. **Como a localização implementa** — onde fica no BC, como as peças se conectam
3. **Pontos de atenção** — o que costuma dar problema
4. **Para praticar** — exercícios sugeridos no ambiente e referências a manuais

As páginas estão organizadas em uma sequência progressiva. Se você está começando do zero, siga a trilha de 5 semanas abaixo. Se já conhece o BC, pule direto para a semana que fizer sentido.

---

## Trilha de 5 semanas

### Semana 1 — Business Central do zero

Objetivo: aprender a navegar no BC e entender os conceitos fundamentais do sistema.

- [Business Central — Fundamentos](Localization-for-Brazil_User-Manual_guia-fiscal-bc-fundamentos.md)
- [Visão Geral da Localização](Localization-for-Brazil_User-Manual_guia-fiscal-visao-geral-plataforma.md)

**Praticar**: navegar no BC, explorar o Centro de Função, usar o Diga-me (pesquisa), criar e registrar uma ordem de venda simples, verificar os movimentos gerados.

**Checkpoint**: você consegue encontrar qualquer página de configuração usando o Diga-me? Entende o que significa "registrar" um documento e que os movimentos são irreversíveis?

---

### Semana 2 — Configuração fiscal

Objetivo: entender como a localização modela a empresa e a estrutura tributária no BC.

- [Configuração de Empresa](Localization-for-Brazil_User-Manual_guia-fiscal-setup-empresa.md)
- [Jurisdições, Áreas e Contabilizações de Impostos](Localization-for-Brazil_User-Manual_guia-fiscal-jurisdicoes-areas-impostos.md)
- [Configuração Fiscal — CFOP, NCM, CST, Matrizes](Localization-for-Brazil_User-Manual_guia-fiscal-setup-fiscal.md)

**Praticar**: revisar a configuração de empresa e fiscal no ambiente. Dado um item, rastrear como a alíquota de ICMS é determinada.

**Checkpoint**: dado um cenário de venda interestadual (SP para MG, contribuinte), você consegue apontar no BC onde está cada configuração — CFOP, alíquota, CST?

---

### Semana 3 — Fluxo fiscal e documentos eletrônicos

Objetivo: entender o ciclo completo documento fiscal, impostos e NF-e.

- [O Documento Fiscal](Localization-for-Brazil_User-Manual_guia-fiscal-documento-fiscal.md)
- [Fluxo de Vendas](Localization-for-Brazil_User-Manual_guia-fiscal-fluxo-vendas.md)
- [Fluxo de Compras](Localization-for-Brazil_User-Manual_guia-fiscal-fluxo-compras.md)
- [Ciclo de Vida da NF-e](Localization-for-Brazil_User-Manual_guia-fiscal-nfe-ciclo.md)
- [NFS-e e ISS](Localization-for-Brazil_User-Manual_guia-fiscal-nfse.md)

**Praticar**: emitir uma NF-e completa, registrar uma compra com retenção, acompanhar o status da NF-e na SEFAZ.

**Checkpoint**: se alguém diz "o imposto saiu errado na NF-e" — você sabe onde olhar e em que ordem?

---

### Semana 4 — Livros fiscais e SPED

Objetivo: entender como os dados dos documentos fiscais alimentam as obrigações acessórias.

- [Visão Geral](Localization-for-Brazil_User-Manual_guia-fiscal-livros-fiscais-visao.md)
- [EFD-ICMS/IPI](Localization-for-Brazil_User-Manual_guia-fiscal-efd-icms-ipi.md)
- [PIS/COFINS](Localization-for-Brazil_User-Manual_guia-fiscal-pis-cofins.md)
- [EFD-Reinf](Localization-for-Brazil_User-Manual_guia-fiscal-efd-reinf.md)
- [IRPJ/CSLL](Localization-for-Brazil_User-Manual_guia-fiscal-irpj-csll.md)
- [ECD](Localization-for-Brazil_User-Manual_guia-fiscal-ecd.md)
- [ECF](Localization-for-Brazil_User-Manual_guia-fiscal-ecf.md)
- [DCTF](Localization-for-Brazil_User-Manual_guia-fiscal-dctf.md)

**Praticar**: gerar um arquivo EFD no ambiente, abrir e identificar os blocos, conferir registros contra os documentos fiscais de origem.

**Checkpoint**: dado um registro do EFD com valor inconsistente, você consegue rastrear de volta ao documento fiscal que o originou?

---

### Semana 5 — Diagnóstico, reforma tributária e imersão

Objetivo: aplicar tudo em situações reais e desenvolver autonomia.

- [Reforma Tributária — IBS/CBS](Localization-for-Brazil_User-Manual_guia-fiscal-reforma-tributaria.md)
- [Troubleshooting — Diagnóstico de Problemas](Localization-for-Brazil_User-Manual_guia-fiscal-troubleshooting.md)

**Praticar**: resolver 3 a 5 cenários de problema simulados no ambiente. Depois, pegar tickets reais de clientes e conduzir a investigação.

**Checkpoint final**: você consegue pegar um ticket fiscal novo e diagnosticar o problema de forma autônoma?

---

## Páginas do guia

### Fundamentos
- [Business Central — Fundamentos](Localization-for-Brazil_User-Manual_guia-fiscal-bc-fundamentos.md)
- [Visão Geral da Localização](Localization-for-Brazil_User-Manual_guia-fiscal-visao-geral-plataforma.md)

### Configuração
- [Configuração de Empresa](Localization-for-Brazil_User-Manual_guia-fiscal-setup-empresa.md)
- [Jurisdições, Áreas e Contabilizações](Localization-for-Brazil_User-Manual_guia-fiscal-jurisdicoes-areas-impostos.md)
- [Configuração Fiscal — CFOP, NCM, CST](Localization-for-Brazil_User-Manual_guia-fiscal-setup-fiscal.md)

### Fluxo Fiscal
- [O Documento Fiscal](Localization-for-Brazil_User-Manual_guia-fiscal-documento-fiscal.md)
- [Fluxo de Vendas](Localization-for-Brazil_User-Manual_guia-fiscal-fluxo-vendas.md)
- [Fluxo de Compras](Localization-for-Brazil_User-Manual_guia-fiscal-fluxo-compras.md)
- [Ciclo de Vida da NF-e](Localization-for-Brazil_User-Manual_guia-fiscal-nfe-ciclo.md)
- [NFS-e e ISS](Localization-for-Brazil_User-Manual_guia-fiscal-nfse.md)

### Livros Fiscais
- [Visão Geral](Localization-for-Brazil_User-Manual_guia-fiscal-livros-fiscais-visao.md)
- [EFD-ICMS/IPI](Localization-for-Brazil_User-Manual_guia-fiscal-efd-icms-ipi.md)
- [PIS/COFINS](Localization-for-Brazil_User-Manual_guia-fiscal-pis-cofins.md)
- [EFD-Reinf](Localization-for-Brazil_User-Manual_guia-fiscal-efd-reinf.md)
- [IRPJ/CSLL](Localization-for-Brazil_User-Manual_guia-fiscal-irpj-csll.md)
- [ECD](Localization-for-Brazil_User-Manual_guia-fiscal-ecd.md)
- [ECF](Localization-for-Brazil_User-Manual_guia-fiscal-ecf.md)
- [DCTF](Localization-for-Brazil_User-Manual_guia-fiscal-dctf.md)

### Avançado
- [Reforma Tributária — IBS/CBS](Localization-for-Brazil_User-Manual_guia-fiscal-reforma-tributaria.md)
- [Troubleshooting](Localization-for-Brazil_User-Manual_guia-fiscal-troubleshooting.md)
