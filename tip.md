* TreeSet存放的对象必须是可比较的,也就是必须实现Comparable。

* 偏，轻，重锁，分别解决三个问题，只有一个线程进入临界区，多个线程交替进入临界区，多线程同时进入临界区。[不错的一篇文章](https://my.oschina.net/u/140462/blog/490895)

* 公平锁可以减少“饥饿”情况的发生，非公平锁性能要优于公平锁。

* archetypeCatalog表示插件使用的archetype元数据，不加这个参数时默认为remote，local，即中央仓库archetype元数据，由于中央仓库的archetype太多了，所以导致很慢，指定internal来表示仅使用内部元数据。

* Always use <property name="ignoreUnresolvablePlaceholders" value="true"/> with PropertyPlaceholderConfigurer, as you never know when and how other modules use your modules. They may have their own PropertyPlaceholderConfigurer as well, so you can break their code unknowingly, leaving you scratching your head.

  ​

* ```xml
  <!-- 一定要加单引号 -->
  <property name="name" value="#{'${test-spel}' eq 'zz' ? 'james' : 'kobe'}"/>
  ```


