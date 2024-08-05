## Redis Max Connection Client Error

#### 에러 발생 ####
ERR max number of clients 에러가 발생했다.  
매번 클라이언트의 수가 하나씩 증가했고, Establish 상태를 유지했다.
<img width="1017" alt="스크린샷 2024-08-06 오전 1 48 54" src="https://github.com/user-attachments/assets/1139e6d9-edf7-441d-83d4-06d816a26951">

#### 긴급 조치 ####
긴급조치로 timeout을 설정해 유휴상태가 100초 이상이면 연결이 끊기도록 했다.  

당연히 Redis를 사용하고 있는 부분은 BullQueue 뿐이라.   
가장 먼저 Bull Queue를 의심했다.  
하지만 그렇다 할 코드의 변경이 없었다.

#### 레디스 이슈 원인 추측 ####
가능한 원인을 하나씩 펼치고 가지치기를 해나갔다. 
- Backend
  - Code: o
  - Bull Queue: x
  - Redis: x
  - Nest: x
- Infra
  - Health Check: o

#### Redis Connection per Request?! ###
그 원인은 인프라 단의 Health Check를 변경하면서 발생하였다.  

이번 신규 기능을 개발하면서 Deprecated 모듈들을 정리하였다.  
임시로 Health Check를 담당하고 있던 api도 정리되었다.  
마침 누군가 개발해둔 health-check API가 있었다.  
그리고 이를 통해 인프라의 Health Check를 했다.  

이 API는 Redis의 현재 연결 상태를 확인하는 코드였다.  
그리고 매 요청마다 Redis Connection을 생성하고 있었다.  
결과적으로 매분마다 Health Check를 하니, 연결이 계속 늘어났던 것이다.    

#### Health Check용 API 생성 ####
Redis의 연결을 글로벌하게 관리하는 코드로 변경했다.  
그리고 terminus모듈을 이용해 Health Check용 API를 만들었다.  
더이상 connection수는 늘어나지 않았고 마음편하게 잠들었다.   

#### TIL ####
사소한 부분을 신경쓰자.  
당연하게 여기지 말고, 돌다리도 두드려보고 건너자.