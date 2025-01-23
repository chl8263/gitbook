# Annotation processor  란?

## lombok 이란?

java 에서 lombok 을 사용하여 많은 기능들을 편리하게 사용할 수있다.

예를들어 setter, getter, hashcode 같은 코드들을 annotation 하나만으로 사용이 가능하다.

```java
@Getter
@Setter
@EqualsAndHashCode
public class Account {
    private int id;
    private String name;
}
```

<figure><img src="../../../.gitbook/assets/Screen Shot 2022-10-20 at 7.59.08 PM.png" alt=""><figcaption></figcaption></figure>

build 하위에 java byte 코드를 보면 아래와 같이 마법같이 코드가 생성된다.

```java
public class Account {
    private int id;
    private String name;

    public Account() {
    }

    public int getId() {
        return this.id;
    }

    public String getName() {
        return this.name;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof Account)) {
            return false;
        } else {
            Account other = (Account)o;
            if (!other.canEqual(this)) {
                return false;
            } else if (this.getId() != other.getId()) {
                return false;
            } else {
                Object this$name = this.getName();
                Object other$name = other.getName();
                if (this$name == null) {
                    if (other$name != null) {
                        return false;
                    }
                } else if (!this$name.equals(other$name)) {
                    return false;
                }

                return true;
            }
        }
    }

    protected boolean canEqual(Object other) {
        return other instanceof Account;
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        result = result * 59 + this.getId();
        Object $name = this.getName();
        result = result * 59 + ($name == null ? 43 : $name.hashCode());
        return result;
    }
}
```

