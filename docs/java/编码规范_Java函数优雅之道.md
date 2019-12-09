## 编码规范 | Java函数优雅之道

**1.**

**导读**

------

随着软件项目代码的日积月累，系统维护成本变得越来越高，是所有软件团队面临的共同问题。持续地优化代码，提高代码的质量，是提升系统生命力的有效手段之一。软件系统思维有句话“Less coding, more thinking（少编码、多思考）”，也有这么一句俚语“Think more, code less（思考越多，编码越少）”。所以，我们在编码中多思考多总结，努力提升自己的编码水平，才能编写出更优雅、更高质、更高效的代码。

本文总结了一套与Java函数相关的编码规则，旨在给广大Java程序员一些编码建议，有助于大家编写出更优雅、更高质、更高效的代码。这套编码规则，通过在高德采集部门的实践，已经取得了不错的成效。

**2.**

**使用通用工具函数**

------

**2.1 案例一**

**现象描述：**

不完善的写法：

```
thisName != null && thisName.equals(name);
```

更完善的写法：

```
(thisName == name) || (thisName != null && thisName.equals(name));
```

**建议方案：**

```
Objects.equals(name, thisName);
```

**2.2 案例二**

**现象描述：**

```
!(list == null || list.isEmpty());
```

**建议方案：**

```
import org.apache.commons.collections4.CollectionUtils;
CollectionUtils.isNotEmpty(list);
```

**2.3 主要收益**

- 函数式编程，业务代码减少，逻辑一目了然；
- 通用工具函数，逻辑考虑周全，出问题概率低。

**3.**

**拆分超大函数**

------

当一个函数超过80行后，就属于超大函数，需要进行拆分。

**3.1 案例一：每一个代码块都可以封装为一个函**

每一个代码块必然有一个注释，用于解释这个代码块的功能。

如果代码块前方有一行注释，就是在提醒你——可以将这段代码替换成一个函数，而且可以在注释的基础上给这个函数命名。如果函数有一个描述恰当的名字，就不需要去看内部代码究竟是如何实现的。

**现象描述：**

``

```
// 每日生活函数
public void liveDaily() {
    // 吃饭
    // 吃饭相关代码几十行

    // 编码
    // 编码相关代码几十行

    // 睡觉
    // 睡觉相关代码几十行
}
```

**建议方案：**

```
// 每日生活函数
public void liveDaily() {
    // 吃饭
    eat();

    // 编码
    code();

    // 睡觉
    sleep();
}

// 吃饭函数
private void eat() {
    // 吃饭相关代码
}

// 编码函数
private void code() {
    // 编码相关代码
}

// 睡觉函数
private void sleep() {
    // 睡觉相关代码
}
```

**3.2 案例二：每一个循环体都可以封装为一个函**

**现象描述：**

```
// 生活函数
public void live() {
    while (isAlive) {
        // 吃饭
        eat();

        // 编码
        code();

        // 睡觉
        sleep();
    }
}
```

**建议方案：**

```
// 生活函数
public void live() {
    while (isAlive) {
        // 每日生活
        liveDaily();
    }
}

// 每日生活函数
private void liveDaily() {
    // 吃饭
    eat();

    // 编码
    code();

    // 睡觉
    sleep();
}
```

**3.3 案例三：每一个条件体都可以封装为一个函**

**现象描述：**

```
// 外出函数
public void goOut() {
    // 判断是否周末
    // 判断是否周末: 是周末则游玩
    if (isWeekday()) {
        // 游玩代码几十行
    }
    // 判断是否周末: 非周末则工作
    else {
        // 工作代码几十行
    }
}
```

**建议方案：**

```
// 外出函数
public void goOut() {
    // 判断是否周末
    // 判断是否周末: 是周末则游玩
    if (isWeekday()) {
        play();
    }
    // 判断是否周末: 非周末则工作
    else {
        work();
    }
}

// 游玩函数
private void play() {
    // 游玩代码几十行
}

// 工作函数
private void work() {
    // 工作代码几十行
}
```

**3.4 主要收益**

- 函数越短小精悍，功能就越单一，往往生命周期较长；
- 一个函数越长，就越不容易理解和维护，维护人员不敢轻易修改；
- 在过长函数中，往往含有难以发现的重复代码。

**4.**

**同一函数内代码块级别尽量一致**

------

**4.1 案例一**

**现象描述：**

```
// 每日生活函数
public void liveDaily() {
    // 吃饭
    eat();

    // 编码
    code();

    // 睡觉
    // 睡觉相关代码几十行
}
```

很明显，睡觉这块代码块，跟eat（吃饭）和code（编码）不在同一级别上，显得比较突兀。如果把写代码比作写文章，eat（吃饭）和code（编码）是段落大意，而睡觉这块代码块属于一个详细段落。而在liveDaily（每日生活）这个函数上，只需要写出主要流程（段落大意）即可。

