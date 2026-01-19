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
3. Rule 객체를 정의했다면, 이제 해당 Rule을 실제 데이터에 적용할 수 있다. <br/> 
Rule Engine에서는 Rule을 대상 객체에 적용하기 위해 두 가지 기본 메서드를 제공한다.
    - `matches()` : 단일 객체 검사 <br/>
    `matches()` 메서드는 하나의 객체가 Rule 조건을 만족하는지를 판단한다. <br/>
    조건을 만족하면 `True`, 만족하지 않으면 `Fasle`를 반환한다.
        ```python
        rule.matches(comics[0]) #True
        ```
        이 메서드는 특정 객체 하나에 대해 조건 충족 여부만 빠르게 확인하고 싶을 때 유용하다.<br/><br/>
    
    - `filter()` : 여러 객체 필터링<br/>
    `filter()` 메서드는 여러 개의 객체를 담은 iterale을 받아 Rule 조건을 만족하는 객체만 필터링하여 반환한다.
        ```python
            # comics 리스트에서 Rule조건을 만족하는 객체만 필터링
            rule.filter(comics)
            # => <generator object Rule.filter at 0x7f2bdafbe650>
        ```

### Attribute-Backend Objects (객체 속성 기반 데이터 사용)
앞선 예제에서는 필터링 대상 객체로 Python 딕셔너리(dict)를 사용했다. <br/>
이 경우 Rule Engine은 기본적으로 딕셔너리의 key를 심볼(symbol)로 사용해 조건을 평가한다.<br/>
하지만 데이터 구조가 딕셔너리가 아니라, 클래스 기반 객체처럼 속성(attribute)을 가지는 형태라면 <br/>
이 방식은 그대로 사용할 수 없다.<br/><br/>
- 예를 들어 딕셔너리가 아닌, 다음과 같은 클래스 객체 형태의 데이터를 사용하는 경우가 이에 해당한다.
    ```python
    # 클래스 기반 만화책 데이터 예제
    class Comic:
        def __init__(self, title, publisher, issue, released):
            self.title = title
            self.publisher = publisher
            self.issue = issue
            self.released = released
    
    comics = [
        Comic('Batman', 'DC', 89, datetime.date(2020, 4, 28)),
        Comic('Flash', 'DC', 753, datetime.date(2020, 4, 28)),
        Comic('Captain Marvel', 'Marvel', 18, datetime.date(2020, 5, 6))
    ]
    ```
    이 경우 만화책의 값들은 `obj["publisher"]`가 아니라 `obj.publisher` 처럼 객체의 속성(attribute)로 접근해야 한다. <br/><br/>

Rule Engine에서는 이런 경우를 위해 Context 객체를 통해 Rule의 동작 방식을 설정할 수 있다. <br/>
Context 객체는 Rule이 어떻게 동작할지를 정의하는 설정 객체이며, 그중 하나가 바로 resolver다.<br/><br/>
resolver는 Rule이 심볼(symbol)을 어떤 방식으로 실제 값에 매핑할지를 결정한다.<br/>
Rule Engine에는 기본적으로 다음 두 가지 resolver가 있다.
- `resolve_itme()`: (Defualt) 딕셔너리의 key를 기준으로 값을 찾는다.<br/>
ex) `obj["publisher"]`
- `resolve_attribute()`: 객체의 속성(attribute)을 기준으로 값을 찾는다.<br/>
ex) `obj.publisher`

<br/>
클래스 객체처럼 속성 기반 데이터 구조를 사용할 경우, <br/>
resolver를 `resolve_attribute()`로 지정한 Context를 생성해야 한다.

