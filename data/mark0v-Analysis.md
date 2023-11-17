# Canto


Findings
--------

Math inefficiency and inaccuracy

``/contracts/src/bonding_curve/LinearBondingCurve.sol``

the stuff going on in the loop is bad news, gas-efficiency wise, and potentially accuracy as well

```
        for (uint256 i = shareCount; i < shareCount + amount; i++) {
            uint256 tokenPrice = priceIncrease * i;
            price += tokenPrice;
            fee += (getFee(i) * tokenPrice) / 1e18;
        }

```

They are calculating
$$
\begin{align}
price = & \sum\limits_{i=M}^N P_I  i \\
fee = & \sum\limits_{i=M}^N \frac{P_I i}{10^{18}}\frac{10^{17}}{log_2(i)}
\end{align}
$$

where $P_I$ is ``priceIncrease``
the first term shouldnt be calculated in a loop, it is 

$$
\begin{align}
 price = & \sum\limits_{i=M}^N P_I * i =
& P_I \frac{(N+1)N - (M+1)M}{2} \\
& \frac{P_I}{2} (N+1)N - (M+1)M
\end{align}
$$

in our case $N$ is ``shareCount+amount``, ``shareCount`` would be $M$ so lets put in terms of ``amount`` I will call $A$: $N=M+A$

$$
(N+1)N - (M+1)M = (M+A+1(M+A) - (M+1)M = A(2M+A+1)
$$

so this should be in code as something like:

```
price = (priceIncrease*amount(2*shareCount+amount+1))>>1

```



$$
\begin{align}
fee = &  \sum\limits_{i=M}^N \frac{1}{10^{18}}\frac{10^{17} P_I i}{log_2(i)} \\
= & \frac{P_I}{10^{18}} \sum\limits_{i=M}^N \frac{10^{17} i}{log_2(i)}
\end{align}
$$

This only the term in the summation there should be calculated in the loop. the $10^{17}$ term is enough to keep calculations accurate. then the multiplication and as usually the division by WAD is the final operation. 

```
for (uint256 i = shareCount; i < shareCount + amount; i++) {
    if (shareCount>1){
        fee += (i*1e17)/log2(i);
    }else{
        fee+= 1e17
    }
}

fee *= priceIncrease
fee /= 1e18
```

That conditional shouldnt be in the loop really, there are many ways to remove it, but it could affect UX for the first buyer. Its maybe easiest to force the first buyer to buy more than 1 share, then have an if statement before the loop to tack on the fee for the first two and increment ``shareCount``




### Time spent:
8 hours