HPC for Graph Neural Networks: Scaling GCNs on Large Datasets
=============================================================

**⚠️ Warning Note:**

-   For this assignment, you can utilize **up to 8 GPUs** for your experiments.

-   Please be mindful of resource usage: any idle instance (without active processes or user interaction) for more than 30 minutes may be automatically terminated. Save your work frequently.

-   **Note on GPU Usage in Provided Examples:** The scaffolded code examples provided are configured to run experiments with up to 3 GPUs for demonstration purposes. If you have more GPUs available (up to the maximum of 8), the code will automatically detect and utilize them for experiments with 1, 2, 4, and 8 partitions, where applicable.

### Assignment Overview

This assignment builds upon your foundational understanding of Graph Convolutional Networks (GCNs) and their application to the PubMed dataset. In this extension, you will tackle the challenges of scaling GCN training to large-scale graphs with millions of nodes and edges. You will explore advanced techniques such as graph partitioning, node/edge sampling, and performance profiling using HPC tools to develop efficient and scalable GCN solutions.

**Dataset:** While scaffolded example uses PubMed, this assignment will focus on a large-scale dataset, specifically a subset of the **Reddit dataset** for node classification/clustering.

**Learning Objectives:** Upon successful completion of this assignment, you will be able to:

1.  Understand the computational challenges of training GCNs on large graphs.

2.  Implement and compare different graph partitioning strategies (e.g., random, METIS) for distributed GCN training.

3.  Design and implement efficient node and edge sampling techniques to reduce computational load and memory footprint.

4.  Integrate partitioning and sampling into a parallel GCN training pipeline.

5.  Utilize HPC profiling tools to identify performance bottlenecks in your GCN workload.

6.  Propose and implement optimization strategies based on profiling results.

7.  Analyze and report on the trade-offs between model accuracy, training time, and resource utilization.

### Prerequisites

Before starting this assignment, ensure you have:

-   A solid understanding of Graph Neural Networks (GNNs), particularly GCNs.

-   Familiarity with graph data structures and graph algorithms.

-   Proficiency in Python and a GNN library (e.g., PyTorch Geometric (PyG), Deep Graph Library (DGL)).

-   Basic understanding of parallel computing concepts.

-   Access to a computational environment capable of handling large datasets (e.g., a server with sufficient RAM, potentially GPUs).

### Part 1: Baseline GCN on PubMed (Scaffolded)

*(This part is assumed to be completed as per your scaffolded example.)*

You should already have a working GCN implementation for the PubMed dataset, where you define and train a GCN to identify clusters (or perform node classification). This serves as your baseline for understanding GCN training mechanics before scaling up.

### Part 2: Scaling GCNs on the Reddit Dataset

The Reddit dataset is a large graph where nodes represent posts and edges represent interactions (e.g., comments, upvotes). The task is typically node classification (e.g., classifying posts into subreddits).

**Dataset Access:** You will use the **Reddit dataset** available through popular GNN libraries. For example, in PyTorch Geometric, you can load it via `torch_geometric.datasets.Reddit`. In DGL, it's available via `dgl.data.RedditDataset`. Be aware that downloading and loading this dataset can take time and significant memory.

**Task:** Your primary task is to train a GCN model on the Reddit dataset to achieve reasonable classification accuracy while optimizing for training efficiency (speed and resource usage).

#### 2.1 Graph Partitioning Strategies

Training on the entire Reddit graph may not be feasible on a single machine or may be too slow. Graph partitioning allows you to divide the graph into smaller subgraphs that can be processed in parallel.

**Requirements:**

1.  **Implement Random Partitioning:** Divide the Reddit graph into approximately equal-sized partitions (e.g., based on node count). For each node, randomly assign it to one of the P partitions. Edges are then assigned to partitions based on their connected nodes.

