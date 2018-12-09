# Infinite Technology ∞ Bobbin

## Purpose

This blueprint defines the solution to process input logging data and store it in a structured way by the means of 
implementing an Slf4j-compatible logging library written in Groovy language.

## In short

Bobbin creates separate log files per thread supporting Groovy Scripts in all aspects of logging configuration.

## Introduction

### Foreword

In Java world logging has always been an underdeveloped area.
This caused a diversity of logging libraries:
- Log4j
- JUL
- Apache Commons Logging
- Logback
- Log4j2

Everyone of them trying to cover the limitations of another, while introducing new pitfalls.

Things got improved with the adaptation of Slf4j as a logging abstraction layer.

It is time to take the full advantage of Slf4j and create an ultimate logger.

### The Problem

> Existing logging solutions have only partial support for scripting in configuration.

We end up doing declarative programming in logger configuration (XML, JSON, YAML, programmatic configuration) rather 
than configuring logger to interpret imperative scripts (Groovy, JavaScript) at run-time.

Let's consider the example of Logback configuration to filter to log only INFO messages:
```xml
<filter class="ch.qos.logback.classic.filter.LevelFilter">
  <level>INFO</level>
  <onMatch>ACCEPT</onMatch>
  <onMismatch>DENY</onMismatch>
</filter>
```

This is a typical example of XML programming.

> Existing logging solutions are either under- or over-isolated.

Loggers are defined either:
- In Class static field - shareable by all threads, but lacking easy per-thread configuration 
- In instance field - causing overhead in initialization and unnecessary isolation

This causes functional limitations in configuration, file naming, separation and archiving.

For example:
- Logback supports maximum 1 "discriminator" in "SiftingAppender"
- SiftingAppender has limitations in support of archiving and rolling policies
- Complicated configuration of "RoutingAppender" in Log4j2

There is a whole subclass of Java applications which is harmed due to this - batch processing multi-threaded applications.

As per our subjective understanding the situation happened because the existing logging solutions were developed without 
keeping in mind Financial and other business areas where such applications are broadly used.

### The Solution

> Bobbin is using the configuration as a placeholder for Groovy scripts that define the run-time operation of the logger.

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

> By it's nature, Bobbin is thread-isolated.

Effectively Bobbin always has 1 logger instance per thread.

Technically this greatly simplifies the code-base by easing the requirements for thread-safety and performance congestion.

On a functional level this helps to employ much more simple file naming and archiving configurations.

For example, there is no more need to use MDC for separating log files per thread - the "threadName" token is readily 
available in file name configuration.  

## Key Features

### Integration

Bobbin fully supports Slf4j integration mechanism and can be used in any project with Slf4j - just by adding Bobbin 
dependency during compilation.

Furthermore, Bobbin can be used both in Java and Groovy projects.

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

Bobbin employs a very simple concept for file archiving:
- for each log message, file name is re-computed based on configuration
- if the file name is changed since the previous message - the previous file is archived

Additionally if Bobbin finds that file with the new file name is already existing - it will automatically archive that 
file to prevent data loss. This is especially useful for house-keeping during multiple restarts of application - old logs
are stored in archives.   

### Thread Isolation

Due to its thread-isolated nature, Bobbin does suffer from thread-safety constraints - such as synchronized IO.
Thus in theory it can achieve more performance under heavy load in multi-threaded environment, comparing to existing loggers.

## Principles

### Code simplicity

Bobbin source code is extremely simple with total 13 classes (including 3 Slf4j binding classes).

Jackson Databind is used to deserialize the Bobbin JSON configuration which directly reflects into runtime logger objects.

Simplicity is one of base principles allowing to by bypass a steep maturation curve associated with existing logging solutions.

We save code lines and efforts thanks to using Groovy language features:
- Dynamic programming
- Memoization
- Profiling using AST annotations

Only 2 destinations are supported for output:
- Console
- File

But it covers ~70% of the use cases.

### Configuration simplicity

One of the main burdens of existing Loggers is excessive verbose configuration.

A configuration to save each log level into separate file exceeds 100 lines in Logback.xml and it grows progressively.

Same configuration takes ~30 lines in Bobbin and it scales effectively.

### Thread safety

Bobbin is intended to be used primarily in multi-threaded Batch applications, thus Thread-safety questions is the foundational
principle of Bobbin design.

### Compatibility

Bobbin should be possible to integrate standalone (or using Slf4j) in any valid Java or Groovy environment:

- Application servers
- Spring context; Spring Boot/Grails
- Core Java

### Performance optimization

## Solution Architecture

## Usage

### Import dependency

### Groovy

### Slf4j

### Standalone

### Configuration

#### Bobbin.json

##### Structure

##### Example configuration

#### Root

##### levels

##### classes

##### Destinations

* Common settings
  * name
  * levels
  * classes
  * format
  * dateFormat
  * dateTimeFormat
* FileDestination
   * Properties
     * fileKey
     * fileName
     * zipFileName
     * cleanupZipFileName
* ConsoleDestination

## Use Cases