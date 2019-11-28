---
layout: post
title: Clustering using Agglomerative algorithm
categories: [python, data-mining, clustering]
tags: [pyclustering]
---

Hi! In this post, I would like to share information about one of the most popular clustering algorithms – Agglomerative. What is it? The idea of this algorithm is quite simple. At the beginning, each point of an input data is considered as a separate cluster. For example, we have an input data with 100 points, in this case we have 100 clusters. What is next? Then two the closest clusters are merged and the algorithm checks whether it is a time to stop by checking current amount of clusters. Clustering quality depends on a link that is used to find two the closest clusters. For example, the single-link clustering method is suitable for elongated clusters and common centroids might be used for well-scattered clusters with a Normal distribution.

In other word, Agglomerative algorithm is an approach how to allocate clusters, but it also consists of a method that defines which clusters should be merged, and you can choose the method. Depending on your choice, you will obtain different results. Let’s consider some of them:

- Single-link clustering method.
- Complete-link clustering method.
- Average distance between objects in clusters.
- Centroid-link clustering method.

I am going to use pyclustering library (version 0.9.2) for that purpose – Python/C++ library for data mining:

```bash
$ pip3 install pyclustering
```

But before we start to code lets answer to the following question. What should we know about an input dataset if we want to apply Agglomerative clustering algorithm? To perform cluster analysis we should know a correct number of clusters. If you do not know then Silhouette or Elbow methods can be used to find proper amount of well-scattered and well-distinguished clusters with more or less spherical form. Both are suitable in case of intension to use the centroid method.

Suppose we know the number of clusters that should be allocated. At the beginning, let’s see how to use Agglomerative algorithm for clustering using pyclustering library. The example below demonstrates clustering using Single-Link method:

```python
from pyclustering.cluster.agglomerative import agglomerative, type_link
from pyclustering.samples.definitions import FCPS_SAMPLES
from pyclustering.utils import read_sample

sample = read_sample(FCPS_SAMPLES.SAMPLE_ATOM)

n_cluster = 2  # amount of cluster to extract

algorithm = agglomerative(sample, n_cluster, type_link.SINGLE_LINK)
clusters = algorithm.process().get_clusters()

```

To demonstrate abilities of each method I am going to use `FCPS` datasets:

```python
from pyclustering.cluster import cluster_visualizer
from pyclustering.cluster.agglomerative import agglomerative, type_link
from pyclustering.samples.definitions import FCPS_SAMPLES
from pyclustering.utils import read_sample

def perform_clustering(path, n):
    sample = read_sample(path)

    sl_clst = agglomerative(sample, n, type_link.SINGLE_LINK).process().get_clusters()
    cl_clst = agglomerative(sample, n, type_link.COMPLETE_LINK).process().get_clusters()
    avg_clst = agglomerative(sample, n, type_link.AVERAGE_LINK).process().get_clusters()
    ct_clst = agglomerative(sample, n, type_link.CENTROID_LINK).process().get_clusters()

    return sample, sl_clst, cl_clst, avg_clst, ct_clst


def cluster_and_visualize(path, n_clusters, visualizer, n_canvas):
    sample, cl1, cl2, cl3, cl4 = perform_clustering(path, n_clusters)
    visualizer.append_clusters(cl1, sample, n_canvas, markersize=1)
    visualizer.append_clusters(cl2, sample, n_canvas + 1, markersize=1)
    visualizer.append_clusters(cl3, sample, n_canvas + 2, markersize=1)
    visualizer.append_clusters(cl4, sample, n_canvas + 3, markersize=1)


visualizer = cluster_visualizer(20, 4)
cluster_and_visualize(FCPS_SAMPLES.SAMPLE_LSUN, 3, visualizer, 0)
cluster_and_visualize(FCPS_SAMPLES.SAMPLE_HEPTA, 7, visualizer, 4)
cluster_and_visualize(FCPS_SAMPLES.SAMPLE_TARGET, 6, visualizer, 8)
cluster_and_visualize(FCPS_SAMPLES.SAMPLE_TWO_DIAMONDS, 2, visualizer, 12)
cluster_and_visualize(FCPS_SAMPLES.SAMPLE_CHAINLINK, 2, visualizer, 16)
visualizer.show()
```

And, here is an illustration of different methods that are used inside Agglomerative algorithm for clustering:
<p align="center">
  <img src="{{site.baseurl}}/images/post/2019-11-28/fcps_various_methods.png">
</p>
<p align="center">
	<b>Fig. 1.</b> Agglomerative clustering using various methods.
</p>

As you can see, single-link is much more suitable for elongated clusters.

By default pyclustering library uses C++ implementation of Agglomerative algorithm to get maximum performance. In addition, it means that you can use this library in your C++ project, just make sure that C++ compiler supports C++14 standard on your target machine. This is an example how you can use C++ version of the algorithm. I was using static linking and pyclustering headers to that.

```cpp
#include <pyclustering/cluster/agglomerative.hpp>

#include <iostream>
#include <string>

using namespace pyclustering;
using namespace pyclustering::clst;

dataset read_data(const std::string & filename) {
    dataset sample;

    // ... read sample from file ...

    return sample;
}

int main() {
    dataset data = read_data("simple1.txt");

    agglomerative_data result;
    agglomerative(2, type_link::SINGLE_LINK).process(data, result);

    for (auto group : result.clusters()) {
        std::cout << "[ ";
        for (auto index : group) { std::cout << index << " "; }
        std::cout << "]";
    }

    return 0;
}
```

I think it is important to say few words about the disadvantage of this algorithm. The biggest disadvantage is performance – complexity of the algorithm O(N3) and O(N2 logN) depending on a method. Thus, this algorithm is not suitable for big data. But it is quite “good” for a relatively small data.

Enjoy Agglomerative clustering algorithm.