```python
# resolver를 attribute 기반으로 설정한 Context 생성
context = rule_engine.Context(resolver=rule_engine.resolve_attribute)

# 해당 Context를 사용해 Rule 생성
rule = rule_engine.Rule('publisher == "DC"', context=context)
```
이렇게 하면 Rule Engine은 `publisher`라는 심볼을 해석할 때 딕셔너리 key가 아닌 객체의 속성에서 값을 가져오게 된다. <br/>
Context를 커스터마이징했더라도, Rule을 사용하는 방식은 기존과 동일하다.<br/><br/>
하나의 Context 객체는 같은 구조의 객체에 대해 여러 Rule에서 재사용할 수 있다. <br/>
다만, 서로 다른 속성 구조를 가진 객체, 예를 들어 만화책(`Comic`)과 작가(`Artist`)처럼 전혀 다른 데이터 모델 같은 경우에는 **같은 Context 객체를 공유하면 안된다.** 객체 구조에 맞는 Context를 각각 따로 만들어야 한다.

## Advanced Usage
Rule Engine에는 더 유연하게 쓰기 위한 고급 기능들이 몇 가지 있다.<br/>
다만 일반적인 사용 사례에서는 굳이 쓸 일이 없는 경우가 많다.<br/>

### Setting a Default Value
Rule Engine은 기본적으로 룰에서 사용한 심볼(symbol)이 대상 데이터이 없으면, 즉 값을 찾을 수 없는 심볼이면 에러를 발생시킨다.<br/>
정확히는 `engine.Rule`(=`rule_engine.Rule`)이 `SymbolResolutionError`를 발생시키면서 "이 심볼을 해석할 수 없다"라고 알려준다.<br/>
하지만 상황에 따라서는, "심볼이 없으면 에러 내지 말고, 없는 값은 기본값(예: `None` / `Null`)로 처리해라" 처럼 동작하길 원하는 경우가 있다.<br/>
이 동작을 바꾸려면, `Context`를 만들 때 `default_value`를 지정하면 된다.
- **✅ 기본 동작: 없는 심볼이면 에러**
    ```python
    # title 키가 없고 default_value도 없으므로 실패
    rule_engine.Rule('title').matches({})
    # => SymbolResolutionError: title
    ```

- **✅ default_value를 주면: 없는 심볼은 기본값으로 처리**
    ```python
    context = rule_engine.Context(default_value=None)
    ```
    이렇게 설정하면, 데이터에 `title`이 없을 때 Rule Engine은 "없는 값"을 에러로 처리하지 않고 `None`으로 간주한다. 
    ```python
    # title이 없을 때, default_value(None)으로 평가됨 → False
    rule_engine.Rule('title', context=context).matches({})
    # => False
    
    # title이 비어있지 않은 문자열 일 때(있을 때), truthy → True
    rule_engine.Rule('title', context=context).matches({'title': 'Batman'})
    # => True
    ```

### Custom Resolvers
Rule Engine에는 기본적으로 두 가지 resolver가 포함되어 있다.<br/>
①딕셔너리처럼 key로 값에 접근하는 resolver<br/> ②클래스 객체처럼 속성(attribute)으로 값에 접근하는 resolver<br/>
대부분의 경우 이 두 가지 resolver면 충분하지만, 대상 객체의 구조가 이 둘과 맞지 않는 경우, 직접 커스텀 resolver를 만들어 사용할 수 있다. 
<br/><br/>
예를 들어, 객체 내부 구조가 중첩되어 있거나, 값이 가공된 형태로 저장되어 있거나, 단순히 `obj[key]`나 `obj.attr` 방식으로 접근할 수 없는 경우에는 기본 resolver 대신 사용자 정의 resolver 함수를 구현해 Rule Engine에 전달하면 된다. <br/><br/>
커스텀 resolver는 `resolver(thing, name)`과 같은 형태를 가진다.<br/>
`thing`은 Rule이 적용되고 있는 대상 객체이고, `name`은 Rule 표현식에서 사용한 심볼 이름(문자열)이다.<br/>
resolver의 역할은, 주어진 객체(`thing`)에서 심볼(`name`)에 해당하는 값을 찾아 반환하는 것이다.
<br/><br/>
resolver가 어떤 이유로든 값을 찾지 못했다면, 반드시 `SymbolResolutionError`를 발생시켜야 한다. 이때 에러를 발생시킬 때는, `thing`을 keyword argument로 함께 전달해야 한다. <br/>
이 규칙을 지키면 Rule Engine 내부에서 에러를 일관된 방식으로 처리할 수 있다.

