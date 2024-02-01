---
title: "Golang Main함수 테스트하기"
date: 2024-02-01T09:50:57+09:00
draft: false
tags: ["gloang"]
---

## 일러두기 
이 글은 <실무에 바로 쓰는 Go 언어 핸즈온 가이드>의 연습 문제 1.1을 다룹니다.
연습 문제를 풀어보면서 제가 어떻게 답을 작성했는지 설명하는 글입니다.
그리고 출제자의 답안과 비교하는 글입니다.
따라서 제가 작성한 코드는 썩 훌륭하지 않을 수 있습니다.
출제자의 의도를 충분히 만족하지 못한 부분도 있을 수 있습니다. 

시간이 없다면 이 글의 앞부분을 건너뛰고 답안 코드를 바로 읽는 게 좋겠습니다.
마침 심심한데 다른 사람이 어떻게 문제를 풀었나 궁금하시다면, 바로 아래부터 읽어나가시면 됩니다. 

## 연습 문제
연습 문제는 책에서 다룬 예제 코드의 main 함수 테스트를 작성하는 것입니다.
예제 코드의 내용은 이렇습니다. 

1. 애플리케이션을 실행할 때 숫자를 인자로 받습니다. 
2. 숫자가 아닌 인자를 받으면 애플리케이션 사용법을 출력하고 끝냅니다.
3. 인자를 숫자로 받았다면 이름을 입력하라는 문자열이 출력됩니다.
4. 사용자가 이름을 입력하고 나면, 인자로 넘긴 수만큼 이름이 반복 출력됩니다.

책은 이 프로그램에서 만든 모든 함수의 테스트 코드를 작성하고 설명합니다.
단 한 가지, main 함수 테스트 코드만 빼고요.

## 연습 문제 요구사항
연습 문제에는 아래와 같은 요구사항이 있습니다.

- 애플리케이션을 빌드 해야한다.
- 다양한 이자를 주어 실행하고 표준 출력과 종료 코드를 확인해 테스트해야 한다.

## 나의 코드 

```go
package main

import (
    "bytes"
    "errors"
    "fmt"
    "os"
    "os/exec"
    "runtime"
    "strings"
    "testing"
)

var usage = `Usage: ./application <integer> [-h|--help]

A greeter application wich prints the name you entered <integer> number of times.
`

func buildApp(appName string) {
    if runtime.GOOS == "windows" {
        appName += ".exe"
    }

    build := exec.Command("go", "build", "-o", appName)
    if err := build.Run(); err != nil {
        fmt.Fprintf(os.Stderr, "Error building %s: %s", appName, err)
        os.Exit(1)
    }
}

func TestMain(m *testing.M) {
    fmt.Println("-> Building...")

    appName := "application"
    buildApp(appName)

    fmt.Println("-> Testing...")

    tests := []struct {
        num    string
        name   string
        output string
        err    error
    }{
        {
            output: "Invalid number of arguments\n" + usage,
            err:    errors.New("exit status 1"),
        },
        {
            num:    "a",
            output: "strconv.Atoi: parsing \"a\": invalid syntax\n" + usage,
            err:    errors.New("exit status 1"),
        },
        {
            num:    "-1",
            output: "Must specify a number greater than 0\n" + usage,
            err:    errors.New("exit status 1"),
        },
        {
            num: "5",
            output: `Your name please? Press the Enter key when done.
You didn't enter your name
`,
            err: errors.New("exit status 1"),
        },
        {
            num:  "5",
            name: "Jaehun Heo\n",
            output: `Your name please? Press the Enter key when done.
