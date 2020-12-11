---
title: mybatis学习笔记
tags:
  - NOTE
categories:
  - NOTE
date: 2019-10-28 17:43:17
---



# MyBatis框架架构图

<img src="../../image/326517643.png" alt="img" style="zoom:50%;" />





参考

- [MyBatis的工作原理以及核心流程介绍]: http://www.mybatis.cn/archives/706.html
- [mybatis原理，配置介绍及源码分析]: https://juejin.im/post/5bd02b3d6fb9a05cee1e2478

- [mybatis官网]: https://mybatis.org/mybatis-3/zh/configuration.html



# MyBatis Generator

【**不纠结了哈，用到再说**】

增加插件，po带comment注释

1. 初心是实现一个xml形式的最佳实践的generator，需求包括：

   1. 增 insert/ batchInsert/
   2. 删 deleteById
   3. 改 update-本质上是selective的update/
   4. 查 findByParam/ listByParam/ findById/ pageByParam/ countByParam

2. 实践发现，mybatis-dynamic-sql基本可以实现上述需求，问题在于学习成本和新功能开发（继续使用dynamic，还是xml？）

   疑问点：

   1. xxStatementProvider的作用？为什么这么设计？——相当于构造SQL的builder
   2. xxDSLCompleter同上
   3. `UpdateDSL<UpdateModel>`同上

3. 实现对比：

   1. 增  

       ```java
      // 1
      int insert(InsertStatementProvider<Pet> insertStatement);
      int insertMultiple(MultiRowInsertStatementProvider<Pet> multipleInsertStatement);
      // 2的这个接口有default实现，本质上转换成了1的InsertStatementProvider在执行
      default int insert(Pet record){}
      default int insertMultiple(Collection<Pet> records){}
      default int insertSelective(Pet record){}
       ```

   2. 删

      ```java
      int delete(DeleteStatementProvider deleteStatement);
      default int delete(DeleteDSLCompleter completer){}
      default int deleteByPrimaryKey(Long id_){}
      ```

   3. 改

      ```java
      int update(UpdateStatementProvider updateStatement);
      default int update(UpdateDSLCompleter completer){}
      static UpdateDSL<UpdateModel> updateAllColumns(Pet record, UpdateDSL<UpdateModel> dsl){}
      static UpdateDSL<UpdateModel> updateSelectiveColumns(Pet record, UpdateDSL<UpdateModel> dsl){}
      default int updateByPrimaryKey(Pet record){}
      default int updateByPrimaryKeySelective(Pet record){}
      ```

   4. 查

      ```java
      Optional<Pet> selectOne(SelectStatementProvider selectStatement);
      List<Pet> selectMany(SelectStatementProvider selectStatement);
      default Optional<Pet> selectOne(SelectDSLCompleter completer){}
      default List<Pet> select(SelectDSLCompleter completer){}
      default List<Pet> selectDistinct(SelectDSLCompleter completer){}
      default Optional<Pet> selectByPrimaryKey(Long id_){}
      long count(SelectStatementProvider selectStatement);
      default long count(CountDSLCompleter completer){}
      ```

名词解释

1. DSL *Domain Specific Language* 领域特定语言

参考

MyBatis官网：https://blog.mybatis.org/p/products.html 看各product的doc



