---
layout: post
title: "[Design Pattern] Dependency Injection"
date: 2020-05-30 18:00:00
author: Jewoo.Song
categories:
  - Design Pattern
tags:
  - Design Pattern
  - Dependency Injection
  - Angular Dependency Injection
  - DI 패턴
  - Dependency
  - Angular DI
---

## DI(Dependency Injection) 패턴

소프트웨어 엔지니어링에서 `Dependency Injection`은 하나의 객체가 다른 객체의 Dependency(의존성)를 제공하는 테크닉이다.

즉, 컴포넌트가 어떤 서비스를 사용할 것인지 지정하는 대신 컴포넌트에게 무슨 서비스를 사용할 것인지를 말해주는 것이다.

이때 `컴포넌트에게 무슨 서비스를 사용할 것인지를 말해주는 것`이 주입(Injection)을 의미한다.

의존성 주입은 역 제어(Inversion of Control, IOC) 테크닉의 한 형태로 어떤 서비스를 호출하려는 컴포넌트는 그 서비스가 어떻게 구성되었는지 알지 못해야 한다.

컴포넌트는 서비스 제공에 대한 책임을 외부 코드(Injector)에게 위임하고 Injector는 이미 존재하거나 Injector에 의해 생성된 서비스를 컴포넌트에게 주입하고 컴포넌트는 서비스를 사용한다.

이는 컴포넌트가 Injector와 서비스에 대해 알 필요가 없음을 의미한다.

즉, 컴포넌트는 서비스의 인터페이스에 대해서만 알면 되고 이는 `구성`의 책임으로부터 `사용`의 책임을 구분한다.

### 1. Dependency

`Dependency`는 코드에서 두 클래스 간의 관계를 뜻한다.

일반적으로 아래와 같은 코드에서 A 클래스에서 B 클래스를 사용할 때 A 클래스가 B 클래스에 대해서 Dependency(의존성)을 가지고 있다고 말한다. 또는 B 클래스가 A 클래스의 Dependency라고도 말할 수 있다.

```ts
class A {
  private b: B;

  constructor() {
    this.b = new B("song");
  }

  printB() {
    this.b.print();
  }
}
```

Dependency에 대해 좀 더 파고들어보면 A 클래스 안에 B 클래스에 대한 구체적인 클래스에 대한 모든 정보를 뜻하는데, 이런 정보를 가지고 있기 때문에 의존관계(dependency relationship)이 생기고 이러한 의존관계를 추상적으로 Dependency라 하다.

> UML에서의 Dependency
> A dependency is a semantic connection between dependent and independent model elements.
> It exists between two elements if changes to the definition of one element (the server or target) may cause changes to the other (the client or source)
>
> https://en.wikipedia.org/wiki/Class_diagram#Dependency

#### Dependency 관점에서의 DI

Dependency Injection은 이러한 Dependency를 제거하는 패턴이다.
실제로 Dependency가 제거되는 것은 아니고 Dependency를 외부로 제거하는 것이라고 할 수 있다.

> 클래스 간의 의존관계가 있을 때 컴파일 타임의 의존관계를 제거하고 런타임 의존관계로 만들어 주고 결합도(coupling)을 낮춰주는 것이 Dependency Injection이다.

### 2. Dependency Injection 구현

Dependency Injection 구현을 설명하기 위한 예제로는 특정 감독이 제작한 영화 목록을 제공하는 컴포넌트를 사용한다.

