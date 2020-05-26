[TOC]

# SystemContract

## 1.cnsManager

各个系统合约在本合约注册，用一个map保存各个合约的地址、合约名称、版本信息。

```c++
struct ContractInfo
{
    std::string name;    // 注册合约名
    std::string version; // 1.0.0.0
    std::string address; // 合约地址 0x...
    std::string origin;  // 创建者地址 0x... 暂保留，具体再讨论
    int64_t create_time;     // 合约创建时间
    bool enabled;        // CNS服务是否激活
    BCWASM_SERIALIZE(ContractInfo, (name)(version)(address)(origin)(create_time)(enabled));
};
```

### 1.1.注册合约 | cnsRegisterFromInit

 **int cnsRegisterFromInit(const char *name, const char *version)**

将已经部署的合约注册到本合约的中存储。保存所有的历史版本。

- 必须从init()函数中调用注册，如果从合约的非init方法中调用，则注册失败。

> input：
>
> - name：
>
>   ​	1.合约名，需要字母数字或者下划线打头
>
>   ​	2.合约名的所有者为第一次注册合约的部署者
>
> - version：
>
>   ​	1.合约版本：需满足[num].[num].[num].[num]的格式，如1.0.0.0
>
>   ​	2.注册版本必须逐步递增，不能回退注册。且版本号需要比已注销用户版本号高。
>
> output:
>
> -  0   注册成功
> -  1    注册失败

### 1.2.注册合约 |cnsRegister

  **int cnsRegister(const char *name, const char *version, const char *address)**

- 只有合约的所有者才能调用该接口
- 注册合约时，由用户或者合约的对外接口调用，不支持合约在自己的init()函数中调用。即如果需要在init()方法中调用统一使用cnsRegisterFromInit()接口。

> input:
>
> - name：
>
>   ​	1.合约名，需要字母数字或者下划线打头
>
>   ​	2.合约名的所有者为第一次注册合约的部署者
>
> - version：
>
>   ​	1.合约版本：需满足[num].[num].[num].[num]的格式，如1.0.0.0
>
>   ​	2.注册版本必须逐步递增，不能回退注册。且版本号需要比已注销用户版        本号高。
>
> - address：合约地址必须符合以太坊合约地址标准格式，"0x"前缀可写可不写。
>
> output:
>
> -  0 注册成功
> -  1  注册失败

### 1.3.注销特定合约 | cnsUnregister

**int cnsUnregister(char *name, char *version)**

- 注销为更改合约状态参数，并非真正“删除”。

- 只有合约的owner才能调用该接口

> input：
>
> - name：
>
>   ​	合约名，需要字母数字或者下划线打头
>
> - version：
>
>   ​	1.合约版本：需满足[num].[num].[num].[num]的格式，如1.0.0.0
>
>   ​	2."latest"代表最新版本,如果最高版本合约注销则选择第二高版本。
>
> output:
>
> -  0 注销成功
> -  1 注销失败

### 1.4. 获取合约地址 |getContractAddress

 **const char *getContractAddress(char *name, char *version) const**

- 非读写操作的获取信息任何人都可操作无需任何权限

> input：
>
> - name：
>
>   ​	合约名，需要字母数字或者下划线打头
>
> - version：
>
>   ​	1.合约版本：需满足[num].[num].[num].[num]的格式，如1.0.0.0
>
>   ​	2."latest"代表最新版本,如果最高版本合约注销则选择第二高版本。
>
> output:
>
> - address：标准以太坊地址（含“0x”前缀）

### 1.5.获取已注册合约 |getRegisteredContracts

**char *getRegisteredContracts(int pageNum, int pageSize) const**

> input：
>
> - pageNum：
>
>   ​	页面编号（从0开始），参数≥0。
>
> - pageSize：
>
>   ​	每页显示条目数, 参数≥1。
>
> output:
>
> - 合约信息的json字符串

### 1.6.获取某人已注册合约 | getRegisteredContractsByAddress

**char *getRegisteredContractsByAddress(char *origin, int pageNum, int pageSize) const**