#### Suggestions
커스텀 resolver는 선택적으로 "이 심볼 이름은 잘못되었고, 대신 이게 맞나요?" 같은 추천(suggestion)도 제공할 수 있다.<br/>
이를 위해 Rule Engine은 `suggest_symbol`이라는 헬퍼 함수를 제공한다.<br/>
잘못된 심볼 이름과 실제로 존재하는 올바른 심볼 이름 목록을 `suggest_symbol`에 전달하면, Rule Engine이 가장 그럴듯한 심볼이름을 추천해준다.<br/>
이 추천 결과를 `SymbolResolutionError`의 `suggestion`인자로 넘기면 된다.<br/> 
이 기능은 룰을 사람이 직접 작성할 경우 실수를 줄이는 것에 도움을 줄 수 있다.

### Type Hinting
심볼(symbol)의 타입 정보는 `Context` 객체를 통해 Rule에 전달할 수 있으며, 이 정보는 룰에 사용된 연산이 서로 호환되는지 검사하는데 사용된다.<br/>
타입 정보가 설정되어 있으면, 예를 들어 숫자 값에 대해 정규표현식 비교(`=~`)를 시도하는 것처럼 호환되지 않는 연산이 발견될 경우 `EvaluationError`가 발생한다.<br/>
이를 통해 Rule을 실제 객체에 적용하기 전에, 룰 문법 자체에 포함된 오류를 미리 확인할 수 있다.<br/>
또한 심볼 타입 정보가 지정된 경우, 심볼을 통해 해석된 값은 지정된 타입과 일치하거나 `NULL`이어야 하며, 그렇지 않으면 심볼을 해석하는 시점에 `SymbolTypeError`가 발생한다.
<br/><br/>
심볼 타입 정보를 정의하려면, `Context` 객체를 생성할 때 `type_resolver` 함수를 전달해야 한다.<br/>
`type_resolver`함수는 심볼 이름(문자열)을 인자로 받아 해당 심볼의 타입을 `DataType` 열거형 값으로 반환해야 한다.
```python
# 만화책 객체의 속성 타입을 정의하는 type_resolver
def type_resolver(name):
    if name == 'title':
        return rule_engine.DataType.STRING
    elif name == 'publisher':
        return rule_engine.DataType.STRING
    elif name == 'issue':
        return rule_engine.DataType.FLOAT
    elif name == 'released':
        return rule_engine.DataType.DATETIME
    # 정의되지 않은 심볼일 경우 에러 발생
    raise rule_engine.errors.SymbolResolutionError(name)

context = rule_engine.Context(type_resolver=type_resolver)
```
`UNDEFINED`는 심볼이 유효하다는 것만 명시하고, 구체적인 타입 검증은 수행하지 않도록 설정할 때 사용할 수 있다. <br/>
이 경우 Rule 객체는 해당 심볼을 사용할 수는 있지만, 그 심볼이 사용된 연산에 대해서는 타입 검사를 수행하지 않는다.<br/><br/>
`type_resolver`가 정의된 상태에서는, Rule에서 참조한 심볼이 `type_resolver`에 의해 정의되어 있지 않으면 즉시 `SymbolResolutionError`가 발생한다.
```python
# 유효한 경우: issus는 type_resolver에 정의된 심볼
rule = rule_engine.Rule('issue == 1', context=context)

# 유효하지 않은 경우: author는 정의되지 않은 심볼
rule = rule_engine.Rule('autor == "Stan Lee"', context=context)
# => SymbolResolutionError: author

# Context를 지정하지 않은 경우: 타입 정보가 없어 허용됨
rule = rule_engine.Rule('author == "Stan Lee"')
# => <Rule text='author == "Stan Lee"'>
```

