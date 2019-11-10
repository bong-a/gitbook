# Docker For Windows 10 Pro

## 설치





## Running In Kitematic

### 설치

### 활용 방법



## Running In Git Bash(MINGW64)

## 설치 

git 설치할때 git bash 도 같이 설치

git bash를 실행하면 리눅스 명령어를 사용할 수 있음.



### Docker로 Jekyll 실행하기

```bash
winpty docker run -v "/$PWD":/srv/jekyll -p 4000:4000 --rm --name jekyll -it jekyll/jekyll jekyll serve --trace
```



### 이슈1

```
the input device is not a TTY.  If you are using mintty, try prefixing the command with 'winpty'
```

- 해결방법

  맨앞 명령어 앞에 `winpty`를 넣는다

  ```bash
  winpty docker run -v ~
  ```

- 매번 `winpty` 입력하기 귀찮으면 다음과 같이 alias 추가

  ```bash
  echo "alias docker='winpty docker'" >> ~/.bash_profile
  source ~/.bash_profile
  ```

  

### 이슈2

컨테니어 실행시 경로 끝에 `;C` 가 붙는 경우

```bash
# case1
The source path "/c/Users/~/;C"
# case2
 OCI runtime create failed: container_linux.go:346
: starting container process caused "exec: \"C:/Program Files/Git/u
sr/bin/bash.exe\": stat C:/Program Files/Git/usr/bin/bash.exe: no s
uch file or directory": unknown.
```

- 원인

  mingw가 경로에 대해 변환하려고함. (` x;x;C:\MinGW\msys\1.0\x `)

- 해결방법

  - 경로변환을 피하기 위해 `/`를 활용

  - `-v "/$PWD:/srv/jekyll"` 이 아닌`-v "/$PWD":/srv/jekyll`로 명령어 설정  

  ```bash
  # ERROR 
  winpty docker run -v "/$PWD:/srv/jekyll"
  # RESOLVED
  winpty docker run -v "/$PWD":/srv/jekyll
  winpty docker run --rm -it ~ //bin/bash
  ```

  