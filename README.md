# go
go文件
# 作业02

1. 完善数字身份智能合约，需要添加的功能如下：

   * 只有管理员可进行个人工作经历的添加（防止删除）

     （如果有想法的同学，该功能可以改进为个人可添加个人的工作经历，但需要管理员进行审核，审核通过后，再进行个人工作经历的添加）

   * 个人可以**授权**给其他人查看自己的个人经历与个人信息，并限制查看次数（若没有授权的人不能查看个人信息）

     ```
     
      pragma solidity ^0.6.0;
     pragma experimental ABIEncoderV2;
     
     contract PersonDID {
         struct Person {
             uint8 id;
             uint8 age;
             string name;
             // string info;
         }
     event AddPerson(uint8 id, uint8 age, string name, uint timestamp);
     
     Person admin;
     Person[] persons;
     mapping(address => Person) public PersonInfo;
     mapping(address=> bool) public isPersonExsist;
     
     constructor (address ip, uint8 id, string memory name, uint8 age) public {
         admin = Person(id, age, name);
         Person memory p = Person(id, age, name);
         persons.push(p);
         PersonInfo[msg.sender] = p;
         isPersonExsist[msg.sender] = true;
     }
     
     function getNumberOfPersons() view public returns (uint256) {
         return persons.length;
     }
     
     function addPerson(uint8 id, uint8 age, string memory name) public returns (bool) {
         require(!((id == 0) || age == 0), "persons info can not be empty!!");
         require(!isPersonExsist[msg.sender], "person can not exsist !!");
         Person memory person = Person(id, age, name);
         persons.push(person);
         PersonInfo[msg.sender] = Person(id, age, name);
         isPersonExsist[msg.sender] = true;
         emit AddPerson(id, age, name, now);
     }
     
     function setPersonAgeSto(address ip, uint8 age) public {
         Person storage p = PersonInfo[ip];
         p.age = age;
     }
     
     function setPersonAgeMem(address ip, uint8 age) public {
         Person memory p = PersonInfo[ip];
         p.age = age;
     }
     
     function getPersonAge(address ip) public view returns (uint8) {
         return PersonInfo[ip].age;
     }
     }
     ```

     **注：个人工作经历添加与授权查看功能需要有事件通知。**

2. 未完成`作业01`的同学，需要完善`作业01`中的智能合约，需要添加功能如下：

   * 注册功能（每人只能注册一个账号）

   * 修改密码功能（只有本人可以修改密码）

         pragma solidity ^0.6.0;
         pragma experimental ABIEncoderV2;
         contract userDID {
         address owner; 
         struct Person{
           bytes32 DID;
           string name;
           uint256 age;
           bool isExists;
         }
         mapping(address=>Person) persons; 
         uint256 autoID;
         
         constructor () public {
            autoID=1;
            owner=msg.sender;
         }
         
         function bindID(string memory _name,string memory _sex,uint256 _age) public {
             require(! persons[msg.sender].isExists,"caller must not exists");
             bytes32 did =keccak256(abi.encode(now,autoID));
             Person memory p= Person (did, _sex, _age, true);
             persons[msg.sender]=p;
             autoID ++;
             }
         
             function verifyPerson(bytes32 _did) public view returns(Person memory){
             Person memory p=persons[msg.sender];
             require(p.DID==_did);
             return p;
             }
             function getDID() public view returns(bytes32){
             return persons[msg.sender].DID;
             }
             }
     
     
     ​      
     ​      

