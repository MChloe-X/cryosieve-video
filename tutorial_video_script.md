��cmj��������д�겢�޸������Ƶ�ű����ƻ�ÿһ��EP��һ��������һ��ת���򻻼�����ÿ�����������������ڷ���¼�Ƽ�¼�ġ�
## EP1 Introduction
### Intro
����Щ���ܲ���û�в����ǿվ�ͷ��Ҫ¼��Ƶ�Ļ��������û�кõĽ���ѽ����
Cryo-electron microscopy (cryo-EM) is widely used to determine near atomic resolution structures of biological macromolecules. Due to the extremely low signal-to-noise ratio, cryo-EM relies on averaging many images.
### Problem
However, a crucial question remains unanswered: how close can we get to the minimum number of particles required to reach a specific resolution in practice, which has impeded progress in understanding sample behavior and the performance of sample preparation methods.
### CryoSieve
CryoSieve, a new iterative particle sorting and/or sieving method, was developed to solve this problem.

Extensive experiments demonstrate that CryoSieve outperforms other cryo-EM particle sorting algorithms, showing that most particles are unnecessary in final stacks. The minority of particles remaining in the final stacks yield superior high-resolution amplitude in reconstructed density maps, sometimes even approaching the theoretical limit.

��CryoSieve��һ����final stack�еĿ�����������ɸѡ���㷨�������ڱ��ֱַ��ʵ�������ɸ��һ���Ŀ�������[�Ƕ����е�final stack������������]
### Principle && Function
To achieve this, CryoSieve employs an iterative approach, alternating between two key steps: 3D reconstruction and particle selection. In each iteration, the algorithm systematically evaluates the particles and eliminates those that are deemed futile, then a cryo-EM single-paritcle reconstruction software selected by the user will be used to reconstruct a new density map with the retained particle images, which is then used in the subsequent iteration. 
To determine which particles should be kept and which should be screened, we developed a CryoSieve score. The CryoSieve score estimates the similarity between a particle and a reference projection above a given frequency. A higher CryoSieve score indicates that the particle and the reference projection share a higher proportion of signal energy, indicating better particle quality.

1. In conclusion, CryoSieve is a powerful software that has the potential to establish a metric for the quantitative evaluation of various sample preparation techniques. By providing a robust framework, CryoSieve empowers researchers to optimize sample preparation methods and drive advancements in cryo-EM.
2. In conclusion, CryoSieve is a powerful software that offers a range of potential applications in cryo-electron microscopy. (improve resolution, enhance amplitude; pick only best particles for subsequent compution-heavy process like symmetry-expension; quantitively jedge variables in sample preparation)

��cmj����һ������1��2����һ����2����Ҫ��ppt�ϵ������ν�������
## EP2 Start CryoSieve
### Guide
If you are highly interested in our software and would like to start using it yourself, please continue reading as I will guide you through the installation and usage of CryoSieve. *չʾ����������ͼ��������ϸ˵��*

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
*�ô�����������*
`conda create -n CRYOSIEVE_ENV python=3.10 cupy=10.2 cudatoolkit=10.2 pytorch=1.12.1=py3.10_cuda10.2_cudnn7.6.5_0 -c conda-forge -c pytorch`
And then activate it.
`source activate CRYOSIEVE_ENV`

Please note that this command is specifically for a CUDA environment version 10.2. If your CUDA env is higher than 10.2, adjust the command based on the suitable variants and versions recommended by the CuPy and PyTorch developers for your specific CUDA environment.
### Install CryoSieve
Next, we install CryoSieve by enter this command:
*��Ƶ������˴���* `pip install cryosieve`

you can use either via pip or conda. If you prefer to use conda, type in this command:
*�������ö��������ϽǸ����˴���* `conda install -c mxhulab cryosieve`
### Verify
You can verify whether CryoSieve has been installed successfully by running this command:`cryosieve -h`

a successful installation should display the help information for CryoSieve.
*�û�����cryosieve -h*
## EP3 Toy Example
So far, our preparation is complete.

To validate your successful installation of CryoSieve and familiarize yourself with its functionalities, we prepare a toy example to test the core particle sieving module--cryosieve-core. This step is not mandatory, you have the option to skip it and proceed directly to the practical application tutorial.
### Dataset Preparation
First, find the toy example on our github, download the dataset and place it into any directory of your choice. Here I put the dataset in a directory named toy.
*lsչʾ�ļ���*

