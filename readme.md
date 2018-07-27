

1.目的：
>解决过往在业务里面写一大堆的校验程序，使程序的可读性特别差，而且耦合度很高，这就很不符合程序设计原则，所以本人就该问题做了一个统一处理的工具。


2.技术栈：
>java反射、策略模式、java8

3.demo:   
   >1.定义异常的接口：
   
   ```
   public interface BaseServiceErrors  {
   // 添加系统各系统的自定义异常    
     ErrorMeta PARAM_INVALID = new ErrorMeta(70002,"请求参数不能为空");    
     // 参数为空校验，采用策略模式    
     ErrorMeta strategy(String para);    
     //属性匹配空值异常     
     static ErrorMeta match(Map<String, ErrorMeta> map,String para){         
        if (map.containsKey(para)) {             
            return map.get(para);         
        }
                 return null;     
     } 
   }
   ```

>2.可以有多个实现类，在这只举一个实现例子

```
   public class QuestionError implements BaseServiceErrors { 
      private Map<String, ErrorMeta> map = new HashMap<>(); 
      public QuestionError() {         
        this.map.put("type", new ErrorMeta(70001, "题目类型不能为空"));     
      } 
      @Override
       public ErrorMeta strategy(String para) {         
        return BaseServiceErrors.match(map,para);     
      }
     }
```

>3.定义策略类：

```
public class StrategyImpl { 
    private BaseServiceErrors baseServiceErrors; 
    public StrategyImpl(Object baseServiceErrors) {      
       this.baseServiceErrors = (BaseServiceErrors) baseServiceErrors;     
    } 
    public ErrorMeta checkPara(String para) {         
       return this.baseServiceErrors.strategy(para);     
    }
}
```
>4.具体业务对象类：

```
@Data 
public class QuestionEntity implements Serializable {      
    private static final long serialVersionUID = -1132588413262899197L;      
    private String description;      
    private Integer type;      
    private Integer currentPage;      
    private Integer pageSize; 
}
```
>5.工具类入口：

```
/**  
 * @param o   参数对象  
 * @param o2  具体策略对象  
 * @throws IllegalAccessException  
 * @throws InstantiationException  
 */ 
public static void checkFieldVal(Object o,Object o2) throws IllegalAccessException, InstantiationException {     
    // 得到类对象     
    Class<?> aClass = o.getClass();     
    // 获取属性集合     
    Field[] fields = aClass.getDeclaredFields();     
    // 获取策略对象     
    StrategyImpl strate = new StrategyImpl( o2.getClass().newInstance());     
    for (Field f : fields) {         
        f.setAccessible(true); // 设置些属性是可以访问的         
        // 执行具体策略         
        ErrorMeta errorMeta = strate.checkPara(f.getName());         
        if (errorMeta != null) {             
            throw new LogicException(errorMeta);         
        }     
    } 
}
```
至此，一套公共的异常处理(本例只是处理空值异常，可根据具体业务横向扩展)一切准备就绪。

下面做具体演示

```
public static void main(String[] args) { 
    QuestionEntity entity = new QuestionEntity(); 
    entity.setCurrentPage(1);     
    entity.setDescription("asdsd"); 
    try {     
      checkFieldVal(entity,new QuestionError()); 
      System.out.println("asdasdsadas"); 
    } catch (Exception e) { 
      e.printStackTrace(); 
    }  
}
```
没有给type赋值，抛出

![
](1.jpg)

说明抛出了业务想要的异常，然后可根据具体场景做捕获操作返回给前端，
该工具类业务方只需要添加以下代码:

---

**checkFieldVal('参数对象','参数策略对象');**

---

一行代码即可实现，解耦，可读性高，开箱即用，使用非常方便。
