```
    def main(args: Array[String]): Unit = {
      val data = spark.read
        .option("inferSchema", "false")
        .load("data/spark/sql/sql 函数/json.txt")
        .toDF()
      data.show()
      data.printSchema()
    }
```



![image-20201218225111149](/Users/liuwenqiang/Library/Application%20Support/typora-user-images/image-20201218225111149.png)

