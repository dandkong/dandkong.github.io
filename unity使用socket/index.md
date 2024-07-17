# Unity使用Socket


## 前言

之前开发一直都在关注客户端的逻辑，没有涉及到网络相关的知识，这几天参考网上的资料学习了一下做个简易的网络聊天室，记录一下。

简单的网络通信于 Unity 客户端借助 Socket 与服务端达成连接，并依据事先约定的协议（诸如 json、protobuf 等）展开通信。

## 客户端

客户端的主要功能：连接，发送数据，接收数据。

这里参考开源的简易[网络框架](https://github.com/gmhevinci/UniFramework/tree/main/UniFramework/UniNetwork)，可以自定义其包体，编码器和解码器。

```jsx
using System.Net;
using System.Net.Sockets;
using System.Text;
using UnityEngine;
using UniFramework.Network;

// 登录请求消息
class LoginRequestMessage
{
    public string Name;
    public string Password;
}

// 登录反馈消息
class LoginResponseMessage
{
    public string Result;
}

// TCP客户端
UniFramework.Network.TcpClient _client = null;

// 创建TCP客户端
void CreateClient()
{
    // 初始化网络系统
    UniNetwork.Initalize();

    // 创建TCP客户端
    int packageMaxSize = short.MaxValue;
    var encoder = new DefaultNetPackageEncoder();
    var decoder = new DefaultNetPackageDecoder();
    _client = UniNetwork.CreateTcpClient(packageMaxSize, encoder, decoder);

    // 连接服务器
    var remote = new IPEndPoint(IPAddress.Parse("127.0.0.1"), 8000);
    _client.ConnectAsync(remote, OnConnectServer);
}

// 关闭TCP客户端
void CloseClient()
{
    if(_client != null)
    {
        _client.Dispose();
        _client = null; 
    }
}

void OnConnectServer(SocketError error)
{
    Debug.Log($"Server connect result : {error}");
    if (error == SocketError.Success)
        Debug.Log("服务器连接成功！");
    else
        Debug.Log("服务器连接失败！");
}

void Update()
{
    // 每帧去获取解析的网络包
    DefaultNetPackage networkPackage = client.PickPackage() as DefaultNetPackage;
    if(networkPackage != null)
    {
        string json = Encoding.UTF8.GetString(networkPackage.BodyBytes);
        LoginResponseMessage message = JsonUtility.FromJson<LoginResponseMessage>(json);
        Debug.Log(message.Result);
    }
}

// 发送登录请求消息
void SendLoginMessage()
{
    LoginRequestMessage message = new LoginRequestMessage();
    message.Name = "hevinci";
    message.Password = "1234567";

    DefaultNetPackage networkPackage = new DefaultNetPackage();
    networkPackage.MsgID = 10001;
    networkPackage.BodyBytes = Encoding.UTF8.GetBytes(JsonUtility.ToJson(message));
    _client.SendPackage(networkPackage);
}
```

## Buffer格式

```jsx
// 写入包头
{
    // 写入消息ID
    ringBuffer.WriteInt(package.MsgID);
    // 写入包体长度
    ringBuffer.WriteInt(bodyData.Length);
}

// 写入包体
ringBuffer.WriteBytes(bodyData, 0, bodyData.Length);
```

- 协议号：4位
- 包体长度：4位
- 内容：其他位，这里可以使用JSON或者Protobuf序列化

## 服务端

服务端主要功能：管理连接的客户端，接收数据，处理数据，发送数据。

参考[【游戏开发实战】Unity使用Socket通信实现简单的多人聊天室（万字详解 | 网络 | TCP | 通信 | Mirror | Networking）_unity网络实战-CSDN博客](https://blog.csdn.net/linxinfa/article/details/118888064)

```jsx
'''
作者：林新发，博客：<https://blog.csdn.net/linxinfa>
功能：简单的Socket通信，聊天室服务端
python版本：3.6.4
'''
import socket  # 导入 socket 模块
from threading import Thread
import time
import json

ADDRESS = ('127.0.0.1', 8712)  # 绑定地址
g_socket_server = None  # 负责监听的socket
g_conn_pool = {}  # 连接池

def accept_client():
    global g_socket_server
    g_socket_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  
    g_socket_server.bind(ADDRESS)
    g_socket_server.listen(5)  # 最大等待数（有很多人理解为最大连接数，其实是错误的）
    print("server start，wait for client connecting...")
    '''
    接收新连接
    '''
    while True:
        client, info = g_socket_server.accept()  # 阻塞，等待客户端连接
        # 给每个客户端创建一个独立的线程进行管理
        thread = Thread(target=message_handle, args=(client, info))
        thread.setDaemon(True)
        thread.start()
 
 
def message_handle(client, info):
    '''
    消息处理
    '''
    handle_id = info[1]
    # 缓存客户端socket对象
    g_conn_pool[handle_id] = client
    while True:
        try:
            data = client.recv(1024)
            response_data = data
            
            msg_id = struct.unpack('I', data[:4])[0]
            body_length = struct.unpack('I', data[4:8])[0]
            body_data = data[8:8+body_length]
            
            #todo 通过json或者protobuf反序列化body_data，按需处理逻辑
            
	          # 这里简单转发给所有客户端
            for key in g_conn_pool:
                g_conn_pool[key].sendall(response_data)
                
        except Exception as e:
            remove_client(handle_id)
            break

def remove_client(handle_id):
    client = g_conn_pool[handle_id]
    if None != client:
        client.close()
        g_conn_pool.pop(handle_id)
        print("client offline: " + str(handle_id))

if __name__ == '__main__':
    # 新开一个线程，用于接收新连接
    thread = Thread(target=accept_client)
    thread.setDaemon(True)
    thread.start()
    # 主线程逻辑
    while True:
        time.sleep(0.1) 
```

## 总结

底层的简单框架就是这样，正式的游戏中还需要更系统的管理，比如接入protobuf，根据不同的协议处理不同的逻辑，可以参考protobuf的文档进行进一步的学习。

## 参考

[【游戏开发实战】Unity使用Socket通信实现简单的多人聊天室（万字详解 | 网络 | TCP | 通信 | Mirror | Networking）_unity网络实战-CSDN博客](https://blog.csdn.net/linxinfa/article/details/118888064)

https://github.com/gmhevinci/UniFramework/blob/main/UniFramework/UniNetwork/Runtime/Package/DefaultNetPackageEncoder.cs

[Unity中使用ProtoBuf-保姆式教程_unity protobuf-CSDN博客](https://blog.csdn.net/flj135792468/article/details/119970151)

https://github.com/protocolbuffers/protobuf

https://github.com/starwing/lua-protobuf

