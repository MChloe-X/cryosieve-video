【cmj：这是我写完并修改完的视频脚本，计划每一个EP算一集，即有一个转场或换集，而每个二级标题则是用于方便录制记录的】
## EP1 Introduction
### Intro
【这些介绍部分没有操作是空镜头，要录视频的话，大家有没有好的建议呀？】
Cryo-electron microscopy (cryo-EM) is widely used to determine near atomic resolution structures of biological macromolecules. Due to the extremely low signal-to-noise ratio, cryo-EM relies on averaging many images.
### Problem
However, a crucial question remains unanswered: how close can we get to the minimum number of particles required to reach a specific resolution in practice.

### CryoSieve
CryoSieve, a new iterative particle sorting method, was developed to solve this problem.

Extensive experiments demonstrate that CryoSieve outperforms other cryo-EM particle sorting algorithms, showing that most particles are unnecessary in final stacks. The minority of particles remaining in the final stacks yield superior high-resolution amplitude in reconstructed density maps, sometimes even approaching the theoretical limit.

### Principle && Function
To achieve this, CryoSieve employs an iterative approach, alternating between two key steps: 3D reconstruction and particle selection. In each iteration, the algorithm systematically evaluates the particles and eliminates those that are deemed futile, then a cryo-EM single-paritcle reconstruction software selected by the user will be used to reconstruct a new density map with the retained particle images, which is then used in the subsequent iteration. 
To determine which particles should be kept and which should be screened, we developed a CryoSieve score. The CryoSieve score estimates the similarity between a particle and a reference projection above a given frequency. A higher CryoSieve score indicates that the particle and the reference projection share a higher proportion of signal energy, indicating better particle quality.

In conclusion, CryoSieve is a powerful software that offers a range of potential applications in cryo-electron microscopy. It can improve resolution and enhance amplitude in reconstructed density maps. It allows for the selection of only the best particles for computationally intensive processes such as symmetry expansion. Additionally, CryoSieve can quantitatively assess variables in sample preparation, providing valuable insights for optimizing experimental conditions. 

## EP2 Start CryoSieve
### Guide
If you are highly interested in our software and would like to start using it yourself, please continue reading as I will guide you through the installation and usage of CryoSieve. *展示所需条件截图，不做详细说明*

At the very beginning, your computer needs to meet these prerequisites:
+ Python version 3.7 or later.
+ NVIDIA CUDA library installed in the user's environment.

Dependencies
The CryoSieve package depends on the following five libraries:
+ numpy>=1.18
+ mrcfile>=1.2
+ starfile>=0.4
+ cupy>=10
+ torch>=1.10

If you have met the above conditions, let's do some preparatory work for the successful operation of our program.
### Prepare CUDA Env
First of all, preparation of CUDA Environment. 
+ If you have already installed the appropriate CUDA, along with the corresponding PyTorch and CuPy packages in your environment, you may disregard this step.
+ As an example, we utilize this approach to create a new environment that fulfills the dependency requirements of CryoSieve. 
*敲创建环境代码*
`conda create -n CRYOSIEVE_ENV python=3.10 cupy=10.2 cudatoolkit=10.2 pytorch=1.12.1=py3.10_cuda10.2_cudnn7.6.5_0 -c conda-forge -c pytorch`
And then activate it.
`source activate CRYOSIEVE_ENV`

Please note that this command is specifically for a CUDA environment version 10.2. If your CUDA env is higher than 10.2, adjust the command based on the suitable variants and versions recommended by the CuPy and PyTorch developers for your specific CUDA environment.
### Install CryoSieve
Next, we install CryoSieve by enter this command:
*视频中敲入此代码* `pip install cryosieve`

you can use either via pip or conda. If you prefer to use conda, type in this command:
*可以利用动画在右上角浮出此代码* `conda install -c mxhulab cryosieve`
### Verify
You can verify whether CryoSieve has been installed successfully by running this command:`cryosieve -h`

a successful installation should display the help information for CryoSieve.
*敲击命令cryosieve -h*
## EP3 Toy Example
So far, our preparation is complete.

To validate your successful installation of CryoSieve and familiarize yourself with its functionalities, we prepare a toy example to test the core particle sieving module--cryosieve-core. This step is not mandatory, you have the option to skip it and proceed directly to the practical application tutorial.
### Dataset Preparation
First, find the toy example on our github, download the dataset and place it into any directory of your choice. Here I put the dataset in a directory named toy.
*ls展示文件夹*

