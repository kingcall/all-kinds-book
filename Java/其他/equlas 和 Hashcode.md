## hashcode 和 equals 

- hashCode的作用是用来获取哈希码，也可以称作散列码。实际返回值为一个int型数据。用于确定对象在哈希表中的位置。
  Object中有hashcode方法，也就意味着所有的类都有hashCode方法。
- 因为hashCode()并不是完全可靠，有时候不同的对象他们生成的hashcode也会一样（生成hash值得公式可能存在的问题），所以hashCode()只能说是大部分时候可靠，并不是绝对可靠，所以我们可以得出

1. equal()相等的两个对象他们的hashCode()肯定相等，也就是用equal()对比是绝对可靠的。
2. hashCode()相等的两个对象他们的equal()不一定相等，也就是hashCode()不是绝对可靠的。

- 所有对于需要大量并且快速的对比的话如果都用equal()去做显然效率太低，所以解决方式是，每当需要对比的时候，首先用hashCode()去对比，如果hashCode()不一样，则表示这两个对象肯定不相等（也就是不必再用equal()去再对比了）,如果hashCode()相同，此时再对比他们的equal()，如果equal()也相同，则表示这两个对象是真的相同了，这样既能大大提高了效率也保证了对比的绝对正确性！
- 这种大量的并且快速的对象对比一般使用的hash容器中，比如hashset,hashmap,hashtable等等，比如hashset里要求对象不能重复，则他内部必然要对添加进去的每个对象进行对比，而他的对比规则就是像上面说的那样，先hashCode()，如果hashCode()相同，再用equal()验证，如果hashCode()都不同，则肯定不同，这样对比的效率就很高了