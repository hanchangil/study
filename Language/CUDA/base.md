# CUDA*(Compute Unified Device Architecture )*

* CUDA는 그래픽 처리 장치(GPU)에서 수행하는 (병렬처리) 알고리즘을 C 프로그래밍 언어를 비롯한 산업 표준 언어를 사용하여 작성할 수 있도록 하는 GPGPU 기술이다.

* CUDA는 엔비디아가 개발해오고 있으며 이 아키텍처를 사용하려면 엔비디아 GPU와 특별한 스트림 처리 드라이버가 필요하다.

* CUDA는 G8X GPU로 구성된 지포스 8 시리즈급 이상에서 동작한다.

* CUDA 플랫폼은 컴퓨터 커널의 실행을 위해 GPU의 가상 명령 집합과 병렬 연산 요소들을 직접 접근할 수 있는 소프트웨어 계층이다.[1]

* GPU에서 생성한 메모리도 CPU에서 접근 가능

* 쿠다에서는 기본적으로 변수를 지원해준다!

- - -

### GPU용 응용프로그램 코드

##### 1. __global__ void GPUFunction()

 * 이 __global__키워드는 다음 함수가 GPU에서 실행되며 전역 적으로 호출 될 수 있음을 나타냅니다 . 여기서는 CPU 또는 GPU를 의미합니다. 
   종종 CPU에서 실행되는 코드를 호스트 코드라고하며 GPU에서 실행되는 코드를 장치 코드 라고합니다 .


##### 2. GPUFunction<<<1, 1>>>();

* 일반적으로 GPU에서 실행할 함수를 호출 할 때이 함수를 커널 이라고 부릅니다.이 함수 는 시작 됩니다.
* 커널을 시작할 때 커널에 예상 인수를 전달하기 직전 의 구문 을 사용하여 실행 구성을 제공해야합니다 <<< ... >>>.
* 높은 수준에서 실행 구성을 사용하면 프로그래머가 커널 실행을위한 스레드 계층 구조 를 지정할 수 있습니다.
* 이 계층 구조 는 스레드 그룹화 ( 블록 이라고 함 )의 수와 각 블록에서 실행할 스레드 수를 정의합니다 .

#### 3. cudaDeviceSynchronize();

* 많은 C / C ++ 코드와 달리, 시작 커널은 비동기식입니다 . CPU 코드는 커널 시작이 완료 될 때까지 기다리지 않고 계속 실행 됩니다 .
* cudaDeviceSynchronizeCUDA 런타임에서 제공하는 함수를 호출 하면 호스트 (CPU) 코드가 장치 (GPU) 코드가 완료 될 때까지 대기 한 다음 CPU에서만 실행을 다시 시작합니다.

##### 커널설정 예제
~~~
#include <stdio.h>

void helloCPU()
{
  printf("Hello from the CPU.\n");
}

/*
 * The addition of `__global__` signifies that this function
 * should be launced on the GPU.
 */

__global__ void helloGPU()
{
  printf("Hello from the GPU.\n");
}

int main()
{
  helloCPU();

- - -
  /*
   * Add an execution configuration with the <<<...>>> syntax
   * will launch this function as a kernel on the GPU.
   */

  helloGPU<<<1, 1>>>();

  /*
   * `cudaDeviceSynchronize` will block the CPU stream until
   * all GPU kernels have completed.
   */

  cudaDeviceSynchronize();
}
~~~
- - -

###  병렬 커널 시작하기 

* 실행 구성을 통해 프로그래머는 여러 GPU 스레드 에서 병렬로 실행되도록 커널을 실행하는 것에
  대한 세부 사항을 지정할 수 있습니다 . 
* 스레드 블록 아니면 그냥 블록 - 얼마나 많은 스레드가 각 스레드 블록이 포함하고자합니다. 
 이 구문은 다음과 같습니다.
   ?    <<< NUMBER_OF_BLOCKS, NUMBER_OF_THREADS_PER_BLOCK>>>
   ?    
##### 병렬커널설정 예제
~~~
#include <stdio.h>

__global__ void firstParallel()
{
  printf("This is running in parallel.\n");
}