**建议方案：**

```
public void liveDaily() {
    // 吃饭
    eat();

    // 编码
    code();

    // 睡觉
    sleep();
}

// 睡觉
private void sleep() {
    // 睡觉相关代码
}
```

**4.2 主要收益**

- 函数调用表明用途，函数实现表达逻辑，层次分明便于理解；
- 不用层次的代码块放在一个函数中，容易让人觉得代码头重脚轻。

**5.**

**封装相同功能代码为函数**

------

**5.1 案例一：封装相同代码为函数**

**现象描述：**

```
// 禁用用户函数
public void disableUser() {
    // 禁用黑名单用户
    List<Long> userIdList = queryBlackUser();
    for (Long userId : userIdList) {
        User userUpdate = new User();
        userUpdate.setId(userId);
        userUpdate.setEnable(Boolean.FALSE);
        userDAO.update(userUpdate);
    }

    // 禁用过期用户
    userIdList = queryExpiredUser();
    for (Long userId : userIdList) {
        User userUpdate = new User();
        userUpdate.setId(userId);
        userUpdate.setEnable(Boolean.FALSE);
        userDAO.update(userUpdate);
    }
}
```

**建议方案：**

```
// 禁用用户函数
public void disableUser() {
    // 禁用黑名单用户
    List<Long> userIdList = queryBlackUser();
    for (Long userId : userIdList) {
        disableUser(userId);
    }

    // 禁用过期用户
    userIdList = queryExpiredUser();
    for (Long userId : userIdList) {
        disableUser(userId);
    }
}

// 禁用用户函数
private void disableUser(Long userId) {
    User userUpdate = new User();
    userUpdate.setId(userId);
    userUpdate.setEnable(Boolean.FALSE);
    userDAO.update(userUpdate);
}
```



**5.2 案例二：封装相似代码为函数**

封装相似代码为函数，差异性通过函数参数控制。

**现象描述：**

```
// 通过工单函数
public void adoptOrder(Long orderId) {
    Order orderUpdate = new Order();
    orderUpdate.setId(orderId);
    orderUpdate.setStatus(OrderStatus.ADOPTED);
    orderUpdate.setAuditTime(new Date());
    orderDAO.update(orderUpdate);
}

// 驳回工单函数
public void rejectOrder(Long orderId) {
    Order orderUpdate = new Order();
    orderUpdate.setId(orderId);
    orderUpdate.setStatus(OrderStatus.REJECTED);
    orderUpdate.setAuditTime(new Date());
    orderDAO.update(orderUpdate);
}
```

**建议方案：**

```
// 通过工单函数
public void adoptOrder(Long orderId) {
    auditOrder(orderId, OrderStatus.ADOPTED);
}

// 驳回工单函数
public void rejectOrder(Long orderId) {
    auditOrder(orderId, OrderStatus.REJECTED);
}

// 审核工单函数
private void auditOrder(Long orderId, OrderStatus orderStatus) {
    Order orderUpdate = new Order();
    orderUpdate.setId(orderId);
    orderUpdate.setStatus(orderStatus);
    orderUpdate.setAuditTime(new Date());
    orderDAO.update(orderUpdate);
}
```

**5.3 主要收益**

- 封装公共函数，减少代码行数，提高代码质量；
- 封装公共函数，使业务代码更精炼，可读性可维护性更强。

**6.**

**封装获取参数值函数**

------

**6.1 案例一**

**现象描述：**

```
// 是否通过函数
public boolean isPassed(Long userId) {
    // 获取通过阈值
    double thisPassThreshold = PASS_THRESHOLD;
    if (Objects.nonNull(passThreshold)) {
        thisPassThreshold = passThreshold;
    }

    // 获取通过率
    double passRate = getPassRate(userId);

    // 判读是否通过
    return passRate >= thisPassThreshold;
}
```

**建议方案：**

```
// 是否通过函数
public boolean isPassed(Long userId) {
    // 获取通过阈值
    double thisPassThreshold = getPassThreshold();

    // 获取通过率
    double passRate = getPassRate(userId);

    // 判读是否通过
    return passRate >= thisPassThreshold;
}

// 获取通过阈值函数
private double getPassThreshold() {
    if (Objects.nonNull(passThreshold)) {
        return passThreshold;
    }
    return PASS_THRESHOLD;
}
```

**6.2 主要收益**

- 把获取参数值从业务函数中独立，使业务逻辑更清晰；
- 封装的获取参数值为独立函数，可以在代码中重复使用。

**7.**

**通过接口参数化封装相同逻辑**

------

**7.1 案例一**

**现象描述：**

