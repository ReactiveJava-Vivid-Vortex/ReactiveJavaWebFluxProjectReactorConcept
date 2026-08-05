# Maven/Gradle Setup

## In Simple Terms

To use Project Reactor, you just need to add its dependency to your build tool. If
you're using Spring Boot with WebFlux, it's already included transitively — but
you can also use Reactor standalone in any plain Java project.

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

If you're building a Spring Boot WebFlux app, you typically just add:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

...and `reactor-core` comes along automatically as a transitive dependency.

## Why It Matters

Getting the setup right — including the `reactor-test` module for `StepVerifier` —
means you can start writing and testing reactive pipelines immediately, without
chasing down missing classes or version mismatches later.
