---
title: xv6 셸(sh.c) 코드 분석
date: 2026-09-21 20:00:00 +0900
categories: [학습, 운영체제]
tags: [xv6, shell, os, fork, exec, pipe, redirection, parser, c]
publish: true
---

> 대상 코드: [guilleiguaran/xv6 — sh.c](https://github.com/guilleiguaran/xv6/blob/master/sh.c) (493줄)
>
> MIT의 교육용 운영체제 xv6에 들어 있는 작은 셸이다. `fork`, `exec`, `pipe`, 파일 디스크립터 재배치를 한 파일에서 살펴볼 수 있다. 설명은 링크된 저장소의 버전을 기준으로 하며, bash나 다른 xv6 버전과 동작이 다를 수 있다.

## 0. 한 줄 요약

셸은 **"문자열 한 줄을 읽어 → 트리로 파싱하고 → 그 트리를 재귀적으로 실행하는 프로그램"** 이다. sh.c는 이 세 단계가 각각 `getcmd`, `parsecmd`, `runcmd` 함수 하나씩으로 딱 떨어진다.

```
사용자 입력 "cat a.txt | wc > out &"
        │
        ▼  getcmd()        — 한 줄 읽기
   char buf[100]
        │
        ▼  parsecmd()      — 재귀 하강 파서
   struct cmd 트리
        │
        ▼  runcmd()        — 트리 순회 + fork/exec/pipe
   실제 프로세스들이 돌아감
```

전체 구조를 함수 단위로 보면 이렇다.

| 구역 | 함수 | 역할 |
|---|---|---|
| 진입점 | `main`, `getcmd`, `panic` | 프롬프트 루프, `cd` 내장 명령 처리 |
| 실행기 | `runcmd`, `fork1` | 파싱된 트리를 실제 프로세스로 만든다 |
| 생성자 | `execcmd`, `redircmd`, `pipecmd`, `listcmd`, `backcmd` | 노드 5종을 `malloc`으로 만든다 |
| 렉서 | `gettoken`, `peek` | 문자열을 토큰으로 자른다 |
| 파서 | `parsecmd`, `parseline`, `parsepipe`, `parseexec`, `parseredirs`, `parseblock`, `nulterminate` | 토큰을 트리로 조립한다 |

---

## 1. 자료구조 — C로 흉내 낸 "상속"

파서가 만들어 내는 명령 트리의 노드는 5종류다.

```c
#define EXEC  1   // ls           실제 프로그램 실행
#define REDIR 2   // > out, < in  리다이렉션
#define PIPE  3   // a | b        파이프
#define LIST  4   // a ; b        순차 실행
#define BACK  5   // a &          백그라운드 실행
```

그리고 노드 구조체들이 이렇게 생겼다.

```c
struct cmd { int type; };                       // 공통 헤더 (부모 역할)

struct execcmd  { int type; char *argv[MAXARGS]; char *eargv[MAXARGS]; };
struct redircmd { int type; struct cmd *cmd; char *file; char *efile; int mode; int fd; };
struct pipecmd  { int type; struct cmd *left, *right; };
struct listcmd  { int type; struct cmd *left, *right; };
struct backcmd  { int type; struct cmd *cmd; };
```

다섯 구조체는 모두 `int type`으로 시작한다. 이 구현은 공통 선두 필드를 읽어 종류를 구분한 뒤, 해당 구조체 포인터로 캐스팅해 나머지 필드에 접근한다.

첫 멤버 앞에 패딩이 없다는 C의 배치 규칙만으로 서로 다른 구조체 포인터를 통한 접근 전체가 이식성 있게 보장되는 것은 아니다. 타입 별칭(aliasing) 규칙도 고려해야 한다. 이 저장소는 [Makefile](https://github.com/guilleiguaran/xv6/blob/master/Makefile)에서 `-fno-strict-aliasing`을 사용한다. 일반적인 C 코드에 이 패턴을 적용할 때는 [GCC의 타입 별칭 설명](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-fstrict-aliasing)도 참고해야 한다.

```c
switch(cmd->type){
case EXEC:
  ecmd = (struct execcmd*)cmd;   // 여기서 비로소 argv에 접근 가능
```

노드 종류에 따라 실행을 분기한다는 점은 객체지향의 다형성과 비슷하다. 다만 실제 가상 함수 테이블은 없고, **타입 태그 + switch + 캐스팅**으로 처리한다.

노드가 트리를 이루므로, 예를 들어 `cat a.txt | wc > out` 은 이렇게 표현된다.

```
        PIPE
       /    \
   EXEC      REDIR (fd=1, "out", O_WRONLY|O_CREATE)
 [cat,a.txt]     │
                EXEC [wc]
```

`argv` 옆에 왜 `eargv`("end argv")가 같이 있는지는 4-5절에서 설명한다.

---

## 2. `main()` — 프롬프트 루프

```c
int
main(void)
{
  static char buf[100];
  int fd;

  // 파일 디스크립터 0, 1, 2가 반드시 열려 있게 만든다
  while((fd = open("console", O_RDWR)) >= 0){
    if(fd >= 3){
      close(fd);
      break;
    }
  }

  while(getcmd(buf, sizeof(buf)) >= 0){
    if(buf[0]=='c' && buf[1]=='d' && buf[2]==' '){
      buf[strlen(buf)-1] = 0;          // 끝의 \n 자르기
      if(chdir(buf+3) < 0)
        printf(2, "cannot cd %s\n", buf+3);
      continue;
    }
    if(fork1() == 0)
      runcmd(parsecmd(buf));
    wait();
  }
  exit();
}
```

### 2-1. 표준 입출력 fd를 확보하는 `while` 루프

`open("console")`을 계속 부르다가 반환값이 3 이상이면 닫고 빠져나온다. 유닉스의 `open`은 **현재 비어 있는 가장 작은 번호의 fd**를 돌려준다는 규칙이 있으므로, 0·1·2가 모두 비어 있고 `open`이 성공한다면 다음과 같이 돈다.

| 호출 | 반환 | 의미 |
|---|---|---|
| 1회차 | 0 | stdin 확보 |
| 2회차 | 1 | stdout 확보 |
| 3회차 | 2 | stderr 확보 |
| 4회차 | 3 | 0·1·2가 다 찼다는 뜻 → 닫고 `break` |

이미 0·1·2가 열린 상태로 실행됐다면 첫 호출부터 3이 나와서 바로 빠져나온다. **0·1·2가 비어 있으면 안 되는 이유**는 리다이렉션 구현(5-2절)이 "fd를 닫고 바로 `open`하면 그 번호가 돌아온다"는 규칙에 통째로 의존하기 때문이다. 만약 fd 1이 원래부터 비어 있었다면, 엉뚱한 명령의 `open`이 1을 가져가 버려 출력이 뒤섞인다.

### 2-2. `cd`는 왜 특별 취급인가

> **별도 프로그램의 `chdir`로는 왜 부모 셸의 디렉터리를 바꿀 수 없는가?**

현재 작업 디렉터리(cwd)는 **프로세스마다 따로 갖는 상태**다. 셸이 다른 명령들처럼 `fork` 해서 자식에서 `chdir`을 호출하면, 디렉터리가 바뀌는 건 곧 죽어 없어질 자식뿐이고 부모 셸은 그대로다. 그래서 `cd`는 반드시 **셸 프로세스 자신이 직접** 호출해야 한다. 코드 주석도 딱 그렇게 적혀 있다 — `Chdir must be called by the parent, not the child.`

이렇게 셸 프로세스가 직접 처리해야 하는 명령을 **내장 명령(builtin)** 이라 하고, bash의 `cd`·`export`·`exit`도 같은 이유로 내장이다. 다만 xv6의 처리는 무척 거칠어서 `buf[0]=='c' && buf[1]=='d' && buf[2]==' '` 로 앞 세 글자만 본다. 그래서 앞에 공백이 붙은 `  cd /` 나 `cd` 단독(뒤에 공백 없음)은 내장 처리가 되지 않고 그냥 실행 경로로 흘러간다.

### 2-3. fork 후 wait — 셸과 명령의 프로세스 분리

```c
if(fork1() == 0)
  runcmd(parsecmd(buf));
wait();
```

명령 한 줄마다 자식을 하나 만들고, 자식이 명령을 수행하는 동안 부모(셸)는 `wait()`로 잠든다. 셸이 **자기 자신을 `exec`로 덮어쓰지 않는 이유**가 여기 있다. `exec`는 성공하면 현재 프로세스의 메모리 이미지를 통째로 새 프로그램으로 갈아치우므로, 셸이 직접 `exec("ls")`를 하면 `ls`가 끝나는 순간 셸도 같이 사라진다. 그래서 **"복제(fork)해서 그 복제본을 갈아치운다(exec)"** 라는 유닉스 특유의 2단 구조를 쓴다.

부수 효과가 하나 더 있다. **파싱도 자식에서 일어난다**(`parsecmd`가 `fork1()` 안쪽에 있다). 덕분에

- 파서가 `malloc`한 노드들을 아무도 `free`하지 않지만, 자식이 `exit`하면 주소 공간째로 회수되므로 누수가 문제되지 않는다.
- 문법 오류로 `panic("syntax")`이 터져도 죽는 건 자식뿐이고, **셸은 살아남아 다음 프롬프트를 띄운다**. 에러 처리를 따로 짜지 않고 프로세스 경계로 해결한 것이다.

### 2-4. 프롬프트를 fd 2로 찍는 이유

```c
int
getcmd(char *buf, int nbuf)
{
  printf(2, "$ ");   // 1(stdout)이 아니라 2(stderr)
  ...
}
```

`sh > log.txt` 처럼 셸의 표준출력이 파일로 돌려진 상황에서도 프롬프트가 파일을 오염시키지 않게 하려는 것이다. 실제 셸들도 프롬프트와 에러 메시지는 stderr로 보낸다. `gets`로 읽은 결과가 빈 문자열이면 EOF로 보고 `-1`을 돌려주며, 그 순간 `main`의 루프가 끝난다.

---

## 3. 파서 (1) — 렉서 `gettoken` / `peek`

파싱은 두 층으로 나뉜다. **렉서**가 문자열을 토큰으로 자르고, **파서**가 토큰을 트리로 조립한다.

```c
char whitespace[] = " \t\r\n\v";
char symbols[]    = "<|>&;()";
```

`gettoken(char **ps, char *es, char **q, char **eq)` 의 인자가 조금 낯설다.

- `ps` — **현재 읽는 위치를 가리키는 포인터의 포인터**. 토큰을 하나 먹을 때마다 `*ps`를 앞으로 민다. C에는 참조 전달이 없으니 이렇게 쓴다.
- `es` — 문자열의 끝(end of string). 버퍼 밖으로 나가지 않기 위한 경계.
- `q`, `eq` — 방금 읽은 토큰의 **시작**과 **끝 바로 다음** 주소를 담아 돌려주는 출력 인자. 필요 없으면 `0`을 넘긴다.

반환값은 토큰의 종류를 나타내는 정수다.

| 반환값 | 의미 |
|---|---|
| `0` | 입력 끝 |
| `'a'` | 일반 단어(명령 이름, 인자, 파일 이름) |
| `'|' '(' ')' ';' '&' '<' '>'` | 그 기호 자신 |
| `'+'` | `>>` (두 글자를 한 토큰으로 묶어 만든 가상의 기호) |

```c
case '>':
  s++;
  if(*s == '>'){   // ">>" 를 발견하면
    ret = '+';     // 별도 토큰 '+' 로 승격
    s++;
  }
  break;
default:           // 그 외에는 전부 단어
  ret = 'a';
  while(s < es && !strchr(whitespace, *s) && !strchr(symbols, *s))
    s++;
  break;
```

단어는 "공백도 기호도 아닌 글자가 이어지는 동안"으로 정의된다. 그래서 `ls>out` 처럼 붙여 써도 `ls` / `>` / `out` 세 토큰으로 잘 잘린다. 대신 **따옴표·이스케이프 처리가 아예 없어서** 공백이 든 파일 이름은 표현할 방법이 없다.

`peek(ps, es, toks)`는 "**소비하지 않고** 다음 토큰이 `toks` 안의 기호 중 하나인지" 만 확인한다. 단, 앞쪽 공백은 실제로 건너뛰어 `*ps`를 갱신한다(부수 효과가 있는 peek이다). 재귀 하강 파서가 분기 결정을 내릴 때 쓰는 **1토큰 미리보기(lookahead)** 역할이다.

---

## 4. 파서 (2) — 재귀 하강

이 파서의 문법을 EBNF 형태로 쓰면 다음과 같다. `*`는 0회 이상 반복, `?`는 생략 가능을 뜻한다. 각 규칙을 함수로 나누는 **재귀 하강 파서(recursive descent parser)** 구조다.

```
line  ::= pipe ('&')* (';' line)?
pipe  ::= exec ('|' pipe)?
exec  ::= block | (redir | word)*
block ::= '(' line ')' redir*
redir ::= ('<' | '>' | '>>') word
```

`exec`는 빈 입력도 허용한다. 빈 줄이나 `ls ;`의 마지막 부분은 `argv[0] == 0`인 EXEC 노드가 된다. 위 문법에는 인자 수와 입력 길이 제한은 포함하지 않았다.

우선순위는 **호출 깊이**로 표현된다. 가장 바깥에서 불리는 `parseline`이 다루는 `;`·`&`가 결합력이 제일 약하고, 가장 안쪽 `parseexec`의 단어들이 제일 강하다. 그래서 `a | b ; c` 가 `(a|b) ; c` 로 묶인다.

### 4-1. `parseline` — `&` 와 `;`

```c
cmd = parsepipe(ps, es);
while(peek(ps, es, "&")){
  gettoken(ps, es, 0, 0);
  cmd = backcmd(cmd);          // 기존 트리를 BACK으로 감싼다
}
if(peek(ps, es, ";")){
  gettoken(ps, es, 0, 0);
  cmd = listcmd(cmd, parseline(ps, es));   // 오른쪽을 재귀로 계속
}
```

`;`는 오른쪽 재귀라서 `a ; b ; c` 는 `LIST(a, LIST(b, c))` 가 된다.

> **주의할 문법 제약** — `&` 처리가 `while`이고 그 뒤에 곧바로 `;`만 확인하므로, `ls & echo hi` 처럼 `&` 뒤에 바로 명령을 붙이면 파싱이 끝난 뒤 `parsecmd`의 `if(s != es)` 검사에 걸려 `leftovers ...` 를 찍고 `panic("syntax")`으로 죽는다. 이어 쓰려면 `ls & ; echo hi` 처럼 `;`를 넣어야 한다. bash와 다른 지점이다.

### 4-2. `parsepipe` — `|`

```c
cmd = parseexec(ps, es);
if(peek(ps, es, "|")){
  gettoken(ps, es, 0, 0);
  cmd = pipecmd(cmd, parsepipe(ps, es));   // 오른쪽 재귀
}
```

`a | b | c` 는 `PIPE(a, PIPE(b, c))` 로 **오른쪽으로 치우친** 트리가 된다. 실행할 때 이 모양이 그대로 "왼쪽 하나 + 나머지 전부" 의 재귀 분할로 이어진다(5-3절).

### 4-3. `parseexec` — 단어와 리다이렉션 섞어 읽기

```c
if(peek(ps, es, "("))
  return parseblock(ps, es);

ret = execcmd();
cmd = (struct execcmd*)ret;          // 같은 객체를 두 이름으로 들고 있는다

argc = 0;
ret = parseredirs(ret, ps, es);      // 단어보다 리다이렉션이 먼저 와도 됨
while(!peek(ps, es, "|)&;")){
  if((tok=gettoken(ps, es, &q, &eq)) == 0)
    break;
  if(tok != 'a')
    panic("syntax");
  cmd->argv[argc] = q;
  cmd->eargv[argc] = eq;
  argc++;
  if(argc >= MAXARGS)
    panic("too many args");
  ret = parseredirs(ret, ps, es);    // 단어 사이사이에서도 다시 확인
}
cmd->argv[argc] = 0;
cmd->eargv[argc] = 0;
return ret;
```

**`ret`과 `cmd`는 역할이 다르다.**

- `cmd`는 계속 **안쪽의 execcmd**를 가리킨다. 단어를 만나면 여기 `argv`를 채운다.
- `ret`은 **바깥쪽 껍데기**다. `parseredirs`를 지날 때마다 `redircmd(...)`로 한 겹씩 감싸지며 바뀐다.

덕분에 `> out.txt echo hi` 처럼 리다이렉션이 앞에 와도, `echo > out.txt hi` 처럼 중간에 끼어도 전부 정상 처리된다. 루프가 끝나고 `cmd->argv[argc] = 0` 으로 마무리하는데, 이때 `cmd`는 이미 여러 겹 REDIR에 싸여 있어도 여전히 같은 execcmd를 가리키므로 문제가 없다.

`argv` 배열 크기는 `MAXARGS`(=10)이지만 마지막 NULL 포인터 자리가 필요하다. **명령 이름 포함 최대 9개**, 즉 추가 인자는 최대 8개다. 열 번째 단어를 읽으면 `argc >= MAXARGS` 검사에서 `panic("too many args")`가 발생한다.

### 4-4. `parseredirs`

```c
while(peek(ps, es, "<>")){
  tok = gettoken(ps, es, 0, 0);
  if(gettoken(ps, es, &q, &eq) != 'a')
    panic("missing file for redirection");
  switch(tok){
  case '<': cmd = redircmd(cmd, q, eq, O_RDONLY, 0); break;
  case '>': cmd = redircmd(cmd, q, eq, O_WRONLY|O_CREATE, 1); break;
  case '+': cmd = redircmd(cmd, q, eq, O_WRONLY|O_CREATE, 1); break;  // >>
  }
}
```

마지막 인자가 **바꿔치기할 fd 번호**다. `<`는 0(stdin), `>`와 `>>`는 1(stdout).

> **이 버전의 `>`와 `>>`는 모두 파일 처음부터 기록한다.** 렉서는 둘을 구분하지만 파일을 여는 모드는 같다. 커널에 `O_APPEND`와 `O_TRUNC`가 없으므로, 이어쓰기나 기존 내용을 비우는 동작을 하지 않는다. 새 내용이 짧으면 기존 뒷부분이 남는다. 예를 들어 `abcdef`가 있는 파일의 처음에 `xy` 두 바이트를 쓰면 `xycdef`가 된다. 일반적인 셸의 `>`와 다른 부분이다. [파일 열기 구현](https://github.com/guilleiguaran/xv6/blob/master/sysfile.c), [플래그 정의](https://github.com/guilleiguaran/xv6/blob/master/fcntl.h)

### 4-5. `nulterminate` — 종료 문자를 나중에 넣는 이유

`argv[i]`는 `malloc`으로 복사한 새 문자열이 아니라 **입력 버퍼 `buf` 안을 그대로 가리키는 포인터**다. 메모리를 아끼는 대신 문제가 하나 생긴다. C 문자열은 `\0`으로 끝나야 하는데, 토큰을 읽는 즉시 그 자리에 `\0`을 박으면 **바로 다음 토큰의 첫 글자를 뭉개 버린다**. `ls>out` 같은 입력에서 치명적이다.

그래서 파싱 중에는 끝 위치만 `eargv` / `efile`에 기록해 두고, **파싱이 완전히 끝난 뒤에** 트리를 한 번 더 순회하면서 그제야 종료 문자를 박는다.

```c
case EXEC:
  ecmd = (struct execcmd*)cmd;
  for(i=0; ecmd->argv[i]; i++)
    *ecmd->eargv[i] = 0;     // 이제 안전하게 NUL을 박는다
  break;
```

`argv`와 `eargv`가 쌍으로 존재하는 이유가 바로 이것이다. 파싱 중에는 `(시작, 끝)` 쌍으로 "길이를 가진 문자열"처럼 다루다가, 마지막에 한 번에 C 문자열로 확정하는 것이다.

---

## 5. `runcmd()` — 트리를 프로세스로

```c
// Execute cmd.  Never returns.
void runcmd(struct cmd *cmd)
```

주석 그대로 **절대 반환하지 않는다.** switch의 모든 분기가 끝나면 마지막 줄의 `exit()`로 프로세스가 죽는다. 이 "돌아오지 않는다"는 성질이 아래 구현들을 크게 단순하게 만든다.

### 5-1. EXEC

```c
ecmd = (struct execcmd*)cmd;
if(ecmd->argv[0] == 0)
  exit();
exec(ecmd->argv[0], ecmd->argv);
printf(2, "exec %s failed\n", ecmd->argv[0]);
```

`exec`이 성공하면 프로세스 이미지가 통째로 교체되므로 **그 다음 줄은 영원히 실행되지 않는다**. 따라서 `printf`에 도달했다는 사실 자체가 곧 실패를 뜻한다. `exec` 뒤에 에러 처리를 붙이는 유닉스의 관용구다.

빈 명령(`argv[0] == 0`)은 조용히 종료한다. `ls ;` 처럼 `;` 뒤가 비어 있을 때 만들어지는 빈 EXEC 노드를 위한 처리다.

경로 탐색(`PATH`)은 없다. `ls` 같은 상대 경로는 현재 작업 디렉터리에서 찾는다. 처음에는 작업 디렉터리가 `/`여서 루트의 명령을 이름만으로 실행할 수 있다. 다른 디렉터리로 이동한 뒤에는 `/ls`, `/cat`처럼 절대 경로가 필요할 수 있다.

### 5-2. REDIR — `close` 후 `open`

```c
rcmd = (struct redircmd*)cmd;
close(rcmd->fd);                       // 예: 1번(stdout)을 닫는다
if(open(rcmd->file, rcmd->mode) < 0){  // 가장 작은 빈 번호 = 방금 닫은 1번
  printf(2, "open %s failed\n", rcmd->file);
  exit();
}
runcmd(rcmd->cmd);                     // 그 상태로 안쪽 명령 실행
```

닫은 fd 번호를 `open`이 다시 사용하도록 만드는 것이 핵심이다.

> `open`은 **현재 비어 있는 가장 작은 번호의 fd**를 반환한다.

`ls > out.txt` 의 실행 흐름:

```
[자식 프로세스]
fd 0 ── console
fd 1 ── console      close(1)      fd 1 ── (빈 자리)
fd 2 ── console   ───────────►     fd 2 ── console
                                       │
                     open("out.txt")   │  1번이 비어 있으므로 1번이 반환됨
                  ────────────────►    ▼
                                   fd 1 ── out.txt
                                       │
                     exec("ls")        ▼
                              ls는 "그냥 1번에 쓸 뿐"인데
                              그 1번이 이미 out.txt다
```

`ls`는 자기가 리다이렉트됐다는 사실을 **전혀 모른다**. 그냥 평소처럼 fd 1에 쓸 뿐이고, 셸이 미리 1번의 의미를 바꿔 놓은 것이다. "모든 것은 파일 디스크립터"라는 유닉스 설계가 프로그램과 입출력 대상을 얼마나 깔끔하게 분리해 주는지 보여주는 대표 예다.

같은 fd를 여러 번 리다이렉트하면 **입력의 역순으로 적용**된다. `echo hi > a > b`의 트리는 다음과 같다.

```text
REDIR(b) → REDIR(a) → EXEC(echo, hi)
```

`b`를 먼저 열고, 안쪽에서 fd 1을 다시 닫아 `a`를 연다. 두 파일 열기가 모두 성공하면 최종 출력은 **a**로 간다. 일반적인 셸에서 마지막 리다이렉션이 우선하는 것과 다르다.

`runcmd`가 재귀 호출이라는 점도 중요하다. `ls > a < b` 처럼 REDIR이 여러 겹이면 바깥부터 차례로 fd를 바꾸고 안쪽으로 내려간다. 리다이렉션은 **자식 프로세스 안에서만** 일어나므로 셸 자신의 fd는 멀쩡하다.

### 5-3. PIPE — 입출력 연결과 불필요한 fd 닫기

```c
pcmd = (struct pipecmd*)cmd;
if(pipe(p) < 0)
  panic("pipe");
if(fork1() == 0){          // 왼쪽 명령: 쓰는 쪽
  close(1);
  dup(p[1]);               // 빈 1번에 p[1]의 복제본이 들어감
  close(p[0]);
  close(p[1]);
  runcmd(pcmd->left);
}
if(fork1() == 0){          // 오른쪽 명령: 읽는 쪽
  close(0);
  dup(p[0]);               // 빈 0번에 p[0]의 복제본이 들어감
  close(p[0]);
  close(p[1]);
  runcmd(pcmd->right);
}
close(p[0]);               // 파이프 관리 프로세스는 양쪽 끝을 닫는다
close(p[1]);
wait();
wait();
```

`pipe(p)`는 `p[0]`(읽기 끝)과 `p[1]`(쓰기 끝), 연결된 fd 두 개를 만든다. `dup(fd)`는 그 fd의 복제본을 **가장 작은 빈 번호**에 만들어 준다 — REDIR과 정확히 같은 규칙을 이용한다.

여기서 **부모는 대화형 셸이 아니라 `runcmd(PIPE)`를 수행하는 프로세스**다. 단순한 파이프 명령에서는 셸의 자식이며, 다시 왼쪽·오른쪽 명령을 위한 자식을 만든다.

`cat a.txt | wc`를 그림으로 보면:

```
                   ┌──────────────┐
                   │  파이프 버퍼  │
                   └──────────────┘
                     ▲           │
              쓰기   │           │   읽기
                     │           ▼
        ┌────────────────┐   ┌────────────────┐
        │  자식 1: cat    │   │  자식 2: wc     │
        │  fd 1 ─► 파이프 │   │  파이프 ─► fd 0 │
        └────────────────┘   └────────────────┘
                     ▲           ▲
                     └─── fork ──┘
                    부모: 양쪽 끝 close 후 wait ×2
```

**쓰기 끝을 불필요하게 열어 두면 EOF를 기다리는 명령이 멈출 수 있다.**

파이프에서 양수 길이로 읽기를 요청할 때, **쓰기 끝의 모든 참조가 닫히고 버퍼의 데이터도 소진되면** `read`가 0을 반환한다(EOF). `cat`이 끝나며 파이프에 연결된 fd 1을 닫아도, **부모가 `p[1]`을 여전히 열어 두고 있으면** 커널이 보기엔 "아직 쓸 사람이 남아 있다"가 된다. 그러면 `wc`는 영원히 읽기를 기다리고, 부모는 `wait()`로 `wc`를 기다리며 셸 전체가 멈춘다. 그래서 자식들도 쓰지 않는 쪽 fd를 즉시 닫고, 부모도 양쪽을 전부 닫는다. **"안 쓰는 파이프 끝은 즉시 닫는다"** 는 파이프 프로그래밍의 제1원칙이다.

`wait()`를 두 번 부르는 것은 자식이 둘이기 때문이다. xv6의 `wait()`는 특정 자식을 지정할 수 없어서 어느 쪽이 먼저 끝나든 그냥 두 번 거둔다.

`a | b | c` 처럼 3단이면 트리가 `PIPE(a, PIPE(b, c))` 이므로, 오른쪽 자식이 `runcmd(PIPE(b,c))`를 실행하며 파이프를 하나 더 만들고 손자 둘을 낳는다. 재귀만으로 임의 길이의 파이프라인이 처리된다.

### 5-4. LIST — `;`

```c
lcmd = (struct listcmd*)cmd;
if(fork1() == 0)
  runcmd(lcmd->left);   // 왼쪽만 자식에게 맡긴다
wait();                 // 끝날 때까지 기다리고
runcmd(lcmd->right);    // 오른쪽은 "자기 자신"이 그대로 이어서 실행
```

왼쪽과 오른쪽의 실행 방식이 다르다. 왼쪽은 `fork`해서 맡기지만 오른쪽은 포크하지 않고 현재 프로세스가 그대로 실행한다. `runcmd`가 절대 반환하지 않으므로, 이 프로세스는 이 시점부터 그냥 "오른쪽 명령"이 되어 버린다. 일종의 꼬리 호출이고, 불필요한 프로세스 하나를 아낀다.

### 5-5. BACK — `&`

```c
bcmd = (struct backcmd*)cmd;
if(fork1() == 0)
  runcmd(bcmd->cmd);
// wait() 없음 → 바로 아래 exit()으로 떨어진다
```

BACK 분기는 자식을 만든 뒤 기다리지 않고 종료한다. 대화형 셸은 이 중간 프로세스만 기다리므로 실제 명령의 종료 전에 다음 입력을 받을 수 있다.

프로세스 관계를 따라가 보면 이렇다.

```
셸(main)
  └─ fork ─► 자식 A : runcmd(BACK)          ← 셸은 A를 wait 중
                └─ fork ─► 자식 B : 실제 명령
             A는 곧바로 exit()  ─────────────► 셸의 wait()가 즉시 풀림
                                              → 프롬프트가 바로 돌아옴
             B는 계속 실행 (부모를 잃고 init에 입양됨)
```

셸이 기다리는 건 A뿐이고 A는 바로 죽으므로 프롬프트가 즉시 돌아온다. 고아가 된 B는 `init`이 물려받아, 끝나면 대신 거둬 준다. 대신 xv6 셸에는 **작업 제어(job control)가 없다** — `jobs`, `fg`, `bg`, Ctrl-Z 같은 것이 전혀 없고, 백그라운드 프로세스의 출력은 그냥 터미널에 섞여 나온다.

---

## 6. 전체 추적 예시: `cat a.txt | wc > out.txt &`

이 버전의 `wc`는 `-l` 옵션을 지원하지 않는다. `-l`을 주면 파일 이름으로 해석하므로 옵션 없이 표준입력을 읽게 한다. 출력은 줄·단어·바이트 수다. [wc.c 원문](https://github.com/guilleiguaran/xv6/blob/master/wc.c)

### 파싱 결과 트리

```
BACK
 └─ PIPE
     ├─ EXEC  [cat, a.txt]
     └─ REDIR (fd=1, file="out.txt", O_WRONLY|O_CREATE)
         └─ EXEC [wc]
```

`parseline`이 `parsepipe`를 먼저 부르므로 `|`가 `&`보다 안쪽에서 묶이고, 마지막에 전체가 BACK으로 감싸진다.

### 실행 흐름

번호는 역할을 따라 설명한 순서다. `fork` 이후 프로세스들의 실제 실행 순서는 스케줄링에 따라 달라질 수 있다.

1. **셸** — `fork1()`으로 자식 A를 만들고 `wait()`로 잠든다.
2. **A** — `parsecmd`로 위 트리를 만든 뒤 `runcmd(BACK)`. 자식 B를 만들고 **기다리지 않고** `exit()`. → 셸의 `wait()`가 풀려 프롬프트가 즉시 돌아온다.
3. **B** — `runcmd(PIPE)`. `pipe(p)` 후 자식 C, D를 만든다.
4. **C** — `close(1); dup(p[1])` 로 stdout을 파이프에 연결, 파이프 fd 정리 후 `exec("cat", ...)`. `a.txt` 내용을 파이프로 흘린다.
5. **D** — `close(0); dup(p[0])` 로 stdin을 파이프에 연결하고 `runcmd(REDIR)`. `close(1)` 후 `open("out.txt", O_WRONLY|O_CREATE)` → fd 1이 out.txt가 된다. 그 다음 `exec("wc", argv)`를 호출한다(`argv`는 `{"wc", 0}`).
6. **B** — 파이프 양쪽 끝을 닫고 `wait()` 두 번. C, D가 끝나면 `exit()`.
7. `out.txt`에 줄·단어·바이트 수가 기록된다. 사용자는 그 사이 다음 명령을 입력할 수 있다. 기존 출력 파일이 더 길었다면 뒷부분이 남을 수 있으므로, 결과를 확인할 때는 새 파일을 사용하는 편이 명확하다.

**`wc`의 입장**에서 보면 자기는 stdin에서 읽어 stdout에 쓰는 평범한 프로그램일 뿐이다. 그 stdin이 파이프이고 stdout이 파일이라는 사실은 셸이 `exec` 직전에 몰래 꾸며 놓은 무대장치다. 이 분리가 유닉스 철학("한 가지만 잘 하는 작은 도구들을 파이프로 잇는다")을 물리적으로 가능하게 만드는 구현적 토대다.

---

## 7. 한계와 알아 둘 점

sh.c를 읽고 나면 "진짜 셸에는 뭐가 더 있어야 하는가"가 반대로 보인다.

| 없는 기능 | 설명 |
|---|---|
| 따옴표·이스케이프 | `"a b.txt"` 같은 공백 포함 파일 이름 불가. 렉서가 공백을 무조건 구분자로 본다 |
| 와일드카드 | `*.c`를 파일 목록으로 확장하지 않는다. 인자로 쓰면 문자열 그대로 실행 프로그램에 전달된다 |
| 환경 변수 | `$HOME`, `export` 없음. 변수 자체가 없다 |
| `PATH` 탐색 | `exec`에 준 이름 그대로 연다 |
| `>>` 이어쓰기 | 토큰은 구분하지만 `>` 와 동일하게 동작(커널에 `O_APPEND` 없음) |
| `2>`, `2>&1` | fd 번호를 지정하는 리다이렉션 없음. `<`는 0, `>`는 1 고정 |
| AND/OR 연결 | bash의 `&&` 나 OR(세로줄 두 개) 같은 조건부 연결 없음. `;` 만 있다 |
| 작업 제어 | `jobs` / `fg` / `bg` / Ctrl-Z 없음 |
| 히스토리·탭 완성 | 없음 |
| 에러 처리 | 자식의 문법·실행 오류는 부모 셸을 종료시키지 않는다. 단, `main`의 `fork1()` 실패는 셸 자체를 종료시킨다 |
| 메모리 해제 | `free` 없음. 자식이 `exit`할 때 통째로 회수 |
| 입력 길이 | 종료 문자 포함 100바이트 버퍼. 명령 이름 포함 최대 9개 단어 |

xv6은 셸을 커널 기능 위에서 어떻게 구성하는지 살펴보기 위한 교육용 코드다. [MIT 6.828의 2018년 셸 과제](https://pdos.csail.mit.edu/6.828/2018/homework/xv6-shell.html)는 별도의 미완성 셸을 사용한다. 명령 실행·리다이렉션·파이프 구현을 다루고, 명령 목록·서브셸·백그라운드 실행은 선택 과제다. 여기서 분석한 `sh.c`와 과제용 코드를 구분해야 한다.

---

## 8. 이 코드에서 가져갈 것 5가지

1. **셸은 사용자 프로그램이다.** `fork`, `exec`, `open`, `close`, `dup`, `pipe`, `wait`, `chdir` 등의 시스템 콜을 조합한다. 입력·출력과 종료에는 `read`, `write`, `exit`도 사용한다.
2. **`fork`와 `exec` 사이에서 입출력을 준비한다.** 자식 프로세스를 만든 뒤 새 프로그램을 실행하기 전에 fd를 재배치한다. 이것이 이 셸의 리다이렉션과 파이프 구현 방식이다.
3. **가장 작은 빈 fd가 재사용된다.** 필요한 낮은 번호의 fd가 열려 있다는 전제 아래, `close(1)` 후 `open`이나 `dup`으로 표준출력을 바꾼다. `main`은 시작할 때 0·1·2가 열려 있도록 준비한다.
4. **재귀 하강 파서는 문법을 함수로 나눈다.** 이 파서는 함수 호출 관계로 우선순위를, 오른쪽 재귀로 파이프와 명령 목록의 결합 구조를 표현한다.
5. **프로세스 경계가 오류의 영향을 제한한다.** 자식에서 파싱 오류가 나도 부모 셸은 다음 명령을 받을 수 있다. 명령 트리의 메모리도 성공한 `exec`나 프로세스 종료 시 회수된다. 부모 셸의 `fork` 실패까지 복구하는 것은 아니다.

## 관련

- [[0907 운영체제 Introduction to Operating System]] — 운영체제 수업 도입부, 시스템 콜과 커널/사용자 모드
- [[0909]] — 프로세스와 시스템 콜 관련 수업 노트
