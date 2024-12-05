# Project-Reactor-and-Spring-WebFlux
Study notes for Java Project Reactor and Spring WebFlux. This document touches on the key concepts of Reactive Programming with Java and Spring Boot

> 📘 Best source of reference documentation: [Project Reactor - Documentation](https://projectreactor.io/docs)

## 1. What is Reactive Programming?

**Reactive programming** is a programming paradigm oriented around data flows and the propagation of change. This means that it should be possible to express static or dynamic data flows with ease in the programming languages used, and that the underlying execution model will automatically propagate changes through the data flow.
- Asynchronous and non blocking
- Data flows as an Event/Message driven stream
- Functional Style Code
- BackPressure on Data Streams
- Netty is a non-blocking server which uses Event Loop Model

<img width="749" alt="image" src="https://github.com/user-attachments/assets/eea309be-39c9-4620-bbaf-ed88925b4945">

<img width="755" alt="image" src="https://github.com/user-attachments/assets/becd4ae7-639e-4916-aa18-486667438a89">

## 2. Reactive Stream

Reactive Stream specification consists of:
1. Publisher
2. Subscriber
3. Subscription
4. Processor

### 2.1. Publisher

```
public interface Publisher<T> {
  public void subscribe(Subscriber<? super T> s);
}
```

A `Publisher` is a provider of a potentially unbounded number of sequenced elements, publishing them according to the demand received
from its `Subscriber`(s).
It represents the actual DataSource
- Database
- RemoteService, etc.,

### 2.2. Subscriber

```
public interface Subscriber<T> {
  public void onSubscribe(Subscription s);
  public void onNext(T t);
  public void onError(Throwable t);
  public void onComplete();
}
```

Will receive call to `onSubscribe(Subscription)` once after passing an instance of `Subscriber` to `Publisher.subscribe(Subscriber)`.

### 2.3. Subscription

```
public interface Subscription {
  public void request(long n);
  public void cancel();
}
```

A `Subscription` represents a one-to-one lifecycle of a `Subscriber` subscribing to a `Publisher`.
It can only be used once by a single `Subscriber`.
It is used to both signal desire for data and cancel demand (and allow resource cleanup).
It is the one which connects the app to the datasource.

### 2.4. Processor

```
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```

A Processor represents a processing stage—which is both a `Subscriber` and a `Publisher` and obeys the contracts of both.
Processor can behave as a Subscriber and Publisher

<img width="854" alt="image" src="https://github.com/user-attachments/assets/b3e3ea4d-4ed0-4543-8290-d646d8f0b494">

<img width="870" alt="image" src="https://github.com/user-attachments/assets/174522df-74b2-40b0-822c-ae94c46f4e5f">

## 3. Mono and Flux

### 3.1. Mono

```
public abstract class Mono<T> extends Object implements CorePublisher<T>
```

A Reactive Streams `Publisher` with basic rx operators that emits at most one item via the `onNext` signal then terminates with an `onComplete` signal (successful Mono, with or without value), or only emits a single `onError` signal (failed Mono).

<img width="857" alt="image" src="https://github.com/user-attachments/assets/9de24fb5-25bb-493b-9620-1568b40f0381">

### 3.2. Flux

```
public abstract class Flux<T> extends Object implements CorePublisher<T>
```

A Reactive Streams `Publisher` with rx operators that **emits 0 to N** elements, and then completes (successfully or with an error).

<img width="855" alt="image" src="https://github.com/user-attachments/assets/7b09b22e-fa45-41e4-9b28-7296715c386c">

### 3.3. Different operators for Mono and Flux

#### Creating a Mono
```
Mono<String> result = Mono.just("alex").log ();
```
**Output:** `["alex"]`

#### Mono FlatMap
```
Mono<List<String>> result = Mono.just("alex")
  .flatMap(s -> (
    var stringArray = s.split("");
    return Mono.just(List.of(stringArray)) ;
  })
  .log() ;
```
**Output:** `List.of ("a", "l", "e", "x")`

#### Mono FlatMapMany
```
Flux<String> result = Mono.just(("alex"))
  .flatMapMany(s -> {
    var strings = name.split("");
    return Flux.fromArray(strings);
  })
  .log();
```
**Output:** `["a", "l", "e", "x"]`

#### Mono Concat With
```
var aMono = Mono.just("A");
var bMono = Mono.just("B");
Flux<String> result = aMono.concatWith(bMono).log();
```
**Output:** `["A", "B"]`

#### Mono Merge With
```
var aMono = Mono.just("A");
var bMono = Mono.just("B");
Flux<String> result = aMono.mergeWith(bMono).log();
```
**Output:** `["A", "B"]`

#### Creating a Flux
```
List<String> INPUT_LIST = List.of("alex", "ben", "chloe");
Flux<String> result = Flux.fromIterable(INPUT_LIST).log();
```
**Output:** `["alex", "ben", "chloe"]`

#### Flux is Immutable
```
Flux<String> namesFlux = Flux.fromIterable(INPUT_LIST);
namesFlux.map(String::toUpperCase);
return namesFlux;
```
**Output:** `["alex", "ben", "chloe"]`

#### Map operator
```
Flux<String> result = Flux.fromIterable(INPUT_LIST)
  .map(String::toUpperCase)
  .log();
```
**Output:** `["ALEX", "BEN", "CHLOE"]`

#### Filter operator
```
Flux<String> result = Flux.fromIterable(INPUT_LIST).map(String::toUpperCase)
  .filter(name -> name.length() > 3)
  .map(name -> name.length() + "-" + name)
  .log();
```
**Output:** `["4-ALEX", "5-CHLOE"]`

#### FlatMap
```
Flux<String> result = Flux.fromIterable(INPUT_LIST)
  .map(String::toUpperCase)
  .filter(name -> name.length() > 3)
  .flatMap(name -> {
    var strings = name.split("");
    var delay = rand.nextInt(1000);
    return Flux.fromArray(strings)
      .delayElements(Duration.ofMillis(delay));
  })
  .log();
```
**Output:** `["C", "H", "A", "L", "O", "E", "L", "E", "X"]`

The sequence is not maintained

#### ConcatMap
```
Flux<String> result = Flux.fromIterable(INPUT_LIST)
  .map(String::toUpperCase)
  .filter(name -> name.length() > 3)
  .concatMap(name -> {
    var strings = name.split("");
    var delay = rand.nextInt(1000);
    return Flux.fromArray(strings)
      .delayElements(Duration.ofMillis(delay));
  })
  .log();
```
**Output:** `["A", "L", "E", "X", "C", "H", "L", "O", "E"]`

Sequence is maintained

#### Transform
```
Flux<String> result = Flux.fromIterable(INPUT_LIST)
  .transform(filterFluxLength(3))
  .log();

// transform takes in a FunctionalInterface Function<T, R>
// where <T> – the type of the input to the function
// and <R> – the type of the result of the function

Function<Flux<String>, Flux<String>> filterFluxLength(int stringLength) {
  return name -> name.map(String::toUpperCase)
    .filter(s -> s.length() > stringLength)
    .flatMap(name -> {
      var strings = name.split("");
      return Flux.fromArray(strings);
    });
```
**Output:** `["A", "L", "E", "X", "C", "H", "L", "O", "E"]`

#### Default If Empty
```
Flux<String> result = Flux.fromIterable(INPUT_LIST)
  .transform(filterFluxLength(6))
  .defaultIfEmpty("default")
  .log();
```
**Output:** `["default"]`

#### Switch If Empty
```
Flux<String> result = Flux.fromIterable(INPUT_LIST)
  .transform(filterFluxLength(6))
  .switchIfEmpty(Flux.just("default").transform(filterFluxLength(6)))
  .log();
```
**Output:** `["D", "E", "F", "A", "U", "L", "T"]`

This allows you to switch the flux input in case the first one returns an empty Flux.

#### Concat
```
var abcFlux = Flux.just("A", "B", "C");
var defFlux = Flux.just("D", "E", "F");
Flux<String> result = Flux.concat(abcFlux, defFlux).log();
```
**Output:** `["A", "B", "C", "D", "E", "F"]`

#### Concat With
```
var abcFlux = Flux.just("A", "B", "C");
var defFlux = Flux.just("D", "E", "F");
Flux<String> result = abcFlux.concatWith(defFlux).log();
```
**Output:** `["A", "B", "C", "D", "E", "F"]`

#### Merge
```
var abcFlux = Flux.just("A", "B", "C")
  .delayElements(Duration.ofMillis(100));
var defFlux = Flux.just("D", "E", "F")
  .delayElements(Duration.ofMillis(125));
Flux<String> result = Flux.merge(abcFlux, defFlux).log();
```
**Output:** `["A", "D", "B", "E", "C", "F"]`

#### Merge With
```
var abcFlux = Flux.just("A", "B", "C")
  .delayElements(Duration.ofMillis(100));
var defFlux = Flux.just("D", "E", "F")
  .delayElements(Duration.ofMillis(125));
Flux<String> result = abcFlux.mergeWith(defFlux).log();
```
**Output:** `["A", "D", "B", "E", "C", "F"]`

#### Merge Sequential
```
var abcFlux = Flux.just("A", "B", "C")
  .delayElements(Duration.ofMillis(100));
var defFlux = Flux.just("D", "E", "F")
  .delayElements(Duration.ofMillis(125));
Flux<String> result = Flux.mergeSequential(abcFlux, defFlux).log();
```
**Output:** `["A", "B", "C", "D", "E", "F"]`

#### Zip - using source1, source2 and BiFunction
```
var abcFlux = Flux.just("A", "B", "C");
var defFlux = Flux.just("D", "E", "F");
Flux<String> result = Flux.zip(abcFlux, defFlux, (first, second) -> first + second).log();
```
**Output:** `["AD", "BE", "CF"]`

#### Zip - using 2 to 8 sources
```
var abcFlux = Flux.just("A", "B", "C");
var defFlux = Flux.just("D", "E", "F");
var num123Flux = Flux.just("1", "2", "3");
var num456Flux = Flux.just("4", "5", "6");
Flux<String> result = Flux.zip(abcFlux, defFlux, num123Flux, num456Flux)
  .map(tuple4 -> tuple4.getT1() + tuple4.getT2() + tuple4.getT3() + tuple4.getT4())
  .log();
```
**Output:** `["AD14", "BE25", "CF36"]`

#### Zip With - using source1, source2 and BiFunction
```
var abcFlux = Flux.just("A", "B", "C");
var defFlux = Flux.just("D", "E", "F");
Flux<String> result = abcFlux.zipWith(defFlux, (first, second) -> first + second)
  .log();
```
**Output:** `["AD", "BE", "CF"]`

## 4. Spring WebFlux

> 📘 Official documentation for [Spring WebFlux](https://docs.spring.io/spring-framework/reference/web/webflux.html#webflux)

Spring WebFlux provides 2 ways to create RESTful APIs:
1. [Annotated Controllers](https://docs.spring.io/spring-framework/reference/web/webflux/controller.html)
2. Functional Endpoints

### 4.1. Annotated Controllers

The implementation of Controller, Model classes, Exception Handling/Controller Advice, Service classes and Repositories are exactly same as done in Spring Boot non reactive programming.
1. Reactive Controllers generally return a `Mono` or `Flux`. More details about various possibilities that can be used is explained in the below section.
2. Model classes generally do not have any `Mono` or `Flux` variables. When using the object of the model classes we wrap them inside a `Mono` or `Flux`.
3. Repository interfaces generally extend a Reactive variance like `ReactiveCrudRepository`, rather than a normal `CrudRepository`
4. Usage of a ControllerAdvice or a Service class is exactly same as done in Non reactive programming.

#### Handling different ResponseEntity

Since the return type of a reactive function is either Mono or Flux, Spring WebFlux provides different ways how you can choose to
customize your response (code, header, body, etc)
1. `ResponseEntity<Mono<T>>` or `ResponseEntity<Flux<T>>`
2. `Mono<ResponseEntity<T>>`
3. `Mono<ResponseEntity<Mono<T>>>` or `Mono<ResponseEntity<Flux<T>>>`
For more details read the documentation - [Web on Reactive Stack - ResponseEntity](https://docs.spring.io/spring-framework/reference/web/webflux/controller/ann-methods/responseentity.html)



