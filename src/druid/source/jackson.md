# Jackson
Jackson是java通过json做序列化， 反序列化的第三方库



## Jackson Annotation

### 序列化注解

#### @JsonAnyGetter
@JsonAnyGetter 序列化java的Map数据结构

Example:

``` java 
public class Bean
{

  public String name;

  private Map<String, String> properties;


  @JsonAnyGetter
  public Map<String, String> getProperties()
  {
    return properties;
  }

  public Bean(String name)
  {

    this.name = name;
    this.properties = new HashMap<String, String>();

  }


  public void add(String k, String v)
  {

    this.properties.put(k, v);

  }
}

```


``` java
public class Main
{


  public static void main(String[] args) throws JsonProcessingException
  {


    Bean bean = new Bean("1");
    bean.add("attr1", "va11");
    bean.add("attr2", "val2");

    String result = new ObjectMapper().writeValueAsString(bean);
    System.out.println(result);
  }
}

```

#### @JsonPropertyOrder
保持序列化的顺序

```java
@JsonPropertyOrder({ "name", "id" })
public class MyBean {
    public int id;
    public String name;
}

```


Output:

```
{
    "name":"My bean",
    "id":1
}
```


####  @JsonRawValue
一般用于嵌入json

```Java
public class RawBean {
    public String name;
 
    @JsonRawValue
    public String json;
}

```

Output:

```
{
    "name":"My bean",
    "json":{
        "attr":false
    }
}

```

#### @JsonValue
序列化实例的单个方法

``` java
public enum Bean
{
  Type1("type 1"), Type2("Type 2");


  private String name;

  Bean(String name)
  {
    this.name = name;
  }

  @JsonValue
  public String getName()
  {
    return name;
  }
}
```

#### @JsonRootName
使用注释来指定要使用的根包装齐器的名称.

```

@JsonRootName(value = "user")
public class Bean
{
  @JsonProperty
  private String name;

  @JsonProperty
  private int Id;

  public Bean(String name, int Id)
  {
    this.name = name;
    this.Id = Id;
  }
}
```


Output:

```
{"user":{"name":"wang","Id":1}}
```


#### @JsonSerialize
按照自定义的格式进行序列化， 常用于时间

### 反序列化注解

#### @JsonCreator
配合JsonProperty将字符串序列化为java的类

``` java
public class Bean
{
  public String name;

  public int Id;

  @JsonCreator
  public Bean(
      @JsonProperty("name") String name,
      @JsonProperty("id") int Id
  )
  {
    this.name = name;
    this.Id = Id;
  }
}
```

#### @JacksonInject
@JacksonInject表示属性将从注入中获取其值，而不是从JSON数据中获取。

 
#### @JsonAnySetter
@jacaksonAnySetter将json映射为java的map类型

#### @JsonSetter
@JsonSetter是@JsonProperty的替代方案

#### @JsonDeserialize
按照自定义时间字符串反序列化java的property

#### @JsonAlias
@JsonAlias在反序列化期间为属性定义了一个或多个备用名称


Example: 

```java
public class AliasBean {
    @JsonAlias({ "fName", "f_name" })
    private String firstName;   
    private String lastName;
}
```

### jacakson一些属性过滤的注解

#### @JsonIgnoreProperties
忽略某个属性

level: class

#### @JsonIgnore
@JsonIgnore注释用于标记属性要在字段级别忽略

level: field

#### @JsonIgnoreType
@JsonIgnoreType标记要忽略的带注释类型的所有属性。

#### @JsonInclude
我们可以使用@JsonInclude来排除具有 empty/ null / defualt的属性。
 
#### @JsonAutoDetect
@JsonAutoDetect可以覆盖哪些属性可见而哪些不可见的默认语义。

序列化私有的properties

```java
@JsonAutoDetect(fieldVisibility = Visibility.ANY)
public class PrivateBean {
    private int id;
    private String name;
}
```

### jackson多态类型处理注释

