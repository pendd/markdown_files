## 1. RESTful

> RESTful 是面向资源的  URI中都是名词

![avatar](https://raw.githubusercontent.com/pendd/picture/master/JPA/restful_uri.png)

> http 对应请求

![avatar](https://raw.githubusercontent.com/pendd/picture/master/JPA/restful_http.png)

前端请求过滤、排序、选择、分页
![avatar](https://raw.githubusercontent.com/pendd/picture/master/JPA/restful_request_method.png)

> 复杂资源关系的表达

**GET /cars/711/drivers/ 返回 使用过编号711汽车的所有司机**

**GET /cars/711/drivers/4 返回 使用过编号711汽车的4号司机**

> 版本化你的API

强制性增加API版本声明，不要发布无版本的API。如：/api/v1/blog

## 2.JPA

> 什么是JPA？

```TEX
JPA 是 Java Persistence API 的简称，是JDK5.0注解或XML描述对象-关系表的映射关系，
并将运行期间的实体对象持久化到数据库中。
JPA是一种标准，SpringDataJPA是一种实现。
```

## 3.SpringData

> 什么是SpringData？

```tex
SpringData是Spring的一个子项目，支持各种数据库的访问技术，包含关系型和非关系型数据库。
SpringData本身又包括多个模块，其中一个模块SpringData-JPA就是针对JPA的一种封装实现。
```

## 4.SpringBoot集成JPA

```YML
spring:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

> 启动类上加上@EnableJpaRepositories

```java
//repositoryBaseClass = BaseRepository.class
// 指定自定义baseRepository类
@EnableJpaRepositories(basePackages = "com.pd.jpa.repository") 
```



## 5.CrudRepository

![avatar](https://raw.githubusercontent.com/pendd/picture/master/JPA/crudRepository.png)

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table
public class Person {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column
  private Integer id;

  @Column private String name;

  @Column private Integer age;
}

```



```java
// 实现CrudRepository接口
public interface PersonRepository 
  extends CrudRepository<Person, Integer> {
  long countByName(String name);

  // JPA 删除需要加@Transactional注解
  @Transactional
  long deleteByName(String name);

  @Transactional
  List<Person> removeByAge(Integer age);
}
```

```java
@SpringBootTest
public class PersonRepositoryTest {

  @Autowired private PersonRepository repository;

  @Test
  void save() {
    repository.save(Person.builder()
                    .name("john").age(25).build());
  }

  @Test
  void saveAll() {
    repository.saveAll(
        Arrays.asList(
            Person.builder().name("lily").age(23).build(),
            Person.builder().name("liu").age(26).build()));
  }

  @Test
  void findAll() {
    Iterable<Person> persons = repository.findAll();
    persons.forEach(System.out::print);
  }

  @Test
  void findById() {
    System.out.println(repository.findById(1));
  }

  @Test
  void findAllById() {
    // findAllById 参数是Iterable  
    // Collection集合接口继承了Iterable接口
    repository.findAllById(Arrays.asList(1, 2));
  }

  @Test
  void existsById() {
    System.out.println(repository.existsById(1));
    System.out.println(repository.existsById(5));
  }

  @Test
  void count() {
    System.out.println(repository.count());
  }

  @Test
  void deleteById() {
    repository.deleteById(1);
    // EmptyResultDataAccessException
    repository.deleteById(5);
  }

  @Test
  void delete() {
    // 对象中要加入主键字段
    repository.delete(Person.builder().id(2).build());
  }

  @Test
  void deleteAllByIterable() {
    repository.deleteAll(
        Arrays.asList(Person.builder().id(3).build(), 					                      											Person.builder().id(4).build()));
  }

  @Test
  void deleteAll() {
    repository.deleteAll();
  }
}

```

## 6.PagingAndSortingRepository

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Table
@Entity
public class Student {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Integer id;

  @Column private String name;

  @Column private Integer age;
}

```

```java
public interface StudentRepository extends PagingAndSortingRepository<Student, Integer> {}

```

```java
// PageRequest extends AbstractPageRequest
// AbstractPageRequest implements Pageable
@SpringBootTest
class StudentRepositoryTest {

  @Autowired private StudentRepository repository;

  // 分页
  @Test
  void findAll() {
    // PageRequest.of  方法 page 下标从0开始，表示第一页
    Page<Student> all = repository.findAll(PageRequest.of(0, 5));
    all.get().forEach(System.out::println);
  }

  // 分页加排序
  @Test
  void findAllByPageable() {
    repository
        .findAll(PageRequest.of(0, 5, 
          Direction.ASC, "age", "name"))
        .get()
        .forEach(System.out::println);
  }

  // 多属性一种排序类型
  @Test
  void findAllBySort() {
    repository
        .findAll(PageRequest.of(0, 5, 
          Sort.by(Direction.ASC, "age", "name")))
        .get()
        .forEach(System.out::println);
  }

  // 多属性多种排序类型
  @Test
  void findAllBySortOrder() {
    repository
        .findAll(Sort.by(new Order(Direction.ASC, "age"),
                         new Order(Direction.DESC, "name")))
        .forEach(System.out::println);
  }
}
```

```java
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {

	/**
	 * Returns all entities sorted by the given options.
	 *
	 * @param sort
	 * @return all entities sorted by the given options
	 */
	Iterable<T> findAll(Sort sort);

	/**
	 * Returns a {@link Page} of entities meeting the paging restriction provided in the {@code Pageable} object.
	 *
	 * @param pageable
	 * @return a page of entities
	 */
	Page<T> findAll(Pageable pageable);
}
```

## 7.Query

> Emp

```java
@Table
@Entity(name = "Emp")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Emp {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column
    private String name;

    @Column
    private Integer age;

}
```

```java
@NoRepositoryBean //告诉JPA不要创建对应接口的bean对象
public interface BaseRepository<T, ID> extends Repository<T, ID> {

    Optional<T> findById(ID id);

    <S extends T> S save(S entity);
}
```

```java
public class BaseRepositoryImpl implements BaseRepository {
    @Override
    public Optional findById(Object o) {
        return Optional.empty();
    }

    @Override
    public Object save(Object entity) {
        return null;
    }

}
```

```java
public interface EmpRepository extends BaseRepository<Emp, Integer> {
    List<Emp> findByNameAndAge(String name, Integer age);

    List<Emp> findDistinctEmpByName(String name);

    // 不分页的话传入 Pageable.unpaged()
    Page<Emp> findByName(String name, Pageable pageable);

    // Page 和 Slice 区别：
    // Page 继承自 Slice
    // Page 每次查询都会返回总数，所以对于查询数据量大的情况下效率不高 count + query
    // Slice 查询不返回总数
    Slice<Emp> findEmpByName(String name, Pageable pageable);

    // 不排序的话传入 Sort.unsorted()
    List<Emp> findByName(String name, Sort sort);

    List<Emp> findDistinctEmpByName(String name, Pageable pageable);

    // 这里from 后面的 Emp 对应的是 @Entity 注解里name的值，只是默认和类名一致而已
    @Query("select e from Emp e where e.name = :name")
    List<Emp> findByName(String name);

    // #{#entityName} 自动获取 @Entity 注解里name的值
    // 这种方式注入可以避免@Entity注解中name值得变化照成其他@Query语句中的修改
    @Query("select e from #{#entityName} e where e.name = ?1")
    List<Emp> findByNameFromEntityName(String name);
}

```

```java
@SpringBootTest
public class EmpRepositoryTest {

    @Autowired private EmpRepository repository;

    @Test
    void findByNameAndAge() {
        System.out.println(repository.findByNameAndAge("lucy", 20));
    }

    @Test
    void findDistinctByName() {
        System.out.println(repository.findDistinctEmpByName("lucy"));
    }

    @Test
    void findByName() {
        Page<Emp> page = repository.findByName("lucy", PageRequest.of(0, 2));
        System.out.println(page);
    }

    @Test
    void findEmpByName() {
        Slice<Emp> slice = repository.findEmpByName("lucy", PageRequest.of(0, 2));
        System.out.println(slice);
    }

    @Test
    void findByNameSort() {
        // 多属性多方向排序
        Sort sort = Sort.by("age").ascending().and(Sort.by("name").descending());
        List<Emp> list = repository.findByName("lucy", Sort.by(Direction.DESC, "age"));
        System.out.println(list);
    }

    @Test
    void findDistinctEmpByName() {
        List<Emp> list = repository.findDistinctEmpByName("lucy", PageRequest.of(0, 2));
        System.out.println(list);
    }

    @Test
    void findByNameQuery() {
        System.out.println(repository.findByName("lucy"));
    }

    @Test
    void findByNameFromEntityName() {
        System.out.println(repository.findByNameFromEntityName("lucy"));
    }
```

