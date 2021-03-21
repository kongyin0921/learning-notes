# Mybatis

## 125.mybatis 中 #{}和 ${}的区别是什么？

**#{}是预编译处理**，**$ {}是字符串替换**。**mybatis在处理#{}时**，会将sql中的**#{}替换为?号**，调用PreparedStatement的**set**方法来赋值；mybatis在处理时 ， 就 是 把 {}替换成变量的值。使用**#{}**可以有效的**防止SQL注入**，**提高系统安全性**。

对于这个题目我感觉要抓住两点：
（1）$ 符号一般用来当作占位符，常使用Linux脚本的人应该对此有更深的体会吧。既然是占位符，当然就是被用来替换的。知道了这点就能很容易区分$和#，从而不容易记错了。
（2）预编译的机制。预编译是提前对SQL语句进行预编译，而其后注入的参数将不会再进行SQL编译。我们知道，**SQL注入是发生在编译的过程**中，因为恶意注入了某些特殊字符，最后被编译成了恶意的执行操作。**而预编译机制则可以很好的防止SQL注入**。

## 126.mybatis 有几种分页方式？

#### 1. 内存分页

内存分页的原理比较sb，就是一次性查询数据库中所有满足条件的记录，将这些数据临时保存在集合中，再通过List的subList方法，获取到满足条件的记录，由于太sb，直接忽略该种方式的分页。

#### 2. 物理分页

​    在了解到通过内存分页的缺陷后，我们发现不能每次都对数据库中的所有数据都检索。然后在程序中对获取到的大量数据进行二次操作，这样对空间和性能都是极大的损耗。所以我们希望能直接在数据库语言中只检索符合条件的记录，不需要在通过程序对其作处理。这时，物理分页技术横空出世。

​    物理分页是借助sql语句进行分页，比如mysql是通过limit关键字，oracle是通过rownum等；其中mysql的分页语法如下：

```sql
select* from table limit 0,30
```

### MyBatis 分页

#### 1.借助sql进行分页

通过sql语句进行分页的实现很简单,我们先在StudentMapper接口中添加sql语句的查询方法，如下：

```java
List<Student>queryStudentsBySql(@Param("offset") intoffset, @Param("limit") intlimit);
```

StudentMapper.xml 配置如下：

```xml
<selectid="queryStudentsBySql"parameterType="map"resultMap="studentmapper">
      select * from student limit #{offset} , #{limit}
</select>
```

客户端使用的时候如下：

```java
publicList<Student>queryStudentsBySql(intoffset, intpageSize) {
       returnstudentMapper.queryStudentsBySql(offset,pageSize);
}
```

sql分页语句如下：`select * from table limit index, pageSize;`

缺点：虽然这里实现了按需查找，每次检索得到的是指定的数据。但是每次在分页的时候都需要去编写limit语句，很冗余, 其次另外如果想知道总条数，还需要另外写sql去统计查询。而且不方便统一管理，维护性较差。所以我们希望能够有一种更方便的分页实现。

#### 2. 拦截器分页

拦截器的一个作用就是我们可以拦截某些方法的调用，我们可以选择在这些被拦截的方法执行前后加上某些逻辑，也可以在执行这些被拦截的方法时执行自己的逻辑而不再执行被拦截的方法。Mybatis拦截器设计的一个初衷就是为了供用户在某些时候可以实现自己的逻辑而不必去动Mybatis固有的逻辑。打个比方，对于Executor，Mybatis中有几种实现：BatchExecutor、ReuseExecutor、SimpleExecutor和CachingExecutor。这个时候如果你觉得这几种实现对于Executor接口的query方法都不能满足你的要求，那怎么办呢？是要去改源码吗？当然不。我们可以建立一个Mybatis拦截器用于拦截Executor接口的query方法，在拦截之后实现自己的query方法逻辑，之后可以选择是否继续执行原来的query方法。

#####  Interceptor接口

