# HappyEver 여행플래너✈️
> ### 프로젝트 기간
> 2022년 5월 17일 ~ 2022년 7월 1일(6주)
> ### 참여 인원
> 7명
> ### 역할
> 조원
> ### 파트
> 메인페이지 및 비밀번호 찾기(이메일 임시비밀번호 발송), 총괄 취합, PPT 제작
> ### 기술 스택
> JSP, JAVA 8, ApacheTomcat 8.5v, Oracle DB, HTML5 & CSS3, JavaScript & JQuery, JSON, MyBatis, 카카오지도API & 한국관광공사 Tour API

<br><br></br>

## 1. 개요📌
![image](https://user-images.githubusercontent.com/101784768/181612074-573dc3ff-263d-4f6d-be04-7bbe8b70e1e5.png)

</br></br></br>

## 2. 화면 구현 및 ERD⚒️
![image](https://user-images.githubusercontent.com/101784768/181613599-5e534eb5-93d8-426d-ac33-a1eb5398d517.png)
![image](https://user-images.githubusercontent.com/101784768/181614439-50ef383b-e488-4d84-bf7a-5e9bd575b64e.png)

</br></br></br>

## 3. 주요 페이지✨
> ### 메인페이지
> 모든 페이지에 헤더와 푸터 연결</br>
> 로그인 시 JSTL 조건문을 통해 USERNAME을 받아 표시한다.</br>
> 검색창에 입력 시 실시간으로 해당 글자가 포함된 여행지가 즉각적으로 검색된다.</br>
> 비로그인 시 플래너 작성 기능 제한</br></br>
![image](https://user-images.githubusercontent.com/101784768/181620845-091c1eb8-8aef-434a-8d61-5a98c3a073ae.png)
![image](https://user-images.githubusercontent.com/101784768/181621042-50137df5-e6eb-4fd1-ae23-bdc3bd76702e.png)

</br></br>

> ### 로그인 & 비밀번호 찾기 페이지
> 헤더에서 모달창으로 로그인 페이지 구현</br>
> 로그인 성공 시 session에 정보 저장</br>
> 실패 혹은 validation 체크 통과 못할 시 안내문구 또는 alert 창으로 처리</br>
> 비밀번호 찾기 진행 시, DB에 저장된 이메일과 이름을 통해 검사 진행</br>
> 이메일과 이름이 저장되어 있다면, 임시비밀번호를 이메일로 발송하면서 DB의 비밀번호를 UPDATE 한다.</br></br>
![image](https://user-images.githubusercontent.com/101784768/181622478-b6beb130-295a-4a08-bab2-21d6491f02d3.png)
![image](https://user-images.githubusercontent.com/101784768/181622601-201b8a2d-1271-400d-a68e-97079099bf54.png)

</br></br>

> ### 여행 플래너 페이지
> 메인페이지에서 선택한 여행지를 지도 API에 띄우고, 한국관광공사 API를 통해 가져온 추천 여행지를 목록 및 지도 위에 보여준다.</br>
> 여행 날짜 입력 시 일수 만큼 좌측에 DAY가 생성되고, DAY마다 여행지 설정이 가능하다.</br>
> 각 DAY는 서로 독립적이다.</br></br>
![image](https://user-images.githubusercontent.com/101784768/181623081-192d6e1e-62ed-4256-bdbe-626481e96a4d.png)

</br></br>

> ### 게시판 페이지
> 여행 플래너 페이지에서 생성된 플래너를 볼 수 있는 게시판이다.</br>
> 여행 키워드를 클릭하거나 검색어를 입력하면 동적으로 변하도록 검색기능을 구현하였다.</br></br>
![image](https://user-images.githubusercontent.com/101784768/181625145-db485479-f58d-4b98-99b0-3b0b025e375d.png)

</br></br></br>

## 4. 핵심 코드🎯
> ### 임시비밀번호 이메일 전송 및 DB UPDATE 하기
> 네이버 메일의 POP3/SMTP 프로토콜을 이용해 임시비밀번호를 적은 메일을 발송한다.</br>
> 임시비밀번호 생성기를 통해 랜덤으로 생성된 비밀번호는 DB에 UPDATE 된다.</br></br>
> FindPwSendEmail.java
```java
package com.happy.app.login;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.Properties;
import java.util.Random;

import javax.mail.Message;
import javax.mail.PasswordAuthentication;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import com.happy.app.action.Action;
import com.happy.app.action.ActionForward;
import com.happy.app.dao.LoginDAO;

public class FindPwSendEmail implements Action {

	@Override
	public ActionForward execute(HttpServletRequest req, HttpServletResponse resp) {
		ActionForward forward = new ActionForward();
		LoginDAO ldao = new LoginDAO();
		
		String email = req.getParameter("email");
		String name = req.getParameter("name");
		
		// mail server 설정
		String host = "smtp.naver.com";
		String user = "anelune@naver.com"; // 자신의 네이버 계정
		String password = "fkspwbdkspfns";// 자신의 네이버 패스워드

		// 메일 받을 주소
		String to_email = ldao.checkEmail(email, name);

		// SMTP 서버 정보를 설정한다.
		Properties props = new Properties();
		props.put("mail.smtp.host", host);
		props.put("mail.smtp.port", 465);
		props.put("mail.smtp.auth", "true");
		props.put("mail.smtp.starttls.enable", "true");
		props.put("mail.smtp.ssl.enable", "true");
		props.put("mail.smtp.ssl.protocols", "TLSv1.2");

		// 임시 비밀번호 생성기
		StringBuffer temp = new StringBuffer();
		Random rnd = new Random();
		for (int i = 0; i < 10; i++) {
			int rIndex = rnd.nextInt(3);
			switch (rIndex) {
			case 0:
				// a-z
				temp.append((char) ((int) (rnd.nextInt(26)) + 97));
				break;
			case 1:
				// A-Z
				temp.append((char) ((int) (rnd.nextInt(26)) + 65));
				break;
			case 2:
				// 0-9
				temp.append((rnd.nextInt(10)));
				break;
			}
		}
		String AuthenticationKey = temp.toString();
		System.out.println(AuthenticationKey);
		
		ldao.updatePw(email, AuthenticationKey);

		Session session = Session.getDefaultInstance(props, new javax.mail.Authenticator() {
			protected PasswordAuthentication getPasswordAuthentication() {
				return new PasswordAuthentication(user, password);
			}
		});
		
		// email 전송
		try {
			MimeMessage msg = new MimeMessage(session);
			msg.setFrom(new InternetAddress(user, "HappyEver"));
			
			msg.addRecipient(Message.RecipientType.TO, new InternetAddress(to_email));

			// 메일 제목
			msg.setSubject("안녕하세요 HappyEver 입니다.");
			// 메일 내용
			msg.setText("임시비밀 번호는 " + temp + " 입니다. \n임시비밀번호로 로그인 후 꼭 비밀번호를 바꿔주세요.");

			Transport.send(msg);
			System.out.println("이메일 전송");
			
			PrintWriter out = resp.getWriter();
			forward.setPath("/main.jsp?flag=true");
		
		} catch (Exception e) {
			e.printStackTrace();
		}
		
		HttpSession saveKey = req.getSession();
		saveKey.setAttribute("AuthenticationKey", AuthenticationKey);

		return forward;
	}
}
```

</br></br>

> ### 한국관광공사 API를 이용해 받아온 JSON 형식의 데이터를 테이블에 INSERT 하기
> API를 통해 JSON 형식으로 받아온 데이터 중 필요한 정보를 추려 LIST 객체에 담아 Parameter로 넘긴다.</br>
> 넘어온 데이터를 DB 테이블과 동일한 순서와 타입으로 세팅해 INSERT 한다.(MVC2 방법 사용)</br></br>
> tourist.js
```jQuery
$(function(){
	getJson();
});

// json을 읽어오는 함수
function getJson() {
	// Seoul.json에서 데이터를 가져옴 -> function(data)로 처리 -> data에 저장
	$.getJSON("areaBasedList.json", function(data) {
		$.each(data, function(key, val) {
			if( key == "SID" ) {
				$("table").attr("border", "1");
				
				// API에서 받아 올 데이터들
				$("thead").append(
					"<tr>"+
						"<th>"+val.ADDR1+"</th>"+	
						"<th>"+val.AREACODE+"</th>"+	
						"<th>"+val.SIGUNGUCODE+"</th>"+
						"<th>"+val.FIRSTIMAGE2+"</th>"+	
						"<th>"+val.READCOUNT+"</th>"+	
						"<th>"+val.TITLE+"</th>"+	
					"</tr>"
				);
			} else {
				var list = val;		// list 변수 : 배열로 정의
				for( var i=0; i<list.length; i++ ) {
					var str = list[i];		// str 변수 : list 배열 안에 있는 하나의 json 데이터
					$("tbody").append(
						"<tr>"+
							"<td>"+str.addr1+"</td>"+
							"<td>"+str.areacode+"</td>"+
							"<td>"+str.sigungucode+"</td>"+
							"<td>"+str.firstimage2+"</td>"+
							"<td>"+str.readcount+"</td>"+
							"<td>"+str.title+"</td>"+
							// db에 값들을 ';'로 나눠서 넘겨줌
							"<input type='hidden' name='list' value='"+
							str.addr1+";"+str.areacode+";"+str.sigungucode+";"
							+str.firstimage2+";"+str.readcount+";"+str.title+"'>"+
						"</tr>"
					);
				}
			}
		});
	});
}
```

</br>

> AddTourlistAction.java
```java
package com.happy.app.action;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import com.happy.app.action.Action;
import com.happy.app.action.ActionForward;
import com.happy.app.dao.TourlistDAO;
import com.happy.app.dto.TourlistDTO;

public class AddTourlistAction implements Action {

	@Override
	public ActionForward execute(HttpServletRequest req, HttpServletResponse resp) {
		System.out.println("AddTourlistAction 도착");
		ActionForward forward = new ActionForward();
		TourlistDAO tdao = new TourlistDAO();
		
//		tdao.delete();
		String[] Tourlist = req.getParameterValues("list");
		
		List<TourlistDTO> tdtos = new ArrayList<TourlistDTO>();
		
		for( int i = 0; i < Tourlist.length; i++ ) {
			String[] tmp = Tourlist[i].split(";");
			System.out.println(Arrays.toString(tmp));
			
			TourlistDTO tdto = new TourlistDTO(tmp[0], Integer.parseInt(tmp[1]), 
                                                    Integer.parseInt(tmp[2]), tmp[3], Integer.parseInt(tmp[4]), tmp[5]);
			
			tdtos.add(tdto);
		}
		
		tdao.AddTourlist(tdtos);
		
		forward.setRedirect(true);
		forward.setPath( req.getContextPath() + "/tour/tourlist.jsp");
		
		return forward;
	}
}
```

</br></br></br>

## 5. 이슈 및 트러블슈팅
> ### 이슈
> 이번 프로젝트의 가장 큰 이슈는 시간 부족이었다.</br>
> 여행플래너의 작성 부분이 이번 프로젝트의 핵심 기능이었던 만큼 많은 공을 들였다.</br>
> 하드코딩이 아닌 코딩을 하려고 노력했었고, 레퍼런스로 삼았던 홈페이지의 기능을 최대한 구현하고자 했다.</br>
> 하지만 공공 API의 데이터를 가지고 와 출력하는 기능, 지도 API에 관광지를 모두 표시하는 기능 등 까다로운 부분이 많았고</br>
> 선택한 여행지들을 DB에 다시 저장하는 것 역시 쉽지 않았다.</br>
> 때문에 프로젝트 발표 직전까지 수정에 수정을 거듭해야 했다.</br>
> 이 이슈를 겪으면서, 프로젝트에 있어 무엇보다 중요한 것은 완성이라는 것을 느꼈다.</br>
> 사회생활 경험을 돌이켜보면, 비즈니스에는 기한이 정해져 있고 그 시간 안에 완성도 높은 서비스를 제공해야 했다.</br>
> 프로젝트 역시 하드코딩을 하더라도 최대한 완성도를 높이는 것이 중요하지 않았을까 싶다.</br>

</br>

> ### 트러블슈팅
> 임시비밀번호를 이메일로 발송하는 것은 보다 완성도 높은 홈페이지를 위한 도전이었다.</br>
> 처음 접하는 프로토콜들에 눈이 뱅글뱅글 돌았다.</br>
> 인터넷을 뒤져 가장 기능할 수 있는 코드를 클론코딩했지만 제대로 작동하지 않았다.</br>
> 코드를 수정하면 다른 에러가 뜨고 또 다시 다른 에러가 뜨는 과정이 반복됐다.</br>
> 방화벽을 열고 프로토콜을 등록하기도 하고, IMAP 프로토콜을 가져다 썼다가 다시 POP3로 돌아오기도 했다.</br>
> 최종적으로 SSL의 TLS 버전을 수정해주고서야 메일이 발송되었다🎉🎉🎉</br>

</br></br></br>

## 6. 한줄평
> ### 해피하지만 아쉬웠던 프로젝트
> 7명이라는 많은 사람들이 모인 것치고는 HappyHajo라는 조이름처럼 해피하게 진행되었던 프로젝트였다.</br>
> 하지만 구현하지 못한 기능들이 있어 아쉬움이 많이 남는다. </br>
