# Atualizações Abril/2026








## Localization for Brazil — 25.5.0.43 <small>28/04/2026</small>

### Novos Recursos

- **MD-e:** a Fatura Compra Registrada gerada a partir do XML do fornecedor passa a manter o vínculo com a Ordem de Compra reconciliada (campos *Nº Ordem* e *Nº Linha Ordem* propagados para a linha registrada)

### Correções

- **ICMS:** suprimido o cálculo do FCP na linha de ICMS quando outra linha tributária da mesma linha do documento (Substituição Tributária ou Dif. Aliq. ICMS) já carrega o *Valor FCP*; novo campo *Ignorar FCP* registra a supressão para auditoria

## Localization for Brazil - Tax Books — 25.0.3.10 <small>17/04/2026</small>

### Novos Recursos

- **EFD-ICMS/IPI:** adicionadas variáveis de classificação "Outro" no mapeamento de Observações EFD (registros 195/197)

### Melhorias

- **ECF:** adaptações para o Leiaute 12 (AC 2025)
- **EFD-ICMS/IPI:** implementado filtro automático de depósitos por filial na apuração

### Correções

- **EFD-Contribuições:** ajuste no tamanho do campo Nº Documento do registro genérico de ajustes de apurações para 50 caracteres (NT 012/2026)

## Localization for Brazil — 25.0.2.10 <small>17/04/2026</small>

### Melhorias

- **DUIMP:** Aumento do limite de caracteres do campo "Local de Desembaraço" para comportar todos os valores disponíveis na tabela de referência
- **NF-e:** Visualizador XML passa a exibir os grupos de entrega e retirada

### Correções

- **Rateio Dimensões:**
  - Contas contábeis das linhas do rateio agora seguem corretamente as contas definidas nas linhas de origem
  - Grupo Contábil de Produto agora é atribuído corretamente em linhas distintas do rateio

## Localization for Brazil — 25.0.0.7303 <small>10/04/2026</small>

### Outros

- **License Monitor:** integração do monitor de licenças Arquiconsult

## Localization for Brazil — 24.0.7.7272 <small>08/04/2026</small>

### Novos Recursos

- **NF-e:** adicionado suporte a Compra Governamental, com mapeamento das tags `gCompraGov` e `gTribCompraGov`

### Melhorias

- **NF-e:** atualização dos métodos de pagamento suportados

### Correções

- **DIFAL:** corrigida a partição entre UF Destino e UF Origem quando a opção *Alteração de Bases* (TaxBaseChange) soma o IPI ao valor da base de cálculo

## Localization for Brazil - Tax Books — 24.0.3.7263 <small>07/04/2026</small>

### Novos Recursos

- **ECF:** adição do registro Y720

### Melhorias

- **Apuração IRPJ/CSLL:**
  - Inclusão de receitas financeiras na base de cálculo do Lucro Presumido
  - Acréscimo de presunção para Regime de Competência e suporte a múltiplas alíquotas

### Correções

- **EFD Contribuições:** correção de base duplicada de PIS/COFINS quando há múltiplas jurisdições (COFINS Majorado)

## Changelog

| Hash | Data | Descrição | Aplicativo | Versão |
|---|---|---|---|---|
| 2026cc8 | 07/04 | Apuração IRPJ/CSLL: receitas financeiras na base de cálculo do Lucro Presumido | Tax Books | 24.0.3.7263 |
| 15d1be7 | 07/04 | EFD Contribuições: correção de base duplicada de PIS/COFINS quando há múltiplas jurisdições (COFINS Majorado) | Tax Books | 24.0.3.7263 |
| fd8181a | 06/04 | Apuração IRPJ/CSLL: acréscimo de presunção para Regime de Competência e múltiplas alíquotas | Tax Books | 24.0.3.7263 |
| e78103d | 02/04 | ECF:  traduções do registro Y720 | Tax Books | 24.0.3.7263 |
| 0c6477d | 25/03 | ECF: registro Y720 | Tax Books | 24.0.3.7263 |
| 76f2feb | 07/04 | DIFAL: correção de partição UF Destino/Origem quando TaxBaseChange (Alteração de Bases) soma IPI ao valor | Localization | 24.0.7.7272 |
| ea1d443 | 02/04 | NF-e: atualização de métodos de pagamento | Localization | 24.0.7.7272 |
| 362474b | 31/03 | Merged PR 436: NF-e: suporte a Compra Governamental (tags gCompraGov e gTribCompraGov) | Localization | 24.0.7.7272 |
| db55c50 | 10/04 | Merged PR 412: License Monitor: integração no pipeline de build | Localization | 25.0.0.7303 |
| 6249b75 | 17/04 | Rateio Dimensões: Correção de Contas das linhas do Rateio não seguindo as das linhas (#9) | Localization | 25.0.2.10 |
| b5d4442 | 16/04 | Rateio Dimensões: Correção de Gr Contábil de Produto não sendo atribuído em linhas diferentes (#8) | Localization | 25.0.2.10 |
| ddb662c | 15/04 | NF-e: visualizador XML exibe grupos entrega e retirada | Localization | 25.0.2.10 |
| 0a45012 | 13/04 | DUIMP: Aumento de caracteres do Local de Desembaraço para se adequar aos valores disponíveis na tabela de referência #BRL-SUP15-43 | Localization | 25.0.2.10 |
| 09fce42 | 17/04 | ECF: adaptações para o Leiaute 12 (AC 2025) | Tax Books | 25.0.3.10 |
| 17a3528 | 16/04 | EFD-Contribuições: ajuste no tamanho do campo Nº Documento genérico de ajustes de apurações para [50] (NT 012 2026) | Tax Books | 25.0.3.10 |
| 170c086 | 16/04 | EFD-ICMS/IPI: filtro automático de depósitos por filial na apuração (#7) | Tax Books | 25.0.3.10 |
| 0b3f748 | 15/04 | EFD-ICMS/IPI: adição de variáveis de classificação Outro no mapeamento de Observaçoes EFD (regs. 195/197) | Tax Books | 25.0.3.10 |
| 45aa6f8 | 27/04 | MD-e: criação de vínculo entre a fatura registrada e a ordem reconciliada | Localization | 25.5.0.43 |
| 1c90488 | 27/04 | ICMS: remover FCP da linha quando ST ou Dif. Aliq. ICMS já o carrega (#15) | Localization | 25.5.0.43 |
