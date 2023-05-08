# 一些常用类

记录一些他人项目中使用到的类

 

# SimpleDateFormat 

安全的时间格式化，该类用来对日期字符串进行解析和格式化输出，但如果使用不小心会导致非常微妙和难以调试的问题，因为 DateFormat 和 SimpleDateFormat 类不都是线程安全的，在多线程环境下调用 format() 和 parse() 方法应该使用同步代码来避免问题

