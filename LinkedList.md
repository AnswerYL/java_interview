#### LinkedList

##### ArrayList vs LinkedList（重点）

- **ArrayList**
  
  - 基于数组需要连续的内存
  
  - 随机访问快（根据下标访问）
  
  - 尾部插入、删除性能好，其他部分插入、删除都会移动大量数据，因此性能降低
  
  - 可以利用CPU缓存，局部性原理（CPU缓存读取数据时会将内存上相邻地址的内容一起读入，ArrayList能够很好的满足，更能提高性能）

- **LinkedList**
  
  - 基于双向链表，无需连续内存
  
  - 随机访问慢（要沿着链表遍历）
  
  - 头尾部插入、删除性能好
  
  - 占用内存多