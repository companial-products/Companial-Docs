# Reforma Tributária — IBS/CBS

A Reforma Tributária do Consumo (EC 132/2023, LC 214/2025) é a maior mudança no sistema tributário brasileiro em décadas. Ela substitui cinco tributos por três novos, e a localização já implementa o cálculo e a geração dos novos campos na NF-e e NFS-e.

---

## O que muda

### Tributos que serão substituídos

| Tributo atual | Esfera | Substituído por |
|---------------|--------|----------------|
| **ICMS** | Estadual | **IBS** (parte estadual) |
| **ISS** | Municipal | **IBS** (parte municipal) |
| **PIS** | Federal | **CBS** |
| **COFINS** | Federal | **CBS** |
| **IPI** | Federal | **IS** (Imposto Seletivo, apenas para produtos específicos) |

### Novos tributos

| Tributo | Nome | Esfera |
|---------|------|--------|
| **IBS** | Imposto sobre Bens e Serviços | Estadual + Municipal (subnacional) |
| **CBS** | Contribuição sobre Bens e Serviços | Federal |
| **IS** | Imposto Seletivo | Federal (apenas para bens nocivos à saúde/meio ambiente) |

### Período de transição

A transição é gradual (2026–2033):
- **2026**: CBS começa com alíquota-teste; IBS introduzido com alíquota mínima
- **2027-2028**: Alíquotas progressivamente aumentam
- **2029-2032**: Redução gradual de ICMS, ISS, PIS, COFINS
- **2033**: Extinção completa dos tributos antigos

Durante a transição, **os dois sistemas coexistem** — a empresa calcula tanto os tributos atuais (ICMS, PIS, COFINS) quanto os novos (IBS, CBS).

---

## Como a localização implementa

### Ativação

A reforma tributária na localização é controlada pelo campo **Status Reforma Tributária** (Tax Reform Status) nas Configurações Fiscais:

| Status | Comportamento |
|--------|--------------|
| **Agendada** (padrão) | Ativa automaticamente a partir de 01/05/2026 |
| **Habilitada** | Sempre ativo (para testes) |
| **Desabilitada** | Desativado |

### Novas Classificações de Imposto

A localização adiciona quatro novos valores de Classificação Imposto:

| Classificação Imposto | Tributo |
|-------------------|---------|
| **IBSUF** | IBS Estadual |
| **IBSMun** | IBS Municipal |
| **CBS** | CBS Federal |
| **IS** | Imposto Seletivo |

### Classificação Tributária (cClassTrib)

Conceito novo da reforma: cada operação recebe um código de classificação tributária de 6 dígitos (ex: `000001` = tributação integral) que determina:

- Se a operação é tributada, reduzida, isenta ou suspensa
- Qual percentual de redução se aplica
- Se há diferimento

### Cálculo

O fluxo de cálculo segue a mesma lógica dos tributos existentes:

1. A linha do documento fiscal recebe o código **cClassTrib**
2. O sistema busca as **alíquotas** de cada jurisdição (CBS, IBS UF, IBS Municipal)
3. Aplica **reduções** e **diferimentos** conforme a classificação
4. Grava os valores nos campos IBS/CBS das linhas de imposto do documento
5. Na geração do XML da NF-e/NFS-e, inclui os **grupos UB** (IS e IBSCBS) com todos os subgrupos

### XML da NF-e e NFS-e

A reforma adiciona novos grupos ao XML da NF-e e NFS-e por item:

```
<imposto>
  <IS>                    ← Imposto Seletivo (quando aplicável)
  <IBSCBS>                ← IBS e CBS
    <CST>
    <cClassTrib>
    <gIBSCBS>
      <gIBSUF>            ← IBS estadual (alíquota, redução, diferimento)
      <gIBSMun>           ← IBS municipal (alíquota, redução)
      <gCBS>              ← CBS (alíquota, redução, diferimento)
      <gEstornoCred>      ← Estorno de crédito (quando aplicável)
    </gIBSCBS>
  </IBSCBS>
</imposto>
```

---

## Conceitos importantes da reforma

### Não cumulatividade plena

Diferente do sistema atual (onde créditos de PIS/COFINS têm restrições), o IBS/CBS tem **não cumulatividade plena**: praticamente toda aquisição tributada gera crédito. Exceções são poucas e explícitas.

### Alíquota no destino

O IBS/CBS é cobrado no **destino** (onde o bem é consumido ou o serviço é prestado), não na origem. Isso é uma mudança fundamental em relação ao ICMS (que tem parte na origem).

### Split Payment

Mecanismo futuro onde o pagamento dos tributos será feito diretamente pelo sistema financeiro no momento da transação, sem que a empresa precise recolher depois. Ainda não regulamentado pelo governo nem implementado na localização.

### Cashback

Devolução de parte do IBS/CBS para consumidores de baixa renda. Previsto na legislação, ainda não regulamentado pelo governo nem implementado na localização.

---

## Impacto na operação diária

### O que muda para quem configura

- **Novas jurisdições nas Áreas de Impostos** — as Áreas de Impostos existentes precisam ser atualizadas com as jurisdições de IBS UF, IBS Municipal e CBS
- **Classificações Tributárias IBS/CBS** — tabela nova a manter
- **Alíquotas por período** — como as alíquotas mudam durante a transição, a tabela de alíquotas tem vigência por data
- **Contabilizações de Imposto novas** — configuração de contabilização para IBS e CBS (crédito, débito, etc.)

### O que muda nos documentos

- NF-e e NFS-e passam a ter os grupos UB no XML
- Documentos fiscais terão campos de IBS/CBS além de ICMS, PIS, COFINS

### O que muda na escrituração

- EFD-ICMS/IPI e EFD-Contribuições continuam durante a transição
- Novas obrigações para IBS/CBS estão sendo definidas pelo governo
- A apuração assistida (governo calcula e a empresa confere) é prevista para IBS/CBS

---

## Pontos de atenção

- **Transição simultânea** — durante 2026-2032, os dois sistemas coexistem. Cuidado redobrado nas configurações.
- **Legislação em evolução** — NTs e regulamentações estão sendo publicadas continuamente. A localização é atualizada a cada nova NT.
- **Classificação tributária** — a tabela de cClassTrib é extensa e em construção. Manter atualizada é fundamental.
- **Alíquotas provisórias** — as alíquotas dos primeiros anos são de teste/transição. Elas mudarão.

---

## Para praticar

1. Verifique o **Status Reforma Tributária** nas Configurações Fiscais — está ativo, programado ou desativado?
2. Abra a tabela de **Classificações Tributárias IBS/CBS** e explore os códigos cadastrados
3. Abra a tabela de **Alíquotas IBS/CBS** e observe as alíquotas por tipo e período
4. Se a reforma estiver ativa, crie um documento fiscal e observe os campos IBS/CBS calculados
5. Leia o manual [Reforma Tributária - Configurações](Localization-for-Brazil_User-Manual_Reforma-Tributaria-Configuracoes.md) para contexto completo

**Referência**: seção de Reforma Tributária do portal.
