[scope](https://so.csdn.net/so/search?q=scope&spm=1001.2101.3001.7020)=“prototype”
(1).scope 属性值
第一个值 默认值，singleton，表示是单实例对象
第二个值 prototype，表示是多实例对象

(2).singleton 和 prototype 区别
第一 singleton 单实例，prototype 多实例
第二 设置 scope 值是 singleton 时候，加载 spring 配置文件时候就会创建单实例对象设置 scope 值是 prototype 时候，不是在加载 spring 配置文件时候创建 对象，在调用getBean 方法时候创建多实例对象

