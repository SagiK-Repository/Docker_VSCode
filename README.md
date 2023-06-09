문서정보 : 2023.04.27.-05.09. 작성, 작성자 [@SAgiKPJH](https://github.com/SAgiKPJH)

<br>

# Docker_VSCode_Server
VSCode Server를 Docker로 돌립니다.

### 목표
- [x] : VSCode Server를 내포하는 Dockerfile을 구성합니다.
- [x] : Docker Hub - Linux VSCode server
- [x] : 보다 높은 성능을 위한 내용

### 제작자
[@SAgiKPJH](https://github.com/SAgiKPJH)

<br><br>

---

<br><br>

# VSCode Server를 내포하는 Dockerfile을 구성합니다.

다음 조건을 만족하는 VSCode Server Docker를 구성합니다.  
1. vscode server 아이디 사전에 설정 가능
2. vscode server 비밀번호 사전에 설정 가능
3. 확장 설치 설정 가능
4. 원하는 포트에 서비스 가능
5. vscode server 이미 실행되어 있게 구성

<br>

### Dockerfile 구성

다음과 같이 dockerfile을 구성합니다.  
이때, code-server를 백그라운드에서 실행하기 위해 `nohup` 명령어를 활용하였습니다.

```dockerfile
FROM ubuntu:latest

# 필요한 패키지 설치, cache 비우기
RUN apt-get update && \
    apt-get install -y curl sudo

# 새로운 사용자 생성 및 비밀번호 설정
ENV USER="user" \
    PASSWORD="password"
RUN useradd -m ${USER} && echo "${USER}:${PASSWORD}" | chpasswd && adduser ${USER} sudo

# code-server 설치 및 세팅
ENV WORKINGDIR="/home/${USER}/vscode"
RUN curl -fsSL https://code-server.dev/install.sh | sh && \
    mkdir ${WORKINGDIR}
    
# 확장 설치
RUN code-server --install-extension "ms-python.python" \ 
                --install-extension "ms-azuretools.vscode-docker"

# code-server 시작
ENTRYPOINT nohup code-server --bind-addr 0.0.0.0:8080 --auth password  ${WORKINGDIR}

# docker build --no-cache -t vscode-docker .
# docker run -it --name vscode-container -p 8080:8080 vscode-docker
```
  
설치하고자 하는 확장은 https://marketplace.visualstudio.com/items?itemName=ms-python.python 사이트에 들어가 `Unique Identifier`항목을 찾아 입력합니다.  
<img src="https://user-images.githubusercontent.com/66783849/234725217-4081ba0a-bd39-4944-86db-afce20b1227f.png">  
<img src="https://user-images.githubusercontent.com/66783849/234725243-686def18-71a5-4319-85f4-2af2c357c8bb.png">  


<br>

### dockerfile 빌드
`vscode-docker`라는 이름의 Docker Image를 만듭니다.  
빌드가 오래걸릴경우, `--progress=plain`옵션을 추가하여 과정을 텍스트로 출력합니다.  
```bash
docker build -t vscode-docker .

docker build --progress=plain -t vscode-docker .
```
<img src="https://user-images.githubusercontent.com/66783849/234724445-877ebefd-96bf-471e-890e-4fe7df0bb44f.png">  

<br>

### docker 실행 후 검증

```bash
docker run -it --name vscode-container -p 8080:8080 vscode-docker
```
다음과 같이 접속 가능함을 확인할 수 있습니다.  
<img src="https://user-images.githubusercontent.com/66783849/236122610-ee1992db-73e6-49cc-923e-87f30581f3b6.png"/>  
<img src="https://user-images.githubusercontent.com/66783849/236748652-04fb3427-a3dd-4bfc-a823-c7605c1aa3d2.png"/>  

다음과 같이 확장을 확인할 수 있습니다.  
<img src="https://user-images.githubusercontent.com/66783849/236140752-2cd56f80-8d89-4c29-9261-8219288c767c.png"/>  

<br><br>

# Docker Hub - Linux VSCode server

- docker hub에선 linux 기반 vscode server를 제공하는 image가 있다. https://hub.docker.com/r/linuxserver/code-server
- 이 이미지를 run하여 docker server를 내포하는 docker를 실행할 수 있다.

<br><br>

# 보다 높은 성능을 위한 내용

- 보안성 및 이미지 용량을 줄이기 위해 다음과 같이 구성할 수 있습니다.  
  - 보안을 위해 User로 실행하고 User로 접근합니다.
  - 불필요한 파일을 삭제하여 용량을 최소화 합니다.
  ```dockerfile
  # Base Image
  FROM ubuntu:latest
  
  # 필요한 패키지 설치, cache 비우기
  RUN apt-get update && \
      apt-get install -y curl sudo && \
      apt-get clean && \
      rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
  
  # 새로운 사용자 생성 및 비밀번호 설정
  ENV USER="user" \
      PASSWORD="password"
  RUN useradd -m ${USER} && echo "${USER}:${PASSWORD}" | chpasswd && adduser ${USER} sudo && \
      sed -i "/^root/ c\root:!:18291:0:99999:7:::" /etc/shadow
  
  # code-server 설치 및 세팅
  ENV WORKINGDIR="/home/${USER}/vscode"
  RUN curl -fsSL https://code-server.dev/install.sh | sh && \
      mkdir ${WORKINGDIR} && \
      su ${USER} -c "code-server --install-extension ms-python.python \
                                 --install-extension ms-azuretools.vscode-docker" && \
      rm -rf ${WORKINGDIR}/.local ${WORKINGDIR}/.cache
  
  # code-server 시작
  USER ${USER}
  ENTRYPOINT nohup code-server --bind-addr 0.0.0.0:8080 --auth password  ${WORKINGDIR}
  
  # docker build -t vscode-docker .
  # docker run -it --name vscode-container -p 8080:8080 vscode-docker
  ```
