# Spring Boot @ConfigurationProperties example

### Exemplo de Spring Boot @ConfigurationProperties

```
package com.isac;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

// By default, Spring boot it loads property from application.properties, we can use @PropertySource to load other .properties files.
//@PropertySource("classpath:custom.properties")
@Component
@ConfigurationProperties("app")
public class AppProperties {

    private String error;
    private List<Menu> menus = new ArrayList<>();
    private Compiler compiler = new Compiler();

    public static class Menu {
        private String name;
        private String path;
        private String title;
```

Spring Boot @ConfigurationProperties está permitindo que o desenvolvedor mapeie todo o arquivo .properties e yml em um objeto facilmente.

P.S testado com Spring Boot 2.1.2.RELEASE

# 1. @Value
#### 1.1 Normalmente, usamos o @Value para injetar o valor .properties um por um, isso é bom para arquivos .properties de estrutura pequena e simples. Por exemplo,

global.properties
```
email=example@gmail.com
pool de threads = 12
```

GlobalProperties.java
```
@Componente
@PropertySource ("classpath: global.properties")
public class GlobalProperties {

     @Value ("$ {thread-pool}")
     private int threadPool;

     @Value ("$ {email}")
     e-mail de string privado;
    
     // getters e setters
```

### 1.2 O equivalente em @ConfigurationProperties

GlobalProperties.java

```
import org.springframework.boot.context.properties.ConfigurationProperties;

@Component
@PropertySource("classpath:global.properties")
@ConfigurationProperties
public class GlobalProperties {

    private int threadPool;
    private String email;

    //getters and setters

}
```

# 2. @ConfigurationProperties
### 2.1 Revise uma estrutura complexa .properties ou arquivo yml abaixo, como iremos mapear os valores via @Value?

application.properties

```
#Logging
logging.level.org.springframework.web=ERROR
logging.level.com.isac=DEBUG

#Global
email=example@gmail.com
thread-pool=10

#App
app.menus[0].title=Home
app.menus[0].name=Home
app.menus[0].path=/
app.menus[1].title=Login
app.menus[1].name=Login
app.menus[1].path=/login

app.compiler.timeout=5
app.compiler.output-folder=/temp/
```

ou o equivalente em YAML.

application.yml

```
logging:
  level:
    org.springframework.web: ERROR
    com.isac: DEBUG
email: example@gmail.com
thread-pool: 10
app:
  menus:
    - title: Home
      name: Home
      path: /
    - title: Login
      name: Login
      path: /login
  compiler:
    timeout: 5
    output-folder: /temp/
  error: /error/
```

```
Observação
@ConfigurationProperties oferece suporte a arquivos .properties e .yml.
```

### 2.2 @ConfigurationProperties vem para resgatar:

AppProperties.java

```
package com.isac;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

@Component
@ConfigurationProperties("app") // prefix app, find app.* values
public class AppProperties {

    private String error;
    private List<Menu> menus = new ArrayList<>();
    private Compiler compiler = new Compiler();

    public static class Menu {
        private String name;
        private String path;
        private String title;

        //getters and setters

        @Override
        public String toString() {
            return "Menu{" +
                    "name='" + name + '\'' +
                    ", path='" + path + '\'' +
                    ", title='" + title + '\'' +
                    '}';
        }
    }

    public static class Compiler {
        private String timeout;
        private String outputFolder;

        //getters and setters

        @Override
        public String toString() {
            return "Compiler{" +
                    "timeout='" + timeout + '\'' +
                    ", outputFolder='" + outputFolder + '\'' +
                    '}';
        }

    }

    //getters and setters
}
```

GlobalProperties.java

```
package com.isac;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties // no prefix, find root level values.
public class GlobalProperties {

    private int threadPool;
    private String email;

    //getters and setters
}
```

# 3. Validação de @ConfigurationProperties
Este @ConfigurationProperties suporta validação de bean JSR-303.

### 3.1 Adicione @Validated na classe @ConfigurationProperties e anotações javax.validation nos campos que queremos validar.

GlobalProperties.java

```
package com.isac;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import org.springframework.validation.annotation.Validated;

import javax.validation.constraints.Max;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotEmpty;

@Component
@ConfigurationProperties
@Validated
public class GlobalProperties {

    @Max(5)
    @Min(0)
    private int threadPool;

    @NotEmpty
    private String email;

    //getters and setters
}
```

### 3.2 Definir pool de threads = 10

application.properties

```
#Global
email=example@gmail.com
thread-pool=10
```

### 3.3 Inicie o Spring Boot e chegaremos à seguinte mensagem de erro:

```
Console

***************************
APPLICATION FAILED TO START
***************************

Description:

Binding to target org.springframework.boot.context.properties.bind.BindException: 
    Failed to bind properties under '' to com.isac.GlobalProperties failed:

    Property: .threadPool
    Value: 10
    Origin: class path resource [application.properties]:7:13
    Reason: must be less than or equal to 5


Action:

Update your application's configuration
```

## 1. How to start
```
$ git clone [https://github.com/isaccanedo/externalize-config-properties-yaml.git](https://github.com/isaccanedo/externalize-config-properties-yaml.git)
$ cd externalize-config-properties-yaml
$ mvn spring-boot:run

access localhost:8080
```