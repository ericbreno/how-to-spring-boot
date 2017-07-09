# Criando uma API Rest + Banco de dados

Olá! Neste tutorial pretendo explicar os pontos básicos sobre a criação
de uma api REST utilizando o framework SpringBoot, assim como a integração
com um Banco de Dados (Serão apresentados MySQL e H2, explicarei a diferença).

Cada um dos ponto será explicado de forma superficial, de forma que seja passado
apenas o necessário para se entender como todo o sistema funciona. No tutorial
será utilizada a IDE Eclipse Mars, com Java 8 e SpringBoot. 

Recomendo a utilização do **Postman**, que pode ser entronado na loja de apps do Chrome,
mas que roda independente do navegador. O Postman é uma ferramenta que vai nos ajudar
a testar a API, realizando as requisições. O Postman será utilizado nos exemplos.

Viu algum erro ou tem uma sugestão? Faça um pull-request! :D

# Iniciando - Maven e POM.xml

Inicialmente, você deve criar um projeto Maven e adicionar o SpringBoot como dependência,
junto do módulo **starter-web**. Isso pode ser feito através deste [site](start.spring.io),
basta adicionar a dependência e baixar o zip.

Será gerado um projeto em branco, apenas com uma classe Main e uma estrutura de pacotes
inicial. Você deve importar o projeto Maven em sua IDE.

Verifique a classe gerada está anotada com **@SpringBootApplication**, isto faz com que o Spring
identifique que esse é o ponto de início da aplicação, a classe também possui um método main,
ele que será executado para a aplicação iniciar.

# DevTools - Uma mão na roda

Para desenvolvimento do tutorial, e, posteriormente, sua aplicação, o uso do DevTools
se faz quase que obrigatório (quase, podemos fazer sem, mas tendo mais trabalho). O 
DevTools é uma ferramenta que, sempre que o código é modificado, re-executa a aplicação,
para as modificações inseridas serem aplicadas. Isso pode ser feito manualmente, mas, é bem chato,
então sugiro utilizar o DevTools. 

Para o utilizar, basta adicionar como dependência no pom.xml, da seguinte forma:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
</dependency>
```

Obs: Se você não estiver utilizando o DevTools, sempre pare a execução atual da aplicação
antes de iniciar uma nova, pois apenas iniciar uma nova execução faz com que a última não seja
terminada, fazendo com que a porta continue sendo utilizada, não permitindo que a nova execução
tenha sucesso. Este erro se parece com:

```
Description:

The Tomcat connector configured to listen on port 8080 failed to start. The port may already be in use or the connector may be misconfigured.
```

Se isso acontecer, basta entrar no gerenciador de processos (windows) ou fazer **ps aux | grep java** (linux)
e matar o processo Java.

# Criando a API
## Criando nosso primeiro endpoint - Hello Word

Agora que já temos a base para aplicação pronta, podemos começar a fazer o que realmente interessa, a API Rest.
Se você não sabe o que é uma API Rest, recomendo ler [aqui](https://pt.stackoverflow.com/questions/45783/o-que-%C3%A9-rest-e-restful), 
e depois continuar.

Seu próximo passo é criar uma classe, no exemplo vou utilizar a classe InterfaceRest.java,
devemos anotar a classe com **@RestController**. Esta anotação faz o Spring identificar a classe como
um controlador Rest, que podemos adicionar enpoints para nossas funcionalidades.

```
@RestController
public class InterfaceRest {
}
```

Dentro da classe vamos criar um método chamado helloWord, que retorna String e não tem parâmetros. 
O método deve ser anotado com **@RequestMapping(value = "/teste", method = RequestMethod.GET)**,
a anotação **@RequestMapping** faz com que o método seja relacionado à um endpoint, ou, 
ao fazermos uma requisição no endpoint "/teste" o método seja executado. O parâmetro **value** da anotação
define qual endpoint (como String) o método está relacionado.

Na anotação nós temos outro parâmetro, o **method**, que indica qual método HTTP (O tipo da requisição)
o endpoint e método Java está relacionado. Logo, um método java está associado à um endpoint e um tipo
de requisição. No nosso caso, o método vai aceitar requisições de **GET** para o endpoint de **"/teste"**.

No corpo do método você deve fazer apenas ele retornar uma String qualquer, ex: "Oi, eu estou funcionando!".
Agora, execute a aplicação, abra o Postman e realize a requisição para **localhost:8080/teste**, verifique que
o retorno da response foi o que você adicionou no método Java.

```
@RestController
public class InterfaceRest {

