# FleetCheck – Build Systems Lab

This project is intentionally incomplete. Follow the worksheet in the order given.

Expected final application output:

```
FleetCheck 1.0
Vehicles loaded: 4
Vehicles requiring service: 2
Average mileage: 37000 km
```

Do not copy the solution POM. The objective is to observe how each build change alters the result.

# Outputs:

## Evidence 1

Command: `mvn clean package`

```
[ERROR] /Users/diogo/Documents/Docs/Uni/5thSem/SoftwareQuality/worksheet4/maven/FleetCheck_Starter/src/main/java/pt/upt/fleetcheck/App.java:[4,38] package com.fasterxml.jackson.databind does not exist
```

Caused by the import on line 4 of `App.java`: `import com.fasterxml.jackson.databind.ObjectMapper;` (line 3, `com.fasterxml.jackson.core.type.TypeReference`, fails the same way). Jackson is not declared in `pom.xml`, so it is not on the compile classpath.

## Step 2 question: why is this a better failure than the one from Step 1?

In Step 1 the build failed at compilation, which only means the project is misconfigured (a dependency is missing). Once Jackson is declared, the code compiles and the build moves on to the test phase, so any failure from then on is about the behaviour of the code, which is what a build should be checking. (In my starter project `src/test` was empty, so the test phase reported "No tests to run".)

## Evidence 4

Command: `java -jar target/fleetcheck-1.0.0.jar` (default JAR, before the Shade plugin)

```
no main manifest attribute, in target/fleetcheck-1.0.0.jar
```

The default JAR only contains the project's own classes and resources, with no `Main-Class` in the manifest and no Jackson. The Shade plugin produces `fleetcheck-1.0.0-all.jar`, which copies the Jackson classes into the JAR and writes `Main-Class: pt.upt.fleetcheck.App` into the manifest, so `java -jar` runs it on its own.

## Step 5 question: which hidden environmental assumption did the wrapper remove?

The assumption that the right version of Maven is installed on the machine. `mvnw` uses the Maven version pinned in `.mvn/wrapper/maven-wrapper.properties` (Maven 3.9.12), so every developer and the CI server build with the same Maven.

## Evidence 7

The SBOM lists the full resolved dependency graph, not only the dependencies typed in `pom.xml`. The build log says `Creating BOM version 1.6 with 3 component(s)`: only `jackson-databind` is declared, but it depends on `jackson-core` and `jackson-annotations`, so Maven pulls them in transitively and they end up in the shaded JAR. The SBOM records everything that is actually shipped.
