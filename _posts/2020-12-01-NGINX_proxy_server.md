---
title     : "NGINX를 이용한 프록시 (forward proxy) 서버 구축 및 파이썬 활용"
categories: 
  - web
toc       : true
toc_sticky: true
layout    : single
classes   : custom_wide
---

IP 우회 또는 내부망을 외부망과 연결하는데 필요한 프록시 서버의 구축 방법을 소개한다.

## NGINX 설치

NGINX는 Appache와 같은 웹 서버 애플리케이션중에 하나이다. 가볍고 사용법이 간편하여 NGINX를 사용해 프록시 서버를 구현하려 한다. 설치 환경은 ubuntu (16.04 또는 18.04)에서 진행한다.

### Install Dependencies and Source File

먼저 아래의 dependency들을 설치한다.

- ```
    sudo apt-get update
    sudo apt-get install gcc
    sudo apt-get install cmake
    sudo apt-get install libpcre3 libpcre3-dev
    sudo apt-get install zlib1g zlib1g-dev
    ```

위의 dependency들을 다 받았으면, wget 명령어를 통해 source file을 받아준다. 추가로 proxy 모듈을 위해 아래와 같이 git 저장소 또한 받아준다.

- ```
    wget http://nginx.org/download/nginx-1.17.0.tar.gz
    tar -xzvf nginx-1.17.0.tar.gz
    git clone https://github.com/chobits/ngx_http_proxy_connect_module.git
    cd nginx-1.17.0/
    patch -p1 < ../ngx_http_proxy_connect_module/patch/proxy_connect_rewrite_101504.patch
    ./configure --add-module=../ngx_http_proxy_connect_module
    sudo make && sudo make install
    ```

### Set Command and Service File

위에서 nginx를 설치했으면, 커맨드 실행을 위해 아래와 같이 커맨드를 등록할 수 있다.

- ```
    cd /usr/bin
    sudo ln -s /usr/local/nginx/sbin/nginx nginx
    ```

이제 system 관리자를 통해 nginx를 다룰 수 있도록 설정 파일을 정의해야한다.
아래와 같은 명령어를 설정 파일을 연다.

- `sudo nano /etc/systemd/system/nginx.service`

그 다음 아래의 내용을 기입해 저장한다.

- ```
    [Unit]
    Description=nginx - high performance web server
    Documentation=https://nginx.org/en/docs/
    After=network-online.target remote-fs.target nss-lookup.target
    Wants=network-online.target

    [Service]
    Type=forking
    ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
    ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
    ExecReload=/bin/kill -s HUP $MAINPID
    ExecStop=/bin/kill -s TERM $MAINPID

    [Install]
    WantedBy=multi-user.target
    ```

위에서 `ExecStartPre` 와 같이 경로를 지정하는 부분에서는 실제 nginx가 설치된 위치로 바꿔주어 적용할 수 있다.

그 뒤에 아래의 명령어로 system을 통해 nginx 등록 후, 서비스를 켜고 끌 수 있다.

- ```
    # register service
    sudo systemctl enable nginx.service
    # start NGINX
    sudo systemctl start nginx.service

    # check NGINX is running 
    sudo systemctl status nginx.service
    # or
    ps aux | grep nginx
    # or
    netstat -tlnp

    # to restart
    sudo systemctl restart nginx.service
    ```

### Set NGINX configuration File

위에서 nginx를 가동하였다. 이제 nginx.conf 파일을 이용해 우리가 원하는 프록시 (forward proxy) 서버가 동작되게끔 설정을 바꿀 것이다. 다른 서버설정에 비해 매우 간단하여 빠르게 구축할 수 있다.

먼저 아래의 명령어를 통해 nginx.conf 파일을 연다.

- ```
    cd /usr/local/nginx/conf
    sudo nano nginx.conf
    ```

그 후 아래의 내용을 기입한다.

- ```
    worker_processes  1;
    events {
        worker_connections  1024;
    }
    http {
        keepalive_timeout  65;
        server {
            listen       1234; # 열고싶은 포트
            server_name  "1.2.3.4"; # 프록시 서버의 도메인 또는 IP 주소
            access_log  logs/access.log;
            # dns resolver used by forward proxying
            resolver                       8.8.8.8;

            # forward proxy for CONNECT request
            proxy_connect;
            proxy_connect_allow            443 563;
            proxy_connect_connect_timeout  10s;
            proxy_connect_read_timeout     10s;
            proxy_connect_send_timeout     10s;
            location / {
                proxy_pass $scheme://$http_host$uri$is_args$args;
            }
        }
    }
    ```

그 뒤에 다음 명령어를 통해 conf 파일에 오류가 없는지 확인할 수 있다.

- `sudo nginx -t`

만약 "OK" 라는 문구가 포함된 2줄의 출력문이 나온다면 오류가 없는 것이다. 오류가 없음을 확인 했으면, 아래와 같이 nginx 서비스를 재시작한다. 그렇게되면 프록시 서버 구축이 완료된다.

- `sudo systemctl restart nginx.service`

### Set Firewall Configuration on Google Cloud Platform (GCP)

만일 Google Cloud Platform (GCP)나 Amazon Web Service (AWS)를 사용해 프록시 서버를 구축한다면, 위에서 설정한 프록시 서버의 포트 번호를 방화벽에서 열어줘야한다. GCP의 경우, 아래와 같이 설정할 수 있다.

- ![1](/assets/images/NGINX_proxy_server/1.png)

- GCP console 페이지의 compute engine 탭에 들어가면 위와 같은 항목들이 있다. 이중에 "방화벽 규칙 설정"을 클릭한다.

- ![2](/assets/images/NGINX_proxy_server/2.png)

- 그럼 "방화벽 규칙 만들기"를 들어간 후,

- ![3](/assets/images/NGINX_proxy_server/3.png)

- 위와 같이 규칙을 생성한다.

### Test the proxy server with python

아주 간단하게 **requests**
 모듈을 사용하여 프록시 서버작동을 테스트할 수 있다.

- ``` python
    import requests

    url = "원하는도메인"
    proxies = {
    'http': 'http://"프록시서버의 도메인이름 또는 IP주소":"설정한 포트번호"',
    'https': 'http://"프록시서버의 도메인이름 또는 IP주소":"설정한 포트번호"',
    }
    res = requests.get(url, proxies=self.proxies)
    ```