int main()
{
  firstParallel<<<5, 5>>>();
  cudaDeviceSynchronize();
}
~~~
- - -
###  쓰레드와 블록 인덱스
* 각 스레드와 블록에서 시작하는 해당 색인은 0입니다.
* 스레드가 스레드 블록으로 그룹화되는 것처럼 블록은 CUDA 스레드 계층에서 가장 높은 엔티티 인 그리드 로 그룹화됩니다 . 
* 요약하면, CUDA 커널은 1 개 이상의 블록 그리드에서 실행되며, 각 블록은 동일한 수의 1 개 이상     의 스레드를 포함합니다.
* CUDA 커널은 커널을 실행중인 스레드의 색인 (인덱스)과 스레드가있는 블록의 인덱스 (그리드 내)를 모두 식별하는 특수 변수에 액세스 할 수 있습니다.
* 변수는 다음과 같습니다. threadIdx.x  blockIdx.x

##### 특정쓰레드 및 블록인덱스 사용 예제

~~~
#include <stdio.h>

__global__ void printSuccessForCorrectExecutionConfiguration()
{

  if(threadIdx.x == 1023 && blockIdx.x == 255)
  {
    printf("Success!\n");
  }
}

int main()
{
  /*
   * This is one possible execution context that will make
   * the kernel launch print its success message.
   */

  printSuccessForCorrectExecutionConfiguration<<<256, 1024>>>();

  /*
   * Don't forget kernel execution is asynchronous and you must
   * sync on its completion.
   */

  cudaDeviceSynchronize();
}
~~~

---

### FOR 루프 가속

* CPU 전용 응용 프로그램의 경우 루프의 각 반복을 순차적으로 실행하는 대신 루프의 각 반복을 자체 스레드에서 병렬로 실행할 수 있습니다.

~~~
int N = 2<<20;
for (int i = 0; i < N; ++i)
{
  printf("%d\n", i);
}
~~~

* 이 루프를 병렬 처리하려면 2 단계를 수행해야합니다.
  커널은 루프 의 단일 반복 작업을 수행하도록 작성되어야합니다 .
  커널은 다른 실행 커널에 대해 불가지론 할 것이므로 실행 구성은 커널이 올바른 횟수 (예 : 반복 횟수)를 실행하도록 설정해야합니다.

##### FOR 루프 가속 예제(단일쓰레드)

~~~
#include <stdio.h>


/*
 * Notice the absence of the previously expected argument `N`.
 */

__global__ void loop()
{
  /*
   * This kernel does the work of only 1 iteration
   * of the original for loop. Indication of which
   * "iteration" is being executed by this kernel is
   * still available via `threadIdx.x`.
   */

  printf("This is iteration number %d\n", threadIdx.x);
}

int main()
{
  /*
   * It is the execution context that sets how many "iterations"
   * of the "loop" will be done.
   */

  loop<<<1, 10>>>();
  cudaDeviceSynchronize();
}
~~~

##### FOR 루프 가속 예제(병렬쓰레드)

~~~
#include <stdio.h>

__global__ void loop()
{
  /*
   * This idiomatic expression gives each thread
   * a unique index within the entire grid.
   */

  int i = blockIdx.x * blockDim.x + threadIdx.x;
  printf("%d\n", i);
}

int main()
{
  /*
   * Additional execution configurations that would
   * work and meet the exercises contraints are:
   *
   * <<<5, 2>>>
   * <<<10, 1>>>
   */

  loop<<<2, 5>>>();
  cudaDeviceSynchronize();
}
~~~

---

### 병렬처리 블록 크기 사용하기

* 스레드 블록에 존재할 수있는 스레드의 수에는 제한이 있습니다. 정확하게 말하면 1024입니다.
  가속화 된 애플리케이션에서 병렬 처리량을 늘리려면 여러 스레드 블록을 조정할 수 있어야합니다.

* 실행 구성 <<<10, 10>>>은 10 개의 스레드로 구성된 10 개의 블록에 포함 된 총 100 개의 스레드로 그리드를 시작합니다. 따라서 우리는 각 스레드가 자신 0과 자신 사이에 고유 한 인덱스를 계산할 수 있기를 바랍니다 .

