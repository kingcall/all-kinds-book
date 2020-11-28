## TreeMap

TreeMap实现SortedMap接口，能够把它保存的记录根据键排序，默认是按键值的**升序排序**，也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。



如果使用排序的映射，建议使用TreeMap。在使用TreeMap时，key必须实现Comparable接口或者在构造TreeMap传入自定义的Comparator，否则会在运行时抛出java.lang.ClassCastException类型的异常。



TreeMap中的元素默认按照keys的自然排序排列（对Integer来说，其自然排序就是数字的升序；对String来说，其自然排序就是按照字母表排序）



