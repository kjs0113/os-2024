# 운영체제 개요

프로그램이 실행될 때, 그것은 단순히 명령어를 실행합니다. 프로세서는 초당 수백만에서 수십억 번 명령어를 가져와 해석하고 실행합니다. 이 과정은 프로그램이 완전히 종료될 때까지 계속됩니다. 이것은 Von Neumann 컴퓨팅 모델의 기본 개념을 설명한 것입니다. 간단해 보이지만, 실제로는 프로그램을 더 쉽게 실행할 수 있도록 돕는 다양한 과정이 포함됩니다.

운영체제는 사용자가 프로그램을 더 쉽게 실행할 수 있도록 도우며, 동시에 여러 프로그램을 실행할 수 있게 하고, 메모리 공유, 장치와의 상호 작용 등을 가능하게 합니다. 운영체제는 가상화라는 기술을 사용해 물리적 자원을 가상의 형태로 만들어 제공합니다. 사용자는 운영체제가 제공하는 API를 통해 필요한 기능을 요청할 수 있습니다. 운영체제는 이러한 요청을 처리하기 위해 다양한 시스템 호출을 제공합니다.

가상화 덕분에 여러 프로그램이 CPU를 공유하고 동시에 실행될 수 있으며, 각각의 프로그램은 자신만의 명령어와 데이터에 접근할 수 있습니다. 운영체제는 자원 관리자 역할도 하여 CPU, 메모리, 디스크 등의 자원을 효율적이고 공정하게 관리합니다. 운영체제의 역할과 중요성을 이해하기 위해 몇 가지 예를 들어보는 것도 좋습니다.

## CPU 가상화

핵심 질문은 운영체제가 어떻게 자원을 가상화하는지에 관한 것입니다. 우리의 주요 관심사는 가상화의 이유보다는 그 방법에 있습니다. 운영체제가 사용하기 쉬운 시스템을 만들기 위해 가상화를 구현하는 다양한 기술과 정책, 그리고 필요한 하드웨어 지원에 대해 알아보겠습니다. 각 주제를 다룰 때 중요한 질문들을 만나게 되며, 이러한 질문들에 대한 해결책이나 기본 개념을 소개합니다.

간단한 예제로, 반복해서 문자열을 출력하는 프로그램을 살펴봅시다. 이 프로그램은 `Spin()` 함수를 호출하여 1초 동안 실행되고, 사용자가 입력한 문자열을 출력한 다음 무한히 반복합니다. 프로그램을 `cpu.c`라는 파일에 저장하고 컴파일 후 실행하면, 지정된 문자열을 1초마다 반복해서 출력합니다. 프로그램은 계속 실행되며, 사용자가 `Control-c`를 눌러 종료할 수 있습니다.

