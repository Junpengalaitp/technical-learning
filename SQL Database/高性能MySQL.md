# 第五章
## 5.1 索引基础
### B-Tree(B+ Tree)索引
* 可以使用B-Tree索引的查询类
	* 全值匹配
	* 匹配最左前缀
	* 匹配列前缀：只匹配第一一列的值的开头部分(A -> Allen)
	* 匹配范围值
	* 精确匹配第一列并范围匹配另外一列
	* 只访问索引的查询(covering index)
	* 因为索引是有序存储的，ORDER BY可以和同样的方式使用索引
* B-Tree索引的限制
	* 如果不按照索引最左列开始查找，无法使用索引
	* 不能跳过索引中的列
	* 如果有某个列的范围查询，其右边的所有列都无法使用索引优化查找
* 全文索引
	* 查找文本中的关键词，而不是直接比较索引中的值

## 5.2 索引优点
* 大大减少了服务器需要扫描的数据量
* 索引可以帮助服务器避免排序和临时表
* 索引可以将随机IO变为顺序IO

### 索引是最佳方案吗
* 只有索引带来的好处大于它的开销时，索引才是有效的。
* 对于特别小的表，大部分情况全表扫描更高效。
* 对于中大型表，索引特别有效。
* 对于特大型表，考虑使用分区技术。
* 如果表的数量特别多，可以简历一个元数据信息表，用来查询需要用到的某些特性

## 5.3 高性能索引策略
### 5.3.1 独立的列
* 索引列不能是表达式的一部分
	* SELECT A FROM table WHERE A + 1 = 5;

### 5.3.2 前缀索引和索引选择性
* 选择足够长的前缀以保证较高得到选择性，同时又不能太长。前缀的基数应该接近于完整列的基数
* 计算完整列的选择性
  * SELECT COUNT(DISTINCT city)/COUNT(*) FROM sakila.city_demo;
* 查询不用前缀长度进行计算
  * SELECT COUNT(DISTINCT LEFT(city, 3))/COUNT(*) AS sel3 FROM sakila.city_demo;

### 5.3.3 多列索引
* 给每个列单独建索引是错误的，在查询中同时使用的情况下没有哪个单列索引是非常有效的。
* 当出现多个索引做相交(AND)操作时，通常意味着需要一个包含所有相关列的多列索引，而不是多个独立的单列索引。
* 当出现多个索引做联合(OR)操作时，通常需要耗费大量的CPU资源在算法的缓存、排序和合并操作上。

### 5.3.4 选择合适的索引列顺序
* 正确的顺序依赖于使用该索引的查询，并且同时需要考虑如何更好地满足排序分组的需要。
* 将选择性最高的列放到索引最前列，当不考虑排序和分组时，是很好的选择，只作用于优化WHERE条件的查找。

### 5.3.5 聚簇索引
* 并不是一种单独的索引类型，而是一种数据存储方式。
* 优点
  * 可以把相关属性保存在一起。例如实现电子邮箱时，可以根据用户ID来聚集数据，这样只需要从磁盘读取少数的数据也就能获取某个用户的全部邮件。
  * 数据访问更快。将索引和数据保存在同一个B-Tree中，因此从聚簇索引中获取的数据通常比在非聚簇索引中查找要快。
  * 使用覆盖索引扫描的查询可以直接使用页节点中主键值。

### 5.3.6 覆盖索引
* 包含所有需要查询字段的索引
* 覆盖索引的优点
  * 索引条目通常远远小于数据行大小, 如果只需要读取索引，会极大地减少访问数据量
  * 索引是顺序存储的
  * 一些存储引擎如MyISAM在内存中只缓存索引，数据缓存依赖于操作系统，因此访问数据需要一次系统调用。这可能会导致严重的性能问题。
  * 由于InnoDB的聚簇索引，覆盖索引对InnoDB表特别有用，它的二级索引在叶节点中保存了行的主键值，所有如果二级主键能够覆盖查询，则可以避免对主键索引的二次查询。
* MySQL只能使用B-Tree索引做覆盖索引

### 5.3.7 使用索引扫描来做排序
* MySQL两种生成有序的结果的方式
  * 通过排序操作
  * 通过按索引顺序扫描
* MySQL可以使用同一个索引既满足排序，又用于查找行，是最优设计
  * 当索引顺序的列顺序和ORDER BY子句的顺序完全一致，并且所有列的排序方向都一致时
  * 关联多张表时，只有ORDER BY子句引用的字段全部为第一个表时，才能使用索引做排序

### 5.3.8 压缩（前缀压缩）索引
* MyISAM使用前缀压缩来减少索引的大小，从而让更多的索引可以放入内存中。

### 5.3.9 冗余和重复索引
* 按照相同的顺序创建相同类型的索引，应该避免这样创建重复索引。
* 如果创建了索引(A,B)，再创建索引(A)就是冗余索引
* 如果已经有了索引(A), 需要索引(A,B)时，扩展索引(A)而不是新建索引(A,B)

### 5.3.10 未使用的索引
* 完全累赘，删除它们

### 5.3.11 索引和锁
* 使用索引可以查询更少的行，那么就会锁定更少的行

## 5.4 索引案例
### 5.4.1 支持多种过滤条件
* 当设计索引时，不要只为现有的查询考虑需要哪些索引，还需要考虑对查询的优化。如果发现某些查询需要创建新索引，但是这个索引又会降低另一些查询的效率，那么应该想一下是否能优化原来的查询。应该同时优化查询和索引以找到最佳的平衡。
* 将范围索引放在索引的最后面。

### 5.4.2 避免多个范围条件
### 5.4.3 优化排序
* 对于选择性非常低的列，可以增加一些特殊的索引来做排序
* 使用延时关联，通过覆盖索引查询返回需要的主键，再根据主键关联原表获得需要的行。

## 5.6 总结
* 单行访问很耗时，如果服务器从存储中读取一个数据块只是为了获取其中一行，那么就浪费了很多工作。最好能从读取的块中包含尽可能多的所需行，最小化块访问数量。使用索引可以创建位置引用以提升效率。
* 按顺序访问很快，一是可以二分搜索，减少访问块的数量，而是服务器无需额外的排序操作
* 覆盖索引查询是很快的，如果一个索引包含了查询需要的所有的列，就不需要再去表里查找，避免了大量的单行访问。
* 尽可能地使用数据原生顺序从而避免额外的排序操作，并尽可能使用索引覆盖查询。