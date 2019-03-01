# Infinite Technology ∞ Bobbin

## Purpose

This blueprint defines the solution to process input logging data and store it in a structured way by the means of 
implementing an Slf4j-compatible logging library written in Groovy language.

## In short

Bobbin creates isolated log files per thread with simple configuration using Groovy Scripts to compute dynamic parameter 
values during run-time.

## Introduction

### Foreword

Possibly due to a certain lack of theoretical provision, Logging has always been an zone of turbulence in the Java world.
This caused a diversity of logging libraries to appear over the time, such as:
- Log4j
- Java Util Logging
- Commons Logging
- Logback
- Log4j2

Aiming to address the limitations of others, each one of them unfortunately has been bringing its own shortcomings.

While from the code standardization perspective things got improved with the adaptation of Slf4j as a logging 
abstraction layer, there are still unresolved issues in the existing logging frameworks.

As an Open Source Community, we are taking an initiative to come up with a new experimental approach to create a 
lightweight yet functionally saturated Logger using up-to-date technology such as JSR 223 and Groovy.

### The Problems

> Existing logging solutions provide only partial support for scripting in configuration.

We end up doing declarative programming in logger configuration (XML, JSON, YAML, programmatic configuration) rather 
than configuring logger dynamically interpret configuration values during run-time using imperative scripting.

Let's consider the example of Logback filter configuration to log only INFO messages:
```xml
<filter class="ch.qos.logback.classic.filter.LevelFilter">
  <level>INFO</level>
  <onMatch>ACCEPT</onMatch>
  <onMismatch>DENY</onMismatch>
</filter>
```

This is a typical example of declarative XML programming.

Note: Logback supports Groovy Script filter - but it is applicable only on specific appender, not on Logger.

And there is no support for Groovy message text formatting (encoder pattern).

> Complicated and verbose configuration

Let's take examples of Logback and Log4j2.

It is impossible to configure log level on the appender level.

- Appenders are defined separately from Loggers and Loggers reference the appender using "AppenderRef".
- Only Loggers accept Level and Class Name configuration

Let's say we would like to exclude Debug of class Foo from a specific log file, without affecting other log files and classes.

In Logback this is solved using Groovy Script Filter on Appender, but when we have many appenders - configuration size grows
quickly.

> Log files per Log Level

We could not find how to store each log level in individual file with efficient configuration.

Existing possibilities require duplication of appenders per log level.

> Overriding Root Level on Appender

While it is possible to reduce log level of specific appenders using filter, it is not possible to use Root Log Level
as only default value, while allowing to **redefine** level of the specific appenders (even increasing it).

> There is a disconnect between how the log data is produced by the application and how it is being consumed by the Logger

As a historical practice, existing Loggers are defined either in:
- Class static field
- Instance field - causing overhead in initialization and unnecessary isolation

This reduces the scope of each specific logger instance to the context of its owner class or object in terms of consumption of 
log data.

However the application flow is **not driven by the class structure** rather being reflected into a dynamic 
**execution stack**, which is having a guaranteed immutable anchor - the **thread** in which the stack is being executed.

In practice this misconception causes functional limitations for existing loggers such as:
- Complicated configuration of file naming, log separation and archiving
- (Depending on log declaration - static or instance):
  - Static logger: shareable by all threads, but lacking easy per-thread configuration 
  - Instance logger: causing overhead in initialization and unnecessary isolation

For example:
- Logback supports maximum 1 "discriminator" in "SiftingAppender"
- SiftingAppender has limitations in support of archiving and rolling policies
- Complicated configuration of "RoutingAppender" in Log4j2

### The Solution

> Full support for scripting in logger run-time configuration files

Bobbin is using the configuration as a placeholder for Groovy scripts that define the run-time operation of the logger.

Here is how the above "filter" example would have looked like when logger is configured to interpret imperative run-time scripts:
```json
{
  "levels": "['info'].contains(level)"
}
```

Each and every aspect of the logger is configurable using Groovy Script:
- Log levels
- Class Names
- Message Formats
- File Names

> Easy and concise configuration

Bobbin does not need Encoders, Patterns, Filters, Discriminators and many other excessive stuff.

It is configured by a very small number of primary conceptual entities:
1) Levels
2) Class Names
3) Destinations
4) Message format

There are also very few supplementary parameters:

1) File name
2) Zip file name

Very easy to remember isn't it? But so much freedom to manipulate.

> Log files per Log Level

