---
title: Javascript的数据结构与算法(一)
date: 2015-8-28 20:24:51
categories: 数据结构与算法
tags: 
  - 数据结构与算法
keywords: javascript,html,css,web
description: Javascript的数据结构与算法(一)
---

# 1数组

## 1.1方法列表

数组的常用方法如下:

-  concat: 链接两个或者更多数据，并返回结果。
-  every: 对数组中的每一项运行给定的函数，如果该函数对每一项都返回true，则返回true。
-  filter: 对数组中的每一项运行给定函数，返回改函数会返回true的项组成的数组。
-  forEach: 对数组中的每一项运行给定函数，这个方法没有返回值。
-  join: 将所有的数组元素链接成一个字符串。
-  indexOf: 返回第一个与给定参数相等的数组元素的索引，没有找到则返回-1。
-  lastIndexOf: 返回在数组中搜索到的与给定参数相等的元素的索引里最大的值。
-  map: 对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。
-  reverse: 颠倒数组中元素的顺序，原先第一个元素现在变成最后一个，同样原先的最后一个元素变成现在的第一个。
-  slice: 传入索引值，将数组中对应索引范围内的元素作为新元素返回。
-  some: 对数组中的每一项运行给定函数，如果任一项返回true，则返回true。
-  sort: 按照字母顺序对数组排序，支持传入指定排序方法的函数作为参数。
-  toString: 将数组作为字符串返回。
-  valueOf: 和toString相似，将数组作为字符串返回。

<!-- more -->

## 1.2数组合并
 concat方法可以向一个数组传递数组、对象或是元素。数组会按照该方法传入的参数顺序 连接指定数组。

```` javascript
    var zero = 0;
    var arr1 = [1,2,3];
    var arr2 = [-1,-2,-3];
    var result = arr2.concat(zero, arr1);
    console.log(result); // 输出结果： [-1, -2, -3, 0, 1, 2, 3]
````

## 1.3迭代器函数
reduce方法接收一个函数作为参数,这个函数有四个参数:previousValue、currentValue、index和array。这个函数会返回一个将被叠加到累加器的 值,reduce方法停止执行后会返回这个累加器。如果要对一个数组中的所有元素求和,这就很有用了。

```` javascript
    var isEven = function(x){
      return (x % 2 == 0);
    }
    var numbers = [1, 2, 3, 4, 5, 6];
    // every方法会迭代数组中的每个元素,直到返回false。
    var result = numbers.every(isEven); // false
    // some方法会迭代数组的每个元 素,直到函数返回true.
    result = numbers.some(isEven); // true
    // forEach对每一项运行给定的函数，没有返回值
    numbers.forEach(function(item, index) {
      console.log(item % 2 == 0);
    });
    // map会迭代数组中的每个值，并且返回迭代结果
    var myMap = numbers.map(isEven); // [false, true, false, true, false, true]
    // filter方法返回的新数组由使函数返回true的元素组成
    var myFilter = numbers.filter(isEven); // [2, 4, 6]
    //reduce函数
    var myReduce = numbers.reduce(function(previous,current,index){
      return previous + "" + current;
    });
    console.log(myReduce); // 123456
````

##  1.4排序

```` javascript
    var numbers = [1,2,3,4,5,6];
    numbers.reverse();//[6, 5, 4, 3, 2, 1]
    /**
    若 a 小于 b，在排序后的数组中 a 应该出现在 b 之前，则返回一个小于 0 的值。
    若 a 等于 b，则返回 0。
    若 a 大于 b，则返回一个大于 0 的值。
    **/
    function compare(a,b){
      if(a > b){
        return 1;
      }
      if(a < b){
        return -1;
      }
      return 0;
    }
    //搜索
    numbers.push(10); [1,2,3,4,5,6,10];
    numbers.indexOf(10); // 6
    numbers.lastIndexOf(10); // 6
    var numbersString = numbers.join('-'); // "1-2-3-4-5-6-10"
````

# 2栈
---

## 2.1栈的创建
对于一个栈，我们需要实现添加、删除元素、获取栈顶元素、已经是否为空，栈的长度、清除元素等几个基本操作。下面是基本定义。

