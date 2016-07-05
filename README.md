These scripts demonstrate some issues with Elasticsearch 5.0.0 alpha4.

Relevant issues are
[18799](http://github.com/elastic/elasticsearch/issues/18799)
and
[18919](http://github.com/elastic/elasticsearch/issues/18919).

All the tests create a local cluster, creates an index with 2 replicas,
shuts down one data node, inserts a document, and tries to restart the
cluster. The main difference in this tests are the order of node
restarts.

The script `loss` sets `discovery.zen.minimum_master_nodes` to only 1,
and after inserting the document to the still present quorum stops all
nodes, and then starts all nodes again, but the first node to start is
the one not present at the quorum write. After all nodes started the
state of the index will be green, but the index won't contain the
document, and even the replicas that accepted the write are overwritten
by the empty one.

The script `progress` uses separate master nodes, and sets
`discovery.zen.minimum_master_nodes` to 2. The master nodes are not
restarted, only the data nodes, and once again the nodes brought back
after the shutdown form a different quorum than the one which accepted
the write.

Running `progress red` restarts only 2 data nodes, but the index state
remains in red. The individual nodes seems to retain their data.

`progress yellow` runs quite similar to `red` but chooses different
nodes to shutdown and restart, which results in a yellow index state.
The document won't get inserted in the replica that missed the original
transaction, and the original acceptor replicas remain unchanged.

`progress green` is almost identical to `yellow` but after reaching
yellow state it starts the last data node too. This causes the index to
become green, and all the nodes will contain the document.

To sum up:

* unfortunate order of stops and starts can cause data loss, but the
  connection between zen and shard quorums is unclear
* unfortunate order of stops and starts can cause unavailability, but
  there's no data loss
* earlier replica healing may be desirable

To run the scripts (on *nix):

* clone this repository
  * `git clone https://github.com/SweetNSourPavement/es5issue`
* copy one of the alpha tarball into the root of the repo 
  *  `wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/5.0.0-alpha4/elasticsearch-5.0.0-alpha4.tar.gz`
* run one of the following commands
  * ./loss
    * completely loses the document, overwrites correct replicas
  * ./progress red
    * the cluster never makes progress after restart, index remains in
      red indefinitely
  * ./progress yellow
    * index gets yellow, lagging replica won't be updated
  * ./progress green
    * index becomes green, all replicas will contain the document
