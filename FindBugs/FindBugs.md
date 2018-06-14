 | 类型 | 描述 |
 | -- | -- |
 | CN_IDIOM_NO_SUPER_CALL | 未调用`super.clone()` |
 | UPM_UNCALLED_PRIVATE_METHOD | 私有方法从未被调用 | 
 | UWF_FIELD_NOT_INITIALIZED_IN_CONSTRUCTOR | 字段未在构造函数中初始化,未经空值检查进行使用,可能导致`NullPointerException` |
 | SIC_INNER_SHOULD_BE_STATIC_ANON | 可以重构为一个静态内部类。匿名内部类会导致实例更大,引用周期更长 |
 | SF_SWITCH_NO_DEFAULT | `Switch`缺少默认情况下的处理（`default`语句）|
 | DM_BOXED_PRIMITIVE_FOR_PARSING | 使用 `parseXXX` 代替 `valueOf` |
 

#### [查看更多](http://findbugs.sourceforge.net/bugDescriptions.html)
