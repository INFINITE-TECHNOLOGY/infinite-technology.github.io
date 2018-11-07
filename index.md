# Home

## About

Infinite Technology ∞ is a non-profit based, non-commercial open-source software foundation established in 2018.

## Contacts

[Infinite Technology at GitHub](https://github.com/INFINITE-TECHNOLOGY/infinite-technology.github.io)

[Discussion and issue tracking](https://github.com/INFINITE-TECHNOLOGY/infinite-technology.github.io/issues)

## Mission

Our mission is to streamline Web and Mobile app development by standardizing and systemizing the primary infrastructure components:

- Logging code automation
- User login & API Security
- HATEOAS (HAL) Frontend SDK

## Projects

### BlackBox

BlackBox is a logging code automation solution.

BlackBox Annotation automatically injects a lot of logging code into user-defined Groovy methods/constructors without affecting the user program logic.

Granularity of injected code can be defined by the user (programmer) up to:

* Method Exception handling transformation, Method Exceptions logging (exception and causing method arguments are logged)
* Method transformation, Method invocation logging (method arguments and result are logged)
* Statement transformation, Statement-level logging
* Expression transformation, Expression-level logging

References:
* [BlackBox Documentation](https://github.com/INFINITE-TECHNOLOGY/BLACKBOX/wiki)
* [BlackBox at GitHub](https://github.com/INFINITE-TECHNOLOGY/BLACKBOX/)
* [Groovydoc](https://i-t.io/BlackBox/groovydoc/2_0_x/)
* [XSD](https://i-t.io/BlackBox/xsd/2_x_x/BlackBox.xsd)

**Try it now!** *Run the below code in Groovy Console:*

```groovy
@GrabResolver(name='infinite.io', root='https://i-t.io/m2') 
@Grab(group='io.infinite', module='blackbox', version='2.0.0')

import io.infinite.blackbox.*

@BlackBox(blackBoxLevel=BlackBoxLevel.EXPRESSION)
String foobar(String foo) {
    String bar = "bar"
    String foobar = foo + bar
    return foobar
}
System.setProperty("blackbox.mode", BlackBoxMode.SEQUENTIAL.value())

foobar("foo")
```