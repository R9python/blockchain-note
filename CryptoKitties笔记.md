准备工作
-----
        先翻墙出去
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
        备注: 如果是部署到公共的开发or生产服务器,可以使用参数来指定ip, port --rpcaddr 192.168.1.235 --rpcport 8545

2. 使用命令行启动Mist

        进入Applications目录, 运行 /Applications/Mist.app/Contents/MacOS/Mist --rpc http://localhost:8545
        备注: 第一次启动会同步区块,需要较长时间,请耐心等待同步完成

3. 代码目录详解

        ABI: 定义外部有哪些接口可以调用
        contracts: 智能合约源代码
        migrations: 部署使用的js脚本
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
        备注: 可以添加 --reset 参数强制更新部署; 这里的--network rinkeby就是告知命令行去truffle.js中找"rinkeby"这个节点的参数配置进行部署

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
        1. COO每隔15秒产生一个0代的猫，并放入拍卖市场（Core createGen0Auction()）
        2. 用户可以购买0代猫（Sale Auction bid()）
        3. 用户可以查询猫的数据（Core getKitty()）
        4. 用户可以繁衍自己的两只猫（Core breedWithAuto()）
        5. 怀孕期满后，生出新的猫（Core giveBirth()）
        6. 用户可以把一只猫作为父亲,放入繁殖市场（Core createSiringAuction()）
        7. 用户可以把一只猫作为母亲,去购买另一只猫的生育服务（Core bidOnSiringAuction()）
        8. 用户可以卖猫,放入卖猫的市场（Core createSaleAuction()）
        9. 用户可以购买在售的猫（Sale Auction bid()）
        10. 用户可以查看被拍卖猫的信息（Sale/Siring Auction getAuction()）
        11. 用户能够取消拍卖（Sale/Siring Auction cancelAuction()）
        12. 用户能够赠送猫（Core transfer()）
        13. 只有CEO能够设置COO, CFO（Core setCEO() setCFO() setCOO()）
        14. COO能够创建Promo猫（Core createPromoKitty()）
        15. COO能够创建0代猫,并放入拍卖市场（Core createGen0Auction()）

Q & A
-----
1. 如果账户被锁,即提示 account locked 怎么办?

        personal.unlockAccount('0x10F834Ed288CcDf11786a6b9d5f3dcFb3e83f415',"11111111",10000000)

2. kitty怀孕后,倒数多少时间才能生出小猫?

        取kitty的cooldownIndex, 然后在cooldowns数组中找到对应的时间,就是倒数时间.

3. gas limit 问题?

        gas limit 表示计算的步数限制, 调用方法的时候可以使用{gas: 250000}设置,该值一定要大于实际使用的数值.

4. rinkeby 网络上的账户没ether怎么办?

        在twitter上发推, 然后copy twitter link, 然后去 https://faucet.rinkeby.io/ 网站粘贴地址并领取ether.

5. 价格参数的单位是wei, ether和wei如何转换?

        //把0.000000002 ether转换为wei为单位的数值, 即 2000000000 wei
        web3._extend.utils.toWei(0.000000002,'ether');

        //把880000000000000000 wei转换为ether为单位的数值,即 0.88 ether
        web3._extend.utils.fromWei(880000000000000000,'ether');

参考链接
-----
        https://blog.abuiles.com/blog/2017/06/13/smart-contracts-for-the-impatient/
        https://blog.abuiles.com/blog/2017/07/09/deploying-truffle-contracts-to-rinkeby/
        https://blog.abuiles.com/blog/2017/07/08/writing-smart-contracts-with-truffle/
        http://truffleframework.com/docs/getting_started/migrations
        http://truffleframework.com/tutorials/pet-shop
        http://truffleframework.com/boxes/tutorialtoken
