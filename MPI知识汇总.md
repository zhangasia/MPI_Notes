一、MPI 知识点

##### 1.MPI是什么
$\qquad$MPI是一个跨平台的通信协议，用于编写并行计算机，支持点对点和广播。MPI是一个信息传递应用程序接口，包括协议和语义说明，他们指明其如何在各种实现中发挥其特性。MPI的目标是高性能，大规模性和可移植性。MPI在今天仍为高性能计算的主要模型。
##### 2.MPI 基本框架函数
* MPI_Init
调用MPI_Init 是为了告知系统进行所必要的初始化设置。参数argc_p个argv_p是指向参数argc和argv的指针。当不需要的时候设置为NULL。
```c
int MPI_Init(
	int * argc_p /*in/out*/,
    char ** argv_p /*in/out*/
);
```
* MPI_Finalize
调用MPI_Finalize是为了告知MPI系统MPI已经使用完毕。为MPI分配的任何资源都可以释放了。
* MPI程序的基本框架
```c
#include <mpi.h>
int main(int argc, char * argv[]) {
    MPI_Init(&argc, &argv);
    ...
    MPI_Finalize();
    return 0;
}
```
##### 3.MPI通信
* 通信子
通信子(communicator)指的是一组可以互相发送信息的进程集合。MPI_Init的其中一个目的，是在用户启动程序时，定义有用户启动的所有进程组成的通信子。称为MPI_COMM_WORLD。
```c
int MPI_Comm_size(
	MPI_Comm comm ,
    int * comm_sz_p
);
int MPI_Comm_rank(
	MPI_Comm comm,
    int * my_rank_p
);
```
$\qquad$第一个参数是一个通信子，它所属的类型是MPI的通信子定义的特殊类型:MPI_Comm.MPI_Comm_size 函数在它的第二个参数返回通信子的进程数。MPI_Comm_rank函数在它的第二个参数返回正在调用进程的通信子中的进程号。
* MPI_Send
```c
int MPI_Send(
	void * 	msg_buf_p,
    int		msg_size,
    MPI_Datatype 	msg_type,
    int 	dest,
    int 	tag,
    MPI_Comm	commmunicator
);
```
第一个参数：msg_buf_p是一个指向消息内容的内存块的指针。
第二个参数：msg_size是指定了要发送的数据量。
第三个参数：msg_type 是指数据类型，MPI数据类型如下表所示。

MPI数据类型 | c语言数据类型
- | -
MPI_CHAR|signed char
MPI_SHORT|signed short int
MPI_INT|signed int
MPI_LONG|signed long int
MPI_LONG_LONG|signed long long int
MPI_UNSIGNED_CHAR|unsigned short int
MPI_UNSIGNED|unsigned int
MPI_UNSIGNED_LONG|unsigned long int
MPI_FLOAT|float
MPI_DOUBLE|double
MPI_LONG_DOUBLE|long double
MPI_BYTE|
MPI_PACKED|
第四个参数：dest指定了要接收消息的进程的进程号。
第五个参数：（MPI_ANY_TAG为-1）tag是个非负int型，用于区分看上去完全一样的消息。
最后一个参数：是一个通信子，用于指定通信范围。通信子指的是一组互相发送消息的进程的集合。一个通信子的进程所发送的消息不能被另一个通信子的进程所接收。
* MPI_Recv
```c
int MPI_Recv(
    void * msg_buf_p,
    int 	buf_size,
    MPI_Datatype	buf_type,
    int 	source,
    int 	tag, 
    MPI_Comm	communicator,
    MPI_Status*	status_p
);
```
参数source用来指定了接收消息应该从哪个进程发送来的，参数tag要与发送消息的参数tag相匹配。参数communicator必须与发送进程所用的通信子匹配。
MPI类型MPI_Status是一个有至少三个成员的结构，MPI_SOURCE,MPI_TAG和MPI_ERROR。将&status作为最后一个参数传递给MPI_Recv函数并调用它后，可以通过检查以下两个成员来确定发送者和标签。
* 消息匹配
假定q号进程调用MPI_Send()函数
```c
MPI_Send(send_buf_p, send_buf_sz, send_type, dest, send_tag, send_comm)
```
假定r号进程调用了MPI_Recv()函数
```c
MPI_Recv(recv_buf_p, recv_buf_sz, recv_type, src, recv_tag, recv_comm,
```
则q号进程调用MPI_Send函数所发送的消息可以被r号进程调用MPI_Recv函数接收，如果
```
(1)recv_comm = send_comm
(2)recv_tag = send_tag
(3)dest=r & src =q
```
在多数的情况下，满足下面的规则就可以了:
```
如果recv_type = send_type 同时recv_buf_sz ≥≥ send_buf_sz 
```
那么由q号进程发送的消息就可以被r号进程成功的接收。
$\qquad$一个进程可以接收多个进程发送的消息，接收进程并不知道其他进程执行发送消息的顺序。MPI提供了一个特殊的常量MPI_ANY_SOURCE，就可以传递给MPI_Recv。
$\qquad$类似的，一个进程有可能接收多条来自另一个进程的有着不同的标签的消息，并且接收进程不知道消息发送的顺序。使用通配符（wildcard）参数时，需要注意的几点：

    只要接收者可以调用通配符参数，发送者必须指定一个进程号与另一个非负整数标签。此外，MPI使用的是所谓的推（push）通信机制，而不是拉（pull）通信机制。
    通信子参数没有通配符，发送者和接收者必须指定通信子。
