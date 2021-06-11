# SimulatedAnnealing Algorithm

# 컴퓨터 알고리즘 기말고사 대체 과제

## 제출자 : 오성환

## 1. 모의담금질(SimulatedAnnealing)이란?

**모의담금질**(SimulatedAnnealing)이란 높은 온도에서 액체 상태인 물질이 온도가 점차 낮아지면서 결정체로 변하는 과정을 모방한 탐색 알고리즘이다. 온도가 점점 낮아지면 분자의 움직임이 점점 줄어들게 되어 결정체가 되는데, 해 탐색 과정이 이와 유사하게 점점 더 규칙적인 방식으로 이루어진다.

일단 기본적으로, 후보해에 대해 이웃하는 해(이웃해)를 정의해야 한다. 최솟값을 탐색한다고 할 때, 아래 방향으로 점차 탐색해 나가게 되는데, 지역 최적해(local optimum)에 도달하면 더 이상 아래로 탐색할 수 없는 상태에 이르른다. 하지만, 이 과정에서 '운 좋게' 위 방향으로(나쁜 해로) 탐색이 이루어져 전역 최적해(global optimum)에 도달하기도 한다.

그러나 유전자 알고리즘과 마찬가지로 모의 담금질 기법 또한 항상 전역 최적해를 찾아준다는 보장은 없다. 더 이상 우수한 해를 찾지 못하거나, 미리 정한 루프의 최대 반복 횟수가 초과되면 종료되게 된다.

## 2. 설계 과정, 수도 코드

SimulatedAnnealing의 수도 코드는 다음과 같다.

