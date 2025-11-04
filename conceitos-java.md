# Guia de Estudos para Entrevista Técnica: Java & Spring

Este guia consolida conceitos essenciais de Java, Arquitetura de Software e Spring Framework, preparando você para responder às perguntas mais comuns em entrevistas técnicas.

---

## 1. Conceitos Fundamentais de Java

### O que é uma Expressão Lambda e qual problema ela resolve?

Uma **Expressão Lambda** é uma forma concisa de representar uma função anônima (uma função sem nome). Introduzida no Java 8, ela promove um estilo de programação mais funcional, especialmente útil ao trabalhar com a API de Streams e interfaces funcionais.

**Problema que resolve:**
- **Redução de Boilerplate:** Elimina a necessidade de criar classes anônimas verbosas.
- **Clareza e Concisão:** Torna o código mais legível e expressivo.
- **Programação Funcional:** Facilita o uso de operações como `map`, `filter`, `forEach` e `reduce` em coleções.

**Exemplo Prático:** Ordenando uma lista de Strings.

**Antes (com Classe Anônima):**
```java
List<String> nomes = Arrays.asList("Ana", "Bruno", "Carlos");
Collections.sort(nomes, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareTo(b);
    }
});
```

**Depois (com Expressão Lambda):**
```java
List<String> nomes = Arrays.asList("Ana", "Bruno", "Carlos");
Collections.sort(nomes, (a, b) -> a.compareTo(b));
// Ou de forma ainda mais concisa com Method Reference:
// Collections.sort(nomes, String::compareTo);
```

### O que é uma Interface Funcional?

Uma **Interface Funcional** é uma interface que declara **exatamente um método abstrato**. Ela serve como o tipo de uma expressão lambda. A anotação `@FunctionalInterface` é usada para garantir essa restrição em tempo de compilação.

**Exemplos comuns no dia a dia:**
- `Predicate<T>`: Recebe um valor e retorna um booleano (`boolean test(T t)`). Usado em filtros.
- `Function<T, R>`: Recebe um valor e retorna outro (`R apply(T t)`). Usado para mapeamento.
- `Consumer<T>`: Recebe um valor e não retorna nada (`void accept(T t)`). Usado para executar ações.
- `Supplier<T>`: Não recebe nada e retorna um valor (`T get()`). Usado para gerar valores.

### O que significa "effectively final" e por que essa restrição existe para lambdas?

Uma variável local é **effectively final** se seu valor não for alterado após a primeira atribuição, mesmo que não seja explicitamente declarada com a palavra-chave `final`.

**Por que a restrição existe?**
Lambdas podem "capturar" variáveis do escopo onde foram criadas. Para garantir a segurança em ambientes concorrentes (multithreading) e evitar inconsistências, a lambda captura apenas o **valor** da variável no momento da sua criação. Se a variável pudesse ser alterada depois, a lambda e o método externo teriam "visões" diferentes do valor, causando comportamento imprevisível. A restrição `effectively final` força a imutabilidade e garante a consistência.

### `parallelStream()` vs `stream()`

- **`stream()`**: Processa os elementos de uma coleção de forma sequencial, em uma única thread.
- **`parallelStream()`**: Divide o processamento em múltiplas threads, utilizando os núcleos disponíveis da CPU para executar as operações em paralelo.

| Característica | `parallelStream()` |
| :--- | :--- |
| **Vantagem** | Ganho de performance em operações computacionalmente intensivas (CPU-bound) e com grande volume de dados. |
| **Desvantagem**| 1. **Overhead:** A criação e sincronização de threads têm um custo. Para coleções pequenas, pode ser mais lento que o `stream()` sequencial.<br>2. **Não ideal para I/O:** Operações de I/O (banco de dados, chamadas de rede) não se beneficiam, pois as threads ficarão bloqueadas esperando.<br>3. **Estado e Ordem:** Não é recomendado para operações que dependem de estado compartilhado ou da ordem dos elementos. |

### `Optional.orElse()` vs `Optional.orElseGet()`

