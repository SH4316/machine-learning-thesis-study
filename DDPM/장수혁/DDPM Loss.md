### 학습 관점
$$L := \mathbb E_{q} \left[ -\log p\left( \mathbf{x}_{T} \right) - \sum_{t \geq 1 } \log \frac{p_{\theta}(\mathbf{x}_{t-1}\mid\mathbf{x}_{t})}{q(\mathbf{x}_{t}\mid\mathbf{x_{t-1}})} \right] 
= \mathbb E_{q} \left[ -\log \frac{p_{\theta}(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T} \mid\mathbf{x}_{0})} \right]$$
- 하나의 데이터로 봤을 때 : $\mathbb E_{q}$ -> $q$는 고정 -> $p_{\theta}$를 학습 
	- : q에서 샘플링 한 상태로 $p_\theta$를 학습시킴
- $q(\mathbf{x}_{1:T} \mid\mathbf{x}_{0})$ : $x_{0}$가 주어졌을 때  나머지 Trajectory (궤적)
	- $q(\mathbf{x}_{1:T} \mid\mathbf{x}_{0})$ = $\Pi_{t=1}^T q(\mathbf{x}_{t} \mid \mathbf{x}_{t-1})$
- $p_{\theta}(\mathbf{x}_{0:T})$ : 는 역방향 예측 Trajectory

~~(설명은 하나의 궤적으로 설명했지만 $p$와 $q$는 여러 궤적이 나타나는 분포로서 존재함)~~


### Loss 식을 풀어 썼을 때

#### $p_{\theta}(x_{0})$의 의미
$$L \geq E [-\log p_{\theta}(\mathbf{x}_{0})]$$
$$p_\theta(x_0) = \int p_\theta(x_{0:T})\,dx_{1:T} = \int p(x_T)\prod_{t=1}^{T} p_\theta(x_{t-1}\mid x_t)\,dx_{1:T}$$

Marginal Likelihood (Margianl Distribution) 으로 적분으로 정의됨

#### 변경하는 이유

논문에서의 식

$$E [-\log p_{\theta}(\mathbf{x}_{0})] \leq \mathbb E_{q} \left[ -\log \frac{p_{\theta}(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T} \mid\mathbf{x}_{0})} \right]$$

실제 의미

$$\mathbb E_{\mathbf x_0\sim q_{\text{data}}} \left[-\log p_\theta(\mathbf x_0)\right] \le \mathbb E_{\substack{ \mathbf x_0\sim q_{\text{data}}\\ \mathbf x_{1:T}\sim q(\mathbf x_{1:T}\mid\mathbf x_0) }} \left[ -\log \frac{ p_\theta(\mathbf x_{0:T}) }{ q(\mathbf x_{1:T}\mid\mathbf x_0) } \right]$$

- 좌측 항은 실제 데이터에서 샘플링 (샘플링 안됨, 적분임)
- 우측항은 실제로 데이터를 얻을 수 있는 noise process 에서 분포를 추출함.

#### Loss 풀이
$$\begin{aligned} p_\theta(\mathbf x_0) &= \int p_\theta(\mathbf x_{0:T})\,d\mathbf x_{1:T} \\[4pt] &= \int q(\mathbf x_{1:T}\mid\mathbf x_0) \frac{p_\theta(\mathbf x_{0:T})} {q(\mathbf x_{1:T}\mid\mathbf x_0)} \,d\mathbf x_{1:T} \\[4pt] &= \mathbb E_{\mathbf x_{1:T}\sim q(\mathbf x_{1:T}\mid\mathbf x_0)} \left[ \frac{p_\theta(\mathbf x_{0:T})} {q(\mathbf x_{1:T}\mid\mathbf x_0)} \right]. \end{aligned}$$

1. Marginal Probability -> 적분식으로 변경
2. Importance Sampling -> 샘플링 분포 변경

$$\mathbb E_{\mathbf x_{1:T}\sim q(\mathbf x_{1:T}\mid\mathbf x_0)} \left[ \frac{p_\theta(\mathbf x_{0:T})} {q(\mathbf x_{1:T}\mid\mathbf x_0)} \right]$$
$$-\log p_\theta(\mathbf x_0) = -\log \mathbb E_q \left[ \frac{p_\theta(\mathbf x_{0:T})} {q(\mathbf x_{1:T}\mid\mathbf x_0)} \right]$$
$$-\log \mathbb E_q \left[ \frac{p_\theta(\mathbf x_{0:T})} {q(\mathbf x_{1:T}\mid\mathbf x_0)} \right] \leq \mathbb E_{q(\mathbf x_{1:T}\mid\mathbf x_0)} \left[ -\log \frac{p_\theta(\mathbf x_{0:T})} {q(\mathbf x_{1:T}\mid\mathbf x_0)} \right]$$
$$-\log p_\theta(\mathbf x_0) \leq \mathbb E_{q(\mathbf x_{1:T}\mid\mathbf x_0)} \left[ -\log \frac{p_\theta(\mathbf x_{0:T})} {q(\mathbf x_{1:T}\mid\mathbf x_0)} \right]$$

1. $-\log$ 이용해서 최적화 방향 변경
2. Jenson 부등식으로 최적화 대상 변경

### 변환
Loss 를 다음과 같이 분해할 수 있다.

$$L := \mathbb E_{q} \left[ -\log p\left( \mathbf{x}_{T} \right) - \sum_{t \geq 1 } \log \frac{p_{\theta}(\mathbf{x}_{t-1}\mid\mathbf{x}_{t})}{q(\mathbf{x}_{t}\mid\mathbf{x_{t-1}})} \right] 
= \mathbb E_{q} \left[ -\log \frac{p_{\theta}(\mathbf{x}_{0:T})}{q(\mathbf{x}_{1:T} \mid\mathbf{x}_{0})} \right]$$
$$\mathbb E_q\left[ \underbrace{ D_{\mathrm{KL}}\!\left( q(\mathbf x_T\mid\mathbf x_0)\Vert p(\mathbf x_T) \right)}_{L_T} + \sum_{t>1} \underbrace{ D_{\mathrm{KL}}\!\left( q(\mathbf x_{t-1}\mid\mathbf x_t,\mathbf x_0) \Vert p_\theta(\mathbf x_{t-1}\mid\mathbf x_t) \right)}_{L_{t-1}} - \underbrace{ \log p_\theta(\mathbf x_0\mid\mathbf x_1) }_{L_0} \right]$$

