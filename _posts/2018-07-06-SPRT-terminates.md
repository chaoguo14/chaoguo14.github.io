---
layout: post
title: Sequential probability ratio test terminates with probability 1
---
**Theorem**: Sequential probability ratio test terminates with probability 1.

_Proof_: Assume $a \le 0 \le b$.

Since $P\left\\{Y_i = 0\right\\} < 1$, there exists some $\delta>0$ such that $\mathbb{P}\left\\{Y_i > \delta\right\\} \ge \delta$, or there exists some $\delta > 0$ such that $\mathbb{P}\left\\{Y_i < -\delta\right\\} \ge \delta$. If this is not the case, we have $\mathbb{P}\left\\{Y_i > 0\right\\} \le 0$ and $\mathbb{P}\left\\{Y_i < 0\right\\}\le 0$, which implies a contradiction.

Without losing generality, assume that there exists $\delta > 0$ such that $\mathbb{P}\left\\{Y_i \ge \delta \right\\} \ge \delta$. Let $r = \left(b-a\right)/\delta$. Note that $\mathbb{P}\left\\{Y_1 + \cdots + Y_r > b-a\right\\}\ge \mathbb{P}\left\\{Y_i > \delta \text{ for }i=1,\cdots,r \right\\}=\delta^r$. So $\mathbb{P}\left\\{ S_r \le b-a\right\\}\le 1 - \delta^r$. So $\mathbb{P}\left\\{ S_r < b\right\\} \le \mathbb{P}\left\\{S_r \le b - a\right\\} \le 1 - \delta^r$.

To bound $\mathbb{P}\left\\{ N > n\right\\}$, we first notice that

$$
\begin{align*}
\mathbb{P}\left\{ N > r\right\} &=\mathbb{P}\left\{ a < S_i < b \text{ for }i=1,\cdots,r\right\} \\
&\le \mathbb{P}\left\{ a < S_r < b\right\} \\
&\le \mathbb{P}\left\{ S_r < b\right\}\\
&\le 1- \delta^r
\end{align*}
$$

Given $N > kr$, if $Y_{kr + 1} + \cdots + Y_{\left(k+1\right)r} > b - a$, then we must have $N \le \left(k+1\right)$. This is because if $N> kr$, then $a < S_{kr} < b$. So $S_{\left(k+1\right)r} > b$. Then the test has to terminate by $\left(k+1\right)r$. This gives us

$$
\begin{align*}
\mathbb{P}\left\{ N \le \left(k+1\right) r \mid N > kr\right\} &\ge \mathbb{P}\left\{ Y_{kr + 1} + \cdots + Y_{(k+1)r} > b-a\mid N > kr\right\}\\
&= \mathbb{P}\left\{Y_{kr + 1} + \cdots + Y_{(k+1)r} > b-a \right\}\\
&= \mathbb{P}\left\{Y_1 + \cdots + Y_r > b-a \right\}\\
&\ge  \delta^r
\end{align*}
$$

So $\mathbb{P}\left\\{ N > \left(k+1\right) r \mid N > kr\right\\}\le1-\delta^r$.

If the test terminates after $kr$, then the test terminates after $kr$ and after $\left(k-1\right)r$. On the other hand, if the test terminates after $kr$ and after $\left(k-1\right)r$, then the test terminates after $kr$. So we apply the previous result recursively and have

$$
\begin{align*}
\mathbb{P}\left\{N > kr\right\} &= \mathbb{P}\left\{N>kr \text{ and } N > \left(k-1\right)\right\}\\
&=\mathbb{P}\left\{N > kr \mid N > \left(k-1\right)r\right\}\mathbb{P}\left\{N > \left(k-1\right)r\right\}\\
&\le \left(1-\delta^r\right) \mathbb{P}\left\{N > \left(k-1\right)r\right\}\\
&\cdots\\
&\le \left(1-\delta^r\right)^k
\end{align*}
$$

Now, for any $n$, we can find some $k$ such that $kr < n < \left(k+1\right)r$. We have

$$\mathbb{P}\left\{ N > n\right\} \le \mathbb{P}\left\{N > kr\right\} \le \left(1 - \delta^r\right)^k \le \left(1 - \delta^r\right)^{n/r} =\frac{1}{\left(1-\delta^r\right)^r}\cdot \left(1-\delta^r\right)^n$$

which shows that $\mathbb{P}\left\\{N > n\right\\} \le C \rho^n$.
