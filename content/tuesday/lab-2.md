# Lab 2 - Changing Tempo

*Note: This lab will start at the end of Lab 1. In case you were unable to complete it, you can download a starting directory here (TODO: add link).*

In this lab, we will be exploring how chemical different chemical composition gradients ($\nabla_{\mu}$, where $\mu$ is the mean-molecular weight) impact the $g$-mode period spacing. 

As a review, the oscillation period, $\Pi_{n, \ell}$ for a $g$-mode of radial order $n$ and spherical degree $\ell$ is given by
$$
\Pi_{n, \ell} = \frac{\Pi_0}{\sqrt{\ell(\ell + 1)}}(n+\epsilon),
$$
Where $\epsilon$ is a small constant and $\Pi_0$ is given by:
$$
\Pi_0 = 2\pi^2\left(\int \frac{N}{r}\,dr\right)^{-1},
$$
Where $N$ is the Brunt-Väisälä frequency and the integral is taken over the propagation cavity of the $g$-mode. The period spacing, $\Delta \Pi_{\ell}$, is then the period difference between two modes of consecutive order and the same spherical degree:
$$
\Delta\Pi_{\ell} = \Pi_{n+1, \ell} - \Pi_{n, \ell}.
$$ 
In the asymptotic limit (large $n$ or $n$) for a chemically homogenous medium, this spacing is approximately a constant for all $n$. This is what we observed when we plotted $\Delta \Pi_{1}$ vs. $\Pi_{n, 1}$ for our zero-age main-sequence (ZAMS) model in Lab 1. At the ZAMS--the start of hydrogen ignition--`MESA` begins with a chemically homogenous mixture; i.e., $\nabla_{\mu}=0$. Thus, for sufficiently high $n$ the asymptotic relation behaves as expected.

However, as soon as the star evolves beyond the ZAMS, hydrogen fusion disrupts the chemical homogeneity and introduces a chemical gradient. The gradient causes sharp spikes in the Brunt-Väisälä frequency, which is--in an ideal gas--described by:
$$
N^2 \approx \frac{g^2\rho}{p}\left(\nabla_{\mathrm{ad}}-\nabla+\nabla_{\mu}\right),
$$
where
$$
\nabla = \frac{d\,\ln T}{d\, \ln p}, \\ \nabla_{\mathrm{ad}}=\left(\frac{\partial \, \ln T}{\partial \, \ln p}\right)_{\mathrm{ad}}, \\ \mathrm{and} \\ \nabla_{\mu}=\frac{d\, \ln \mu}{d\, \n p}.
$$