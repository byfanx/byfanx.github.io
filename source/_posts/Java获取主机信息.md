---
title: Java获取主机信息
route: JavaGetHostInfo
tags: [Java]
date: 2021-12-22 14:36:39
categories: 知识点
image: /images/cover/JavaGetHostInfo.jpeg
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最近在做一个主机资源监控的需求，首先是获取一些最简单的基本参，像一些主机名称、系统类型、ip、cpu、内存和磁盘等等这些数据，看起来虽然很简单，Java的基本库就能完成，但是真的去使用的时候，还是有一些坑的。记录一下，已备后用。
<!-- more -->

# Java获取主机信息

## 1. 获取基本信息

### 1.1 获取主机名称和系统

主机名称可以通过网络类**InetAddress**来获取，主机系统和用户可以通过**System**类进行获取。

```java
public static void getLocalHost(){
    try{
        InetAddress ip = InetAddress.getLocalHost();
        String localName = ip.getHostName();
        String osName = System.getProperty("os.name");
        String userName = System.getProperty("user.name");
        String osVersion = System.getProperty("os.version");
        String osArch = System.getProperty("os.arch");
        
        System.out.println("当前用户：" + userName);
        System.out.println("用户的主目录："+props.getProperty("user.home"));
        System.out.println("用户的当前工作目录："+props.getProperty("user.dir"));
        System.out.println("主机名称：" + localName);
        System.out.println("主机系统：" + osName);
        System.out.println("系统版本：" + osVersion);
        System.out.println("系统架构：" + osArch);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

### 1.2 获取用户信息

用户信息都是使用**System**类进行获取。

```java
public static void getUserInfo(){
    try{
        String userName = System.getProperty("user.name");
        String userHome = System.getProperty("user.home");
        String userDir = System.getProperty("user.dir");
        
        System.out.println("当前用户：" + userName);
        System.out.println("用户主目录："+ userHome);
        System.out.println("当前工作目录："+ userDir);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



### 1.3 获取主机IP等信息

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主机的ip可以通过网络类**InetAddress**进行获取，但是这个方法很玄学，机器上多网卡还有虚拟机时，获取到就不准确了。目前做的获取的方法是痛殴便利网卡来获取ip。因为遍历网卡来获取ip要过滤一些不重要的网卡，过滤的方法是来自**“经验”**的笨方法，可以借鉴，但不保证日后网卡条件复杂的情况下获取不准确。测试的是Linux、Mac和Windows系统可用。因为过滤条件不一样，所以分为Windows获取和非Windows获取。

**Windows系统获取IP：**

```java
public static void getWindowsIpAndMac(){
    try {
        Enumeration<NetworkInterface> allNetInterfaces = NetworkInterface.getNetworkInterfaces();
        // 遍历网卡接口
        while (allNetInterfaces.hasMoreElements()) {
            NetworkInterface netInterface = allNetInterfaces.nextElement();
            // 去除回环接口，子接口，未运行和接口
            if (netInterface.isLoopback() || netInterface.isVirtual() || !netInterface.isUp()) {
                continue;
            }
			
            // 重点来了：“经验”之谈
            // 为了过滤掉虚拟机的网卡，可以通过网卡名来进行基础过滤。windows主机ip对应的网卡名会包含下面三个：Intel  无线、Realtek  网线、Ethernet  兼容xp系统
            if (!netInterface.getDisplayName().contains("Intel")
                && !netInterface.getDisplayName().contains("Realtek")
                && !netInterface.getDisplayName().contains("Ethernet")) {
                continue;
            }
            
            String ip = "";
            String mac = "";
            String niName = "";
            Enumeration<InetAddress> addresses = netInterface.getInetAddresses();
            while (addresses.hasMoreElements()) {
                InetAddress ia = addresses.nextElement();
                // 去除本地回环地址，子接口，未运行和地址
                if (ia != null && !ia.isLoopbackAddress() && ia.isSiteLocalAddress() && !ia.isAnyLocalAddress()) {
                    // 判断是否是ip v4地址
                    if (ia instanceof Inet4Address) {
                        ip = ia.getHostAddress();
                        // 获取MAC地址
                        mac = getMac(ia);
                        niName = netInterface.getName();
                        if (StringUtils.isNotBlank(ip) && StringUtils.isNotBlank(mac) && StringUtils.isNotBlank(niName)){
                            System.out.println("当前网卡："+niName);
                            System.out.println("当前主机ip："+ip);
                            System.out.println("当前主机MAC："+mac);
                            return;
                        }
                    }
                }
            }
        }
    } catch (SocketException e) {
        e.printStackTrace();
    }
}
```

**非Windows系统获取IP：**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其实和windows获取的差不多，也是遍历网卡然后进行过滤，不过这个没有“经验”，不知道要过滤那些，所以用**InetAddress**进行获取，经测试这个在非windows上获取的还是准确的（可能我linux网卡单一）。不过为了获取当前的网卡用了一个更笨的方法，既然当前获取的ip是准确的，那就根据ip去获取网卡。不过目前没有找到这个方法，所以可以在遍历网卡时取出符合当前ip的网卡。（此方法在我这个需求里是可以的，不保证拿走就能用）。

![机智如我](image-20211222173816612.png)

```java
public static void getLinuxIpAndMac(AgentMonitor agentMonitor){
    try {
        // 先获取ip
        InetAddress iad = InetAddress.getLocalHost();
        String localIp = iad.getHostAddress();

        // 遍历网卡
        Enumeration<NetworkInterface> allNetInterfaces = NetworkInterface.getNetworkInterfaces();
        while (allNetInterfaces.hasMoreElements()) {
            NetworkInterface netInterface = allNetInterfaces.nextElement();
            // 去除回环接口，子接口，未运行和接口
            if (netInterface.isLoopback() || netInterface.isVirtual() || !netInterface.isUp()) {
                continue;
            }

            String ip = "";
            String mac = "";
            String niName = "";
            Enumeration<InetAddress> addresses = netInterface.getInetAddresses();
            while (addresses.hasMoreElements()) {
                InetAddress ia = addresses.nextElement();
                if (ia != null && !ia.isLoopbackAddress() && ia.isSiteLocalAddress() && !ia.isAnyLocalAddress()) {
                    // 判断是否是ip v4地址且是否和已获取的ip一致
                    if (ia instanceof Inet4Address && ia.getHostAddress().equals(localIp)) {
                        ip = ia.getHostAddress();
                        // 获取MAC地址
                        mac = getMac(ia);
                        niName = netInterface.getName();
                        if (StringUtils.isNotBlank(ip) && StringUtils.isNotBlank(mac) && StringUtils.isNotBlank(niName)){
                            System.out.println("当前网卡："+niName);
                            System.out.println("当前主机ip："+ip);
                            System.out.println("当前主机MAC："+mac);
                            return;
                        }
                    }
                }
            }
        }

    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

**获取MAC地址**

```java
public static String getMac(InetAddress ia){
    try {
        //获取网卡，获取地址
        byte[] mac = NetworkInterface.getByInetAddress(ia).getHardwareAddress();
        StringBuffer sb = new StringBuffer();
        if (mac != null && mac.length>0){
            for(int i=0; i<mac.length; i++) {
                if(i!=0) {
                    sb.append("-");
                }
                //字节转换为整数
                String str = Integer.toHexString(mac[i] & 0xff);
                if(str.length()==1) {
                    sb.append("0").append(str);
                }else {
                    sb.append(str);
                }
            }
        }
        return sb.toString().toUpperCase();
    } catch (SocketException e) {
        e.printStackTrace();
        return null;
    }
}
```

## 2. 获取CPU信息

获取CPU的信息这里选用的是**[oshi](https://github.com/oshi/oshi)**工具，经测试这个获取的还是比较准确的，而且该工具还可以获得其他硬件信息，能获取到的还是比较全面的。首先需要引入**oshi**的依赖。

```xml
<dependency>
    <groupId>com.github.oshi</groupId>
    <artifactId>oshi-core</artifactId>
    <version>3.12.2</version>
</dependency>
```

> **oshi**是依赖于**JNA**，需要导入`jna`和`jna-platform`我这里用的**oshi**是*3.12.2*版本，对应使用的**JNA**的版本是*5.2.0*。springboot项目是自带JNA的，如果不是springboot项目需要额外导入。如果springboot项目自带的JNA版本过低，也需要额外导入高版本的JNA。
>
> ```xml
> <dependency>
>     <groupId>net.java.dev.jna</groupId>
>     <artifactId>jna</artifactId>
>     <version>5.2.0</version>
> </dependency>
> ```

![JNA版本](image-20211223104609629.png)



### 2.1 获取CPU核数

**oshi**中的**CentralProcessor**进行获取。获取CPU物理可用的核数，如果有开启超频，那么获取的CPU核数可能会大于物理核数。

```java
public static void getCpuCount(){
    try {
        // 获取SystemInfo实例
        SystemInfo systemInfo = new SystemInfo();
        // 获取CentralProcessor实例
        CentralProcessor processor = systemInfo.getHardware().getProcessor();
        // 获取CPU核数
        int cpuCount = processor.getLogicalProcessorCount();
        System.out.println("CPU核数："+cpuCount);
    } catch (SocketException e) {
        e.printStackTrace();
    }
}
```

### 2.2 获取CPU使用率

获取系统范围的CPU负载时，一共获取7个部分的负载。

- CPU 空闲且系统没有未完成的磁盘 I/O 请求的时间。
- 在系统有未完成的磁盘 I/O 请求期间一个或多个 CPU 空闲的时间。在windows不可用。在MacOS不可用。
- CPU 用于服务硬件 IRQ 的时间。在MacOS不可用。
- 在具有良好优先级的用户级别执行时发生的 CPU 利用率。在windows不可用。
- CPU 用于服务软 IRQ 的时间。
- 管理程序专用于系统中其他来宾的时间。
- 在系统级别（内核）执行时发生的 CPU 利用率。
- 在用户级别（应用程序）执行时发生的 CPU 使用率。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;要使用此方法计算总体空闲时间，就要包括上面所有部分，这样计算出来的结果更准确且兼容各种平台。分两次获取上面信息，间隔1秒。这样就能计算出1秒的CPU各方面使用的差值，通过每一项的差值除以总量，便可以得到每一项的CPU使用率。

通过下面方法还可以获得CPU时间间隔内的使用率和总使用率。

```java
public static void getCpuInfo() {
    try {
        SystemInfo systemInfo = new SystemInfo();
        CentralProcessor processor = systemInfo.getHardware().getProcessor();
        // 获取系统范围的cpu负载技计数
        long[] prevTicks = processor.getSystemCpuLoadTicks();
        // 睡眠1s
        TimeUnit.SECONDS.sleep(1);
        long[] ticks = processor.getSystemCpuLoadTicks();
        // 具有良好优先级的用户级别
        long nice = ticks[CentralProcessor.TickType.NICE.getIndex()] - prevTicks[CentralProcessor.TickType.NICE.getIndex()];
        // 硬件服务
        long irq = ticks[CentralProcessor.TickType.IRQ.getIndex()] - prevTicks[CentralProcessor.TickType.IRQ.getIndex()];
        // 软服务使用
        long softirq = ticks[CentralProcessor.TickType.SOFTIRQ.getIndex()] - prevTicks[CentralProcessor.TickType.SOFTIRQ.getIndex()];
        // 管理程序使用
        long steal = ticks[CentralProcessor.TickType.STEAL.getIndex()] - prevTicks[CentralProcessor.TickType.STEAL.getIndex()];
        // 系统使用
        long cSys = ticks[CentralProcessor.TickType.SYSTEM.getIndex()] - prevTicks[CentralProcessor.TickType.SYSTEM.getIndex()];
        // 用户使用
        long user = ticks[CentralProcessor.TickType.USER.getIndex()] - prevTicks[CentralProcessor.TickType.USER.getIndex()];
        // 等待使用
        long iowait = ticks[CentralProcessor.TickType.IOWAIT.getIndex()] - prevTicks[CentralProcessor.TickType.IOWAIT.getIndex()];
        // 空闲使用
        long idle = ticks[CentralProcessor.TickType.IDLE.getIndex()] - prevTicks[CentralProcessor.TickType.IDLE.getIndex()];
        long totalCpu = user + nice + cSys + idle + iowait + irq + softirq + steal;
        double sysRate = cSys * 1.0 / totalCpu;
        double userRate = user * 1.0 / totalCpu;
        double waitRate = cSys * 1.0 / totalCpu;
        double idleRate = cSys * 1.0 / totalCpu;
        double betweenRate = processor.getSystemCpuLoadBetweenTicks();
        double cpuLoad = processor.getSystemCpuLoad();
        System.out.println("cpu系统使用率:" + new DecimalFormat("#.##%").format(sysRate));
        System.out.println("cpu用户使用率:" + new DecimalFormat("#.##%").format(userRate));
        System.out.println("cpu当前等待率:" + new DecimalFormat("#.##%").format(waitRate));
        System.out.println("cpu当前空闲率:" + new DecimalFormat("#.##%").format(idleRate));
        // 获取cpu最近(时间间隔内)使用率
        System.out.println("CPU load: "+ new DecimalFormat("#.##%").format(betweenRate) +"(counting ticks)");
        // 获取cpu使用率
        System.out.println("CPU load: "+ new DecimalFormat("#.##%").format(cpuLoad) +"(OS MXBean)");
    }catch (Exception e){
        e.printStackTrace();
    }
}
```

## 3. 获取内存信息

### 3.1 获取主机内存

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;获取内存信息可以使用**OperatingSystemMXBean** 来获取。内存信息可以获取到的有**内存总量**和**可用内存**，通过这两个值在计算出内存已经使用的量和内存的使用率，测试时在Linux获取的数据不太精确，只获取到的空闲内存，获取不到可用内存。获取内存信息同样也可以使用**oshi**包中的**SystemInfo**类进行获取，这个获取的还是挺准确的。

**OperatingSystemMXBean**获取：

```java
public static void getMemInfo(){
    try {
        OperatingSystemMXBean osmxb = (OperatingSystemMXBean) ManagementFactory.getOperatingSystemMXBean();
        // 总内存，单位：字节
            long total = osmxb.getTotalPhysicalMemorySize();
            // 空闲内存，单位：字节
            long free = osmxb.getFreePhysicalMemorySize();
            // 可用内存，单位：字节
            long usable = osmxb.getFreePhysicalMemorySize();
            // 已使用内存，单位：字节
            long used = total - free;
            // 内存使用率
            double useRate = used * 1.0 / total;
            System.out.println("总共内存：" + new DecimalFormat("#.##").format(total*1.0 / Math.pow(1024,3)) + "G");
            System.out.println("空闲内存：" + new DecimalFormat("#.##").format(free*1.0 / Math.pow(1024,3)) + "G");
            System.out.println("已用内存：" + new DecimalFormat("#.##").format(used*1.0 / Math.pow(1024,3)) + "G");
            System.out.println("可用内存：" + new DecimalFormat("#.##").format(usable*1.0 / Math.pow(1024,3)) + "G");
            System.out.println("内存使用率：" + new DecimalFormat("#.##%").format(useRate * 100.0));

    }catch (Exception e){
        e.printStackTrace();
    }
}
```

**oshi.SystemInfo**获取：

```java
public static void getMemInfo(){
    try {
        SystemInfo systemInfo = new SystemInfo();
        GlobalMemory memory = systemInfo.getHardware().getMemory();
        // 总内存，单位：字节
        long total = memory.getTotal();
        // 可用内存，单位：字节
        long usable = memory.getAvailable()；
        // 已使用内存，单位：字节
        long used = total - usable;
        // 内存使用率
        double useRate = used * 1.0 / total;
        System.out.println("总共内存：" + new DecimalFormat("#.##").format(total*1.0 / Math.pow(1024,3)) + "G");
        System.out.println("已用内存：" + new DecimalFormat("#.##").format(used*1.0 / Math.pow(1024,3)) + "G");
        System.out.println("可用内存：" + new DecimalFormat("#.##").format(usable*1.0 / Math.pow(1024,3)) + "G");
        System.out.println("内存使用率：" + new DecimalFormat("#.##%").format(useRate * 100.0));
    }catch (Exception e){
        e.printStackTrace();
    }
}
```

### 3.2 获取JVM内存

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;获取`JVM`的内存信息需要使用**MemoryMXBean**接口中的**MemoryUsage**类。JVM信息主要是在系统运行时对JVM的使用情况。包括初始的内存大小、最大可用的内存以及当前已经使用的内存大小。

```java
public static void getJvmMemInfo(){
    try {
        MemoryMXBean memoryMXBean = ManagementFactory.getMemoryMXBean();
        // 椎内存使用情况
        MemoryUsage memoryUsage = memoryMXBean.getHeapMemoryUsage();
        // jvm初始总内存，单位：字节
        long initTotalMemorySize = memoryUsage.getInit();
        // jvm最大可用内存，单位：字节
        long free = osmxb.getFreePhysicalMemorySize();
        // jvm已使用的内存，单位：字节
        long usable = osmxb.getFreePhysicalMemorySize();
        
        System.out.println("jvm初始总内存：" + new DecimalFormat("#.##").format(total*1.0 / Math.pow(1024,3)) + "G");
        System.out.println("jvm最大可用内存：" + new DecimalFormat("#.##").format(free*1.0 / Math.pow(1024,3)) + "G");
        System.out.println("jvm已使用的内存：" + new DecimalFormat("#.##").format(used*1.0 / Math.pow(1024,3)) + "G");

    }catch (Exception e){
        e.printStackTrace();
    }
}
```

## 4. 获取磁盘信息

获取磁盘的使用情况用的是基础的**File**类。首先是从根目录遍历所有磁盘信息，通过下面方法获取磁盘信息。

- **file.getTotalSpace()** ：获取当前磁盘的总内存
- **file.getFreeSpace()** ：获取当前磁盘的空闲内存
- **file.getUsableSpace()** ：获取当前磁盘的可用内存

通过上面获取的三个参数，可以计算磁盘总的已使用内存和当前磁盘的内存使用率。

在计算每一个磁盘的信息时，通过全局变量统计所有磁盘的信息总和，然后计算出主机总的磁盘内存和使用率。

```java
/**
 * @param RADIX 内存进制大小，"经验"之谈是：Windows下进制是1024，Mac和Linux是1000
 */
public static void getDiskInfo(int RADIX){
    // 统计总内存
    long total = 0;
    // 统计总空闲
    long free = 0;
    // 统计总可用
    long usable = 0;
    // 统计总已用
    long used = 0;
    // 磁盘总使用
    double usedRate = 0.0；
    try{

        File[] disks = File.listRoots();
        for (File file : disks){
            // 统计总量
            total += file.getTotalSpace();
            free += file.getFreeSpace();
            usable += file.getUsableSpace();
            used += file.getTotalSpace() - file.getFreeSpace();
            
            String diskPath = file.getPath();
            long diskTotal = file.getTotalSpace();
            long diskFree = file.getFreeSpace();
            long diskUsable = file.getUsableSpace();
            long diskUsed = diskTotal - diskFree;
            double diskUsedRate = diskUsed * 1.0 / diskTotal;
         
            System.out.println("磁盘路径：" + diskPath);
            System.out.println("总共空间："+ new DecimalFormat("#.##").format(diskTotal*1.0 / Math.pow(RADIX,3)) + "G");
            System.out.println("空闲空间："+ new DecimalFormat("#.##").format(diskFree*1.0 / Math.pow(RADIX,3)) + "G");
            System.out.println("可用空间："+ new DecimalFormat("#.##").format(diskUsable*1.0 / Math.pow(RADIX,3)) + "G");
            System.out.println("已用空间："+ new DecimalFormat("#.##").format(diskUsed*1.0 / Math.pow(RADIX,3)) + "G");
            System.out.println("空间使用率：" + new DecimalFormat("#.##%").format(diskUsedRate*100));
            
        }
        
        String rootPath = "/";
        usedRate = used * 1.0 / total;
        
        System.out.println("磁盘根路径："+ rootPath);
        System.out.println("主机总共空间："+ new DecimalFormat("#.##").format(total*1.0 / Math.pow(RADIX,3)) + "G");
        System.out.println("主机总空闲空间："+ new DecimalFormat("#.##").format(free*1.0 / Math.pow(RADIX,3)) + "G");
        System.out.println("主机总可用空间："+ new DecimalFormat("#.##").format(usable*1.0 / Math.pow(RADIX,3)) + "G");
        System.out.println("主机总已用空间："+ new DecimalFormat("#.##").format(used*1.0 / Math.pow(RADIX,3)) + "G");
        System.out.println("主机总使用率：" + new DecimalFormat("#.##%").format(usedRate*100.0));
        
    }catch (Exception e){
        e.printStackTrace();
    }
}
```

## 5. 获取Java环境信息

这块就是补充说明了，暂时没用到，先保留一下，已备后用。

```java
public static void getJavaInfo(){
    Properties props=System.getProperties();
    System.out.println("Java的运行环境版本："+props.getProperty("java.version"));
    System.out.println("Java的运行环境供应商："+props.getProperty("java.vendor"));
    System.out.println("Java供应商的URL："+props.getProperty("java.vendor.url"));
    System.out.println("Java的安装路径："+props.getProperty("java.home"));
    System.out.println("Java的虚拟机规范版本："+props.getProperty("java.vm.specification.version"));
    System.out.println("Java的虚拟机规范供应商："+props.getProperty("java.vm.specification.vendor"));
    System.out.println("Java的虚拟机规范名称："+props.getProperty("java.vm.specification.name"));
    System.out.println("Java的虚拟机实现版本："+props.getProperty("java.vm.version"));
    System.out.println("Java的虚拟机实现供应商："+props.getProperty("java.vm.vendor"));
    System.out.println("Java的虚拟机实现名称："+props.getProperty("java.vm.name"));
    System.out.println("Java运行时环境规范版本："+props.getProperty("java.specification.version"));
    System.out.println("Java运行时环境规范供应商："+props.getProperty("java.specification.vender"));
    System.out.println("Java运行时环境规范名称："+props.getProperty("java.specification.name"));
    System.out.println("Java的类格式版本号："+props.getProperty("java.class.version"));
    System.out.println("Java的类路径："+props.getProperty("java.class.path"));
    System.out.println("加载库时搜索的路径列表："+props.getProperty("java.library.path"));
    System.out.println("默认的临时文件路径："+props.getProperty("java.io.tmpdir"));
    System.out.println("一个或多个扩展目录的路径："+props.getProperty("java.ext.dirs"));
   
    System.out.println("文件分隔符："+props.getProperty("file.separator"));//在 unix 系统中是＂／＂ System.out.println("路径分隔符："+props.getProperty("path.separator"));//在 unix 系统中是＂:＂ System.out.println("行分隔符："+props.getProperty("line.separator"));//在 unix 系统中是＂/n＂ System.out.println("用户的账户名称："+props.getProperty("user.name
}
```