Ambos fornecem um valor padrão caso o `Optional` esteja vazio, mas com uma diferença crucial de performance.

- `orElse(getValorPadrao())`: O método `getValorPadrao()` **sempre será executado**, mesmo que o `Optional` contenha um valor.
- `orElseGet(() -> getValorPadrao())`: O `Supplier` (a expressão lambda) **só será executado se o `Optional` estiver vazio**.

**Conclusão:** Prefira `orElseGet()` quando a geração do valor padrão for uma operação custosa (ex: uma chamada ao banco de dados ou um cálculo complexo).

---

## 2. Arquitetura de Software

### Padrões de Arquitetura

| Padrão | Conceito Principal | Estrutura Típica | Quando Usar |
| :--- | :--- | :--- | :--- |
| **MVC (Model-View-Controller)** | Padrão de design de interação que separa **dados** (Model), **apresentação** (View) e **lógica de controle** (Controller). | `Controller` -> `Service` -> `Repository` -> `Entity` | Aplicações web simples e como base para outras arquiteturas. |
| **Layered (Em Camadas)** | Organiza o sistema em camadas horizontais, onde cada camada só pode se comunicar com a camada imediatamente abaixo dela. | `Presentation` -> `Business/Service` -> `Persistence/Data` -> `Database` | Projetos que precisam de uma separação clara de responsabilidades, muito comum em sistemas corporativos. |
| **Clean Architecture** | Isola as **regras de negócio** (núcleo) de detalhes externos (frameworks, UI, banco de dados). A dependência aponta sempre para dentro. | `Entities` -> `Use Cases` -> `Interface Adapters` -> `Frameworks & Drivers` | Projetos grandes e complexos onde a longevidade e a independência de tecnologia são cruciais. |
| **Hexagonal (Ports & Adapters)** | O "núcleo" da aplicação (domínio) é isolado e se comunica com o mundo exterior através de **Portas** (interfaces) e **Adaptadores** (implementações). | `Core (Domain + Use Cases)` -> `Ports (Input/Output)` -> `Adapters (Web, Database, Messaging)` | Aplicações que precisam ser altamente testáveis e desacopladas, permitindo trocar tecnologias (ex: banco de dados) sem impactar o núcleo. |

---

## 3. Spring & Spring Boot

### Spring Framework vs. Spring Boot

- **Spring Framework:** É um ecossistema abrangente que fornece, entre muitas coisas, um container de **Inversão de Controle (IoC)** para gerenciar o ciclo de vida de objetos (Beans) e suas dependências. Ele é poderoso, mas exige configuração manual (seja XML ou JavaConfig).
- **Spring Boot:** É um framework construído sobre o Spring que simplifica drasticamente a criação de aplicações. Ele segue o princípio de **"Convenção sobre Configuração"**, oferecendo:
  - **Autoconfiguração:** Configura automaticamente a aplicação com base nas dependências presentes no classpath.
  - **Servidor Embutido:** Vem com servidores como Tomcat ou Netty prontos para uso.
  - **Starters:** Simplificam o gerenciamento de dependências (ex: `spring-boot-starter-web`).

### O que é Inversão de Controle (IoC) e Injeção de Dependências (DI)?

- **Inversão de Controle (IoC):** É um princípio de design onde o controle sobre a criação e gerenciamento de objetos (dependências) é transferido da aplicação para um container ou framework. Em vez de o seu código criar suas dependências (ex: `new UserRepository()`), o container faz isso por você.

- **Injeção de Dependências (DI):** É a **implementação** do princípio de IoC. O container "injeta" as dependências em um objeto quando ele é criado. As formas mais comuns de DI no Spring são:
  1. **Injeção por Construtor (Recomendada):** As dependências são declaradas como parâmetros do construtor.
  2. **Injeção por Setter:** Usa métodos `set...()`.
  3. **Injeção por Campo (Não recomendada para produção):** Usa a anotação `@Autowired` diretamente no campo.