```` javascript
    function Stack(){
      this.items = [];
    }
    Stack.prototype = {
      constructor: Stack,
      push: function(element) {
        this.items.push(element);
      },
      pop: function() {
        return this.items.pop();
      },
      // 栈顶元素
      peek: function() {
        return this.items[this.items.length - 1];
      },
      isEmpty: function() {
        return this.items.length == 0;
      },
      clear: function() {
        this.items = [];
      },
      size: function() {
        return this.items.length;
      },
      print: function() {
        console.log(this.items.toString());
      }
    }
````

## 2.2栈的基本使用
栈的基本操作。

```` javascript
    var stack = new Stack();
    console.log(stack.isEmpty()); // true
    stack.push(5);
    stack.push(8);
    console.log(stack.peek()); // 8
    stack.push(11);
    console.log(stack.size()); // 3
    console.log(stack.isEmpty()); //false
    stack.push(15);
    stack.pop(); // 15
    stack.pop(); // 11
    console.log(stack.size()); // 2
    console.log(stack.print()); // 5,8
````
通过栈实现对正整数的二进制转换。

```` javascript
    function divideBy2(decNumber){
      var decStack = new Stack();
      var rem;
      var decString = '';
      while(decNumber > 0) {
        rem = decNumber % 2;
        decStack.push(rem);
        decNumber = Math.floor(decNumber/2);
      }
      while(!decStack.isEmpty()){
        decString += decStack.pop().toString();
      }
      return decString;
    }
    console.log(divideBy2(10));//1010
````

# 3队列
---

## 3.1队列的创建
队列是遵循FIFO(First In First Out,先进先出,也称为先来先服务)原则的一组有序的项。队列在尾部添加新元素,并从顶部移除元素。最新添加的元素必须排在队列的末尾。队列要实现的操作基本和栈一样，只不过栈是FILO(先进后出)。

````javascript
    function Queue() {
      this.items = [];
    }
    Queue.prototype = {
      constructor: Queue,
      enqueue: function(elements) {
        this.items.push(elements);
      },
      dequeue: function() {
        return this.items.shift();
      },
      front: function() {
        return this.items[0];
      },
      isEmpty: function() {
        return this.items.length == 0;
      },
      size: function() {
        return this.items.length;
      },
      clear: function() {
        this.items = [];
      },
      print: function() {
        console.log(this.items.toString());
      }
    }
````
队列的基本使用

````javascript
    var queue = new Queue();
    console.log(queue.isEmpty()); // true
    queue.enqueue('A');
    queue.enqueue('B');
    queue.print(); // A,B
    queue.size(); // 2
    queue.isEmpty(); // false
    queue.enqueue('C'); 
    queue.dequeue(); 
    queue.print(); //B C
````

## 3.2 优先队列
元素的添加和移除是基于优先级的。实现一个优先队列,有两种选项:设置优先级,然后在正确的位置添加元素;或者用入列操 作添加元素,然后按照优先级移除它们。
我们在这里实现的优先队列称为最小优先队列,因为优先级的值较小的元素被放置在队列最 前面(1代表更高的优先级)。最大优先队列则与之相反,把优先级的值较大的元素放置在队列最 前面。

### 3.2.1 优先队列的定义
我们在这里使用组合继承的方式继承自Queue队列。

````javascript
    function PriorityQueue(){
      Queue.call(this);
    };
    PriorityQueue.prototype = new Queue();
    PriorityQueue.prototype.constructer = PriorityQueue;
    PriorityQueue.prototype.enqueue = function(element, priority) {
      function QueueElement(tempelement, temppriority){
        this.element = tempelement;
        this.priority = temppriority;
      }
      var queueElement = new QueueElement(element, priority);
      if(this.isEmpty()){
        this.items.push(queueElement);
      }else {
        var added = false;
        for(var i = 0; i < this.items.length; i++){
          if(this.items[i].priority > queueElement.priority){
            this.items.splice(i, 0, queueElement);
            added = true;
            break;
          }
        }
        if(!added){
            this.items.push(queueElement);
        }
      }
      
    }
    //这个方法可以用Queue的默认实现
    PriorityQueue.prototype.print = function() {
      var result ='';
      for(var i = 0; i < this.items.length;i++){
        result += JSON.stringify(this.items[i]);
      }
      return result;
    }
