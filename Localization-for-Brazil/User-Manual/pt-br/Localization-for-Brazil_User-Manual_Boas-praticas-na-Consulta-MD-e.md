# Boas práticas na Consulta MD-e

        
          
          
      
    
          
      
    
          
      
    
          
      
    
        
      

## Por que não se deve usar dois sistemas diferentes para consultar as notas fiscais da SEFAZ

Se sua empresa utiliza sistemas para consultar e armazenar notas fiscais eletrônicas (NF-e) diretamente da SEFAZ (Secretaria da Fazenda), é fundamental entender uma limitação técnica importante: **não é possível utilizar dois sistemas diferentes para fazer esse tipo de consulta ao mesmo tempo, utilizando o mesmo CNPJ**.

Neste artigo, explicamos de forma simples e direta por que isso ocorre e quais os riscos de não seguir essa recomendação.

### Como funciona a consulta de NF-e na SEFAZ

A Receita Federal, por meio de sua infraestrutura nacional, disponibiliza um serviço chamado **Distribuição de Documentos Fiscais Eletrônicos (DF-e)**. Esse serviço permite que empresas consultem e baixem documentos fiscais eletrônicos (como NF-e e NFC-e) nos quais tenham participação, seja como emitente, destinatário ou transportador.

O funcionamento desse serviço é baseado em um número sequencial chamado **NSU (Número Sequencial Único)**. A cada nova nota emitida com envolvimento da empresa, a SEFAZ incrementa esse número.

Para realizar a consulta, o sistema da empresa precisa informar qual foi o **último NSU recebido** (tag ultNSU). Assim, a SEFAZ sabe a partir de que ponto deve enviar novos documentos. Esse processo é contínuo e incremental.

### O que acontece ao utilizar dois sistemas simultaneamente

Quando dois sistemas diferentes tentam fazer a mesma consulta com o mesmo CNPJ, eles não compartilham o histórico do ultNSU utilizado. Cada sistema mantém seu próprio controle, o que pode gerar os seguintes problemas:

- **Perda de notas fiscais**: um sistema pode avançar o NSU, fazendo com que o outro "pule" documentos sem saber.
- **Notas duplicadas**: o mesmo documento pode ser processado mais de uma vez, causando inconsistência de dados.
- **Bloqueio temporário pela SEFAZ**: o uso incorreto ou excessivo do serviço pode levar a restrições de acesso por parte da Receita.
- **Erros nos sistemas**: relatórios, históricos e cadastros podem apresentar falhas devido à falta de sincronização.

### Qual é a solução?

A recomendação técnica e fiscal é clara: **utilize apenas um sistema para fazer a consulta direta ao web service da SEFAZ (****nfeDistDFeInteresse****)**.

Esse sistema deve ser responsável por manter o controle do ultNSU e armazenar os documentos fiscais recebidos. Caso outros sistemas da empresa também precisem acessar esses documentos, o ideal é que eles se integrem ao sistema principal, por meio de APIs, bancos de dados compartilhados ou exportações.

### Boas práticas recomendadas

- Defina um único sistema como fonte oficial de consulta de notas fiscais na SEFAZ.
- Compartilhe os documentos recebidos com outros sistemas da empresa por meios controlados e centralizados.
- Evite integrações paralelas que possam comprometer a integridade dos dados fiscais.

### Conclusão

A utilização correta do serviço de distribuição de documentos fiscais da SEFAZ é essencial para garantir a integridade, a segurança e a conformidade fiscal dos processos da empresa. Embora a ideia de dividir o trabalho entre múltiplos sistemas possa parecer vantajosa, ela traz riscos significativos.

Centralizar a consulta em um único sistema é a melhor maneira de evitar perdas de informações, duplicidades e problemas com o fisco.
