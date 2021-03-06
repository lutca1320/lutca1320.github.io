---
title: A* 알고리즘 - 길찾기 알고리즘
author: RUKA SPROUT
date: 2020-10-28 22:30:00 +0900
categories: [Algorithm]
tags: [programming, pathfinding]
---

![demons](https://i.imgur.com/7FG19y6.gif)


---


## 개요
`A*` 알고리즘은 길찾기 (Pathfinding) 에 자주 사용되는 알고리즘이다. `A*` 알고리즘은 다익스트라 (Dijkstra) 알고리즘의 업드레이드 버젼이라고 생각하면 쉽다. `A*` 는 다익스트라 알고리즘처럼 최단 거리를 보장하지는 없지만, 검색 속도가 빠르다.

위의 예시 경로를 탐색하는 방법을 하나씩 확인함으로써 알고리즘을 설명해 보겠다. 코드로 구현하는 것은 다음 포스트에 담도록 하겠다.

포스트를 쓰면서 첨부한 모든 사진의 프로그램은 직접 `Python Tkinter` 를 사용해 만들었다.

- [Visualizer](https://github.com/lutca1320/Astar-Visualizer-Python/blob/master/main.py){: target="_blank"} : GUI 비주얼라이져
- [Core Code](https://github.com/lutca1320/Astar-Visualizer-Python/blob/master/core_code.py){: target="_blank"} : A* 알고리즘 코드

---


## 개념 설명

그 전에, 아래 그림을 이해할 필요가 있다.

아래 그림에서 보듯이 출발 노드와 도착 노드가 이미 정해져 있다. `A*` 알고리즘은 항상 도착 노드가 존재해야 한다.

![a 00](https://i.imgur.com/EklAnDO.png)

#### **1. FGH 값**

각 노드들에 숫자 3개가 적혀져 있는데, 이는 다음과 같은 의미를 가진다.

![fgh](https://i.imgur.com/qpTyg6h.png)

- **G 값은 출발 노드부터 현재 노드까지의 경로의 길이**다. 그렇기 떄문에 출발 노드의 G 값은 0이 된다.
- **H (휴리스틱 함수) 값은 현재 노드에서 도착 노드까지의 '예상 경로' 길이**다. '예상 경로' 이기 떄문에 이는 추정으로만 구할 수 있는데, **장애물은 고려하지 않고 계산**한다.


H 값을 구하는 방법은 다음과 같이 생각해볼 수 있다.
- 유클리드 거리로 계산 : 실제 두 노드의 좌표의 거리를 계산한다.
- 인덱스 거리로 계산 : 두 노드의 인덱스(i, j) 차이를 이용해 거리를 계산한다.

![index](https://i.imgur.com/KncXuvD.png)

위 그림은 인덱스 거리로 계산하는 방법을 보인 것인데,

- 한 칸을 이동 : 10
- 대각선으로 이동 : 14

으로 정의한다면, `58 = 10 * 3 + 14 * 2` 와 같이 H 값을 구할 수 있다. 위 그림은 경로 중 하나일 뿐으로, 여러 경로가 있을 수 있으나, 결국 직선 3개 대각선 2개는 바뀌지 않으므로 상관없다.

이를 코드로 구현하는 법은 나중에 따로 다루도록 하겠다.

#### **2. OPEN, CLOSED 리스트**

위 그림에서 보면, 출발 노드의 색깔이 초록색인 것을 확인할 수 있다. 이는 OPEN 리스트에 추가되어 있는 것을 의미하는데, 이는 다음과 같은 의미를 가지고 있다.

- **OPEN 은 '탐색 되어져야 하는 노드들의 리스트'** (초록색)
- **CLOSED 는 '이미 탐색된 노드들의 리스트'** (빨간색)

![a 56](https://i.imgur.com/iN85RhB.png)

`A*` 알고리즘은 OPEN 리스트에서 노드를 꺼내 적절한 노드를 CLOSED 리스트에 넣는 절차로 작동한다. 이는 아래에서 예시로 설명하겠다.


---


## 어떻게 작동하는가?

자, 그럼 이제 실제 어떤 절차로 경로를 구하는지 하나씩 살펴보자.

#### **1. 출발 노드를 OPEN 리스트에 넣는다.**

출발 노드부터 탐색할 것이기 때문에 출발 노드를 OPEN 리스트에 넣고, FGH 값을 업데이트한다. 다른 노드들은 모두 0으로 초기화되어 있다. 검은색으로 칠해진 노드는 장애물 노드이다.

*현재 노드 (current node) 는 출발 노드로 설정된다.*

![9HMPNPf](https://i.imgur.com/jUKpC3Q.png)

#### **2. 주변 노드들을 OPEN 리스트에 넣는다.**

출발 노드 주변에 있는 **인접한 8개 노드를 OPEN 리스트에 넣고, FGH 값을 업데이트**한다. 이때 출발 노드는 '탐색 되었다' 고 할 수 있으므로 CLOSED 노드에 넣는다.

![ne](https://i.imgur.com/xUqeYSf.png)

그러나 현재 출발 노드의 주변에는 노드가 5개밖에 없기 때문에 5개만 업데이트 해준다.

![a 10](https://i.imgur.com/HZxl27y.png)

#### **3. 현재 노드 선택**

그다음 OPEN 리스트에 있는 노드들 중 어느 노드를 선택해 현재 노드로 지정해야 할 지 정해야 한다.

![a 1022](https://i.imgur.com/6E4tm2D.png)

- 먼저 F 값이 작은 노드를 선택한다.
- 만약 위와 같이 같은 F 값을 가진다면, H 값 (도착지까지의 예상 경로) 이 작은 노드를 선택한다.

위 상황에서는 오른쪽 위 노드가 H 값이 더 작기 때문에 이를 선택해준다.

#### **4. 주변 노드 업데이트**

현재 노드의 주변 8개 노드를 OPEN 리스트에 넣고 값을 업데이트한다. 또한 현재 노드를 CLOSED 리스트에 넣는다.

- CLOSED 리스트에 이미 등록되었거나 장애물 노드인 경우 리스트에 넣지 않는다.
- 주변 노드들 중 이미 OPEN 리스트에 등록되어 있는 노드인 경우 상황에 따라 값을 업데이트한다. (현재는 업데이트 X)

그렇기 때문에 아래와 같이 5개의 노드만 OPEN 리스트에 넣고 값을 업데이트 해준다.

![a 232](https://i.imgur.com/XKlXAjU.png)

#### **5. 현재 노드 선택**

여기에서도 초록색으로 칠해진 OPEN 리스트에 있는 노드들 중에서 F값이 가장 작은 노드를 선택하면 된다. 오른쪽 노드와 아래 노드가 있는데, 오른쪽 노드의 H 값이 더 작기 때문에 오른쪽 노드를 선택해준다.

![a 2322](https://i.imgur.com/GNGjk6W.png)

#### **6. 주변 노드 업데이트, 현재 노드 선택**

주변 노드들을 OPEN 리스트에 넣고, 값을 업데이트한다. 여기에서는 왼쪽 아래 노드의 F값이 가장 작으므로 이를 현재 노드로 지정해준다.

![a 362](https://i.imgur.com/0UCJBdd.png)

#### **7. 주변 노드 업데이트 (OPEN 리스트에 있는 노드 값 업데이트)**

자 이제 아래 그림을 살펴보자. 자세히 보면 현재 노드 오른쪽 노드의 값이 업데이트되어 있는 것을 확인할 수 있다.

위에서 상황에 따라 OPEN 리스트에 있어도 값이 업데이트되는 경우가 존재할 수 있다고 했는데, 지금이 딱 그 상황이다.

![a 42](https://i.imgur.com/Iidr99s.png)

다음과 같은 상황에서는 OPEN 리스트에 있어도 값이 업데이트된다.

- **원래 경로로 계산한 F 값 > 새로운 경로로 계산한 F 값**

이게 도대체 무슨 말일까? 아래 그림을 보면서 생각해 보자.

![r1](https://i.imgur.com/8tsjWQu.png)

위 그림은 아까 현재 노드를 업데이트 하기 전의 모습이다. G 값 (현재 경로의 길이) 아 28이 된 이유는 대각선 방향으로 2번 이동했기 때문에 `14 * 2 = 28` 이다.

![r2](https://i.imgur.com/Fz0i1oN.png)

위 그림은 현재 노드를 업데이트한 상황인데, 위 그림과 같은 경로가 원래 경로보다 더 짧은 경로임을 알 수 있다. 직선으로 2번 이동했기 때문에 `10 * 2 = 20` 이다.

결국 새로운 경로의 F 값이 더 작으므로, 새로운 경로로 구한 F 값으로 업데이트된다.

#### **8. 반복**

이러한 작업을 현재 노드가 도착 노드가 될 때 까지 반복해주면 된다.

![a 52](https://i.imgur.com/IRK4Yib.png)

![a 56](https://i.imgur.com/uRQDGGn.png)

![a 66](https://i.imgur.com/9qH1TM7.png)

![a 73](https://i.imgur.com/iuNZ8nl.png)

#### **9. 경로 역추적**

그런데 과연 어떻게 위 그림처럼 경로를 찾을 수 있는 것일까? 위에서 설명하지 않은 것이 있다.

현재 노드의 주변 노드들을 OPEN 리스트에 넣을 때, 각 주변 노드들은 현재 노드를 `parent` 로 설정한다.

![ne2](https://i.imgur.com/XecC5iD.png)

아까 본 그림을 다시 활용하면, 위 그림에서 보는 것과 같이 초록색으로 칠해진 주변 노드들의 `parent` 값을 현재 노드로 설정해준다.

이렇게 되면 경로를 역추적할 수 있다. 아래 그림을 보자.

![a 7322](https://i.imgur.com/hfZXLLp.png)

도착 노드까지 도달하게 되면, 도착 노드의 `parent` 노드를 가져온다. 그러면 왼쪽 아래의 노드가 될 것이고, 이 노드의 `parent` 노드를 가져오면 또 왼쪽 아래의 노드가 될 것이다.

이렇게 하다보면 결국 출발 노드에까지 도달할 수 있고, 이들을 PATH 리스트에 각각 넣어준다면 경로를 완성할 수 있다.


---


### Pseudo code (유사 코드)

![A_](https://i.imgur.com/xVyCH6m.png)

---

### 다른 예시

![ex](https://i.imgur.com/LXXsJwb.png)
