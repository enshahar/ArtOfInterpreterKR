# 인터프리터의 기술(간략하게 번역/스칼라로 번안)

## 소개

### 모듈화

프로그램으로 만드는 대상(entity)은 아주 복잡하다. 큰 프로그램을 잘 만들기 위해서는 이런 복잡성을 잘 다뤄야 한다. 복잡성을 다루는 기법 중 대부분은 문제를 거의 독립적인 더 작은 하위 문제로 나누는 방법이다. 이렇게 나누고 나면 각각의 하위 문제를 한번에 하나씩 다른 문제는 신경쓰지 않고 해결할 수 있다. 각 하위문제를 풀면 다시 이를 한데 합칠 수 있어야 한다. 이렇게 프로그래밍 문제를 관리 가능한 하위 단위로 나눠서 풀었을 때, 우리는 전체 프로그램을 모듈화(modular) 했다고 말한다. 잘 설계한 프로그래밍 언어는 모듈화된 프로그램을 구축하도록 지원해야 한다.

한가지 분할 방법으로는 어떤 언어를 사용하는 공통 패턴을 한 패키지로 묶는 데 있다. 이를 추상화(abstraction)라 한다. 예를 들어 알골(Algol)의 for는 if와 goto를 사용하는 일반적인 패턴을 포장한 것이다. 공통 패턴을 묶는 것은 단순히 타이핑 횟수를 절약하기 위해서만 필요한 것이 아니다. 잘 만든 추상화는 전체를 내부 구현과 분리해 사고할 수 있는 더 높은 수준의 개념역할을 할 수 있다. 일단 이런 추상화를 완성하면, 프로그래머는 그 내부와 관계 없이 이를 사용 가능하다. 왜냐하면 그 추상화 패키지가 다루는 내용이 문제와 딱 맞아 떨어지기 때문이다.

패키지는 그 패키지를 사용한 주변 문맥과 독립적일 때 다른 패키지와의 가장 쓸모가 많다. 이런 패키지를 일컬어 참조 투명(referentially transparen)하다고 말한다. 직관적으로, 참조 투명성은 프로그램의 각 부분의 의미가 명확하고 변하지 않기 때문에, 각 부분을 믿고 의존할 수 있다는 의미이다. 특히나, 어떤 모듈 내부에 있는 이름은 다른 모듈에 영향을 끼치거나, 다른 모듈로부터 영향을 받지 않아야 한다. 어떤 모듈을 외부에서 관찰한 동작은 그 내부에서 사용하는 지역적(local)인 식별자의 이름을 어떻게 선택하느냐와는 무관해야만 한다.

모률화된 프로그램을 만들기 위해서는, 계산 과정이 상태를 가지는 것으로 생각할 필요가 있는 경우가 많다. 그런 경우 상태를 서로 독립적인 부분으로 나눌 수 있다면, 상태의 각 부분을 처리하는 여러 부분으로 프로그램 나누는 것이 중요한 분할 방법이다.

이 논문에서는 모듈화를 달성하기 위한 여러가지 기술을 다룰 것이다. 각 기술이 상호보완적일 것이라 생각할 지 모르겠다. 하지만, 실제로는 각각이 서로 충돌한다는 사실을 발견했다. 어떤 기술을 극단까지 몰고가면 다른 부분에서는 많이 양보해야만 한다.

### 리스프 계열 언어

원 논문에서는 리스프(LISP) 계열 언어를 다뤘다. 리스프 계열 언어의 진화 과정을 간단한 장난감 언어(toy language)를 사용해 쫓아가면서 살펴본다. 

다만, 내가 번안한 이 글에서는 구체적 문법보다는 문법을 분석해 나온 구문 구조(보통 컴파일러 이론에서는 AST라고 부르는 단계)를 다룰 것이다. 실제 구문을 파싱해 AST로 만드는 과정은 이 논문에서 중요한 부분은 아니다. 리스프의 경우에는 데이터와 프로그램이 구조가 똑같고 인터프리터도 리스프로 되어 있고 했기 때문에 이런 부분 처리가 상대적으로 간단하지만, 실제 일반적인 언어는 단순하지는 않다. 파싱해서 AST를 만드는 부분은 별도의 박스에 간단하게 코드 위주로 포함시킬 것이다.

