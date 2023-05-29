---
title:  "DevSecOps"

categories:
  - DevOps
tags:
  - devsecops
  - devops
# 목차
toc: true
---

# DevSecOps

## 1. 시작하기전

* 아래의 내용은 kodekloud의 DevSecOps 강의를 토대로 작성한 위키입니다.
* 흥미를 위해 다소 깊은 내용은 생략하였으니, 깊은 내용을 알고 싶다면 해당 강의를 수강하시길 권장합니다.
* 해당 위키에 사용된 내용과 사진 자료는 전적으로 kodekloud에게 저작권이 있음을 알립니다.

### 1.1 왜 알아야 (또는 알면) 좋은가?

* DevOps에서 더 나아가서 빌드/배포 과정에서 보안적으로 문제가 되는 부분이 없는 지, 빌드가 완료된 결과물에 문제가 없는 지를 확인을 하는 것도 중요함

## 2. DevSecOps

![DevSecOps-1](/assets/img/devsecops-1.png)

* 말그대로 DevOps에 Security를 포함시킨 것이다.
* 위 사진에서 붉은색으로 표시된 부분이 바로 DevOps 파이프라인에 추가된 Security 요소라고 보면 된다.


### 2.1 Talisman - Pre-commit / Pre-Publish(Push) Hook

* Git으로 소스 관리를 한다고 치면, commit 또는 push하기 전에 소스 코드에 민감한 정보 (access key, access token, ssh key)가 포함되어있는 지를 확인
* Talisman이 바로 이를 실현하기 위한, open-source tool임
* Talisman은 pattern matching 기반으로 감지하며, pre-commit, pre-push 2개의 hooking point가 존재
* Talisman 설치는 개발 machine ( =local)에 하는 것이며, 일반적으로 global 하게 적용하나 single project에도 적용할 수 있음
* 설치하게 되면, $HOME/.git-template/hooks/pre-commit 파일이 생성되는 것을 확인할 수 있음
* install : https://thoughtworks.github.io/talisman/docs/installation/global-hook/
* 적용 예시

![DevSecOps-2](/assets/img/devsecops-2.png)

* 예시에서 test.txt 파일에 password라는 pattern이 감지되었다는 정보를 확인하여 console 화면에 노출
* 물론, 강제를 할 수 없는 부분이기 때문에, commimt/push에 반영할 지에 대해서 y/n를 물어보며 반영을 할 예정이면 해당 파일에 대한 검수를 skip하기 위해 .talismanrc 파일이 생성됨
* 이런 문제를 해소하기 위해, Vault (또는 AWS Secret Manager)를 사용하는 것임

### 2.2 PIT - Mutation Tests

* Mutation testing is some aspect is testing your actual Unit Test cases
* PIT runs your unit tests against automatically modified versions of your application code.
* 돌연변이 테스트 (MT)는 고의적으로 소스 코드에 변형을 주어, 작성된 unit test code가  견고하고 (=세밀하게), 올바르게 작성되었는 지를 확인하는 테스트임
  * ex. conditional boundary 변경, return value 변경
* maven project에서 jenkins PIT Mutation plugin을 이용하여 MT를 수행할 수 있음
* 이 부분은 개발자의 양심에 맡길 영역으로 보이며, 부지런함과 꼼꼼함을 요함
* 예시

![DevSecOps-3](/assets/img/devsecops-3.png)

* 설명을 위해 위 예시에서 맨 위부터 1번, 2번, 3번 case라고 칭하겠음
* 1번 case는 MT 없이, 작성한 Unit Test를 수행시킨 것이고, 단순하게 보면 올바른 unit test라고 볼 수 있음
* 하지만 2번 case에서는 MT를 적용하여, 조건문을 고의적으로 변경하였음에도 unit test가 통과되었기 때문에 이는 unit test를 다시 살펴볼 필요가 있다는 것을 의미함
* 3번 case에서는 Unit Test code를 수정하여, MT가 적용되었을 때 unit test가 실패한 것이기 때문에 unit test를 올바르게 수정했다고 볼 수 있음

### 2.3 SonarQube - SAST (Static Application Security Testing)

* SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code.
* 소나큐브는 작성된 소스 코드를 기반으로 코드에 문제가 없는 지 "정적"분석을 하는 도구임
* method : Quality Gates, Leak Period
  * Quality Gates : 코드 품질 측정 값에 대한 threshold 값을 지정
  * Leak Period : 현재 코드와 선택한 특정 지점 사이의 코드 품질 측정 값의 차이를 확인
* 예시

![DevSecOps-4](/assets/img/devsecops-4.png)

* 위 예시에서는 providedClass가 null일 경우, null에 접근할 시 exception을 발생시킬 수 있다는 냄새 (code smell)를 맡음

### 2.4 Vulnerability Basic

![DevSecOps-5](/assets/img/devsecops-5.png)

* Vulnerability는 사전적 정의로 취약점을 의미하며, 일반적으로 보안에 있어서의 취약점을 의미함
* Exploit은 사전적 의미로 악용을 의미하며, 취약점을 이용하여 프로그램에 영향을 줄 수 있는 공격코드를 의미하기도 함
* Threat는 사전적 의미로 위협을 의미하며, 공격코드를 이용하여 프로그램에 영향을 주는 행위를 의미
* Risk는 위 3가지가 결합되면 발생할 수 있는 잠재적 위험을 의미
* 그 외에도 여러가지 용어들이 있는데, 하나만 더 살펴보면 CVE가 있는데, 공개적으로 알려진 컴퓨터 보안 결함 목록을 의미함
* ex. https://avd.aquasec.com/nvd/cve-2022-1304