    @RequestMapping(value="/teste", method=RequestMethod.GET)
    public String helloWord() {
        return "Oi, eu estou funcionando!";
    }
}
```

## Recursos? - Responsabilidades diferentes

Agora já podemos criar endpoints que retornam informações através de uma interface Rest, mas podemos
fazer isso de uma forma mais bonita do que vimos até agora, que é separando cada classe **RestController**
por recurso, ex: UsuarioRest, SerieRest, DocumentoRest, fazendo com que nossa aplicação tenha endpoints do tipo

```
/usuario/cadastrar - Cadstrar um usuário - POST
/usuario/regibalbo - Recuperar informações para o usuário regibalbo - GET
/serie/1 - Recuoperar informações da série 1.
```

Isso poderia ser feito utilizando o caminho completo em cada método, da mesma forma que já vimos,
mas também podemos realizar uma padronização, separando cada classe para um recurso, e seus métodos
apenas terem o fim do endpoint para o serviço. Isso pode ser feito adicionando a anotação **@RequestMapping** 
na própria classe, e definindo um **value** que será a "base" para os endpoints definidos nos métodos da classe.
Ex:

```
@RestController
@RequestMapping(value = "/usuario")
public class Teste {

    @RequestMapping(value = "/consulta/{nome}", method = RequestMethod.GET)
    public String consultarUsuario(@PathVariable String nome) {
        return nome;
    }
}
```

Isso vai fazer com que o método java consultarUsuario seja executado ao realizar uma requisição de POST
em **/usuario/consulta/eric**, por exemplo.

**@PathVariable**: Anota um parâmetro do método que está relacionado à um endpoint. Os endpoints podem ter variáveis, ou,
pedaços não fixos para a requisição, como no exemplo, utilizamos "/consulta/**{nome}**", onde a variável
é definida dentro de {}, e recuperada no método se identificando o parâmetro com @PathVariable. Se realizarmos
uma requisição get **get** em /usuario/consulta/eric, veremos que no método, a variável nome vai ter valor "eric". Teste isso.
Você pode fazer o método retornar uma String, sendo essa o parâmetro, para fins de debug.

## Enviando informações para nossa API - POST, PUT

Agora já sabemos como criar endpoints e os separar por recursos, vamos ver como podemos adicionar métodos que recebem
**objetos** através das requisições, utilizando métodos http de POST e PUT.

Primeiro, precisamos definir um objeto para ser retornado, crie uma classe qualquer, que contenha um campo de String e um
inteiro, adicione gets e sets para cada campo. 
Obs: É necessário que os campos tenham os gets e sets.

```
public class ObjetoTeste {
    private String nome;
    private int quantidade;
    
    public String getNome() {
        return nome;
    }
    public void setNome(String nome) {
        this.nome = nome;
    }
    /**/
}
```

Uma vez que você criou sua primeira classe, implemente um método com tipo GET que retorna algum objeto
que você criou.

```
@RequestMapping(value="/teste", method=RequestMethod.GET)
public ObjetoTeste testeRecuperar() {
    ObjetoTeste obj = new ObjetoTeste();
    obj.setNome("clemison");
    obj.setQuantidade(12382);
    return obj;
}
```

Agora, faça uma requisição e verifique como o objeto foi retornado, seguindo o padrão JSON, e que 
cada chave representa o nome do atributo para o objeto. Você vai enviar os dados para o servidor 
neste mesmo formato.

## Realizando o POST

Agora que já sabemos como a interface Rest trata os dados, e como podemos retornar objetos completos
por vez, vamos ver como fazer a api receber objetos.

Crie um método com tipo POST, pode utilizar o mesmo endpoint*.

* O mesmo endpoint pode suportar diversos tipos de requisição, o método java que vai ser executado fica
de acordo com o tipo da requisição e o tipo definido em sua anotação.

```
@RequestMapping(value="/teste", method=RequestMethod.POST)
public String testeReceber(@RequestBody ObjetoTeste objetoRecebido) {
    return "Recebi um objeto com " + objetoRecebido.getNome() + " - e quantidade " + objetoRecebido.getQuantidade();
}
```

A anotação **@RequestBody** identifica que o corpo da requisição deve ser transformado no objeto
anotado pela mesma.

No Postman, você deve mudar o tipo da requisição para POST, ir para a opção de **body**, marcar a opção **raw**
e depois definir o tipo para **JSON**. Defina o body a ser enviado com um JSON no formato do seu objeto. Para
o caso do exemplo:

```
{
    "nome": "clebinho",
    "quantidade": "197"
}
```

Agora, faça a requisição e veja que foi retornada uma String com as informações do objeto. Criar um método de PUT
funciona da mesma forma, os dois têm apenas a diferença semantica de que POST é utilizado para inserção/cadastro
e o PUT é utilizado para atualização.

Agora já sabemos como:
* Fazer consultas de informações, com GET, e passando parâmetros opcionais no endpoint.
* Enviar informações para nossa API, com POST e PUT.

## Testes - O que mais podemos fazer?

Agora você pode fazer testes para verificar a API funcionando e descobrir o que mais podemos fazer, implemente o seguinte:
* Um método de GET retornar uma **lista** (ou array) de objetos
* Um método POST guardar o objeto enviado em um *Map*, que seja um campo estático, e depois em um GET
recuperar esse objeto do *Map* (simulando persistência). Você pode guardar os objetos no Map com as chaves
sendo **nome**, e recuperar através do GET passando o nome em seu endpoints, como mostrado anteriormente.
* Um método de PUT que atualiza um objeto no Map, e em seguida um GET para recuperar o objeto atualizado.

# Integrando com Banco de Dados H2 - Agora as coisas ficam interessantes

## H2?
H2 é um banco de dados em memória, ou seja, ele simula um BD, mas sempre que a aplicação é reiniciada os dados
são perdidos, diferente de um BD real, como MySQL. A vantagem de se utilizar H2 é que não precisaremos configurar 
um BD localmente para a máquina, nem criar tabelas e colunas.

## Driver para conexão
Para se comunicar com o BD utizaremos drivers, que são interfaces que convertem os comandos Java para comandos 
de banco de dados. O driver que mostrarei aqui segue a especificação do JPA (Java Persistence API), que torna fácil
a migração para outro BD, sem precisarmos mudar o código, apenas alterando o driver que vai ser utilizado, desde que siga
o mesmo padrão.

## Adicionando JPA
Adicione como dependência do pom a biblioteca do JPA:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

## Adicionando driver do H2
Para adicionar o driver do H2, você deve adicionar como dependência do pom:

```
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