3. 阅读以太坊源码中的`oracle.sol`智能合约，并编写该智能合约的分析说明文档

   注：需要说明每个函数的作用与功能，如果觉得有需要修改的地方也可以修改并改进，把分析与修改的文档说明整理成一个md文件。

   ```
   pragma solidity ^0.6.0;
   
   /**
    * @title CheckpointOracle
    * @author Gary Rong<garyrong@ethereum.org>, Martin Swende <martin.swende@ethereum.org>
    * @dev Implementation of the blockchain checkpoint registrar.
    */
   contract CheckpointOracle {     //合同检查点
       /*
           Events  //事件
   
       */
   
       // NewCheckpointVote is emitted when a new checkpoint proposal receives a vote.
       event NewCheckpointVote(uint64 indexed index, bytes32 checkpointHash, uint8 v, bytes32 r, bytes32 s);// //当新的检查点建议收到投票时，将发出NewCheckpointVote。
   
   事件NewCheckpointVote（uint64索引索引、bytes32检查点哈希、uint8 v、bytes32 r、bytes32 s）；
   
   
   
       /*
           Public Functions//公共职能
       */
   
       constructor(address[] memory _adminlist, uint _sectionSize, uint _processConfirms, uint _threshold) public {
           for (uint i = 0; i < _adminlist.length; i++) {
               admins[_adminlist[i]] = true;
               adminList.push(_adminlist[i]);
           }
           sectionSize = _sectionSize;
           processConfirms = _processConfirms;
           threshold = _threshold;
       }
   
       /**
        * @dev Get latest stable checkpoint information.//获取最新稳定的检查点信息。
        * @return section index//返回段索引
        * @return checkpoint hash//散列
        * @return block height associated with checkpoint//与检查点关联的返回块高度
        */
       function GetLatestCheckpoint()
       view
       public
       returns(uint64, bytes32, uint) {
           return (sectionIndex, hash, height);
       }
   
       // SetCheckpoint sets  a new checkpoint. It accepts a list of signatures
       设置新的检查点。它接受签名列表
       // @_recentNumber: a recent blocknumber, for replay protection
       一个最近的blocknumber，用于重播保护
       // @_recentHash : the hash of `_recentNumber`
       
       // @_hash : the hash to set at _sectionIndex
       要在\u sectionIndex设置的哈希
       // @_sectionIndex : the section index to set
       // @v : the list of v-values//:v值列表
       // @r : the list or r-values//r:列表或r值
       // @s : the list of s-values
       function SetCheckpoint(
           uint _recentNumber,
           bytes32 _recentHash,
           bytes32 _hash,
           uint64 _sectionIndex,
           uint8[] memory v,
           bytes32[] memory r,
           bytes32[] memory s)
           public
           returns (bool)
       {
           // Ensure the sender is authorized.
           require(admins[msg.sender]);
   
           // These checks replay protection, so it cannot be replayed on forks,
           // accidentally or intentionally
           require(blockhash(_recentNumber) == _recentHash);
   
           // Ensure the batch of signatures are valid.
           require(v.length == r.length);
           require(v.length == s.length);
   
           // Filter out "future" checkpoint.
           if (block.number < (_sectionIndex+1)*sectionSize+processConfirms) {
               return false;
           }
           // Filter out "old" announcement
           if (_sectionIndex < sectionIndex) {
               return false;
           }
           // Filter out "stale" announcement
           if (_sectionIndex == sectionIndex && (_sectionIndex != 0 || height != 0)) {
               return false;
           }
           // Filter out "invalid" announcement
           if (_hash == ""){
               return false;
           }
   
           // EIP 191 style signatures
           //
           // Arguments when calculating hash to validate
           // 1: byte(0x19) - the initial 0x19 byte
           // 2: byte(0) - the version byte (data with intended validator)
           // 3: this - the validator address
           // --  Application specific data
           // 4 : checkpoint section_index(uint64)
           // 5 : checkpoint hash (bytes32)
           //EIP 191样式签名
   
   //
   
   //计算要验证的哈希时的参数
   
   //1:字节（0x19）-初始0x19字节
   
   //2:字节（0）-版本字节（与预期验证器的数据）
   
   //3:这是验证器地址
   
   //--应用程序特定数据
   
   //4：检查点区段索引（uint64）
   
   //5:检查点哈希（bytes32）
   
   
           //     hash = keccak256(checkpoint_index, section_head, cht_root, bloom_root)
           bytes32 signedHash = keccak256(abi.encodePacked(byte(0x19), byte(0), this, _sectionIndex, _hash));
   
           address lastVoter = address(0);
   
           // In order for us not to have to maintain a mapping of who has already
           // voted, and we don't want to count a vote twice, the signatures must
           // be submitted in strict ordering.
           for (uint idx = 0; idx < v.length; idx++){
               address signer = ecrecover(signedHash, v[idx], r[idx], s[idx]);
               require(admins[signer]);
               require(uint256(signer) > uint256(lastVoter));
               lastVoter = signer;
               emit NewCheckpointVote(_sectionIndex, _hash, v[idx], r[idx], s[idx]);
   
               // Sufficient signatures present, update latest checkpoint.
               if (idx+1 >= threshold){
                   hash = _hash;
                   height = block.number;
                   sectionIndex = _sectionIndex;
                   return true;
               }
           }
           // We shouldn't wind up here, reverting un-emits the events
           revert();
       }
   
       /**
        * @dev Get all admin addresses
        * @return address list
        */
       function GetAllAdmin()
       public
       view
       returns(address[] memory)
       {
           address[] memory ret = new address[](adminList.length);
           for (uint i = 0; i < adminList.length; i++) {
               ret[i] = adminList[i];
           }
           return ret;
       }
   
       /*
           Fields//领域
       */
       // A map of admin users who have the permission to update CHT and bloom Trie root
       mapping(address => bool) admins;
   
       // A list of admin users so that we can obtain all admin users.
       address[] adminList;
   
       // Latest stored section id
       uint64 sectionIndex;
   
       // The block height associated with latest registered checkpoint.
       uint height;
   
       // The hash of latest registered checkpoint.
       bytes32 hash;
   
       // The frequency for creating a checkpoint
       //
       // The default value should be the same as the checkpoint size(32768) in the ethereum.
       uint sectionSize;
   
       // The number of confirmations needed before a checkpoint can be registered.
       // We have to make sure the checkpoint registered will not be invalid due to
       // chain reorg.
       //
       // The default value should be the same as the checkpoint process confirmations(256)
       // in the ethereum.
       uint processConfirms;
   
       // The required signatures to finalize a stable checkpoint.
       uint threshold;
   }
   ```

