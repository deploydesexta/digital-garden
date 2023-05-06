# API Security

# Introdução

Antes de entrarmos nas profundezas da segurança de APIs é necessário garantir que temos o conhecimento de alguns conceitos básicos. 

O primeiro de todos é o Transport Layer Security (TLS), que é uma forma de garantir que os dados trafegados pela internet não serão expostos por qualquer agente interceptador no meio do caminho, ou seja, garantimos integridade e privacidade nas mensagens trafegadas.

Temos também **HTTPS** que é HTTP over SSL/TLS. É a versão do HTTP que “encrypta” toda informação trafegada. Por padrão é utilizado a porta 443 e começa com "https://".

É importante saber que antes do TLS utilizávamos o Secure Sockets Layer (SSL), que foi descontinuado em 2015, em sua versão 3.0, por conta de vários problemas com vulnerabilidades. Hoje em dia a comunidade ainda utiliza o termo SSL, mas se referindo ao TLS que é a continuação do SSL e se encontra na versão 1.3.

# Transport Layer Secure (TLS)

## Funcionamento

Tudo começa com uma conexão TCP sendo estabelecida entre o cliente e o servidor. Com a conexão de pé o cliente envia uma mensagem "Hello" para o servidor informando quais tipos de certificado ele suporta, quais versões de TLS suporta e os tipos de compressão que entende.

Em seguida, o servidor lê a primeira mensagem e devolve o certificado pertinente que o cliente usará para se comunicar. O cliente valida o certificado e assegura-se de sua autenticidade e que pertence de fato ao servidor em que deseja se comunicar.

Dando tudo certo começa a parte do *handshake* entre o cliente e o servidor e as chaves são trocadas entre si. Com o *handshake* concluído, a troca de mensagem pode ser iniciada com a garantia da segurança.

## Como funciona a verificação do certificado?

Assim que o cliente obtém o certificado do servidor, uma série de instruções são realizadas para validar o certificado recebido. Primeiro, o cliente verifica que o certificado não está expirado e que seu domínio ou IP são os mesmos do servidor. Depois, o cliente verifica se o certificado está corretamente assinado e de acordo com a assinatura que a certificadora autorizou.

É improvável que o certificado do server seja assinado diretamente por uma Root CA que o cliente confia. Entretanto, o cliente pode confiar nos certificados emitidos por uma CA intermediária contanto que a cadeia de certificados eventualmente leve para uma Root CA e que esteja corretamente assinado de acordo com a chave pública.

![](/assets/microservices-security/cadeia-de-autoridades.png)

## Entendendo melhor os certificados

### O que são os SSL/TLS Server Certificates?

Esses certificados são pequenos arquivos que ficam no servidor com informações do mesmo. Essas informações contidas no certificado são verificada por uma cadeia de ”Certificate Authorities” que faz a ponte entre os certificados que o cliente confia e os que o server confia.

### Certificado Privado (self-signed)

Um certificado privado, ou *self-signed*, é facilmente gerado usando ferramentas como OpenSSL. Esses certificados precisam conter uma chave pública e uma privada o qual os clientes e o servidor usaram para comunicação criptografada.

Esse tipo de certificado é normalmente usado em redes privadas para estabelecer comunicação segura entre os serviços, o que chamamos de Mutual TLS (mTLS).

### Certificados Verificados (Trusted Certificates)

Se tratando de internet pública os certificados *self-signed* não funcionam por não ser possível garantir que o certificado pertence a quem realmente diz que é. Nesse cenário precisamos de certificados emitidos por autoridades certificadores que são confiáveis como órgãos governamentais ou corporações.

Esses certificados funcionam da mesma forma como estamos acostumados na vida real quando solicitam algum documento com foto e a pessoa simplesmente confia que a certificadora do documento verificou sua identidade e não é algo forjado. Esse tipo de verificação é conhecido como "Cadeia de confiança” (*chain of trust*) e cada cliente (sistema operacional, navegador, etc) tem uma lista de emissores de certificados as quais confiam e são conhecidos como "Root Certificate Authorities” (Autoridades de Certificação Superiores).

Dois exemplos de serviços que facilitam muito a vida na gestão de certificados são: *Amazon Certificate Manager,* *Let's encrypt e CertiSign*, que são Autoridades de Certificação Intermediárias. 

# Autenticação

Autenticação é o processo de provar quem você realmente é. Na vida real fazemos isso frequentemente exibindo um documento com foto. Na internet essa prova é normalmente baseada em um *login* onde o usuário digita suas credenciais como o e-mail e a senha.

> Quem é você?
> 

## HTTP Basic Authentication

Essa é a forma mais simples de autenticação. É a união do usuário com a senha encodados em base64.

```sql
Authorization: Basic base64(username:password)
```

# Autorização

Autorização é o processo de verificar quais permissões o usuário tem e o que ele pode realizar.

> O que você pode fazer?
> 

## Autorização baseado em Roles

## Autorização baseado em Permissões

# Production-ready Architecture