Just put "${level}" into the file name mask parameter of Bobbin configuration.

> By it's nature, Bobbin Logger is thread-isolated.

Bobbin Logger always has 1 logger instance per thread.

This architecture is well aligned with producer-consumer model of Java applications (log data is produced by threads).

This simplifies the Bobbin code-base by easing the requirements for thread-safety and performance congestion.

On a functional level this helps to allow flexibility in log file separation and naming.

For example, there is no more need to use MDC/discriminators for separating log files per thread - the "threadName"
token is readily available in file name configuration.

Furthermore - if someone will write their own Bobbin Destination - they **do not have to worry** about log separation
and thread safety:

> **Destinations belong to logger instance specific to 1 and only 1 thread.**

## Key Features

### Integration

Bobbin fully supports Slf4j integration mechanism and can be used in any project with Slf4j - just by adding Bobbin 
dependency during compilation.

Furthermore, Bobbin can be used both in Java and Groovy projects as it is available as Maven dependency.

### Configuration

#### Events

In Object-oriented terms, existing logging frameworks consider log messages as events belonging to the same hierarchy:
- Trace
- Debug extends Trace
- Info extends Debug
- Warning extends Info
- Error extends Warning

This happens due to *comparable* nature of conventional Log Levels: e.g. Trace < Debug < Info < Warning < Error.

This is a traditional paradigm which is so old that people just take it is as something granted and immutable.

We change this paradigm.

> Bobbin considers log messages as separate *independent events* with different types (as per their level).

With Bobbin it is possible to configure enabled logger levels in a granular way without filter, using array:

```json
{
  "levels": "['info'].contains(level)"
}
```

#### Classes

A conventional thinking: a logger is assigned to one and only one class name or package name.

The Bobbin thinking: a logger can have multiple classes and package names attached to it.

```json
{
  "classes": "className.contains('io.infinite.') || className.endsWith('.HttpSender')"
}
```

#### Scripting

All parameters of Bobbin configuration are interpreted as Groovy scripts:
- Log Levels
- Class Names
- File Names
- Message format

Here is an example of message format configuration:

```json
{
  "format": "\"${dateTime}|${level}|${threadName}|${className}|${event.message}\""
}
```

### Archiving

Bobbin provides a simple way of file archiving:
- for each log message, file name is re-computed based on configuration
- if the file name is changed since the previous message - the previous file is archived

Additionally if Bobbin finds that file with the new file name is already existing - it will automatically archive that 
file to prevent data loss. This is especially useful for house-keeping during multiple restarts of application - old logs
are stored in archives.   

## Solution Architecture

<img src="https://i-t.io/Bobbin/documentation/images/Bobbin_Diagram.png" alt="Bobbin Architecture Diagram"/>

- Actual Bobbin loggers are stored in ThreadLocal Singleton - 1 and only 1 per thread
- Logging messages are dispatched to Bobbin loggers via BobbinNameAdapters which can be referenced in instance or static fields
in classes
- BobbinNameAdapter is lightweight class and provides its individual instances via LoggerFactory.getLogger with quick initialization.
- All thread-specific Bobbin loggers share same configuration (Bobbin.json) and can multiple File Destinations
- File Destination holds File Name Mask which is computed for every Log message, therefore resulting in multiple 
log files attached to every File Destination.

## Usage

### Import dependency

