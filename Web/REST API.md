# REST API

### REST API란? ( REST API의 대한 설명과 RESTful의 대한 개념도 잘 설명해 주었다.&nbsp;: <a href='https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html'>바로가기</a> )

<br/>

## 백엔드 프레임워크 선택

### GitHub에서 가장 인기있는 백엔드 프레임워크 TOP Nine
![image](https://user-images.githubusercontent.com/66985977/189573140-b97104a8-dbaf-4cf8-a982-4ffe3a0c0f0b.png)<br/>
인기있는 프레임워크의 대한 설명 : <a href='https://geekloving.net/ko/2022%EB%85%84-%EC%B5%9C%EA%B3%A0%EC%9D%98-%EB%B0%B1%EC%97%94%EB%93%9C-%EC%9B%B9-%EA%B0%9C%EB%B0%9C-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC-10%EC%84%A0'>바로가기</a>
<br/><br/>
` Node.js가 인기가 많은 이유는 자바스크립트 개발자가 웹 서버 개발까지 가능하기에 프론트엔드 개발자가 백엔드 개발을 동시에 할 수 있는 풀스택 개발 능력을 갖출 수 있으며, 빠른 웹 애플리케이션 개발이 가능하기 때문에 개발 생산성이 향상되기 때문이다. `

( Node.js에 대한 간략한 설명 : <a href='https://velog.io/@yjs_177076/Node-JS%EB%8A%94-%EC%96%B8%EC%96%B4%EC%9D%B8%EA%B0%80'>Node JS는 언어인가?</a> )

` Node.js에서 웹 애플리케이션과 API 서버를 구축하는데 가장많이 사용되는 대표적인 프레임워크는 5위에 해당하는 Express이며, 7위 Meteor, 8위 Koa, 9위 Nest, Strapi 모두 Node.js다. `

<br/>

1위에 해당하는 Laravel은 php언어를 사용하고 있지만 ~~php를 사용하는건 전부 legacy 이며, 죽어가는 언어라는 의견이 많다.~~

<br/>

### 구축 목적별 추천 프레임워크<br/>
웹사이트 서버 구축 : MVC 아키텍처 기반인 라라벨, 스프링 등<br/>
REST API 서버 구축 : API를 빠르고 쉽게 만들 수 있는 익스프레스 등

<br/>

## 회원정보 CRUD 예제

` C(INSERT) : POST Method ` <br/>
` R(SELECT) : GET Method ` <br/>
` U(UPDATE) : PUT Method ` <br/>
` D(DELETE) : DELETE Method ` <br/>

### 시작하기전 REST API 테스트 요청 프로그램 추천
(다운로드 페이지 : <a href='https://www.postman.com/'>POSTMAN</a>)<br/>
(OS 환경에 맞게 설치하면됨.)<br/>
<br/>

### 포스트맨 테스트
	@RestController // RestController는 @Controller 와 @ResponseBody 포함
	@RequestMapping("/rest/api/v1")
	public class APIController {

	    @RequestMapping(value = "/test/{testData}", method = RequestMethod.GET, produces = MediaType.APPLICATION_JSON_VALUE)
	    public Map<String, Object> test(@PathVariable("testData") String testData) {
		Map<String, Object> result = new HashMap<String, Object>();

		result.put("yourData", testData);

		return result;
	    }

	}

![image](https://user-images.githubusercontent.com/66985977/189596904-2cfbe188-053d-4cee-86b6-be1e1384baf1.png) <br/>
<br/>

회원정보를 관리할 테이블 설계

![image](https://user-images.githubusercontent.com/66985977/190161233-d2a43d3d-c248-40e6-a845-e7b2f17dbb6b.png)<br/>
<br/>

### C(INSERT) : POST Method
<br/>
요청

![image](https://user-images.githubusercontent.com/66985977/190418589-f21ca003-03d2-417e-8a44-96cd078e3021.png)<br/>

응답 처리
	
	// Controller
	@RequestMapping(value="/user/register", method = RequestMethod.POST, produces = "application/json; charset=utf-8")
	public Map<String, Object> register(HttpServletRequest request) throws Exception {
		Map<String, Object> result = new HashMap<String, Object>();

		if(apiService.idCheck(request.getParameter("userId")) != 0) {
		    result.put("errorCode", "1001");
		    result.put("errorMsg", "아이디가 중복되었음.");
		} else if (!request.getParameter("userPw").equals(request.getParameter("userPwRe"))) {
		    result.put("errorCode", "1002");
		    result.put("errorMsg", "비밀번호 확인이 잘못되었음.");
		} else {
		    Map<String, Object> userRegisterInfo = new HashMap<String, Object>();
		    userRegisterInfo.put("userId", request.getParameter("userId"));
		    userRegisterInfo.put("userPw", request.getParameter("userPw"));
		    userRegisterInfo.put("userNm", request.getParameter("userNm"));
		    userRegisterInfo.put("regIp", (null != request.getHeader("X-FORWARDED-FOR")) ? request.getHeader("X-FORWARDED-FOR") : request.getRemoteAddr());

		    try {
			apiService.userRegister(userRegisterInfo);
			result.put("successMsg", "회원가입 성공.");
		    } catch (Exception e) {
			result.put("errorCode", "1003");
			result.put("errorMsg", "예기치 못한 에러 발생.");
		    }
		}

		return result;
	    }
	    
데이터 저장 완료

![image](https://user-images.githubusercontent.com/66985977/190419987-480cccc4-793a-4c41-a37c-191c420ccbdf.png)<br/>
 <br/>
 <br/>
 <br/>
 <br/>
 ____________ 내용 작성중 ___________________
