# Lab 2 - Changing Tempo

## Learning Goals

In this lab, we will be exploring how chemical different chemical composition gradients ($\nabla_{\mu}$, where $\mu$ is the mean-molecular weight) impact the *g*-mode period spacing. To do this, we will learn how to:

- Run `MESA` starting from a precomputed `.mod` file
- Modify `run_star_extras.f90` to output profiles at set points in time 

## Background Science
As a review, the oscillation period, $\Pi_{n, \ell}$ for a *g*-mode of radial order $n$ and spherical degree $\ell$ is given by
$$
\Pi_{n, \ell} = \frac{\Pi_0}{\sqrt{\ell(\ell + 1)}}(n+\epsilon),
$$
Where $\epsilon$ is a small constant and $\Pi_0$ is given by:
$$
\Pi_0 = 2\pi^2\left(\int \frac{N}{r}\,dr\right)^{-1},
$$
Where $N$ is the Brunt-Väisälä frequency and the integral is taken over the propagation cavity of the *g*-mode. The period spacing, $\Delta \Pi_{\ell}$, is then the period difference between two modes of consecutive order and the same spherical degree:
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

If $\nabla_{\mu} \neq 0$, then spikes in the Brunt-Väisälä will *trap* *g*-modes; such mode trapping leads to periodic *dips* in a plot of $\Delta \Pi_{1}$ vs. $\Pi_{n, 1}$. We are going to induce a chemical gradient in our model by evolving through the main-sequence to investigate these dips.

## Task 1: A Fresh Start
**Start by copying a clean working directory from `$MESA_DIR/star/work` and placing it where you want it to be.** It is helpful to rename it something descriptive at this point as well--something like `day2_lab2`.

> [!WARNING]
> It is generally not a great idea to work directly inside the clean `$MESA_DIR/star/work` work directory. It is instead best practice to copy it and place it somewhere else before making any changes.

{{< details title="Hint: Copying a work directory" closed="true" >}}
We will copy the clean directory from `$MESA_DIR/star/work` and place it somewhere else using the Linux command `cp`. If you are already where you want the work directory to be, you can use the shortcut `./` to say "place this here". Otherwise, replace `./` with the path to your intended location. If you were to use the shortcut, it would look like this:
```bash
cp -r $MESA_DIR/star/work ./day2_lab2
```
The `-r` is a flag that tells the system to copy the work directory *recursively*. In other words, it copies all of the contents inside the directory, not just the directory itself. If you have any problems, make sure that your `MESA` environment variables are set. This will not work if `$MESA_DIR` is undefined.
{{< /details >}}

## Task 2: Getting the (`&star_`) Job Done
For now, we are only going to edit `inlist_project`; open it up and take a second to see what the basic work inlist looks like.

