---
layout: post
title: 2018-9-12-Iterable,Collection及List源码分析
date:   2018-09-13 15:14:54
categories: Java
tags: List,Collection,Iterable，源码
---
注:本文使用的编码环境是JDK1.8
## Iterable源码分析


```
//List接口继承自Collection接口，而Collection接口继承自Iterable接口
public interface List<E> extends Collection<E> {
```
首先看Iterable接口中的方法
```
public interface Iterable<T> {
//iterator方法返回一个Iterator对象，范围默认为default，表示同类，同包。
Iterator<T> iterator();
```

```
//接口中使用default修饰方法，JDK1.8新特性，Consumer也是一个接口，其中只是定义了accept方法，并未实现该方法
default void forEach(Consumer<? super T> action) {
        //Objects为final类，requireNonNull是静态方法，验证是否为空，为空则抛出空指针异常
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

```
//Spliterators是一个静态类，spliteratorUnknownSize方法返回一个spliterator类
default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
```
spliterator类是一个并行遍历迭代器，是JDK1.8新增特性，在此不研究其源码

## Collection源码分析

```
//Collection继承自Iterable
public interface Collection<E> extends Iterable<E> {
    //返回大小数据
    int size();
    //若集合为空，返回true
    boolean isEmpty();
    //如果集合包含指定对象，返回true
    boolean contains(Object o);
    //返回一个迭代器
    Iterator<E> iterator();
    //返回一个包含collection中所有元素的数组
    Object[] toArray();
    //返回一个指定类型的包含collection中所有元素的数组
    <T> T[] toArray(T[] a);
    //添加元素，添加成功返回true
    boolean add(E e);
    //移除某个元素
    boolean remove(Object o);
    //是否包含另一个collection的所有元素
    boolean containsAll(Collection<?> c);
    //添加另一个collection的所有元素，要求两个collection的元素不冲突
    boolean addAll(Collection<? extends E> c);
    //移除另一个collection包含的所有元素
    boolean removeAll(Collection<?> c);
    //是否包含在另一个collection中
    boolean retainAll(Collection<?> c);
    //清除所有的元素
    void clear();
    //是否相等
    boolean equals(Object o);
    //返回hashCode
    int hashCode();
```
上面都是接口的抽象方法，接下来是接口的默认方法

```
//Predicate是一个函数式编程接口，需要实现test方法
default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
```
该方法通过传入一个Predicate接口的实现类，用来移除某些符合要求的元素
顺便提一句函数式接口@functionalInterface注解要求接口只有一个抽象方法，而默认方法和静态方法都不算抽象方法

```
//重写Iterable接口的spliterator方法
@Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }
    //增加了新方法，返回一个流
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
    //新增方法，返回一个并发流
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
```
## List源码分析
接下来看看List源码，因为与Collection接口有部分抽象方法相同，故只记录不同的方法及新增的方法。

```
public interface List<E> extends Collection<E> {
    //第一个参数指定插入位置，在插入位置右边依次插入元素
    boolean addAll(int index, Collection<? extends E> c);
    //获取指定位置的元素
    E get(int index);
    //将指定位置的元素替换，并返回以前在此位置的元素
    E set(int index, E element);
    //插入单个元素，第一个参数指定插入位置
    void add(int index, E element);
    //移除指定位置的元素，并返回该元素
    E remove(int index);
    //返回该元素在集合中第一次出现的位置
    int indexOf(Object o);
    //返回该元素在集合中最后一次出现的位置
    int lastIndexOf(Object o);
    //返回一个ListIterator对象
    ListIterator<E> listIterator();
    //返回从某一位置开始的一个ListIterator对象
    ListIterator<E> listIterator(int index);
    //从原集合截取的一部分，并以List返回
    List<E> subList(int fromIndex, int toIndex);
```    
接下来是默认方法
```
//传入一个Comparator对象
default void sort(Comparator<? super E> c) {
        //调用toArray方法，返回一个数组，元素为Object
        Object[] a = this.toArray();
        调用Arrays类的sort静态方法
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            //第一次执行指向第一个元素
            i.next();
            //替换元素
            i.set((E) e);
        }
    }
//重写Collection接口的spliterator方法
@Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.ORDERED);
    }

default void replaceAll(UnaryOperator<E> operator) {
        //检测是否为null
        Objects.requireNonNull(operator);
        //ListIterator是不可变的
        final ListIterator<E> li = this.listIterator();
        while (li.hasNext()) {
            li.set(//替换当前指向的元素
            operator.apply(//执行apply方法
            li.next()//获取当前指向的元素，并指向下一个元素
            ));
        }
    }
```