> input：
>
> - origin：
>
>   ​	合约地址必须符合以太坊合约地址标准格式，“0x”前缀可写可不写
>
> - pageNum：
>
>   ​	页面编号（从0开始），参数≥0。
>
> - pageSize：
>
>   ​	每页显示条目数 , 参数≥1 。
>
> output:
>
> - 合约信息的json字符串

### 1.7. 是否已注册 |ifRegisteredByName

 **int ifRegisteredByName(char *name) const**

> input:
>
> - name :合约名，需要字母数字或者下划线打头
>
> output:
>
> - 0 未注册
> - 1 已注册

### 1.8. 是否已注册 |ifRegisteredByAddress

 **int ifRegisteredByAddress(char *address) const**

> input:
>
> - address：合约地址必须符合以太坊合约地址标准格式，"0x"前缀可写可不写
>
> output:
>
> - 0 未注册
> - 1 已注册

### 1.9.查询历史合约(包含已注销合约) | getHistoryContractsByName

   **char *getHistoryContractsByName(char *name) const**

> input:
>
> - name：合约名，需要字母数字或者下划线打头
>
> output:
>
> - 合约信息的json字符串

### 1.10.获取合约信息 | getContractInfoByAddress

  **char* getContractInfoByAddress(char *address) const**

> input:
>
> - address：合约地址必须符合以太坊合约地址标准格式，"0x"前缀可写可不写
>
> output:
>
> - 0 未注册
> - 1 已注册

## 2.userRegister

申请用户指的是，用户将个人的账户地址、用户名称、电话、邮箱、个人说明，向合约发起申请，合约保存信息。

```c++
	struct RegisterInfo {
		std::string user_address;     // 用户账户地址
		std::string name;             // 用户名称
		std::string mobile;           // 手机号
		std::string email;            // 邮箱
		std::string remark;           // 个人说明
		unsigned int user_state;      // 平台用户状态：1审核中，2已激活，3已拒绝
		std::string auditor_address;  // 审核人的地址
	//	std::string roles;            // 角色请求列表
		std::vector<string> roles;   // 角色请求列表
	};
```

### 2.1.添加用户申请信息 | registerUser

  **int registerUser(const char* registJson)**

- 需本人申请
- 限制申请次数1次，审核失败或者已激活用户无法再次申请

> input:
>
> - registJson ：
>
>   用户申请信息JSON结构体字符串，字符串长度>800则注册失败
>
> output:
>
> - 0 未注册
> - 1 已注册

Json字符串参考格式如下：

```json
'{"address":"0x33d253386582f38c66cb5819bfbdaad0910339b3","name":"xiaoluo","mobile":"13111111111","email":"luodahui@qq.com","roles":["a","b","c"],"remark":"平台用户申请"}'
```

### 2.2.审核 | approve

  **int approve(const char* userAddress, int auditStatus)**

- 对申请用户进行审核，审核成功后在待审核中列表中删除。审核人需为有效用户及管理员。

> input:
>
> - userAddress 用户地址。
>
> - auditStatus 审核状态： 2 激活，3 拒绝。
>
> output:
>
> - int 审核成功返回0，失败返回-1。

### 2.3.查询账户信息 | getAccountByAddress

**const char* getAccountByAddress(const char* address) const**

> input:
>
> - address 用户地址
>
> output:
>
> - const char* 返回json格式字符串，格式如下：
>
>   ```json
>   '{"code":0,"msg":"succeed","data":{ ....}}'
>   ```
>
>   如果用户申请不存在，返回错误信息：
>
>   ```json
>   '{"code":1,"msg":"The user does not exist in the userRegisterManager", "data":""}'
>   ```

### 2.4.查询账户信息 | getAccountByUsername

**const char* getAccountByUsername(const char* UserName) const**

> input:
>
> - UserName 用户名
>
> output:
>
> - const char* 返回json格式字符串，格式如下：
>
>   ```json
>   '{"code":0,"msg":"succeed","data":{ ....}}'
>   ```
>
>   如果用户申请不存在，返回错误信息：
>
>   ```json
>   '{"code":1,"msg":"The user does not exist in the userRegisterManager", "data":""}'
>   ```

