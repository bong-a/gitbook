# Chocolatey

## Chocolatey란?

>  Chocolatey is a software management solution unlike anything else you've ever experienced on Windows. Chocolatey brings the concepts of true package management to allow you to version things, manage dependencies and installation order, better inventory management, and other features. 
>
> 출처 :  https://chocolatey.org/ 

<b>Chocolatey는 진정한 패키지 관리 개념을 도입하여 버전 관리, 종속성 및 설치 순서 관리, 더 나은 재고 관리 및 기타 기능을 제공합니다.</b>

리눅스의 apt-get,yum 그리고 macOS의 HomeBrew와 같은 개념의 패키지관리 도구

윈도우에서는 Chocolatey를 주로 쓰는 듯하다. 그 외에 윈도우에서 설치할수 있는 패키지관리도구로 `sdkman`가 있다.



## 설치 조건

- Windows 7+ / Windows Server 2003+
- PowerShell v2+
- .NET Framework 4+ (the installation will attempt to install .NET 4.0 if you do not have it installed)

## 설치 방법

1. 파워셀을 관리자 권한으로 연다.

2. 아래 명령어를 실행

   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
   ```

   - 자세한 설치는  https://chocolatey.org/install 를 참고.

3. 설치 확인

   ```powershell
   choco -? 	# 0.10.15 버전이 뜨면 설치 완료
   ```



## 명령어

choco 명령어 :  https://chocolatey.org/docs/commands-reference 

1.  choco upgrade [package name]  : 패키지 업그레이드 명령어

     choco upgrade all : 설치된 모든 패키지 업그레이드

2. choco search [package name] : 패키지 검색

3. choco install [package name] : 패키지 설치

   ```
   # 설치 명령어는 powerShell-관리자모드 에서만 설치명령어가 올바르게 작동한다.
   # --version : 설치할 패키지 버전 명시
   # -m : 버전 여러개 설치하는거 허용 여부 
   choco install ruby --version 1.9.3.55100 -my
   ```





