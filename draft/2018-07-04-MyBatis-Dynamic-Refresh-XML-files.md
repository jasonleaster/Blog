# MyBatis 动态刷新XML文件

### 需求背景

经常遇到开发中一个“痛点”，就是MyBatis的XML文件不能动态加载，如果在debug过程中改动了XML文件，必须重新启动微服务(Java进程)，才能够使得刚由于调试原因改动的XML文件得到加载。

思路: org.mybatis.spring.SqlSessionFactoryBean做适当修改或者考虑继承该类自定义一个类去实现refresh的想法 
想在奇怪的地方就是为什么Java New 一个inputstream的时候内容始终是一样的。