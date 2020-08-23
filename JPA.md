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

## 2.CrudRepository

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

## 3.PagingAndSortingRepository

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

