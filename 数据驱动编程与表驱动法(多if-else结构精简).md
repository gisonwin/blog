# 数据驱动编程与表驱动法(多if-else结构精简)

==[参考网址](https://blog.csdn.net/qq_22555107/article/details/78884261)==

### 一.数据驱动编程的核心

其核心出发点是相对于程序逻辑,人类更擅长于处理数据.数据比程序逻辑更容易驾驭.所以我们应该尽可能的将设计的复杂度从程序代码转移至数据.

举例如下:某程序需要处理其他程序发送的消息,消息类型是字符串,每个消息都需要一个函数进行处理.代码如下:

```c
void msg_proce(const char *msg_type,const char *msg_buf){
    if(0 == strcmp(msg_type,"invite")){
        invite_func(msg_buf);
    }else if(0 == strcmp(msg_type,"tring_100")){
        tring_func(msg_buf);
    }else if(0 == strcmp(msg_type,"ring_180")){
        ring_180_func(msg_buf);
    }else if(0 == strcmp(msg_type,"ring_181")){
        ring_181_func(msg_buf);
    }else if(0 == strcmp(msg_type,"ring_182")){
        ring_182_func(msg_buf);
    }else if(0 == strcmp(msg_type,"ring_183")){
        ring_183_func(msg_buf);
    }else if(0 == strcmp(msg_type,"ok_200")){
        ok_200_func(msg_buf);
    }else if(0 == strcmp(msg_type,"fail_400")){
        fail_400_func(msg_buf);
    }else if(0 == strcmp(msg_type,"fail_500")){
        fail_500_func(msg_buf);
    }else if(0 == strcmp(msg_type,"fail_300")){
        fail_300_func(msg_buf);
    }else{
        log("unrecognized message type %s\n",msg_type);
    }
}
```

上面的消息类型取自SIP协议,消息类型可能还会增加.大家看着这么长的判断流程是不是很累,检测一下中间件某个消息有没有处理也很费劲,而且每增加一个消息就要增加一个流程分支.

如果按照数据驱动编程设计,可能会设计如下

```c
typedef void (*SIP_MSG_FUN) (const char *)
    
typedef struct __msg_fun_st{
    const char *msg_type;//
    SIP_MSG_FUN fun_prt;//函数指针
}msg_fun_st;

msg_fun_st msg_flow[]={
    {"invite",invite_fun},{"tring_100",tring_fun},{"ring_180",ring_180_fun},
     {"ring_181",ring_181_fun},{"ring_182",ring_182_fun},{"ring_183",ring_183_fun},
     {"ok_200",ok_200_fun},{"fail_400",fail_400_fun},{"fail_500",fail_500_fun},             {"fail_300",fail_300_fun} 
};
void msg_proc (const char *msg_type,const char *msg_buf){
    int type_num =sizeof(msg_flow)/sizeof(msg_fun_st);
    for(int i=0;i<type_num;i++){
        if(0==strcmp(msg_flow[i].msg_type,msg_type)){}
        msg_flow[i].fun_prt(msg_buf);
        return;
    }
    }
	log("unrecognized message type %s\n",msg_type);
}
```

修改后的思路优势

- 可读性更强,消息处理流程一目了然
- 更容易修改,要增加新的消息,只需要修改数据即可,不需要修改流程
- 重用.第一种方案的很多if-else,其实只是消息类型和处理函数的不同,但逻辑是一样的.修改后的方案就是将这种相同的逻辑提取出来,而把容易发生变化的部分提到外面.

### 二.隐含思想

很多设计思路背后的原理其实都是想通的,隐含在数据驱动编程后的思想如下

- 控制复杂度.通过把程序逻辑的复杂度转移到人类更容易处理的数据中来,从而达到控制复杂度的目标.
- 隔离变化.上面例子,每个消息处理的逻辑是不变的,但是消息可能是变化的,所以把容易变化的消息和不容易变化的逻辑分离.
- 机制和策略的分离.机制就是消息的处理逻辑,策略就是不同的消息处理.

### 三.数据驱动编程能够做什么

可以应用在函数级的设计中,也可以应用在程序级的设计中,典型的如用表驱动法实现一个状态机,还可以用来系统级的设计中,比如DSL.

### 四.表驱动法定义

表驱动法(Table-Driven Approach,简单讲就是用查表的方式获取值)就是一种编程模式,从表里面查找信息而不使用逻辑语句.事实上,凡是能通过逻辑语句来选择的事物,都可以通过查表来选择.对简单情况而言,使用逻辑语句更为容易和直白,而随着逻辑链越来越复杂,查表法也就愈发显得更具吸引力

### 五.常用查表方式

- 直接访问(直接访问表)
- 索引访问(索引访问表)
- 分段访问(阶梯访问表)

### 六.直接访问表

```java
//将一维数组作为表
String today="Sun";
switch(index){
    case 0:
        today="Sun";
         break;
    case 1:
        today="Mon";
         break;
    case 2:
        today="Tue";
         break;
    case 3:
        today="Wed";
        break;
    case 4:
        today="Thu";
        break;
    case 5:
        today="Fri";
        break;
    default:
        today="Sat";
        break;   
}
```

使用表驱动的直接访问表方法

```java
String[] weekday = new String[]{"Sun","Mon","Tue","Wed","Thu","Fri","Sat"};
String today=weekday[index];
```

### 七.索引访问表

将HashMap作为表,首先要构建hash的Key-Value.

```java
//存入hashMap结构的key为weekday,value为
String[] weekday = new String[]{"Sun","Mon","Tue","Wed","Thu","Fri","Sat"};
String[] working= new String[]{"Travel","Meeting","Duty","Learning","Sport","Duty","Recreation"};
//构建hash的key-value pair
Map<String,Object> map = new HashMap<String,Object>();
for(int i=0;i<weekday.length;i++){
    map.put(weekday[i],working[i]);
}
public static String doWhat(String today){
    return map.get(today);
}
```

### 八.阶梯访问表

举例:用两个数组:0~59分是不及格,F级  60~79分是及格,E级  80~84分是普通 D级 85~89分是良好 C级

90~94分是优秀 B级 95~100分是太棒了 A级

最重要的是我的表里面存放了区间的上界,用来区分等级

```java
int[] grade = {59,79,84,89,94,100};
String[] level={"F","E","D","C","B","A"};
public String getLevel(int score){
    for(int i=0;i<grade.length;i++){
        if(score <= grade[i]){
            return level[i];
        }
    }
}
```



