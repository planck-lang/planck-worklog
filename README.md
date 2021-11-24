# planck-worklog

## 2021-07-07
다시 시작.
세번째.
이번에는 VM을 먼저 만들고 검증한다. 그 후에 해당 VM 위에서 동작하는 프로그래밍 언어를 만든다.

## 2021-07-08
https://github.com/planck-lang/planck/commit/a99a431d7de8ce58811d4e258c230b97b55f49df

VM의 명령어와 레지스터, 메모리 구조를 추가.
그냥 생각나는 대로 추가함

## 2021-07-10
https://github.com/planck-lang/planck/commit/4a5820f87b59ab8015c8e6f17e446b605c7c7a81

초기화 코드 추가.
VM의 메모리는 미리 할당해 놓는다. 추후에 동적으로 크기가 늘어날 수 있도록 고려.
status register 추가.

## 2021-07-13
https://github.com/planck-lang/planck/commit/277d4e0e66b62dc5f6ecaba39e780ae2bacaacea

opcode 추가.
64비트로 결정함. 그래서 PC와 SP도 64비트 씩 커진다.
fetch와 decode 기본 코드 작성 (그냥 switch-case 껍데기)
opcode를 표현하는 큰 union 구조체 작성 시작.
리턴 어드레스 레지스터(ld)도 선언

## 2021-07-14
https://github.com/planck-lang/planck/commit/ea5bfff9a0d79cf965b386c096a87a7d84672eda
https://github.com/planck-lang/planck/commit/7a902381e64a89f3138bad821bdb917adfbc0b53

decode와 execute 단계를 분리. 딱히 큰 의미는 없는데 그냥 분리.
opcode union 구조체는 계속 다듬는 중

## 2021-07-21
https://github.com/planck-lang/planck/commit/e5246070904daa44d027f841e15af7c190fcae26

대충 opcode union 구조체 작성 완료. 그래도 계속 고쳐야 함.

## 2021-08-05
https://github.com/planck-lang/planck/commit/e9f8d1d7e14184fd896071c886c84c32cfe261e0

시험삼아 스택은 요렇게 동작해야 한다라고 코드 써봄.
abort exception 상황도 추가. 별다른 동작은 없음. 그냥 논리상 필요해서.

## 2021-08-07
https://github.com/planck-lang/planck/commit/0f8c6a0868e45291ca66fcf34e98774cf4867efa

내가 만든 simple C unittest framework를 붙였다.
첫 유닛 테스트로 스택 동작을 검증하는 유닛 테스트를 작성.
유닛 테스트 통과하면서 기존 구현했던 스택 동작 코드 고침.

## 2021-08-16
https://github.com/planck-lang/planck/commit/0734917a888e55fb2d12b957b3dfed6d0fe0dd09
https://github.com/planck-lang/planck/commit/be5deb792b15a70eaf3158dbcdc78ae15737c9d1
https://github.com/planck-lang/planck/commit/462fd9bde01cd4ebbe04a5f9a808ca2425534309
https://github.com/planck-lang/planck/commit/c465d661858b27ac01b919099280d160e5034ef2
https://github.com/planck-lang/planck/commit/f34094e69bb206484fad5b0cf9db25071e1f34b2
https://github.com/planck-lang/planck/commit/fa1e2f26411d0020ae9a6bb7b48630bc9b565af1

gitignore 파일 추가
유닛 테스트 프레임 워크 코드 개선.
스택 push 유닛 테스트
뭔가 32비트 넘어가는 비트 연산이 코딩하기 짜증난다. 그래서 opcode에 32비트가 넘어가는 비트맵을 모두 32비트 이하로 줄였다. 남는 비트는 그냥 reserved로 남겨둠
스택 pop, push 동작 검증

코딩을 많이 했네.

## 2021-08-17
https://github.com/planck-lang/planck/commit/85c41c54eee0cfea788d96b6f6623736538105be
https://github.com/planck-lang/planck/commit/38670a177e4aa6a7fdbe61ddf42b650e466b1d32
https://github.com/planck-lang/planck/commit/f235896523406157ed16b6d9f252140e94858277

구조체 padding 하기 싫어서 opcode field 크기 조정.
opcode 구조체 멤버 변수 이름을 약간 통일감있게 고침.
mov instruction의 유닛 테스트와 동작 코드를 썼다.
nop instruction의 유닛 테스트와 동작 코드를 썼다.

## 2021-10-07
https://github.com/planck-lang/planck/commit/5fd95da5ee417f53d93ae5517fac37be53395228

메모리 operation을 하는 STR와 LDR instruction 구현 준비.
유닛 테스트 껍데기만 쓰고 말았음. 여기까지 쓰고 하기 싫어서.

## 2021-10-12
https://github.com/planck-lang/planck/commit/ccedf89cd011fd59a6e8bf861c69d760b478d66d

STR instruction 유닛 테스트 조금 더 썼음.

## 2021-11-20
https://github.com/planck-lang/planck/commit/2a086e4b593dcdd7185f84e008d679556e8d5032

STR instruction 중에 RegId = Reg bitmap 형태 opcode를 검증하는 유닛 코드를 썼다.