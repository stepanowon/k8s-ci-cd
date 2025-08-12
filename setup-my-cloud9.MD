# EKS - Bastion Cloud9 인스턴스 설정
------------------------
### 루트 어카운트 생성후 로그인하고 eks-admin 사용자 생성
- Mgmt Console 액세스 허용
- 관리자 권한 부여
- 로그인 링크 기억할 것. 다음과 같은 형식
  * https://111111111111.signin.aws.amazon.com/console
- eks-admin 사용자로 로그인(MFA 적용 권장)

### IAM Role 생성
- EC2에 대한 Cloud9-Role 역할 생성
- AdministratorAccess 권한 부여

### Cloud9 배스쳔 인스턴스 생성
- 서울 리전인지 확인
- Name: eks-bastion
- 나머지는 기본값 : Cost Saving을 위해 자동 종료 시간을 사용하는데 조정 가능함.

### EC2 인스턴스에서 볼륨 크기 조정, 
- eks-bastion 이름이 포함된 인스턴스의 세부 항목으로 들어가서 스토리지를 찾음
- vol-xxxxx 형태의 링크를 찾아서 클릭
- EBS 볼륨 화면에서 우측 상단의 '작업' 드롭다운을 클릭하여 '볼륨 수정' 클릭. 10G-->20G로 늘림
- 다시 eks-bastion ec2 인스턴스 화면으로 이동

### Cloud9 인스턴스에 Role 지정, Theme 지정
- EC2 서비스로 이동 eks-bastion 이름이 포함된 EC2 인스턴스 체크박스 체크
- '작업' 드롭다운에서 보안 - IAM 역할 수정 클릭
- Cloud9-Role 연결
- Cloud9 설정 화면으로 이동(우측상단 톱니바퀴 클릭)
- Themes 에서 밝은색으로 선택(예:Flat Theme) --> 선택적 작업
- AWS Settings - Credentials 화면에서 AWS managed temporary credentials를 off 상태로 변경
- 적용 후 터미널에서 다음 명령어 실행하여 기존 영구자격증명 삭제하고 Cloud9-Role로 자격증명이 변경되었는지 확인
  * rm ~/.aws/credentials
  * aws configure 명령 실행 후 region과 포맷만 설정 : ap-northeast-2, json
  * aws sts get-caller-identity 

### 유틸리티 도구 설치
~~~
## AWS CLI 업데이트
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --update

## 몇몇 유틸리티 설치
## * jq: json 파싱 유틸리티
## * bash_completion : bash 쉘 자동 완성 설정
sudo yum -y install jq bash-completion 

## kubectl 설치
cd ~
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.0/2025-05-01/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc

## eksctl 설치
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz"
tar -xzf eksctl_Linux_amd64.tar.gz
rm eksctl_Linux_amd64.tar.gz
chmod +x ./eksctl
mkdir -p $HOME/bin && mv ./eksctl $HOME/bin/eksctl && export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bash_profile

## kubectl 자동 완성 및 kubectl 대신 k 사용
sudo apt install bash-completion
source /usr/share/bash-completion/bash_completion

echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl k' >>~/.bashrc
source ~/.bashrc

## docker 업데이트
sudo yum install docker -y

## EBS 볼륨 확장
#크기를 조정할 파티션 확인
lsblk
#파티션 크기 조정
sudo growpart /dev/nvme0n1 1
#조정 여부 확인
lsblk
#파일 시스템 확장
sudo xfs_growfs -d /
#확장된 파일 시스템 확인
df -h
~~~
