---
---
# 标准表达式语法
### 变量表达式`${ }`
在控制器中往页面传递几个变量：
```java
@Controller
public class IndexController {
    
    @RequestMapping(value="/index",method=RequestMethod.GET)
    public String index(HttpSession session, Model model){
        User user = new User();
        user.setName("KangKang");
        user.setAge(25);
        user.setHabbit(new String[]{"football","basketball","swim"});
        session.setAttribute("user", user);
        model.addAttribute(user);
        return "index";
    }
```
在页面中使用变量表达式${}来获取它们：

```html
<p th:utext="${user.name}"></p>
<p th:utext="${session.user.getName()}"></p>
<p th:utext="${session.user.upcaseName()}"></p>
<p th:utext="${user.habbit[0]}"></p>
```
可以看到变量表达式不但可以获取变量的属性值，甚至还可以访问变量的方法（getName()和upcaseName()）。session代表HttpSession对象。

### 选择表达式*{ }

```html
<div th:object="${session.user}">
   <p>name: <span th:text="*{name}"></span></p>
   <p>age: <span th:text="*{age}"></span></p>
   <p>habbit: <span th:text="*{habbit[0]}"></span></p>
</div>
```
`*{}代指th:object所指定的对象，即${session.user}。`

### URL链接表达式@{ }
URL链接表达式会给URL自动添加上下文的名字。比如：
```html
<a th:href="@{/main}">main</a>
```
解析后的href值为http://localhost:8080/thymeleaf/main。

当需要在URL中传递参数时，比如这样http://localhost:8080/thymeleaf/main?name=KangKang，可以如下操作：
```html
<a th:href="@{/main(name=${session.user.name})}">main</a>
```
传递多个参数：
```html
<a th:href="@{/main(name=${session.user.name},age=${session.user.age})}">main</a>
```
路径变量的写法：
```html
<a th:href="@{/main/{name}(name=${session.user.name})}">main</a>
```
后端接受路径变量：
```java
@RequestMapping(value="main/{name}")
public String main(@PathVariable String name){
   System.out.println("pathValue: "+name);
   return "main";
}
```
### 字面量
* 文本常量
文本常量指的是单引号之间的字符串，比如：
```html
<p th:text="'Welcome KangKang'"></p>
```
* 数字常量
```html
<p>The year is <span th:text="2017">1492</span>.</p>
<p>In two years, it will be <span th:text="2017 + 2">1494</span>.</p>
```
* Boolean类型的常量
Boolean类型的常量就是true和false。例如：
```html
<div th:if="${user.isAdmin()} == false"> ...
```
* Null 常量
```html
<div th:if="${variable.something} == null"> ...
```
### 字面量替换
除了使用'...' + ${}来连接字面量和变量外，还可以使用|...|来代替，比如：
```html
<p th:utext="|hello,${session.user.name},your age is ${session.user.age}|"></p>
```
等价于：
```html
<p th:utext="'hello,'+${session.user.name}+',your age is '+${session.user.age}"></p>
```
注意： `在| ... |字面替换中只允许有变量表达式${...}`

### 条件表达式
条件表达式实际上就是三目运算符。比如：
```html
<tr th:class="${row.even}? 'even' : 'odd'">
  ...
</tr>
```
条件表达式也可以使用括号嵌套：
```html
<tr th:class="${row.even}? (${row.first}? 'first' : 'even') : 'odd'">
  ...
</tr>
```
else表达式也可以省略，在这种情况下，如果条件为false，则返回空值：
```html
<tr th:class="${row.even}? 'even'">
  ...
</tr>
```
### 默认表达式
默认表达式是一种特殊类型的条件值，不带then部分。比如：
```html
<p th:utext="${session.user.sex} ?: 'sex is unknown'"></p>
```
表示，当${session.user.sex}为null时，值为sex is unknown，否则为表达式的值。这就好像为表达式指定了一个默认值一样。其等价于：
```html
<p th:utext="${session.user.sex != null} ? ${session.user.sex}: 'sex is unknown'"></p>
```





