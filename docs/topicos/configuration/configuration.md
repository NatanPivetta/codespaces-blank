# Microprofile Configuration

O sistema de configuração de um Quarkus utiliza a especificação [Microprofile Config.](https://github.com/eclipse/microprofile-config) implementada com o [SmallRye Config](https://github.com/smallrye/smallrye-config). As configurações são provenientes de várias [fontes](https://quarkus.io/guides/config-reference#environment-variables) de dados, como por exemplo: propriedades do sistema, arquivos `.env`, arquivos yaml, entre outros.

## Propriedade de sistema

Imagine a declaração de duas propriedades no arquivo `application.properties`:

```yaml
    pw2.message=hello
    pw2.name=world
```

Para injetar no serviço o valor da propriedade `pw2.message` utilizamos a anotação `@ConfigProperty`:

```java
@ConfigProperty(name = "pw2.message",  defaultValue="" )
String message;

@ConfigProperty(name = "pw2.name")
Optional<String> name;
```

Se você não fornecer um valor para um propriedade, a inicialização do serviço lançará uma exceção: `javax.enterprise.inject.spi.DeploymentException`. Nesse caso, é possível codificar um valor padrão para a variável `message` (uma String vazia). O exemplo também mostra que uma classe Optional vazio também é injetado se a configuração não fornecer um valor para a variável `name`. Veja um exemplo de utilização de inicialização do atributo `name`.

```java
@GET
@Produces(MediaType.TEXT_PLAIN)
public String hello() {
    return message + " " + name.orElse("world");
}
```

O Microprofile Config. também permite acessar valores de configuração de maneira programática. No exemplo abaixo, a classe `ConfigProvider` permite que você acesse, o valor da chave `database.name`.

```java
    String databaseName = ConfigProvider.getConfig().getValue("database.name", String.class);
    Optional<String> maybeDatabaseName = ConfigProvider.getConfig().getOptionalValue("database.name", String.class);
```

As propriedades de sistemas não necessariamente necessitam serem passadas para a aplicação por meio de arquivos `application.properties`, você pode atribuir valores para propriedade no momento da execução do serviço, por meio do parâmetro `-D`, observe exemplo abaixo:

    ./mvnw compile quarkus:dev -Dpw2.message=hello


🚨 O próprio Quarkus é configurado por meio do mesmo mecanismo do ser serviço. Quarkus reserva o  namespace `quarkus.` para sua própria configuração. Por exemplo, para configurar a porta do servidor HTTP, você pode definir `quarkus.http.port` no arquivo application.properties. Assim, nunca utilize o prefixo `quarkus.`como prefixo das suas variáveis.

Algumas configurações do Quarkus só têm efeito durante o tempo de construção (*build*), o que significa que não é possível alterá-las no tempo de execução. Essas configurações ainda estarão disponíveis em tempo de execução, mas, para somente leitura. As configurações de tempo de construção são marcadas com um ícone de cadeado (🔒) na [lista](https://quarkus.io/guides/all-config) de todas as opções de configuração. Uma mudança em qualquer uma dessas configurações de contstrução requer uma reconstrução do seu serviço (*rebuild*).

### Perfis

Podemos criar [perfils](https://quarkus.io/guides/config-reference#profiles) de configurações específicas para cada tempo do desenvolvimento de um serviço. Por padrão, o Quarkus possui suporte para três perfis: `dev`, `test` e `prod`, ou seja, podemos ter configurações para o tempo de desenvolvimento, teste e produção. No Arquivo `application.properties`, conseguimos separar as configurações de cada perfil por meio do seletor `%`, veja um exemplo:

```yaml
    pw2.jdbc=jdbc:mysql://localhost:3306/pw2
    %prod.pw2.jdbc=jdbc:mysql://rpmhub.dev:3307/pw2
```

O exemplo acima ilustra uma situação bastante comum no desenvolvimento de sistemas, termos endereços distintos para o tempo de desenvolvendo e produção. Note que não foi necessário utilizar o `%dev` ou `%test` na primeira linha para indicar que se trata de uma configuração utilizada no desenvolvimento e teste.

Se o serviço possuir um conjunto grande de configurações, você poderá dividir os perfis em arquivos distintos (`application-{nome do perfil}.properties`), nesse caso, sem a necessidade da utilização do prefixo `%` dentro do arquivo específico. Um exemplo, se você desejar ter um perfil apenas para teste, você poderá criar um arquivo de configuração com o nome `application-test.properties`.

Caso você deseje, você também poderá criar seus [próprios perfis](https://quarkus.io/guides/config-reference#custom-profiles) de configuração por meio de um prefixo. Por exemplo, imagine que você deseja criar um perfil chamado "build", como exemplo, observe o arquivo `application.properties`: 

```yaml
    quarkus.http.port=9090
    %build.quarkus.http.port=9999
```

Para executar um serviço com um determinado perfil, você deve utilizar a propriedade `quarkus.profile`, por exemplo:

    ./mvnw compile quarkus:dev -Dquarkus.profile=build

