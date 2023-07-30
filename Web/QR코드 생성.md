# QR코드 생성

## 구글에서 제공하는 QR CODE 라이브러리 zxing를 사용

### 의존성 추가 
```
<!-- https://mvnrepository.com/artifact/com.google.zxing/core -->
<!-- 구글바코드 오픈소스 -->
<dependency>
	<groupId>com.google.zxing</groupId>
	<artifactId>core</artifactId>
	<version>3.1.0</version>
/dependency>
<!-- https://mvnrepository.com/artifact/com.google.zxing/javase -->
<dependency>
	<groupId>com.google.zxing</groupId>
	<artifactId>javase</artifactId>
	<version>3.1.0</version>
</dependency>
```

## 이슈사항

### Can't create cache file! Error 발생 <br/><br/>

조치<br/>
ImageIO.write 를 할때 cache 사용 여부를 명시적으로 설정하지 않았을때 디폴트 값은 false이기 때문에<br/>
ImageIO.setUseCache(false); 를 통해 cache 사용을 하지 않음을 명시적으로 설정한다.<br/>