#### Compound Data Types(복합 데이터 타입)
`ARRAY`나 `MAPPING`과 같은 복합 데이터 타입은 필요한 경우 내부 요소(member)의 타입 정보를 함께 지정할 수 있다. 이때는 각 데이터 타입에 대응하는 타입 생성 함수를 호출해 정의한다.<br/>
예를 들어, 문자열로 이루어진 배열은 다음과 같이 정의할 수 있다.
```python
DataType.ARRAY(DataType.STRING)
```
키는 문자열이고 값은 숫자(float)인 매핑 타입은 다음과 같이 정의할 수 있다.
```python
DataType.MAPPING(DataType.STRING, DataType.FLOAT)
```
`ARRAY`, `MAPPING` 타입에 대한 자세한 내용은 각각의 함수 문서를 참고하면 된다.
<br/><br/>

복합 데이터 타입에서 지정할 수 있는 멤버 타입은 하나의 데이터 타입만 허용된다.<br/>
일부 경우에는 멤버 타입을 **nullable**로 설정할 수도 있는데, 이 경우 해당 값은 지정된 타입이거나 `NULL`일 수도 있다.<br/>
예를들어, 값이 모두 nullable 문자열인 `MAPPING` 타입은 정의할 수 있다.<br/>
하지만, 하나의 `MAPPING`안에서 어떤 값은 `STRING`, 어떤 값은 `BOOLEAN`처럼 서로 다른 타입을 동시에 지정하는 것은 허용되지 않는다.<br/>
이처럼 값의 타입을 하나로 정의할 수 없는 경우에는, 키 타입만 명시하고 값 타입은 기본값인 `UNDEFINED`로 설정할 수 있다.

#### Function Data Types(함수 데이터 타입)
복합 데이터 타입과 마찬가지로, 함수(Function)도 타입 정보를 함께 정의할 수 있다. <br/>
이때는 `FUNCTION` 타입을 사용해 함수의 타입 정보를 지정한다.<br/><br/>
Rule Engine에서 함수는 위치 기반 인자(positional arguments)만 지원하며, 키워드 인자(keyword arguments)는 지원하지 않는다. <br/>
다만 `minimum_arguments` 옵션을 사용하면, 필수 인자 개수와 선택 인자 개수를 지정할 수 있다.<br/><br/>
예를 들어 내장 함수 `split`은 최소 1개, 최대 3개의 인자를 받을 수 있다. <br/>
첫 번째 인자는 항상 필수, 두 번째, 세번째 인자는 선택(단, 세 번째 인자를 사용하려면 두 번째 인자도 반드시 함께 사용해야 한다.)<br/>
`split` 함수의 인자 구성은 다음과 같다.
1. 분할할 문자열
2. 기준이 되는 구분자 문자열
3. 최대 분할 횟수
```python
rule_engine.DataType.FUNCTION(
    # 에러 메시지에 사용될 함수 이름
    'split',
    # 반환 타입: 문자열 배열
    return_types=ast.DataYpe.ARRAY(ast.DataType.STRING),
    # 각 인자의 타입 정의
    argument_types=(
        ast.DataType.STRING, # 첫 번재 인자: 분할할 문자열
        ast.DataType.STRING, # 두 번째 인자: 구분자
        ast.DataType.FLOAT   # 세 번째 인자: 최대 분할 횟수
    ),
    # 최소 인자 개수 (두 번째, 세번째 인자는 선택)
    minimum_arguments=1
)
```
함수의 반환 타입이나 인자 타입을 지정하지 않으면, 해당 함수에 대해서는 타입 검사가 수행되지 않는다.