Um dos maiores desafios da arquitetura de micro-serviços em produção é manter uma rede integra e segura. Nessa arquitetura é comum crescer a ponto de ter milhares de serviços e fazer a todo momento os desenvolvedores se preocuparem com segurança é responsabilidade demais, é mais interessante criar componentes e abstrações para garantir a segurança mantendo os desenvolvedores focados em funcionalidades de negócio. O mercado tem seguido dois padrões arquiteturais para implementar segurança: API Gateway e Service Mesh.

O API Gateway é o que chamamos de *edge security* (lida com o tráfego norte/sul), enquanto o Service Mesh lida com o tráfego leste/oeste.

![](/assets/microservices-security/orientacao-de-trafego.webp)

## Segurança do trafego north/south com API Gateway

O API Gateway é a primeira barreira de entrada entre seus serviços e o usuário final. Pode ser um simples *proxy* até uma camada mais completa que adiciona segurança, QoS, *rate-limiting*, *IPs blacklist*, monitoria, resiliência, etc. Na Netflix todos os micro-serviços são  expostos via Zuul API Gateway. Existem vários API Gateways no mercado, tais como: Google Apigee, Netflix Zuul, AWS Api Gateway, Kong API Gateway, Tyk, etc.

### Desacoplando segurança dos micro-serviços

Um dos benefícios pelos quais optamos por utilizar essa arquitetura é pelo fato de isolar bem as responsabilidades tornando cada serviço simples e com uma única responsabilidade. Acoplar uma camada de segurança dentro de cada micro-serviço pode adicionar uma complexidade a qual não desejamos, como:

- Extrair o header, cookie ou parâmetro de autenticação para cada requisição;
- Conectar ao serviço de autenticação para autenticar e autorizar;
- Lidar com erros relacionados a autenticação de maneira amigável;

Para só depois começarmos a pensar nas funcionalidades de negócio. Sem contar que o serviço de autenticação precisa ser muito robusto e com uma disponibilidade impecável, pois uma pequena falha impossibilitaria a comunicação entre todos os micro-serviços.

### Segurança nas pontas

Uma alternativa segura e que desacopla os serviços nos dando agilidade para desenvolver nossos micro-serviços é transferir a responsabilidade de autenticação e autorização para a ponta no API Gateway.

## Segurança do trafego east/west com certificados

*Edge security* com API Gateways lida com autenticação e autorização em nome do usuário final através de um usuário ou um outro micro-serviço. Além dessa camada, precisamos também nos preocupar com a camada interna da rede virtual, que é a conversa segura entre os micro-serviços usando a técnica mais utilizada conhecida como mTLS.

### Por que usar mTLS?

TLS protege a comunicação entre duas partes garantindo confidencialidade e integridade. Além disso, utilizar TLS permite identificar o server o qual queremos nos conectar.

### **Como construir uma confiança entre o cliente e o servidor?**

Para garantir que um certificado é realmente válido, é necessário que o mesmo seja assinado por um terceiro o qual ambas as partes confiam conhecido como Certificate Authority (CA). Todos que quiserem utilizar um certificado para se comunicar precisa ter um certificado assinado por essa CA.

### Mutual TLS para identificar o cliente e o servidor

O TLS em si é uma via de mão única, ele ajuda o cliente a identificar o servidor o qual está chamando, mas não o contrário. Two-way TLS, ou Mutual TLS, preenche esse *gap* ajudando ao servidor identificar o cliente o qual o chama.

![](/assets/microservices-security/mtls-example.png)

### Desafios ao gerenciar as keys no mTLS

Uma das coisas mais difíceis é o gerenciamento de chaves, envolve: bootstrap com confiança, provisionar chaves e certificados para micro-serviços, revogar chave, rotacionar chave e monitorar uso das chaves.

**Provisionamento de chaves e bootstrap com confiança**

Manualmente não é possível criar o JKS ou conjunto de chave privada e certificado público. O ideal é que seja feito ou na pipeline do CI/CD ou no startup da aplicação.

**Provisionamento de chaves na Netflix**

Netflix tem milhares de micro-services e toda a comunicação é feita com mTLS. Netflix utiliza o Lemur, um software open source de gerenciamento de certificado que atua como um intermediário entre o deploy de um serviço e um CA. Basicamente durante do processo de CI/CD é injetado umas credenciais (JWT) de longa duração na imagem da aplicação e no startup da aplicação essas credenciais permitem chamar o Lemur API, o qual gera o certificado público que o micro-service irá utilizar.

> Utiliza-se JWT para a credencial de longa duração, que normalmente é um processo mais moroso para se construir e fica em um SECRETS onde dificulta o vazamento. Para o certificado de curta duração a informação é mantida em memória.
> 

### Revogar um certificado

A revogação de um certificado normalmente acontece por duas razões: o certificado privado da CA usado para assinar certificado foi comprometido, ou, a chave privada da aplicação foi comprometida.

****CERTIFICATE REVOCATION LISTS****

Essa é uma das maneiras em que no processo de TLS é feito uma verificação da CA para obter todas as cadeias de certificado que foram revogados. É um processo em que no próprio certificado temos uma URL o qual a CA que o assinou expõe para que os clientes possam fazer download de toda a cadeia de certificado revogados. Pelo fato de que pode escalar a megabytes de download e frequentemente chamado, é comum que os clientes automaticamente façam cache de CRL por CA, por isso não é tão confiável em ambientes que precisam estar 100% seguros a todo tempo.

