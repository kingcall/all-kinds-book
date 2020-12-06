- Structs： structs内部的数据可以通过DOT（.）来存取，例如，表中一列c的类型为STRUCT{a INT; b INT}，我们可以通过c.a来访问域a
Maps（K-V对）：访问指定域可以通过["指定域名称"]进行，例如，一个Map M包含了一个group->gid的kv对，gid的值可以通过M['group']来获取
Arrays：array中的数据为相同类型，例如，假如array A中元素['a','b','c']，则A[1]的值为'b'