* MPI_Send和MPI_Recv
$\qquad$发送进程可以缓冲消息，也可以阻塞。
$\qquad$如果是缓冲消息，则MPI系统将会把消息放置在自己内部存储器里，并返回MPI_Send的调用。如果是系统发生阻塞，那么它将一直等待，知道可以开始发送消息，并不立即返回MPI_Send的调用。
$\qquad$MPI_Send 的精确行为可以由MPI实现所决定的，但是，典型的实现方法有一个默认的消息截止大小，如果一条消息的大小小于截止大小，就会被缓冲，如果大于截止大小，就会被阻塞。
$\qquad$MPI_Recv函数总是阻塞是，直到接收到一条匹配消息，当MPI_Recv函数调用返回时，就知道一条消息已经存储在接收缓冲区中，接收消息函数同样可以替代，系统检查是否有一条匹配的消息并返回。
$\qquad$MPI要求消息是不可超越的（nonvertaking)，如果q号进程发送两条消息给r号进程，那么q号发送的第一条消息必须在第二条消息之前可用，但是如果消息来自不同的进程的消息的到达顺序是没有限制的。
* MPI_Sendrecv
利用mpi求解微分方程时，经常会遇到不同进程的通讯，特别是如下形式的通讯：

　　　　进程0->进程1->进程2->进程3...->进程n->进程0

$\qquad$这时，若单纯的利用MPI_Send, MPI_Recv函数进行通讯的话，容易造成死锁，下面介绍MPI_Sendrecv的来解决这个问题。顾名思义，MPI_Sendrecv表示的作用是将本进程的信息发送出去，并接收其他进程的信息，其调用方式如下：
```c
MPI_Sendrecv( 
    void *sendbuf, //initial address of send buffer
    int sendcount, //number of entries to send
    MPI_Datatype sendtype, //type of entries in send buffer
    int dest, //rank of destination
    int sendtag, //send tag,99
    void *recvbuf, //initial address of receive buffer
    int recvcount, //max number of entries to receive
    MPI_Datatype recvtype, 
    //type of entries in receive buffer  (这里数目是按实数的数目，若数据类型为     MPI_COMPLEX时，传递的数目要乘以2) 　　　　　　　 
    int source, //rank of source 　　　　　　 
    int recvtag, //receive tag 　　　　　　 
    MPI_Comm comm, //group communicator　　　　　　
    MPI_Status status //return status;
)
```
例子：
```c
#include "mpi.h"
#include <stdio.h>
 
int main(int argc, char *argv[])
{
    int myid, num, left, right;
    int sendBuffer[10], getBuffer[10];
    MPI_Status status;
    MPI_Init(&argc,&argv);
    MPI_Comm_size(MPI_COMM_WORLD, &num);
    MPI_Comm_rank(MPI_COMM_WORLD, &myid);
    right = (myid + 1) % num;
    left = myid - 1;
    if (left < 0)
        left = numprocs - 1;
    MPI_Sendrecv(sendBuffer, 10, MPI_INT, left, 123, getBuffer, 10, MPI_INT, right, 123, MPI_COMM_WORLD, &status);
    printf("rank:%d,from %d to %d",myid,left,right);
    MPI_Finalize();
    return 0;
}
```
* MPI_Sendrecv_replace
发送和接收只使用一个buffer。
```c
int MPI_Sendrecv_replace(
       void *buf, int count, MPI_Datatype datatype, 
       int dest, int sendtag, int source, int recvtag,
       MPI_Comm comm, MPI_Status *status)
```
* MPI_Probe和MPI_Iprobe
$qquad$MPI_Probe()和MPI_Probe()函数探测接收消息的内容，但不影响实际接收到的消息。我们可以根据探测到的消息内容决定如何接收这些消息，比如根据消息大小分配缓冲区等等。需要说明的是，这两个函数第一个是阻塞方式，即只有探测到匹配的消息才返回；第二个是非阻塞方式，即无论探测到与否都立即返回。
```c
int MPI_Probe (int source /* in */,
            int tag /* in */,
            MPI_Comm comm /* in */,
            MPI_Status* status /*out*/)
```
以上是阻塞型探测，直到有一个符合条件的消息到达，返回MPI_ANY_SOURCE和 MPI_ANY_TAG
```c
int MPI_Iprobe (int source /* in */,
            int tag /* in */,
            MPI_Comm comm /* in */,
            int * flag /*out*/,
            MPI_Status* status /*out*/)
```
以上是非阻塞型探测，无论是否有一个符合条件的消息到达，立即返回。有flag=true；否则flag=false。 