对于拦截器Mybatis为我们提供了一个Interceptor接口，通过实现该接口就可以定义我们自己的拦截器。我们先来看一下这个接口的定义：

```java
packageorg.apache.ibatis.plugin;  
importjava.util.Properties;  
  
publicinterfaceInterceptor{  
  
 Objectintercept(Invocationinvocation) throwsThrowable;  
  
 Objectplugin(Objecttarget);  
  
 voidsetProperties(Propertiesproperties);  
  
}  
```

我们可以看到在该接口中一共定义有三个方法，intercept、plugin和setProperties。plugin方法是拦截器用于封装目标对象的，通过该方法我们可以返回目标对象本身，也可以返回一个它的代理。当返回的是代理的时候我们可以对其中的方法进行拦截来调用intercept方法，当然也可以调用其他方法，这点将在后文讲解。setProperties方法是用于在Mybatis配置文件中指定一些属性的。

​    定义自己的Interceptor最重要的是要实现plugin方法和intercept方法，在plugin方法中我们可以决定是否要进行拦截进而决定要返回一个什么样的目标对象。而intercept方法就是要进行拦截的时候要执行的方法。

​    对于plugin方法而言，其实Mybatis已经为我们提供了一个实现。Mybatis中有一个叫做Plugin的类，里面有一个静态方法wrap(Object target,Interceptor interceptor)，通过该方法可以决定要返回的对象是目标对象还是对应的代理。这里我们先来看一下Plugin的源码：

```java
packageorg.apache.ibatis.plugin;  
  
importjava.lang.reflect.InvocationHandler;  
importjava.lang.reflect.Method;  
importjava.lang.reflect.Proxy;  
importjava.util.HashMap;  
importjava.util.HashSet;  
importjava.util.Map;  
importjava.util.Set;  
  
importorg.apache.ibatis.reflection.ExceptionUtil;  
  
publicclassPluginimplementsInvocationHandler{  
  
 privateObjecttarget;  
 privateInterceptorinterceptor;  
 privateMap<Class<?>, Set<Method>>signatureMap;  
  
 privatePlugin(Objecttarget, Interceptorinterceptor, Map<Class<?>, Set<Method>>signatureMap) {  
   this.target=target;  
   this.interceptor=interceptor;  
   this.signatureMap=signatureMap;  
}  
  
 publicstaticObjectwrap(Objecttarget, Interceptorinterceptor) {  
   Map<Class<?>, Set<Method>>signatureMap=getSignatureMap(interceptor);  
   Class<?>type=target.getClass();  
   Class<?>[] interfaces=getAllInterfaces(type, signatureMap);  
   if(interfaces.length>0) {  
     returnProxy.newProxyInstance(  
         type.getClassLoader(),  
         interfaces,  
         newPlugin(target, interceptor, signatureMap));  
  }  
   returntarget;  
}  
  
 publicObjectinvoke(Objectproxy, Methodmethod, Object[] args) throwsThrowable{  
   try{  
     Set<Method>methods=signatureMap.get(method.getDeclaringClass());  
     if(methods!=null&&methods.contains(method)) {  
       returninterceptor.intercept(newInvocation(target, method, args));  
    }  
     returnmethod.invoke(target, args);  
  } catch(Exceptione) {  
     throwExceptionUtil.unwrapThrowable(e);  
  }  
}  
  
 privatestaticMap<Class<?>, Set<Method>>getSignatureMap(Interceptorinterceptor) {  
   InterceptsinterceptsAnnotation=interceptor.getClass().getAnnotation(Intercepts.class);  
   if(interceptsAnnotation==null) { // issue #251  
     thrownewPluginException("No @Intercepts annotation was found in interceptor "+interceptor.getClass().getName());       
  }  
   Signature[] sigs=interceptsAnnotation.value();  
   Map<Class<?>, Set<Method>>signatureMap=newHashMap<Class<?>, Set<Method>>();  
   for(Signaturesig: sigs) {  
     Set<Method>methods=signatureMap.get(sig.type());  
     if(methods==null) {  
       methods=newHashSet<Method>();  
       signatureMap.put(sig.type(), methods);  
    }  
     try{  
       Methodmethod=sig.type().getMethod(sig.method(), sig.args());  
       methods.add(method);  
    } catch(NoSuchMethodExceptione) {  
       thrownewPluginException("Could not find method on "+sig.type() +" named "+sig.method() +". Cause: "+e, e);  
    }  
  }  
   returnsignatureMap;  
}  
  
 privatestaticClass<?>[] getAllInterfaces(Class<?>type, Map<Class<?>, Set<Method>>signatureMap) {  
   Set<Class<?>>interfaces=newHashSet<Class<?>>();  
   while(type!=null) {  
     for(Class<?>c: type.getInterfaces()) {  
       if(signatureMap.containsKey(c)) {  
         interfaces.add(c);  
      }  
    }  
     type=type.getSuperclass();  
  }  
   returninterfaces.toArray(newClass<?>[interfaces.size()]);  
}  
  
}  
```

