---
title: "Submitting a kernel to the CUDA kernel competition"
author: Gabriel Castro
---

I didn't find myself repeating the same instructions to many people over discord, but nonetheless, I’ve decided to compile a brief tutorial on how to submit a kernel to the CUDA kernel competiton using the AI Safety discord bot in one place.

### Commands
Use the following commands to interact with the bot:
- `/submit` - Submit a kernel to the competition
- `/show` -  Show the leaderboard for a challenge
- `/submissions` - Get a list of all your personal submissions

### How to submit a kernel
![submit](/assets/submit_img_v2.png)

Note that the bot will accept `.cu` and supports `ptx` files as well. You will also need to make sure `name` matches the name of you `.cu` file. 
\
\
The `local_size` refers to blockDim (threadblock dimensions) and `global_size` refers to gridDim (grid dimensions). Also, if you only want a 2D threadblock and gridDim you have to input a tuple i.e. `(16, 16, 1)` for `local_size` and `global_size`. (The 1 at the end is just used as a placeholder for the third dimension).
\
\
Additionally, if you designed your dims to be like (M/64, N/64), then for the submission you would actually need to calculate your size. So given that the challenge has fixed dimensions M=N=4096, then you would need to input (64, 64, 1) for `global_size`.
\
\
There are also two other optional parameters you can add to your submission: `transpose_a` and `transpose_b`. These are booleans that indicate whether you want to transpose the input matrices A and B respectively. The default value for both is `false`, so if you want to transpose them, you need to set them to `true`.

### Reference Matmul Kernel
```
#define m 4096
#define n 4096
#define d 4096


extern "C" __global__ void matmul(float* A, float* B, float* C){
    // A is (m,d), B is (d,n), C is (m,n)

    int i = blockIdx.x*blockDim.x + threadIdx.x;
    int j = blockIdx.y*blockDim.y + threadIdx.y;

    if(i < m && j < n){
        float inner_prod = 0.0f;
        for(int k = 0; k < d; k++){
            inner_prod += A[i*d+k] * B[k*n+j];
        }
        C[i*n+j] = inner_prod;
    }
}
```


#### Important caveats to take note of when submitting your kernel:
- The kernel name must match the name of the file i.e. `matmul.cu` must have a kernel named `matmul`
- You must use the `extern "C"` keyword in your kernel definition
- The matmul challenge will only expect the standard 3 matrix arguments to be passed to the kernel, so you must define your kernel with the 3 parameters (A, B, C)
- Make sure you define any dimensions you need as constants at the top of the file (since for the matmul challenge, the dimensions are fixed)

### Online leaderboard and Prizes
- To be verified on the online leaderboard you must run this executable on ynez `~kernelbot/scratch/register [your discord id]`
- You can find your discord id by going to your discord settings, then to "Advanced" and enabling "Developer Mode". After that, you can right click on your profile and select "Copy ID"
- Only verified users will be displayed on the online leaderboard and be eligible for prizes

##### s/o to chris, the AIS supreme leader, for creating this amazing bot! ^_^


