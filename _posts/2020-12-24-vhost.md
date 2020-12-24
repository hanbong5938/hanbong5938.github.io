#인증서 발급 - 도커 사용

```
docker run -it --rm --name certbot \
  -v '/etc/letsencrypt:/etc/letsencrypt' \
  -v '/var/lib/letsencrypt:/var/lib/letsencrypt' \
  certbot/certbot certonly -d 'example.com' --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```
- -it : interactive + tty. 터미널을 통해 컨테이너와 상호작용 위해 필요
- -rm : 컨테이너 종료시 삭제
- -v '...' : 인증서가 저장될 경로를 볼륨으로 설정
- -d 'example.com' : 인증서를 발급받을 도메인
- -manual --preferred-challenges dns : DNS를 통해 도메인을 인증
- -server https://~~~ : 발급을 요청할 let's encrypt 서버 주소

이메일 주소 입력 및 약관 확인 동의

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.example.com with the following value:

코드

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

도메인의 txt 레코드 설정
``` 
호스트 이름 : _acme-challenge.example.com 
레코드 값 : 코드
```

이후 적용 되었는지 확인
``` 
dig txt _acme-challenge.example.com
```

ANSWER SECTION 부분에 설정 값이 출력 되는 것을 확인한 후 진행, 발급 완료시 아래 디렉토리에 생성
```
/etc/letsencrypt/live/example.com/
```

#vhost 설정


1. Apache의 SSL 모듈 활성화

```
sudo a2enmod ssl
```

2.http -> https 리다이렉트를 위하여 설정파일 생성
```
/etc/apache2/sites-available/http설정파일.conf
```

http설정파일.conf 내용
```
<VirtualHost *:80>
	RewriteEngine On
	RewriteCond %{HTTPS} !on
	RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R,L]

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

```


3. 각 도메인 별로 /etc/apache2/sites-available/에 설정 파일을 만든다.
ProxyPass에는 로컬에서 실행 중인 서버포트 입력해준다.
(로컬에서 https 사용하면 에러 발생)
(NameVirtualHost는 인증서에 등록된 도메인과 같아야 여러개 호스트 사용가능)
```
<VirtualHost *:443>
	NameVirtualHost example.com:443
	ServerName example.com

	SSLProxyEngine on
	ProxyPass / http://localhost:51000/
	ProxyPassReverse / http://localhost:51000/

	#자체인증서 사용 하는 경우
	#SSLProxyCheckPeerExpire off

	SSLEngine on
    	SSLCertificateFile /etc/letsencrypt/live/example.com/cert.pem
    	SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem


	ErrorLog ${APACHE_LOG_DIR}/error.log

</VirtualHost>

```

4. 사용하려는 설정 파일들을 아파치에서 사용하도록 등록

```
a2ensite example-ssl.conf
```

5. /etc/apache2/에 있는 ports.conf에 NameVirtualHost *:443 추가
```
Listen 80

<IfModule ssl_module>
	Listen 443
    NameVirtualHost *:443       
</IfModule>

<IfModule mod_gnutls.c>
	Listen 443
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

```

6. 아파치 재시작
```
service apache2 restart
```

#spring boot 위한 p12 추가

OOOO에는 server.ssl.key-alias 와 같은 이름으로 설정
패스워드입력창이 이후 나오면 spring properties와 맞춰준다.
```
openssl pkcs12 -export -in fullchain.pem  -inkey privkey.pem  -out keystore.p12 -name OOOO  -CAfile chain.pem  -caname root

```





#인증서 3개월 후 재발급
```
docker run -it --rm --name certbot \
  -v '/etc/letsencrypt:/etc/letsencrypt' \
  -v '/var/lib/letsencrypt:/var/lib/letsencrypt' \
  certbot/certbot renew --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```
