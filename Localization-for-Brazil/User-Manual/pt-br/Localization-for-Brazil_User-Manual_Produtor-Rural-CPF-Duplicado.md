<!-- Commits: 9855f178 -->
# Produtor Rural: CPF Duplicado Permitido

**Data**: Novembro/2022

---

## Sumário

Esta atualização permite o cadastro do mesmo **CPF** para mais de um cliente ou fornecedor quando o campo **Produtor Rural** (`RuralProducerFNX`) está marcado. Anteriormente, o sistema bloqueava qualquer duplicidade de CNPJ/CPF entre cadastros da mesma categoria, sem exceção.

## 1. Contexto

### O que é

No Brasil, um **produtor rural pessoa física** pode possuir múltiplas propriedades rurais com inscrições estaduais distintas, mas todas vinculadas ao mesmo CPF. Em cenários onde cada propriedade precisa ser cadastrada como um cliente ou fornecedor separado no Business Central (por exemplo, para controle de NF-e por inscrição estadual), o sistema impedia o cadastro porque detectava duplicidade de CPF.

### Quando se aplica

- Cadastro de **clientes** ou **fornecedores** do tipo pessoa física que são produtores rurais
- Quando o mesmo CPF precisa estar vinculado a mais de um cadastro na mesma categoria

---

## 2. Comportamento

### Regra de validação do CNPJ/CPF

| Situação | Produtor Rural | Resultado |
|----------|---------------|-----------|
| CPF já existe em outro cadastro da mesma categoria | Desmarcado | Erro: CPF duplicado não permitido |
| CPF já existe em outro cadastro da mesma categoria | Marcado | Permitido sem erro |
| CNPJ já existe em outro cadastro da mesma categoria | Qualquer | Erro: CNPJ duplicado não permitido, exceto em clientes autorizados a repetir o CNPJ (ver abaixo) |

A exceção de duplicidade se aplica **apenas ao CPF** (pessoa física) e **apenas quando o campo Produtor Rural está marcado** no cadastro sendo editado. A validação de CNPJ (pessoa jurídica) não muda com o campo Produtor Rural: em clientes, a repetição de CNPJ é tratada por outra configuração, descrita em [Clientes: Múltiplos Cadastros com o Mesmo CNPJ](Localization-for-Brazil_User-Manual_Cliente-Multiplos-Cadastros-Mesmo-CNPJ.md).

> **Nota**: O campo **Produtor Rural** (`RuralProducerFNX`) deve ser marcado **antes** de preencher o campo **CNPJ/CPF** (`CNPJCPFFNX`) para que a exceção seja aplicada.
