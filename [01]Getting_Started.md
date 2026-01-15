# Getting Started
Rule Engine은 개발자나 사용자들이 정한 규칙을 기준으로 다양한 Python 객체를 필터링할 수 있게 해주는 도구입니다.<br/>
규칙은 Python과 비슷한 문법의 전용 문자열 언어로 작성되며 `exec` 나 `eval`을 사용하지 않아 외부 입력에도 안전합니다.<br/>

<details>
    <summary>
        <code>exec</code>와 <code>eval</code> 명령어의 위험성
    </summary>

### 1️⃣ `eval`과 `exec`
```python
eval("1 + 2") #3
```
`eval` 명령어는 문자열을 그대로 Python 코드로 실행한다.

```python
exec("print('hello')")
```
`exec` 명령어는 여러 줄의 Python 코드를 실행한다.

### 2️⃣ 악의적인 입력값이 들어올 때의 위험성
예를 들어 아래와 같은 코드가 `eval` 이나 `exec` 입력값으로 들어오게 되었을 때,
```python
__import__("os").system("rm -rf /")
```
서버를 조종할 수 있는 권한을 주는 꼴이 되게 된다. <br/>
이러한 것을 Code Injection(코드 주입 공격)이라 한다.
</details>

## Basic Usage
1. Rule Engine을 사용하려면 먼저 필터링 대상이 될 데이터 구조를 정해야 한다. <br/>
데이터는 보통 여러 개의 속성을 가진 객체 형태이다. <br/>
이 문서에서는 예제로 **만화책(comic book)** 데이터를 사용한다.<br/><br/>
    - 만화책 객체는 다음과 같은 속성을 가진다.
        | Attribute | Python Type    | Rule Engine Type |
        | --------- | -------------- | ---------------- | 
        | title     | `str`          | `STRING`         |
        | publisher | `str`          | `STRING`         |
        | issue     | `int`          | `FLOAT`          |
        | released  | `datetime.date`| `DATETIME`       |
        <br/>
    - 예를 들면 만화책 데이터는 아래와 같은 형태일 수 있다.
        ```python
        comics = [
            {
                'title': 'Batman',
                'publisher': 'DC',
                'issue': 89,
                'released': datetime.date(2020, 4, 28)
            },
            {
                'title': 'Flash',
                'publisher': 'DC',
                'issue': 753,
                'reeased': datetime.date(2020, 5, 5)
            },
            {
                'title': 'Captain Marvel',
                'publisher': 'Marvel',
                'issue': 18,
                'released': datetime.date(2020, 5, 6)
            }
        ]
        ```
2. 필터링 대상 데이터 구조를 정했다면, 이제 해당 데이터를 기준으로 조건을 정의하는 Rule 객체를 생성해야 한다. <br/><br/>
Rule Engine에서는 객체가 가진 각 속성(attribute)이 자동으로 룰에서 사용할 수 있는 심볼(symbol)이 된다.<br/>
즉, 별도의 설정 없이도 객체의 속성 이름을 그대로 조건식에 사용할 수 있다.<br/><br/>
Rule 객체는 `Rule` 클래스를 생성하면서 만들며, 이때 Rule Engine 문법으로 작성한 조건식을 문자열로 전달한다.<br/>
    ```python
    rule = rule_engine.Rule("조건식")
    ```

    - 앞서 정의한 만화책 데이터 기준으로, Rule에서 사용할 수 있는 심볼(symbol)은 `title`, `publisher`, `issue`, `released` 가 있다. 이 심볼 이름들은 모두 Rule Engine에서 허용하는 형식이다. <br/><br/>
    Rule Engine의 심볼 규칙은 Python 변수명 규칙과 동일하게, <br/> ① 문자로 시작해야 하며 <br/> ② 공백이나 특수문자를 포함할 수 없다. <br/>
    예를 들어 `released`은 유효한 심볼이지만, `Released Data`는 공백이 포함되어 있어 사용할 수 없다.
    <br/><br/>
    
    - 간단한 예제로 출판사가 `"DC"`인 만화책만 필터링하고 싶다면, 다음과 같이 Rule을 정의할 수 있다.
        ```python
        rule = rule_engine.Rule(
            #DC에서 출판된 책만 매칭
            'publisher == "DC"'
        )
        ```
    <br/>
    
    - Rule Engine은 단순 비교뿐만 아니라 날짜 리터럴과 논리 조건을 조합한 표현식도 지원한다. <br/>
    예를 들어 2020년 5월에 발행된 DC 만화책만 필터링하려면 다음과 같이 작성할 수 있다. <br/>
        ```python
        rule = rule_engine.Rule(
            # 2020년 5월에 발행된 DC 만화
            'released >= d"2020-05-01" and released < d"2020-06-01" and publisher == "DC"'
        )
        ```
        여기서 날짜는 문자열 형태로 작성하며, 앞에 `d`를 붙여 날짜 리터럴임을 명시한다.<br/>
        형식: `d"YYYY-MM-DD HH:MM:SS"`<br/>
        시간을 생략한 경우에는 자동으로 `00:00:00`(자정)으로 처리 된다.
    <br/><br/>
    
    - 일부 데이터 타입은 추가 속성을 제공하며, 이러한 속성은 점(`.`) 연산자를 통해 접근할 수 있다. <br/>
    예를 들어 출판사 이름의 대소문자 차이를 무시하고 비교하려면 다음과 같이 작성할 수 있다.<br/>
        ```python
        rule = rule_engine.Rule(
            # 'Dc', 'dc' 등의 표기를 모두 'DC'로 통일
            'publisher.as_upper == "DC"'
        )
        ``` 
    <br/>

    - Rule Engine은 정규표현식 기반 문자열 비교도 지원한다. <br/>
    이 경우, 왼쪽에는 비교 대상 문자열, 오른쪽에는 정규 표현식, 비교 연산자는 `=~`를 사용한다. <br/>
    예를 들어 제목이 `"Captain "`으로 시작하는 만화책을 찾고 싶다면 다음과 같이 작성할 수 있다. <br/>
        ```python
        rule = rule_engine.Rule(
            # 제목이 'Captain '으로 시작하는 책
            'title =~ "Captain\\s\\S+"'
        )
        ```