간단한 코드 예제는 다음과 같습니다:

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <assert.h>
#include “common.h”

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, “사용법: cpu <문자열>\n”);
        exit(1);
    }
    char *str = argv[1];
    while (1) {
        Spin(1);
        printf(“%s\n”, str);
    }
    return 0;
}
```

이 코드는 사용자가 입력한 문자열을 1초마다 계속 출력합니다. 결과는 복잡하지 않지만, 프로그램이 어떻게 끊임없이 실행되는지 보여줍니다. 프로그램을 종료하려면 `Control-c`를 누르면 됩니다.

이번에는 여러 개의 프로그램을 동시에 실행시켜 보겠습니다. 다음 명령어를 사용하면 프로그램 네 개가 동시에 실행되며, 각각 다른 문자(A, B, C, D)를 출력합니다. 비록 우리가 하나의 프로세서만 가지고 있음에도 불구하고, 네 프로그램이 모두 동시에 실행되는 것처럼 보입니다. 이는 운영체제가 하드웨어의 도움으로 단 하나의 CPU를 여러 개의 가상 CPU가 있는 것처럼 보이게 만들기 때문입니다. 이를 CPU 가상화라고 하며, 우리가 여기서 다루는 주제입니다.

프로그램을 실행하고, 중지하고, 어떤 프로그램을 다음에 실행할지 결정하는 것은 운영체제가 담당합니다. 이를 위해 운영체제는 사용자와 소통할 수 있는 API를 제공합니다. 우리는 이 수업을 통해 이러한 API에 대해 논의할 것입니다.

여러 프로그램을 동시에 실행하는 것은 새로운 문제들을 가져옵니다. 예를 들어, 어떤 프로그램을 우선 실행할지 결정하는 것은 운영체제의 정책에 따라 달라집니다. 운영체제는 자원 관리자로서, 동시에 실행되는 프로그램들 사이에서 자원을 공정하게 배분하는 역할을 합니다.

아래는 메모리 접근과 관련된 프로그램의 예입니다. 이 프로그램은 메모리에 정수를 저장하고, 1초마다 그 값을 1씩 증가시킨 후 출력합니다. 각 프로그램은 고유한 메모리 주소를 가지며, 실행될 때마다 해당 주소에 있는 값이 어떻게 변하는지 보여줍니다.

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include “common.h”

int main(int argc, char *argv[]) {
    int *p = malloc(sizeof(int)); // 정수형 포인터 p에 메모리 할당
    assert(p != NULL); // p가 NULL이 아닌지 확인
    printf(“(%d) p의 메모리 주소: %08x\n”, getpid(), (unsigned) p); // 프로세스 ID와 p의 메모리 주소 출력
    *p = 0; // p가 가리키는 값을 0으로 초기화
    while (1) {
        Spin(1); // 1초 대기
        *p = *p + 1; // p가 가리키는 값을 1 증가
        printf(“(%d) p: %d\n”, getpid(), *p); // 프로세스 ID와 p가 가리키는 값 출력
    }
    return 0;
}
```

이 코드는 프로그램이 메모리를 어떻게 사용하는지와 프로세스마다 고유한 메모리 주소 공간을 가지고 있음을 보여줍니다.

## 메모리 가상화

메모리 가상화를 통해 운영체제는 각 프로그램이 자신만의 메모리 공간을 가지고 있는 것처럼 만듭니다. 실제로 모든 프로그램은 같은 물리 메모리를 공유하지만, 운영체제가 각 프로그램에게 독립된 가상 주소 공간을 제공하여 다른 프로그램과 메모리를 공유하지 않는 것처럼 보이게 합니다.

예를 들어, `malloc()` 함수를 사용하여 메모리를 할당하는 프로그램을 실행하면, 할당된 메모리의 주소와 그 주소에 저장된 값을 출력합니다. 프로그램이 계속 실행되면서 값을 갱신하고, 각 출력마다 프로그램의 고유 식별자(PID)와 함께 메모리 주소와 값을 보여줍니다.

만약 이 프로그램을 여러 번 동시에 실행하면, 모든 프로그램이 같은 메모리 주소에 값을 할당받는 것처럼 보입니다. 그러나 실제로는 각 프로그램이 독립된 가상 메모리 공간을 사용하고 있기 때문에, 각 프로그램은 자신만의 메모리 공간에서 작업을 수행합니다. 이로 인해 한 프로그램의 작업이 다른 프로그램에 영향을 주지 않습니다.

메모리 가상화는 운영체제가 각 프로그램에게 독립된 가상 주소 공간을 제공하여, 실제 물리 메모리가 아니라 가상 메모리를 사용하는 것처럼 만듭니다. 이는 프로그램들이 서로 메모리를 방해하지 않고 안전하게 실행될 수 있게 해주는 중요한 기능입니다.

## 병행성

이 수업에서 다루는 또 다른 중요한 주제는 병행성입니다. 병행성은 프로그램이 여러 작업을 동시에 수행하려고 할 때 발생하는 문제들을 의미합니다. 이러한 문제는 운영체제 내부에서뿐만 아니라, 멀티 쓰레드 프로그램에서도 발생합니다.