****ONLINE CERTIFICATE STATUS PROTOCOL****

Diferente do CRL essa forma conhecida como OCSP funciona como um endpoint onde apenas o certificado atual é verificado na CA. Apesar de ser uma maneira mais simples e mais rápida, ainda gera uma grande carga no emissor do certificado e também é orientado a cache. Existem navegadores, como o Chrome, que optaram por NÃO suportar essa abordagem pela sobrecarga de cache que gera.

****SHORT-LIVED CERTIFICATES****

O consenso atual em termos de revogar certificados é de usar certificados de curta duração, para evitar ter que lidar com uma expiração forçada. Dessa forma um certificado comprometido é rapidamente renovado em um próximo deploy. Normalmente quando fala-se em curta duração estamos falando de no máximo quatro dias.

### Rotação de chaves

Toda as chaves provisionadas nos micro-serviços devem ser rotacionadas de preferência antes de sua expiração. Não são todas as empresas que se preocupam com a rotação antecipada e muitas esperam receber erro de certificado expirado para começar a agir. Algumas companhias rotacionam mensalmente, a netflix rotaciona há cada quatro minutos em cada deployment, cada um tem uma regra.

É importante lembrar que apenas os certificados clientes que são rotacionados. O certificado raiz da CA interna não precisa ser rotacionado, pois é algo bem custoso de se fazer.

## Segurança do trafego east/west com JWT

De fato, a maneira mais comum de segurança na comunicação entre micro-serviços é o mTLS, mas também é possível utilizar JWT para propagar uma coleção de claims ou atributos de um serviço para outro. É possível utilizar o JWT para carregar a identidade do micro-serviço cliente, ou do usuário final, ou do sistema o qual iniciou a requisição.

### Casos de uso do JWT

JWT resolve dois principais problemas relacionados a segurança de micro-serviços: seguridade na comunicação entre micro-serviços e trafegar o contexto do usuário final. 

****Compartilhando o contexto do usuário com um JWT compartilhado****

Quando a identidade do micro-serviço que está chamando não é relevante, um simples JWT com o contexto do usuário é útil e não é necessário mTLS.

1. O usuário final inicia um fluxo com uma request.
2. O *edge gateway* autentica o usuário. O *edge gateway* depois intercepta as chamadas e extrai o token de autenticação da requisição. O token é submetido a um STS (Security Token Service) para que o mesmo o valide.
3. Depois de validar o token, o *edge gateway* cria um novo JWT com o conteúdo do antigo, mas assinado assinado por ele mesmo.
4. O *API Gateway* faz o forward da chamada junto do novo token (normalmente no header `Authorization`) para o micro-serviço destino. O micro-serviço *target* valida a assinatura do JWT para garantir que foi assinado por um STS em que confia e também valida o campo `aud` que veio no JWT para garantir que é para ele mesmo. Como esse é um JWT compartilhado, o campo de audiência normalmente vem com um wildcard para que todos os micro-serviços da rede possam aceitar: *`*.ecomm.com`*
5. Quando esse micro-serviço precisa chamar outro, é o mesmo JWT que será usado e o novo serviço de destino fará as mesmas validações que o anterior (assinatura e audiência).

****Compartilhando o contexto do usuário com um JWT para cada interação****

É uma pequena variação da solução anterior, e ainda é para situação onde a identidade do micro-serviço que está chamando não é relevante.

O que muda nessa variação é que o *edge gateway* gera um JWT com uma audiência especifica e esse micro-serviço precisa falar com o STS e gerar um novo JWT com a audiência ajustada antes de chamar um terceiro micro-serviço.

### Gerando o JWT próprio

## Segurança do tráfego reativo

### Utilizando TLS no Kafka para proteger mensagens em transito

### Utilizando mTLS para autenticar os microserviços conectados ao Kafka

### Limitar acesso aos tópicos do Kafka via Access Control Lists

# Mitigando DDoS

# Higienizando os dados (Sanitizing)

# Referências

- [https://developer.okta.com/books/api-security/](https://developer.okta.com/books/api-security/)
- Microservices Security in Action
- [https://www.baeldung.com/x-509-authentication-in-spring-security](https://www.baeldung.com/x-509-authentication-in-spring-security)
- [https://kofo.dev/how-to-mtls-in-golang](https://kofo.dev/how-to-mtls-in-golang)
- [https://smallstep.com/hello-mtls/doc/client/go](https://smallstep.com/hello-mtls/doc/client/go)
- [Why Is It Important? East-West and North-South Traffic Security - SOCRadar® Cyber Intelligence Inc.](https://socradar.io/east-west-and-north-south-traffic-why-is-it-security-important/)
- [Tip of the Day: Demystifying Software Defined Networking Terms - The Cloud Compass: SDN Data Flows | Microsoft Docs](https://docs.microsoft.com/pt-br/archive/blogs/tip_of_the_day/tip-of-the-day-demystifying-software-defined-networking-terms-the-cloud-compass-sdn-data-flows)