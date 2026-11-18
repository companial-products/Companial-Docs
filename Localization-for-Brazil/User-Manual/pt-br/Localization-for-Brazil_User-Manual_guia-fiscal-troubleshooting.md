# Troubleshooting — Diagnóstico de Problemas

Este capítulo reúne os problemas mais comuns que surgem na operação fiscal e como diagnosticá-los. O objetivo é desenvolver um método sistemático de investigação — saber **onde olhar** e **em que ordem**.

---

## Método geral de investigação

Para qualquer problema fiscal, siga esta sequência:

```
1. IDENTIFICAR o sintoma
   "Imposto errado", "NF-e rejeitada", "EFD com valor diferente"

2. LOCALIZAR o documento
   Encontrar o documento fiscal no BC
   (Documentos Fiscais, filtrar por número, período, participante)

3. CONFERIR os dados do documento
   CFOP, NCM, CST, Área Impostos, valores de impostos

4. RASTREAR a configuração
   Se o dado estiver errado no documento, rastrear de onde veio:
   - Área Impostos → Contabilização Imposto → Contas Impostos
   - Matriz Impostos → Alíquotas ICMS por UF → Exceção Imposto

5. VERIFICAR a origem
   O documento do BC (fatura registrada) está correto?
   O item está configurado corretamente?
   O cliente/fornecedor tem a categoria certa?

6. CORRIGIR
   Corrigir na configuração (se é padrão) ou no documento (se é pontual)
```

---

## Problemas com impostos

### Imposto zerado ou faltando

| Causa | Onde verificar |
|-------|---------------|
| Área Impostos não inclui a jurisdição | Linhas da Área Impostos usada na linha |
| Contabilização Imposto com todos os flags vazios | Contabilização Imposto da jurisdição |
| Exceção fiscal zerando a alíquota | Exceções Imposto aplicáveis ao item |

**Sequência**: Linha do documento → Área Impostos → Jurisdição → Exceção

### Alíquota de ICMS incorreta

| Causa | Onde verificar |
|-------|---------------|
| Alíquotas ICMS por UF desatualizada | Alíquotas ICMS por UF (UF origem × UF destino) |
| Exceção fiscal com alíquota diferente | Exceções Imposto |
| UF do participante errada | Cartão do cliente/fornecedor (Cód. UF) |

### Retenção não calculada

| Causa | Onde verificar |
|-------|---------------|
| Área sem configuração de retenção | Área de Impostos para a operação de serviço |
| Contabilização Imposto de retenção sem flags de Pagamento | Contabilização Imposto da jurisdição (IRRF, PCC, etc.) |
| Valor abaixo do mínimo para retenção | Regra de R$ 10,00 mínimo por DARF |

### Contabilização incorreta

| Causa | Onde verificar |
|-------|---------------|
| Contabilização Imposto com flags errados | Contabilização Imposto: Custo/Preço Item, Financeiro, Imposto a Pagar, Despesa, Crédito Imposto |
| Contas Impostos com conta errada | Contas Impostos por Classificação Imposto |
| Crédito fiscal em operação sem direito | Crédito Imposto = Sim em operação de uso/consumo |

---

## Problemas com NF-e

### NF-e rejeitada pela SEFAZ

As rejeições mais comuns:

| Código | Motivo | Solução |
|--------|--------|---------|
| **203** | Emitente não habilitado | Verificar IE da filial |
| **225** | Falha no Schema XML | Campo obrigatório vazio ou formato inválido |
| **204** | Duplicidade de NF-e | A NF-e provavelmente foi autorizada em uma tentativa anterior. Não exclua o documento nem reutilize o número: registre sem **NF-e Automática** e use **Consultar Status** no **Documento Fiscal Registrado** |
| **539** | Duplicidade de NF-e com diferença na chave de acesso | O número já foi usado por uma NF-e autorizada com outra chave. Consulte na SEFAZ a chave informada na mensagem: pode ser este mesmo documento, autorizado em uma tentativa anterior com outra data ou valor |
| **611** | cEAN inválido | Conferir o código de barras (GTIN) do item |
| **778** | NCM inexistente | NCM do item não existe na tabela da SEFAZ |

**Onde ver o erro**: no Documento Fiscal, campos **Resultado** e **Código Status**.

