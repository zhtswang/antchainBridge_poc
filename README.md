## 蚂蚁跨链桥总结

1. **背景**： 各自独立的区块链数据，需要通过数据的共享来创造更多的数据价值

   1.1 选择蚂蚁跨链桥的原因是因为IEEE3222-2023标准是由其提出的，并且是基于该标准实现的

   1.2 项目分为多个组件，技术栈如下

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

     - 插件服务

     - 中继服务

2. 