1.  임의의 후보해 s를 선택한다.
2.  초기 T를 정한다.
3.  repeat
4.      for i = 1 to kt {
5.      s의 이웃해 중에서 랜덤하게 하나의 해 s'를 선택한다.
6.      d = (s'의 값) = (s의 값)
7.      if (d < 0)
8.          s = s'
9.      else
10.         q = (0,1 사이의 랜덤한 수)
11.         if (q < p)  s = s'
12.     }
13.     T = aT
14. until (종료 조건이 만족될 때까지)
15. return s
## 3. 2차 함수 모의담금질 코드

```java
package com.company;

public class Main {
    public static void main(String[] args) {
        SimulatedAnnealing sa = new SimulatedAnnealing(10);
        Problem p = new Problem() {
            @Override
            public double fit(double x) {
                return -x*x + 38*x + 80;
                // x=19 , f(x)=441
            }

            @Override
            public boolean isNeighborBetter(double f0, double f1) {
                return f0 < f1;
            }
        };
        double x = sa.solve(p, 100, 0.99, 0, 0, 31);
        System.out.println(x);
        System.out.println(p.fit(x));
        System.out.println(sa.hist);
    }
}
```

```java
package com.company;

public interface Problem {
    double fit(double x);
    boolean isNeighborBetter(double f0, double f1);
}
```

```java
package com.company;

import java.util.ArrayList;
import java.util.Random;

public class SimulatedAnnealing {
    private int niter;
    public ArrayList<Double> hist;

    public SimulatedAnnealing(int niter) {
        this.niter = niter;
        hist = new ArrayList<>();
    }

    public double solve(Problem p, double t, double a, double lower, double upper) {
        Random r = new Random();
        double x0 = r.nextDouble() * (upper - lower) + lower;
        return solve(p, t, a, x0, lower, upper);
    }

    public double solve(Problem p, double t, double a, double x0, double lower, double upper) {
        Random r = new Random();
        double f0 = p.fit(x0);
        hist.add(f0);

        for (int i=0; i<niter; i++) {
            int kt = (int) t;       // kt : 반복횟수
            for(int j=0; j<kt; j++) {
                double x1 = r.nextDouble() * (upper - lower) + lower;       //r.nextDouble() : 0~1 중 double 범위 랜덤
                double f1 = p.fit(x1);

                if(p.isNeighborBetter(f0, f1)) {
                    x0 = x1;
                    f0 = f1;
                    hist.add(f0);
                } else {
                    double d = Math.sqrt(Math.abs(f1 - f0));
                    double p0 = Math.exp(-d/t);
                    if(r.nextDouble() < 0.0001) {
                        x0 = x1;
                        f0 = f1;
                        hist.add(f0);
                    }
                }
            }
            t *= a;
        }
        return x0;
    }
}
```

수업 시간에 작성한 2차 함수의 전역 최적점을 찾는 모의담금질 알고리즘 코드이다.

2차함수를 위로 볼록한 모양, -x^2+38x+80으로 지정한 다음 초기온도 T를 100, 후보해 x0를 0, 감소 비율 a를 0.99, x의 최댓값과 최솟값 upper와 lower를 각각 31, 0으로 설정하였다.

담금질 과정은 이웃해 x1을 설정해 이 값으로 얻어지는 f(x1)값을 f1으로 설정하고, f1이 f0보다 더 큰 경우 x1과 f1을 x0와 f0에 각각 대입하고 f0값을 hist 리스트에 추가하는 형식이다. 이를 통해 최종적으로 전역 최적점으로 선정된 값과 값의 변화 과정을 알 수 있다.

![2](https://user-images.githubusercontent.com/80510972/121697258-d72a1100-cb07-11eb-9d15-556dcf8a7dca.png)

## 4. 4차 함수의 전역 최적점

4차 함수의 그래프를 먼저 입력해 보았다.

4차 함수의 그래프는 x^4의 계수가 양수라고 할때,

- 극댓값 1개와 극솟값 2개가 존재 : 극솟값이 서로 다른 경우

- 극댓값 1개와 극솟값 2개가 존재 : 극솟값이 서로 같은 경우

- 극솟값이 하나만 존재 : f'(x)=0 이 서로 다른 두 근을 갖고 그 중 하나가 중근인 경우

- 극솟값이 하나만 존재 : f'(x)=0 이 허근 두 개와 실근 하나를 갖는 경우
- 극솟값이 하나만 존재 : f'(x)=0 이 삼중근을 갖는 경우

의 5가지로 나눌 수 있다. 전역 최적점을 다룰 수 있는 그래프는 많지만 내가 사용한 그래프는 x^4의 계수가 음수이고, 서로 다른 극댓값을 2개 가지는 그래프를 사용했다.

![3](https://user-images.githubusercontent.com/80510972/121697283-de511f00-cb07-11eb-9a45-b1835a55b41c.png)

```java
package com.company;

public class Main {
    public static void main(String[] args) {
        SimulatedAnnealing sa = new SimulatedAnnealing(10);
        Problem p = new Problem() {
            @Override
            public double fit(double x) {
                return (-x-5)*(x-1)*(-x+5)*(-x+1);
                // x=-3.294 , f(x)=260.896
            }

            @Override
            public boolean isNeighborBetter(double f0, double f1) {
                return f0 < f1;
            }
        };
        double x = sa.solve(p, 100, 0.99, 0, -5, 5);
        System.out.println(x);
        System.out.println(p.fit(x));
        System.out.println(sa.hist);
    }
}

```

나머지 코드는 똑같이 하되, 4차 함수를 입력하고 (넓은 범위에서 탐색하는 적당한 4차함수를 찾기가 힘들었음), 최댓값인 x=-3.294에의 f(x)=260.896임을 표시해 두었다. upper와 lower를 -5와 5로 변경하였다. 결과값은 다음과 같다.

![4](https://user-images.githubusercontent.com/80510972/121697287-dee9b580-cb07-11eb-9125-90c7e232bb94.png)

위와 같이 정상적으로 전역 최적점을 찾는 것을 볼 수 있다.

## 5. 3차 함수의 전역 최적점

3차 함수 최댓값이 딱 떨어지는 식을 찾아서 3차 함수 또한 입력해 보았다.

3차 함수 그래프가 일정 범위 내에서 양끝점 이외에 최댓값이나 최솟값을 가지는 경우는 f'(x)=0 이 서로 다른 두 실근을 가지는 경우(사인 곡선과 비슷한 모양인 경우) 밖에 없기 때문에 이러한 그래프를 사용했다.

![5](https://user-images.githubusercontent.com/80510972/121697259-d7c2a780-cb07-11eb-93dc-3c1bbab8fda1.png)

```java
package com.company;

public class Main {
    public static void main(String[] args) {
        SimulatedAnnealing sa = new SimulatedAnnealing(10);
        Problem p = new Problem() {
            @Override
            public double fit(double x) {
                return -x*x*x+3*x*x+144*x-432;
                // x=8 , f(x)=400
            }

            @Override
            public boolean isNeighborBetter(double f0, double f1) {
                return f0 < f1;
            }
        };
        double x = sa.solve(p, 100, 0.99, 0, 0, 12);
        System.out.println(x);
        System.out.println(p.fit(x));
        System.out.println(sa.hist);
    }
}
```

이번에도 마찬가지로 나머지 코드는 일치하되, 함수 식과 lower, upper 값을 변경해 주었다. 결과값을 보자.

![6](https://user-images.githubusercontent.com/80510972/121697292-e0b37900-cb07-11eb-852b-70aeb1663e6b.png)

위와 같이 정상적으로 전역 최적점을 찾는 것을 볼 수 있다.

## 6. Curve Fitting

위에서 진행한 4차 함수에 대해서 후보해로 선정되는 순서에 따른 최적점까지의 Curve Fitting을 진행해보면

![7](https://user-images.githubusercontent.com/80510972/121697295-e14c0f80-cb07-11eb-965e-13a82304f86e.png)

다음과 같이 나타나며, 최댓값인 260.896에 가까워짐을 볼 수 있다.
## 7. 성능분석 및 결과
모의담금질 기법을 적용할 3~4차 함수를 찾아보면서 최적해를 찾기 위해서는 최댓값이 포함된 x값의 범위를 설정하는 것이 중요하게 작용했다. 때문에 복잡한 문제에 대해서 모의 담금질 기법을 사용하기에는 성능 면에서 적용이 어려울 것 같다는 생각을 했다.
하지만 모의담금질 기법을 통한 최적점을 찾는 과정에서 최적해에 굉장히 가까운 값을 찾을 수 있다는 점에서는 큰 장점을 가진 알고리즘이다.