```
// 发送审核员结算数据函数
public void sendAuditorSettleData() {
    List<WorkerSettleData> settleDataList = auditTaskDAO.statAuditorSettleData();
    for (WorkerSettleData settleData : settleDataList) {
        WorkerPushData pushData = new WorkerPushData();
        pushData.setId(settleData.getWorkerId());
        pushData.setType(WorkerPushDataType.AUDITOR);
        pushData.setData(settleData);
        pushService.push(pushData);
    }
}

// 发送验收员结算数据函数
public void sendCheckerSettleData() {
    List<WorkerSettleData> settleDataList = auditTaskDAO.statCheckerSettleData();
    for (WorkerSettleData settleData : settleDataList) {
        WorkerPushData pushData = new WorkerPushData();
        pushData.setId(settleData.getWorkerId());
        pushData.setType(WorkerPushDataType.CHECKER);
        pushData.setData(settleData);
        pushService.push(pushData);
    }
```

**建议方案：**

```
// 发送审核员结算数据函数
public void sendAuditorSettleData() {
    sendWorkerSettleData(WorkerPushDataType.AUDITOR, () -> auditTaskDAO.statAuditorSettleData());
}

// 发送验收员结算数据函数
public void sendCheckerSettleData() {
    sendWorkerSettleData(WorkerPushDataType.CHECKER, () -> auditTaskDAO.statCheckerSettleData());
}

// 发送作业员结算数据函数
public void sendWorkerSettleData(WorkerPushDataType dataType, WorkerSettleDataProvider dataProvider) {
    List<WorkerSettleData> settleDataList = dataProvider.statWorkerSettleData();
    for (WorkerSettleData settleData : settleDataList) {
        WorkerPushData pushData = new WorkerPushData();
        pushData.setId(settleData.getWorkerId());
        pushData.setType(dataType);
        pushData.setData(settleData);
        pushService.push(pushData);
    }
}

// 作业员结算数据提供者接口
private interface WorkerSettleDataProvider {
    // 统计作业员结算数据
    public List<WorkerSettleData> statWorkerSettleData();
}
```

**7.2 主要收益**

- 把核心逻辑从各个业务函数中抽析，使业务代码更清晰更易维护；
- 避免重复性代码多次编写，精简重复函数越多收益越大。

**8.**

**减少函数代码层级**

------

如果要使函数优美，建议函数代码层级在1-4之间，过多的缩进会让函数难以阅读。

**8.1 案例一：利用return提前返回函数**

**现象描述：**

```
// 获取用户余额函数
public Double getUserBalance(Long userId) {
    User user = getUser(userId);
    if (Objects.nonNull(user)) {
        UserAccount account = user.getAccount();
        if (Objects.nonNull(account)) {
            return account.getBalance();
        }
    }
    return null;
}
```

**建议方案：**

```
// 获取用户余额函数
public Double getUserBalance(Long userId) {
    // 获取用户信息
    User user = getUser(userId);
    if (Objects.isNull(user)) {
        return null;
    }

    // 获取用户账户
    UserAccount account = user.getAccount();
    if (Objects.isNull(account)) {
        return null;
    }

    // 返回账户余额
    return account.getBalance();
}
```

**8.2 案例二：利用continue提前结束循环**

**现象描述：**

```
// 获取合计余额函数
public double getTotalBalance(List<User> userList) {
    // 初始合计余额
    double totalBalance = 0.0D;

    // 依次累加余额
    for (User user : userList) {
        // 获取用户账户
        UserAccount account = user.getAccount();
        if (Objects.nonNull(account)) {
            // 累加用户余额
            Double balance = account.getBalance();
            if (Objects.nonNull(balance)) {
                totalBalance += balance;
            }
        }
    }

    // 返回合计余额
    return totalBalance;
}
```

**建议方案：**

```
// 获取合计余额函数
public double getTotalBalance(List<User> userList) {
    // 初始合计余额
    double totalBalance = 0.0D;

    // 依次累加余额
    for (User user : userList) {
        // 获取用户账户
        UserAccount account = user.getAccount();
        if (Objects.isNull(account)) {
            continue;
        }

        // 累加用户余额
        Double balance = account.getBalance();
        if (Objects.nonNull(balance)) {
            totalBalance += balance;
        }
    }

    // 返回合计余额
    return totalBalance;
}
```

**特殊说明**

其它方式：在循环体中，先调用案例1的函数getUserBalance(获取用户余额)，再进行对余额进行累加。

在循环体中，建议最多使用一次continue。如果需要有使用多次continue的需求，建议把循环体封装为一个函数。

**8.3 案例三：利用条件表达式函数减少层级**

请参考下一章的"案例2: 把复杂条件表达式封装为函数"

**8.4 主要收益**

- 代码层级减少，代码缩进减少；
- 模块划分清晰，方便阅读维护。



**9.**

**封装条件表达式函数**

------

**9.1 案例一：把简单条件表达式封装为函数**

**现象描述：**

