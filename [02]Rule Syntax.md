# Rule Syntax
Rule을 작성하는 문법은 True(조건을 만족함) 또는 False(조건을 만족하지 않음)로 평가되는 논리 표현식을 기반으로 한다. <br/>
Rule Engine은 제한된 수의 데이터 타입을 지원하며, 이 타입들은 리터럴 값으로 직접 작성하거나 Rule이 적용되는 Python 객체로부터 값을 해석해 가져오는 방식으로 사용된다.<br/>
지원되는 데이터 타입의 전체 목록은 [Data Types 표](./[03]Data%20Types.md)를 참고하면 된다.
<br/>

모든 연산이 모든 데이터 타입에서 동작하는 것은 아니다.<br/>
어떤 연산이 어떤 타입에서 사용 가능한지는 아래에 나오는 표에 정리되어 있다. <br/>
또한 Rule Engine은 일반적인 연산자 우선순위(order of operations)를 따른다.
<details>
<summary>
    order of operations
</summary>

|순서| 연산자 | 설명|
|--- | --- | --- |
| 1 | `()` `[]` `->` `.` `::`|함수 호출, 범위 지정, 배열/멤버 접근|
| 2 | `!` `~` `-` `+` `*` `&` `sizeof` <br/> type cast `++` `--`|단항 연산자, 논리부정, 부호, 증가/감소 등
| 3 | `*` `/` `%` `MOD` | 곱셈, 나눗셈, 나머지 |
| 4 | `+` `-` | 덧셈, 뺄셈 |
| 5 | `<<` `>>` | 비트 시프트 연산자 |
| 6 | `<` `<=` `>` `>=` | 비교 연산(크기 비교)|
| 7 | `==` `!=` | 동등 / 비동등 비교 | 
| 8 | `&` | 비트 AND | 
| 9 | `^` | 비트 XOR(배타적 OR) |
| 10 | `\|` | 비트 OR |
| 11 | `&&` | 논리 AND | 
| 12 | `\|\|` | 논리 OR | 
| 13 | `?:` | 조건 연산자(삼항 연산자) | 
| 14 | `=` `+=` `-=` `*=` `/=` `%=` `&=` `!=` `^=` `<<=` `>>=` | 대입 연산자 |
| 15 | `,` | 콤마 연산자 


[(참고) Programming Languages](https://en.wikipedia.org/wiki/Order_of_operations#Programming_languages)


</details>

## Grammar
Rule Engine의 표현식 문법은 숫자 데이터에 대한 기본 산술 연산과 문자열에 대한 정규표현식 연산 등 여러 가지 연산을 지원한다.<br/>

각 연산은 타입을 인식하여 동작하며, 호환되지 않는 타입이 사용될 경우 예외(exception)가 발생한다.

### supported Operations
다음 표는 Rule Engine 표현식에서 사용할 수 있는 모든 연산자와 각 연산자가 지원하는 데이터 타입을 정리한 것이다.<br/><br/>
**1️⃣ Arithmetic Operators(산술 연산자)**
| 연산자 | 설명 | 지원 데이터 타입 |

#### Accessor Operators
#### Array Comprehension
#### Ternary Operators

### Function Calls

### Reserved Keywords

### Literal Values
#### Literal DATETIME Values
#### Literal FLOAT Values
#### Literal TIMEDELTA Values

### Comments