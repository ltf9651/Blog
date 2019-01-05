## LVS

基于**IP地址**调度

IP负载均衡技术：通过IPVS内核模块实现，IPVS是LVS集群系统的核心软件

过程：访问的请求首先经过VIp到达负载调度器（Director)，由负载调度器从RealServer列表中去一个服务节点响应请求

IPVS实现机制
  - NAT 地址转换技术
  - TUN IP隧道模式
  - DR 直接路由模式

### NAT
1. Director将VIP分配到DirectorServer
1. 将收到的集群服务请求保温目标地址转换成根据算法计算得出的后端IP地址
1. 后端主机将响应报文发送至Director
1. 再由Director将源地址转换成VIP的地址


### DR
1. VIP将Request根据算法选择进行处理Server
1. Request数据地址改为RealServer的MAC地址（ARP协议）
1. 数据发送给真正的RealServer（根据MAC地址）

RealServer在收到数据后会判断目标IP地址是否为自己的IP，如果是则对数据进行处理，若不是则丢弃该数据

须在所有RealServer机器绑定VIP的IP地址

在DR模式下，RealServer处理后将目标IP改成Client IP，直接发送给Client IP，不经过LVS，效率高

### TUN
1. Client通过VIP发送Request到Director
1. Director收到数据后根据LVS设置的算法选择一个适合的RealServer，并把Client发送的数据包装到一个新的IP包里，新的IP包的目标地址为RealServer的IP
1. RealServer收到包后判断目标IP是否为自己的IP（如果是则对数据进行处理，若不是则丢弃该数据，所以须在所有RealServer机器绑定VIP的IP地址）