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

* [See more](https//i-t.io/BlackBox/)

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