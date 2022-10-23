# System_Programming_Outline
시스템 프로그래밍 정리( 어셈블리, 컴파일러)

#### 컴퓨터의 기본 구성 요소

1. 컴퓨터를 구성하는 기본적인 요소는 CPU와 주기억장치(메모리)이다.
2. CPU와 메모리는 3개의 버스로 연결되어 있다.

주소 버스 : 메모리에 접근하기 전 접근하고자 하는 주소를 전달   
제어 버스 : 메모리에 읽기 쓰기 등 기능 및 통제 신호 전달   
데이터 버스 : 읽기 때는 저장값이 버스로 실려 전달 쓰기 때는 버스로 전달되는 값이 메모리에 저장

#### 레지스터

메모리와 같은 대량 저장 장치외에 소량의 저장 장치를 레지스터라고 한다.   
PC, MAR, MBR, IR, PSW, AC, IX 등이 레지스터이다.

#### CPU 제어 회로

CPU 제어 회로는 발진기에서 공급되는 클록에 따라 단계적 처리가 진행되도록 설계되어 있다.   
그러므로 발진기의 속도가 곧 CPU의 처리 속도를 결정한다.   
   
1KHz : 1초에 1024번 진동   
1MHz : 1초에 1024*1024번(100만번) 진동   
1GHz : 1초에 1024*1024*1024(10억번) 진동

#### 컴퓨터가 연산(계산)을 진행하는 과정

1. 메모리에서 기계 명령어를 가져온다. 이 때 가져올 기계 명령어의 위치는 주소버스를 통해 메모리에 전달
2. 기계 명령어에 포함된 피연산자 주소를 참조하여 다시 메모리에 접근한다.   
만약 읽어야 할 명령어라면 CPU내 레지스터로 읽어들이고 써야 할 명령어라면 레지스터에 보관중인 값을 메모리에 쓴다.
3. 단순히 읽기만 하지않고 읽은 후 연산을 의미한다면 읽어들인 값에 대해 주어진 연산을 처리한다.

#### 연산 레지스터 AC

ADD나 SUB 명령어에서와 연산이 이루어질 수 있는 CPU 내 레지스터를 누산기라고 한다.   
레지스터 이름을 A 혹은 AC라고 부른다.
```assembly
x = y // load 명령어로 메모리의 변수 y 값을 CPU 레지스터에 읽는다. store 명령어로 cpu 레지스터 값을 메모리 변수 x에 저장한다.
z = x + y // load 명령어로 메모리의 변수 y값을 CPU 누산기에 읽는다. add 명령어로 cpu 누산기에 메모리의 변수 y 값을 더한다. store 명령어로 cpu 누산기의 값을 변수 z에 저장한다.
```

#### 기계 사이클

CPU에 설계된 회로가 기계 명령어를 읽어서 처리하는 일련의 과정을 기계 사이클이라고 한다.   
크게 보면 인출, 해독, 피연산자 획득, 실행 등 네 사이클의 과정을 영원히 되풀이한다.

1. 인출 사이클 : CPU 내 PC 레지스터에 저장된 주소의 메모리에서 기계 명령어 하나를 읽어온다.
2. 해독 사이클 : 기계 명령어의 연산 코드가 어떤 것인지 식별한다.
3. 피연산자 사이클 : 해독된 결과에 따라 메모리로부터 피연산자 값을 획득한다.
4. 실행 사이클 : 획득한 피연산자 값과 누산기 사이에 연산 코드에 대응되는 연산을 처리한다.

PC (Program count) -> MAR(주소 버스)         
M -> MBR(데이터버스)      
PC + 4 -> PC (다음 명령어 4는 명령어길이)      
MBR(20~23) -> IR : 어떤 연산인가를 판별   
MBR(0~19) -> MAR : MAR에 복사하여 주소 버스에 실어서    
M -> MBR : MAR로 복사      
AC + MBR -> AC : 연산을 처리하여 그 결과를 AC에 보관   
   
### SVM(Simple Virtual Machine) 단순 가상 기계 어셈블리 프로그래밍

#### svm 기계 구조

1. 기계 명령어의 길이 모든 기계 명령어는 32비트(4바이트)이다.   
2. 연산 코드의 개수 : 연산 코드 8비트 중 상위 두 비트는 00으로 고정 나머지 6비트에 의해 총 64개의 서로 다른 연산 코드 지원   
3. 피연산자의 개수 : 오직 하나의 주소 피연산자만을 표기 가능   
4. 주소를 표현하는 비트 수 : 23번째 X비트가 뒤에 주어진 주소 값에 X레지스터 값을 더하여 최종 주소로 사용할 것인가를 지시 따라서   
0~2의 23승 -1 이 최종 주소 표현 범위 -> 8M-1

#### 레지스터 용도

U : 점프 시 돌아올 주소의 번지를 보관, 리턴 명령어에서 그 주소를 참조   
X : 배열 인덱스를 구현하기 위해 사용 X 레지스터 값을 0,1,2와 같이 증가시키면 첨자기반 배열 접근 가능   
A : SVM 모든 연산은 A레지스터에서 이루어지고 결과가 A레지스터에 누적된다.   

#### 적재, 저장 연산 정의

```assembly
LDA m // m부터 시작하는 4바이트 내용을 A레지스터에 적재
LDAW m // m부터 시작하는 2바이트 내용을 A레지스터에 적재
LDAB m // m부터 시작하는 1바이트 내용을 A레지스터에 적재

LDX m // m부터 시작하는 4바이트 내용을 X레지스터에 적재
LDU m // m부터 시작하는 4바이트 내용을 U레지스터에 적재

STA m // A레지스터 내용을 m부터 시작하는 메모리 4바이트에 적재
STAW m // A레지스터 내용을 m부터 시작하는 메모리 2바이트에 적재
STAB m // A레지스터 내용을 m부터 시작하는 메모리 1바이트에 적재

STX m // X레지스터의 내용을 m부터 시작하는 메모리 4바이트에 적재
STU m // U레지스터의 내용을 m부터 시작하는 메모리 4바이트에 적재
```