---
---
# 表达式工具类

Thymeleaf默认提供了丰富的表达式工具类，这里列举一些常用的工具类。

### Objects工具类
```
/*
 * 当obj不为空时，返回obj，否则返回default默认值
 * 其同样适用于数组、列表或集合
 */
${#objects.nullSafe(obj,default)}
${#objects.arrayNullSafe(objArray,default)}
${#objects.listNullSafe(objList,default)}
${#objects.setNullSafe(objSet,default)}
```
### String工具类
```
/*
 * Null-safe toString()
 */
${#strings.toString(obj)}   // 也可以是 array*、list* 或 set*

/*
 * 检查String是否为空(或null)。在检查之前执行trim()操作也同样适用于数组、列表或集合
 */
${#strings.isEmpty(name)}
${#strings.arrayIsEmpty(nameArr)}
${#strings.listIsEmpty(nameList)}
${#strings.setIsEmpty(nameSet)}

/*
 * 对字符串执行“isEmpty()”检查, 如果为false则返回它, 如果为true则默认为另一个指定的字符串。
 * 也同样适用于数组、列表或集合
 */
${#strings.defaultString(text,default)}
${#strings.arrayDefaultString(textArr,default)}
${#strings.listDefaultString(textList,default)}
${#strings.setDefaultString(textSet,default)}

/*
 * 检查字符串中是否包含片段，比如 ${#strings.containsIgnoreCase(user.name,'kang')}
 * 也同样适用于数组、列表或集合
 */
${#strings.contains(name,'ez')}               // 也可以是 array*、list* 或 set*
${#strings.containsIgnoreCase(name,'ez')}     // 也可以是 array*、list* 或 set*

/*
 * 检查字符串是否以片段开始或结束
 * 也同样适用于数组、列表或集合
 */
${#strings.startsWith(name,'Don')}            // 也可以是 array*、list* 或 set*
${#strings.endsWith(name,endingFragment)}     // 也可以是 array*、list* 或 set*

/*
 * 子串相关操作
 * 也同样适用于数组、列表或集合
 */
${#strings.indexOf(name,frag)}                // 也可以是 array*、list* 或 set*
${#strings.substring(name,3,5)}               // 也可以是 array*、list* 或 set*
${#strings.substringAfter(name,prefix)}       // 也可以是 array*、list* 或 set*
${#strings.substringBefore(name,suffix)}      // 也可以是 array*、list* 或 set*
${#strings.replace(name,'las','ler')}         // 也可以是 array*、list* 或 set*

/*
 * 附加和前置
 * 也同样适用于数组、列表或集合
 */
${#strings.prepend(str,prefix)}               // 也可以是 array*、list* 或 set*
${#strings.append(str,suffix)}                // 也可以是 array*、list* 或 set*

/*
 * 大小写转换
 * 也同样适用于数组、列表或集合
 */
${#strings.toUpperCase(name)}                 // 也可以是 array*、list* 或 set*
${#strings.toLowerCase(name)}                 // 也可以是 array*、list* 或 set*

/*
 * 拆分和拼接
 */
${#strings.arrayJoin(namesArray,',')}
${#strings.listJoin(namesList,',')}
${#strings.setJoin(namesSet,',')}
${#strings.arraySplit(namesStr,',')}          // 返回String []
${#strings.listSplit(namesStr,',')}           // 返回List<String>
${#strings.setSplit(namesStr,',')}            // 返回Set<String>

/*
 * Trim
 * 也同样适用于数组、列表或集合
 */
${#strings.trim(str)}                         // 也可以是 array*、list* 或 set*

/*
 * 计算长度
 * 也同样适用于数组、列表或集合
 */
${#strings.length(str)}                       // 也可以是 array*、list* 或 set*

/*
 * 缩写文本, 使其最大大小为n。如果文本较大, 它将被剪辑并在末尾附加“...”
 * 也同样适用于数组、列表或集合
 */
${#strings.abbreviate(str,10)}                // 也可以是 array*、list* 或 set*

/*
 * 将第一个字符转换为大写(反之亦然)
 */
${#strings.capitalize(str)}                   // 也可以是 array*、list* 或 set*
${#strings.unCapitalize(str)}                 // 也可以是 array*、list* 或 set*

/*
 * 将每个单词的第一个字符转换为大写
 */
${#strings.capitalizeWords(str)}              // 也可以是 array*、list* 或 set*
${#strings.capitalizeWords(str,delimiters)}   // 也可以是 array*、list* 或 set*

/*
 * 转义字符串
 */
${#strings.escapeXml(str)}                    // 也可以是 array*、list* 或 set*
${#strings.escapeJava(str)}                   // 也可以是 array*、list* 或 set*
${#strings.escapeJavaScript(str)}             // 也可以是 array*、list* 或 set*

${#strings.unescapeJava(str)}                 // 也可以是 array*、list* 或 set*
${#strings.unescapeJavaScript(str)}           // 也可以是 array*、list* 或 set*

/*
 * 空安全比较和连接
 */
${#strings.equals(first, second)}
${#strings.equalsIgnoreCase(first, second)}
${#strings.concat(values...)}
${#strings.concatReplaceNulls(nullValue, values...)}

/*
 * 随机数
 */
${#strings.randomAlphanumeric(count)}
```

