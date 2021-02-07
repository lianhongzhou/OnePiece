# 如何连接数据库

1. 注册驱动

   ```
   比如Mysql
   Class.forName("com.mysql.jdbc.Driver")
   ```

2. 建立数据库连接

   ```
   DriverManager.getConnection(url,user,password)
   ```

3. 建立Statement对象或者PreparedStatement对象

4. 执行SQL语句

5. 访问结果集ResultSet对象

6. 关闭资源

   
   
   

# 需要注意的问题

最后一定要释放资源



# 优化的地方

释放资源的地方一大堆if(xx!=null){ try{xx.close();}catch{}}如何改善

1. 可以放在try-withresource中
2. 封装成一个工具类，需要调用的时候用getConnection()
3. spring框架为我们提供了jdbcTemplate,HibernaterTemplate.用模板代码消除了大量的样板代码









# Class.forName作用

把类嘉爱到JVM，返回一个带有字符串名的类或者接口相关联的Class对象，并且JVM会加载这个类，同时JVM会执行该类的静态代码段



# Statement，PreparedStatement有什么区别

PreparedStament有以下优点

1. 效率更高

   ```
   PreparedStatement执行SQL命令的时候，命令会被数据库进行编译和解析，并放在命令缓冲区。然后，每当执行同一个PareparedStatement对象时，由于在缓冲区可以发现预编译的命令，虽然它会被再解析一遍，但是不会被再次编译，是可以重复使用的，能够有效地提高系统性能。
   ```

2. 安全性更高

   SQL注入只对编译过程有破坏作用，由于是预编译的就免除了这个烦恼

3. 可读性更好

   statement需要用字符串连接符连接各个输入值

   但是PreparedStatement直接调用各个set方法进行设置就好了





# getString和getObject方法区别

1. 当调用getString,getInt等方法时，程序是直接将数据放到内存，但如果数据过大，内存装不下，这个时候就会出错
2. 而getObject正是为了解决数据量过大的问题，调用此方法，每次都会从数据库中取，而不是一次性读到内存中

















​          