```
// 获取门票价格函数
public double getTicketPrice(Date currDate) {
    if (Objects.nonNull(currDate) && currDate.after(DISCOUNT_BEGIN_DATE)
        && currDate.before(DISCOUNT_END_DATE)) {
        return TICKET_PRICE * DISCOUNT_RATE;
    }
    return TICKET_PRICE;
}
```

**建议方案：**

```
// 获取门票价格函数
public double getTicketPrice(Date currDate) {
    if (isDiscountDate(currDate)) {
        return TICKET_PRICE * DISCOUNT_RATE;
    }
    return TICKET_PRICE;
}

// 是否折扣日期函数
private static boolean isDiscountDate(Date currDate) {
    return Objects.nonNull(currDate) && 
currDate.after(DISCOUNT_BEGIN_DATE)
        && currDate.before(DISCOUNT_END_DATE);
}
```

**9.2 案例二：把复杂条件表达式封装为函数**

**现象描述：**

```
// 获取土豪用户列表
public List<User> getRichUserList(List<User> userList) {
    // 初始土豪用户列表
    List<User> richUserList = new ArrayList<>();

    // 依次查找土豪用户
    for (User user : userList) {
        // 获取用户账户
        UserAccount account = user.getAccount();
        if (Objects.nonNull(account)) {
            // 判断用户余额
            Double balance = account.getBalance();
            if (Objects.nonNull(balance) && balance.compareTo(RICH_THRESHOLD) >= 0) {
                // 添加土豪用户
                richUserList.add(user);
            }
        }
    }

    // 返回土豪用户列表
    return richUserList;
}
```

**建议方案：**

```
// 获取土豪用户列表
public List<User> getRichUserList(List<User> userList) {
    // 初始土豪用户列表
    List<User> richUserList = new ArrayList<>();

    // 依次查找土豪用户
    for (User user : userList) {
        // 判断土豪用户
        if (isRichUser(user)) {
            // 添加土豪用户
            richUserList.add(user);
        }
    }

    // 返回土豪用户列表
    return richUserList;
}

// 是否土豪用户
private boolean isRichUser(User user) {
    // 获取用户账户
    UserAccount account = user.getAccount();
    if (Objects.isNull(account)) {
        return false;
    }

    // 获取用户余额
    Double balance = account.getBalance();
    if (Objects.isNull(balance)) {
        return false;
    }

    // 比较用户余额
    return balance.compareTo(RICH_THRESHOLD) >= 0;
}
```

以上代码也可以用采用流式(Stream)编程的过滤来实现。

**9.3 主要收益**

- 把条件表达式从业务函数中独立，使业务逻辑更清晰；
- 封装的条件表达式为独立函数，可以在代码中重复使用。



**10.**

**尽量避免不必要的空指针判断**

------

本章只适用于项目内部代码，并且是自己了解的代码，才能够尽量避免不必要的空指针判断。对于第三方中间件和系统接口，必须做好空指针判断，以保证代码的健壮性。

**10.1 案例一：调用函数保证参数不为空，被调用函数尽量避免不必要的空指针判断**

**现象描述：**

```
// 创建用户信息
User user = new User();
... // 赋值用户相关信息
createUser(user);

// 创建用户函数
private void createUser(User user){
    // 判断用户为空
    if(Objects.isNull(user)) {
        return;
    }

    // 创建用户信息
    userDAO.insert(user);
    userRedis.save(user);
}
```

**建议方案：**

```
// 创建用户信息
User user = new User();
... // 赋值用户相关信息
createUser(user);

// 创建用户函数
private void createUser(User user){
    // 创建用户信息
    userDAO.insert(user);
    userRedis.save(user);
}
```

**10.2 案例二：被调用函数保证返回不为空,调用函数尽量避免不必要的空指针判断**

**现象描述：**

```
// 保存用户函数
public void saveUser(Long id, String name) {
    // 构建用户信息
    User user = buildUser(id, name);
    if (Objects.isNull(user)) {
        throw new BizRuntimeException("构建用户信息为空");
    }

    // 保存用户信息
    userDAO.insert(user);
    userRedis.save(user);
}

// 构建用户函数
private User buildUser(Long id, String name) {
    User user = new User();
    user.setId(id);
    user.setName(name);
    return user;
}
```

**建议方案：**

```
// 保存用户函数
public void saveUser(Long id, String name) {
    // 构建用户信息
    User user = buildUser(id, name);

    // 保存用户信息
    userDAO.insert(user);
    userRedis.save(user);
}

// 构建用户函数
private User buildUser(Long id, String name) {
    User user = new User();
    user.setId(id);
    user.setName(name);
    return user;
}
```

**10.3 案例三：赋值逻辑保证列表数据项不为空，处理逻辑尽量避免不必要的空指针判断**

**现象描述：**