### Dates工具类
```
/*
 * 使用标准区域设置格式格式化日期
 * 也同样适用于数组、列表或集合
 */
${#dates.format(date)}
${#dates.arrayFormat(datesArray)}
${#dates.listFormat(datesList)}
${#dates.setFormat(datesSet)}

/*
 * 使用ISO8601格式格式化日期
  * 也同样适用于数组、列表或集合
 */
${#dates.formatISO(date)}
${#dates.arrayFormatISO(datesArray)}
${#dates.listFormatISO(datesList)}
${#dates.setFormatISO(datesSet)}

/*
 * 使用指定的格式格式化日期，比如 ${#dates.format(date,'yyyy-MM-dd HH:mm:ss')}
 * 也同样适用于数组、列表或集合
 */
${#dates.format(date, 'dd/MMM/yyyy HH:mm')}
${#dates.arrayFormat(datesArray, 'dd/MMM/yyyy HH:mm')}
${#dates.listFormat(datesList, 'dd/MMM/yyyy HH:mm')}
${#dates.setFormat(datesSet, 'dd/MMM/yyyy HH:mm')}

/*
 * 获取日期属性
 * 也同样适用于数组、列表或集合
 */
${#dates.day(date)}                    // 也可以是 arrayDay(...), listDay(...)之类的
${#dates.month(date)}                  // 也可以是 arrayMonth(...), listMonth(...)之类的
${#dates.monthName(date)}              // 也可以是 arrayMonthName(...), listMonthName(...)之类的
${#dates.monthNameShort(date)}         // 也可以是 arrayMonthNameShort(...), listMonthNameShort(...)之类的
${#dates.year(date)}                   // 也可以是 arrayYear(...), listYear(...)之类的
${#dates.dayOfWeek(date)}              // 也可以是 arrayDayOfWeek(...), listDayOfWeek(...)之类的
${#dates.dayOfWeekName(date)}          // 也可以是 arrayDayOfWeekName(...), listDayOfWeekName(...)之类的
${#dates.dayOfWeekNameShort(date)}     // 也可以是 arrayDayOfWeekNameShort(...), listDayOfWeekNameShort(...)之类的
${#dates.hour(date)}                   // 也可以是 arrayHour(...), listHour(...)之类的
${#dates.minute(date)}                 // 也可以是 arrayMinute(...), listMinute(...)之类的
${#dates.second(date)}                 // 也可以是 arraySecond(...), listSecond(...)之类的
${#dates.millisecond(date)}            // 也可以是 arrayMillisecond(...), listMillisecond(...)之类的

/*
 * 根据year,month,day创建日期(java.util.Date)对象，比如 ${#dates.create('2008','08','08')}
 */
${#dates.create(year,month,day)}
${#dates.create(year,month,day,hour,minute)}
${#dates.create(year,month,day,hour,minute,second)}
${#dates.create(year,month,day,hour,minute,second,millisecond)}

/*
 * 创建当前日期和时间创建日期(java.util.Date)对象，比如 ${#dates.format(#dates.createNow(),'yyyy-MM-dd HH:mm:ss')}
 */
${#dates.createNow()}

${#dates.createNowForTimeZone()}

/*
 * 创建当前日期创建一个日期(java.util.Date)对象(时间设置为00:00)
 */
${#dates.createToday()}

${#dates.createTodayForTimeZone()}
```

