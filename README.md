# cosmos-sdk 메뉴얼 실습

## 실습환경
구분 | 내용
---|---
OS | win10
IDE | GoLand
Go 버전 | go1.11.5
sdk 버전 |  v0.32.0
---
## 실습과정
&nbsp;
### 1. 참고문서 준비

영문
https://cosmos.network/docs/

한글(번역 진행중)
https://github.com/cosmos/cosmos-sdk/tree/develop/docs/translations/kr

&nbsp;
### 2. 진행순서
    1. cosmos-sdk 다운로드 및 특정 stable 버전 체크아웃
    2. gaia 어플리케이션 빌드및 설치
    3. 자체 private gaia 테스트넷 세팅
    4. gaia 테스트넷에 gaiacli 연결하기(Light-client)
    5. gaia 풀노드에서 동작할  sdk application 만들기

&nbsp;
### 3. 따라하기
#### 1. cosmos-sdk 다운로드 및 특정 stable 버전 체크아웃
https://github.com/cosmos/cosmos-sdk.git 저장소에서 설치
 branch: stable/0.32.x, tag: v0.32.0 버전 체크아웃

&nbsp;
#### 2. gaia 어플리케이션 빌드및 설치
https://github.com/cosmos/cosmos-sdk/blob/develop/docs/translations/kr/gaia/installation.md

* Go 설치(버전 1.11.5)
https://golang.org/dl/

* GOPATH, GOBIN, PATH 환경세팅(win10 환경)
https://github.com/arahansa/golkorea/wiki/01.-Go%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95_Windows-%ED%8E%B8

* 실행파일 빌드를 위한 IDE GoLand 설치(커맨드라인 빌드 하실 분들은 GoLand 설치 불필요)
https://www.jetbrains.com/go/ 페이지에서 Download(Free 30-day trial) 버튼을 통해 다운로드.
~~~
주의사항: GoLand는 intelliJ와 마찬지로 jetbrain사의 유료 IDE임. trial버전으로 30일 사용가능.
~~~
* GoLand의 File=>Setting 메뉴를 열고 GOROOT, GOPATH세팅.
~~~
GOPATH는 Global GOPATH외에도 따로 지정된 폴더를 만들어서 Project GOPATH 세팅해 줍니다.
dep를 사용해서 의존성 패키지 관리할 경우는 GOPATH는 Global GOPATH기본 설치시 설정되는것 하나만 유지.
~~~
* gaia 어플리케이션(gaiad.exe, gaiacli.exe) 빌드설정
~~~
GoLand의 Run=>Edit Configurations 선택.
(+)버튼 Add New Configuration눌러 Go build 추가.

Name을 입력하고(gaiad 빌드파일의 이름이 됩니다)
Run kind를 File에 놓고 Files를
\project_path(ex:cosmos-sdk)\cmd\gaia\cmd\gaiad\main.go 로 세팅
Output directory를 project root path나 다른 디렉토리로 세팅하고
빌드만 할 것이므로 아래의 Run after build 체크박스 체크를 없애줍니다.

gaiacli의 경우도 마찬가지로 새로 Go build를 추가하고 Name을 gaiacli
Files를 \project_path(ex:cosmos-sdk)\cmd\gaia\cmd\gaiacli\main.go 로 세팅
그외는 위와 동일하게 하고 빌드 설정을 마칩니다.(아래 스크린샷 참고)

이후 빌드 선택창에서 빌드할 설정을 선택하고 ▶클릭하면 실행파일이 빌드되어 Output directory에 만들어지게 됩니다.

의존성 패키지들은 Gopkg.lock, Gopkg.toml을 참고하여 dep으로 연동.
~~~

