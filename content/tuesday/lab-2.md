# Lab 2 - Changing Tempo

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
\nabla = \frac{d\ln T}{d\ln p}, \quad
\nabla_{\mathrm{ad}} = \left(\frac{\partial \ln T}{\partial \ln p}\right)_{\mathrm{ad}}, \quad
\mathrm{and} \quad
\nabla_{\mu} = \frac{d\ln \mu}{d\ln p}.
$$

If $\nabla_{\mu}\neq = 0$, then spikes in the Brunt-Väisälä will *trap* $g$-modes; such mode trapping leads to periodic *dips* in a plot of $\Delta \Pi_{1}$ vs. $\Pi_{n, 1}$. We are going to induce a chemical gradient in our model by evolving through the main-sequence to investigate these dips.

Start by copying a clean working directory from `$MESA_DIR/star/work` and placing it where you want it to be. It is helpful to rename it something descriptive at this point as well--something like `day2_lab2`.

> [!WARNING]
> It is generally not a great idea to work directly inside the clean `$MESA_DIR/star/work` work directory.
> It is instead generally best practice to copy it and place it somewhere else before making any changes.

{{< details title="Hint: Copying a work directory" closed="true" >}}
We will copy the clean directory from `$MESA_DIR/star/work` and place it somewhere else using the Linux command `cp`. If you are already where you want the work directory to be, you can use the shortcut `./` to say "place this here". Otherwise, replace `./` with the path to your intended location. If you were to use the shortcut, it would look like this:
```
cp -r $MESA_DIR/star/work ./day2_lab2
```
The `-r` is a flag that tells the system to copy the work directory *recursively*. In other words, it copies all of the contents inside the directory, not just the directory itself. If you have any problems, make sure that your `MESA` environment variables are set. This will not work if `$MESA_DIR` is undefined.
{{< /details >}}
