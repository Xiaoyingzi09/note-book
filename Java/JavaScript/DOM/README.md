# DOM(文档对象模型)

- DOM

  - 浏览器已经为我们提供 文档节点 对象这个对象是window属性
  - 可以在页面中直接使用，文档节点代表的是整个网页

  

- 事件

  - 事件，就是用户和浏览器之间的交互行为，比如：点击按钮，鼠标移动、关闭窗口。。。
  - 我们可以在事件对应的属性中设置一些js代码，这样当事件被触发时，这些代码将会执行
    这种写法我们称为**结构和行为耦合**，不方便维护，不推荐使用
  - eg：可以为按钮的对应事件绑定处理函数的形式来响应事件。这样当事件被触发时，其对应的函数将会被调用。绑定一个单击事件像这种为单击事件绑定的函数，我们称为单击响应函数。

- 文档的加载
  - 浏览器在加载一个页面时，是按照自上向下的顺序加载的，读取到一行就运行一行,如果将script标签写到页面的上边，在代码执行时，页面还没有加载，页面没有加载DOM对象也没有加载会导致无法获取到DOM对象
  - onload事件会在整个页面加载完成之后才触发，为window绑定一个onload事件
    - 该事件对应的响应函数将会在页面加载完成之后执行，这样可以确保我们的代码执行时所有的DOM对象已经加载完毕了
  - 将js代码编写到页面的下部就是为了，可以在页面加载完毕以后再执行js代码
  - for循环会在页面加载完成之后立即执行，而响应函数会在超链接被点击时才执行，当响应函数执行时，for循环早已执行完毕




## DOM查询

- DOM查询
  - *document*（获取元素节点，在整个页面查询）
    - *document.getElementById()*
      - 通过id属性获取一个元素节点对象
    - *document.getElementByTagName()*
      - 通过标签名获取一组元素节点对象
      - *childNodes*，表示当前节点的**所有子节点**
        - *childNodes*属性会获取包括文本节点在呢的所有节点，根据DOM标签标签间空白也会当成文本节点；在IE8及以下的浏览器中，不会将空白文本当成子节点
        - *children*属性可以获取当前元素的所有**子元素**
      - *firstChild*，表示当前节点的第一个子节点
        - *firstChild*可以获取到当前元素的**第一个子节点（包括空白文本节点）**
        - *firstElementChild*获取当前元素的**第一个子元素**：*firstElementChild*不支持IE8及以下的浏览器，如果需要兼容他们尽量不要使用
      - *lastChild*，表示当前节点的**最后一个子节点**
      - 获取父节点和兄弟节点
        - *parentNode*：表示当前节点的父节点
        - *previousSibling*：表示当前节点的前一个兄弟节点，也可能获取到空白的文本。*previousElementSibling*获取前一个兄弟元素，IE8及以下不支持
        - *nextSibling*：表示当前节点的后一个兄弟节点
    - *document.getElementByName()*
      - 通过name属性获取一组元素节点对象
  - *document.getElementsByTagName*()：可以根据标签名来获取一组元素节点对象。这个方法会给我们返回一个类数组对象，所有查询到的元素都会封装到对象中。即使查询到的元素只有一个，也会封装到数组中返回。
  - *innerHTML* 通过这个属性可以获取到元素内部的*html*代码，对于自结束标签，这个属性没有意义
  - *innerText* 该属性可以获取到元素内部的文本内容，它和*innerHTML*类似，不同的是它会自动将*html*去除
  - 如果需要读取元素节点属性，
    - 直接使用 元素.属性名
    - 例子：元素.id 元素.name 元素.value
      注意：class属性不能采用这种方式，
      读取class属性时需要使用 元素.*className*
  - 在事件的响应函数中，响应函数是给谁绑定的this就是谁
  - DOM查询的其他方法
    - 获取body标签//*var* *body* = *document.getElementsByTagName*("*body*")[0],在document中有一个属性body，它保存的是body的引用
    - *document.documentElement*保存的是html根标签
    - *document.all*代表页面中所有的元素
    - 根据元素的class属性值查询一组元素节点对象
    -  *getElementsByClassName*()可以根据class属性值获取一组元素节点对象，但是该方法不支持IE8及以下的浏览器
    - 获取页面中的所有的div://*var* *divs* = *document.getElementsByTagName*("div")
    - *document.querySelector*()
      - 需要一个选择器的字符串作为参数，可以根据一个CSS选择器来查询一个元素节点对象
      - 虽然IE8中没有*getElementsByClassName*()但是可以使用*querySelector*()代替
      - 使用该方法总会返回唯一的一个元素，如果满足条件的元素有多个，那么它只会返回第一个
      - *document.querySelectorAll*()
        - 该方法和*querySelector*()用法类似，不同的是它会将符合条件的元素封装到一个数组中返回
        -  即使符合条件的元素只有一个，它也会返回数组



