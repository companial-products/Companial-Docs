<!-- PR: #274, #296, #312 -->
# NFS-e: Manuais de Emissão por Município e Plataforma

**Data**: Outubro/2026

---

## Sumário

Cada prefeitura, ou plataforma que atende vários municípios, tem exigências próprias para a emissão da NFS-e: credenciais, códigos de serviço e de tributação, numeração do RPS, tamanho do lote, ambiente de homologação, consulta e cancelamento. Esta página reúne os manuais de configuração e emissão de cada integração disponível na lista de cidades da **Config. NF-e** da filial.

Todos os manuais seguem a mesma estrutura: contexto, configuração da filial campo a campo, emissão, consulta e cancelamento, DANFSe, pontos de atenção e resumo da configuração.

---

## 1. Como encontrar o manual da filial

A integração usada pela filial é escolhida na **Config. NF-e** (página `NFeSetupFNX`), aba **NFS-e**, campo **Cód. Cidade WS**. Procure o município da filial na [seção 2](#2-manuais-por-município). Se ele não estiver lá, o município pode emitir por uma das plataformas da [seção 3](#3-manuais-por-plataforma).

Os conceitos gerais da NFS-e e do ISS estão em [NFS-e e ISS](Localization-for-Brazil_User-Manual_guia-fiscal-nfse.md), e a configuração dos cadastros da Reforma Tributária em [NFS-e e Reforma Tributária — Configuração e Emissão](Localization-for-Brazil_User-Manual_NFSe-Reforma-Tributaria.md). O que cada integração envia de IBS/CBS e retenções está em [Reforma Tributária - Suporte à Emissão de Notas de Serviço](Localization-for-Brazil_User-Manual_Reforma-Acompanhamento-Emissao-NotaServico.md).

---

## 2. Manuais por município

| UF | Município | Provedor / Plataforma | Opção em **Cód. Cidade WS** | Manual |
| --- | --- | --- | --- | --- |
| BA | Salvador | ABRASF | **Salvador - BA** (`BA-Salvador`) | [NFS-e: Emissão em Salvador - BA](Localization-for-Brazil_User-Manual_NFSe-Salvador-BA.md) |
| CE | Eusébio | EL (layout nacional) | **Eusébio - CE (EL)** (`CE-Eusebio-EL`) | [NFS-e: Emissão em Eusébio - CE](Localization-for-Brazil_User-Manual_NFSe-Eusebio-CE-EL.md) |
| CE | Fortaleza | Ginfes | **Fortaleza - CE** (`CE-Fortaleza`) | [NFS-e: Emissão em Fortaleza - CE](Localization-for-Brazil_User-Manual_NFSe-Fortaleza-CE.md) |
| DF | Brasília | ISSNet Nacional (layout nacional) | **Brasília - DF (ISSNet Nacional)** (`DF-Brasilia-ISSNetNacional`) | [NFS-e: Emissão em Brasília - DF](Localization-for-Brazil_User-Manual_NFSe-Brasilia-DF-ISSNet-Nacional.md) |
| GO | Goiânia | ISSNet Nacional (layout nacional) | **Goiânia - GO (ISSNet Nacional)** (`GO-Goiania-ISSNetNacional`) | [NFS-e: Emissão em Goiânia - GO](Localization-for-Brazil_User-Manual_NFSe-Goiania-GO-ISSNet-Nacional.md) |
| MG | Contagem | Governança Brasil | **Contagem - MG (Governança Brasil)** (`MG-Contagem-GovBr`) | [NFS-e: Emissão em Contagem - MG](Localization-for-Brazil_User-Manual_NFSe-Contagem-MG-Governanca-Brasil.md) |
| PA | Barcarena | ISSNet Nacional (layout nacional) | **Barcarena - PA (ISSNet Nacional)** (`PA-Barcarena-ISSNetNacional`) | [NFS-e: Emissão em Barcarena - PA](Localization-for-Brazil_User-Manual_NFSe-Barcarena-PA-ISSNet-Nacional.md) |
| PE | Ipojuca | Tinus | **Ipojuca - PE (Tinus)** (`PE-Ipojuca-TINUS`) | [NFS-e: Emissão em Ipojuca - PE](Localization-for-Brazil_User-Manual_NFSe-Ipojuca-PE-Tinus.md) |
| PE | Olinda | Tinus | **Olinda - PE (Tinus)** (`PE-Olinda-TINUS`) | [NFS-e: Emissão em Olinda - PE](Localization-for-Brazil_User-Manual_NFSe-Olinda-PE-Tinus.md) |
| RJ | Saquarema | Governança Brasil | **Saquarema - RJ (Governança Brasil)** (`RJ-Saquarema-GovBr`) | [NFS-e: Emissão em Saquarema - RJ](Localization-for-Brazil_User-Manual_NFSe-Saquarema-RJ-Governanca-Brasil.md) |
| RN | São Gonçalo do Amarante | Tinus | **São Gonçalo do Amarante - RN (Tinus)** (`RN-SaoGoncaloDoAmarante-TINUS`) | [NFS-e: Emissão em São Gonçalo do Amarante - RN](Localization-for-Brazil_User-Manual_NFSe-Sao-Goncalo-do-Amarante-RN-Tinus.md) |
| RS | Rio Grande | Sigcorp (layout nacional) | **Rio Grande - RS (Sigcorp)** (`RS-RioGrande`) | [NFS-e: Emissão em Rio Grande - RS](Localization-for-Brazil_User-Manual_NFSe-Rio-Grande-RS.md) |
| SC | Itajaí | ABRASF | **Itajaí - SC** (`SC-Itajai`) | [NFS-e: Emissão em Itajaí - SC](Localization-for-Brazil_User-Manual_NFSe-Itajai-SC.md) |
| SE | Aracaju | WebISS | **Aracaju - SE (WebISS)** (`SE-Aracaju-WebISS`) | [NFS-e: Emissão em Aracaju - SE](Localization-for-Brazil_User-Manual_NFSe-Aracaju-SE-WebISS.md) |
| SP | Barueri | Customizado | **Barueri - SP** (`SP-Barueri`) | [NFS-e: Emissão em Barueri - SP](Localization-for-Brazil_User-Manual_NFSe-Barueri-SP.md) |
| SP | Cotia | Emissor Público Nacional | **Emissor Público Nacional** (`SEFIN`) | [NFS-e: Emissão em Cotia - SP](Localization-for-Brazil_User-Manual_NFSe-Cotia-SP-Emissor-Nacional.md) |
| SP | Diadema | GISS | **Diadema - SP (GISS)** (`SP-Diadema-GISS`) | [NFS-e: Emissão em Diadema - SP](Localization-for-Brazil_User-Manual_NFSe-Diadema-SP-GISS.md) |
| SP | Embu das Artes | Giap/Obaratec | **Embu das Artes - SP (Giap / Obaratec)** (`SP-EmbuDasArtes-GIAP_OB`) | [NFS-e: Emissão em Embu das Artes - SP](Localization-for-Brazil_User-Manual_NFSe-Embu-das-Artes-SP-Giap-Obaratec.md) |
| SP | Jundiaí | GISS | **Jundiaí - SP (GISS)** (`SP-Jundiai-GISS`) | [NFS-e: Emissão em Jundiaí - SP](Localization-for-Brazil_User-Manual_NFSe-Jundiai-SP-GISS.md) |
| SP | Santos | GISS | **Santos - SP (GISS)** (`SP-Santos-GISS`) | [NFS-e: Emissão em Santos - SP](Localization-for-Brazil_User-Manual_NFSe-Santos-SP-GISS.md) |
| SP | São Bernardo do Campo | GINFES (layout 2026) | **São Bernardo do Campo - SP** (`SP-SaoBernardoDoCampo`) | [NFS-e: Emissão em São Bernardo do Campo - SP](Localization-for-Brazil_User-Manual_NFSe-Sao-Bernardo-do-Campo-SP-GINFES.md) |
| SP | São Paulo | Customizado (versão 2026) | **São Paulo - SP 2026** (`SP-SaoPaulo2026`) | [NFS-e: Emissão em São Paulo - SP](Localization-for-Brazil_User-Manual_NFSe-Sao-Paulo-SP.md) |

A opção de Itajaí pode ainda aparecer na lista como **Santa Catarina - SC**: o código `SC-Itajai` é o mesmo.

### Cotia, Brasília e Goiânia: integrações antigas

As opções **Cotia - SP**, **Cotia - SP (Giap / Obaratec)**, **Brasília - DF (ISSNet)** e **Goiânia - GO (ISSNet)** não aparecem mais na lista de cidades. As filiais que já estão configuradas com elas continuam emitindo normalmente. Os manuais de [Cotia](Localization-for-Brazil_User-Manual_NFSe-Cotia-SP-Emissor-Nacional.md), [Brasília](Localization-for-Brazil_User-Manual_NFSe-Brasilia-DF-ISSNet-Nacional.md) e [Goiânia](Localization-for-Brazil_User-Manual_NFSe-Goiania-GO-ISSNet-Nacional.md) descrevem a integração atual e o que revisar na filial ao trocar.

---

## 3. Manuais por plataforma

As plataformas abaixo atendem vários municípios. O manual vale para qualquer município atendido pela plataforma.

| Plataforma | Opção em **Cód. Cidade WS** | Manual |
| --- | --- | --- |
| Emissor Público Nacional (Sefin Nacional) | **Emissor Público Nacional** (`SEFIN`) | [NFS-e: Emissão pelo Emissor Público Nacional (SEFIN)](Localization-for-Brazil_User-Manual_NFSe-Emissor-Publico-Nacional-SEFIN.md) |
| Betha Cloud | **Betha Cloud (Reforma Tributária)** (`BethaCloud`) | [NFS-e: Emissão pela plataforma Betha Cloud](Localization-for-Brazil_User-Manual_NFSe-Betha-Cloud.md) |

A relação dos municípios atendidos por cada plataforma será publicada nesta página.

---

## 4. Integrações sem manual

**PlugNotas** e **São João da Barra - RJ (ISSWeb)** estão na lista de cidades, mas ainda não têm manual. Para elas, procure o suporte.
