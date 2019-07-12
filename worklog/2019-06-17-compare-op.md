# 2019-06-17

## 비교 연산자

비교 연산자는 딱히 고민할 것 없이 C 언어의 비교 연산자 규칙을 그대로 구현했다. 플랑크는 문자열도 프리미티브 타입이므로 추가로 문자열도 비교 대상에 포함하게 구현했다. 문자열 비교는 strncmp() 함수의 결과를 그대로 사용했다. 다른 프로그래밍 언어들도 문자열간 비교는 strncmp() 함수의 알고리즘을 사용하므로 이 방법이 보편적이라고 볼 수 있다.

```
>> 3 > 4
false
>> 3 < 4
true
>> 5 != 5
false
>> 5 != 6
true
>> 23 >= 34
false
>> 23 <= 34
true
>> 5 == 5
true
>>
```

숫자 비교 연산은 자명한 수학 규칙을 그대로 따른다.

```
>> "ab" == "ab"
true
>> "ab" == "abc"
false
>> "ab" > "ac"
false
>> "ab" < "ac"
true
>> "aeg" == "ab"
false
>> "aeg" != "ab"
true
>> "aed" >= "ab"
true
>> "aed" <= "ab"
false
```

문자열 비교도 직관적으로 알 수 있는 결과를 내 보낸다.

```
>> "ab" > 5
Runtime Error [0x0001] Type mismatch error on this operator.
[Error] -> "ab" > 5
>> "ab" >= 5
Runtime Error [0x0001] Type mismatch error on this operator.
[Error] -> "ab" >= 5
>> "ab" <= 5
Runtime Error [0x0001] Type mismatch error on this operator.
[Error] -> "ab" <= 5
```

타입을 섞으면 비교 연산자는 동작하지 않는다. 에러 메시지는 더 개선해야 한다.


