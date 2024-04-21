# CAN bus communication in Java

I have been working on a project that enables CAN bus access from Java. It uses JNA to connect to the Linux SocketCAN
API. The code is available [on GitHub](https://github.com/jazdw/jnaCan).

```java
RawCanSocket canSocket = new RawCanSocket();
canSocket.open();
CanInterface canInterface = new CanInterface("can0");
canSocket.setLoopback(true);
canSocket.bind(canInterface);
canSocket.setReceiveTimeout(100);

CanFrame txFrame = new CanFrame(0x100, new byte[] {1, 2, 3});
canSocket.send(txFrame);
CanFrame rxFrame = canSocket.receive(); // recieve reply

canSocket.setFilters(
                new CanFilter(0x100, 0xFFF),
                new CanFilter(0x200, 0xFFF),
                new CanFilter(0x300, 0xFF0));
rxFrame = canSocket.receive(); // recieve reply

canSocket.close();
```
