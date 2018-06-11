# category与extension
## category
### category作用
* 可以减少单个文件的体积
* 可以把不同的功能组织到不同的category里
* 可以由多个开发者共同完成一个类
* 可以按需加载想要的category

```
比如jpush中对Delegate添加的category,避免系统中的Delegate文件过大
而且可以标记子delegate文件的功能就是'推送'相关

```
### category特点

* category只能给某个已有的类扩充方法，不能扩充成员变量。

```
可以使用曲线救国的方式,实现添加成员变量,思路是,建立set方法和get方法,
然后使用runtime的方式obj_association... 的方式完成动态添加成员变量
```

* 如果category中的方法和类中原有方法同名，运行时会优先调用category中的方法。

```
也就是，category中的方法会覆盖掉类中原有的方法。
所以开发中尽量保证不要让分类中的方法和原有类中的方法名相同。
避免出现这种情况的解决方案是给分类的方法名统一添加前缀。比如category_methodName。
```

* 如果多个category中存在同名的方法，运行时到底调用哪个方法由编译器决定，最后一个参与编译的方法会被调用。

## extension

### extension是什么
```
@interface ViewController ()
 
@end
```

extension更像是一个类的封装,一般存在于.m文件中(也可以在.h中)

### extension作用
* extension一般用来隐藏类的私有信息

### extension与category的区别
* extension的逻辑处于编译期
* category的逻辑更多的处于运行期(runtime)

```
正因为这样,所以category可以去未已有的类型添加方法,或者重载方法
而extension,更多像是类的一部分,只是他构建的属性和方法更加私密
```







