#####  进程组的创建
* MPI_Comm_Group
```c
int MPI_Comm_group(
  MPI_Comm comm,
  MPI_Group *group
);
```
把相同的通信子进程放到一个组内。
```c
#include<stdio.h>
#include<mpi.h>
#include<stdlib.h>
#include<time.h>
int main(int argc, char** argv)
{
	int world_rank, world_size, rank, size;
	MPI_Comm dup_comm_world,world_comm;
	MPI_Group world_group;
	MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
	MPI_Comm_size(MPI_COMM_WORLD, &world_size);
	MPI_Comm_dup(MPI_COMM_WORLD, &dup_comm_world);
	MPI_Comm_group(dup_comm_world, &world_group);
	MPI_Comm_create(dup_comm_world, world_group, &world_comm);
	MPI_Comm_rank(world_comm, &rank);
	if (rank != world_rank)
	{
        //验证复制后有无变化，如果正确的话，该代码块不会被执行。
		printf("incorrect rank in world comm:%d\n", rank);
	}
	MPI_Finalize();
	return 0;
}
```
* MPI_Comm_split
```c
int MPI_Comm_split(
  MPI_Comm comm,
  int color,
  int key,
  MPI_Comm *newcomm
);
```
>MPI_Comm_split主要有以下四个参数：
>第一个参数comm为原来的域的整体范围，也就是被划分的范围。
>第二个参数为color，相同的color的节点会被划分成同一个子域。如果color被置为MPI_UNDEFINED，进程不会被纳入通信域。
>第三个参数为key，在每个子域中会有诸多节点，节点在子域中的rank是多少，是通过key从小到大进行排列从而产生的。rank最小的进程会被置为0，第二小的会被置为1，以此类推。
第四个参数为newcomm，也就是一个新的通信域。
```c
#include<stdio.h>
#include<mpi.h>
#include<stdlib.h>
#include<time.h>
int main(int argc, char** argv)
{
	int row_rank,row_size,world_rank, world_size;
	MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
	MPI_Comm_size(MPI_COMM_WORLD, &world_size);
	int color = world_rank / 4;
	MPI_Comm row_comm;
	MPI_Comm_split(MPI_COMM_WORLD, color, world_rank, &row_comm);
	MPI_Comm_rank(row_comm, &row_rank);
	MPI_Comm_size(row_comm, &row_size);
	printf("world_rank/size:%d/%d \t row_rank/size:%d/%d\n", world_rank, world_size, row_rank, row_size);
	MPI_Comm_free(&row_comm);
	MPI_Finalize();
	return 0;
}
```
* MPI_Group_union
```c
int MPI_Group_union(
  MPI_Group group1,
  MPI_Group group2,
  MPI_Group *newgroup
);
```
求并集

* MPI_Group_intersection
```c
int MPI_Group_intersection(
  MPI_Group group1,
  MPI_Group group2,
  MPI_Group *newgroup
);
```
交集
* MPI_Group_difference
```c
int MPI_Group_difference(
  MPI_Group group1,
  MPI_Group group2,
  MPI_Group *newgroup
);
```
* MPI_Group_incl
```c
int MPI_Group_incl(
  MPI_Group group,
  int n,
  int *ranks,
  MPI_Group *newgroup
);
```
在一个group的基础上创建一个新的group。

