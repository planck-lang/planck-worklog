# 2019-05-28
첫 작업을 시작했다. main.c 파일과 Makefile 파일을 만들었다.

## main.c
일단 인터프리터로 동작해야 하니 프롬프트를 만들었다.

```
#include <stdio.h>
#include <stdbool.h>
#include <stdint.h>
#include <stdlib.h>
#include <string.h>
#include <readline/readline.h>
#include <readline/history.h>

int main(int argc, char* argv[])
{
    while (true)
    {
        char* buf = readline(">> ");
        if (buf == NULL)
        {
            puts("\nAnnyuonghi Gaseyo.\n");
            break;
        }

        if (strlen(buf) > 0)
        {
            add_history(buf);
            printf("%s\n", buf);
        }

        free(buf);
    }
}
```

이게 전체 코드다. readline() 함수로 프롬프트와 히스토리 기능은 따로 만들지 않고 처리했다. 입력을 받아서 그대로 출력하는 코드다. ctrl+D나 ctrl+C를 입력하면 끝낸다. 아니면 계속 반복하면서 입력을 출력한다. 종료할 때는 "안녕히 가세요"를 영어 자모로 출력한다.

readline() 함수는 내부에서 malloc()해서 문자열 포인터를 리턴하므로 매 루프마다 buf를 해제해야 한다.

이게 가장 기본 코드다.

## Makefile
소스 파일이 많아지면 어차피 만들어야 하니, 시작할 때 Makefile을 만들어 두는 것이 좋다.

```
PLANCK = planck

VB = @ 	# verbose

SRCDIR = src
OBJDIR = obj
BINDIR = bin

VPATH = src

CCFLAGS = -c -std=c11
LDFLAGS = -lreadline

OBJS = $(OBJDIR)/planck.o

all : create_dir $(OBJS)
	$(VB)echo "Linking.."
	$(VB)gcc -o $(BINDIR)/$(PLANCK) $(OBJS) $(LDFLAGS)
	
clean : create_dir
	$(VB)rm -f $(OBJDIR)/*
	$(VB)rm -f $(BINDIR)/*

create_dir :
	$(VB)mkdir -p $(OBJDIR)
	$(VB)mkdir -p $(BINDIR)

$(OBJDIR)/%.o : %.c
	$(VB)echo "compile " $<
	$(VB)gcc $(CCFLAGS) -o $@ $<
```

간단하다. 파일을 추가한다 해도 큰 틀은 딱히 변할 것이 없다. 라이브러리를 더 쓰면 LDFLAGS에 내용을 추가한다. 소스 파일을 새로 만들면 OBJS에 추가한다. 오브젝트 파일은 obj 디렉터리 및에 생기고 실행 파일은 bin 디렉터리 및에 생긴다.

