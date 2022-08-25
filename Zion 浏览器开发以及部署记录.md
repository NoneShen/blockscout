# Zion 浏览器开发以及部署记录

# 开发

**服务端 [https://github.com/NoneShen/blockscout/tree/server_stable](https://github.com/NoneShen/blockscout/tree/server_stable)**

1. 修改向geth获取内部交易调用的RPC方法，新增方法 ”debug_traceJsTranasction”，不再为每一笔交易的请求传输大段的js代码，而是把js代码提前放在节点的代码中，大幅增加了获取内部交易的速度
https://github.com/NoneShen/blockscout/commit/85e05c9f752aced735bfcfea749b3ebd1f003d47
2. 在获取内部交易时，过滤掉调用zion内部合约的交易，并移除了在内部交易处理的模块中对于是否获取了所有交易的内部交易的检查
https://github.com/NoneShen/blockscout/commit/85e05c9f752aced735bfcfea749b3ebd1f003d47 https://github.com/NoneShen/blockscout/commit/da66080321330b67c5ca46d412269cfbba36b34f
3. 优化了在RPC节点不可访问时，浏览器的请求逻辑，加大请求延迟，修复了因为无法连接RPC节点会输出大量日志占满磁盘以及区块indexer异常退出的问题
https://github.com/NoneShen/blockscout/commit/560472da7a7c290a0688056f5dc43504ce656f57

**客户端 [https://github.com/NoneShen/blockscout/tree/client](https://github.com/NoneShen/blockscout/tree/client)**

1. 将前端msgId如ETH, Ether修改为ZION
 https://github.com/NoneShen/blockscout/commit/04fc0de191bf97ad50539ed19452d14f76320112
2. 移除由于内部交易未同步完而一直在页面顶部的警告横幅
 https://github.com/NoneShen/blockscout/commit/04fc0de191bf97ad50539ed19452d14f76320112
3. 将SOCKET_ROOT以及BLOCKSCOUT_HOST加入`content-security-policy`中，使得客户端可以访问服务端机器的websocket，避免由于ngnix转发导致客户端使用127.0.0.1作为host
 https://github.com/NoneShen/blockscout/commit/bb6377f1a32fe014046a63cafc31b87e5d2eab24 https://github.com/NoneShen/blockscout/commit/b4021e50b243089ba62765e1d6d374793ce139c1
4. 优化交易详情页面get请求无参数访问情况下的内部请求url，解决请求路径多一个/的问题
5. 修改GraphQL API的前端请求路径使得可以适配 /testnet 和/mainnet的额外路径配置
 https://github.com/NoneShen/blockscout/commit/c299bcb8dc3ad9d02a30b587b0c09edfb65616f2
6. 修改右上角搜索的请求链接，是的可以适配 /testnet 和/mainnet的额外路径配置
https://github.com/NoneShen/blockscout/commit/6552b3b1edb5c04e7abb382b76ead245e8e2807e
7. 修改客户端的indexer初始化模块，由原来的完全不启动任何模块变为仅仅启动CoinBalanceOnDemand，使得任意钱包地址的balance在有前端请求的时候定期的更新
    https://github.com/NoneShen/blockscout/commit/ecc65298c85a6a22a8ac6599425c56170bb112d2
8. 修改客户端的Graphql初始化模块，使得默认的websocket url带有/testnet或者/mainnet
 https://github.com/NoneShen/blockscout/commit/c4bdc8ed2964c65d5919828f17b3ceb118bec47e

# 部署

**服务端 docker配置**

1. 将NETWORK_PATH, SOCKET_ROOT, SECRET_KEY_BASE 配置进 Dockerfile，以保证项目编译中前端js代码可以获取到这两个参数
NETWORK_PATH和 SOCKET_ROOT 为 /testnet 或者 /mainnet
SECRET_KEY_BASE 本地 mix phx.gen.secret 生成
2. 环境变量
`ETHEREUM_JSONRPC_VARIANT` ：geth
`DATABASE_URL`: 密码不可以用&以外的特殊符号
`INDEXER_DISABLE_BLOCK_REWARD_FETCHER`: true
`INDEXER_DISABLE_EMPTY_BLOCK_SANITIZER`：true 
`BLOCKSCOUT_HOST`: 服务端机器的外网IP地址
`NETWORK_PATH=/`
`API_PATH=/`
`SOCKET_ROOT=/`
`IPC_PATH=` 留空
`PORT`:一定要和docker file中的PORT一致
`POOL_SIZE`  : 设置的越大，处理区块交易速度越快，硬件压力越大。
`POOL_SIZE_API` ：浏览器api对于数据库最大连接数
`OTHER_EXPLORERS` ：加入主网和测试网的链接使得可以互相切换
`ETHEREUM_JSONRPC_TRANSPORT` ：目前为http后期改成https
`BLOCKSCOUT_PROTOCOL` 留空
`LOGO` ：logo的路径
`LOGO_FOOTER` ： footer logo的路径
`NETWORK` ： 于浏览器页面标题有关
`CHAIN_SPEC_PATH` ： 链初始化文件的地址，用于获取验证人等信息

**客户端docker配置**

1. 将NETWORK_PATH, SOCKET_ROOT, SECRET_KEY_BASE， BLOCKSCOUT_HOST， API_PATH 配置进 Dockerfile，以保证项目编译中前端js代码可以获取到这三个参数
  NETWORK_PATH和 SOCKET_ROOT   /testnet 或者 /mainnet
  SECRET_KEY_BASE 由 mix phx.gen.secret 生成
  BLOCKSCOUT_HOST  explorer.zion.network
  API_PATH  /testnet 或者/mainnet
2. 移除docker file中所有的ARG条目
3. 环境变量
  `DISABLE_INDEXER=true` 把客户端indexer关闭
  `ETHEREUM_JSONRPC_VARIANT` ：geth
  `DATABASE_URL`: 密码不可以用&以外的特殊符号
  `INDEXER_DISABLE_BLOCK_REWARD_FETCHER`: true
  `INDEXER_DISABLE_EMPTY_BLOCK_SANITIZER`：true 
  `BLOCKSCOUT_HOST`: 域名`explorer.zion.network`
  `NETWORK_PATH=` ： /testnet或者/mainnet
  `API_PATH=` ：/testnet或者/mainnet
  `IPC_PATH=` 留空
  `PORT`:一定要和docker file中的PORT一致
  `POOL_SIZE`  : 设置的越大，支持的并发量越大
  `POOL_SIZE_API` ：浏览器api对于数据库最大连接数
  `OTHER_EXPLORERS` ：加入主网和测试网的链接使得可以互相切换
  `ETHEREUM_JSONRPC_TRANSPORT` ：目前为http后期改成https
  `BLOCKSCOUT_PROTOCOL` 目前为http后期改成https
  `LOGO` ：logo的路径
  `LOGO_FOOTER` ： footer logo的路径
  `NETWORK` ： 于浏览器页面标题有关
  `CHAIN_SPEC_PATH` ： 链初始化文件的地址，用于获取验证人等信息
  `CHAIN_ID=60803` 链的公网chainid
  `SECRET_KEY_BASE` 和dockerfile中一致
4. 从服务端复制静态文件到客户端
   1. 在服务端和客户端的机器上使用docker inspect --format='{{.GraphDriver.Data.MergedDir}}' <容器 ID> 找到容器的 merge 路径。
   2. 进入服务端merge路径下的opt/app/apps/block_scout_web/priv/static/js/，找到类似于 verification-form-612da4911c3b42e18d3981f15fd5fc36.js  和  verification-form-612da4911c3b42e18d3981f15fd5fc36.js.gz的两个文件（文件名中间的哈希不同）
   3. 将两个文件分别复制到客户端docker merged目录下的app/lib/block_scout_web-0.0.1/priv/static/js/文件夹中
      例如： /var/lib/docker/overlay2/07abc979947e4adfb54d91dcdcce19c8d1871c1e07a75f69eff8a3df3a659d4b/merged/app/lib/block_scout_web-0.0.1/priv/static/js/verification-form-612da4911c3b42e18d3981f15fd5fc36.js  /var/lib/docker/overlay2/07abc979947e4adfb54d91dcdcce19c8d1871c1e07a75f69eff8a3df3a659d4b/merged/app/lib/block_scout_web-0.0.1/priv/static/js/verification-form-612da4911c3b42e18d3981f15fd5fc36.js.gz

**测试网代理配置**

1. 将域名与客户端机器80端口绑定
2. 将对域名根目录的http请求 转到 /testnet/
3. 将对于客户端机器80端口 路径为/testnet/ 的http请求转到 客户端机器4000端口
4. 将对于客户端机器80端口 路径为/testnet/verify_smart_contract/contract_verifications 的http请求路径替换为服务端ip地址对应的同路径
5. 将对于客户端机器80端口 路径为/testnet/socket 的websocket请求路径替换为服务端ip地址对应的同路径