# what is elasticsearch

Elasticsearch is a highly scalable open-source full-text search and analytics engine. It allows you to store, search and analyze big volumes of data quickly and in near real time.

# Basic Concepts

* Near Realtime (NRT)
	* Elasticsearch is a near real time search platform, which means is there is slight latency from the time you index a document until the time it becomes searchable.
* Cluster
	* A cluster is a collection of one or more nodes(servers) that together holds your entire data and provides federated indexing and search capabilities across all nodes. A cluster is identified by a unique name which by default is "elasticsearch". This ame is important because a node can only be part of a cluster if the node is set up to join the cluster by its name.
	Make sure you don't resue the same cluster names in different environments, otherwise you might end up with nodes joining the wrong cluster.
	It's valid and perfectly fine to have a cluster with only a single node in it. Furthermore, you may also have multiple independent clusters each with its own unique cluster name.
* Node
	* A node is a single server that is part of your cluster, stores your data and participates in the cluster's indexing and search capabilities. Just like a cluster, a node is identified by a name which by default is a random Marvel character anme that is assigned to the node at startup. You can define any node name you want if you do not want the default. This name is important for administration purposes where you want to identify which servers in your network correspond to which nodes in your elasticsearch cluster.
	A node can be configured to join a specific cluster by the cluster name, by default, each node is set up to join cluster named `elasticsearch` which means that if you start up a number of nodes on your network and assuming they can discover each other.
* Index
	* An index is a collection of documents that have somewhat similiar characteristics. For example, you can have an index for customer data, another index for a product catalog and yet another index for order data.
	An index is identified by a name(all lowercase) and this name is used to refer to the index when performing indexing, search, update and delete operations against the documents in it.
* Type
	* Within an index, you can define one or more types. A type is a logical category/partition of your index whose semantics is completely up to you.
	In general, a type is defined for documents that have a set of common fields.
* Document
	* A document is a basic unit of information that can be indexed. For example, you can have a document for a single customer, another document for a single product, and another for a single order. This document is expressed in JSON which is an ubiquitous internet data interchange format.
	Within an index/type, you can store as many documents as you want. Note that although a document physically resides in an index, a document actually must be indexed/assigned to a type inside an index.
* Shards & Replicas
	* An index can potentially store a large amount of data that can exceed the hardware limits of a single node. For example, a single index of a billion documents taking up 1TB of disk space may not fit on the disk of a single node or may be too slow to serve request from a single node alone.
	To solve this problem, elasticsearch provides the ability to subdivide your index into multiple pieces called `shards`. When you create an index, you can simply define the number of shards that you want. Each shard is in itself a fully-functional and independent `index` that can be hosted on any node in the cluster.

	Sharding is important for two primary reasons:

		* It allows you to horizontally split/scale your content volume.
		* It allows you to distribute and parallelize operations across shards thus increasing performance.
	The mechanics of how a shard is distributed and also how its documents are aggregated back into search requests are completely managed by Elasticsearch and is transparent to you as the user.
	In a network/cloud environment where failures can be expected anytime, it is very useful and highly recommended to have a failover machanism in case a shard/node somehow goes offline or disappears for whatever reason. To this end, elasticsearch allows you to make one or more copies of your index's shard into what are called `replica shards` or `replicas` for short.

	Replications is important for two primary reasons:
	
		* It provides high availability in case a shard/node failes.
		* It allows you to scale out your search volume since searches can be excuted on all replicas in parallel.

To summarize, each index can be split into multiple shards. An index can also be replicated zero or more times. Once replicated, each index will have primary shards and replica shards. The number of shards and replicas can be defined per index at the time the index is created.
After the index is created, you may change the nuber of replicas dynamically antytime but you cannot change the number shards after-the-fact.

By default, each idnex in elasticsearch is allocated 5 primary shards and 1 replica which means that if you have at least two nodes in your cluster, your index will have 5 primary shards and another 5 replica shards for a total of 10 shards per index.





