```
// 查询用户列表
List<UserDO> userList = userDAO.queryAll();
if (CollectionUtils.isEmpty(userList)) {
    return;
}

// 转化用户列表
List<UserVO> userVoList = new ArrayList<>(userList.size());
for (UserDO user : userList) {
    UserVO userVo = new UserVO();
    userVo.setId(user.getId());
    userVo.setName(user.getName());
    userVoList.add(userVo);
}

// 依次处理用户
for (UserVO userVo : userVoList) {
    // 判断用户为空
    if (Objects.isNull(userVo)) {
        continue;
    }

    // 处理相关逻辑
    ...
}
```

**建议方案：**

```
// 查询用户列表
List<UserDO> userList = userDAO.queryAll();
if (CollectionUtils.isEmpty(userList)) {
    return;
}

// 转化用户列表
List<UserVO> userVoList = new ArrayList<>(userList.size());
for (UserDO user : userList) {
    UserVO userVo = new UserVO();
    userVo.setId(user.getId());
    userVo.setName(user.getName());
    userVoList.add(userVo);
}

// 依次处理用户
for (UserVO userVo : userVoList) {
    // 处理相关逻辑
    ...
}
```

**10.4 案例四：MyBatis查询函数返回列表和数据项不为空，可以不用空指针判断**

MyBatis是一款优秀的持久层框架，是在项目中使用的最广泛的数据库中间件之一。通过对MyBatis源码进行分析，查询函数返回的列表和数据项都不为空，在代码中可以不用进行空指针判断。

**现象描述：**

这种写法没有问题，只是过于保守了。

```
// 查询用户函数
public List<UserVO> queryUser(Long id, String name) {
    // 查询用户列表
    List<UserDO> userList = userDAO.query(id, name);
    if (Objects.isNull(userList)) {
        return Collections.emptyList();
    }

    // 转化用户列表
    List<UserVO> voList = new ArrayList<>(userList.size());
    for (UserDO user : userList) {
        // 判断对象为空
        if (Objects.isNull(user)) {
            continue;
        }

        // 添加用户信息
        UserVO vo = new UserVO();
        BeanUtils.copyProperties(user, vo);
        voList.add(vo);
    }

    // 返回用户列表
    return voList;
}
```

**建议方案：**

```
// 查询用户函数
public List<UserVO> queryUser(Long id, String name) {
    // 查询用户列表
    List<UserDO> userList = userDAO.query(id, name);

    // 转化用户列表
    List<UserVO> voList = new ArrayList<>(userList.size());
    for (UserDO user : userList) {
        UserVO vo = new UserVO();
        BeanUtils.copyProperties(user, vo);
        voList.add(vo);
    }

    // 返回用户列表
    return voList;
}
```

**10.5 主要收益**

- 避免不必要的空指针判断，精简业务代码处理逻辑，提高业务代码运行效率；
- 这些不必要的空指针判断，基本属于永远不执行的Death代码，删除有助于代码维护。

**11.**

**内部函数参数尽量使用基础类型**

------

**11.1 案例一：内部函数参数尽量使用基础类型**

**现象描述：**

```
// 调用代码
double price = 5.1D;
int number = 9;
double total = calculate(price, number);

// 计算金额函数
private double calculate(Double price, Integer number) {
    return price * number;
}
```

**建议方案：**

```
// 调用代码
double price = 5.1D;
int number = 9;
double total = calculate(price, number);

// 计算金额函数
private double calculate(double price, int number) {
    return price * number;
}
```

**11.2 案例二：内部函数返回值尽量使用基础类型**

**现象描述：**

```
// 获取订单总额函数
public double getOrderAmount(List<Product> productList) {
    double amount = 0.0D;
    for (Product product : productList) {
        if (Objects.isNull(product) || Objects.isNull(product.getPrice())
            || Objects.isNull(product.getNumber())) {
            continue;
        }
        amount += calculate(product.getPrice(), product.getNumber());
    }
    return amount;
}

// 计算金额函数
private Double calculate(double price, double number) {
    return price * number;
}
```

**建议方案：**

```
// 获取订单总额函数
public double getOrderAmount(List<Product> productList) {
    double amount = 0.0D;
    for (Product product : productList) {
        if (Objects.isNull(product) || Objects.isNull(product.getPrice())
            || Objects.isNull(product.getNumber())) {
            continue;
        }
        amount += calculate(product.getPrice(), product.getNumber());
    }
    return amount;
}

// 计算金额函数
private double calculate(double price, double number) {
    return price * number;
}
```



此处只是举例说明这种现象，更好的方式是采用流式(Stream)编程。

**11.3 主要收益**

- 内部函数尽量使用基础类型，避免了隐式封装类型的打包和拆包；
- 内部函数参数使用基础类型，用语法上避免了内部函数的参数空指针判断；
- 内部函数返回值使用基础类型，用语法上避免了调用函数的返回值空指针判断。

**12.**

