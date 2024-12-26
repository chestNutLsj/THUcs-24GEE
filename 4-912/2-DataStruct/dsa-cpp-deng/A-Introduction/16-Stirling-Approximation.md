---
publish:
---
## Intro

In mathematics, **Stirling's approximation** (or **Stirling's formula**) is an approximation for factorials. It is a good approximation, leading to accurate results even for small values of $n$. It is named after James Stirling, though a related but less precise result was first stated by Abraham de Moivre.

![[16-Stirling-Approximation-compare-to-n!.png]]

One way of stating the approximation involves the logarithm of the factorial:
$${\displaystyle \ln(n!)=n\ln n-n+O(\ln n),}$$
where the big O notation means that, for all sufficiently large values of $n$, the difference between ${\displaystyle \ln(n!)}$ and ${\displaystyle n\ln n-n}$ will be at most proportional (正比于) to the logarithm. 


In computer science applications such as the worst-case lower bound for comparison-based sorting, it is convenient to instead use the binary logarithm, giving the equivalent form  
$${\displaystyle \log _{2}(n!)=n\log _{2}n-n\log _{2}e+O(\log _{2}n).}$$
The error term in either base can be expressed more precisely as ${\displaystyle {\tfrac {1}{2}}\log(2\pi n)+O({\tfrac {1}{n}})}$, corresponding to an approximate formula for the factorial itself, ${\displaystyle n!\sim {\sqrt {2\pi n}}\left({\frac {n}{e}}\right)^{n}.}$ Here the sign $\sim$ means that the two quantities are asymptotic, that is, that their ratio tends to 1 as $n$ tends to infinity. 

The following version of the bound holds for all ${\displaystyle n\geq 1}$  , rather than only asymptotically: 
$${\displaystyle {\sqrt {2\pi n}}\ \left({\frac {n}{e}}\right)^{n}e^{\frac {1}{12n+1}}<n!<{\sqrt {2\pi n}}\ \left({\frac {n}{e}}\right)^{n}e^{\frac {1}{12n}}.}$$

## Derivation

Roughly speaking, the simplest version of Stirling's formula can be quickly obtained by approximating the sum
 ${\displaystyle \ln(n!)=\sum _{j=1}^{n}\ln j}$ with an integral: ${\displaystyle \sum _{j=1}^{n}\ln j\approx \int _{1}^{n}\ln x\,{\rm {d}}x=n\ln n-n+1.}$

The full formula, together with precise estimates of its error, can be derived as follows. Instead of approximating $n!$, one considers its natural logarithm, as this is a slowly varying function:

 ${\displaystyle \ln(n!)=\ln 1+\ln 2+\cdots +\ln n.}$

The right-hand side of this equation minus

${\displaystyle {\tfrac {1}{2}}(\ln 1+\ln n)={\tfrac {1}{2}}\ln n}$ is the approximation by the trapezoid rule of the integral ${\displaystyle \ln(n!)-{\tfrac {1}{2}}\ln n\approx \int _{1}^{n}\ln x\,{\rm {d}}x=n\ln n-n+1,}$

