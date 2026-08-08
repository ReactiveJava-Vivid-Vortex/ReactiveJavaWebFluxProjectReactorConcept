# Maven/Gradle Setup

## In Simple Terms

To use Project Reactor, you just add it as a dependency in your build file. If
you're using Spring Boot with WebFlux, you already have it — it comes along
automatically. You can also use it on its own, in any plain Java project.

## Simple Example

### Maven (`pom.xml`)

```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.6.0</version>
</dependency>

<!-- Optional but highly recommended for testing -->
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <version>3.6.0</version>
    <scope>test</scope>
</dependency>
```

### Gradle (`build.gradle`)

```groovy
dependencies {
    implementation 'io.projectreactor:reactor-core:3.6.0'
    testImplementation 'io.projectreactor:reactor-test:3.6.0'
}
```

If you're building a Spring Boot WebFlux app, you usually just add this instead:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

...and `reactor-core` tags along automatically.

## Why It Matters

Getting the setup right from the start — especially adding `reactor-test` for
`StepVerifier` — means you can start writing and testing reactive code right
away, instead of hunting down missing classes later.
