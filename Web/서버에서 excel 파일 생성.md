# 서버에서 엑셀 파일을 생성하고 읽을 수 있다.

## 라이브러리 가져오기 (Maven)


```

<!--apachi poi - excel file read and write lib-->
<dependency>
	<groupId>org.apache.poi</groupId>
	<artifactId>poi</artifactId>
	<version>5.0.0</version>
</dependency>
<dependency>
	<groupId>org.apache.poi</groupId>
	<artifactId>poi-ooxml</artifactId>
	<version>5.0.0</version>	
</dependency>
	
```

artifactId - poi : HSSFWorkbook 사용가능 <br/>
	- HSSF - Excel 97(-2007) 파일 포맷을 사용할 때 사용 , ex) HSSFWorkbook, HSSFSheet <br/>
	- 엑셀 97 - 2003 까지 <br/> <br/>
artifactId - poi-ooxml : XSSFWorkbook 사용가능 <br/>
	- XSSF - Excel 2007 OOXML (.xlsx) 파일 포맷을 사용할 때 사용 , ex) XSSFWorkbook, XSSFSheet <br/> 
	- 엑셀 2007 이상 <br/> <br/>

### Maven을 사용하지 않을 경우 다음 URL에서 수동으로 다운로드 <br/> <br/>

poi : https://mvnrepository.com/artifact/org.apache.poi/poi <br/><br/>
poi-ooxml : https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml <br/>

## Excel Download
	
### Server - Poi Excel Write

```
	
	package org.util;

	import java.io.IOException;
	import java.io.OutputStream;
	import java.util.ArrayList;
	import java.util.HashMap;
	import java.util.List;
	import java.util.Map;

	import javax.servlet.http.HttpServletResponse;

	import org.apache.poi.ss.usermodel.CellStyle;
	import org.apache.poi.ss.usermodel.Font;
	import org.apache.poi.xssf.usermodel.XSSFCell;
	import org.apache.poi.xssf.usermodel.XSSFRow;
	import org.apache.poi.xssf.usermodel.XSSFSheet;
	import org.apache.poi.xssf.usermodel.XSSFWorkbook;

	public class ApachePoiUtil {
		public void ExcelDownload(HttpServletResponse response) throws Exception {
			XSSFWorkbook xssfWorkbook = ExcelWrite();

			// 엑셀 파일명 설정
			final String fileName = "userList.xlsx";
			response.setHeader("Content-Disposition", "attachment;filename=" + fileName);
			response.setContentType("application/vnd.ms-excel;charset=EUC-KR");


			OutputStream outputStream = response.getOutputStream();
			try {
				xssfWorkbook.write(outputStream);
			} catch (IOException e) {
				e.printStackTrace();
			} finally {
				if(outputStream != null) outputStream.close();
				response.getOutputStream().flush();
				response.getOutputStream().close();
			}
		}

		public XSSFWorkbook ExcelWrite() {
			@SuppressWarnings("resource") // 주의 억제
			XSSFWorkbook workbook = new XSSFWorkbook();


			Map<String, String> map = new HashMap<String, String>();
			List<Map<String, String>> listData = new ArrayList<Map<String, String>>();

			map.put("userName", "홍길동");
			map.put("userAge", "20");
			map.put("address", "서울시");
			listData.add(map);

			map = new HashMap<String, String>();
			map.put("userName", "김길동");
			map.put("userAge", "25");
			map.put("address", "부산시");
			listData.add(map);

			map = new HashMap<String, String>();
			map.put("userName", "강길동");
			map.put("userAge", "23");
			map.put("address", "충북");
			listData.add(map);

			if (listData != null && listData.size() > 0) {
				/* 엑셀 그리기 */
				final String[] colNames = {"No", "성명", "나이", "거주지"};

				// 헤더 사이즈
				final int[] colWidths = {3000, 5000, 5000, 3000};

				XSSFSheet sheet = null;
				XSSFCell cell = null;
				XSSFRow row = null;

				// Font
				Font fontHeader = workbook.createFont();
				fontHeader.setFontName("맑은 고딕"); // 글씨체
				fontHeader.setFontHeight((short) (9 * 20)); // 사이즈
				Font font9 = workbook.createFont();
				font9.setFontName("맑은 고딕"); // 글씨체
				font9.setFontHeight((short) (9 * 20)); // 사이즈
				// 엑셀 헤더 셋팅
				CellStyle headerStyle = workbook.createCellStyle();
				headerStyle.setFont(fontHeader);
				// 엑셀 바디 셋팅
				CellStyle bodyStyle = workbook.createCellStyle();
				bodyStyle.setFont(font9);
				// 엑셀 왼쪽 설정
				CellStyle leftStyle = workbook.createCellStyle();
				leftStyle.setFont(font9);

				// rows
				int rowCnt = 0;
				int cellCnt = 0;
				int listCount = listData.size();

				// 엑셀 시트명 설정
				sheet = workbook.createSheet("사용자현황");
				row = sheet.createRow(rowCnt++);
				// 헤더 정보 구성
				for (int i = 0; i < colNames.length; i++) {
					cell = row.createCell(i);
					cell.setCellStyle(headerStyle);
					cell.setCellValue(colNames[i]);
					sheet.setColumnWidth(i, colWidths[i]); // column width 지정
				}
				// 데이터 부분 생성
				for (Map<String, String> mapDt : listData) {
					cellCnt = 0;
					row = sheet.createRow(rowCnt++);
					// 넘버링
					cell = row.createCell(cellCnt++);
					cell.setCellStyle(bodyStyle);
					cell.setCellValue(listCount--);
					// 성명
					cell = row.createCell(cellCnt++);
					cell.setCellStyle(bodyStyle);
					cell.setCellValue(mapDt.get("userName"));
					// 나이
					cell = row.createCell(cellCnt++);
					cell.setCellStyle(bodyStyle);
					cell.setCellValue(mapDt.get("userAge"));

					// 주소
					cell = row.createCell(cellCnt++);
					cell.setCellStyle(bodyStyle);
					cell.setCellValue(mapDt.get("address"));
				}
			}

			return workbook;
		}
	}

```

