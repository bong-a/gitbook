# Ambari Agent 설치 가이드

## 사전 준비

- 에이전트 서버
- 암바리 서버

## 사전 작업

ambari server의 bigstation계정(ambari server 호스트)의 공개키를 agent 서버의 root계정의 ssh authorized_keys에 등록

```shell
# ambari server 측의 공개키
# 사용자 계정(ambari server 호스트)
cat ~/.ssh/id_rsa.pub
```

```shell
ssh-rsa Q2WQE123DQWDWDDAQABAAABAQDLxgBa4Z4gY5ALVNOKnj... public key username@nn01
```

위의 공개키를 agent서버에 등록

```shell
# ambari agent 
# root 계정이어야함
echo [pubkey] >> ~/.ssh/authorized_keys
```

- Agent 서버의 root계정에 ambari server의 공개키가 등록되어있어야 한다. 

- [SSH 터널링](https://swalloow.github.io/ssh-tunneling)

  

## Agent 추가  방법

### 0. Hosts -> Actions -> Add New Hosts

### 1. Install Options

   - `TargetHosts`에 추가할 agent 서버 호스트 네임을 넣어준다.

   - `Host Registration Information` - Provide your [SSH Private Key](javascript:void(null)) to automatically register hosts 선택

   - ambari server의 bigstation계정(ambari server 호스트)의 private key를 ssh private key에 넣어준다.

     ```
     # ambari server 측의 비밀키
     cat ~/.ssh/id_rsa
     ```

     - SSH User Account : root
     - SSH Port Number : 22

### 2. Host Checks

   - `All host checks passed on 1 registered hosts. [Click here to see the check results.](http://nn01:8080/#)` 해당 메세지 출력시에 해결하는게 좋음

   - warning으로 뜨는 부분 해결해야함

   - 대부분 추가하려는 agent 서버의 미설정부분일 확률이 큼

   - `Ambari - Client Server 설치 과정 중 이슈 해결` 참고

     

### 3. Assign Slaves and Clients

   - `client` 체크 
   - datanode 확장 시에는 `datanode`와 `nodeManager` 체크

### 4. Configurations

   - 특별히 설정할게 없다면 default 그대로  유지

### 5. Review

   - 설치 전 구성설정 확인
   - deploy 클릭

### 6. Install, Start and Test

   - 설치 및 테스트 단계라 수 분 소요된다.
   - 아래 메세지 뜨면 완료

   ```
   The cluster consists of 8 hosts
   Installed and started services successfully on 1 new host
   ```

   