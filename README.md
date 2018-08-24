# **[Abstract]**
 이 논문의 저자는 low-dimensional regression 을 수행함에 있어서 training sample들을 나타내는 function을 구해낼 때(근사시킬때) 효과적인 empirical risk minimization framework를 제안한다. empirical 실증적인이란 의미는 training sample들을 학습하는 과정에서 직접 function을 근사시키기 때문에 사용하였다고 생각하였다.

> 시스템의 특성을 이해하기 위해서는 수학적인 모델이 필요하다. 그리고 이 시스템 모델에 바탕을 두고 제어기를 설계한다. 예를 들면 전압- 전류식이 있다면 이 수식내에는 저항 등이 포함될 것이다. 이 저항값을 알아야 원하는 전압이나 전류를 제어할 수 있을 것이다.
수학적인 모델이란 시스템의 입력과 출력 사이의 관계를 나타내는 방정식이다. 또한 수학적 모델은 수식내의 파라메터 성격에 따라 parametric model과 non-parametric model로 나누어 생각할 수 있다. 

> parametric model은 수식내에 실제 파라메터가 포함되어 있는 경우이며, 예를 들면 전압, 전류 사이의 관계식에서의 저항값 등 실제 존재하는 값을 의미하는 파라메터가 포함되어 있 는 수식같은 것이 이에 해당된다.

> Non-parametric model은 임펄스 응답이나 주파수 스펙트럼 같은 의미없는 파라메터가 포함되어 있는 수식 모델을 말한다.

 대부분의 non-parametric regression technique들은 하드웨어에서의 효율적인 실행을 고려한 모델들을 만들어내지 않는다. 대표적으로 support vector regression, Gaussian process regression들이 있으며, 이는 regression 과정에서 kernel computation들이 요구된다. 그리고 K-NN 같은 기법에서도 실제 실행순간 nearest neighbors를 찾는 연산을 요구 한다.

# **[Introduction]**

domain이 알려져 있거나, bounded 되어있는 경우에서의 함수를 근사화 시킬 때, 효과적인 regression을 위한 접근법은 domain 내의 value들의 regular lattice를 저장하는 방법이다. 규칙적인 점의 배열을 격자(lattice)라고도 한다. lattice vertice들로부터 test sample을 interpolate (보간)을 수행한다. 

이 논문의 이해를 위해 Interpolation에 대해서 얘기를 적어보겠다. 

> http://darkpgmr.tistory.com/117 에서 Interpolation에 대한 기본적인 개념을 공부할 수 있으며 일부를 발췌해왔다. 

> Interpolation(인터폴레이션, 보간)이란 알려진 지점의 값 사이(중간)에 위치한 값을 알려진 값으로부터 추정하는 것을 말한다.예를 들어, 어떤 사람이 20살일때 키와 40살에서의 키를 보고 30살에서의 키를 추측하는 것은 interpolation이라고 한다. 

> 아래 그림에서 x1, x2에서 데이터 값을 알고 있을 때 x1<=xi<=x2에서 값을 추정하는 것은 interpolation이고 범위 밖인 xj에서 값을 추정하는 것은 extrapolation이라 한다.