Then, we navigate to this directory
*���� cd ~/toy/*

and initiate CryoSieve with the following command:
*���� `cryosieve-core --i CNG.star --o my_CNG_1.star --angpix 1.32 --volume CNG_A.mrc --volume CNG_B.mrc --mask CNG_mask.mrc --retention_ratio 0.8 --frequency 40`*

The parameter "i" represents the input star file path, while "o" represents the output star file path. The "angpix" parameter corresponds to the pixel size in Angstrom, which we typically set to 1.32. The "volume" parameter denotes the list of volume file paths, also known as density maps. Additionally, the "mask" parameter refers to the file path of the mask used in the process.

One of the most crucial parameters is "retention_ratio," which represents the fraction of particles retained during the analysis. By default, it is set to 0.8, but can be adjusted as needed. Another important parameter is "frequency," which allows setting the cut-off highpass frequency.

I'd like to mention the `--num_gpus` parameter as well. When used with a value larger than 1, CryoSieve's core program utilizes multiple GPUs to accelerate the sieving process. Each of the processes will use exactly one GPU.For instance, on a machine equipped with 4 GPUs, you can include: *ĩβ����` --num_gpus 4`* at the end of the command.
��cmj�������num_gpus��ѡ���˱������ܣ���Ϊ����������û�г������������ʵ��Ӧ���п��ܻᱻ���ԡ�
Here we use only one gpu is enough.*ɾ��` --num_gpus 4`* 

Upon successful execution, the command will generate two star files;*`cd ~/toy/��ls`չʾ`my_CNG_1.star` `my_CNG_1_sieved.star`ָ���һ���ļ���*This file contain the information of the remaining particles, *ָ��ڶ����ļ�*and this one contains the sieved particles.

If executed correctly, they should contain the same particles.
You can use this command to see if two file contain the same particle information.
*���� `diff my_CNG_1.star my_CNG_1_sieved.star`*
## EP4 Practical Utility
### Dataset Preparation
In this section, we provide a hands-on example of how to utilize CryoSieve for processing the final stack in a real-world experimental dataset. For this tutorial, we'll use the final particle stack from the `EMPIAR-11233` dataset. 

To download the final particle stack, navigate to your desired working directory and execute the following command:
`wget -nH -m ftp://ftp.ebi.ac.uk/empiar/world_availability/11233/data/Final_Particle_Stack/`

Go to the folder where the data set is stored, this directory contains a star file with all particle information and an mrcs file representing the final stack.
*cd+ls*
Additionally, you'll need a mask file. You can generate a mask file using any cryo-EM software, based on the reconstructed volume. If you prefer not to generate a mask file, we've provided one used in our experiments which you can download from our github. Once you have the mask file, move it into the `Final_Particle_Stack` directory.����һ���б�Ҫ�𣿡�

*��������ʾ���������*

����Ϊһ��tutorial�����Դ�����EMPIAR��ʼ���𣬵�Ȼ�û�ʵ��ʹ�õ�ʱ�򲻻�����������ģ���Ϊ���Ƕ��Ǵ����Լ������ݡ�����cmj������˵Ĭ�������˶���EMPIAR����������ҵ�����Ļ��չʾһ����������Ҳ�ǿ��Եİɣ���

### Iterative Reconstruction and Sieving

To achieve optimal results with real-world datasets, the sieving process generally involves several iterations. For your convenience, we have developed a single-run command, 'cryosieve', which automates all these steps.

Before using it, activate your CUDA environment using the following command:
*��������`source activate xxx`*
Please modify the environment name according to your specific configuration.

### start
To start our program, follow these steps:
Firstly, navigate to `XXX/data/Final_Particle_Stack`. I have already in this directory.
Then, make sure that  `relion_reconstruct` or `relion_reconstruct_mpi` and `relion_postprocess` are accessible since our automatic program currently uses Relion for 3D reconstruction and postprocessing. 
Once confirmed, run the following command:
*��������`cryosieve --reconstruct_software relion_reconstruct --postprocess_software relion_postprocess --i diver2019_pmTRPM8_calcium_Krios_6Feb18_finalParticleStack_EMPIAR_composite.star --o output/ --mask mask.mrc --angpix 1.059 --num_iters 10 --frequency_start 40 --frequency_end 3 --retention_ratio 0.8 --sym C4`*
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

�������ļ��ֱ��ǣ���n���ع����ܶ�ͼ1���ܶ�ͼ2����־1����־2��ɸѡ���Ŀ�����ɸѡ����־�������Ŀ�����������Щ�ļ�Ӧ�û���ȫ����ʹ��postprocess��ʱ����postprocess�������ļ�����

## Ending

In this video, we introduced the CryoSieve algorithm and show you how to apply the software in practical usage. 

That concludes the complete introduction and tutorial on CryoSieve. If you would like to learn more about CryoSieve, we recommend visiting our GitHub page and referring to our research paper. Thank you for your support.

��Good job!��
