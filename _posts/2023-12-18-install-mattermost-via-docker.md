---
layout: post
title: Install Mattermost via Docker
categories: [DOCKER,MATTEROMOST]
excerpt: Docker로 Mattermost설치하기
---

[공식Guide](https://docs.mattermost.com/install/install-docker.html#deploy-mattermost-on-docker-for-production-use)

Ubuntu 기준으로 작성하였습니다.  

mattermost 디렉터리를 생성 후 mattermost git 저장소를 clone 합니다.  
``` bash
mkdir mattermost
cd mattermost 
git clone https://github.com/mattermost/docker .
```

`env.example`를 `.env`로 복사합니다.  
``` bash
cp env.example .env
```

필수 디렉토리를 생성 하고 권한을 부여합니다.  
``` bash 
mkdir -p ./volumes/app/mattermost/{config,data,logs,plugins,client/plugins,bleve-indexes}
sudo chown -R 2000:2000 ./volumes/app/mattermost
```

인증서를 생성 합니다.  
```
bash scripts/issue-certificate.sh -d <YOUR_MM_DOMAIN> -o ${PWD}/certs
```

`.env` 파일에서 아래 부분을 수정합니다.  
``` 
 2 DOMAIN=mm.example.com
 8 TZ=Asis/Seoul
41 CERT_PATH=./certs/etc/letsencrypt/live/${DOMAIN}/fullchain.pem
42 KEY_PATH=./certs/etc/letsencrypt/live/${DOMAIN}/privkey.pem
```

도커로 mattermost를 실행 합니다.   
``` bash
sudo docker compose -f docker-compose.yml -f docker-compose.nginx.yml up -d
```

종료 명령어 입니다.  
``` bash
sudo docker compose -f docker-compose.yml -f docker-compose.nginx.yml down
```