We are going to build our inlist one namelist at a time. Starting at the top, we are going to alter the `&star_job` namelist first. For this section, the [star_job reference page](https://docs.mesastar.org/en/26.4.1/reference/star_job.html) will be a helpful resource.

Rather than create a pre-main sequence model every time we run our inlist, we are going to start from a pre-computed zero-age main sequence model named `zams.mod`, which you can download [here](download=/content/tuesday/zams.mod). This model is for a $5 \, \mathrm{M_{\odot}}$ star with solar metallicty at the zero-age main sequence.

We also won't need to save a model at the end of our run, since we are only interested in what is happening *during* the main-sequence (we are going to terminate at the terminal-age main sequence). Therefore, we **can remove the three lines that refer to creating a pre-main sequence model or saving a model at termination**.

In their place, we need to load our `zams.mod` file and tell `MESA` to start our new run there. **Look at the `star_job` reference page and see if you can sort out how to load a saved model**. If you get stuck you can expand the hint below.

{{< details title="Hint: Loading a saved model" closed="true" >}}
To load a saved model, we need to tell `MESA` to load a model, *and* tell it the name of the model we want to load. We do with these to lines:
```fortran
load_saved_model = .true.
load_model_filename = 'zams.mod'
```
At this point, your `&star_job` namelist should look something like this:
```fortran
&star_job

   ! see star/defaults/star_job.defaults

   ! start run with zams.mod
   load_saved_model = .true.
   load_model_filename = 'zams.mod'

   ! display on-screen plots
   pgstar_flag = .true.

/ ! end of star_job namelist
```
{{< /details >}}

We don't need to make any changes to the `&eos` and `&kap` namelists. They are sufficient for what we are trying to do here.

> [!NOTE]
> For science cases, you may want to consider changing parameters in these namelists. It isn't guaranteed that the defaults will always be right for your particular case.

Now we can move on to the `&controls` namelist. For this section, the [controls reference page](https://docs.mesastar.org/en/26.4.1/reference/controls.html) will be helpful.

The basic `inlist_project` file helpfully breaks the `&controls` namelist into smaller chunks that can tell you about what the namelist can do. 

Starting in the "starting specifications" section, we need to **change the `initial_mass` to 5** since our initial model will expect a 5 solar-mass star. We can leave the metallicity as it is.

We can tell our model when to terminate in the "when to stop" section. The current stopping condition is set to terminate at the zero-age main sequence. Since we are *starting* at the ZAMS, we can go ahead and **delete everything from `! when to stop` to `! wind`**. Instead, we will add our own stopping condition. Since we are interested mainly in what is happening during the main-sequence, we really only need to pick a stopping condition that terminates our model after the section we are interested in. To make sure this happens, we are going to end our model at the terminal-age main sequence. This will also give us an opportunity to see what the period spacing looks like there. **Explore the "when to stop" section of the controls reference page and find a stopping condition that terminates at the TAMS**. If you get stuck, feel free to expand the following hint.

{{< details title="Hint: Stopping at the TAMS" closed="true" >}}
It is likely that in your exploration you came across this stopping condition:
```fortran
stop_at_phase_TAMS = .true.
```
This condition will definitely terminate when we want it to, but the preset `stop_at_phase_X` conditions have actual definitions based on the physics happening at that phase; it is a good idea to familiarize yourself with the actual definitions before using a phase-based stopping condition. 

If you look for the subroutine `set_phase_of_evolution` in `$MESA_DIR/star/private/star_utils.f90` and search for the definition of `phase_TAMS`, you will see that it is defined by the core hydrogen abundance:
```fortran
         else if (center_h1 <= 1d-6) then
            s% phase_of_evolution = phase_TAMS
```
In other words, `phase_TAMS` is triggered when central hydrogen dips below 1 part in a million. That means that `stop_at_phase_TAMS = .true.` is okay to use as a stopping condition; however, an entirely equivalent stopping condition could have been:
```fortran
xa_central_lower_limit_species(1) = 'h1'
xa_central_lower_limit(1) = 1d-6
```
At this point, your `&controls` namelist should look something like:
```fortran
&controls

   ! see star/defaults/controls.defaults

   ! starting specifications
   initial_mass = 5 ! in Msun units
   initial_z = 0.02

   ! when to stop
   ! stop at the terminal age main sequence (TAMS)
   ! defined by center_h1 <= 1d-6 (core hydrogen depletion)
   ! see line 2957 of $MESA_DIR/star/private/star_utils.f90  
   stop_at_phase_TAMS = .true.

   ! wind

   ! atmosphere

   ! rotation

   ! element diffusion

   ! mlt

   ! mixing

   ! timesteps

   ! mesh

   ! solver
   ! options for energy conservation (see MESA V, Section 3)
   energy_eqn_option = 'dedt'
   use_gold_tolerances = .true.

   ! output

/ ! end of controls namelist
```
{{< /details >}}

The next section we will need to change is the "mesh" section. This section is for controls related to the spacial resolution of our model. `MESA` operates by separating a stellar model into concentric rings with some thickness we will call $\Delta r$. To make sure that important physics are captured, `MESA` also has an adaptive mesh refinement (AMR) algorithm that will decrease $\Delta r$ in regions as needed or increase $\Delta r$ in regions where high resolution isn't necessary. However, it is entirely possible that the AMR algorithm doesn't fully capture the physics you are interested in--sometimes you may want a higher spatial resolution. In our case, `GYRE` is sensitive to spatial resolution, so we will want to increase the default resolution.

To increase the resolution of our model, we are going to use the variable `mesh_delta_coeff`, which is a multiplicative factor on $\Delta r$. In other words, it is $a$ in the expression $a\Delta r$. By default, `MESA` sets `mesh_delta_coeff = 1`. If we want to *increase* the resolution, we need to set `mesh_delta_coeff` to a value *less than 1* to break our model into smaller chunks.

Increasing the resolution of a model can be a bit more computationally expensive depending on your machine. TODO: section about picking a dmesh value

Finally, we will need to edit the "output" section of the namelist to make sure that our model makes the data we need to give to GYRE. `MESA` does not create a pulsation profile by default, so we need to turn on the correct flag to make sure it does. **Explore the "controls for output" section of the controls reference page and find the flag to write pulsation profiles for GYRE**. Check the following hint if you get stuck or want to check your answer

{{< details title="Hint: Saving pulsation profiles for `GYRE`" closed="true" >}}
The flag we are interested in is `write_pulse_data_with_profile`, which will save the data we need for `GYRE` later; set it to `.true.`.

We also need to specify the format of the pulsation data. We are going to use `.FGONG` files, which are compatible with `GYRE`. Altogether, your output section should look something like:
```fortran
   ! output

   ! setting profile format for GYRE
   write_pulse_data_with_profile = .true.
   pulse_data_format = 'FGONG'
```
{{< /details >}}

With the correct flag, `MESA` will write a unique `.FGONG.data` file for every profile it creates. We want to make sure that `MESA` only writes these profiles when we want them and doesn't write any profiles otherwise. This will require some extra code in our `./src/run_star_extras.f90`, which we will get to later. In the meantime, we are going to turn profile generation off by including the following line in our inlist:
```fortran
profile_interval = -1
```
Since you can't write a profile "every minus one" steps, `MESA` interpets this `-1` as `off`.

This final step is not strictly necessary but can be helpful if you are rerunning multiple times and want to keep a clean working directory. Since we are interested in only specific points in time for this lab and not the evolution of the star over it's whole lifetime, we don't really need a `history.data` file. Similarly, since we aren't going to be restarting our runs in the middle, we won't need photos either. We can turn history file and photo generation off with the following lines:
```fortran
photo_interval = -1
do_history_file = .false.
```
>[!IMPORTANT]
> You should only do this if you are *certain* that you don't need such data. Otherwise, you may be keeping yourself from the data you need.

At this point, our inlist should be almost complete. To make sure everything is correct, we can run our inlist:
```fortran
./clean; ./mk; ./rn
```
>[!NOTE]
> Placing a semicolon between subsequent commands lets you run each command in one line rather than typing/running each command individually.

If data starts printing to the terminal, your inlist is correct and you are ready to move to the next step. If you have gotten an error, work with your TA to find the cause or expand the following hint for a complete inlist.

{{< details title="Hint: Complete ZAMS to TAMS Inlist" closed="true" >}}
At this point, your full inlist should look something like this:
```fortran

&star_job

   ! see star/defaults/star_job.defaults

   ! start run with zams.mod
   load_saved_model = .true.
   load_model_filename = 'zams.mod'

   ! display on-screen plots
   pgstar_flag = .true.

/ ! end of star_job namelist


&eos

   ! eos options
   ! see eos/defaults/eos.defaults

/ ! end of eos namelist


&kap
   ! kap options
   ! see kap/defaults/kap.defaults
   use_Type2_opacities = .true.
   Zbase = 0.02

/ ! end of kap namelist


&controls

   ! see star/defaults/controls.defaults

   ! starting specifications
   initial_mass = 5 ! in Msun units
   initial_z = 0.02

   ! when to stop
   ! stop at the terminal age main sequence (TAMS)
   ! defined by center_h1 <= 1d-6 (core hydrogen depletion)
   ! see line 2957 of $MESA_DIR/star/private/star_utils.f90  
   stop_at_phase_TAMS = .true.

   ! wind

   ! atmosphere

   ! rotation

   ! element diffusion

   ! mlt

   ! mixing

   ! timesteps

   ! mesh
   ! increase mesh "resolution"
   mesh_delta_coeff = 0.8

   ! solver
   ! options for energy conservation (see MESA V, Section 3)
   energy_eqn_option = 'dedt'
   use_gold_tolerances = .true.

   ! output

   ! setting profile format for GYRE
   write_pulse_data_with_profile = .true.
   pulse_data_format = 'FGONG'

   ! don't save profiles unless told otherwise
   ! NECESSARY; otherwise we can't guarantee
   ! profiles will save at the expected times
   profile_interval = -1
   
   ! don't save photos or a history file to save space
   ! not strictly necessary
   photo_interval = -1
   do_history_file = .false.

/ ! end of controls namelist

```
Make sure you have an empty line at the end! If you don't have an empty line `MESA` will not run!
{{< /details >}}