O que fazer depois da rejeição (com ou sem **NF-e Automática**): ver [Rejeição](Localization-for-Brazil_User-Manual_guia-fiscal-nfe-ciclo.md#rejeição) em Ciclo de Vida da NF-e.

### NF-e denegada

Os códigos **110**, **301**, **302** e **303** são de **uso denegado** (irregularidade fiscal do emitente ou do destinatário), não de rejeição. A NF-e não pode ser corrigida, retransmitida nem inutilizada. Verifique a situação cadastral da empresa ou do cliente e cancele a fatura no sistema: ver [Denegação](Localization-for-Brazil_User-Manual_guia-fiscal-nfe-ciclo.md#denegação) em Ciclo de Vida da NF-e.

### NF-e sem resposta da SEFAZ

Se o registro terminou com o aviso *"Estão ocorrendo atrasos nas respostas da SEFAZ e não foi possível obter o status do envio do documento"*, abra o **Documento Fiscal Registrado** e use **NF-e** → **Consultar Status**. Ver [NF-e: novo tratamento de demoras no processamento da SEFAZ](Localization-for-Brazil_User-Manual_NFe-novo-tratamento-de-demoras-SEFAZ.md).

### NF-e não transmitida

| Causa | Onde verificar |
|-------|---------------|
| Transmissão automática desligada | Config. NF-e da filial: campo **NFe Automática**; no documento, campo **NF-e Automática** |
| Certificado digital expirado | Config. NF-e: campo **ID Certificado** → Certificados Digitais |
| Ambiente errado (teste/produção) | Config. NF-e: campo **Ambiente** |
| Serviço SEFAZ fora do ar | Verificar status SEFAZ do estado; se for o caso, usar a contingência: ver [Contingência](Localization-for-Brazil_User-Manual_guia-fiscal-nfe-ciclo.md#contingência) em Ciclo de Vida da NF-e |

### NFS-e não transmitida

| Causa | Onde verificar |
|-------|---------------|
| Web service municipal não configurado | Config. NF-e: seção NFS-e (Tipo de WS, Endereço de WS NFS) |
| Código de Cidade sem mapeamento | Tabela de Municípios: Códigos de Cidade |
| Credenciais inválidas | Config. NF-e: Nome Usuário WS / Senha WS |
| Código de serviço errado | Código do serviço no documento vs. tabela do município |

---

## Problemas com EFD/SPED

### Registro com valor inconsistente

**Método de rastreamento**:

1. Identifique o registro no arquivo EFD (ex: C170, linha X)
2. O registro C170 contém o número do documento (campo 02 do C100 pai)
3. No BC, localize o Documento Fiscal pelo número
4. Abra a linha correspondente e confira os valores
5. Se o valor está errado no documento → problema é na configuração fiscal
6. Se o valor está correto no documento mas errado no EFD → problema é na geração do arquivo

### Documento fiscal faltando no EFD

| Causa | Onde verificar |
|-------|---------------|
| Documento fora do período | Data Registro do documento |
| Documento não registrado | Status do documento fiscal |
| Filtro de filial | Apuração configurada para outra filial |
| Documento cancelado | NF-e com status 101 pode não aparecer (dependendo da configuração) |

### Apuração com saldo incorreto

| Causa | Onde verificar |
|-------|---------------|
| Documento fiscal com imposto errado | Conferir documentos do período um a um |
| Ajuste manual faltando | Bloco E: registros E111/E113 (ajustes) |
| Crédito indevido | Documento de compra com Crédito Imposto ativo sem direito |
| Período não reprocessado | Após correção, reprocessar a Apuração Impostos |

---

## Problemas com DCTF e obrigações

### Valor da DCTF diferente da apuração

| Causa | Onde verificar |
|-------|---------------|
| DARF não vinculado | Linhas de pagamento (R11) da DCTF |
| Código de receita errado | Código da jurisdição vs. código na DCTF |
| Período diferente | A DCTF é mensal — conferir se o DARF é do mesmo mês |

---

## Checklist rápido — o que verificar primeiro

Para qualquer problema fiscal, estes são os 5 pontos que resolvem 80% dos casos:

1. **CFOP** — está correto para a operação? (Interno vs. interestadual, venda vs. compra)
2. **Área Impostos** — inclui todos os impostos necessários?
3. **Contabilização Imposto** — os flags estão corretos para o cenário? (Crédito vs. custo vs. despesa)
4. **Categoria do participante** — o cliente/fornecedor está classificado corretamente?
5. **Certificado digital** — está válido e no ambiente correto?

---

## Para praticar

1. **Cenário 1**: Localize um documento fiscal com ICMS calculado. Rastreie o caminho completo: Área Impostos → Jurisdição → Matriz → Alíquotas ICMS por UF → Contabilização Imposto → Contas
2. **Cenário 2**: Encontre uma NF-e rejeitada (ou simule uma) e leia o motivo da rejeição no campo **Resultado**. Identifique qual dado corrigir.
3. **Cenário 3**: Abra o EFD-ICMS/IPI, localize um registro C100 e rastreie de volta ao documento fiscal no BC. Os valores conferem?
4. **Cenário 4**: Verifique se todos os fornecedores com serviço têm categoria de participante e configuração de retenção.
5. **Cenário 5**: Confira se o certificado digital da filial principal está válido e no ambiente correto.
