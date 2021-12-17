## PaaS-TA Application Platform log4j 조치 방법 가이드

본 가이드는 log4j와 관련하여 PaaS-TA Application Platform의 조치 방법에 대한 가이드다.

### CVE 주요 내용 
- Apache Log4j 2*에서 발생하는 원격코드 실행 취약점(CVE-2021-44228)
- Apache Log4j 2에서 발생하는 서비스 거부 취약점(CVE-2021-45046)
- Apache Log4j 1.x에서 발생하는 원격코드 실행 취약점(CVE-2021-4104)

#### 영향을 받는 버전
- paasta-deployment 5.6.3 이하 버전

#### 영향을 받는 모듈
- BOSH (UAA, Credhub)
- PaaS-TA AP Core (UAA, Credhub, java-buildpack의 일부 모듈, php-buildpack의 일부 모듈)

#### 조치 방안 (log4j 2.16 업데이트)
- [paasta-deployment 5.6.0 ~ 5.6.3 버전 사용 시](#1)
  - log4j 관련 패치를 적용한 paasta-deployment 5.6.4 업데이트 (BOSH, PaaS-TA AP, java-buildpack의 일부 모듈, php-buildpack의 일부 모듈)
     ※ UAA : 75.1.0-PaaS-TA-v3.1 , Credhub : 2.9.0-PaaS-TA-v2.1, PHP Buildpack Release : 4.4.53, Java Buildpack Release : 4.45 

+ [paasta-deployment 5.5.2 이하 버전 사용 시](#2)
  + UAA와 Credhub를 설치된 버전에 맞춰 릴리즈를 생성 후 릴리즈만 반영하여 업데이트 (BOSH, PaaS-TA AP)

***

#### 조치 방안 세부 설명
##### <div id='1'> 1. paasta-deployment 5.6.0 ~ 5.6.3 버전 사용 시 (paasta-deployment 5.6.4 업데이트)

###### 1.1. paasta-deployment 5.6.4 다운로드

```
$ git clone https://github.com/PaaS-TA/paasta-deployment.git -b v5.6.4 paasta-deployment-5.6.4
```

###### 1.2. 관련 변수 설정
사용중인 IaaS에 맞춰서 전에 배포하였던 변수 파일을 paasta-deployment-5.6.4에 옮기거나 설치 관련 변수를 작성한다.

- 이동 필요 파일
    - bosh/{사용중인 IaaS}/creds.yml
    - bosh/{사용중인 IaaS}/state.json
-  이동 혹은 수정 필요 파일
    - bosh/{사용중인 IaaS}-vars.yml
    - bosh/deploy-{사용중인 IaaS}.sh
    - paasta/vars.yml
    - paasta/deploy-{사용중인 IaaS}.sh
- cce.yml 사용 파일 위치
    - bosh/deploy-{사용중인 IaaS}.sh
    - paasta/deploy-{사용중인 IaaS}.sh

- 해당 내용이 있는지 체크 (JAVA의 버전과 PHP의 버전이 올라가니 주의 필요)  
각 Java Buildpack과 PHP Buildpack의 정보는 Release Notice를 참고한다.  
[Java Buildpack v4.45](https://github.com/cloudfoundry/java-buildpack/releases/tag/v4.45), [PHP Buildpack v4.4.53](https://github.com/cloudfoundry/php-buildpack/releases/tag/v4.4.53)  

```
$ vi bosh/cce.yml
- type: replace
  path: /releases/name=uaa?
  value:
    name: uaa
    sha1: d31457bdd200b7bb02e4e6f5ca24e3dbf69ffe82
    url: https://nextcloud.paas-ta.org/index.php/s/f2mjNX3RT93RxJg/download
    version: 75.1.0-PaaS-TA-v3.1
- type: replace
  path: /releases/name=credhub?
  value:
    name: credhub
    sha1: 3da7d1901351c87e79dea4285ba85b3f2b013a2a
    url: https://nextcloud.paas-ta.org/index.php/s/fLYAatkdj8XE35R/download
    version: 2.9.0-PaaS-TA-v2.1
  

$ vi paasta/operations/cce.yml
- type: replace
  path: /releases/name=uaa
  value:
    name: uaa
    sha1: d31457bdd200b7bb02e4e6f5ca24e3dbf69ffe82
    url: https://nextcloud.paas-ta.org/index.php/s/f2mjNX3RT93RxJg/download
    version: 75.1.0-PaaS-TA-v3.1
- type: replace
  path: /releases/name=credhub
  value:
    name: credhub
    sha1: 3da7d1901351c87e79dea4285ba85b3f2b013a2a
    url: https://nextcloud.paas-ta.org/index.php/s/fLYAatkdj8XE35R/download
    version: 2.9.0-PaaS-TA-v2.1
- type: replace
  path: /releases/name=php-buildpack
  value:
    name: php-buildpack
    sha1: a91a3a479012dbc37f1185d28cc2b0cb110bcc92
    url: https://bosh.io/d/github.com/cloudfoundry/php-buildpack-release?v=4.4.53
    version: 4.4.53
- type: replace
  path: /releases/name=java-buildpack
  value:
    name: java-buildpack
    sha1: b11614fa65da34b171701287f1f1aa6da4ebba63
    url: https://bosh.io/d/github.com/cloudfoundry/java-buildpack-release?v=4.45
    version: 4.45
```

```
## 해당 내용이 있는지 체크
$ vi bosh/deploy-{사용중인 IaaS}.sh
-o cce.yml \

$ vi paasta/deploy-{사용중인 IaaS}.sh
 -o operations/cce.yml \
```

###### 1.4. 배포
BOSH 와 PaaS-TA AP Core를 배포한다.
- BOSH 배포
```
## BOSH 배포
$ cd bosh
$ source deploy-{사용중인 IaaS}.sh

## 배포 완료 시 기존 사용중이던 VM 확인
$ bosh vms
```

AP Core에 배포되는 UAA와 Credhub의 경우 업데이트 시 인스턴스의 용량이 부족할 수 있으므로 여유 공간을 확인 후 부족할 시 VM Type을 변경하여 배포한다.
```diff
$ bosh -d paasta ssh uaa/0

uaa/29704389-1778-417c-be1e-002e0c403d9c:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        974M     0  974M   0% /dev
tmpfs           994M     0  994M   0% /dev/shm
tmpfs           994M  104M  890M  11% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           994M     0  994M   0% /sys/fs/cgroup
/dev/vda1       2.8G  1.8G  835M  69% /
+/dev/vda3        15G  921M   14G   7% /var/vcap/data
tmpfs            16M  304K   16M   2% /var/vcap/data/sys/run
tmpfs           199M     0  199M   0% /run/user/1006

## Avail 이 1G 이상 확보되어야 배포 가능, 부족할 시 Cloud-config를 참고하여 vars.yml의 vm_type을 변경하여 배포한다.
## VM Type을 변경 시 필요에 따라 로그파일 백업을 진행한다.
```

- PaaS-TA AP Core 배포
```
## PaaS-TA AP Core 배포
$ cd paasta
$ source deploy-{사용중인 IaaS}.sh


releases:
- name: credhub
-   sha1: f60d84d89a2ca5ef0b4c904e7d618b6f20bae761
+   sha1: 3da7d1901351c87e79dea4285ba85b3f2b013a2a
-   url: https://nextcloud.paas-ta.org/index.php/s/s6Dz7ZQoDN2fAad/download
+   url: https://nextcloud.paas-ta.org/index.php/s/fLYAatkdj8XE35R/download
-   version: 2.9.0-PaaS-TA
+   version: 2.9.0-PaaS-TA-v2.1
- name: uaa
-   sha1: c56b0bd3031a673b3753fba35d3f2caff2f497e2
+   sha1: d31457bdd200b7bb02e4e6f5ca24e3dbf69ffe82
-   url: https://nextcloud.paas-ta.org/index.php/s/B2wZSE6LEnRJn3c/download
+   url: https://nextcloud.paas-ta.org/index.php/s/f2mjNX3RT93RxJg/download
-   version: 75.1.0-PaaS-TA-v2
+   version: 75.1.0-PaaS-TA-v3.1
- name: java-buildpack
-   sha1: 0d4d939490b0f16f4d2f8dfd937185479bf9f877
+   sha1: b11614fa65da34b171701287f1f1aa6da4ebba63
-   url: https://bosh.io/d/github.com/cloudfoundry/java-buildpack-release?v=4.37
+   url: https://bosh.io/d/github.com/cloudfoundry/java-buildpack-release?v=4.45
-   version: '4.37'
+   version: '4.45'
- name: php-buildpack
-   sha1: f169ae43c11d5e872e5020e1e65d32375d82cb31
+   sha1: a91a3a479012dbc37f1185d28cc2b0cb110bcc92
-   url: https://bosh.io/d/github.com/cloudfoundry/php-buildpack-release?v=4.4.36
+   url: https://bosh.io/d/github.com/cloudfoundry/php-buildpack-release?v=4.4.53
-   version: 4.4.36
+   version: 4.4.53


- manifest_version: v5.6.2

+ manifest_version: v5.6.4
Task 51

Task 51 | 03:20:10 | Preparing deployment: Preparing deployment (00:00:20)
.....
Task 51 Started  Wed Dec 15 03:20:10 UTC 2021
Task 51 Finished Wed Dec 15 03:25:56 UTC 2021
Task 51 Duration 00:05:46
Task 51 done

Succeeded

```

##### <div id='2'> 2. paasta-deployment 5.5.2 이하 버전 사용 시 (UAA 와 Credhub 릴리즈 직접 생성)
AP의 업데이트가 아닌 모듈의 업데이트 시 **UAA**와 **Credhub**의 작업을 진행하여 부분적으로 업데이트 한다.  
이 가이드는 5.5.2의 UAA 74.29.0-PaaS-TA 을 기준으로 작성하였다. (실제 생성시 UAA와 Credhub를 같이 작업한다  
모듈 최신버전 github  
uaa :  https://github.com/PaaS-TA/uaa-release/tree/75.1.0-PaaS-TA-v3.1  
credhub : https://github.com/PaaS-TA/credhub-release/tree/2.9.0-PaaS-TA-v2.1  

###### 2.1. git 다운로드 (작업하고 싶은 릴리즈 버전으로 다운로드)
```
$ git clone https://github.com/PaaS-TA/uaa-release.git -b 74.29.0-PaaS-TA
```

###### 2.2. git submodule 업데이트
```
$ cd uaa-release
$ git submodule init
$ git submodule update
```
###### 2.3. log4j version fix
https://github.com/PaaS-TA/uaa-release/blob/75.1.0-PaaS-TA-v3.1/PaaS-TA_README.md 와 https://github.com/PaaS-TA/credhub-release/blob/2.9.0-PaaS-TA-v2.1/PaaS-TA_README.md 를 확인 후 해당되는 버전에 맞게 보안조치 작업 후 log4j 버전에 대한 업데이트를 진행한다. (UAA와 Credhub의 최신 패치 버전을 참고한다.)

```diff
## uaa의 경우
$ vi src/uaa/dependencies.gradle
+ ext['log4j2.version'] = '2.16.0'             #추가
  
  
  
  
## credhub의 경우
$ vi src/uaa/dependencies.gradle
+ log4jVersion = '2.16.0'               #추가


$ vi src/credhub/applications/credhub-api/build.gradle
+  implementation("org.apache.logging.log4j:log4j-api:${log4jVersion}")          #추가
+  implementation("org.apache.logging.log4j:log4j-core:${log4jVersion}")          #추가
+  implementation("org.apache.logging.log4j:log4j-jul:${log4jVersion}")          #추가
+  implementation("org.apache.logging.log4j:log4j-slf4j-impl:${log4jVersion}")          #추가

```


###### 2.4. 릴리즈 생성
```
$ export UAA_VERSION=74.29.0
$ bosh create-release --name uaa --sha2 --force --tarball ./uaa-release-74.29.0-PaaS-TA-v3.1.tgz --version 74.29.0-PaaS-TA-v3.1
```

###### 2.5 sha1값 확인
```
## sha1 값 확인
$ sha1sum uaa-release-74.29.0-PaaS-TA-v3.1.tgz
198a2b5d769c856c55813506306e89a64f178256  uaa-release-74.29.0-PaaS-TA-v3.1.tgz
```


###### 2.6. 배포 deployment에 cce.yml 변경
과거 설치되었던 Deployment의 릴리즈 파일을 사용할 수 있게 작업을 진행한다.
```
$ vi bosh/cce.yml
$ vi paasta/operations/cce.yml


# 해당 내용 삽입
- type: replace
  path: /releases/name=uaa?
  value:
    name: uaa
    sha1: {UAA RELEASE SHA1}
    url: {UAA RELEASE PATH}
    version: 74.29.0-PaaS-TA
- type: replace
  path: /releases/name=credhub?
  value:
    name: credhub
    sha1: {Credhub RELEASE SHA1}
    url: {Credhub RELEASE PATH}
    version: 2.8.0-PaaS-TA

```



###### 2.7. 배포

BOSH 와 PaaS-TA AP Core를 배포한다.
- BOSH 배포
```
## BOSH 배포
$ cd bosh
$ source deploy-{사용중인 IaaS}.sh

## 배포 완료 시 기존 사용중이던 VM 확인
$ bosh vms
```

AP Core에 배포되는 UAA와 Credhub의 경우 업데이트 시 인스턴스의 용량이 부족할 수 있으므로 여유 공간을 확인 후 부족할 시 VM Type을 변경하여 배포한다.
```diff
$ bosh -d paasta ssh uaa/0

uaa/29704389-1778-417c-be1e-002e0c403d9c:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        974M     0  974M   0% /dev
tmpfs           994M     0  994M   0% /dev/shm
tmpfs           994M  104M  890M  11% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           994M     0  994M   0% /sys/fs/cgroup
/dev/vda1       2.8G  1.8G  835M  69% /
+/dev/vda3        15G  921M   14G   7% /var/vcap/data
tmpfs            16M  304K   16M   2% /var/vcap/data/sys/run
tmpfs           199M     0  199M   0% /run/user/1006

## Avail 이 1G 이상 확보되어야 배포 가능, 부족할 시 Cloud-config를 참고하여 vars.yml의 vm_type을 변경하여 배포한다.
## VM Type을 변경 시 필요에 따라 로그파일 백업을 진행한다.
```

- PaaS-TA AP Core 배포
```
## PaaS-TA AP Core 배포
$ cd paasta
$ source deploy-{사용중인 IaaS}.sh


Task 51

Task 51 | 03:20:10 | Preparing deployment: Preparing deployment (00:00:20)
.....
Task 51 Started  Wed Dec 15 03:20:10 UTC 2021
Task 51 Finished Wed Dec 15 03:25:56 UTC 2021
Task 51 Duration 00:05:46
Task 51 done

Succeeded

```

  
###### 2.8. Buildpack 업데이트

Java Buildpack과 PHP Buildpack의 특정 모듈을 PaaS-TA AP를 사용하는 유저가 사용한다면 빌드팩의 버전을 최신버전으로 업데이트 하여야 한다.  
(이 경우 JAVA의 버전과 PHP의 버전이 올라가니 주의 필요)  
각 Java Buildpack과 PHP Buildpack의 정보는 Release Notice를 참고한다.  
[Java Buildpack v4.45](https://github.com/cloudfoundry/java-buildpack/releases/tag/v4.45), [PHP Buildpack v4.4.53](https://github.com/cloudfoundry/php-buildpack/releases/tag/v4.4.53)

- CVE 검출 모듈
  - Java Buildpack
    - AppDynamics Java Agent
    - New Relic Java Agent
    - Elastic APM Java Agent
    - Geode Tomcat Session Store
  - PHP Buildpack
    - AppDynamics PHP agent

```
## 새로운 빌드팩 사용 시
$ cf create-buildpack java-buildpack_445 https://github.com/cloudfoundry/java-buildpack/releases/download/v4.45/java-buildpack-v4.45.zip 14
$ cf create-buildpack php_buildpack_4453 https://github.com/cloudfoundry/php-buildpack/releases/download/v4.4.53/php-buildpack-cflinuxfs3-v4.4.53.zip 14

## 빌드팩 업데이트 시
$ cf update-buildpack java_buildpack -p https://github.com/cloudfoundry/java-buildpack/releases/download/v4.45/java-buildpack-v4.45.zip
$ cf update-buildpack php_buildpack -p https://github.com/cloudfoundry/php-buildpack/releases/download/v4.4.53/php-buildpack-cflinuxfs3-v4.4.53.zip

```
