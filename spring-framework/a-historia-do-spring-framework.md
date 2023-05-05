# Spring Framework

## Introdução

Henry Ford certa vez disse: "Se existe um segredo para o sucesso, ele consiste na capacidade de entender o ponto do outro e enxergar não só com seus olhos, mas também com os dele*".* Eu sei de uma empresa que talvez não conhecia essa frase ou decidiu apenas ignorar e pagar com as consequências: Oi, Sun Microsystems! Não fez sentido? Vou lhe explicar.

## Um passeio no passado...

Era 2002, *EJB* (*Enterprise Java Beans*) era o padrão de desenvolvimento imposto pela Sun Microsystems, e um pequeno grande homem, Rod Johnson, insatisfeito, dá o primeiro passo que veio a desencadear uma nova de desenvolvimento de *Software* e publica um livro conhecido como *Expert One-To-One J2EE Design and Development*.

O que contém neste livro? Os melhores e mais convincentes argumentos de que *EJB* é sim de fato excelente! MAS para um problema muito peculiar que é a necessidade de desenvolver aplicações distribuídas utilizando RMI. Todo o resto seria usar umma bazooka para matar uma formiga e trazer dor de cabeça para o dia-a-dia do dev.

Um ano depois, em 2003, Rod começa o desenvolvimento do Spring Framework. Em março de 2004, o mundo diz olá ao Spring Framework. Ainda em 2004, Rod eleva a popularidade do *framework* com o lançamento de seu livro *Expert One-To-One J2EE Development Without EJB,* que exibe ao mundo como criar aplicações java sem utilizar *EJB*. É um guia intensivo que prove boas práticas para criar de maneira simples e efetiva aplicações usando JavaServer Pages, Servlets e *frameworks* mais leves que emergiam na época como o próprio Spring e o Hibernate.

**Recapitulando os problemas da época:**

- Não existia GitHub, git, a ferramenta era CVS;
- Não existia Generics, Annotations, vieram no Java 5;
- EJB foi concebido através de um comitê e não da observação dos problemas da época;
- Sua evolução era um processo lento por ter muitos interesses envolvidos;

**O que brilhava os olhos dos desenvolvedores na época:**

- Gerenciamento de transações;
Abstração que entrega flexibilidade (JTA, JDBC, Hibernate, JPA, ou JDO) e controle   transacional declarativo.
- Separar código de infraestrutura;
Capacidade de rodar a aplicação em qualquer *server*, o que facilita os testes unitários;
- Soluções leves, modulares e flexíveis.
*IoC container* ultra leve e intuitivo. Modular a ponto de permitir usar apenas o *bean container* com Struts em cima ou Hibernate como implementação para a camada JDBC.

**Além,** **o que brilha nossos olhos hoje?**

- Maturidade e comunidade
Documentação
- Suporte ao gerenciamento de configurações por ambiente (Properties e Implementações)
- Suporte a logs
- Suporte a banco de dados
- Suporte a múltiplos tipos de retorno (JSON, XML, TEXT)
- Suporte a outras linguagens (Java, Kotlin, Groovy)

**E qual seria a solução?**

## Apresento: Spring Framework

O Framework é baseado em Injeção de Dependência que é uma especialização da Inversão de Controle. No Spring, temos um container de injeção de dependência que se encarrega por injeta-las na classe.

**Da** **Comunidade para a Comunidade!**

PicoContainer surgiu com a idéia de injetar dependências via construtor;