## Hibernate
Para realizar as operações de consultar, guardar e atualizar informações, vamos utilizar o framework Hibernate,
ele se comunicará com o driver do H2 de forma automática. O Hibernate é um framework robusto, que contém diversas
funcionalidades e uma vasta aplicação e usabilidade, mas aqui abordaremos apenas o básico para uma aplicação o utilizar.

Adicione as configurações necessárias para o Hibernate, no arquivo **/src/main/resources/application.properties**:

```
spring.datasource.url = jdbc:h2:file:~/h2/app_db;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.username = sa
spring.datasource.password = 
spring.datasource.driverClassName = org.h2.Driver
spring.jpa.hibernate.ddl-auto = update
	
logging.level.org.hibernate.SQL=INFO
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

## Camada de comunicação com o BD - Os DAO's
Para a comunicação com o BD vamos utilizar uma camada específica, que chamaremos de DAO (Data Access Object), 
ela realizará todas as operações com o BD.

Crie um DAO, no exemplo utilizarei o TesteDAO. Sua classe DAO deve estar anotada com **@Repository**, indicando que
esta classe é um classe Repositório(de comunicação com o BD). Dentro de sua classe DAO você deve *injetar* o objeto
do Hibernate que vai tratar as operações, da seguinte forma:

```
@Repository
public class TesteDAO {
    @PersistenceContext
    private EntityManager em;
}
```

Veja sobre Injeção de dependências [aqui](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-spring-beans-and-dependency-injection.html).
A anotação **@PersistenceContext** vai adicionar (automaticamente) uma instância de EntityManager na classe, assim que ela for iniciada. 
O EntityManager (Gerenciador de entidades) que vai tratar as operações.

Os principais métodos do EntityManager que precisamos conhecer são:
* Persist(entidade) - Vai inserir uma entidade no BD
* Merge(entidade) - Vai atualizar uma entidade no BD
* CreateTypedQuery(String, TipoObjeto) - Cria consultas por Objetos.

Agora, adicione um método na classe DAO para persistir seu *ObjetoTeste*, da seguinte forma:

```
public ObjetoTeste persisteObjeto(ObjetoTeste obj) {
    em.persist(obj);
    return obj;
}
```

E um para recuperar do BD, aqui criaremos uma Query:

```
public ObjetoTeste consultaObjeto(Long id) {
    TypedQuery<ObjetoTeste> query = em.createTypedQuery("select obj from ObjetoTeste where obj.id = :id");
    query.setParameter("id", id);
    return query.getSingleResult();
}
```

Note que a sintaxe da query, que é estrutura em HQL (Hibernate Query Language) é parecida com a sintaxe 
de SQL, mas simplifica a query em alguns casos. Saiba mais sobre HQL aqui. O método **getSingleResult** 
vai nos retornar o objeto de resultado da consulta, de acordo com o ID passado (falarei sobre ID agora).

Uma das diferenças do SQL se dá por não consultarmos por tabelas, mas sim por tipos de objetos, como no exemplo,
estamos pegando dentre ObjetoTeste, o Hibernate consegue distinguir o que estamos buscando por os objetos
persistidos terem a classe anotada por @Entity.

Você pode adicionar parâmetros para a busca utilizando **:nomePropriedade**, e realizando a comparação
como mostrado na query de exemplo acima. Para substituir os parâmetros, deve-se utilizar o método **query.setParameter**.

Também é possível realizar uma consulta por outros parâmetros, seguindo o ex:

```
public ObjetoTeste consultaObjeto(Long id, String nomeObjeto) {
    TypedQuery<ObjetoTeste> query = em.createTypedQuery("select obj from ObjetoTeste where obj.id = :id and obj.nomeObjeto = :nomeObjeto", ObjetoTeste.class);
    query.setParameter("id", id);
    query.setParameter("nomeObjeto", nomeObjeto);
    return query.getSingleResult();
}
```

Deve-se apenas ter cuidado ao fazer consultas que não sejam por id porque essas podem retornar mais de um objeto,
enquanto por id sempre retornará apenas 1, ou nenhum, caso o objeto não exista.

## A entidade que vai ser persistida
* Lembre-se de sempre criar gets e sets para as propriedades dos objetos a serem persistidos.

Para podermos persistir uma entidade, ela precisa ser devidamente identificada e distinguível de outras do mesmo
tipo, fazemos isso utilizando as anotações do próprio JPA, seguindo:

* @Entity - Anota a classe, identifica que esse objeto pode ser persistido no BD
* @Column - Anota uma propriedade, identifica que a propriedade vai ser uma coluna do BD
* @Id - Anota uma propriedade, diz que essa propriedade será o identificador para a entidade, utilizaremos identificadores do tipo Long.

Na anotação @Column podemos ainda adicionar qual nome específico da Coluna e regras, como de a propriedade não poder
ser nula, ex:

```
@Column(name = "nome_usuario", nullable = false)
private String nomeUsuario;
```

Isso vai fazer com que não seja permitido inserir a propriedade nomeUsuario com valor null, assim como
a propriedade será identificada no BD pela coluna **nome_usuario**.

Adicione no seu objeto um campo id do tipo Long, e o anote da seguinte forma:

```
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