예를 들어, 메인 프로그램이 `Pthread_create()` 함수를 사용하여 두 개의 쓰레드를 만들고, 각 쓰레드가 `worker()` 함수를 실행하여 카운터 값을 증가시키는 상황을 생각해 볼 수 있습니다. 만약 각 쓰레드가 카운터를 1000번 증가시키도록 설정된다면, 최종 카운터 값은 2000이 될 것으로 예상할 수 있습니다.

```bash
gcc -o thread thread.c -Wall -pthread
./thread 1000
```

실행 결과, 초기 카운터 값은 0이고, 최종 값은 2000이 됩니다. 이는 각 쓰레드가 카운터를 1000번씩 증가시키기 때문입니다. 그러나 실제로 프로그램을 실행할 때마다 결과가 항상 같은 것은 아니며, 병행성 문제로 인해 예상치 못한 결과가 나올 수 있습니다. 이 수업에서는 이러한 병행성 문제와 그 해결 방법에 대해 자세히 다룰 예정입니다.

병행성은 프로그램이 동시에 여러 작업을 처리할 때 발생하는 문제들을 다룹니다. 이는 운영체제뿐만 아니라 멀티 쓰레드 프로그램에서도 마찬가지로 발생합니다. 예를 들어, 두 쓰레드가 같은 카운터 값을 동시에 증가시키려 할 때, 예상과 다른 결과를 얻을 수 있습니다.

아래 코드 예제에서는 두 쓰레드가 `loops` 변수를 이용해 카운터 값을 반복적으로 증가시키는 작업을 수행합니다. `loops` 값이 큰 경우, 예상했던 최종 카운터 값이 아닌 다른 결과를 얻게 되는 경우가 있습니다. 이는 각 쓰레드가 카운터 값을 증가시키는 작업이 여러 단계로 이루어져 있고, 이 단계들이 동시에 다른 쓰레드와 겹치면서 올바르지 않은 결과를 초래하기 때문입니다.

```c
#include <stdio.h>
#include <stdlib.h>
#include "common.h"
volatile int counter = 0;
int loops;

void *worker(void *arg) {
    for (int i = 0; i < loops; i++) {
        counter++;
    }
    return NULL;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "사용법: threads <값>\n");
        exit(1);
    }
    loops = atoi(argv[1]);
    pthread_t p1, p2;
    printf("초기 값 : %d\n", counter);

    Pthread_create(&p1, NULL, worker, NULL);
    Pthread_create(&p2, NULL, worker, NULL);
    Pthread_join(p1, NULL);
    Pthread_join(p2, NULL);
    printf("최종 값 : %d\n", counter);
    return 0;
}
```

예를 들어, `loops` 값을 100,000으로 설정하고 프로그램을 실행시키면, 카운터의 최종 값이 200,000이 아니라 예상치 못한 다른 값이 나옵니다. 이는 카운터 값을 증가시키는 명령어가 원자적으로 실행되지 않기 때문에 발생하는 문제입니다.

병행성 문제를 해결하기 위해서는 올바르게 동작하는 병행 프로그램을 어떻게 작성해야 하는지, 운영체제와 하드웨어가 제공해야 하는 기본 기법과 기능이 무엇인지를 이해해야 합니다. 이 수업의 후반부에서는 이러한 병행성 문제와 그 해결 방법에 대해 자세히 다룰 예정입니다.

## 영속성

이 수업에서 다루는 세 번째 중요한 주제는 영속성입니다. DRAM과 같은 저장 장치는 전원이 꺼지면 데이터를 잃어버리기 때문에, 데이터를 영구적으로 보관할 방법이 필요합니다. 이를 위해 하드 드라이브나 SSD와 같은 하드웨어와, 이들을 관리하는 파일 시스템이라는 소프트웨어가 사용됩니다.

파일 시스템은 사용자가 만든 파일을 디스크에 저장하고, 이를 효율적으로 관리하는 역할을 합니다. 다만, 운영체제는 프로그램마다 별도의 가상 디스크를 만들지 않고, 대신 파일 정보를 여러 사용자와 프로그램이 공유할 수 있도록 합니다. 예를 들어, C 프로그램을 만들 때, 에디터를 사용해 코드를 작성하고 저장한 후, 컴파일러를 통해 실행 파일을 만들어 실행하는 과정에서 파일이 여러 단계에 걸쳐 사용됩니다.