例子：

```c
#include "mpi.h"
#include <stdio.h>
 
#define MAX_BUF_SIZE_LG 22
#define NUM_MSGS_PER_BUF_SIZE 5
char buf[1 << MAX_BUF_SIZE_LG];

int main(int argc, char **argv)
{
    int p_size;
    int p_rank;
    int msg_size_lg;
    int errs = 0;
    int mpi_errno;
    MPI_Init(&argc, &argv);
    MPI_Comm_size(MPI_COMM_WORLD, &p_size);
    MPI_Comm_rank(MPI_COMM_WORLD, &p_rank);
    for (msg_size_lg = 0; msg_size_lg <= MAX_BUF_SIZE_LG; msg_size_lg++)
    {
        const int msg_size = 1 << msg_size_lg;
        int msg_cnt;
        printf( "testing messages of size %d\n", msg_size );fflush(stdout);
        for (msg_cnt = 0; msg_cnt < NUM_MSGS_PER_BUF_SIZE; msg_cnt++)
        {
            MPI_Status status;
            const int tag = msg_size_lg * NUM_MSGS_PER_BUF_SIZE + msg_cnt;
            printf( "Message count %d\n", msg_cnt );fflush(stdout);
            if (p_rank == 0)
            {
                int p;
                for (p = 1; p < p_size; p ++)
                {
                    /* Wait for synchronization message */
                    MPI_Recv(NULL, 0, MPI_BYTE, MPI_ANY_SOURCE, tag, MPI_COMM_WORLD, &status);
                    if (status.MPI_TAG != tag)
                    {
                        printf("ERROR: unexpected message tag from MPI_Recv(): lp=0, rp=%d, expected=%d, actual=%d, count=%d\n", status.MPI_SOURCE, status.MPI_TAG, tag, msg_cnt);fflush(stdout);
                    }
                    /* Send unexpected message which hopefully MPI_Probe() is already waiting for at the remote process */
                    MPI_Send (buf, msg_size, MPI_BYTE, status.MPI_SOURCE, status.MPI_TAG, MPI_COMM_WORLD);
                }
            }
            else
            {
                int incoming_msg_size;
                /* Send synchronization message */
                MPI_Send(NULL, 0, MPI_BYTE, 0, tag, MPI_COMM_WORLD);
                /* Perform probe, hopefully before the master process can send its reply */
                MPI_Probe(MPI_ANY_SOURCE, MPI_ANY_TAG, MPI_COMM_WORLD, &status);
                MPI_Get_count(&status, MPI_BYTE, &incoming_msg_size);
                if (status.MPI_SOURCE != 0)
                {
                    printf("ERROR: unexpected message source from MPI_Probe(): p=%d, expected=0, actual=%d, count=%d\n", p_rank, status.MPI_SOURCE, msg_cnt);fflush(stdout);
                }
                if (status.MPI_TAG != tag)
                {
                    printf("ERROR: unexpected message tag from MPI_Probe(): p=%d, expected=%d, actual=%d, count=%d\n", p_rank, tag, status.MPI_TAG, msg_cnt);fflush(stdout);
                }
                if (incoming_msg_size != msg_size)
                {
                    printf("ERROR: unexpected message size from MPI_Probe(): p=%d, expected=%d, actual=%d, count=%d\n", p_rank, msg_size, incoming_msg_size, msg_cnt);fflush(stdout);
                }
 
                /* Receive the probed message from the master process */
                MPI_Recv(buf, msg_size, MPI_BYTE, 0, tag, MPI_COMM_WORLD, &status);
                MPI_Get_count(&status, MPI_BYTE, &incoming_msg_size);
                if (status.MPI_SOURCE != 0)
                {
                    printf("ERROR: unexpected message source from MPI_Recv(): p=%d, expected=0, actual=%d, count=%d\n", p_rank, status.MPI_SOURCE, msg_cnt);fflush(stdout);
                }
                if (status.MPI_TAG != tag)
                {
                    printf("ERROR: unexpected message tag from MPI_Recv(): p=%d, expected=%d, actual=%d, count=%d\n", p_rank, tag, status.MPI_TAG, msg_cnt);fflush(stdout);
                }
                if (incoming_msg_size != msg_size)
                {
                    printf("ERROR: unexpected message size from MPI_Recv(): p=%d, expected=%d, actual=%d, count=%d\n", p_rank, msg_size, incoming_msg_size, msg_cnt);fflush(stdout);
                }
            }
        }
    }
    MPI_Finalize();
    return 0;
}
```
* MPI_Get_count

  根据 status 和 datatype，查询实际接受到了数据个数保存在 *count 中。
