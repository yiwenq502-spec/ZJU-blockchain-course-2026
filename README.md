# ZJU-blockchain-course-2026

# 实验操作过程
本项目的完整操作流程主要分为以下几个阶段：

## 1、启动本地区块链网络
打开Ganache软件并启动，准备好本地20个测试账户，并获取本地的RPC服务地址（本地地址为http://127.0.0.1:7545）。
<img width="1280" height="1964" alt="afde33ae4d3093f3d25d5da61460185f" src="https://github.com/user-attachments/assets/74cd826d-48db-4ad4-9427-055513cb945e" />

## 2、编写与部署智能合约
在Remix IDE中编写并编译了包含TokenA、TokenB、LPToken、SimpleDEX的智能合约。
通过选择外部HTTP提供者（Web3 Provider）连接到本地Ganache网络，并依次完成部署。
<img width="1080" height="720" alt="屏幕截图 2026-10-23 103705" src="https://github.com/user-attachments/assets/5700cac6-77c5-4d27-bb8d-62bfe9e446e7" />
<img width="477" height="365" alt="2f39d7e4aa6f57e540c2160a7ae3ec98" src="https://github.com/user-attachments/assets/ed96d507-15cc-4aa1-8201-125cb9411dc1" />
<img width="461" height="248" alt="4522c83b8e664e6c27c35e45147da222" src="https://github.com/user-attachments/assets/9c7e4174-0638-49a6-8071-2f9037271c1c" />
<img width="463" height="325" alt="ea1a961a262920c941a44e2b2064259c" src="https://github.com/user-attachments/assets/f9b26a40-9c24-4495-b4c2-f6d14ba9af0b" />
<img width="458" height="251" alt="66d6026bb67ee008dfa5d5abdfac0719" src="https://github.com/user-attachments/assets/f1917421-a476-427e-9d2d-e8119f81072e" />

## 3、钱包网络配置与账户导入
启动Metamask，并在MetaMask钱包中添加Ganache本地网络（RPC: `http://127.0.0.1:7545`，Chain ID: `1337`），并导入Ganache提供的测试账户私钥，每个账户拥有100ETH测试资金。

## 4、前端网页运行
在本地安装项目依赖，并运行如下命令：
      npm install web3
      npm run dev
启动React前端服务，并在浏览器中打开http://localhost:3000`。
<img width="449" height="128" alt="image" src="https://github.com/user-attachments/assets/2e929456-e4f7-47ab-9bdf-d0b4d6a9e1e5" />
<img width="1080" height="720" alt="670b6c751297197df97ee5ea2292533f" src="https://github.com/user-attachments/assets/8db60b0f-34f6-4b1d-b3f4-9b9cb66141e8" />
<img width="1080" height="720" alt="e1613bef9e6886705c14acd8d14cdba7" src="https://github.com/user-attachments/assets/07326e11-5298-4760-b42f-0a3789aadadb" />

## 5、功能测试与验证
在前端网页和Remix页面中进行交互测试，完成以下功能：
（1）领取测试币：调用合约的 `faucet` 方法，成功领取TokenA和TokenB。
（2）授权：授权 DEX 合约可以支配用户的代币。
（3）添加流动性：向池子注入代币，并通过ERC721合约成功获得了代表流动性份额的NFT凭证。
（4）执行交易：模拟用户执行`swapAForB`，用TokenA兑换TokenB。
<img width="1588" height="1024" alt="ec69c3716cbf7351df4312c0162cf12e" src="https://github.com/user-attachments/assets/71355bf4-b3f6-4d48-bdd0-8d357ebe0e7f" />
<img width="839" height="521" alt="599e363a96dec03d3ab8b0667a0fd78c" src="https://github.com/user-attachments/assets/9c18844e-587b-45c8-9578-0a4de68f2668" />
<img width="1660" height="1107" alt="image" src="https://github.com/user-attachments/assets/6ce3824e-efea-41f5-82d6-aaab8012f2d0" />
（5）交易结果验证：交易完成后，调用Remix页面的`balanceOf`查询TokenA余额。
可以看见结果显示，原本拥有 1,000,000 个TokenA，交易后余额显示为“999900000...”，证明10个TokenA已成功扣除并打入了交易池。
<img width="968" height="433" alt="e4783fec9bc15f0bbaee5022ce8a4b34" src="https://github.com/user-attachments/assets/1423cb59-5450-46c4-b523-71fa8a9fa477" />
<img width="314" height="132" alt="a29698639e8cd327d804a3c780aa1b2f" src="https://github.com/user-attachments/assets/9ec8ee54-709f-47fc-800c-599c9e3c04cb" />

