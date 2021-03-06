---
title: 도로 제작 - 2. Snapping 시스템
author: RUKA SPROUT
date: 2020-07-17 18:54:00 +0900
categories: [Game Dev - Intersection]
tags: [Unity]
---

도로를 그릴 때 그리드에 맞춰 Snapping 이 된다면 훨씬 깔끔하고 규격화되어 있는 도로를 만들 수 있을 것이다. 이를 구현하기 위해 아래 2가지 함수를 제작하였다.

---

## 1. `SnapGrid` 함수

Snapping 에는 여러가지 방법이 있지만, 먼저 Mathf.Round (반올림) 를 이용해 쉽게 만들 수 있다.

```csharp
float SnapGrid(float value, int snapsize)
{
    if (value < 0)
    {
        return Mathf.Round(Mathf.Abs(value / snapsize)) * snapsize * -1;
    }
    else
    {
        return Mathf.Round(value / snapsize) * snapsize;
    }
}
```

`value` 값을 `snapsize` 만큼의 그리드로 반올림한다. 음수인 경우는 절대값을 반올림한 뒤 -1을 곱해 따로 리턴 값을 준비해 주었다.

먼저 `value / snapsize` 를 통해 값을 `snapsize` 에 맞춘 소수로 변경한 후, 반올림한 값에 `snapsize` 를 곱함으로써 적절한 값을 얻을 수 있다.

- `value = 12, snapsize = 10` 이면 10 으로 내림되어 리턴
- `value = 18, snapsize = 10` 이면 20 으로 올림되어 리턴

이 함수를 사용하다 보니 문제가 하나 있었다. 위 함수를 이용하면 x 값 따로 z 값 따로 리턴을 받기 때문에 `x = z = snapsize / 2` 일때를 제외하고 대각선으로 한번에 이동할 수 없다. `x = z = snapsize / 2` 가 성립하려면 매우 어렵기 때문에 결국 **대각선 방향으로 한번에 움직이지 못한다.**

![1](https://user-images.githubusercontent.com/25034289/89159832-09471a00-d5ab-11ea-9d0b-c701e48d9dad.gif)

그리드를 따라 이동할 때마다 도로가 Append 되기 때문에 대각선 방향으로 한번에 이동하지 않는다면 대각선 방향의 도로는 그릴 수 없다. 반드시 고쳐저야 하는 부분이다.

---

## 2. `SnapToGridPoint` 함수

이 문제를 해결하기 위해 새로운 함수가 필요해 보였다.

문제를 해결하기 위해 주변에 Area 를 지정해 **각 Area 에 들어가면 지정된 위치로 이동하는 방법** 을 사용하였다. 이 방법을 사용하면 대각선으로 이동하는 Area 를 지정해 놓고, 그 Area 에 들어오면 대각선으로 이동하게 할 수 있다.

![주석 2020-08-03 152713](https://i.imgur.com/WCBAUBA.png)

위 그림처럼 여러 Area 로 나누어 두었는데, 다음과 같이 정의하였다.
- 초록색 : 대각선 이동
- 주황색 : 가로 이동
- 파란색 : 세로 이동

회색은 하나만 그려 두었는데, 이는 다른 Grid 에 있는 Area 로 현재의 Grid 와 겹치는 부분이 존재하지 않게 만들었음을 보여준다.

아래는 이를 코드로 구현한 것이다.

```csharp
Vector3 SnapToGridPoint(Vector3 pos, int _snapsize)
{
    float snapsize = (float)_snapsize;

    if (!isVectorInXZArea(pos, -snapsize + last_pos.x, snapsize + last_pos.x,
        -snapsize + last_pos.z, snapsize + +last_pos.z))
    {
        UnityEngine.Debug.LogWarning("Out of range!");
    }
    else
    {
        if (isVectorInXZArea(pos, snapsize / 2 + last_pos.x, snapsize + last_pos.x,
        last_pos.z - snapsize / 4, last_pos.z + snapsize / 4))
        {
            last_pos = last_pos + new Vector3(snapsize, 0, 0);
            return last_pos;
        }
        else if (isVectorInXZArea(pos, last_pos.x - snapsize, last_pos.x - snapsize / 2,
            last_pos.z - snapsize / 4, last_pos.z + snapsize / 4))
        {
            last_pos = last_pos - new Vector3(snapsize, 0, 0);
            return last_pos;
        }
        else if (isVectorInXZArea(pos, last_pos.x - snapsize / 4, last_pos.x + snapsize / 4,
            last_pos.z + snapsize / 2, last_pos.z + snapsize))
        {
            last_pos = last_pos + new Vector3(0, 0, snapsize);
            return last_pos;
        }
        else if (isVectorInXZArea(pos, last_pos.x - snapsize / 4, last_pos.x + snapsize / 4,
            last_pos.z - snapsize, last_pos.z - snapsize / 2))
        {
            last_pos = last_pos - new Vector3(0, 0, snapsize);
            return last_pos;
        }
        else if (isVectorInXZArea(pos, last_pos.x + snapsize / 2, last_pos.x + snapsize,
            last_pos.z + snapsize / 2, last_pos.z + snapsize))
        {
            last_pos = last_pos + new Vector3(snapsize, 0, snapsize);
            return last_pos;
        }
        else if (isVectorInXZArea(pos, last_pos.x - snapsize, last_pos.x - snapsize / 2,
            last_pos.z + snapsize / 2, last_pos.z + snapsize))
        {
            last_pos = last_pos + new Vector3(-snapsize, 0, snapsize);
            return last_pos;
        }
        else if (isVectorInXZArea(pos, last_pos.x + snapsize / 2, last_pos.x + snapsize,
            last_pos.z - snapsize, last_pos.z - snapsize / 2))
        {
            last_pos = last_pos + new Vector3(snapsize, 0, -snapsize);
            return last_pos;
        }
        else if (isVectorInXZArea(pos, last_pos.x - snapsize, last_pos.x - snapsize / 2,
            last_pos.z - snapsize, last_pos.z - snapsize / 2))
        {
            last_pos = last_pos - new Vector3(snapsize, 0, snapsize);
            return last_pos;
        }
    }

    return last_pos;
}
```

다소 복잡해 보이지만, pos 값이 어느 Area 에 위치하는지를 체크하고 지정된 값 만큼 이동시킨 뒤 리턴하는 코드이다.
last_pos 이라는 전역 변수가 있는데, 이는 그리드에 Snapping 된 전 pos 값으로 위의 그림에서 중심에 해당하는 부분이다.

이 함수의 치명적인 문제점이 있는데, **마우스가 위의 그림 Grid 를 벗어나면 Snapping 이 불가능** 하다는 것이다. 만약 마우스를 매우 빠르게 움직이거나 줌을 통해 마우스의 위치가 이동될 때, 아래 그림처럼 마우스의 위치가 last_pos 을 기준으로 한 Grid 에서 벗어나게 된다면, 위 코드는 작동되지 않는다.

![주석 2020-08-05 234557](https://i.imgur.com/8YM9kV0.png)

위 함수를 고쳐서 다시 만들 수도 있지만... 더 쉬운 방법이 있다.

- 마우스 버튼 누르기 전 : `SnapGrid` 함수
- 마우스 버튼 누르고 있을 때 : `SnapToGridPoint` 함수

어차피 Build 모드에서는 대각선으로 그릴 일이 없기 때문에 이를 통해 문제를 보완하였다.