중요한 질문은 데이터를 어떻게 영구적으로 저장할 수 있는가입니다. 파일 시스템은 이러한 작업을 수행하는 운영체제의 일부로, 데이터를 안전하게, 효율적으로 저장하고 접근할 수 있는 다양한 기법과 정책이 필요합니다. 또한, 하드웨어나 소프트웨어에 문제가 생겨도 데이터를 안전하게 보호할 방법에 대해서도 고려해야 합니다.

이 코드 예제는 "/tmp/file"이라는 파일을 만들고, 그 안에 "hello world\n"이라는 문자열을 쓰는 프로그램입니다. 프로그램은 세 가지 주요 시스템 호출을 사용합니다: `open()`으로 파일을 생성하고 열고, `write()`로 파일에 데이터를 쓰고, 마지막으로 `close()`로 파일을 닫습니다. 이 과정은 파일 시스템을 통해 운영되며, 필요에 따라 에러 메시지를 반환할 수 있습니다.

```c
#include <stdio.h>
#include <unistd.h>
#include <assert.h>
#include <fcntl.h>
#include <sys/types.h>

int main(int argc, char *argv[]) {
    int fd = open("/tmp/file", O_WRONLY | O_CREAT | O_TRUNC, S_IRWXU);
    assert(fd > -1); // 파일 열기를 확인
    int rc = write(fd, "hello world\n", 13); // "hello world\n" 문자열 쓰기
    assert(rc == 13); // 쓰기 작업을 확인
    close(fd); // 파일 닫기
    return 0;
}
```

운영체제가 디스크에 데이터를 쓰기 위해 수행하는 작업은 상당히 복잡합니다. 파일 시스템은 새 데이터를 디스크 어디에 저장할지 결정해야 하고, 다양한 자료 구조를 통해 데이터 상태를 추적해야 합니다. 이 과정에서 저장 장치를 읽거나 갱신해야 할 수도 있습니다. 장치 드라이버를 다루는 것은 복잡한 작업이며, 저수준의 장치 인터페이스와 시맨틱에 대한 깊은 이해가 필요합니다. 운영체제는 시스템 호출이라는 표준화된 방법을 통해 이러한 장치들에 접근할 수 있게 해줍니다.

성능 향상을 위해, 많은 파일 시스템은 쓰기 요청을 지연시키고 한 번에 처리합니다. 시스템이 갑자기 고장 날 경우를 대비해 저널링이나 쓰기-시-복사 같은 복잡한 쓰기 기법을 사용하기도 합니다. 이런 기법들은 고장이 발생해도 시스템을 정상 상태로 복구할 수 있도록 쓰기 순서를 조정합니다. 효율적인 디스크 작업을 위해 다양한 종류의 자료 구조를 사용합니다. 이런 복잡한 주제들은 이 수업에서 더 자세히 다룰 예정입니다, 여기서는 디스크, RAID, 파일 시스템 등에 대한 상세한 내용을 포함하여 장치와 입출력 전반에 대해 설명할 것입니다.

## 설계 목표

운영체제의 역할을 이해하기 시작했다면, 이제 그것이 어떤 목표를 가지고 설계되어야 하는지 알아보겠습니다. 운영체제는 컴퓨터의 CPU, 메모리, 디스크와 같은 물리적 자원을 가상화하고, 병행성 문제를 다루며, 데이터를 영구적으로 저장합니다. 이러한 시스템을 만들기 위해서는 추상화, 성능, 보호, 신뢰성 등 여러 목표를 염두에 둬야 합니다.

첫 번째로, 시스템을 사용하기 쉽게 만드는 데 필요한 추상화를 정의하는 것이 중요합니다. 추상화는 복잡한 프로그램을 이해하기 쉬운 작은 부분으로 나누는 데 도움을 줍니다. 또한, 시스템의 성능을 최적화하면서 오버헤드를 최소화하는 것도 중요한 목표입니다.

