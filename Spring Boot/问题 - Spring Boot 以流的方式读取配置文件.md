**问题说明**

> 　　在以往的项目中，我们有时会读取资源目录下的`*.config`文件，有时会读取绝对路径，还挺好用，但是这种做法在Spring Boot项目中就不好使了。因为Spring Boot项目把文件都打了一个`*.jar`包，这是绝对路径就不好使了，要使用流的方式读取文件。

**示例代码：**
```
Resource resource = new ClassPathResource("config/*.config");
```
