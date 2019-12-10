## Java中数组复制的几种方式

* 1.Object.clone()

简单直接，只能对源数组完整地复制

* 2.Arrays.copyOf(T[] original, int newLength)

可以只复制源数组中部分元素，但复制的起始位置固定为0，基于System.arraycopy()。

* 3.Arrays.copyOfRange(T[] original, int from, int to)

可以指定复制的起始位置，基于System.arraycopy()。

* 4.System.arraycopy(Object src, int srcPos, Object dest, int destPos, int length)

复杂，但可以将源数组中的部分元素复制到目标数组的指定位置（此方法最灵活，可实现上述1、2、3的功能）。


**总结：System.arraycopy() 为native原生方法，效率高，最先考虑使用。1、2、3都具有一定的局限性（返回一个新的数组，无法将源数组中的元素复制到已存在的数组中）。**


