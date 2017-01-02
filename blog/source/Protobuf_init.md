title: "Porotobuf 3的简单集成和使用"
date: 2017-01-02 13:06:00 +0800
update: 2017-01-02 13:06:00 +0800
author: me
cover: "-/images/protobuf1.png"
tags:
    - protobuf
    - message
preview: 难题不会，会题不难

---

## 关于评论
目前使用的评论功能好像被墙了，过段时间试试迁移到国内运营商试试。

## 什么是Protobuf
> What are protocol buffers?
Protocol buffers are a flexible, efficient, automated mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages. You can even update your data structure without breaking deployed programs that are compiled against the "old" format.

简单来说就是一种体积小，速度快，跨平台的数据结构/传输协议，甚至可以跨版本支持不同的数据结构。
Protobuf被广泛的应用于即时聊天等应用，比如`微信`，`whatsapp`等。

[最近很火的Mars的Protobuf文件](https://github.com/Tencent/mars/tree/master/samples/iOS/iOSDemo/Proto)

## 编译Protobuf
首先去下载[Protobuf](https://github.com/google/protobuf)。然后安装相关依赖`autoconf`，`automake`，`libtool`。
可以使用brew方便的安装: `brew install autoconf automake libtool`。

完成后进入protobuf根目录，执行`./objectivec/Devtools/full_mac_build.sh`。
经过慢长的等待后，在根目录里找到`src`文件夹。里面应该有编译好的`protoco`文件了。
![protobuf](-/images/protobuf3.png)

## 生成proto文件
首先新建一个文件(vim filename.proto)，然后按以下格式输入：

```
syntax = "proto3";

message Person {
  string name = 1;
  int32 userId = 2;
  string email = 3;
}
```
其中`syntax`指定的是版本，这里使用的`proto3`。message后面跟着的`Person`就是这个类的名字。
`string`, `int32`都是固定的数据类型，具体可用的类型可以参考[这里](https://developers.google.com/protocol-buffers/docs/proto3#scalar)。

有了这个文件后，我们就可以调用`protoc`来生成objective-c的类了。在src文件夹下面输出：

`protoc --proto_path=Person文件的目录 --objc_out=输出objc文件的地址 Person.proto`

生成好的文件如下:
![protobuf](-/images/protobuf4.png)

## 整合Protobuf到项目里
把`objectivec`文件夹下面所有的`.h`和`.m`(包括子文件夹) 都拖到项目里，除了`DevTools`文件夹, `generate_well_known_types.sh`, `GPBProtocolBuffers.m`。
![protobuf](-/images/protobuf5.png)

然后对所有的`.m`(包括刚生成的Person.m) 都加上了`-fno-objc-arc`，注意不要给原有的m文件加上。他们还是需要ARC的 ^_^。
![protobuf](-/images/protobuf6.png)

Build一下，可能会有报错，一般都是`.h`文件没找到，重新定向一下就好了。

## 简单使用

``` objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    Person *user = [Person new];
    user.name = @"LawrenceHan";
    user.userId = 1;
    user.email = @"yangfei6565@163.com";
    Person_PhoneNumber *phoneNumber = [Person_PhoneNumber new];
    phoneNumber.type = Person_PhoneType_Mobile;
    phoneNumber.number = @"+86 12345678901";
    user.phoneArray = [@[phoneNumber] mutableCopy];
    
    NSData *data = [user data];
    NSLog(@"data: %@", data);
    
    Person *parseredUser = [Person parseFromData:data error:nil];
    NSLog(@"%@", parseredUser);
}
```

Log如下:
``` text
data: <0a0b4c61 7772656e 63654861 6e10011a 1379616e 67666569 36353635 40313633 2e636f6d 22130a0f 2b383620 31323334 35363738 39303110 01>

<Person 0x6080000d6810>: {
    name: "LawrenceHan"
    userID: 1
    email: "yangfei6565@163.com"
    phone {
      number: "+86 12345678901"
      type: MOBILE
    }
}
```

[测试项目地址](https://github.com/LawrenceHan/LHWInstantMessage)
因为准备开源一个即时聊天功能项目，所以就放在一起了。 