我们先看一下Plugin的wrap方法，它根据当前的Interceptor上面的注解定义哪些接口需要拦截，然后判断当前目标对象是否有实现对应需要拦截的接口，如果没有则返回目标对象本身，如果有则返回一个代理对象。而这个代理对象的InvocationHandler正是一个Plugin。所以当目标对象在执行接口方法时，如果是通过代理对象执行的，则会调用对应InvocationHandler的invoke方法，也就是Plugin的invoke方法。所以接着我们来看一下该invoke方法的内容。这里invoke方法的逻辑是：如果当前执行的方法是定义好的需要拦截的方法，则把目标对象、要执行的方法以及方法参数封装成一个Invocation对象，再把封装好的Invocation作为参数传递给当前拦截器的intercept方法。如果不需要拦截，则直接调用当前的方法。Invocation中定义了定义了一个proceed方法，其逻辑就是调用当前方法，所以如果在intercept中需要继续调用当前方法的话可以调用invocation的procced方法。

​    这就是Mybatis中实现Interceptor拦截的一个思想，如果用户觉得这个思想有问题或者不能完全满足你的要求的话可以通过实现自己的Plugin来决定什么时候需要代理什么时候需要拦截。以下讲解的内容都是基于Mybatis的默认实现即通过Plugin来管理Interceptor来讲解的。

​    对于实现自己的Interceptor而言有两个很重要的注解，一个是@Intercepts，其值是一个@Signature数组。@Intercepts用于表明当前的对象是一个Interceptor，而@Signature则表明要拦截的接口、方法以及对应的参数类型。

首先我们看一下拦截器的具体实现，在这里我们需要拦截所有以PageDto作为入参的所有查询语句，自动以拦截器需要继承Interceptor类，PageDto代码如下：