![빌드설정](https://user-images.githubusercontent.com/36718155/53856770-ccbe6a00-4016-11e9-99be-076ffef8e8d8.png)

* gaia 어플리케이션(gaiad.exe, gaiacli.exe) 작업 디렉토리에 설치
~~~
리눅스 환경과는 달리 win10 환경에서 해당 실행파일들을 실행하면 실행파일의 위치와 무관하게 작업 폴더로 해당 드라이브의 루트 디렉토리에 두 종류의 디렉토리가 생성됩니다.
(아래 3번을 수행하면 생성됨)
\.gaiad  <== cosmos 블록체인 데이터
\.gaiacli <== account,key 등의 사용자 관련 데이터

리눅스는 홈디렉토리에 생성
~/.gaiad
~/.gaiacli

여기까지 진행되어 각 파일 빌드가 되었다면 프라이빗 테스트넷을 운영할 준비가 된 것입니다.
~~~

&nbsp;
#### 3. 자체 private gaia 테스트넷 세팅

참고 링크:
https://github.com/cosmos/cosmos-sdk/blob/develop/docs/translations/kr/gaia/deploy-testnet.md

* 제네시스 파일 만들기
~~~
gaiad init --chain-id=testing testing

다시 세팅할려면 작업 디렉토리(\.gaiad) 파일들을 모두 삭제후 위의 명령어로 재 생성합니다.
~~~
위와 같이 하면 .\gaiad\config 디렉토리에 genesis.json 이 생성됨.
* Validator accont 생성 및 관리
~~~
gaiacli keys add validator    <== 신규생성(name: validator)
gaiacli keys list             <== 목록확인
gaiacli keys update validator <== 비번 갱신
gaiacli keys delete validator <== 비번 갱신

gaiacli keys add validator --recover <== mnemonic 을 통해 account key 복구
Enter a passphrase to encrypt your key to disk: 암호입력
Repeat the passphrase: 암호 재확인
> Enter your bip39 mnemonic
24개의 mnemonic 입력(신규 생성시 출력되는 mnemonic들을 저장해 놓아야 복구가능)
~~~
생성된 account는 .\gaiacli\keys 아래에 생성됩니다.

* 제네시스 파일에 validator account 추가하기
~~~
지금 단계에서는 genesis.json 에 account가 없는 상태입니다.
따라서 블록체인을 생성하기 전에 제네시스 파일에 위에서 생성한 key address를 아래의 명령어로 추가해 줍니다.

gaiad add-genesis-account (validator들의 ADDRESS) 100000000stake,100000000validatortoken

위에서 메뉴얼대로 1000stake,1000validatortoken 으로 설정하면 오류발생. 100000000처럼 더 큰 값이 필요.
~~~
이렇게 하면 genesis.json의 account가 최초 생성시 "accounts": null  에서 validator의 정보들로 설정되게 됩니다.

* validator를 위한 트랜젝션 생성
~~~
gaiad gentx --name validator
명령어를 입력하면 validator 의 암호를 묻고 이를 입력하면
Password to sign with 'validator': ********
Genesis transaction written to "\\.gaiad\\config\\gentx\\gentx-92b48c98aa3f8b628a63ac73dd9a53a97c6a8359.json"

이와같이 트랜젝션이 생성됩니다.
~~~

* 제네시스 파일에 validator의 트랜젝션 추가.
~~~
gaiad collect-gentxs
명령어를 입력하면 위에서 만들어진 gentx가 genesis.json에 추가됩니다.
~~~

* Gaia 시작
~~~
gaiad start
명령어를 입력하면 가이아 프라이빗 테스트넷이 시작됩니다.
시작되면 블록들이 5초마다 생성되는데 테스트를 위해 쌓인 데이터들을 리셋하려면 다음괴 같이 입력합니다.(For test develop)

gaiad unsafe-reset-all
테스트넷 리셋 모든 블록데이터 날아감.
gaiad start
첫 블록부터 재 시작.
~~~

&nbsp;
#### 4. gaia 테스트넷에 gaiacli 연결하기(Light-client)

연결 전 준비사항. gaiacli에 swagger-ui 리소스 추가.
~~~
gaiad정보를 조회하기 위해 gaiacli를 gaiad에 연결하고 rest-server로 띄울수 있는데 rest api로 동작하려면 http 파일 폴더를 resource 폴더 형태로 사용할수 있도록 statik(https://github.com/rakyll/statik)을 사용해서 폴더를 statik.go 파일로 변환해 주는 작업을 하고 그 파일을 소스에 추가한 후 gaiacli를 다시 빌드해 사용해야 합니다.
(리눅스용 MakeFile보면 아래와 같은 스크립트를 볼수있다.)
@statik -src=client/lcd/swagger-ui -dest=client/lcd -f
~~~
먼저 statik을 사용하기 위해 bin파일을 생성
~~~
go get github.com/rakyll/statik
$GOPATH\bin에 statik.exe 파일 생성 확인
~~~
swagger-ui 폴더를 파일로 변환
~~~
GoLand IDE에서 Terminal 탭을 열어 project root directory에서 swagger-ui 폴더를 찾아 커맨드라인 명령으로 statik.go 파일로 변환해줍니다.
(아래 스샷 참고)
statik -src=client/lcd/swagger-ui -dest=client/lcd -f
그러면 ./client/lcd/statik  폴더에 statik.go파일이 생깁니다.

이제 다시 빌드하면 rest server에서 사용되는 api페이지들이 정상적으로 보여집니다.
이 작업을 하지 않으면 404error발생.
~~~

![statik_swagger-ui](https://user-images.githubusercontent.com/36718155/53862059-655de580-4029-11e9-948d-ee156b52d3a5.png)

gaiacli를 gaiad에 연결(라이트 클라이언트)
참고링크: https://github.com/cosmos/cosmos-sdk/blob/develop/docs/translations/kr/clients/lite/getting_started.md
~~~
statik.go 파일이 소스에 추가된것을 확인하고 gaiacli를 다시 빌드하면 이제 rest server로 동작하는 gailcli를 gaiad에 연결할수 있게됩니다.

연결은 다음과 같은 명령으로 연결합니다.

gaiacli rest-server --chain-id=testing --laddr=tcp://localhost:1317 --node tcp://localhost:26657 --trust-node=false
~~~
&nbsp;
이제 아래 스샷처럼 gaiacli의 rest-api를 통해 gaia정보를 확인할수 있습니다.
사용 가능한 rest-api 리스트는 다음 링크에서 확인 가능합니다.
https://cosmos.network/rpc/#/
&nbsp;
![gaiacli_rest_api](https://user-images.githubusercontent.com/36718155/53863027-6fcdae80-402c-11e9-8689-9d614dabdad9.png)

gaia full node explorer 웹 서비스를 구축하려면 다음 저장소를 참고하면 될것 같습니다.
(시간이 되지 않아서 해보지 못함)
https://github.com/cosmos/explorer

gaiacli 한글메뉴얼
https://github.com/cosmos/cosmos-sdk/blob/develop/docs/translations/kr/gaia/gaiacli.md

&nbsp;
#### 5.  gaia 풀노드에서 동작할 sdk application 만들기
지금까지 cosmos-sdk를 활용하여 private gaia full node를 구축하고 이를 조회할수 있는 rest-api 서비스를 띄워 보는 것까지
진행해 보았습니다.
이제 이 gaia full node와 연결되는 custom blockchain을 만들어 full node와 연동해 보도록 합니다.

참고링크: (영문) https://github.com/cosmos/sdk-application-tutorial/tree/master/tutorial

위 링크에서 단계별로 폴더와 소스파일들을 만들고 코드들을 실습하면서 진행하다보면 마지막으로 아래와 같은 구조가 됩니다.

```bash
./nameservice
├── Gopkg.toml
├── Makefile
├── app.go
├── cmd
│   ├── nscli
│   │   └── main.go
│   └── nsd
│       └── main.go
└── x
    └── nameservice
        ├── client
        │   ├── cli
        │   │   ├── query.go
        │   │   └── tx.go
        │   ├── rest
        │   │   └── rest.go
        │   └── module_client.go
        ├── codec.go
        ├── errors.go
        ├── handler.go
        ├── keeper.go
        ├── msgs.go
        ├── querier.go
        └── types.go
```

튜토리얼을 따라 오면서 위와 같은 구조를 가진 디렉토리가 되었다면 nscli와 nsd를 컴파일하여
빌드되는지를 확인해 봅니다.

그리고 빌드에 성공 하였다면 다음 메뉴얼을 참고하여 자신만의 custom blockchain을 생성해 보도록 합니다.

참고링크: https://cosmos.network/docs/tutorial/build-run.html#running-the-live-network-and-using-the-commands

gaiad와 gaiacli로 블록체인 네트워크를 시작하는 메뉴와 유사하므로 앞서 실습에서 해 보셨다면 도움이 되리라 생각됩니다.

위에서 따라해본 custom blockchain 소스파일은 아래 경로에 올려져 놓았사오니 메뉴얼대로 진행하시면서 참고하시면 될거 같습니다.

https://github.com/jsyun-epi/cosmos-app-test

(예제 코드의 nameservice를 epitomecl로 변경함. 컴파일까지는 하였으나 실제 네트워크 실행해보지는 못함)

메뉴얼과는 달리 최신 cosmos-sdk 커밋에서 nscli에서 api 로 key관련 작업하는 부분을 제거해서 메뉴얼대로 진행하시면
빌드 에러가 하나 발생하는데 이부분은 주석 처리하면 될거 같습니다.

https://github.com/cosmos/cosmos-sdk/commit/e39debd359a71768ebd2cc021cd5e76f2d49664b#diff-18dba5950615b0728971c4da291580fe
