# Nginx

## 경량 웹서버

- 정적 파일 응답 web server
- Reverse Proxy Server → WAS 서버의 부하를 줄일 수 있는 로드 밸런서로 활용

## 동작 흐름

Event - Driven

비동기 방식으로 요청을 처리 → Concurrency 하게 처리할 수 있다.

![Nginx%20711803488f4a412c9c7dfd8800343524/_2021-06-09__2.43.17.png](Nginx%20711803488f4a412c9c7dfd8800343524/_2021-06-09__2.43.17.png)

프로세스와 스레드를 생성하지 않아도 됨 → 적은 자원으로도 효율적인 운용이 가능

## 구조

하나의 Master Process + 여러개의 Worker Process 구성되어 실행

Master Process : 설정 파일 읽고, 유효성 검사, Woker Process 관리

Worker Process : 모든 요청 처리

Nginx는 이벤트 기반 모델을 사용 → Worker Process 사이에 요청을 효율적으로 분배하기 위해 OS에 의존적인 메커니즘을 사용 

Worker Process의 개수는 설정 파일에서 정의, 정의된 프로세스 개수와 사용 가능한 CPU 코어 숫자에 맞게 자동으로 조정

![Nginx%20711803488f4a412c9c7dfd8800343524/_2021-06-09__2.53.03.png](Nginx%20711803488f4a412c9c7dfd8800343524/_2021-06-09__2.53.03.png)

## 설정

### 1. Directives, Blocks, Contexts

/etc/nginx/nginx.conf 

Directives

- Blocks or Contexts
- user, worker_processes, error_log, pid → 메인 컨텍스트, 어떠한 블록이나, 컨텍스트에 포함되지 않음

### 2. HTTP Block

http 블록은 웹 트래픽을 처리하는 디렉티브들을 담고있음

Universal → nginx가 제공하는 모든 웹 Configuration에 전달

### 3. Server Block

```java
server {
    listen         80 default_server;
    listen         [::]:80 default_server;
    server_name    example.com www.example.com;
    root           /var/www/example.com;
    index          index.html;
    try_files $uri /index.html;
}
```

### 4. location block

리소스에 대한 요청을 어떻게 응답할지 설정

리터럴 스트링에 대한 매치

root 디렉토리 위치

index 제공 되는 파일

![Nginx%20711803488f4a412c9c7dfd8800343524/_2021-06-09__3.38.40.png](Nginx%20711803488f4a412c9c7dfd8800343524/_2021-06-09__3.38.40.png)

```java
location / {
        proxy_pass http://localhost:8081;
				//요청을 이 주소로~
        proxy_set_header X-Forwarded-For $remote_addr;
				//HTTP 프록시나 로드 밸런서를 통해 웹 서버에 접속하는 클라이언트의 원 IP 주소를 식별하는 사실상의 표준 헤더
        proxy_set_header X-Real-IP $remote_addr;
				// 위랑 비슷한데 변조 불가능
        proxy_set_header Host $http_host;
				//HTTP Request 의 Host 헤더값
  }
```

```java
location / {
    try_files $uri $uri/ /test/index.html;
		// ex ) images/test.jpeg 가 있는지 확인 -> 없으면 images/ 에서 먼저 요청된게 있는지 확인 -> 없으면 /test/index.html 제공 -> 없으면 오류
}
```