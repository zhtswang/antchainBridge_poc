## 蚂蚁跨链桥总结

### 1.  **背景**： 

### 	各自独立的区块链数据，需要通过数据的共享来创造更多的数据价值

   #### 1.1 选择蚂蚁跨链桥的原因是因为IEEE3222-2023标准是由其提出的，并且是基于该标准实现的

   #### 1.2 项目分为多个组件，技术栈如下

   - JAVA 8 （现在的relayer中使用了8中特有的一些已经过时的类，方法，所以升级时得特别处理）

   - 组件列表

     - BCDNS

       - Embedded BCDNS 工开发或者测试使用，和Relayer是绑定的，在Relayer的 Application.yml文件中加上如下配置

         ```yaml
         acb:
           bcdns:
             embedded:
               server-on: true
               server-host: 0.0.0.0
               server-port: 8090
               sign-algo: keccak256_with_secp256k1
               sign-cert-hash-algo: keccak_256
               root-cert-file: classpath:bcdns_certs/embedded-bcdns-root.crt
               root-private-key-file: classpath:bcdns_certs/embedded-bcdns-root-private-key.key
         ```

       - 基于星火链的BCDNS

         - 参考项目bcdns-credential-server,该项目会使用BIF（星火链）的SDK，通过RPC来请求对应的合约方法
         - 主要是几个smart contract
           - DomainNameManager.sol （管理区块链的域名合约）
           - RelayerManager.sol （管理Relayer身份的合约）
           - PTCManager.sol （管理证明转化的合约）

       - 其它考虑支持的实现（计划中）

     - [插件服务](https://github.com/AntChainOpenLabs/AntChainBridgePluginServer)

       - 依赖跨链桥SDK
       - 提供RPC服务供Relayer调用。比如管理插件服务等
       - 调用插件的方法的服务（由ps-service提供，可以用postman来进行测试其连通性等）

     - [中继服务](https://github.com/AntChainOpenLabs/AntChainBridgeRelayer)

       - 提供中继services （当前只支持http， https， 后续会有grpc(s)）的支持
       - 实际上是一组job运行的结果
         - Anchor Process
           - Poll Header （比较当前块高度和远端区块链块高度，若落后则开启poll Header）
           - Poll Blocks ( 根据poll header的结果来判断是否需要同步块或块头，并把结果放入Redis中)
           - Block Notify（读取Redis中的block数据，并终止存入MySQL中）
         - Validation 对于Block Notify的结果，从DB中取出pending的UCP，然后验证并更新UCP & AUTH状态
         - Authentic Message Process
           - 读取Proved的Auth message，并解析SDP message，如果当前的中继无法处理该目的区块链跨链数据，则根据目标区块链id来转发请求到对应的中继
         - Committer 
           - 将SDP messages以byte格式推送到目标链插件提供的服务
         - AM Confirm Service
           - 最终确定跨链是否成功

       

       ![image-20241028110004710](https://github.com/zhtswang/antchainBridge_poc/blob/main/image-20241028110004710.png)

     - [跨链桥 SDK]([GitHub - AntChainOpenLabs/AntChainBridgePluginSDK: AntChainBridge 插件SDK和插件池](https://github.com/AntChainOpenLabs/AntChainBridgePluginSDK))

       - Embedded BCDNS （需要编译其对应的projects，两个spring starters）
       - Plugin Manager （提供插件开发和插件服务来管理插件）


