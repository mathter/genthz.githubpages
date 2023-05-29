---
layout: page_doc
title: Docs
menu_type: main
page_type: page_doc
---

# Documentation. Version 3

## Getting started

To make use of GenThz within your project, simply add the maven dependency available from Maven Central repository, like
so:

### Installation

```xml
<dependency>
    <groupId>org.genthz</groupId>
    <artifactId>genthz-core</artifactId>
    <version>3.0.2</version> <!-- or latest version -->
</dependency>
<dependency>
    <groupId>org.genthz</groupId>
    <artifactId>genthz-logging-slf4j</artifactId>
    <version>3.0.2</version> <!-- or latest version -->
</dependency>
```

<div class="alert alert-block alert-danger">
<b>Please verify the latest version</b>
</div>
### Sources
There are source codes at [github.com](https://github.com/mathter/genthz){:target="_blank"}
### HelloWorld sample
It is required to get [`ObjectFactory`]({{site.source_base}}/genthz-core/src/main/java/org/genthz/ObjectFactory.java){:target="_blank"}
firstly to start using **GenThz Project**.
It is possible to use a default realization for generating values for your project.
```java
// The simplest way to create ObjectFactory
ObjectFactory objectFactory = new DashaObjectFactory();
```

All java objects are based on simple types such as ``long``, ``int``, ``java.lang.String`` and etc
(all simple types description is [here](topic_00_100_simple_types)). There
is default realization [`DashaDsl`]({{site.source_base}}/genthz-core/src/main/java/org/genthz/dasha/dsl/DashaDsl.java){:target="_blank"}
for generating all such types. And you can use a simple code to produce a new object:

```java
// Creating ObjectFactory using generation provider.
GenerationProvider gp = new DashaDsl()  // Enable Dsl configuration of object generation.
        .def()                          // Init instance builders and object fillers for simple types such as long, java.lang.String and etc.
        .build();                       // Create generation provider.
        
ObjectFactory objectFactory = new DashaObjectFactory(gp);
String value =  objectFactory.get(String.class);
```

```java
// Creating ObjectFactory from DashaDsl directly.
ObjectFactory dsl = new DashaDsl()  // Enable Dsl configuration of object generation.
        .def()                      // Init instance builders and object fillers for simple types such as long, java.lang.String and etc.
        ..objectFactory()           // Create generation provider.
        
String value =  objectFactory.get(String.class);
```

### Complex object generation

What about complex objects? It is very simple! All complex objects consist of [simple types](topic_100_simple_types)
such as ``long``, ``int``, ``java.lang.String``
and/or another complext objects. And Generated engine can automatically produce such objects automatically using
[`DefaultFiller`]({{site.source_base}}/genthz-core/src/main/java/org/genthz/function/DefaultFiller.java){:target="_blank"}:

```java
public class Person {
    private UUID uuid;
    private String name;
    private String lastName;
    private Date birthDate;

    public Person() {
    }
}

ObjectFactory objectFactory = new DashaObjectFactory();
Person value = objectFactory.get(Person.class);
```

### Custom object generation

Creation of the object consists of two phases: **instance creation** (using [constructor](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Constructor.html){:target="_blank"})
phase and **filling** one. There are default
instance builder and default object filler to create objects.
If you want to create object using instance builder only use
[InstanceBuilderFirst#simple()]({{site.source_base}}/genthz-core/src/main/java/org/genthz/dsl/InstanceBuilderFirst.java#L33){:target="_blank"}
. In this case **filling** phase will be skipped.

```java
import java.util.UUID;

public class Person {
    private UUID uuid;
    private String name;
    private String lastName;
    private Date birthDate;

    public Person() {
    }
}

Person value = new new DashaDsl() {
        {
            strict(Person.class)
                .simple(ctx -> {
                    Person person = new Person();
                    person.setUuid(UUID.randomUUID());
                    person.setName("Alex");
                    person.setLastName("Brown");
                    person.setBirthday(new Date());
                    
                    return person;
                    }))
        }
        }.def()
         .objectFactory()
         .get(Person.class);
```

Next sample illustrates using **filler** phase after custom instance creation. `name`, `lastName` and `birthday` fields
will be filled selected values, but fiedls: `uuid` and `idCard` will be filled by default filler.

```java
public class Person {
    private UUID uuid;
    private String name;
    private String lastName;
    private Date birthDate;
    private IdCard idCard;

    public Person() {
    }
}

public class IdCard {
    private UUID uuid;

    private String name;

    public IdCard() {
    }
}

    Person value = new DefaultConfiguration() {
        {
            reg(
                    strict(Person.class)
                            .use(s -> Arrays.asList(
                                    s.instance(ctx -> {
                                        Person person = new Person();

                                        person.setName("Alex");
                                        person.setLastName("Brown");
                                        person.setBirthday(new Date());

                                        return person;
                                    }),
                                    s.filler(Fillers.excluding("name", "lastName", "birthday"))
                                    )
                            )
            );
        }
    }.build().factory().build(Person.class);
```

In these cases, custom configuration provides creation of class instance and fills all the fields of the object
automatically by
using [DefaultFiller](https://github.com/mathter/genthz/blob/master/genthz-api/src/main/java/org/genthz/function/DefaultFiller.java){:target="_blank"}
excluding specified fields.
[In this section](topic_00_110_custom_configuration.html) you can get detailed information about custom configurations.

#### Recursion

What will happen if there is a recursion in the object structure?

```java
public class Recursion {
    private Recursion recursion;
}
```

Of couse ``java.lang.StackOverflowError`` will be generated!
To avoid this custom configuration or
[`DefaultConfiguration`]({{site.source_base}}/genthz-api/src/main/java/org/genthz/configuration/dsl/DefaultConfiguration.java)
{:target="_blank"} can be used. Next sample shows generating the object with 10 deep.

```java
public class Recursion {
    private Recursion recursion;
}

    Recursion value = new DefaultConfiguration() {
        @Override
        public Supplier<Long> maxGenerationDeep() {
            return () -> 10L;
        }
    }.build().factory().build(Recursion.class);
```

#### Path selectors

It is possible to specify custom
[InstanceBuildered](https://github.com/mathter/genthz/blob/master/genthz-api/src/main/java/org/genthz/configuration/dsl/InstanceBuildered.java){:target="_blank"}
or
[Fillered](https://github.com/mathter/genthz/blob/master/genthz-api/src/main/java/org/genthz/configuration/dsl/Fillered.java){:target="_blank"}
for selected object fields. Next example shows custom filler for field ``name`` of the ``Person`` class.

```java
public class Person {
    private UUID uuid;
    private String name;
    private String lastName;
    private Date birthDate;

    public Person() {
    }
}

    // Using instance builder
    Person value = new DefaultConfiguration() {
        {
            reg(
                    path("name").instance(ctx -> "Alex")
            );
        }
    }.build().factory().build(Person.class);

    // Using filler
    Person value = new DefaultConfiguration() {
        {
            reg(
                    path("name").filler((ctx, value) -> "Alex")
            );
        }
    }.build().factory().build(Person.class);

    // Type of the field can be specified
    Person value = new DefaultConfiguration() {
        {
            reg(
                    path("name").strict(String.class).instance(ctx -> "Alex")
            );
        }
    }.build().factory().build(Person.class);
```

It is possible to specify the nesting level of the field. An next samples shows filling of the ``person.name`` field of
the class `User`.

```java
public class User {
    private Person person;

    private String login;

    private String password;
}

public class Person {
    protected String name;

    protected String lastName;

    protected Date birthday;

    private IdCard idCard;
}

    User value = new DefaultConfiguration() {
        {
            reg(path("person/name").instance(c -> "Alex"));
        }
    }.build().factory().build(User.class);
```

You can use the ``*`` character to specify any matches any characters in the field name.

```java
User value=new DefaultConfiguration(){
        {
        reg(path("person/n*e").instance(c->"Alex"));
        }
        }.build().factory().build(User.class);
```

``/`` symbol at the start of the path points to root object. The next sample describes
the ``name`` (``java.lang.String`` class)
field any object assigned to the ``person`` field any another root object.

```java
public class User {
    private Person person;

    private String login;

    private String password;
}

public class Person {
    protected String name;

    protected String lastName;

    protected Date birthday;

    private IdCard idCard;
}

    User value = new DefaultConfiguration() {
        {
            reg(path("/person/name").instance(c -> "Alex"));
        }
    }.build().factory().build(User.class);
```

The next sample describes the ``name`` (``java.lang.String`` class)
field any object assigned to the ``father`` field any another object.

```java
public class User {
    private Person person;

    private String login;

    private String password;
}

public class Person {
    protected String name;

    protected String lastName;

    protected Date birthday;

    private IdCard idCard;
}

    User value = new DefaultConfiguration() {
        {
            reg(
                    path("person/name")
                            /*
                            Matches:
                            '/person/name'
                            or '/user/person/name'
                            or 'user/person/name'
                            and etc...
                            */
                            .instance(c -> "Alex")
            );
        }
    }.build().factory().build(User.class);
```

Fixed nested lavel:

```java

path("/name"); // first lavel
path("/../name"); // second lavel
path("/..5/name"); // 5th lavel
```

Please see more samples at [`PathTest`](https://github.com/mathter/genthz/blob/master/genthz-core/src/test/java/org/genthz/configuration/dsl/PathTest.java){:target="_blank"}

### Predefined instance builders
There are several predefined instance builders that can be used to generate intance of object.
Please see [`InstanceBuilders`](https://github.com/mathter/genthz/blob/master/genthz-api/src/main/java/org/genthz/configuration/dsl/InstanceBuilders.java){:target="_blank"}.
There are several methods for creating instance builders that are based on object constructors. These instance builders are in the package
[`org.genthz.function`](https://github.com/mathter/genthz/tree/master/genthz-api/src/main/java/org/genthz/function){:target="_blank"}.
There are samplace can be found in
[`InstanceBuildersTest`](https://github.com/mathter/genthz/blob/master/genthz-core/src/test/java/org/genthz/configuration/dsl/InstanceBuildersTest.java){:target="_blank"}.