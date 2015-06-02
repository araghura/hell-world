MapReduce Implemented in C++
=============================

Authors / Owners
-----------------
Anantha Raghuraman 
-------------------
Jayakumaran Ravi 
-----------------



# Perform MapReduce operation using MPI and OpenMP Frameworks

Goal of the Projects
---------------------
1) Read all the words from multiple files and obtain a count of all the unique words.

2) Use MPI framework to parallelize the work and run on multiple nodes. 

3) Further, on each node, use OpenMP to run the work on multiple threads.


Project Overview
-----------------------
MapReduce is a programming model that involves two steps. The first, the map step, takes an input set I and groups it into N equivalence classes I0, I1, I2, ..., IN-1. I can be thought of as a set of tuples <key, data>, and the function map maps I into the equivalence classes based on the value of key. In the second reduce step, the equivalence classes are processed, and the set of tuples in an equivalence class Ij are reduced into a single value.

1) The project uses MPI and OpenMP frameworks by including 'mpi.h' and 'omp.h' header files and using the right compilation commands (see later).


2) There are 4 important files in this implementation. They are described below:

	a) list.txt: The list of files to be read is stored here.

	b) mapreduce_omp.cpp: This defines a function that does initial mapreduce. This function is used by processes in mapreduce_mpi.cpp.

	c) mapreduce_omp.h: This header file declares the function defined in mapreduce_omp.cpp.

	d) mapreduce_mpi.cpp: This uses the MPI libraries to parallelzie Mapreduce oprations on multiple nodes (processes). It finally prints the result to stdout.


mapreduce_omp.cpp
-----------------

a) If there are N files to read and k processes, then (approx) N/k files are read by each process. Each process further divides the workload amongst multiple threads.

b) There are four kinds of threads:
	i. Reader threads (all even thread ids), which read files and put the data read into a work queue 1. Each work item will be a word.

	ii. Mapper threads (all odd thread ids), which execute in parallel with Reader threads, create combined records of words. That is, if one mapper thread dequeues 100 instances of "cat" and 50 instances of "dog" and so on.., its output will have {<"cat", 100>, <"dog", 50>, ...}.

	Mappers and Readers work in parallel and interact by means of the work queue 1. The access to the work queue is controlled by locks. After Reader threads are finished and the work queue is empty, the Reader threads become Reducer threads.

	iii. Reducer threads that operate on work queue entries created by mapper threads and combine (reduce) them to a single record.  Thus, for the word “cat”, there is potentially a <“cat”,counti> record sent by every mapper thread ti in the system and it will sum all of the counts and place it on a work queue 2.   For each word there is exactly one Reducer thread in the system that handles it.

	iv. Writer threads that take a sum from the work queue and write it to a file.
You may not need threads for for each of these but only different work queue entries.  Thus, Reader 
and Writer threads run at different times.  Mapper and Reducer threads, within a node, can be made to 
run at different times.  These threads can be made to do different tasks by pulling different work out 
of work queues.  This is not mandatory.
2.
A work queue for each reducer thread.  Mapper threads will put work items into this queue.  For load 
balance purposes it is desirable that the range of function 
H
 that determines which reducer will get a 
work item be from 0 to R where 
R = k
⋅
numMappers
, and 
k
 is some constant.
3.
You need to have mechanisms to ensure that Mapper threads wait until all Readers have finished 
before considering themselves complete, i.e. the work queue Mapper threads get work from may be 
empty because they have gotten ahead of the Reader threads.
4.
Mappers will need to put their data on a reducer’s work queue based on the key (word) for that data:
a.
As mentioned above, the reducer of a key should be determined by some sort of hash function 
g 
= H(key)
.  All keys that map onto reducer 
g
 should be added to 
g
’s work queue. 
b.
Each process can assume it will be receiving data from every other node.  This will simplify the 
communication structure of your program when you go to the MPI version.  A node that sends no 
data should send an “empty” record letting the other process no it will get no data from it.
5.
As each process finishes its reduce work, it should write its results to an output file, close it, and 
notify the master thread that it is finished so that it can terminate the job, and then terminate itself.