```java
importjava.util.Date;
importjava.util.List;

/**
* Created by chending on 16/3/27.
*/
publicclassPageDto<T>{

   privateIntegerrows=10;

   privateIntegeroffset=0;

   privateIntegerpageNo=1;

   privateIntegertotalRecord=0;

   privateIntegertotalPage=1;

   privateBooleanhasPrevious=false;

   privateBooleanhasNext=false;

   privateDatestart;

   privateDateend;

   privateTsearchCondition;

   privateList<T>dtos;


   publicDategetStart() {
       returnstart;
  }

   publicvoidsetStart(Datestart) {
       this.start=start;
  }

   publicDategetEnd() {
       returnend;
  }

   publicvoidsetEnd(Dateend) {
       this.end=end;
  }

   publicvoidsetDtos(List<T>dtos){
       this.dtos=dtos;
  }

   publicList<T>getDtos(){
       returndtos;
  }

   publicIntegergetRows() {
       returnrows;
  }

   publicvoidsetRows(Integerrows) {
       this.rows=rows;
  }

   publicIntegergetOffset() {
       returnoffset;
  }

   publicvoidsetOffset(Integeroffset) {
       this.offset=offset;
  }

   publicIntegergetPageNo() {
       returnpageNo;
  }

   publicvoidsetPageNo(IntegerpageNo) {
       this.pageNo=pageNo;
  }

   publicIntegergetTotalRecord() {
       returntotalRecord;
  }

   publicvoidsetTotalRecord(IntegertotalRecord) {
       this.totalRecord=totalRecord;
  }


   publicTgetSearchCondition() {
       returnsearchCondition;
  }

   publicvoidsetSearchCondition(TsearchCondition) {
       this.searchCondition=searchCondition;
  }

   publicIntegergetTotalPage() {
       returntotalPage;
  }

   publicvoidsetTotalPage(IntegertotalPage) {
       this.totalPage=totalPage;
  }

   publicBooleangetHasPrevious() {
       returnhasPrevious;
  }

   publicvoidsetHasPrevious(BooleanhasPrevious) {
       this.hasPrevious=hasPrevious;
  }

   publicBooleangetHasNext() {
       returnhasNext;
  }

   publicvoidsetHasNext(BooleanhasNext) {
       this.hasNext=hasNext;
  }
}
```

自定义拦截器PageInterceptor 代码如下：

