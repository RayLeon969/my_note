# Socket概述

​	Java的网络编程主要涉及到的内容是Socket编程。Socket，套接字，就是两台主机之间逻辑连接的端点。TCP/IP协议是传输层协议，主要解决数据如何在网络中传输，而HTTP是应用层协议，主要解决如何包装数据。Socket是通信的基石，是支持TCP/IP协议的网络通信的基本操作单元。它是网络通信过程中端点的抽象表示，包含进行网络通信必须的五种信息：连接使用的协议、本地主机的IP地址、本地进程的协议端口、远程主机的IP地址、远程进程的协议端口。

　　应用层通过传输层进行数据通信时，TCP会遇到同时为多个应用程序进程提供并发服务的问题。多个TCP连接或多个应用程序进程可能需要通过同一个TCP协议端口传输数据。为了区别不同的应用程序进程和连接，许多计算机操作系统为应用程序与TCP/IP协议交互提供了套接字（Socket）接口。应用层可以和传输层通过Socket接口，区分来自不同应用程序进程或网络连接的通信，实现数据传输的并发服务。

　　Socket，实际上是对TCP/IP协议的封装，Socket本身并不是协议，而是一个调用接口（API），通过Socket，我们才能使用TCP/IP协议。实际上，Socket跟TCP/IP协议没有必然的关系，Socket编程接口在设计的时候，就希望也能适应其他的网络协议。所以说，Socket的出现，只是使得程序员更方便地使用TCP/IP协议栈而已，是对TCP/IP协议的抽象，从而形成了我们知道的一些最基本的函数接口，比如create、listen、accept、send、read和write等等。网络有一段关于socket和TCP/IP协议关系的说法比较容易理解：

　　“TCP/IP只是一个协议栈，就像操作系统的运行机制一样，必须要具体实现，同时还要提供对外的操作接口。这个就像操作系统会提供标准的编程接口，比如win32编程接口一样，TCP/IP也要提供可供程序员做网络开发所用的接口，这就是Socket编程接口。” 

 　  实际上，传输层的TCP是基于网络层的IP协议的，而应用层的HTTP协议又是基于传输层的TCP协议的，而Socket本身不算是协议，就像上面所说，它只是提供了一个针对TCP或者UDP编程的接口。socket是对端口通信开发的工具,它要更底层一些。



# Socket整体流程

​	Socket编程主要涉及到客户端和服务端两个方面，首先是在服务器端创建一个服务器套接字（ServerSocket），并把它附加到一个端口上，服务器从这个端口监听连接。端口号的范围是0到65536，但是0到1024是为特权服务保留的端口号，我们可以选择任意一个当前没有被其他进程使用的端口。

　　客户端请求与服务器进行连接的时候，根据服务器的域名或者IP地址，加上端口号，打开一个套接字。当服务器接受连接后，服务器和客户端之间的通信就像输入输出流一样进行操作。

![image-20231116151605492](assets\image-20231116151605492.png)

```java
//服务端
public class TcpServerDemo1 {
    public static void main(String[] args) {


        ServerSocket serverSocket = null;
        Socket accept = null;
        InputStream is = null;
        ByteArrayOutputStream baos = null;
        try {
            //1.我得有一个地址 此时就是localhost：9999
            serverSocket = new ServerSocket(9999);
            while(true){
                //2.等待客户端链接过来
                accept = serverSocket.accept();
                //3.读取客户端的消息
                is = accept.getInputStream();
                //管道流
                baos = new ByteArrayOutputStream();
                byte[] buffer = new byte[1024];
                int len;
                while((len=is.read(buffer))!=-1){
                    baos.write(buffer,0,len);
                    baos.flush();
                }
                System.out.println(baos.toString());
            }


        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //关闭资源
            try {
                baos.close();
                is.close();
                accept.close();
                serverSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }


        }
    }




}
```

```java
//客户端
public class TcpClientDemo1 {
    public static void main(String[] args) {
        Socket socket = null;
        OutputStream os = null;
        try {
            //1.要知道服务器的地址
            InetAddress serverIP = InetAddress.getByName("127.0.0.1");
            //2.端口号
            int port = 9999;
            //3.创建一个socket连接
            socket = new Socket(serverIP,port);
            //4.发送消息
            os = socket.getOutputStream();
            os.write("你好，欢迎学习Java网络编程".getBytes());




        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                os.close();
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }


}
```



# 关于TCP实现文件上传

客户端

```java
package com.internet;


import java.io.*;
import java.net.InetAddress;
import java.net.Socket;


public class TcpClientFile {


    public static void main(String[] args) throws Exception{
        //创建一个Socket连接
        Socket socket = new Socket(InetAddress.getByName("localhost"),9000);
        //获取输出流
        OutputStream os = socket.getOutputStream();


        //将文件读入输入流
        FileInputStream fis = new FileInputStream(new File("ZMY.JPG"));
        byte[] buffer = new byte[1024];
        int len;
        while ((len=fis.read(buffer))!=-1){
            os.write(buffer,0,len);
            os.flush();
        }




        //通知服务器，我已经结束了
        socket.shutdownOutput();


        //确定服务器接收完毕才能断开连接
        InputStream is = socket.getInputStream();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        byte[] buffer1 = new byte[1024];
        int len1;
        while ((len1=is.read(buffer))!=-1){
            baos.write(buffer1,0,len);
        }
        System.out.println(baos.toString());




        fis.close();
        os.close();
        socket.close();




    }
}
```

