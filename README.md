# geth를 활용한 이더리움 네트워크 구축1

[참고]
링크-> [도커문법 tutorial]
https://github.com/pilsa0327/docker-tutorial 

### A. 이더리움 이미지 받기  
`$ docker pull pjt3591oo/ethereum-geth:1.90`

### B. 이더리움 컨테이너 생성 및 실행  
`$ docker run -it --name ethereum.geth.com -p 8545:8545 -p 30303:30303 pjt3591oo/ethereun-geth:1.90 /bin/bash`

### C. genesis block 생성    
 
 cf) 블록에 담기는 정보
 - Height : 블록의 높이, 몇 번째 블록인지를 나타냄  
 - TimeStamp : 해당 블록 생성 시간  
 - Hash : 해당 블록 해시값  
 - Parent Hash : 이전 블록의 해쉬값  
 - Mined by : 블록을 생성한 지갑 주소, 명시된 지갑주소로 블록 보상 지급  
 - Block Reward : 블록을 생성하고 얼마만큼의 보상을 제공했는지를 나타냄  
 - Difficulty : 난이도, 블록 생성 속도에 따라 조정  
 - Total Difficulty : 지금까지 생성된 모든 블록의 Difficulty 총합  
 - Size : 해당 블록의 크기, 블록 사이즈 제한 존재  
 - Gas Used : 해당 블록에 포함된 트랜잭션의 Gas 사용량 총합 (Gas란? 트랜잭션이 실행할 떄 소모되는 수수료)  
 - Gas Limit : 해당 블록에 포함된 트랜잭션 Gas 한도의 총합  

#### 1. 계정 생성
 노드의 데이터를 쌓을 디렉터리 생성 후 계정 생성($PWD 현재 경로를 나타냄)  
   `$ mkdir node1`  
   `$ cd node1`  
   `$ geth --datadir $PWD account new`  
  -> 이렇게 생성된 계정은 keystore파일 형식으로 저장
  -> keystore파일은  비밀번화와 함께 개인키를 만들어주는 내용 포함, 개인키보다 보안 측면에서 좋음

#### 2. 제네시스 파일 생성
 `$ geth --datadir $PWD init genesis1.json`  
 node1 디렉토리에 json파일(여기서는 genesis1.json)를 생성하고 제네시스 블록 생성  

 genesis1.json 참고  
   alloc: 초기에 이더리움을 할당하는 부분, 앞서 생성한 계정 주소를 삽입  
   balance: 초기에 할당하는 이더리움의 양, (이더리움에서의 기본 단위 wei, 1ether = 10^18wei  

#### 3. 노드 실행 및 제네시스 블록 확인  
 `$ geth --detadir $PWD --networkid 1234 console`  
 -> --networkid : 프라이빗 네트워크 실행시 필요한 옵션, 1 ~ 4까지는 메인넷과 테스트넷이기때문에,   
 1~4를 제외하고 아무 수치 삽입  
 -> console : console를 넣어주면 이더리움과 명령어를 주고받을 수 있는 프로그램 실행 상태 유지  
 (넣지않아도 이더리움 노드는 실행됨)         
 cf) geth를 console 모드로 실행시 사용할 수 있는 모듈  
 admin, debug, eth, ethash, miner, net, personal, rpc, txpool, web3  
 콘솔 화면에서 입력하면 해당 모듈 사용법을 알 수 있음   
 
 
ex)   
 > `ether.bloackNumber` : 블록 개수 확인  
 > `eth.getBlock(블록 번호)` : 블록 정보 확인  
 > `eth.accounts` : 지갑 주소 리스트 출력  
 > `personal.listWallets` : 지갑 리스트 조회(키스토어 파일 위치, locked/unlocked 상태, 주소 출력)  
 > `eth.getBalance(지갑 주소)` : 계정 잔액 조회  
 > `personal.newAccount(비밀번호)` : 지갑 생성  

 cf) 이더리움 지갑 주소 구분  
 EOA(External Owner Account) : 외부 소유 계정, 앞선 방식으로 만들어진 어카운트  
 CA(Contract Account) : 스마트 컨트랙트를 배포하면 발생하는 어카운트 

  이더리움 어카운트에 저장되는 정보  
   balance: 이더 보유량  
   nonce : 트랜잭션 발생 횟수  
   code : 컨트랙트 코드  
   data : 데이터 저장 공간  

  차이점
  1) EOA는 code 와 data가 비어있음, CA는 code 와 data 안에 스마트 컨트랙트 코드와 관리하는 데이터 존재
  2) private key의 존재 여부 : EOA만 private key 소유, 따라서 CA는 자체적으로 트랜잭션 발생 불가
    EOA가 CA의 특정 기능을 호출하여 기능을 호출한 EOA의 private ket 로 해당 트랜잭션에 서명하여 EOA가 트랜잭션을 발생 


### D. 트랜잭션 발생하기  
   `> eth.sendTransaction({from: 보내는 계정, to: 받는 계정, value: 전송량})`  
      트랜잭션 발생

   -> 이더를 보내는 계정의 status가 Locked 상태이면 트랜잭션 발생 시 에러 발생    
   -> `personal.unlockAccount`(계정, 비밀번호) : 해당 계정 잠금 해제  

  `> eth.getTransaction(트랜잭션 해시)` : 트랜잭션 조회    
  `> eth.pendingTransactions` : pending 상태 트랜잭션 조회  
  --> 이 과정까지는 단순히 트랜잭션을 발생했을 뿐, 실제적으로는 트랜잭션이 처리되지 않아서
      이더 전송이 되지 않음.

### E. 트랜잭션 처리(마이닝)  
   `> eth.coinbase`  
    블록 생성 시 보상받을 지갑 확인  
   `> miner.setEtherbase`(이더 계정)  
   보상받을 지갑 수정  
   `> miner.start()`  
   마이닝 시작
   -> DAG(마이닝을 하기위해 필요한 파일)가 설치됨, .ethash에 저장 
   -> 생성된 블록에 트랜잭션이 성공적으로 들어가면 이더 전송이 완료됨
   -> getBlock()을 통해 해당 블록 정보를 조회하면 트랜잭션 해시가 포함된 것을 확인가능












