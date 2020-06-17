#####  MPI示例
* MPI时间函数测试
```c
#include<stdio.h>
#include<mpi.h>
#include<stdlib.h>
#include<time.h>
#include<windows.h>
int main(int argc, char** argv)
{
	int err = 0;
	double t1, t2;
	double tick;
	MPI_Init(&argc, &argv);
	t1 = MPI_Wtime();
	t2 = MPI_Wtime();
	if (t2 - t1 > 0.1 || t2 - t1 < 0.0)
	{
		/* 若连续的两次时间调用得到的时间间隔过大 这里是超过0.1秒 或者后调用的函数
           得到的时间比先调用的时间小 则时间调用有错*/
		printf("Two success calls to MPI_Wtime gave strange results:(%f)(%f)\n", t1, t2);
	}
	for (int i = 0; i < 10; i++)
	{
		t1 = MPI_Wtime();
		Sleep(1);
		t2 = MPI_Wtime();
		if (t2 - t1 >= (1.0 - 0.01) && t2 - t1 <= 5.0)
		{
			printf("计数准确\n");
			break;
		}
		if (t2 - t1 > 5.0)
		{
			printf("计数错误\n");
			tick = MPI_Wtick();
			if (tick > 1.0 || tick < 0.0)
			{
				printf("MPI_Wtick gave a strange result:%f\n", tick);
			}
			break;
		}
		MPI_Finalize();
		return 0;
	}
```
* MPI获取进程名称和MPI版本号
```c
#include<stdio.h>
#include<mpi.h>
#include<stdlib.h>
#include<time.h>
int main(int argc, char** argv)
{
	int rank, size, len,version,sub_version;
	char name[MPI_MAX_PROCESSOR_NAME];
	MPI_Init(&argc, &argv);
	MPI_Comm_size(MPI_COMM_WORLD, &size);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);
	MPI_Get_processor_name(name, &len);
	MPI_Get_version(&version, &sub_version);
	printf("Hello, world, I am % d of % d on % s in MPI_Version %d.%d\n", rank, size, name,version,sub_version);
	MPI_Finalize();
	return 0;
}
```
* MPI通信接力
```c
#include<stdio.h>
#include<mpi.h>
#include<stdlib.h>
#include<time.h>
int main(int argc, char** argv)
{
	int rank, size, value = 10;
	MPI_Status status;
	MPI_Init(&argc, &argv);
	MPI_Comm_size(MPI_COMM_WORLD, &size);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);
	do 
	{
		if (rank == 0)
		{
	//		printf("Please give new value = ");
	//		scanf_s("%d", &value);
	//		printf("%d read <----- %d\n", rank, value);
			if (size > 1)
			{
				MPI_Send(&value, 1, MPI_INT, rank + 1, 0, MPI_COMM_WORLD);
				printf("%d send %d -----> %d",rank,value,rank + 1);
			}
		}
		else
		{
			MPI_Recv(&value, 1, MPI_INT, rank - 1, 0, MPI_COMM_WORLD, &status);
			printf("%d receive %d <----- %d\n", rank, value, rank - 1);
			if (rank < size - 1)
			{
				MPI_Send(&value, 1, MPI_INT, rank + 1, 0, MPI_COMM_WORLD);
			}
		}
		MPI_Barrier(MPI_COMM_WORLD);
	} while (value >= 0);
	MPI_Finalize();
	return 0;
}
```
