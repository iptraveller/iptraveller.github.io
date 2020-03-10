---
layout: post
title: 5G专网
excerpt: ""
categories: [5g]
comments: true

---

## 概念
5G专网称为Non-public Networks，也可称为Private Networks

## 场景
* 工业物联网（IIoT）：制造业、采矿、物流、港口

## 技术方案
### 两种形式
* 独立建立私网
* 共享运营商网络（网络切片）

![01](/img/2020-03-10-01.png) 

### 七种部署方案
* 1）企业建立的隔离5G局域网（本地5G频率，完全私有，不共享）

![02](/img/2020-03-10-02.png) 

企业在其场所（站点/建筑物）内部署5G网络全套（gNB，UPF，5GC CP，UDM，MEC）。企业中的5G频率是本地5G频率，而不是移动运营商的许可频率。  
**优点：**  
隐私和安全性：专用网络与公用网络物理上分离，提供完整的数据安全性  
超低延迟：由于设备和应用程序服务器之间的网络延迟在几毫秒内，因此可以实现URLLC应用程序服务。  
建筑物无光纤：无需进行回程以保持本地服务正常运行。5G服务可以立即提供给没有光回程链路的企业，例如农村地区的工厂。  
即使发生移动运营商的5G网络故障：即使移动运营商的设施烧毁，该公司的5G专用网络也可以正常工作。  
**缺点：**  
部署成本：普通企业要自费购买和部署全套5G网络并不容易。特别是对于小型企业。  
运营人员：现有的私有LAN（有线以太网LAN，无线Wi-Fi LAN）运营团队没有专门知识来构建和运营5G网络。企业需要有合适的工程师。  

* 2）由移动运营商构建的隔离5G局域网（获得许可的5G频率，完全私有，无共享）

![03](/img/2020-03-10-03.png) 

与部署方案1）相同。的唯一区别是，移动运营商使用自己的许可5G频率在企业中构建和运营5G局域网。

* 3）私有网络和公共网络之间的RAN共享

![04](/img/2020-03-10-04.png) 

UPF，5GC CP，UDM和MEC部署在企业中，并与公共网络物理隔离。私有和公共网络之间仅共享企业内部的5G基站。  
属于专用网络的设备的数据流量被传递到企业中的私有UPF，属于公共网络的设备的数据流量被传递到企业中的UPF。移动运营商的边缘云。换句话说，诸如内部设备控制数据，内部视频数据等之类的专用网络流量仅保留在企业中，而诸如语音和Internet之类的公共网络服务流量则被传输到移动运营商的网络。  
** 优点：**  
安全性：专用和专用5GC CP和UDM内置在企业中，因此企业中专用网络设备的订阅信息和操作信息可在内部存储和管理，以免泄漏到企业外部。  
性能：UPF和MEC位于企业中，可在设备-gNB-UPF-MEC之间提供超低延迟的通信，使其非常适合使用URLLC应用程序的公司，例如自动驾驶和实时机器人/无人机控制。  

* 4）专用网络和公共网络之间的RAN和控制平面共享

![05](/img/2020-03-10-05.png) 

私有和专用UPF，MEC内置于企业中。5GC CP、5G基站，移动运营商边缘云中的UDM在私有和公共网络之间共享（RAN和控制平面共享）。  
gNB，5GC CP和UDM在逻辑上在专用网络和公用网络之间是分开的，而UPF和MEC在物理上是分开的。  
属于专用网络的设备的数据流量被传递到企业中的私有UPF，属于公共切片公共网络的设备的数据流量被传递到UPF。  
** 优点：**
性能：由于UPF和MEC位于企业中，因此它提供了设备-gNB-UPF-MEC之间的超低延迟通信，并且适合使用URLLC应用程序的公司，例如自动驾驶和实时机器人/无人机控制。  
** 缺点：**
安全性：专用网络设备和公用网络设备的控制平面功能（身份验证，移动性等）由移动运营商网络中的5GC CP和UDM执行。也就是说，企业中的专用网络设备，gNB和UPF与移动运营商的网络互通并由其管理（通过N2，N4接口）。对于专用网络设备的操作信息和订阅信息存储在移动运营商的服务器中而不是内部存储，可能是一个问题。  


* 5）专用网络和公共网络之间的RAN和核心共享（端到端网络切片）

![06](/img/2020-03-10-06.png) 

仅在企业内部部署gNB且UPF和MEC仅存在于移动运营商的边缘云中。专用网络和公用网络共享“逻辑上分开的5G RAN和核心”（gNB，UPF，5GC，MEC，UDM）。  
与UPF和MEC位于企业中的3）、4）不同，在这种情况下，企业中只有gNB。因此，私有5G设备与PC或本地Intranet服务器之类的Intranet（LAN）设备之间没有本地流量路径，因此流量必须到达运营商边缘云中的UPF，然后再返回到运营商边缘云中。企业通过专线与局域网设备进行通信。  
另外，为企业中的5G设备提供5G应用服务的MEC位于远离设备的移动运营商边缘云中。  
** 优点：**  
成本：与需要在企业内部部署UPF和/或5GC CP的情况2、3和4相比，此架构为移动运营商建立专用5G网络的成本最低。  
** 缺点：**  
网络时延：在这种体系结构中，取决于企业（5G设备）与运营商边缘云（UPF，MEC）之间的距离，网络延迟（RTT）可能是一个主要问题。  
安全性：由于专用网络设备的流量已从企业转移到移动运营商的网络，因此存在数据流量安全性的问题。尽管移动运营商将在其边缘云上划分UPF和MEC，以使我们的专用网络流量与公共和其他专用网络流量分开，但胆小的首席执行官担心例如其内部CCTV视频流量正在泄漏到企业外部这一事实。与4）一样，对于企业而言，将操作和订阅信息存储在移动运营商的网络上而不是公司的专用网络上也是令人不安的。  

## 参考资料
* [7 Deployment Scenarios of Private 5G Networks](https://www.netmanias.com/en/post/blog/14500/5g-edge-kt-sk-telecom/7-deployment-scenarios-of-private-5g-networks)