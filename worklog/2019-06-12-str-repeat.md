# 2019-06-12
많이 쓰진 않지만 있으면 좋은 것들이 있다. 문자열 반복 연산자도 그 중 하나라고 생각한다. 문자열과 숫자를 피연산자(operand)로 지정하면 숫자 만큼 문자열을 반복해서 새로운 문자열을 만드는 연산자다.

문자열 합치기 연산자가 ##이므로 나는 처음에 #\*을 생각했다. 그러나 막상 써 보니 생각보다 가독성이 좋지 않았고 타이핑하기도 편치 않았다.

```
> "abc" #* 3
"abcabcabc"
```

그리고 생각해보니 그냥 곱하기 연산자를 써도 직관에 딱히 위배하는 것이 없었다.

```
직관적 사용
> "ab" * 3
"ababab"

교환법칙
> 3 * "ab"
"ababab"
```

구현도 간단하다. 기존에 number 타입용으로 만든 곱하기 오퍼레이터 처리 함수에 문자열과 숫자에 대한 처리 코드만 추가하면 된다.

```
static object_t op_mul(object_t op1, object_t op2)
{
    object_t ret;

    if (op1.type == object_type_number && op2.type == object_type_number)
    {
        ret.type = object_type_number;
        ret.value.number = op1.value.number * op2.value.number;
    }
    else if (op1.type == object_type_number && op2.type == object_type_string)   // number * "string"
    {
        ret = Obj_rept_string(op2, op1);
    }
    else if (op1.type == object_type_string && op2.type == object_type_number)  // "string" * number 
    {
        ret = Obj_rept_string(op1, op2);
    }
    else
    {
        add_error_msg(error_code_type_mismatch);
    }

    return ret;
}
```

"숫자 \* 문자열" 혹은 "문자열 \* 숫자" 형태를 둘 다 처리해야 하므로 Obj_rept_string()를 호출하는 코드가 두 가지 다른 형태로 두 곳에 보인다.