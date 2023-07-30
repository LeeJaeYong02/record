# UIWebView 스크립트와 상호작용

1. [UIWebView 기준]
 - Objective-C 언어 사용
 - StoryBoard 사용
2. [WKWebView 기준]
 - Swift 언어 사용
 - Swift UI 사용
<br/>

## UIWebview와 WKWebview 스크립트와 상호작용 특징

기본적으로 WKWebView나 UIWebView는 자바스크립트간의 통신이 비동기적이기 때문에 안드로이드처럼 바로 자바스크립트에서 값을 바로 리턴받는 식으로 동작할 수 없다.

Script to App 호출을 하였을때 특정값을 리턴받고 싶다면 임의로 CallBack 메서드를 정의하여 App to Script 메서드를 호출 하도록 처리하여야 한다.

동기방식이 아니기도 하며, script에서 url scheme(혹은 webview request)을 단시간내에 여러번 보낸다면 마지막 요청만 인식을 하고 동작한다.<br/>
~~(함수가 동작할때 메서드를 재호출 시 작중인 처리를 멈추고 호출된 메서드를 처리하는 C의 특징과 닮은게 아닌가싶다)~~

여러번 보내는 url scheme이 있다면 구간마다 끊어 네이티브에서 각각 반응하도록 해야한다.

## [UIWebView 기준]

참고사항. <br/>
현재 UIWebView *Deprecated 상태이다. UIWebView를 사용한 신규앱은 출시할 수 없으며,<br/>
기존 UIWebView를 사용하는 앱에 대해서 업데이트를 허용하지 않을것 이라고 애플에서 밝혔다. <br/>
현재시점에서 UIWebView를 사용하고 있는 앱은 모두 *Legacy라는 소리이다.<br/><br/>
` Deprecated(디프리케이티드) : 중요도가 떨어져 더 이상 사용되지 않고 앞으로는 사라지게 될`<br/>
` Legacy(레거시) : 과거로 부터 물려 내려온 것들 `
<br/>

<details>
<summary>시작하기전 UI웹뷰 기본 세팅 <접기/펼치기></summary>
<div markdown="1">

<img src="https://user-images.githubusercontent.com/66985977/193061841-209d19bf-7466-41ab-9a67-ea9e46e59b26.png"><br/>

 WebKit Framework 추가
 
 <br/>
 
<img src="https://user-images.githubusercontent.com/66985977/193058359-75b69fe3-2034-4bc2-920a-92f70e97ba78.png"><br/>
 
 1. 커스텀 클래스를 웹뷰를 사용할 컨트롤러에 헤더로 지정
 2. 프로퍼티스 변수를 선언
 3. 해당 변수를 생성한 웹뷰에 연결
 

 ### ViewController.m
	
``` 
#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

@synthesize webView;

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.

    // 웹뷰 초기화
    [self initWebView];
}

- (void) initWebView{
    [super viewDidLoad];
    NSString *urlAddress = @"https://naver.com";
    self.webView.delegate = self;
    [webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:urlAddress]
				cachePolicy:NSURLRequestReloadIgnoringLocalCacheData
				timeoutInterval:60]];
}

@end

```
<br/>
	
<img src="https://user-images.githubusercontent.com/66985977/193284141-477b26c1-9177-482a-b560-68fbf5049e6f.png"><br/>	

	
</div>
</details>

<br/>
<br/>
	
## [Script to App 파라미터 두개를 앱내 콘솔에서 로그출력]
	
### Script to App : Script part

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
    <script>
        function Evt() {
            window.location = "ioscall://callMethod?param1=p1&param2=p2";

            // ioscall:// - scheme, 요청의 프로토콜명 해당 값으로 webview로 request를 보낼때 페이지로 이동할지 혹은
            // 앱에서 정의된 scheme을 호출을 통해 앱 메서드를 호출할지 결정된다.

            // callMethod - host, 앱 메서드 이름

            // ?param1=p1&param2=p2 - query, 파라미터이다.
        }
    </script>
</head>
<body>
    <button onclick="javascript:Evt()">Evt!!</button>
</body>
</html>
```

### Script to App : App part
	
```
#import <UIKit/UIKit.h>
 