#### 수리 연산 코드 정의
```assembly
ADD m // m부터 시작하는 메모리 4바이트의 내용을 A 레지스터에 가산
ADDW m // m부터 시작하는 메모리 2바이트의 내용을 A 레지스터에 가산
ADDB m // m부터 시작하는 메모리 1바이트의 내용을 A 레지스터에 가산

SUB m // m부터 시작하는 메모리 4바이트의 내용을 A 레지스터에 감산
SUBW m // m부터 시작하는 메모리 2바이트의 내용을 A 레지스터에 감산
SUBB m // m부터 시작하는 메모리 1바이트의 내용을 A 레지스터에 감산

MOD m // m부터 시작하는 메모리 4바이트 내용으로 A레지스터 4바이트에 mod 연산
```

#### 논리(비트) 연산
```assembly
AND m // m부터 시작하는 메모리 4바이트의 내용을 A 레지스터에 AND 연산
OR m // m부터 시작하는 메모리 4바이트의 내용을 A 레지스터에 OR 연산
XOR m // m부터 시작하는 메모리 4바이트의 내용을 A 레지스터에 XOR 연산

SHL m // m부터 시작하는 메모리 4바이트의 내용만큼 A 레지스터 4바이트를 왼쪽으로 시프트
SHR m // m부터 시작하는 메모리 4바이트의 내용만큼 A 레지스터 4바이트를 오른쪽으로 시프트
```

#### 비교 연산 

```assembly
CMP m, (CMPW m, CMPB m) // A 레지스터 4(2, 1)바이트와 주소 m부터 시작하는 내용을 뺄셈으로 비교
IXC m // X레지스터 4바이트를 4 증가시키고 그 결과와 주소 m부터 시작하는 메모리 4바이트를 뺄셈으로 비교
IXWC m // X레지스터 4바이트를 2 증가시키고 그 결과와 주소 m부터 시작하는 메모리 4바이트를 뺄셈으로 비교
IXBC m // X레지스터 4바이트를 1 증가시키고 그 결과와 주소 m부터 시작하는 메모리 4바이트를 뺄셈으로 비교
```

#### 분기 연산 정의

```assembly
JZ m // PSW 레지스터의 Z 플래그가 1이면 m 번지로 점프
JNZ m // PSW 레지스터의 NZ 플래그가 1이면 m 번지로 점프
JC m // PSW 레지스터의 C플래그가 1이면 m 번지로 점프
JMP m // 무조건 주소 m번지로 점프

GSUB m // 서브루틴으로 점프
RSUB // 서브루틴에서 주프로그램으로 점프
```

#### 입,출력 연산 정의

```assembly
CID m // m 부터 시작하는 메모리에 주어진 입력 장치 포트의 상태를 체크
COD m // m 부터 시작하는 메모리에 주어진 출력 장치 포트의 상태를 체크
IN m // m부터 시작하는 메모리 4바이트에 주어진 입력장치포트에서 1바이트를 읽어 A레지스터 하위 1바이트에 적재
OUT m // A레지스터 하위 1바이트 내용을 m부터 시작하는 메모리 4바이트에 주어진 출력 장치 포트로 출력
```

#### 어셈블리 프로그램에서 상수 및 변수 공간 확보

기계명령어와는 달리 주어진 위치에 주어진 형태의 공간 확보를 지시한다는 의미로 지시자가 존재한다.
   
어셈블러 지시자 정의
```assembly
DD vd0 // 4바이트의 공간을 확보하고 특정 값을 채움
DW vw0 // 2바이트의 공간을 확보하고 특정 값을 채움
DB vb0 // 1바이트의 공간을 확보하고 특정 값을 채움
RD nd // 4바이트 단위의 공간을 하나 이상 확보
RW nw // 2바이트 단위의 공간을 하나 이상 확보
RB nb // 1바이트 단위의 공간을 하나 이상 확보
```

#### 변수 주소의 기호화

변수 공간 확보 명령어 앞에 기호를 두고 이들 기호가 주소를 의미하도록 코딩할 수 있다.

#### DMP 니모닉 코드(연산 명령어)

디버깅을 위해 레지스터나 메모리에 저장된 값을 출력하는 DMP 명령어를 정의한다.
```assembly
DMPR r // PC(:0), U(:1), X(:2), A(:3) 레지스터 덤프
DMP m // 주소에 해당하는 메모리 내용 덤프
DMP m[X] // 주소에 해당되는 메모리 내용 덤프
DMPS m // 주소에서 시작하는 문자열 덤프
```

#### HALT 니모닉 코드

START로 시작하고 END로 끝을 맺는다. 실행도중 프로그램을 종료시킬 수 있는 명령어가 필요하다.   
길이가 4바이트고 피연산자가 필요없으며 연산 코드는 3Fh이다.

#### FILE 지시 명령어

SVM의 IN, OUT 기계 명령어 지원을 위해 입출력 장치포트 번호를 디스크 파일과 매핑하는 FILE 지시 명령어를 제공한다.
```assembly
FILE 124, I, DATA.TXT
```

#### 난수 생성 내장 변수

LDA RND 혹은 LDA 7FFFFFh를 사용하면 4바이트 난수가 얻어진다.   
반대로 STA RND 혹은 STA 7FFFFh를 사용하면 난수 씨앗을 새롭게 설정한다.

