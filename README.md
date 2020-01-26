# EasyEval Manual (Korean)


## 내려받기

> 주의: LGPL 라이선스를 사용합니다!   
> 아직 베타 버전입니다! 소스에 심각한 문제가 있을 수 있으며, 추후 API 등이 변경될 수 있습니다.

[DropBox 링크](https://www.dropbox.com/s/7gojtbkexdjn38g/EasyEval-2.1b.js?dl=0)

버전 2.1b   
개발 LsmLands

---

### 업데이트 내역

#### 2.1b

* 스레드 풀을 통한 실행 속도 개선 및 멀티코어 지원
* 'Context'의 자바(Java) 클래스 비활성화 시, 외부 클래스 접근 규칙을 사용할 수 없는 문제 해결


#### 2.0b

* EasyEval 공개


## 목차

1. EasyEval 소개
2. EasyEval 상수 및 메소드 목록
3. EasyEval 간단한 사용 방법
4. 참고사항


## 1. EasyEval 소개

<!-- >> 과거, [Rhino 엔진](https://github.com/mozilla/rhino)을 사용한 봇에서 안전하게 'Evaluate' 기능을 사용하는 것은 많이 어려웠습니다. 아무리 자바스크립트의 여러 메소드를 사용하여 막아놓았다 한들, 자바스크립트 특유의 다양한 문법 덕에 대다수의 봇은 빠르게 해킹되었죠. 하지만 몇개월 후, 봇 구동 어플리케이션에서 사용하는 Rhino 엔진의 메소드를 직접 사용하여 자바스크립트 안에서 자바스크립트를 구동하는 봇이 많이 공개되었습니다. 다만 이 방식도 얼마 지나지 않아, 나중에는 여러 봇 제작자들 사이에 Rhino 엔진의 자체 취약점이 공유가 되면서 이 엔진을 직접 사용하는 봇들도 결국 해킹이 되고 말았죠. -->

반갑습니다! 봇 관련 네이버 카페에서 SafeEval 소스를 올렸던 LsmLands 입니다.   
이번에 공개하는 SafeEval의 차기 버전, 'EasyEval'에는 많은 분께서 원하셨던 기능 및 다양한 고급 설정이 포함되어 있습니다.   
EasyEval에서 새롭게 변경된 사항을 함께 알아봅시다!


### 1.1. 'SafeEval.run' 메소드의 사용이 중단되었습니다.

> 'SafeEval.run'이라면, 기존 SafeEval의 핵심적인 메소드 아닌가요?

맞습니다! 기존 SafeEval의 'SafeEval.run' 메소드에서 다양한 기능의 사용이 어렵다는 피드백을 많이 받았습니다. 따라서 이번 EasyEval에서는 단일 메소드가 아닌, 및에서 후술할 객체지향 방식을 사용하도록 권고드리고 있습니다. 물론, 현재 EasyEval 버전 '2.0b'에서는 기존 단일 메소드 또한 사용이 가능합니다. 다만, 기존 메소드의 'isClassVisible' 매개변수는 사용이 불가능합니다.


### 1.2. 기존 SafeEval의 'Context'가 봇 구동 스레드의 'Context'였던 문제가 해결되었습니다.

> 기존 SafeEval도 사용하는 것 자체에는 아무런 문제가 없었는데요.

이 사항에 대한 피드백은 거의 받지 못하였으나, 매우 중대한 문제였습니다. 기존 SafeEval에서는 안전하게 자바스크립트를 실행하기 위한 Rhino 엔진의 'Context' 설정이 무려 봇 구동 스레드에도 적용되는 문제가 있었습니다. 따라서 SafeEval을 실행한 후, 간헐적으로 봇 스크립트의 정상적인 동작이 불가능했던 경우가 있었죠. EasyEval에서는 자바스크립트를 실행할 경우, Rhino 엔진을 새로운 스레드 내부에서 구동하도록 변경되었습니다.


### 1.3. 다양한 기능 및 설정이 추가되었습니다.

내부에서 구동되는 자바스크립트에서의 외부 자바(Java) 클래스 접근 권한에 대한 세부적인 설정이 가능해졌습니다. 또한, 다른 작업 없이도 봇 사용자가 전송한 명령을 직접 처리하는 기능이 추가되었습니다. 이 외에도 각 사용자당 개별적인 'Scope' 사용 등 다양한 기능이 추가되었습니다.


## 2. EasyEval 메소드 목록

> 다음 주제인 EasyEval의 사용 방법을 알아보기 전에, 상수와 메소드 목록부터 보고 가시죠!

```

const SAFEEVAL_DEFAULT_NAME = "EasyEval.js"     실행할 소스의 기본 이름
const SAFEEVAL_DEFAULT_TIEMOUT = 3000           실행할 소스의 최대 실행 시간(ms)
const SAFEEVAL_DEFAULT_DEPTH = 1000             실행할 소스의 최대 스택 깊이
const SAFEEVAL_DEFAULT_RULE_ALLOW = false       실행할 소스의 자바 클래스 접근 규칙의 기본 권한 (true: 허가, false: 차단)

const EASYEVAL_ENABLE = true                    EasyEval 명령 처리 활성화
const EASYEVAL_PREFIX = "@eval"                 EasyEval 명령 접두사

const USEREVAL_ENABLE = true                    UserEval (개인용 EasyEval) 명령 처리 활성화
const USEREVAL_PREFIX = "@user"                 UserEval (개인용 EasyEval) 명령 접두사
const USEREVAL_PREFIX_RESET = "@reset"          UserEval (개인용 EasyEval) 초기화 명령 접두사
const USEREVAL_MAX_USER = 20                    UserEval (개인용 EasyEval) 최대 사용자 인원

function EasyEval(name)                         EasyEval 객체 생성자 (name: 실행할 소스의 이름)
    EasyEval.prototype.timeout                      EasyEval 소스의 최대 실행 시간(ms)
    EasyEval.prototype.depth                        EasyEval 소스의 최대 스택 깂이
    EasyEval.prototype.isJavaEnabled                EasyEval 소스에서 자바(Java) 클래스 활성화 여부
    EasyEval.prototype.putObject(key, value)        EasyEval 소스에 삽입할 속성 추가 (key: 속성(변수)의 이름, value: 해당 속성의 값)
    EasyEval.prototype.putFunction(key, func, obj)  EasyEval 소스에 삽입할 함수 추가 (key: 속성(변수)의 이름, func: 해당 속성의 함수, obj: 'this'로 사용할 객체)
    EasyEval.prototype.removeObject(key)            EasyEval 소스에 삽입할 속성 제거 (key: 속성(변수)의 이름)
    EasyEval.prototype.addRule(regex, isAllow)      EasyEval 소스의 자바 클래스 접근 규칙 추가, 먼저 추가된 규칙을 우선합니다. (regex: 클래스 이름과 완전하게 일치하는 정규식, isAllow: true 허가 / false 차단)
    EasyEval.prototype.removeRule(index)            EasyEval 소스의 자바 클래스 접근 규칙 제거 (index: 규칙 인덱스)
    EasyEval.prototype.exec(src, scope, callback)   EasyEval 소스 실행 (src: 소스, scope: 사용할 Scope (없으면 null 사용), callback: 결과 콜백)
    EasyEval.prototype.runCommand(replier, msg)     EasyEval 명령 처리 (replier: 답장자, msg: 명령 메시지)

function UserEval(easyEval)                     UserEval 객체 생성자 (easyEval: EasyEval 객체)
    UserEval.prototype.exec(userName, src, scope, callback) UserEval 소스 실행 (userName: 사용자 이름 (없으면 자동으로 생성), src: 소스, scope: 사용할 Scope (없으면 null 사용), callback: 결과 콜백)
    UserEval.prototype.runCommand(replier, sender, msg)     UserEval 명령 처리 (replier: 답장자, sender: 사용자, msg: 명령 메시지)

```


## 3. EasyEval 간단한 사용 방법

> 이제 본격적으로 EasyEval을 사용해 봅시다!   
> [ Sample Source for ManDongI-Family Bot ]

```

var easyEval = new EasyEval(SAFEEVAL_DEFAULT_NAME);
var userEval = new UserEval(easyEval);

function response(room, msg, sender, isGroupChat, replier) {
    easyEval.runCommand(replier, msg);
    userEval.runCommand(replier, sender, msg);
}

```

위의 소스는 'ManDongI'님의 봇 및 그와 유사한 API를 사용하는 봇에 호환되는 샘플입니다.   
자바스크립트와 봇에 대하여 잘 알지 못하신다면, EasyEval 소스 아래에 샘플 소스를 그대로 삽입하여 사용하실 수 있습니다.   
'function response( ... )' 바로 위에 아래의 소스를 추가하여 봅시다.

```

easyEval.isJavaEnabled = true;                  // 실행할 소스에서 자바(Java) 클래스를 활성화합니다.

easyEval.putObject("myName", "Bot");            // 문자열 "Bot" 값을 가지는 'myName' 변수를 실행할 소스에 추가합니다.
easyEval.putFunction("sumNum", function(a, b) {
    return a + b;
});                                             // 두 숫자 (a, b)를 더하는 'sumNum(a, b)' 함수를 실행할 소스에 추가합니다.

easyEval.addRule(/java\.lang\.String.*/, true); // 'java.lang.String*' 클래스의 접근을 허가하는 규칙을 추가합니다. 나머지 자바 클래스의 접근은 차단(기본 설정)됩니다. 'java.lang.Class' 클래스의 접근은 허가할 수 없습니다.

```


## 4. 참고사항

저는 언제나 여러분의 의견을 존중합니다!   
혹시 잘못된 부분이나 문제가 있다면, EasyEval을 소개한 사이트(네이버 카페 등) 혹은 메신저(카카오톡 등)로 연락하시기 바랍니다.
