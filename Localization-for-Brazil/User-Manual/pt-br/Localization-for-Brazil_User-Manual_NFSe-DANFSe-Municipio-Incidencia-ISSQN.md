<!-- PR: #210, #337 -->
# NFS-e: Município de Incidência do ISSQN no DANFSe

**Data**: Setembro/2026

---

## Sumário

O campo **Município de Incidência do ISSQN** do DANFSe (layout padrão nacional) passa a ter valor próprio, em vez de repetir o **Local da Prestação**. Uma nova opção na **Config. NF-e**, por filial, indica que o ISSQN daquela filial incide no próprio município do estabelecimento prestador: com ela ativada, o impresso mostra o município da filial; desativada, continua mostrando o município da prestação. Quando não há ISSQN a recolher, o campo sai em branco (`-`) em qualquer filial.

A mudança é apenas na montagem do documento impresso. Nada muda no cálculo do ISSQN, na escrituração nem no conteúdo enviado à prefeitura.

---

## 1. Contexto

Em algumas prefeituras o ISSQN é devido no município do estabelecimento prestador mesmo quando o serviço é executado em outro município. A declaração transmitida à prefeitura já respeitava essa regra, mas o DANFSe montado pelo Business Central preenchia o campo **Município de Incidência do ISSQN** com uma cópia do **Local da Prestação**. Sempre que os dois municípios não coincidiam, o documento entregue ao tomador e arquivado pela empresa contradizia a nota efetivamente declarada.

Com esta atualização, o município de incidência passa a ser decidido no momento em que o DANFSe é montado, a partir da configuração da filial emissora e do ISSQN apurado na nota. A regra vale para todas as filiais que imprimem o DANFSe pelo layout padrão, tanto as que transmitem pelo padrão nacional quanto as que transmitem pelo web service da própria prefeitura.

Para ativar o layout padrão e conhecer as formas de obter o DANFSe, veja [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md).

---

## 2. Configuração

1. Navegue até **Config. NF-e** (página `NFeSetupFNX`) da filial desejada.
2. Na aba **NFS-e**, localize o grupo **Layout Padrão**.
3. Ative o campo **Incidência do ISSQN no Município do Prestador** (`NFSISSIncidenceAtProviderCity`).

O campo fica ao lado de **Utilizar Layout Padrão** e só produz efeito quando este último está ativado, pois é ele que liga a geração do DANFSe pelo sistema.

A opção nasce **desativada** em todas as filiais. Nenhuma filial existente muda o que imprime nas notas tributadas sem que alguém ative a opção.

> **Atenção**: a opção afeta **apenas o documento impresso**. O conteúdo enviado à prefeitura não muda. Ative-a somente nas filiais em que a prefeitura declara o ISSQN como devido no município do prestador; do contrário, o impresso passa a divergir do que foi declarado. O sistema não emite aviso nem bloqueia a ativação em nenhuma filial.

---

## 3. Comportamento

### Regra de preenchimento

O sistema decide o valor de **Município de Incidência do ISSQN** para cada NFS-e conforme a tabela abaixo:

| Situação da nota | Opção desativada (padrão) | Opção ativada |
| --- | --- | --- |
| Há ISSQN a recolher (apurado ou retido pelo tomador) | Município da prestação | **Município da filial** |
| Não há ISSQN a recolher (imunidade, exportação de serviço, não incidência, exigibilidade suspensa) | `-` | `-` |
| Serviço prestado no exterior (sem município brasileiro de prestação) | `-` | `-` |

Observações:

- A nota com **ISSQN retido pelo tomador** segue a opção da filial como qualquer outra nota tributada. Com a opção ativada, ela também imprime o município da filial.
- O que decide se há ISSQN a recolher é o valor do imposto apurado na nota, e não o código de tributação configurado na filial. Filiais que usam códigos de tributação próprios são tratadas da mesma forma.
- O campo **Local da Prestação** não muda: continua mostrando o município onde o serviço foi prestado.

