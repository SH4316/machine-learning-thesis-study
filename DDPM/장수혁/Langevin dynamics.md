Langevin-dynamics : Dynamic 분자 시스템을 수학적 모델링하는 것

- Langevin equation 사용

###### Score 
-> 현재 위치에서 데이터 밀도가 가장 빠르게 증가하는 방향
$$s(x)=\nabla_x \log p(x)$$

###### Lengevin dynamics의 연속시간표현?
$$d x = \frac{1}{2}\nabla_x \log p(x)\,dt + dW_t$$
위치로 표현 -> $x_{k+1}=x_k+\epsilon s(x_k)+\sqrt{2\epsilon}z_k$