### Calendars工具类
```
/*
 * 使用标准区域设置格式格式化日历
 * 也同样适用于数组、列表或集合
 */
${#calendars.format(cal)}
${#calendars.arrayFormat(calArray)}
${#calendars.listFormat(calList)}
${#calendars.setFormat(calSet)}

/*
 * 使用ISO8601格式格式化日历
 * 也同样适用于数组、列表或集合
 */
${#calendars.formatISO(cal)}
${#calendars.arrayFormatISO(calArray)}
${#calendars.listFormatISO(calList)}
${#calendars.setFormatISO(calSet)}

/*
 * 使用指定的格式格式化日历
 * 也同样适用于数组、列表或集合
 */
${#calendars.format(cal, 'dd/MMM/yyyy HH:mm')}
${#calendars.arrayFormat(calArray, 'dd/MMM/yyyy HH:mm')}
${#calendars.listFormat(calList, 'dd/MMM/yyyy HH:mm')}
${#calendars.setFormat(calSet, 'dd/MMM/yyyy HH:mm')}

/*
 * 获取日历属性
 * 也同样适用于数组、列表或集合
 */
${#calendars.day(date)}                // 也可以是 arrayDay(...), listDay(...)之类的
${#calendars.month(date)}              // 也可以是 arrayMonth(...), listMonth(...)之类的
${#calendars.monthName(date)}          // 也可以是 arrayMonthName(...), listMonthName(...)之类的
${#calendars.monthNameShort(date)}     // 也可以是 arrayMonthNameShort(...), listMonthNameShort(...)之类的
${#calendars.year(date)}               // 也可以是 arrayYear(...), listYear(...)之类的
${#calendars.dayOfWeek(date)}          // 也可以是 arrayDayOfWeek(...), listDayOfWeek(...)之类的
${#calendars.dayOfWeekName(date)}      // 也可以是 arrayDayOfWeekName(...), listDayOfWeekName(...)之类的
${#calendars.dayOfWeekNameShort(date)} // 也可以是 arrayDayOfWeekNameShort(...), listDayOfWeekNameShort(...)之类的
${#calendars.hour(date)}               // 也可以是 arrayHour(...), listHour(...)之类的
${#calendars.minute(date)}             // 也可以是 arrayMinute(...), listMinute(...)之类的
${#calendars.second(date)}             // 也可以是 arraySecond(...), listSecond(...)之类的
${#calendars.millisecond(date)}        // 也可以是 arrayMillisecond(...), listMillisecond(...)之类的

/*
 * 从其组件创建日历(java.util.Calendar)对象
 */
${#calendars.create(year,month,day)}
${#calendars.create(year,month,day,hour,minute)}
${#calendars.create(year,month,day,hour,minute,second)}
${#calendars.create(year,month,day,hour,minute,second,millisecond)}

${#calendars.createForTimeZone(year,month,day,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,second,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,second,millisecond,timeZone)}

/*
 * 为当前日期和时间创建一个日历(java.util.Calendar)对象
 */
${#calendars.createNow()}

${#calendars.createNowForTimeZone()}

/*
 * 为当前日期创建日历(java.util.Calendar)对象(时间设置为00:00)
 */
${#calendars.createToday()}

${#calendars.createTodayForTimeZone()}
```

