---
title: "Desvendando Vulnerabilidades: Entendendo IDOR, BAC e BOLA"
author: 0ta
date: 2024-08-09 01:00:00 -0300
categories: [Estudos]
tags: [Pentest, Relatorio, Estudo]
alt: "Vulnerabilidades"
---



# TL;DR

Este artigo busca desmistifica as diferenças entre Insecure Direct Object Reference (IDOR), Broken Access Control (BAC) e Broken Object Level Authorization (BOLA), todas destacadas no OWASP.

# Introdução 

![dc](/img/posts/dc-04.png)

## Qual a diferença entre IDOR, BAC e BOLA ? assunto complicado


Esse é um daqueles assuntos que parece ter sido criado só para nos dar dor de cabeça. Explicar a diferença entre IDOR, BAC e BOLA é como tentar montar um quebra-cabeça sem saber se todas as peças estão na caixa. As informações estão espalhadas por aí como se fossem Pokémons, e nós temos que sair de **New Bark** para coletar várias insígnias nos GYMs antes de finalmente participar da Liga Pokémon da segurança cibernética.

Para abordar essa questão, iniciaremos com uma introdução à OWASP (Open Worldwide Application Security Project), focando especificamente nas 10 vulnerabilidades mais críticas no cenário web. Em seguida, exploraremos o BAC (Broken Access Control), incluindo conceitos e exemplos de IDOR (Insecure Direct Object References) e BOLA (Broken Object Level Authorization).
# Open Worldwide Application Security Project (OWASP)

O Open Worldwide Application Security Project (OWASP) é uma organização sem fins lucrativos que fornece recursos para melhorar a segurança de aplicações. Um de seus principais recursos é o OWASP Top 10, uma lista das principais vulnerabilidades de segurança em aplicações web, que ajuda desenvolvedores a identificar e mitigar riscos críticos.


# OWASP Top 10

O OWASP Top 10 é atualizado geralmente a cada três ou quatro anos, refletindo as mudanças no cenário de segurança e as contribuições da comunidade global. Exemplos de edições incluem 2003, 2004, 2007, 2010, 2013, 2017 e 2021.

Além das aplicações web, a OWASP desenvolve outros projetos voltados para diferentes contextos de segurança:

- [API Security Top 10 2023](https://owasp.org/www-project-api-security/): Focado em vulnerabilidades específicas de APIs.
- [Mobile Top 10 2024](https://owasp.org/www-project-mobile-top-10/): Voltado para segurança em aplicações móveis.
- [OWASP Docker Top 10](https://github.com/OWASP/Docker-Security#how-to-build-pdf-version): Para segurança no uso de contêineres.

Essas listas são valiosas para desenvolvedores e profissionais de segurança, ajudando-os a entender e mitigar as principais ameaças em diferentes plataformas e tecnologias.

# Comparação OWASP Web Top 10: Edições 2013 e 2017

Na edição de 2007, o **A4: Insecure Direct Object Reference (IDOR)** foi uma das vulnerabilidades destacadas e continuou nas edições de 2010 e 2013. Em 2017, essa vulnerabilidade foi combinada com **A7: Missing Function Level Access Control**, resultando na nova categoria **A5:2017 Broken Access Control (BAC)**, que abrange uma gama mais ampla de problemas de controle de acesso, destacando a importância de proteger recursos e funções contra acessos não autorizados.


![dc](/img/posts/dc-02.png)

# Broken Access Control (BAC)

O **Broken Access Control** não é uma vulnerabilidade única, mas uma categoria abrangendo várias falhas de segurança relacionadas ao controle de acesso inadequado. Existem 34 Common Weakness Enumerations (CWEs) mapeadas para o [**A01:2021-Broken Access Control**](https://owasp.org/Top10/A01_2021-Broken_Access_Control/), incluindo:

- **Violação do princípio do menor privilégio**: Recursos devem ser restritos, mas estão disponíveis para qualquer pessoa.
- **Visualização ou edição de contas de terceiros**: Permitir acesso a contas de outros usuários usando identificadores únicos, caracterizando IDOR.
- **Elevação de privilégio**: Permitir que um usuário aja como outro ou que um usuário comum atue como administrador.
- **Manipulação de metadados**: Alteração de tokens de controle de acesso, como JWTs, ou modificação de cookies para elevar privilégios.

Essas vulnerabilidades destacam a importância de implementar controles de acesso robustos para proteger sistemas contra acessos não autorizados.

# Insecure Direct Object Reference (IDOR)

IDOR ocorre quando invasores acessam ou alteram objetos manipulando identificadores presentes em URLs ou parâmetros de um aplicativo web. Isso acontece quando o sistema falha em verificar se o usuário está autorizado a acessar determinados dados.

**Exemplo 01**

No aplicativo web **Toca do Mago**, ao interceptar uma requisição para modificar o idioma da plataforma, a seguinte URL foi observada:

```html
/directory/1337/updatePreferredLocale?newLocale=Pt-Br
```

O número `1337` representa o ID do usuário atualmente logado, neste caso, o usuário `Ainz`. O parâmetro `newLocale=` especifica o idioma a ser utilizado, e a requisição é feita usando o método `PUT`.

Ao alterar o ID de `1337` para `1338` na URL, tornou-se possível modificar o idioma que a plataforma apresenta para outro usuário.

**Exemplo 02**

No aplicativo **Grand Line Navigator**, que permite aos mugiwaras gerenciar suas missões e tesouros, cada usuário recebe um JWT (JSON Web Token) após o login. Este token contém informações codificadas sobre o usuário, incluindo seu identificador único.

Imagine que o token JWT de um usuário chamado `Usopp` contém o seguinte payload:

```json
{
  "userId": "101",
  "username": "Usopp",
  "role": "crew"
}
```

Um atacante, após interceptar uma requisição que contém esse JWT, modifica o campo `userId` para o identificador de um usuário diferente, `Luffy`, alterando o token para:

```json
{
  "userId": "102",
  "username": "Luffy",
  "role": "crew"
}
```

Com essa alteração, o atacante pode enviar o JWT modificado para o servidor do **Grand Line Navigator**, e devido à falta de validação adequada no backend, o sistema aceita o token como válido para `Luffy`. Assim, o atacante consegue acessar missões e tesouros que pertencem exclusivamente ao capitão `Luffy`, explorando uma vulnerabilidade de IDOR e deixando a nami P da vida, slk a mulher ia ficar estressada demais se roubassem os seus tesouros.

# Broken Object Level Authorization (BOLA)

BOLA ocorre em APIs quando os endpoints não aplicam corretamente funções ou permissões de usuário, permitindo acesso não autorizado a objetos. Isso geralmente leva à divulgação não autorizada de informações, modificação ou destruição de dados.

**Exemplo 01**

Um exemplo básico de vulnerabilidade de Autorização de Nível de Objeto Quebrada (BOLA) pode ser visto em um sistema de gerenciamento de usuários onde diferentes níveis de permissão são atribuídos. Imagine um aplicativo que tem uma interface para gerenciar usuários, permitindo que um usuário com permissão de Gestor (manager) veja uma lista de usuários com seus respectivos níveis de permissão.

Nesse cenário, ao acessar o menu de administradores, um gestor vê três usuários:

- **Batata** (Administrador)
- **Ciclano** (Gestor)
- **Judas** (Gestor)

A aplicação permite que o gestor edite as permissões de outros gestores, mas não as de administradores. No entanto, ao interceptar e modificar a requisição HTTP com uma ferramenta como o Burp Suite, o gestor pode alterar a própria permissão para "owner" (administrador) simplesmente modificando o parâmetro "role" no corpo da requisição.

```json
{ "fullName": "Ciclano", "role": "owner" }
```

Com essa alteração, o gestor consegue elevar suas próprias permissões para administrador, permitindo modificar e acessar dados de qualquer outro usuário, incluindo o administrador "Batata". Essa situação demonstra uma falha de BOLA, onde a aplicação não valida adequadamente as permissões do usuário antes de permitir a alteração de dados sensíveis, permitindo que um usuário escale privilégios de forma não autorizada.

**Exemplo 02**

Em um aplicativo chamado **Sabão Eater**, os usuários podem participar de diferentes grupos temáticos para interagir e compartilhar arquivos. No entanto, funcionalidades como alterar o nome ou a imagem do grupo são restritas aos administradores. Durante uma análise das requisições, foi descoberto que essas restrições podem ser contornadas.

Ao interceptar uma requisição `GET`, foi possível acessar informações detalhadas sobre os grupos, incluindo o `groupId` e o nome do grupo:

```html
GET /sabaoeater/v1/companies/groups/company/1234 HTTP/2
Host: api.sabaoeater.app
```

Utilizando a informação do `groupId`, foi possível enviar uma requisição `PATCH` para modificar indevidamente as configurações de um grupo. Por exemplo, o grupo "Jogatina de Yu-Gi-Oh" foi identificado com o `groupId` "1337". Ao alterar o `groupId` na requisição `PATCH`, foi possível modificar o nome do grupo para "Cassino do Magikarp" e trocar a imagem do logo:

```json
PATCH /sabaoeater/v1/companies/1234/group HTTP/2
Host: api.sabaoeater.app
Content-Type: application/json

{"groupId":1337,"groupName":"Cassino do Magikarp","logo":"http://example.com/image.png"}

```

Após a modificação, o grupo "Jogatina de Yu-Gi-Oh" foi renomeado e teve sua imagem alterada sem autorização, demonstrando uma vulnerabilidade de BOLA. Além disso, foi possível desativar grupos ao alterar o atributo `isActive` para `false`, removendo-os da visibilidade dos usuários. Esse processo também permitiu modificar múltiplos grupos simultaneamente ao manipular diversos IDs em uma única requisição `PATCH`.

# Broken Access Control (BAC) vs. Insecure Direct Object Reference (IDOR)

Conforme mencionado anteriormente, o Broken Access Control (BAC) não é uma vulnerabilidade única, mas sim uma categoria que abrange várias falhas de segurança relacionadas ao controle de acesso inadequado. De acordo com a [Top 10 Web Application Security Risks](https://owasp.org/www-project-top-ten/), o BAC é uma categoria ampla de vulnerabilidades.

- [**A01:2021-Broken Access Control**](https://owasp.org/Top10/A01_2021-Broken_Access_Control/) moves up from the fifth position; 94% of applications were tested for some form of broken access control. The 34 Common Weakness Enumerations (CWEs) mapped to Broken Access Control had more occurrences in applications than any other category.

Entre essas CWEs, destaca-se a [CWE-639](https://cwe.mitre.org/data/definitions/639.html), que inclui termos alternativos, como o Insecure Direct Object Reference (IDOR).

Portanto, é correto afirmar que a diferença entre Broken Access Control e Insecure Direct Object Reference reside em suas classificações: o BAC é uma categoria abrangente que engloba múltiplas vulnerabilidades relacionadas a falhas de controle de acesso, enquanto o IDOR é uma vulnerabilidade específica que pode ser encontrada dentro dessa categoria.

# Insecure Direct Object Reference (IDOR) vs. Broken Object Level Authorization (BOLA)

Compreender as diferenças entre Insecure Direct Object Reference (IDOR) e Broken Object Level Authorization (BOLA) é fundamental para diferenciar as abordagens de segurança necessárias em aplicações web tradicionais e APIs modernas. Embora ambas as vulnerabilidades permitam o acesso não autorizado a objetos, seus contextos e focos de segurança são distintos.

Tanto IDOR quanto BOLA são mencionados na [CWE-639](https://cwe.mitre.org/data/definitions/639.html), onde ambos são considerados alternativas para descrever a mesma fraqueza subjacente em um contexto geral de controle de acesso. 

![dc](/img/posts/dc-03.png)

No entanto, ao analisar a documentação da OWASP, podemos identificar divergências que indicam que IDOR e BOLA não são exatamente a mesma coisa.

IDOR ocorre quando não há proteção adequada para referências diretas a objetos, permitindo que usuários não autorizados manipulem identificadores para acessar dados restritos. Essa vulnerabilidade é comum em aplicações web tradicionais, onde parâmetros em URLs ou consultas são alterados para obter acesso a dados que não deveriam ser acessíveis ao usuário.

BOLA, por outro lado, destaca a importância de controles de acesso adequados em todas as operações realizadas em objetos. Isso é especialmente relevante em APIs, onde a autorização precisa ser verificada para cada ação em um objeto específico. BOLA requer a implementação de verificações robustas de autorização, garantindo que apenas usuários autorizados possam acessar ou manipular os dados, independentemente de como os objetos são referenciados ou manipulados

**Diferenças Principais**

- **Contexto e Aplicação**:
    
    - **IDOR** é mais prevalente em aplicações web tradicionais, focando em acessos diretos através de URLs ou parâmetros modificáveis.
    - **BOLA** é mais relevante em APIs, onde a autorização precisa ser gerida em um nível mais granular para cada operação de objeto.

- **Natureza da Vulnerabilidade**:
    
    - **IDOR** concentra-se na exploração de referências diretas a objetos, permitindo acesso baseado em identificadores previsíveis ou modificáveis.
    - **BOLA** cobre uma gama mais ampla de falhas na autorização em operações de objeto, independentemente de como os identificadores são fornecidos.

- **Segurança e Controle**:

    - **IDOR** exige a proteção contra acessos não autorizados diretos a objetos.
    - **BOLA** requer verificações abrangentes de permissões em todas as operações que envolvem objetos, assegurando que os usuários só possam acessar dados que lhes foram permitidos.

# Diferenças no Emprego do Termo "Objeto" em Relação a IDOR e BOLA

**Conceito de Objeto em IDOR**:

De acordo com o [Web Security Testing Guide (WSTG) da OWASP - Testing for Insecure Direct Object References](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References), o termo "acesso direto a objetos com base na entrada fornecida pelo usuário" refere-se a um cenário em que um aplicativo permite que os usuários acessem diretamente objetos usando identificadores fornecidos pelo usuário, como IDs em URLs ou parâmetros de consulta.


**Conceito de Objeto em BOLA**:

No [OWASP API Security Top 10 2023 - API1:2023 - Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/), o termo "autorização a nível de objeto" é frequentemente abordado. Ele refere-se a um controle de acesso mais granular, onde a autorização é verificada para operações em cada objeto individual, garantindo que o usuário tenha permissão específica para interagir com o objeto em questão.

# Diferença entre  IDOR, BAC e BOLA ?

Após explorar os conceitos de Insecure Direct Object Reference (IDOR), Broken Access Control (BAC) e Broken Object Level Authorization (BOLA), é essencial destacar as diferenças principais entre essas vulnerabilidades, garantindo uma compreensão clara para mitigar riscos de segurança em aplicações web e APIs.

**IDOR** é prevalente em aplicações web, focando em acessos diretos por identificadores manipuláveis, explora referências diretas a objetos sem validação adequada e requer proteção contra acessos diretos não autorizados.

**BAC** abrange tanto aplicações web quanto APIs, representando uma categoria de falhas de controle de acesso, engloba várias falhas de controle de acesso, incluindo IDOR e BOLA e necessita de controle abrangente de todas as formas de acesso e autorização.

**BOLA** é específico para APIs, focando na autorização a nível de objeto, trata de falhas de autorização em operações de objeto dentro de APIs e exige verificações detalhadas de autorização para cada operação em APIs.

Em conclusão, enquanto IDOR, BAC e BOLA compartilham algumas semelhanças na facilitação de acesso não autorizado a dados, suas aplicações, escopos e soluções de mitigação diferem significativamente. Compreender essas distinções é crucial para implementar medidas de segurança eficazes e proteger sistemas contra vulnerabilidades críticas.

# Referências


- [OWASP top 10 2017](https://github.com/OWASP/Top10/blob/master/2017/OWASP%20Top%2010-2017%20(en).pdf)
- [History of OWASP TOP 10](https://www.hahwul.com/cullinan/history-of-owasp-top-10/)
- [Release notes owasp 2017](https://owasp.org/www-project-top-ten/2017/Release_Notes)
- [Testing for Insecure Direct Object References](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
- [Insecure Direct Object Reference Prevention Cheat Sheet](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
- [CWE-285: Improper Authorization](https://cwe.mitre.org/data/definitions/285.html)
- [API Security Top 10 2023](https://owasp.org/www-project-api-security/)
- [CWEs vs OWASP top 10?](https://dev.to/caffiendkitten/cwes-vs-owasp-top-10-4imm)
- [API1:2023 Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [OWASP Access Control](https://owasp.org/www-community/Access_Control#:~:text=Access%20control%20governs%20decisions%20and,%3B%20essentially%2C%20what%20is%20allowed.)
- [PortSwigger - Insecure direct object references (IDOR)](https://portswigger.net/web-security/access-control/idor)
- [OWASP top 10 2023](https://owasp.org/Top10/en/)


