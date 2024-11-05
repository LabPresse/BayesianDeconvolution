# BayesianDeconvolution

Bayesian Deconvolution code accurately incorporates all sources of noise in the data and provides stricly positive solutions without arbitrary smoothness constraints or ad-hoc parameter tuning. It implements Markov chain Monte Carlo (MCMC) algorithms to learn probability distributions over the main function of interest: object intensity map. These tools can be used in a simple plug and play manner. Check the following paper to see details of all the mathematics involved.

https://arxiv.org/abs/2411.00991

## Julia Installation

**System Requirements: BayesianDeconvolution is a parallelized code and has been fully tested on modern desktops and laptops with at least 4 processor cores.  In addition to about twice the memory required to load raw image globally, the code typically requests about 250 MB of RAM to load Julia packages on each processor core.**

All the codes are written in Julia language for high performance/speed (similar to C and Fortran) and its open-source/free availability. Julia also allows easy parallelization of all the codes. To install julia, please download and install julia language installer from their official website (see below) for your operating system or use your package manager. The current version of the code has been successfully tested on linux (Ubuntu 22.04), macOS 12, and Windows.

https://julialang.org/

Like python, Julia also has an interactive environment (commonly known as REPL) which can be used to add packages and perform simple tests as shown in the picture below.


![Screenshot from 2023-11-08 14-36-41](https://github.com/ayushsaurabh/B-SIM/assets/87823118/05bdffb9-6857-4209-9d8d-97cedd3a3578)


In Windows, this interactive environment can be started by clicking on the Julia icon on Desktop that is created upon installation or by going into the programs menu directly. On Linux or macOS machines, julia REPL can be accessed by simply typing julia in the terminal. We use this environment to install some essential julia packages that help simplify linear algebra and statistical calculations, and plotting. To add these packages via julia REPL, **first enter the julia package manager by executing `]` command in the REPL**. Then simply execute the following command to install all these packages at the same time. 

```add Distributed, Random, SpecialFunctions, Distributions, LinearAlgebra, Statistics, Plots, HDF5, TiffImages, FFTW```


![Screenshot from 2023-11-08 14-40-31](https://github.com/ayushsaurabh/B-SIM/assets/87823118/27ffde07-7eb8-40a5-871b-cc4ea0e34859)

**To get out of the package manager, simply hit the backspace key.**

### Environment Creation
**This is for advanced users who already have Julia installed.**
If you already have Julia and do not want to alter your default environment, you can go to the directory where this software is, then 
1. Run Julia then type `]` and `activate .`;
2. Or run Julia in terminal via `julia --project`.
   
These two ways are equivalent. Both of them create a new Julia environment the first time you run it, or otherwise switch to this environment.

## Test Examples

Complex programs like BayesianDeconvolution require scripts for better organization instead of typing functions into the REPL for every run. The code is currently organized into multiple scripts. The main script "main.jl" calls all the functions performing deconvolution and the input parameter script "input_parameters.jl" defines all the input parameters needed to perform reconstruction (see the image below).

![image](https://github.com/user-attachments/assets/86bcf040-fbf9-4f1c-b0f6-ca04294bb68b)

These parameters define the shape of the microscope point spread function (numerical aperture, magnification, light wavelength), camera noise (gain, CCD sensitivity, readout), directory (folder) where files are located, file name, parallelization and inference settings. 

Using parameter files similar to the the image above, we here provide  simple plug and play example to test the functioning of BayesianDeconvolution on a personal computer. In the provided example for experimental mitochondria image in HeLa cells, we provide the input "raw_image.tif" file. 

Currently, BayesianDeconvolution accepts rectangular images. Furthermore, the current version provides two options for the PSF: "gaussian" and "airy_disk", but can be modified easily to incorporate any other shape.

With the settings in the image above, the code divides all the images into 4 sets of sub-images of equal sizes (a 2x2 grid). Next, each set of sub-images is then sent to their assigned processor and inference is performed on the object intensity map. The number of processors can be changed if running on a more powerful computer by changing "n_procs_per_dim_x" and "n_procs_per_dim_y" parameters, which are set to 2 by default.

To run this example, we suggest putting the scripts and the input tif files in the same folder/directory. Next, if running on a Windows machine, first confirm the current folder that julia is being run from by executing the following command in the REPL:

```pwd()```

**Please note here that Windows machines use backslashes "\\" to describe folder paths unlike Linux and macOS where forward slashes "/" are used.** Appropriate modifications therefore must be made to the folder paths. Now, if the output of the command above is different from the path containing the scripts and tiff files, the current path can be changed by executing the following command:

```cd("/home/username/BayesianDeconvolution/example_mitochondria/")```

BayesianDeconvolution code can now be executed by simply importing the "main.jl" in the REPL as shown in the picture below

```include("main.jl")```


![image](https://github.com/ayushsaurabh/B-SIM/assets/87823118/3c2056cf-7ee2-4d47-bc4d-d428e9c5a095)



On a linux or macOS machine, the "main.jl" script can be run directly from the terminal after entering the code directory and executing the following command:

```julia main.jl```

**WARNING: Please note that when running the code through the REPL, restart the REPL if the code throws an error. Every execution of the code adds processors to the julia REPL and processor ID or label increases in value. To make sure that processor labels always start at 1, we suggest avoiding restarting the code in the same REPL.**

Now, BayesianDeconvolution is a fully parallelized code and starts execution by first adding the required number of processors. Next, all the input tif files are imported and divided according to the parallelization grid (2x2 by default). The sub-images are then sent to each processor. All the functions involved in deconvolution are compiled next. Finally, the sampler starts and with each iteration outputs the log(posterior) values and a temperature parameter that users are not required to modify (see picture below). At the end of each iteration, sub-images are sent back to the master processor and combined into one image.

![image](https://github.com/user-attachments/assets/16d8569b-e245-4f26-bba3-2818cf2af63b)


Depending on the chosen plotting frequency in the "input_parameters.jl" file, the code also generates a plot showing the the log(posterior), the input raw image, current sample, and a mean of the previous samples (depending on averaging frequency) as shown in the picture below.

![image](https://github.com/user-attachments/assets/8bec8594-249b-4b7a-a0f2-d88dabdebbcf)



Finally, as samples are collected, the code saves intermediate samples and analysis data onto the hard drive in the TIFF format with file names that look like "mean_inferred_object_2.0.tif" and "inferred_object_2.0.tif". The save frequency can be modified by changing a few inference parameters in the "input_parameters.jl" file: "chain_burn_in_period" during which an optimizer quickly converges to a reasonable deconvolved image after the code is started; simulated annealing is restarted at regular intervals after chain burn-in and is set by the parameter "annealing_frequency"; simulated annealing starts with temperature set by "annealing_starting_temperature" and then the temperature decays exponentially with time constant set by "annealing_time_constant"; samples to be averaged are collected after the "annealing_burn_in_period" during which the sampler converges after increasing the temperature; and lastly, samples are collected at the "averaging_frequency" after the annealing burn-in period. Use of simulated annealing here helps uncorrelate the chain of samples by smoothing and widening the posterior at intermediate iterations by raising temperature, allowing the sampler to easily move far away from the current sample. Based on these parameters, the samples are saved whenever the following conditions are satisfied: 

```
 if (draw == chain_burn_in_period) || ((draw > chain_burn_in_period) &&
            ((draw - chain_burn_in_period) %
             annealing_frequency > annealing_burn_in_period ||
             (draw - chain_burn_in_period) % annealing_frequency == 0) &&
            ((draw - chain_burn_in_period) % averaging_frequency == 0))


```
where % indicates remainder upon division. For instance, using the default settings in the "input_parameters.jl" file shown above, the samples will be saved at iterations:

``` 
1000, 1310, 1320, 1330, 1340, 1350, 1660, 1670, 1680, 1690, 1700, ...

```
**Note: Please note that the first sample after the intial optimizer has converged typically provides a very good estimate of the deconvolved object and can be used right away for analysis (see the picture below). Further Monte Carlo samples should be collected to improve the quality of the deconvolved image even further.**

![image](https://github.com/user-attachments/assets/b52a78b9-43fb-4b8c-b372-acad660d00fa)


## A Brief Description of the Sampler

The sampler here executes a Markov Chain Monte Carlo (MCMC) algorithm (Gibbs) where samples for object intensity at each pixel are generated sequentially from their corresponding conditional probability distributions (posterior). First, the sampler creates/initiates arrays to store all the samples and joint posterior probability values. Next, new samples are then iteratively proposed using proposal (normal) distributions for each pixel, to be accepted or rejected by the Metropolis-Hastings step if direct sampling is not available. If accepted, the proposed sample is stored in the arrays otherwise the previous sample is stored at the same MCMC iteraion. 

The variance of the proposal distribution typically decides how often proposals are accepted/rejected. A larger covariance or movement away from the previous sample would lead to a larger change in likelihood/posterior values. Since the sampler prefers movement towards high probability regions, a larger movement towards low probability regions would lead to likely rejection of the sample compared to smaller movement.

The collected samples can then be used to compute statistical quantities and plot probability distributions. The plotting function used by the sampler in this code allows monitoring of posterior values and current fluorescence intensity sample.

As mentioned before, sampler prefers movement towards higher probability regions of the posterior distribution. This means that if parameters are initialized in low probability regions of the posterior, which is typically the case, the posterior would appear to increase initially for many iterations (hundreds to thousands depending on the complexity of the model). This initial period is called burn-in. After burn-in, convergence is achieved where the posterior would typically fluctuate around some mean/average value. The convergence typically indicates that the sampler has reached the maximum of the posterior distribution (the most probable region), that is, sampler generates most samples from higher probability region. In other words, given a large collection of samples, the probability density in a region of parameter space is proportional to the number of samples collected from that region. 
 
All the samples collected during burn-in are usually ignored when computing statistical properties and presenting the final posterior distribution. 