### Numbers工具类

```
/*
 * ==========================
 * 格式化整数
 * ==========================
 */

/* 
 * 设置最小整数位数。
 * 也同样适用于数组、列表或集合
 */
${#numbers.formatInteger(num,3)}
${#numbers.arrayFormatInteger(numArray,3)}
${#numbers.listFormatInteger(numList,3)}
${#numbers.setFormatInteger(numSet,3)}


/* 
 * 设置最小整数位数和千位分隔符：
 * 'POINT'、'COMMA'、'WHITESPACE'、'NONE' 或 'DEFAULT'(根据本地化)。
 * 也同样适用于数组、列表或集合
 */
${#numbers.formatInteger(num,3,'POINT')}
${#numbers.arrayFormatInteger(numArray,3,'POINT')}
${#numbers.listFormatInteger(numList,3,'POINT')}
${#numbers.setFormatInteger(numSet,3,'POINT')}


/*
 * ==========================
 * 格式化十进制数
 * ==========================
 */

/*
 * 设置最小整数数字和(精确的)十进制数字。
 * 也同样适用于数组、列表或集合
 */
${#numbers.formatDecimal(num,3,2)}
${#numbers.arrayFormatDecimal(numArray,3,2)}
${#numbers.listFormatDecimal(numList,3,2)}
${#numbers.setFormatDecimal(numSet,3,2)}

/*
 * 设置最小整数数字和(精确的)小数位数, 以及小数分隔符。
 * 也同样适用于数组、列表或集合
 */
${#numbers.formatDecimal(num,3,2,'COMMA')}
${#numbers.arrayFormatDecimal(numArray,3,2,'COMMA')}
${#numbers.listFormatDecimal(numList,3,2,'COMMA')}
${#numbers.setFormatDecimal(numSet,3,2,'COMMA')}

/*
 * 设置最小整数数字和(精确的)十进制数字, 以及千位和十进制分隔符。
 * 也同样适用于数组、列表或集合
 */
${#numbers.formatDecimal(num,3,'POINT',2,'COMMA')}
${#numbers.arrayFormatDecimal(numArray,3,'POINT',2,'COMMA')}
${#numbers.listFormatDecimal(numList,3,'POINT',2,'COMMA')}
${#numbers.setFormatDecimal(numSet,3,'POINT',2,'COMMA')}

/*
 * ==========================
 * 实用方法
 * ==========================
 */

/*
 * 创建一个从x到y的整数序列(数组)
 */
${#numbers.sequence(from,to)}
${#numbers.sequence(from,to,step)}
```

比如：

```
<p th:utext="${#numbers.formatInteger(0.1024,3)}"></p>
<p th:utext="${#numbers.formatInteger(1.024,3)}"></p>
<p th:utext="${#numbers.formatInteger(10.24,3)}"></p>
<p th:utext="${#numbers.formatInteger(102.4,3)}"></p>
```
> POINT指的是.，COMMA指的是,，WHITESPACE指的是空格。三个数位为一组，使用指定的分隔符分割。比如1.024并不是小数，而是使用了.分隔的1024。

### Booleans工具类
```
/*
 * 评估条件, 类似于 th:if 标签
 * 也同样适用于数组、列表或集合
 */
${#bools.isTrue(obj)}
${#bools.arrayIsTrue(objArray)}
${#bools.listIsTrue(objList)}
${#bools.setIsTrue(objSet)}

/*
 * 用否定来评估条件
 * 也同样适用于数组、列表或集合
 */
${#bools.isFalse(cond)}
${#bools.arrayIsFalse(condArray)}
${#bools.listIsFalse(condList)}
${#bools.setIsFalse(condSet)}

/*
 * 评估条件并执行与操作
 * 接收数组、列表或集合作为参数
 */
${#bools.arrayAnd(condArray)}
${#bools.listAnd(condList)}
${#bools.setAnd(condSet)}

/*
 * 评估条件并执行或操作
 * 接收数组、列表或集合作为参数
 */
${#bools.arrayOr(condArray)}
${#bools.listOr(condList)}
${#bools.setOr(condSet)}

```

