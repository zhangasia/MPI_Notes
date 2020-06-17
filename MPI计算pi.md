##### MPI计算$\pi$
* 利用公式
$$
\int_0^1 \frac{4}{1+x^2}dx = \pi
$$

<img src="images\计算pi\计算pi示意图.png" alt="计算pi示意图" style="zoom:30%;" />

```c
#include<stdio.h>
#include<mpi.h>
#include<stdlib.h>
#include<time.h>
int main(int argc, char** argv)
{
	int rank, size;
	int n = 100;
	double sum,width,local,mypi,pi;
	MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);
	MPI_Comm_size(MPI_COMM_WORLD, &size);
	MPI_Bcast(&n, 1, MPI_INT, 0, MPI_COMM_WORLD);
	sum = 0.0;
	width = 1.0 / n;
	/*
	* 把0~1分成n份，把计算任务分成size份。
	*/
	for (int i = rank; i < n; i += size)
	{
		local = width * ((double)i + 0.5);
		sum += 4.0 / (1.0 + local * local);
	}
	mypi = width * sum;
	MPI_Reduce(&mypi, &pi, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);
	if (rank == 0)
	{
		printf("pi is %.20f\n", pi);
		fflush(stdout);
	}
	MPI_Finalize();
	return 0;
}
```