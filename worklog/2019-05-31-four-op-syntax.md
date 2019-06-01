# 2019-05-29 ~ 2019-05-31

## 사칙연산
가장 먼저 할 일은 당연히 전혀 새로움이 필요없는 문법을 먼저 작업하는 것이다. lisp처럼 전위 연산 할 것이 아니면 사칙연산은 거의 모든 프로그래밍 언어의 문법이 똑같다.

### bision (yacc)

```
expr : NUMBER               {CodeGen_add_number($1); $$ = 0;}
     | '(' expr ')'         {$$ = $2;}
     | expr '+' expr        {CodeGen_add_opcode(opcode_add); $$ = 0;}
     | expr '-' expr        {CodeGen_add_opcode(opcode_sub); $$ = 0;}
     | expr '*' expr        {CodeGen_add_opcode(opcode_mul); $$ = 0;}
     | expr '/' expr        {CodeGen_add_opcode(opcode_div); $$ = 0;}
     | expr '%' expr        {CodeGen_add_opcode(opcode_mod); $$ = 0;}
     ;
```

전체 코드는 레포를 참고하고, 문법 코드는 위와 같다. 사실 저렇게 하면 문법이 모호하다(ambiguous). 그래서 yacc에 추가 정보를 주어 모호함을 없앤다.

```
%left '+' '-'
%left '*' '/' '%'
```

%left 지시어 다음에 나오는 기호를 만나면 왼쪽을 먼저 해석하라는 거다. 물론 %right 지시어도 있다. 그리고 이 지시어는 아랫 줄에 있는 지시어가 윗 줄에 있는 지시어보다 우선순위가 높다. 그러므로 곱하기, 나누기, 나머지가 더하기, 빼기보다 우선순위가 높다. 수학 규칙하고 같다.

### flex (lex)
현재까지 토큰은 숫자 밖에 없다. 연산자 기호는 따로 토큰으로 만들지 않고 그냥 바로 보낸다.

```
[ \t]*      ;

[0-9]+("."[0-9]*)? {
    char* ptr;
    yylval.double_value = strtod(yytext, &ptr);
    return NUMBER;
}

.           {       // It must be the last rule because it can match ANY characters
    return *yytext;
}
```

공백과 탭은 무시한다. 그냥 숫자 혹은 소수점 붙은 숫자를 인식해서 NUMBER 토큰으로 보내고 값은 double 타입 값으로 만들어서 bison으로 보낸다. 그외에 한 글자 짜리 문자는 그대로 bison으로 보낸다. 한 글자 바로 보내는 규칙은 반드시 제일 아래에 있어야 한다. 그렇지 않으면 한 글자만 입력했을 때 다른 규칙을 해석하지 않는다.

## 코드 생성기
뭐 쉽게 만들려면 파싱하면서 그냥 바로 계산해도 되지만, 이러면 확장성이 많이 떨어진다. 그리고 뽀대도 안난다. 바이트 코드 생성해서 VM으로 보내 해석하는게 멋져 보이니까 그렇게 만들기로 했다. 그래서 bison에서는 문법을 파싱하면서 바로 결과를 계산하는 것이 아니라 코드 생성기에 바이트 코드를 생성한다. 위 bison 문법 소스 코드 보면 알 것이다.

```
static struct _generated_code_status_t_ {
    uint64_t*   buffer;
    uint32_t    len;
    uint32_t    limit;
} s_generated_code = {0};
```

바이트 코드를 따로 파일로 만드는 것이 아니기 때문에 그냥 메모리 버퍼에 바이트 코드를 만든다. 이 메모리 버퍼를 관리하는 구조체와 구조체 변수다. buffer에 메모리를 할당하고 len이 현재까지 사용한 버퍼, limit가 버퍼 최대 크기다. 왜 구조체 구성을 이렇게 했느냐. 의도가 뻔히 보이지 않는가? buffer에 바이트 코드를 쌓으면 len이 증가한다. 그러다가 len이 limit에 근접하면 buffer 할당 메모리 크기를 늘리고 limit를 늘린다.

```
static void check_code_buffer(void)
{
    const uint32_t one_time_buffer_uint = 4096;
    const uint32_t margin = 4;

    if (s_generated_code.buffer == 0)
    {
        s_generated_code.buffer = (uint64_t*)malloc(sizeof(uint64_t) * one_time_buffer_uint);
        s_generated_code.limit = one_time_buffer_uint;
    }

    if (s_generated_code.len >= (s_generated_code.limit - margin))
    {
        uint32_t new_size = s_generated_code.limit + one_time_buffer_uint;
        s_generated_code.buffer = (uint64_t*)realloc(s_generated_code.buffer, (sizeof(uint64_t) * new_size));
        s_generated_code.limit = new_size;
    }
}
```

이 동작하는 코드가 위 코드다. 한 번에 할당하는 기본 메모리 크기는 바이트 코드 버퍼 4096칸이다. 현재 코드에서 나는 바이트 코드 버퍼 한 칸을 uint64_t 타입으로 정했다. 그래서 8 바이트 * 4096 = 32768 바이트 (32KB) 단위로 한 번에 할당한다. 만약 바이트 코드가 32KB를 넘어가면 그 다음에 코드 버퍼는 64KB로 늘어난다.