![](https://user-images.githubusercontent.com/40893452/42389625-430bff62-8184-11e8-90c3-1c7b50a06d4e.png)

> 두 지점을 보간하는 방법은 polynomial 보간, spline 보간 등 여러 가지가 있으나 그 중 선형 보간법(linear interpolation)은 두 지점 사이의 값을 추정할 때 그 값을 두 지점과의 직선 거리에 따라 선형적으로 결정하는 방법이다.

> 두 지점 x1, x2에서의 데이터 값이 각각 f(x1), f(x2)일 때, x1, x2 사이의 임의의 지점 x (x1≤x≤x2)에서의 데이터 값 f(x)는 선형보간법을 사용할 경우 다음과 같이 계산된다.

![](https://t1.daumcdn.net/cfile/tistory/2320753852D3A66F03)

> 단, d1은 x에서 x1까지의 거리, d2는 x에서 x2까지의 거리.

![](https://t1.daumcdn.net/cfile/tistory/2279BF3852D3A61413)

>  원래는 f(x) = f(x1) + d1/(d1+d2)*(f(x2)-f(x1)) 인데, 이를 정리하면 식 (1)이 나온다. 만일 거리의 비를 합이 1이 되도록 정규화하면 즉, α = d1/(d1+d2), β = d2/(d1+d2)라 잡으면 식 (1)은 다음과 같이 좀더 단순화될 수 있다 (α + β = 1).

![](https://t1.daumcdn.net/cfile/tistory/222A9C3F52D3B4150D)

> 수업시간에 배운 affine form 의 수식이 된다!

> Bilinear Interpolation의 경우는 우리 말로 적자면 쌍선형 보간법, 또는 이중선형 보간법 정도가 되며 1차원에서의 선형 보간법을 2차원으로 확장한 것이다.

> Bilinear interpolation 방법을 설명하기 위해 아래 그림과 같이 직사각형의 네 꼭지점에서의 값이 주어져 있을 때, 이 사각형의 변 및 내부의 임의의 점에서의 값을 추정하는 문제를 생각해 보자.

![](https://t1.daumcdn.net/cfile/tistory/2220224D52D3B98204)

> 그림과 같이 점 P에서 x축 방향으로 사각형의 변까지의 거리를 w1, w2, y축 방향으로 거리를 h1, h2라 하고, 알려진 네 점에서의 데이터 값을 A, B, C, D라 할 때, P에서의 데이터 값은 bilinear interpolation에 의해 다음과 같이 계산된다 (단, α=h1/(h1+h2), β=h2/(h1+h2), p=w1/(w1+w2), q=w2/(w1+w2)).

![](https://t1.daumcdn.net/cfile/tistory/2207424652D3B5AF07)

> 계산 원리는 쉽게 짐작할 수 있듯이 A, B를 보간하여 M을 얻고 C, D를 보간하여 N을 얻은 후에 M, N을 보간하여 P를 얻는 방식이다. 또는 그 순서를 바꾸어 U와 V를 먼저 얻은 후에 U, V를 보간해도 동일한 결과를 얻을 수 있다.

> 그런데, 위의 보간 방법은 원래의 데이터의 위치가 직사각형을 이룰 경우에만 적용할 수 있는 방법이다. 만일 아래 그림처럼 임의의 형태의 사각형에서 사각형 내부를 보간하고자 할 때에는 어떻게 해야 할까?

![](https://t1.daumcdn.net/cfile/tistory/212DDF4852D3BB7D26)

> 이 경우에는 아래 그림처럼 원래의 사각형을 어떤 직사각형으로 워핑(warping)시킨 후 워핑된 사각형에서 보간(interpolation)을 수행하면 된다.워핑할 사각형은 임의의 직사각형이 가능하지만 편의상 네 꼭지점의 좌표가 (0,0), (0,1), (1,1), (1,0)인 단위 정사각형으로 워핑시킨다고 하자.

![](https://t1.daumcdn.net/cfile/tistory/2222AF3552D3BFDB2A)

> 이 경우 보간 방법은 원래 사각형의 네점 ABCD를 A'B'C'D'으로 변환시키는 선형변환(linear transformation) T를 구한 후, 구한 T를 이용하여 P를 변환시킨 P'를 구하고 단위 정사각형에서 bilinear interpolation을 수행하면 된다. 선형변환 T는 2D homography 를 통해서 구할 수 있다. 

> Trilinear Interpolation은 삼선형 보간법 정도로 번역할 수 있겠으며 1차원에서의 선형보간법을 3차원으로 확장한 것이다. Trilinear interpolation 방법은 3차원 공간에서 8개의 꼭지점으로 이루어진 육면체의 변 및 내부의 임의의 점에서의 데이터 값을 선형적으로 보간하는 방법을 일컫는 말이다.

![](https://t1.daumcdn.net/cfile/tistory/260ACF3952D3C6AC2C)

> Trilinear interpolation 방법은 위 그림과 같이 bilinear interpolation과 원리는 동일하며 P에서의 보간값을 구하기 위해 먼저 M, N에서의 값을 보간하고 이로부터 R에서의 값을 보간한다. 마찬가지로 U, V에서의 값을 보간한 후 이로부터 S에서의 값을 보간한다. 마지막으로 R, S로부터 P를 보간한다. 만일 원래의 데이터 점들이 직육면체를 이루지 않을 경우에는 bilinear interpolation에서 설명한 방법과 마찬가지로 원래의 육면체를 임의의 직육면체로 매핑(워핑)한 후 워핑된 직육면체에서 보간값을 계산하는 방법을 사용한다.

Lattice를 활용한 test sample의 evaluation은 training set의 size에 영향을 받지 않으며 input space의 차원에 따라 exponential하게 연산 요구랑이 증가 하게 된다. 

Digital color management 분야에서 interpolated look-up table (LUT) 접근법은 real-time performance 상황에서 매 초마다 수백만의 evaluation을 요구하는 상황에서 가장 인기있는 사용법이다.

> http://www.cemtool.co.kr/education/5_AdvancedEngineeringMathematics/scr/chapter/7.htm 에서 interpolation error의 개념을 살펴보았습니다.
> https://hermit1004computer.blogspot.com/2015/01/blog-post.html

> https://www.youtube.com/watch?v=Rm9xqpQs5eg

training data를 통해 lattice를 학습해야하는 분야에서, 기존의 접근법은 문제를 가진다. training data로부터 function을 추정하며 추정된 function을 lattice point를 가지고 평가한다. 그러나 function을 평가할 때 lattice node들로부터의 보간의 효과를 고려하지 않아 suboptimal을 제공하게된다. 이는 다시말해 lattice output들을 통해서 training data를 보간을 통해서 정확히 다시 만들어 낼 수 있어? 라는 문제이다.

> 고차 다항식보간의 위험성이며 다항식 차수가 높아지면 진동 발생 위험이 있고 고차다항식의 경우 반올림 오차에 민감하여 불량한 조건이 되기 쉽기 때문이다

그래서 Iterative post-processing solution을 통해 post-interpolation error를 줄이기 위한 방법으로 lattice를 지속적으로 update하는 연구들이 이루어 졌다.

> 데이터 입력단계에서 입력된 소정수치에 대해 해석수단을 이용하여 해석을 실행하고, 해석을 통해 출력되는 데이터를 계산하여 시뮬레이션을 하는 시뮬레이션단계, 시뮬레이션 수행 도중 오류가 발생하여 시뮬레이션 수행이 실패하면, 보간법을 이용하여 반복적으로 오류를 수정

이 논문에서는 "lattice regression"이라고 부르는 새로운 해결법을 제시한다. 이 방법은 모든 lattice output들을 training data에서 정규화된 보간 에러 (regularized interpolation error) 를 최소화하는 것을 협력적으로 평가하는 방법이다. 

# **[Lattice Regression]**

이 논문에서 제안하는 lattice regression 방법은 training data를 정확하게 보간할 수 있는 lattice node를 위한 lattice output들을 협력적으로 선택하는 방법이다. 선형 보간법이 squared error of training data를 최소화하는 lattice node output 을 위한 해결 방법으로 변환될 수 있다는 것이 이 추정 방법의 핵심이다. 

그러나 추정 방법의 문제는 training data가 충분하지 않다면 unique한 solution이 나오지 않는다는 것이다. 그러므로 이 추정에서의 다양한 variance를 줄이기 위해서 training data에 정확하게 fitting 시키는 것을 피하는 것이 좋은 방법이다. 

이 논문에서는 이를 해결하기 위해서 2개의 정규화(regularization) 방법을 통해 보간 에러의 감소를 이루려고 한다.

> 앞에서 언급한 LUT (Look Up Table) 이란 무엇일까?

> http://api.unrealengine.com/KOR/Engine/Rendering/PostProcessEffects/UsingLUTs/

> http://montoo.tistory.com/52 에서 기본적인 개념을 소개하고 있다.





## Empirical Risk
 
data는 

![](https://user-images.githubusercontent.com/40893452/42406927-dbb16a8c-81ec-11e8-80e8-8a965736efa2.png) 

의 d 차원 공간에서 가져온다고 생각한다. 즉, 데이터 x(i) 는 d 개의 feature로 구성되어있다고 고려.

그리고 lattice의 output이 되는 결과는 

![](https://user-images.githubusercontent.com/40893452/42406934-e9356fc8-81ec-11e8-9733-42bb6d52653f.png)

의 p 차원 공간으로 매핑된다고 한다.

![](https://user-images.githubusercontent.com/40893452/42406937-f1bcf576-81ec-11e8-94a0-9fe57bbf7458.png)

데이터는 이와같이 d x n 의 행렬 X 로 구성이 된다.

![](https://user-images.githubusercontent.com/40893452/42406938-fc8224ea-81ec-11e8-9f81-b1b76fd4a291.png)

output은 p x n 의 행렬 Y 로 구성이 된다.

이때 lattice 는 m 개의 노드들로 구성이 되어있다고 가정 한다.

![](https://user-images.githubusercontent.com/40893452/42406944-17bc42a4-81ed-11e8-9885-1151fbab6710.png)

> LUT 에 대한 참조 https://help.videovillage.co/article/7-what-is-a-lut