4. 把PPT后面的`references`都看一遍

https://learnblockchain.cn/2017/12/05/solidity1/  Solidity类型分为两类：值类型(Value Type) - 变量在赋值或传参时，总是进行值拷贝。引用类型(Reference Types)

https://learnblockchain.cn/2017/12/12/solidity2/  地址类型（Address）

https://learnblockchain.cn/2017/12/12/solidity_func/  函数类型

https://ethfans.org/posts/how-to-build-updateable-smart-contract-part-1 在区块链上建立可更新的智慧合约

https://ethfans.org/hh3755/articles/345  区块链Solidity语言函数可见性深入详解｜入门系列

https://ethfans.org/hh3755/articles/338 三分钟入门区块链语言 Solidity 函数

[https://docs.soliditylang.org/en/v0.8.3/units-and-global-variables.html?highlight=addmod#mathematical-and-cryptographic-functions ](https://docs.soliditylang.org/en/v0.8.3/units-and-global-variables.html?highlight=addmod)   Units and Globally Available Variables

官方文档:

https://learnblockchain.cn/docs/solidity/  Solidity 最新(0.8.0)中文文档





**作业提交方式：**

在自己的`git`上创建相应的项目，并将代码上传到自己的`git`上。每个题目需要写一个`Readme.md`文档说明介绍，自己合约函数的功能与特点，将对应作业的`git`地址发给我（微信或者邮箱都行）。

注：作业不要求全部解答，可根据自己的能力完成。