```c
int MPI_Get_count(
  MPI_Status *status,
  MPI_Datatype datatype,
  int *count
);
```

* MPI_Isend和MPI_Irecv

  与MPI_Send与MPI_Recv不同，MPI_Isend与MPI_Irecv均为非阻塞式通信。
```c
int MPI_Isend(
  void *buf,
  int count,
  MPI_Datatype datatype,
  int dest,
  int tag,
  MPI_Comm comm,
  MPI_Request *request
);
```
```c
int MPI_Irecv(
  void *buf,
  int count,
  MPI_Datatype datatype,
  int source,
  int tag,
  MPI_Comm comm,
  MPI_Request *request
);
```
例子：
```c
#include<stdio.h>
#include "mpi.h"
int main( int argc, char* argv[] ){
    int rank, nproc;
    int isbuf, irbuf, count;
    MPI_Request request;
    MPI_Status status;
    int TAG = 100;
 
    MPI_Init( &argc, &argv );
    MPI_Comm_size( MPI_COMM_WORLD, &nproc );
    MPI_Comm_rank( MPI_COMM_WORLD, &rank );

    if(rank == 0) {
         isbuf = 9;
         MPI_Isend( &isbuf, 1, MPI_INT, 1, TAG, MPI_COMM_WORLD, &request );
    } else if(rank == 1) {
        MPI_Irecv( &irbuf, 1, MPI_INT, 0, TAG, MPI_COMM_WORLD, &request);
        MPI_Wait(&request, &status);
        MPI_Get_count(&status, MPI_INT, &count);
        printf( "irbuf = %d source = %d tag = %d count = %d\n", 
                   irbuf, status.MPI_SOURCE, status.MPI_TAG, count);
        }
    MPI_Finalize();
    return 0;
}
```
* MPI_Wait，MPI_Waitany，MPI_Waitall和MPI_Waitsome
```c
int MPI_Wait(
  MPI_Request *request,
  MPI_Status *status
);
```
​        当程序运行到时MPI_Wait，会堵塞直到MPI_Wait传入的句柄参数对应的函数被完成。以非阻塞通信对象作为参数，一直等到相应的非阻塞通信完成后才成功返回，将相关信息放入status中，并释放这一非阻塞通信对象。
```c
int MPI_Waitall(
  int count,
  MPI_Request array_of_requests[],
  MPI_Status array_of_statuses[]
);
```
​         所有通信操作完成之后才返回，否则将一直等待。当所有的非阻塞通信完成时才成功返回，第i个非阻塞通信对象对应的通信完成信息存放在array_of_statuses[i]中，并释放非阻塞完成对象数组。
```c
int MPI_Waitany(
  int count,
  MPI_Request array_of_requests[],
  int *index,
  MPI_Status *status
);
```
​         当所有请求句柄中至少有一个已经完成通信操作，就返回，如果有多于一个请求句柄已经完成，MPI_waitany将随机选择其中的一个并立即返回。当存在多个非阻塞通信对象时，我们用MPI_Request request[count] 来定义非阻塞完成对象数组。MPI_Wait用于等待非阻塞通信对象数组中的任意一个非阻塞通信对象的完成，一旦有一个非阻塞通信完成后，就返回该非阻塞通信所对应非阻塞通信对象在上述数组中的标签index，并释放该非阻塞通信对象。同时，把该通信的相关信息存放在status中返回。
```c
int MPI_Waitsome(
  int incount,
  MPI_Request array_of_requests[],
  int *outcount,
  int array_of_indices[],
  MPI_Status array_of_statuses[]
);
```
​         MPI_Waitsome与其余完成调用最大的不同之处在于增加了下标数组array_of_indices，当任意数目的非阻塞通信完成时，MPI_Wait便返回，完成非阻塞通信的数目记录在outcount中，相应的非阻塞通信对象的下标存放在下标数组中，对应通信的相关信息存放在array_of_statuses。

