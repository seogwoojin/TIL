# `디스패처 서블릿`

## 서블릿

디스패처 서블릿을 알기 전에 서블릿은 뭘까?

고대 시절 자바로 WAS를 만들 때 서블릿을 이용했다고 한다.

``` 
package com.servlet;

import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Servlet implementation class servletBlog1
 */
@WebServlet("/servletBlog1") //어노테이션으로 서블릿의 이름을 표시하며, JSP파일에서 접근하기 위한 이름이다.
public class servletBlog1 extends HttpServlet {
	private static final long serialVersionUID = 1L;
       
    /**
     * @see HttpServlet#HttpServlet()
     */
    public servletBlog1() {
        super();
        // TODO Auto-generated constructor stub
    }

	/**
	 * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		System.out.println("doGet call!!!!");
//		1. data get
		String name = request.getParameter("username");
		String id = request.getParameter("userid");
		int area = Integer.parseInt(request.getParameter("area"));
		
//		2. logic >> 인사의 글자색을 서울 : 파랑, 대전 : 오렌지, 구미 : 핑크, 광주 : 초록, 부울경 : 보라
		String color[] = {"blue", "orange", "pink", "green", "purple"};
		
//		3. response page
		response.setContentType("text/html;charset=utf-8");
		PrintWriter out = response.getWriter();
		out.println("<html>");
		out.println("	<body>");
		out.println("	<h2 style=\"color:" + color[area] + "\">안녕하세요. " + name + "(" + id + ")님!!!</h2>");
		out.println("	</body>");
		out.println("</html>");
	}
	/**
	 * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		//POST방식 코드 작성
	}

}
```

서블릿의 구현 코드는 대략 이런 식이다. 모듈화는 찾아보기 힘들며, Html을 직접 만들어내는 모습을 볼 수 있다.

이렇게 만든 여러 서블릿들을 URI에 따라 매핑해주는 역할을 서블릿 컨테이너가 한다. (톰캣, 제티)

<br>
개발자들은 아무래도 이런 구조를 만족스럽게 보지 않았고, Response를 만드는 부분과 Request를 핸들링하고 파라미터를 매핑하는 과정을 모두 쪼갰다. (MVC 등장)


## 디스패처 서블릿

스프링이 등장하고는 우리는 이 서블릿에 대해 알 필요가 줄어들었습니다. 
더 이상 HttpServlet의 Request, Response에 신경 쓰지 않고, 순수 자바 코드와 클래스 만으로 내부 로직을 설계하는 데 집중할 수 있게됐습니다.

<img width="734" alt="image" src="https://github.com/user-attachments/assets/8e329bd8-dd36-4750-a526-aa5b12be7630" />

어떤 URI를 어떤 컨트롤러 빈에 할당할 지, 어떤 파라미터나 바디 객체가 필요한 지 디스패처 서블릿이 판단하고 매핑합니다. 

<br>

![image](https://github.com/user-attachments/assets/dbf0aac2-db57-42c0-9bdf-8b6a5d7fa9ff)

디스패처 서블릿은 가장 큰 단위고, + 컨트롤러 빈은 서비스나 레포지토리의 빈과 조금 다른 취급을 받는 것 같습니다. 
(핵심은 역전 호출은 불가능한 구조 인듯 합니다. 서비스 빈 -> 컨트롤러 빈 호출 같은)

<br>

정리하면 다음과 같은 구조일 것 같습니다. 

- 라우팅을 통해서 바이트 스트림 형태로 톰캣까지 요청이 전달 (HttpServletRequest 객체로 매핑)
- (Filter를 거침)
- 톰캣이 유휴 디스패처 서블릿 중 하나를 꺼내서 요청 전달
- 디스패처 서블릿이 Handler Mapping을 통해서 Request 정보에 맞는 컨트롤러의 메소드를 발견
- 발견한 메소드의 Argument 정보를 ArgumentResolver를 통해서 주입
- (Interceptor를 거침)
- 컨트롤러 빈의 메소드를 정보를 담아 호출


# Reference

토비의 스프링부트 3.0
https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet.html