#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    // 웹뷰 초기화
    [self initWebView];
}

- (void) initWebView{
    [super viewDidLoad];
    
    // 해당 웹뷰의 Delegate를 현재 ViewController로 지정해준다.
    self.webView.delegate = self;
    
    NSString *urlAddress = @"http://192.168.35.62:8091/link/AsyncTest.jsp";
    [_webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:urlAddress]
                                cachePolicy:NSURLRequestReloadIgnoringLocalCacheData
                                timeoutInterval:60]];
}

#pragma mark - UIWebView Delegate Methods
 
// 웹뷰의 리퀘스트가 시작되면 (주소가 바뀌면) 실행되는 함수
// UIWebView일때 location.href 혹은 <a href=''.../> 를 사용하면 아래 함수가 실행된다.
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{
    if([request.URL.scheme rangeOfString:@"ioscall"].location != NSNotFound){
        
        // request.URL.scheme : 요청의 프로토콜명 보통은 http / https가 넘어온다.
        // request.URL.host : 함수의 이름
        // request.URL.query : 파라미터
        
        
        SEL _selector_from_web; // 넘겨받은 함수를 확인 및 실행하기 위한 NSSelector 변수
        
        if(request.URL.query) // 변수가 있는지 없는지 확인한다. // 변수가 존재하는 경우에는 Selector에 : 가 붙으므로 나누어 선언해준다.
            _selector_from_web = NSSelectorFromString([NSString stringWithFormat:@"%@:" , request.URL.host]);
        else
            _selector_from_web = NSSelectorFromString(request.URL.host);
        
        if([self respondsToSelector:_selector_from_web]){
            if(request.URL.query == nil){ // 파라미터가 미존재.
                [self performSelector:_selector_from_web];
            } else {
                NSArray *queries_arr = [[request.URL.query stringByRemovingPercentEncoding] componentsSeparatedByString:@"&"];
                
                NSMutableDictionary *queries = [[NSMutableDictionary alloc]init];
                for(int idx = 0 ; idx < queries_arr.count ; idx++){
                    if([[queries_arr objectAtIndex:idx] rangeOfString:@"="].location == NSNotFound){ // 형식이 잘못되었을 경우
                        break;
                    }
                    NSArray *separated = [[queries_arr objectAtIndex:idx] componentsSeparatedByString:@"="];
                    [queries setObject:[separated objectAtIndex:1] forKey:[separated objectAtIndex:0]];
                }
                
                [self performSelector:_selector_from_web withObject:queries];
            }
        } else{
            NSLog(@"App Method Not Found!!");
            return NO;
        }
    }
 
    return YES;
}

#pragma mark - JS -> APP Methods
- (BOOL)callMethod:(NSMutableDictionary *)params {
    NSLog(@"param1: %@",[params objectForKey:@"param1"]);
    NSLog(@"param2: %@",[params objectForKey:@"param2"]);
    
    return NO;
}

@end

```

App Part 에서 파라미터가 존재하는지 체크를 하여 분기를 처리하는 이유는
(BOOL)callMethod 라는 메서드는 하나의 파라미터를 받도록 선언되어있으며, Selector 메서드 호출 방식의 의해 ':' 가 포함되어 있어야 하기 때문이다.

Selector를 통해 메서드를 호출때 하나의 파라미터를 받도록 선언되어있을때 ':'를 포함하지 않았을때 현재 선언한 메서드와 다른 메서드를 호출하는걸로 판단되기 때문에
메서드를 찾을 수 없을것이다.

### result

![image](https://user-images.githubusercontent.com/66985977/193435067-b474ccd3-91c0-4189-b68a-3ecf164e6beb.png)<br/>
	
<br/>

## [Script -> App -> Script number Type에 값을 앱으로 보내서 합의 결과를 Script로 전달]
	
### Script -> App -> Script : Script Part
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
    <script>
        function Evt() { // 앱 메서드를 호출할 콜 메서드 정의
            window.location = "ioscall://callMethod?param1=1&param2=2"; //임의로 파라미터를 1과 2로 
        }
        function EvtCallBack(sum) { // 앱에서 호출할 콜백 메서드 정의
            alert(sum);
        }
    </script>
</head>
<body>
    <button onclick="javascript:Evt()">Evt!!</button>
</body>
</html>
	
```
	
