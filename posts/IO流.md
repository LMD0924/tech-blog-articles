# 📁 Java IO流完全指南：从字节流到高级操作

## 🎯 前言
Java I/O流是处理输入输出的核心API，用于读写文件、网络通信等。掌握IO流是Java程序员的必备技能！让我们一起来系统学习吧~ 😊

## 📊 IO流体系总览

```
Java IO流
├── 按数据类型分
│   ├── 字节流（Byte Streams）
│   │   ├── InputStream（抽象类）
│   │   └── OutputStream（抽象类）
│   └── 字符流（Character Streams）
│       ├── Reader（抽象类）
│       └── Writer（抽象类）
├── 按功能分
│   ├── 节点流（直接操作数据源）
│   └── 处理流（包装节点流，增强功能）
└── 特殊流
    ├── 对象流（序列化）
    ├── 打印流
    ├── 转换流
    └── 压缩流
```

## 🔢 字节流（Byte Streams）

### **FileOutputStream** - 字节输出流 💾

```java
import java.io.FileOutputStream;
import java.io.IOException;

public class FileOutputStreamDemo {
    public static void main(String[] args) {
        // 方法1：直接使用文件路径
        FileOutputStream fos1 = null;
        // 方法2：使用File对象（更灵活）
        // FileOutputStream fos2 = new FileOutputStream(new File("test.txt"));
        
        try {
            // 1. 创建对象（会清空原文件内容）
            fos1 = new FileOutputStream("test.txt");
            
            // 2. 写入数据
            fos1.write(97);           // 写入字节 'a'
            fos1.write("\r\n".getBytes());  // 换行
            fos1.write("Hello Java!".getBytes());
            
            // 3. 续写模式（不会清空原内容）
            FileOutputStream fos2 = new FileOutputStream("test.txt", true);
            fos2.write("\n追加内容".getBytes());
            fos2.close();
            
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 4. 释放资源（必须执行）
            try {
                if (fos1 != null) {
                    fos1.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### 💡 注意事项：
- 换行符：Windows用 `\r\n`，Linux/Mac用 `\n`
- 路径不存在会自动创建，但父目录必须存在
- 使用 try-with-resources 更安全（Java 7+）

```java
// 推荐：使用try-with-resources自动关闭流
try (FileOutputStream fos = new FileOutputStream("test.txt")) {
    fos.write("自动关闭流，安全！".getBytes());
} catch (IOException e) {
    e.printStackTrace();
}
```

### **FileInputStream** - 字节输入流 📖

```java
import java.io.FileInputStream;
import java.io.IOException;

