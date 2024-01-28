#### ArrayList

##### 初始容量

###### 无参构造，使用长度为0的数组

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

###### 有参构造

- 参数为initialCapacity，使用指定initialCapacity长度的数组
  
  ```java
  public ArrayList(int initialCapacity) {
          if (initialCapacity > 0) {
              this.elementData = new Object[initialCapacity];
          } else if (initialCapacity == 0) {
              this.elementData = EMPTY_ELEMENTDATA;
          } else {
              throw new IllegalArgumentException("Illegal Capacity: "+
                                                 initialCapacity);
          }
      }
  ```

- 参数为集合，使用集合c的大小作为长度的数组
  
  ```java
  public ArrayList(Collection<? extends E> c) {
          elementData = c.toArray();
          if ((size = elementData.length) != 0) {
              // c.toArray might (incorrectly) not return Object[] (see 6260652)
              if (elementData.getClass() != Object[].class)
                  elementData = Arrays.copyOf(elementData, size, Object[].class);
          } else {
              // replace with empty array.
              this.elementData = EMPTY_ELEMENTDATA;
          }
      }
  ```

##### 扩容机制

###### 核心：创建一个容量更多的新数组，将老数组元素复制到新数组中

###### 步骤：

- **add(E e)，首次扩容为10，再次扩容为oldCapacity + (oldCapacity >> 1)，大约为上次容量的1.5倍**

- **addAll(Collection<? extends E> c)，首次扩容为Math.max(10,c.length)，有元素时为Math.max(oldCapacity + (oldCapacity >> 1),oldCapacity+c.length)**

##### 迭代器（fail-fast与fail-safe）

- **ArrayList是fail-fast的典型代表，遍历的同时不能修改，尽快失败**
  
  - 构造迭代器时，会将ArrayList的modCount值赋值给迭代器的expectedModCount，若另一线程更改ArrayList变更modCount，造成`modCount != expectedModCount`抛出异常
  
  ```java
  final void checkForComodification() {
              if (modCount != expectedModCount)
                  throw new ConcurrentModificationException();
          }
  ```

- **CopyOnWriteArrayList是fail-safe的典型代表，遍历的同时可以修改，原理是读写分离，两个数组一个读一个写**
  
  - 构造迭代器时，会将列表复制一份，赋值给snapshot，遍历查询时使用的都是snapshot中的元素
  
  ```java
  private COWIterator(Object[] elements, int initialCursor) {
              cursor = initialCursor;
              snapshot = elements;
          }
  ```
  
  - 另一线程操作CopyOnWriteArrayList时，同样是复制一份新的数组进行操作，然后再将新数组赋值回去
    
    ```java
    public boolean add(E e) {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                Object[] elements = getArray();
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len + 1);
                newElements[len] = e;
                setArray(newElements);
                return true;
            } finally {
                lock.unlock();
            }
        }
    ```
  
  - 通过上述复制新数组的方式，实现fail-safe