응용 프로그램 간의 보호와 운영체제 자체의 보호도 필수적입니다. 멀티프로그래밍 환경에서는 한 프로그램의 오류가 다른 프로그램이나 운영체제 전체에 영향을 주지 않도록 해야 합니다. 또한, 운영체제는 계속해서 안정적으로 실행되어야 하며, 복잡해질수록 신뢰성을 유지하는 것이 더 어려워집니다.

에너지 효율성, 보안, 이동성 등 다른 중요한 목표들도 있습니다. 특히 현재와 같은 네트워크 연결 환경에서는 보안이 더욱 중요해졌고, 모바일 장치 사용이 증가함에 따라 운영체제의 이동성도 중요해졌습니다. 이 수업을 통해 다양한 운영체제의 주제들을 탐구하면서 이러한 목표들이 어떻게 실현되는지 배우게 될 것입니다.

## 역사

운영체제의 발전 과정을 간단히 살펴보겠습니다.

### 초창기 운영체제 : 단순 라이브러리

처음에 운영체제는 복잡한 기능을 많이 하지 않았습니다. 주로 자주 사용되는 함수들을 모아둔 라이브러리 형태였죠. 이런 구성은 개발자들이 반복적으로 저수준의 입출력 처리 코드를 작성하지 않도록 도와줬습니다.

과거 메인프레임 시스템을 사용할 때는, 컴퓨터를 조작하는 사람이 프로그램을 한 번에 하나씩 실행시켰습니다. 오늘날 운영체제가 처리하는 많은 작업, 예를 들어 작업 순서를 정하는 것과 같은 업무는 당시에는 컴퓨터 관리자가 담당했습니다. 그래서 프로그래머들은 자신의 작업이 우선적으로 실행되도록 컴퓨터 관리자에게 잘 보이려고 노력했습니다.

작업이 준비되면, 컴퓨터 관리자는 이를 일괄적으로 처리하는 일괄 처리 방식을 사용했습니다. 당시에는 컴퓨터가 매우 비싸서 사용자가 직접 컴퓨터 앞에서 작업하는 대화형 방식으로 사용되지 않았습니다. 컴퓨터 앞에서 할 일 없이 기다리게 만들면 많은 비용이 낭비되기 때문이었습니다.

### 라이브러리를 넘어서 : 보호

초기 운영체제가 단순한 라이브러리에서 더 발전하여 컴퓨터를 더 효과적으로 관리하는 중심적 역할을 하게 된 주요 이유 중 하나는 운영체제 코드가 특별하게 다뤄져야 한다는 인식 때문입니다. 운영체제 코드는 장치를 제어하기 때문에, 일반 응용 프로그램 코드와 다르게 취급됩니다. 만약 모든 프로그램이 원하는 파일을 자유롭게 읽을 수 있다면, 사생활 보호는 불가능해집니다. 따라서, 파일 시스템을 단순한 라이브러리로만 구현하는 것은 부족하며, 보다 체계적인 접근 방식이 필요합니다.

Atlas 컴퓨팅 시스템에 의해 소개된 시스템 콜이라는 개념은 운영체제를 단순한 라이브러리가 아니라 특별한 하드웨어 명령어와 결합하여 운영체제로의 전환을 가능하게 합니다. 시스템 콜을 사용하면, 제어가 운영체제로 넘어갈 때 하드웨어의 특권 수준을 상향 조정할 수 있습니다.

응용 프로그램은 보통 사용자 모드에서 실행되어 하드웨어적으로 제한된 작업만 수행할 수 있습니다. 예를 들어, 사용자 모드에서는 디스크 입출력이나 물리 메모리 페이지 접근 같은 작업을 할 수 없습니다. 시스템 콜은 트랩이라는 특별한 하드웨어 명령어를 통해 호출되며, 이 과정에서 특권 수준이 커널 모드로 격상됩니다. 커널 모드에서는 운영체제가 시스템 하드웨어에 자유롭게 접근하여 다양한 서비스를 제공할 수 있습니다. 운영체제가 작업을 마치면, return-from-trap 명령어를 사용해 제어권을 사용자 모드로 돌려주고, 응용 프로그램이 중단됐던 지점에서 다시 실행을 계속합니다.