* MPI_Group_excl
```c
int MPI_Group_excl(
  MPI_Group group,
  int n,
  int *ranks,
  MPI_Group *newgroup
);
```
与MPI_Group_incl相反，取不在ranks里的组成新group。
* MPI_Group_rance_incl
```c
int MPI_Group_range_incl(
  MPI_Group group,
  int n,
  int ranges[][3], 
  MPI_Group *newgroup
);

```
* MPI_Group_rance_excl
```c
int MPI_Group_range_excl(
  MPI_Group group,
  int n,
  int ranges[][3], 
  MPI_Group *newgroup
);
```
例子：
```c
#include<stdio.h>
#include<mpi.h>
#include<stdlib.h>
#include<time.h>
int main(int argc, char** argv)
{
	int row_rank,row_size,world_rank, world_size;
	int ranks1[5] = { 0,1,2,3,4 };
	int ranks2[5] = { 5,6,7,8,9 };
	int result;
	MPI_Group g1, g2, g3, g4;
	MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
	MPI_Comm_size(MPI_COMM_WORLD, &world_size);
	MPI_Comm_group(MPI_COMM_WORLD, &g1);
	MPI_Group_incl(g1, 5, ranks1, &g2);
	MPI_Group_incl(g1, 5, ranks2, &g3);
	MPI_Group_union(g2, g3, &g4);
	MPI_Group_compare(g1, g4, &result);
	printf("the result is %d for union", result);
	MPI_Finalize();
	return 0;
}
```
##### 进程组管理

* MPI_Group_size
```c
int MPI_Group_size(
  MPI_Group group,
  int *size
);
```
* MPI_Group_rank
```c
int MPI_Group_rank(
  MPI_Group group,
  int *rank
);
```
例子：
```c
#include<stdio.h>
#include<mpi.h>
#include<stdlib.h>
#include<time.h>
int main(int argc, char** argv)
{
	int world_rank, world_size;
	int re1, re2;
	int r1,r2;
	int ranks[5] = { 0, 1, 2, 3, 4 };
	MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
	MPI_Comm_size(MPI_COMM_WORLD, &world_size);
	MPI_Group world_group,g1,g2;
	MPI_Comm_group(MPI_COMM_WORLD, &world_group);
	MPI_Group_incl(world_group, 5, ranks, &g1);
	MPI_Group_excl(world_group, 5, ranks, &g2);
	MPI_Group_size(g1, &re1);
	MPI_Group_size(g2, &re2);
	MPI_Group_rank(g1, &r1);
	MPI_Group_rank(g2, &r2);
	printf("rank1:%d \t rank2:%d\n", r1,r2);
	printf("re1:%d \t re2:%d\n", re1, re2);
	MPI_Finalize();
	return 0;
}
```
* MPI_Group_translate_ranks
```c
int MPI_Group_translate_ranks(
  MPI_Group group1,
  int n,
  int *ranks1,
  MPI_Group group2,
  int *ranks2
);
```
为了防止忘记rank具体进程号
例子：
```c
#include<stdio.h>
#include<mpi.h>
#include<stdlib.h>
#include<time.h>
int main(int argc, char** argv)
{
	int world_rank, world_size;
	int *ranks;
	int *ranks_out;
	MPI_Comm newcomm;
	MPI_Group basegroup,g1;
	MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
	MPI_Comm_size(MPI_COMM_WORLD, &world_size);
	MPI_Comm_group(MPI_COMM_WORLD, &basegroup);
	MPI_Comm_split(MPI_COMM_WORLD, 0, world_size - world_rank, &newcomm);
	MPI_Comm_group(newcomm, &g1);
	ranks = (int*)malloc(world_size * sizeof(int));
	ranks_out = (int*)malloc(world_size * sizeof(int));
	for (int i = 0; i < world_size; i++) ranks[i] = i;
	MPI_Group_translate_ranks(g1, world_size, ranks, basegroup, ranks_out);
	for (int i = 0; i < world_size; i++) {
		printf("Translate ranks got %d expected %d\n", ranks_out[i], (world_size - 1) - i);
	}
	MPI_Finalize();
	return 0;
}
```
* MPI_Comm_create
```c
int MPI_Comm_create(
  MPI_Comm comm,
  MPI_Group group,
  MPI_Comm *newcomm
);
```
* MPI_Group_free
```c
int MPI_Group_free(
  MPI_Group *group
);
```
* MPI_Group_compare
```c
int MPI_Group_compare(
  MPI_Group group1,
  MPI_Group group2,
  int *result
);
```
result有三种情况，如果两个组相同输出MPI_IDENT；如果只有部分元素相同，输出MPI_SIMILAR；其他输出MPI_UNEQUAL。