```java
importjava.sql.Connection;
importjava.sql.PreparedStatement;
importjava.sql.ResultSet;
importjava.sql.SQLException;
importjava.util.List;
importjava.util.Properties;
importme.elw.elog.Log;
importme.wlw.elog.LogFactory;
importme.elw.gaos.common.util.CommonUtil;
importorg.apache.ibatis.executor.parameter.ParameterHandler;
importorg.apache.ibatis.executor.statement.RoutingStatementHandler;
importorg.apache.ibatis.executor.statement.StatementHandler;
importorg.apache.ibatis.mapping.BoundSql;
importorg.apache.ibatis.mapping.MappedStatement;
importorg.apache.ibatis.mapping.ParameterMapping;
importorg.apache.ibatis.plugin.Interceptor;
importorg.apache.ibatis.plugin.Intercepts;
importorg.apache.ibatis.plugin.Invocation;
importorg.apache.ibatis.plugin.Plugin;
importorg.apache.ibatis.plugin.Signature;
importorg.apache.ibatis.scripting.defaults.DefaultParameterHandler;


/**
*
* 分页拦截器，用于拦截需要进行分页查询的操作，然后对其进行分页处理。
*
*/
@Intercepts({@Signature(type=StatementHandler.class,method="prepare",args={Connection.class,Integer.class})})
publicclassPageInterceptorimplementsInterceptor{
   privateStringdialect=""; //数据库方言

   privateLoglog=LogFactory.getLog(PageInterceptor.class);

   @Override
   publicObjectintercept(Invocationinvocation) throwsThrowable{
       if(invocation.getTarget() instanceofRoutingStatementHandler){
           RoutingStatementHandlerstatementHandler=(RoutingStatementHandler)invocation.getTarget();
           StatementHandlerdelegate=(StatementHandler) CommonUtil.getFieldValue(statementHandler, "delegate");
           BoundSqlboundSql=delegate.getBoundSql();
           Objectobj=boundSql.getParameterObject();
           if(objinstanceofPageDto) {
               PageDtopage=(PageDto) obj;
               //获取delegate父类BaseStatementHandler的mappedStatement属性
               MappedStatementmappedStatement=(MappedStatement)CommonUtil.getFieldValue(delegate, "mappedStatement");
               //拦截到的prepare方法参数是一个Connection对象
               Connectionconnection=(Connection)invocation.getArgs()[0];
               //获取当前要执行的Sql语句
               Stringsql=boundSql.getSql();
               //给当前的page参数对象设置总记录数
               this.setTotalRecord(page, mappedStatement, connection);
               //给当前的page参数对象补全完整信息
               //this.setPageInfo(page);
               //获取分页Sql语句
               StringpageSql=this.getPageSql(page, sql);
               //设置当前BoundSql对应的sql属性为我们建立好的分页Sql语句
               CommonUtil.setFieldValue(boundSql, "sql", pageSql);
          }
      }
       returninvocation.proceed();
  }

   /**
    * 给当前的参数对象page设置总记录数
    *
    * @param page Mapper映射语句对应的参数对象
    * @param mappedStatement Mapper映射语句
    * @param connection 当前的数据库连接
    */
   privatevoidsetTotalRecord(PageDtopage, MappedStatementmappedStatement, Connectionconnection) throwsException{
       //获取对应的BoundSql
       BoundSqlboundSql=mappedStatement.getBoundSql(page);
       //获取对应的Sql语句
       Stringsql=boundSql.getSql();
       //获取计算总记录数的sql语句
       StringcountSql=this.getCountSql(sql);
       //通过BoundSql获取对应的参数映射
       List<ParameterMapping>parameterMappings=boundSql.getParameterMappings();
       //利用Configuration、查询记录数的Sql语句countSql、参数映射关系parameterMappings和参数对象page建立查询记录数对应的BoundSql对象。
       BoundSqlcountBoundSql=newBoundSql(mappedStatement.getConfiguration(), countSql, parameterMappings, page);
       //通过mappedStatement、参数对象page和BoundSql对象countBoundSql建立一个用于设定参数的ParameterHandler对象
       ParameterHandlerparameterHandler=newDefaultParameterHandler(mappedStatement, page, countBoundSql);
       //通过connection建立一个countSql对应的PreparedStatement对象。
       PreparedStatementpstmt=null;
       ResultSetrs=null;
       try{
           pstmt=connection.prepareStatement(countSql);
           //通过parameterHandler给PreparedStatement对象设置参数
           parameterHandler.setParameters(pstmt);
           //执行获取总记录数的Sql语句。
           rs=pstmt.executeQuery();
           if(rs.next()) {
               inttotalRecord=rs.getInt(1);
               //给当前的参数page对象设置总记录数
               page.setTotalRecord(totalRecord);
          }
      } catch(SQLExceptione) {
           log.error(e);
           thrownewSQLException();
      } finally{
           try{
               if(rs!=null)
                   rs.close();
               if(pstmt!=null)
                   pstmt.close();
          } catch(SQLExceptione) {
               log.error(e);
               thrownewSQLException();
          }
      }
  }

   /**
    * 根据原Sql语句获取对应的查询总记录数的Sql语句
    * @param sql 原sql
    * @return 查询总记录数sql
    */
   privateStringgetCountSql(Stringsql) {
       intindex=newString(sql).toLowerCase().indexOf("from");
       return"select count(*) "+sql.substring(index);
  }

   /**
    * 给page对象补充完整信息
    *
    * @param page page对象
    */
   privatevoidsetPageInfo(PageDtopage) {
       IntegertotalRecord=page.getTotalRecord();
       IntegerpageNo=page.getPageNo();
       Integerrows=page.getRows();

       //设置总页数
       IntegertotalPage;
       if(totalRecord>rows) {
           if(totalRecord%rows==0) {
               totalPage=totalRecord/rows;
          } else{
               totalPage=1+(totalRecord/rows);
          }
      } else{
           totalPage=1;
      }
       page.setTotalPage(totalPage);

       //跳转页大于总页数时,默认跳转至最后一页
       if(pageNo>totalPage) {
           pageNo=totalPage;
           page.setPageNo(pageNo);
      }

       //设置是否有前页
       if(pageNo<=1) {
           page.setHasPrevious(false);
      } else{
           page.setHasPrevious(true);
      }

       //设置是否有后页
       if(pageNo>=totalPage) {
           page.setHasNext(false);
      } else{
           page.setHasNext(true);
      }
  }

   /**
    * 根据page对象获取对应的分页查询Sql语句
    * 其它的数据库都 没有进行分页
    *
    * @param page 分页对象
    * @param sql 原sql语句
    * @return 分页sql
    */
   privateStringgetPageSql(PageDtopage, Stringsql) {
       StringBuffersqlBuffer=newStringBuffer(sql);
       if("mysql".equalsIgnoreCase(dialect)) {
           //int offset = (page.getPageNo() - 1) * page.getRows();
           sqlBuffer.append(" limit ").append(page.getOffset()).append(",").append(page.getRows());
           returnsqlBuffer.toString();
      }
       returnsqlBuffer.toString();
  }

   /**
    * 拦截器对应的封装原始对象的方法
    */
   @Override
   publicObjectplugin(Objectarg0) {

       if(arg0instanceofStatementHandler) {
           returnPlugin.wrap(arg0, this);
      } else{
           returnarg0;
      }
  }

   /**
    * 设置注册拦截器时设定的属性
    */
   @Override
   publicvoidsetProperties(Propertiesp) {

  }

   publicStringgetDialect() {
       returndialect;
  }

   publicvoidsetDialect(Stringdialect) {
       this.dialect=dialect;
  }

}
```

