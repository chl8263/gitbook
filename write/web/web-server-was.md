# Web server, WAS 의 이해

###  **Web Server란?**

  웹 클라이언트로부터 HTTP Request를 받아 Static file\(html, css, js\)을 제공하는 프로그램이다. ex\) Apache, nginx

1. 정적인 컨텐츠 제공  
   * WAS를 거치지 않고 바로 자원을 제공
2. 동적인 컨텐츠 제공을 위한 요청 전달
   * 클라이언트 요청을 WAS에 보내고, WAS가 처리한 결과를 client에게 response한다.

###  **WAS란?**

  WAS\(Web Application Server\)란 DB 조회나 다양한 로직 처리를 요구하는 dynamic 컨텐츠를 제공하기 위해 만들어진 application Server이다.  HTTP를 통해 컴퓨터나 장치에 애플리케이션을 수행해주는 **미들웨어**이다.

**웹 컨테이너** \(Web container\) 혹은 **서블릿 컨테이너** \(Servlet Container\)라고도 불리운다.

* 웹 컨테이너는 JSP, Servlet을 실행시킬 수 있는 소프트웨어를 말한다.
* WAS는 JSP, Servlet 구동 환경을 제공한다.

#### WAS의 역할

  WAS = Web Server + Web Container

  웹서버의 기능들을 구조적으로 분리하여 처리하고자 하는 목적으로 생긴 개념이다.

* 분산 트랜잭션, 보안, 메시징, 쓰레드 처리 등의 기능을 처리하는 분산 환경에서 사용된다.
* 주로 DB 서버와 같이 수행된다.

  현재는 WAS가 가지고 있는 WebServer도 정적인 컨텐츠를 처리하는 데 있어서 성능상 큰 차이가 없다.

#### WAS의 기능

* 프로그램 실행 환경과  DB접속 기능 제공
* 여러개의 트랜잭션 관리 기능
* 업무를 처리하는 비지니스 로직 수행

#### WAS의 예

  Apache Tomcat, JBoss, Jeus, Web Sphere