#### Defining Types From A Dictionary(딕셔너리로 타입 정의하기)
편의를 위해, 심볼 이름과 해당하는 `DataType`을 매핑한 딕셔너리로부터 `type_resolver`함수를 생성해주는 `type_resolver_from_dict()`함수를 사용할 수 있다.<br/>
또한, v2.1.0 이후 버전부터는, `type_resolver` 인자에 딕셔너리를 직접 전달하는 경우, 내부적으로 `type_resolver_from_dict()`함수가 자동으로 사용된다.<br/><br/>
아래는 심볼이름과 타입을 딕셔너리로 정의해 `Context`에 전달하는 예시이다.
```python
context = rule_engine.Context(
    type_resolver=rule_engine.type_resolver_from_dict({
        # 심볼 이름과 데이터 타입 매핑
        'title':        rule_engine.DataType.STRING,
        'publisher':    rule_engine.DataType.STRING,
        'issue':        rule_engine.DataType.FLOAT,
        'released':     rule_engine.DataType.DATETIME
    })
)
```

### Changing Builtin Symbols
Rule Engine에는 기본적으로 몇 가지 내장 심볼(builtin symbols)이 제공된다.<br/>
이 기본 심볼들을 제거하거나, 다른 값으로 바꾸거나, 새로운 심볼을 추가할 수 있다. <br/><br/>
기본으로 제공되는 내장 심볼을 모두 제거하려면, 값이 비어 있는 딕셔너리를 사용해 `Builtins` 인스턴스를 초기화하면 된다.<br/>
이렇게 하면 모든 기본 내장 심볼이 제거되며, 필요하다면 이후에 원하는 값만 딕셔너리에 추가할 수 있다.<br/><br/>
기본 내장 심볼은 유지하면서, 추가 심볼을 등록하거나 일부 값을 덮어쓰고 싶은 경우에는 `from_defaults`생성자를 사용하면 된다.<br/>
이 방식에서는, 기본 내장 심볼은 그대로 유지되고, 전달한 딕셔너리에 포함된 값은 기존 심볼과 이름이 같으면 덮어쓰고, 이름이 겹치지 않으면 새 심볼로 추가된다.<br/><br/>
아래 예시는 `Context`를 상속받아 기본 내장 심볼에 `$version`심볼을 추가하는 방법을 보여준다.<br/>
```python
class CustomBuiltinsContext(rule_engine.Context):
    def __init__(self, *args, **kwargs):
        # 부모 클래스의 초기화 로직을 먼저 호출
        # default_timezone 설정을 위해 필요
        super(CustomBuiltinsContext, self).__init__(*args, **kwargs)

        self.builtins = rule_engine.builtins.Builtins.from_defaults(
            # $version 심볼을 외부로 노출
            {'version': rule_engine.__version__},
            # Context에 설정된 기본 타임존 사용
            timezone = self.default_timezone
        )
```
이렇게 정의하면 Rule Engine에서 `$version` 심볼을 사용해 Rule Engine의 버전 정보를 참조할 수 있다.

## Rule Inspection
Rule Engine에서는 Rule 객체의 상태를 확인하거나, 작성한 규칙을 분석하기 위한 몇 가지 방법을 제공한다.<br/>
- `is_valid()`: <br/>
    `is_valid()`는 Rule 표현식이 유효한지 검사하는 클래스 메서드이다.<br/>
    규칙에 문법 오류가 있는 경우 `False`를 반환한다.<br/>

    이 메서드는 Rule을 실제 데이터에 적용하기 전에, 표현식 자체에 문제가 없는지 미리 확인할 때 유용하다.
- `symbols`: <br/>
    Rule 객체에는 `context` 속성이 있으며, 그 안에는 `symbols` 속성이 포함되어 있다.<br/>

    `symbols`에는 Rule 표현식에서 사용된 심볼 이름 목록이 들어 있다.<br/>
    이를 통해 해당 Rule이 어떤 속성들을 참조하고 있는지 확인할 수 있다.
- `to_graphviz()`: <br/>
    `to_graphviz()`메서드는 Rule 표현식으로부터 생성된 추상 구문 트리(AST)를 Graphviz 형식의 방향 그래프(directed graph)로 만들어준다.<br/>

    이 기능은 조건이 복잡한 Rule을 디버깅할 때 도움이 된다.<br/>
    단, 이 기능을 사용하려면 Python의 `graphviz` 패키지가 설치되어 있어야 한다.