服务器端

```java
package com.internet;


import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;


//TCP实现文件传输
public class TcpServerFile {


    public static void main(String[] args) throws Exception{
        //1.创建服务
        ServerSocket serverSocket = new ServerSocket(9000);


        //2.客户端监听
        Socket accept = serverSocket.accept();


        //3.获取输入流
        InputStream is = accept.getInputStream();


        //4.文件输出
        FileOutputStream fos = new FileOutputStream(new File("recieve.jpg"));
        byte[] buffer = new byte[1024];
        int len;
        while((len=is.read(buffer))!=-1){
            fos.write(buffer,0,len);
            fos.flush();
        }


        //告诉客户端我接收完毕了
        OutputStream os = accept.getOutputStream();
        os.write("我接受完毕".getBytes());


        fos.close();
        is.close();
        accept.close();
        serverSocket.close();
    }
}
```



# 关于UDP进行消息传输

发送方

```java
package com.internet;


import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;


public class Udpsend {


    public static void main(String[] args) throws Exception{
        //建立socket
        DatagramSocket socket = new DatagramSocket();


        //建立包，因为UDP是传输数据包的
        String msg = "hello ";
        InetAddress localhost = InetAddress.getByName("localhost");
        int port = 9090;


       DatagramPacket packet  = new DatagramPacket(msg.getBytes(),0,msg.getBytes().length,localhost,port);
       //发送包
       socket.send(packet);


       socket.close();


    }
}
```

接收方

```java
package com.internet;


import java.net.DatagramPacket;
import java.net.DatagramSocket;


public class Udprecieve {
    public static void main(String[] args) throws Exception{
        //开放端口
        DatagramSocket socket = new DatagramSocket(9090);


        //接收数据包
        byte[] buffer = new byte[1024];
        DatagramPacket packet = new DatagramPacket(buffer,0,buffer.length);
        socket.receive(packet);


        System.out.println(packet.getAddress().getHostAddress());
        System.out.println(new String(packet.getData(),0,packet.getLength()));


        socket.close();
    }
}
```



# 使用UDP实现聊天功能

要实现聊天功能，就要编写两个线程

一个接收线程一个发送线程

发送线程如下：

```java
package com.chat;


import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.*;


public class TalkSend implements Runnable{


    DatagramSocket socket = null;
    BufferedReader reader = null;


    private int fromPort;
    private  String toIP;
    private  int toPort;


    public TalkSend(int fromPort, String toIP, int toPort) {  //在构造器中就完成Socket的初始化以及IO流的初始化
        this.fromPort = fromPort;
        this.toIP = toIP;
        this.toPort = toPort;


        try {
            socket = new DatagramSocket(fromPort);
            reader = new BufferedReader(new InputStreamReader(System.in));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    @Override
    public void run() {
        while(true){
            String data = null;
            try {
                data = reader.readLine();
                byte[] datas = data.getBytes();
                DatagramPacket packet = new DatagramPacket(datas,0,datas.length,new InetSocketAddress(this.toIP,this.toPort));
                socket.send(packet);
                if (data.equals("bye")) break;


            } catch (Exception e) {
                e.printStackTrace();
            }




        }


    }
}
```

接收方

```java
package com.chat;


import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.SocketException;


public class TalkReceive implements Runnable{


    DatagramSocket socket = null;


    private int port;


    public TalkReceive(int port) {
        this.port = port;


        try {
            socket = new DatagramSocket(port);
        } catch (SocketException e) {
            e.printStackTrace();
        }
    }


    @Override
    public void run() {


        while(true){

            try {
                byte[] container = new byte[1024];        //接受数据的容器
                DatagramPacket packet = new DatagramPacket(container, 0, container.length);
                socket.receive(packet);


                byte[] data = packet.getData();
                String receiveData = new String(data,0,data.length);


                System.out.println(receiveData);


                if (receiveData.equals("bye")){
                    break;
                }


            } catch (Exception e) {
                e.printStackTrace();
            }




        }


    }
}
```

然后用两个主函数分别启动这两个线程

```java
package com.chat;


public class TalkStudent {
    public static void main(String[] args) {
        new Thread(new TalkSend(7777,"localhost",9999)).start();
        new Thread(new TalkReceive(8888)).start();
    }
}
```

```java
package com.chat;


public class TalkTeacher {
    public static void main(String[] args) {
        new Thread(new TalkSend(5555,"localhost",8888)).start();
        new Thread(new TalkReceive(9999)).start();
    }
}
```

