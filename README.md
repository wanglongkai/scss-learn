# scss学习记录
## Live Sass Compiler	
vscode中实时编译scss文件为css文件：Live Sass Compiler插件is very goood.

## 变量
```scss
$heighlight-color: #F90;
```
任何可以用作css属性值的赋值都可以用作sass的变量值，甚至是以空格分割的多个属性值。
比如：     
```scss
$border: 1px solid red;
```
scss变量是 **{ }** 作用域:    
```scss
$nav-color: #F90;
nav {
  $width: 100px;
  width: $width;
  color: $nav-color;
}

//编译后
nav {
  width: 100px;
  color: #F90;
}
```

`$link-color`和`$link_color`其实指向的是同一个变量。**两种用法相互兼容**。个人偏好下划线，因为可以在编辑器内双击选中。


## 嵌套