### 멀티프로그래밍 시대

컴퓨터의 역사에서 미니컴퓨터 시대는 운영체제 발전에 중요한 시기였습니다. PDP 컴퓨터 등장으로 인해 컴퓨터 가격이 크게 떨어지며, 이전에는 하나의 메인프레임을 사용하던 큰 기관들이 여러 소그룹별로 컴퓨터를 갖게 되었습니다. 이로 인해 컴퓨터 분야에 더 많은 인재가 모이고, 컴퓨터가 더 다양하고 흥미로운 작업을 할 수 있게 되었습니다.

이 시기에는 멀티프로그래밍이라는 기법이 널리 사용되기 시작했습니다. 멀티프로그래밍은 여러 프로그램을 동시에 메모리에 올려두고 CPU가 빠르게 작업을 전환하며 실행함으로써, CPU 사용률을 향상시키는 방법입니다. 이는 특히 입출력 장치가 느릴 때 유용하며, 입출력 작업이 진행되는 동안 다른 작업으로 전환해 CPU 시간을 절약할 수 있습니다.

멀티프로그래밍의 도입과 함께 메모리 보호, 인터럽트 처리, 병행성 문제 등 여러 혁신이 이루어졌습니다. 한 프로그램이 다른 프로그램의 메모리에 접근하는 것을 방지하고, 인터럽트 발생 시에도 운영체제가 제대로 동작하도록 하는 것은 중요한 과제였습니다.

이 시기의 또 다른 중요한 사건은 Unix 운영체제의 등장입니다. Unix는 Ken Thompson과 Dennis Ritchie에 의해 Bell 연구소에서 개발되었으며, Multics, TENEX, Berkeley Time-Sharing System 같은 다른 시스템에서 좋은 아이디어를 많이 가져와 사용하기 쉽게 변형시켰습니다. Unix 소스 코드가 전세계에 배포되면서 많은 사람들이 이에 기여하고 새로운 기능을 추가했습니다. 이러한 과정은 Unix의 발전과 확산에 크게 기여했습니다.

```{admonition} Unix의 중요성
Unix의 역사에서 그 중요성을 강조하는 것은 결코 과장이 아닙니다. Unix는 초기에 MIT의 Multics 프로젝트에서 영감을 받아 Bell Labs에서 시작되었습니다. Unix의 핵심은 간단하고 효율적인 프로그램들이 서로 연결되어 사용될 수 있다는 점입니다. 예를 들어, 텍스트 파일에서 특정 단어를 찾아 그 단어가 포함된 행의 수를 세고 싶다면, grep과 wc(word count) 프로그램을 파이프로 연결해 간단히 해결할 수 있습니다.

Unix는 프로그래머와 개발자에게 매우 친숙한 환경을 제공했고, 새로운 C 언어로 작성된 컴파일러를 포함하여 프로그램 작성과 공유를 용이하게 했습니다. 소스 코드를 누구나 접근할 수 있게 하여 공개 소스 소프트웨어의 초기 형태를 이루었습니다.

Unix 시스템의 가독성이 좋은 소스 코드는 많은 사람들이 커널에 기능을 추가하고 개선할 수 있도록 했습니다. 예를 들어, Berkeley에서 Bill Joy가 주도한 그룹은 BSD(Berkeley Systems Distribution)라는 훌륭한 버전을 만들어냈습니다. 이 버전은 개선된 가상 메모리, 파일 시스템, 네트워킹 서브시스템을 포함하고 있었습니다.

하지만 회사들이 소유권을 주장하고 이익 추구를 시작하면서 Unix의 발전은 다소 둔화되었습니다. 여러 회사가 자신만의 Unix 버전을 소유하게 되었고, 이로 인해 법적 분쟁이 발생하기도 했습니다. Windows의 등장과 PC 시장 장악으로 Unix의 미래에 대한 의문이 제기되었지만, Unix의 영향력은 여전히 크며, 현대의 많은 시스템과 기술에 기반이 되고 있습니다.
```