````

### 3.2.1 优先队列的基本使用

````javascript
    var pQueue = new PriorityQueue();
    pQueue.enqueue("B", 2);
    pQueue.enqueue("C", 3);
    pQueue.enqueue("A", 1);
    console.log(pQueue.print());//{"element":"A","priority":1}{"element":"B","priority":2}{"element":"C","priority":3}
    console.log(pQueue.size()); // 3
    console.log(pQueue.dequeue()); // { element="A",  priority=1}
    console.log(pQueue.size()); // 2
````

# 3链表
---

数组的大小是固定的,从数组的起点或中间插入 或移除项的成本很高,因为需要移动元素(尽管我们已经学过的JavaScript的Array类方法可以帮 我们做这些事,但背后的情况同样是这样)。链表存储有序的元素集合,但不同于数组,链表中的元素在内存中并不是连续放置的。每个 元素由一个存储元素本身的节点和一个指向下一个元素的引用(也称指针或链接)组成。

相对于传统的数组,链表的一个好处在于,添加或移除元素的时候不需要移动其他元素。然 而,链表需要使用指针,因此实现链表时需要额外注意。数组的另一个细节是可以直接访问任何 位置的任何元素,而要想访问链表中间的一个元素,需要从起点(表头)开始迭代列表直到找到 所需的元素

## 3.1.1链表的创建

我们使用动态原型模式来创建一个链表。列表最后一个节点的下一个元素始终是null。

````javascript
    function LinkedList(){
      function Node(element) {
        this.element = element;
        this.next = null;
      }
      this.head = null;
      this.length = 0;
      // 通过对一个方法append判断就可以知道是否设置了prototype
      if((typeof this.append !== 'function') && (typeof this.append !== 'string')){
        // 添加元素
        LinkedList.prototype.append = function(element) {
          var node = new Node(element);
          var current;
          if(this.head === null){
            this.head = node;
          } else {
            current = this.head;
            while(current.next !== null){
              current = current.next;
            }
            current.next = node;
          }
          this.length++;
        };
        //插入元素，成功true，失败false
        LinkedList.prototype.insert = function(position, element) {
          if(position > -1 && position < this.length){
            var current = this.head;
            var previous;
            var index = 0;
            var node = new Node(element);
            if(position == 0){
              node.next = current;
              this.head = node;
            }else {
              while(index++ < position){
                previous = current;
                current = current.next;
              }
              node.next = current;
              previous.next = node;
            }
            this.length++;
            return true;
          }else{
            return false;
          }
        };
        // 根据位置删除指定元素，成功 返回元素， 失败 返回null
        LinkedList.prototype.removeAt = function(position) {
          if(position > -1 && position < this.length){
            var current = this.head;
            var previous = null;
            var index = 0;
            if(position == 0) {
              this.head = current.next;
            } else {
              while(index++ < position) {
                previous = current;
                current = current.next;
              }
              previous.next = current.next;
            }
            this.length--;
            return current.element;
          }else{
            return null;
          }
        };
        //根据元素删除指定元素，成功 返回元素， 失败 返回null
        LinkedList.prototype.remove = function(element){
          var index = this.indexOf(element);
          return this.removeAt(index);
        };
        // 返回给定元素的索引，如果没有则返回-1
        LinkedList.prototype.indexOf = function(element) {
          var current = this.head;
          var index = 0;
          while(current){
            if(current.element === element){
              return index;
            }
            index++;
            current = current.next;
          }
          return -1;
        };
        LinkedList.prototype.isEmpty = function() {
          return this.length === 0;
        };
        LinkedList.prototype.size = function() {
          return this.length;
        };
        LinkedList.prototype.toString = function(){
            var string = '';
            var current = this.head;
            while(current) {
              string += current.element;
              current = current.next;
            }
            return string;
        };
        LinkedList.prototype.getHead = function(){
          return this.head;
        };
      }
    }
````

## 3.1.2链表的基本使用