**Exemplo (Injeção por Construtor):**
```java
@Service
public class UserService {
    private final UserRepository userRepository;

    // O Spring injeta a instância de UserRepository aqui
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### `BeanFactory` vs `ApplicationContext`

Ambos são interfaces para o container IoC do Spring, mas `ApplicationContext` é uma extensão mais completa.

- **`BeanFactory`:** É a interface raiz, o "motor" básico. Fornece o essencial: gerenciamento de beans e injeção de dependências.
- **`ApplicationContext`:** É o "carro completo". Herda tudo do `BeanFactory` e adiciona mais funcionalidades, como:
  - Gerenciamento do ciclo de vida dos beans.
  - Suporte à internacionalização (i18n).
  - Propagação de eventos.
  - Integração com aspectos (AOP).

Em 99% dos casos, você interagirá com o `ApplicationContext`.

### Escopos de Beans no Spring

O escopo define o ciclo de vida de uma instância de bean.

| Escopo | Descrição |
| :--- | :--- |
| **`singleton`** | (Padrão) Uma única instância do bean por `ApplicationContext`. |
| **`prototype`** | Uma nova instância é criada toda vez que o bean é solicitado/injetado. |
| **`request`** | Uma instância por requisição HTTP (apenas em aplicações web). |
| **`session`** | Uma instância por sessão de usuário (apenas em aplicações web). |

### Fluxo de Inicialização do Spring Boot

1.  **`@SpringBootApplication`**: A aplicação inicia a partir do método `main`, que chama `SpringApplication.run()`. Esta anotação combina `@Configuration`, `@EnableAutoConfiguration` e `@ComponentScan`.
2.  **Criação do `ApplicationContext`**: O Spring Boot cria a implementação apropriada do `ApplicationContext`.
3.  **`@ComponentScan`**: Escaneia o classpath em busca de beans (classes anotadas com `@Component`, `@Service`, `@Repository`, `@Controller`, etc.) e os registra no container.
4.  **`@EnableAutoConfiguration`**: Com base nos "starters" e outras dependências, a autoconfiguração entra em ação, configurando beans essenciais (como `DataSource`, `EntityManagerFactory`, etc.).
5.  **Injeção de Dependências**: O container instancia os beans e injeta suas dependências.
6.  **Runners**: Executa beans do tipo `CommandLineRunner` e `ApplicationRunner` após o contexto ser totalmente inicializado.
7.  **Servidor Embutido**: Inicia o servidor web (ex: Tomcat) e a aplicação fica pronta para receber requisições.

---

## 4. Comunicação e Integração

### O que é Mensageria?

**Mensageria** é um padrão de comunicação **assíncrona** entre sistemas, onde as mensagens são trocadas através de um intermediário (um *message broker*), em vez de chamadas diretas e síncronas (como uma API REST).

| Broker | Características | Casos de Uso |
| :--- | :--- | :--- |
| **RabbitMQ** | Broker tradicional que implementa o protocolo AMQP. Focado em filas, roteamento complexo e entrega confiável de tarefas. | Fila de tarefas (ex: processar um pagamento), comunicação entre microsserviços. |
| **Apache Kafka** | Plataforma de streaming de eventos distribuída. Focada em alto throughput, armazenamento durável e reprocessamento de eventos em tempo real. | Streaming de eventos, logs, métricas, integração de dados em larga escala. |

**Benefícios:**
- ✅ **Desacoplamento:** O produtor não conhece o consumidor.
- ✅ **Resiliência:** Se um serviço consumidor estiver offline, as mensagens são enfileiradas e processadas quando ele voltar.
- ✅ **Escalabilidade:** Múltiplos consumidores podem processar mensagens em paralelo.
- ✅ **Performance:** O produtor não precisa esperar por uma resposta, liberando recursos mais rápido.

**Desvantagens:**
- ❌ **Complexidade:** Adiciona uma nova peça de infraestrutura para gerenciar.
- ❌ **Debugging:** Rastrear um fluxo assíncrono pode ser mais desafiador.
- ❌ **Garantias de Entrega:** Requer configuração cuidadosa para lidar com ordem, duplicidade e perda de mensagens.