Then, we navigate to this directory
*输入 cd ~/toy/*

and initiate CryoSieve with the following command:
*输入 `cryosieve-core --i CNG.star --o my_CNG_1.star --angpix 1.32 --volume CNG_A.mrc --volume CNG_B.mrc --mask CNG_mask.mrc --retention_ratio 0.8 --frequency 40`*

The parameter "i" represents the input star file path, while "o" represents the output star file path. The "angpix" parameter corresponds to the pixel size in Angstrom, which we typically set to 1.32. The "volume" parameter denotes the list of volume file paths, also known as density maps. Additionally, the "mask" parameter refers to the file path of the mask used in the process.

One of the most crucial parameters is "retention_ratio," which represents the fraction of particles retained during the analysis. By default, it is set to 0.8, but can be adjusted as needed. Another important parameter is "frequency," which allows setting the cut-off highpass frequency.

I'd like to mention the `--num_gpus` parameter as well. When used with a value larger than 1, CryoSieve's core program utilizes multiple GPUs to accelerate the sieving process. Each of the processes will use exactly one GPU.For instance, on a machine equipped with 4 GPUs, you can include: *末尾加上` --num_gpus 4`* at the end of the command.
Here we use only one gpu is enough.*删除` --num_gpus 4`* 

Upon successful execution, the command will generate two star files;*`cd ~/toy/并ls`展示`my_CNG_1.star` `my_CNG_1_sieved.star`指向第一个文件：*This file contain the information of the remaining particles, *指向第二个文件*and this one contains the sieved particles.

If executed correctly, they should contain the same particles.
You can use this command to see if two file contain the same particle information.
*敲入 `diff my_CNG_1.star my_CNG_1_sieved.star`*
## EP4 Practical Utility

### Dataset Preparation
In this section, we provide a hands-on example of how to utilize CryoSieve for processing the final stack in a real-world experimental dataset. For this tutorial, we'll use the final particle stack from the `EMPIAR-10097` dataset.

To download the final particle stack, navigate to your desired working directory and execute the following command:
`wget -nH -m ftp://ftp.ebi.ac.uk/empiar/world_availability/10097/data/Final_Particle_Stack/`

Go to the folder where the data set is stored, this directory contains a star file with all particle information and an mrcs file representing the final stack.
*cd+ls*
Additionally, you'll need a mask file. You can generate a mask file using any cryo-EM software, based on the reconstructed volume. If you prefer not to generate a mask file, we've provided one used in our experiments which you can download from our github. Once you have the mask file, move it into the `Final_Particle_Stack` directory.

*仅进行提示，无需操作*

【作为一个tutorial，可以从下载EMPIAR开始讲起，当然用户实际使用的时候不会做这个操作的，因为他们都是处理自己的数据。】【cmj：不是说默认所有人都从EMPIAR下载数据嘛？我单独屏幕上展示一下下载命令也是可以的吧？】

### Iterative Reconstruction and Sieving

To achieve optimal results with real-world datasets, the sieving process generally involves several iterations. For your convenience, we have developed a single-run command, 'cryosieve', which automates all these steps.

Before using it, activate your CUDA environment using the following command:
*输入命令`source activate xxx`*
Please modify the environment name according to your specific configuration.

### start
To start our program, follow these steps:
Firstly, navigate to `XXX/data/Final_Particle_Stack`. I have already in this directory.
Then, make sure that  `relion_reconstruct` or `relion_reconstruct_mpi` and `relion_postprocess` are accessible since our automatic program currently uses Relion for 3D reconstruction and postprocessing. 
Once confirmed, run the following command:
*输入命令`cryosieve --reconstruct_software relion_reconstruct --postprocess_software relion_postprocess --i T40_HA_130K-Equalized_run-data_CryoSPARC_refined.star --o output/ --mask mask.mrc --angpix 1.3099979 --num_iters 10 --frequency_start 40 --frequency_end 3 --retention_ratio 0.8 --sym C3 --num_gpus 1 --balance grep "+ FINAL RESOLUTION:" output/_postprocess*.txt`*
The parameters "reconstruction_software" and "postprocess_software" specify the reconstruction and post-processing software. 
"i," "o," "mask," "angpix," and "retention_ratio" are used in the same way as the parameters in the CryoSieve-Core command. They are used to specify the input and output file paths, mask file path, pixel size in Angstrom, and fraction of retained particles in each iteration (default value: 0.8). 
"frequency_start" and "frequency_end" set the starting and ending threshold frequencies in Angstrom units. The default values are 50A and 3A, respectively. 
"sys" represents the molecular symmetry, with a default value of "c1".

## EP5 Result Analysis
The entire process may take over an hour, depending on your system resources. Multiple result files will be generated and saved in the `output/` directory. The reconstructed density maps from the nth iteration are saved as `iter{n}_half.mrc` files. The reconstruction log is stored in `reconstruct_half.txt`. Filtered particles are stored in `sieved.star` along with the corresponding log in a `.txt` file. Retained particles are stored in `.star` files. If post-processing is used, additional result files with the prefix `postprocess_iter` followed by the iteration number are generated.
`_iter{n}_half1.mrc`
`_iter{n}_half2.mrc`
`_iter{n}_reconstruct_half1.txt`
`_iter{n}_reconstruct_half2.txt`
`_iter{n}_sieved.star`
`_iter{n}_sieve.txt`
`_iter{n}.star`

【上述文件分别是：第n轮重构的密度图1、密度图2、日志1、日志2，筛选掉的颗粒、筛选的日志、保留的颗粒。但是这些文件应该还不全，在使用postprocess的时候还有postprocess产生的文件。】

## EP6, re-esimate poses via CryoSPARC

*敲入`grep "RESOLUTION" *.txt`*
After several times sieving, we have to find out the iterations that best meet our demand. We address this issue by re-estimating poses via cryoSPARC.

The first step is to pick out some images stacks output by CryoSieve that might be suitable, and import them to cryoSPARC. Once the stacks are imported, we perform ab initio modeling. This step generates an initial 3D model of the macromolecule based solely on the experimental density maps. After the ab initio model is generated, we begin the refinement process, also known as single-particle reconstruction. Instead of re-do GS split, we keep the gold-standard criteria. Finally, we will get the results. The iteration with lesser particles, yet exhibiting better resolution and contrast, will be selected and referred to as the "finest subset".

We begin with importing stacks output by CryoSieve  that might suitable to cryoSPARC.

## Ending

In this video, we introduced the CryoSieve algorithm and show you how to apply the software in practical usage. 

That concludes the complete introduction and tutorial on CryoSieve. If you would like to learn more about CryoSieve, we recommend visiting our GitHub page and referring to our research paper. Thank you for your support.

【Good job!】
