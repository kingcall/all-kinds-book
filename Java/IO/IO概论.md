我们从两个方面来理解Java IO，数据源（流）、数据传输，即IO的核心就是对数据源产生的数据进行读写并高效传输的过程。

## 一. 数据源（流）

数据源可以理解为水源，指可以产生数据的事物，如硬盘（文档、数据库等文件...）、网络（填写的form表单、物联感知信息..），在Java中有对文件及文件夹操作的类File，常用的文件方法如下：

```java
public static void printFileDetail(File file) throws IOException {
    System.out.println("文件是否存在：" + file.exists());
    if(!file.exists()){
        System.out.println("创建文件：" + file.getName());
        file.createNewFile();
    }
    if(file.exists()){
        System.out.println("是否为文件：" + file.isFile());
        System.out.println("是否为文件夹：" + file.isDirectory());
        System.out.println("文件名称：" + file.getName());
        System.out.println("文件构造路径：" + file.getPath());
        System.out.println("文件绝对路径：" + file.getAbsolutePath());
        System.out.println("文件标准路径：" + file.getCanonicalPath());
        System.out.println("文件大小：" + file.length());
        System.out.println("所在文件夹路径：" + file.getParentFile().getCanonicalPath());
        System.out.println("设置为只读文件：" + file.setReadOnly());
    }
}
public static void main(String[] args) throws IOException {
    File file = new File("./遮天.txt");
    printFileDetail(file);
}
```

结果如下：

```java
文件是否存在：false
创建文件：遮天.txt
是否为文件：true
是否为文件夹：false
文件名称：遮天.txt
文件构造路径：.\遮天.txt
文件绝对路径：E:\idea-work\javase-learning\.\遮天.txt
文件标准路径：E:\idea-work\javase-learning\遮天.txt
文件大小：0
所在文件夹路径：E:\idea-work\javase-learning
设置为只读文件：true
```

## 二. 数据传输

数据传输的核心在于传输数据源产生的数据，Java IO对此过程从两方面进行了考虑，分别为输入流和输出流，输入流完成外部数据向计算机内存写入，输出流则反之。

而针对输入流和输出流，Java IO又从字节和字符的不同，再次细分了字节流和字符流。
![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:16:03-1677914-20191124222522420-1813095470.png)

说明：Java中最小的计算单元是字节，没有字符流也能进行IO操作，只是因为现实中大量的数据都是文本字符数据，基于此单独设计了字符流，使操作更简便。

4个顶层接口有了，接下来Java IO又从多种应用场景（包括了基础数据类型、文件、数组、管道、打印、序列化）和传输效率（缓冲操作）进行了考虑，提供了种类众多的Java IO流的实现类，看下图：
![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/28/23:16:03-1677914-20191124222541240-1305367404.png)

当然我们不用都记住，而实际在使用过程中用的最多的还是文件类操作、转换类操作、序列化操作，当然在此基础上我们可以使用Buffered来提高效率（Java IO使用了装饰器模式）。下面我们通过文件拷贝来简单说明一下主要类的使用

```java
    /**
     * 文件拷贝（所有文件，文档、视频、音频、可执行文件...），未使用缓冲
     * @param sourceFileName 源文件路径
     * @param targetFileName 拷贝后目标文件路径
     * @throws IOException IO异常
     */
    public static void slowlyCopyFile(String sourceFileName, String targetFileName) throws IOException{
        //获取字节输入流
        FileInputStream fileInputStream = new FileInputStream(sourceFileName);
        //File targetFile = new File(targetFileName);
        //获取字节输出流
        FileOutputStream fileOutputStream = new FileOutputStream(targetFileName);
        byte[] bytes = new byte[1024];
        //当为-1时说明读取到最后一行了
        while ((fileInputStream.read(bytes)) != -1) {
            fileOutputStream.write(bytes);
        }
        fileInputStream.close();
        fileOutputStream.close();
    }
    
    /**
     * 文件拷贝（所有文件，文档、视频、音频、可执行文件...），使用缓冲
     * @param sourceFileName 源文件路径
     * @param targetFileName 拷贝后目标文件路径
     * @throws IOException IO异常
     */
    public static void fastCopyFile(String sourceFileName, String targetFileName) throws IOException{
        //获取字节输入流
        FileInputStream fileInputStream = new FileInputStream(sourceFileName);
        //缓冲字节输入流
        BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
        //获取字节输出流
        FileOutputStream fileOutputStream = new FileOutputStream(targetFileName);
        //缓冲字节输出流
        BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
        byte[] bytes = new byte[1024];

        //当为-1时说明读取到最后一行了
        while ((bufferedInputStream.read(bytes)) != -1) {
            bufferedOutputStream.write(bytes);
        }
        bufferedOutputStream.flush();
        bufferedInputStream.close();
        fileInputStream.close();
        bufferedOutputStream.close();
        fileOutputStream.close();
    }

    public static void main(String[] args) throws IOException {
        long startTime = System.currentTimeMillis();
        //文件215M
        slowlyCopyFile("D:\\Download\\jdk-8u221.exe","D:\\jdk-8u221.exe");//执行：1938ms
        fastCopyFile("D:\\Download\\jdk-8u221.exe","D:\\jdk-8u221.exe");//执行：490ms
        System.out.println(System.currentTimeMillis() - startTime);
    }
    /**
     * 文本文件拷贝，不使用缓冲
     * @param sourceFileName 源文件路径
     * @param targetFileName 拷贝后目标文件路径
     * @throws IOException IO异常
     */
    public static void slowlyCopyTextFile(String sourceFileName, String targetFileName) throws IOException {
        FileReader fileReader = new FileReader(sourceFileName);
        FileWriter fileWriter = new FileWriter(targetFileName);
        int c;
        while ((c = fileReader.read()) != -1) {
            fileWriter.write((char)c);
        }
        fileReader.close();
        fileWriter.close();
    }

    /**
     * 文本文件拷贝，使用缓冲
     * @param sourceFileName 源文件路径
     * @param targetFileName 拷贝后目标文件路径
     * @throws IOException IO异常
     */
    public static void fastCopyTextFile(String sourceFileName, String targetFileName) throws IOException {
        FileReader fileReader = new FileReader(sourceFileName);
        BufferedReader bufferedReader = new BufferedReader(fileReader);
        FileWriter fileWriter = new FileWriter(targetFileName);
        BufferedWriter bufferedWriter = new BufferedWriter(fileWriter);
        String str;
        while ((str = bufferedReader.readLine()) != null) {
            bufferedWriter.write(str + "\n");
        }
        bufferedReader.close();
        fileReader.close();
        bufferedWriter.close();
        fileWriter.close();
    }

    public static void main(String[] args) throws IOException {
        long startTime = System.currentTimeMillis();
        //文件30M
        slowlyCopyTextFile("D:\\Download\\小说合集.txt","D:\\小说合集.txt");//3182ms
        fastCopyTextFile("D:\\Download\\小说合集.txt","D:\\小说合集.txt");//1583ms
        System.out.println(System.currentTimeMillis() - startTime);
    }
```

## 三. 总结

本文主要对Java IO相关知识点做了结构性梳理，包括了Java IO的作用，数据源File类，输入流，输出流，字节流，字符流，以及缓冲流，不同场景下的更细化的流操作类型，同时用了一个文件拷贝代码简单地说明了主要的流操作，若有不对之处，请批评指正，望共同进步，谢谢！。