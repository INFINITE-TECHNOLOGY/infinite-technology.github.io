# Infinite Technology ∞ Maven Repository

Welcome to [Infinite Technology ∞](https://i-t.io) **Maven Repository**.

This is a Maven compatible repository so it can be imported like any other Maven repository into compatible build system (Maven, Gradle, Grape, etc).

This repository contains artifacts related to Infinite Technology [Projects](https://i-t.io/index.html#projects).

All artifacts in this repository belong to group **io.infinite**.

> To host your artifacts in our repository create a 
[pull request](https://github.com/INFINITE-TECHNOLOGY/infinite-technology.github.io/compare?expand=1)
(m2 directory).

## Example Usage

```groovy
repositories {
    mavenCentral()
    mavenLocal()
    jcenter()
    maven {
        url "https://i-t.io/m2/"
    }
}
dependencies {
    compile 'io.infinite:blackbox:2.0.0'
}
```

## Artifacts

Please refer to the below list of available published artifacts:

|Name|Version|Comments|
|---|---|---|
|BlackBox|2.0.0|Demo version of BlackBox 2.0.0|