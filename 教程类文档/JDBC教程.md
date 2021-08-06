

**1,用jdbc技术连接数据库时，要导入相应驱动程序的jar包**



我用jdbc技术连接MySQL数据库时，要导入jdbc厂商提供的MySQL的驱动程序包中的jar包。步骤如下：

1. 在项目目录下，新建一个文件夹，命名为lib
2. 在网上下载mysql-connector-java-5.1.45.jar.(或者更高的版本)，复制到lib文件夹中。

![img](https://img-blog.csdn.net/20180807103119500?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXpob25neWVmaXJzdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3.在lib文件夹下，选中mysql-connector-java-5.1.45.jar然后，右键—>Build Path->Add to path.

右键，==Add as Library==

![img](https://img-blog.csdn.net/20180807103119546?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXpob25neWVmaXJzdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

4.完成！

---

**2,SQL**:

```sql
create database `testjdbc`;
use testjdbc;

create table `student`(
    sno int(10) not null,
    sname varchar(30) default null,
    sage int(100) default null,
    sdept varchar(100) default null,
    primary key (`sno`)
)ENGINE = INNODB DEFAULT CHARSET UTF8

insert into `student`(sno, sname, sage, sdept)
values ( 1,'哈哈',18,'人事部'),
    (2,'呵呵',20,'市场部');
```



## 1.第一个JDBC程序

### **一、搭建实验环境**

新建一个Java工程，并导入数据驱动

### **二、编写程序，在程序中加载数据库驱动**

```
方法①
DriverManager.registerDriver(Driver driver)
DriverManager.registerDriver(new OracleDriver());

方法②【常用】
Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
```

### **三、建立连接*(Connection)***

`Connection conn = DriverManager.*getConnection*(url,user,pass);`

### **四、创建用于向数据库发送SQL的Statement对象，并发送sql**

  ```java
Statement st = conn.createStatement();

ResultSet rs = st.excuteQuery(sql);
  ```

### **五、从代表结果集的ResultSet中取出数据，打印到命令行窗口**

### **六、断开与数据库的连接，并释放相关资源**

### 测试

**TestJDBC.java**

```java
package com.myt;

import java.sql.*;

public class TestJDBC {
    private static Connection conn = null;
    private static Statement statement = null;
    private static ResultSet rs = null;

    public static void main(String[] args) {
        //executeUpdate测试
        //String sql = "insert into student (sno, sname, sage, sdept)\n" +
        //        "values ( 3,'嘻嘻',22,'人事部'),\n" +
        //        "    (4,'呦呦',20,'市场部')";
        //executeUpdate(sql);

        String sql2 = "select * from student";
        executeQuery(sql2);
    }

    /**
     * 查
     * @param sql
     */
    public static void executeQuery(String sql){

        try {
            // 1, 向JDBC注册(需要操作的)数据库操作
            // ①,将驱动引入到项目中
            // ②,通过反射机制注册驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            //2.通过DriverManager获取Java和数据库之间的连接
            conn = DriverManager.getConnection(
                    "jdbc:mysql://49.234.197.183:3306/testjdbc", "root",
                    "zhou2020");
            // 3,准备要操作的SQL语句
            // 4,从连接conn中获取sql 编译工具
            statement = conn.createStatement();
            // 5,通过sql语句的编译工具执行sql语句
            rs = statement.executeQuery(sql);
            while(rs.next()){
                int age = rs.getInt("sage");
                System.out.println(rs.getString("sno") + "\t"
                        + rs.getString("sname") + "\t" + age + "\t" + ","
                        + rs.getString("sdept"));
            }
            //关闭数据库连接
        } catch (SQLException e) {
            System.out.println("连接数据库失败");
            e.printStackTrace();
        }catch (ClassNotFoundException e){
            System.out.println("加载驱动失败！");
            e.printStackTrace();
        }finally {
            //关闭顺序：ResultSet->Statement->Connection
            if(rs != null){
                try{
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(statement != null){
                try {
                    statement.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (conn != null){
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 增删改
     * @param sql
     */
    // Statement 中的boolean flag=st.statement(sql)方法:如果操作成功,返回false,如果操作失败,则返回true;
    public static void executeUpdate(String sql){
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            //2.通过DriverManager获取Java和数据库之间的连接
            Connection conn = DriverManager.getConnection(
                    "jdbc:mysql://49.234.197.183:3306/testjdbc", "root",
                    "zhou2020");
            Statement st = conn.createStatement();
            int i = st.executeUpdate(sql);
            if(i > 0){
                System.out.println("操作成功");
            }else {
                System.out.println("操作失败");
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            if (statement != null) {
                try {
                    statement.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```



## 2.DriverManager

jdbc程序中的DriverManager用于①加载驱动，并②创建与数据库的连接。

这个类的常用方法：

1.

`DriverManager.registerDriver(new SQLServerDriver());`

注意：在实际开发中，并不推荐采用这个方法注册驱动。

==查看Driver的源代码可以看到，如果采用此种方式，会导致驱动程序加载两次，也就是在内存中会有两个Driver对象。==

推荐方式:

`Class.forName("oracle.jdbc.driver.OracleDriver");`

这种方式**不会导致驱动对象在内存中重复出现**，并且采用这种方式，程序**仅仅只需要一个字符串，不需要import驱动的API**，可使程序**不依赖具体的驱动**，使程序的灵活性更高。

2.

`DriverManager.getConnection(url, username, password)`，根据url获取数据库的链接。



## 3.Connection接口

jdbc程序中的Connection，它用于**代表数据库的链接**，是数据库编程中最重要的一个对象，客户端与数据库所有交互都是通过connection对象完成的。

这个对象的常用方法：

| 返回值类型        | 方法                                  | 说明                                                  |
| ----------------- | ------------------------------------- | ----------------------------------------------------- |
| Statement         | createStatement()                     | 创建向数据库发送sql的statement对象                    |
| PreparedStatement | prepareStatement(String sql)          | 创建向数据库发送预编译sql的PrepareSatement对象。      |
| CallableStatement | prepareCall(String sql)               | 创建一个 CallableStatement 对象来调用数据库存储过程。 |
| void              | **setAutoCommit(boolean autoCommit)** | **设置事务是否自动提交**。                            |
| void              | **commit()**                          | **提交事务**                                          |
| void              | **rollback()**                        | **回滚事务**。                                        |

**面试题：** 什么时候，需要把setAutoCommit设为 false？

答：当有多个 DML （增删改）同时执行，将其看作一个整体提交，则使用 事务管理，则需要把 setAutoCommit 设为false。

```java
// 1加载驱动
Class.forName("oracle.jdbc.driver.OracleDriver");
// 2得到连接
ct = DriverManager.getConnection("jdbc:oracle:thin:@127.0.0.1:1521:orcl", "scott", "tiger");
// 把事务设为不自动提交
ct.setAutoCommit(false);
// 3.创建sql对象(Statement / PreparedStatement /CallableStatement)
statement = ct.createStatement();
// 4.通过statement向数据库发出sql 指令.
// 需求: 对emp表进行操作: 把SMITH 的sal -10 给 KING sal+10
statement.executeUpdate("update emp set sal=sal-10 where ename='SMITH'");
int i = 9 / 0;
statement.executeUpdate("update emp set sal=sal+10 where ename='KING'");
// 提交所有事务
ct.commit();
```

## 4.ResultSet接口

### 1.介绍

- jdbc程序中的  ResultSet  **用于代表SQL语句的执行结果**。Resultset封装执行结果时，采用的类似于表格的方式。ResultSet 对象维护了一个指向表格数据行的**游标**，初始的时候，游标在第一行之前，调用ResultSet.next() 方法，可以使游标指向具体的数据行，进行调用方法获取该行的数据

### 2.提供的方法

- ResultSet 既然用于封装执行结果的，所以该对象提供的都是**用于获取数据的 get 方法**：
  - 获取任意类型的数据
    - getObject(int index)     根据索引
    - getObject(String columnName)  根据列名
  - 获取指定类型的数据，例如：
    - getString(int index)
    - getString(String columnName)
- ResultSet 还提供了**对结果集进行滚动的方法**：
  - next()  ：移动到下一行
  - previous() ：移动到前一行
  - absolute(int row)：移动到指定行【row从1开始计算】
  - beforeFirst()：移动到resultSet的最前面
  - afterLast()：移动到resultSet的最后面



### 3.ResultSet的说明

**在默认情况下，我们的rs结果集，只能向前移动，这样 rs 结果就不能复用 ，如果希望复用，则可以这样做**:



ResultSet 的可选项有：

- resultSetType  -结果集类型
  - 是   ResultSet.TYPE_FORWARD_ONLY、    ResultSet.TYPE_SCROLL_INSENSITIVE        或 ResultSet.TYPE_SCROLL_SENSITIVE          之一
- resultSetConcurrency  -并发类型
  - 是    ResultSet.CONCUR_READ_ONLY     或    ResultSet.CONCUR_UPDATABLE 之一

```java
//需求：
//通过java 来查询所有的雇员.
//假设我们希望rs结果，可以滚动(可以向前，亦可向后)

ct = DriverManager.getConnection("jdbc:oracle:thin:@127.0.0.1:1521:orcl", "scott", "tiger");
// 假设我们希望rs 结果，可以滚动（可以向前，也可以向后）
// statement=ct.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE,
// ResultSet.CONCUR_READ_ONLY);
rs = statement.executeQuery("select * from emp");
while (rs.next()) {
   System.out.println(rs.getString("ename"));
}
System.out.println("*************");
rs.beforeFirst();
while (rs.next()) {
   System.out.println(rs.getString("ename"));
}
```



 提问：数据库中列的类型是varchar2，获取该列的数据调用什么方法？Int类型呢？bigInt类型呢？Boolean类型？(详见后ppt)



## 5.JDBC-释放资源

jdbc程序运行完后，**切记要释放**程序在运行过程中，创建的那些与数据库进行交互的对象，这些对象通常是**ResultSet**, **Statement **和 **Connection** 对象。



**特别是Connection对象，它是非常稀有的资源，用完后必须马上释放，如果Connection不能及时、正确的关闭，极易导致系统宕机。Connection的使用原则是尽量晚创建，尽量早的释放。**



**为确保资源释放代码能运行，资源释放代码也一定要放在==finally==语句中。**

## 6.使用JDBC对数据库进行CRUD

- Jdbc中的**statement对象用于向数据库发送SQL语句**，想完成对数据库的增删改查，只需要通过这个对象向数据库发送增删改查语句即可。
- Statement对象的executeUpdate方法，用于向数据库发送增、删、改的sql语句，executeUpdate执行完后，将会返回一个整数(即增删改语句导致了数据库几行数据发生了变化)。
- Statement.executeQuery方法用于向数据库发送查询语句，executeQuery方法返回代表查询结果的ResultSet对象。

### 1.CRUD操作-create

使用**executeUpdate(String sql)**方法完成数据**添加**操作

```java
Statement st = conn.createStatement();
String sql = "insert into user(...) values(...)";
int num = st.executeUpdate(sql);
if(num > 0){
    System.out.println("插入成功！！！");
}
```

### 2.CRUD操作-update

使用**executeUpdate(String sql)**方法完成数据**更新**操作

```java
Statement st = conn.createStatement();
String sql = "update user set name = '' where name = ''";
int num = st.executeUpdate(sql);
if(num > 0){
    System.out.println("修改成功！！！");
}
```

### 3.CRUD操作-delete

使用**executeUpdate(String sql)**方法完成数据**删除**操作

```java
Statement st = conn.createStatement();
String sql = "delete from user where id=1";
int num = st.executeUpdate(sql);
if(num > 0){
    System.out.println("删除成功！！");
}
```

### 4.CRUD操作-retrieve（read）

使用 **executeQuery(String sql)** 方法完成数据查询操作

```java
Statement st = conn.createStatement();
String sql = "select * from user where id = 1";
ResultSet rs = st.executeQuery(sql);
while(rs.next()){
    //根据获取列的数据类型，分别调用rs的相应方法
    //映射到Java对象中
}
```

## 7.JDBC-防止SQL注入

==SQL注入：是用户利用某些系统没有对输入数据进行充分的检查，从而进行恶意破坏的行为。==



1，statement 存在sql注入攻击问题，例如登录用户名采用 ‘name’ or ‘a’=’a’

`select * from student where sname='name' or 'a'='a';`

可以查出所有信息。

2，**防止SQL注入，需要采用 PreparedStatement 取代 Statement**

3，或者通过程序来控制。这个会比较麻烦，推荐使用PreparedStatement来完成。

```java
import java.io.*;
import java.sql.*;
public class TestInsert {

   public static void main(String[] args) {
      try {
        BufferedReader br = new BufferedReader(new InputStreamReader(
              System.in));
        System.out.println("请输入学号:");
        String sno = br.readLine();
        System.out.println("请输入姓名:");
        String sname = br.readLine();
        System.out.println("请输入性别:");
        String ssex = br.readLine();
        System.out.println("请输入年龄:");
        int sage = Integer.parseInt(br.readLine());
        System.out.println("请输入专业:");
        String sdept = br.readLine();
          
          
        Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
        Connection conn = DriverManager.getConnection(
              "jdbc:sqlserver://localhost:1433;database=mydb", "sa",
              "19891005a");
        String sql = "insert into students (sno,sname,ssex,sage,sdept) values(?,?,?,?,?)";
        // 采用预编译处理SQL语句(在获取PreparedStatement对象的时候加载SQL语句)
        PreparedStatement ps = conn.prepareStatement(sql);
        ps.setString(1, sno);
        ps.setString(2, sname);
        ps.setString(3, ssex);
        ps.setInt(4, sage);
        ps.setString(5, sdept);
        int n = ps.executeUpdate();
        if (n > 0) {
           System.out.println("添加成功!");
        } else {
           System.out.println("添加失败!");
        }
        if (ps != null) {
           ps.close();
        }
        if (conn != null) {
           conn.close();
        }
      } catch (Exception e) {
        e.printStackTrace();
      }
   }
}
```



## 8.CallableStatement 调用存储过程

**用于执行 SQL 存储过程的接口**

| 返回值类型 | 方法                                                        | 说明                                                     |
| ---------- | ----------------------------------------------------------- | -------------------------------------------------------- |
| void       | **registerOutParamenter**(int paramenterIndex, int sqlType) | 按顺序位置parameterIndex 将OUT参数注册为JDBC类型 sqlType |
| xxx        | **getXxx**(int parameterIndex)                              | 获取存储过程中输出值                                     |

步骤：

1）注册驱动

 `Class.forName("");`

2）建立连接

`Connection conn = DriverManager.getConnection("","","");`

准备SQL：

```java
String sql1 = "select * from student";

String sql2 = "insert into student(sno,sname,ssex,sage,sdept) values(?,?,?,?,?)";

String sql3 = "{call proc_split(?,?,?,?)}";
```

3）获取 Statement 对象

 如果**没有参数**，获取**Statement(不能处理参数)
 如果sql语句中有参数获取，PreparedStatement** (只扩展了处理输入参数的方法  **ps.setString(1,no))**

如果sql语句是**调用存储过程**则获取**CallableStatement(**扩展了对输出参数的操作方法**)**

4）执行SQL获取结果

5）处理结果

6）关闭连接

//分页存储过程

```sql
create proc proc_split
@pageSize int,
@pageNum int,
@recordCount int output,
@pageCount int output
as
   select @recordCount=count(*) from student;
   set @pageCount=(@recordCount-1)/@pageSize+1;
   if(@pageNum<1)begin
      set @pageNum=1;
   end else if(@pageNum>@pageCount)begin
      set @pageNum=@pageCount;
   end
   select top (@pageSize) * from student where sno not in
        (select top ((@pageNum-1)*@pageSize) sno from student);
go
```

//调用存储过程

```java
public List<StudentDTO> splitPage(int pageSize, int pageNum) {
    
   ArrayList<StudentDTO> stus = new ArrayList<StudentDTO>();
   try {
      conn = DBManager.getConnection();
      
      String sql = "{call proc_split(?,?,?,?)}";
      cs = conn.prepareCall(sql);
      cs.setInt(1, pageSize);
      cs.setInt(2, pageNum);
      // 调用CS中所特有用来处理存储过程中输入输出参数的方法:注册输出参数
      cs.registerOutParameter(3, Types.INTEGER);
      cs.registerOutParameter(4, Types.INTEGER);
       
      rs = cs.executeQuery();
      while (rs.next()) {
         StudentDTO stu = new StudentDTO();
        stu.setStuNo(rs.getString("sno"));
        stu.setStuName(rs.getString("sname"));
        stu.setStuSex(rs.getString("ssex"));
        stu.setStuAge(rs.getInt("sage"));
        stu.setStuDept(rs.getString("sdept"));
        stus.add(stu);
      }
      // 获取两个输出参数的值
      int recordCount = cs.getInt(3);
      int pageCount = cs.getInt(4);
      System.out.println("总记录数:" + recordCount);
      System.out.println("第" + pageNum + "页");
      System.out.println("如果每页显示" + pageSize + "条,可分为" + pageCount + "页");
   } catch (SQLException e) {
      e.printStackTrace();
   }
   return stus;
}
```

## 9.数据库连接池 DBCP

**DBCP**（DataBase Connection Pool） 是apache上的一个Java连接池项目。DBCP通过连接池预先同数据库建立一些连接放在内存中(即连接池中)，应用程序需要建立数据库连接时直接到从接池中申请一个连接使用，用完后由连接池回收该连接，从而达到连接复用，减少资源消耗的目的。



数据库连接池：将一定数量的 数据库连接 放在一起集中管理，当有数据库操作发生的时候，无需再创建连接，只需要从连接池中获取一个连接使用，使用完成之后重新放回连接池；这样以来就可以循环使用这些连接，省去了**多次重复创建**和**销毁连接**的系统开销。



**DBCP所依赖的jar包（以下例子基于如下jar包版本）**

**commons-dbcp2-2.1.1.jar    commons-logging-1.2.jar    commons-pool2-2.4.2.jar**



**使用数据库连接池**

①将三个帮助 jar 包导入项目中

②创建一个类来创建一个数据库连接池

​	定义四个连接数据库所需要的常量信息

​	再定义一个整型的常量来执行连接池中最大连接数

​	定义一个BasicDataSource对象

​	在静态块中实例化BasicDataSource（数据库连接）对象，并且设置连接参数

​	提供一个共有的静态方法 getConn，返沪一个数据库连接

```java
import java.sql.Connection;
import java.sql.SQLException;
import org.apache.commons.dbcp.BasicDataSource;

public class DBCP {
   private static final String DRIVER = "com.microsoft.sqlserver.jdbc.SQLServerDriver";
   private static final String URL = "jdbc:sqlserver://localhost:1433;databaseName=mydb";
   private static final String USERNAME = "sa";
   private static final String PASSWORD = "admin123";
   // 定义一个整形常量来表示执行连接池中最大连接数
   private static final int MAX = 15;
   private static BasicDataSource bds = null;

   static {
      bds = new BasicDataSource();
      // 在实例化之后的bds对象中设置连接池参数
      bds.setDriverClassName(DRIVER);
      bds.setUrl(URL);
      bds.setUsername(USERNAME);
      bds.setPassword(PASSWORD);
      bds.setMaxActive(MAX);
   }

   public static Connection getConnection() {
      Connection conn = null;
      try {
        conn = bds.getConnection();
      } catch (SQLException e) {
        e.printStackTrace();
      }
      return conn;
   }
}
```

## 10.DBManager.java

**DBManager.java**

```java
import java.sql.*;

public class DBManager {
   private static final String DRIVER = "com.microsoft.sqlserver.jdbc.SQLServerDriver";
   private static final String URL = "jdbc:sqlserver://localhost:1433;databaseName=mydb";
   private static final String USERNAME = "sa";
   private static final String PASSWORD = "admin123";
   static {
      try {
         Class.forName(DRIVER);
      } catch (ClassNotFoundException e) {
         e.printStackTrace();
      }
   }

   public static Connection getConnection() {
      Connection conn = null;
      try {
         conn = DriverManager.getConnection(URL, USERNAME, PASSWORD);
      } catch (SQLException e) {
         e.printStackTrace();
      }
      return conn;
   }
 
   public static void close(ResultSet rs, Statement ps, Connection conn) {
      if (rs != null) {
         try {
            rs.close();
         } catch (SQLException e) {
            rs = null;
            e.printStackTrace();
         }
      }
      if (ps != null) {
         try {
            ps.close();
         } catch (SQLException e) {
            ps = null;
            e.printStackTrace();
         }
      }
      if (conn != null) {
         try {
            conn.close();
         } catch (SQLException e) {
            conn = null;
            e.printStackTrace();
         }
      }
   }
}
```



**StudentDTO.java**

```java
import java.io.Serializable;

public class StudentDTO implements Serializable {
   private String stuNo;
   private String stuName;
   private String stuSex;
   private int stuAge;
   private String stuDept;
   public StudentDTO() {
      super();
   }

   public StudentDTO(String stuNo, String stuName, String stuSex, int stuAge,
         String stuDept) {
      super();
      this.stuNo = stuNo;
      this.stuName = stuName;
      this.stuSex = stuSex;
      this.stuAge = stuAge;
      this.stuDept = stuDept;
   }

   public String getStuNo() {
      return stuNo;
   }

   public void setStuNo(String stuNo) {
      this.stuNo = stuNo;
   }

   public String getStuName() {
      return stuName;
   }

   public void setStuName(String stuName) {
      this.stuName = stuName;
   }

   public String getStuSex() {
      return stuSex;
   }

   public void setStuSex(String stuSex) {
      this.stuSex = stuSex;
   }

   public int getStuAge() {
      return stuAge;
   }

   public void setStuAge(int stuAge) {
      this.stuAge = stuAge;
   }

   public String getStuDept() {
      return stuDept;
   }

   public void setStuDept(String stuDept) {
      this.stuDept = stuDept;
   }
   public String toString() {
      return stuNo + "\t" + stuName + "\t" + stuSex + "\t" + stuAge + "\t"
            + stuDept;
   }
}
```



**StudentDAO.java**

```java
import java.sql.*;
import java.util.*;

public class StudentDAO {
   private Connection conn = null;
   private Statement st = null;
   private PreparedStatement ps = null;
   private ResultSet rs = null;
   private CallableStatement cs = null;

   public boolean insert(StudentDTO stu) {
      boolean flag = false;
      try {
         conn = DBManager.getConnection();
         String sql = "insert into student (sno,sname,ssex,sage,sdept) values(?,?,?,?,?)";
         ps = conn.prepareStatement(sql);
         ps.setString(1, stu.getStuNo());
         ps.setString(2, stu.getStuName());
         ps.setString(3, stu.getStuSex());
         ps.setInt(4, stu.getStuAge());
         ps.setString(5, stu.getStuDept());
         flag = !ps.execute();
      } catch (SQLException e) {
         e.printStackTrace();
      } finally {
         DBManager.close(null, ps, conn);
      }
      return flag;
   }
 
   public List<StudentDTO> list() {
      ArrayList<StudentDTO> stus = new ArrayList<StudentDTO>();
      try {
         conn = DBManager.getConnection();
         String sql = "select * from student";
         ps = conn.prepareStatement(sql);
         rs = ps.executeQuery();
         while (rs.next()) {
            StudentDTO stu = new StudentDTO(rs.getString("sno"),
                   rs.getString("sname"), rs.getString("ssex"),
                   rs.getInt("sage"), rs.getString("sdept"));
            stus.add(stu);
         }
      } catch (SQLException e) {
         e.printStackTrace();
      } finally {
         DBManager.close(rs, ps, conn);
      }
      return stus;
   }

   public StudentDTO queryBySno(String sno) {
      StudentDTO stu = null;
      try {
         conn = DBManager.getConnection();
         ps = conn.prepareStatement("select * from student where sno=?");
         ps.setString(1, sno);
         rs = ps.executeQuery();
         while (rs.next()) {
            stu = new StudentDTO(rs.getString("sno"),
                   rs.getString("sname"), rs.getString("ssex"),
                   rs.getInt("sage"), rs.getString("sdept"));
         }
      } catch (SQLException e) {
         e.printStackTrace();
      } finally {
         DBManager.close(rs, ps, conn);
      }
      return stu;
   }
 
   public boolean delete(String sno) {
      boolean flag = false;
      try {
         conn = DBManager.getConnection();
         String sql = "delete from student where sno=?";
         ps = conn.prepareStatement(sql);
         ps.setString(1, sno);
         flag = !ps.execute();
      } catch (SQLException e) {
         e.printStackTrace();
      } finally {
         DBManager.close(null, ps, conn);
      }
      return flag;
   }

   public boolean update(StudentDTO stu) {
      boolean flag = false;
      try {
         conn = DBManager.getConnection();
         String sql = "update student set sname=?,ssex=?,sage=?,sdept=? where sno=?";
         ps = conn.prepareStatement(sql);
         ps.setString(1, stu.getStuName());
         ps.setString(2, stu.getStuSex());
         ps.setInt(3, stu.getStuAge());
         ps.setString(4, stu.getStuDept());
         ps.setString(5, stu.getStuNo());
         flag = !ps.execute();
      } catch (SQLException e) {
         e.printStackTrace();
      } finally {
         DBManager.close(null, ps, conn);
      }
      return flag;
   }
 
    // 调用存储过程
   public List<StudentDTO> splitPage(int pageSize, int pageNum) {
      ArrayList<StudentDTO> stus = new ArrayList<StudentDTO>();
      try {
         conn = DBManager.getConnection();
         String sql = "{call proc_split(?,?,?,?)}";
         cs = conn.prepareCall(sql);
         cs.setInt(1, pageSize);
         cs.setInt(2, pageNum);
         // 调用CS中所特有用来处理存储过程中输入输出参数的方法:注册输出参数
         cs.registerOutParameter(3, Types.INTEGER);
         cs.registerOutParameter(4, Types.INTEGER);
         rs = cs.executeQuery();
         while (rs.next()) {
            StudentDTO stu = new StudentDTO();
            stu.setStuNo(rs.getString("sno"));
            stu.setStuName(rs.getString("sname"));
            stu.setStuSex(rs.getString("ssex"));
            stu.setStuAge(rs.getInt("sage"));
            stu.setStuDept(rs.getString("sdept"));
            stus.add(stu);
         }
         // 获取两个输出参数的值
         int recordCount = cs.getInt(3);
         int pageCount = cs.getInt(4);
         System.out.println("总记录数:" + recordCount);
         System.out.println("第" + pageNum + "页");
         System.out.println("如果每页显示" + pageSize + "条,可分为" + pageCount + "页");
      } catch (SQLException e) {
         e.printStackTrace();
      }
      return stus;
   }
}
```



## 11.SqlHelper 工具类封装

**SQLHelper.java**

```java
import java.io.FileInputStream; import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DriverManager;
//防止 sql注入漏洞
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.Properties;  
public class SQLHelper {

   // 定义需要的变量
   private static Connection ct = null;
   private static PreparedStatement ps = null;
   private static ResultSet rs = null;
   private static String driver = null;
   private static String url = null;
   private static String username = null;
   private static String password = null;
   private static Properties prop = null;
   private static CallableStatement cs = null;
   private static InputStream in = null;
    
   public static Connection getCt() {
      return ct;
   }

   public static PreparedStatement getPs() {
      return ps;
   }

   // 加载驱动，只需要一次

   static {
      prop = new Properties();
      try {
        // 配置信息放在src/properties下，根目录是工程目录
        // in = new FileInputStream(
        // "src/properties/dbinfo.properties");
        // 如果打jar包，根目录是src
        in = SQLHelper.class
              .getResourceAsStream("/properties/dbinfo.properties");
        prop.load(in);

        driver = prop.getProperty("driver");
        url = prop.getProperty("url");
        username = prop.getProperty("username");
        password = prop.getProperty("password");

        Class.forName(driver);
      } catch (FileNotFoundException e) {
        e.printStackTrace();
      } catch (IOException e) {
        e.printStackTrace();
      } catch (ClassNotFoundException e) {
        e.printStackTrace();
      } finally {
        try {
           in.close();
        } catch (IOException e) {
           e.printStackTrace();
           in = null;
        }
      }
   }

   // 得到链接

   private static Connection getConnection() {
      try {
        ct = DriverManager.getConnection(url, username, password);
      } catch (SQLException e) {
        e.printStackTrace();
      }
      return ct;
   }

   // update/delete/insert 单个增删改操作，不需要考虑从事务

   public static void executeUpdate(String sql, String params[]) {

      try {
        ct = getConnection();
        ps = ct.prepareStatement(sql);
        if (params != null) {
           for (int i = 0; i < params.length; i++) {
              ps.setString(i + 1, params[i]);
           }
        }
        ps.executeUpdate();
      } catch (SQLException e) {
        e.printStackTrace();
        throw new RuntimeException(e.getMessage());
      } finally {
        close(rs, ps, ct);
      }
   }

   // 多个 update / delete / insert 语句，考虑事务

   public static void executeUpdate(String sql[], String params[][]) {

      try {
        ct = getConnection();
        // 因为用户传入的可能是多个sql语句,设置不自动提交
        ct.setAutoCommit(false);
        for (int i = 0; i < sql.length; i++) {
           ps = ct.prepareStatement(sql[i]);
           if (params != null) {
              if (params[i] != null) {
                 for (int j = 0; j < params[i].length; j++) {
                    ps.setString(j + 1, params[i][j]);
                 }
              }
           }
           ps.executeUpdate();
        }
        ct.commit();
      } catch (Exception e) {
        e.printStackTrace();
        try {
           ct.rollback();
        } catch (SQLException e1) {
           e1.printStackTrace();
        }
      }
   }

   // 查询数据库
    
   public static ResultSet executeQuery(String sql, String params[]) {

      // 根据实际情况我们队sql语句 ？进行赋值

      try {
        ct = getConnection();
        ps = ct.prepareStatement(sql);
        if (params != null) {
           for (int i = 0; i < params.length; i++) {
              ps.setString(i + 1, params[i]);
           }
        }
        rs = ps.executeQuery();
        return rs;
      } catch (SQLException e) {
        throw new RuntimeException("查询失败");
      }
   }

   // 对查询语句进行升级
   public ArrayList<Object[]> executeQuery2(String sql, String params[]) {
      PreparedStatement preparedStatement = null;
      Connection connection = null;
      ResultSet resultSet = null;
      try {
        connection = getConnection();
        preparedStatement = connection.prepareStatement(sql);
        if (params != null) {
           for (int i = 0; i < params.length; i++) {
              ps.setString(i + 1, params[i]);
           }
        }
        resultSet = preparedStatement.executeQuery();
        ArrayList<Object[]> list = new ArrayList<Object[]>();
        ResultSetMetaData rsmd = resultSet.getMetaData();
        // 返回此 ResultSet 对象中的列数。
        int column = rsmd.getColumnCount();
        while (resultSet.next()) {
           Object obj[] = new Object[column];
           for (int i = 1; i <= column; i++) {
              obj[i - 1] = resultSet.getObject(i);
           }
           list.add(obj);
        }
        return list;
      } catch (Exception e) {
        throw new RuntimeException("查询失败" + e.getMessage());
      } finally {
        close(resultSet, preparedStatement, connection);
      }
   }

   // 调用存储过程 无返回值
   // sql {call 过程(?,?,?)}
   public static void calPro1(String sql, String params[]) {
      try {
        ct = getConnection();
        cs = ct.prepareCall(sql);
        if (params != null) {
           for (int i = 0; i < params.length; i++) {
              cs.setString(i + 1, params[i]);
           }
        }
        cs.execute();
      } catch (SQLException e) {
        e.printStackTrace();
      } finally {
        close(rs, cs, ct);
      }
   }

   // 调用存储过程 有返回值
   public static CallableStatement callPro2(String sql, String[] inparameters,
        Integer[] outparameters) {
      try {
        ct = getConnection();
        cs = ct.prepareCall(sql);
        if (inparameters != null) {
           for (int i = 0; i < inparameters.length; i++) {
              cs.setObject(i + 1, inparameters[i]);
           }
        }
        // 给 out参数赋值
        if (outparameters != null) {
           for (int i = 0; i < outparameters.length; i++) {
              cs.registerOutParameter(inparameters.length + 1 + i,
                    outparameters[i]);
           }
        }
        cs.execute();
      } catch (Exception e) {
        e.printStackTrace();
        throw new RuntimeException(e.getMessage());
      } finally {
      }
      return cs;
   }

   public static void close(ResultSet rs, Statement ps, Connection ct) {
      if (rs != null) {
        try {
           rs.close();
        } catch (SQLException e) {
           e.printStackTrace();
        }
        rs = null;
      }
      if (ps != null) {
        try {
           ps.close();
        } catch (SQLException e) {
           e.printStackTrace();
        }
        ps = null;
      }
      if (null != ct) {
        try {
           ct.close();
        } catch (SQLException e) {
           e.printStackTrace();
        }
        ct = null;
      }
   }
}
```