````javascript
    var linkedList = new LinkedList();
    console.log(linkedList.isEmpty()); // true;
    linkedList.append('A');
    linkedList.append('B')
    linkedList.insert(1, 'C');
    console.log(linkedList.toString()); // ABC
    console.log(linkedList.indexOf('C')); // 2
    console.log(linkedList.size()); // 3
    console.log(linkedList.removeAt(2)); // C
    console.log(linkedList.toString()); // AB
````

## 3.2.1双向链表的创建

链表有多种不同的类型,这一节介绍双向链表。双向链表和普通链表的区别在于,在链表中, 一个节点只有链向下一个节点的链接,而在双向链表中,链接是双向的:一个链向下一个元素, 另一个链向前一个元素。

双向链表和链表的区别就是有一个tail属性，所以必须重写insert、append、removeAt方法。每个节点对应的Node也多了一个prev属性。

````javascript
   // 寄生组合式继承实现，详见javascript高级程序设计第七章
   function inheritPrototype(subType, superType) {
       function object(o) {
           function F() {}
           F.prototype = o;
           return new F();
       }
      //  var prototype = Object.create(superType.prototype);
       var prototype = object(superType.prototype);
       prototype.constructor = subType;
       subType.prototype = prototype;
   }
   function DoublyLinkedList() {
       function Node(element) {
           this.element = element;
           this.next = null;
           this.prev = null;
       }
       this.tail = null;
       LinkedList.call(this);
       // 与LinkedList不同的方法自己实现。
       this.insert = function(position, element) {
           if (position > -1 && position <= this.length) {
               var node = new Node(element);
               var current = this.head;
               var previous;
               var index = 0;
               if (position === 0) {
                   if (!this.head) {
                       this.head = node;
                       this.tail = node;
                   } else {
                       node.next = current;
                       current.prev = node;
                       this.head = node;
                   }
               } else if (position == this.length) {
                   current = this.tail;
                   current.next = node;
                   node.prev = current;
                   this.tail = node;
               } else {
                   while (index++ < position) {
                       previous = current;
                       current = current.next;
                   }
                   previous.next = node;
                   node.next = current;
                   current.prev = node;
                   node.prev = previous;
               }
               this.length++;
               return true;
           } else {
               return false;
           }
       };
       this.append = function(element) {
           var node = new Node(element);
           var current;
           if (this.head === null) {
               this.head = node;
               this.tail = node;
           } else {
               current = this.head;
               while (current.next !== null) {
                   current = current.next;
               }
               current.next = node;
               node.prev = current;
               this.tail = node;
           }
           this.length++;
       };
       this.removeAt = function(position) {
           if (position > -1 && position < this.length) {
               var current = this.head;
               var previous;
               var index = 0;
               if (position === 0) {
                   this.head = current.next;
                   if (this.length === 1) {
                       this.tail = null;
                   } else {
                       this.head.prev = null;
                   }
               } else if (position === (this.length - 1)) {
                   current = this.tail;
                   this.tail = current.prev;
                   this.tail.next = null;
               } else {
                   while (index++ < position) {
                       previous = current;
                       current = current.next;
                   }
                   previous.next = current.next;
                   current.next.prev = previous;
               }
               this.length--;
               return current.element;
           } else {
               return false;
           }
       };
   }
   inheritPrototype(DoublyLinkedList, LinkedList);
````

## 3.2.2双向链表的基本使用

````javascript
    var doublyList = new DoublyLinkedList();
    console.log(doublyList.isEmpty()); //true;
    doublyList.append('A');
    doublyList.append('C')
    doublyList.insert(1, 'B');
    console.log(doublyList.toString()); // ABC
    console.log(doublyList.indexOf('C')); // 2
    console.log(doublyList.size()); // 3
    console.log(doublyList.removeAt(2)); // C
    console.log(doublyList.toString()); //AB
````

### 3.2.3 循环链表

循环链表可以像链表一样只有单向引用,也可以像双向链表一样有双向引用。循环链表和链 表之间唯一的区别在于,最后一个元素指向下一个元素的指针(tail.next)不是引用null, 而是指向第一个元素(head)。双向循环链表有指向head元素的tail.next,和指向tail元素的head.prev。

代码后续补充~~~