* MPI_Test，MPI_Testany， MPI_Testsome，MPI_Testall和MPI_Test_cancelled
```c
int MPI_Test(
  MPI_Request *request,
  int *flag,
  MPI_Status *status
);
```
与MPI_Wait不同，MPI_Test在调用后会立刻返回，若相应非阻塞通信已完成，则完成标志flag=true。反之，完成标志flag=false。
```c
int MPI_Testany(
  int count,
  MPI_Request array_of_requests[],
  int *index,
  int *flag,
  MPI_Status *status
);
```
用于测试非阻塞通信数组中是否有任何一个对象已经完成，若有对象完成（若有多个，任取一个），令flag=true，并释放该对象。
```c
int MPI_Testsome(
  int incount,
  MPI_Request array_of_requests[],
  int *outcount,
  int array_of_indices[],
  MPI_Status array_of_statuses[]
);
```
立即返回，有几个非阻塞通信已经完成，就令outcount等于几，且将完成对象的下标记录在下标数组中。若没有非阻塞通信完成，则返回outcount=0。（并不立flag - -)
```c
int MPI_Testall(
  int count,
  MPI_Request array_of_requests[],
  int *flag,
  MPI_Status array_of_statuses[]
);
```
当非阻塞通信数组中有任意一个非阻塞通信对象对应的非阻塞通信没有完成时，令flag=false并立即返回。当所有通信都已经完成时，令flag=true并返回。
```c
int MPI_Test_cancelled(
  MPI_Status *status,
  int *flag
);
```
如果与状态对象关联的通信已成功取消，则返回flag = true。否则 返回flag = false。

* MPI_Send_init和MPI_Recv_init
```c
int MPI_Send_init(
  void *buf,
  int count,
  MPI_Datatype datatype,
  int dest,
  int tag,
  MPI_Comm comm,
  MPI_Request *request
);
```
```c
int MPI_Recv_init(
  void *buf,
  int count,
  MPI_Datatype datatype,
  int source,
  int tag,
  MPI_Comm comm,
  MPI_Request *request
);
```
   如果一个通信在一个并行计算的内部循环中不断地以同样的参数被执行。在这种情况下，该通信可以优化：把这些通信参数一次性捆绑到一个坚持式通信请求，然后不断用该请求初始化和完成消息。

* MPI_Start和MPI_Startall
```c
int MPI_Start(
  MPI_Request *request
);
```
  处于非活动状态的请求发出调用后，请求将变为活动状态。
```c
int MPI_Startall(
  int count,
  MPI_Request array_of_requests[]
);
```
   MPI_Startall的调用的效果与MPI_Start 以某种任意顺序为 i=0 ,..., count-1 执行的调用的效果相同。
* MPI_Request_free
```c
int MPI_Request_free(
  MPI_Request *request
);
```
   此例程通常用于释放使用MPI_Recv_init或MPI_Send_init创建的非活动持久请求。也可以释放活动请求。但是，一旦释放，请求不能再用于wait或test例程。
* MPI_Cancel
```c
int MPI_Cancel(
  MPI_Request *request
);
```
   MPI_Cancel操作允许取消挂起的通信。这是清理所必需的。发布发送或接收会占用用户资源（发送或接收缓冲区），可能需要取消以正常释放这些资源。MPI_Cancel可用于取消使用持久请求的通信，就像用于非持久请求一样。成功取消将取消活动通信，但不会取消请求本身。在呼叫MPI_Cancel以及随后对MPI_Wait或MPI_Test的呼叫后，请求将变为非活动状态，可以激活以进行新的通信。