重点讲解：

1. @Intercept注解中的@Signature中标示的属性，标示当前拦截器要拦截的那个类的那个方法，拦截方法的传入的参数

2. 首先要明白，Mybatis是对JDBC的一个高层次的封装。而JDBC在完成数据操作的时候必须要有一个陈述对象。而陈述对应的SQL语句是在是在陈之前产生的。所以我们的思路就是在生成报表之前对SQL进行下手。更改SQL语句成我们需要的！对于MyBatis的，其声明的英文生成在RouteStatementHandler中。所以我们要做的就是拦截这个处理程序的prepare方法！然后修改的Sql语句!

   ```java
   @Override
      publicObjectintercept(Invocationinvocation) throwsThrowable{
           // 其实就是代理模式！
          RoutingStatementHandlerhandler=(RoutingStatementHandler) invocation.getTarget();
          StatementHandlerdelegate=(StatementHandler)ReflectUtil.getFieldValue(handler, "delegate");
         Stringsql=delegate.getBoundSql().getSql();
          returninvocation.proceed();
     }
   ```

3. 我们知道利用Mybatis查询一个集合时传入Rowbounds对象即可指定其Offset和Limit，只不过其没有利用原生sql去查询罢了，我们现在做的，就是通过拦截器拿到这个参数，然后织入到SQL语句中，这样我们就可以完成一个物理分页！

##### 注册拦截器

在Spring文件中引入拦截器

```xml
  <!--MyBatis分页拦截器-->
   <beanid="paginationInterceptor"class="xx.xx.interceptor.PageInterceptor">
       <propertyname="dialect"value="mysql"/>
   </bean>
   
  ...
    <beanid="sqlSessionFactory"class="org.mybatis.spring.SqlSessionFactoryBean">
       <propertyname="dataSource"ref="dataSource"/>
       <!--自动扫描mapping.xml文件-->
       <propertyname="mapperLocations"value="classpath*:evileye/*.xml"/>
       <propertyname="plugins"ref="paginationInterceptor"/>
   </bean>
```

分页定义的接口：

```java
   List<Student>selectForSearch(PageDto<Student>pageDto);
```

客户端调用如下：

```java
      PageDtopageDto=newPageDto<>();
       Studentstudent =newStudent();
       student.setId(1234);
       student.setName("sky");
       pageDto.setSearchCondition(student);
```

## 127.RowBounds 是一次性查询全部结果吗？为什么？

## 128.mybatis 逻辑分页和物理分页的区别是什么？

## 129.mybatis 是否支持延迟加载？延迟加载的原理是什么？

## 130.说一下 mybatis 的一级缓存和二级缓存？

## 131.mybatis 和 hibernate 的区别有哪些？

## 132.mybatis 有哪些执行器（Executor）？

## 133.mybatis 分页插件的实现原理是什么？

## 134.mybatis 如何编写一个自定义插件？