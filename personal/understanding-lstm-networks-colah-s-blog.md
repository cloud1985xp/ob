---
tags:
  - links
  - bookmarks
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Understanding LSTM Networks -- colah's blog

Created: 2019年7月26日 下午8:20
URL: http://colah.github.io/posts/2015-08-Understanding-LSTMs/?source=post_page---------------------------

![](Understanding%20LSTM%20Networks%20--%20colah's%20blog/RNN-unrolled.png)

An unrolled recurrent neural network.

![](Understanding%20LSTM%20Networks%20--%20colah's%20blog/RNN-shorttermdepdencies.png)

![](Understanding%20LSTM%20Networks%20--%20colah's%20blog/RNN-longtermdependencies.png)

![](Understanding%20LSTM%20Networks%20--%20colah's%20blog/LSTM3-SimpleRNN.png)

The repeating module in a standard RNN contains a single layer.

![](Understanding%20LSTM%20Networks%20--%20colah's%20blog/LSTM3-chain.png)

The repeating module in an LSTM contains four interacting layers.

Darwin Kim —

I think there's a much simpler way to prove that the optimal code makes the cost equal to the word frequency. However, many of the clever tricks and ideas in the proof are still used.

I will use code word property functions as variables to avoid confusion between constant and variable functions.

- Let f be the (derivative of the) average length contribution, f(x) = p, such that

∫f(x)dx from 0 to L

equals the average length contribution.

- Let g be the cost of the code word, g(x) = 1/(2^x). Consequently,

∫g(x)dx from L to infinity,

the area of the cost as shown in many of the illustrations used, becomes

[1/ln(2)]*g(L)

1. As shown above, the area of the cost function is g(x), the true cost, scaled by a constant.

2. The total cost must be no larger than 1, the total availible cost. Therefore, the total area of the cost must be 1/ln(2), a constant value, using code words optimally.

3. The optimal encoding must minimize the total length contribution, the sum of all ∫f(x)dx. Since the sum of all ∫g(x)dx is constant, this is equivalent to minimizing the total area,

the sum of all ∫f(x)dx + ∫g(x)dx,

ignoring interval notation.

- Let the graph when

word frequency = cost of word

be h, equal to the graph of the intersection of the two functions.

h(x) = p when x ≤ log2(1/p) 1/(2^x) when x > log2(1/p)

N.B. The definition of h(x) is not nessesary for this proof.

4. The total area will always be greater than or equal to that of h(x), with equality holding if and only if p = cost. The total area, in any situation, regardless of the value of L, can never be larger than

∫h(x)dx from 0 to infinity,

since for all positive values of x, h(x) ≤ f(x) and h(x) ≤ g(x). Any deviance of L from this point will result in a portion of the graph "sticking out" when compared with h(x), as can be observed in some of the illustrations. Therefore h(x) is the solution to the optimal encoding problem.

- The key of this proof is to understand that the sum of the cost remains constant. Minimizing the sum of all average length contributions is the same as minimizing the total area of all graphs for every codeword. The inequality of h and "optimalness" of h becomes intuitive when the graphs are visualized correctly. I hope the limitations of language and UTF-8 does not keep you from seeing the clarity and intuitiveness of this proof that I had seen.