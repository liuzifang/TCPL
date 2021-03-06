# Section 2.9. 简单的递归

我们已经看过了我们如何通过if,and,or,cond控制流程。我们可以通过上述的语句，调用一个在结束时调用自身的函数来执行多遍同样的任务，而这种非书写形式地执行多遍同样操作的情形被称之为循环，但在循环中程序结尾调用自身的情形被称为递归。

比如这样一个函数
```clojure
;这是一种无限递归，你不能在repl中输入它
(defn goodbye [] (goodbye))
(goodbye)
```

虽然它是一个无意义的纯粹是浪费资源的函数，但它仍然是一种合法的递归。当这个函数goodbye被调用的时候，所做的唯一一件事就是调用goodbye，而这个goodbye仍然是继续调用goodbye，如此往复，没有尽头。

显然，要写一个实用的递归函数，我们还必须添加一个出口，也就是一个结束的条件。一个递归函数的基本要素就在于出口以及每次执行时传入对象的改变，而这些改变是每次运行得出不同结果的唯一要素。

让我们来考虑一种用以计算列表长度的循环函数。首先我们要明确当什么情况下，这个循环会结束，每次循环返回值的变化如何写(recursion step)。对于计算列表长度的情形来说，初始的参数应该是一个空表，因为它的长度为0，所以函数返回值也应该是0，没有更少的可能。每次执行时，我们都对这个列表进行一次去头操作，每成功完成一个去头操作，就说明这个表已经被证明有一个原子，而当这个表无法进行去头操作时，说明它已经是个空表，没有表头了，也就到了递归的出口，将经过每次执行改变的参数返回，即得到表的长度。
```clojure
(defn length [ls]
  (if (empty? ls)
    0
    (+ (length (rest ls)) 1)))

(length '())  ;=> 0
(length '(1))  ;=> 1
(length '(1 2))  ;=> 2
```

其中的if表达式询问了传入的列表是否为空，为空返回0。这就是这段递归函数的出口。否则将去头的表送进第二次执行，去头的表与之前的表，就是每次执行的改变，也就是步进。

我们再来写一个叫做list-copy的函数，它复制了它接收到的列表并返回这个新的列表。这意味着新的列表和老的列表是同样但不是同一的存在。当这个函数被使用时应该是这种样子。
```clojure
(list-copy '())  ;=> ()
(list-copy '(1 2 3))  ;=> (1 2 3)
```

在你看到下面的答案之前，不妨先自己想想该如何书写这样一个函数。
```clojure
(defn list-copy [ls]
  (if (empty? ls)
    '()
    (cons (first ls) (list-copy (rest ls)))))
```

list-copy的定义很像length的定义。判断出口的条件都是(empty? ls\)。只是遇见出口时的执行结果为()而不是0，因为我们构建的是一个新的列表，而不是一个数。函数的调用也是一样的，只是list-copy的参数多了一个而已。

如果你要问为什么递归函数只有一个出口，那是因为我们只写了一个出口。下面的函数memv接收两个参数，一个对象一个列表。它对表进行取头操作，当取出的表原子和对象值相等时，返回该表去头之后的结果。如果传入的表为空，则返回一个false，如果不相等，就对表去头再取头继续比较。
```clojure
(defn memv [x ls]
  (cond
    (empty? ls) false
    (= (first ls) x) ls
    :else (memv x (rest ls))))

(memv 1 '(1 2 3 4))  ;=> (1 2 3 4)
(memv 2 '(1 2 3 4))  ;=> (2 3 4)
(memv 5 '(1 2 3 4))  ;=> false
(memv 4 '(1 2 3 4))  ;=> (4)
(if (memv 2 '(1 2 3 4)) "y" "n")  ;=> "y"
```
对于上面的示例，通过cond，memv有两个出口。第一个出口是表为空时，返回假，第二个出口是遇见了相等情况时返回表。一旦条件不满足这两个出口，就必须向下执行。

除了出口之外，我们也可以有不同的入口。下面的remv有点像memv，这个函数也接收一个对象一个列表。它返回一个由与x值不相等的原子组成的新表。
```clojure
(defn remv [x ls]
  (cond 
    (empty? ls) '()
    (= (first ls) x) (remv x (rest ls))
    :else (cons (first ls) (remv x (rest ls)))))

(remv 1 '(1 2 2 4))  ;=> (2 2 4)
(remv 2 '(1 2 2 4))  ;=> (1 4)
(remv 3 '(1 2 2 4))  ;=> (1 2 2 4)
(remv 4 '(1 2 2 4))  ;=> (1 2 2)
```

这个函数的定义和之前的memv相比，当达成相等条件时，函数并没有退出递归，而是继续执行。而不满足相等条件时，只要表不为空，仍然是继续执行。这就相当于在递归的每一层，有两个入口可以进入不同的下一层。

现在，我们来接触一个新的函数，map。map接收一个函数，一个列表，将列表的参数逐一送入接收的函数中，就仿佛把函数映射到了每个表原子上。
```clojure
(map inc '(1 2 3 4 5))  ;=> (2 3 4 5 6)
```

inc函数可以将接收到的参数加1返回，在map表达式中，inc被泼撒到每个表原子上，所以得出了(2 3 4 5 6)这样的新表。

当然，我们也可以传入lambda表达式，比如对一个表的每个表原子进行平方。
```clojure
(map (fn [x] (* x x)) '(1 -3 5 -7))  ;=> (1 9 25 49)
```

当传入的函数需要多个参数的时候，map接收多个表
```clojure
(map + '(1 2 3) '(4 5 6))  ;=> (5 7 9)
(map + '(1 2 4) '(3 2 1) '(4 5 6))  ;=> (8 9 11)
```

对于第一个例子而言，分别是把两个表的对应元素相加，而第二个例子，则是三个表的对应元素相加。

假设，我们用递归自己定义一个最简单的map函数，该如何写呢？
```clojure
(defn map1 [f ls]
  (if (empty? ls)
    '()
    (cons (f (first ls)) (map1 f (rest ls)))))

(map1 inc '(1 2 3 4))  ;=> (2 3 4 5)
```

实际中的map必然不是如此定义的，但map1演示了map的递归流程表达。

### Exercise 2.9.1
定义一个make-list函数，它接收一个数字和一个列表，生成接收的数字个接收的列表，它的表现如下
```clojure
(make-list 3 '(1 2 3))  ;=> ((1 2 3) (1 2 3) (1 2 3))
```

### Exercise 2.9.2
有list-ref和list-tail两个函数，它们像下面这样
```clojure
(list-ref '(1 2 3 4) 0)  ;=> 1
(list-tail '(1 2 3 4) 0)  ;=> (1 2 3 4)
(list-ref '(a (nested) list) 1)  ;=> (nested)
(list-tail '(a (nested) list) 1)  ;=> ((nested) list)
```
他们同样接收一个列表和一个数字，分别返回第数字个表原子和第一个表原子是数字个原表原子的表。定义出这两个函数。
