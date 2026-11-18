<!-- Source: internal documentation -->
# MIT — Módulo de Inclusão de Tributos

## Sumário

O MIT (Módulo de Inclusão de Tributos) é o formato de arquivo utilizado pela Receita Federal para informar à **DCTFWeb** os débitos de tributos federais que não são apurados automaticamente pelo eSocial ou pela EFD-Reinf. Na Localização Brasileira do Business Central, a geração do arquivo MIT está integrada ao módulo de **Apuração Guias de Impostos** (`TaxBillsSettlementFNX`). O usuário apura os tributos, registra a provisão contábil e exporta o arquivo JSON diretamente do cartão da apuração, para importação manual no portal da DCTFWeb.

---

## 1. Contexto

### O que é

O MIT é um arquivo JSON com estrutura definida pela Receita Federal. Ele é importado no portal da DCTFWeb para declarar débitos de tributos federais que não chegam ao sistema por integração automática. Os tributos contemplados são: **IRPJ**, **CSLL**, **IRRF**, **IPI**, **PIS/PASEP**, **COFINS** e **CIDE**.

### Quando utilizar

O MIT é necessário quando a empresa possui débitos de tributos federais que devem ser declarados na DCTFWeb e que não são originados do eSocial nem da EFD-Reinf. O arquivo deve ser importado manualmente no portal da Receita Federal após a exportação pelo Business Central.

---

## 2. Fluxo de Trabalho

O processo completo segue as etapas abaixo:

1. **Criar a Apuração.** Na página **Apurações de Guias de Impostos** (`TaxBillsSettlementsFNX`), crie um novo registro informando o período (**Data Inicial** e **Data Final**, obrigatoriamente no mesmo mês).

2. **Processar.** Acione a ação **Processar** (`Process`). O sistema lê os documentos fiscais e os registros de fornecedor do período, agrupa-os por **Jurisdição Imposto** (`TaxJurisdictionFNX`) e cria as linhas de tributo com os respectivos documentos.

3. **Definir o tipo DCTFWeb.** Nas linhas de tributo, verifique que o campo **Tipo Guia Pagamento** (`PaymentBillType`) está definido como **DCTFWeb**. Esse valor é preenchido automaticamente conforme a configuração da jurisdição.

4. **Registrar Provisão DCTFWeb.** Acione a ação **Registrar Provisão DCTFWeb** (`DCTFWebPostProvision`). O sistema cria os lançamentos contábeis: débito no fornecedor DCTFWeb da jurisdição e créditos nos fornecedores originais de cada documento, com aplicação automática.

5. **Exportar Arquivo MIT.** Acione a ação **Exportar Arquivo MIT** (`ExportMITFile`). O sistema gera o arquivo JSON no formato exigido pela Receita Federal. O nome do arquivo segue o padrão `CNPJ(8 dígitos)-MIT-AAAAMM.json`.

6. **Importar na DCTFWeb.** O arquivo gerado deve ser importado manualmente no portal da DCTFWeb na Receita Federal.

7. **Reverter (se necessário).** Caso seja preciso desfazer a provisão, acione a ação **Reverter Provisão DCTFWeb** (`DCTFWebReverseProvision`). O sistema desaplica e reverte os lançamentos contábeis criados.

---

## 3. Configuração

### 3.1 Configuração DCTF — Seção MIT

Acesse a página **Configuração DCTF** (`DCTFSetupFNX`) e preencha os campos da seção **MIT**:

| Campo | Descrição | Opções |
|-------|-----------|--------|
| **Qualificação Empresa** (`MITCompanyQualification`) | Tipo de pessoa jurídica conforme o layout MIT | 1 – PJ em geral; 2 – Agência de Fomento/Banco de Desenvolvimento; 3 – Cooperativa de Crédito; 4 – Sociedade Corretora de Seguros; 5 – Sociedade Seguradora/Capitalização; 6 – Entidade Fechada de Previdência; 7 – Sociedade Cooperativa; 8 – Cooperativa de Produção Agropecuária/Consumo; 9 – Autarquia/Fundação Pública; 10 – Empresa Pública/Economia Mista; 11 – Estado/DF/Município/Órgão Público; 12 – Mais de uma qualificação no mês |
| **Forma de Tributação** (`MITTributationType`) | Regime de tributação do lucro | 1 – Real Anual; 2 – Real Trimestral; 3 – Presumido; 4 – Arbitrado; 5 – Imune do IRPJ; 6 – Isenta do IRPJ; 7 – Optante pelo Simples Nacional |
| **Variações Monetárias** (`MITMonetaryVariations`) | Regime de reconhecimento das variações monetárias | 1 – Regime de Caixa; 2 – Regime de Competência; 3 – Regime de Caixa (elevada oscilação cambial) |

