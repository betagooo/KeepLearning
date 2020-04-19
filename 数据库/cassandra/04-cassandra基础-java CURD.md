# Spring Data for Appache Cassandra

> [Spring Data for Apache Cassandra](https://spring.io/projects/spring-data-cassandra#learn)

spring data 提供了以下几种方法去访问Cassandra数据库：

- [`CqlTemplate`](https://docs.spring.io/spring-data/cassandra/docs/2.2.6.RELEASE/reference/html/#cassandra.cql-template) 和 [`ReactiveCqlTemplate`](https://docs.spring.io/spring-data/cassandra/docs/2.2.6.RELEASE/reference/html/#cassandra.reactive.cql-template) 是经典的Spring CQL 方法，也是最流行的。
- [`CassandraTemplate`](https://docs.spring.io/spring-data/cassandra/docs/2.2.6.RELEASE/reference/html/#cassandra.template) 。CassandraTemplate 封装了CqlTemplage，提供查询结果对对象的映射，以及使用SELECT、INSERT、UPDATE和DELETE方法，而不是编写CQL语句。这种方法提供了更好的文档和更简易的使用。
- [`ReactiveCassandraTemplate`](https://docs.spring.io/spring-data/cassandra/docs/2.2.6.RELEASE/reference/html/#cassandra.reactive.template)。ReactiveCassandraTemplate 封装了一个 ReactiveCqlTemplate去提供查询结果对对象的映射，以及使用SELECT、INSERT、UPDATE和DELETE方法，而不是编写CQL语句。这种方法提供了更好的文档和更简易的使用。
- Repository Abstraction。Repository Abstraction 允许你在数据访问层创建repository声明。目标是显著减少为各种持久性存储实现数据访问层所需的样板代码数量。



# Spring项目接入

> 本次测试使用的是spring-data-cassandra的2.2.6.RELEASE版本，主要使用cassandraTemplate

**spring 版本要求**

```xml
<spring.framework.version>5.2.5.RELEASE</spring.framework.version>
```

spring-data-cassandra不同版本变化可能有点大，spring的版本尽量保持和要求的一致。

**maven依赖**

```xml
<dependencies>

  <dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-cassandra</artifactId>
    <version>2.2.6.RELEASE</version>
  </dependency>

</dependencies>
```

**Cassandra连接配置**

application.properties：

```
spring.data.cassandra.keyspace-name=mytest
spring.data.cassandra.contact-points=localhost
spring.data.cassandra.port=9042
#spring.data.cassandra.username=test
#spring.data.cassandra.password=test
```

**log日志**

可配置打印cql语句：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder charset="UTF-8">
            <pattern>[%date{yyyy-MM-dd HH:mm:ss.SSS}] %level [LOGID:%X{logId}]  %thread [%file:%line] - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="info">
        <appender-ref ref="Console" />
    </root>
    <!--cassandra日志，debug级别可输出cql语句-->
    <logger name="org.springframework.data.cassandra.core.cql.CqlTemplate" level="debug">
        <appender-ref ref="Console" />
    </logger>
</configuration>
```

**cassandra 测试表**

```sql
CREATE KEYSPACE mytest 
    WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '3'}  
    AND durable_writes = true;

create table mytest.tt(
    a text,
    b text,
    c text,
    d text,
    e text,
    f text,
    g timestamp,
    primary key ( a, b, c )
)with clustering order by(b asc, c desc);

insert into tt (a,b,c,d,e,f,g) values ('a', 'aa', 'aaa', 'aaaa1','eeee1', 'fffff', '2013-01-01 00:05:01.500');
insert into tt (a,b,c,d,e,f,g) values ('a', 'aa', 'aaa1', 'aaaa1','eeee', 'fffff1', '2014-01-01 00:05:01.500');
insert into tt (a,b,c,d,e,f,g) values ('a', 'aa', 'aaa', 'aaaa2','eeee2', 'fffff', '2015-01-01 00:05:01.500');
insert into tt (a,b,c,d,e,f,g) values ('a', 'bb', 'bbb', 'bbbb1','eeee', 'fffff2', '2016-01-01 00:05:01.500');
insert into tt (a,b,c,d,e,f,g) values ('a', 'bb', 'bbb1', 'bbbb1','eeee3', 'fffff', '2017-01-01 00:05:01.500');
insert into tt (a,b,c,d,e,f,g) values ('a', 'bb', 'bbb', 'bbbb2','eeee4', 'fffff3', '2018-01-01 00:05:01.500');
insert into tt (a,b,c,d,e,f,g) values ('a', 'cc', 'ccc', 'cccc','eeee', 'fffff', '2019-01-01 00:05:01.500');
insert into tt (a,b,c,d,e,f,g) values ('b', 'cc', 'ccc', 'cccc','eeee5', 'fffff4', '2020-01-01 00:05:01.500');
insert into tt (a,b,c,d,e,f,g) values ('c', 'cc', 'ccc', 'cccc','eeee', 'fffff', '2021-01-01 00:05:01.500');

create index if not exists idx_d on tt(d);
create custom index idx_contains_f on mytest.tt(f)
    USING 'org.apache.cassandra.index.sasi.SASIIndex'
    WITH OPTIONS = { 'mode': 'CONTAINS' };
```



## CassandraTemplate 

> [CassandraTemplate API](https://docs.spring.io/spring-data/cassandra/docs/2.2.6.RELEASE/api/org/springframework/data/cassandra/core/CassandraTemplate.html)

### PO类

```java
package com.betago.arsenal.java.cassandra;

import lombok.Data;
import org.springframework.data.cassandra.core.cql.PrimaryKeyType;
import org.springframework.data.cassandra.core.mapping.Column;
import org.springframework.data.cassandra.core.mapping.PrimaryKeyColumn;
import org.springframework.data.cassandra.core.mapping.Table;
import java.util.Date;

@Data
@Table(value = "tt")
public class TtPO {
    @PrimaryKeyColumn(name = "a", type= PrimaryKeyType.PARTITIONED)
    private String a;
    @Column("b")
    private String b;
    @Column("c")
    private String c;
    @Column("d")
    private String d;
    @Column("e")
    private String e;
    @Column("f")
    private String f;
    @Column("g")
    private Date g;
}

```



### 常用CURD方法

```java
package com.betago.arsenal.java.cassandra;

import com.datastax.driver.core.querybuilder.QueryBuilder;
import com.datastax.driver.core.querybuilder.Select;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.cassandra.core.CassandraTemplate;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class CassandraTemplateDAO {
    @Autowired
    private CassandraTemplate cassandraTemplate;

    /**
     * cql字符串查询
     */
    public List<TtPO> selectByCql(String a, String b, String c) {
        String cql = String.format("select * from tt where a='%s' and b='%s' and c='%s'", a, b, c);
        return cassandraTemplate.select(cql, TtPO.class);
    }

    /**
     * QueryBuilder eq & non-eq查询
     */
    public List<TtPO> selectByQuery(String a, String b, String c) {
        Select select = QueryBuilder.select().from("tt");
        select.where(QueryBuilder.eq("a", a))
                .and(QueryBuilder.eq("b", b))
//                .and(QueryBuilder.eq("c", c))
//                .and(QueryBuilder.lt("c", c))
//                .and(QueryBuilder.gt("c", c))
                .and(QueryBuilder.gte("c", c));
        return cassandraTemplate.select(select, TtPO.class);
    }

    /**
     * QueryBuilder - allow filtering查询
     */
    public List<TtPO> selectByQueryAllowFiltering(String a, String b, String c, String f) {
        Select select = QueryBuilder.select().from("tt").allowFiltering();
        select.where(QueryBuilder.eq("a", a))
                .and(QueryBuilder.eq("b", b))
                .and(QueryBuilder.gte("c", c))
                .and(QueryBuilder.eq("f", f));
        return cassandraTemplate.select(select, TtPO.class);
    }

    /**
     * QueryBuilder - IN查询
     */
    public List<TtPO> selectByQueryIn(String a, String b, List<String> cList) {
        Select select = QueryBuilder.select().from("tt");
        select.where(QueryBuilder.eq("a", a))
                .and(QueryBuilder.eq("b", b))
                .and(QueryBuilder.in("c", cList));
        return cassandraTemplate.select(select, TtPO.class);
    }
    
    /**
     * 排序查询
     */
    public List<TtPO> sort(String a) {
        Select select = QueryBuilder.select().from("tt")
                .orderBy(QueryBuilder.asc("b"), QueryBuilder.desc("c"));
        select.where(QueryBuilder.eq("a", a));
        return cassandraTemplate.select(select, TtPO.class);
    }
    /**
     * 模糊查询
     */
    public List<TtPO> fuzzyQuery(String f) {
        Select select = QueryBuilder.select().from("tt");
        select.where(QueryBuilder.like("f", f));
        return cassandraTemplate.select(select, TtPO.class);
    }
    
    /**
     * count
     */
    public Long count(String a){
        Query query = Query.query(Criteria.where("a").is(a));
        return cassandraTemplate.count(query, TtPO.class);
    }
    
    /**
     * 插入
     */
    public TtPO insert(TtPO tt) {
        return cassandraTemplate.insert(tt);
    }
    
    /**
     * 更新
     */
    public TtPO update(TtPO tt) {
        return cassandraTemplate.update(tt);
    }

    /**
     * 条件更新
     */
    public boolean update(String a, String b, String c, TtPO tt) {
        Query query = Query.query(Criteria.where("a").is(a)
                , Criteria.where("b").is(b)
                , Criteria.where("c").is(c)
//                , Criteria.where("c").lte(c)
//                , Criteria.where("c").gte(c)
        );

        Update update = Update.empty()
                .set("d", tt.getD())
                .set("e", tt.getE())
                .set("d", tt.getD())
                .set("e", tt.getE())
                .set("f", tt.getF())
                .set("g", tt.getG());

        return cassandraTemplate.update(query, update, TtPO.class);
    }

    /**
     * 条件更新
     */
    public boolean updateSelective(TtPO tt) {
        Query query = Query.query(Criteria.where("a").is(tt.getA())
                , Criteria.where("b").is(tt.getB())
                , Criteria.where("c").is(tt.getC())
        );
        Update update = Update.empty();
        if (tt.getD() != null) {
            update = update.set("d", tt.getD());
        }
        if (tt.getE() != null) {
            update = update.set("e", tt.getE());
        }
        if (tt.getF() != null) {
            update = update.set("f", tt.getF());
        }
        if (tt.getG() != null) {
            update = update.set("g", tt.getG());
        }

        return cassandraTemplate.update(query, update, TtPO.class);
    }
    
    /**
     * 删除
     */
    public void delete(TtPO tt) {
        cassandraTemplate.delete(tt);
    }

    public void delete(String a, String b, String c) {
        Query query = Query.query(
                Criteria.where("a").is(a)
                , Criteria.where("b").is(b)
                , Criteria.where("c").is(c)
//                , Criteria.where("c").lte(c)
//                , Criteria.where("c").gte(c)
        );
        cassandraTemplate.delete(query, TtPO.class);
    }

}
```

**注意：**Cassandra 更新数据时，没有数据会插入新数据。



## CrudRepository

### PO类

```java
package com.betago.arsenal.java.cassandra;

import lombok.Data;
import org.springframework.data.cassandra.core.cql.PrimaryKeyType;
import org.springframework.data.cassandra.core.mapping.Column;
import org.springframework.data.cassandra.core.mapping.PrimaryKeyColumn;
import org.springframework.data.cassandra.core.mapping.Table;
import java.util.Date;

@Data
@Table(value = "tt")
public class TtPO {
    @PrimaryKeyColumn(name = "a", type= PrimaryKeyType.PARTITIONED)
    private String a;
    @Column("b")
    private String b;
    @Column("c")
    private String c;
    @Column("d")
    private String d;
    @Column("e")
    private String e;
    @Column("f")
    private String f;
    @Column("g")
    private Date g;
}

```



### CrudRepository interface

CrudRepository类 基本方法有：

```java
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    <S extends T> S save(S var1);
    <S extends T> Iterable<S> saveAll(Iterable<S> var1);
    Optional<T> findById(ID var1);
    boolean existsById(ID var1);
    Iterable<T> findAll();
    Iterable<T> findAllById(Iterable<ID> var1);
    long count();
    void deleteById(ID var1);
    void delete(T var1);
    void deleteAll(Iterable<? extends T> var1);
    void deleteAll();
}
```

定义一个interface，继承CrudRepository 类

```java
package com.betago.arsenal.java.cassandra;

import org.springframework.data.cassandra.repository.Query;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import java.util.Date;
import java.util.List;

@Repository
public interface CurdRepositoryDao extends CrudRepository<TtPO, String> {

    @Query("Select * from tt where a = ?0 and  b = ?1 and c>= ?2")
    List<TtPO> select1(String a, String b, String c);

    @Query("Select * from tt where a = ?0 and  b = ?1 and f= ?2 allow filtering")
    List<TtPO> select2(String a, String b, String f);

    @Query("updat tt set f = ?3 and g = ?4 where a = ?0 and  b = ?1 and c= ?2")
    boolean update(String a, String b, String c, String f, Date g);

    @Query("select count (*) from tt where a = ?0 and  b = ?1")
    Long count(String a, String b);
}

```

测试类

```java
package com.betago.arsenal.java.cassandra;

import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import java.util.Date;
import static org.junit.jupiter.api.Assertions.*;

@RunWith(SpringRunner.class)
@SpringBootTest
class CurdRepositoryDaoTest {

    @Autowired
    CurdRepositoryDao curdRepositoryDao;
    @Test
    void select() {
        System.out.println("result1: " + curdRepositoryDao.select1("a", "aa", "aaa"));
        System.out.println("result2: " + curdRepositoryDao.select2("a", "cc", "fffff"));
        System.out.println("result3: " + curdRepositoryDao.count("a", "cc"));
    }

    @Test
    void insert() {
        TtPO tt = new TtPO();
        tt.setA("x");
        tt.setB("xx");
        tt.setC("xxx");
        tt.setE("xxxx");
        tt.setF("xxxxx");
        tt.setG(new Date());
        curdRepositoryDao.save(tt);
        System.out.println("result1: " + curdRepositoryDao.select1("x", "xx", "xxx"));
    }

    @Test
    void update() {
        curdRepositoryDao.update("x", "xx", "xxx", "mmmmmmm", new Date());
        System.out.println("result1: " + curdRepositoryDao.select1("x", "xx", "xxx"));
    }

}
```



