# [Tomcat] OPENSHIFT4 Template Disconnect Build

## 사전준비사항

### LocalPC의 hosts File에 주소등록

- C:\Windows\System32\drivers\etc\hosts

```
10.1.0.251	gitlab.apps.ocp4.example.io console-openshift-console.apps.ocp4.example.io oauth-openshift.apps.ocp4.example.io
```

### 개인환경

| 이름 | 용도 | PublicIP | PrivateIP | ID/PW |
| --- | --- | --- | --- | --- |
| LIM | Proxy | 10.1.0.230 | 10.199.107.230 | root/P@ssw0rd |
|  | Nexus | 10.1.0.233 | 10.199.107.233 | root/P@ssw0rd |
| MIN | Proxy | 10.1.0.231 | 10.199.107.231 | root/P@ssw0rd |
|  | Nexus | 10.1.0.234 | 10.199.107.234 | root/P@ssw0rd |
| LEE | Proxy | 10.1.0.232 | 10.199.107.232 | root/P@ssw0rd |
|  | Nexus | 10.1.0.23 | 10.199.107.235 | root/P@ssw0rd |

## Proxy Server

### Squid 설치

```bash
$ yum install squid
```

### 기본 구성

```bash
$ vi /etc/squid/squid.conf
#
# Recommended minimum configuration:
… 생략
# 허용할 대역
acl localnet src 10.199.107.0/24
cache_dir ufs /var/spool/squid 10000 16 256
$ mkdir -p /var/spool/squid
$ chown squid:squid /var/spool/squid
$ systemctl enable --now squid
```

### Test

```bash
$ curl -O -L "https://www.redhat.com/index.html" -x "proxy.ocp4.example.io:3128"
$ ls /var/spool/squid/
00  01  02  03  04  05  06  07  08  09  0A  0B  0C  0D  0E  0F  swap.state
```

- Default Port : 3128

## Standalone Nexus

### Nexus 설치

```bash
$ mkdir /opt ; cd /opt
$ wget http://download.sonatype.com/nexus/3/latest-unix.tar.gz
$ tar xvf latest-unix.tar.gz
$ mv /opt/nexus-3.43.0-1 /opt/nexus
$ vi /opt/nexus/etc/nexus-default.properties
## DO NOT EDIT - CUSTOMIZATIONS BELONG IN $data-dir/etc/nexus.properties
##
# Jetty section
application-port=8081
application-host=0.0.0.0
nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-requestlog.xml
nexus-context-path=/

# Nexus section
nexus-edition=nexus-oss-edition
nexus-features=\
 nexus-oss-feature

nexus.hazelcast.discovery.isEnabled=true
$ nohup /opt/nexus/bin/nexus run &
```

### Web Console접속

```bash
Firefox or Chrom > http://{localhostIP}:8081
```

![<사진1>](%5BTomcat%5D%20OPENSHIFT4%20Template%20Disconnect%20Build%20ee6743746b6d44b7a8fa245bca97691e/Untitled.png)

<사진1>

![<사진2>](%5BTomcat%5D%20OPENSHIFT4%20Template%20Disconnect%20Build%20ee6743746b6d44b7a8fa245bca97691e/Untitled%201.png)

<사진2>

### Proxy 설정

![<사진3>](%5BTomcat%5D%20OPENSHIFT4%20Template%20Disconnect%20Build%20ee6743746b6d44b7a8fa245bca97691e/Untitled%202.png)

<사진3>

## OPENSHIFT4 Template를 이용하여 배포

### Template 접근하여 값추가

```
Web Console접속 > 개발자 > +추가 > 모든서비스 > tomcat
```

![<사진4>](%5BTomcat%5D%20OPENSHIFT4%20Template%20Disconnect%20Build%20ee6743746b6d44b7a8fa245bca97691e/Untitled%203.png)

<사진4>

![<사진5>](%5BTomcat%5D%20OPENSHIFT4%20Template%20Disconnect%20Build%20ee6743746b6d44b7a8fa245bca97691e/Untitled%204.png)

<사진5>

![<사진6>](%5BTomcat%5D%20OPENSHIFT4%20Template%20Disconnect%20Build%20ee6743746b6d44b7a8fa245bca97691e/Untitled%205.png)

<사진6>

- 네임스페이스 : demo01
- Application Name : hello-world
- Git Repository URL : [http://gitlab.apps.ocp4.example.io/example/example-tomcat-war.git](http://gitlab.apps.ocp4.example.io/example/example-tomcat-war.git)
- Git Reference : dev
- Context Directory: /
- Maven mirror URL : [http://10.199.107.240:8081/repository/maven-central/](http://10.1.0.251:8081/repository/maven-central/)

![<사진7>](%5BTomcat%5D%20OPENSHIFT4%20Template%20Disconnect%20Build%20ee6743746b6d44b7a8fa245bca97691e/Untitled%206.png)

<사진7>

- <사진7> 과 같이 Gitlab에서 소스를 가지고 올때 오픈시프트 프로젝트에서 Username/Password 에 대한 Secret 정보가 없어 에러가 발생

### Secret 생성

```bash
$ oc create secret generic git-secret --from-literal=username=example --from-literal=password=123qwe.. --type=kubernetes.io/basic-auth
```

![<사진8>](%5BTomcat%5D%20OPENSHIFT4%20Template%20Disconnect%20Build%20ee6743746b6d44b7a8fa245bca97691e/Untitled%207.png)

<사진8>

- 생성한 Secret를 sourceSecret 정보에 넣어준후 다시 빌드 진행

```yaml
...(생략)
source:
    type: Git
    git:
      uri: 'http://gitlab.apps.ocp4.example.io/example/example-tomcat-war.git'
      ref: dev
    contextDir: /
    sourceSecret:
      name: git-secret
  triggers:
    - type: GitHub
...(생략)
```

## TEST

![<사진9>](%5BTomcat%5D%20OPENSHIFT4%20Template%20Disconnect%20Build%20ee6743746b6d44b7a8fa245bca97691e/Untitled%208.png)

<사진9>