**尽量避免返回的数组和列表为null**

------

**12.1 案例一：尽量避免返回的数组为null，引起不必要的空指针判断**

**现象描述：**

```
// 调用代码
UserVO[] users = queryUser();
if (Objects.nonNull(users)) {
    for (UserVO user : users) {
        // 处理用户信息
    }
}

// 查询用户函数
private UserVO[] queryUser() {
    // 查询用户列表
    List<UserDO> userList = userDAO.queryAll();
    if (CollectionUtils.isEmpty(userList)) {
        return null;
    }

    // 转化用户数组
    UserVO[] users = new UserVO[userList.size()];
    for (int i = 0; i < userList.size(); i++) {
        UserDO user = userList.get(i);
        users[i] = new UserVO();
        users[i].setId(user.getId());
        users[i].setName(user.getName());
    }

    // 返回用户数组
    return users;
}
```

**建议方案：**

```
// 调用代码
UserVO[] users = queryUser();
for (UserVO user : users) {
    // 处理用户信息
}

// 查询用户函数
private UserVO[] queryUser() {
    // 查询用户列表
    List<UserDO> userList = userDAO.queryAll();
    if (CollectionUtils.isEmpty(userList)) {
        return new UserVO[0];
    }

    // 转化用户数组
    UserVO[] users = new UserVO[userList.size()];
    for (int i = 0; i < userList.size(); i++) {
        UserDO user = userList.get(i);
        users[i] = new UserVO();
        users[i].setId(user.getId());
        users[i].setName(user.getName());
    }

    // 返回用户数组
    return users;
}
```

**12.2 案例二：尽量避免返回的列表为null，引起不必要的空指针判断**

**现象描述：**

```
// 调用代码
List<UserVO> userList = queryUser();
if (Objects.nonNull(userList)) {
    for (UserVO user : userList) {
        // 处理用户信息
    }
}

// 查询用户函数
private List<UserVO> queryUser(){
    // 查询用户列表
    List<UserDO> userList = userDAO.queryAll();
    if(CollectionUtils.isEmpty(userList)) {
        return null;
    }

    // 转化用户列表
    List<UserVO> userVoList = new ArrayList<>(userList.size());
    for(UserDO user : userList) {
        UserVO userVo = new UserVO();
        userVo.setId(user.getId());
        userVo.setName(user.getName());
        userVoList.add(userVo);
    }

    // 返回用户列表
    return userVoList;
}
```

**建议方案：**

```
// 调用代码
List<UserVO> userList = queryUser();
for (UserVO user : userList) {
   // 处理用户信息
 }

// 查询用户函数
private List<UserVO> queryUser(){
    // 查询用户列表
    List<UserDO> userList = userDAO.queryAll();
    if(CollectionUtils.isEmpty(userList)) {
        return Collections.emptyList();
    }

    // 转化用户列表
    List<UserVO> userVoList = new ArrayList<>(userList.size());
    for(UserDO user : userList) {
        UserVO userVo = new UserVO();
        userVo.setId(user.getId());
        userVo.setName(user.getName());
        userVoList.add(userVo);
    }

    // 返回用户列表
    return userVoList;
}
```

**12.3 主要收益**

- 保证返回的数组和列表不为null, 避免调用函数的空指针判断。

**13.**

**封装函数传入参数**

------

**13.1 案例一：当传入参数过多时，应封装为参数类**

Java规范不允许函数参数太多，不便于维护也不便于扩展。

**现象描述：**

```
// 修改用户函数
public void modifyUser(Long id, String name, String phone, Integer age, 
    Integer sex, String address, String description) {
    // 具体实现逻辑
}
```

**建议方案：**

```
// 修改用户函数
public void modifyUser(User user) {
    // 具体实现内容
}

// 用户类
@Getter
@Setter
@ToString
private class User{
    private Long id;
    private String name;
    private String phone;
    private Integer age;
    private Integer sex;
    private String address;
    private String description;
}
```

**13.2 案例二：当传入成组参数时，应封装为参数类**

既然参数成组出现，就需要封装一个类去描述这种现象。

**现象描述：**

```
// 获取距离函数
public double getDistance(double x1, double y1, double x2, double y2) {
    // 具体实现逻辑
}
```

**建议方案：**

```
// 获取距离函数
public double getDistance(Point point1, Point point2) {
    // 具体实现逻辑
}

// 点类
@Getter
@Setter
@ToString
private class Point{
    private double x;
    private double y;
}
```

**13.3 主要收益**

- 封装过多函数参数为类，使函数更便于扩展和维护；
- 封装成组函数参数为类，使业务概念更明确更清晰。

**14.**

**尽量用函数替换匿名内部类的实现**

------

Java匿名内部类的优缺点:
在匿名内部类（包括Lambda表达式）中可以直接访问外部类的成员，包括类的成员变量、函数的内部变量。正因为可以随意访问外部变量，所以会导致代码边界不清晰。