### 3.2 Configuração DCTF — Seção Responsável pelo Preenchimento

Na mesma página **Configuração DCTF** (`DCTFSetupFNX`), preencha a seção **Responsável pelo Preenchimento** com os dados do contador responsável:

| Campo | Descrição |
|-------|-----------|
| **CPF** | CPF do responsável pelo preenchimento |
| **CRC** | Número do registro no CRC |
| **DDD** / **Telefone** | Telefone de contato |
| **E-Mail** | E-mail de contato |

Todos esses campos são obrigatórios para que a exportação do arquivo MIT seja concluída com sucesso.

### 3.3 Configuração EFD Contribuições

Em determinadas combinações de **Qualificação Empresa** e **Forma de Tributação** (por exemplo, PJ em geral com Real Anual), o sistema exige que o regime de incidência de PIS/COFINS (cumulativo, não cumulativo ou ambos) esteja preenchido na página **Configuração EFD Contribuições** (`EFDContribSetupFNX`). Preencha esse campo antes de exportar o arquivo MIT.

### 3.4 Códigos Receita Tributo

Na página **Códigos Receita Tributo** (`TaxRevenueCodesFNX`), cada código de receita que deve ser incluído no arquivo MIT precisa estar com o campo **Exportação MIT** (`MITExported`) marcado como **Sim**. Somente os códigos marcados são exportados.

### 3.5 Jurisdição Imposto

No **Cartão Jurisdição Imposto** (`TaxJurisdictionCardFNX`), cada jurisdição envolvida na apuração deve ter configurados:

| Campo | Descrição |
|-------|-----------|
| **Identificação imposto** (`TaxIdentification`) | Classificação do tributo: IRPJ, CSLL, IRRF, IPI, PIS, COFINS ou CIDE |
| **Periodicidade** (`Periodicidade`) | Como o período do débito é informado no arquivo: Diário, Decendial ou Quinzenal |
| **Tipo Guia Pagamento** (`PaymentBillType`) | Deve ser definido como **DCTFWeb** para que as linhas da apuração sejam incluídas no MIT |
| **Cód. Receita Tributo** (`TaxRevenueCode`) | Código de receita associado à jurisdição |

---

## 4. Particularidades

**Tributos suportados**: o arquivo MIT contempla os grupos IRPJ, CSLL, IRRF, IPI, PIS/PASEP, COFINS e Contribuições Diversas (CIDE). Tributos fora desses grupos não são incluídos.

**Sem movimento**: se a apuração não contiver nenhum débito, o arquivo é gerado com o indicador `SemMovimento = true`, e os campos de **Forma de Tributação** e **Variações Monetárias** são omitidos automaticamente.

**CNPJ do estabelecimento**: para tributos de IPI e CIDE, quando o documento fiscal está associado a uma filial, o sistema inclui no arquivo o CNPJ do estabelecimento (os 6 dígitos de filial/ordem), identificando qual unidade gerou o débito.

**Periodicidade**: o sistema calcula automaticamente o período do débito conforme a **Periodicidade** configurada na jurisdição:

| Periodicidade | Comportamento |
|---------------|---------------|
| Diário | Informa o dia específico do débito |
| Decendial | Divide o mês em 3 decêndios: dias 1–10, 11–20, 21+ |
| Quinzenal | Divide o mês em 2 quinzenas: dias 1–15, 16+ |

**Regime PIS/COFINS**: o campo de regime PIS/COFINS só é incluído no arquivo quando a combinação de **Qualificação Empresa** e **Forma de Tributação** exige essa informação, conforme as regras do layout oficial do MIT. Em apurações sem movimento, esse campo é sempre omitido.

**Datas no mesmo mês**: a **Data Inicial** e a **Data Final** da apuração devem estar no mesmo mês. O sistema retorna um erro caso as datas pertençam a meses diferentes.
