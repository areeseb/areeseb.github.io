---
layout: post
author: Alexander Brown
---

**Map Matching Polylines and Polygons Using a Geometric Heuristic Algorithm**

This little blog post will provide a explanation and summary of work I have been doing on implementing a map matching algorithm for varied (spatial) vector data. 

#### Background
___

Broadly, Map Matching is a problem common in the spatial sciences which involves relating one set of geometry to another reference set. Most commonly this takes the form of associating a GPS trace with a reference road network. However, many others variations to the problem exist (sometimes with different names), for example, deriving a reference Polyline from a set of different GPS traces, matching and merging separate Polygon datasets, or even matching extracted feature polygons (buildings) from remote sensing imagery. 

![map_matching_wiki](images/Map_Matching_Example_with_GraphHopper.png)

_By OpenStreetMap and contributors - openstreetmap.org, CC BY-SA 2.0_

Map Matching is very much an open problem and the appropriate solutions are dictated by on a number of factors such as data format/quality/scale, computational resources, or time constraints (realtime/static). 

The majority of approaches can be divided into very broad categories:

* **Geometric Solutions**

    Geometric solutions typically relate features based on the similarity of their geometry (e.g. the distances points in a GPS trace to a reference road). Geometric methods are often used when the input format is varied or doesn't conform to the traditional GPS trace/network configuration. 

* **Hidden Markov Models**

    Hidden Markov Models are often used for the GPS trace/road network format. In this case, the sequence of points in a GPS trace are the observations and road segments in the network are the hidden states. The matched series of road segments in the network is the set of states with the most likely transitions (series of road segments) based on the observations (gps trace).

For a few good examples of modern map matching see:

Saki, S., Hagen, T. A Practical Guide to an Open-Source Map-Matching Approach for Big GPS Data. SN COMPUT. SCI. 3, 415 (2022). https://doi.org/10.1007/s42979-022-01340-5

Map Matching at Uber.  
https://www.youtube.com/watch?v=ChtumoDfZXI

Douriez, Marie. A New Real-Time Map-Matching Algorithm at Lyft. https://eng.lyft.com/a-new-real-time-map-matching-algorithm-at-lyft-da593ab7b006

I elected to explore **geometric solutions** for a few reasons:
* Less sensitive to input data format and quality (i.e. I don't have a reference network to work off of)
* Computational cost is less of an issue given that I am working off with (relatively) small data sets. 

The two common traditional geometric similarity measures for curves/lines that are readily available in Python are **Hausdorff** and **Frechet** distance.

**Hausdorf Distance** is defined intuitively as the greatest of all distances from a point in one set, to it's closest point in another set.

The mathematical definition is roughly, 

_Let P and Q be two point sets in some metric space._

_The directed Hausdorff distance from P to Q, denoted by_ $h(P, Q)$, is max $p∈P$ _min_ $q∈Q ||p − q||$.

_(In other words, the max distance from a point in P to it's closest point in Q)_

_The Hausdorff distance between P and Q, is the symmetric max_ {$h(P, Q), h(Q, P)$}.

_(i.e the maximum of all distances for a point in one set to the closest point in another)_

Hausdorff distance has many applications but is often used in the spatial sciences to compare similarity of Polygons.

In my case, the Hausdorff distance between two Polylines, would be the max distance one can travel between vertices in one line, to the closest vertex in another line. So if two linestrings represent the same road/trail on the ground, they should have a small Hausdorff distance. 

Here's a kinda silly video showing an optimized Hausdorff distance being used to compare 3D renderings of animals. 

https://www.youtube.com/watch?v=R7WZFTnir_k

**Frechet Distane** has some similarity to Hausdorff distance but instead, it takes the order of points into account. This makes it often more apt for comparing the similarity of polylines.

One common way to understand Frechet distance is to imagine two separate paths (polylines), with a person on one path, and a dog on the other. The person and dog are connected via a leash, and can walk forward on their respective paths but not backward. They can also walk at any speed they desire and the leash can stretch to any arbitrary distance. The Frechet distance is the minimum length of leash that can be used for the dog and person to walk their paths.

The mathematical definition of Frechet distance is a bit more complex but for those interested, here is a great video explaining both the intuitive and the mathematical definition: 

https://www.youtube.com/watch?v=TJeeZFNXi9M&t=139s


#### Problem
___

My specific problem is to ingest and integrate medium (~100k features) Polyline and Polygon datasets into an existing spatial database where there is significant overlap. This means that features in input data that represent the same object as a feature in the reference data set need to be dropped or merged.

However, the quality of input data is very suspect, so there is essentially no guarentee of any shared attribute data or identical geometry. Additionally, the reference data set is not a proper network in a GIS sense. There is no topology and no other features typical of networked data (i.e. direction, junctions/nodes, elevation, etc). So all there is to go off is the similarity of the geometry. 
Finally, the input data and spatial tables are large enough that it procludes any kind of manual inspection.

I should also note that I need to complete a similar task for both Polylines and Polygons.

Given these constraints, the best I can hope for is a kind of filter that can find the obvious cases of where input Polyline or Polygon geometry is identical or very similar to the reference data. Once I get the obvious cases, the rest of the close(ish) matches and be identifed and corrected over time.

Here's an example of two polylines that'd need to be merged or dropped: 
![line_merge](images/easy_example.png)

Here's an example of two polygons that'd need to be merged or dropped:
![polygon_merge](images/easy_polygon.png)

#### Solution

At first glance, it seems like the solution would just be to go to every input polyline/polygon, see if there are any reference polyglines/polygons that have a very low Hausdorff/Frechet distance and then drop them. The remaining polylines/polygons would be integrated or merged into the reference dataset. 

Unfortunately, there are a number of problems with this simple solution. For simplicity's sake, I'll divide the problems into three categories,

1. Issues with distance metrics (Hausdorff/Frechet distance).

:   The main poblem with Hausdorff and Frechet distance is that they don't work well with messy geometry and have many edge cases where they all apart.





2. Data issues.
3. Computational limitations.









