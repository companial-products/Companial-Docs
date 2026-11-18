# Ciclo de Vida da NF-e

A NF-e (Nota Fiscal Eletrônica, modelo 55) é o documento fiscal eletrônico obrigatório para operações de circulação de mercadorias no Brasil. Este capítulo explica todo o ciclo de vida: da geração do XML até o cancelamento.

---

## O que é

A NF-e é um arquivo XML que contém todos os dados da operação fiscal. Ela é assinada digitalmente pela empresa e autorizada pela SEFAZ (Secretaria da Fazenda) do estado. Sem a autorização da SEFAZ, a NF-e não tem validade.

O fluxo básico:

```
BC gera dados fiscais → Serviço Azure gera XML + assinatura → Envia à SEFAZ
    → SEFAZ valida → Autoriza (ou rejeita) → Retorna protocolo
        → Serviço gera o DANFe (PDF) a partir do XML autorizado
```

---

## Como a localização implementa

### Comunicação com a SEFAZ

A localização **não** se comunica diretamente com a SEFAZ. O BC gera um arquivo em formato intermediário com todos os dados fiscais e o envia, junto com o certificado digital, a um serviço hospedado no Azure. O serviço:

1. Recebe os dados e o certificado digital
2. Gera o XML definitivo da NF-e no formato exigido pela SEFAZ
3. Realiza a assinatura digital
4. Transmite à SEFAZ e retorna a resposta
5. Gera o DANFe (PDF) a partir do XML autorizado

### Código Status

Cada NF-e tem um **Código Status**, retornado pela SEFAZ, que indica seu estado atual. O texto da resposta fica no campo **Resultado**:

