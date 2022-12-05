---
linktitle: CUDA事件计时
summary: Summary of CUDA事件计时
type: book
weight: 3
---
# CUDA事件计时
## 1.快速上手
```c
#include <math.h>
#include <stdio.h>
const int NUM_REPEATS = 10;
const float a = 1.23, b = 2.34, c = 3.57;
void __global__ add(const float *x, const float *y, float *z, const int N){
    const int n = blockDim.x * blockIdx.x + threadIdx.x;
    if (n < N) z[n] = x[n] + y[n];
}
int main(void)
{
    const int N = 100000000;

    const int M = sizeof(float) * N; //字节

    float *h_x = (float*) malloc(M);

    float *h_y = (float*) malloc(M);

    float *h_z = (float*) malloc(M);

  

    for (int n = 0; n < N; ++n){

        h_x[n] = a;

        h_y[n] = b;

    }

  

    float *d_x, *d_y, *d_z;

    cudaMalloc((void **)&d_x, M);

    cudaMalloc((void **)&d_y, M);

    cudaMalloc((void **)&d_z, M);

    cudaMemcpy(d_x, h_x, M, cudaMemcpyHostToDevice);

    cudaMemcpy(d_y, h_y, M, cudaMemcpyHostToDevice);

  

    const int block_size = 128;

    const int grid_size = (N + block_size - 1) / block_size;

  

    float t_sum = 0;

    for (int repeat = 0; repeat <= NUM_REPEATS; ++repeat){

        cudaEvent_t start, stop;

        cudaEventCreate(&start);

        cudaEventCreate(&stop);

        cudaEventRecord(start);

        cudaEventQuery(start);

  

        add<<<grid_size, block_size>>>(d_x, d_y, d_z, N);

  

        cudaEventRecord(stop);

        cudaEventSynchronize(stop);

        float elapsed_time;

        cudaEventElapsedTime(&elapsed_time, start, stop);

        printf("Time = %g ms.\n", elapsed_time);

  

        if (repeat > 0) t_sum += elapsed_time;

        cudaEventDestroy(start);

        cudaEventDestroy(stop);

    }

    const float t_ave = t_sum / NUM_REPEATS;

    printf("Average Time = %g ms.\n", t_ave);

  

    free(h_x);free(h_y);free(h_z);

    return 0;

}
```
更完整的代码：
https://gitee.com/wangyiou/cuda-programming/blob/master/src/05-prerequisites-for-speedup/add1cpu.cu
## 2.原理
参考：https://blog.csdn.net/qq_24990189/article/details/89602618
**cudaEventCreate**：创建事件
**cudaEventRecord**：视为一条记录当前时间的语句，并且把这条语句放入GPU的未完成队列中。
**cudaEventSynchronize**：同步等待某事件执行完毕
**cudaEventElapsedTime**：获取两个事件之间的时间差
**cudaEventDestroy**：销毁事件

相当于是在GPU执行的过程中，在GPU里进行时间记录。在CPU端，由于调用kernel是异步的，所以会在GPU执行结束之前返回，所以计时的时候，GPU实际还没执行完，计时也就不准。

问题：
1）CPU计时+同步等待GPU执行结束，是否能达到同样效果？
2）CPU计时的延时问题：OS线程调度导致计时不准确？