### 현대

미니컴퓨터 시대 이후, 가격이 저렴하고 빠른 속도를 지닌 개인용 컴퓨터(PC)가 대중화되었습니다. Apple II나 IBM PC와 같은 초기 PC들은 미니컴퓨터 대비 훨씬 저렴한 가격으로 개인이 하나씩 소유할 수 있게 되면서 컴퓨팅 분야의 주류로 자리 잡았습니다.

하지만, 초기 PC 시대의 운영체제는 기술적으로 몇 가지 후퇴를 겪었습니다. 예를 들어, DOS와 같은 초기 운영체제는 메모리 보호의 중요성을 간과했고, 악의적인 소프트웨어나 오류가 있는 소프트웨어에 의해 메모리가 손상될 수 있는 위험이 있었습니다. 1세대 Mac OS는 프로그램이 무한 루프에 빠질 경우 시스템 전체를 멈출 수 있는 협력 스케줄링을 사용했습니다.

운좋게도 이런 암흑기를 지나고, 미니컴퓨터 시대의 운영체제 기술이 다시 데스크톱 운영체제에 적용되기 시작했습니다. Mac OS X는 Unix 기반으로 만들어져 Unix의 강력한 기능을 모두 제공합니다. Windows도 많은 중요한 컴퓨팅 아이디어를 받아들여 특히 Windows NT부터 큰 발전을 이루었습니다. 오늘날의 스마트폰도 Linux와 같은 고급 운영체제를 실행하며, 이는 1980년대 PC 용 운영체제보다는 1970년대 미니컴퓨터 운영체제와 더 비슷합니다.

이처럼, 운영체제의 발전 과정을 통해 좋은 아이디어가 계속해서 발전하고 새로운 시스템에 적용되는 것을 볼 수 있습니다. 이러한 발전은 운영체제를 더욱 기능적이고 사용자 친화적으로 만들어가고 있습니다.

```{admonition} 그리고 LINUX가 나왔다
Unix의 계보에 중요한 이정표는 핀란드의 젊은 해커, Linus Torvalds가 자신의 Unix 버전을 개발하기로 결심한 것입니다. 그는 Unix의 기본 원칙을 따르면서도, 법적 문제를 피하기 위해 기존의 코드를 사용하지 않았습니다. 그의 요청에 전 세계의 개발자들이 응답하면서 Linux가 탄생했고, 현대의 오픈소스 소프트웨어 운동의 시작점이 되었습니다.

인터넷 시대가 도래하면서 Google, Amazon, Facebook 등 많은 회사들이 무료이면서도 수정이 용이한 Linux를 선택했습니다. 이러한 시스템이 없었다면 이들 회사의 성공은 상상하기 어려웠을 것입니다. 스마트폰(Android)이 주요 사용 플랫폼이 되면서 Linux의 사용 범위는 더욱 확장되었습니다. Steve Jobs는 Unix 기반의 NeXTStep 운영 환경을 Apple로 가져와 데스크톱에서도 Unix를 대중화했습니다. 많은 Apple 사용자들이 이 사실을 모를 수도 있습니다. 이렇게 Unix는 현재에도 여전히 중요한 역할을 하고 있습니다.

이처럼 Unix는 그 역사를 통해 컴퓨터 운영체제 발전에 큰 발자취를 남겼습니다. Linux의 탄생과 오픈소스 운동의 시작은 Unix의 철학이 얼마나 멀리와 깊게 영향을 미쳤는지 보여줍니다. 이 모든 변화와 발전 덕분에 우리는 오늘날 더욱 강력하고 다양한 컴퓨터 시스템을 사용할 수 있게 되었습니다.
```