> Martin Fowler의 [Inversion of Control Containers and the Dependency Injection pattern](https://www.martinfowler.com/articles/injection.html)에서 인용해서 typescript로 구현했습니다.

아래와 같은 예제가 있을 때 먼저 finder 객체를 통해 전체 영화 목록을 구한 뒤, 그 목록에서 특정 감독이 제작한 영화를 추려내는 작업을 한다.

핵심은 finder와 MovieLister를 어떻게 연결하느냐 하는 것인데, 이때 MovieLister의 moviesDirectedBy 메서드의 코드는 finder 객체에 대해 의존성을 가지고 있다.

```ts
class MovieLister {
  // ...
  public moviesDirectedBy(director: string) {
    const allMovies = this.finder.findAll();

    return allMovies.filter((movie) => movie.director === director);
  }
}
```

아래와 같이 인터페이스를 정의함으로써 findAll 메서드의 사용법을 정의하고 MovieLister가 인터페이스에 의존하게 함으로써 MovieLister와 MovieFinder의 결합도를 낮출 수 있다.

```ts
interface MovieFinder {
  findAll(): Array<Movie>;
}

class MovieLister {
  private finder?: MovieFinder;

  constructor() {
    // finder 할당
  }

  public moviesDirectedBy(director: string) {
    const allMovies = this.finder.findAll();

    return allMovies.filter((movie) => movie.director === director);
  }
}
```

그러나 여전히 실제 Movie 목록을 구하기 위해서는 MovieLister 클래스가 사용할 MovieFinder의 구현 클래스를 알아야 한다.

즉, 위 코드에서 finder 할당 부분에서는 실제 MovieFinderImpl(ColonDelimitedMovieFinder) 클래스의 인스턴스를 생성해서 할당해 주어야 하므로 구현 클래스에 의존성이 생기게 된다.

```ts
 constructor() {
    // finder 할당
    this.finder = new ColonDelimitedMovieFinder("movies1.txt");
  }
```

다음 UML은 MovieLister 클래스가 MovieFinder 인터페이스와 그 구현 클래스에 모두 의존하고 있는 것을 보여준다.

![img](/assets/img/designpattern/dependencyinjection1.png)

이를 해결하는 패턴이 바로 Dependency Injection이다.

Dependency Injection을 구현하는 방식에는 세 가지 종류가 있다.

- 생성자 방식
- setter 방식
- 인터페이스 방식

이 중 생성자 방식을 간단하게 구현해 볼 수 있다.

> 이번 예제에서는 간단하게 구현하는 것을 보고, 이후에 Angular의 Dependency Injection에 대해서도 포스팅을 할 예정인데, 그 포스팅에서는 Angular의 DI와 Injector의 구현에 대해서도 일부 다뤄보도록 하겠습니다.

```ts
class MovieLister {
  private finder?: MovieFinder;

  constructor(movieFinder: MovieFinder) {
    this.finder = movideFinder;
  }

  public moviesDirectedBy(director: string) {
    const allMovies = this.finder.findAll();

    return allMovies.filter((movie) => movie.director === director);
  }
}

// ...

const movieLister = new MovieLister(
  new ColonDelimitedMovieFinder("movies1.txt")
);
movieLister.moviesDirectedBy("song");
```

위와 같이 사용함으로써 MovieLister와 MovieFinderImpl 간의 Dependency를 제거할 수 있다.

![img](/assets/img/designpattern/dependencyinjection2.png)

### 3. Dependency Injection의 장점

- 객체 간의 의존성을 낮출 수 있다.
- 코드의 재사용성을 높여준다.
- Unit Test에 용이해진다.
  - 실제 서비스 구현체 대신에 Mock을 사용한 테스트를 쉽게 수행할 수 있다.
- 객체간의 커플링을 느슨하게하고 유연한 코드를 작성할 수 있다.
- 좀 더 보기 쉬운 코드를 작성할 수 있다.

### 4. 정리

Dependency Injection은 프로그램의 디자인이 느슨하게 커플링되도록 하고, `의존관계 역전 원칙(Dependency Inversion Principle)`과 `단일 책임 원칙(single responsibility principles)`을 따르도록 클라이언트의 생성에 대한 의존성을 클라이언트의 행위로부터 분리하는 것이다.

이는 클라이언트가 의존성을 찾기 위해 그들이 사용하는 시스템에 대해 알도록 하는 `서비스 로케이터 패턴(service locator pattern)`과 정반대되는 것이다.

> SOLID 원칙
>
> 좋은 객체지향 설계를 위해서 다음 5원칙을 따르는 것이 좋다.
> 5가지 원칙의 앞 글자를 따서 SOLID 원칙이라 한다.
>
> - SRP(Single Responsibility Principle, 단일 책임 원칙)
>   - 소프트웨어의 설계 부품은 단 하나의 책임만을 가져야 한다.
>   - 즉 소프트웨어를 수정할 이유가 오직 하나여야 한다.
> - OCP(Open-Closed Principle, 개방-폐쇄 원칙)
>   - 기존의 코드를 변경하지 않고(Closed) 기능을 수정하거나 추가할 수 있도록(Open) 설계해야 한다.
>   - 자주 변경되는 내용은 수정하기 쉽게 설계하고, 변경되지 않아야 하는 것은 수정되는 내용에 영향을 받지 않게 한다.
>   - 예) 인터페이스
> - LSP(Liskov Substitution Principle, 리스코프 치환 원칙)
>   - 자식 클래스는 부모 클래스에서 가능한 행위를 수행할 수 있어야 한다.
>   - 즉, 부모 클래스와 자식 클래스 사이의 행위에는 일관성이 있어야 한다.
> - ISP(Interface Segregation Principle, 인터페이스 분리 원칙)
>   - 한 클래스는 자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다. 하나의 일반적인 인터페이스보다는, 여러 개의 구체적인 인터페이스가 낫다.
> - DIP(Dependency Inversion Principle, 의존관계 역전 원칙)
>   - 첫 번째, 상위 모듈은 하위 모듈에 의존해서는 안된다. 둘 다 추상화에 의존해야 한다
>   - 두 번째, 추상화는 세부사항에 의존하지 않는다. 세부사항이 추상화에 의존하여 달라져야 한다.
>   - 의존 관계를 맺을 때, 변화하기 쉬운 것보단 변화하기 어려운 것에 의존해야 한다.
>     - 이때 변화하기 쉬운 것이란 구체적인 것을 말하고, 변화하기 어려운 것이란 추상적인 것을 말한다.
>     - 객체지향 관점에서 변화하기 쉬운 것이란 구체화된 클래스를 말하고, 변화하기 어려운 것이란 인터페이스를 의미한다.
>   - DIP를 만족한다는 것은 의존관계를 맺을 때, 상위 모듈(클라이언트 클래스)는 하위 모듈(서비스 클래스)이 되는 구체적인 클래스보다 인터페이스나 추상 클래스와 관계를 맺는다는 것을 의미한다.
>   - DIP를 만족하면 `Dependency Injection` 패턴을 적용해서 변화에 유연한 설계를 할 수 있다.

### 참고

- [Inversion of Control Containers and the Dependency Injection pattern](https://www.martinfowler.com/articles/injection.html)
  - https://javacan.tistory.com/entry/120
- https://ko.wikipedia.org/wiki/%EC%9D%98%EC%A1%B4%EC%84%B1_%EC%A3%BC%EC%9E%85
- [Dependency란 무엇인가?](https://groups.google.com/forum/#!topic/ksug/77jLQan8Q-E)
- [SOLID 원칙](https://dev-momo.tistory.com/entry/SOLID-%EC%9B%90%EC%B9%99)
- [객체지향 5대 원칙](https://blog.martinwork.co.kr/theory/2017/12/10/oop-solid-principle.html)
