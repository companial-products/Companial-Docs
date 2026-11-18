# NF-e: novo tratamento de demoras no processamento da SEFAZ

## Introdução

O envio de notas fiscais eletrônicas para a SEFAZ utiliza um modelo Assíncrono de processamento de documentos, através de dois serviços web separados, para Autorização **(1)** e Retorno **(2)** dos documentos.

Isso sempre se provou um desafio para a emissão de notas fiscais eletrônicas em um ERP complexo como o Business Central, onde após o registro de um documento, poucas informações podem ser modificadas para tratamento de possíveis erros detectados pelo validador da SEFAZ.

Para evitar uma quantidade grande de estornos de documentos, a função de envio automático durante o registro de documentos foi criada, que de forma simplificada, funciona assim:

1. Usuário clica em Registrar/Transmitir;
2. BC abre uma **transação**no banco de dados e faz todos os processamentos internos de registro de documento;
3. Documento já está registrado, mas a **transação**do banco de dados se mantem **aberta**;
4. BC se comunica com a SEFAZ, primeiro através do serviço de **Autorização**;
5. Serviço de autorização retorna um nº de **recibo**do documento;
6. BC aguarda 1 segundo e se comunica novamente com a SEFAZ através do serviço de **Retorno**, para consultar o status do envio utilizando o nº do **recibo:**
  1. Caso o documento ainda não tenha sido processado pela SEFAZ, realiza uma nova consulta, aumentando o tempo de espera;
  2. Caso o processamento esteja completo, retorna para o BC o status do documento.
7. BC verifica através do status se a NF-e foi de fato transmitida:
  1. Se sim, encerra a transação, efetivando 100% o registro;
  2. Se não, faz rollback de toda a transação, retornando ao estado inicial.

## Problema

Em um período que a SEFAZ esteja com sua velocidade de processamento de documentos normalizada, o processo funciona bem e evita estornos desnecessários de documentos. Já em um cenário que a SEFAZ esteja apresentando lentidão em excesso, nós temos hoje um problema na etapa abaixo:

Ao fazer a 6ª tentativa de consulta ao mesmo documento em um curto período, a SEFAZ bloqueia consultas por alguns minutos, retornando o erro 656 - Consumo Indevido. A consequência disso é que o registro do documento é abortado, **sem que o BC consiga determinar se o documento foi ou não transmitido**.

Nesse momento, temos no sistema um documento ainda em aberto, mas que pode já ter sido autorizado na SEFAZ, inclusive com um nº de nota fiscal que ainda poderá ser utilizado novamente dentro do BC, até mesmo por um outro documento completamente diferente.

Para evitar piores consequências, o usuário deve estar atento ao erro, desmarcar o flag “NF-e Automática” desse documento e realizar o registro sem transmissão, para que o status do documento possa ser consultado manualmente após o registro. Caso esse procedimento não seja seguido, o nº da nota fiscal pode acabar sendo reutilizado por outro documento diferente, criando vários problemas em cascata.

## Solução

Para garantir a integridade dos dados e evitar problemas mais graves, a etapa problemática foi alterada da seguinte forma:

1. No máximo 4 tentativas de consulta são realizadas;
2. Ao final da 4ª tentativa, se o BC ainda não conseguir obter o status do envio, o documento será forçadamente registrado.

Para alertar ao usuário da situação, a mensagem abaixo é exibida após a conclusão do registro:

> Atenção! Ação do usuário necessária.
>
> Estão ocorrendo atrasos nas respostas da SEFAZ e não foi possível obter o status do envio do documento.
>
> Para manter a integridade do sistema, o registro foi concluído e você deverá verificar o status da NF-e na página de documentos fiscais registrados.
>
> Último status conhecido: &lt;status&gt;

O usuário então deve abrir o **Documento Fiscal Registrado** e usar a ação **NF-e** → **Consultar Status**. O sistema consulta a NF-e na SEFAZ e atualiza os campos **Código Status** e **Resultado** do documento:

- **Autorizada** (**Código Status** 100 ou 150) — nada mais a fazer.
- **Ainda em processamento** (103 ou 105) — aguarde alguns minutos e use **Consultar Status** de novo.
- **Lote processado** (104) — o processamento terminou; use **Consultar Status** de novo para obter o resultado da NF-e (autorização ou rejeição).
- **Denegada** (110, 301, 302 ou 303) — ver [Denegação](Localization-for-Brazil_User-Manual_guia-fiscal-nfe-ciclo.md#denegação) em Ciclo de Vida da NF-e.
- **Rejeitada** — ver o parágrafo abaixo.

Se a NF-e for denegada durante o registro, a mensagem é outra: *"Essa NF-e foi denegada pela SEFAZ. Por favor, verifique a situação fiscal da sua empresa ou do seu cliente."*. O documento também é registrado.

Na situação em que ocorra um erro de validação, caberá análise do erro para determinar se o documento precisará ser estornado ou se poderá ser corrigido através da alteração de algum campo permitido (ver [Editar Campos do Documento Fiscal Registrado](Localization-for-Brazil_User-Manual_Editar-Campos-Documento-Fiscal-Registrado.md)) e transmitido de novo pela ação **NF-e** → **Transmitir**.
