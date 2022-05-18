#### equals()和==和hashcode()

#### String,StringBuffer,StringBuilder

- String不可变，StringBuffer和StringBuilder可变
- StringBuffer线程安全，内部使用synchronized进行同步， StringBuilder线程不安全

#### Java创建对象的方式

- 使用new 关键字
- 使用反射class.newInstance()
- 调用对象clone()方法