원문의 순서와 리프스 코드는 그대로 사용하되, 리스프 코드에 대한 자세한 설명은 생략하고, 동일한 역할을 하는 스칼라 코드를 설명할 것이다. 이론적인 설명이나 추상적인 설명은 가능한 모두 다 포함시킬 것이다.

### 논문의 구조

여러 인터프리터를 매번 조금씩 바꿔가면서 만든다.

0장에서는 인터프리터 기본 구조와 메타 순환 인터프리터(meta circular interpreter)를 간단한 제한적인 리스프 언어를 가지고 다룬다.


## 0. LIST, AST 인터프리터

### 재귀 수식

사람들의 믿음과는 달리 리스프가 처치의 람다 대수(lambda calculus)에서 나온 것은 아니다. 최초의 리스프에는 잘 정의한 자유변수(free variable) 개념이나 프로시저에 대한 개념이 없었다. 최초의 리스프 프로그램은 재귀적인 식과 비슷하게 함수를 기호 식(Symbolic Expression, "S-Expression")에 정의하는 형태였다. 재귀 함수 이론(Kleene)과 다른점은 재귀함수 이론에는 "패턴에 따른 호출(pattern directed invocation)"을 했는데, 리스프는 "맥카시 조건식(McCarthy conditional)"이라는 조건문을 도입했다는 것에 있었다.

재귀합수 이론에서

		factorial(0) = 1
        factorial(successor(x)) = successor(x) * factorial(x)

라고 정의하던 것을, 초기 리스프에서는 다음과 같이 쓸 수 있었다.

		factorial[x] = [x=0 -> 1; T -> x*factoiral[x-1]]

여기서, `[a -> b; T -> c]`는 기본적으로 "if a then b else c"를 의미한다. 

재귀함수 이론에서는 자연수에 대한 함수를 다루는 것이 목적이지만, 리스프에서는 수식(algebraic expression)을 다루는 것이 목적이다. 물론 이런 수식을 숫자로 인코딩 할 수 있지만(괴델(Kurt Godel)의 "산술화(arithmetization)"), 그건 쓸모가 없다(역주: 알아볼 수 있어야 쓸모가 있지!). 따라서 이런 인코딩 밥법으로 새로이 S-식을 정의했다. S 식은 페아노(Peano)가 자연수를 정의하느라 사용한 공리계에 대응시킬 수 있는 형식적인(역주: 형식적이란 말은 수학적으로 엄밀한 절차를 거쳐 기술하는 것을 말한다) 귀납적 공리(formal inductive axiom)로 정의할 수 있다. 다만, S-식에 대해 여기서는 불안전하고 비형식적인 정의만을 제공할 것이다.

이 논문의 목적을 달성하기 위해 S-식에서 필요한 것은 아톰(atom)과 리스트(list) 이다. 아톰은 "더 이상 나눌 수 없는" 데이터 객체로 글자나 숫자(0-9)를 사용한다. 아톰 중 숫자만 사용한 아톰은 수로 간주한다. "+"나 "-"등도 글자로 취급한다. 리스트는 여는 괄호 사이에(`()`) S-식을 차례로 나열한 S-식이다. 각 S식 사이는 공백으로 나누며, 비어있는 리스트도 있다. 예를 들어, `(FOO 43 BAR)`는 `FOO`, `43`, `BAR`로 이뤄진 리스트이다. 여기서 리스트의 정의가 재귀저임에 유의하라. 예를 들어, `(DEFINE (SECOND X) (CAR (CDR X)))`는 세 요소, 즉 `DEFINE`이라는 아톰, `(SECOND X)`라는 리스트, `(CAR (CDR X))`라는 복잡한 리스트로 이루어진 리스트이다.

S-식을 사용해 산술식을 표기할 때는 "캠브리지 폴란드(Cambridge Polish)" 표기를 사용한다. 이는 "폴란드 전위 표기법(Polish prefix notation)"에 괄호를 추가한 것이다. 수는 수로, 다른 아톰은 아톰 그대로 표기한다. 리스트의 첫 원소는 연산자(operator), 나머지는 피연산자(operand)이다. 예를 들어 `a*b+c*d`는 `(+ (* a b) (* c d))` 라고 표현할 수 있다. 이렇게 표현하면 연산자의 계산 순서가 명확하기 때문에 우선순위가 필요 없다.