Nice to meet you Jaehun Heo
Nice to meet you Jaehun Heo
Nice to meet you Jaehun Heo
Nice to meet you Jaehun Heo
Nice to meet you Jaehun Heo
`,
        },
    }

    for _, tc := range tests {

        var cmd *exec.Cmd

        if tc.num == "" {
            cmd = exec.Command("./" + appName)
        } else {
            cmd = exec.Command("./"+appName, tc.num)
        }

        cmd.Stdin = strings.NewReader(tc.name)

        var output bytes.Buffer
        cmd.Stdout = &output

        err := cmd.Run()
        if err != nil && err.Error() != tc.err.Error() {
            fmt.Printf("Expacted error: %v, Got: %v\n", tc.err.Error(), err.Error())
            os.Exit(1)
        }
        if tc.output != output.String() {
            fmt.Printf("Expacted stdout message to be: \n%v, \nGot: \n%v\n", tc.output, output.String())
            os.Exit(1)
        }
    }

    fmt.Println("-> Getting done...")
    os.Remove(appName)

    fmt.Println("-> Start unit tests...")
    result := m.Run()

    os.Exit(result)
}
```

## TestMain()으로 테스트 환경을 갖추기
`TestMain()`을 이용해 테스트 전에 필요한 작업을 할 수 있습니다.
일반 테스트 함수와 다르게 매개변수 타입이 `*testing.M`인 점을 주의해야 합니다. 
`TestMain()`의 구조는 대략 이렇습니다.

```go
func TestMain(m *testing.M) {

    // 이곳에서 단위 테스트 실행 전에 처리할 일을 구현
    // ...

    // 단위 테스트 실행
    result := m.Run()

    // 필요하다면 후처리 과정을 이곳에 구현
    // ...
    
    os.Exit(result)
}
```

## 바이너리 파일 빌드
책에서 지시한 첫 번째 요구사항은 바이너리 파일 빌드입니다.
그래서 `m.Run()`을 실행하기 전에 빌드하는 과정을 추가합니다. 
빌드할 파일 이름을 입력받는 `buildApp()`이라는 함수를 만들었습니다.
여기서 코드를 실행하는 OS를 구분해 환경에 맞게 바이너리 파일을 빌드합니다.

``` go
func buildApp(appName string) {
    if runtime.GOOS == "windows" {
        appName += ".exe"
    }

    build := exec.Command("go", "build", "-o", appName)
    if err := build.Run(); err != nil {
        fmt.Fprintf(os.Stderr, "Error building %s: %s", appName, err)
        os.Exit(1)
    }
}
```

### OS 구분하기
Linux나 macOS에서는 확장자가 없어도 바이너리 파일이 잘 실행됩니다.
하지만 Windows에서는 `.exe` 확장자가 붙어야 바이너리 파일을 실행할 수 있습니다.
`runtime` 패키지의 `GOOS` 변숫값으로 코드를 실행하고 있는 OS를 알 수 있습니다.

### 빌드 명령 실행하기
`exec` 패키지를 이용하면 코드 안에서 외부 명령을 실행할 수 있습니다.
사용 방법은 간단합니다.
`exec`의 `Command()`함수에 실행할 명령과 인자를 넘깁니다.
그러면 `*Cmd`타입의 구조체가 만들어집니다. 
이 `*Cmd`구조체에 포함된 `Run()`함수를 호출하면 명령이 실행됩니다.

