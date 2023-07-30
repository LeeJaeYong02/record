
# 해시뱅(HashBang)

![화면 캡처 2022-09-05 014833](https://user-images.githubusercontent.com/66985977/188324447-0b445616-706d-4480-b66e-bba7c87bec7e.png)

https://www.naver.com/#!location

해당 URL은 해시뱅(#!)이 포함되어 있으며, 해시뱅(#!)을 기준으로 다음에 내용이 변경되어도 페이지가 전환되지 않는다.

이러한 해시뱅(#!)의 대표적인 활용사례로는 

GET방식으로 파라미터를 통해 페이지를 구분하여 접근한다면 간편하게 웹페이지를 공유하고 접근할 수 있지만 페이지가 계속 전환되어야 한다는 단점이 있다.
페이지가 계속 전환된다는것은 불필요하게 사용자의 리소스를 소모하고 시각적 불편함을 느끼게 할 수 있다.

하지만 해시뱅(#!)을 활용하여 ajax를 통해 페이지를 전환시킨다면 GET방식의 장점을 그대로 사용하며, 최소한의 리소스 소모와 시각적으로 매끄러운 전환이 가능하다.
또한 간편하게 해당 웹페이지를 공유하고 접근하는것도 가능하다.

<br/>
<hr/>
<br/>

# 예제
### JQuery를 사용하였다. (HTML 5 pushState는 사용하지 않았다)
![image](https://user-images.githubusercontent.com/66985977/189350033-ca4b11f0-bc38-4111-be5e-e2f5b81dd732.png)<br/>
(메뉴 화면)
<br/>

			<body>
				<table>
					<tr>
						<td colspan="3">
							페이지 리스트
						</td>
					</tr>
					<tr>
						<td class="radioTd"><input type="radio" id="page_1st" name="pageLocationBtn"><label for="page_1st">첫번째 페이지</label></td>
						<td class="radioTd"><input type="radio" id="page_2nd" name="pageLocationBtn"><label for="page_2nd">두번째 페이지</label></td>
						<td class="radioTd"><input type="radio" id="page_3rd" name="pageLocationBtn"><label for="page_3rd">세번째 페이지</label></td>
					</tr>
				</table>
				<div id="pageContentDiv">
					<!-- result 페이지로 변경될 영역 -->
				</div>
			</body>
(메뉴 HTML)
<br/>

![image](https://user-images.githubusercontent.com/66985977/189347490-94cafee0-1a4b-4472-8932-6320535f1161.png)<br/>
(호출할 페이지 항목 구성)

<br/>

### 화면에 보이는 페이지 전환 버튼을 눌렀을때 이벤트 처리<br/>

` 버튼의 id와 실제 호출할 페이지 id를 매핑시켰다. ` <br/>
` location.href 를 사용하였지만 동기화는 진행되지 않는다. (현재의 url만 변경되며, history에 기록된다.) ` <br/>
` href가 변경될때마다 서버 통신 메서드를 호출한다. (웹 브라우저에 존재하는 backward, forward 버튼을 클릭했을때도 동작한다.) `  <br/>

		$(function() {
			$("input[name='pageLocationBtn']").click(function() {
			    location.href = window.location.pathname.split('#!')[0] +"#!" + $(this).attr('id');
			});

			window.addEventListener('popstate', function() {
			    content_Transeform(window.location.href.split('#!')[1]);
			});
		});

<br/>

### 서버와 통신하는 메서드(html 데이터를 호출할 ajax)
		function content_Transeform(pageName) { 
			$.ajax({
			    url : "/hashBang_locationTO_" + pageName,
			    type : "POST",
			    async : true,
			    dataType : "html",
			    cache : false,
			    success : function(result) {
				$("#pageContentDiv").html(result);
			    }
			});
		    }

### 서버 처리
		@RequestMapping(value = "/hashBang_locationTO_{pageId}", method = RequestMethod.POST)
			public String hashBangContentTranceForm(@PathVariable("pageId") String pageId) throws Exception {
				
				return "HashBangPages/" + pageId;
			}

<br/>

![image](https://user-images.githubusercontent.com/66985977/189350086-2cd76baf-d5a5-4392-a6af-994c24e65619.png)<br/>
(비동기식으로 페이지 영역이 전환되었음)

<br/>

### 동기식으로 페이지가 선택되게하는 메서드
		$(document).ready(function() {
			content_Transeform(window.location.href.split('#!')[1]);
			$("input:radio[name='pageLocationBtn']:radio[id='" + targetPage + "']").prop('checked', true); // 메뉴가 자동으로 선택되도록.
		    });

(http://localhost:8091/hashBang#!page_1st << 해당 URL로 접근시 자동으로 메뉴가 선택되어 페이지가 보일것이다.)

<br/>
<hr/>
<br/>

# 해시뱅의 심각한 단점

### 1. 자바스크립트가 동작하지 않는 환경이거나 자바스크립트에 오류가 발생한다면 해당 웹애플리케이션은 마비가 된다.
<br>

### 2. 해시뱅 형태의 URL은 검색 엔진에 크롤링되지 않는다.
<br>


<hr/>