**lombok 의 원리는 컴파일 시점에 애노테이션 프로세서를 사용하여 소스코드의** [**AST**](https://javaparser.org/inspecting-an-ast/)**(abstract syntax tree)를 조작함.**

## AST란?

> AST
>
> [컴퓨터 과학](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99)에서 **추상 구문 트리**(abstract syntax tree, AST), 또는 간단히 **구문 트리**(syntax tree)는 [프로그래밍 언어](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D_%EC%96%B8%EC%96%B4)로 작성된 [소스 코드](https://ko.wikipedia.org/wiki/%EC%86%8C%EC%8A%A4_%EC%BD%94%EB%93%9C)의 추상 [구문](https://ko.wikipedia.org/wiki/%ED%86%B5%EC%82%AC%EB%A1%A0) 구조의 [트리](https://ko.wikipedia.org/wiki/%ED%8A%B8%EB%A6%AC)이다. 이 트리의 각 노드는 소스 코드에서 발생되는 구조를 나타낸다. 구문이 추상적이라는 의미는 실제 구문에서 나타나는 모든 세세한 정보를 나타내지는 않는다는 것을 의미한다.

> 추상 구문 트리는 [컴파일러](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%8C%8C%EC%9D%BC%EB%9F%AC)에 널리 사용되는 [자료 구조](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0)인데, 이는 프로그램 코드의 구조를 표현하는 프로퍼티이기 때문이다. AST는 일반적으로 컴파일러의 [구문 분석](https://ko.wikipedia.org/wiki/%EA%B5%AC%EB%AC%B8_%EB%B6%84%EC%84%9D) 단계의 결과물이다. 컴파일러가 요구하는 여러 단계를 통해 프로그램의 중간 표현의 역할을 하며 컴파일러의 최종 결과물에 대해 강력한 영향을 준다.

### 컴파일러와의 관계

컴파일러가 필요한 언어에서 AST를 접할 수 있는 이유는 AST가 컴파일 단계 중 **구문 분석**(syntax analyzing) 단계의 결과물이기 때문이다.

**컴파일 단계**

```
1. 어휘 분석(lexical analyze) or 스캔(scan)
2. 구문 분석(syntax analyzung) - 결과물로 구문트리(syntax tree) 또는 추상 구문 트리(abstract syntax tree)가 생성됨!
3. 의미 분석(semantic analysis)
4. 중간 표현의 생성(intermediate representation)
5. 코드 생성(code generation)
6. 최적화(optimization)
7. 어셈블러
```

자바 뿐만 아니라 대부분의 프로그래밍 언어로 작성된 소스(개발자에 의해 작성된 소스)는 컴파일러(자바에서는 javac)에 의해 컴파일 과정에서 AST를 구성한다.

AST 는 프로그램 코드(소스)의 구조를 표현하는 프로퍼티이며, 컴파일러의 구문 분석 결과물이라고 할 수 있다.

다만, 모든 세세한 정보를 나타내지는 않기에 (세미콜론, 괄호, 구두점, 주석 생략) AST 로부터 원래 소스로 완벽하게 복구하는 것은 거의 불가능.



예를들어 아래와 같은 코드를 AST로 나타낸다면 어떻게 나올까?

```java
class Test { 
    int age = 30; 
}
```

> 해당 소스의 구문분석을 위해 아래의 라이브러리를 사용
>
> ```kotlin
> implementation("com.github.javaparser:javaparser-symbol-solver-core:3.24.7")
> ```

위의 코드를 구문분석하여 아래와 같이 계층형 트리구조로 표현할 수 있다.

```
root(Type=CompilationUnit): 
    types: 
        - type(Type=ClassOrInterfaceDeclaration): 
            isInterface: "false"
            name(Type=SimpleName): 
                identifier: "Test"
            members: 
                - member(Type=FieldDeclaration): 
                    variables: 
                        - variable(Type=VariableDeclarator): 
                            initializer(Type=IntegerLiteralExpr): 
                                value: "30"
                            name(Type=SimpleName): 
                                identifier: "age"
                            type(Type=PrimitiveType): 
                                type: "INT"
```

AST는 프로그래밍 언어로 작성된 소스 코드의 추상 구문 구조의 트리이며, 이 트리의 각 노드는 소스코드에서 발생되는 구조를 나타냅니다.

쉽게 말하면 우리가 작성한 **소스코드를 문법에 맞게 노드들로 쪼개서 만든 트리이다**.

## Controversy of lombok&#x20;

결국 Lombok 은 아래와 같이 구문분석 단계에서 AST 를 조작하여(setter, getter 같은 코드를 트리에 추가하는 작업 등) 컴파일러 구문분석 다음 단계에 넘기는 일을 진행하게된다.

<figure><img src="../../../.gitbook/assets/Screen Shot 2022-10-20 at 8.37.35 PM.png" alt=""><figcaption></figcaption></figure>

이로인해 많은 논란거리를 피할 수는 없었다고한다. 아래와 같이 어떤사람은 "lombok 은 완전히 '핵'이다" 라고 말하는 사람도있다.

> [http://jnb.ociweb.com/jnb/jnbJan2010.html#controversy](http://jnb.ociweb.com/jnb/jnbJan2010.html#controversy)
>
> ```
> It's a total hack. Using non-public API. Presumptuous casting (knowing that an
> annotation processor running in javac will get an instance of JavacAnnotationProcessor,
> which is the internal implementation of AnnotationProcessor (an interface), which
> so happens to have a couple of extra methods that are used to get at the live AST).
>
> On eclipse, it's arguably worse (and yet more robust) - a java agent is used to inject
> code into the eclipse grammar and parser class, which is of course entirely non-public
> API and totally off limits.
>
> If you could do what lombok does with standard API, I would have done it that way, but
> you can't. Still, for what its worth, I developed the eclipse plugin for eclipse v3.5
> running on java 1.6, and without making any changes it worked on eclipse v3.4 running
> on java 1.5 as well, so it's not completely fragile.
> ```

lombok 은 위와같은 원리로 <mark style="color:blue;">**컴파일 시점에**</mark>**&#x20;**<mark style="color:red;">**애노테이션 프로세서**</mark><mark style="color:blue;">**를 사용하여 소스코드의 AST를 조작한다.**</mark>

## Annotation processer 에 대해

### Annotation 이란?

애노테이션이란 무엇일까?

사실 우리 모두가 이미 정의된 애노테이션을 쓰고 있다. 예를 들면, @Override 어노테이션을 사용하여 메소드를 재정의하고 싱글톤 패턴을 사용하기 위해 @Singleton을 사용하고 @NonNull, @StringRes, @IntRes 등과 같은 애노테이션을 사용다.&#x20;

애노테이션은 자바 소스 코드에 추가 할 수있는 메타 데이터의 한 형태이다. 클래스, 인터페이스, 메소드, 변수, 매개 변수 등에 추가 할 수 있습니다. 애노테이션은 소스 파일에서 읽을 수도 있고, 컴파일러에 의해 생성된 클래스 파일에 내장되어 읽힐 수도 있으며, Runtime에 Java VM에 의해 유지되어 리플렉션에 의해 읽어 낼 수도 있다.

### Annotation Processor란?

일반적으로 애노테이션에 대한 코드베이스를 검사, 수정 또는 생성하는데 사용된다. 본질적으로 애노테이션 프로세서는 java 컴파일러의 플러그인의 일종.

* Annotation Processor는 실제로 javac 컴파일러의 일부이므로 모든 처리가 런타임보다는 컴파일시간에 발생한다.

### Annotation processor 의 동작 순서

1. 자바 컴파일러가 컴파일을 수행한다. (자바 컴파일러는 애노테이션 프로세서에 대해 미리 알고 있어야 함.)
2. 실행되지 않은 애노테이션 프로세서들을 수행. (각각의 프로세서는 모두 각자에 역할에 맞는 구현이 되어있어야합니다.)
3. 프로세서 내부에서 애노테이션이 달린 Element(변수, 메소드, 클래스 등)들에 대한 처리. (보통 이곳에서 자바 클래스를 생성.)
4. 컴파일러가 모든 애노테이션 프로세서가 실행되었는지 확인하고, 그렇지 않다면 반복해서 위 작업을 수행.

### Annotation processor 생성하기

Annotation processor 는  `javax.annotation.processing` 의 `AbstractProcessor` 를 상속받아 사용한다.

```java
public class NiceProcessor extends AbstractProcessor {
    ....
}
```

AbstractProcessor 의 대표적인 메소드를 몇가지 살펴보자.



1. getSupportedAnnotationTypes

해당 메소드는 어떤 annotation 을 지원할 것인지에 대해 표시하는 기능이다.

return 부분에 Set으로 지원하고자 하는 Annotation class 의 이름을 적어주면 `process` 메소드에 지원하는 annotation 를 다룰 수 있다.

```java
public class NiceProcessor extends AbstractProcessor 
   @Override
    public Set<String> getSupportedAnnotationTypes() {  // Which annotations are supported
        return new HashSet<>(Collections.singletonList(Nice.class.getName()));
    }
}
```

2\. getSupportedSourceVersion

어떤 Java 버전을 지원할 것인가에 대한 것.

```java
public class NiceProcessor extends AbstractProcessor 
    @Override
    public SourceVersion getSupportedSourceVersion() {  // Which source code are supported
        return super.getSupportedSourceVersion();
    }
}
```

3\. process

실제 annotation processor 가 실행되는 구간.

지원하는 annotation 이 class, interface, method 에 붙어있는지 확인 가능하고 또한 에러까지 낼 수 있다.

해당 메소드에서 실질적으 Java 코드를 삽입하는 구간.

```java
public class NiceProcessor extends AbstractProcessor 

    // element : The unit on source code. For instance class element, method element etc..
    // round : Annotation processor work like round, the processor find element sequentially. When find one of element the processor will process next element.
        // It is working like Spring filter chain.
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(Nice.class);
        for (Element element : elements) {
            if (element.getKind() != ElementKind.INTERFACE) {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, "Nice annotation can not be used on " + element.getSimpleName());
            } else {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.NOTE, "processing " + element.getSimpleName());
            }

            // create java file in here
            Filer filer = processingEnv.getFiler();
            try {
                JavaFile.builder(className.packageName(), magicMoja)
                        .build()
                        .writeTo(filer);
            } catch (IOException e) {
                processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR, "FATAL
                        ERROR: " + e);
            }
            ///////////////////////////
        }

        return true; // If return true, other annotation processor never process this annotation
    }
}
```