### 2.5.分页查询某状态的用户 | getAccountsByStatus

**const char* getAccountsByStatus(int pageNum, int pageSize, int accountStatus) const**

> input:
>
> - pageNum 页码 ，参数≥0。
> - pageSize 每页的数据条数，参数≥1。
> - accountStatus 用户状态：1审核中，2已激活，3已拒绝
>
> output:
>
> - const char* 返回json格式字符串，格式如下：
>
>   ```json
>   '{"code":0,"msg":"succeed","data":{ ....}}'
>   ```
>
> 错误输出：
>
> ```c++
>  unsigned int  startIndex = pageNum * pageSize;
>  unsigned int endIndex = startIndex + pageSize - 1;
>        if (startIndex >= size) 
>            {
>                code = "1";
>              message = "Adjust pageNum and pageSize";
>             }
> ```
>
> ```c++
>  {
>           //  bcwasm::println("状态用户不存在：[", accountStatus, "],请检查");
>                  code = "2";
>                 message = "The user status for the query does not exist in the UserRegister";
>                  userInfos = "\"\"";
>    }
> ```
>
> ```c++
>   {
>                  code = "3";
>                  message = "user information is empty in the UserRegister";
>                   userInfos = "\"\"";
>  }
> ```

### 2.6.查询用户状态 | getStatusByAddress

  **int getStatusByAddress(const char* address) const** 

> input:
>
> - address：用户地址
>
> output:
>
> - int 查询成功返回用户状态1,2,3，失败返回-1。

## 3.userManager

用来管理平台用户的邮箱，手机号及用户状态。

```c++
    struct UserInfo
    {
        string user_address;    // 用户地址
        string name;//用户名称 
        string email; //用户邮箱 
       string mobile; //手机号
        unsigned int status;        // 是否禁用用户：[0:可用; 1:禁用; 2:删除] 
    };
```

### 3.1.添加用户 | addUser

**int addUser(const char* userJson)**

- 调用者需为有效用户及管理员，用户需为已激活用户。

> input:
>
> - userJson 用户信息JSON结构体字符串
>
> output:
>
> - int 添加用户成功返回0，失败返回-1.

Json字符串参考格式如下：

```json
'{"address":"0x33d253386582f38c66cb5819bfbdaad0910339b3","name":"xiaoluo","mobile":"13111111111","email":"luodahui@qq.com","roles":["a","b","c"],"remark":"平台用户申请"}'
```

### 3.2.设置用户可用 | enable

 **int enable(const char* userAddr)**

- 调用者需为可用用户且role为管理员。其中超管状态不可被修改恒为可用状态，普管状态可由任何普管（包括自己）和超管修改。
- 已删除用户状态不能被变更，仅有可用和不可用状态可以被修改。

> input:
>
> - userAddr 用户地址
>
> output:
>
> - int 设置成功返回0，失败返回-1.

### 3.3.禁用用户 | disable

**int disable(const char* userAddr)**

> input:
>
> - userAddr 用户地址
>
> output:
>
> - int 设置成功返回0，失败返回-1.

### 3.4.删除用户 | delUser

 **int delUser(const char* userAddr)**

- 已删除用户不能变更状态

> input:
>
> - userAddr 用户地址
>
> output:
>
> - int 设置成功返回0，失败返回-1.

### 3.5.更新用户信息  | update

 **int update(const char* userAddr, const char* updateJson)**

功能：更新用户信息，只能更新用户的email,mobile信息

- 每个可用用户都可以修改自己的信息。

- 修改他人信息时，调用者需为有效用户且为管理员，其中超管有修改一切的权限，普管仅能修改普通用户。且用户需为有效用户。

> input:
>
> - userAddr 用户地址
> - updateJson 更新信息的json结构体字符串
>
> output:
>
> - int 添加用户成功返回0，失败返回-1.

### 3.6.获取用户信息 | getAccountByName

 const char* getAccountByName(const char* name) const