Bobbin artifacts are hosted in [Infinite Technology Maven repository](https://bintray.com/infinite-technology/m2):

<a href='https://bintray.com/infinite-technology/m2/bobbin?source=watch' alt='Get automatic notifications about new "bobbin" versions'><img src='https://www.bintray.com/docs/images/bintray_badge_color.png'></a>

[ ![Download](https://api.bintray.com/packages/infinite-technology/m2/bobbin/images/download.svg) ](https://bintray.com/infinite-technology/m2/bobbin/_latestVersion)

#### Maven

Add Infinite Technology Maven repository to pom.xml:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<settings xsi:schemaLocation='http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd'
          xmlns='http://maven.apache.org/SETTINGS/1.0.0' xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance'>
    
    <profiles>
        <profile>
            <repositories>
                <repository>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                    <id>bintray-infinite-technology-m2</id>
                    <name>bintray</name>
                    <url>https://dl.bintray.com/infinite-technology/m2</url>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                    <id>bintray-infinite-technology-m2</id>
                    <name>bintray-plugins</name>
                    <url>https://dl.bintray.com/infinite-technology/m2</url>
                </pluginRepository>
            </pluginRepositories>
            <id>bintray</id>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>bintray</activeProfile>
    </activeProfiles>
</settings>
```

Add Bobbin dependency to pom.xml:

```xml
<dependency>
  <groupId>io.infinite</groupId>
  <artifactId>bobbin</artifactId>
  <version>2.0.0</version>
</dependency>
```

#### Gradle

Add repository to build.gradle:

```groovy
repositories {
    maven {
        url  "https://dl.bintray.com/infinite-technology/m2" 
    }
}
```

Add Bobbin dependency to build.gradle:

```groovy
compile 'io.infinite:bobbin:2.0.0'
```

### Groovy

Use [Slf4j](http://docs.groovy-lang.org/latest/html/documentation/#xform-Slf4j) Groovy annotation:

```groovy
@groovy.util.logging.Slf4j
class Greeter {
    void greet() {
        log.debug 'Called greeter'
        println 'Hello, world!'
    }
}
```

### Slf4j

Use explicit Slf4j logger factory:

```groovy
import org.slf4j.LoggerFactory
import org.slf4j.Logger

class Greeter {
    private static final Logger log = LoggerFactory.getLogger(Greeter)
    void greet() {
        if (log.isDebugEnabled()) {
            log.debug 'Called greeter'
        }
        println 'Hello, world!'
    }
}
```

### Standalone

```groovy
import io.infinite.bobbin.Bobbin
import io.infinite.bobbin.BobbinFactory

class Bar {

    Bobbin bobbin = BobbinFactory.getBobbin()
    static String className = "Bar"

    void foo() {
        bobbin.debug(className, "Hello world")
    }

}
```

### Configuration

Bobbin is configured using only 1 file: Bobbin.json

> IMPORTANT: Bobbin configuration is conceptually different from other Logging Frameworks.
Please do not try to interpret Bobbin configuration in the legacy context. Let's start with a fresh thinking.

Bobbin configuration is very intuitive and directly reflects on Bobbin run-time operation.

#### Bobbin.json

This is one and only configuration file needed for Bobbin.

Bobbin.json should be located either in application working directory or in resources directory.

Bobbin.json is de-serialized using Jackson Databind into BobbinConfiguration class.

##### BobbinConfiguration Structure

```groovy
/**
* Root logger configuration
*/
class BobbinConfig {

    /**
    * Logger destination (appender)
    */
    static class Destination {
        String name //destination class name
        Map<String, String> properties = [:] //destination-specific configuration parameters
        String format //message format Groovy Script
        String levels //level Groovy Script
        String classes //class name Groovy Script
        String dateFormat = "yyyy-MM-dd"
        String dateTimeFormat = "yyyy-MM-dd HH:mm:ss:SSS"
    }

    String levels = "false" //root logger level Groovy Script
    String classes = "false" //root logger class name Groovy Script
    List<Destination> destinations = [] //logger destinations (appenders) - 0 to ∞

}
```

##### Example configuration

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
      "classes": "className.contains('io.infinite.')"
    },
    {
      "name": "io.infinite.bobbin.destinations.FileDestination",
      "properties": {
        "fileName": "\"./LOGS/WARNINGS_AND_ERRORS_${date}.log\"",
        "zipFileName": "\"./LOGS/WARNINGS_AND_ERRORS_${date}.zip\"",
        "cleanupZipFileName": "\"${origFileName}_${System.currentTimeMillis().toString()}.zip\""
      },
      "format": "\"${dateTime}|${level}|${threadName}|${className}|${event.message}\\n\"",
      "levels": "['warn', 'error'].contains(level)"
    },
    {
      "name": "io.infinite.bobbin.destinations.ConsoleDestination",
      "format": "\"${dateTime}|${level}|${threadName}|${className}|${event.message}\\n\"",
      "levels": "['warn', 'error'].contains(level)"
    }
  ]
}
```

#### Root

At the root level we define global Bobbin levels and classes, using Groovy Script:
```
  "levels": "['debug', 'info', 'warn', 'error'].contains(level)",
  "classes": "all"
```

These settings identify global Bobbin logging level as well as enabled classes.

Underlying Bobbin Destinations can override these settings, but unless they explicitly specify them - the settings
are inherited by default.

##### levels

In the root configuration, "levels" script supports the following special tokens:
- level (String)
  - Based on logging method used to initiate message logging, possible values are same as in Slf4j:
    - error
    - warn
    - info
    - debug
    - trace
- all (Boolean)

##### classes

In the root configuration, "classes" script supports the following special tokens:
- className (String) - as passed to logger factory or logging method
- all (Boolean)

##### Destinations

Bobbin Destination is similar to Appender, but with possibility to re-define its logging level and enabled classes.

**NOTE:** All Groovy Scripts for Destination Configuration support the below tokens (per event basis):
- level (String)
  - Based on logging method used to initiate message logging, possible values are same as in Slf4j:
    - error
    - warn
    - info
    - debug
    - trace
- all (Boolean)
- event - logging event itself (io.infinite.bobbin.Event)
- threadName (String) - current Thread Name
- date (String) - event date formatted using "dateFormat"
- dateTime (String) - event date+time formatted using "dateTimeFormat"
- className (String) -  - as passed to logger factory or logging method

* **Common settings** - these settings are common for all destinations
  * **name** - destination class name; any class extending io.infinite.bobbin.destinations.Destination Abstract Class.
    * currently 2 classes are supported:
      * io.infinite.bobbin.destinations.ConsoleDestination - output to System.out.
      * io.infinite.bobbin.destinations.FileDestination - storage in physical files with archiving
  * **levels** - Groovy Script overriding the Root "levels" parameter for this Destination
  * **classes** - Groovy Script overriding the Root "classes" parameter for this Destination
  * **format** - Groovy Script for message format
  * **dateFormat** - Groovy Script to format event date
  * **dateTimeFormat** - Groovy Script to format event dateTime
* **FileDestination**
  * **Properties**
    * **fileKey** - Groovy Script for identifying static file name to group files for archiving
    * **fileName** - Groovy Script identifying file name with variable portion to choose files for archiving
    * **zipFileName** - Groovy Script to name archived files. **Computed at time of initial file creation**
    * **cleanupZipFileName** - Groovy Script to archive (cleanup) existing log files during application restart.
      * Supports "origFileName" token
* **ConsoleDestination**
  * Console destination does not have its own settings.

## Example Use Cases

### Log file per log level

Just add:
```groovy
${level}
```
to file name mask:

```json
{
  "levels": "all",
  "classes": "all",
  "destinations": [
    {
      "name": "io.infinite.bobbin.destinations.FileDestination",
      "properties": {
        "fileKey": "threadName + level",
        "fileName": "\"./LOGS/${level}/${level}_${date}.log\"",
        "zipFileName": "\"./LOGS/${level}/ARCHIVE/${level}_${date}.zip\"",
        "cleanupZipFileName": "\"${origFileName}_${System.currentTimeMillis().toString()}.zip\""
      },
      "format": "\"${dateTime}|${level}|${threadName}|${className}|${event.message}\\n\"",
      "classes": "className.contains('io.infinite.')"
    }
  ]
}
```

### Log file per thread

Just add:
```groovy
${threadName}
```
to file name mask:

```json
{
  "levels": "all",
  "classes": "all",
  "destinations": [
    {
      "name": "io.infinite.bobbin.destinations.FileDestination",
      "properties": {
        "fileKey": "threadName + level",
        "fileName": "\"./LOGS/${threadName}/${threadName}_${date}.log\"",
        "zipFileName": "\"./LOGS/${threadName}/ARCHIVE/${threadName}_${date}.zip\"",
        "cleanupZipFileName": "\"${origFileName}_${System.currentTimeMillis().toString()}.zip\""
      },
      "format": "\"${dateTime}|${level}|${threadName}|${className}|${event.message}\\n\"",
      "classes": "className.contains('io.infinite.')"
    }
  ]
}
```

### Batch processing application

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
      "classes": "className.contains('io.infinite.')"
    },
    {
      "name": "io.infinite.bobbin.destinations.FileDestination",
      "properties": {
        "fileName": "\"./LOGS/WARNINGS_AND_ERRORS_${date}.log\"",
        "zipFileName": "\"./LOGS/WARNINGS_AND_ERRORS_${date}.zip\"",
        "cleanupZipFileName": "\"${origFileName}_${System.currentTimeMillis().toString()}.zip\""
      },
      "format": "\"${dateTime}|${level}|${threadName}|${className}|${event.message}\\n\"",
      "levels": "['warn', 'error'].contains(level)"
    },
    {
      "name": "io.infinite.bobbin.destinations.ConsoleDestination",
      "format": "\"${dateTime}|${level}|${threadName}|${className}|${event.message}\\n\"",
      "levels": "['warn', 'error'].contains(level)"
    }
  ]
}
```
