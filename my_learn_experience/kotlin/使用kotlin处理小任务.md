在 **上手** 这门语言之后我迫不及待的粗略看了一遍它的标准库实现 (`stdlib`)，大概看了一下它做了什么增强，结论是：**这里面有海量的 通过 `扩展函数` 实现的语法糖** 。

```
kotlin
    io
    	AccessDeniedException
        ByteStreamsKt.class
        ConsoleKt.class
        ConstantsKt.class
        ExceptionsKt.class
        FileAlreadyExistsException
        FilePathComponents
        FilesKt.class
        FileSystemException
        FileTreeWalk
        FileWalkDirection
        LinesSequence
        NoSuchFileException
        OnErrorAction
        TerminateException
        TextStreamsKt.class
```



曾经，在学校里初学java的时候，我曾被 `io` 弄得死去活来，`util` 我认识了 装饰器，所以我第一时间去找了 `kotlin.io`这个包。

 `kotlin`除了对 `java.io`包下的类做一些扩展外，它还 多了`FileWalkDirection` 和 `FileTreeWalk`。

翻阅一些文献后了解到这是它对`遍历文件树`这个需求 给出的解决方案。

它 实现了 `kotlin.sequences.Sequence`这个接口 ，并且因 `kotlin.sequences.SequencesKt` （这个 函数库 拥有较 `java8` `stream` 更丰富的api）,因搭了它的便车，处理遍历规则变得十分直观。

之后有一天 突然 在工作中，被分配了 "列出来所有项目中的定时任务的时间" 这样类似的任务。

我的第一反应是来一发`ctrl + shift + f`，结果是结果集太多了（因为是个多模块项目，文件非常多，定义位置也并非都在相同名字的包，既没有规律），我试着想用代码解决这个问题。

思路很简单，首先能遍历所有`.java`,其次是找到带有`@Scheduled` 注解的 然后找到它下一行(规律是 注解定义在方法上，故大部分时候只要取下一行就是方法)

```kotlin
import java.io.File
import java.lang.String.format

fun main() {
    val result = mutableMapOf<Triple<String, String, String>, String>()  
    val root = "C:\\Users\\test11\\Desktop\\abc"  //项目目录
    val fileTree: FileTreeWalk = File(root).walk()
    val fileList = fileTree
        .filter { it.isFile }	//是文件（不是文件夹）
        .filter { it.extension == "java" }//扩展名是.java
        .toList()
    fileList.forEach { file ->
        val readLines = file.readLines()
        for (index in readLines.indices) {	//遍历每行 index->行号
            if ("@Scheduled" in readLines[index]) {
                val arr = file.path.removePrefix(root).split("\\src\\main\\java\\")
                result[Triple(arr[0].removePrefix("\\"),arr[1],
                              readLines[index + 1])] = readLines[index]
            }
        }
    }
    result.forEach {
        print(format("%-20s","模块:${it.key.first}"))
        print(format("%-80s","类名:${it.key.second}"))
        print(format("%-80s","方法名:${it.key.third}"))
        print(format("%-20s","注解:${it.value}"))
        println()
    }
}
```

这里 用 `map` + `triple` 实现 保存了四个结果。在`java`中，方法最多只能有一个返回值，如果你想返回多个结果就得自定义一个类或者 使用集合来包装它来返回，在语言层面的支持颇少，在`kotlin` 中 虽然也不能返回多个值，但是  向你提供了 `Pair` 和 `Triple`，两个均为`data class`。