# NFS-e e ISS

A NFS-e (Nota Fiscal de Serviços Eletrônica) é o documento fiscal para prestação de serviços. Diferente da NF-e, que é padronizada nacionalmente via SEFAZ, a NFS-e é **municipal** — cada prefeitura pode ter seu próprio sistema, layout e web service.

---

## O que é

### ISS — Imposto Sobre Serviços

O ISS é um imposto **municipal** que incide sobre a prestação de serviços. Suas características:

- **Competência**: do município onde o serviço é prestado (com exceções)
- **Alíquota**: definida por cada município (geralmente entre 2% e 5%)
- **Base de cálculo**: preço do serviço
- **Lista de serviços**: definida por lei complementar federal (LC 116/2003), cada município adota e regulamenta

### NFS-e

A NFS-e é o documento que formaliza a prestação de serviço para fins fiscais. O fluxo:

```
Empresa presta serviço → Emite RPS (Recibo Provisório de Serviços)
    → Transmite RPS ao sistema da prefeitura → Prefeitura retorna NFS-e
        → NFS-e com número definitivo
```

O **RPS** é o documento provisório gerado pela empresa. Ele é convertido em NFS-e pela prefeitura após a transmissão.

---

## Como a localização implementa

### Fluxo de emissão

```
1. Venda de serviço registrada no BC
   → Documento Fiscal criado (tipo NFS-e)
   → ISS calculado

2. Geração do RPS
   → Dados do documento fiscal formatados no layout do município
   → Pode ser individual ou em lote (batch)

3. Transmissão
   → Web Service da prefeitura recebe o RPS
   → Retorna: número definitivo da NFS-e, código de verificação
   → Ou retorna erro (dados inconsistentes, serviço não cadastrado, etc.)

4. Atualização
   → Documento fiscal atualizado com número e dados da NFS-e
```

### Diversidade municipal

Este é o maior desafio da NFS-e: **não existe um padrão único**. A localização lida com isso através de:

- **Configuração por filial**: cada filial pode ter seu próprio web service, credenciais e tipo de integração
- **Serviço intermediário no Azure**: assim como na NF-e, a localização envia os dados em um formato único ao serviço no Azure, que se comunica com o web service de cada prefeitura (~40 municípios suportados)
- **Tipos de web service**: SOAP, REST, arquivo TXT — varia por município
- **Códigos de Cidade**: mapeamento do código IBGE para o código específico do sistema municipal

### Transmissão em lote (Batch)

Para empresas com alto volume de NFS-e, a localização suporta transmissão em lote:
- RPS são agrupados em lotes
- O lote é transmitido de uma vez
- O retorno pode ser síncrono (resultado imediato) ou assíncrono (consultar depois)

### Config. NFS-e (por filial)

Na Config. NF-e da filial, existe uma seção específica para NFS-e:

| Campo | O que configura |
|-------|----------------|
| **Nº Série RPS/Temporário** (RPS/Temporary Nos.) | Nº Série para RPS |
| **Tipo de WS** (WS Type) | Tipo de web service do município |
| **Cód. Cidade WS** (WS City Code) | Código do município no sistema municipal |
| **Nome Usuário WS** / **Senha WS** (WS Username / WS Password) | Credenciais de acesso |
| **ID Certificado** (Certificate ID) | Certificado digital (pode ser diferente do NF-e) |
| **Homologação** (Homologation) | Ambiente de teste |

---

## Retenção de ISS

Em determinadas situações, o ISS é **retido pelo tomador** (quem contrata o serviço) e não pelo prestador. Isso acontece quando:

- O serviço está na lista de retenção do município
- O tomador é pessoa jurídica
- O serviço é prestado em município diferente do prestador (em alguns casos)

Na localização, a retenção de ISS é configurada como qualquer outra retenção — via Contabilização Imposto com flags de Pagamento ou Registro, dependendo de quando o efeito contábil deve ocorrer.

---

## Layout Padrão Nacional

O governo federal está implementando um **layout padrão nacional** para NFS-e, que visa unificar os sistemas municipais. A localização já suporta este layout onde disponível.

Quando o layout padrão está ativo, a comunicação vai para um sistema federal único em vez do sistema de cada prefeitura.

---

## Apuração e conciliação de ISS

Além da emissão de NFS-e, a localização oferece um módulo de **apuração de ISS** no app Tax Books. Ele permite:

- **Importação de NFS-e da prefeitura** — baixa o que a prefeitura tem registrado (suporta 6 municípios)
- **Conciliação bidirecional** — compara o que o BC emitiu com o que a prefeitura registrou, identificando divergências
- **Demonstrativo de ISS** — demonstrativo fiscal via Tax Statement
- **Relatórios legais** — Livro Registro de ISS, Registro de Saídas

Este fluxo é tipicamente executado mensalmente, junto com as demais apurações (ICMS/IPI, PIS/COFINS).

---

## Pontos de atenção

- **Município sem web service configurado** — a NFS-e não será transmitida. Verifique a Config. NF-e da filial e o mapeamento de city codes na tabela de Municípios.
- **Código de serviço** — cada município tem sua lista de códigos de serviço (baseada na LC 116/2003, mas com variações). O código deve estar correto para o município aceitar.
- **Credenciais expiradas** — alguns municípios exigem renovação periódica das credenciais de acesso ao web service.
- **Descrição do serviço** — a Config. NF-e oferece flags para customizar a descrição enviada na NFS-e:
  - **Lista Itens Detalhada** (Detailed Items List) — envia a lista de itens individualmente em vez de resumida
  - **Descartar Descrições Itens** (Supress Item Descriptions) — omite as descrições dos itens (mutuamente exclusivo com Lista Detalhada)
  - **Utilizar Textos Estendidos** (Use Extended Texts) — adiciona os textos estendidos do item à descrição
  - **Adicionar Dados de Vencimentos** (Add Due Date Information) — inclui informações de vencimento das parcelas
  - **Tamanho Máx. Discriminação** (Max. Description Length) — limita o comprimento da descrição enviada
- **RPS sem retorno** — se a transmissão falhar e o RPS não for convertido, ele pode expirar (prazo municipal). É necessário retransmitir ou cancelar.

---

## Para praticar

1. Crie uma venda de **serviço** (item do tipo Serviço com ISS configurado). Registre e localize o Documento Fiscal
2. Verifique o ISS calculado: qual a alíquota? Qual o município de competência?
3. Abra a **Config. NF-e** da filial e localize a seção de NFS-e. Qual web service está configurado?
4. Na tabela de **Municípios**, confira se o município tem Códigos de Cidade mapeados
5. Se o ambiente permitir, transmita uma NFS-e e acompanhe o retorno

**Referência**: consulte o manual "NFS-e" e o documento "Layout Padrão DANFSe" disponíveis nos documentos internos.
