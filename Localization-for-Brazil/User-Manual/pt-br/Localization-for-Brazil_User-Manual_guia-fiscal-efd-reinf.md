# EFD-Reinf

O EFD-Reinf é a obrigação acessória que informa à Receita Federal as retenções de contribuições previdenciárias (INSS) e, desde 2023, as retenções de IR, PIS, COFINS e CSLL sobre pagamentos (substituindo a DIRF).

---

## O que é

O EFD-Reinf complementa o eSocial: enquanto o eSocial trata da folha de pagamento (vínculos trabalhistas), o Reinf trata das **retenções em serviços e pagamentos** a terceiros.

### Duas séries de eventos

| Série | O que informa | Eventos |
|-------|-------------|---------|
| **R-2000** | Retenções de INSS e CPRB | R-2010, R-2020, R-2050, R-2055, R-2060 |
| **R-4000** | Retenções de IRRF, PIS, COFINS, CSLL (ex-DIRF) | R-4010, R-4020 |

### Formato

Diferente dos outros SPEDs (arquivo texto), o Reinf usa **XML**. Cada evento é um XML, e os eventos são transmitidos ao web service da Receita Federal em lotes de até 50 eventos.

---

## Eventos principais

### Série R-2000 (INSS e CPRB)

| Evento | Nome | O que informa |
|--------|------|-------------|
| **R-2010** | Serviços Tomados | INSS retido sobre serviços que a empresa **contratou** |
| **R-2020** | Serviços Prestados | INSS retido sobre serviços que a empresa **prestou** |
| **R-2060** | CPRB | Contribuição previdenciária sobre a receita bruta (desoneração da folha) |
| **R-2050/R-2055** | Produção Rural | Vendas e aquisições de produção rural |
| **R-2099** | Fechamento | Fecha o período da série R-2000 |

### Série R-4000 (DIRF)

| Evento | Nome | O que informa |
|--------|------|-------------|
| **R-4010** | Pagamentos a Pessoa Física | IRRF sobre pagamentos a pessoas físicas |
| **R-4020** | Pagamentos a Pessoa Jurídica | IRRF, PIS, COFINS, CSLL retidos sobre pagamentos a empresas |
| **R-4099** | Fechamento | Fecha o período da série R-4000 |

### Eventos de controle

| Evento | Nome | O que faz |
|--------|------|----------|
| **R-1000** | Informações do Contribuinte | Cadastro/atualização dos dados da empresa |
| **R-1070** | Processos | Processos administrativos/judiciais que suspendem tributos |
| **R-9000** | Exclusão | Exclui um evento enviado anteriormente |

---

## Como a localização implementa

### Fluxo mensal

```
1. SUGESTÃO AUTOMÁTICA
   O sistema lê os impostos retidos dos documentos fiscais do período
   e gera automaticamente os eventos:
   - R-2010/R-2020 a partir de retenções de INSS
   - R-4010/R-4020 a partir de retenções de IRRF/PCC

2. REVISÃO
   O usuário revisa os eventos gerados e ajusta se necessário

3. TRANSMISSÃO
   Os eventos são empacotados em XML e transmitidos à Receita Federal
   em lotes de até 50 eventos (via serviço intermediário no Azure)

4. RETORNO
   O sistema consulta o resultado do lote: aceito, rejeitado,
   em processamento ou não consta no lote

5. FECHAMENTO
   Após todos os eventos do mês transmitidos com sucesso:
   - R-2099 fecha a série R-2000
   - R-4099 fecha a série R-4000
```

### Status dos eventos

| Status | Significado |
|--------|------------|
| *(vazio)* | Não transmitido |
| **Sucesso** (Success) | Aceito pela Receita Federal |
| **Erro** (Error) | Rejeitado — verificar mensagem de erro |
| **Processando** (Processing) | Enviado, aguardando processamento |
| **Excluído** (Deleted) | Excluído via R-9000 |

O envio em lotes e os resultados da ação **Consultar Lote** estão detalhados no manual [EFD-Reinf: Envio de Eventos em Lotes e Resultado do Consultar Lote](Localization-for-Brazil_User-Manual_EFD-Reinf-Envio-Lotes-Consultar-Lote.md).

---

## Pontos de atenção

- **R-1000 obrigatório** — o evento de cadastro do contribuinte deve ser enviado antes de qualquer outro evento. Se não foi enviado, nada funciona.
- **Configuração do fornecedor** — o código de natureza de rendimento deve estar configurado corretamente para cada fornecedor (Configurações Reinf do Fornecedor). Sem isso, os eventos R-4010/R-4020 não são gerados.
- **Fechamento em ordem** — não é possível fechar R-4099 se houver eventos R-4010/R-4020 pendentes com erro.
- **Reabertura** — se for necessário corrigir um período já fechado, é preciso reabrir (R-2098 ou R-4099 com reabertura) antes de enviar correções.

---

## Para praticar

1. Localize a configuração do Reinf no BC (Diga-me > "Configurações Reinf")
2. Abra a página de períodos do Reinf (Diga-me > "Reinf - Períodos") e observe os status das séries R-2000 e R-4000
3. Execute a sugestão automática de eventos para um mês
4. Revise os eventos gerados — verifique se os valores de retenção conferem com os documentos fiscais
5. Se possível, transmita um evento em ambiente de teste e acompanhe o retorno

**Referência**: consulte o manual "Fiscal REINF" disponível na seção de [Manuais](Localization-for-Brazil_User-Manual_Contents.md).
