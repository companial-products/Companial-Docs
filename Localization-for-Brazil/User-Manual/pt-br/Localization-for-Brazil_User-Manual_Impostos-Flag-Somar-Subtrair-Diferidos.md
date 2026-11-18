# Impostos: flag para permitir somar/subtrair impostos diferidos da base de cálculo de outro

Impostos: flag para permitir somar/subtrair impostos diferidos da base de cálculo de outro

### Ajuste na base de cálculo para operações com Impostos Diferidos

Para atender à legislação vigente, o sistema realiza a exclusão do valor do ICMS da base de cálculo do PIS e da COFINS. No entanto, em operações com ICMS Diferido, o sistema subtraía da base apenas o valor do ICMS já com o diferimento aplicado, e não o valor cheio (destacado) do imposto. Isso resultava em uma base de cálculo para PIS e COFINS maior que a devida, exigindo contornos manuais que poderiam gerar inconsistências no livro fiscal e no XML.

Para corrigir este cenário e garantir a conformidade fiscal, foi criado um novo parâmetro através da flag **“Considerar Valor Desonerado”** na função Alterações Base disposta no ribbon do Cartão Jurisdição Imposto. Ao ativar este parâmetro, mesmo em uma operação com ICMS Diferido, o sistema passará a excluir o valor cheio do ICMS para formar a base de cálculo do PIS e da COFINS, garantindo o valor correto do tributo e a integridade do documento fiscal.

**1 de 3**