참고로 `*Cmd`타입 구조체는 1회용입니다.
`Run()`이나 `Output()`으로 명령을 실행하고 나면 이 구조체를 [재사용할 수 없습니다.](https://pkg.go.dev/os/exec#Cmd) 

## 테스트 케이스
앱을 실행할 때 입력할 인자, 이름, 예상되는 출력값, 에러 묶음을 정의합니다.
버그가 새어 나가지 않도록 빈틈없이 케이스를 만듭니다. 
저는 5가지 경우가 있을 수 있다고 생각하여 아래처럼 작성하였습니다.

``` go
func TestMain(m *testing.M) {
    
    // ...

    tests := []struct {
        num    string
        name   string
        output string
        err    error
    }{
        {
            output: "Invalid number of arguments\n" + usage,
            err:    errors.New("exit status 1"),
        },
        {
            num:    "a",
            output: "strconv.Atoi: parsing \"a\": invalid syntax\n" + usage,
            err:    errors.New("exit status 1"),
        },
        {
            num:    "-1",
            output: "Must specify a number greater than 0\n" + usage,
            err:    errors.New("exit status 1"),
        },
        {
            num: "5",
            output: `Your name please? Press the Enter key when done.
You didn't enter your name
`,
            err: errors.New("exit status 1"),
        },
        {
            num:  "5",
            name: "Jaehun Heo\n",
            output: `Your name please? Press the Enter key when done.
Nice to meet you Jaehun Heo
Nice to meet you Jaehun Heo
Nice to meet you Jaehun Heo
Nice to meet you Jaehun Heo
Nice to meet you Jaehun Heo
`,
        },
    }
    
    // ...
}
```

## 테스트 실행

바이너리 파일을 빌드할 때와 같이 `exec.Comman()`로 앱을 실행할 준비를 합니다.
인자가 있는 경우와 없는 경우가 있어 구분 지었습니다. 

준비한 테스트 케이스대로 입력하면 기대한 출력값이 나오는지 확인해야 합니다.
그러려면 실행할 명령의 표준 입력과 표준 출력을 가로채야 합니다. 
저는 이 과정에서 해결할 방법을 몰라 한참을 헤맸었습니다. 

간단하게도 `*Cmd` 구조체에 준비된 인터페이스를 이용하면 됩니다.
`Stdin` 인터페이스에 준비한 입력을 넣습니다.
그리고 `Stdout` 인터페이스에 버퍼를 연결합니다. 

이제 실행해야죠. 
`cmd.Run()`으로 명령을 실행한 뒤 출력 버퍼와 에러가 예상과 같은지 확인합니다. 
만약 테스트에 실패한다면 에러 문구를 출력하고 `os.Exit(1)`로 테스트를 끝냅니다.

```go
    for _, tc := range tests {

        var cmd *exec.Cmd

        if tc.num == "" {
            cmd = exec.Command("./" + appName)
        } else {
            cmd = exec.Command("./"+appName, tc.num)
        }

        cmd.Stdin = strings.NewReader(tc.name)

        var output bytes.Buffer
        cmd.Stdout = &output

        err := cmd.Run()
        if err != nil && err.Error() != tc.err.Error() {
            fmt.Printf("Expacted error: %v, Got: %v\n", tc.err.Error(), err.Error())
            os.Exit(1)
        }
        if tc.output != output.String() {
            fmt.Printf("Expacted stdout message to be: \n%v, \nGot: \n%v\n", tc.output, output.String())
            os.Exit(1)
        }
    }

```

## 테스트 뒷 마무리
여기서는 테스트한 바이너리 파일을 지우는 일을 합니다. 
굳이 지우지 않아도 됩니다. 
하지만 저는 테스트는 테스트일 뿐이라는 생각으로 지우기로 했습니다. 

한편으로는 바이너리 파일을 그대로 두는 것이 더 낫지 않을까도 생각합니다.
테스트를 거친 결과물임을 보장하는 것이니까요.
테스트에 실패했을 경우에만 지우는 것도 방법이겠네요.

그다음은 단위 테스트를 할 차례입니다. 
`m.Run()`으로 실행할 수 있습니다. 
`m`이라는 변수가 갑자기 어디서 튀어나왔나 싶지요. 
이 글 초반에 설명한 `TestMain()`함수의 매개변수입니다. 

`m.Run()`은 모든 단위 테스트를 통과하면 0을, 실패하면 1을 반환합니다.
실행 결괏값을 `os.Exit()`에 넘겨주고 모든 테스트를 끝마칩니다. 

```go
    fmt.Println("-> Getting done...")
    os.Remove(appName)

    fmt.Println("-> Start unit tests...")
    result := m.Run()

    os.Exit(result)
```

## 실행 결과
작성한 테스트 코드를 `go test -v`명령으로 실행해 봅니다. 
결과는 아래와 같습니다.

```shell
$ go test -v
-> Building...
-> Testing...
-> Getting done...
-> Start unit tests...
=== RUN   TestParseArgs
--- PASS: TestParseArgs (0.00s)
=== RUN   TestRunCmd
--- PASS: TestRunCmd (0.00s)
=== RUN   TestValidateArgs
--- PASS: TestValidateArgs (0.00s)
PASS
ok      practical-go/chap01/manual-parse        1.853s
```

바이너리 파일을 빌드하고 애플리케이션 실행 테스트도 통과했습니다.
연습 문제를 풀기 전에 작성해 둔 단위 테스트들도 잘 실행되었고요.
모든 요구사항을 만족한 것 같습니다.

여기까지 왔을 때 기분이 꽤 뿌듯했었지요.
그런데 답안 코드를 살펴보고 나니 역시나 제 것은 아주 허술하구나 싶었습니다. 
이다음부터는 답안 코드를 분석해 보겠습니다.

## 답안 코드
제가 작성한 것과 답안 코드의 차이는 두 가지라고 생각합니다.

- 앱 실행 테스트를 위한 전용 테스트 함수(`TestXXXX(t *testing.T)`)를 만듦
- 테스트에 타임아웃 컨텍스트를 추가함 

### 앱 실행 테스트도 단위 테스트 중 하나
저는 `TestMain()` 함수 안에 앱 실행 테스트를 구현했습니다.
그리고 단위 테스트를 따로 실행하도록 했지요.
그래서 Go 언어의 단위 테스트 모음에 앱 실행 테스트는 포함되지 않았습니다.

한편, 답안에는 앱 실행 테스트도 하나의 단위 테스트로 구현되어 있었습니다. 

혹시나 그렇게 하면 main함수도 coverage에 포함되나 궁금했습니다. 
그래서 실행해본 결과는 '포함되지 않는다'입니다. 

```shell
$ go test -cover
PASS
coverage: 71.2% of statements
ok      practical-go/chap01/manual-parse        0.262s
```

그럼에도 테스트와 준비 과정을 명확히 나눌 수 있는 것은 장점입니다.
훨씬 깔끔해 보입니다.
테스트 도구가 출력하는 PASS/FAIL 결과도 알아서 뽑아주는 것도 좋고요.

### 타임아웃 컨텍스트를 더해서 우아하게 만들기
답안 코드는 빌드 과정과 앱 실행 테스트에 타임아웃 컨텍스트를 추가했습니다. 
타임아웃 컨텍스트는 교착 상태에 빠지거나 실행이 지연될 때를 위한 장치입니다.
아무리 늦어도 n 시간 안에는 테스트 결과가 나오도록 하는 우아한 방식입니다. 

사실 컨텍스트 기능은 책에서 이 연습 문제 이후에 설명된 내용이라 사용해 볼 생각을 못했습니다. 
저자가 반칙을 썼네요! 
그래도 유용해 보이는 기능을 답안에 사용해 주어서 덕분에 덤으로 배운 느낌입니다.

```go
package main

import (
    "bytes"
    "context"
    "fmt"
    "log"
    "os"
    "os/exec"
    "path"
    "runtime"
    "strings"
    "testing"
    "time"
)

var binaryName string

func TestMain(m *testing.M) {
    if runtime.GOOS == "windows" {
        binaryName = "manual-parse-app.exe"
    } else {
        binaryName = "manual-parse-app"
    }

    ctx, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
    defer cancel()
    cmd := exec.CommandContext(ctx, "go", "build", "-o", binaryName)
    err := cmd.Run()
    if err != nil {
        os.Exit(1)
    }
    defer func() {
        err = os.Remove(binaryName)
        if err != nil {
            log.Fatalf("Error removing built binary: %v", err)
        }
    }()
    m.Run()
}

func TestApplication(t *testing.T) {
    ctx, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
    defer cancel()
    curDir, err := os.Getwd()
    if err != nil {
        t.Fatal(err)
    }
    binaryPath := path.Join(curDir, binaryName)
    t.Log(binaryPath)

    tests := []struct {
        args                []string
        input               string
        expectedOutputLines []string
        expectedExitCode    int
    }{
        {
            args:             []string{},
            expectedExitCode: 1,
            expectedOutputLines: []string{
                "Invalid number of arguments",
                fmt.Sprintf("Usage: %s <integer> [-h|-help]", binaryPath),
                "",
                "A greeter application which prints the name you entered <integer> number of times.",
                "",
            },
        },
        {
            args:             []string{"-h"},
            expectedExitCode: 1,
            expectedOutputLines: []string{
                "Must specify a number greater than 0",
                fmt.Sprintf("Usage: %s <integer> [-h|-help]", binaryPath),
                "",
                "A greeter application which prints the name you entered <integer> number of times.",
                "",
            },
        },
        {
            args:                []string{"a"},
            expectedExitCode:    1,
            expectedOutputLines: []string{},
        },
        {
            args:             []string{"2"},
            input:            "jane doe",
            expectedExitCode: 0,
            expectedOutputLines: []string{
                "Your name please? Press the Enter key when done.",
                "Nice to meet you jane doe",
            },
        },
    }

    byteBuf := new(bytes.Buffer)
    for _, tc := range tests {
        t.Logf("Executing:%v %v\n", binaryPath, tc.args)
        cmd := exec.CommandContext(ctx, binaryPath, tc.args...)
        cmd.Stdout = byteBuf
        if len(tc.input) != 0 {
            cmd.Stdin = strings.NewReader(tc.input)
        }
        err := cmd.Run()

        if err != nil && tc.expectedExitCode == 0 {
            t.Fatalf("Expected application to exit without an error. Got: %v", err)
        }

        if cmd.ProcessState.ExitCode() != tc.expectedExitCode {
            t.Log(byteBuf.String())
            t.Fatalf("Expected application to have exit code: %v. Got: %v", tc.expectedExitCode, cmd.ProcessState.ExitCode())

        }

        output := byteBuf.String()
        lines := strings.Split(output, "\n")
        for num := range tc.expectedOutputLines {
            if lines[num] != tc.expectedOutputLines[num] {
                t.Fatalf("Expected output line to be:%v, Got:%v", tc.expectedOutputLines[num], lines[num])
            }
        }
        byteBuf.Reset()
    }
}
```


## 실행 결과 (답안 코드)

실행해 보니 예상대로 앱 실행 테스트도 Go 언어의 단위 테스트 중의 하나로 처리되는 것으로 보입니다. 
앱 실행 테스트의 경우 테스트 케이스를 실행할 때마다 로그를 남겨두었네요.
어떤 인자와 함께 실행했을 때 에러가 났는지 금방 파악할 수 있는 좋은 습관인 것 같습니다. 

```shell
$ go test -v
=== RUN   TestApplication
    main_test.go:50: /home/jaehun/Dev/go-study/practical-go/chap01/manual-parse/manual-parse-app
    main_test.go:98: Executing:/home/jaehun/Dev/go-study/practical-go/chap01/manual-parse/manual-parse-app []
    main_test.go:98: Executing:/home/jaehun/Dev/go-study/practical-go/chap01/manual-parse/manual-parse-app [-h]
    main_test.go:98: Executing:/home/jaehun/Dev/go-study/practical-go/chap01/manual-parse/manual-parse-app [a]
    main_test.go:98: Executing:/home/jaehun/Dev/go-study/practical-go/chap01/manual-parse/manual-parse-app [2]
--- PASS: TestApplication (0.01s)
=== RUN   TestParseArgs
--- PASS: TestParseArgs (0.00s)
=== RUN   TestRunCmd
--- PASS: TestRunCmd (0.00s)
=== RUN   TestValidateArgs
--- PASS: TestValidateArgs (0.00s)
PASS
ok      practical-go/chap01/manual-parse        0.239s
```