> input:
>
> - name 用户名
>
> output:
>
>  - const char* 返回json格式字符串，格式如下：
>   
>   ```json
>    '{"code":0,"msg":"succeed","data":{ ....}}'
>    ```
>   
>   ​	如果用户申请不存在，返回错误信息：
>   
>    ```json
>    '{"code":1,"msg":"The user does not exist in the userRegisterManager", "data":""}'
>    ```
>

### 3.7.是否有效 | isValidUser

  **int isValidUser(const char* userAddr) const**

> input:
>
> - userAddr 用户地址
>
> output:
>
> - int 返回0为有效用户，-1为非有效用户

### 3.8.获取用户信息 | getAccountByAddress

 **const char* getAccountByAddress(const char* address) const**

> input:
>
> - address 用户地址
>
> output:
>
> - const char* 返回json格式字符串，格式如下：
>
>    ```json
>    '{"code":0,"msg":"succeed","data":{ ....}}'
>    ```
>   
>    ​	如果用户申请不存在，返回错误信息：
>   
>   ```json
>    '{"code":1,"msg":"The user does not exist in the userRegisterManager", "data":""}'
>   ```
>

## 4.roleRegister

用以申请角色、审批角色、以及查询角色申请信息。

```c++
 typedef struct RegisterUserInfo {
       std::string userAddress;       //账户地址
        std::string userName;            //用户名
        int roleRequireStatus;  //角色申请状态 1 申请中 2 已批准 3 已拒绝
        std::vector<std::string> requireRoles;
        std::string approver;         //审核人的地址
        BCWASM_SERIALIZE(RegisterUserInfo, (userAddress)(userName)(roleRequireStatus)(requireRoles)(approver));
    }RegisterUserInfo_st;
```

具体角色及权限请参照PlatONE合约指导文档。

### 4.1.注册角色 | registerRole

  **int registerRole(const char* roles)** 

- 记录用户申请信息,一个账户可以同时申请多个角色，只能申请一次。

