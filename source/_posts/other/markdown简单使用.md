#### 声明标题

- 使用`#`开始声明标题 后续添加title的内容。

  例如：`# this is big title #` or `### this is small title #`#的数量为字体大小，最大字号为一个#号，最多不得超过6个。

#### 字体样式

- suzeyu     默认
- *suzeyu*     斜体    —-> `*suzeyu*`
- **suzeyu**      加粗    —-> `**suzeyu**`
- ~~suzeyu~~     删除线  —-> `~~suzeyu~~`

#### 列表样式

> #### 无序

- 使用 `+`, `-`,`*` 可以创建无序列表

  > 例如 
  >
  > - red
  > - yellow
  > - blue

  - red
  - yellow
  - blue

> #### 有序
>
> - 使用
>
>    
>
>   ```
>   1.
>   ```
>
>   , 数字加上点
>
>   ​
>
>   ​
>
>   例如
>
>   1. first
>   2. second
>   3. third

```
1. first
2. second
3. third

```

#### 代码区块

- 行首四个空格或者一个制表符可进入代码区域

#### 分割线

- 任意一种都可以

  ```
  ---
  ***

  ```

#### 超链接 加载图片

- 添加一个超链接 [点我跳转github](http://www.github.com/)

  ```
  [ name ](http://xxx.xx.xx)

  ```

- 添加一个图片![logo](http://szysky.com/2016/03/24/markdown-%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8/image/demo.png)

  ```
  ![alt text](image/demo.png)

  ```

### 表格

| first | second | third |
| ----- | ------ | ----- |
| 1     | 2      | 3     |
| 4     | 5      | 6     |
| 7     | 8      | 9     |

code:

```
| first | second | third |
| ------| ------ | ----- |
|1|2|3|
|4|5|6|
|7|8|9|
```