---

## 功能实现分析
本项目完整实现了以下核心功能：

## 1、ERC20 Token 创建与领取
实现了TokenA和TokenB合约，内置了faucet()水龙头方法。根据前述截图展示，通过前端点击“领取测试币”，即可获取100个TokenA和100个TokenB用于交互测试。

代码实现：
function faucet() external { 
     _mint(msg.sender, 100 * 10 ** decimals()); 
}

     // 在LPToken (ERC721) 中，限制只有 DEX 合约才能铸造和销毁 NFT
     function mint(address to) external returns (uint256) {
     require(msg.sender == dex, "only dex");
     uint256 id = nextTokenId++;
     _mint(to, id);
     return id;
}

## 2、AMM交易池与恒定乘积定价，并提供流动性
通过SimpleDEX合约维护了reserveA和reserveB储备量。当用户发起swap交易时，合约基于恒定乘积公式（扣除 0.3% 手续费）自动计算兑换比例，并在交易后更新储备量，实现自动做市。
用户调用 addLiquidity 注入TokenA和TokenB后，DEX 会调用 LPToken 合约为用户Mint一个ERC721 NFT。该NFT记录了用户的流动性份额，作为后续退出池子的唯一凭证。

代码实现：
function addLiquidity(uint256 amountA, uint256 amountB) external {
    require(amountA > 0 && amountB > 0, "zero");
    // 将用户的代币转入 DEX 合约
    tokenA.transferFrom(msg.sender, address(this), amountA);
    tokenB.transferFrom(msg.sender, address(this), amountB);
    
    // 更新池子储备量
    reserveA += amountA; 
    reserveB += amountB;
    
    // 给用户铸造一个NFT作为凭证
    uint256 id = lp.mint(msg.sender);
    // 记录这个NFT代表的资金份额
    records[id] = Record(amountA, amountB);
}

## 3、移除流动性
也即合约的实现与接口验证，合约底层实现了 removeLiquidity(uint256 id) 方法。用户调用此方法时，系统会校验 NFT 所有权，随后销毁凭证，并根据当初记录的份额，将本金及分红退还给用户。为了保持前端界面的极简与核心逻辑清晰，前端暂未放置撤销按钮，但在实验测试中，已通过 Remix 的合约接口直接调用该函数，验证了这一逻辑：验证 NFT 所有权->销毁 NFT->根据份额将本金与累积的手续费退回给用户，完成资产的闭环流转。

代码实现：
function removeLiquidity(uint256 id) external {
    // 验证调用者确实是该 NFT 的拥有者
    require(lp.ownerOf(id) == msg.sender, "not owner");
    Record memory r = records[id];
    
    // 销毁凭证
    lp.burn(id);
    
    // 将代币退还给用户
    tokenA.transfer(msg.sender, r.amountA);
    tokenB.transfer(msg.sender, r.amountB);
    
    // 扣除池子中的储备量
    reserveA -= r.amountA; 
    reserveB -= r.amountB;
}



## 参考内容

- 课程的参考Demo见：[DEMOs](https://github.com/LBruyne/blockchain-course-demos)。

- 快速实现 ERC721 和 ERC20：[模版](https://wizard.openzeppelin.com/#erc20)。记得安装相关依赖，例如 ``"@openzeppelin/contracts": "^5.0.0"``。

- 如何实现ETH和ERC20的兑换？ [参考讲解](https://www.wtf.academy/en/docs/solidity-103/DEX/)

如果有其它参考的内容，也请在这里陈列。
