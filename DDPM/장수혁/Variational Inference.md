### 정의
계산하기 어려운 **확률분포**를
**파라미터**를 가지고 **근사**하는 것

#### 사용방식
보통 다음 KL divergence를 최소화하는 것이 목표
$$\phi^* := \arg\min_\phi D_{\mathrm{KL}} \left( q_\phi(\mathbf z\mid\mathbf x) \;\|\; p(\mathbf z\mid\mathbf x) \right)$$