首先推荐用Lambda表达式简化匿名内部类，其次推荐用函数替换复杂的Lambda表达式的实现。

**14.1 案例一：尽量用函数替换匿名内部类（包括Lambda表达式）的实现**

**现象描述：**

```
// 发送结算数据
sendWorkerSettleData(WorkerPushDataType.CHECKER, () -> {
    Date beginDate = DateUtils.addDays(currDate, -aheadDays);
    Date endDate = DateUtils.addDays(currDate, 1);
    return auditTaskDAO.statCheckerSettleData(beginDate, endDate);
});
```

**建议方案：**

```
// 发送结算数据
sendWorkerSettleData(WorkerPushDataType.CHECKER, () -> statCheckerSettleData(currDate, aheadDays));

// 统计验收员结算数据函数
private List<WorkerSettleData> statCheckerSettleData(Date currDate, int aheadDays) {
    Date beginDate = DateUtils.addDays(currDate, -aheadDays);
    Date endDate = DateUtils.addDays(currDate, 1);
    return auditTaskDAO.statCheckerSettleData(beginDate, endDate);
}
```

其实，还有一个更简单的办法。在调用函数sendWorkerSettleData（发送作业员结算数据）之前计算开始日期、结束日期，就直接可以用函数auditTaskDAO.statCheckerSettleData(beginDate, endDate)代替匿名内部类实现。

**14.2 案例二：拆分复杂匿名内部类实现接口为多个函数类接口**

如果一个匿名内部类实现的接口几个函数间关联性不大，可以把这个接口拆分为几个函数式接口，便于使用Lambda表达式。

**现象描述：**

```
// 清除过期数据
cleanExpiredData("用户日志表", new CleanExpiredDataOperator() {
    @Override
    public List<Date> queryExpiredDate(Integer remainDays) {
        return userDAO.queryExpiredDate(remainDays);
    }
    @Override
    public void cleanExpiredData(Date expiredDate) {
        userDAO.cleanExpiredData(expiredDate);
    }
});

// 清除过期数据函数
private void cleanExpiredData(String tableName, CleanExpiredDataOperator ,
cleanExpiredDataOperator) {
    // 功能实现代码
}

// 清除过期操作接口
interface CleanExpiredDataOperator {
    // 查询过期日期
    public List<Date> queryExpiredDate(Integer remainDays);
    // 清除过期数据
    public void cleanExpiredData(Date expiredDate);
}
```

**建议方案：**

```
// 清除过期数据
cleanExpiredData("用户日志表", userDAO::queryExpiredDate,userDAO::cleanExpiredData);

// 清除过期数据函数
private void cleanExpiredData(String tableName, QueryExpiredDateOperator queryExpiredDateOperator, CleanExpiredDataOperator cleanExpiredDataOperator) {
    // 功能实现代码
}

// 查询过期日期接口
interface QueryExpiredDateOperator {
    // 查询过期日期
    public List<Date> queryExpiredDate(Integer remainDays);
}

// 清除过期操作接口
interface CleanExpiredDataOperator {
    // 清除过期数据
    public void cleanExpiredData(Date expiredDate);
}
```

**14.3 主要收益**

- 定义函数并指定参数，明确规定了匿名内部类的代码边界；
- 利用Lambda表达式简化匿名内部类实现，使代码更简洁。

**15.**

**利用return精简不必要的代码**

------

**15.1 案例一：删除不必要的if**

**现象描述：**

```
// 是否通过函数
public boolean isPassed(Double passRate) {
    if (Objects.nonNull(passRate) && passRate.compareTo(PASS_THRESHOLD) >= 0) {
        return true;
    }
    return false;
}
```

**建议方案：**

```
// 是否通过函数
public boolean isPassed(Double passRate) {
    return Objects.nonNull(passRate) && passRate.compareTo(PASS_THRESHOLD) >= 0;
}
```

**15.2 案例二：删除不必要的else**

**现象描述：**

```
// 结算工资函数
public double settleSalary(Long workId, int workDays) {
    // 根据是否合格处理
    if (isQualified(workId)) {
        return settleQualifiedSalary(workDays);
    } else {
        return settleUnqualifiedSalary(workDays);
    }
}
```

**建议方案：**

```
// 结算工资函数
public double settleSalary(Long workId, int workDays) {
    // 根据是否合格处理
    if (isQualified(workId)) {
        return settleQualifiedSalary(workDays);
    }
    return settleUnqualifiedSalary(workDays);
}
```

**15.3 案例三：删除不必要的变量**

**现象描述：**

```
// 查询用户函数
public List<UserDO> queryUser(Long id, String name) {
    UserQuery userQuery = new UserQuery();
    userQuery.setId(id);
    userQuery.setName(name);
    List<UserDO> userList = userDAO.query(userQuery);
    return userList;
}
```

