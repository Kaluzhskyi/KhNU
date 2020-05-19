#include <mpi.h>
#include <iostream>
#include <fstream>
#include <string.h>
using namespace std;
MPI_Status status;
MPI_Request request;
int arr[10] = { 9, 8, 7, 6, 0, 4, 3, 2, 1, 0 };
int msg[2];
int a[10];
int bf[4];
int *buf;
int t = 30;
void bubblesort(int *a, int n)
{
int i, j;
for (i = n - 1; i >= 0; i--)
{
for (j = 0; j < i; j++)
{
if (a[j] > a[j + 1])
swap(a[j], a[j + 1]);
}
}
}
void p(int rank, int p1, int p2)
{
if (rank == p2)
{
buf = new int[t + MPI_BSEND_OVERHEAD];
MPI_Buffer_attach(buf, t + MPI_BSEND_OVERHEAD);
10
MPI_Ibsend(msg, 2, MPI_INT, p1, 0, MPI_COMM_WORLD,
&request);
MPI_Buffer_detach(&buf, &t);
MPI_Wait(&request, &status);
MPI_Irecv(msg, 2, MPI_INT, p1, 0, MPI_COMM_WORLD,
&request);
MPI_Wait(&request, &status);
}
if (rank == p1)
{
MPI_Irecv(bf, 2, MPI_INT, p2, 0, MPI_COMM_WORLD,
&request);
MPI_Wait(&request, &status);
for (int i = 2; i < 4; i++)
{
bf[i] = msg[i - 2];
}
bubblesort(bf, 4);
for (int i = 0; i < 2; i++)
{
msg[i] = bf[i];
}
buf = new int[t + MPI_BSEND_OVERHEAD];
MPI_Buffer_attach(buf, t + MPI_BSEND_OVERHEAD);
MPI_Ibsend(bf + 2, 2, MPI_INT, p2, 0, MPI_COMM_WORLD,
&request);
MPI_Buffer_detach(&buf, &t);
MPI_Wait(&request, &status);
}
}
int main(int argc, char *argv[])
{
int rank;
int size;
MPI_Init(&argc, &argv);
MPI_Comm_rank(MPI_COMM_WORLD, &rank);
MPI_Comm_size(MPI_COMM_WORLD, &size);
if (size == 5)
{
MPI_Scatter(arr, 2, MPI_INT, msg, 2, MPI_INT, 0,
MPI_COMM_WORLD);
p(rank, 0, 1);
11
p(rank, 0, 2);
p(rank, 0, 3);
p(rank, 0, 4);
p(rank, 1, 2);
p(rank, 1, 3);
p(rank, 1, 4);
p(rank, 2, 3);
p(rank, 2, 4);
p(rank, 3, 4);
MPI_Gather(msg, 2, MPI_INT, a, 2, MPI_INT, 0,
MPI_COMM_WORLD);
if (rank == 0)
{
for (int i = 0; i < 10; i++)
{
cout << arr[i] << " ";
}
cout << endl;
for (int i = 0; i < 10; i++)
{
cout << a[i] << " ";
}
}
}
MPI_Finalize();
return 0;
}