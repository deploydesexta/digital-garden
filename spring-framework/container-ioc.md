
# O container de IoC

O Spring Framework é baseado em "Injeção de Dependência" (DI) que é uma especialização da "Inversão de Controle" (IoC). A implementação do Spring conta com um *core* *container* que faz a gestão das dependências e é responsável por fazer uma análise das classes de sua aplicação, montar a árvore de dependência e iniciar o processo de criação dos *beans* através da instânciação da classe injetando as respectivas dependências.

É fundamental que você consiga enxergar o que os termos DI e IoC significam. Caso esteja com dificuldade, dê uma espiada em [Injeção de Dependência na prática](## ## Injeção de Dependência na prática).

> **O que são os beans?**
> No Spring, todos os objetos cujo ciclo de vida é gerenciado pelo próprio Spring é considerado um *bean*. Veremos com mais detalhes a seguir.

## Injeção de Dependência na prática

Para que você consiga enxergar com mais facilidade o porque dos nomes serem "Inversão de Controler" e "Injeção de Dependência", vamos a um exemplo prático.

Temos o seguinte cenário:
```java
class UserService {
	private UserRepository userRepository;
	// ... ignorando o restante da classe ...
}
```

A classe `UserService` depende de `UserRepository`. Sem utilizar o conceito de injeção de dependência, o que você provavelmente teria seria algo como:
```java
class UserService {
	private UserRepository userRepository;

	public UserService() {
		this.userRepository = new UserDAO();
	}
	// ... ignorando o restante da classe ...
}
```

Quais são os problemas dessa abordagem? Dentre vários, destaco aqui os principais:
- O acoplamento da classe `UserService` com a implementação `UserDAO`
- A dificuldade em testar isoladamente `UserService`, pois ela agora instância `UserDAO` internamente e é bem provavel que ela tenha dependências externas
- A dificuldade em alterar a implementação de `UserRepository` dentro de `UserService`.

Parece algo extraordinário, mas é muito comum precisarmos alterar implementações de classes dependendo do ambiente em que estamos executando a aplicação. Poderiamos, por exemplo, utilizar uma persistência em memória para os testes e como que ficaria a implementação?
```java
class UserService {
	private UserRepository userRepository;

	public UserService() {
		if (isTestEnvironment()) {
			this.userRepository = new UserImMemoryDB();
		} else {
			this.userRepository = new UserDAO();
		}
	}
	// ... ignorando o restante da classe ...
}
```

Agora imagine que isso é em apenas uma classe, em projetos grandes existem dezenas de outras classes, com mais dezenas de dependências, se torna inviável esse tipo de prática. Sem contar ainda que existe a possibilidade de que surjam outros ambientes e esse if só cresça.

Como evitar que isso aconteça? A forma mais utilizada hoje é através do padrão "Injeção de Dependência". Nele contamos com um componente externo que faz as instânciações e junto a injeção da implementação, ficando assim:
```java
class UserService {
	private UserRepository userRepository;

	public UserService(final UserRepository userRepository) {
		this.userRepository = userRepository;
	}
	// ... ignorando o restante da classe ...
}

class DIContainer {

	public void start() {
		boolean testing = isTestEnvironment();
		
		UserRepository userRepository = 
			testing ? new UserImMemoryDB() : new UserDAO();

		UserService userService = new UserService(userRepository); // <- Injeção
		// ... 
	}
}
```

Pronto. Agora o `DIContainer` é o responsável por entender a árvore de dependências, instânciar as classes na ordem correta injetando as dependências necessárias. A classe `UserService` apenas declara suas dependências e não precisa se preocupar de onde vem ou qual sua implementação. Claro que esse código é apenas um exemplo simplificado. Na prática, o container de IoC do Spring é muito mais genioso que apenas essa demonstração.