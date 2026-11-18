# Visão Geral

O app de **Livros Fiscais** (Tax Books) é o segundo app da Localização. Enquanto o app da Localização cuida dos documentos fiscais e do cálculo de impostos, o app de Livros Fiscais é responsável por **escriturar** esses dados — transformando os documentos fiscais em livros fiscais, apurações e obrigações acessórias exigidas pelo governo.

---

## O que é

Livros fiscais são registros obrigatórios que consolidam todas as operações fiscais de uma empresa em um período. Historicamente eram físicos (livros de entrada, saída, apuração); hoje são digitais, transmitidos eletronicamente via SPED (Sistema Público de Escrituração Digital).

O app de Livros Fiscais gera todos esses registros a partir dos documentos fiscais que o app da Localização criou.

---

## Como funciona

### O conceito central: Apuração Impostos (Tax Settlement)

Tudo no app de Livros Fiscais gira em torno da **Apuração Impostos** (Tax Settlement) — um período de apuração. Você cria uma apuração informando:

- **Período**: data inicial e final (geralmente mensal, trimestral ou anual)
- **Filial**: qual estabelecimento

A apuração então processa os documentos fiscais do período e gera:

- **Valores da Apuração** — valores consolidados por imposto (débitos, créditos, saldos)
- **Documentos da Apuração** — documentos vinculados a cada valor

### Do documento fiscal ao livro

```
Documento Fiscal (app da Localização)
    │
    │  O app de Livros Fiscais lê os documentos do período
    │
    ▼
Apuração Impostos
    │
    ├── EFD-ICMS/IPI → arquivo texto SPED Fiscal
    ├── EFD-Contribuições → arquivo texto SPED Contribuições
    ├── EFD-Reinf → XML transmitido à Receita Federal
    ├── ECD → arquivo texto SPED Contábil
    ├── ECF → arquivo texto SPED Fiscal Contábil
    ├── MIT → débitos federais para a DCTFWeb (a DCTF só nos períodos anteriores)
    ├── Apuração IRPJ/CSLL → demonstrativos e DARFs
    └── Declarações de Impostos (Tax Statements) → PIS, COFINS, ISS, IRPJ, CSLL
```

---

## Módulos do app de Livros Fiscais

### Escrituração eletrônica (arquivos SPED)

| Módulo | O que gera | Periodicidade | Destino |
|--------|-----------|---------------|---------|
| **EFD-ICMS/IPI** | SPED Fiscal — escrituração de ICMS e IPI | Mensal | SPED (Receita Federal + SEFAZ) |
| **EFD-Contribuições** | SPED Fiscal — escrituração de PIS e COFINS | Mensal | SPED (Receita Federal) |
| **EFD-Reinf** | Retenções previdenciárias (INSS) e IRRF/PCC | Mensal | Receita Federal (XML) |
| **ECD** | Escrituração Contábil Digital (diário, balancete) | Anual | SPED |
| **ECF** | Escrituração Contábil Fiscal (IRPJ/CSLL) | Anual | SPED |

### Declarações e apurações

| Módulo | O que faz | Periodicidade |
|--------|----------|---------------|
| **DCTF** | Declaração de Débitos e Créditos Tributários Federais | Mensal |
| **IRPJ/CSLL** | Apuração de Imposto de Renda e CSLL | Trimestral/Anual |
| **PIS/COFINS** | Apuração de PIS e COFINS (cumulativo e não cumulativo) | Mensal |
| **ISS** | Apuração de ISS com conciliação de NFS-e | Mensal |
| **DIRF** | Declaração de IR Retido na Fonte — sugestão automática, exportação de arquivo | Extinta |
| **GIA** | Guia de Informação e Apuração do ICMS (SP e RJ) | Mensal |
| **Declarações de Impostos** (Tax Statements) | Demonstrativos fiscais genéricos | Conforme necessidade |
| **Guias de Impostos** (Tax Bills) | Guias de pagamento (DARF, GNRE), provisão DCTFWeb | Conforme necessidade |

---

## Fluxo de trabalho típico (mensal)

Um ciclo mensal completo:

```
1. CONFERÊNCIA
   Verificar se todos os documentos fiscais do mês estão registrados
   e com impostos corretos

2. APURAÇÕES
   Criar Apurações de Impostos para o mês:
   - Apuração de ICMS/IPI
   - Apuração de PIS/COFINS
   - Apuração de ISS
   Revisar débitos, créditos e ajustes

3. GERAÇÃO DE ARQUIVOS
   - Gerar EFD-ICMS/IPI
   - Gerar EFD-Contribuições
   - Transmitir eventos EFD-Reinf
   - Gerar o arquivo MIT, para a DCTFWeb

4. GUIAS DE PAGAMENTO
   Gerar DARFs para os impostos apurados

5. VALIDAÇÃO
   Importar os arquivos no PVA (Programa Validador) da Receita Federal
   Corrigir inconsistências se houver
```

---

## Pontos de atenção

- **Documento fiscal errado = livro fiscal errado** — o app de Livros Fiscais apenas escritura o que o app da Localização calculou. Se o imposto está errado no documento, estará errado na escrituração.
- **Período fechado vs. aberto** — algumas apurações não permitem reprocessamento após fechamento. Verifique antes de alterar documentos.
- **Validação no PVA** — o arquivo gerado pelo app de Livros Fiscais deve ser importado e validado no PVA da Receita Federal antes da transmissão. É comum haver ajustes.
- **Versão de layout** — cada ano pode trazer uma nova versão do layout SPED. O app de Livros Fiscais suporta múltiplas versões e seleciona automaticamente pela data do período.

---

## Para praticar

1. Use o Diga-me para encontrar "Apurações" — explore a lista de apurações existentes
2. Abra uma apuração e veja os valores consolidados — débitos, créditos, saldo por imposto
3. Identifique quais módulos estão disponíveis no ambiente: EFD-ICMS/IPI, EFD-Reinf, DCTF, etc.
4. Se possível, gere um arquivo EFD de teste e abra em um editor de texto — observe a estrutura de blocos e registros

**Referência**: consulte o manual "Fiscal Apuração ICMS-IPI" e os demais manuais de obrigações acessórias disponíveis na seção de [Manuais](Localization-for-Brazil_User-Manual_Contents.md).
