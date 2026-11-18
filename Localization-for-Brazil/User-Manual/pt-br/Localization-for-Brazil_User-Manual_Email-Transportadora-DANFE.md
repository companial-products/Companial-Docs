# DocumentosFiscais: Envio de email à transportadora com informações do DANFE ao registrar fatura de venda

-Configuração do email para transportadora na config NFE
-Adicionado suporte ao email de transportadora no módulo Word Doc Email

### Localização

#### Configuração

Em primeiro lugar foi adicionada uma nova coluna à Page 52112529 “Transportadoras”, de nome "E-mail NF-e". O e-mail cadastrado neste campo será utilizado para direcionar o envio de e-mails à transportadora. Sem seu cadastro não será possível o envio de e-mails através da funcionalidade.

Além disso foram adicionados novos campos na página 52112626 Config. NF-e:

- O Campo Habilitado debaixo da Tab “E-mail Transportadora”. Serve para ativar o envio de e-mail para as transportadoras.
- No campo “Email CCO” serão adicionados e-mails para Cópia Oculta no Envio, de qualquer e-mail para as transportadoras.
- O Campo "Mensagem Adicional" será cadastrada uma mensagem que constará no final do e-mail enviado.

Será utilizado o mesmo e-mail Remetente da configuração geral para o envio à transportadora. Só é possível realizar o envio para a transportadora caso a configuração geral de envio de e-mails de documento esteja feita.

#### Funcionamento

Se estas configurações estiverem feitas no momento que uma Fatura de venda é registrada, o e-mail para a transportadora entrará na fila de E-mails a serem enviados. Como é possível ver na página 52112439 “E-mails de Documento” foi adicionada uma nova coluna “Categoria E-mail” para identificar se é um e-mail para a transportadora ou um e-mail default para o cliente. A partir desse momento o envio do e-mail à transportadora é independente do envio do E-mail para o cliente, podendo ser feitos em momentos separados à depender das pendências para seus envios.

### Word Doc Email

A Funcionalidade também é compatível com o nosso módulo Word Doc Email. Caso possua o App instalado haverá mais um campo na página "Config. NF-e". Layout E-mail Transportadora. Nele é possível selecionar qual o layout em Word Doc será utilizado para os e-mails para transportadora.

Não é possível selecionar layouts diferentes para transportadoras diferentes. No entanto é possível a inserção de campos dinâmicos no Layout, tendo assim uma única mensagem default, mas que constará com os dados do transportador correto, caso sejam usados mais de um transportador no sistema.

Para configurar um layout Dinâmico vá até a tela “Layouts Word de E-mail de Documento” selecione algum layout e vá em exportar. Acessando o Documento Word pesquise por “Painel de mapeamento XML”. Dentro do painel busque pela parte XML personalizada referente à "microsoft-dynamics-nav" lá estarão listados todos os campos dinâmicos que podem ser utilizados em seu Layout Word. Selecione o campo que deseja utilizar, clique com o botão direito e vá em “Inserir Controle de Conteúdo”→ “Rich Text” o campo dinâmico será então inserido em seu Layout.

Feito o Layout Customizado basta Importar ele para a linha desejada.
