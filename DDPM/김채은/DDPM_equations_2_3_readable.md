# DDPM 식 (2)

- 식 (1)은 이미지를 복원하는 역방향 과정
- 식 (2)는 노이즈를 추가하는 정방향 과정
- 식 (3)은 모델이 복원을 잘하도록 학습시키는 손실함수

## ① 전체 정방향 과정

$$q(x_{1:T}\mid x_0):=\prod_{t=1}^{T}q(x_t\mid x_{t-1})$$

왼쪽 `q(x₁:ₜ | x₀)`는 원본 데이터 x₀가 주어졌을 때 중간 상태 x₁, …, xₜ가 만들어질 확률분포

오른쪽 식:

$$
\prod_{t=1}^{T}q(x_t\mid x_{t-1})
=q(x_1\mid x_0)q(x_2\mid x_1)\cdots q(x_T\mid x_{T-1})
$$

정방향 과정은 Markov chain으로, xₜ를 만들 때 바로 이전 상태 xₜ₋₁만 사용

## ② xₜ₋₁에서 xₜ를 만드는 한 단계

$$q(x_t\mid x_{t-1}):=\mathcal N\left(x_t;\sqrt{1-\beta_t}x_{t-1},\beta_tI\right)$$

- 새로 만들 값: xₜ
- 평균: √(1−βₜ)xₜ₋₁
- 공분산: βₜI

## 실제 계산식

표준 가우시안 노이즈를 하나 뽑음

$$\epsilon_t\sim\mathcal N(0,I)$$

그다음 다음과 같이 계산

$$x_t=\sqrt{1-\beta_t}x_{t-1}+\sqrt{\beta_t}\epsilon_t$$

이 식은

- 기존 신호: √(1−βₜ)xₜ₋₁
- 새로 추가하는 노이즈: √βₜ εₜ

즉, 기존 이미지를 조금 약하게 만들고 그 자리에 랜덤 노이즈를 조금 넣음

## βₜ의 의미

여기에서 βₜ는 t번째 단계에서 넣을 노이즈의 양을 정하는 값

$$0<\beta_t<1$$

단계별 노이즈 크기 전체를 β₁, β₂, …, βₜ라고 하는데, 이 값들의 배열을 variance schedule이라고 함

→초반에는 노이즈를 조금 넣고 뒤로 갈수록 더 많이 넣는 방식

중요한 점은 βₜ가 신경망이 학습하는 값이 아니라 사람이 미리 정한 값이라는 것

## √βₜ를 곱하는 이유

확률변수에 숫자 a를 곱하면 분산에는 a²이 곱해짐

$$\mathrm{Var}(aX)=a^2\mathrm{Var}(X)$$

우리는 노이즈의 공분산을 $\beta_tI$로 만들어야 함

기본 노이즈는 $\epsilon_t\sim\mathcal N(0,I)$이므로 여기에 $\sqrt{\beta_t}$를 곱하면 다음과 같음

$$\sqrt{\beta_t}\,\epsilon_t\sim\mathcal N(0,\beta_tI)$$

## 기존 신호에 √(1−βₜ)를 곱하는 이유

노이즈만 계속 더하면 값의 전체 크기가 계속 커질 수 있음

따라서 새로운 노이즈를 넣는 만큼 기존 신호를 조금 줄여야 함

xₜ₋₁의 분산이 1이라고 가정하면:

- 기존 신호 부분의 분산: 1−βₜ
- 새 노이즈 부분의 분산: βₜ

따라서 전체 분산은 다음과 같음

$$\left(1-\beta_t\right)+\beta_t=1$$

# DDPM 식 (3)

원본에 노이즈를 넣은 다음 모델이 그 노이즈를 얼마나 잘 제거하여 원본 이미지로 돌아오는지 확인.

잘 돌아오면 L이 작아지고, 학습하면서 L을 계속 줄임

## 왼쪽 식

$$
\mathbb E[-\log p_\theta(x_0)]
\leq
\mathbb E_q\left[-\log\frac{p_\theta(x_{0:T})}{q(x_{1:T}\mid x_0)}\right]
$$

## 오른쪽 식

$$
\mathbb E_q\left[-\log p(x_T)-\sum_{t\geq1}\log\frac{p_\theta(x_{t-1}\mid x_t)}{q(x_t\mid x_{t-1})}\right]=:L
$$

목적은 모델이 실제 데이터 x₀을 높은 확률로 생성하게 만드는 것