**建议方案：**

```
// 查询用户函数
public List<UserDO> queryUser(Long id, String name) {
    UserQuery userQuery = new UserQuery();
    userQuery.setId(id);
    userQuery.setName(name);
    return userDAO.query(userQuery);
}
```

**15.4 主要收益**

- 精简不必要的代码，让代码看起来更清爽

**16.**

**利用临时变量优化代码**

------

在一些代码中，经常会看到a.getB().getC()...getN()的写法，姑且叫做“函数的级联调用”，代码健壮性和可读性太差。建议：杜绝函数的级联调用，利用临时变量进行拆分，并做好对象空指针检查。

**16.1 案例一：利用临时变量厘清逻辑**

**现象描述：**

```
// 是否土豪用户函数
private boolean isRichUser(User user) {
    return Objects.nonNull(user.getAccount())
        && Objects.nonNull(user.getAccount().getBalance())
        && user.getAccount().getBalance().compareTo(RICH_THRESHOLD) >= 0;
}
```

这是精简代码控的最爱，但是可读性实在太差。

**建议方案：**

```
// 是否土豪用户函数
private boolean isRichUser(User user) {
    // 获取用户账户
    UserAccount account = user.getAccount();
    if (Objects.isNull(account)) {
        return false;
    }

    // 获取用户余额
    Double balance = account.getBalance();
    if (Objects.isNull(balance)) {
        return false;
    }

    // 比较用户余额
    return balance.compareTo(RICH_THRESHOLD) >= 0;
}
```

这个方案，增加了代码行数，但是逻辑更清晰。
有时候，当代码的精简性和可读性发生冲突时，个人更偏向于保留代码的可读性。

**16.2 案例二：利用临时变量精简代码**

**现象描述：**

```
// 构建用户函数
public UserVO buildUser(UserDO user) {
    UserVO vo = new UserVO();
    vo.setId(user.getId());
    vo.setName(user.getName());
    if (Objects.nonNull(user.getAccount())) {
        vo.setBalance(user.getAccount().getBalance());
        vo.setDebt(user.getAccount().getDebt());
    }
    return vo;
}
```

这么写，大约是为了节约一个临时变量把。

**建议方案：**

```
// 构建用户函数
public UserVO buildUser1(UserDO user) {
    UserVO vo = new UserVO();
    vo.setId(user.getId());
    vo.setName(user.getName());
    UserAccount account = user.getAccount();
    if (Objects.nonNull(account)) {
        vo.setBalance(account.getBalance());
        vo.setDebt(account.getDebt());
    }
    return vo;
}
```

**16.3 主要收益**

- 利用临时变量厘清逻辑，显得业务逻辑更清晰；
- 利用临时变量精简代码，看变量名称即知其义，减少了大量无用代码；
- 如果获取函数比较复杂耗时，利用临时变量可以提高运行效率；
- 利用临时变量避免函数的级联调用，可有效预防空指针异常。

**17.**

**仅保留函数需要的参数**

------

在一些代码中，经常会看到a.getB().getC()...getN()的写法，姑且叫做“函数的级联调用”，代码健壮性和可读性太差。建议：杜绝函数的级联调用，利用临时变量进行拆分，并做好对象空指针检查。

**17.1 案例一：删除多余的参数**

**现象描述：**

```
// 修改用户状态函数
private void modifyUserStatus(Long userId, Integer status, String unused) {
    userCache.modifyStatus(userId, status);
    userDAO.modifyStatus(userId, status);
}
```

其中，unused参数是无用参数。

**建议方案：**

```
// 修改用户状态函数
private void modifyUserStatus(Long userId, Integer status) {
    userCache.modifyStatus(userId, status);
    userDAO.modifyStatus(userId, status);
}
```

**17.2 案例二：用属性取代对象**

**现象描述：**

```
// 删除用户函数
private void deleteUser(User user) {
    userCache.delete(user.getId());
    userDAO.delete(user.getId());
}
```

**建议方案：**

```
// 删除用户函数
private void deleteUser(Long userId) {
    userCache.delete(userId);
    userDAO.delete(userId);
}
```

**建议方案：**

调用函数时，参数对象不需要专门构建，而函数使用其属性超过3个，可以不必使用该规则。

**17.3 主要收益**

- 仅保留函数需要的参数，明确了调用时需要赋值的参数，避免了调用时还要去构造些无用参数。

**18.**

**后记**

------

"众人拾柴火焰高"。如果有更多更好的观点，亦或有更好的代码案例，欢迎大家进行补充说明。

笔者希望以此文抛砖引玉，如果最终形成一套完善的Java编码规范，善莫大焉





### 文章来源

[Java函数优雅之道](https://mp.weixin.qq.com/s/CKNAereooHGLWFzmHbdWdw)