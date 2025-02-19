# js运行机制

- [绑定事件的不同种方式，执行顺序，事件委托](#绑定事件的不同种方式，执行顺序，事件委托)
- [js 数据类型与隐式转换](#js数据类型与隐式转换)
- [js 事件循环机制](#js事件循环机制)
- [闭包数据缓存与垃圾回收](#闭包数据缓存与垃圾回收)
- [DOM 树解析和更改与遍历](#DOM树解析和更改与遍历)
- [运算符优先级](#运算符优先级)
- [作用域与变量等题目](#作用域与变量等题目)
- [严格模式](#严格模式)
- [前端路由原理](#前端路由原理)
- [抽象语法树 AST 与 babel](#抽象语法树AST与babel)
- [判断js运行环境](#判断js运行环境)
- [let、const、var的区别](#let、const、var的区别)
- [箭头函数与普通函数](#箭头函数与普通函数)
- [rest扩展运算符](#rest扩展运算符)
- [map和weakMap](#map和weakMap)

---

### 绑定事件的不同种方式，执行顺序，事件委托

1. 参考链接：

  [看懂此文，不再困惑于 javascript 中的事件绑定、事件冒泡、事件捕获和事件执行顺序](https://blog.csdn.net/aitangyong/article/details/43231111)

  [js 中的事件委托或是事件代理详解](https://www.cnblogs.com/liugang-vip/p/5616484.html)

  [「2021」高频前端面试题汇总之浏览器原理篇](https://juejin.cn/post/6916157109906341902/)

2. 详解：

  * 事件绑定3 种方式：

  ```html
  <p id="btn" onclick="hello()"></p>
  <script>
    document.getElementById("btn").onclick = function () {};
    document.getElementById("btn").addEventListener("click", function () {});
  </script>
  ```

  其中 addEventListener 可重复绑定同一元素，先绑定先执行。

  对于层叠元素，则需要区分事件冒泡和事件捕获，冒泡：从底向面，捕获：从面向底。addEventListener((type, listener, useCapture)

  * 阻止冒泡
  
    只执行当前元素事件，不执行层叠元素事件。event.stopPropagation()

  * 事件委托
  
    利用事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。例如 ul 下有很多 li，逐一绑定事件很影响性能，且新 li 加入也要重新绑定事件，会十分麻烦，所以 li 事件需要委托其上一级 ul 代为执行事件。

    ```html
    <ul id="list">
      <li>item 1</li>
      <li>item 2</li>
      <li>item 3</li>
      ......
      <li>item n</li>
    </ul>
    ```

    ```js
    window.onload = function () {
      var oUl = document.getElementById("ul1");
      oUl.onclick = function (ev) {
        var ev = ev || window.event;
        var target = ev.target || ev.srcElement;
        if (target.nodeName.toLowerCase() == "li") {
          alert(123);
          alert(target.innerHTML);
        }
      };
    };
    ```

    * 特点

      1. 减少内存消耗

        如果给每个列表项一一都绑定一个函数，那对于内存消耗是非常大的，效率上需要消耗很多性能。因此，比较好的方法就是把这个点击事件绑定到他的父层，也就是 ul 上，然后在执行事件时再去匹配判断目标元素，所以事件委托可以减少大量的内存消耗，节约效率。

      2. 动态绑定事件

        每个列表项都绑定事件，在很多时候，需要通过 AJAX 或者用户操作动态的增加或者去除列表项元素，那么在每一次改变的时候都需要重新给新增的元素绑定事件，给即将删去的元素解绑事件；如果用了事件委托就没有这种麻烦了，因为事件是绑定在父层的，和目标元素的增减是没有关系的，执行到目标元素是在真正响应执行事件函数的过程中去匹配的，所以使用事件在动态绑定事件的情况下是可以减少很多重复工作的。

    * 局限性

      1. focus、blur 之类的事件没有事件冒泡机制，所以无法实现事件委托
      2. mousemove、mouseout 这样的事件，虽然有事件冒泡，但是只能不断通过位置去计算定位，对性能消耗高，因此也是不适合于事件委托的。
      3. 事件委托会影响页面性能，主要影响因素有：

        * 元素中，绑定事件委托的次数；
        * 点击的最底层元素，到绑定事件元素之间的DOM层数；

      * 在必须使用事件委托的地方，可以进行如下的处理

        * 只在必须的地方，使用事件委托，比如：ajax的局部刷新区域
        * 尽量的减少绑定的层级，不在body元素上，进行绑定
        * 减少绑定的次数，如果可以，那么把多个事件的绑定，合并到一次事件委托中去，由这个事件委托的回调，来进行分发。

  * 关于 onclick,addEventListener('click'),\$('...').on('click')

    1. 实际上，3 种写法都可以转化为 addEventListener('click',function,options)的形式

    2. 在没有设置 options 的情况下，click 后会先事件捕获，然后事件冒泡，options 中默认捕获为 false，所以 function 是在冒泡阶段执行，由内层冒泡至外层

    3. options 中设置为 true，则 function 在捕获阶段执行

    4. options 还有其它 2 个参数，可与 capture 一起以对象的形式传入

      ```js
      document
        .getElementsByClassName("middle")[0]
        .addEventListener("click", handler, {
          capture: true,
          once: true, //只执行一次
          passive: true, //永不调用 preventDefault(),如果调用，控制台会出现警告
        });
      ```

    5. click 的元素指向内层的元素，外层捕获进来后到达最内层，最内层按先后顺序执行完 function，再冒泡向上

    6. event.stopPropagation()表示停止传播，在哪一层的事件中设置了，就不会传播到下一层，例如在 middle 层设置 capture 为 true，则不会再向下捕获再冒泡

    7. 点击哪一层元素，则以哪一层元素为捕获和冒泡的底部

    8. 例子

      ```html
      <div class="out" onclick="console.log('out')">
        <div class="middle" onclick="console.log('middle')">
          <div class="inner" onclick="console.log('inner')"></div>
        </div>
      </div>
      <style>
        .out {
          width: 100px;
          height: 100px;
          border: 1px solid red;
        }
        .middle {
          width: 80px;
          height: 80px;
          border: 1px solid blue;
        }
        .inner {
          width: 40px;
          height: 40px;
          border: 1px solid green;
        }
      </style>
      <script>
        $(() => {
          $(".out").on("click", function () {
            console.log("out1");
          });
          $(".middle").on("click", function () {
            console.log("middle1");
          });
          $(".inner").on("click", function () {
            console.log("inner1");
          });
          document
            .getElementsByClassName("out")[0]
            .addEventListener("click", function () {
              console.log("out2");
            });
          document
            .getElementsByClassName("middle")[0]
            .addEventListener("click", function () {
              console.log("middle2");
            });
          document
            .getElementsByClassName("inner")[0]
            .addEventListener("click", function () {
              console.log("inner2");
            });
          document.getElementsByClassName("out")[0].addEventListener(
            "click",
            function () {
              console.log("out3");
            },
            true
          );
          document.getElementsByClassName("middle")[0].addEventListener(
            "click",
            function () {
              console.log("middle3");
            },
            true
          );
          document.getElementsByClassName("inner")[0].addEventListener(
            "click",
            function () {
              console.log("inner3");
            },
            true
          );
        });
      </script>
      <!--点击inner的输出：
          out3 middle3 inner inner1 inner2 inner3 middle middle1 middle2 out out1 out2
          点击middle的输出：
          out3 middle middle1 middle2 middle3 out out1 out2
      -->
      ```

### js 数据类型与隐式转换

1. 参考链接：

   [JS 的隐式转换 从 [] ==false 说起](https://www.cnblogs.com/nanchen/p/7905528.html)

   [JS 中 [] == ![]结果为 true，而 {} == !{}却为 false， 追根刨底](https://blog.csdn.net/magic_xiang/article/details/83686224)

   [JavaScript 中 valueOf、toString 的隐式调用](https://www.cnblogs.com/barrior/p/4598354.html)

   [JavaScript 中的变量在内存中的具体存储形式](https://www.jianshu.com/p/80bb5a01857a)

   [JavaScript 的数据类型](https://www.cnblogs.com/cider/p/11875832.html)

   [null 与 undefined 的区别？](https://www.cnblogs.com/shengmo/p/8671803.html)

   [面试造火箭，看下这些大厂原题](https://juejin.im/post/6859121743869509646)

   [12 道腾讯前端面试真题及答案整理](https://mp.weixin.qq.com/s/mouL2lrCvttHpMwP4iesKw)

   [ECMAScript w3cschool](https://www.w3school.com.cn/js/pro_js_operators_equality.asp)

   [JS类型转换规则详解](https://www.cnblogs.com/Renyi-Fan/p/9189441.html)

   [昨天面试的6道面试题](https://juejin.cn/post/6904653808153362439)

   [JavaScript 数据类型之 Symbol、BigInt](https://blog.csdn.net/lx11573/article/details/107250033)

   [「万字总结」熬夜总结50个JS的高级知识点，全都会你就是神！！！](https://juejin.cn/post/7022795467821940773)

   [「2021」高频前端面试题汇总之JavaScript篇（上）](https://juejin.cn/post/6940945178899251230)

2. 详解：

   - js 数据类型

     原始值（primitives）：undefined， null， booleans， numbers，strings， symbol（es6），BigInt（es6）

     对象值（objects）：Object

     对象键支持的类型：string,symbol

   - 数据类型检测

      typeof
      ```js
      console.log(typeof 2);               // number
      console.log(typeof true);            // boolean
      console.log(typeof 'str');           // string
      console.log(typeof []);              // object    
      console.log(typeof function(){});    // function
      console.log(typeof {});              // object
      console.log(typeof undefined);       // undefined
      console.log(typeof null);            // object
      ```

      instanceof
      ```js
      console.log(2 instanceof Number);                    // false
      console.log(true instanceof Boolean);                // false 
      console.log('str' instanceof String);                // false 
      console.log([] instanceof Array);                    // true
      console.log(function(){} instanceof Function);       // true
      console.log({} instanceof Object);                   // true
      ```

      constructor
      ```js
      console.log((2).constructor === Number); // true
      console.log((true).constructor === Boolean); // true
      console.log(('str').constructor === String); // true
      console.log(([]).constructor === Array); // true
      console.log((function() {}).constructor === Function); // true
      console.log(({}).constructor === Object); // true
      ```

      Object.prototype.toString.call()
      ```js
      var a = Object.prototype.toString;
      console.log(a.call(2));//[object Number]
      console.log(a.call(true));//[object Boolean]
      console.log(a.call('str'));//[object String]
      console.log(a.call([]));//[object Array]
      console.log(a.call(function(){}));//[object Function]
      console.log(a.call({}));//[object Object]
      console.log(a.call(undefined));//[object Undefined]
      console.log(a.call(null));//[object Null]
      ```

   - symbol

      它的静态方法会暴露全局的symbol注册，且类似于内建对象类，但作为构造函数来说它并不完整，因为它不支持语法：“new Symbol()”。

      每个从Symbol()返回的symbol值都是唯一的。一个symbol值能作为对象属性的标识符；这是该数据类型仅有的目的。

      ```js
      const s = Symbol();
      typeof s; // symbol
      const s1 = Symbol('foo');
      const s2 = Symbol('foo');
      s1 === s2; // false
      const s3 = Symbol.for('foo');
      const s4 = Symbol.for('foo');
      s3 === s4; // true
      const s5 = Symbol.for('true');
      const s6 = Symbol.for(true);
      s5 === s6; // true
      ```

      - 应用场景

        1. 使用Symbol来作为对象属性名,使指定属性不被枚举和JSON.stringify

          ```js
          const gender = Symbol('gender')
          const obj = {
            name: 'Sunshine_Lin',
            age: 23,
            [gender]: '男'
          }
          console.log(obj['name']) // 'Sunshine_Lin'
          console.log(obj['age']) // 23
          console.log(obj[gender]) // 男
          console.log(Object.keys(obj)) // [ 'name', 'age' ]
          for(const key in obj) {
            console.log(key) // name age
          }
          console.log(JSON.stringify(obj)) // {"name":"Sunshine_Lin","age":23}
          ```

          获取Symbol属性
          ```js
          // 方法一
          console.log(Object.getOwnPropertySymbols(obj)) // [ Symbol(gender) ]
          // 方法二
          console.log(Reflect.ownKeys(obj)) // [ 'name', 'age', Symbol(gender) ]
          ```

        2. 使用Symbol定义类的私有属性

          ```js
          class Login {
            constructor(username, password) {
              const PASSWORD = Symbol()
              this.username = username
              this[PASSWORD] = password
            }
            checkPassword(pwd) { return this[PASSWORD] === pwd }
          }

          const login = new Login('123456', 'hahah')

          console.log(login.PASSWORD) // 报错
          console.log(login[PASSWORD]) // 报错
          console.log(login[PASSWORD]) // 报错
          ```

   - BigInt

      ES2020 引入了一种新的数据类型 BigInt，它可以表示任意精度格式的整数。为了与 Number 类型进行区分，BigInt 类型的数据必须添加后缀n。

      ```js
      12 	// 普通Number
      12n // BigInt
      
      // BigInt 的运算
      1n + 2n // 3n

      // 与Number 类型进行运算
      1 + 1n // Uncaught TypeError

      12n === 12 // false

      BigInt(number) // 将一个 Number 转换为 BigInt
      Number(bigint) // 将一个 BigInt 转换为 Number

      typeof 12n // 'bigint'

      //不能使用 new BigInt() 的方式来构建实例
      new BigInt() // Uncaught TypeError: BigInt is not a constructor at new BigInt

      //创建一个 BigInt 的时候，参数必须为整数
      BigInt(1.2) // Uncaught RangeError: The number 1.2 cannot be converted to a BigInt because it is not an integer
      ```

   - null 和 undefined

      Undefined 和 Null 都是基本数据类型，这两个基本数据类型分别都只有一个值，就是 undefined 和 null

      undefined 代表的含义是未定义，null 代表的含义是空对象。一般变量声明了但还没有定义的时候会返回 undefined，null主要用于赋值给一些可能会返回对象的变量，作为初始化。

      typeof null 的结果是Object，typeof undefined 的结果是undefined

      因为 undefined 是一个标识符，所以可以被当作变量来使用和赋值，但是这样会影响 undefined 的正常判断。表达式 void ___ 没有返回值，因此返回结果是 undefined。void 并不改变表达式的结果，只是让表达式不返回值。因此可以用 void 0 来获得 undefined。

   - 内置对象

      1. 值属性，这些全局属性返回一个简单值，这些值没有自己的属性和方法。例如 Infinity、NaN、undefined、null 字面量
      2. 函数属性，全局函数可以直接调用，不需要在调用时指定所属对象，执行结束后会将结果直接返回给调用者。例如 eval()、parseFloat()、parseInt() 等
      3. 基本对象，基本对象是定义或使用其他对象的基础。基本对象包括一般对象、函数对象和错误对象。例如 Object、Function、Boolean、Symbol、Error 等
      4. 数字和日期对象，用来表示数字、日期和执行数学计算的对象。例如 Number、Math、Date
      5. 字符串，用来表示和操作字符串的对象。例如 String、RegExp
      6. 可索引的集合对象，这些对象表示按照索引值来排序的数据集合，包括数组和类型数组，以及类数组结构的对象。例如 Array
      7. 使用键的集合对象，这些集合对象在存储数据时会使用到键，支持按照插入顺序来迭代元素。例如 Map、Set、WeakMap、WeakSet
      8. 矢量集合，SIMD 矢量集合中的数据会被组织为一个数据序列。例如 SIMD 等
      9. 结构化数据，这些对象用来表示和操作结构化的缓冲区数据，或使用 JSON 编码的数据。例如 JSON 等
      10. 控制抽象对象。例如 Promise、Generator 等
      11. 反射。例如 Reflect、Proxy
      12. 国际化，为了支持多语言处理而加入 ECMAScript 的对象。例如 Intl、Intl.Collator 等
      13. WebAssembly
      14. 其他。例如 arguments

   - 变量储存形式

     1. 普通变量存在栈，对象(如 object，array)存在堆。
     2. let b = { x: 10 };b 存在栈，值为对象访问地址，指向{ x: 10 }的堆空间。
     3. 基本类型复制会在栈中分配新空间，引用类型复制(浅复制)栈中的新空间的值会指向旧堆的地址。

     - 栈

       储存基本数据类型，按值访问，储存的值大小固定，系统分配内存空间，空间小，运行效率高，先进后出。

     - 堆

       储存引用类型，按引用访问，储存的值大小不固定，可动态调整，代码进行指定分配，空间大，运行效率低，无序储存。

   - 有了基本类型为什么还要包装类型？

     3 个特殊的引用类型：Boolean、Number 和 String，每当读取一个基本类型值的时候，会创建一个对应的基本包装类型的对象，从而能够调用一些方法来操作这些基本类型(substring 等)。每个包装类型都映射到同名的基本类型。

   - 装箱和拆箱

     1. 装箱就是把基本类型转换为对应的内置对象，这里可分为隐式和显式装箱。

        - 隐式装箱

          ```txt
          var s1 = "stringtext";
          var s2 = s1.substring(2); 基本类型本来是没方法的，装箱后就有了
          （1）创建String类型的一个实例 var s1 = new String("stringtext");
          （2）在实例上调用指定的方法 var s2 = s1.substring(2);
          （3）摧毁这个实例 s1 = null;
          ```

        - 显式装箱

          ```js
          var obj = new Object("stringtext");
          console.log(obj instanceof String);
          //true
          ```

     2. 拆箱就是与装箱相反，把对象转变为基本类型的值。

        调用了 JavaScript 引擎内部的抽象操作，ToPrimitive(转换为原始值)，对原始值(null,undefined,number,string,boolen,symbol)不发生转换处理，只针对引用类型(object)

   - 数学运算

     a+b=a 的原数据类型+b 的原数据类型

     有 string 为 string，没 string 为 number，[].toString()->""，{}.toString()->"[object Object]"

     - [] + [] = "" + "" = ""
     - [] + {} = "" + "[object Object]" = "[object Object]"

      * 加法运算操作符
        * 两个操作值都是数值：
          * 如果一个操作数为NaN，则结果为NaN
          * 如果是Infinity+Infinity，结果是Infinity
          * 如果是-Infinity+(-Infinity)，结果是-Infinity
          * 如果是Infinity+(-Infinity)，结果是NaN
          * 如果是+0+(+0)，结果为+0
          * 如果是(-0)+(-0)，结果为-0
          * 如果是(+0)+(-0)，结果为+0
        * 如果有一个操作值为字符串类型，则将另一个操作值转换为字符串，最后连接起来。

      * 乘法运算符
        * 如果结果太大或太小，那么生成的结果是 Infinity 或 -Infinity。
        * 如果某个运算数是 NaN，结果为 NaN。
        * Infinity 乘以 0，结果为 NaN。
        * Infinity 乘以 0 以外的任何数字，结果为 Infinity 或 -Infinity。
        * Infinity 乘以 Infinity，结果为 Infinity。

      * 除法运算符
        * 如果结果太大或太小，那么生成的结果是 Infinity 或 -Infinity。
        * 如果某个运算数是 NaN，结果为 NaN。
        * Infinity 被 Infinity 除，结果为 NaN。
        * Infinity 被任何数字除，结果为 Infinity。
        * 0 除一个任何非无穷大的数字，结果为 NaN。
        * Infinity 被 0 以外的任何数字除，结果为 Infinity 或 -Infinity。

      * 取模运算符
        * 如果被除数是 Infinity，或除数是 0，结果为 NaN。
        * Infinity 被 Infinity 除，结果为 NaN。
        * 如果除数是无穷大的数，结果为被除数。
        * 如果被除数为 0，结果为 0。

     ```js
     console.log([] + [])->console.log("" + "")->""
     console.log({} + [])->console.log("[object Object]" + "")->"[object Object]"
     console.log([] == ![])->console.log(0 == !true)->console.log(0 == false)->true
     console.log(true + false)->console.log(1 + 0)->1
     ```

   - 逻辑操作符（!、&&、||）

      * 逻辑非（！）操作符首先通过Boolean()函数将它的操作值转换为布尔值，然后求反。
      * 逻辑与（&&）操作符，如果一个操作值不是布尔值时，遵循以下规则进行转换：
        * 如果第一个操作数经Boolean()转换后为true，则返回第二个操作值，否则返回第一个值（不是Boolean()转换后的值）
          * 如果一个运算数是对象，另一个是 Boolean 值，返回该对象。
          * 如果两个运算数都是对象，返回第二个对象
        * 如果有一个操作值为null，返回null
        * 如果有一个操作值为NaN，返回NaN
        * 如果有一个操作值为undefined，返回undefined
      * 逻辑或（||）操作符，如果一个操作值不是布尔值，遵循以下规则：
        * 如果第一个操作值经Boolean()转换后为false，则返回第二个操作值，否则返回第一个操作值（不是Boolean()转换后的值）
          * 如果一个运算数是对象，并且该对象左边的运算数值均为 false，则返回该对象。
          * 如果两个运算数都是对象，返回第一个对象。
        * 如果最后一个运算数是 null，并且其他运算数值均为 false，则返回 null。
        * 如果最后一个运算数是 NaN，并且其他运算数值均为 false，则返回 NaN。
        * 如果最后一个运算数是 undefined，并且其他运算数值均为 false，则返回 undefined。

   - 关系操作符（<, >, <=, >=）

      * 如果两个操作值都是数值，则进行数值比较
      * 如果两个操作值都是字符串，则比较字符串对应的字符编码值
      * 如果只有一个操作值是数值，则将另一个操作值转换为数值，进行数值比较
      * 如果一个操作数是对象，则调用valueOf()方法（如果对象没有valueOf()方法则调用toString()方法），得到的结果按照前面的规则执行比较
      * 如果一个操作值是布尔值，则将其转换为数值，再进行比较
      * NaN是非常特殊的值，它与任何类型的值比较大小时都返回false。

   - 比较运算

     - x===y,只有类型和值相等为 true,否则为 false
     - x == y
       - xy 都为 Null 或 undefined 为 true, null == undefined->true, null === undefined->false
       - x 或 y 为 NaN 为 false, NaN == NaN->false
       - 如果 x 和 y 为 String，Number，Boolean 并且类型不一致，都转为 Number 再进行比较
       - 如果存在 Object，转换为原始值，比较
       - !可将变量转换成 boolean 类型，null、undefined、NaN 以及空字符串('')取反都为 true，其余都为 false

     | value      | toNumber | toString          | toBoolean |
     | ---------- | -------- | ----------------- | --------- |
     | NaN        | NaN      | "NaN"             | false     |
     | Infinity   | Infinity | "Infinity"        | true      |
     | []         | 0        | ""                | true      |
     | [1]        | 1        | "1"               | true      |
     | null       | 0        | "null"            | false     |
     | undefined  | NaN      | "undefined"       | false     |
     | {}         | NaN      | "[object Object]" | true      |
     | function() | NaN      | "function"        | true      |

      * 如果一个运算数是 Boolean 值，在检查相等性之前，把它转换成数字值。false 转换成 0，true 为 1。
      * 如果一个运算数是字符串，另一个是数字，在检查相等性之前，要尝试把字符串转换成数字。
      * 如果一个运算数是对象，另一个是字符串，在检查相等性之前，要尝试把对象转换成字符串。
      * 如果一个运算数是对象，另一个是数字，在检查相等性之前，要尝试把对象转换成数字。
      * 值 null 和 undefined 相等。
      * 在检查相等性时，不能把 null 和 undefined 转换成其他值。
      * 如果某个运算数是 NaN，等号将返回 false，非等号将返回 true。
      * 如果两个运算数都是对象，那么比较的是它们的引用值。如果两个运算数指向同一对象，那么等号返回 true，否则两个运算数不等。

     ```txt
     []==false,[]==![],[]==0,''==0,""=="" true
     {}==false,{}==!{},{}==0,NaN==0 false
     ```

   - 隐式调用：对象生成时会自动调用(不同对象会有不同的隐式调用)

     - function:toString/valueOf
     - 事件:handleEvent
     - JSON 对象:toJSON
     - promise:then
     - object:get/set
     - 遍历器接口:Symbol.iterator

      ```js
      const a = {}
      const b = Symbol('1')
      const c = Symbol('1')
      a[b] = '子君'
      a[c] = '君子'

      // 输出子君:Symbol()函数会返回「symbol」类型的值，而Symbol函数传的参数仅仅是用于标识的，不会影响值的唯一性
      console.log(a[b])

      const d = {}
      const e = {key: '1'}
      const f = {key: '2'}
      d[e] = '子君'
      d[f] = '君子'

      // 输出君子:因为e和f都是对象，而对象的key只能是数值或字符，所以会将对象转换为字符，对象的toString方法返回的是[object Object], 所有输出的是君子
      console.log(d[e])
      ```

   - 每个对象的 toString 和 valueOf 方法都可以被改写，每个对象执行完毕，如果被用以操作 JavaScript 解析器就会自动调用对象的 toString 或者 valueOf 方法

   - 什么情况下会发生布尔值的隐式强制类型转换？

      1. if (..) 语句中的条件判断表达式。
      2. for ( .. ; .. ; .. ) 语句中的条件判断表达式（第二个）。
      3. while (..) 和 do..while(..) 循环中的条件判断表达式。
      4. ? : 中的条件判断表达式。
      5. 逻辑运算符 ||（逻辑或）和 &&（逻辑与）左边的操作数（作为条件判断表达式）。

### js 事件循环机制

1. 参考链接：

  [详解 JavaScript 中的 Event Loop（事件循环）机制](https://www.cnblogs.com/cangqinglang/p/8967268.html)

  [谈谈 Event Loop（事件循环）机制](https://www.jianshu.com/p/6e9f4eb7fdbb)

  [Javascript 异步编程之 setTimeout 与 setInterval 详解分析](https://www.cnblogs.com/tugenhua0707/p/4083475.html)

  [理解 JavaScript 执行机制及异步回调（setTimeout/setInterval/Promise）](https://blog.csdn.net/zuggs_/article/details/82381558)

  [js 异步执行顺序](https://www.jianshu.com/p/ca480f9e7dea)

  [为什么要用 setTimeout 模拟 setInterval?](https://blog.csdn.net/b954960630/article/details/82286486)

  [详解 setTimeout、setImmediate、process.nextTick 的区别](https://www.cnblogs.com/onepixel/articles/7605465.html)

  [setTimeout/setImmediate/process.nextTick 的区别](https://www.jianshu.com/p/77f03673aa06)

  [简单理解 Vue 中的 nextTick](https://www.jianshu.com/p/a7550c0e164f)

  [浅谈 async/await](https://www.jianshu.com/p/1e75bd387aa0)

  [async 函数的含义和用法](http://www.ruanyifeng.com/blog/2015/05/async.html)

  [面试向：Async/Await 代替 Promise.all()](https://juejin.im/post/5d56f89b518825415d0608be)

  [ECMAScript 6 入门](https://es6.ruanyifeng.com/#docs/async)

  [Async/Await 替代 Promise 的 6 个理由](https://www.cnblogs.com/fundebug/p/6667725.html)

  [js 基础之 setTimeout 与 setInterval 原理分析](https://blog.csdn.net/qq_41694291/article/details/93974595)

  [关于 setTimeout 和 setInterval 的实现原理](https://blog.csdn.net/sinat_30443713/article/details/78128088)

  [什么是事件循环？](https://cloud.tencent.com/developer/news/566935)

  [面试造火箭，看下这些大厂原题](https://juejin.im/post/6859121743869509646)

  [浏览器与Node的事件循环(Event Loop)有何区别?](https://zhuanlan.zhihu.com/p/54882306)

  [做一些动图，学习一下EventLoop](https://juejin.cn/post/6969028296893792286)

  [requestAnimationFrame && setTimeout 原理剖析](https://www.jianshu.com/p/1e782d5702ef)

  [Node.js 有难度的面试题，你能答对几个？](https://mp.weixin.qq.com/s/MLp1r3pDa91_EG07uZID-Q)

  [「2021」高频前端面试题汇总之浏览器原理篇](https://juejin.cn/post/6916157109906341902/)

  [使用 queueMicrotask 来执行微任务](https://blog.csdn.net/littlelyon/article/details/102927823)

2. 详解：

  - js 是单线程的非阻塞语言：因为如果是多线程，一边绑定事件，一边移除元素，会引起冲突。另外，如果引入锁，则大大增加复杂度，所以采取单线程。

  - 执行栈：方法排队执行的地方，每个单元对应一个 context，包含作用域中的 this。

  - 事件(消息)队列：异步事件返回结果后，js 会将这个事件加入与当前执行栈不同的另一个队列，被放入事件队列不会立刻执行其回调，而是等待当前执行栈中的所有任务都执行完毕，主线程处于闲置状态时，再把事件放回执行栈。

  - 事件循环：主线程不断的从消息队列中取消息，执行消息，这个过程称为事件循环，这种机制叫事件循环机制，取一次消息并执行的过程叫一次循环。

  - 事件循环机制：由执行栈和事件队列构成的无限循环

  - 事件循环描述：所有同步任务都在主线程上执行，形成一个执行栈，主线程之外，还存在一个”任务队列”，只要异步任务有了运行结果，就在”任务队列”之中放置一个事件。一旦”执行栈”中的所有同步任务执行完毕，系统就会读取”任务队列”，于是异步任务结束等待状态，进入执行栈，开始执行。主线程从”任务队列”中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为 Event Loop（事件循环）。

  - 宏任务：setInterval()，setTimeout()，setImmediate()，xhr回调

    - setTimeout：在指定的毫秒数后，将定时任务处理的函数添加到事件队列的队尾。
    - setInterval：按照指定的周期(以毫秒数计时)，将定时任务处理函数添加到事件队列的队尾。
      - 因为 js 是单线程的，如果处于堵塞状态计不了时，它必须依赖外部计时并触发定时，所以队列中的定时事件也是异步事件。
      - 每个 setTimeout 产生的任务会直接 push 到任务队列中；而 setInterval 在每次把任务 push 到任务队列前，都要进行一下判断(看上次的任务是否仍在队列中)。
      - setInterval 有两个缺点：
        - 某些间隔会被跳过；
        - 可能多个定时器会连续执行；
      - setTimeout 模拟 setInterval,解决 setInterval 缺点
      ```js
      //每次执行的时候都会创建一个新的定时器
      var a = setTimeout(function () {
        // 任务
        setTimeout(a, interval); //获取当前函数的引用，并且为其设置另一个定时器
      }, interval);
      //在前一个定时器执行完前，不会向队列插入新的定时器
      //保证定时器间隔
      ```

  - 微任务 1：new Promise()、fetch()

    - Promise 是异步的，是指他的 then()和 catch()方法，Promise 本身还是同步的
    - 有 resolve()后，才能执行 then(),有 reject()，不执行 then()
    - fetch基于promise

  - 微任务 2：async await

    - async function(){} 表示函数内存在异步操作
    - await 强制下面代码等待，直到这行代码得出结果(await setTimeout 无效，适用于 ajax)

  - 微任务 3：MutationObserver

  - 微任务 4：process.nextTick

  - 微任务 5：queueMicrotask

    浏览器微任务底层指派api，兼容性chrome71+

    ```js
    self.queueMicrotask(() => {
      // 函数的内容
    })
    ```

  - 操作系统任务、浏览器重绘任务：requestAnimationFrame

    官方解释：window.requestAnimationFrame()告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行。

    requestAnimationFrame它不需要你去手动设置执行间隔时间，它是跟随系统的屏幕刷新频率走的，如果屏幕刷新频率是60Hz，那么它的执行间隔就是16.7ms(1000/60≈16.7)，如果屏幕刷新频率是100Hz，那么它的执行间隔就是10ms(1000/100=10)，这样就能够保证它的执行与屏幕的刷新频率保持一致，从而避免丢帧现象。

    requestAnimationFrame的回调不会通过任务队列去排队，因为在浏览器离开本标签页之后，requestAnimationFrame回调函数直接停止运行。

  - 事件优先级：同步任务>异步任务(微任务>宏任务(取决于延时时间))

  - Node 事件循环的流程

    在进程启动时，Node 便会创建一个类似于 while(true)的循环，每执行一次循环体的过程我们称为 Tick。

    每个 Tick 的过程就是查看是否有事件待处理。如果有就取出事件及其相关的回调函数。然后进入下一个循环，如果不再有事件处理，就退出进程。

    每个事件循环中有一个或者多个观察者，而判断是否有事件需要处理的过程就是向这些观察者询问是否有要处理的事件。

    在 Node 中，事件主要来源于网络请求、文件的 I/O 等，这些事件对应的观察者有文件 I/O 观察者，网络 I/O 的观察者。

    事件循环是一个典型的生产者/消费者模型。异步 I/O，网络请求等则是事件的生产者，源源不断为 Node 提供不同类型的事件，这些事件被传递到对应的观察者那里，事件循环则从观察者那里取出事件并处理。

    在 windows 下，这个循环基于 IOCP 创建，在*nix 下则基于多线程创建

    异步 I/O 的流程:
    
      发起调用->封装请求对象->设置参数和回调函数->请求对象放入线程池等待执行->线程可用时执行请求对象中的I/O操作->执行结果放在请求对象中->通知IOCP调用完成->归还线程

      I/O观察者获取完成的结果->读取请求对象->执行回调函数->事件循环此过程

  - node定时器的特殊情况
  
    1. 执行顺序随机例子

      ```js
      setTimeout(() => {console.log('setTimeout')}, 0)
      setImmediate(() => {console.log('setImmediate')})
      ```

      对于以上代码来说，setTimeout 可能执行在前，也可能执行在后

        - 首先 setTimeout(fn, 0) === setTimeout(fn, 1)，这是由源码决定的
        - 进入事件循环也是需要成本的，如果在准备时候花费了大于 1ms 的时间，那么在 timer 阶段就会直接执行 setTimeout 回调
        - 那么如果准备时间花费小于 1ms，那么就是 setImmediate 回调先执行了

    2. 执行顺序一定是固定

      ```js
      const fs = require('fs')
      fs.readFile(__filename, () => {
          setTimeout(() => {
              console.log('timeout');
          }, 0)
          setImmediate(() => {
              console.log('immediate')
          })
      })
      ```

      setImmediate 永远先执行。因为两个代码写在 IO 回调中，IO 回调是在 poll 阶段执行，当回调执行完毕后队列为空，发现存在 setImmediate 回调，所以就直接跳转到 check 阶段去执行回调了。

  - Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境，比 js 少了 DOM/BOM,多了 http/file system,事件执行顺序与 js 不同

    - 浏览器是先把一个栈以及栈中的微任务走完，才会走下一个栈。node 环境里面是把所以栈走完，才走微任务
    - setTimeout setImmediate 都是宏任务
    - nextTick 和 then 都属于微任务
    - i/o 文件操作为宏任务

  - 浏览器和node的事件循环区别

    ```js
    setTimeout(()=>{
        console.log('timer1')
        Promise.resolve().then(function() {
            console.log('promise1')
        })
    }, 0)
    setTimeout(()=>{
        console.log('timer2')
        Promise.resolve().then(function() {
            console.log('promise2')
        })
    }, 0)
    //浏览器:timer1=>promise1=>timer2=>promise2
    //node:timer1=>timer2=>promise1=>promise2
    ```

    node会在每个阶段之间执行微任务，6个阶段:

    1. timers 阶段：这个阶段执行 timer（setTimeout、setInterval）的回调

      初次进入事件循环，会从计时器阶段开始。此阶段会判断是否存在过期的计时器回调（包含 setTimeout 和 setInterval），如果存在则会执行所有过期的计时器回调，执行完毕后，如果回调中触发了相应的微任务，会接着执行所有微任务，执行完微任务后再进入 Pending callbacks 阶段。

    2. I/O callbacks 阶段：处理一些上一轮循环中的少数未执行的 I/O 回调
    3. idle, prepare 阶段：仅 node 内部使用
    4. poll 阶段：获取新的 I/O 事件, 适当的条件下 node 将阻塞在这里

      * 当回调队列不为空时：会执行回调，若回调中触发了相应的微任务，这里的微任务执行时机和其他地方有所不同，不会等到所有回调执行完毕后才执行，而是针对每一个回调执行完毕后，就执行相应微任务。执行完所有的回调后，变为下面的情况。
      * 当回调队列为空时（没有回调或所有回调执行完毕）：但如果存在有计时器（setTimeout、setInterval和setImmediate）没有执行，会结束轮询阶段，进入 Check 阶段。否则会阻塞并等待任何正在执行的I/O操作完成，并马上执行相应的回调，直到所有回调执行完毕。

    5. check 阶段：执行 setImmediate() 的回调

      会检查是否存在 setImmediate 相关的回调，如果存在则执行所有回调，执行完毕后，如果回调中触发了相应的微任务，会接着执行所有微任务，执行完微任务后再进入 Close callbacks 阶段。

    6. close callbacks 阶段：执行 socket 的 close 事件回调

    process.nextTick是独立于 Event Loop 之外的，它有一个自己的队列，当每个阶段完成后，如果存在 nextTick 队列，就会清空队列中的所有回调函数，并且优先于其他 microtask 执行

    永远都是先把 nextTick 全部打印出来
    ```js
    setTimeout(() => {
    console.log('timer1')
    Promise.resolve().then(function() {
      console.log('promise1')
    })
    }, 0)
    process.nextTick(() => {
    console.log('nextTick')
    process.nextTick(() => {
      console.log('nextTick')
      process.nextTick(() => {
        console.log('nextTick')
        process.nextTick(() => {
          console.log('nextTick')
        })
      })
    })
    })
    ```

  - nextTick、setTimeout、setImmediate 的区别

    nodejs 中，setTimeout、setImmediate 是宏任务，process.nextTick()是微任务，因此 setTimeout、setImmediate 回调函数插入到任务队列的尾部，nextTick 回调函数加入到当前执行栈的尾部，所以 nextTick 会先执行。

    setTimeout、setImmediate 相差不大，但延时设为 0 时，setImmediate 会更快加入任务队列。

    vue 中，created()钩子函数进行的 DOM 操作一定要放在 Vue.nextTick()的回调函数中，因为此时 DOM 没渲染，需要等到 mounted()后执行。另外，数据变化导致 DOM 变化，也应用 Vue.nextTick()

  - setInterval，setTimeout 注意的地方

    1. function 中代码执行时间超过指定等待时间，settimeout 会在执行完后立即输出，setinterval 不会再这段时间向任务队列添加回调函数(因为已经存在上一个回调函数没执行完)，但不会影响后续计时。

    ```js
    var start = new Date();
    setTimeout(function () {
      var end = new Date();
      console.log("Time elapsed: ", end - start, "ms");
    }, 500);

    while (new Date() - start <= 1000) {}
    //Time elapsed:  1018 ms
    ```

    2. 如果 setinterval 执行时间略小于等待时间，则可能会出现连续执行的情况，所以需要用 settimeout 模拟 setinterval

    ```js
    function func(args){
        //函数本身的逻辑
        ...
    }
    var timer = setInterval(func, 100, args);

    var timer;
    function func(args){
        //函数本身的逻辑
        ...
        //函数执行完后，重置定时器
        timer = setTimeout(func, 100, args);
    }
    timer = setTimeout(func, 100, args);
    ```

    3. 延时设为 0，也不会马上执行，因为浏览器有各自最低等待时间

    ```js
    function a() {
      setTimeout(function () {
        console.log(1);
      }, 0);
      console.log(2);
    }
    a();
    // 2 1
    ```

  - 例题：

    ```js
    //浏览器
    (function () {
      setTimeout(() => {
        console.log(0);
      });

      new Promise((resolve) => {
        console.log(1);

        setTimeout(() => {
          resolve();
          Promise.resolve().then(() => {
            console.log(2);
            setTimeout(() => console.log(3));
            Promise.resolve().then(() => console.log(4));
          });
        });

        Promise.resolve().then(() => console.log(5));
      }).then(() => {
        console.log(6);
        Promise.resolve().then(() => console.log(7));
        setTimeout(() => console.log(8));
      });

      console.log(9);
    })();
    //1、9、5、0、6、2、7、4、8、3
    //node.js
    console.log("1");
    setTimeout(function () {
      console.log("2");
      process.nextTick(function () {
        console.log("3");
      });
      new Promise(function (resolve) {
        console.log("4");
        resolve();
      }).then(function () {
        console.log("5");
      });
    });
    process.nextTick(function () {
      console.log("6");
    });
    new Promise(function (resolve) {
      console.log("7");
      resolve();
    }).then(function () {
      console.log("8");
    });
    setTimeout(function () {
      console.log("9");
      process.nextTick(function () {
        console.log("10");
      });
      new Promise(function (resolve) {
        console.log("11");
        resolve();
      }).then(function () {
        console.log("12");
      });
    });
    //1，7，6，8，2，4，3，5，9，11，10，12
    let test = async function () {
      await new Promise((resolve, reject) => {
        console.log(1);
        setTimeout(() => {
          resolve();
        }, 3000);
      }).then(() => {
        console.log(2);
      });
      console.log(3);
      await new Promise((resolve, reject) => {
        console.log(4);
        setTimeout(() => {
          resolve();
        }, 1000);
      }).then(() => {
        console.log(5);
      });
      console.log(6);
    };
    test();
    //1

    //2
    //3
    //4

    //5
    //6
    ```

    如下为一段代码，请完善sum函数，使得 sum(1,2,3,4,5,6) 函数返回值为 21 ,需要在 sum 函数中调用 asyncAdd 函数，且不能修改asyncAdd函数

    ```js
    /**
    * 请在 sum函数中调用此函数，完成数值计算
    * @param {*} a 要相加的第一个值
    * @param {*} b 要相加的第二个值
    * @param {*} callback 相加之后的回调函数
    */
    function asyncAdd(a,b,callback) {
      setTimeout(function(){
        callback(null, a+b)
      },100)
    }

    /**
    * 请在此方法中调用asyncAdd方法，完成数值计算
    * @param  {...any} rest 传入的参数
    */
    async function sum(...rest) {
      // 请在此处完善代码
    }

    let start = window.performance.now()
    sum(1, 2, 3, 4, 5, 6).then(res => {
      // 请保证在调用sum方法之后，返回结果21
      console.log(res)
      console.log(`程序执行共耗时: ${window.performance.now() - start}`)
    })
    ```

    普通方法
    ```js
    async function sum(...rest) {
      // 取出来第一个作为初始值
      let result = rest.shift()
      // 通过for of 遍历 rest, 依次相加
      for(let num of rest) {
        // 使用promise 获取相加结果
        result = await new Promise(resolve => {
          asyncAdd(result, num, (_,res) => {
            resolve(res)
          })
        })
      }
      // 返回执行结果
      return result
    }
    ```

    优化：两两执行
    ```js
    async function sum(...rest) {
      // 如果传的值少于2个，则直接返回
      if (rest.length <= 1) {
        return rest[0] || 0
      }
      const promises = []
      // 遍历将数组里面的值两个两个的执行
      for (let i = 0; i < rest.length; i += 2) {
        promises.push(
          new Promise(resolve => {
            // 如果 rest[i+1] 是 undefined, 说明数组长度是奇数，这个是最后一个
            if (rest[i + 1] === undefined) {
              resolve(rest[i])
            } else {
              // 调用asyncAdd 进行计算
              asyncAdd(rest[i], rest[i + 1], (_, result) => {
                resolve(result)
              })
            }
          })
        )
      }
      // 获取第一次计算结果
      const result = await Promise.all(promises)
      // 然后将第一次获取到的结果即 [3,7,11] 再次调用 sum执行
      return await sum(...result)
    }
    ```
    ```js
    async function sum(...rest) {
      let arr = rest;
      let r = 0;
      while(arr.length > 0){
        let a = arr.pop();
        let b = 0;
        if(arr.length > 0){
          b = arr.pop();
        }
        let result = await new Promise((resolve,reject)=>{
          asyncAdd(a,b,(tmp,res)=>{
            resolve(res);
          });
        });
        r += result;
      }
      return r;
    }
    ```

    再优化:隐式转换迭代+promise.all
    ```js
    async function sum(...rest) {
      let result = 0
      // 隐式类型转换， 对象 + 数字，会先调用对象的toString 方法
      const obj = {}
      obj.toString = function() {
        return result
      }
      const promises = []
      for(let num of rest) {
        promises.push(new Promise((resolve) => {
          asyncAdd(obj, num, (_, res) => {
            resolve(res)
          })
        }).then(res => {
          // 在这里将 result的值改变之后，obj.toString 的返回值就变了，这时候下一个setTimeout调用时就使用了新值
          result = res
        }))
      }
      await Promise.all(promises)
      return result
    }
    ```

### 闭包数据缓存与垃圾回收

1. 参考链接：

  [JS 闭包异步获取数据并缓存](https://blog.csdn.net/weixin_43820866/article/details/87107035)

  [js async await 终极异步解决方案](https://www.cnblogs.com/CandyManPing/p/9384104.html)

  [闭包+内存泄露+垃圾回收](https://blog.csdn.net/yushuangyushuang/article/details/79301694)

  [从 4 个面试题了解「浏览器的垃圾回收」](https://mp.weixin.qq.com/s/vbG24Ogc5qMpTz-_Hz61KQ)

  [js的内存泄漏场景、监控以及分析](https://www.cnblogs.com/dasusu/p/12200176.html)

  [「2021」高频前端面试题汇总之JavaScript篇（下）](https://juejin.cn/post/6941194115392634888)

2. 详解：

  - 闭包的形成原理

    活动对象被引用着无法被销毁而导致的

  - 闭包 2 个作用

    1. 内层函数可通过作用域链访问外层函数变量
    2. 缓存机制(垃圾回收机制)：函数执行完会释放内存，但如果内层函数被引用，则不会释放内存，所以可能会造成内存泄露

      - 注意：闭包指向同一处内存才能共享数据

      ```js
      function F() {
        let i = 0;
        return function () {
          console.log(i++);
        };
      }

      let f1 = new F();
      let f2 = new F();
      f1(); //0
      f1(); //1
      f2(); //0
      ```

  - 优点

    1. 避免全局变量被污染
    2. 方便调用上下文的局部变量
    3. 加强封装性

  - 缺点

    1. 闭包常驻内存，内存消耗很大
    2. 可能导致内存泄露

      例子

      ```js
      window.onload = function () {
        var el = document.getElementById("id");
        el.onclick = function () {
          alert(el.id);
        };
      };
      ```

      解决方法

      ```js
      window.onload = function () {
        var el = document.getElementById("id");
        var id = el.id; //解除循环引用
        el.onclick = function () {
          alert(id);
        };
        el = null; // 将闭包引用的外部函数中活动对象清除
      };
      ```

  - 例子

  向接口请求数据时，数据多次使用，但不想保存在全局变量中，就需要将数据存储在缓存中。查找数据时，如果缓存找不到，则调用 API，然后设置缓存，如果找到，直接返回查找到的值即可。闭包正好可以做到这一点，且不会释放外部的引用，从而函数内部的值可以得以保留。

  ```js
  const getList = (function() {
      // 闭包存储data
      let data = {};
      const getData = () => {
          return new Promise((resolve, reject) => {
              $.ajax({
                  url: '/your/api',
                  data: {
                      normal: 1
                  },
                  success: function (result) {
                      data = result.data;
                      resolve();
                  }
              });
          })
      }
      // 异步函数，当调用一个 async 函数时，会返回一个 Promise 对象。
      const result = async function (type) {
          if (JONS.stringify(data) === '{}') {
              //await 只能出现在 async 函数中。
              await getData();//等待异步操作执行完成，再执行后面的操作，相当于把后面的代码写在success里，但用await会比较简洁
              //如果 async 函数没有返回值，它会返回 Promise.resolve(undefined)。如果有返回值data，就会resolve(data),把data传入then
              //当 async 函数抛出异常时，Promise 的 reject 方法也会传递这个异常值。
              return data;
          } else {
              return data;
          }
      }

      return result;
  })();

  // 第一次调用通过api请求数据
  getList().then(res => {
      console.log(res);

      // 第二次调用则直接拿取缓存数据
      getList().then(res => {
          console.log(res);
      }
  });
  ```

  - 浏览器

    * 什么是垃圾数据？

      不再被引用的数据

    * 垃圾回收算法

      可以将这个过程想象成从根溢出一个巨大的油漆桶，它从一个根节点出发将可到达的对象标记染色， 然后移除未标记的。

      1. 标记空间中「可达」值。

        从根节点（Root）出发，遍历所有的对象,可以遍历到的对象，是可达的（reachable）,没有被遍历到的对象，不可达的（unreachable）

        在浏览器环境下，根节点有很多，主要包括这几种:全局变量 window，位于每个 iframe 中,文档 DOM 树,存放在栈上的变量

      2. 回收「不可达」的值所占据的内存。

        在所有的标记完成之后，统一清理内存中所有不可达的对象

      3. 做内存整理

        在频繁回收对象后，内存中就会存在大量不连续空间，专业名词叫「内存碎片」。

        当内存中出现了大量的内存碎片，如果需要分配较大的连续内存时，就有可能出现内存不足的情况。

        所以最后一步是整理内存碎片。(但这步其实是可选的，因为有的垃圾回收器不会产生内存碎片，比如接下来我们要介绍的副垃圾回收器。)

    * 什么时候垃圾回收？

      浏览器进行垃圾回收的时候，会暂停 JavaScript 脚本，等垃圾回收完毕再继续执行。

      对于普通应用这样没什么问题，但对于 JS 游戏、动画对连贯性要求比较高的应用，如果暂停时间很长就会造成页面卡顿。

      什么时候进行垃圾回收，可以避免长时间暂停?

      * 分代收集

        将堆分为新生代与老生代，多回收新生代，少回收老生代。

        这样就减少了每次需遍历的对象，从而减少每次垃圾回收的耗时。

        浏览器将数据分为两种，一种是「临时」对象，一种是「长久」对象。

        1. 临时对象：

          大部分对象在内存中存活的时间很短。

          比如函数内部声明的变量，或者块级作用域中的变量。当函数或者代码块执行结束时，作用域中定义的变量就会被销毁。

          这类对象很快就变得不可访问，应该快点回收。

        2. 长久对象：

          生命周期很长的对象，比如全局的 window、DOM、Web API 等等。

          这类对象可以慢点回收。
          
          这两种对象对应不同的回收策略，所以，V8 把堆分为新生代和老生代两个区域， 新生代中存放临时对象，老生代中存放持久对象。

          并且让副垃圾回收器、主垃圾回收器，分别负责新生代、老生代的垃圾回收。

        * 主垃圾回收器

          负责老生代的垃圾回收，有两个特点：

          1. 对象占用空间大。
          2. 对象存活时间长。

          它使用「标记-清除」的算法执行垃圾回收。

          * 首先是标记。

            从一组根元素开始，递归遍历这组根元素。

            在这个遍历过程中，能到达的元素称为活动对象，没有到达的元素就可以判断为垃圾数据。

          * 然后是垃圾清除。

            多次标记-清除后，会产生大量不连续的内存碎片，需要进行内存整理。

        * 副垃圾回收器

          负责新生代的垃圾回收，通常只支持 1~8 M 的容量。

          新生代被分为两个区域：一般是对象区域，一半是空闲区域。

          新加入的对象都被放入对象区域，等对象区域快满的时候，会执行一次垃圾清理。

          1. 先给对象区域所有垃圾做标记。
          2. 标记完成后，存活的对象被复制到空闲区域，并且将他们有序的排列一遍。

            这就回到我们前面留下的问题 -- 副垃圾回收器没有碎片整理。因为空闲区域里此时是有序的，没有碎片，也就不需要整理了。

          3. 复制完成后，对象区域会和空闲区域进行对调。将空闲区域中存活的对象放入对象区域里。

          这样，就完成了垃圾回收。

          因为副垃圾回收器操作比较频繁，所以为了执行效率，一般新生区的空间会被设置得比较小。

          一旦检测到空间装满了，就执行垃圾回收。

      * 增量收集

        如果脚本中有许多对象，引擎一次性遍历整个对象，会造成一个长时间暂停。

        所以引擎将垃圾收集工作分成更小的块，每次处理一部分，多次处理。

        这样就解决了长时间停顿的问题。

      * 闲时收集

        垃圾收集器只会在 CPU 空闲时尝试运行，以减少可能对代码执行的影响。

    * 如何减少垃圾回收？

      虽然浏览器可以进行垃圾自动回收，但是当代码比较复杂时，垃圾回收所带来的代价比较大，所以应该尽量减少垃圾回收。

      * 对数组进行优化： 在清空一个数组时，最简单的方法就是给其赋值为[ ]，但是与此同时会创建一个新的空对象，可以将数组的长度设置为0，以此来达到清空数组的目的。
      * 对object进行优化： 对象尽量复用，对于不再使用的对象，就将其设置为null，尽快被回收。
      * 对函数进行优化： 在循环中的函数表达式，如果可以复用，尽量放在函数的外面。

    * 哪些情况会导致内存泄露？

      * 意外的全局变量： 由于使用未声明的变量，而意外的创建了一个全局变量，而使这个变量一直留在内存中无法被回收。
      * 被遗忘的计时器或回调函数： 设置了 setInterval 定时器，而忘记取消它，如果循环函数有对外部变量的引用的话，那么这个变量会被一直留在内存中，而无法被回* 收。
      * 脱离 DOM 的引用： 获取一个 DOM 元素的引用，而后面这个元素被删除，由于一直保留了对这个元素的引用，所以它也无法被回收。
      * 闭包： 不合理的使用闭包，从而导致某些变量一直被留在内存当中。

      以 Vue 为例，通常有这些情况：

        * 监听在 window/body 等事件没有解绑
        * 绑在 EventBus 的事件没有解绑
        * Vuex 的 $store，watch 了之后没有 unwatch
        * 使用第三方库创建，没有调用正确的销毁函数

      解决办法：beforeDestroy 中及时销毁

        * 绑定了 DOM/BOM 对象中的事件 addEventListener ，removeEventListener。
        * 观察者模式 $on，$off处理。
        * 如果组件中使用了定时器，应销毁处理。
        * 如果在 mounted/created 钩子中使用了第三方库初始化，对应的销毁。
        * 使用弱引用 weakMap、weakSet。

  - 内存泄露监控

    通过chrome控制台-performance-memory监控指定时间内存，观察图形/jsHeap等指标

    1. 场景一：在某个函数内申请一块内存，然后该函数在短时间内不断被调用

      函数执行时，发现内存不足，垃圾回收机制工作，回收上一个函数申请的内存，因为上个函数已经执行结束了，内存无用可被回收了

      所以图中呈现内存使用量的图表就是一条横线过去，中间出现多处竖线，其实就是表示内存清空，再申请，清空再申请

    2. 场景二：在某个函数内申请一块内存，然后该函数在短时间内不断被调用，但每次申请的内存，有一部分被外部持有

      此时不再是一条横线了，而且横线中的每个竖线的底部也不是同一水平，可以判断出是内存泄露了

      每次函数执行完，这部分被外部持有的数组内存也依旧回收不了，所以每次只能回收一部分内存
      
      梯状上升的就是发生内存泄漏了，每次函数调用，总有一部分数据被外部持有导致无法回收

  - 如何分析内存泄漏，找出有问题的代码

    可以抓取两份快照，两份快照中间进行内存泄漏操作，最后再比对两份快照的区别，查看增加的对象是什么，回收的对象又是哪些

    还可以从垃圾回收机制角度出发，查看从 GC root 根节点出发，可达的对象里，哪些对象占用大量内存

    当有嫌疑对象时，可以利用多次内存快照间比对，中间手动强制 GC 下，看下该回收的对象有没有被回收

### DOM 树解析和更改与遍历

1. 参考链接：

   [Window](https://developer.mozilla.org/zh-CN/docs/Web/API/Window)

2. 详解：

   - XML/HTML 源代码解析为 DOM Document：DOMParser.parseFromString，相反操作：XMLSerializer.serializeToString

   ```js
   let parser = new DOMParser(),
     doc = parser.parseFromString(
       XML / HTML源代码(可url),
       指定类型字符串(
         "application/xml",
         "image/svg+xml",
         "text/html",
         "application/xml"
       )
     );

   var s = new XMLSerializer();
   var d = document;
   var str = s.serializeToString(d);
   ```

   - 监视对 DOM 树所做更改

   ```js
   let targetNode = document.querySelector(`#id`);

   // 配置
   let config = {
     attributeFilter: [特定属性名称], //要监视的特定属性名称的数组
     attributeOldValue: true, //记录任何有改动的属性的上一个值,无默认值
     attributes: true, //监视元素属性值变更,默认false
     characterData: true, //监视目标节点子树节点所包含的字符数据的变化,无默认值
     characterDataOldValue: true, //记录节点文本的先前值.无默认值
     childList: true, //监视目标节点添加或删除新的子节点,默认false
     subtree: true, //监视目标节点子树添加或删除新的子节点,默认false
   };

   // 更改被监测到时执行
   const mutationCallback = (mutationsList) => {
     for (let mutation of mutationsList) {
       let type = mutation.type;
       switch (type) {
         case "childList":
           console.log("A child node has been added or removed.");
           break;
         case "attributes":
           console.log(`The ${mutation.attributeName} attribute was modified.`);
           break;
         case "subtree":
           console.log(`The subtree was modified.`);
           break;
         default:
           break;
       }
     }
   };

   let observer = new MutationObserver(mutationCallback);

   //开始监测
   observer.observe(targetNode, config);

   //停止监测
   observer.disconnect();
   ```

   - 遍历文档的子树中的所有节点及其位置 document.createTreeWalker()

   ```js
   treeWalker = document.createTreeWalker(根节点, 过滤某些内容节点[option], NodeFilter 对象, 标识符(已废弃));
   ```

   ```js
   var treeWalker = document.createTreeWalker(
     document.body,
     NodeFilter.SHOW_ELEMENT,
     {
       acceptNode: function (node) {
         return NodeFilter.FILTER_ACCEPT;
       },
     },
     false
   );

   var nodeList = [];

   while (treeWalker.nextNode()) nodeList.push(treeWalker.currentNode);
   ```

   1. option
      - NodeFilter.SHOW_ALL -1 显示所有节点
      - NodeFilter.SHOW_ATTRIBUTE 2 显示特性 Attr 节点 废弃
      - NodeFilter.SHOW_CDATA_SECTION 8 显示 CDTA CDATASection 节点 废弃
      - NodeFilter.SHOW_COMMENT 128 显示注释 Comment 节点
      - NodeFilter.SHOW_DOCUMENT 256 显示文档 Document 节点
      - NodeFilter.SHOW_DOCUMENT_FRAGMENT 1024 显示文档片段 DocumentFragment 节点
      - NodeFilter.SHOW_DOCUMENT_TYPE 512 显示文档类型 DocumentType 节点
      - NodeFilter.SHOW_ELEMENT 1 显示元素 Element 节点
      - NodeFilter.SHOW_ENTITY 32 显示实体 Entity 节点 废弃
      - NodeFilter.SHOW_ENTITY_REFERENCE 16 显示实体引用 废弃
      - NodeFilter.SHOW_NOTATION 2048 显示符号 Notation 节点 废弃
      - NodeFilter.SHOW_PROCESSING_INSTRUCTION 64 显示处理指令 ProcessingInstruction 节点
      - NodeFilter.SHOW_TEXT 4 显示文字 Text nodes 节点
   2. NodeFilter

   ```js
   var nodeIterator = document.createNodeIterator(
     document.getElementById("someId"),

     NodeFilter.SHOW_TEXT,

     {
       acceptNode: function (node) {
         if (!/^\s*$/.test(node.data)) {
           return NodeFilter.FILTER_ACCEPT;
           //return NodeFilter.FILTER_REJECT;
           //return NodeFilter.FILTER_SKIP;
         }
       },
     },
     false
   );

   var node;

   while ((node = nodeIterator.nextNode())) {
     alert(node.data);
   }
   ```


### 运算符优先级

1. 参考链接：

   - [('b' + 'a' + + 'a' + 'a').toLowerCase()输出 banana 的剖析](https://juejin.im/post/5d537c71e51d4561c94b0faa)

2. 详解：

   ```txt
   ('b' + 'a' + + 'a' + 'a').toLowerCase()
   =('b' + 'a' + (+ 'a') + 'a').toLowerCase()//正号优先级大于加号
   =('b' + 'a' + Number('a') + 'a').toLowerCase()//隐式转换
   =('ba' + NaN + 'a').toLowerCase()
   =('ba' + 'NaN' + 'a').toLowerCase()
   =('baNaNa').toLowerCase()
   ='banana'
   ```

### 作用域与变量等题目

1. 参考链接

   - [8 个问题看你是否真的懂 JS](https://juejin.im/post/5d2d146bf265da1b9163c5c9)
   - [七个简单但棘手的 JS 面试问题](https://segmentfault.com/a/1190000020722239)
   - [前端常见 20 道高频面试题深入解析](https://mp.weixin.qq.com/s/jx-4p32EA9cHkDzll3BoYQ)
   - [2 万字 | 前端基础拾遗 90 问](https://juejin.im/post/5e8b261ae51d4546c0382ab4#heading-37)
   - [JS 中的最大安全整数是多少？](https://blog.csdn.net/weixin_43675244/article/details/89518309)
   - [BigInt](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/BigInt)
   - [重学前端学习笔记（二十）--try 里面放 return，finally 还会执行吗？](https://www.ucloud.cn/yun/109647.html)
   - [实现 Lazyman](https://segmentfault.com/a/1190000009018654?utm_source=tag-newest)
   - [触及盲区，这道题前端群里发疯了](https://mp.weixin.qq.com/s/YQEBZo1pdy-5B1Jz8cvWmw)
   - [8 个原生 JS 知识点 | 面试高频](https://mp.weixin.qq.com/s/tIasEjYJRaVqFMN_aVtpiw)
   - [11 个 JavaScript 小技巧](https://mp.weixin.qq.com/s/qBuTTXzt7ZNFttXwu5ryMw)
   - [当裸辞遇到了面试难，你需要了解一下这些面试题](https://juejin.im/post/6866920515420815374)
   - [js解决0.1+0.2==0.3的问题的几种方法](https://blog.csdn.net/weixin_34018202/article/details/88596223)

2. 详解

   - 意外的全局变量

     ```js
     function foo() {
       let a = (b = 0); //声明了全局变量b=0与局部变量a
       a++;
       return a;
     }

     foo();
     typeof a; // undefined
     typeof b; // number
     ```

   - 数组长度属性

      ```js
      const clothes = ["jacket", "t-shirt"];
      clothes.length = 0; //改变数组长度相当于删除项
      clothes[0]; // undefined
      clothes.length = 2; //改回来，数据也丢失了
      clothes[0]; // undefined
      ```

      push方法会将数组的length + 1, 然后将值放在索引为length - 1的位置，该方法和 call() 或 apply() 一起使用时，可应用在类似数组的对象上。
      ```js
      const arr = new Array(2)
      // 输出  2 [empty * 2]
      console.log(arr.length, arr)
      arr.push(1)
      // 输出  3 [empty * 2, 1]
      console.log(arr.length, arr)
      ```
      nodejs输出
      ```js
      var obj = {
          '2': 3,
          '3': 4,
          'length': 2,
          'splice': Array.prototype.splice,
          'push': Array.prototype.push
      }
      obj.push(1)
      obj.push(2)
      console.log(obj)

      { '2': 1,
        '3': 2,
        length: 4,
        splice: [Function: splice],
        push: [Function: push] }
      ```

      chrome控制台是如何判断打印的内容是数组还是其他对象呢？chrome就是通过判断对象上面是否有splice和length这两个属性来判断的
      ```js
      [empty × 2, 1, 2, splice: ƒ, push: ƒ]
      ```

      将splice去掉之后，就会输出以下内容
      ```js
      {2: 1, 3: 2, length: 4, push: ƒ}
      ```

   - 自动插入分号

     ```js
     function arrayFromValue(item) {
       return; //js不写分号也能自成一行
       [items];
     }

     arrayFromValue(10); //undefined
     ```

   - 事件循环机制

     ```js
     let i;
     for (i = 0; i < 3; i++) {
       //先执行完循环，再调用时钟宏任务
       const log = () => {
         console.log(i);
       };
       setTimeout(log, 100);
     }
     //3 3 3
     ```

   - 实现 let 和 const

     let

     ```js
     for (var i = 0; i < 10; i++) {
       (function (i) {
         //闭包缓存
         setTimeout(() => {
           console.log(i);
         }, 0);
       })(i);
     }
     ```

     const

     ```js
     function _const(key, value) {
       const desc = {
         value,
         writable: false,
       };
       Object.defineProperty(window, key, desc);
     }

     _const("obj", { a: 1 }); //定义obj
     obj.b = 2; //可以正常给obj的属性赋值
     obj = {}; //抛出错误，提示对象read-only
     ```

   - 精度丢失

     二进制导致小数大多只能取近似值

     双精度存储（double precision），占用 64 bit

     1. 1 位用来表示符号位
     2. 11 位用来表示指数
     3. 52 位表示尾数(整数+小数)

     ```js
     0.1 + 0.2 === 0.3//false
     (0.1 * 10 + 0.2 * 10) / 10) == 0.3; // true
     
     function numbersequal(a,b){ return Math.abs(a-b)<Number.EPSILON; } 
     Number.EPSILON=(function(){   //解决兼容性问题
       return Number.EPSILON?Number.EPSILON:Math.pow(2,-52);
     })();
     var a=0.1+0.2， b=0.3;
     console.log(numbersequal(a,b)); //true
     ```

     bigint 类型:小数部分会取整

     ```js
     const theBiggestInt = 9007199254740991n;

     const alsoHuge = BigInt(9007199254740991);
     // ↪ 9007199254740991n

     const hugeString = BigInt("9007199254740991");
     // ↪ 9007199254740991n

     const hugeHex = BigInt("0x1fffffffffffff");
     // ↪ 9007199254740991n

     const hugeBin = BigInt(
       "0b11111111111111111111111111111111111111111111111111111"
     );
     // ↪ 9007199254740991n
     ```

   - try catch finally 执行

     try 里面放 return，finally 会执行完再 return

     ```js
     // return 执行了但是没有立即返回，而是先执行了finally
     function kaimo() {
       try {
         return 0;
       } catch (err) {
         console.log(err);
       } finally {
         console.log("a");
       }
     }

     console.log(kaimo()); // a 0
     // finally 中的 return 覆盖了 try 中的 return。
     function kaimo() {
       try {
         return 0;
       } catch (err) {
         console.log(err);
       } finally {
         return 1;
       }
     }

     console.log(kaimo()); // 1
     ```

   - js 变量生命周期

     ```js
     a; // undefined 变量提升
     b; // ReferenceError 临时死区 const
     c; // ReferenceError 临时死区 let

     var a = "value";
     const b = 3.14;
     let c = true;

     a; //'value'
     b; //3.14
     c; //true
     ```

     ```js
     var a = 10;
     function foo() {
       console.log(a); // undefined
       var a = 20; // 使上面的a变成局部作用域
     }
     foo();
     ```

     ```js
     var a = 10;
     function foo() {
       console.log(a); // ReferenceError
       let a = 20; // 使上面的a变成局部作用域
     }
     foo();
     ```

     ```js
     var a = 10;
     function foo() {
       console.log(a); // 全局作用域10
     }
     foo();
     ```

     ```js
     var a = 10;
     function foo() {
       console.log(a); // undefined
       var a = 20; // 使上面的a变成局部作用域
       console.log(a); //20
     }
     console.log(a); //10
     foo();
     ```

   - this 的指向

     ```js
     var x = 10; // global scope
     var foo = {
       x: 90,
       getX: function () {
         return this.x;
       },
     };
     foo.getX(); // 90,函数里的this指向foo
     let xGetter = foo.getX;
     xGetter(); // 10,函数里的this指向window
     let getFooX = foo.getX.bind(foo); //使用call和apply同理
     getFooX(); // 90
     ```

     ```js
      var num = 1;
      let obj = {
          num: 2,
          add: function() {
              this.num = 3;
              // 这里的立即指向函数，因为我们没有手动去指定它的this指向，所以都会指向window
              (function() {
                  // 所有这个 this.num 就等于 window.num
                  console.log(this.num);
                  this.num = 4;
              })();
              console.log(this.num);
          },
          sub: function() {
              console.log(this.num)
          }
      }
      // 下面逐行说明打印的内容

      /**
      * 在通过obj.add 调用add 函数时，函数的this指向的是obj,这时候第一个this.num=3
      * 相当于 obj.num = 3 但是里面的立即指向函数this依然是window,
      * 所以 立即执行函数里面console.log(this.num)输出1，同时 window.num = 4
      *立即执行函数之后，再输出`this.num`,这时候`this`是`obj`,所以输出3
      */
      obj.add() // 输出 1 3

      // 通过上面`obj.add`的执行，obj.name 已经变成了3
      console.log(obj.num) // 输出3
      // 这个num是 window.num
      console.log(num) // 输出4
      // 如果将obj.sub 赋值给一个新的变量，那么这个函数的作用域将会变成新变量的作用域
      const sub = obj.sub
      // 作用域变成了window window.num 是 4
      sub() // 输出4
     ```

   - 执行上下文

     代码被解析和执行时所在环境的抽象概念，类型分为全局和函数，创建过程：

     - 创建变量对象：首先初始化函数的参数 arguments，提升函数声明和变量声明。

     - 创建作用域链（Scope Chain）：在执行期上下文的创建阶段，作用域链是在变量对象之后创建的。

     - 确定 this 的值，即 ResolveThisBinding

   - 执行栈

     具有 LIFO (后进先出) 结构，用于存储在代码执行期间创建的所有执行上下文。

     执行规则：

     - 首次运行 JavaScript 代码的时候,会创建一个全局执行的上下文并 Push 到当前的执行栈中，每当发生函数调用，引擎都会为该函数创建一个新的函数执行上下文并 Push 当前执行栈的栈顶。

     - 当栈顶的函数运行完成后，其对应的函数执行上下文将会从执行栈中 Pop 出，上下文的控制权将移动到当前执行栈的下一个执行上下文。

   - 作用域链

     从当前作用域开始一层一层向上寻找某个变量，直到找到全局作用域还是没找到，就宣布放弃。这种一层一层的关系，就是作用域链。

   - 闭包

     闭包是指有权访问另一个函数作用域中的变量的函数

     作用：

     - 能够访问函数定义时所在的作用域(阻止其被回收)

     - 私有化变量(函数里声明变量)

     - 模拟块级作用域(for var let 的问题)

     - 创建模块(函数中的函数)

     模块模式具有两个必备的条件：

     - 必须有外部的封闭函数，该函数必须至少被调用一次(每次调用都会创建一个新的模块实例)

     - 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

   - 私有/公有属性/方法与变量提升

     ```js
     function Foo() {
       getName = function () {
         alert(1);
       };
       return this;
     }
     Foo.getName = function () {
       alert(2);
     };
     Foo.prototype.getName = function () {
       alert(3);
     };
     var getName = function () {
       alert(4);
     };
     function getName() {
       alert(5);
     }

     //答案：
     Foo.getName(); //2
     getName(); //4
     Foo().getName(); //1，相当于this.getName()
     getName(); //1
     new Foo.getName(); //2,相当于(new (Foo.getName))(),.成员访问(18)->new有参数列表(18)
     new Foo().getName(); //3,相当于(new Foo()).getName(),new有参数列表(18)->.成员访问(18)->()函数调用(17)
     new new Foo().getName(); //3,相当于new ((new Foo()).getName)(),new有参数列表(18)->new有参数列表(18)
     ```

     - 私有/公有 属性/方法

       ```js
       function User(name) {
         var name = name; //私有属性
         this.name = name; //公有属性
         function getName() {
           //私有方法
           return name;
         }
       }
       User.prototype.getName = function () {
         //公有方法
         return this.name;
       };
       User.name = "Wscats"; //静态属性
       User.getName = function () {
         //静态方法
         return this.name;
       };
       var Wscat = new User("Wscats"); //实例化
       ```

       - 注意：

       1. 调用公有方法，公有属性，必需先 new，公有方法是不能调用私有方法和静态方法

       2. 静态方法和静态属性就是我们无需 new 就可以调用

       3. 私有方法和属性,外部是不可以访问

     - 声明式变量提升

       ```js
       getName() //oaoafly
           var getName = function() {
           console.log('wscat')
       }
       getName() //wscat
       function getName() {
           console.log('oaoafly')
       }
       getName() //wscat
       --------------------------------------
       var getName;
       console.log(getName) //undefined
       getName() //Uncaught TypeError: getName is not a function
       var getName = function() {
           console.log('wscat')
       }
       --------------------------------------
       var getName;
       console.log(getName) //function getName() {console.log('oaoafly')}
       getName() //oaoafly
       function getName() {
           console.log('oaoafly')
       }
       ```

       - 注意

       1. 声明式函数会被提升到作用域的最前面，即使代码写在最后

       2. 声明式函数要在赋值完成才能调用

     - 构造函数与原型函数和返回值

       ```js
       function Foo() {
         this.getName = function () {
           console.log(3);
           return {
             getName: getName, //这个就是第六问中涉及的构造函数的返回值问题
           };
         }; //这个就是第六问中涉及到的，JS构造函数公有方法和原型链方法的优先级
         getName = function () {
           console.log(1);
         };
         return this;
       }
       Foo.getName = function () {
         console.log(2);
       };
       Foo.prototype.getName = function () {
         console.log(6);
       };
       var getName = function () {
         console.log(4);
       };
       function getName() {
         console.log(5);
       }
       //答案：
       Foo.getName(); //2
       getName(); //4
       console.log(Foo());
       Foo().getName(); //1
       getName(); //1
       new Foo.getName(); //2
       new Foo().getName(); //3
       //多了一问
       new Foo().getName().getName(); //3 1
       new new Foo().getName(); //3
       ```

       - 注意

       1. 如果构造函数和原型链都有相同的方法，默认会拿构造函数的公有方法(this.getName)而不是原型链(Foo.prototype.getName)

       2. JS 中构造函数可以有返回值，没有返回值则按照其他语言一样返回实例化对象

       3. 构造函数若有返回值为基本类型（String,Number,Boolean,Null,Undefined）则与无返回值相同，实际返回其实例化对象

       4. 构造函数若有返回值为引用类型(对象)，则实际返回值为这个引用类型(返回 object 具体内容，返回 this 同理)

   - lazyman

     考察对象使用，链式调用，闭包缓存参数，函数队列，事件循环

     题目

     ```txt
     实现一个LazyMan，可以按照以下方式调用:
     LazyMan(“Hank”)输出:
     Hi! This is Hank!

     LazyMan(“Hank”).sleep(10).eat(“dinner”)输出
     Hi! This is Hank!
     //等待10秒..
     Wake up after 10
     Eat dinner~

     LazyMan(“Hank”).eat(“dinner”).eat(“supper”)输出
     Hi This is Hank!
     Eat dinner~
     Eat supper~

     LazyMan(“Hank”).sleepFirst(5).eat(“supper”)输出
     //等待5秒
     Wake up after 5
     Hi This is Hank!
     Eat supper
     ```

     ```js
     function Lazyman(name) {
       return new _Lazyman(name);
     }
     class _Lazyman {
       constructor(name) {
         this.tasks = []; //设置任务队列
         let self = this;
         let task = (function (name) {
           return function () {
             console.log("Hello I'm " + name);
             self.next(); //因为没法for循环执行，所以只能console完就调用next来执行下一个
           };
         })(name); //闭包传参能缓存参数，最终返回函数，否则无法在next中向function传参
         this.tasks.push(task);
         //此时首次进入构造函数，tasks为[f]，调用方式为tasks[0]()
         //通过settimeout的方法，先执行链式调用和传参，最后才执行next()
         setTimeout(function () {
           self.next();
         }, 0);
       }
       next() {
         //取出一个任务并执行
         let task = this.tasks.shift();
         if (task) {
           task();
         }
       }
       eat(food) {
         let self = this;
         let task = (function (food) {
           return function () {
             console.log("Eat " + food);
             self.next();
           };
         })(food);
         this.tasks.push(task);
         return this; //链式调用
       }
       sleep(time) {
         let self = this;
         let task = (function (time) {
           return function () {
             setTimeout(function () {
               console.log("Wake up after " + time + " s!");
               self.next(); //setTimeout执行完才能执行下一个
             }, time * 1000);
           };
         })(time);
         this.tasks.push(task);
         return this;
       }
       sleepFirst(time) {
         let self = this;
         let task = (function (time) {
           return function () {
             setTimeout(function () {
               console.log("Wake up after " + time + " s!");
               self.next();
             }, time * 1000);
           };
         })(time);
         this.tasks.unshift(task); //函数最先执行，向队列头部插入函数
         return this;
       }
     }
     // Lazyman('Hank');
     // Lazyman('Hank').sleep(5).eat('dinner');
     // Lazyman('Hank').eat('dinner').eat('supper');
     Lazyman("Hank").sleepFirst(5).eat("supper");
     ```

   - let 与函数块级作用域

     1. 变量提升

        ```js
        console.log(a); //undefined
        var a = 0;
        ```

        相当于

        ```js
        var a;
        console.log(a); //undefined
        a = 0;
        ```

     2. 暂时性死区

        let,const 在同一个作用域，同一个变量只能被一次“特殊声明”,var 的声明是如果已经声明了，后者直接忽略声明

            ```js
            var x = 0;
            function a(){
                console.log(x);//Uncaught ReferenceError: Cannot access 'x' before initialization,已有x声明，但未初始化
                let x = 1;
            }
            a();
            ```
            ```js
            let a = a;
            let a = a;//Uncaught SyntaxError: Identifier 'a' has already been declared，违反let只声明一次的原则
            ```
            同理
            ```js
            var a = a;
            let a = a;//Uncaught SyntaxError: Identifier 'a' has already been declared
            ```

            ```html
            <p>a</p>
            <p>b</p>
            <p>c</p>
            <p>d</p>
            <p>e</p>
            <script>
              window.onload = function() {
                var p = document.getElementsByTagName('p');
                for(var i = 0;i < p.length;i++){
                  p[i].addEventListener('click',function(){
                    console.log(this.innerHTML,i)
                  });
                }
                //输出：
                //a 5
                //b 5
                //c 5
                //d 5
                //e 5
              }
            </script>
            ```
            ```html
            <p>a</p>
            <p>b</p>
            <p>c</p>
            <p>d</p>
            <p>e</p>
            <script>
              window.onload = function() {
                var p = document.getElementsByTagName('p');
                for(let i = 0;i < p.length;i++){
                  p[i].addEventListener('click',function(){
                    console.log(this.innerHTML,i)
                  });
                }
                //输出：
                //a 0
                //b 1
                //c 2
                //d 3
                //e 4
              }
            </script>
            ```
    
        3. 函数提升
    
            ```js
            console.log(a);// undfined
            var a = function (){}
    
            console.log(a); // function a
            function a(){}
            ```
    
        4. 块级作用域
    
            ```js
            console.log(a);// undefined
            if(true){
                console.log(a); // function a
                function a(){}
            }
            ```
            预解析为
            ```js
            var a; //  函数 a 的声明
            console.log(a);// undefined
            if(true){
                function a(){} // 函数 a 的定义
                console.log(a); // function a
            }
            ```
    
            catch 里面遵循的是块作用域
            ```js
            try{
                console.log(a);// undefined
                aa.c;
            }catch(e){
                var a = 1;
            }
            console.log(a);// 1
            console.log(e);// Uncaught ReferenceError: e is not defined
            ```
    
            ```js
            var a = 0;
            if(true){
                a = 1;
                function a(){}
                a = 21;
                console.log("里面",a);//里面 21
            }
            console.log("外部",a);//外部 1
            ```
            预解析如下
            ```js
            var a = 0;
            if(true){
                console.log(a,window.a);// 函数提升，是块级作用域，输出 function a 和 0
                a = 1;  // 取作用域最近的块级作用域的 function a ,且被重置为 1了，本质又是一个 变量的赋值。
                console.log(a,window.a);// a 是指向块级作用域的 a, 输出 1 和 0
                function a(){} // 函数的声明，将执行函数的变量的定义同步到函数级的作用域。
                console.log(a,window.a);// 输出 1 和 1
                a = 21; // 仍然是函数定义块级作用域的 a ,重置为 21
                console.log(a,window.a); // 输出为函数提升的块级作用域的 a, 输出 21，1
                console.log("里面",a);
            }
            console.log("外部",a);
            ```
    
    * 隐式调用
    
        让(a==1&&a==2&&a==3)为true
    
        1. toString
    
            ```js
            let a = {
                i : 1,
                toString: function(){
                    return a.i++
                }
            }
            if(a==1&&a==2&&a==3){
                console.log('success')
            } else {
                console.log('fail')
            }
            ```
    
        2. defineProperty
    
            ```js
            var val = 0;
            Object.defineProperty(window, 'a', {
                get: function() {
                    return ++val;
                }
            });
            if (a == 1 && a == 2 && a == 3) {
                console.log('yay');
            }
            ```
    
        3. Array.join
    
            ```js
            let a = [1,2,3];
            a.join = a.shift;
            if(a==1&&a==2&&a==3){
                console.log('success')
            } else {
                console.log('fail')
            }
            ```


    * 属性读取
    
        cannot read property of undefined 是一个常见的错误，如果意外的得到了一个空对象或者空值，便会报错
    
        obj.user && obj.user.posts
    
        与或运算
        ```js
        let one = 1, two = 2, three = 3;
        console.log(one && two && three); // Result: 3
        console.log(0 && null); // Result: 0
        ```
    
        新特性：?.操作符
        ```js
        this.state.data?.()
        this.state？.data
        ```


### 严格模式

1. 参考链接：

   [JavaScript 严格模式(use strict)](https://www.runoob.com/js/js-strict.html)

2. 详解

   1. 变量必须用 var 等声明，变量不能重名
   2. 不允许 delete 除变量属性的值
   3. 不能使用 8 进制，转义字符\
   4. 不能使用保留字段作为变量名，如 public，arguments，eval 等
   5. 不能调用 eval 创建的变量
   6. this 不会指向全局，而是会 undefined
   7. 构造函数后，必须使用 new 才能访问对象属性

   ```js
   var a = function () {
     this.b = 3;
   };
   var c = new a();
   a.prototype.b = 9;
   var b = 7;
   a();

   console.log(b); //3，a()修改this.b指向全局，覆盖b=7
   console.log(c.b); //3
   ```

   ```js
   "use strict";
   var a = function () {
     this.b = 3;
   }; //Cannot set property 'b' of undefined，this指向全局为undefined
   var c = new a();
   a.prototype.b = 9;
   var b = 7;
   a();

   console.log(b); //没有执行，直接上方报错
   console.log(c.b);
   ```

### 前端路由原理

1. 参考链接：

   [前端路由的前生今世及实现原理](https://segmentfault.com/a/1190000011967786)

2. 详解：

   - 后端渲染路由

     1.浏览器发出请求

     2.服务器监听到 80/443 端口的请求，并解析 url 路径

     3.根据服务器的路由配置，返回相应信息（html、json、image）

     4.浏览器根据数据包的 Content-Type 来决定如何解析数据

   - 前端路由

     检测 url 的变化，截获 url 地址，然后解析来匹配路由规则。

     https://...#value 井号后面的 value 为 hash，hash 变化不会请求后端，只会触发 hashchange 事件，然后 js 解析新的页面内容。回退使用 history.go(-1)，前进使用 hashchange 事件，刷新使用 load 事件。

     pushState 和 replaceState 方法，以及 onpopstate 事件，能够使 url 不出现井号跳转，原理和 hash 相同，如 vue 的 history 模式，但是刷新页面依然会发请求导致 404，因此需要服务器转发请求，重定向到根页面，如使用 nginx。回退使用 popstate 事件，前进使用 pushState，刷新使用服务器重定向，再 load。

     ```txt
     server {
         listen       8083;
         server_name  localhost;

         location / {
             root   D:\wwwroot;
             try_files $uri $uri/ /index.html;
             index  index.html index.htm;
         }

         location /api {
             add_header 'Access-Control-Allow-Origin' '*';
             proxy_pass http://localhost:7675/api;
         }

         error_page   500 502 503 504  /50x.html;
         location = /50x.html {
             root   html;
         }
     }
     ```

   - pushState 和 replaceState

     history.pushState(状态对象, 标题 , URL);创建新的历史记录条目

     history.replaceState(状态对象, 标题 , URL);修改历史记录条目

     状态对象是能被序列化的对象(小于 640k)，用户导航到新的状态，popstate 事件就会被触发，且该事件的 state 属性包含该历史记录条目状态对象的副本。

     获取当前状态：let currentState = history.state;


### 抽象语法树 AST 与 babel

1. 参考链接：

   - [一看就懂的 JS 抽象语法树](https://segmentfault.com/a/1190000012943992)
   - [中高级前端大厂面试秘籍，为你保驾护航金三银四，直通大厂(上)](https://juejin.im/post/5c64d15d6fb9a049d37f9c20)

2. 详解

   抽象语法树：将代码逐字母解析成树状对象的形式，是语言之间的转换、代码语法检查，代码风格检查，代码格式化，代码高亮，代码错误提示，代码自动补全等等的基础

   - 样例

     ```js
     var a = 1;
     var b = a + 1;
     {
         "type": "Program",
         "body": [
             {
                 "type": "VariableDeclaration",
                 "declarations": [
                     {
                         "type": "VariableDeclarator",
                         "id": {
                             "type": "Identifier",
                             "name": "a"
                         },
                         "init": {
                             "type": "Literal",
                             "value": 1,
                             "raw": "1"
                         }
                     }
                 ],
                 "kind": "var"
             },
             {
                 "type": "VariableDeclaration",
                 "declarations": [
                     {
                         "type": "VariableDeclarator",
                         "id": {
                             "type": "Identifier",
                             "name": "b"
                         },
                         "init": {
                             "type": "BinaryExpression",
                             "operator": "+",
                             "left": {
                                 "type": "Identifier",
                                 "name": "a"
                             },
                             "right": {
                                 "type": "Literal",
                                 "value": 1,
                                 "raw": "1"
                             }
                         }
                     }
                 ],
                 "kind": "var"
             }
         ],
         "sourceType": "script"
     }
     ```

     - 常用引擎

       - esprima
       - acron
       - Traceur
       - UglifyJS2
       - shift

     - 使用示例

       ```js
       //npm i esprima estraverse escodegen --save
       const esprima = require("esprima");
       let code = "const a = 1";
       const ast = esprima.parseScript(code);
       console.log(ast);
       //Script {
       //type: 'Program',
       //body:
       //[ VariableDeclaration {
       //    type: 'VariableDeclaration',
       //    declarations: [Array],
       //    kind: 'const' } ],
       //sourceType: 'script' }
       const estraverse = require("estraverse");
       estraverse.traverse(ast, {
         enter: function (node) {
           node.kind = "var";
         },
       });
       console.log(ast);
       //Script {
       //type: 'Program',
       //body:
       //[ VariableDeclaration {
       //    type: 'VariableDeclaration',
       //    declarations: [Array],
       //    kind: 'var' } ],
       //sourceType: 'script' }
       const escodegen = require("escodegen");
       const transformCode = escodegen.generate(ast);
       console.log(transformCode);
       //var a = 1;//把const a = 1编译成这个
       ```

     - babel

       1. babylon 将 ES6/ES7 代码解析成 AST
       2. babel-traverse 对 AST 进行遍历转译，得到新的 AST
       3. 新 AST 通过 babel-generator 转换成 ES5


### 判断js运行环境

1. 参考链接：

   - [如何判断当前脚本是运行在浏览器还是node环境中](https://www.cnblogs.com/zhangyue690811/p/12041230.html)

2. 详解

  判断global对象是否为window
  ```js
  this === window ? console.log('browser') : console.log('node');
  ```

### let、const、var的区别

1. 参考链接：

  - [「2021」高频前端面试题汇总之JavaScript篇（上）](https://juejin.cn/post/6940945178899251230)

2. 详解

* 块级作用域

  块作用域由 { }包括，let和const具有块级作用域，var不存在块级作用域。块级作用域解决了ES5中的两个问题：

  * 内层变量可能覆盖外层变量
  * 用来计数的循环变量泄露为全局变量

* 变量提升

  var存在变量提升，let和const不存在变量提升，即在变量只能在声明之后使用，否在会报错。

* 给全局添加属性

  浏览器的全局对象是window，Node的全局对象是global。var声明的变量为全局变量，并且会将该变量添加为全局对象的属性，但是let和const不会。

* 重复声明

  var声明变量时，可以重复声明变量，后声明的同名变量会覆盖之前声明的遍历。const和let不允许重复声明变量。

* 暂时性死区

  在使用let、const命令声明变量之前，该变量都是不可用的。这在语法上，称为暂时性死区。使用var声明的变量不存在暂时性死区。

* 初始值设置

  在变量声明时，var 和 let 可以不用设置初始值。而const声明变量必须设置初始值。

* 指针指向

  let和const都是ES6新增的用于创建变量的语法。 let创建的变量是可以更改指针指向（可以重新赋值）。但const声明的变量是不允许改变指针的指向。

### 箭头函数与普通函数

1. 参考链接：

  - [「2021」高频前端面试题汇总之JavaScript篇（上）](https://juejin.cn/post/6940945178899251230)

2. 详解

* 区别

  1. 箭头函数比普通函数更加简洁

    * 如果没有参数，就直接写一个空括号即可
    * 如果只有一个参数，可以省去参数的括号
    * 如果有多个参数，用逗号分割
    * 如果函数体的返回值只有一句，可以省略大括号
    * 如果函数体不需要返回值，且只有一句话，可以给这个语句前面加一个void关键字。最常见的就是调用一个函数：

      ```js
      let fn = () => void doesNotReturn();
      ```

  2. 箭头函数没有自己的this

  箭头函数不会创建自己的this， 所以它没有自己的this，它只会在自己作用域的上一层继承this。所以箭头函数中this的指向在它在定义时已经确定了，之后不会改变。

  3. 箭头函数继承来的this指向永远不会改变

  ```js
  var id = 'GLOBAL';
  var obj = {
    id: 'OBJ',
    a: function(){
      console.log(this.id);
    },
    b: () => {
      console.log(this.id);
    }
  };
  obj.a();    // 'OBJ'
  obj.b();    // 'GLOBAL'
  new obj.a()  // undefined
  new obj.b()  // Uncaught TypeError: obj.b is not a constructor
  ```

  对象obj的方法b是使用箭头函数定义的，这个函数中的this就永远指向它定义时所处的全局执行环境中的this，即便这个函数是作为对象obj的方法调用，this依旧指向Window对象。需要注意，定义对象的大括号{}是无法形成一个单独的执行环境的，它依旧是处于全局执行环境中。

  4. call()、apply()、bind()等方法不能改变箭头函数中this的指向

  ```js
  var id = 'Global';
  let fun1 = () => {
      console.log(this.id)
  };
  fun1();                     // 'Global'
  fun1.call({id: 'Obj'});     // 'Global'
  fun1.apply({id: 'Obj'});    // 'Global'
  fun1.bind({id: 'Obj'})();   // 'Global'
  ```

  5. 箭头函数不能作为构造函数使用

  构造函数在new的步骤在上面已经说过了，实际上第二步就是将函数中的this指向该对象。 但是由于箭头函数时没有自己的this的，且this指向外层的执行环境，且不能改变指向，所以不能当做构造函数使用。

  6. 箭头函数没有自己的arguments

  箭头函数没有自己的arguments对象。在箭头函数中访问arguments实际上获得的是它外层函数的arguments值。

  7. 箭头函数没有prototype

  8. 箭头函数不能用作Generator函数，不能使用yeild关键字

* new一个箭头函数的会怎么样

  箭头函数是ES6中的提出来的，它没有prototype，也没有自己的this指向，更不可以使用arguments参数，所以不能New一个箭头函数。
  
  Uncaught TypeError: a is not a constructor

  * new操作符的实现步骤如下：

    1. 创建一个对象
    2. 将构造函数的作用域赋给新对象（也就是将对象的__proto__属性指向构造函数的prototype属性）
    3. 指向构造函数中的代码，构造函数中的this指向该对象（也就是为这个对象添加属性和方法）
    4. 返回新的对象

  所以，上面的第二、三步，箭头函数都是没有办法执行的。

* 箭头函数的this指向

  箭头函数不同于传统JavaScript中的函数，箭头函数并没有属于⾃⼰的this，它所谓的this是捕获其所在上下⽂的 this 值，作为⾃⼰的 this 值，并且由于没有属于⾃⼰的this，所以是不会被new调⽤的，这个所谓的this也不会被改变。

  Babel
  ```js
  // ES6 
  const obj = { 
    getArrow() { 
      return () => { 
        console.log(this === obj); 
      }; 
    } 
  }
  // ES5，由 Babel 转译
  var obj = { 
    getArrow: function getArrow() { 
      var _this = this; 
      return function () { 
          console.log(_this === obj); 
      }; 
    } 
  };
  ```

### rest扩展运算符

1. 参考链接：

  - [「2021」高频前端面试题汇总之JavaScript篇（上）](https://juejin.cn/post/6940945178899251230)

2. 详解

  1. 对象扩展运算符

  对象的扩展运算符(...)用于取出参数对象中的所有可遍历属性，拷贝到当前对象之中。

  ```js
  let bar = { a: 1, b: 2 };
  let baz = { ...bar }; // { a: 1, b: 2 }
  //上述方法实际上等价于:
  let bar = { a: 1, b: 2 };
  let baz = Object.assign({}, bar); // { a: 1, b: 2 }
  ```

  Object.assign方法用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target）。Object.assign方法的第一个参数是目标对象，后面的参数都是源对象。(如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性)。

  同样，如果用户自定义的属性，放在扩展运算符后面，则扩展运算符内部的同名属性会被覆盖掉。
  ```js
  let bar = {a: 1, b: 2};
  let baz = {...bar, ...{a:2, b: 4}};  // {a: 2, b: 4}
  ```

  利用上述特性就可以很方便的修改对象的部分属性。在redux中的reducer函数规定必须是一个纯函数，reducer中的state对象要求不能直接修改，可以通过扩展运算符把修改路径的对象都复制一遍，然后产生一个新的对象返回。

  需要注意：扩展运算符对对象实例的拷贝属于浅拷贝。

  2. 数组扩展运算符

  数组的扩展运算符可以将一个数组转为用逗号分隔的参数序列，且每次只能展开一层数组。
  ```js
  console.log(...[1, 2, 3])
  // 1 2 3
  console.log(...[1, [2, 3, 4], 5])
  // 1 [2, 3, 4] 5
  ```

    * 数组的扩展运算符的应用

      将数组转换为参数序列
      ```js
      function add(x, y) {
        return x + y;
      }
      const numbers = [1, 2];
      add(...numbers) // 3
      ```

      复制数组
      ```js
      const arr1 = [1, 2];
      const arr2 = [...arr1];
      ```

      扩展运算符(…)用于取出参数对象中的所有可遍历属性，拷贝到当前对象之中，这里参数对象是个数组，数组里面的所有对象都是基础数据类型，将所有基础数据类型重新拷贝到新的数组中。

      合并数组
      ```js
      const arr1 = ['two', 'three'];const arr2 = ['one', ...arr1, 'four', 'five'];// ["one", "two", "three", "four", "five"]
      ```

      扩展运算符与解构赋值结合起来，用于生成数组
      ```js
      const [first, ...rest] = [1, 2, 3, 4, 5];first // 1rest  // [2, 3, 4, 5]
      ```
      需要注意：如果将扩展运算符用于数组赋值，只能放在参数的最后一位，否则会报错。
      ```js
      const [...rest, last] = [1, 2, 3, 4, 5];         // 报错const [first, ...rest, last] = [1, 2, 3, 4, 5];  // 报错
      ```

      将字符串转为真正的数组
      ```js
      [...'hello']    // [ "h", "e", "l", "l", "o" ]
      ```

      任何 Iterator 接口的对象，都可以用扩展运算符转为真正的数组，比较常见的应用是可以将某些数据结构转为数组：
      ```js
      // arguments对象
      function foo() {
        const args = [...arguments];//用于替换es5中的Array.prototype.slice.call(arguments)写法。
      }
      ```

      使用Math函数获取数组中特定的值
      ```js
      const numbers = [9, 4, 7, 1];
      Math.min(...numbers); // 1
      Math.max(...numbers); // 9
      ```

### map和weakMap

1. 参考链接：

  - [「2021」高频前端面试题汇总之JavaScript篇（上）](https://juejin.cn/post/6940945178899251230)

2. 详解

  * Map

    map本质上就是键值对的集合，但是普通的Object中的键值对中的键只能是字符串。而ES6提供的Map数据结构类似于对象，但是它的键不限制范围，可以是任意类型，是一种更加完善的Hash结构。如果Map的键是一个原始数据类型，只要两个键严格相同，就视为是同一个键。

    实际上Map是一个数组，它的每一个数据也都是一个数组，其形式如下：
    ```js
    const map = [
        ["name","张三"],
        ["age",18],
    ]
    ```

    * Map数据结构有以下操作方法

      * size： map.size 返回Map结构的成员总数。
      * set(key,value)：设置键名key对应的键值value，然后返回整个Map结构，如果key已经有值，则键值会被更新，否则就新生成该键。（因为返回的是当前Map对象，所以可以链式调用）
      * get(key)：该方法读取key对应的键值，如果找不到key，返回undefined。
      * has(key)：该方法返回一个布尔值，表示某个键是否在当前Map对象中。
      * delete(key)：该方法删除某个键，返回true，如果删除失败，返回false。
      * clear()：map.clear()清除所有成员，没有返回值。

    * Map结构原生提供是三个遍历器生成函数和一个遍历方法

      * keys()：返回键名的遍历器。
      * values()：返回键值的遍历器。
      * entries()：返回所有成员的遍历器。
      * forEach()：遍历Map的所有成员。

      ```js
      const map = new Map([
          ["foo",1],
          ["bar",2],
      ])
      for(let key of map.keys()){
          console.log(key);  // foo bar
      }
      for(let value of map.values()){
          console.log(value); // 1 2
      }
      for(let items of map.entries()){
          console.log(items);  // ["foo",1]  ["bar",2]
      }
      map.forEach( (value,key,map) => {
          console.log(key,value); // foo 1    bar 2
      })
      ```

  * WeakMap

    WeakMap 对象也是一组键值对的集合，其中的键是弱引用的。其键必须是对象，原始数据类型不能作为key值，而值可以是任意的。

    * WeakMap有以下几种方法：

      * set(key,value)：设置键名key对应的键值value，然后返回整个Map结构，如果key已经有值，则键值会被更新，否则就新生成该键。（因为返回的是当前Map对象，所以可以链式调用）
      * get(key)：该方法读取key对应的键值，如果找不到key，返回undefined。
      * has(key)：该方法返回一个布尔值，表示某个键是否在当前Map对象中。
      * delete(key)：该方法删除某个键，返回true，如果删除失败，返回false。

      其clear()方法已经被弃用，所以可以通过创建一个空的WeakMap并替换原对象来实现清除。

      WeakMap的设计目的在于，有时想在某个对象上面存放一些数据，但是这会形成对于这个对象的引用。一旦不再需要这两个对象，就必须手动删除这个引用，否则垃圾回收机制就不会释放对象占用的内存。

      而WeakMap的键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。

  * 总结

  1. Map 数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。
  2. WeakMap 结构与 Map 结构类似，也是用于生成键值对的集合。但是 WeakMap 只接受对象作为键名（ null 除外），不接受其他类型的值作为键名。而且 WeakMap 的键名所指向的对象，不计入垃圾回收机制。