조건식 `[a1->b1; a2->b2; ...; an->bn]` (a1이 참이면 b1, a1이 참이 아니면 다시 a2부터 같은 로직 반복)은 `(COND (a1 b1) (a2 b2) ... (an bn))`이라고 표기한다. 

이제 리스프 재귀식을 S-식으로 표현 가능하다. 예를 들어 앞의 `factorial[x] = [x=0 -> 1; T -> x*factoiral[x-1]]`은 다음과 같이 쓸 수 있다.

		(DEFINE (FACORIAL X)
			(COND ((= X 0) 1)
                   (T (* X (factorial (- X 1))))))

이 명령은 `X`를 인자로 받는 `FACTORIAL`이라는 함수를 만든다. 

이제 프로그램을 기술할 방법은 정의를 했지만, S-식을 분해해서 어떤 종류인지 알아내고, 다시 이를 묶어낼 방법이 필요한다.

- ATOM : `(ATOM X)`라고 호출하면 `X`가 아톰인 경우 참을 그렇지 않은 경우 거짓을 돌려준다. 참은 `T`, `NIL`(빈 리스트)이 거짓 역할을 한다.
- NUMBERP : `(NUMBER X)`라고 호출하면 `X`가 수인 경우 참, 그렇지 않은 경우 거짓을 돌려준다.
- CAR: (CAR list)라고 하면 list의 맨 앞의 원소를 돌려준다. 즉, `(CAR (DEFINE (SECOND X) (CAR (CDR X))))`는 `CAR`라는 아톰을 돌려준다. 역사적으로 리스트를 구현할 때 IBM 704에서 레지스터의 address 부분에 리스트의 최초 원소가 들어가서 CAR(Content of Address part of Register)라고 불렀던 데서 유래했다.
- CDR: (CDR list)라고 하면 리스트의 나머지 부분을 돌려준다. 즉, `(CDR (DEFINE (SECOND X) (CAR (CDR X))))`는 `((SECOND X) (CAR (CDR X)))`를 돌려준다. "Content of Decrement part of Register"에서 유래했다.
- CADR, CADDR: `(CADR X)`은 `(CAR (CDR X))`이고, `(CADDR X)`는 `(CAR (CDR (CDR X)))`이다. `CAR`와 `CDR`이 `first`와 `rest` 등보다 더 편리하게 쓰이는 이유 중 하나가 바로 이런 복잡한 함수 합성을 쉽게 표현할 수 있어서 이다. 

   * 연습문제: `(CDDADR X)`를 `CAR`과 `CDR`로 나타내면?

인터프리터를 만들기 위한 몇가지 프로그램을 짜 두자.

먼저, EQUAL은 두 S-식이 같은지 살펴본다.

        (DEFINE (EQUAL X Y)
            (COND ((NUMBERP X)
                    (COND ((NUMBERP Y) (= X Y)))
                        (T NIL)))
                  ((ATOM X) (EQ X Y))
                  ((ATOM Y) NIL)
                  ((EQUAL (CAR X) (CAR Y))
                   (EQUAL (CDR X) (CDR Y)))))

참고로 위 식을 스칼라 함수로 표현해 본다면 다음과 같다(isNumber, isAtom, first, rest는 구현이 있다고 생각하자).

        def equal(x:sExpr, y:sExpr):Boolean = 
            if(x.isNumber) 
                if(y.isNumber) x == y else false
            else if(x.isAtom) x == y
            else if(y.isAtom) false
            else if(equal(x.first, y.first)) equal(x.rest, y.rest)
            else false


`EQ`는 두 아톰이 같은지를 비교한다. 리스프 프로그램을 S-식으로 표현하면 상수 표현시 문제가 생긴다. 예를 들어 X가 FOO라는 아톰인지 확인하려면, 다음과 같이 쓰면 어떨까 생각할 지도 모르겠다.

		(EQ X FOO)

하지만, 이렇게 쓰면 `X`를 `FOO`라는 변수와 비교하는 것이 되어버린다. 이를 막기 위해서는 임의의 S-식을 프로그램 안에서 상수로 표현할 수 있어야 한다. 이런 목적으로 사용하는게 `QUOTE`이다. `(QUOTE FOO)`라고 쓰면 FOO라는 아톰을 만든다. 따라서 앞의 비교는 다음과 같이 쓰면 제대로 동작한다.

		 (EQ X (QUOTE FOO))

