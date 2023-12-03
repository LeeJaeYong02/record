
## Keychain 이란 Apple에서 제공하는 암호화된 데이터 저장 공간(encrypted database)을<br/> 의미하며, 금융 정보, 인증서, 토큰 정보등의 데이터는 해당 공간의 저장하는것이 원칙이다.

<br/>

![다운로드](https://user-images.githubusercontent.com/66985977/189514014-8e2ab8a0-e16c-46f0-b73c-26a8d17d40e8.png)

<br/>

# 예제
## 임의 테스트 값을 KeyChain 저장공간에 저장시킨다.<br/>
` Objective-C 를 사용하였다. ` <br/>
` KeychainItemWrapper 클래스를 활용하였다. ` <br/>
<br/>

## KeychainItemWrapper 클래스준비
Keychain의 API들은 이미 Objective-C 내 Security.framework에 정의되어 있지만 C함수로 정의가 되어 있어 사용이 매우 까다롭다.<br/>
이런 이유로 애플에서 KeychainItemWrapper 클래스를 제공하고 있으며, 이를 사용하는 경우가 대다수이다.<br/>
<br/>
(Class Download : <a href='https://developer.apple.com/library/ios/samplecode/GenericKeychain/Introduction/Intro.html#//apple_ref/doc/uid/DTS40007797-Intro-DontLinkElementID_2' target='_blank'>다운로드 바로가기</a>)<br/>
해당 사이트에서 파일을 받을 수 있지만, 이제는 Swift용 파일밖에 존재하지 않기 떄문에 Objective-C 용 파일을 따로 첨부.<br/>
<br/>
( 다운로드 : [KeychainItemWrapper.zip](https://github.com/StreetStudy/Solution/files/9541948/KeychainItemWrapper.zip) )<br/>
<br/>

![image](https://user-images.githubusercontent.com/66985977/189516129-78608d05-8527-484a-98e9-618c8b26f4bc.png)<br/>

<br/>

## KeyChain 실제 사용
먼저 해당 Class Header를 import 한다

      #import "KeychainItemWrapper.h"

### 앱이 로드되는 시점에서 UUID라는 아이템에 테스트값을 삽입/수정

      @interface ViewController ()

      @end

      @implementation ViewController

      - (void)viewDidLoad {
          [super viewDidLoad];
          // Do any additional setup after loading the view.

          // KeychainItemWrapper 인스터스는 하나의 keychain item을 다루며 identifier와 accessGroup 을 설정하여 초기화 한다.
          // initWithIdentifier : 키체인 아이템을 구분짓는 이름
          // accessGroup : 아이템이 저장될 키체인 그룹을 의미하며, 설정시 다른앱에서도 접근이 가능하지만 nil로 설정시 앱 내부에서만 접근가능
          KeychainItemWrapper *keychain = [[KeychainItemWrapper alloc] initWithIdentifier:@"UUID" accessGroup:nil];

          NSString *data = @"test1";

          [self KeyChainSave:keychain:data]; //method call
      }

      - (void)KeyChainSave: (KeychainItemWrapper*)keychain :(NSString*)dataOne { // method
          // 삽입
          [keychain setObject:dataOne forKey:(id)kSecAttrAccount];
      }

여기서 forkey의 용도<br/>
kSecValueData : 비밀번호 정보<br/>
kSecAttrAccount : 계정 ID 정보<br/>
kSecAttrServer : 접속하려는 서버 정보<br/>
kSecClass : Keychain Item Class<br/>

### UUID라는 아이템을 조회
      - (void)viewDidLoad {
          [super viewDidLoad];
          // Do any additional setup after loading the view.
            
          // ~
          KeychainItemWrapper *keychain = [[KeychainItemWrapper alloc] initWithIdentifier:@"UUID" accessGroup:nil];

          [self KeyChainSelect: (KeychainItemWrapper*)keychain]; // method call
      }
      
      - (void)KeyChainSelect: (KeychainItemWrapper*)keychain { // method
          // 조회
          NSString *selectData = [keychain objectForKey: (id)kSecAttrAccount];

          NSLog(@"조회 데이터 : %@", selectData);
      }

![image](https://user-images.githubusercontent.com/66985977/189517152-0aeea91a-45ee-4955-a1fb-24c5895eaa6d.png)


### UDID라는 아이템 초기화(혹은 삭제)
      - (void)viewDidLoad {
          [super viewDidLoad];
          // Do any additional setup after loading the view.

          // ~
          KeychainItemWrapper *keychain = [[KeychainItemWrapper alloc] initWithIdentifier:@"UUID" accessGroup:nil];

          [self KeyChainDelete: (KeychainItemWrapper*)keychain]; // method call
      }
      
      - (void)KeyChainDelete: (KeychainItemWrapper*)keychain { // method
          // 삭제
          [keychain resetKeychainItem];
      }
      
      - (void)KeyChainAllDelete { // method
             // 모두 삭제
             NSArray *secItemClasses = @[(__bridge id)kSecClassGenericPassword,
                                    (__bridge id)kSecClassInternetPassword,
                                    (__bridge id)kSecClassCertificate,
                                    (__bridge id)kSecClassKey,
                                    (__bridge id)kSecClassIdentity];
              for (id secItemClass in secItemClasses) {
                  NSDictionary *spec = @{(__bridge id)kSecClass: secItemClass};
                  SecItemDelete((__bridge CFDictionaryRef)spec);
              }
      }
