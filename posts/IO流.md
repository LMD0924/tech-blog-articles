# IO流

### 字节流

**字节输入流：`InputStream`**

**字节输出流：`OutputStream`**

#### **FileOutputStream**

操作本地文件的字节输出流，可以把程序中的数据写到本地文件中。

步骤：

```Java
//①创建字节输出流对象
//FileOutputStream fos = new FileOutputStream(new File("文件地址"));
FileOutputStream fos = new FileOutputStream("文件地址");
//②写数据
fos.writer("写入的内容");
//③释放资源
fos.close();
```

补充：

​	换行写：\r\n

​	续写:FileOutputStream("文件地址",true);

#### **FileInputStram**

操作本地文件的字节输出流，可以把本地文件中的数据读取到程序中。

步骤：

```Java
//①创建字节输出流对象
FileInputSrteam fis = new FileInputStram("文件地址");
//②读取数据
fis.read();
//③释放资源
fis.close();
```

循环读取：

```java
int b;
while((b=fis.read())!=-1){
    System.out.println((char)b);
}
```

### 字符流

字符流的底层其实就是字节流

字符流=字节流+字符集

特点：

​	输入流：一次读取一个字节，遇到中文时一次读取多个字节。

​	输出流：底层会把数据按照指定的编码方式进行编码，变成字节再写到文件中。

**字符输入流：Reader**

**字符输出流：Writer**

#### FileReader

步骤：

```java
//①创建字符输入流对象
FileReader fr =new FileReader("文件地址");
//②读取数据
/**
read()细节：
read()：默认也是一个字节一个字节的读取，如果遇到中文就会一次读取多个字节
再读取后，方法的底层还会进行解码并转成十进制
最终把这个十进制作为返回值
这个十进制的数字也表示字符集上的数字
*/
int ch;
while((ch=fr.read())!=-1){
    System.out.println((char)ch);
}
//③释放资源
fr.close();
```

**有参read()**

```java
//创建对象
FileReader fr =new FileReader("文件地址");
//读取数据
char[] chars = new char[2];
int len;
//read(chars):读取数据，解码，强转三步合并了，把强转之后的字符放到数组当中
//空参的read+强转类型转换
while((len=fr.read(chars))!=-1){
    //把数组中的数据变成字符串在打印
    System.out.println(new String(chars,0,len));
}
//释放资源
fr.close();
```

#### FileWriter

**fileWriter书写细节**

①创建字符输出流对象

​	细节1：参数是字符串表示的路径或者File对象都是可以的

​	细节2：如果文件不存在创建一个新的文件，但是要保证父级路径是存在的

​	细节3：如果文件已经存在，则会清空文件，如果不想清空可以打开续写开关

②写数据：

​	细节：如果writer方法的参数是整数，但实际上写到本地文件中的是整数在字符集上对应的字符

③释放资源

​	细节：每次使用完流之后都要释放资源

**字符流原理解析**

①创建字符输入流对象

​	底层：关联文件，并创建缓冲区（长度为8192的字节数组）

②读取数据

​	底层：1.判断缓冲区是否有数据可以读取

​		   2.**缓冲区没有数据**：就从文件中获取，装到缓冲区中，每次尽可能装满缓冲区

​						   如果文件中也没数据，返回-1

​                   3.**缓冲区有数据：**就从缓冲区中读取。

​				空参的read方法：一次读取一个字节，遇到中文一次读多个字节，把字节解码并转成十进制返回

​				有参的read方法：把读取字节，解码，强转三步合一了，强转之后的字符放到数组中

### 编码&解码

#### 编码：

```Java
//编码
String str="ai你哟";
byte[] bytes1 = str.getBytes();
System.out.prontln(Arrays.toString(bytes1));

byte[] bytes2=str.getBytes("GBK");
System.out.println(Arrays.toString(bytes2));
```

#### 解码：

```java
//解码
Stirng str2=new String(bytes1);
System.out.println(str2);

String str3=new String(bytes1,"GBK");
System.out.println(str3);
```

### 缓冲流

#### BufferedInputStream$BufferedOutputStream

```
//创建缓冲流对象

```

