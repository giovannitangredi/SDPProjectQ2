# Assignment Quer Project 2

Group # 27 
Students: Giovanni Tangredi s276086 , Francesco Xia s276509

Link per scaricare il codice sorgente 
https://github.com/giovannitangredi/SDPProjectQ2

## Suggested action for compiling and executing the project
<ins>This project has been written and tested for a Linux machine</ins>

Execute the following commands:
```bat
make
./bin/main <graph_file> <num_intervals> <query_file> [-l] [-q]

OPTIONS
	graph_file    = relative or absolute path
	n             = number of intervals
	query_file    = relative or absolute path
	-l            = print generated labels to "test/output/labels_out.txt" (optional)
	-q            = print generated query results to "test/output/queries_out.txt" (optional)
```
NOTE: <ins>make sure TEST (constants.h) is set to 0 when running the program. Otherwise the generation of the intervals is not random. </ins>

To run the test suites:
```bat
make test_node
./bin/test
```
The other rules, related to testing, are: test_label, test_graph, test_query, test_bitmap

NOTE: make sure TEST (constants.h) is set to 1 when running test cases
## Project Stucture
``` bash
SDPProjectQ2
├── README.md
├── bin
├── gen_query.c
├── include
│   ├── bitmap.h
│   ├── constants.h
│   ├── graph.h
│   ├── label.h
│   ├── menu.h
│   ├── query.h
│   └── time_tracker.h
├── makefile
├── src
│   ├── bitmap.c
│   ├── graph.c
│   ├── label.c
│   ├── main.c
│   ├── menu.c
│   ├── query.c
│   └── time_tracker.c
└── test
    ├── input
    │   ├── grafo20.gra
    │   ├── grafo20.png
    │   ├── grafo20_25.que
    │   └── grafo_con_rango.png
    │ 
    │       
    ├── output
    │   ├── labels_out.txt
    │   └── queries_out.txt
    └── tests
        ├── bitmap_test.c
        ├── graph_test.c
        ├── label_test.c
        ├── node_test.c
        └── query_test.c
```
The repository is divided as the following:
* the **bin folder** contains the executables.
* the **src folder** contains the source files of the project.
* the **test folder** is further divided in:
    * the input folder contains a sample of a graph plus query.
    * the output folder contains the output generated by the program (i.e. the labels generated, the query results, and copy of the graph).
    * the tests folder contains the actual test source files.
* the **include folder** contains the header files.

Note: by changing the values of the macros from the file *constant.h* allows the user to customize extra features, such as:
* DEBUG: if set, it prints to stdout extra information useful for debugging purposes (it is strongly recommended to turn it on)
* TEST: this flag must be set when running tests, otherwise it recommended to be turned off. Warning: during normal exucution (i.e. not running tests), if TEST is set the labels are not generated randomly.
* MAX_THREADS_GRAPH: number of running threads used during the graph generation phase. MUST be > 0.
* MAX_THREADS_QUERY: number of running threads used to solve the queries. MUST be > 0.
* ALL_NODES: macro passed to graph_print_to_stdout to print all the nodes on stdout.

The tests were created using the Check.h Framework (thus, it needs to be pre-installed when running the test suites).

## Program output
If the option *-q* is passed, the program will generate a file called *queries_out.txt*, such that for each line it contains:
*source_id destination_id {0/1}* (unreachable/reachable).

If the option *-q* is missing, the user will be able to interactively ask the reachability state of a query.

Similarly passing the option *-l*, it will create a file named *labels_out.txt*, with the following format:
*node_id: [l1_l, l1_r] [l2_l, l2_r] ...*

## Data Stuctures 
The graph is represented using a data structure similar to an Adjacency list. A vector of Node pointers serves as the base for the graph. Each node is indexed, through graph->nodes, with its node_id. Furthermore, each node saves:
* a set of node indeces, which replicates the list of its outgoing edges.
* a vector of intervals of size *num_intervals* 

The queries are stored in a static struct called *queries* under query.c
* a single query is represented as a *route* (route.src and route.dst);
* the result of a query is saved as a bit in a bitmap inside *queries*, which state indicates either non-reachability/reachability of the route (0/1). The i-th bit of the bitmap represents the state of i-th query (i.e. the query found at i-th lines of the file <query_file>, counting from 0);

## Adopted strategies
The generation of the graph follows this main ideas:
* the generation is done concurrently by running *MAX_THREADS_GRAPH number* of threads 
* at each iteration, each threads
    * reads *x* lines
    * increment shared counter of read lines (see p_curr_iteration)
    * parse those lines (e.g. find the node_id of the current node and of its children)
    * allocates a single chunk of memory, big enough to store *x* Nodes (see node_create_multiple) 
    * save each created Node at graph->nodes[<node_id>] = node;

repeat until end of file.

The intervals are generated concurrently by running *n* threads, where *n* is equal to the passed option *num_intervals*. The interval generated by i-th thread is saved in the corresponding Node.intervals[i-th];

The label generator is an implementation of the "Algorithm 1: GRAIL Indexing: Randomized Intervals" found in the GRAIL official publication. 

Similarly during the second phase, the queries, read from file, are equally distributed in *n* blocks, where *n* is empirically set to 4 (can be changed through constants.h). Each block then is assigned to a thread, which task is to find the non-reachability/reachability of every query of its block.
The query solver is an implementation of the "Algorithm 2: GRAIL Query:Reachability Testing" found in the GRAIL official publication".