public class FileInputStreamDemo {
    public static void main(String[] args) {
        // 1. 单个字节读取（效率低）
        try (FileInputStream fis = new FileInputStream("test.txt")) {
            int b;
            while ((b = fis.read()) != -1) {
                System.out.print((char) b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 2. 字节数组读取（高效推荐）
        try (FileInputStream fis = new FileInputStream("test.txt")) {
            byte[] buffer = new byte[1024];  // 1KB缓冲区
            int len;
            while ((len = fis.read(buffer)) != -1) {
                System.out.println(new String(buffer, 0, len));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 3. 读取指定长度
        try (FileInputStream fis = new FileInputStream("test.txt")) {
            byte[] buffer = new byte[5];
            int len = fis.read(buffer, 0, 5);  // 最多读5个字节
            System.out.println("读取了 " + len + " 字节");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 🔤 字符流（Character Streams）

### **FileReader** - 字符输入流 📚

```java
import java.io.FileReader;
import java.io.IOException;

public class FileReaderDemo {
    public static void main(String[] args) {
        // 1. 单个字符读取
        try (FileReader fr = new FileReader("test.txt")) {
            int ch;
            while ((ch = fr.read()) != -1) {
                System.out.print((char) ch);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 2. 字符数组读取（推荐）
        try (FileReader fr = new FileReader("test.txt")) {
            char[] chars = new char[1024];
            int len;
            while ((len = fr.read(chars)) != -1) {
                System.out.print(new String(chars, 0, len));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 3. 读取到指定数组位置
        try (FileReader fr = new FileReader("test.txt")) {
            char[] chars = new char[10];
            int len = fr.read(chars, 2, 5);  // 从数组索引2开始存，最多读5个
            System.out.println("实际读取字符数：" + len);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### **FileWriter** - 字符输出流 📝

```java
import java.io.FileWriter;
import java.io.IOException;

public class FileWriterDemo {
    public static void main(String[] args) {
        // 1. 基本写入
        try (FileWriter fw = new FileWriter("output.txt")) {
            fw.write("Hello World!\n");
            fw.write(65);  // 写入字符 'A'
            fw.write(new char[]{'B', 'C', 'D'});
            fw.write("Java编程", 0, 4);  // 写入"Java"
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 2. 追加模式
        try (FileWriter fw = new FileWriter("output.txt", true)) {
            fw.write("\n这是追加的内容");
            fw.flush();  // 刷新缓冲区，立即写入
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 3. 写入字符数组的一部分
        try (FileWriter fw = new FileWriter("output.txt")) {
            char[] data = {'J', 'a', 'v', 'a', '编', '程'};
            fw.write(data, 0, 4);  // 只写入"Java"
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 🔄 编码与解码详解

### **编码过程** 🔡→🔢

```java
import java.util.Arrays;

public class EncodingDemo {
    public static void main(String[] args) throws Exception {
        String str = "ai你哟❤";
        
        System.out.println("=== 不同编码方式比较 ===");
        
        // 1. 平台默认编码（通常是UTF-8）
        byte[] defaultBytes = str.getBytes();
        System.out.println("默认编码（" + System.getProperty("file.encoding") + "）：");
        System.out.println(Arrays.toString(defaultBytes));
        System.out.println("字节数：" + defaultBytes.length);
        
        // 2. UTF-8编码（变长，中文3-4字节）
        byte[] utf8Bytes = str.getBytes("UTF-8");
        System.out.println("\nUTF-8编码：");
        System.out.println(Arrays.toString(utf8Bytes));
        System.out.println("字节数：" + utf8Bytes.length);
        
        // 3. GBK编码（中文2字节）
        byte[] gbkBytes = str.getBytes("GBK");
        System.out.println("\nGBK编码：");
        System.out.println(Arrays.toString(gbkBytes));
        System.out.println("字节数：" + gbkBytes.length);
        
        // 4. ISO-8859-1（不支持中文，会丢失信息）
        byte[] isoBytes = str.getBytes("ISO-8859-1");
        System.out.println("\nISO-8859-1编码（不支持中文）：");
        System.out.println(Arrays.toString(isoBytes));
        System.out.println("解码后：" + new String(isoBytes, "ISO-8859-1"));
    }
}
```

### **解码过程** 🔢→🔡

```java
public class DecodingDemo {
    public static void main(String[] args) throws Exception {
        String original = "你好Java🎉";
        
        // 1. 正确解码（编码解码一致）
        byte[] utf8Bytes = original.getBytes("UTF-8");
        String correct = new String(utf8Bytes, "UTF-8");
        System.out.println("正确解码：" + correct);
        
        // 2. 错误解码（编码解码不一致）
        byte[] gbkBytes = original.getBytes("GBK");
        String wrong1 = new String(gbkBytes, "UTF-8");
        System.out.println("错误解码（GBK→UTF-8）：" + wrong1);
        
        // 3. 恢复乱码
        byte[] wrongBytes = wrong1.getBytes("UTF-8");
        String recovered = new String(wrongBytes, "GBK");
        System.out.println("恢复后：" + recovered);
    }
}
```

## ⚡ 缓冲流（高效流）

### **字节缓冲流** 🚀

```java
import java.io.*;

public class BufferedStreamDemo {
    public static void main(String[] args) {
        String srcFile = "largefile.mp4";
        String destFile = "copy.mp4";
        
        // 1. 基本拷贝（单个字节，慢！）
        long start1 = System.currentTimeMillis();
        try (FileInputStream fis = new FileInputStream(srcFile);
             FileOutputStream fos = new FileOutputStream("copy1.mp4")) {
            int b;
            while ((b = fis.read()) != -1) {
                fos.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long end1 = System.currentTimeMillis();
        System.out.println("基本拷贝耗时：" + (end1 - start1) + "ms");
        
        // 2. 缓冲流拷贝（高效）
        long start2 = System.currentTimeMillis();
        try (BufferedInputStream bis = new BufferedInputStream(
                new FileInputStream(srcFile));
             BufferedOutputStream bos = new BufferedOutputStream(
                new FileOutputStream("copy2.mp4"))) {
            int b;
            while ((b = bis.read()) != -1) {
                bos.write(b);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long end2 = System.currentTimeMillis();
        System.out.println("缓冲流拷贝耗时：" + (end2 - start2) + "ms");
        
        // 3. 缓冲流+数组（最快）
        long start3 = System.currentTimeMillis();
        try (BufferedInputStream bis = new BufferedInputStream(
                new FileInputStream(srcFile));
             BufferedOutputStream bos = new BufferedOutputStream(
                new FileOutputStream("copy3.mp4"))) {
            byte[] buffer = new byte[8192];  // 8KB缓冲区
            int len;
            while ((len = bis.read(buffer)) != -1) {
                bos.write(buffer, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long end3 = System.currentTimeMillis();
        System.out.println("缓冲流+数组拷贝耗时：" + (end3 - start3) + "ms");
    }
}
```

### **字符缓冲流** 📚

```java
import java.io.*;

public class BufferedReaderWriterDemo {
    public static void main(String[] args) {
        // 1. BufferedReader - 按行读取
        try (BufferedReader br = new BufferedReader(
                new FileReader("test.txt"))) {
            String line;
            int lineNumber = 1;
            while ((line = br.readLine()) != null) {
                System.out.println(lineNumber++ + ": " + line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 2. BufferedWriter - 高效写入
        try (BufferedWriter bw = new BufferedWriter(
                new FileWriter("output.txt"))) {
            bw.write("第一行");
            bw.newLine();  // 跨平台换行
            bw.write("第二行");
            bw.flush();  // 立即写入
            
            // 批量写入
            String[] lines = {"第三行", "第四行", "第五行"};
            for (String l : lines) {
                bw.write(l);
                bw.newLine();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 3. 标准输入读取
        try (BufferedReader consoleReader = new BufferedReader(
                new InputStreamReader(System.in))) {
            System.out.print("请输入：");
            String input = consoleReader.readLine();
            System.out.println("您输入的是：" + input);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 🔀 转换流（InputStreamReader/OutputStreamWriter）

```java
import java.io.*;

public class ConvertStreamDemo {
    public static void main(String[] args) throws Exception {
        // 1. 读取GBK编码文件，转换为UTF-8
        try (InputStreamReader isr = new InputStreamReader(
                new FileInputStream("gbkfile.txt"), "GBK");
             OutputStreamWriter osw = new OutputStreamWriter(
                new FileOutputStream("utf8file.txt"), "UTF-8")) {
            
            char[] buffer = new char[1024];
            int len;
            while ((len = isr.read(buffer)) != -1) {
                osw.write(buffer, 0, len);
            }
        }
        
        // 2. 控制台指定编码读取
        System.out.println("请输入中文：");
        try (BufferedReader br = new BufferedReader(
                new InputStreamReader(System.in, "GBK"))) {
            String input = br.readLine();
            System.out.println("您输入的是：" + input);
            
            // 以UTF-8保存
            try (OutputStreamWriter osw = new OutputStreamWriter(
                    new FileOutputStream("input.txt"), "UTF-8")) {
                osw.write(input);
            }
        }
        
        // 3. 文件编码检测和转换
        convertFileEncoding("source.txt", "GBK", "target.txt", "UTF-8");
    }
    
    // 通用编码转换方法
    public static void convertFileEncoding(String srcFile, String srcCharset,
                                          String destFile, String destCharset) 
                                          throws IOException {
        try (InputStreamReader isr = new InputStreamReader(
                new FileInputStream(srcFile), srcCharset);
             OutputStreamWriter osw = new OutputStreamWriter(
                new FileOutputStream(destFile), destCharset)) {
            
            char[] buffer = new char[4096];
            int len;
            while ((len = isr.read(buffer)) != -1) {
                osw.write(buffer, 0, len);
            }
        }
    }
}
```

## 💾 序列化流（ObjectInputStream/ObjectOutputStream）

```java
import java.io.*;
import java.util.Date;

// 必须实现Serializable接口
class User implements Serializable {
    // 序列化版本ID，防止类修改后反序列化失败
    private static final long serialVersionUID = 1L;
    
    private String name;
    private transient String password;  // transient修饰的字段不序列化
    private int age;
    private Date birthday;
    
    // 构造方法、getter/setter省略...
    
    // 自定义序列化过程（可选）
    private void writeObject(ObjectOutputStream oos) throws IOException {
        oos.defaultWriteObject();  // 默认序列化
        // 自定义序列化逻辑
    }
    
    private void readObject(ObjectInputStream ois) 
            throws IOException, ClassNotFoundException {
        ois.defaultReadObject();  // 默认反序列化
        // 自定义反序列化逻辑
    }
}

public class SerializationDemo {
    public static void main(String[] args) {
        User user = new User("张三", "123456", 25, new Date());
        
        // 1. 序列化（对象→字节）
        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream("user.dat"))) {
            oos.writeObject(user);
            oos.writeInt(100);  // 可以写入基本类型
            oos.writeObject(new Date());
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 2. 反序列化（字节→对象）
        try (ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream("user.dat"))) {
            User readUser = (User) ois.readObject();
            int num = ois.readInt();
            Date date = (Date) ois.readObject();
            
            System.out.println("用户名：" + readUser.getName());
            System.out.println("密码：" + readUser.getPassword());  // null
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
        
        // 3. 序列化集合
        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream("list.dat"))) {
            java.util.List<User> users = new java.util.ArrayList<>();
            users.add(new User("李四", "111", 30, new Date()));
            users.add(new User("王五", "222", 28, new Date()));
            oos.writeObject(users);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 🖨️ 打印流（PrintStream/PrintWriter）

```java
import java.io.*;
import java.util.Date;

public class PrintStreamDemo {
    public static void main(String[] args) throws Exception {
        // 1. PrintStream（字节打印流）
        try (PrintStream ps = new PrintStream("print.txt")) {
            ps.println("Hello PrintStream!");
            ps.printf("当前时间：%tF %tT%n", new Date(), new Date());
            ps.print("不换行");
            ps.println("接着写");
            
            // 不会抛出异常，用checkError()检查
            ps.write(65);  // 'A'
            if (ps.checkError()) {
                System.err.println("写入出错！");
            }
        }
        
        // 2. PrintWriter（字符打印流，更常用）
        try (PrintWriter pw = new PrintWriter(
                new FileWriter("print2.txt"), true)) {  // true表示自动刷新
            
            pw.println("=== 学生信息 ===");
            pw.printf("姓名：%-10s 年龄：%d%n", "张三", 20);
            pw.printf("分数：%.2f%n", 95.5);
            pw.println("==============");
            
            // 支持链式调用
            pw.append("追加内容")
              .append("\n")
              .append("另一行");
        }
        
        // 3. 重定向System.out
        try (PrintStream fileOut = new PrintStream("system_out.txt")) {
            PrintStream originalOut = System.out;
            System.setOut(fileOut);  // 重定向
            
            System.out.println("这行会写到文件里");
            System.out.println("控制台看不到我");
            
            System.setOut(originalOut);  // 恢复
            System.out.println("回到控制台");
        }
        
        // 4. 格式化输出
        try (PrintWriter pw = new PrintWriter("format.txt")) {
            pw.printf("商品\t单价\t数量\t小计%n");
            pw.printf("%-10s\t%8.2f\t%4d\t%8.2f%n", 
                     "Java书", 89.50, 2, 179.00);
            pw.printf("%-10s\t%8.2f\t%4d\t%8.2f%n",
                     "鼠标", 150.00, 1, 150.00);
            pw.printf("%-10s\t%8s\t%4s\t%8.2f%n",
                     "", "", "合计", 329.00);
        }
    }
}
```

## 📦 压缩与解压缩流

```java
import java.io.*;
import java.util.zip.*;
import java.util.Enumeration;

public class ZipDemo {
    public static void main(String[] args) {
        // 1. 单个文件压缩
        try (FileInputStream fis = new FileInputStream("test.txt");
             FileOutputStream fos = new FileOutputStream("test.zip");
             ZipOutputStream zos = new ZipOutputStream(fos)) {
            
            ZipEntry entry = new ZipEntry("test.txt");
            zos.putNextEntry(entry);
            
            byte[] buffer = new byte[1024];
            int len;
            while ((len = fis.read(buffer)) != -1) {
                zos.write(buffer, 0, len);
            }
            zos.closeEntry();
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 2. 多个文件压缩
        String[] files = {"file1.txt", "file2.txt", "file3.txt"};
        try (ZipOutputStream zos = new ZipOutputStream(
                new FileOutputStream("multi.zip"))) {
            
            for (String file : files) {
                try (FileInputStream fis = new FileInputStream(file)) {
                    ZipEntry entry = new ZipEntry(file);
                    zos.putNextEntry(entry);
                    
                    byte[] buffer = new byte[1024];
                    int len;
                    while ((len = fis.read(buffer)) != -1) {
                        zos.write(buffer, 0, len);
                    }
                    zos.closeEntry();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 3. 解压缩
        try (ZipInputStream zis = new ZipInputStream(
                new FileInputStream("multi.zip"))) {
            
            ZipEntry entry;
            while ((entry = zis.getNextEntry()) != null) {
                System.out.println("解压: " + entry.getName());
                
                // 创建目录结构
                File file = new File("extracted/" + entry.getName());
                if (entry.isDirectory()) {
                    file.mkdirs();
                } else {
                    file.getParentFile().mkdirs();
                    
                    try (FileOutputStream fos = new FileOutputStream(file)) {
                        byte[] buffer = new byte[1024];
                        int len;
                        while ((len = zis.read(buffer)) != -1) {
                            fos.write(buffer, 0, len);
                        }
                    }
                }
                zis.closeEntry();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        // 4. 使用ZipFile类（更方便）
        try (ZipFile zipFile = new ZipFile("multi.zip")) {
            Enumeration<? extends ZipEntry> entries = zipFile.entries();
            
            while (entries.hasMoreElements()) {
                ZipEntry entry = entries.nextElement();
                System.out.println("文件: " + entry.getName() + 
                                 " 大小: " + entry.getSize());
                
                if (!entry.isDirectory()) {
                    try (InputStream is = zipFile.getInputStream(entry)) {
                        // 处理文件内容
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 🔧 Apache Commons IO 工具库

### **添加依赖** 📦
```xml
<!-- Maven -->
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.13.0</version>
</dependency>
```

### **常用工具类** 🛠️

```java
import org.apache.commons.io.FileUtils;
import org.apache.commons.io.IOUtils;
import org.apache.commons.io.FilenameUtils;
import org.apache.commons.io.FileExistsUtils;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.List;

public class CommonsIODemo {
    public static void main(String[] args) throws Exception {
        // 1. FileUtils - 文件操作神器
        File src = new File("source.txt");
        File dest = new File("backup.txt");
        
        // 复制文件
        FileUtils.copyFile(src, dest);
        FileUtils.copyFile(src, dest, true);  // 强制覆盖
        
        // 复制目录
        FileUtils.copyDirectory(new File("srcDir"), new File("destDir"));
        
        // 读取文件内容
        String content = FileUtils.readFileToString(src, "UTF-8");
        List<String> lines = FileUtils.readLines(src, "UTF-8");
        
        // 写入文件
        FileUtils.writeStringToFile(dest, "Hello Commons IO", "UTF-8");
        FileUtils.writeLines(dest, "UTF-8", lines, true);  // true表示追加
        
        // 删除目录（包括子目录）
        FileUtils.deleteDirectory(new File("tempDir"));
        FileUtils.forceDelete(new File("file.txt"));  // 强制删除
        
        // 清空目录
        FileUtils.cleanDirectory(new File("cache"));
        
        // 2. IOUtils - 流操作简化
        try (InputStream is = new FileInputStream("input.txt");
             OutputStream os = new FileOutputStream("output.txt")) {
            
            // 复制流（自动处理缓冲和关闭）
            IOUtils.copy(is, os);
            
            // 读取全部字节
            byte[] bytes = IOUtils.toByteArray(is);
            
            // 读取为字符串
            String text = IOUtils.toString(is, "UTF-8");
            
            // 写入字符串到流
            IOUtils.write("Hello", os, "UTF-8");
            
            // 关闭多个流
            IOUtils.closeQuietly(is, os);  // 静默关闭，不抛异常
            
            // 3. 资源复制（自动关闭）
            try (InputStream in = new FileInputStream("a.txt");
                 OutputStream out = new FileOutputStream("b.txt")) {
                IOUtils.copy(in, out);
            }
        }
        
        // 3. FilenameUtils - 文件名处理
        String filename = "C:/test/file.txt";
        
        System.out.println("扩展名: " + FilenameUtils.getExtension(filename));
        System.out.println("文件名（无扩展名）: " + FilenameUtils.getBaseName(filename));
        System.out.println("完整文件名: " + FilenameUtils.getName(filename));
        System.out.println("路径: " + FilenameUtils.getFullPath(filename));
        System.out.println("规范路径: " + FilenameUtils.normalize(filename));
        
        // 路径比较
        boolean same = FilenameUtils.equals(
            "C:/test/../test/file.txt", 
            "C:/test/file.txt");  // true
        
        // 4. FileSystemUtils - 文件系统工具
        long freeSpace = FileSystemUtils.freeSpaceKb("C:");
        System.out.println("C盘剩余空间: " + freeSpace + "KB");
        
        // 5. 监控文件变化
        // FileAlterationMonitor monitor = new FileAlterationMonitor(1000);
        // FileAlterationObserver observer = new FileAlterationObserver("watchDir");
        // observer.addListener(new FileAlterationListenerAdaptor() {
        //     public void onFileCreate(File file) {
        //         System.out.println("创建: " + file.getName());
        //     }
        // });
        // monitor.addObserver(observer);
        // monitor.start();
    }
}
```

## 🎯 最佳实践总结

### **1. 选择正确的流类型**
| 场景       | 推荐流                           | 说明                   |
| ---------- | -------------------------------- | ---------------------- |
| 文本文件   | FileReader/FileWriter            | 字符流处理文本更合适   |
| 二进制文件 | FileInputStream/FileOutputStream | 图片、视频等用字节流   |
| 需要高效   | BufferedXXX                      | 总是使用缓冲流提升性能 |
| 网络传输   | 字节流                           | 网络传输都是字节       |

### **2. 资源管理规范**
```java
// ❌ 错误：忘记关闭流
FileInputStream fis = new FileInputStream("file.txt");
// 处理数据...
// fis.close();  // 忘记调用

// ✅ 正确1：try-with-resources（Java 7+）
try (FileInputStream fis = new FileInputStream("file.txt");
     FileOutputStream fos = new FileOutputStream("output.txt")) {
    // 自动关闭
}

// ✅ 正确2：传统try-catch-finally
FileInputStream fis = null;
try {
    fis = new FileInputStream("file.txt");
    // 处理数据
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (fis != null) {
        try {
            fis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### **3. 性能优化建议**
```java
// 缓冲区大小选择（经验值）
byte[] buffer = new byte[8192];  // 8KB - 适合大多数场景
// 或
char[] charBuffer = new char[8192];

// 批量操作优于单字节操作
while ((len = input.read(buffer)) != -1) {
    output.write(buffer, 0, len);  // 批量写入
}

// 使用NIO（大文件或高并发）
Path source = Paths.get("largefile.iso");
Path target = Paths.get("copy.iso");
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
```

### **4. 编码处理黄金法则**
```java
// 原则：编码解码必须一致
String text = "你好世界";

// 保存时指定编码
try (OutputStreamWriter osw = new OutputStreamWriter(
        new FileOutputStream("file.txt"), StandardCharsets.UTF_8)) {
    osw.write(text);
}

// 读取时使用相同编码
try (InputStreamReader isr = new InputStreamReader(
        new FileInputStream("file.txt"), StandardCharsets.UTF_8)) {
    // 读取...
}

// 网页处理使用UTF-8
response.setCharacterEncoding("UTF-8");
request.setCharacterEncoding("UTF-8");
```

## 🚀 快速参考卡

| 操作         | 字节流             | 字符流              | 缓冲流                      | Commons IO                      |
| ------------ | ------------------ | ------------------- | --------------------------- | ------------------------------- |
| **读取文件** | `FileInputStream`  | `FileReader`        | `BufferedReader`            | `FileUtils.readFileToString()`  |
| **写入文件** | `FileOutputStream` | `FileWriter`        | `BufferedWriter`            | `FileUtils.writeStringToFile()` |
| **复制文件** | 手动循环           | 手动循环            | 缓冲+数组                   | `FileUtils.copyFile()`          |
| **读取行**   | N/A                | `readLine()`        | `BufferedReader.readLine()` | `FileUtils.readLines()`         |
| **处理编码** | 字节数组+String    | `InputStreamReader` | 指定Charset                 | 自动UTF-8                       |
| **关闭资源** | try-with-resources | try-with-resources  | try-with-resources          | 自动管理                        |

## 📝 常见问题解答

**Q: 字节流和字符流怎么选择？**
A: 文本用字符流（Reader/Writer），二进制数据用字节流（InputStream/OutputStream）

**Q: 为什么用缓冲流？**
A: 减少物理读写次数，提升性能几十到几百倍！

**Q: 如何处理大文件？**
A: 使用缓冲流+适当缓冲区，或考虑NIO的FileChannel

**Q: 中文乱码怎么解决？**
A: 确保编码一致，推荐统一使用UTF-8

**Q: 流为什么要关闭？**
A: 释放系统资源，避免内存泄漏和文件锁定

希望这篇全面的IO流指南能帮助你！编程愉快！ 🚀💻