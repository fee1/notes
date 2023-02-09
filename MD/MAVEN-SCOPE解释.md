# 1.compile
```text
    compile表示被依赖项目需要参与当前项目的编译，包括后续的测试，运行周期也参与其中，同时打包的时候也会包含进去。是最常用的，所以也是默认的。
```
# 2.runtime 
```text
    runtime表示被依赖项目无需参与项目的编译，不过后期的测试和运行周期需要其参与。与compile相比，跳过编译而已
```
# 3.test
```text
    test表示只会在测试阶段使用，在src/main/java里面的代码是无法使用这些api的，并且项目打包时，也不会将"test"标记的打入"jar"包或者"war"包
```
# 4.system 
```text
    system依赖不是由maven仓库，而是本地的jar包，因此必须配合systemPath标签来指定本地的jar包所在全路径。这类jar包默认会参与编译、测试、
运行，但是不会被参与打包阶段。如果也想打包进去的话，需要在插件里做配置<includeSystemScope>true</includeSystemScope>
```
# 5.provided 
```text
    provided表示的是在编译和测试的时候有效，在执行（mvn package）进行打包成war、jar包的时候不会加入，比如：servlet-api，
因为servlet-api，tomcat等web服务器中已经存在，如果在打包进去，那么包之间就会冲突
```
# 6.import
```text
    import比较特殊，他的作用是将其他模块定义好的 dependencyManagement 导入当前 Maven 项目 pom 的 dependencyManagement 中，
都是配合<type>pom</type>来进行的。所以import作用的是pom类型，不是jar包。
```