- @JsonTypeInfo – 表示序列化中包含的类型信息的详细信息
- @JsonSubTypes - 表示带注释的类型的子类型
- @JsonTypeName - 定义用于带注释的类的逻辑类型名称

Example
```java
public class Zoo
{

  public Animal animal;


  @JsonTypeInfo(
      use = JsonTypeInfo.Id.NAME,
      include = JsonTypeInfo.As.PROPERTY,
      property = "type"
  )
  @JsonSubTypes(
      {
          @JsonSubTypes.Type(value = Dog.class, name = "dog"),
      }
  )
  public static class Animal
  {
    public String name;

  }


  public Zoo(Animal a)
  {
    this.animal = a;
  }

  @JsonTypeName("dog")
  public static class Dog extends Animal
  {
    public double barkVolume;

    public Dog(String name)
    {
      this.name = name;
    }

  }
}
```

```java
public class Main
{
  public static void main(String[] args) throws JsonProcessingException
  {

    Zoo.Dog dog = new Zoo.Dog("Lacy");

    Zoo bean = new Zoo(dog);


    String result = new ObjectMapper().writeValueAsString(bean);

    System.out.println(result);
  }
}
```

**output:** {"animal":{"type":"dog","name":"Lacy","barkVolume":0.0}}

### Jackson 常用注解

#### @JsonProperty
我们可以添加@JsonProperty注释来指示JSON中的属性名称

#### @JsonFormat
@JsonFormat注释指定序列化日期/时间值时的格式。


Example:

```java
public class Event {
    public String name;
 
    @JsonFormat(
      shape = JsonFormat.Shape.STRING,
      pattern = "dd-MM-yyyy hh:mm:ss")
    public Date eventDate;
}

```

#### @JsonUnwrapped

@JsonUnwrapped定义了序列化/反序列化时应解包/展平的值。

Example:

```java
public class UnwrappedUser {
    public int id;
 
    @JsonUnwrapped
    public Name name;
 
    public static class Name {
        public String firstName;
        public String lastName;
    }
}
```

#### @JsonView
@JsonView表示将包含属性以进行序列化/反序列化的视图。

Example:

```java
public class Views {
    public static class Public {}
    public static class Internal extends Public {}
}



public class Item {
    @JsonView(Views.Public.class)
    public int id;
 
    @JsonView(Views.Public.class)
    public String itemName;
 
    @JsonView(Views.Internal.class)
    public String ownerName;
}
```


#### @JsonManagedReference, @JsonBackReference

@JsonManagedReference和@JsonBackReference注释可以处理父/子关系并解决循环问题。

```java
public class ItemWithRef
{

  public int id;
  public String itemName;

  @JsonManagedReference
  public UserWithRef owner;


  public ItemWithRef(int id, String itemName, UserWithRef o)
  {
    this.id = id;

    this.itemName = itemName;

    this.owner = o;

  }
}
```


```java
public class UserWithRef
{

  public int id;
  public String name;


  @JsonBackReference
  public List<ItemWithRef> userItems;

  public UserWithRef(int id, String name)
  {

    this.id = id;
    this.name = name;

    this.userItems = new LinkedList<ItemWithRef>();

  }


  public void add(ItemWithRef item)
  {
    userItems.add(item);
  }

}
```


```java
public class Main
{


  public static void main(String[] args) throws JsonProcessingException
  {


    UserWithRef user = new UserWithRef(1, "john");

    ItemWithRef item = new ItemWithRef(2, "book", user);

    user.add(item);


    String result = new ObjectMapper().writeValueAsString(item);

    System.out.println(result);


  }
}

```

**output:** {"id":2,"itemName":"book","owner":{"id":1,"name":"john"}}


#### @JsonIdentityInfo
@JsonIdentityInfo指示在序列化/反序列化值时应使用对象标识 - 例如，处理无限递归类型的问题。


#### @JsonFilter
@JsonFilter注释指定在序列化期间使用的过滤器。
