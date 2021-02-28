Apêndice A: Segurança 

## Práticas de Segurança

Código seguro deveria ser uma regra em qualquer design de aplicação. Em alguns ramos, são tão importantes quanto qualquer regra de qualidade que se possa ter. Para microsserviços não é diferente, e aqui não existe nenhum segredo, é a mesma regra para qualquer desenho de aplicação: no mínimo se deve olhar para o [Top 10 do OWASP](https://owasp.org/www-project-top-ten/) e seguir essas recomendações no seu projeto. O mínimo que o seu desenvolvedor precisa saber é sobre essas vulnerabilidades.

Sério mesmo, não há nada novo aqui, e qualquer um que esteja tentando convencer você sobre um "Cross Microservice Injection" ou algo do tipo está tentando te enrolar. 

No caso de Microsserviços, a abordagem que muda é em relação a Autorização e Autenticação, e vamos discutir isso mais à frente.

Uma dica importante, sempre que se olha em termos de código seguro, é colocar algo na sua esteira de CI/CD e fazer a validação do seu código para procurar isso, e não apenas verificar se o seu código traz vulnerabilidades. É interessante olhar se a bibliotecas de terceiros que você esteja usando possam estar causando algum problema no seu ambiente.

E ferramentas para isso há várias, entre as quais temos o [Fortify](https://www.microfocus.com/en-us/solutions/application-security), [Snyk](https://snyk.io/), [JFrog Xray](https://jfrog.com/xray/). Porque às vezes uma dependência desatualizada pode colocar seu serviço em posição de risco, então a melhor prática no código e uma ferramenta para ajudar a apontar onde melhorar formam um time imbatível.

Outra prática que eu vejo em algumas empresas, principalmente quando os serviços estão expostos apenas internamente e não para fora, é não usar o HTTPS, ou melhor, usar um TLS (Transport Layer Security). E para que você precisa disso - privacidade, integridade e identificação?

Quando estamos falando de microsserviços, um cenário que vai acabar sempre existindo é termos que falar com servidores de autorização, e podemos estar falando de um API Key, ou de um "client secret", ou até mesmo de credenciais para uma autenticação básica. Então a primeira regra básica é: não deixe essas chaves no seu repositório de fonte, esses caras precisam ser variáveis de ambientes ou chaves de configuração externa, e elas devem estar sempre encriptadas.

Como estamos falando de containers, as práticas valem também para lá; nunca rode seu container como "root". Você precisa assumir a premissa de que seu sistema nunca é 100% seguro, alguém vai conseguir explorar algo. Então você não pode só prevenir, você precisa detectar e reagir a isso.

A chave é seguir ao menos cinco pilares:

- Seguro por desenho.
- Vasculhe suas dependências.
- Use sempre HTTPS.
- Use Tokens de Acesso e Identidade.
- Proteja e Encripte seus segredos.

#### Soluções para Autenticação e Autorização

Para o mundo de microsserviços, o principal ponto é verificar quem você é (Autenticação) e aquilo que você pode fazer (Autorização). Dentro da arquitetura de microsserviços, você vai estar espalhado em muitos serviços pela rede e terá que lidar com alguns problemas em relação a como resolver isso.

Autenticação e Autorização precisam ser resolvidos em cada um dos microsserviços, e parte dessa lógica global vai ter que ser replicada em todas os serviços. Nesse caso, um jeito para resolver isso é criar bibliotecas para padronizar essa implementação, só que isso vai fazer com que você perca um pouco da flexibilidade em termos de quais tecnologias usar, pois a linguagem ou framework precisa suportar essa biblioteca padrão.

Outro ponto em que o uso da biblioteca ajuda é em não quebrar o princípio da responsabilidade única, já que o serviço deveria se preocupar apenas com a lógica de negócio.

E outro ponto que é necessário ser analisado é que os microsserviços devem ser *stateless*, então é necessário usar soluções que consigam manter isso.

Podemos abordar a Autorização e Autenticação pelo modelo de sessão distribuída, usando ferramentas para você armazenar essa sessão, e onde você pode abordar, manter a sessão das seguintes maneiras:

**Sticky Session** - A ideia aqui é usar o load balancer e manter o usuário sempre no mesmo servidor em que veio o request. Só que esse cara vai fazer você só conseguir expandir horizontalmente.

**Replicação de Sessão** - Ou seja, toda instância salva a sessão e sincroniza através da rede. Só que aqui vai lhe causar um "overhead" de rede. Quanto mais instâncias, mais terá que replicar e lidar com a latência disso.

**Sessão Centralizada** - Isso significa que os dados podem ser recuperados em um repositório compartilhado. Em vários cenários, esse é um ótimo desenho, porque pode dar alto desempenho para as aplicações, e você deixa o status do login escondido dentro dessa sessão. Mas claro que existe a desvantagem de você precisar criar mecanismos para proteger essa sessão e replicar entre as aplicações, o que pode também adicionar latências na sua rede.

Mas quando estamos nesse cenário de microsserviços, a recomendação passa a ser o uso de Tokens, cuja maior diferença para o modelo de sessão descrito acima é que deixamos de ter algo centralizado em um servidor e passamos a responsabilidade para o próprio usuário.

O Token vai ter a informação de identificação de usuário, e toda vez que chega ao servidor, podemos validar no server a identidade e a autorização. O token é encriptado e pode seguir um padrão como o [JWT](https://jwt.io/).

Usando token conseguimos delegar a responsabilide do estado do usuário, para algum processo que possa exigir a validade do mesmo. Habilitamos vários tipos de validações de segurança que podem ser colocadas na malha (Service Mesh) ou no seu gateway de entrada e retirar essas responsabilidades dos serviços e aplicações, e mesmo assim ainda continuar garantindo a segurança.

Com o uso do JWT, você passa a ter um "client token": você vai passar a algum servidor para que ele possa fazer a validação/criação do mesmo. 

E quando se fala em Tokens, a chave é não querer reinventar a roda, e sim usar aquilo que já está consolidado. Aqui entra o OpenId e o OAuth/OAuth2. O Oauth 2 é praticamente o padrão mais utilizado para autenticação.

O padrão de OpenID é aquele usado quando você pode se conectar ou usar o token para se logar em vários sites ou serviços. Mas no seu padrão local, a recomendação é o OAuth, que estabelece um protocolo para que você tenha acesso aos recursos de que você precisa, e ele trabalha com quatro papéis:

**Resource Owner** - Este é o papel que controla os acessos aos recursos.

**Resource Server** - É onde ficam os serviços a serem acessados, ou seja, aqui é onde estão as suas API's, aplicações e etc.

**Client** - É quem faz a solicitação ao Resource Owner do recurso que ele precisa consumir.

**Authorization Server** - Quem gera os tokens de acesso, permite que o Client chegue aos recursos que foram permitidos, com o nível de acesso definido.

E como funciona isso? Basicamente o "client" solicita ao Resource Owner acesso ao recurso, e este, quando autoriza, envia para o "Client" o "authorization grant", que é a credencial que representa que o Resource Owner autorizou a passagem. Então o "Client" vai solicitar ao Authoration Server um Token de acesso; tudo sendo válido, o Client recebe o seu token de acesso, que vai ser repassado para o Resource Server para que ele possa consumir aquilo que foi solicitado.

Existem quatro tipos fluxos para obtermos o token de acesso ao padrão OAuth. Temos o Authorization Code, que é o tipo mais comum; o Implicit, que é muito utlizado por aplicações SPA; o Resource Owner, que estabelece a confiança entre as aplicações; e o Client Credentials, que é usado para falar de um serviço para outro.

Tirando o que toca a Autenticação e Autorização, que precisam de um cuidado à parte, segurança de microsserviços é como a segurança de qualquer aplicação, e precisamos estar atentos a isso.