## DOM增删改

- DOM增
  - document.createElement()：可以用于创建一个元素节点对象，它需要一个标签名作为参数，将会根据该标签名创建元素节点对象，并将创建好的对象作为返回值返回。
  - document.createTextNode()：可以用来创建一个文本节点对象，需要一个文本内容作为参数，将会根据该内容创建文本节点，并将新的节点返回。
  - appendChild()：向一个父节点中添加一个新的子节点，用法：父节点.appendChild(子节点)
  - insertBefore()：可以在指定的子节点前插入新的子节点。语法：父节点.insertBefore(新节点,旧节点)
  - replaceChild()：可以使用指定的子节点替换已有的子节点。 语法：父节点.replaceChild(新节点,旧节点)
  - 使用innerHTML也可以完成DOM的增删改的相关操作，一般我们会两种方式结合使用：创建一个li，向li中设置文本
- DOM删
  - removeChild()： 可以删除一个子节点。语法：父节点.removeChild(子节点)；常用：子节点.parentNode.removeChild(子节点)；



## 使用DOM操作CSS

- 修改样式
  - 通过JS修改元素的样式：语法：元素.style.样式名 = 样式值 
    - 注意：如果CSS的样式名中含有-，这种名称在JS中是不合法的比如background-color，需要将这种样式名修改为驼峰命名法，去掉-，然后将-后的字母大写。
    - 我们通过style属性设置的样式都是内联样式，而内联样式有较高的优先级，所以通过JS修改的样式往往会立即显示。但是如果在样式中写了!important，则此时样式会有最高的优先级，即使通过JS也不能覆盖该样式，此时将会导致JS修改样式失效，所以尽量不要为样式添加!important
  - 读取样式：语法：元素.style.样式名
    - 通过style属性设置和读取的都是内联样式，无法读取样式表中的样式
    - 获取元素的当前显示的样式：
      - 语法：元素.currentStyle.样式名
      - 它可以用来读取当前元素正在显示的样式，如果当前元素没有设置该样式，则获取它的默认值，**currentStyle只有IE浏览器支持**，其他的浏览器都不支持
    - 在其他浏览器中可以使用：getComputedStyle()这个方法来获取元素当前的样式
      - 这个方法是window的方法，可以直接使用，需要两个参数：
        - 第一个：要获取样式的元素
        - 第二个：可以传递一个伪元素，一般都传null
      - 该方法会返回一个对象，对象中封装了当前元素对应的样式
        - 可以通过对象.样式名来读取样式
        - 如果获取的样式没有设置，则会获取到真实的值，而不是默认值，比如：没有设置width，它不会获取到auto，而是一个长度。但是该方法不支持IE8及以下的浏览器。通过currentStyle和getComputedStyle()读取到的样式都是只读的，不能修改，如果要修改必须通过style属性
    -  定义一个函数，用来获取指定元素的当前的样式。参数：obj 要获取样式的元素；name 要获取的样式名。正常浏览器的方式，具有getComputedStyle()方法。**IE8的方式，没有getComputedStyle()方法**。
  - 其他样式操作
    - clientWidth/clientHeight
      -  这两个属性可以获取元素的可见宽度和高度
      - 这些属性都是不带px的，返回都是一个数字，可以直接进行计算
      - 会获取元素宽度和高度，包括内容区和内边距
      - 这些属性都是只读的，不能修改
    - offsetWidth/offsetHeight
      - 获取元素的整个的宽度和高度，包括内容区、内边距和边框
    - offsetParent
      - 可以用来获取当前元素的定位父元素
      - 会获取到离当前元素最近的开启了定位的祖先元素
      - 如果所有的祖先元素都没有开启定位，则返回body
    - offsetLeft：当前元素相对于其定位父元素的水平偏移量
    - offsetTop：当前元素相对于其定位父元素的垂直偏移量
    - scrollWidth/scrollHeight：可以获取元素整个滚动区域的宽度和高度
    - scrollLeft：可以获取水平滚动条滚动的距离
    - scrollTop：可以获取垂直滚动条滚动的距离
    - 当满足scrollHeight - scrollTop == clientHeight：说明垂直滚动条滚动到底了
    - 当满足scrollWidth - scrollLeft == clientWidth：说明水平滚动条滚动到底











