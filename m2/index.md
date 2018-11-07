# Infinite Technology ∞ Maven Repository

Welcome to Infinite Technology ∞ Maven Repository.

This is a Maven compatible repository so it can be imported like any other Maven repository into compatible build system (Maven, Gradle, Grape, etc).

This repository contains artifacts related to Infinite Technology [Projects](https://i-t.io/index.html#Projects).

All artifacts in this repository belong to group **io.infinite**.

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

|Name|Version|Comments|
|---|---|---|
|BlackBox|2.0.0|Demo version of BlackBox 2.0.0|