### ExcelDownload Controller

```

	@RequestMapping(value = "/downloadPoi", method = RequestMethod.GET)
	public void downloadPoi(HttpServletResponse response) throws  Exception {
		ApachePoiUtil apachePoiUtil = new ApachePoiUtil();
		apachePoiUtil.ExcelDownload(response);
	}

```

### ExcelDownload View(Jsp)

```

	<%@ page contentType="text/html;charset=UTF-8" language="java" %>
	<html>
	<head>
	    <title>Title</title>

	</head>
	<body>
	    <input id="DownloadBtn" type="button" value="엑셀 다운로드">
	</body>
	<script>
	    document.getElementById("DownloadBtn").onclick = function() {
		location.href = "/downloadPoi";
	    }
	</script>
	</html>

```

VIEW 에서 GET방식으로 파일을 요청하면 Write 한 file을 Response 받아 local에 저장할 수 있다.<br/>

![image](https://user-images.githubusercontent.com/66985977/192105864-e195012c-8c5d-409f-9b1f-4dc6024f0e36.png) <br/><br/>

![image](https://user-images.githubusercontent.com/66985977/192105894-51df5ea2-be72-4181-8f78-f24fe07074da.png) <br/><br/>

![image](https://user-images.githubusercontent.com/66985977/192107047-6b4ce4bb-6036-4dc5-85b1-5b2bc432c4ac.png) <br/><br/>

한글 깨지는 이유좀..

## 엑셀 파일 내용중 한글이 깨질때

한글이 깨지는 이유는 매우 다양하다 IDE 설정문제, response에 header 설정 문제 등
<br/>
본인은 이클립스에서 문제가 없었던 소스를 인텔리J에서 리뷰중 해당 원인이 발생하였고,<br/>
Response Header 정보를 수정하여도 계속 문제가 발생하였다.<br/>
결국 원인은 IDE 설정 문제였다. 해당 클래스 파일 인코딩이 UTF-8이 아니었다.. 그렇기에 사진과 같이 수정하였음.<br/>

![image](https://user-images.githubusercontent.com/66985977/192131247-bd45fa2a-3dd1-4e64-8d9b-eb1c80305a59.png) <br/>
<br/>
관련 이슈로 아무리 서치를 해도 관련 사례가 나오지 않으며, 파일 인코딩 중요성에 대해서 항상 숙지하고 있어야 한다.<br/>
<br/>
Apache poi 3.0.0 이상부터는 Sheet에 대해서 별도로 인코딩 설정을 할 필요가 없다.<br/>
<br/>
![image](https://user-images.githubusercontent.com/66985977/192131609-5f6d90c6-9842-4d17-b4de-12c36ad6918b.png)
~~5시간 삽질~~