### Script -> App -> Script : App Part
```
// ...shouldStartLoadWithRequest 메서드는 동일하게 사용
	
#pragma mark - JS -> APP Methods Call
- (BOOL)callMethod:(NSMutableDictionary *)params {
    NSString *param1 = [params objectForKey:@"param1"];
    NSString *param2 = [params objectForKey:@"param2"];
    
    if([self numberCheck:param1] && [self numberCheck:param2]) {
        NSInteger sum = [self nSStringToNSInteger:param1] + [self nSStringToNSInteger:param2];
        [_webView stringByEvaluatingJavaScriptFromString:[NSString stringWithFormat:@"EvtCallBack('%ld');", sum]]; // Script Method Call
        
    } else {
        NSLog(@"Type Error!!");
    }
    
    
    return NO;
}

#pragma mark - NumberType Check Method
- (BOOL)numberCheck:(NSString *)str {
    NSString *check = @"^[0-9]";
    NSRange match = [str rangeOfString:check options:NSRegularExpressionSearch];
    
    if(NSNotFound == match.location)
        return NO;
    else
        return YES;
    
}

#pragma mark - NSString to NSInteger Type Change Method
- (NSInteger)nSStringToNSInteger:(NSString *)str {
    return [self numberCheck:str] ? [str intValue] : 0;
}
	
```

### Result

