# Kùzu 0.0.2 Release
This post is about the second release of Kùzu. However, since these blogs are our only
way to reach some people publicly, we want to start with a much more
important topic:

**Donate to the Victims of [Türkiye-Syria Earthquake](https://www.bbc.com/news/world-middle-east-64590946):**
Our hearts and prayers go to all the victims, those who survived and those who passed,
in Syria and Türkiye. 
There will be a very difficult winter for all those who survived so everyone needs to help. 
Here are two pointers for trustworthy organizations we know of that are trying to help
victims on the ground. For Türkiye (where Semih is from), you can donate to [Ahbap](https://ahbap.org/bagis-kategorisi/5)
(put a large number because the donation currency is in TL and 1 CAD = 14 TL; 1 USD = 19TL); and for Syria 
you can donate to the [White Helmets](https://www.whitehelmets.org/en/). Be generous! We'll leave pointers to several 
other organizations below in this footnote[^1].

## Overview of Kùzu 0.0.2
Back to the post. Kùzu codebase is changing fast but this release still has a focus: we 
have worked quite hard since the last release to integrate Kùzu to import data from
different formats and export data to different formats. There are also several important 
features in the new Cypher clauses and queries we support and additional string 
processing capabilities. We will give a summary of each of these below. The full
[release notes are here](https://github.com/kuzudb/kuzu/releases).

## Exporting Query Results to Pytorch Geometric and NetworkX
Perhaps most excitingly, we have added the first capabilities to integrate with 2 popular 
graph data science
libraries: (i) [Pytorch Geometric](https://github.com/pyg-team/pytorch_geometric) (PyG) for performing 
graph machine learning; and (ii) [NetworkX](https://networkx.org/) for a variety of 
graph analytics, including visualization. 

### Pytorch Geometric: `QueryResult.get_as_torch_geometric()` function
Our [Python API](https://kuzudb.com/docs/client-apis/python-api/overview.html) now has a 
new `QueryResult.get_as_torch_geometric()` function that 
converts results of queries to PyG's in-memory graph representation 
[`torch_geometric.data`](https://pytorch-geometric.readthedocs.io/en/latest/modules/data.html).
If your query results contains nodes and relationship objects, then the function uses 
those nodes and relationships to construct either `torch_geometric.data.Data` or 
`torch_geometric.data.HeteroData` objects. The function also auto-converts any numeric or boolean property 
on the nodes into tensors on the nodes that can be used as features in the `Data/HeteroData` objects.
Any property that cannot be auto-converted, or edge properties are also returned in case you need
want to manually put them into the `Data/HeteroData` objects.

**Colab Demonstrations**
Here are 2 Colab notebooks that you can play around with to see how you can develp graph learning
pipelines using Kùzu as your GDBMSs:
1. [Node property prediction](https://colab.research.google.com/drive/1fzcwBwTY-M19p7OOTIaynfgHFcAQo9NK?usp=sharing#scrollTo=nyXPXQ2dMesl)
2. [Link prediction](https://colab.research.google.com/drive/1QdX7CDdajIAb04lqaO5PfJlpKG-ljG28?usp=sharing#scrollTo=KIZPfDBkVJSB)

The examples demonstrate how to extract a subgraph,
train graph convolutional or neural networks (GCNs or GNNs), make some node property
or link predictions and save them back in Kùzu so you can query these predictions.

**NetworkX**


[^1]: For Türkiye two other organizations are [AFAD](https://en.afad.gov.tr/earthquake-campaign), which is the public
institute for coordinating natural disaster response and [Akut](https://www.akut.org.tr/en/donation), a volunteer-based
and highly organized search and rescue group. For Syria, another campaign I can
recommend is [Molham Team](https://molhamteam.com/en/campaigns/439?fbclid=IwAR3_t443XME9Gh0r75KM4VpQ58WLNPd8w8tyMV2JprdObwecPwhWAdX2FOQ), which is an organization founded by Syrian refugee students.
