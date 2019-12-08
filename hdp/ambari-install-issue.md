## Ambari - Client Server 설치 과정 중 이슈 해결

------

## INDEX

1. [Transparent Huge Pages Issues](#transparent-huge-pages-issues )

2. [Service Issues](#service-issues)

3. [Hostname Resolution Issues](hostname-resolution-issues)

------



### Transparent Huge Pages Issues 

The following hosts have Transparent Huge Pages (THP) enabled. THP should be disabled to avoid potential Hadoop performance issues.  -->  <b>'THP disable.'</b>

- THP란? Transparent Huge Pages의 약자.  자세한 내용은 http://allthatlinux.com/dokuwiki/doku.php?id=thp_transparent_huge_pages_%EA%B8%B0%EB%8A%A5%EA%B3%BC_%EC%84%A4%EC%A0%95_%EB%B0%A9%EB%B2%95 참고.

  

- THP 옵션 활성화 여부 확인 방법

  - 설정 여부 확인

    ```sh
    cat /sys/kernel/mm/transparent_hugepage/enabled
    
    >> [always] madvise never #활성화된 상태
    >> always madvise [never] #비활성화된 상태
    ```

  - 메모리 확인

    ```shell
    cat /proc/meminfo
    
    #아래 항목들이 모두 0이어야 비활성화된 상태. -> 아래 출력 예시는 활성화상태임.
    AnonHugePages:    598016 kB
    HugePages_Total:       0
    HugePages_Free:        0
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    ```

    <!--THP 비활성화 후 재부팅하고 다시 메모리 확인했을 때, AnonHugePages의 크기는 확연히 줄었으나 완전히 0kb로는 내려가지않았다... -->

    

- THP 옵션 비활성화

  ```shell
  sudo apt-get install hugepages
  sudo hugeadm --thp-never
  #재부팅후에 해당 설정이 유지되지 않기때문에 '/etc/rc.local'에 다음을 추가해준다.
  sudo /bin/sed -i '$i /usr/bin/hugeadm --thp-never' /etc/rc.local
  ```

  <!--rc.local파일은 부팅시 자동 실행명령어 스크립트이다. 서버 부팅시마다 매번 자동 실행되길 원하는 명령어는 rc.local에 넣어준다.-->

----



## Service Issues

The following services should be up `ntp or chrony` --> <b>해당 서비스 실행</b>

- <b>ntp</b> : Network Time Protocol의 약자,  인터넷상의 시간을 정확하게 유지시켜 주기 위한 통신망 시간 규약
- <b>chrony</b> : RHEL 7에서 기본으로 제공하는 **NTP daemon/client** 



- 시간 동기화 확인 : `NTP synchronized: yes`로 되어있어야 동기화 설정 되어 있는 것임.

  ```shell
  timedatectl
  
  >> Local time: 월 2019-03-04 22:54:30 KST
    Universal time: 월 2019-03-04 13:54:30 UTC
          RTC time: 월 2019-03-04 13:54:18
         Time zone: Asia/Seoul (KST, +0900)
   Network time on: yes
  NTP synchronized: yes
  ```

  

- NTP 설치

  ```shell
  sudo apt-get install ntp
  ```

- NTP 확인

  ```shell
  ntpq -p
  ```

- NTP 설정 

  ```shell
  sudo vi /etc/ntp.conf
  
  #아래 내용 추가?
  server 1.kr.pool.ntp.org 
  server 1.asia.pool.ntp.org 
  server time.bora.net
  ```

  



## Hostname Resolution Issues

Not all hosts could resolve hostnames of other hosts. Make sure that host resolution works properly on all hosts before continuing.  --> <b>다른 서버에도 추가하려는 서버 호스트 추가</b>

<!-- hosts파일에 추가하려는 서버 추가하고 다시 체크해봐도 해결 상태로 넘어가지 않는다. 해당 이슈를 해결하지 않고 설치 진행한 결과 아무 이상은 없었다. 그래도 혹시 몰라 리서치를 해본 결과, ambari 서버를 재시작해보라는 답변을 찾았다...-->



## Hive start failed because of ambari error: mysql-connector-java.jar due to HTTP error: HTTP Error 404: Not Found

Ambari Server:(Ref: https://discuss.pivotal.io/hc/en-us/articles/115001611807-Hive-Services-Fail-to-Start-giving-Error-HTTP-Error-404-Not-Found- )

```shell
sudo yum install mysql-connector-java*
# ubunbu의 경우 다른 namenode 에서 카피해서 아래와 같이 진행함....
ls -al /usr/share/java/mysql-connector-java.jar
cd /var/lib/ambari-server/resources/
ln -s /usr/share/java/mysql-connector-java.jar mysql-connector-java.jar
```