### 2.5 Dependency-Check - Vulnerability Scan

* Dependency-check is an open source project created by OWASP (Open Worldwide Application Security Project)
* 취약점에 대한 소프트웨어 구성 분석, 제품에서 사용 중인 open source에 대한 취약점 검증
* pom.xml, packages로부터 정보 수집
* Jenkins에서도 사용할 수 있는데, dependency-check plugin 설치 후, $ mvn dependency-check:check 명령어로 수행 후 xml 파일로 report 추출하여 결과 확인 가능
* 사내 환경에서도 사용해보려했으나, 동적으로 접근하는 public url이 있어서 proxy 이슈로 적용이 불가하였음 (whitelist 적용 필요)
* 예시

![DevSecOps-6](/assets/img/devsecops-6.png)

### 2.6 Trivy - Vulnerability Scan

* Trivy is a simple and comprehensive vulnerability scanner for containers and other artifacts.
* container registry인 harbor에서 주로 사용
* 예시

![DevSecOps-7](/assets/img/devsecops-7.png)

### 2.7 Conftest - OPA

* OPA (Open Policy Agent) is an open source, general-purpose policy engine that unifies policy enforcement across the stack
* Conftest는 configuration data를 정적 분석해주는 도구임
* Conftest는 Rego라는 언어을 사용하여 점검할 rule을 작성 (ex. docker.rego)
* install doc : conftest.dev/install/
* 예시

![DevSecOps-8](/assets/img/devsecops-8.png)

### 2.8 Kubesec - security scan

* kubesec is an open-source k8s security scanner and analysis tool
* kubesec.io에 검토 요청을 하여, yaml 파일 검증을 할 수 있음
* 이를 통해 자신이 작성한 yaml 파일에 대해, 점검을 해볼 수 있음
* doc : kubesec.io

![DevSecOps-9](/assets/img/devsecops-9.png)

### 2.9 ZAP (Zed Attack Proxy) - DAST (Dynamic Application Security Testing)

* ZAP is an open-source web application security scanner

![DevSecOps-10](/assets/img/devsecops-10.png)

* ZAP는 브라우저와 앱 사이에서 전송된 메세지를 가로채서 검사
* ZAP API scan
  * doc : zaproxy.org/docs/docker/api-scan/
  * 우선 테스트 대상 프로젝트의 api 문서를 생성해야함 (테스트할 api를 알기 위함)
    * 이를 위해 보통 openapi를 사용 (+ swagger ui)
  * zap docker image를 이용하여 api-docs url과 필요한 parameter를 주어서 zap-api-scan.py를 실행
  * 실행해서 결과를 html 문서로 추출하여 jenkins에서 확인가능 (물론 pipeline script 작성 필요)
  * 예시

![DevSecOps-11](/assets/img/devsecops-11.png)

### 2.10 Tools and Technologies for other Programming Languages

![DevSecOps-12](/assets/img/devsecops-12.png)

### 2.11 SDLC with security tools

![DevSecOps-13](/assets/img/devsecops-13.png)

### 2.12 Reference

* Talisman – Keep your secrets secret
  * https://thoughtworks.github.io/talisman/docs
* PITest – Mutation Tests
  * https://pitest.org/quickstart/maven/
* Dependency CHeck  Configuration
  * https://jeremylong.github.io/DependencyCheck/dependency-check-gradle/configuration.html
* Trivy Docs
  * https://aquasecurity.github.io/trivy/v0.18.3/
* OPA Conftest – Docker Security
  * https://github.com/gbrindisi/dockerfile-security
* OWASP ZAP – API Scan
  * https://www.zaproxy.org/docs/docker/api-scan/

## 3. K8S operations and security

### 3.1 CIS Benchmarking

* CIS (Center for Internet Security) releases benchmarks for best practice security recommendations
* kube-bench is a Go application that checks whether k8s is deployed securely by running the checks documented in the CIS k8s benchmark.
* 검수 대상이 앱이 아닌 k8s cluster임!

### 3.2 Kube-bench

* cis benchmark 기반으로 version별/module별(ex. etcd, master)로 체크가능
* install : https://github.com/aquasecurity/kube-bench/blob/main/docs/installation.md
* 예시

![DevSecOps-14](/assets/img/devsecops-14.png)

### 3.3 Kube-bench

* Pod-Pod Communication - Need for mTLS

![DevSecOps-15](/assets/img/devsecops-15.png)

* istio 쓰면 자연스럽게 해결 된다.

### 3.4 Falco

* Falco, the open-source cloud-native runtime security project.
* Falco detects unexpected application behavior and alerts on threats at runtime.
* install : falco.org/docs/getting-started/installation/

![DevSecOps-16](/assets/img/devsecops-16.png)

* 위 예시에서는 falco가 동작중일 때, kubectl command를 이용해서 container에서 /bin/bash를 실행하였을 때, 감지되는 것을 확인할 수 있다.

## 4. 마치며

* DevSecOps에 대한 개념과 이를 이루기 위해 사용되는 open source tool들에 대해서 살펴보았다.
* 필자는 위에 나열되어있는 tool 중에서 정적분석도구인 SonarQube에 대해서 깊게 살펴볼 예정이다.
