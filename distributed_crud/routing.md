### 将文档路由到从库中

When you index a document, it is stored on a single primary shard. How does
Elasticsearch know which shard a document belongs to?  When we create a new
document, how does it know whether it should store that document on shard 1 or
shard 2?

The process can't be random, since we may need to retrieve the document in the
future. In fact, it is determined by a very simple formula:

    shard = hash(routing) % number_of_primary_shards

The `routing` value is an arbitrary string, which defaults to the document's
`_id` but can also be set to a custom value. This `routing` string is passed
through a hashing function to generate a number, which is divided by the
number of primary shards in the index to return the _remainder_. The remainder
will always be in the range `0` to `number_of_primary_shards - 1`, and gives
us the number of the shard where a particular document lives.

This explains why the number of primary shards can only be set when an index
is created and never changed:  if the number of primary shards ever changed in
the future, all previous routing values would be invalid and documents would
never be found.

All document APIs (`get`, `index`, `delete`, `bulk`, `update` and `mget`)
accept a `routing` parameter that can be used to customize the document-to-
shard mapping. A custom routing value could be used to ensure that all related
documents -- for instance all the documents belonging to the same user -- are
stored on the same shard. We discuss in detail why you may want to do this in
<<scale>>.


    当你索引一个文档，它是被存储在一个单独的主分片中，es怎么知道一个文档属于哪个分片呢？当我们创建一个文档，es怎么知道应该把文档存在分片1还是分片2呢？这小子咋这么聪明呢？
    这个过程其实不是随机的，当我们需要检索一个文档的时候，实际上这个过程是由一个非常简单的公式决定的
 shard = hash(routing) % number_of_primary_shards
    这个"routing" 是一个“比较随机”的数，默认是这个文档的id，但也可以是个自定义的数，这个"routing" 通过一个哈希函数生成一串数字，然后这串数字对索引中主分片的总数进行取余操作获得了最后的存放文档的分片位置。文档存放的分片位置一定位于0-主分片总数之间（念过书的都知道），当我们需要使用一个文档的时候根据这个公式可以获取所在主分片的位置。
    这说明了为啥主分片只能在索引创建的时候进行设置并且不能改变。如果将来主分片的数量改变了，先前所有的路由将不可用文档也将不能找到。
    所有的API操作加上"routing"参数可以定制文档到分片的映射，定制“routing”设置主要用于相关的文档中，例如所有文档属于同一个用户，并被存储在同一主分片中，我们将详细讨论我们为什么要这样做。。。
