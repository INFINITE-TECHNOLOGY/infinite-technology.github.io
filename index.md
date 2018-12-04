# Mission

Infinite Technology ∞ is a non-profit based, non-commercial open-source software foundation established in 2018.

Our mission is to streamline Web and Mobile app development by standardizing and systemizing the primary infrastructure components:

* Logging code automation
* User login & API Security
* HATEOAS (HAL) Frontend SDK

# Projects

Our projects are hosted in [Infinite Technology Maven Repository](https://i-t.io/m2/)

## BlackBox

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

## Bobbin

Bobbin is a Groovy Slf4j-compatible logger designed for multi-threaded applications.

Bobbin leverages the concept of Logback sifting appender while providing much more easier configuration.

References:
* [Bobbin at GitHub](https://github.com/INFINITE-TECHNOLOGY/BOBBIN/)

**Sample configuration:**

```json
{
  "levels": "['debug', 'info', 'warn', 'error'].contains(level)",
  "classes": "all",
  "destinations": [
    {
      "name": "io.infinite.bobbin.destinations.FileDestination",
      "properties": {
        "fileKey": "threadName + level",
        "fileName": "\"./LOGS/${threadName}/${level}/${threadName}_${level}_${date}.log\"",
        "zipFileName": "\"./LOGS/${threadName}/${level}/ARCHIVE/${threadName}_${level}_${date}.zip\"",
        "cleanupZipFileName": "\"${origFileName}_${System.currentTimeMillis().toString()}.zip\""
      },
      "format": "\"${dateTime}|${level}|${threadName}|${className}|${event.message}\\n\"",
      "levels": "all",
      "classes": "className.contains('io.infinite.')"
    },
    {
      "name": "io.infinite.bobbin.destinations.FileDestination",
      "properties": {
        "fileName": "\"./LOGS/App_${date}.log\"",
        "zipFileName": "\"./LOGS/App_${date}.zip\"",
        "cleanupZipFileName": "\"${origFileName}_${System.currentTimeMillis().toString()}.zip\""
      },
      "format": "\"${dateTime}|${level}|${threadName}|${className}|${event.message}\\n\"",
      "levels": "['warn', 'error'].contains(level)",
      "classes": "all"
    },
    {
      "name": "io.infinite.bobbin.destinations.ConsoleDestination",
      "format": "\"${dateTime}|${level}|${threadName}|${className}|${event.message}\\n\"",
      "levels": "['warn', 'error'].contains(level)",
      "classes": "all"
    }
  ]
}
```