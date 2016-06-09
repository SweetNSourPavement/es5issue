These scripts demonstrate some issues with Elasticsearch 5.0.0 alpha3.

All the tests create a local cluster, creates an index with 2 replicas,
shuts down one data node, inserts a document, and tries to restart the
cluster.

The script `loss` has only 1 for `discovery.zen.minimum_master_nodes`,
but the destruction of correct replica data is unexpected.

The script `progress` uses separate master nodes, and sets
`discovery.zen.minimum_master_nodes` to 2.

Running `progress red` restarts only 2 data nodes, but the index state
remains in red. The individual nodes seems to retain their data.

`progress yellow` runs quite similar to `red` but chooses different
nodes to shutdown and restart, which results in a yellow index state.
The document won't get inserted in the replica that missed the original
transaction.

`progress green` is almost identical to `yellow` but after reaching
yellow state it starts the last data node too. This causes the index to
become green, and all the nodes will contain the document.

To sum up:

* unfortunate order of stops and starts can cause data loss
* unfortunate order of stops and starts can cause unavailability
* earlier replica healing may be desirable

To run the scripts:

* clone this repository
  * `git clone https://github.com/SweetNSourPavement/es5issue`
* copy one of the alpha tarball into the root of the repo 
  *  `wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/5.0.0-alpha3/elasticsearch-5.0.0-alpha3.tar.gz`
* run one of the following commands
  * ./loss
    * completely loses the document, overwrites correct replicas
  * ./progress red
    * the cluster never makes progress after restart, index remains in
      red indefinitely
  * ./progress yellow
  * ./progress green