Rails foi precursores em criar uma convenção over configuração - aceitar padrões default ao invés de sempre re-inventar a roda ([Spring Roo](https://projects.spring.io/spring-roo/));

Comunidade no StackOverflow é gigante. Há grande chances do problema ter uma issue bem detalhada lá.

**Design Philosophy**

- Prover escolhas em primeiro lugar;
    
    Além disso, Spring permite postergar decisões impactantes como trocar o banco de dados a qualquer momento SEM alterar código - Abstrações.
    
- Contempla diversas perspectivas;
Spring abraça a flexibilidade e não é opinativo sobre o que usar nas aplicações ou como as coisas devem ser feitas.
- Foco na retrocompatibilidade;
- Se preocupa com o design das APIs;
O time coloca bastante esforço para desenhar APIs que são intuitivas e vão durar muitos anos.
- Barra elevada na qualidade de código.
O código do Spring é um bom professor mas dificilmente precisamos de algo parecido, foquemos no negócio.

## Injeção de Dependencia (DI)

Quando a onda dos *containers* ficaram populares, o *marketing* era que implementavam a tal da Inversão de Controle (IoC). Todavia, o termo é bem genérico e confundiam os desenvolvedores, não ficava claro o que, como, ou onde, IoC era implementada.

No começo de 2004, Martin Fowler trouxe justamente esse questionamento e sugeriu que deixassem de usar o termo "Inversão de Controle" e passassem a usar "Injeção de Dependência" no lugar. É um termo mais explicativo dado que a inversão é na maneira como é feita uma busca na implementação de uma abstração.

### **O IoC Container**

O BeanFactory e ApplicationContext

Beans

### Dependencias

**Formas de injeção de dependencia**

- Injeção via Construtor (PicoContainer & Spring);
- Injeção via Setter (Spring);
- Injeção via Propriedade (Spring);

**Injeção via Construtor**

- Considerado boas práticas pelo time do Spring;
- Objetos imutáveis;
- Assegura dependencias não nulas;
- Previne dependencia ciclica;
- Indicativo claro de *code smells* ao ver muitos argumentos.

**Injeção via Setter**

- Dependencia opcional;
- Postergar injeção.

**Bean Scopes**

- Singleton
- Prototype
- Request
- Session

**Anotações auxiliares**

`@Component`, `@Service`, `@Repository`,  `@Controller`, `@RestController`

`@Qualifier`, `@Primary`, `@Bean`

`@WebMvcTest`, `@DataJpaTest`, `@JsonTest`

### **Proxying Mechanisms**

Spring AOP faz uso de JDK dynamic proxies ou CGLIB. A diferença é que JDK dynamic proxies é a implementação padrão da JDK, já a CGLIB é uma implementação open-source.

A regra base é que se uma classe implementa alguma interface, é utilizado JDK dynamic proxies. Do contrário, utiliza-se CGLIB. Devemos nos atentar que isso implica em "proxear" apenas chamadas externas. Quando já passou pelo proxy e o scope da função é interna no Objeto, não é passado pelo proxy novamente.

Em contra partida, se deseja ter um full proxy mesmo implementando alguma interface, é necessário configurar *proxy-target-class* para true.

## Gerenciamento de transações (Transaction Management)

**Desmistificando os termos**

Hibernate surgiu em meados de 2003 e veio com uma proposta de estruturação de abstração de SQL e idealização de ORM. Com o passar do tempo, a JCP(Java Community Process) coletou muito input da Hibernate e outros fornecedores e idealizou a JSR 220 (JPA 1.0). Logo em seguida a JSR 317 conhecida como JPA 2.0.

"JPA is the Art, Hibernate is the artist"

***Global transactions***

*Global transactions* são gerenciadas pelo servidor de aplicações, utilizando Java Transaction API (JTA) e EJB Container-Managed Transactions (CMT).

***Local transactions***

*Local transactions* são gerenciadas localmente por recurso, normalmente associadas a uma conexão JDBC, por exemplo.

### **Spring Consistent Programming Model**

Um dos principais motivos pelo qual o Spring foi amplamente adotado é o fato de que, sem a necessidade de alterar seu código, ele suporta diferentes *transaction* *APIs*, como*:* Java Transaction API (JTA), JDBC, Hibernate, e o Java Persistence API (JPA).

Isso é possível pois o Spring Framework fornece abstrações para gerenciar as transações.

**Programmatic Transaction Model**

Como o nome já diz, o controle transacional é feito programaticamente pelo desenvolvedor. O Spring Framework disponibiliza duas formas:

- *TransactionTemplate* ou a versão reativa *TransactionalOperator*;
É uma interface *helper* que utiliza o gerenciador e seu fim é diminuir ainda mais o boilerplate.
- Uma uso vanilla do gerenciador: TransactionManager *(*PlatformTransactionManager / ReactiveTransactionManager*)*;
É o gerenciador propriamente dito. Controla os comportamentos de timeout, propagation, synchronization, isolation, etc..

**Declarative Transaction Model**

O modo mais utilizado e recomendado pelos advocates do Spring, Declarative Transaction, usa e abusa de Aspect-Oriented Programming (AOP). A principal vantagem é ser não invasiva e permitir isolamento da regra de negócio dos frameworks.

**Escolhendo qual *Transaction Model* utilizar**

Quando a quantidade de transações é bem reduzida, ter um controle manual programático pode ser uma boa ideia. Quando usamos *Clean Arch* também sentimos a necessidade de ter um *gateway* cujo papel é abrir executar uma *Unit of Work* - mais de uma chamada a *gateway* - dentro de uma transação.

Quando a aplicação começa a escalar, é mais viável usar a forma declarativa mesmo `@Transactional` no Spring não é tão custoso como os do EJB, por exemplo.

**Propagation**

- Required
Caso já existe uma transação prévia ela será utilizada, caso não, será criada.
- Mandatory
Diferente do Required, essa não cria transação.
- Requires New
Suspende a Transação anterior e inicia uma nova Transação isolada. De certa forma quebra o ACID mas pode ser útil para registros de auditoria, por exemplo.
- Supports
Caso exista, utiliza a Transação e, caso não, não faz diferença.
- Not Supported
Suspende a Transação atual (caso exista) e invoca é o método sem alguma transação.
- Never
Próximo ao Not Supported mas é mais radical. Caso tenha transação, uma exception é lançada.

**Isolation**

- READ_COMMITTED
- READ_UNCOMMITTED
- REPEATABLE_READ
- SERIALIZABLE
- DEFAULT

**Transaction Design Patterns**

Client Owner Transaction Design Pattern

Domain Service Owner Transaction Design Pattern

Server Delegate Owner Transaction Design Pattern

## The Web

Spring Framework suporta diferentes bibliotecas para camada de apresentação. Além de sua própria implementação, fornece suporte a outras tecnologias como Struts e JSF.

Front Controller Pattern

### **Web on Servlet Stack**

Spring Web MVC é o *framework web* padrão que a Spring provê para os desenvolvedores e foi incluída bem no comecinho do Spring Framework.

DispatcherServlet

**View Technologies**

- Thymeleaf
- FreeMarker
- JSP and JSTL
- PDF and Excel
- Jackson
- XML Marshalling

### **Web on Reactive Stack**

Paralelo ao Web MVC, a partir do Spring Framework 5.0 foi introduzido a *stack* reativa cuja nomenclatura é "Spring WebFlux".

DispatcherHandler

**View Technologies**

- Thymeleaf
- FreeMarker
- Script Views
JSR-223 Java Scripting Engine (Nashorn)
- JSON and XML

### **Web**

**Annotated Controllers**

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

**Filters**

- [Form Data](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-http-put)
- [Forwarded Headers](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-forwarded-headers)
- [Shallow ETag](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-shallow-etag)
- [CORS](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#filters-cors)

**CORS**

- Annotation based
- Global Config
- Filter

Data Binding

**HTTP Caching**

- Cache Control
- ETag

**Managing Exceptions**

- ExceptionHandler
- ControllerAdvice

MVC - Interceptors, ContentNegotiation, Views

HTTP Streaming

Web Security

HTTP/2

## Dicas finais

- Confie na mágica. É bem testado! Além que a magia é muito bem documentada.
- Definitivamente use Spring Boot.
- Considere usar Kotlin - make it simple!
- O código do Spring é um bom professor mas dificilmente precisamos de algo parecido, foquemos no negócio.
- Atenção no mecanismo de Proxy - JDK dynamic proxies ou CGLIB.

## Referências

[https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview-history](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview-history)

[https://www.casadocodigo.com.br/products/livro-spring-framework](https://www.casadocodigo.com.br/products/livro-spring-framework)

Eighteen Years of Spring - [https://www.youtube.com/watch?v=UKx9YkOF03Q](https://www.youtube.com/watch?v=UKx9YkOF03Q)

[https://titanwolf.org/Network/Articles/Article?AID=8014b065-fb39-4ad2-ade8-ec3beb204088#gsc.tab=0](https://titanwolf.org/Network/Articles/Article?AID=8014b065-fb39-4ad2-ade8-ec3beb204088#gsc.tab=0)

Injection - [https://martinfowler.com/articles/injection.html](https://martinfowler.com/articles/injection.html)

Setter injection versus constructor injection and the use of @Required - [https://spring.io/blog/2007/07/11/setter-injection-versus-constructor-injection-and-the-use-of-required](https://spring.io/blog/2007/07/11/setter-injection-versus-constructor-injection-and-the-use-of-required)

Java Transaction - [https://www.infoq.com/minibooks/JTDS/](https://www.infoq.com/minibooks/JTDS/)