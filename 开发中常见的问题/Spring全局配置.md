# 1. pojo类

```java
// Dog.java
@Component
@ConfigurationProperties(prefix = "dog")
public class Dog {
    private String name;
    private Integer age;
    ......
    // 省略get和set方法
    // 省略了toString方法
```

```java
// Person.java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private boolean happy;
    private Date birth;
    private Map<String,Object> map;
    private List<Object> list;
    private Dog dog;
    ......
    // 省略get和set方法
    // 省略了toString方法
}
```

# 2. 使用properties配置文件

```properties
<!-- application.properties -->
dog.name=哈士奇
dog.age=2

person.name=hoshino
person.age=17
person.happy=false
person.birth=2020/06/01
person.map.k1=v1
person.map.k2=v2
person.list=phone,music,video
person.dog.name=金毛
person.dog.age=3
```

# 3. 使用yaml配置文件

```yaml
<!-- application.yaml -->
dog:
  name: 哈士奇
  age: 2

person:
  #字面量：单个的、不可再分的值。date、boolean、string、number、null
  name: hoshino
  age: 17
  happy: false
  birth: 2020/06/01

  #对象：键值对的集合。map、hash、set、object
  #行内写法：
  map: {k1: v1,k2: v2}
  #键值对的另一种方式
  #  map:
  #    k1: v1
  #    k2: v2

  #数组：一组按次序排列的值。array、list、queue
  #行内写法：
  list: [phone,music,video]
  #数组的另一种方式
  #  list:
  #    - phone
  #    - music
  #    - video

  dog:
    name: 金毛
    age: 3
```

#### 4. 测试类

```
// SpringbootYamlApplicationTests.java
@SpringBootTest
class ApplicationTests {

    @Autowired
    private Dog dog;

    @Autowired
    private Person person;

    @Test
    void contextLoads() {
        System.out.println(dog);
        System.out.println(person);
    }

}
```

- ==**注：application.properties配置文件的优先级要高于application.yaml配置文件**==