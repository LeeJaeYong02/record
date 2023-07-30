내가 파일(이미지)을 base64형태로 전환하려는 이유는 사용자가 등록한 파일을 물리적 형태로 NAS에 저장시키고 해당 파일에 대한 정보를 테이블에 기록하는것은
매우 비효율적이고 리스크가 있다고 생각하기 때문이다. (물리적 파일 손상, 해당 파일에 대한 정보를 기록한 데이터가 의도치 않게 달라지는 경우)
그렇기에 등록한 파일을 base64형태로 인코딩하여 파일을 관리하는게 앞서 말한 단점을 해결 할 수 있을거라고 판단했다.<br/>
` 다른서버와 파일연계를 할때도 용이할듯. `
_________________________________________________________________________________________________________________________________________________________________________

# 예제 - 
1. `화면에서 이미지를 직접 등록함.`
2. `전송과 동시에 서버에서 이미지 정보와 base64형태로 인코딩된 데이터를 테이블에 저장함.`
3. `base64형태로 인코딩된 데이터를 서버측에서 리턴받아서 화면에 뿌려줌.`
<br/>

![image](https://user-images.githubusercontent.com/66985977/189162018-8b81b66e-1ea5-402a-bd0e-464eb303c002.png)<br/>
(기본 폼 화면 *특정 이미지를 선택한 상태)
<br/><br/>

	<body>
		<table>
			<tr>
				<th>이미지 등록</th>
			</tr>
			<tr>
				<td><input id="uploadImage" type="file" accept=".png, .jpg, .gif, .jpeg" value="로컬 파일 가져오기"></td>
			</tr>
			<tr>
				<td style="text-align: center;"><input id="submitBtn" type="button" value="서버로 전송"></td>
			</tr>
		</table>
		<div>
			<img id="returnImg">
		</div>
	</body
(기본 폼 HTML)<br/><br/>

	
## 스크립트 서버통신 CASE

### 1. jQuery ajax를 사용한 방법

		/*
			(서버 전송 이벤트 처리)
			form에서 encrypt 설정과
			ajax에서 
			processData : false, // 데이터 객체를 문자열로 바꿀지에 대한 값이다. true면 일반문자
			contentType : false, // 해당 타입을 true로 하면 일반 text로 구분되어 진다.
			cache: false,
			설정은 웹페이지에서 서버로 파일을 전송하기 위한 필수 세팅값이다.
		*/
		<script>
			document.getElementById("submitBtn").onclick = function() {
				const formData = new FormData();
				formData.set("encrypt", "multipart/form-data");
				formData.append("image", $("#uploadImage")[0].files[0]);

				$.ajax({
					type : "POST",
					url : "/imageSend",
					processData : false,
					contentType : false,
					cache: false,
					data : formData,
					dataType: "json",
					success : function(result){
						if(result.errorMsg == undefined)
							document.getElementById("returnImg").src = "data:image/;base64," + result.returnImage;
					},
					error : function(error){
						console.log(error);
					}
				});
			}
		</script>

### 2. 바닐라(퓨어) 스크립트 ajax 사용한 방법 <br/>
` 모듈, 라이브러리를 사용하지 않은 순수한 스크립트 ` <br/>
` 모던 스크립트를 사용하였다. `

			<script>
				document.getElementById("submitBtn").onclick = function() {
					const formData = new FormData();
					formData.set("encrypt", "multipart/form-data");
					formData.append("image", document.getElementById("uploadImage").files[0]);

					const _xml = new XMLHttpRequest();

					if(_xml) {
						_xml.open('POST', '/imageSend');
						_xml.responseType = 'json';
						_xml.onload = function() {
							if(_xml.status === 200) {
								if(_xml.response.errorMsg == undefined)
									document.getElementById("returnImg").src = "data:image/;base64," + _xml.response.returnImage;
							} else {
								alert("Response Error!");
							}
						}
						_xml.send(formData);
					} else {
						alert("Request Error!");
						return false;
					}
				}
			</script>
개인적으로 jquery 사용을 지양하며, 바닐라 스크립트 사용을 지향한다. <br/><br/>

	
	// 실제 서버 처리
	@RequestMapping(value = "/imageSend", method = RequestMethod.POST)
	@ResponseBody
	public Map<String, Object> imageSend(HttpServletRequest request, MultipartHttpServletRequest fileRequest) throws Exception{
		Map <String, Object> result = new HashMap<String, Object>();

		MultipartFile multipartFile = fileRequest.getFile("image");

		if(!multipartFile.isEmpty()) {
			String MultipartFile_to_Base64 = Base64.getEncoder().encodeToString(multipartFile.getBytes());
			String fileName = multipartFile.getOriginalFilename();

			Map<String, Object> map = new HashMap<String, Object>();
			map.put("MultipartFile_to_Base64", MultipartFile_to_Base64);
			map.put("fileName", fileName);
			map.put("fileByte", multipartFile.getSize());
			map.put("fileType", fileName.substring(fileName.lastIndexOf(".") + 1));

			try {
				sampleService.imageSend(map);

				result.put("returnImage", MultipartFile_to_Base64);
			} catch(Exception e) {
				result.put("errorMsg", "데이터 삽입중 에러발생!");
				e.printStackTrace();
			}
		} else {
			result.put("errorMsg", "파일이 존재하지 않음!");
		}

		return result;
	}
	
<br/>

만약 바닐라 스크립트 통신을 사용하였다면 RequestMapping 메서드 설정을 다음과 같이 수정한다.<br/>
응답데이터를 JSON 형태로 보내주기 위해서다.

		@RequestMapping(value = "/imageSend", method = RequestMethod.POST, produces = MediaType.APPLICATION_JSON_VALUE)

<br/><br/>

	<!-- 사용할 테이블의 구조 -->
	CREATE TABLE TB_FILE_DATA(
	    FILE_ID NUMBER(38) PRIMARY KEY,
	    FILE_DATA CLOB, -- 문자 대형 객체 ( 물리적 파일을 base64 형태로 인코딩한 문자 기반 데이터를 보관하기 위해 사용함 )
	    FILE_NAME VARCHAR2(200),
	    FILE_BYTE VARCHAR2(200),
	    FILE_TYPE VARCHAR2(10)
	);

<br/>
(MyBatis를 호출하는 service와 dao는 생략)

	<!-- MyBatis에서 사용할 동적 쿼리 -->
	<insert id="imageSend">
		INSERT INTO TB_FILE_DATA(FILE_ID, FILE_DATA, FILE_NAME, FILE_BYTE, FILE_TYPE)
		VALUES(
		       	(SELECT NVL(MAX(FILE_ID)+1, 1) FROM TB_FILE_DATA),
		       	#{MultipartFile_to_Base64, jdbcType=CLOB},
				#{fileName},
				#{fileByte, jdbcType=VARCHAR},
				#{fileType}
			  )
	</insert>

<br/>

![image](https://user-images.githubusercontent.com/66985977/189171531-b982c6b6-708d-4143-8d89-d0ee0159a3fa.png)
(실제 삽입된 데이터)

<br/>

![image](https://user-images.githubusercontent.com/66985977/189171808-e64f7d6a-45b4-47d2-942b-147215d7ff95.png)
(최종적으로 리턴된 base64형태의 데이터를 웹페이지에서 디코딩하여 화면에 보여줌)

<br/><br/>

<hr/>

# 물론 이 방법이 좋은방법일까? 


### 바이트 크기 문제

실제 파일을 등록할때와 base64 형태로 전환된 후 테이블에 저장했을떄 바이트 크기를 비교해보자.

![image](https://user-images.githubusercontent.com/66985977/189172722-715b111a-5a3d-4841-8bec-b1b99346dedf.png)
<br/>

실제 크기가 181,282바이트로 확인됐다. 그렇다면 테이블에 저장된 데이터 크기는?
<br/>

![image](https://user-images.githubusercontent.com/66985977/189174158-00bf67a2-67ae-40fd-83a9-38870f58d520.png)
<br/>
60,430 바이트나 차이가 난다. 이러한 결과를 봤을때 디스크 관리 측면에서는 물리적 형태로 관리하는것이 더 효율적이라고 보인다.

<br/><br/>

### 서버 과부화 문제

게시판에서 사용하려는 이미지일 경우 많은 사람들이 서버로 요청을 보낼것 이고 그로인해 트랜잭션 과부하가 일어날 수 있을것이다.<br/>
이때는 물리적 파일 형태의 경로를 호출하는 방식을 사용해야할 것이다.

<hr/>

# 해당 소스 적용중 발생할 수 있는 이슈

### 1. 디버깅이 되지 않음

![image](https://user-images.githubusercontent.com/66985977/189175117-d001108e-d776-47b7-bde2-992e04ae2d8f.png)
<br/><br/>
분명 선언한 form에 이미지를 append 하였음에도 변수에는 아무런 값이 들어있지 않다.<br/>
원인은 브라우저 정책이다. (참고 : https://stackoverflow.com/questions/17066875/how-to-inspect-formdata)
<br/><br/>
하지만 값을 확인할 수 있는 방법이 존재한다.
<br/><br/>

		for (var key of formData.keys()) { // 키 조회 // https://developer.mozilla.org/en-US/docs/Web/API/FormData/keys
		  console.log(key)
		}
		for (var value of formData.values()) { // 값 조회 // https://developer.mozilla.org/en-US/docs/Web/API/FormData/values
		  console.log(value);
		}
(꼭 디버깅을 해보고싶다면 이 방법을 이용하면됨.)

<br/><br/>

### 2. 파일을 들고 컨트롤러(서버)를 호출하기 위해선 MultipartHttpServletRequest를 사용해야 하지만 에러가 발생함
		<!-- Step1 -->
		<!-- 해당 내용을 pom.xml에 추가한다 -->
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3.3</version>
		</dependency>
			
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.2</version>
		</dependency>
		
		<!-- Step2 -->
		<!-- 해당 내용을 Servelt에 추가한다 -->
		<beans:bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver" >
			<beans:property name="maxUploadSize" value="-1" /> <!-- 최대 업로드 가능한 바이트 크기 ('-1'은 제한 없음) -->
			<beans:property name="maxInMemorySize" value="2048" /> <!-- 임시 파일을 생성하기 전에 메모리에 보관할 수 있는 최대 바이트 크기 -->
		</beans:bean>

<br/><br/>

### 3. 서버로 부터 에러가 반환되는 경우(500, 415)
		<!-- 해당 내용을 pom.xml에 추가한다 -->
		<!-- 파일 업로드에 대해서 필수적인 요소는 아니지만 원할한 데이터 통신을 위해 추가해준다. -->
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>2.9.5</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.dataformat</groupId>
			<artifactId>jackson-dataformat-xml</artifactId>
			<version>2.9.5</version>
		</dependency>
		
<br/>
<hr/>
<br/>

### base64 형태로 변환하지 않고 DB에 저장할수도 있다.

CLOB 데이터 타입에 파일을 이진객체로 담을 수 있다. 
