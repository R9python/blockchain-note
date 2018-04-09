准备工作
-----
        安装nodejs和npm
        安装git
        安装truffle: npm install -g truffle
        安装mist: https://github.com/Ethereum/Mist/releases
        安装geth: https://github.com/ethereum/go-ethereum/wiki/Installation-Instructions-for-Mac
        下载CryptoKitties代码: https://github.com/dapperlabs/cryptokitties-bounty

部署CryptoKitties
-----
1. 启动geth,并使之连接到rinkeby测试网络

        geth --rpccorsdomain '*' --rinkeby --rpc console --rpcapi "eth,net,web3,admin,personal"
        note: 如果是部署到公共的开发or生产服务器,可以使用参数来指定ip, port --rpcaddr 192.168.1.235 --rpcport 8545

2. 使用命令行启动Mist

        进入Applications目录, 运行 /Applications/Mist.app/Contents/MacOS/Mist --rpc http://localhost:8545
        note: 第一次启动会同步区块,需要较长时间,请耐心等待同步完成

3. 代码目录详解

        ABI: 定义外部有哪些接口可以调用
        contracts: 智能合约源代码
        migrations: 部署使用的js脚步
        scripts: client调用示例
        test: 测试
        truffle.js: 运行truffle migrate部署时会使用到该文件

4. 安装依赖

        运行 npm install, 生成node_modules

5. 编写migration文件用来部署

        var core = artifacts.require("./KittyCore.sol");
        var sale = artifacts.require("./Auction/SaleClockAuction.sol");
        var sire = artifacts.require("./Auction/SiringClockAuction.sol");
        var gene = artifacts.require("./ExternalInterface/GeneScience.sol");

        module.exports = function(deployer) {
                deployer.deploy(core)//部署core
                .then(function(){
                //core部署完成后,部署处理销售拍卖的智能合约,使用core的合约地址,并设置3.75%的tax
                deployer.deploy(sale, core.address, 375);
                })
                .then(function(){
                    //core部署完成后,部署处理繁殖拍卖的智能合约,使用core的合约地址,并设置3.75%的tax
                    deployer.deploy(sire, core.address, 375);
                })
                .then(function(){
                    //部署基因合约
                    deployer.deploy(gene);
                });
        };

6. 编写truffle.js

        module.exports = {
        networks: {
            // https://www.rinkeby.io/
            // https://blog.abuiles.com/blog/2017/07/09/deploying-truffle-contracts-to-rinkeby/
            rinkeby: {
            host: "localhost",
            port: 8545,
            from: "0x10F834Ed288CcDf11786a6b9d5f3dcFb3e83f415", // enter your local rinkeby unlocked address
            network_id: 4,
            gas: 4500000, // Gas limit used for deploys
            gasPrice: 10000000000 // 10 gwei
            }
        },
        solc: {
            optimizer: {
            enabled: true,
            runs: 200
            }
        },
        // https://truffle.readthedocs.io/en/beta/advanced/configuration/
        mocha: {
            bail: true
        }
        };

7. 使用truffle部署

        truffle migrate --network rinkeby
        note: 可以添加 --reset 参数强制更新部署

8. 部署完成后,会得到4个合约地址

        core
        0x1d6265585ab4f9d94c392736b781a3dd6da05fe5
        sale
        0x39bceff813f5fe5d083754a37f0ab6e07600645a
        siring
        0xaac0ba8ede39174fb6d7b82ca00f475bb63b0cec
        gene
        0xcf6df2d8b22470f25f64d1faa8910c3a8fc7c7e3

9. 对部署完成的合约进行设置

        设置销售拍卖的合约地址: set sale auction address
        设置繁殖拍卖的合约地址: set siring auction address
        设置基因合约地址: set gene science address
        启动: unpause (由于合约部署完成后,默认是paused状态,因此需要运行unpause)

玩转CryptoKitties
-----
1. COO每隔15分钟产生一个0代的猫，并进行拍卖（Main createGen0Auction()）
2. 用户可以购买0代猫（Sale Auction bid()）
3. 用户可以查询猫的数据（Main getKitty()）
4. 用户可以自己繁衍猫（Main breedWith() or breedWithAuto()）: gas:200000, value
5. 修养期后，用户可以得到新的猫（Main giveBirth()）
6. 用户可以把一只猫作为父亲，拍卖他的生育服务（Main createSiringAuction()）
7. 用户把一只猫作为父亲，为某个以太坊地址提供生育服务（Main approveSiring()）
8. 用户可以购买一只猫的生育服务（Main createSiringAuction()）
9. 用户可以拍卖他的猫（Main createSaleAuction()）
10. 用户可以购买被拍卖的猫（Sale Auction bid()）
11. 用户可以查看被拍卖猫的信息（Sale/Siring Auction getAuction()）
12. 用户能够取消拍卖（Sale/Siring Auction cancelAuction()）
13. 用户能够赠送猫（Main transfer()）
14. 用户能够指定另一个用户能够获得他的猫的权限（Main approve()）
15. 用户可以认领自己被指定获得权限的猫（Main transferFrom()）
16. 只有CEO能够替换COO或者CTO（Main setCEO() setCFO() setCOO()）
17. COO能够创建和操作特殊的猫（Main createPromoKitty()）
18. COO能够转移拍卖的收入（Main withdrawAuctionBalances()）
19. CFO能够转移主协议的收益（Main withdrawBalance()）


参考链接
-----
        https://blog.abuiles.com/blog/2017/06/13/smart-contracts-for-the-impatient/
        https://blog.abuiles.com/blog/2017/07/09/deploying-truffle-contracts-to-rinkeby/
        https://blog.abuiles.com/blog/2017/07/08/writing-smart-contracts-with-truffle/
