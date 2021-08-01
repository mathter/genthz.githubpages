---
layout: page_doc
title: Docs V1
menu_type: main
page_type: page_doc
---

# Documentation

## Getting started

To make use of GenThz within your project, simply add the maven dependency available from Maven Central repository, like
so:

### Installation

```xml

<dependency>
    <groupId>org.genthz</groupId>
    <artifactId>genthz-api</artifactId>
    <version>1.1.0</version> <!-- or latest version -->
</dependency>
<dependency>
    <groupId>org.genthz</groupId>
    <artifactId>genthz-core</artifactId>
    <version>1.1.0</version> <!-- or latest version -->
</dependency>
```

<div class="alert alert-block alert-danger">
<b>Please verify the latest version</b>
</div>
### Sources
There are source codes at [github.com](https://github.com/mathter/genthz){:target="_blank"}
### HelloWorld sample
It is required to get [`ObjectFactory`]({{site.source_base}}/genthz-api/src/main/java/org/genthz/ObjectFactory.java){:target="_blank"}
firstly to start using **GenThz Project**.
It is possible to use a default configuration for generating values for your project.
```java
// Getting producer
ObjectFactoryProducer producer = ObjectFactoryProducer.producer()
ObjectFactory objectFactory = producer.factory();
```

All java objects are based on simple types such as ``long``, ``int``, ``java.lang.String`` and etc
(all simple types description is [here](topic_00_100_simple_types)). There
is [`DefaultConfiguration`]({{site.source_base}}/genthz-api/src/main/java/org/genthz/configuration/dsl/DefaultConfiguration.java){:target="_blank"}
for generating all such types. And you can use a simple code to produce a new object:

```java
ObjectFactoryProducer producer=ObjectFactoryProducer.producer();
ObjectFactory objectFactory=producer.factory();

String value = objectFactory.build(String.class);
// Or more complexity
String value = objectFactory.build(Init.builder(String.class).build());
```

### Complex object generation

What about complex objects? It is very simple! All complex objects consist of [simple types](topic_100_simple_types)
such as ``long``, ``int``, ``java.lang.String``
and/or another complext objects. And Generated engine can automatically produce such objects:

```java
public class Person {
    private UUID uuid;
    private String name;
    private String lastName;
    private Date birthDate;

    public Person() {
    }
}

ObjectFactoryProducer producer = ObjectFactoryProducer.producer();
ObjectFactory objectFactory = producer.factory();
Person person = objectFactory.build(Person.class);
```

### Custom object generation

Creation of the object consists of two phases: **instance creation** phase and **filling** one. If class don't have
default constructor these phases are united. There are default instance builder and default object filler to create
objects. If you want to create object using instance builder only use
[instanceBuilder](https://github.com/mathter/genthz/blob/master/genthz-api/src/main/java/org/genthz/configuration/dsl/InstanceBuildered.java){:target="_blank"}

```java
public class Person {
    private UUID uuid;
    private String name;
    private String lastName;
    private Date birthDate;

    public Person() {
    }
}

ObjectFactory objectFactory = ObjectFactoryProducer
        .producer()
        .factory(new DefaultConfiguration() {
            {
                reg(
                        strict(Person.class)
                                .instanceBuilder(context -> {
                                    Person person = new Person();

                                    person.setName("John");
                                    person.setLastName("Braun");
                                    person.setBirthDate(new Date());
                                    person.setUuid(UUID.randomUUID());

                                    return person;
                                })
                );
            }
        });

Person person = objectFactory.build(Person.class);
```

It's possible to no skip **filler** phase but skip auto filling same fields of the object.

```java
public class Person {
    private UUID uuid;
    private String name;
    private String lastName;
    private Date birthDate;

    public Person() {
    }
}

ObjectFactory objectFactory = ObjectFactoryProducer
        .producer()
        .factory(new DefaultConfiguration() {
            {
                reg(
                        strict(Person.class)
                                .defaultFiller()
                                .excluding("name", "lastName")
                                .custom((context, object) -> {
                                    object.setName("Olen");
                                    object.setLastName("Brown");
                                    return object;
                                })
                );
            }
        });

Person person = objectFactory.build(Person.class);
```

In these cases, custom configuration provides creation of class instance and fills all the fields of the object
automatically by
using [``DefaultFiller``]({{site.source_base}}/genthz-api/src/main/java/org/genthz/configuration/dsl/DefaultFiller.java){:target="_blank"}
excluding specified fields.
[In this section](topic_00_110_custom_configuration.html) you can get detailed information about custom configurations.

#### Recursion

What will happen if there is a recursion in the object structure?

```java
public class WithRecursion {
    private UUID uuid;
    private WithRecursion next;
}
```

Of couse ``java.lang.StackOverflowError`` will be generated!
To avoid this custom configuration can be used. Next sample shows generating the object with 10 deep.

```java
public class WithRecursion {
    private UUID uuid;
    private WithRecursion next;
}

ObjectFactory factory = ObjectFactoryProducer
        .producer()
        .factory(new DefaultConfiguration() {
            @Override
            public Supplier<Long> maxGenerationDeep() {
                return () -> 10L;
            }
        });

WithRecursion value = factory.build(WithRecursion.class);

```

#### Path selector

It is possible to specify custom
[InstanceBuilder]({{site.source_base}}/genthz-api/src/main/java/org/genthz/InstanceBuilder.java){:target="_blank"} or
[Filler]({{site.source_base}}/genthz-api/src/main/java/org/genthz/Filler.java) for selected object fields. Next example
shows custom filler for field ``name`` of ``Persone`` class.

```java
public class Person {
    private UUID uuid;
    private String name;
    private String lastName;
    private Date birthDate;

    public Person() {
    }
}

ObjectFactory objectFactory = ObjectFactoryProducer
        .producer()
        .factory(new DefaultConfiguration() {
            {
                reg(
                        strict(Person.class)
                                .defaultFiller()
                                .excluding("name")
                                .custom((context, object) -> {
                                    object.setName("Olen");
                                    return object;
                                })
                );
            }
        });

Person person = objectFactory.build(Person.class);
```

It is possible to specify the nesting level of the field. An next samples shows filling of the ``father.name`` field.

```java
public class Family {
    private Person father;
    ...
};

public class Person {
    private UUID uuid;
    private String name;
    private String lastName;
    private Date birthDate;

    public Person() {
    }
    ...
}

ObjectFactory objectFactory = ObjectFactoryProducer
        .producer()
        .factory(new DefaultConfiguration() {
            {
                reg(
                        path("/father/name")
                                .strict(String.class)
                                .instanceBuilder(c -> "Olen")
                                .simple()
                );
            }
        });

Family person = objectFactory.build(Family.class);
```

You can use the ``*`` character to specify any matches any characters in the field name.

```java
public class MyConfiguration extends DefaultConfiguration {
    {
        reg(
                path("/fa*/name")
                        .strict(String.class)
                        .instanceBuilder(c -> "Olen")
        );
    }
}
```

``/`` symbol at the start of path points to root object. The next sample describes the ``name`` (``java.lang.String`` class)
field any object assigned to the ``father`` field any another root object.
```java
public class Family {
    private Person father;
}

public class Person {
    private UUID uuid;
    private String name;
    private String lastName;
    private Date birthDate;

    public Person() {
    }
}

public class MyConfiguration extends DefaultConfiguration {
    {
        reg(
                path("/father/name")
                        .strict(String.class)
                        .instanceBuilder(c -> "Olen")
        );
    }
}
```

The next sample describes the ``name`` (``java.lang.String`` class)
field any object assigned to the ``father`` field any another object.
```java
public class Family {
    private Person father;
}

public class Person {
    private UUID uuid;
    private String name;
    private String lastName;
    private Date birthDate;

    public Person() {
    }
}

public class MyConfiguration extends DefaultConfiguration {
    {
        reg(
                path("father/name")
                        /*
                        Matches:
                        '/father/name'
                        or 'family/father/name'
                        or '/family/father/name'
                        and etc...
                        */
                        .strict(String.class)
                        .instanceBuilder(c -> "Olen")
        );
    }
}
```

Fixed nested lavel:
```java

path("/name"); // first lavel
        
path("/../name"); // second lavel
        
path("/..5/name"); // 5th lavel
```