## 使用

```gradle
// ...代表相对路径
apply from: '.../findbugs.gradle'
```

**findbugs.gradle**
```gradle
    apply plugin: "findbugs"
    
    task findbugs(type: FindBugs, dependsOn: 'assembleDebug') {
        ignoreFailures = true
        effort = "max"
        showProgress = true
        reportLevel = "low" // high medium low
        println("$project.buildDir")
        excludeFilter = rootProject.file('tools/FindBugsFilter.xml')
        classes = files("$project.buildDir/intermediates/javac")
        source = fileTree("src/main/java/")
        classpath = files()
        reports {
            xml.enabled false
            html.enabled true
            html.setDestination(new File("$project.buildDir/findbugs.html"))
        }
    }
```

**FindBugsFilter.xml**
```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <FindBugsFilter>
        <!-- 屏蔽R文件 和 清单文件 -->
        <Match>
            <Class name="~.*\.R\$.*"/>
        </Match>
        <Match>
            <Class name="~.*\.Manifest\$.*"/>
        </Match>
        <!-- 屏蔽测试代码 -->
        <Match>
            <Class name="~.*\.*Test" />
            <Not>
                <Bug code="IJU" />
            </Not>
        </Match>
    </FindBugsFilter>
```

## 常见 Bug 类型

 | 类型 | 描述 |
 | -- | -- |
 | CN_IDIOM_NO_SUPER_CALL | 未调用`super.clone()` |
 | UPM_UNCALLED_PRIVATE_METHOD | 私有方法从未被调用 | 
 | UWF_FIELD_NOT_INITIALIZED_IN_CONSTRUCTOR | 字段未在构造函数中初始化,未经空值检查进行使用,可能导致`NullPointerException` |
 | SIC_INNER_SHOULD_BE_STATIC_ANON | 可以重构为一个静态内部类。匿名内部类会导致实例更大,引用周期更长 |
 | SF_SWITCH_NO_DEFAULT | `Switch`缺少默认情况下的处理（`default`语句）|
 | DM_BOXED_PRIMITIVE_FOR_PARSING | 使用 `parseXXX` 代替 `valueOf` |
 | DP_DO_INSIDE_DO_PRIVILEGED  | `xxx.setAccessible(xxx)` 代码块使用 `AccessController.doPrivileged(...)` 包装 |

## [查看更多](http://findbugs.sourceforge.net/bugDescriptions.html)