Como mencionado acima, a anotação @Id vai fazer com o que campo seja o elemento que vai distinguir dois objetos para
o mesmo tipo. A anotação **@GeneratedValue** vai fazer com que ele seja gerado automaticamente, e não precisemos
nos preocupar em guardar dois objetos do mesmo tipo com um mesmo id. Todas as entidades que forem ser persistidas
devem ter um ID.

Feito isso, anote outras propriedades que deseja guardar no BD, da mesma forma que o exemplo.

## Usando o DAO
Para utilizar o DAO para se recuperar e persistir os objetos, vamos utilizar a injeção de dependências, como já
mencionado anteriormente, da seguinte forma:

```
@Autowired
private TesteDAO dao;
```

Você deve adicionar esta injeção na sua classe Rest. Agora, faça com que algum método de POST receba o ObjetoTeste,
o persista no BD, e em seguida o de GET recupere do BD, passando o id do objeto persistido. Lembre de retornar o objeto
persistido no POST, para que você possa saber qual o id para a consulta.

* Para realizar a consulta por ID, utilize as variáveis de parâmetro do endpoint, e passe a variável para o método de consulta do DAO.
* Adicione no DAO um método que atualiza uma entidade, este método vai utilizar o **EntityManager.merge**.
* Adicione um método do tipo PUT que vai atualizar uma entidade no BD, lembre que a entidade passada deve ter o ID.

## Relacionando objetos
E se nós quisermos dizer que o usuário vai ter ObjetoTeste? Ou uma lista de objetos? Precisamos saber como relacionar os objetos agora,

Existem algumas formas de fazer isso, aqui apresentarei o formato de relação inversa, onde um usuário vai ter uma lista
de objetos. Para isso, faremos o nosso ObjetoTeste ter uma propriedade chamada **idUsuario**, e sempre persistiremos
o objeto com esta propriedade sendo o id de um usuário do sistema, e para a consulta nós buscaremos por *todas as séries que pertencem a este usuário*.

Nosso objeto terá um novo campo:

```
@Column(name="idUsuario")
private Long idUsuario;
```

E se quisermos consultar todos os objetos de um usuário, podemos fazer isso com um método no DAO desta forma:

```
public List<ObjetoTeste> consultarObjetosDoUsuario(Long idUsuario) {
    TypedQuery query = CreateTypedQuery("select obj from ObjetoTeste where obj.idUsuario = :idUsuario", ObjetoTeste.class)
    query.setParameter("idUsuario", idUsuario);
    return query.getResultList();
}
```

O método getResultList vai nos retornar uma lista de objetos, onde todos tem o idUsuario igual ao passado por parâmetro,
ou seja, no dá todas os objetos do usuário.

## Finalizando
Pronto, agora você já tem configurado no seu projeto o H2, utilizando o Hibernate, a classe Rest que se comunica com o DAO
e realiza as operações com o BD, servidor finalizado!