| Código Status | Significado | O que fazer |
|--------|------------|-------------|
| *(vazio)* | Criada, não transmitida | Transmitir |
| **100** | **Autorizada** | Sucesso — emitir DANFe, enviar e-mail |
| **120** | Autorizada com alerta | Equivalente a 100 |
| **150** | Autorizada fora de prazo | Equivalente a 100 |
| **101** | **Cancelada** | NF-e cancelada com sucesso |
| **102** | Numeração inutilizada | Número reservado inutilizado |
| **103** | Lote recebido (assíncrono) | Aguardar e usar **Consultar Status** |
| **104** | Lote processado | Usar **Consultar Status** |
| **105** | Lote em processamento | Aguardar e usar **Consultar Status** |
| **135** | Evento de cancelamento registrado, em processamento | Usar **Consultar Status** |
| **110**, **301**, **302**, **303** | **Denegada** (uso denegado) | A NF-e existe na SEFAZ com situação denegada: não pode ser corrigida, retransmitida nem inutilizada, e o número não pode ser reutilizado. Ver [Denegação](#denegação) |
| **217** | NF-e não encontrada | Retransmitir |
| **156** a **188** e **>= 201** (exceto 301 a 303) | **Rejeitada** | Corrigir e retransmitir. Ver [Rejeição](#rejeição) |

### Fluxo visual

```
Documento Fiscal registrado
    │
    ▼
Transmissão à SEFAZ
    │
    ├── 100/150 → AUTORIZADA ✓
    │       │
    │       ├── Cancelamento → 101 (Cancelada)
    │       ├── Carta de Correção (CCe)
    │       └── Eventos SEFAZ
    │
    ├── 156-188 / >= 201 → REJEITADA ✗
    │       └── Corrigir dados → Retransmitir
    │
    ├── 110/301/302/303 → DENEGADA ✗ (definitivo)
    │       └── Verificar situação cadastral → Cancelar a fatura no sistema
    │
    ├── 103/104/105 → EM PROCESSAMENTO ⏳
    │       └── Consultar Status
    │
    └── Sem resposta da SEFAZ → documento registrado com aviso
            └── Consultar Status
```

### Rejeição

- **Com NF-e Automática** — quando a SEFAZ rejeita a NF-e durante o registro, o registro é desfeito e o documento de origem continua aberto. Leia o motivo na mensagem, corrija o documento de origem (ou a configuração) e registre novamente.
- **Rejeição por duplicidade (204)** — a NF-e provavelmente foi autorizada em uma tentativa anterior cuja resposta não chegou ao Business Central. Não exclua o documento nem use o número em outro documento: registre-o sem **NF-e Automática** e, no **Documento Fiscal Registrado**, use **NF-e** → **Consultar Status** para recuperar a autorização.
- **Duplicidade com diferença na chave de acesso (539)** — o número já foi usado por uma NF-e autorizada com outra chave. Consulte na SEFAZ a chave informada na mensagem para descobrir qual NF-e usa o número; pode ser este mesmo documento, autorizado em uma tentativa anterior com outra data ou valor.
- **NF-e transmitida depois do registro** — no **Documento Fiscal Registrado**, leia o **Resultado**, corrija os campos permitidos com **Editar Campos** (ver [Editar Campos do Documento Fiscal Registrado](Localization-for-Brazil_User-Manual_Editar-Campos-Documento-Fiscal-Registrado.md)) e use **NF-e** → **Transmitir** novamente.

### Denegação

A denegação (uso denegado) ocorre por irregularidade fiscal do emitente ou do destinatário. Não é uma rejeição: a NF-e fica registrada na SEFAZ com situação denegada e o número é consumido. Quando isso acontece no registro com **NF-e Automática**, o documento é registrado e o sistema exibe a mensagem *"Essa NF-e foi denegada pela SEFAZ. Por favor, verifique a situação fiscal da sua empresa ou do seu cliente."*. Então:

1. Verifique a situação cadastral (I.E.) da empresa ou do cliente junto à SEFAZ.
2. Cancele a fatura no sistema (**Cancelar**) para estornar os lançamentos. A NF-e denegada não é cancelada na SEFAZ nem inutilizada.
3. Depois de regularizada a situação, emita um novo documento, com novo número.

### Sem resposta da SEFAZ

Quando a SEFAZ demora a processar o lote, o documento é registrado mesmo sem o status final e o sistema avisa que a NF-e precisa ser consultada. Abra o **Documento Fiscal Registrado** e use **NF-e** → **Consultar Status**. Ver [NF-e: novo tratamento de demoras no processamento da SEFAZ](Localization-for-Brazil_User-Manual_NFe-novo-tratamento-de-demoras-SEFAZ.md).

---

## Operações da NF-e

### Transmissão

Pode ser:
- **Automática** — ao registrar o documento de venda, a NF-e é transmitida automaticamente (padrão definido na Config. NF-e, campo **NFe Automática**, e marcado no documento pelo campo **NF-e Automática**)
- **Manual** — o usuário transmite depois, pela ação **NF-e** → **Transmitir** do **Documento Fiscal Registrado** ou pela página **NF-e - Transmitir Pendentes** (ações **Adicionar Docs. Pendentes**, **Transmitir** e **Consultar**)
- **Em lote (assíncrona)** — múltiplas NF-e são enviadas em um lote; o resultado é consultado depois

### Cancelamento

Uma NF-e autorizada pode ser cancelada dentro do prazo legal (em geral, 24 horas a partir da autorização). O cancelamento parte da fatura registrada:
1. Na fatura registrada, use **Cancelar** ou **Corrigir**. O sistema registra o estorno e marca o documento fiscal como **Cancelado**.
2. Com **Transmitir cancelamento automaticamente** ligado na Config. NF-e da filial, o sistema pede a justificativa e cancela a NF-e na SEFAZ na mesma ação. Caso contrário, conclua no **Documento Fiscal Registrado** pela ação **NF-e** → **Cancelar NF-e**.
3. Quando a SEFAZ homologa o cancelamento, o **Código Status** passa a 101.

Prazo por filial, bloqueio depois do prazo e a lista de cancelamentos pendentes: ver [NF-e: Cancelamento na SEFAZ no Mesmo Fluxo, Prazo de Cancelamento por Filial e Cancelamentos Pendentes](Localization-for-Brazil_User-Manual_NFe-Cancelamento-SEFAZ-Prazo-Pendencias.md).

### Carta de Correção Eletrônica (CCe)

Permite corrigir informações na NF-e **sem** cancelá-la. Limitações:
- Não pode alterar valores, impostos, CNPJ, IE
- O CFOP só pode ser corrigido por CC-e quando a correção não altera o imposto
- O texto de correção deve ter entre 15 e 1000 caracteres
- Até 20 CCe por NF-e
- Cada nova CCe substitui as anteriores
- A CC-e pode ser enviada até 720 horas (30 dias) após a autorização, conforme a legislação; confirme na SEFAZ da sua UF

**Caminho no BC**: no **Documento Fiscal Registrado**, use **NF-e** → **Cartas de Correção**.

### Inutilização de numeração

Quando números de NF-e ficam sem uso (ex: falha no sistema que pula números), eles devem ser **inutilizados** junto à SEFAZ. A inutilização deve ser feita até o 10º dia do mês seguinte ao da falha na numeração, conforme a legislação; confirme na SEFAZ da sua UF. Uma NF-e autorizada, cancelada ou denegada não pode ter o número inutilizado.

- **Faixa de números** — na **Config. NF-e** da filial, use a ação **Inutilizar Nº de NF-e**. Informe o **Tipo Documento Fiscal**, o **Nº Inicial**, o **Nº Final** e a **Justificativa** e use **Inutilizar Faixa**. Cada pedido aceita no máximo 10 números.
- **Número de um documento** — no **Documento Fiscal Registrado** cuja NF-e não foi autorizada, use **NF-e** → **Inutilizar Nº NF**.

### Manifestação do Destinatário (MD-e)

Para NF-e **recebidas** (de fornecedores), a empresa pode manifestar:

| Evento | Significado |
|--------|------------|
| **Ciência da Operação** | "Sei que esta NF-e existe" |
| **Confirmação da Operação** | "Recebi a mercadoria/serviço" |
| **Desconhecimento da Operação** | "Não conheço esta operação" |
| **Operação não Realizada** | "A operação não aconteceu" |

A localização possui um sistema automático de MD-e que:
1. Consulta periodicamente a SEFAZ por NF-e destinadas à empresa
2. Envia automaticamente o evento de "Ciência"
3. Permite classificar e criar documentos de compra a partir do XML recebido

---

## DANFe

O **DANFe** (Documento Auxiliar da Nota Fiscal Eletrônica) é a representação visual/impressa da NF-e. Ele **não** é a nota fiscal — a nota fiscal é o XML. O DANFe serve para:

- Acompanhar a mercadoria em trânsito
- Facilitar a consulta pelo destinatário
- Conter o código de barras com a chave de acesso

O DANFe é gerado como PDF pelo serviço intermediário no Azure, a partir do XML autorizado pela SEFAZ.

---

## Chave de acesso

Toda NF-e tem uma **chave de acesso** de 44 dígitos que a identifica unicamente. A chave contém:

```
UF + AAMM + CNPJ + Modelo + Série + Número + Tipo Emissão + Código + DV
```

Exemplo: `35260412345678000123550010000001231123456785`

A chave é usada para consultar a NF-e no portal da SEFAZ e para vincular documentos (referências, devoluções, complementares).

---

## Contingência

Quando a SEFAZ da UF está fora do ar, a localização suporta a contingência online pelo SVC (Sefaz Virtual de Contingência):

1. Na **Config. NF-e** da filial, altere **Tipo Emissão** para o serviço de contingência da UF (**Contingência SVC-AN** ou **Contingência SVC-RS**).
2. Informe **Data/Hora Entrada Contingência** e **Justificativa Contingência** (obrigatórios para transmitir em contingência).
3. As NF-e passam a ser transmitidas e autorizadas pelo SVC, normalmente; não há retransmissão posterior.
4. Quando a SEFAZ da UF normalizar, volte **Tipo Emissão** para **Normal**. Os campos de contingência são limpos.

A contingência offline (FS-DA, EPEC) não é suportada.

---

## Visualizador de XML

A localização inclui um **Visualizador de XML** integrado diretamente na página do Documento Fiscal. Ele exibe o conteúdo do XML da NF-e em formato navegável, sem necessidade de exportar o arquivo ou usar ferramentas externas.

**Caminho**: abra um Documento Fiscal → no FactBox lateral, clique em **Visualizar XML**.

Para mais detalhes, consulte [NF-e: Visualizador de XML no Business Central](Localization-for-Brazil_User-Manual_NFeXmlViewer.md).

---

## Pontos de atenção

- **NF-e rejeitada** — a maioria das rejeições são por dados inconsistentes: IE inválida, CNPJ incorreto, NCM inexistente, CST incompatível com CFOP. Leia sempre o **motivo da rejeição** (campo **Resultado** no documento fiscal).
- **Cancelamento fora do prazo** — após 24h (em geral), o cancelamento não é aceito pela SEFAZ. A alternativa é emitir uma NF-e de devolução/entrada.
- **NF-e autorizada com erro** — se o erro é em dados secundários, use CCe. Se é em valores ou impostos, é necessário cancelar (se dentro do prazo) ou emitir NF-e complementar ou NF-e de devolução/entrada.
- **Certificado digital** — se expirado ou inválido, nenhuma operação SEFAZ funciona. Verifique o certificado na Config. NF-e da filial.
- **Ambiente de teste vs. produção** — verifique sempre qual ambiente está configurado. NF-e transmitida em ambiente de teste não tem validade fiscal.

---

## Para praticar

1. Na lista de Documentos Fiscais, filtre por NF-e autorizadas (**Código Status** 100). Abra uma e localize: chave de acesso, protocolo, XML, DANFe
2. Identifique uma NF-e rejeitada e leia o motivo da rejeição. Qual dado está inconsistente?
3. Se possível, transmita uma NF-e em **ambiente de teste** e acompanhe o status mudar de 0 → 100
4. Abra a Config. NF-e da filial e verifique: **Ambiente** (teste/produção), certificado, **NFe Automática**
5. Explore a lista de **MD-e** — quais NF-e foram recebidas de fornecedores?

**Referência**: consulte os manuais de "NF-e" disponíveis nos documentos internos.