and the error in this approximation is given by the Euler–Maclaurin formula:

 ![{\displaystyle {\begin{aligned}\ln(n!)-{\tfrac {1}{2}}\ln n&={\tfrac {1}{2}}\ln 1+\ln 2+\ln 3+\cdots +\ln(n-1)+{\tfrac {1}{2}}\ln n\\&=n\ln n-n+1+\sum _{k=2}^{m}{\frac {(-1)^{k}B_{k}}{k(k-1)}}\left({\frac {1}{n^{k-1}}}-1\right)+R_{m,n},\end{aligned}}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/559acb779d1d9e2ca828e3f8ebec37ebee2a8b6e)

where  ![B_{k}](https://wikimedia.org/api/rest_v1/media/math/render/svg/6a6457760e36cf45e1471e33bcc1536cb4802fb9) is a Bernoulli number, and _R__m_,_n_ is the remainder term in the Euler–Maclaurin formula. Take limits to find that

 ![{\displaystyle \lim _{n\to \infty }\left(\ln(n!)-n\ln n+n-{\tfrac {1}{2}}\ln n\right)=1-\sum _{k=2}^{m}{\frac {(-1)^{k}B_{k}}{k(k-1)}}+\lim _{n\to \infty }R_{m,n}.}](https://wikimedia.org/api/rest_v1/media/math/render/svg/e1de677b895e298f6e743494ca20df4f6c33349f)

Denote this limit as  ![y](https://wikimedia.org/api/rest_v1/media/math/render/svg/b8a6208ec717213d4317e666f1ae872e00620a0d). Because the remainder _R__m_,_n_ in the Euler–Maclaurin formula satisfies

 ![{\displaystyle R_{m,n}=\lim _{n\to \infty }R_{m,n}+O\left({\frac {1}{n^{m}}}\right),}](https://wikimedia.org/api/rest_v1/media/math/render/svg/385ea6e95b78cc25641016606c4d729fb1d9dfc3)

where big-O notation is used, combining the equations above yields the approximation formula in its logarithmic form:

 ![{\displaystyle \ln(n!)=n\ln \left({\frac {n}{e}}\right)+{\tfrac {1}{2}}\ln n+y+\sum _{k=2}^{m}{\frac {(-1)^{k}B_{k}}{k(k-1)n^{k-1}}}+O\left({\frac {1}{n^{m}}}\right).}](https://wikimedia.org/api/rest_v1/media/math/render/svg/28b026d2e5f73bf6ce88c47c248844d4cf622a73)

Taking the exponential of both sides and choosing any positive integer  ![m](https://wikimedia.org/api/rest_v1/media/math/render/svg/0a07d98bb302f3856cbabc47b2b9016692e3f7bc), one obtains a formula involving an unknown quantity  ![e^{y}](https://wikimedia.org/api/rest_v1/media/math/render/svg/7ff26085237c8dc6802eba0882a8aea22e890183). For _m_ = 1, the formula is

 ![{\displaystyle n!=e^{y}{\sqrt {n}}\left({\frac {n}{e}}\right)^{n}\left(1+O\left({\frac {1}{n}}\right)\right).}](https://wikimedia.org/api/rest_v1/media/math/render/svg/c3280cd1e77e7ec163297311e38d28a2bcbc33c1)

The quantity  ![e^{y}](https://wikimedia.org/api/rest_v1/media/math/render/svg/7ff26085237c8dc6802eba0882a8aea22e890183) can be found by taking the limit on both sides as  ![n](https://wikimedia.org/api/rest_v1/media/math/render/svg/a601995d55609f2d9f5e233e36fbe9ea26011b3b) tends to infinity and using Wallis' product, which shows that  ![{\displaystyle e^{y}={\sqrt {2\pi }}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/aa0fcfb78cc1c5ec93e19890d73eb3631fbfb6aa). Therefore, one obtains Stirling's formula:
$${\displaystyle n!={\sqrt {2\pi n}}\left({\frac {n}{e}}\right)^{n}\left(1+O\left({\frac {1}{n}}\right)\right)}$$

## Alternative derivations

An alternative formula for  ![n!](https://wikimedia.org/api/rest_v1/media/math/render/svg/bae971720be3cc9b8d82f4cdac89cb89877514a6) using the gamma function is

 ![{\displaystyle n!=\int _{0}^{\infty }x^{n}e^{-x}\,{\rm {d}}x.}](https://wikimedia.org/api/rest_v1/media/math/render/svg/ec3d16f1466e7bef37e4674bce9f3ca8baa5a43b) (as can be seen by repeated integration by parts). Rewriting and changing variables _x_ = _ny_, one obtains  ![{\displaystyle n!=\int _{0}^{\infty }e^{n\ln x-x}\,{\rm {d}}x=e^{n\ln n}n\int _{0}^{\infty }e^{n(\ln y-y)}\,{\rm {d}}y.}](https://wikimedia.org/api/rest_v1/media/math/render/svg/b5c1d168cbbe92d515939df7b8703b02bac5aa0f) Applying Laplace's method one has  ![{\displaystyle \int _{0}^{\infty }e^{n(\ln y-y)}\,{\rm {d}}y\sim {\sqrt {\frac {2\pi }{n}}}e^{-n},}](https://wikimedia.org/api/rest_v1/media/math/render/svg/1cb2e06231b3f5f3502bb001fface31c74836498) which recovers Stirling's formula:  ![{\displaystyle n!\sim e^{n\ln n}n{\sqrt {\frac {2\pi }{n}}}e^{-n}={\sqrt {2\pi n}}\left({\frac {n}{e}}\right)^{n}.}](https://wikimedia.org/api/rest_v1/media/math/render/svg/b91b7ad167af44b3b4bb07390fce77ba1ba23879)

### Higher orders

In fact, further corrections can also be obtained using Laplace's method. From previous result, we know that  ![{\displaystyle \Gamma (x)\sim x^{x}e^{-x}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/65cbf7864fae628b3dbee15f718cedf1167b6b03), so we "peel off" this dominant term, then perform a change of variables, to obtain:

 ![{\displaystyle x^{-x}e^{x}\Gamma (x)=\int _{\mathbb {R} }e^{x(1+t-e^{t})}dt}](https://wikimedia.org/api/rest_v1/media/math/render/svg/16c287e185c84a185918e750559381cf0a02e94f) Now the function  ![{\displaystyle t\mapsto 1+t-e^{t}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/df85098ac94220d1b2f6f7f33d40c86ca1b969bb) is unimodal, with maximum value zero. Locally around zero, it looks like  ![{\displaystyle -t^{2}/2}](https://wikimedia.org/api/rest_v1/media/math/render/svg/87a40dd6615f474e0416abe2054ab61156437375), which is why we are able to perform Laplace's method. In order to extend Laplace's method to higher orders, we perform another change of variables by  ![{\displaystyle 1+t-e^{t}=-\tau ^{2}/2}](https://wikimedia.org/api/rest_v1/media/math/render/svg/fdc08f8f909f9fa9fb1c11a37fe2532327ffedf9). This equation cannot be solved in closed form, but it can be solved by serial expansion, which gives us  ![{\displaystyle t=\tau -\tau ^{2}/6+\tau ^{3}/36+a_{4}\tau ^{4}+O(\tau ^{5})}](https://wikimedia.org/api/rest_v1/media/math/render/svg/f0309cb95f6a07fb980fc574633583ad027ea72e). Now plug back to the equation to obtain  ![{\displaystyle x^{-x}e^{x}\Gamma (x)=\int _{\mathbb {R} }e^{-x\tau ^{2}/2}(1-\tau /3+\tau ^{2}/12+4a_{4}\tau ^{3}+O(\tau ^{4}))d\tau ={\sqrt {2\pi }}(x^{-1/2}+x^{-3/2}/12)+O(x^{-5/2})}](https://wikimedia.org/api/rest_v1/media/math/render/svg/53413a29c58763f2cc53f7f32fce6513f57ee2b1) notice how we don't need to actually find  ![a_4](https://wikimedia.org/api/rest_v1/media/math/render/svg/c9fde542c1a0ee6390f05d9c0a58e9de213e4415), since it is cancelled out by the integral. Higher orders can be achieved by computing more terms in  ![{\displaystyle t=\tau +\cdots }](https://wikimedia.org/api/rest_v1/media/math/render/svg/914f0925da4067a78b1976f1bd9be2966c62d1ba).

Thus we get Stirling's formula to two orders:

 ![{\displaystyle n!={\sqrt {2\pi n}}\left({\frac {n}{e}}\right)^{n}\left(1+{\frac {1}{12n}}+O\left({\frac {1}{n^{2}}}\right)\right).}](https://wikimedia.org/api/rest_v1/media/math/render/svg/6cfc1fbd35d1f53867d7cce1047d3ceaf6741f4e)

### Complex-analytic version

A complex-analysis version of this method is to consider  ![{\displaystyle {\frac {1}{n!}}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/13afd849707d08294eb548c3947c0f29801a4ad2) as a Taylor coefficient of the exponential function  ![{\displaystyle e^{z}=\sum _{n=0}^{\infty }{\frac {z^{n}}{n!}}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/62afa4bdef0bbdf72a026e48d8089103e84fa37d), computed by Cauchy's integral formula as

 ![{\displaystyle {\frac {1}{n!}}={\frac {1}{2\pi i}}\oint \limits _{|z|=r}{\frac {e^{z}}{z^{n+1}}}\,\mathrm {d} z.}](https://wikimedia.org/api/rest_v1/media/math/render/svg/9c8e7fbd26bd3cb1707d6e67604a7f64734ba243)

This line integral can then be approximated using the saddle-point method with an appropriate choice of contour radius  ![{\displaystyle r=r_{n}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/402caef90569e54e6e8864e299f869dd849df91e). The dominant portion of the integral near the saddle point is then approximated by a real integral and Laplace's method, while the remaining portion of the integral can be bounded above to give an error term.

Speed of convergence and error estimates
----------------------------------------

![](//upload.wikimedia.org/wikipedia/commons/thumb/3/37/Stirling_series_relative_error.svg/400px-Stirling_series_relative_error.svg.png)The relative error in a truncated Stirling series vs.  ![n](https://wikimedia.org/api/rest_v1/media/math/render/svg/a601995d55609f2d9f5e233e36fbe9ea26011b3b), for 0 to 5 terms. The kinks in the curves represent points where the truncated series coincides with Γ(_n_ + 1).

Stirling's formula is in fact the first approximation to the following series (now called the **Stirling series**):

 ![{\displaystyle n!\sim {\sqrt {2\pi n}}\left({\frac {n}{e}}\right)^{n}\left(1+{\frac {1}{12n}}+{\frac {1}{288n^{2}}}-{\frac {139}{51840n^{3}}}-{\frac {571}{2488320n^{4}}}+\cdots \right).}](https://wikimedia.org/api/rest_v1/media/math/render/svg/3b2cf3c9b6fe771f6b3c1179a846ed77f19f4f89)

An explicit formula for the coefficients in this series was given by G. Nemes. Further terms are listed in the On-Line Encyclopedia of Integer Sequences as A001163 and A001164. The first graph in this section shows the relative error vs.  ![n](https://wikimedia.org/api/rest_v1/media/math/render/svg/a601995d55609f2d9f5e233e36fbe9ea26011b3b), for 1 through all 5 terms listed above. (Bender and Orszag p. 218) gives the asymptotic formula for the coefficients:

 ![{\displaystyle A_{2j+1}\sim (-1)^{j}2(2j)!/(2\pi )^{2(j+1)}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/7e99c9ce19477b939bdfffc501b95a1ed9df9b3a) which shows that it grows superexponentially, and that by ratio test the radius of convergence is zero.

![](//upload.wikimedia.org/wikipedia/commons/thumb/a/a8/Stirling_error_vs_number_of_terms.svg/400px-Stirling_error_vs_number_of_terms.svg.png)The relative error in a truncated Stirling series vs. the number of terms used

As _n_ → ∞, the error in the truncated series is asymptotically equal to the first omitted term. This is an example of an asymptotic expansion. It is not a convergent series; for any _particular_ value of  ![n](https://wikimedia.org/api/rest_v1/media/math/render/svg/a601995d55609f2d9f5e233e36fbe9ea26011b3b) there are only so many terms of the series that improve accuracy, after which accuracy worsens. This is shown in the next graph, which shows the relative error versus the number of terms in the series, for larger numbers of terms. More precisely, let _S_(_n_, _t_) be the Stirling series to  ![t](https://wikimedia.org/api/rest_v1/media/math/render/svg/65658b7b223af9e1acc877d848888ecdb4466560) terms evaluated at   ![n](https://wikimedia.org/api/rest_v1/media/math/render/svg/a601995d55609f2d9f5e233e36fbe9ea26011b3b). The graphs show

 ![{\displaystyle \left|\ln \left({\frac {S(n,t)}{n!}}\right)\right|,}](https://wikimedia.org/api/rest_v1/media/math/render/svg/451995218e426dea9963cd62fd1caaa712d0ae8c) which, when small, is essentially the relative error.

Writing Stirling's series in the form

 ![{\displaystyle \ln(n!)\sim n\ln n-n+{\tfrac {1}{2}}\ln(2\pi n)+{\frac {1}{12n}}-{\frac {1}{360n^{3}}}+{\frac {1}{1260n^{5}}}-{\frac {1}{1680n^{7}}}+\cdots ,}](https://wikimedia.org/api/rest_v1/media/math/render/svg/7ea7279df14c37194e55c825ac63e749ba4c2a98) it is known that the error in truncating the series is always of the opposite sign and at most the same magnitude as the first omitted term.

More precise bounds, due to Robbins, valid for all positive integers  ![n](https://wikimedia.org/api/rest_v1/media/math/render/svg/a601995d55609f2d9f5e233e36fbe9ea26011b3b) are

 ![{\displaystyle {\sqrt {2\pi n}}\left({\frac {n}{e}}\right)^{n}e^{\frac {1}{12n+1}}<n!<{\sqrt {2\pi n}}\left({\frac {n}{e}}\right)^{n}e^{\frac {1}{12n}}.}](https://wikimedia.org/api/rest_v1/media/math/render/svg/7326478542ba974e9a5b120756b3ca0cd834a404) A looser version of this bound is that  ![{\displaystyle {\frac {n! e^{n}}{n^{n+{\frac {1}{2}}}}}\in ({\sqrt {2\pi }}, e]}]( https://wikimedia.org/api/rest_v1/media/math/render/svg/809b4def5f2cd3a4f66ee3651b61f8c5d1f7e5b8 ) for all  ![n\geq 1](https://wikimedia.org/api/rest_v1/media/math/render/svg/d8ce9ce38d06f6bf5a3fe063118c09c2b6202bfe).

Stirling's formula for the gamma function
-----------------------------------------

For all positive integers,

 ![{\displaystyle n!=\Gamma (n+1),}](https://wikimedia.org/api/rest_v1/media/math/render/svg/07589a814a57c666e6403264017e580514e75bc3) where Γ denotes the gamma function.

However, the gamma function, unlike the factorial, is more broadly defined for all complex numbers other than non-positive integers; nevertheless, Stirling's formula may still be applied. If Re (_z_) > 0, then

 ![{\displaystyle \ln \Gamma (z)=z\ln z-z+{\tfrac {1}{2}}\ln {\frac {2\pi }{z}}+\int _{0}^{\infty }{\frac {2\arctan \left({\frac {t}{z}}\right)}{e^{2\pi t}-1}}\,{\rm {d}}t.}](https://wikimedia.org/api/rest_v1/media/math/render/svg/c398a644d2ccc7ffaf82be5478403b9a5baea09b)

Repeated integration by parts gives

 ![{\displaystyle \ln \Gamma (z)\sim z\ln z-z+{\tfrac {1}{2}}\ln {\frac {2\pi }{z}}+\sum _{n=1}^{N-1}{\frac {B_{2n}}{2n(2n-1)z^{2n-1}}},}](https://wikimedia.org/api/rest_v1/media/math/render/svg/8d257681480aa1bed0237cb9239f1f722a1786e7)

where  ![B_{n}](https://wikimedia.org/api/rest_v1/media/math/render/svg/2f568bf6d34e97b9fdda0dc7e276d6c4501d2045) is the  ![n](https://wikimedia.org/api/rest_v1/media/math/render/svg/a601995d55609f2d9f5e233e36fbe9ea26011b3b) th Bernoulli number (note that the limit of the sum as  ![N\to \infty ](https://wikimedia.org/api/rest_v1/media/math/render/svg/e23159ea0d291e21c5709a6dd7486bed7f18febe) is not convergent, so this formula is just an asymptotic expansion). The formula is valid for  ![z](https://wikimedia.org/api/rest_v1/media/math/render/svg/bf368e72c009decd9b6686ee84a375632e11de98) large enough in absolute value, when |arg (_z_)| < π − _ε_, where ε is positive, with an error term of _O_(_z_−2_N_+ 1). The corresponding approximation may now be written:

 ![{\displaystyle \Gamma (z)={\sqrt {\frac {2\pi }{z}}}\,{\left({\frac {z}{e}}\right)}^{z}\left(1+O\left({\frac {1}{z}}\right)\right).}](https://wikimedia.org/api/rest_v1/media/math/render/svg/5ed4a7a21192f91f42e90e0f2dbc5cbc851a089a)

where the expansion is identical to that of Stirling's series above for  ![n!](https://wikimedia.org/api/rest_v1/media/math/render/svg/bae971720be3cc9b8d82f4cdac89cb89877514a6), except that  ![n](https://wikimedia.org/api/rest_v1/media/math/render/svg/a601995d55609f2d9f5e233e36fbe9ea26011b3b) is replaced with _z_ − 1.

A further application of this asymptotic expansion is for complex argument z with constant Re (_z_). See for example the Stirling formula applied in Im (_z_) = _t_ of the Riemann–Siegel theta function on the straight line .

Error bounds
------------

For any positive integer  ![N](https://wikimedia.org/api/rest_v1/media/math/render/svg/f5e3890c981ae85503089652feb48b191b57aae3), the following notation is introduced:

 ![{\displaystyle \ln \Gamma (z)=z\ln z-z+{\tfrac {1}{2}}\ln {\frac {2\pi }{z}}+\sum \limits _{n=1}^{N-1}{\frac {B_{2n}}{2n\left({2n-1}\right)z^{2n-1}}}+R_{N}(z)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/19a5d992f27054d73eb99a5ab9d9f15acfbec0a3) and  ![{\displaystyle \Gamma (z)={\sqrt {\frac {2\pi }{z}}}\left({\frac {z}{e}}\right)^{z}\left({\sum \limits _{n=0}^{N-1}{\frac {a_{n}}{z^{n}}}+{\widetilde {R}}_{N}(z)}\right).}](https://wikimedia.org/api/rest_v1/media/math/render/svg/56d62236ddd9cbfabb820fba18895c77655c433b)

Then

$${\displaystyle {\begin{aligned}|R_{N}(z)|&\leq {\frac {|B_{2N}|}{2N (2N-1)|z|^{2N-1}}}\times {\begin{cases}1&{\text{ if }}\left|\arg z\right|\leq {\frac {\pi }{4}},\\\left|\csc (\arg z)\right|&{\text{ if }}{\frac {\pi }{4}}<\left|\arg z\right|<{\frac {\pi }{2}},\\\sec ^{2N}\left ({\tfrac {\arg z}{2}}\right)&{\text{ if }}\left|\arg z\right|<\pi ,\end{cases}}\\[6pt]\left|{\widetilde {R}}_{N}(z)\right|&\leq \left ({\frac {\left|a_{N}\right|}{|z|^{N}}}+{\frac {\left|a_{N+1}\right|}{|z|^{N+1}}}\right)\times {\begin{cases}1&{\text{ if }}\left|\arg z\right|\leq {\frac {\pi }{4}},\\\left|\csc (2\arg z)\right|&{\text{ if }}{\frac {\pi }{4}}<\left|\arg z\right|<{\frac {\pi }{2}}.\end{cases}}\end{aligned}}}$$

For further information and other error bounds, see the cited papers.

## A convergent version of Stirling's formula
------------------------------------------

Thomas Bayes showed, in a letter to John Canton published by the Royal Society in 1763, that Stirling's formula did not give a convergent series. Obtaining a convergent version of Stirling's formula entails evaluating Binet's formula:

 ![{\displaystyle \int _{0}^{\infty }{\frac {2\arctan \left({\frac {t}{x}}\right)}{e^{2\pi t}-1}}\,{\rm {d}}t=\ln \Gamma (x)-x\ln x+x-{\tfrac {1}{2}}\ln {\frac {2\pi }{x}}.}](https://wikimedia.org/api/rest_v1/media/math/render/svg/d912992e78260a9dbbd0ba446b96be2d2dfca212)

One way to do this is by means of a convergent series of inverted rising factorials. If

 ![{\displaystyle z^{\bar {n}}=z(z+1)\cdots (z+n-1),}](https://wikimedia.org/api/rest_v1/media/math/render/svg/5b8f38ab7b86f762cedbb9395458a1afb36b0578) then  ![{\displaystyle \int _{0}^{\infty }{\frac {2\arctan \left({\frac {t}{x}}\right)}{e^{2\pi t}-1}}\,{\rm {d}}t=\sum _{n=1}^{\infty }{\frac {c_{n}}{(x+1)^{\bar {n}}}},}](https://wikimedia.org/api/rest_v1/media/math/render/svg/ad3f584996c74270de2e6681eee0f6bf2f07ef91) where  ![{\displaystyle c_{n}={\frac {1}{n}}\int _{0}^{1}x^{\bar {n}}\left(x-{\tfrac {1}{2}}\right)\,{\rm {d}}x={\frac {1}{2n}}\sum _{k=1}^{n}{\frac {k|s(n,k)|}{(k+1)(k+2)}},}](https://wikimedia.org/api/rest_v1/media/math/render/svg/7bcdb1c8323f6e91e60b7038b1fb3e6be337fd11) where _s_(_n_, _k_) denotes the Stirling numbers of the first kind. From this one obtains a version of Stirling's series  ![{\displaystyle {\begin{aligned}\ln \Gamma (x)&=x\ln x-x+{\tfrac {1}{2}}\ln {\frac {2\pi }{x}}+{\frac {1}{12(x+1)}}+{\frac {1}{12(x+1)(x+2)}}+\\&\quad +{\frac {59}{360(x+1)(x+2)(x+3)}}+{\frac {29}{60(x+1)(x+2)(x+3)(x+4)}}+\cdots ,\end{aligned}}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/e92480dc9ee30c76420a560f402a4e660d5c3c93) which converges when Re (_x_) > 0. Stirling's formula may also be given in convergent form as  ![{\displaystyle \Gamma (x)={\sqrt {2\pi }}x^{x-{\frac {1}{2}}}e^{-x+\mu (x)}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/22fb124c11382df1c29111d531098cdfd56654fe) where  ![{\displaystyle \mu \left(x\right)=\sum _{n=0}^{\infty }\left(\left(x+n+{\frac {1}{2}}\right)\ln \left(1+{\frac {1}{x+n}}\right)-1\right).}](https://wikimedia.org/api/rest_v1/media/math/render/svg/0972561e44aa7b84fd9cb35f77abbecd8049b731)

Versions suitable for calculators
---------------------------------

The approximation

 ![{\displaystyle \Gamma (z)\approx {\sqrt {\frac {2\pi }{z}}}\left({\frac {z}{e}}{\sqrt {z\sinh {\frac {1}{z}}+{\frac {1}{810z^{6}}}}}\right)^{z}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/8117fdf62ad1e1991e255eefde12ec06ab4ce6eb) and its equivalent form  ![{\displaystyle 2\ln \Gamma (z)\approx \ln(2\pi )-\ln z+z\left(2\ln z+\ln \left(z\sinh {\frac {1}{z}}+{\frac {1}{810z^{6}}}\right)-2\right)}](https://wikimedia.org/api/rest_v1/media/math/render/svg/b70f63ef14c40546863b26a0635f6a8cd199992a) can be obtained by rearranging Stirling's extended formula and observing a coincidence between the resultant power series and the Taylor series expansion of the hyperbolic sine function. This approximation is good to more than 8 decimal digits for z with a real part greater than 8. Robert H. Windschitl suggested it in 2002 for computing the gamma function with fair accuracy on calculators with limited program or register memory.

Gergő Nemes proposed in 2007 an approximation which gives the same number of exact digits as the Windschitl approximation but is much simpler:

 ![{\displaystyle \Gamma (z)\approx {\sqrt {\frac {2\pi }{z}}}\left({\frac {1}{e}}\left(z+{\frac {1}{12z-{\frac {1}{10z}}}}\right)\right)^{z},}](https://wikimedia.org/api/rest_v1/media/math/render/svg/7bf6a375a8346a2ff56722096601b85a4347863c) or equivalently,  ![{\displaystyle \ln \Gamma (z)\approx {\tfrac {1}{2}}\left(\ln(2\pi )-\ln z\right)+z\left(\ln \left(z+{\frac {1}{12z-{\frac {1}{10z}}}}\right)-1\right).}](https://wikimedia.org/api/rest_v1/media/math/render/svg/7f05deb7cc6fb2336e1a3b2e62b68e7334251b0a)

An alternative approximation for the gamma function stated by Srinivasa Ramanujan (Ramanujan 1988[_clarification needed_]) is

 ![{\displaystyle \Gamma (1+x)\approx {\sqrt {\pi }}\left({\frac {x}{e}}\right)^{x}\left(8x^{3}+4x^{2}+x+{\frac {1}{30}}\right)^{\frac {1}{6}}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/e5a63e515690cf06b6bd412329fe7c1a625b74ea) for _x_ ≥ 0. The equivalent approximation for ln _n_! has an asymptotic error of 1/1400_n_3 and is given by  ![{\displaystyle \ln n!\approx n\ln n-n+{\tfrac {1}{6}}\ln(8n^{3}+4n^{2}+n+{\tfrac {1}{30}})+{\tfrac {1}{2}}\ln \pi .}](https://wikimedia.org/api/rest_v1/media/math/render/svg/2c561b7b2741bd7253fc53b352eb1c1b07967fd5)

The approximation may be made precise by giving paired upper and lower bounds; one such inequality is

 ![{\displaystyle {\sqrt {\pi }}\left({\frac {x}{e}}\right)^{x}\left(8x^{3}+4x^{2}+x+{\frac {1}{100}}\right)^{1/6}<\Gamma (1+x)<{\sqrt {\pi }}\left({\frac {x}{e}}\right)^{x}\left(8x^{3}+4x^{2}+x+{\frac {1}{30}}\right)^{1/6}.}](https://wikimedia.org/api/rest_v1/media/math/render/svg/70abbc9761518df503553faca29aac924e88cdb4)

## History
-------

The formula was first discovered by Abraham de Moivre in the form

 ${\displaystyle n!\sim [{\rm {constant}}]\cdot n^{n+{\frac {1}{2}}}e^{-n}.}$

De Moivre gave an approximate rational-number expression for the natural logarithm of the constant. Stirling's contribution consisted of showing that the constant is precisely  ![{\displaystyle {\sqrt {2\pi }}}](https://wikimedia.org/api/rest_v1/media/math/render/svg/7a9b009153bbbb3273a7e7279cb6b084fd650a80).

## See also
--------

*   Lanczos approximation
*   Spouge's approximation

## References
----------

1.  **^**
2.  ^ Jump up to: a b Le Cam, L. (1986), "The central limit theorem around 1935", _Statistical Science_, **1** (1): 78–96, doi: 10.1214/ss/1177013818, JSTOR 2245503, MR 0833276; see p. 81, "The result, obtained using a formula originally proved by de Moivre but now called Stirling's formula, occurs in his 'Doctrine of Chances' of 1733."
3.  ^ Jump up to: a b Pearson, Karl (1924), "Historical note on the origin of the normal curve of errors", _Biometrika_, **16** (3/4): 402–404 [p. 403], doi: 10.2307/2331714, JSTOR 2331714, I consider that the fact that Stirling showed that De Moivre's arithmetical constant was  ![{\sqrt {2\pi }}](https://wikimedia.org/api/rest_v1/media/math/render/svg/7a9b009153bbbb3273a7e7279cb6b084fd650a80) does not entitle him to claim the theorem, [...]
4.  **^** Flajolet, Philippe; Sedgewick, Robert (2009), Analytic Combinatorics, Cambridge, UK: Cambridge University Press, p. 555, doi: 10.1017/CBO9780511801655, ISBN 978-0-521-89806-5, MR 2483235, S2CID 27509971
5.  **^** Olver, F. W. J.; Olde Daalhuis, A. B.; Lozier, D. W.; Schneider, B. I.; Boisvert, R. F.; Clark, C. W.; Miller, B. R. & Saunders, B. V., "5.11 Gamma function properties: Asymptotic Expansions", _NIST Digital Library of Mathematical Functions_, Release 1.0.13 of 2016-09-16
6.  **^** Nemes, Gergő (2010), "On the coefficients of the asymptotic expansion of  ![n!](https://wikimedia.org/api/rest_v1/media/math/render/svg/bae971720be3cc9b8d82f4cdac89cb89877514a6)", _Journal of Integer Sequences_, **13** (6): 5
7.  **^** Bender, Carl M.; Orszag, Steven A. (2009). _Advanced mathematical methods for scientists and engineers. 1: Asymptotic methods and perturbation theory_ (Nachdr. ed.). New York, NY: Springer. ISBN 978-0-387-98931-0.
8.  **^** Robbins, Herbert (1955), "A Remark on Stirling's Formula", _The American Mathematical Monthly_, **62** (1): 26–29, doi: 10.2307/2308012, JSTOR 2308012
9.  **^** Spiegel, M. R. (1999), _Mathematical handbook of formulas and tables_, McGraw-Hill, p. 148
10.  **^** Schäfke, F. W.; Sattler, A. (1990), "Restgliedabschätzungen für die Stirlingsche Reihe", _Note di Matematica_, **10** (suppl. 2): 453–470, MR 1221957
11.  **^** G. Nemes, Error bounds and exponential improvements for the asymptotic expansions of the gamma function and its reciprocal, _Proc. Roy. Soc. Edinburgh Sect. A_ **145** (2015), 571–596.
12.  **^** Bayes, Thomas (24 November 1763), "A letter from the late Reverend Mr. Thomas Bayes, F. R. S. to John Canton, M. A. and F. R. S." (PDF), _Philosophical Transactions of the Royal Society of London Series I_, **53**: 269, Bibcode: 1763RSPT... 53.. 269B, archived (PDF) from the original on 2012-01-28, retrieved 2012-03-01
13.  **^** Artin, Emil (2015). _The Gamma Function_. Dover. p. 24.
14.  **^** Toth, V. T. Programmable Calculators: Calculators and the Gamma Function (2006) Archived 2005-12-31 at the Wayback Machine.
15.  **^** Nemes, Gergő (2010), "New asymptotic expansion for the Gamma function", _Archiv der Mathematik_, **95** (2): 161–169, doi: 10.1007/s00013-010-0146-9, S2CID 121820640
16.  **^** Karatsuba, Ekatherina A. (2001), "On the asymptotic representation of the Euler gamma function by Ramanujan", _Journal of Computational and Applied Mathematics_, **135** (2): 225–240, Bibcode: 2001JCoAM. 135.. 225K, doi: 10.1016/S0377-0427 (00) 00586-0, MR 1850542
17.  **^** Mortici, Cristinel (2011), "Ramanujan's estimate for the gamma function via monotonicity arguments", _Ramanujan J._, **25** (2): 149–154, doi: 10.1007/s11139-010-9265-y, S2CID 119530041
18.  **^** Mortici, Cristinel (2011), "Improved asymptotic formulas for the gamma function", _Comput. Math. Appl._, **61** (11): 3364–3369, doi: 10.1016/j.camwa. 2011.04.036.
19.  **^** Mortici, Cristinel (2011), "On Ramanujan's large argument formula for the gamma function", _Ramanujan J._, **26** (2): 185–192, doi: 10.1007/s11139-010-9281-y, S2CID 120371952.
