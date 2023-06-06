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
```scss
.a{
  color:red;
  .b{
    font-size: 15px;
    .c{
      text-align: center;
    }
  }
}

// 编译为：
.a {
  color: red;
}
.a .b {
  font-size: 15px;
}
.a .b .c {
  text-align: center;
}
```

1. 父选择器站位`&`
```scss
.a{
  color:red;
  .b{
    font-size: 15px;
    // 注意：& 站位的是 .b
    &_c{
      text-align: center;
    }
  }
}
//编译为：
.a {
  color: red;
}
.a .b {
  font-size: 15px;
}
.a .b_c {
  text-align: center;
}

.a, .b{
  .c, .d{
    color: red;
  }
}
//编译为：
.a .c, .a .d, .b .c, .b .d {
  color: red;
}
```

## 占位符选择器`%`
占位符选择器中的规则不会包含到css中。
```scss
.alert:hover, %strong-alert {
  font-weight: bold;
}

// %strong-alert为占位符选择器，不会真实编译，类似于后端语言中的接口，没有实际应用，但可以被继承@extend
%strong-alert:hover {
  color: red;
}
// 编译为
.alert:hover {
  font-weight: bold;
}
```

## partials
 partial 是一个以下划线开头的 Sass 文件。您可以将其命名为 _partial.scss 。下划线让 Sass 知道该文件只是一个部分文件，不应将其生成到 CSS 文件中。 Sass 部分与 @use 规则一起使用。
 1. 声明一个partials文件
   ```scss
   <!-- _part.scss -->
    @mixin name {
      border: 1px solid blue;
    }
   ```
 2. 使用partials
   ```scss
   @use 'part';
  .nameouter{
    @include part.name
  }
  //编译为：
  .nameouter {
    border: 1px solid blue;
  }
   ```

## Mixins
创建要在整个站点中重用的 CSS 声明组。甚至可以传入值来mixin 更加灵活。
```scss
@mixin theme($theme: DarkGray) {
  background: $theme;
  box-shadow: 0 0 1px rgba($theme, .25);
  color: #fff;
}

.info {
  @include theme;
}
.alert {
  @include theme($theme: DarkRed);
}
.success {
  @include theme($theme: DarkGreen);
}
// 编译为：
.info {
  background: DarkGray;
  box-shadow: 0 0 1px rgba(169, 169, 169, 0.25);
  color: #fff;
}

.alert {
  background: DarkRed;
  box-shadow: 0 0 1px rgba(139, 0, 0, 0.25);
  color: #fff;
}

.success {
  background: DarkGreen;
  box-shadow: 0 0 1px rgba(0, 100, 0, 0.25);
  color: #fff;
}
```

## 继承/扩展
使用 @extend 可以扩展/继承一些规则。
```scss

%message-shared {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}

.message {
  @extend %message-shared;
}

.success {
  @extend %message-shared;
  border-color: green;
}

.error {
  @extend %message-shared;
  border-color: red;
}

.warning {
  @extend %message-shared;
  border-color: yellow;
}
//编译为：
.message, .success, .error, .warning {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}

.success {
  border-color: green;
}

.error {
  border-color: red;
}

.warning {
  border-color: yellow;
}
```

## @use
加载其他模块，包括sass提供的内置模块。    
**注意：**  @use 规则必须位于除 @forward 之外的任何规则之前，包括样式规则。但是，您可以在 @use 规则之前声明变量以在配置模块时使用。
### `@use "<url>"`
导入模块，模块名为最后一个组成部分。    
```scss
// src/_corners.scss
$radius: 3px;

@mixin rounded {
  border-radius: $radius;
}
```
```scss
// style.scss
@use "src/corners";

.button {
  @include corners.rounded;
  padding: 5px + corners.$radius;
}
```
### `@use "<url>" as <namespace>`
为导入模块命名。    
```scss
// src/_corners.scss
$radius: 3px;

@mixin rounded {
  border-radius: $radius;
}
```
```scss
// style.scss
@use "src/corners" as c;

.button {
  @include c.rounded;
  padding: 5px + c.$radius;
}
```

### `@use "<url>" as * `
加载没有命名空间的模块。建议您只对确定的样式表执行此操作；否则，可能会引入新成员，导致名称冲突！
```scss
// src/_corners.scss
$radius: 3px;

@mixin rounded {
  border-radius: $radius;
}
```
```scss
// style.scss
@use "src/corners" as *;

.button {
  @include rounded;
  padding: 5px + $radius;
}
```

### 定义模块私有成员
Sass 通过以 `- 或 _ `开头的名称使定义私有成员。    
这些成员将在定义它们的样式表中正常工作，但它们不会成为模块公共 API 的一部分。这意味着加载模块的样式表看不到它们！

### `@use <url> with (<variable>: <value>, <variable>: <value>)`
样式表可以使用 `!default` 标志定义变量以使其可配置。配置的值将覆盖变量的**默认值**
```scss
$color: red !default;
@mixin name {
  border: 1px solid $color; // 默认为red;
}
```
```scss
@use 'part' with (
  $color: blue, // 配置为blue
);
```