### Arrays工具类
```
/*
 * 转换为数组, 试图推断数组组件类。注意, 如果结果数组为空, 或者目标对象的元素不是全部相同的类, 则
 * 此方法将返回Object []。
 */
${#arrays.toArray(object)}

/*
 * 转换为指定组件类的数组。
 */
${#arrays.toStringArray(object)}
${#arrays.toIntegerArray(object)}
${#arrays.toLongArray(object)}
${#arrays.toDoubleArray(object)}
${#arrays.toFloatArray(object)}
${#arrays.toBooleanArray(object)}

/*
* 计算数组长度
 */
${#arrays.length(array)}

/*
 * 检查数组是否为空
 */
${#arrays.isEmpty(array)}

/*
 * 检查数组中是否包含元素或元素集合
 */
${#arrays.contains(array, element)}
${#arrays.containsAll(array, elements)}
```

### Lists工具类

```
/*
 * 转化为 list
 */
${#lists.toList(object)}

/*
 * 计算大小
 */
${#lists.size(list)}

/*
 */
${#lists.isEmpty(list)}

/*
 * 检查list中是否包含元素或元素集合
 */
${#lists.contains(list, element)}
${#lists.containsAll(list, elements)}

/*
 * 排序给定列表的副本。列表的成员必须
 * 实现comparable, 或者必须定义comparator。
 */
${#lists.sort(list)}
${#lists.sort(list, comparator)}
```

### Sets工具类
```
/*
 * 转化为 to set
 */
${#sets.toSet(object)}

/*
 * 计算大小
 */
${#sets.size(set)}

/*
 * 检查set是否为empty
 */
${#sets.isEmpty(set)}

/*
 * 检查set中是否包含元素或元素集合
 */
${#sets.contains(set, element)}
${#sets.containsAll(set, elements)}
```

### Maps工具类

```
/*
 * 计算大小
 */
${#maps.size(map)}

/*
 * 检查map是否为空
 */
${#maps.isEmpty(map)}

/*
 * 检查map中是否包含key/s或value/s
 */
${#maps.containsKey(map, key)}
${#maps.containsAllKeys(map, keys)}
${#maps.containsValue(map, value)}
${#maps.containsAllValues(map, value)}
```

`注意事项`:
值得注意的是，在使用工具类对某个表达式进行处理时候，你可能会写成：

```${#strings.isEmpty(${session.user.name})}。```
实际上这种写法是错误的，将抛出异常。正确的写法为：

```${#strings.isEmpty(session.user.name)}。```



---
---

# 迭代
在Thymeleaf中，使用`th:each`标签可对集合类型进行迭代，支持的类型有：

1.任何实现了`java.util.List`的对象；

2.任何实现了`java.util.Iterable`的对象；

3.任何实现了`java.util.Enumeration`的对象；

4.任何实现了`java.util.Iterator`的对象;

5.任何实现了`java.util.Map`的对象。当迭代maps时，迭代变量是`java.util.Map.Entry`类型；

6.任何数组。

一个简单的例子：
```html
<tr th:each="prod : ${prods}">
   <td th:text="${prod.name}">Onions</td>
   <td th:text="${prod.price}">2.41</td>
</tr>
```
其中${prods}为迭代值，prod为迭代变量。除此之外，我们还可以通过状态变量获取迭代的状态信息，比如：
```html
<tr th:each="prod,stat : ${prods}"> ...
```
其中stat就是状态变量。默认为迭代变量加上Stat后缀，在本例中，不直接申明stat，则状态变量名称为prodStat。状态变量包含以下信息：

1.index，当前迭代下标，从0开始；

2.count，当前迭代位置，从1开始；

3.size，迭代变量中的总计数量；

4.current，每次迭代的迭代变量；

5.even/odd，当前迭代是偶数还是奇数；

