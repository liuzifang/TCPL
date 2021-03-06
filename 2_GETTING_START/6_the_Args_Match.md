# Section 2.6. 参数匹配

在上一章中，我们看过了简单的参数传入函数的形式，但在实际开发中，还存在着参数匹配这一情形，就像是python中的元组赋值，Haskell和Erlang的模式匹配一样。

```clojure
(let [f (fn [a [b c]] (list a b c))]
  (f 1 [3 5]))
```

你能猜出这个表达式将返回的值是什么么？如果你见识过很多语言，你应该能猜出来，答案是(1 3 5)。让我们先回答这样一个问题，lambda表达式返回的函数接收几个参数？是三个还是两个？

答案是两个，一个是a，一个是由[b c]，也就是b和c组成的矢量。那么，在我们调用f的时候，我们传入了几个参数呢？是三个还是两个？

答案也是两个，1和[3 5]。回忆一下最简单的参数传入形式，是不是[var var1 ...]，而我们放入函数中的参数则如(#<fun> arg arg1 ...)将arg与var一一对应绑定？[a [b c]]与[1 [3 5]]也是一样。现将1与a绑定,然后是[b c],[3 5]，而后者因为结构的关系再次重复了第一步b,3,c,5一一对应绑定。这种更复杂的传参形式其实和最简单的形式没有什么区别，只是又多了一步匹配过程而已。

再来看几个例子
```clojure
(let [f (fn [[a [b c]] d] (list a b c d))]
  (f [2 [1 3]] 4))  ;=> (2 1 3 4)

(let [f (fn [a & [b & c]] (list a b c))]
  (f 1 2 3 4 5))  ;=> (1 2 (3 4 5))

(let [f (fn [[x {a :pasa b :pasb} [y z]]] (list a b x y z))]
  (f [1 {:pasb 12 :pasa 13} [3 2]]))  ;=> (13 12 1 3 2)
```

第一个例子并没由什么好说的，无非是嵌套层更多而已。在第二个例子中，a理所当然的绑定上了1。而我们在之前的章节中说过\&之前的变量会一一对应传入的参数而绑定，\&后的变量将绑定上剩下参数组成的一个列表中，所以[b \& c]就绑定上了[2 3 4 5]，根据之前学的东西，之后的匹配过程就很简单了吧？b绑定上2，c绑定上(3 4 5)，结果就理所当然的是(1 2 (3 4 5))。不过要注意一点，实际情况或许有细微差异，比如[]和()，只有[]才是真的可以匹配，而\&后接收的参数是作为()绑定的，但它们偏偏就匹配了，这是一种约定优化，是没有道理却又合情合理的一种情况。

有了第二个例子的讲解，第三个例子也就容易理解了很多。lambda只有接收一个变量，所以我们传入的这个矢量就匹配并绑定了这唯一的变量，接着我们进行第二层匹配，1匹配绑定上x，两个映射的指定匹配我们之前说过，再次不再赘述，b匹配上12，a匹配上13，[y z]匹配上[3 2],于是在下一层匹配中，y匹配3，z匹配2。

你应该在开发中适当的使用这些匹配技巧，而不是不闻不问或是滥用。除了lambda表达式之外，你还能想到别的表达式或许也拥有这种匹配形式么？

我们说过也强调过let表达式和lambda表达式之间的关系，以它们的亲密程度，let表达式理所当然的在用来绑定的[]中和lambda有完全一致的效果。有lambda表达式的例子在上，对于let就不再重复那些换汤不换药的例子了，你可以在后面的习题中自行揣摩。

### Exercise 2.6.1
设想下面表达式的值，并在repl中验证自己的答案

```clojure
(let [[{a :pasa b :pasb} [y z] & o] [{:pasb 1 :pasa 2} [11 22] 5 [6 7] 8] 
      f (fn [[x y] & [{a :pasa b :pasb} & z]]
          (cons (list a b z y) o))]
  (f [a b] {:pasa z :pasb y} o))
```
