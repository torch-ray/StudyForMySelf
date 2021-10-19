## 디렉토리별 Github 계정 설정하기
회사에서 회사 계정과 개인 계정을 별도로 사용하기 위함...  
일찍 출근해서 ps하고 커밋 하려고 정리하는 내용:)

#### SSH Keys 만들기

```
$ cd ~/.ssh
$ ssh-keygen -t rsa -C "public@my_email.com" -f "id_rsa_public"
$ ssh-keygen -t rsa -C "personal@my_email.com" -f "id_rsa_personal"
```

터미널을 열고 위와 같이 명령어를 입력합니다.  
public은 회사용, personal은 개인용이고 당연히 원하는대로 바꿔도 무방!  
my_email도 상황에 맞게 바꿔주시고요 ~ 
  
```
id_rsa_public
id_rsa_public.pub
id_rsa_personal
id_rsa_personal.pub
```
여기까지 진행하면 .ssh 디렉토리에 4개의 파일이 생성됐어야합니다.  
.pub 확장자 파일을 복사해서 github 계정에 추가해줄 예정입니다.  

#### Github 계정에 SSH Keys 추가하기

```
$ pbcopy < ~/.ssh/id_rsa_personal.pub
$ pbcopy < ~/.ssh/id_rsa_public.pub
```

personal을 복사해서 개인 github 계정에 추가해주시고 ~  
public을 복사해서 회사 github 계정에 추가해주시면 됩니다.  

github 계정에 SSH Key 추가하는 방법은  
- 본인 계정 아이콘을 클릭해서 "Settings" 클릭
- SSH and GPG Keys 클릭
- New SSH Key 클릭
- 제목 적어주시고 pbcopy 명령어로 복사한 내용을 붙여넣기
- Add SSH Key 눌러주면 끝~

#### Configuration 만들기

개인 계정을 사용할 디렉토리로 이동해주시고 ~ 

```
vi .gitconfig
```

gitconfig 파일을 열어서 

```
[user]
   user = 님이름
   email = 님@이메일.com
```

생성해주시면 되고 같은 작업을 회사 디렉토리로 이동해서도 해줍니다.  

```
vi .gitconfig
```

```
[user]
   user = 님이름 or 회사이름
   email = 회사@이메일.com
```

#### 로컬에 key identities 저장하기

```
$ cd ~
$ ssh-add -D
```

이건 이전에 세팅해놓은 정보 삭제구요 ~ 

```
$ ssh-add id_rsa_personal
$ ssh-add id_rsa_public
```

이건 이번에 생성한 정보 추가입니다!  

```
$ ssh-add -l
```

이렇게 잘 추가됐는지 확인해보고 

```
$ ssh -T github.com-personal
$ ssh -T github.com-public
```

위 두 명령어로 개인/회사 계정이 잘추가됐는지 확인해봅니다.  

```
Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```
	
잘됐으면 위와 같이 터미널에 메시지가 등장합니다.  

#### 레포 가져와서 테스트하면 끝

```
git clone git@github.com-personal:USERNAME/test-ssh.git
```

요걸"git@github.com-personal"  꼭 해줘야합니다.  
```
git clone git@github.com:USERNAME/test-ssh.git
```

원래 이런 주소라면 com 뒤에 -personal / -public 을 추가해줘야 된다는 의미!
