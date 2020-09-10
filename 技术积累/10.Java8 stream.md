#### 取出/跳过前几条数据
```java
//取出前5条数据
list.stream().limit(5).collect(Collectors.toList());

//跳过前两条数据
list.stream().skip(2).collect(Collectors.toList());
```



#### 选出List中的某一列数据
```java
//单据
List<InvoiceApproval> list = invoiceApprovalService.doSearch(map);

//单据id
List<Long> idList = list.stream()
                        .map(InvoiceApproval::getId)
                        .collect(Collectors.toList());



过滤数据
List<String> list = Arrays.asList("12345 789".split(""));

//筛选出不为空字符串的String
List<String> collect = list.stream()
            .filter(i -> !i.equals(" "))
            .collect(Collectors.toList());
```





#### 将list转成map

```java
Map<Long,String> map = list.stream()
            .collect(Collectors.toMap(User::getId,User::getType));
```





#### 计算个数

```java
List<User> list = userMapper.selectAll();

//计算风险值为L的个数
long lowCount = list.stream()
                    .filter(l -> "L".equals(l.getRisk()))
                    .count();
```





#### List去重

```java
//distinct,过滤调一样的元素基于hashcode和equals

list.stream().distinct().collect(Collectors.toList());


//userName 和 nickName 组合不能重复
//如存在多条：allen,bobo  则去重后这样的数据只有一条
List<User> unique = list.stream().collect(
        Collectors.collectingAndThen(
                Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(d -> d.getUserName() + "," + d.getNickName()))), ArrayList::new)
);


//或者以下数据视为一条的去重 
//allen,bobo  bobo,allen  这样的数据视为一条，同样要去除
List<User> userList = unique.stream().map(u -> list.stream()
        .filter(l -> !u.getUserName().equals(l.getNickName()) && !u.getNickName().equals(l.getUserName()))
        .findAny()
        .orElse(null)
)
        .filter(Objects::nonNull)
        .collect(Collectors.toList());
```



#### 根据多列分组，Collectors.groupingBy
```java
List<User> list = userMapper.selectAll();

//根据客户和sapid分组
Map<String, List<User>> groupMap = originList.stream()
        .collect(Collectors.groupingBy(i -> i.getCustomName() + "," + i.getSapsid()));

```





#### Reduce

```java
//求和
//0代表初始值
list.stream().reduce(0, Integer::sum);//max/min

//可以不指定初始值，返回值变为Optional<Integer>以防止可能为空的情况
Optional<Integer> sum = list.stream().reduce(Integer::sum);//max/min
```





#### 排序

```java
List<Integer> list = list.stream()
                .sorted(Comparator.comparingInt(Integer::intValue))
                .collect(Collectors.toList());
```





#### 两个集合的交集/差集/并集

```java
//交集
list1.stream().filter(s1 -> list2.contains(s1))
        .collect(Collectors.toList());

//交集
list1.stream()
        .map(s -> list2.stream()
                            .filter(l -> s.equals(l))
                            .findAny()
                            .orElse(null))
        .filter(Objects::nonNull)
        .collect(Collectors.toList());
        
//差集
list2.stream()
    .filter(item -> !list1.contains(item))
    .collect(toList());
    
//差集
List<String> collect = list1.stream()
        .filter(s -> !userList.stream()
                .map(User::getName)
                .collect(Collectors.toList())
                .contains(s))
        .collect(Collectors.toList());
        
//并集
list1.addAll(list2);
```





#### 查找/匹配

```java
//集合中所有元素都匹配，返回true/false
list.stream().allMatch(i -> i.equals("9"));

//存在一个或多个
list.stream().anyMatch(i -> i.equals("9"));

//不存在
list.stream().noneMatch(i -> i.equals("9"));

//返回当前流中的任意元素
Optional<Menu> anyMenu = list.stream().findAny();

//返回当前流中第一个元素
Optional<Menu> firstMenu = list.stream().findFirst();

```