2.  **Implement METIS-based Partitioning:** Utilize the [METIS](https://www.google.com/search?q=http://glaros.dtc.umn.edu/gkhome/metis/metis/overview "null") library (or a Python wrapper like `PyMetis` or `metis` if available in your environment) to partition the Reddit graph into P partitions. METIS aims to minimize edge cuts while balancing partition sizes.

    -   **Note:** You might need to install METIS and its Python bindings separately. Ensure your graph representation is compatible with METIS's input format (e.g., adjacency list or CSR).

3.  **Analyze Partitioning Quality:** For both random and METIS partitioning, for P=2,4,8, report:

    -   The number of nodes and edges in each partition.

    -   The total number of "cut edges" (edges connecting nodes in different partitions).

    -   The time taken for the partitioning process.

    -   Discuss the trade-offs between random and METIS partitioning in terms of balance, edge cuts, and partitioning time.

#### 2.2 Node and Edge Sampling Techniques

Even within partitions, the subgraphs can still be large. Instead of training on the entire graph, **sampling techniques** allow you to train on a smaller, representative subset of the graph data at each training iteration, significantly reducing memory and computation, and impacting accuracy.

**Requirements:**

1.  **Implement at least two distinct sampling techniques:**

    -   **Neighbor Sampling (Node-centric):** For each training node, sample a fixed number of its neighbors (e.g., 5-10 neighbors for the first layer, then 5-10 for the second layer's neighbors, etc.). This is common in mini-batch GCN training.

        -   *Hint:* PyTorch Geometric offers `NeighborSampler` or `DataLoader` with `NeighborLoader`. DGL has similar constructs. These are often used for efficient mini-batch training on large graphs.

    -   **Random Edge Sampling (Graph-centric):** Randomly select a fixed percentage of edges from the graph (or subgraph) at each epoch/iteration to form a mini-batch.

    -   **Your Choice:** You may also explore other techniques like random walk-based sampling (e.g., as used in Node2Vec), importance sampling, or subgraph sampling (e.g., Cluster-GCN \cite{cluster_gcn_ref}).

2.  **Integrate Sampling into Training:** Modify your GCN training loop to incorporate your chosen sampling techniques. You will likely train in a mini-batch fashion, where each batch is constructed using sampled nodes/edges.

3.  **Evaluate Impact of Sampling:**

    -   Train your GCN using different sampling rates/parameters.

    -   Report the impact of different sampling strategies and parameters on:

        -   Model accuracy (e.g., F1-score for node classification).

        -   Training time per epoch.

        -   Memory consumption during training.

    -   Discuss the trade-offs between sampling efficiency and model performance.

#### 2.3 Parallel Training with Partitioning and Sampling

Combine your partitioning and sampling strategies to enable parallel training of your GCN model across multiple processes (simulating distributed training on a cluster or multiple CPU cores).

**Requirements:**

1.  **Design a Parallel Training Pipeline:** Outline how you would distribute the training process across P processes/workers. Consider:

    -   How will data (partitions) be loaded by each worker?

    -   How will model parameters be synchronized (e.g., parameter server, all-reduce)?

    -   How will gradients be aggregated?

    -   *Hint:* Consider using PyTorch's `DistributedDataParallel` (DDP) or DGL/PyG's built-in distributed training functionalities if available and suitable. If not, a simpler approach could involve each process training on its partition and periodically exchanging model weights.

2.  **Implement Parallel Training:** Implement your chosen parallel training strategy using the partitioned and sampled data.

3.  **Scalability Analysis:** Train your GCN with P=1,2,4,8 partitions/workers.

    -   Report the total training time for each configuration.

    -   Calculate the speedup achieved for P1.

    -   Discuss the scalability of your approach and identify any communication overheads observed.

#### 2.4 Performance Profiling and Optimization

Identifying bottlenecks is crucial for optimizing HPC applications. You will use profiling tools to pinpoint where your code spends most of its time and resources.

**Requirements:**

1.  **Choose and Use HPC Profiling Tools:** Select at least one profiling tool relevant to your environment (e.g., `cProfile` for Python, `perf` for Linux, `NVIDIA Nsight Systems` or `nvprof` if using GPUs, `htop` for general resource monitoring).

2.  **Profile Your Workload:** Run your parallel GCN training (e.g., with P=4 partitions and your best sampling strategy) under the chosen profiler.

3.  **Identify Bottlenecks:** Analyze the profiling reports to identify the most significant bottlenecks in your code. These could be:

    -   Data loading and preprocessing.

    -   Graph partitioning.

    -   Node/edge sampling.

    -   GCN forward/backward passes.

    -   Communication overhead between parallel processes.

    -   Memory access patterns.

4.  **Propose and Implement Optimizations:** Based on your profiling results, propose at least two concrete optimization opportunities. Implement at least one of these proposed optimizations.

    -   *Examples:* If data loading is slow, consider pre-loading or using more efficient data structures. If communication is high, explore reducing synchronization frequency or compressing data. If GPU utilization is low, ensure proper batching and kernel fusion.

5.  **Report Findings:** Present your profiling results (e.g., flame graphs, call stacks, time spent in functions). Clearly explain the bottlenecks identified and detail the optimization strategies you proposed and/or implemented, along with their impact on performance.

### Deliverables

Submit the following:

1.  **Jupyter Notebook:** A well-organized Jupyter Notebook (`.ipynb` file) containing:

    -   **All necessary library installations at the top** using `conda` or `pip` commands.

        -   **Example for Conda:**

            ```
            # Create a new conda environment (optional, but recommended)
            conda create -n gcn_hpc python=3.9
            conda activate gcn_hpc

            # Install PyTorch (adjust CUDA version if necessary, e.g., cudatoolkit=11.3)
            conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia

            # Install PyTorch Geometric and its dependencies
            pip install torch_geometric
            pip install pyg-lib torch-cluster -f https://data.pyg.org/whl/torch-2.1.0+cu118.html # Adjust torch/cuda version

            # Install Dask and distributed
            conda install dask distributed -c conda-forge

            # Install scikit-learn for Logistic Regression and clustering metrics
            conda install scikit-learn

            # Install matplotlib for plotting
            conda install matplotlib

            # Install PyMetis (requires METIS library first)
            # On Ubuntu/Debian: sudo apt-get update && sudo apt-get install libmetis-dev
            pip install PyMetis

            ```

        -   **Example for Pip:**

            ```
            # Install PyTorch (adjust CUDA version if necessary, e.g., torch==2.1.0+cu118)
            pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

            # Install PyTorch Geometric and its dependencies
            pip install torch_geometric
            pip install pyg-lib torch-cluster -f https://data.pyg.org/whl/torch-2.1.0+cu118.html # Adjust torch/cuda version

            # Install Dask and distributed
            pip install "dask[distributed]"

            # Install scikit-learn for Logistic Regression and clustering metrics
            pip install scikit-learn

            # Install matplotlib for plotting
            pip install matplotlib

            # Install PyMetis (requires METIS library first)
            # On Ubuntu/Debian: sudo apt-get update && sudo apt-get install libmetis-dev
            pip install PyMetis

            ```

    -   All your Python code for GCN implementation, partitioning, sampling, and parallel training.

    -   Clear instructions and explanations within Markdown cells on how to set up the environment, download the dataset, run your code, and reproduce your results.

2.  **Technical Report (PDF):** A comprehensive report (3-5 pages, excluding code snippets and appendices) detailing your work. The report should include:

    -   **Introduction:** Briefly state the problem and your approach.

    -   **Part 2.1: Graph Partitioning:**

        -   Description of random and METIS partitioning implementations.

        -   Table summarizing partitioning metrics (nodes/edges per partition, cut edges, time) for P=2,4,8.

        -   Discussion of trade-offs.

    -   **Part 2.2: Node and Edge Sampling:**

        -   Description of implemented sampling techniques.

        -   Table/graphs showing the impact of sampling parameters on accuracy, training time, and memory.

        -   Discussion of trade-offs.

    -   **Part 2.3: Parallel Training:**

        -   Detailed design of your parallel training pipeline.

        -   Table/graphs showing total training time and speedup for P=1,2,4,8.

        -   Analysis of scalability and communication overhead.

    -   **Part 2.4: Performance Profiling and Optimization:**

        -   Description of the profiling tool(s) used.

        -   Key profiling results (e.g., screenshots of profiler output, summarized metrics).

        -   Clear identification of bottlenecks.

        -   Proposed optimization strategies (at least two).

        -   Description of the implemented optimization(s) and their measured impact on performance.

    -   **Conclusion:** Summarize your findings and key takeaways.

    -   **References:** Cite any external libraries, papers, or resources used.

3.  **Discord Reflection:** A brief reflection (100-200 words) posted in the `#assignment1` channel on Discord. This reflection should summarize your key learning points, challenges faced, and insights gained during the assignment.

### Grading Rubric

-   **Correctness & Functionality (40%):**

    -   GCN model correctly implemented and functional on Reddit.

    -   Partitioning strategies correctly implemented and produce valid partitions.

    -   Sampling techniques correctly implemented and integrated.

    -   Parallel training pipeline correctly implemented and functional.

-   **Performance & Efficiency (30%):**

    -   Demonstrated understanding of performance bottlenecks.

    -   Effective use of profiling tools.

    -   Meaningful optimization strategies proposed and/or implemented.

    -   Clear analysis of speedup, memory, and training time.

-   **Analysis & Discussion (15%):**

    -   Clear and insightful discussion of trade-offs (partitioning, sampling).

    -   Thorough explanation of profiling results and optimization rationale.

    -   Well-structured and coherent technical report.

-   **Code Quality & Documentation (5%):**

    -   Clean, readable, and well-commented code.

    -   Comprehensive `README.md` with clear instructions.

    -   Proper use of version control.

-   **Discord Reflection (10%):**

    -   Thoughtful summary of learning points, challenges, and insights.

    -   Posted in the correct Discord channel (`#assignment1`).

### Resources and Hints

-   **GNN Libraries:**

    -   [PyTorch Geometric (PyG) Documentation](https://pytorch-geometric.readthedocs.io/en/latest/ "null")

    -   [Deep Graph Library (DGL) Documentation](https://www.google.com/search?q=https://docs.dgl.ai/ "null")

-   **Graph Partitioning:**

    -   [METIS Official Website](https://www.google.com/search?q=http://glaros.dtc.umn.edu/gkhome/metis/metis/overview "null")

    -   `PyMetis` or `metis` Python packages (check installation instructions carefully).

-   **Sampling in GNNs:**

    -   Look for `NeighborSampler` or similar classes in PyG/DGL examples for large graph training.

    -   Relevant research papers on GNN sampling (e.g., GraphSAGE \cite{graphsage_ref}, Cluster-GCN \cite{cluster_gcn_ref}).

-   **Parallel Programming in Python:**

    -   [PyTorch Distributed Overview](https://www.google.com/search?q=https://pytorch.org/tutorials/beginner/ddp_series_overview.html "null")

    -   Python's `multiprocessing` module for simpler CPU-based parallelism.

-   **HPC Profiling:**

    -   [Python's `cProfile` and `pstats` module](https://www.google.com/search?q=%5Bhttps://docs.python.org/3/library/profile.html%5D(https://docs.python.org/3/library/profile.html) "null")

    -   `perf` (Linux command-line tool, requires root/sudo for full functionality)

    -   [NVIDIA Nsight Systems](https://developer.nvidia.com/nsight-systems "null") (for GPU profiling)

    -   `htop` (interactive process viewer for Linux)

Good luck with the assignment! This will be a challenging but highly rewarding experience in scaling machine learning models for real-world applications.
