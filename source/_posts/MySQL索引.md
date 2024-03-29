---
title: MySQL索引
date: 2022-10-28 04:38:20
---
# MySQL索引

## 前缀索引

所谓前缀索引**说白了就是对文本的前几个字符建立索引**。

使用前缀索引，可能都是因为整个字段的数据量太大，没有必要针对整个字段建立索引，前缀索引仅仅是选择一个字段的部分字符作为索引，这样一方面可以节约索引空间，另一方面则可以提高索引效率，当然很明显，这种方式也会降低**索引的选择性**。

索引选择性最高为 1，如果索引选择性为 1，就是唯一索引了，搜索的时候就能直接通过搜索条件定位到具体一行记录！这个时候虽然性能最好，但是也是最费空间的，**这不符合我们创建前缀索引的初衷**。

我们一开始之所以要创建前缀索引而不是唯一索引，**就是希望能够在索引的性能和空间之间找到一个平衡**，我们希望能够选择足够长的前缀以保证较高的选择性（这样在查询的过程中就不需要扫描很多行），但是又希望索引不要太过于占用存储空间。