> input:
>
> - inputs : const char* roles
>
>
>   eg: [ \"chainAdmin\",\"nodeAdmin\"]
>
> output:
>
> - return 含义如下：
>    0 注册成功
>    1 地址状态不正确
>    4 输入参数不正确，不是list类型
>    6 没有有效的申请信息
>    7 角色名称无效

### 4.2.审批角色 | approveRole

  int approveRole(const char* address, int status)

只能审核比自己权限更低的角色申请。 

> input: address ， status
>
> output:
>
> - return 含义如下：
>
>   0 注册成功
>
>   1 账号地址格式错误
>
>   1审批人用户状态不正确
>
>   2 角色权限不足
>
>   3 没有角色申请信息 
>
>   4 未找到角色管理合约
>
>   5 不是合法的审批值
>
>   6 当前是非审批状态 
>
>   7 添加角色失败

### 4.3.查询角色申请信息 | getRegisterInfoByAddress

const char * getRegisterInfoByAddress(const char* address) const

> input: address
>
> output:
>
> - const char* 返回json格式字符串 ： {状态值, msg, RegisterUserInfo}

output返回格式如下：

```json
{
    code: 0,
    msg: "ok",
    data: {
        "userAddress": "0x111222",
        "userName": "xiaoming",
        "roleRequireStatus": 1,
        "requireRoles":["Admin", "ContractCaller"],
        "approveor": "01xxx"
    }
}
```

错误输出格式：

```json
{
    code:1,
    msg: "not found",
    data: {
    }
}
```

### 4.4.查询角色申请信息 | getRegisterInfoByName

 const char * getRegisterInfoByName(const char* name) const

> input: name
>
> output:
>
> - const char* 返回json格式字符串 ： {状态值, msg, RegisterUserInfo}

### 4.5.查询角色申请信息 | getRegisterInfosByStatus

 const char * getRegisterInfosByStatus(int status, int pageNum, int pageSize) const

> input:
>
> - pageNum 页码 ，参数≥0。
> - pageSize 每页的数据条数，参数≥1。
> - accountStatus 用户状态：1审核中，2已激活，3已拒绝
>
> output:
>
> - const char* 返回json格式字符串

## 5.roleManager

用来管理平台用户的角色。

```c++
namespace SystemContract
{
    struct userRole{
        bcwasm::Address address;
        string name;
        vector<string> roles;
        BCWASM_SERIALIZE(userRole,(address)(name)(roles));
    };
```

### 5.1.添加角色 | addRole

int addRole(const char *name, const char *address, const char *roles)

- 对相同地址来说，如果已存在并用户名一样，则更新角色；用户名不一样返回6

> input: name ， address , roles
>
> output:
>
> - return 含义如下：
>
>   0  Add roles success
>
>   1 Address format is invalid.
>
>   2 Caller unavailable, status: 
>
>   3  User unavailable, status: 
>
>   4 Roles invalid: 
>
>   5 No permission to remove/addrole for roles: 
>
>   6  Name must be unique
>
>   7 Add Roles failed 

### 5.2.删除角色 | removeRole

  int removeRole(const char *address,const char *roles)

- 调用者需为普管或者超管。其中超管状态不可被修改恒为可用状态，普管状态可由任何普管（包括自己）和超管修改。

> input: address , roles
>
> output:
>
> - return 含义如下：
>
>   0	Remove roles success. 
>
>   1     Address format is invalid. 
>
>   2 	Caller unavailable, status: "+ to_string(userStatus) + " , Address: " + string(address)
>
>   3	 Cannot found address: " +  string(address)
>
>   4	Roles invalid: " + string(roles)
>
>   5	No permission to remove for roles: " + string(roles)

### 5.3.查询角色 | getRolesByAddress

const char* getRolesByAddress(const char *address)const

> input:
>
> - address 用户地址
>
> output:
>
> - const char* 返回json格式字符串，格式如下：
>
> ```json
> '{"code":0,"msg":"Success","data":{ ....}}'
> ```
>
> ​	错误返回如下两种情况：
>
> ```json
> '{"code":1,"msg":"Address format is invalid.", "data":""}'     
> '{"code":1,"msg":"Not Found", "data":""}'
> ```

### 5.4.查询角色 | getRolesByName

 const char* getRolesByName(const char *name)const

> input:
>
> - address 用户地址
>
> output:
>
> - const char* 返回json格式字符串，格式如下：
>
> ```json
> '{"code":0,"msg":"Success","data":{ ....}}'
> ```
>
> ​	错误返回如下：
>
> ```json
> '{"code":1,"msg":"Not Found", "data":""}'
> ```

### 5.5.查询账户 | getAccountsByRole

 const char* getAccountsByRole(const char *role)const

> input:
>
> - address 用户地址
>
> output:
>
> - const char* 返回json格式字符串，格式如下：
>
> ```json
> '{"code":0,"msg":"Success","data":{ ....}}'
> ```
>
> ​	错误返回如下：
>
> ```json
> '{"code":1,"msg":"Not Found", "data":""}'
> '{"code":1,"msg":"role parameter invalid", "data":""}'
> ```

### 5.6.是否有角色 | hasRole

 int hasRole(const char* addr, const char* role) const

> input:
>
> - address 用户地址
> - role  角色json格式字符串
>
> output:
>
> - int 返回0为没有，1为有

## 6.nodeRegister

```c++
struct RegisterInfo
{
    std::string name;      //节点名称
    std::string owner;     //申请者的地址
    std::string desc;      //节点描述,长度限制1000
    int type;              //1:共识节点；0:观察者节点
    std::string publicKey; //公钥
    std::string externalIP;
    std::string internalIP;
    int rpcPort;
    int p2pPort;
    int status;           //0:未处理；1:申请通过；2:拒绝申请
    bool root;            //true 根节点 false 非跟节点
    std::string approver; //审核人的地址
    int64_t registerTime; //申请时间
    BCWASM_SERIALIZE(RegisterInfo, (name)(owner)(desc)(type)(publicKey)(externalIP)(internalIP)(rpcPort)(p2pPort)(status)(root)(approver)(registerTime));
};
```

### 6.1.注册节点信息 | registerNode

int registerNode(const char *nodeJson)

- 只能由chainCreater、chainAdmin、nodeAdmin调用
-  name：只可包含字母、数字、下划线、连字符，审核通过的name不可以重复
- desc：长度限制为4-1000
- type：必须是0或者1，0表示观察者节点
- publicKey：长度限制为4-1000，publicKey不能重复申请
-  IP和端口：任意组合审核通过后，不能重复申请

> inputs: 
>
> RegisterInfo是Json字符串，包含以下内容：
>
> ```json
> '{
>     "name":"node1",
>     "desc":"i love this world",
>     "type":1,
>   "publicKey":"acb2281452fb9fc25d40113fb6afe82b498361de0eea6449c2502",
>     "externalIP":"127.0.0.1",
>     "internalIP":"127.0.0.1",
>     "rpcPort":4789,
>     "p2pPort":14789
> }'
> ```
>
> output: int
>
> ​	0	成功
>
> ​	1	参数不规范
>
> ​	2	无权限

### 6.2.审核节点申请 | approve

 int approve(char *publicKey, int status)

-  只能由chainCreater、chainAdmin调用
-  如果状态是已审核，不能重新审核
-  审核通过的name、IP+Port不可以再次审核通过
-  当审核通过（status = 1）后，系统调用NodeManager合约的add()方法把节点信息放入节点管理合约

> inputs: 
>
> - publicKey:不能为空，如果没申请过则无效
> -  status：必须是1或者2，1表示通过，2表示拒绝
>
> output: int
>
> ​	0	成功
>
> ​	1	参数不规范
>
> ​	2	无权限

### 6.3.获取申请信息 | getRegisterInfoByStatus

const char *getRegisterInfoByStatus(int status, int pageNum, int pageSize) const

> inputs: 
>
> - status:查询状态，0申请未处理、1申请通过、2申请拒绝
> - pageNum：页面编号（从0开始）
> - pageSize：每页显示条目数
>
> output: 
>
> ```json
> {
>     "code":0,
>     "msg":"",
>     "data":{
>         "name":"node1",
>         "owner":"0x4FCd6fe35f0612C7866943cb66C1d93eb0746bcC",
>         "desc":"i love this world",
>         "type":1,     "publicKey":"acb2281452fb9fc25d40113fb6afe82b498361de0eea6449c2502",
>         "externalIP":"127.0.0.1",
>         "internalIP":"127.0.0.1",
>         "rpcPort":4789,
>         "p2pPort":14789,
>         "status":0,        "approveor":"0x4FCd6fe35f0612C7866943cb66C1d93eb0746bcC",
>         "registerTime":23489723
>     }
> }
> ```
>
> 错误输出格式：
>
> ```json
> {
>     code:1,
>     msg: "Can't find info",
>     data: {
>     }
> }
> ```
>
> ```json
> {
>     code:2,
>     msg: "Parameter error!",
>     data: {
>     }
> }
> ```

### 6.4.获取申请信息 | getRegisterInfoByPublicKey

 const char *getRegisterInfoByPublicKey(char *publicKey) const

> inputs: 
>
> - publicKey：申请过的公钥
>
> output:  返回Json字符串

### 6.5.获取申请信息 | getRegisterInfoByOwnerAddress

 const char *getRegisterInfoByOwnerAddress(char *owner) const

> inputs: 
>
> - ownerAddress：申请者地址
>
> output:  返回Json字符串

## 7.nodeManager

```c++
//此结构体已注释掉，代码部分不是通过结构体方式定义以下变量
{
    string name;       // 节点名字
    address owner;     // 申请者的地址
    string desc;       // 节点描述
    int type;          // 0:观察者节点；1:共识节点
    string publicKey;  // 节点公钥
    string externalIP; // 外网 IP
    string internalIP; // 内网 IP
    int rpcPort;       // rpc 通讯端口
    int p2pPort;       // p2p 通讯端口
    int status;        // 1:正常；2：删除
    address approveor; // 审核人的地址
    int delayNum;      // 共识节点延迟设置的区块高度 (可选, 默认实时设置)
}
```

### 7.1.添加节点信息 | add

  int add(const char *nodeJsonStr)

- 必要的key:{"name", "owner", "desc", "type", "publicKey", "externalIP", "internalIP", "rpcPort", "p2pPort", "status"}
- 只有chainCreator，chainAdmin，nodeAdmin可添加节点信息
- 公钥必须唯一
- 节点名字不能与非删除节点重复
- 添加节点时类型必须是观察者节点，即type=0
-  添加节点的status只能是1或2

> inputs: 
>
> - nodeJsonStr 节点信息JSON结构体字符串，如：
>
> ```json
> '{
>     "name": "node1",
>     "owner": "0x4FCd6fe35f0612C7866943cb66C1d93eb0746bcC",
>     "approveor": "0xdF6518e51e0d90A3CBa26e4AdFE74454E2D90BdE",
>     "desc": "i love this world",
>     "type": 1,
>     "publicKey": "0x81ec63a2335c1f79244cbe485eb6bffef48cfd7df58b1009411c6114670eefd27da865914c70f7e49ceeb1002f1c24f4930975a2eb05cb5ac1373bed83a9932a",
>     "externalIP": "127.0.0.1",
>     "internalIP": "127.0.0.1",
>     "rpcPort": 6789,
>     "p2pPort": 16789,
>     "status": 1
> }'
> ```
>
> output: int
>
> ​	0	成功
>
> ​	1	参数不规范
>
> ​	2	无权限

### 7.2.查询所有节点信息  | getAllNodes

 const char *getAllNodes() const

> inputs: 无
>
> outputs：
>
> ​	nodes 所有节点信息JSON结构体字符串
>
> ```json
> {
>   "code": 0,
>   "msg": "success",
>   "data": [...]
>            }
> ```

### 7.3.查询节点信息 | getNodes

   const char *getNodes(const char *nodeJsonStr) const

> inputs: 
>
> ​	根据特定条件，返回符合条件的节点信息。比如，我需要查询节点的名字为"node1"的节点信息，那么传入字符串 {"name":"node1"}；如果需要查询所有正常节点，那么传入字符串{"status":1}；如果我需要返回正常节点而且是共识节点，那么传入字符串{"status":1, "type":1}。总之，你可以根据节点信息可多个组合进行查询。
>
> outputs：
>
> ​	nodes 所有符合条件的节点信息JSON结构体字符串
>
> ```json
> {
>   "code": 0,
>   "msg": "success",
>   "data": [...]
>            }
> ```

### 7.4.符合条件节点个数 | nodesNum

  int nodesNum(const char *nodeJsonStr) const

> inputs: 
>
> ​	一些条件的JSON结构体字符串
>
> outputs：
>
> ​	nodes 所有符合条件的节点个数。

### 7.5.更新信息 | update

int update(const char *name, const char *nodeJsonStr)

- CHAIN_CREATOR，CHAIN_ADMIN，NODE_ADMIN才有权限update
- 非正常状态的节点, 不能进行更新（status!=1）
- 节点status 只能从1更新为2
- 节点type 只能在0和1之间进行更新
- 更新节点信息只能是：desc, type, status, delayNum

> inputs: 
>
> - name 节点名字
> - nodeJsonStr 需要更新的信息，以json表示。
>
> output: int
>
> ​	0	失败
>
> ​	1	成功

### 7.6.某公钥有效节点个数 | validJoinNode

  int validJoinNode(const char *publicKey) const

> inputs: 
>
> - publicKey
>
> output: 
>
> - int   输出某公钥下所有有效的（status=1）节点个数

### 7.7.正常状态节点个数 | getNormalEnodeNodes

const char *getNormalEnodeNodes() const

> inputs: 无
>
> output: 
>
> - int   输出正常状态的节点个数

### 7.8.删除状态节点个数 | getDeletedEnodeNodes

const char *getDeletedEnodeNodes() const

> inputs: 无
>
> output: 
>
> - int   输出删除状态的节点个数