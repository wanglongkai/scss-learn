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

## 插值表达式`#{}`
几乎可以在 Sass 样式表中的任何地方使用,只是插值对于将值注入字符串很有用，但除此之外，在 SassScript 表达式中很少需要插值。
```scss
@mixin corner-icon($name, $top-or-bottom, $left-or-right) {
  .icon-#{$name} {
    background-image: url("/icons/#{$name}.svg");
    position: absolute;
    #{$top-or-bottom}: 0;
    #{$left-or-right}: 0;
  }
}

@include corner-icon("mail", top, left);
```
```css
.icon-mail {
  background-image: url("/icons/mail.svg");
  position: absolute;
  top: 0;
  left: 0;
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
1. 定义混合的语法：`@mixin <name> { ... } 或 @mixin name(<arguments...>) { ... }`
2. `@include <name> 或 @include <name>(<arguments...>)`,使用混合。
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
3. 可以通过定义一个默认值来使参数可选。
4. 除了按参数列表中的位置传递参数外，还可以按名称传递参数。
```scss
@mixin square($size, $radius: 0) { // 定义默认值
  width: $size;
  height: $size;

  @if $radius != 0 {
    border-radius: $radius;
  }
}

.avatar {
  @include square(100px, $radius: 4px); // 按名称传参
}
```

### Mixins内容块`@content`
1. mixin 可以通过在其正文中包含 @content 来声明它接受内容块。
```scss
@mixin hover {
  &:not([disabled]):hover {
    @content;
  }
}

.button {
  border: 1px solid black;
  @include hover {
    border-width: 2px;
  }
}
```
```css
.button {
  border: 1px solid black;
}
.button:not([disabled]):hover {
  border-width: 2px;
}
```
2. 使用`@content(<arguments...>)`可以给内容块指定参数，然后使用`@include <name> using (<arguments...>)`来给内容库传参
```scss
@mixin media($types...) {
  @each $type in $types {
    @media #{$type} {
      @content($type);
    }
  }
}

@include media(screen, print) using ($type) {
  h1 {
    font-size: 40px;
    @if $type == print {
      font-family: Calluna;
    }
  }
}
```
```css
@media screen {
  h1 {
    font-size: 40px;
  }
}
@media print {
  h1 {
    font-size: 40px;
    font-family: Calluna;
  }
}
```




## 继承/扩展
使用 @extend 可以扩展/继承一些规则。
1. 继承`%站位`内容。
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
2. `@extend <selector>` ,它告诉 Sass 一个选择器应该继承另一个选择器的样式。
```scss
.error {
  border: 1px #f00;
  background-color: #fdd;

  &--serious {
    @extend .error;
    border-width: 3px;
  }
}
```
```css
.error, .error--serious {
  border: 1px #f00;
  background-color: #fdd;
}
.error--serious {
  border-width: 3px;
}
```
只能扩展简单的选择器（如 .info 或 a 等单个选择器）

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

## `@function`定义函数
`@function <name>(<arguments...>) { ... }`语法格式，以及必须用`@return`返回。
```scss
@function sum($numbers...) {
  $sum: 0;
  @each $number in $numbers {
    $sum: $sum + $number;
  }
  @return $sum;
}

.micro {
  width: sum(50px, 30px, 100px);
}
```
```css
.micro {
  width: 180px;
}

```

## `@forward`转发模块
1. `@forward "<url>"`    
直接转发一个模块
2. `@forward "<url>" as <prefix>-*`    
添加前缀然后转发
  ```scss
    // src/_list.scss
    @mixin reset {
      margin: 0;
      padding: 0;
      list-style: none;
    }
    // bootstrap.scss
    @forward "src/list" as list-*;
    // styles.scss
    @use "bootstrap";

    li {
      @include bootstrap.list-reset;
    }
  ```
3. `@forward "<url>" hide <members...> 或 @forward "<url>" show <members...>`    
准确控制转发哪些成员
  ```scss
  // src/_list.scss
    $horizontal-list-gap: 2em;
    @mixin list-reset {
      margin: 0;
      padding: 0;
      list-style: none;
    }
    @mixin list-horizontal {
      @include list-reset;
      li {
        display: inline-block;
        margin: {
          left: -2px;
          right: $horizontal-list-gap;
        }
      }
    }

    // bootstrap.scss
    @forward "src/list" hide list-reset, $horizontal-list-gap;
  ```

4. `@forward "<url> with (<variable>: <value>, <variable>: <value>)"`     
转发前可以配置被转发的模块
  ```scss
    // _library.scss
    $black: #000 !default;
    $border-radius: 0.25rem !default;
    $box-shadow: 0 0.5rem 1rem rgba($black, 0.15) !default;

    code {
      border-radius: $border-radius;
      box-shadow: $box-shadow;
    }
    // _opinionated.scss
    @forward 'library' with (
      $black: #222 !default,
      $border-radius: 0.1rem !default
    );
    // style.scss
    @use 'opinionated' with ($black: #333);
  ```