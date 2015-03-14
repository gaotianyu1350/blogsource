title: pb_ds应用
date: 2015-02-17 09:48:40
tags: [STL,C++,pb_ds]
categories: 总结
---
<!--more-->
## 堆

### 定义
包含：`ext/pb_ds/priority_queue.hpp`
声明：`__gnu_pbds::priority_queue<T>`
模板参数：
```c++
template < typename Value_Type ,
	typename Cmp_Fn = std :: less < Value_Type >,
	typename Tag = pairing_heap_tag ,
	typename Allocator = std :: allocator <char > >
class priority_queue
```
`Value_Type`：类型
`Cmp_Fn`：自定义比较器
`Tag`：堆的类型。可以是`binary_heap_tag`（二叉堆）`binomial_heap_tag`（二项堆）`rc_binomial_heap_tag` `pairing_heap_tag`（配对堆）`thin_heap_tag`
`Allocator`：不用管

### 使用
相比`STL`中的`priority_queue`，可以
- 用`begin()`和`end()`获取迭代器从而遍历
- 删除单个元素 `void erase(point_iterator)`
- 增加或减少某一元素的值 `void modify(point_iterator, const_reference)`
- 合并 `void join(priority_queue &other)`，把`other`合并到`*this`，并把`other`清空

### 性能分析
五种操作：`push`、`pop`、`modify`、`erase`、`join`
- `pairing_heap_tag`：`push`和`join`$O(1)$，其余均摊$O(logn)$
- `binary_heap_tag`：只支持`push`和`pop`，均为均摊$O(logn)$
- `binomial_heap_tag`：`push`为均摊$O(1)$，其余为$O(logn)$
- `rc_binomial_heap_tag`：`push`为$O(1)$，其余为$O(logn)$
- `thin_heap_tag`：`push`为$O(1)$，不支持`join`，其余为$O(logn)$；但是如果只有`increase_key`，`modify`均摊$O(1)$
- `不支持`不是不能用，而是用起来很慢

经过实践检测得到的结论：
- `Dijkstra`算法中应用`pairing_heap_tag`，速度与手写数据结构相当。
- 配对堆在绝大多数情况下优于二项堆
- 只有`push`，`pop`和`join`操作时，二叉堆速度较快
- 有`modify`操作时，可以考虑`thin_heap_tag`或者配对堆，或手写数据结构。

## 树
### 定义
包含：`ext/pb_ds/assoc_container.hpp`和`ext/pb_ds/tree_policy.hpp`
声明：`__gnu_pbds::tree<Key,T>`
模板参数：
```c++
template <
	typename Key , typename Mapped ,
	typename Cmp_Fn = std :: less <Key >,
	typename Tag = rb_tree_tag ,
	template <
		typename Const_Node_Iterator ,
		typename Node_Iterator ,
		typename Cmp_Fn_ , typename Allocator_ >
	class Node_Update = null_tree_node_update ,
	typename Allocator = std :: allocator <char > >
class tree ;
```
`Tag`：`tree`的类型，可以是`rb_tree_tag`，`splay_tree_tag`，`ov_tree_tag`
`Node_Update`：可以为空，也可以用`pb_ds`自带的`tree_order_statistics_node_update`，这样这个`tree`就会获得两个函数`find_by_order`和`order_of_key`
- `iterator find_by_order(size_type order)` 找第`order+1`小的元素的迭代器，如果`order`太大会返回`end()`
- `size_type order_of_key(const_key_reference r_key)` ：询问这个`tree`中有多少比`r_key`小的元素

### 使用
与`map`使用方法基本相同，包括`begin()`，`end()`，`size()`，`empty()`，`clear()`，`find(const Key)`，
`lower_bound(const Key)`，`upper_bound(const Key)`，`erase(iterator)`，`erase(const Key)`，
`insert(const pair<Key, T>)`，`operator[]`

如果想改成`set`，只需要将第二个参数`Mapped`改为`null_type`（在`4.4.0`及以下版本的编译器中应用`null_mapped_type`）就可以了。此时迭代器指向的类型会从`pair`变成`Key`，和`set`几乎没有区别。

当然还有一些其他用法，如：
- `void join(tree &other)` 把`other`中所有元素移动到`*this`上（值域不能相交，否则抛出异常。
- `void split(const_key_reference r_key, tree &other)` 清空`other`，然后把`*this`中所有大于`r_key`的元素移动到`other`。

自定义`Node_Update`使用方法。
以下代码实现了区间求和
```c++
template < class Node_CItr , class Node_Itr ,
	class Cmp_Fn , class _Alloc	 >
struct my_node_update {
	virtual Node_CItr node_begin () const = 0;
	virtual Node_CItr node_end () const = 0;
	typedef int metadata_type ;
	inline void operator ()( Node_Itr it , Node_CItr end_it ){
		Node_Itr l = it. get_l_child (), r = it. get_r_child ();
		int left = 0, right = 0;
		if(l != end_it ) left = l. get_metadata ();
		if(r != end_it ) right = r. get_metadata ();
		const_cast < metadata_type &>( it. get_metadata ())
			= left + right + (* it)-> second ;
	}
	inline int prefix_sum (int x) {
		int ans = 0;
		Node_CItr it = node_begin ();
		while (it != node_end ()) {
			Node_CItr l = it. get_l_child (), r = it. get_r_child ();
			if( Cmp_Fn ()(x, (* it)-> first )) it = l;
			else {
				ans += (* it)-> second ;
				if(l != node_end ()) ans += l. get_metadata ();
				it = r;
			}
		}
		return ans;
	}
	inline int interval_sum (int l, int r){
		return prefix_sum (r) - prefix_sum (l - 1);
	}
}
int main() {
	tree <int , int , std :: less <int >, rb_tree_tag , my_node_update > T;
	T [2] = 100; T [3] = 1000; T [4] = 10000;
	printf ("%d\n", T. interval_sum (3, 4));
	printf ("%d\n", T. prefix_sum (3));
}
```
注意：
- 对`Node_Itr`可以做的事情：用`get_l_child`和`get_r_child`获取左右儿子，用两个星号（一个星号只是获取了`iterator`）获取节点信息，用`get_metadata`获取节点额外信息。
- `operator()`的功能是将节点`it`的信息更新，传入的`end_it`表示空节点。

### 性能分析
和手写数据结构差不多，`rb_tree_tag`要更快。

## 哈希
### 定义
包含：`ext/pb_ds/assoc_container.hpp`和`ext/pb_ds/hash_policy.hpp`
声明：
- `__gnu_pbds::cc_hash_table <Key, Mapped>`
- `__gnu_pbds::gp_hash_table <Key, Mapped>`

### 使用
支持`operator[]`和`find`

## 总结
- `priority_queue`，与`STL`相比支持了`modify`，`erase`和`join`
- `tree`，相当于`STL`中的`set/map`，还支持`split`和`join`，运用`tree_order_statistics_node_update`还支持查询`rank`和`k`小值；更可以自定义`Node_Update`来维护更多信息。
- （目前比赛环境中）`STL`没有的两种`hash_table`
- 无脑用`pb_ds`代替`std::set/map/priority_queue`不会使程序变得更慢

## 参考文献

于纪平WC2015`《C++的pb_ds库在OI中的应用》`
感谢`saffah`让我们认识到平板电视的神奇之处！