```
void CodeGen_add_opcode(opcode_t opcode)
{
    check_code_buffer();
    s_generated_code.buffer[s_generated_code.len++] = (uint64_t)opcode;
}

void CodeGen_add_number(double number)
{
    check_code_buffer();
    CodeGen_add_opcode(opcode_push);
    memcpy(&s_generated_code.buffer[s_generated_code.len++], &number, sizeof(double));
}
```

현재 바이트 코드는 opcode와 숫자만 들어간다. 숫자는 double 타입이지만 바이트 코드 버퍼는 uint64_t 타입이기 때문에 memcpy로 넣는다. 이 글을 쓰다보니 굳이 저러지 말고 바이트 코드 버퍼를 공용체로 만들면 될 것 같다는 생각이 든다. 글 다 쓰고 고쳐야지.

## 버추얼 머신
버추얼 머신은 opcode를 해석한다. 현재 opcode는 사칙연산과 그것을 수행하는데 필요한 최소의 opcode만 있다.

```
typedef enum _opcode_t_ {
    opcode_nop = 0,
    opcode_push,
    opcode_add,
    opcode_sub,
    opcode_mul,
    opcode_div,
    opcode_mod,
    opcode_halt,
    opcode_MAXNUM
} opcode_t;
```

nop는 아무것도 안하는 거다. push는 스택에 숫자를 넣는다. add, sub, mul, div, mod는 더하기, 빼기, 곱하기, 나누기, 나머지다. hal는 VM을 종료한다. MAXNUM은 opcode의 전체 개수를 추적하려고 만든거다.

VM이기 때문에 스택이 있어야 한다. 그리고 코드를 읽는 가장 기본적인 레지스터가 필요하다.

```
static struct _vm_registers_t_ {
    uint64_t*   pc;
    uint64_t*   lr;
    uint32_t    sp;
} s_vm_registers = {0};

static struct _vm_stack_t_ {
    double*   stack;
    uint32_t    limit;
} s_vm_stack = {0};
```

pc는 바이트 코드 버퍼를 포인팅한다. program counter다. 현재 실행 중인 opcode 혹은 데이터의 위치를 가리킨다. sp는 stack pointer다. 스택 메모리의 top을 포인팅한다. stack은 스택 메모리 버퍼 포인터다. 스택에는 현재 숫자만 들어가니까, double 타입 포인터다. limit는 스택 메모리의 최댓값이다. 바이트 코드 버퍼와 같은 알고리즘으로 스택 메모리를 limit에 근접하게 쓰면 스택 메모리 할당을 키운다.

```
static void check_stack(void)
{
    const uint32_t margin = 4;

    if (s_vm_registers.sp >= (s_vm_stack.limit - margin))
    {
        uint32_t new_size = s_vm_stack.limit + (sizeof(double) * BLOCK_STACK_SIZE);
        s_vm_stack.stack = (double*)realloc(s_vm_stack.stack, new_size);
        s_vm_stack.limit = new_size;
    }
}
```

코드 형태는 check_code_buffer() 함수와 같다. BLOCK_STACK_SIZE는 4096이다. double도 8 바이트므로 한 번에 늘리는 메모리 크기는 32KB다.

```
        case opcode_push:
        {
            double value;

            pc++;                       // next code has value which will be pushed into stack
            memcpy(&value, pc, sizeof(double));
            push_stack(value);          // value has been pushed into stack and increase pc to execute next code
            pc++;
            break;
        }
        case opcode_add:
        case opcode_sub:
        case opcode_mul:
        case opcode_div:
        case opcode_mod:
        {
            double op2 = pop_stack(); // get value from stack. Pop the 2nd operand first because it is stack.
            double op1 = pop_stack(); // get value from stack. Pop the 1st operand.
            double ret = 0;
            switch (opcode)             // calculation
            {
            case opcode_add:
                ret = op1 + op2;
                break;
            case opcode_sub:
                ret = op1 - op2;
                break;
            case opcode_mul:
                ret = op1 * op2;
                break;
            case opcode_div:
                ret = op1 / op2;
                break;
            case opcode_mod:
                ret = (uint64_t)op1 % (uint64_t)op2;
                break;
            default:
                return;
            }
            push_stack(ret);            // result value has been pushed into stack
            pc++;                       // increase pc to execute next code
            break;
        }
```

opcode 실행은 큰 switch-case 구문이다. 전체 코드는 레포에서 다운받고, 이 글에는 필요한 코드만 실었다. push면 바이트 코드 버퍼에서 값을 읽어서 double 타입으로 복사한 다음 스택에 넣는다. 더하기, 빼기, 곱하기, 나누기, 나머지 (이제 보니 오칙연산이네..) 연산은 한 번에 처리한다. 어차피 연산자 말고는 처리하는 순서가 같기 때문이다.
```
