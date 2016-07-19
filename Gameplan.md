
matrices are transferred back and forth as index row partitioned matrices

1. the spark program will take as input the file listing the spark slaves and the number of mpi processes to use for the computation (some will serve double duty as handler and receiver/sender processes below) 

2. to do LA, the driver will start an mpi handler process which will take this file as input along with any information needed on the matrix on the mpi side, such as the matrix size and the number of partitions in the output matrix

3. spark will also construct a new rdd to hold the output (by parallelizing an array of partition indices). each node will send its partition indices to the mpi handler, so that the handler knows how to partition the output matrix and where to send the chunks

4. the handler will start one or several mpi receiver process on each node, and continue running in the background

5. the spark executors on each node will send their partitions from the input RDD to the receiver processes on that node --- there needs to be a handshake procedure since several partitions will be writing to the same receiver potentially simultaneously

6. once we've received all the expected partitions (we'll know because we know the matrix size), the mpi processes will redistribute the data into an Elemental DistMatrix and run whatever computation

7. once the mpi computation is done, the computed matrix will be repartitioned into appropriate partitions and sent to the sender mpi processes on each node.

8. each node will fill its partitions by piping in data from the sender process on that node (it'll pass in its partition number). alternatively, all the output can be collected to the driver directly by contacting the handler.
 
9. spark will tell the mpi handler process to shut everything down

Future extensions:
 keep the matrix in memory on the mpi side, and maintain handler in the background, so can do multiple operations on the same matrix
 support commands taking multiple RDD inputs and multiple RDD outputs