![image](https://user-images.githubusercontent.com/66985977/193458297-4e77c063-1653-4380-925c-5266e351b2d3.png)<br/>
	

<br/>
	
## UIWebView 상호작용중 발생하였던 이슈

기존에 로그인을 시도후 성공 시 세션을 생성하고 메인페이지로 이동하도록 로직이 구성되어 있었다.<br/>
하지만 이번에 추가되는 기능을 구현하기 위해서는 로그인이 성공하는 시점에서 세션에 저장되는 값의 일부 데이터를 앱으로 받아와야 하는 일이 생겨서<br/>
Script에서 App으로 메서드를 호출하도록 코드를 작성하였었다. 하지만 실제로 테스트를 해보니 앱으로 요청을 보내지 않고 바로 메인페이지로 이동되었다.<br/>
<br/>
발생원인은 처음에 말했다시피 비동기식 처리방식에다 호출한 메서드에 대해서 리턴을 받아올 수 없는 구조적인 문제와 <br/>
단시간내 location.href를 요청을때 마지막 요청만 동작을 한다는 문제가 있기 때문이었다. <br/>
( 이것은 webview에서만 나타나는 증상이 아니라 브라우저에서 랜더링될 때까지 현재 페이지의 스크립트를 계속 실행하는 특징이 문제가 되는것이다. )<br/>
<a href='https://stackoverflow.com/questions/37521172/is-javascript-location-href-call-asynchronous'>참고 : Is JavaScript location.href call asynchronous?</a><br/>
<br/>
이때 가장이상적인 처리방식은 script에서 요청한 App 메서드 처리가 다 끝났을때 App에서 Script로 콜백 메서드를 호출하여 자연스럽게 다음동작이 이루어지도록
처리하는것이다.<br/>
<br/>
하지만.. 이러한 작업이 부담스러웠던 시점이라 편법을 시도하였다.

### 첫번째 시도 

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
    <script>
        function FirstMethod() {
            location.href="https://www.daum.net";

            sleep();

            LastMethod();
        }

        function LastMethod() {
            location.href = "https://www.naver.com";
        }

        function sleep() {
            const seconds = new Date().getTime() / 1000;

            while (true) {
                if (new Date().getTime() / 1000 - seconds >= 2) {
                    console.log("Good, looped for 2 seconds");
                    break;
                }
            }
        }
    </script>
</head>
<body>
<button onclick="javascript:FirstMethod()">Evt!!</button>
</body>
</html>
	
```
### 동기식인점을 활용하여, sleep 메서드를 정의하고 n초 후 다음 작업이 이루어질 수 있도록 해보았지만 실패하였다.<br/>
	
<br/>
	
### 웹뷰가 아닌 브라우저에서의 동작 : 첫번째 location.href 요청인 daum.net으로 이동함<br/>
	
(sleep메서드가 동작중 랜더링을 마친 첫번째 location.href 요청이 동작되었다.)<br/>
<br/>
### 웹뷰에서의 동작 : 마지막 location.href 요청인 naver.com으로 이동함<br/>
	
(sleep메서드가 모두 동작한 이후에야 location.href 요청이 전송되었으며,<br/>
uiwebview에서 단시간내에 location.href를 여러번 요청했을때 최종 location.href 요청만 동작한다는 단점이 작용된 것 같다.)
<br/>

<hr/>
	
### 두번째 시도

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
    <script>
        function FirstMethod() {
            iFrame = document.createElement("IFRAME");
            iFrame.setAttribute("src", "https://www.daum.net");
            document.body.appendChild(iFrame);
            iFrame.parentNode.removeChild(iFrame);
            iFrame = null;

            LastMethod();
        }

        function LastMethod() {
            location.href = "https://www.naver.com";
        }
    </script>
</head>
<body>
<button onclick="javascript:FirstMethod()">Evt!!</button>
</body>
</html>
```

### iFrame창을 body에 추가하여 daum.net를 요청하고 바로 없앤 이후에 naver.com을 요청한 방식으로 성공하였다.<br/>
<br/>
결론적으로 브라우저나 웹뷰에서의 동작은 naver.com 페이지로 이동하였지만 첫번째 방식과 다른점은<br/>
daum.net를 한번 요청한 이후에 naver.com페이지 이동했다는 것이다.<br/>
<br/>
	
<hr/>
	
### 위 방식을 활용하여 실제 앱 메서드를 호출했을때 동작 비교
<br/>

확인을 위해 shouldStartLoadWithRequest 메서드 상단에 요청 URL을 로그로 출력했다.
	
![image](https://user-images.githubusercontent.com/66985977/193500581-80e00536-a9b4-4e2f-b949-a2b3b9bdcc45.png)<br/>
	
#### 첫번째 방식
```
<script>
        function FirstMethod() {
            location.href="ioscall://callMethod?param1=10&param2=20";

            sleep();

            LastMethod();
        }

        function LastMethod() {
            location.href = "https://www.naver.com";
        }

        function sleep() {
            const seconds = new Date().getTime() / 1000;

            while (true) {
                if (new Date().getTime() / 1000 - seconds >= 2) {
                    console.log("Good, looped for 2 seconds");
                    break;
                }
            }
        }
</script>
```

![image](https://user-images.githubusercontent.com/66985977/193500640-377dce5a-37cb-4aa3-bc97-ecb51bf33d5a.png) <br/>

최초에 앱메서드를 호출했지만 실제로는 처리가 되지 않고 마지막으로 요청한 naver.com 페이지 이동만 처리되었다.
<br/>

#### 두번째 방식 
<a href='https://stackoverflow.com/questions/2934789/triggering-shouldstartloadwithrequest-with-multiple-window-location-href-calls'>(참고)</a>
	<br/>
```
<script>
        function FirstMethod() {
            iFrame = document.createElement("IFRAME");
            iFrame.setAttribute("src", "ioscall://callMethod?param1=10&param2=20");
            document.body.appendChild(iFrame);
            iFrame.parentNode.removeChild(iFrame);
            iFrame = null;

            LastMethod();
        }

        function LastMethod() {
            location.href = "https://www.naver.com";
	}
</script>	
```

![image](https://user-images.githubusercontent.com/66985977/193501246-51a188f6-50f3-4c6b-9bc7-7c242083be55.png) <br/>

처음에 요청한 메서드 동작과 페이지가 전환되었다.

<hr/>

해결방법은 찾았지만 아직 의문점이 드는것은
웹뷰에서 sleep 메서드가 모두 동작한 이후에 location.href가 동작하여 요청이 시작된다는 것이다.
어떤 원리의 의해서 이런식으로 동작하는건지 찾아보려고 하였지만 알아내지 못하였다.

<hr/>
	<br/>
	<br/>
	
## [WKWebView 기준]