이와 비슷하게, `X`가 `(LIST X Y Z)`와 같은지 비교하려면, 

		(EQUAL X (LIST X Y X))

가 아니라, 다음과 같이 해야 한다.

		(EQUAL X (QUOTE (LIST X Y X)))

`QUOTE`는 역따옴표로 줄여쓸 수 있다. 즉, `(QUOTE (LIST X Y))`는 ``(LIST X Y)`라 쓸 수 있다.

#### AST(Abstract Syntax Tree)

앞의 S식은 리스프 프로그램이다. 리스프에서 리스프 프로그램 인터프리터를 만들때는 특별한 파싱이 필요 없이 리스프를 처리할 수 있다. 하지만, 이 번안판에서는 스칼라를 사용한다. 그래서 여기서는 스칼라 프로그램에서 s-식을 표현할 데이터 구조와 리스프 프로그램을 그 데이터구조로 옮기는 프로그램을 먼저 작성한다.

먼저 s-식은 다음과 같이 정의할 수 있다.

            abstract class SExpr
            case class LIST(elements:List[SExpr]) extends SExpr
            case class ATOM(value: String) extends SExpr

간단하게 리스트와 아톰으로만 구분했다. 예를 들어 `(def (foo X Y) (COND ((= X Y) (- X Y)) (T 10)))`는 다음과 같은 구조가 된다.

		 LIST(List(
		         ATOM(def), LIST(List(
ATOM(foo), ATOM(X), ATOM(Y))), LIST(List(ATOM(COND), LIST(List(LIST(List(ATOM(=)
, ATOM(X), ATOM(Y))), LIST(List(ATOM(-), ATOM(X), ATOM(Y))))), LIST(List(ATOM(T)
, ATOM(10)))))))

문자열을 입력받아 위의 `SExpr`로 파싱하는 객체는 다음과 같다.

          import scala.util.parsing.combinator._
          object Parser extends RegexParsers {  // Regex(정규식)을 사용해 토큰을 매치할 수 있는 파서를 확장
            // 아톰은 \s(공백), \(, \) (오른쪽과 왼쪽 괄호)가 아닌 문자들이 1개 이상 반복된 것임.
            def Atom: Parser[String] = 
              """[^\s\(\)]+""".r

            // 여는 괄호
            def LParen: Parser[String] = """\(""".r 

            // 닫는 괄호
            def RParen: Parser[String] = """\)""".r

            //
            //  SExpr = ( SExprList ) | () | Atom
            //
            def SExpr: Parser[SExpr] = 
              LParen ~> SExprList <~ RParen  ^^ { case lst => lst } | 
              LParen ~ RParen ^^ {  case l~r => LIST() } |
              Atom ^^ { case atom => ATOM(atom) }

            //
            //  SExprList = Atom | Atom SExprList | SExpr | SExprList
            //  SExprList = SExpr | SExpr SExprList
            //
            def SExprList: Parser[List[SExpr]] = 
              SExpr ^^ { case sexpr => List(sexpr) } |
              SExpr ~ SExprList ^^ { case sexpr ~ el  => sexpr :: el } |
              //Atom ~ SExprList ^^ { case atom ~ el  => ATOM(atom) :: el } |
              //SExpr ~ SExprList ^^ { case sexpr ~ el  => sexpr :: el } |
              //Atom ^^ { case atom => List(ATOM(atom)) } |
              //SExpr ^^ { case sexpr => List(sexpr) }

            def parse(text : String) = parseAll(SExpr, text) 
          }

이 식을 사용하면, 예를 들어 앞의 factorial을 정의한 sExpression은 다음과 같이 표현할 수 있다.

    abstract class SExpr
    case class ID(value: String) extends SExpr  // ID는 문자열(아톰)
    case class Number(value: Integer) extends SExpr // Number는 정수값
    case class Cond(cases: List[(SExpr,SExpr)]) extends SExpr  // COND의 (조건식, 실행식)쌍을 리스트로 cases에 넣음
    case class Define(fname:ID, farg: List[ID], fbody: SExpr) extends SExpr // 함수 정의는 이름, 인자목록, 함수 본문
    case class Apply(operator: SExpr, operand: List[SExpr]) extends SExpr // 연산자와 피연산자를 구분

이 식을 파싱하는 것은 다음과 같다. 자세한 부분은 설명하지 않을 것이다.


