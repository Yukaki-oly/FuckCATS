# 银行家算法

银行家算法是资源和死锁避免的算法，由艾兹格·迪杰斯特拉（Edsger Dijkstra） 设计的算法用于测已确定总数量的资源分配的安全性，在决定是否该分配应该被允许并进行下去之前，通过“s-state”校验码测试资源分配活动期间产生死锁条件的可能性。 
 该算法是为为THE操作系统设计并且最在在EWD108描述。当一个新的进程进入系统时，进程必须声明所需每个资源实例最大的数量和类型。显然，资源数量不不能超过系统最大的资源数。与此同时，进程获得的资源必须在有限的时间内释放。

 

### 资源

对于银行家算法的实现，需要知道三件事： 

- 每个进程所能获取的每种资源数量是多少[MAX] 
- 每个进程当前所分配到的每种资源的数量是多少[ALLOCATED] 
- 系统当前可分配的每种的资源数量是多少[AVAILABLE]

 

只有当资源满足以下条件，资源才会被分配：

1. request <= max, 也可设置错误条件，当进程所请求的资源超过最大的要求
2. request <= available, 或者进程一直等直到资源可分配

一些资源在实际的系统被跟踪，如：内存，信号量以及接口。

 银行家算法名字源于该算法实际上是用于确保银行系统不会用尽系统资源，因为当银行系统不再满足所有客户的需求，系统将不会分配钱（看作资源）给客户，银行必须确保对钱的请求不会导致银行系统处于不安全状态。如果上述情况不会发生，则该情况下请求是被允许的，否则，客户必须等到其他客户往银行存进足够银行分配的资金。

基本数据结构用于维护运行银行家算法： 
用n表示系统资源数量，m表示系统资源类型。则我们需要以下的数据结构: 

- Available: 长度为m的向量用来表示每种资源可分配的数量。如果available[j]=k, 资源类型为Rj可分配数量为k。 
- Max： n * m矩阵，定义，每个进程最大的资源需求。如果Max[i,j]=k. 表明Pi对类型为Rj资源的请求为k. 
- Allocation: n * m矩阵定义每个进程已分配到的每种资源的数量。如果Allocation[i,j] = k,进程Pi已分配到类型为Rj的资源数量为k。 
- Need: n * m 矩阵表明每个进程所需的资源数量，如果Need[i,j] = k， 进程Pi需要至少得到k数量的资源Rj，才能完成任务。

 

公式:Need[i,j] = Max[i,j] - Allocation[i,j]

例子：

> 系统资源总数： 
> A B C D 
> 6 5 7 6
>
> 系统可分配资源数： 
> A B C D 
> 3 1 1 2
>
> 进程 (当前分配到的资源数): 
> A B C D 
> P1 1  2  2  1 
> P2 1  0  3  3 
> P3 1  2  1  0
>
> 进程 (最大资源需求数): 
> A B C D 
> P1 3  3  2  2 
> P2 1  2  3  4 
> P3 1  3  5  0
>
> Need= Max - Allocation 
> 进程 (需要的资源数): 
> A B C D 
> P1  2 1  0  1 
> P2  0 2  0  1 
> P3  0 1  4  0

 

### 安全和不安全状态

如果该状态下所有进程都可以结束运行，则该状态是安全。因为系统无法知道什么时候一个进程结束运行，或有多少资源被进程请求，系统只是假设所有的进程最终会试图获取他们所规定的最大资源，并且在获得资源使用完之后会结束运行。在大多数的情况下，该假设是很合理的，因为系统不会特别的关心每个进程的运行多久（至少不是从死锁避免的角度）。

对于该猜想，算法确定是否一个状态是安全通过找到一个猜想性的进程请求序列，允许所有进程获取最大的资源数并顺利结束运行。而任何无法达到上诉要求的的状态都是不安全的状态。

  我们可以得到之前例子的安全状态，只要能使每个进程获得最大资源并结束运行。

1. P1 得到 2 A，1 B 和 1 D，达到进程需求的最大资源数 

- [可分配资源：<3 1 1 2> - <2 1 0 1> = <1 0 1 1>] 
- 系统当前有1 A, 0 B, 1 C, 和 1 D 资源可分配

 

2. P1 结束运行，释放3 A, 3 B, 2 C, 和2 D资源给系统。 

- [可分配资源：<1 0 1 1> + <3 3 2 2> = <4 3 3 3>] 
- 系统当前有4 A, 3 B, 3 C, 和 3 D 资源可分配

 

3. P2 结束运行，请求0 A, 2 B, 0 C, 和1 D资源，之后运行结束，释放资源给系统 

- [可分配资源：<4 3 3 3> - <0 2 0 1> + <1 2 3 4>= <5 3 6 6>] 
- 系统当前有5 A, 3 B, 6 C, 和 6 D 资源可分配

 

4. P3 请求0 A, 1 B, 4 C, 和0 D资源，之后运行结束，释放资源给系统 

- [可分配资源：<5 3 6 6> - <0 1 4 0> + <1 3 5 0>= <6 5 7 6>] 
- 系统现在用所有的资源6 A, 5 B, 7 C 和 6 D

 

5. 由于所有的进程可以结束运行，该状态是安全的状态。

 

### 请求

当系统收到对资源请求信号时，系统运行银行家算法判断允许请求是否安全。

1.该请求是否可以运行? 

如果不允许，该请求则是不可行的，必须要么拒绝请求或插入到等待队列。 

2.假设请求被允许 

3.是否安全？ 

如果安全，请求授予 

否则，要么拒绝或插入到等待队列

 

不论系统拒绝请求或请求延迟或不安全请求都是系统的特殊决定。。 

P1无法完成资源B的请求 

P2无法完成资源B的请求 

P3无法完成资源B的请求 

没有进程能获得足够的资源结束运行，故该状态是不安全的。

 

同其他的算法相似，银行家算法运行时也有一些局限。特别是，必须知道每个进程所能请求的资源。在大多数系统，该信息是不可知的，使的无法顺利运行银行家算法。而且也无法去假设进程数量是不变的，因为大多数的系统的进程数量是动态变化的。再者，对于正确的算法，进程会释放其全部的资源（当进程结束运行），然而对于实际中的系统则是不可行。为了资源的释放等上数小时甚至几天，通常是不可接受的。