6.first，当前迭代的是不是第一个；

7.last，当前迭代的是不是最后一个；

例子：
```html
<div th:each="habbits,stat : ${user.habbit}">
   <span th:text="${habbits}"></span>
   <span th:text="${stat.index}"></span>
   <span th:text="${stat.count}"></span>
   <span th:text="${stat.size}"></span>
   <span th:text="${stat.current}"></span>
   <span th:text="${stat.even}"></span>
   <span th:text="${stat.first}"></span>
   <span th:text="${stat.last}"></span>
</div>
```
页面显示如下：
![槑槑离](https://upload-images.jianshu.io/upload_images/3710706-21caa298dbdbf66c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
---

# Thymeleaf 条件语句
* `if 与 unless`
假如现在有一个商品列表，当商品有评论时，显示view按钮，否则不显示。这时候就可以使用Thymeleaf的th:if标签来实现:
```html
<a href="comments.html"
   th:href="@{/product/comments(prodId=${prod.id})}" 
   th:if="${not #lists.isEmpty(prod.comments)}">view</a>
```
当prod.comments不为空时，页面将渲染出该<a>标签。
另外，th:if有一个反向属性th:unless，用于代替上面的not：
```html
<a href="comments.html"
   th:href="@{/comments(prodId=${prod.id})}" 
   th:unless="${#lists.isEmpty(prod.comments)}">view</a>
```

* `switch`
Thymeleaf中的th:switch和其他语言的switch case语句差不多：

```html
<div th:switch="${user.role}">
  <p th:case="'admin'">User is an administrator</p>
  <p th:case="'manager'">User is a manager</p>
</div>
```
th:case="*"表示默认选项，相当于default：

```html
<div th:switch="${user.role}">
  <p th:case="'admin'">User is an administrator</p>
  <p th:case="'manager'">User is a manager</p>
  <p th:case="*">User is some other thing</p>
</div>
```

---
---

# Thymeleaf 模板布局
### th:include,th:insert和th:replace

在模板的编写中，通常希望能够引入别的模板片段，比如通用的头部和页脚。Thymeleaf模板引擎的`th:include`，`th:insert`和`th:replace`属性可以轻松的实现该需求。不过从Thymeleaf 3.0版本后， 不再推荐使用`th:include`属性。

在index.html页面路径下创建一个footer.html：
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
 <body>
   <footer th:fragment="copy">
     &copy; 2016 - 2017  MrBird Powered by Hexo
   </footer>
 </body>
</html>
```

在footer.html中，使用`th:fragment`属性定义了`<footer>`片段，然后在index.html中引用它：

```html
<div th:include="~{footer :: copy}"></div> 
<div th:insert="~{footer :: copy}"></div> 
<div th:replace="~{footer :: copy}"></div>
```
其中footer为被引用的模板名称（templatename），copy为th:fragment标记的片段名称（selector），~{...}称为片段表达式，由于其不是一个复杂的片段表达式，所以可以简写为：
```html
<div th:include="footer :: copy"></div>
```
页面显示如下：
![image.png](https://upload-images.jianshu.io/upload_images/3710706-d855bdb676a670d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过观察渲染出的源码可发现th:include，th:insert和th:replace的区别所在：

```html
<div>
    &copy; 2016 - 2017  MrBird Powered by Hexo
</div> 
<div>
    <footer>
        &copy; 2016 - 2017  MrBird Powered by Hexo
    </footer>
</div> 
<footer>
    &copy; 2016 - 2017  MrBird Powered by Hexo
</footer>
```
注意：`引用本页面的片段可以略去templatename，或者使用this来代替。`

### 引用没有th:fragment的片段
如果片段不包含th:fragment属性，我们可以使用CSS选择器来选中该片段，如：
```html
<div id="copy-section">
  &copy; 2011 The Good Thymes Virtual Grocery
</div>
```
引用方式：
```html
<div th:insert="~{footer :: #copy-section}"></div>
```


### 在th:fragment中使用参数
使用th:fragment定义的片段可以指定一组参数：
```html
<div th:fragment="frag (onevar,twovar)">
	<p th:text="${onevar} + ' - ' + ${twovar}">...</p>
</div>
```
然后在引用的时候给这两个参数赋值，有如下两种方式：
```html
<div th:replace="this :: frag ('value1','value2')"></div>
<div th:replace="this :: frag (onevar='value_one',twovar='value_two')"></div>
```
对于第二种方式，onevar和twovar的顺序不重要，并且使用第二种方式引用片段时，片段可以简写为：
```html
<div th:fragment="frag">
   <p th:text="${onevar} + ' - ' + ${twovar}">...</p>
</div>
```


### th:remove
* th:remove用于删除片段，支持的属性值有：
* all：删除标签及其所有子标签。
* body：不删除包含的标签，但删除其所有子代。
* tag：删除包含的标签，但不要删除其子标签。
* all-but-first：删除除第一个之外的所有包含标签的子标签。
* none。

比如有如下片段：

```html
<table border="1">
   <tr>
      <th>NAME</th>
      <th>PRICE</th>
      <th>IN STOCK</th>
      <th>COMMENTS</th>
   </tr>
   <tr th:remove="value">
      <td>Mild Cinnamon</td>
      <td>1.99</td>
      <td>yes</td>
      <td>
         <span>3</span> comment/s
         <a href="comments.html">view</a>
      </td>
   </tr>
</table>
```
当value为all时，页面渲染为：
```html
<table border="1">
   <tr>
      <th>NAME</th>
      <th>PRICE</th>
      <th>IN STOCK</th>
      <th>COMMENTS</th>
   </tr>
</table>
```
当value为body时，页面渲染为：
```html
<table border="1">
   <tr>
      <th>NAME</th>
      <th>PRICE</th>
      <th>IN STOCK</th>
      <th>COMMENTS</th>
   </tr>
   <tr>
   </tr>
</table>
```
当value为tag时，页面渲染为：
```html
<table border="1">
   <tr>
      <th>NAME</th>
      <th>PRICE</th>
      <th>IN STOCK</th>
      <th>COMMENTS</th>
   </tr>
      <td>Mild Cinnamon</td>
      <td>1.99</td>
      <td>yes</td>
      <td>
         <span>3</span> comment/s
         <a href="comments.html">view</a>
      </td>
</table>
```
当value为all-but-first时，页面渲染为：

```html
<table border="1">
   <tr>
      <th>NAME</th>
      <th>PRICE</th>
      <th>IN STOCK</th>
      <th>COMMENTS</th>
   </tr>
   <tr th:remove="value">
      <td>Mild Cinnamon</td>
   </tr>
</table>
```

---
---


# 局部变量
在Thymeleaf模板引擎中，使用` th:with`属性来声明一个局部变量
```html
<div th:with="firstPer=${persons[0]}">
  <p>The name of the first person is <span th:text="${firstPer.name}">Julius Caesar</span>.</p>
</div>
```
在上面div中，`th:width`属性声明了一个名为firstPer的局部变量，内容为`${persons[0]}`。该局部变量的作用域为整个div内。

也可以一次性定义多个变量：
```html
<div th:with="firstPer=${persons[0]},secondPer=${persons[1]}">
  <p>
    The name of the first person is <span th:text="${firstPer.name}">Julius Caesar</span>.
  </p>
  <p>
    But the name of the second person is 
    <span th:text="${secondPer.name}">Marcus Antonius</span>.
  </p>
</div>
```
th:with属性允许重用在同一个属性中定义的变量：
```html
<div th:with="company=${user.company + ' Co.'},account=${accounts[company]}">...</div>
```

---

参考：[https://mrbird.cc/Thymeleaf-%E5%B1%80%E9%83%A8%E5%8F%98%E9%87%8F.html](https://mrbird.cc/Thymeleaf-%E5%B1%80%E9%83%A8%E5%8F%98%E9%87%8F.html)
