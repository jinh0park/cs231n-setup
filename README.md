# Tutorial for cs231n Assignment Setup with GCP(Google Cloud Platform)

원래 cs231n assignments를 위한 Tutorials는 [cs231n 공식 Github](http://cs231n.github.io/)에 잘 정리되어 있다. 하지만 여기를 그대로 따라하다 보면, 높은 확률로 **Create an image from our provided disk** 파트에서 막히고 말 것이다. Disk Image Source의 Repo(Bucket) Path를 입력하면 '개체가 없거나 액세스 권한이 없으므로 개체를 사용할 수 없습니다. 대신 개체를 탐색해보세요'라는 에러 문구가 출력되면서 다음 단계로 넘어갈 수가 없다.

![](.img/1.JPG)

검색을 해보니 Stanford에서 cs231n의 학기가 끝나면 이 Bucket의 접근 권한을 막아둔다고 한다. 이 Disk Image에는 anaconda, torch, tensorflow, CUDA 등 assignments에 필요한 패키지들을 설치해놓은 것 같은데, 이 Image를 사용할 수 없으므로 따로 세팅을 해야한다(이 튜토리얼을 만든 가장 큰 이유이기도 하다).

클라우드 컴퓨팅, 리눅스 등에 익숙하지 않은 본인 같은 초심자가 assignments를 하는데 어려움이 없도록 Tutorial들을 작성하였으며, 목차는 크게 다음과 같다.

1. **GCP(Google Cloud Platform) Tutorial**  
과제를 수행하기 위한 Compute Instance를 생성하는 방법에 대해 다룬다.
2. **Instance Setup Tutorial**  
Instance를 생성한 후, cs231n에서 제공하는 Disk Image 없이 과제를 수행하기 위한 환경을 세팅하는 방법에 대해 다룬다.
3. **Jupyter Notebook Tutorial**  
과제는 보통 Jupyter Notebook 상에서 이루어진다. 이를 쉽게 설정하고 구동하는 방법에 대해 다룬다.
4. **PyTorch + CUDA Setup Tutorial (for Assginment 2, 3)**  
assignment 1까지는 GPU를 달 일이 없지만, assignment 2, 3를 수월하게 하기 위해서는 GPU를 Instance에 달고, CUDA와 Pytorch를 설치하는 방법에 대해 다룬다.

---

## ※필독※

클라우드 컴퓨팅 서비스를 이용하면서 가장 신경써야할 부분은 요금이다. 인스턴스 하나마다 시간당 적게는 0.2달러 정도에서 GPU를 달면 시간당 0.4달러 이상까지 올라간다. 만약 인스턴스를 한달 내내 켜놓고 있으면, **수백달러가 넘는 비용** 이 월말에 청구될 수 있다.

따라서, 사용하지 않는 인스턴스는 반드시 중지를 해야한다. 본인은 AWS(Amazon Web Service)를 쓰다가 GCP로 넘어왔는데, GCP는 중지한 인스턴스에 대해서 요금을 부과하지 않는 것이 좋다고 느껴졌었다. AWS는 중지를 해도 인스턴스가 남아있으면 그것에 대해 계속 요금을 부과하기 때문에 관리가 까다로웠기 때문이다. (최근에 중지중에도 요금을 부과하지 않게 하는 방법이 나왔다고는 한다) 아무튼, **인스턴스를 쓰지 않을 때에는 중지시키는 습관을 들이자!**

---

## GCP(Google Cloud Platform) Tutorial

### 인스턴스 생성하기

[cs231n 공식 GCP Tutorial](http://cs231n.github.io/gce-tutorial/)에서 **Create and Configure Your Account** 파트까지 그대로 따라한다. 달리 어려운 작업은 없고, 계정을 만들고 카드를 등록하는 간단한 과정이다.

과정을 따라서 계정을 등록했으면, **Compute Engine** 메뉴에 들어간다. 생성해놓은 인스턴스가 없다면 아래와 같은 화면이 나올 것이다.

![](.img/2.JPG)

**만들기** 버튼을 누르고 인스턴스를 생성한다.

![](.img/3.JPG)

인스턴스 이름과 스펙을 설정한다. 맞춤설정을 하지 않고 왼쪽의 드롭다운에서 vCPU 8개를 선택하면 그에 알맞는 메모리 용량을 같이 할당해준다.  

맞춤설정을 누르면 GPU를 추가할 수 있는데, assignment1과 assignment2 초반에는 GPU를 사용하지 않기 때문에 굳이 추가하지 않아도 된다. GPU를 달면 그만큼 비용이 많이 발생하기 때문에(GPU가 특히 비싸다), GPU가 필요한 부분에서만 인스턴스를 수정하여 GPU를 추가하는 것이 경제적이다.

![](.img/6.JPG)

![](.img/4.JPG)

인스턴스의 부팅 디스크를 설정한다. 여기서 **변경** 버튼을 누르고 `Ubuntu 16.04 LTS`를 선택한다. 디스크의 크기는 적당히 40GB 정도로 한다. 어차피 요금 얼마 안나온다.

![](.img/5.JPG)

그 외 옵션들을 설정한다. Jupyter Notebook으로 접근해야하므로 HTTP, HTTPS 트래픽을 모두 허용하고, **인스턴스 삭제 시 부팅 디스크 삭제** 체크박스를 해제한다.

여기까지 따라 왔다면 **만들기** 버튼을 눌러 인스턴스를 생성한다.

![](.img/7.JPG)

성공적으로 인스턴스를 생성했다면, 실행 중인 인스턴스의 모습이 보일 것이다.

### 고정 IP 주소 할당하기

**탐색 메뉴** -> **VPC 네트워크** -> **외부 IP 주소** 로 들어가서 **고정 주소 예약** 을 클릭한다.

![](.img/12.JPG)

위 그림과 같은 설정으로 고정 주소를 예약한다. **연결 대상** 에는 우리가 위에서 방금 만든 인스턴스를 선택하면 된다.

![](.img/13.JPG)

조금 기다린 후 **Compute Engine** 탭에 다시 들어오면, 인스턴스에 고정 IP 주소가 할당된 모습을 볼 수 있다.


### 방화벽 규칙 설정하기

우리는 Jupyter Notebook으로 작업을 할 것이기 때문에 방화벽 설정에서 포트를 하나 열어주어야한다. **탐색 메뉴** -> **VPC 네트워크** -> **방화벽 규칙** 으로 들어간다.

![](.img/9.JPG)

**방화벽 규칙 만들기** 를 클릭해 새로운 규칙을 하나 만든다.

![](.img/10.JPG)

다른 설정들은 건드리지 않고, **대상: 네트워크의 모든 인스턴스**, **소스 IP 범위: 0.0.0.0/0**, **프로토콜 및 포트: tcp:8888** 로 하고 **만들기** 를 한다. 규칙 이름은 아무거나 상관 없다. 본인은 cs231n-firewall라는 이름으로 규칙을 생성했다.

![](.img/11.JPG)

성공적으로 방화벽 규칙이 생성되었다!

---

## Instance Setup Tutorial

cs231n Github의 [Setup Tutorial](http://cs231n.github.io/setup-instructions/)에서 **Working locally** 부분을 따라하면 된다. `Anaconda`와 python `virtualenv`를 사용하는 방법이 있는데, `Anaconda`를 이용한 방법이 좀 더 간단하므로 그것을 선택했다.  

한 가지 **주의사항** 이 있는데, 각 assignment 폴더에 있는 `requirements.txt` python virtualenv를 사용할 때만 설치해주어야한다. `conda` 가상환경 내에서 해당 패키지들을 설치하면 이미 설치된 패키지들과 충돌을 일으키면서 난리가 나므로... 해당 파일들은 깔끔하게 무시해주자.

### anaconda3 설치하기

```
sudo apt update
wget https://repo.continuum.io/archive/Anaconda3-2018.12-Linux-x86_64.sh
bash Anaconda3-2018.12-Linux-x86_64.sh
```

yes, yes를 하면서 진행하다 보면 설치가 완료된다. 다만 anaconda3가 설치된 후 `Do you wish to proceed with the installation of Microsoft VSCode? [yes|no]`라면서 VSCode의 설치를 권유하는데, 여기서 굳이 VSCode를 깔 이유는 없으니 `no`를 입력한다.

### conda 가상환경 생성하기

우선 `conda` 커맨드를 사용하기 위해서는 PATH 등록을 해주어야 한다. 다음 명령어를 입력한다(`<user_directory>`은 ubuntu user name으로 바꾸어 주어야 한다. SSH shell(아래 그림)에서 빨간 동그라미를 친 부분이 username에 해당한다).

![](.img/8.JPG)

```
export PATH=/home/<user_directory>/anaconda3/bin:$PATH
```

conda 가상환경을 생성해준다. `-n cs231n`은 가상환경의 이름을 cs231n으로 지정한 것이고, 그 뒤는 python의 버전의 명시(python 3.6이 가장 stable한 것 같다)와, anaconda의 패키지들을 상속하겠다는 것을 의미한다.

```
conda create -n cs231n python=3.6 anaconda
```

다소 긴 시간을 기다려서 가상환경 생성이 완료되면, 가상환경을 activate / deactivate하는 방법에 대해서 알려주는 문구가 출력된다. 제대로 생성이 되었는지 확인하기 위해 다음 명령어로 가상환경을 activate 해본다.

```
source activate cs231n
```

입력창 왼쪽에 `(cs231n)`이라고 가상환경이 activate 되었음을 나타내는 괄호 표시가 뜨면 성공이다. 확인을 했으므로  `deactivate` 명령으로 가상환경을 deactivate 한다.

### assignment code 다운로드

assignment code 파일들은 모두 cs231n 공식 github에 있지만, 편의를 위해 본인의 github repo에도 올려놓았다. 다음 명령어로 assignment code들을 다운받는다.

```
git clone https://github.com/jinh0park/cs231n-setup.git
```

그 후, 각 Assginment에 필요한 dataset들도 미리 다운받아주자.

```
cd cs231n-setup
bash assignment1/cs231n/datasets/get_datasets.sh
bash assignment2/cs231n/datasets/get_datasets.sh
bash assignment3/cs231n/datasets/get_datasets.sh
```

---

## Jupyter Notebook Tutorial

Python을 어느 정도 공부해본 사람이라면 Jupyter Notebook을 써본 경험이 있을 것이다. Local 환경에서 구동할 때는 주로 `jupyter notebook` 명령어로 `localhost:8888`에 Notebook을 띄워서 실행하는데, 클라우드에서 이를 이용할 때는 몇 가지 설정이 더 필요하다. 가장 중요한 설정은 password라고 생각하는데, local에서 작업할 때는 다른 누군가가 내 Notebook에 들어올 염려가 없지만, 클라우드에서 작업을 할 때는 고정 IP를 주소창에 입력하면 누구든 내 Notebook에 들어올 수 있기 때문에 password를 설정하는 작업이 필요하다. (이는 IP주소를 제한한다던지, 기타 여러 방법이 있지만 GCP는 이게 가장 편한 것 같다)

이제, 인스턴스를 켜고 `ipython` 명령어를 친다. `ipython` 콘솔이 켜지면 순서대로 `from IPython.lib import`, `passwd()`를 입력해 비밀번호에 대한 sha1 해시값을 받는다. (아래 참고) `ipython` 명령어를 찾지 못한다고 하면 . 위에서 한 번 썼던 `export PATH=/home/<user_directory>/anaconda3/bin:$PATH`를 다시 입력해 Path에 추가해준다.

```
ubuntu@cs231n:~$ ipython
Python 3.7.1 (default, Dec 14 2018, 19:28:38)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.2.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from IPython.lib import passwd                                                                             

In [2]: passwd()                                                                                                   
Enter password:
Verify password:
Out[2]: 'sha1:90b7ff928564:e704593562aa65899ad09940fd9ed4145799cdde'
```

마지막에 출력된 `sha1:~~~`를 복사하여 메모장 같은 곳에 적어둔다.

`Ctrl+Z`를 눌러 `ipython`을 종료한 후, 다음 명령으로 config 파일을 생성한다.

```
jupyter notebook --generate-config
```

https 연결을 위한 certificates를 만든다.

```
mkdir certs

cd certs

sudo openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem
```

모두 `Enter`키를 치고 넘어가도 된다.

그 후 전에 생성한 Jupyter Notebook config file을 수정한다.

```
cd ~/.jupyter/

vi jupyter_notebook_config.py
```

주석으로 가득한 파일이 나오는데, 아무데나 편한대로(맨 윗줄도 괜찮다) 다음 코드들을 붙여넣는다. 주의할 점은, 아래 코드에서 `c.NotebookApp.certfile`의 위치와 `c.NotebookApp.password`의 해시값(메모장에 적어둔 그 값)을 각자 알맞는 값으로 수정해주어야 한다.

```
c = get_config()
# Kernel config
c.IPKernelApp.pylab = 'inline'  # if you want plotting support always in your notebook
# Notebook config
c.NotebookApp.certfile = u'/home/<user_directory>/certs/mycert.pem' #location of your certificate file
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.open_browser = False  #so that the ipython notebook does not opens up a browser by default
c.NotebookApp.password = u'sha1:b962c2993b85:3e45a67aa4adf59909c8deb322e6e016bdf8d671'  #the encrypted password we generated above
# Set the port to 8888, the port we set up in the AWS EC2 set-up
c.NotebookApp.port = 8888
```

수정을 했으면, assignments들이 모여있는 폴더로 이동해서 앞서 생성한 `conda` enviroment를 활성화한 후 Jupyter Notebook을 실행한다.

```
cd ~/cs231n-setup
source activate cs231n
jupyter Notebook
```

Jupyter Notebook을 실행했으니 이제 웹브라우저에서 접속해보자. 위에서 할당했던 ip주소 + 포트번호(8888)을 웹브라우저에 입력하면 Jupyer Notebook에 접속할 수 있다. 단, 크롬의 경우 아래와 같은 경고 창이 뜨는데, 우리가 만든 certificates가 공인되지 않았기 때문에 발생하는 당연한 결과이다. 경고를 무시하고 **고급** 을 눌러 **~~~(안전하지 않음)으로 이동** 을 눌러준다.

![](.img/14.JPG)

![](.img/15.JPG)

비밀번호를 입력하면, Local에서 작업할때와 같은 Jupyter Notebook의 화면이 성공적으로 나온 것을 볼 수 있다.

## PyTorch + CUDA Setup Tutorial (for Assginment 2, 3)

### Pytorch 설치

```
source activate cs231n
pip install torch
```

### GPU 추가

assignment2의 후반부부터 GPU를 사용해야 하는(pytorch/tensorflow를 쓸 때) 과제가 나온다. GCP를 처음 사용하면 GPU의 한도가 0으로 되어있어 GPU를 인스턴스에 추가할 수 없다. 그럴때는 할당량을 직접 늘려야하는데, 이 과정이 하루정도 소요될 수 있으니 미리 한도를 늘려놓기를 권장한다.

![](.img/18.JPG)

인스턴스 세부사항-수정에 들어가서 GPU를 위와 같이 추가하면 문제 없이 인스턴스가 잘 수정이 된다. 하지만 수정을 한 후 인스턴스를 시작하면, 다음과 같은 오류 문구가 뜨면서 인스턴스가 시작하지 않는다.

![](.img/17.JPG)

위에서 말한 할당량 문제인데, 이를 해결하기 위해 **탐색 메뉴** -> **IAM 및 관리자** -> **할당량** 탭으로 들어간다.

![](.img/16.JPG)

![](.img/19.JPG)

**측정항목** 에서 **GPUs** 를 고르면, 할당량이 0으로 되어있는것을 확인할 수 있다. 할당량 수저을 눌러 이를 원하는 수치로 입력 후 제출을 누르면 GCP 측에 요청이 전달되며, 관련된 메일들이 오는 것을 확인할 수 있다. 하루 정도 기다리면 한도가 정상적으로 늘려졌다는 메일과 함께 GPU를 달은 인스턴스를 정상적으로 실행할 수 있게 된다.

### CUDA 설치

GPU를 달았지만, pytorch에서 GPU 연산을 이용하기 위해서는 CUDA가 설치되어 있어야한다.

우선, 다음 명령으로 nvidia driver를 설치한다.

```
sudo apt update
sudo apt upgrade
sudo apt install nvidia-smi
```

다음으로는 CUDA를 설치하기 위해 shell script를 하나 만들어준다.

```
vi setup_cuda.sh
```

해당 파일에 아래 내용을 copy&paste한다.

```
#!/bin/bash
echo "Checking for CUDA and installing."
# Check for CUDA and try to install.
if ! dpkg-query -W cuda-10-0; then
  # The 16.04 installer works with 16.10.
  curl -O http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_10.0.130-1_amd64.deb
  dpkg -i ./cuda-repo-ubuntu1604_10.0.130-1_amd64.deb
  apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
  apt-get update
  apt-get install cuda-10-0 -y
fi
# Enable persistence mode
nvidia-smi -pm 1
```

작성한 스크립트를 sudo로 실행해주면, CUDA 설치가 진행된다.

```
sudo bash setup_cuda.sh
```

설치를 완료했으면 `conda` 환경에서 `python`을 실행한 후 다음 명령어를 통해 CUDA를 사용할 수 있는지 확인할 수 있다.

```
from torch.cuda import is_available
is_available()
```

설치가 잘 되었다면 `True`를 아니라면 `False`를 반환한다.