먼저 pθ(x₀)는 모델이 실제 데이터 x₀에 부여한 확률밀도. 모델이 실제 이미지를 잘 생성한다면 이 값이 커야 함.

## 로그를 붙이는 이유

확률은 여러 단계의 확률을 곱해서 계산

$$p(x_T)p_\theta(x_{T-1}\mid x_T)\cdots p_\theta(x_0\mid x_1)$$

작은 숫자를 곱하면 값이 매우 작아져 컴퓨터로 계산하기 불안정, 로그를 사용하면 곱셈이 덧셈으로 바뀜

로그는 증가함수이므로 다음 관계가 성립

$$p_\theta(x_0)\text{가 커짐}\iff\log p_\theta(x_0)\text{도 커짐}$$

## 마이너스를 붙이는 이유

원래 클수록 높은 확률. 여기에 마이너스를 붙이면 반대로 최소화할수록 높은 확률이 됨.

머신러닝에서의 표현은

> 잘 예측함 → 손실(loss)이 작음

하지만 이 값을 계산하기 어려워 상한선 L을 만듬.

$$\mathbb E[-\log p_\theta(x_0)]\leq L$$

## 기댓값 E와 q에 대한 기댓값으로 변환

`E[−log pθ(x₀)]`에서 E는 기댓값, 평균을 의미. 

따라서 `E[−log pθ(x₀)]`는 전체 데이터에 대한 평균을 의미.

문제 : 원래 모든 경로를 더해야 하지만 가능한 중간 경로가 너무 많아 직접 적분하기 어려움

해결 : 적분을 q에서 노이즈를 뽑아 계산하는 평균으로 바꿈

평균의 정의는 다음과 같음

$$\mathbb E_{z\sim q}[f(z)]=\int q(z)f(z)\,dz$$

이는 다음과 같이 쓸 수 있음

$$\int q(z\mid x_0)\frac{p_\theta(x_0,z)}{q(z\mid x_0)}\,dz$$

## Jensen 부등식 적용

여기에 −log를 적용하고 Jensen 부등식을 사용하면:

$$-\log\mathbb E_q\left[\frac{p_\theta(x_{0:T})}{q(x_{1:T}\mid x_0)}\right]$$

이 다음 값보다 작거나 같다는 결과가 나옴

$$\mathbb E_q\left[-\log\frac{p_\theta(x_{0:T})}{q(x_{1:T}\mid x_0)}\right]$$

## 최종 상한

따라서:

$$-\log p_\theta(x_0)\leq\mathbb E_q\left[-\log\frac{p_\theta(x_{0:T})}{q(x_{1:T}\mid x_0)}\right]$$

## 분수의 위와 아래

$$\frac{p_\theta(x_{0:T})}{q(x_{1:T}\mid x_0)}$$

위쪽의 pθ(x₀:ₜ)는 모델의 역방향 생성 경로이고, 아래쪽의 q(x₁:ₜ | x₀)는 정방향 노이즈 경로

이 손실을 줄이면서 q가 만든 노이즈 경로를 반대로 복구하도록 학습

## 다른 형태로 전개

식 (1)에 따르면:

$$p_\theta(x_{0:T})=p(x_T)\prod_{t=1}^{T}p_\theta(x_{t-1}\mid x_t)$$

식 (2)에 따르면:

$$q(x_{1:T}\mid x_0)=\prod_{t=1}^{T}q(x_t\mid x_{t-1})$$

이를 분수에 넣으면:

$$\frac{p_\theta(x_{0:T})}{q(x_{1:T}\mid x_0)}=\frac{p(x_T)\prod_{t=1}^{T}p_\theta(x_{t-1}\mid x_t)}{\prod_{t=1}^{T}q(x_t\mid x_{t-1})}$$

같은 t의 항끼리 묶으면:

$$p(x_T)\prod_{t=1}^{T}\frac{p_\theta(x_{t-1}\mid x_t)}{q(x_t\mid x_{t-1})}$$

가 된다.

로그의 성질 `log(ab) = log a + log b` 때문에

$$\log\prod_{t=1}^{T}a_t=\sum_{t=1}^{T}\log a_t$$

따라서:

$$-\log\left[p(x_T)\prod_{t=1}^{T}\frac{p_\theta(x_{t-1}\mid x_t)}{q(x_t\mid x_{t-1})}\right]$$

을 풀면:

$$-\log p(x_T)-\sum_{t=1}^{T}\log\frac{p_\theta(x_{t-1}\mid x_t)}{q(x_t\mid x_{t-1})}$$

가 된다.