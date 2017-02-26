---
layout: post
title: '[SLAM] Least Squares (최소자승법)'
tags: [SLAM]
description: >
  Graph SLAM의 접근방법인 least square 방법에 대해서 설명한다.
sitemap :
  changefreq : weekly
  priority : 1.0
---

**본 글은 University Freiburg의 [Robot Mapping](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 바탕으로 이해하기 쉽도록 정리하려는 목적으로 작성되었습니다. 개인적인 의견을 포함하여 작성되기 때문에 틀린 내용이 있을 수도 있습니다. 틀린 부분은 지적해주시면 확인 후 수정하겠습니다.**

SLAM은 로봇의 위치를 추정함과 동시에 주변 환경에 대한 맵도 같이 생성하는 기술이다. SLAM의 방법은 크게 3가지로 나눌 수 있다.

* [Kalman Filter](http://jinyongjeong.github.io/2017/02/14/lec03_kalman_filter_and_EKF/)
* [Particle Filter](http://jinyongjeong.github.io/2017/02/22/lec11_Particle_filter/)
* Graph-based

이번글에서 부터는 Graph-based SLAM에 대해서 설명할 것이며, 이 글에서는 Graph-based SLAM의 접근방법인 Least square에 대해서 설명한다.

### Least square

* Least square는 "overdetermined system"의 해를 구하기 위한 방법이다. overdetermined system이란 미지수의 갯수보다 식의 수가 더 많기 때문에, 모든 식을 만족하는 해가 존재하지 않는 시스템을 말한다.
* Least square는 에러의 제곱합(sum of the squared error)을 최소화 하는 방법으로 해를 구한다.

### Problem of Least square

* $$\mathbf{x}$$는 state vector
* $$\mathbf{z}_i$$는 실제 measurement(실제 센서 측정값)
* $$\hat{\mathbf{x}}_i=f_i(x)$$는 현재 state vector $$\mathbf{x}$$에서 예상되는 measurement값
* 목표: 센서로부터 실제 얻어진 measurement($$\mathbf{z}_i$$)에 가장 적합한 $$\mathbf{x}$$ 추정

<img align="middle" src="/images/post/SLAM/lec12_least_square/least_square.png" width="100%">

위 그림은 least square를 그림으로 표현하였다. Least square는 센서로부터 측정된 measurement에 가장 적합한 로봇의 state를 계산하는 방법이다. Sensor observation model처럼 로봇의 상태를 알고 있을 때 예상되는 measurement를 예상할 수 있고($$\mathbf{\hat{z}}_i$$) 실제 measurement와 예상되는 measurement와의 차이를 최소화 함으로써 최적화된 로봇의 state를 계산할 수 있다.

### Error Function

Error($$\mathbf{e}_i$$)는 실제 measurement와 예상되는 measurement와의 차이를 나타내며, measurement와 같은 dimension을 갖는 vector이다.

$$
\mathbf{e}_i(\mathbf{x}) = \mathbf{z}_i - f_i(\mathbf{x})
$$

위 식에서 measurement의 noise는 평균이 0이며, Gaussian 분포를 따른다고 가정한다.
Information matrix($$\Omega_i$$)는 measurement의 covariance matrix의 inverse이며, measurement($$\mathbf{z}_i$$)와 dimension의 크기가 같은 matrix이다.

squared error($$e_i(\mathbf{x})$$)는 아래와 같으며, vector가 아닌 scalar값을 갖는다.

$$
e_i(\mathbf{x}) = \mathbf{e}_i^T(\mathbf{x}) \mathbf{\Omega}_i \mathbf{e}_i(\mathbf{x})
$$

### Find Minimum of Error Function

로봇의 최적화된 state를 계산하는 문제는, 주어진 모든 센서 measurement를 이용한 error function의 합이 최소화 되는 state를 찾는 문제이다. 식으로 표현하면 다음과 같다.

$$
\begin{aligned}
x* &= \text{argmin}_{\mathbf{x}} F(\mathbf{x})\\
&=\text{argmin}_{\mathbf{x}} \sum_i e_i(\mathbf{x})\\
&=\text{argmin}_{\mathbf{x}} \sum_i \mathbf{e}_i^T(\mathbf{x}) \mathbf{\Omega}_i \mathbf{e}_i(\mathbf{x})
\end{aligned}
$$

위 식에서 각 element들에 대해서는 error function에서 설명했다. 이 식을 보고 넘어갈 때 각 항들의 dimension이 어떻게 되는지 확인하고 넘어가도록 하자. 만약 센서 measurement의 dimension이 n일 때 $$\mathbf{e}_i$$는 $$n\times1$$, $$\mathbf{\Omega}_i$$는 $$n\times n$$의 dimension을 갖는다.
위와 같은 optimization 문제는 복잡하며, closed form(미지수의 해가 수식으로 정리되는 형태)이 아니기 때문에 수학적인 접근방법으로 해를 구한다.

Optimization 방법으로 최적의 로봇 state($$\mathbf{x}* $$)를 구하는 순서는 다음과 같다.

1. 현재의 state를 기준으로 error term($$\mathbf{e(x)}$$)의 선형화
2. 선화된 error term으로 계산되는 squared error function($e_i(\mathbf{x})$)의 1차 미분계산
3. squared error function은 quadratic form이므로 1차 미분값이 0이 되는 점이 최소값이므로 미분값이 0인 solution을 계산
4. 계산된 solution을 이용하여 state를 update
5. 위의 과정을 수렴할 때까지 반복

위의 과정을 통해 iterative하게 optimal한 state를 계산할 수 있다. 각 단계의 과정을 자세히 살펴보도록 하자.

#### 1. Error term의 선형화

vector의 형태를 갖는 error term은 현재 measurement와 예상되는 measurement의 차이이다.

$$
\mathbf{e}_i(\mathbf{x}) = \mathbf{z}_i - f_i(\mathbf{x})
$$

이때 $$f_i(\mathbf{x})$$는 observation model로 많은 경우에 비선형형태를 갖는다. observation model에 대해 익숙하지 않다면 이전 글인 [motion model and observation model](http://jinyongjeong.github.io/2017/02/14/lec02_motion_observation_model/)을 참고하기 바란다. 따라서 식을 풀기 위해서는 현재 state기준으로 선형화를 수행한다. 선형화 방법으로는 EKF와 마찬가지로 Tayor expansion방법을 사용한다.

$$
\begin{aligned}
\mathbf{e}_i(\mathbf{x}+\triangle\mathbf{x}) &\approx \mathbf{e}_i(\mathbf{x}) + \mathbf{J}_i(\mathbf{x})((\mathbf{x}+\triangle \mathbf{x})-\mathbf{x})\\
&= \mathbf{e}_i(\mathbf{x}) + \mathbf{J}_i(\mathbf{x})\triangle \mathbf{x}
\end{aligned}
$$

위 식에서 $$\mathbf{x}$$는 initialize의 기준이 되는 현재 state를 의미하며 $$\mathbf{x} = (\mathbf{x}_1, \mathbf{x}_2, \cdots, \mathbf{x}_n)$$이다. 이때 Jacobian $$\mathbf{J}_i$$는 다음과 같다.

$$
\mathbf{J}_i(x) = \begin{pmatrix} \frac{\partial f_1(x)}{\partial x_1} & \frac{\partial f_1(x)}{\partial x_2} & \cdots & \frac{\partial f_1(x)}{\partial x_n} \\
\frac{\partial f_2(x)}{\partial x_1} & \frac{\partial f_2(x)}{\partial x_2} & \cdots & \frac{\partial f_2(x)}{\partial x_n} \\
\cdots & \cdots & \cdots & \cdots \\
\frac{\partial f_m(x)}{\partial x_1} & \frac{\partial f_m(x)}{\partial x_2} & \cdots & \frac{\partial f_m(x)}{\partial x_n}
\end{pmatrix}
$$

#### 2. Squared Error 계산

이제 위에서 선형화한 error term을 이용하여 squared error를 계산해보자.

$$
\begin{aligned}
e_i(\mathbf{x}+\triangle\mathbf{x}) & =
\mathbf{e}_i^T(\mathbf{x}+\triangle\mathbf{x}) \mathbf{\Omega}_i \mathbf{e}_i(\mathbf{x}+\triangle\mathbf{x})\\
&\approx (\mathbf{e}_i(\mathbf{x}) + \mathbf{J}_i(\mathbf{x})\triangle \mathbf{x})^T \mathbf{\Omega}_i (\mathbf{e}_i(\mathbf{x}) + \mathbf{J}_i(\mathbf{x})\triangle \mathbf{x})\\
&= \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{e}_i + \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{J}_i \triangle \mathbf{x} +  \triangle \mathbf{x}^T \mathbf{J}_i^T  \mathbf{\Omega}_i \mathbf{e}_i + \triangle \mathbf{x}^T \mathbf{J}_i^T  \mathbf{\Omega}_i \mathbf{J}_i \triangle \mathbf{x}\\
&= \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{e}_i + 2 \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{J}_i \triangle \mathbf{x} + \triangle \mathbf{x}^T \mathbf{J}_i^T  \mathbf{\Omega}_i \mathbf{J}_i \triangle \mathbf{x}\\
&= c_i + 2 \mathbf{b}_i^T \triangle \mathbf{x} + \triangle \mathbf{x}^T \mathbf{H}_i \triangle \mathbf{x}
\end{aligned}
$$

계산된 squared error는 위와 같으며, 마지막 식에서 $$c_i, \mathbf{b}_i, \mathbf{H}_i$$는 다음과 같다.

* $$c_i = \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{e}_i$$
* $$\mathbf{b}_i = \mathbf{e}_i^T \mathbf{\Omega}_i \mathbf{J}_i $$
* $$\mathbf{H}_i = \mathbf{J}_i^T  \mathbf{\Omega}_i \mathbf{J}_i $$

즉 H는 선형화 후의 information matrix로 Gaussian covariance matrix의 선형화 후의 형태와 같다.

그렇다면 이제 squared error function의 마지막 식을 살펴보자.

$$
\begin{aligned}
e_i(\mathbf{x}+\triangle\mathbf{x})
&= c_i + 2 \mathbf{b}_i^T \triangle \mathbf{x} + \triangle \mathbf{x}^T \mathbf{H}_i \triangle \mathbf{x}
\end{aligned}
$$

위의 식은 $$\triangle \mathbf{x}$$에 대해서 quadratic한 형태를 띄고 있다. quadratic form은 일반 2차방정식의 형태와 유사하다. 즉, 함수의 최대 혹은 최대값은 함수의 미분이 0이 되는 지점을 계산함으로써 계산할 수 있다.




**본 글을 참조하실 때에는 출처 명시 부탁드립니다.**