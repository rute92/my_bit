# 객체 인식 및 알고리즘

## 객체 인식
### darknet 오픈소스 이용한 특정 객체 인식 및 추적.
#### 컨셉 2가지
Jetson nano board <----TCP----> 2070 GPU Desktop PC (192.168.0.38)  
Jetson nano board <----TCP----> Jetson TX2 board (192.168.0.xx)
#### nano 보드
1. camera 이미지를 소켓으로 전송

#### TX2 또는 데스크탑
1. 소켓으로 이미지를 받아서 darknet 으로 객체 인식  
2. 특정 클래스(현재는 사람class) 감지해서 일정 시간이상 감지 시 이미지 저장.  
3. 추적 모드일 때 강아지 class의 box 좌표 및 크기로 추적. (추적관련 제어는 파일로 저장)  
-----------
#### 그외  

Jetson nano board의 환경은 Linux여서 ip만 바꿔주면 되지만 darknet을 사용 할 환경은 Linux 또는 Windows임.  

darknet 소스도 조금 다르고 socket 통신 구현도 조금 다름. 각각 프로그램 구현.  
(pjreddie darknet과 alexeyAB darknet이 있으며 전자는 linux만, 후자는 linux window 다됨.  
  alexeyAB darknet이 기능이 더 많아 이걸 사용할 예정)

## 알고리즘
### A* 알고리즘 (최단거리 경로 찾기)
* 참조링크
> <http://egloos.zum.com/cozycoz/v/9748811> (설명)  
> <https://alleysark.tistory.com/102> (소스)  
> <https://withmule.tistory.com/16> (소스)

1. 탐색 영역 둘러보기
시작지점(A) 및 목표지점(B) 설정. 그리고 탐색 지역을 네모난 grid 로 나누어 단순화시킴.  
각 사각형을 노드라고 칭함.

2. 탐색 시작
A점부터 B까지 인접 사각형을 확인하며 길을 만들어나감.  
  a. 시작점 A를 '열린 목록'에 넣음. 일종의 장바구니  
  b. 시작점에 인접한 이동 불가능한 노드는 무시하고 이동 가능한 노드를 '열린 목록'에 넣어줌.  
      이 노드들의 부모를 시작점 A로 지정.  
  c. '열린 목록'에서 시작점 A 노드를 삭제하고 다시 볼 필요 없는 '닫힌 목록'에 추가  
  d. '열린 목록'에 있는 노드 중 가장 작은 비용(F)를 가진 하나를 선택해 위 순서로 반복 처리.  

3. 경로 채점
비용(F) 채점하기  
F = G + H
- G : 시작점 A로부터 현재 노드까지의 경로를 따라 이동하는데 소요되는 비용
- H : 현재 노드에서 목적지 B까지의 예상 이동 비용. 이동 불가능한 노드는 무시하고 단순 거리로.
       (대각선 이동을 생각하는지 아닌지에 따라 다름. 여기선 대각선 x)
- F : 현재까지 이동하는데 걸린 비용과 예상 비용을 합친 총 비용(F)

* G 비용  
예를들어 수직/수평 이동에 대해선 비용이 10, 대각선 이동은 14의 비용을 할당.  
현재 노드의 G 비용 계산은 부모노드의 G 비용 + 부모노드로부터 현재 노드의 비용.  

* H(휴리스틱스) 비용
다양한 방식으로 추정 가능하며 예로 맨하탄(Manhattan) 방법이 있음.   
맨하탄 방법 - 현재 노드에서 대상 노드에 도달하기 위해 대각선 운동과 장애물은 무시하고 수평/수직 이동 비용만 계산.

* F 비용
최종적으로 G 비용과 H 비용을 더하여 어떤 경로가 가장 비용이 싼지 채점.

4. 계속 탐색하기
'열린 목록'에서 가장 작은 F비용을 가지고 있는 노드를 선택하고 아래와 같이 진행  
  a. 선택한 노드를 '열린 목록'에서 빼고 '닫힌 목록'에 넣어줌.  
  b. 인접한 노드를 확인. '닫힌 목록'에 있거나 벽은 무시하고 '열린 목록'에 없는 노드가 있다면 추가.  
  c. 인접한 노드가 이미 '열린 목록'에 있다면 해당 노드의 비용이 더 좋은지 확인.  
     즉, 선택한 노드와 비교해 G 점수가 어떤 것이 더 낮은지 확인하여 해당 노드가 더 낮다면  
     인접 노드의 부모를 새로운 노드로 바꿈. 마지막으로 다시 그 노드의 F와 G를 다시 계산.  


##### 전체 과정 요약
1. 시작 노드에서 검색된 인접 노드들을 열린 목록에 넣음.
2. 다음의 과정 반복 
  a. 열린 목록에서 가장 낮은 F 비용을 찾아 현재 노드로 선택.  
  b. 이것을 열린목록에서 꺼내 닫힌목록으로 넣음.  
  c. 선택한 노드에 인접한 8개의 노드에 대해 탐색  
    - 만약 인접한 노드가 갈 수 없는 벽이거나 닫힌목록에 있으면 무시, 그렇지 않은것은 다음을 계속
    - 만약 열린목록에 있지 않다면 열린 목록에 추가하고 이 노드의 부모를 현재 노드로 만듦.
      사각형의 F, G, H 비용을 기록
    - 만약 이미 열린목록에 있다면 G 비용을 이용해 이 노드가 더 나은지 알아보고 그것의 G비용이 더
      작다면 부모 노드를 그 노드(G 비용이 더 작은)로 바꿈, 그리고 그 노드의 G, F 비용을 다시 계산  
  d. 그만해야 할 때  
    - 길을 찾는 중 목표 노드를 열린 목록에 추가했을 때.
    - 열린 목록이 비어있게 될 때. (이 경우는 길이 없는 경우임)
3. 길 저장하기
  - 목표 노드로부터 각 노드의 부모노드를 향해 시작 노드에 도착할 때 까지 거슬러 올라감.


### Coverage Path Planning (IB 동작 이용)
*참고 논문
> <http://kiise.or.kr/e_journal/2013/10/sa/pdf/04.pdf>

*IB(Intellectual Boustrophedon) 동작 알고리즘 순서  

>if (남쪽 and 서쪽이 안막혀있다면)  
	서쪽 이동;  
else if (북쪽 and 서쪽이 안막혀있다면)  
	서쪽 이동;  
else if (북쪽 and 남쪽이 안막혀있다면)  
	남쪽 이동;  
else if (남쪽 방향이 S-동작 청소 영역이면)  
	남쪽 이동;  
else if (북쪽이 안막혀있으면)  
	북쪽 이동;  
else if (남쪽이 안막혀있으면)  
	남쪽 이동;  
else if (동쪽이 안막혀있으면)  
	동쪽 이동;  
else if (서쪽이 안막혀있으면)  
	서쪽 이동;  