* 데이터인덱스계산 : threadIdx.x + blockIdx.x * blockDim.x



---

### GPU 및 CPU에서 액세스 할 메모리 할당

* 최신 버전의 CUDA (버전 6 이상)를 사용하면 CPU 호스트와 모든 GPU 장치에서 사용할 수있는 메모리를 쉽게 할당 할 수 있다
  GPU에 할당 된 메모리도 CPU에서 접근 할 수 있다.

##### 메모리할당 예제

~~~
// CPU-only

int N = 2<<20;
size_t size = N * sizeof(int);

int *a;
a = (int *)malloc(size);

// Use `a` in CPU-only program.

free(a);
// Accelerated

int N = 2<<20;
size_t size = N * sizeof(int);

int *a;
// Note the address of `a` is passed as first argument.
cudaMallocManaged(&a, size);

// Use `a` on the CPU and/or on any GPU in the accelerated system.

cudaFree(a);
~~~

---

### 그리드보다 더 큰 데이터 처리

* 그리드보다 큰 데이터 세트 
  선택에 따라, 종종 가장 실행 가능한 실행 구성을 만들거나 필요에 따라 그리드의 스레드 수가 데이터 세트의 크기보다 작을 수 있습니다. 1000 개의 요소가있는 배열과 250 개의 스레드가있는 격자를 생각해보십시오 (설명의 편의를 위해 여기에 사소한 크기를 사용함). 여기에서 그리드의 각 스레드는 4 번 사용해야합니다. 이렇게하는 일반적인 방법 중 하나 는 커널 내에서 그리드 스트라이드 루프 를 사용하는 것 입니다.

##### 그리드<데이터 예제

~~~
__global void kernel(int *a, int N)
{
  int indexWithinTheGrid = threadIdx.x + blockIdx.x * blockDim.x;
  int gridStride = gridDim.x * blockDim.x;

  for (int i = indexWithinTheGrid; i < N; i += gridStride)
  {
    // do work on a[i];
  }
}
~~~

---

### 오류처리

* 모든 애플리케이션에서와 마찬가지로 가속화 된 CUDA 코드에서의 오류 처리가 필수적입니다.
  대부분의 CUDA 함수 (예 : 메모리 관리 함수 참조 ) cudaError_t는 함수 호출 중 오류가 발생했는지 여부를 확인하는 데 사용할 수있는 type 값을 반환 합니다.
  다음에 대한 호출에 대해 오류 처리가 수행되는 예제가 있습니다 cudaMallocManaged.

##### 오류처리 예제

~~~
cudaError_t err;
err = cudaMallocManaged(&a, N)                    // Assume the existence of `a` and `N`.

if (err != cudaSuccess)                           // `cudaSuccess` is provided by CUDA.
{
  printf("Error: %s\n", cudaGetErrorString(err)); // `cudaGetErrorString` is provided by CUDA.
}
리턴하기 위해 정의 된 시작 커널 void은 유형의 값을 리턴하지 않습니다 cudaError_t. 예를 들어 실행 구성에 오류가있는 경우와 같이 커널을 시작할 때 발생하는 오류를 확인하기 위해 CUDA는 cudaGetLastErrortype 값을 반환하는 함수를 제공합니다 cudaError_t.

/*
 * This launch should cause an error, but the kernel itself
 * cannot return it.
 */

someKernel<<<1, -1>>>();  // -1 is not a valid number of threads.

cudaError_t err;
err = cudaGetLastError(); // `cudaGetLastError` will return the error from above.
if (err != cudaSuccess)
{
  printf("Error: %s\n", cudaGetErrorString(err));
}
~~~

* 마지막으로, 예를 들어 비동기 커널을 실행하는 동안 비동기 적으로 발생하는 오류를 잡으려면 다음과 같은 후속 동기화 cuda 런타임 API 호출에 의해 반환 된 상태를 확인해야합니다.
  예를 들어 cudaDeviceSynchronize, 다음 중 하나 인 경우 오류를 반환합니다. 이전에 실행 된 커널은 실패합니다.








