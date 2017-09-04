
---
title: Node.js学习笔记 lesson1
date: 
---

#### Golbal

1. `__dirname` : 全路径名

2. `__filename`:文件的绝对路径

3. `console.log(--dirname)`:日志输出

4. `module.exports`:定义一个模块导出什么，且通过`require()`引入。`module`实际上不是全局变量，而是每个模块内部的。

5. `require()`：用于引入变量，也是每个模块内部的。

#### module

1. 在Node.js中，文件和模块是一一对应的，每个文件被视为一个独立模块。

2. ```
   const circle = require('./circle.js');
   console.log('半径为 4 的圆的面积是 ${circle.area(4)}')
   ```

   ```
   const PI = Math.PI;
   exports.area = (r) =>PI *r *r;
   exports.circumference = (r) =>2 * PI *r;
   ```

   模块内的本地变量是私有的，因为模块被Node.js包装在一个函数中。

3. 如果希望模块根导出为一个函数(比如构造函数)，或者一次导出一个完整的对象而不是每次创建一个属性，可以将它赋值给`module.exports`，而不是`exports`。

4. ```
   const square = require('./square.js');
   var mySquare = square(2);
   console.log('正方形的面积是 ${mySquare.area()}');
   ```

5. ```
   module.exports = (width) =>{
     return {
       area:() => width * width;
     }
   }
   ```

6. 模块第一次加载后会被缓存。以后每次调用都会解析到同一个文件，返回相同的对象。多次调用`require(foo)`不会导致模块代码被执行多次，如果想多次执行一个模块，可以先导出一个函数，然后调研该函数。

7. 模块是基于其缓存的模块名进行解析。调用模块的位置不同，模块可能别解析成不同的文件名，这样就不能保证`require('foo')`总能返回同一个对象。

8. 为了防止无限的循环，相互调用时，会先返回一个未完成的副本。

9. 以`/`为前缀的模块是文件的绝对路径。

   以`./`为前缀的模块是相对于调用`require()`的文件的。

   ​
