# Section 2.8. 条件表达式

目前为止，我们所说过的所有表达式都是没有分支的语句。假设我们要写一个求取绝对值的函数，当它的参数x是负数，返回-x，否则，就返回x本身。对于这种需求，我们就必须使用条件表达式来解决，比如if表达式。
```clojure
(def abs
  (fn [x]
    (if (< x 0) (- 0 x) x)))

(abs 1)  ;=> 1
(abs -1)  ;=> 1
```

if表达式接收三个参数，第一个参数是判断条件，也就是(< x 0)，当第一个参数为真时，返回第二个参数，否则返回第三个参数。当我们传入1时，1大于0，第一个参数为假，所以返回第三个参数也就是1，而传入-1时，-1的确小于0，所以返回了0-(-1)。

abs函数还有其他很多种定义方法，现在演示其中几样
```clojure
(def abs (fn [x] (if (>= x 0) x (- 0 x))))

(defn abs [x] (if (not (< x 0)) x (- 0 x)))

(def abs #(if (= % 0) 0 (if (< % 0) (- 0 %) %)))
```

在上述第一个表达式中，>=表示一个用来判断大于等于的函数，即当x大于等于0时，返回x。在第二个表达式中，not表示一个取反函数，把真当作参数传给它就会返回假，把假当作参数传给它就会返回真。而第三个表达式演示了if语句的嵌套，当传入的参数不为0时，它可能大于0也可能小于0，所以我们进行第二次判断。

如果，在Clojure中，仅输入比较值的表达式，会返回什么？
```clojure
(< -1 0)  ;=> true
(> -1 0)  ;=> false
```

true表示真，false表示假。除了true和false之外，还有能表示真假的对象么？所有对象都可以用来表示真与假。
```clojure
(if 1 true false)  ;=> true
(if 0 true false)  ;=> true
(if "" true false)  ;=> true
(if "i am str" true false)  ;=> true
(if [] true false)  ;=> true
(if [1 2 3] true false)  ;=> true
(if {} true false)  ;=> true
(if + true false)  ;=> true
(if true true false)  ;=> true
(if false true false)  ;=> false
(if nil true false)  ;=> false
```

的确是所有的对象都可以用来表示真假，但除了false和nil之外，所有的对象都表示真。

我们现在来看看和not相似的and与or函数
```
(and true true)  ;=> true
(and true false)  ;=> false
(and false false)  ;=> false
(not true)  ;=> false
(not false)  ;=> true
(or true true)  ;=> true
(or true false) ;=> true
(or false false)  ;=> false
```

or和and都可以接收任意多个参数，or的参数中只要有一个为真，返回结果为真，而and则相反，and的参数中只要有一个为假，返回结果为假。

如果一个函数返回值或者对象为真或者假，它的标识符名称应该以?结尾。比如，用来判断表是否为空表的empty?,判断值是否为数字的number?,判断值是否未字符串的str?
```clojure
(empty? '(1))  ;=> false
(empty? '())  ;=> true
(number? '())  ;=> false
(number? 1)  ;=> true
(str? "")  ;=> true
(str? 1)  ;=> false
```

当需要判断的条件呈现有规律的增长时，if的嵌套层数也会越来越多，为了表达与书写的方便，在这时，你应该使用cond语句。
```clojure
(defn f [x]
  (cond
    (= x 0) x
    (> x 0) "+"
    (< x 0) "-"))
(f 1)  ;=> "+"
(f 0)  ;=> 0
(f -1)  ;=> "-"
```

cond接收偶数个表达式做参数，第奇数个参数的表达式为真，则执行第奇数+1个表达式。

### Exercise 2.8.1

定义一个函数atom?,当它的参数不以表形态存在时返回真。
