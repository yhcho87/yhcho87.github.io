# Jenkins CI + OCI DevOps CD 환경 준비

## 소개

Jenkins CI Pipeline과 OCI DevOps CD Pipeline 환경을 연동하는 실습을 진행하기 위한 환경을 준비합니다.

![Dashboard](images/devops-with-jenkins.png " ")

소요시간: 20 minutes

### 목표

-  Oracle Cloud Infrastructure (OCI) Cloud Native 환경 구성.  

### 사전 준비사항

1. 실습을 위한 노트북 (Windows, MacOS)
1. Oracle Free Tier 계정

## Task 1 : Compute Instance 생성 및 Instance 기본 설정

> **Note**: 화면 언어는 English로 설정하고 진행합니다. 언어 변경은 우측 상단의 **Language** 아이콘을 선택하고 변경할 수 있습니다.

### Instance 생성

1. 좌측 상단의 **햄버거 아이콘**을 클릭하고, **Compute**을 선택한 후 **Instances**를 클릭합니다.
   ![Compute Instances](images/compute-instance.png " ")

2. **Create instance**를 클릭합니다.
   ![Compute Instance Screen](images/compute-instance-screen.png " ")

3. 인스턴스 이름과 구획을 선택 합니다
   - Name: Enter **instanceForDemoApp**
   - Create in compartment: **OCIDevOpsHandsOn**
   - Availability domain : **AP-SEOUL-1-AD-1 (Seoul 리전 기준)**
     ![Compute Create #1](images/compute-create-1.png " ")

4. 설치할 이미지와 Instance의 Shape을 선택 합니다.
   - Image : **Oracle Linux8 - 2022.02.25-0**
   - Shape : **VM.Standard.E4.Flex (1 OCPU, 16 GB Memory)**
     ![Compute Create #2](images/compute-create-2.png " ")

5. 네트워크 관련 옵션을 선택 합니다
   - Virtual cloud network : **VCNforDevOpsHandsOn**
   - Subnet : **Public Subnet-VCNforDevOpsHandsOn**
   - Public IP address : **Assign a public IPv4 address**
     ![Compute Create #3](images/compute-create-3.png " ")

6. VM에 접속할때 사용할 SSH Keys 추가 합니다.
   - 이번 실습에서는 **Generate a key pair for me** 를 선택 후 Private Key, Public Key를 다운받아 잘 보관 합니다.
   - Boot volume 관련 옵션은 기본 설정을 유지 합니다.
     ![Compute Create #4](images/compute-create-4.png " ")

7. **Create** 버튼을 클릭 후 생성
   - 생성 후 Running 상태를 확인 합니다
     ![Compute Create #6](images/compute-create-6.png " ")

### Instance 접속 및 프로그램 설치

1. 인스턴스 접속
   - Windows 사용자 (PuttyGen , Putty 사용)
      - PuTTYgen을 실행합니다
      - **Load** 를 클릭 하고 인스턴스를 생성할 때 생성된 프라이빗 키를 선택합니다. 키 파일의 확장자는 **.key**
      - **Save Private Key** 를 클릭 합니다.
      - 키파일의 이름을 지정 합니다. (개인 키의 확장자는 **.ppk**로 저장합니다)
      - **Save** 를 클릭합니다.
      - 새로운 키 파일을 이용하여 인스턴스에 접속 합니다.
      - 상세내용은 링크를 통해 확인 가능 합니다. [접속 가이드 링크](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/accessinginstance.htm#linux__putty)
   - MacOS 사용자
      - 다운로드 받은 키파일의 권한을 조정합니다.
        ````shell
          <copy>
           chmod 400 <private_key_file> #엑세스 하려는 키 파일의 전체 경로 와 이름을 입력합니다.
          </copy>
         ````
      - 다음 명령어를 입력하여 인스턴스에 접속합니다.
        ````shell
          <copy>
           ssh -i <private_key_file> opc@<public-ip-address>
          </copy>
         ````
2. 기본으로 OS에 적용되어 있는 방화벽을 중지 시키기 위해 VM 연결 후 아래 코드를 순차적으로 입력 합니다.
    ````shell
      <copy>
      sudo firewall-cmd --permanent --zone=public --add-port=8080/tcp
      sudo firewall-cmd --reload
      sudo systemctl disable firewalld
      sudo systemctl stop firewalld
      </copy>
    ````
3. Spring Boot 및 Docker 실행을 위한 JDK를 설치합니다.
    ````shell
      <copy>
      sudo yum install jdk1.8.x86_64 -y
   
      #정상설치 확인
      java -version
      </copy>
    ````
   - 결과 출력 예시:
    ````shell
      java version "1.8.0_321"
      Java(TM) SE Runtime Environment (build 1.8.0_321-b07)
      Java HotSpot(TM) 64-Bit Server VM (build 25.321-b07, mixed mode)
    ````
4. Docker 설치 및 서비스 활성화
    ````shell
      <copy>
      # Docker 설치
      sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
      sudo yum install docker-ce docker-ce-cli containerd.io -y
      
      # Docker 서비스 시작 및 연결
      sudo systemctl start docker
      sudo systemctl enable docker
      
      # Docker 서비스 실행권한 조정
      sudo chmod 777 /var/run/docker.sock
   
      # 정상 설치 확인
      docker version
      </copy>
    ````
   - 결과 출력 예시:
    ````shell
      Client: Docker Engine - Community
      Version:           20.10.14
      API version:       1.41
      Go version:        go1.16.15
      Git commit:        a224086
      Built:             Thu Mar 24 01:47:44 2022
      OS/Arch:           linux/amd64
      Context:           default
      Experimental:      true
      
      Server: Docker Engine - Community
      Engine:
      Version:          20.10.14
      API version:      1.41 (minimum version 1.12)
      Go version:       go1.16.15
      Git commit:       87a90dc
      Built:            Thu Mar 24 01:46:10 2022
      OS/Arch:          linux/amd64
      Experimental:     false
      containerd:
      Version:          1.5.11
      GitCommit:        3df54a852345ae127d1fa3092b95168e4a88e2f8
      runc:
      Version:          1.0.3
      GitCommit:        v1.0.3-0-gf46b6ba
      docker-init:
      Version:          0.19.0
      GitCommit:        de40ad0
    ````
5. 기타 설정
   - sudo 명령어를 비밀번호 없이 사용할 수 있도록 설정 합니다.
    ````shell
      <copy>
      sudo vi /etc/sudoers
      
      # 파일 하단에 아래 내용 추가
      ocarun  ALL=(ALL)  NOPASSWD: ALL
      </copy>
    ````
   - OS 재부팅시 자동으로 Docker 명령어 권한을 지정하도록 설정 합니다._???재부팅하면 안됨 ㅠㅠ_
    ````shell
      <copy>
      sudo vi /etc/rc.local
      
      # 파일 하단에 아래 내용 추가
      sudo chmod 777 /var/run/docker.sock
      </copy>
    ````

## Task 2: Jenkins 및 플러그인 설치

## Task 3: OCI DevOps 서비스 호출을 위한 자격증명 설정

## Task 4: Pipeline 생성 및 테스트 진행


[다음 랩으로 이동](#next)