### Onde a regra se aplica

O mesmo valor aparece nas três formas de obter o DANFSe, pois todas montam o documento pela mesma rotina:

| Origem | Como obter |
| --- | --- |
| Impressão direta | Ação **Imprimir** na página **NFS-e - Envio de Lote** (`TransmitNFSeBatchFNX`) ou **Imprimir RPS** na página **Documento Fiscal de Venda** (`SalesFiscalDocumentFNX`) |
| Prévia antes do lançamento | Ação **Gerar Rascunho do DANFSe** na Ordem de Venda ou na Fatura de Venda |
| PDF armazenado no documento fiscal | Ações **Abrir DANFSe** e **Atualizar DANFSe** na página **Documento Fiscal Registrado** (`PostedFiscalDocumentFNX`) |

A regra vale para o DANFSe montado com os dados do Business Central. O DANFSe de uma NFS-e já emitida é montado com o XML da nota autorizada e mostra o município de incidência que consta na nota, qualquer que seja a opção da filial. A opção continua valendo para a prévia (DPS), para o rascunho e para o DANFSe impresso com os valores do Business Central quando o XML não está disponível (veja [NFS-e: Layout Padrão DANFSe](Localization-for-Brazil_User-Manual_NFSe-Layout-Padrao-DANFSe.md#origem-dos-dados)).

### Notas emitidas antes desta atualização

Use **Atualizar DANFSe** no documento fiscal registrado para gerar o PDF de novo. Quando o XML da nota está disponível, o novo PDF traz o município de incidência que consta na nota autorizada. Não é necessário cancelar nem reemitir a nota.

### Mudança sem ação de configuração

Existe uma única mudança de impresso que ocorre em todas as filiais, mesmo sem ativar a opção: nas notas **sem ISSQN a recolher** (imunidade, exportação de serviço, não incidência e exigibilidade suspensa), o campo **Município de Incidência do ISSQN** passa a sair com `-`, em vez de repetir o local da prestação. É a correção do mesmo defeito: o campo mostrava um valor que não era dele. As notas tributadas de filiais que não ativarem a opção continuam exatamente como antes.

### O que não muda

- Transmissão da NFS-e e conteúdo da declaração enviada à prefeitura.
- Cálculo, retenção e recolhimento do ISSQN; apuração e escrituração do imposto.
- Demais campos e diagramação do DANFSe, inclusive o bloco de **Tributação IBS / CBS**, que continua mostrando o município da prestação como incidência e será tratado no trabalho da Reforma Tributária.
- O relatório de RPS usado pelas filiais que não adotaram o layout padrão, que não possui esse campo.

---

## 4. Exemplo Prático

A filial de **Olinda (PE)** presta um serviço em **Recife (PE)**. A prefeitura de Olinda exige que o ISSQN seja declarado como devido em Olinda, o município do prestador.

| Configuração da filial de Olinda | Local da Prestação | Município de Incidência do ISSQN |
| --- | --- | --- |
| Opção desativada (comportamento anterior) | Recife - PE | Recife - PE |
| Opção ativada | Recife - PE | **Olinda - PE** |

Com a opção ativada, o DANFSe passa a mostrar Olinda, o mesmo município que a declaração transmitida à prefeitura informa. Se a mesma nota tiver o ISSQN retido pelo tomador, o resultado é o mesmo: Olinda. Se a nota for de exportação de serviço ou estiver amparada por imunidade, o campo sai com `-` nas duas configurações.

---

## 5. Resumo dos Campos Novos

### Tabela Config. NF-e (`NFeSetupFNX`)

| Campo | Localização | Tipo | Descrição |
| --- | --- | --- | --- |
| **Incidência do ISSQN no Município do Prestador** (`NFSISSIncidenceAtProviderCity`) | Aba NFS-e > Layout Padrão | Booleano | Indica que o DANFSe imprime o município da filial como município de incidência do ISSQN, em vez do município da prestação. Afeta apenas o documento impresso. |
