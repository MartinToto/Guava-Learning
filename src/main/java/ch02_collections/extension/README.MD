# 集合扩展工具类

### 1. Forwarding装饰器
通过创建ForwardingXXX的子类并实现delegate()方法，可以选择性地覆盖子类的方法来增加装饰功能，而不需要自己委托每个方法
```java
// 在元素被添加到列表时增加特定的行为
AddLoggingList<Integer> loggingList = new AddLoggingList<>();
loggingList.add(1);

/**
 * Forwarding装饰器
 * 通过创建ForwardingXXX的子类并实现delegate()方法，可以选择性地覆盖子类的方法来增加装饰功能，而不需要自己委托每个方法
 * @author
 * @create 2018-05-07 下午11:33
 **/
class AddLoggingList<E> extends ForwardingList<E> {
    final List<E> delegate = Lists.newArrayList(); // backing list
    @Override
    protected List<E> delegate() {
        return delegate;
    }
    @Override
    public void add(int index, E elem) {
        log(index, elem);
        super.add(index, elem);
    }
    @Override
    public boolean add(E elem) {
        return standardAdd(elem); // implements in terms of add(int, E)
    }
    @Override
    public boolean addAll(Collection<? extends E> c) {
        return standardAddAll(c); // implements in terms of add
    }

    private void log(int index, E elem) {
        System.out.println(String.format("add %s in %d", elem, index));
    }
}
```


### 2. PeekingIterator
Iterators提供一个Iterators.peekingIterator(Iterator)方法，来把Iterator包装为PeekingIterator，这是Iterator的子类，它能让你事先窥视[peek()]到下一次调用next()返回的元素。

注意：Iterators.peekingIterator返回的PeekingIterator不支持在peek()操作之后调用remove()方法
```java

// 举个例子：复制一个List，并去除连续的重复元素
List<Integer> numbers = Ints.asList(1, 1, 2, 3, 3);
List<Integer> results = Lists.newArrayList();

PeekingIterator<Integer> iterator = Iterators.peekingIterator(numbers.iterator());
while (iterator.hasNext()) {
    Integer current = iterator.next();
    while (iterator.hasNext() && iterator.peek().equals(current)) {
        // skip this duplicate element
        iterator.next();
    }
    results.add(current);
}
// [1, 2, 3]
System.out.println(results);
```



### 3. AbstractIterator
实现自己的Iterator, 例如包装一个iterator跳过长度为1的字符串
```java
Iterator<String> skipNulls2 = skipSingle(Lists.newArrayList("aa", "b", "cc").iterator());
// aa cc
while (skipNulls2.hasNext()) {
    System.out.print(skipNulls2.next() + " ");
}

// 包装一个iterator跳过长度为1的字符串
public static Iterator<String> skipSingle(final Iterator<String> in) {
    return new AbstractIterator<String>() {
        protected String computeNext() {
            while (in.hasNext()) {
                String s = in.next();
                if (s.length() > 1) {
                    return s;
                }
            }
            return endOfData();
        }
    };
}
```