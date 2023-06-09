### 已经给出了代码了，不重写，但是要补充类似的功能，这样才不会纸上谈兵

专门做一个面试章节，把难记的知识点都转换成业务：（想起一个就加一个）

- 线程池的原理与使用：官方给的 & 自定义的都写一个

[https://wx.zsxq.com/mweb/views/topicdetail/topicdetail.html?topic_id=584111424885424&inviter_id=48524228242588&share_from=ShareToWechat&keyword=0aAblb2oc](https://wx.zsxq.com/mweb/views/topicdetail/topicdetail.html?topic_id=584111424885424&inviter_id=48524228242588&share_from=ShareToWechat&keyword=0aAblb2oc)

分布式并发：[https://wx.zsxq.com/mweb/views/topicdetail/topicdetail.html?topic_id=181548424212822&inviter_id=48524228242588&share_from=ShareToWechat&keyword=0bGynl1Ap](https://wx.zsxq.com/mweb/views/topicdetail/topicdetail.html?topic_id=181548424212822&inviter_id=48524228242588&share_from=ShareToWechat&keyword=0bGynl1Ap)

[https://tech.meituan.com/2017/12/22/ddd-in-practice.html](https://tech.meituan.com/2017/12/22/ddd-in-practice.html) ⭐️⭐️⭐️⭐️⭐️

Lottery学习经验：

![|500](https://cdn.nlark.com/yuque/0/2023/png/300264/1685859726534-6d6afcbc-290d-4cc2-a650-82c710f23d95.png)

Lottery相关简历 ⭐️⭐️⭐️

![|575](https://cdn.nlark.com/yuque/0/2023/png/300264/1676811694977-c795faa1-4b83-4aaa-8594-87c41ade8fd8.png)![|425](https://cdn.nlark.com/yuque/0/2023/png/300264/1675863569592-719ce5bb-064a-4009-89f6-ca95efef70d2.png)

# 零、前置知识

## DDD介绍

## 1. 概念讲解

DDD，domain drive design

- 普通的MVC，有DAO层，如Student，里面只有属性的getter,setter，这叫做「**贫血模型**」（_如果问DDD和MVC的区别，这个要答出来是核心_）
- 充血模型：把实体的操作都封装到里面

比如有一个Student的领域，一个Course的领域，领域内模型是充血的，其各自是独立的，那么如果想要交互呢，那么就在上面抽象出一个「领域服务」进行两个领域的交互，这两个领域可以独立发展

- _**领域可以独立发展，只要领域服务足够稳定**_
- 域是相对独立的业务层
- 仓库Resposity用于领域的存储，只是存储用

![](https://cdn.nlark.com/yuque/0/2022/jpeg/300264/1671524306031-1ebcb4a5-53ab-4385-b567-fd7fba544cdd.jpeg)

![](https://cdn.nlark.com/yuque/0/2022/png/300264/1671524604780-34582428-39d2-4051-9b9c-366080c8d2c8.png)

- 防腐层：比如数据存储，并不只是MySQL，在数据存储上又抽象出防腐层：到时候可以更改数据库为其他的，而底层的数据存储不用变

## RPC

- 提供者提供RPC接口，然后打包
- 消费者导入这个RPC包，去调用这个服务
- 本质上相当于提供者提供了一个快照，消费者去调用这个快照，刚好消费者这些快照内容都在，就成功调用了

- 如果消费者更改了内容，和之前打包的内容不同，那就出现问题了

![](https://cdn.nlark.com/yuque/0/2022/png/300264/1671357838259-0221154e-360f-41eb-b669-7910904484bd.png)

RPC底层通信是Netty，Socket通信

## 设计模式

美团的这篇设计模式涉及到本文多个：[https://mp.weixin.qq.com/s/H2toewJKEwq1mXme_iMWkA](https://mp.weixin.qq.com/s/H2toewJKEwq1mXme_iMWkA)

# 一、全局处理

## 请求响应码

以一个枚举类的形式，封装在 Constant类

public class Constants {

    public enum ResponseCode {
        SUCCESS("0000", "成功"),
        UN_ERROR("0001","未知失败"),
        ILLEGAL_PARAMETER("0002","非法参数"),
        INDEX_DUP("0003","主键冲突");

        private String code;
        private String info;

        ResponseCode(String code, String info) {
            this.code = code;
            this.info = info;
        }

        public String getCode() {
            return code;
        }

        public String getInfo() {
            return info;
        }
    }
}

还要很多

## 请求响应结果

public class Result implements Serializable {

    private static final long serialVersionUID = -3826891916021780628L;
    private String code;
    private String info;

    public static Result buildResult(String code, String info) {
        return new Result(code, info);
    }

    public static Result buildSuccessResult() {
        return new Result(Constants.ResponseCode.SUCCESS.getCode(), Constants.ResponseCode.SUCCESS.getInfo());
    }

    public static Result buildErrorResult() {
        return new Result(Constants.ResponseCode.UN_ERROR.getCode(), Constants.ResponseCode.UN_ERROR.getInfo());
    }

    // 省略 getter  setter

}

### 使用

return new Result(Constants.ResponseCode.UN_ERROR.getCode(), Constants.ResponseCode.UN_ERROR.getInfo());

---

# 总体架构

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672746838141-98227e63-477f-4601-b9e0-bb220839a957.png)

## 防腐层如何体现？

即`lottery-interface` 防腐体现在：对DTO进行处理，然后执行抽奖过程，然后返回结果，就是对`lottery-application`的一层浅封装

返回的结果是 VO，在Vue显示的结果

## 单独模块结构

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672747123652-53e32187-e859-4829-9e38-473f150d39a9.png)

---

# 数据库设计

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672746883959-6382b235-26df-48f4-bf79-4559c49d0eb4.png)

`**user_take_activity**`**是用户参与了活动就插入一条记录**

**但是参与了不一定中奖，并且如果中奖了也是之后才发送**

**所以**`**user_strategt_export_xxx**`**记录用户的中奖记录**

---

# 三、Dubbo的RPC调用：广播模式

## Server端
> [!note]
> 


- `**@Service**`**是Dubbo的注解，加上了就是要暴露出的RPC接口**
- `import org.apache.dubbo.config.annotation.Service;`

```java
@Service
public class ActivityBooth implements IActivityBooth {

    @Resource
    private IActivityDao activityDao;

    @Override
    public ActivityRes queryActivityById(ActivityReq req) {

        Activity activity = activityDao.queryActivityById(req.getActivityId());

        ActivityDto activityDto = new ActivityDto();
        activityDto.setActivityId(activity.getActivityId());

        return new ActivityRes(new Result(Constants.ResponseCode.SUCCESS.getCode(), Constants.ResponseCode.SUCCESS.getInfo()), activityDto);
    }
}
```

# Dubbo 广播方式配置
```yaml
dubbo:
  application:
    name: Lottery
    version: 1.0.0
  registry:
    address: multicast://224.5.6.7:1234		// 广播模式
  protocol:
    name: dubbo
    port: 20880
  scan:
    base-packages: cn.itedus.lottery.rpc
```

## Client调用端

1. 先导入Server端的依赖（需要在Server端先install打包）

```xml
<dependency>
    <groupId>cn.itedus.lottery</groupId>
    <artifactId>lottery-rpc</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

2. 使用

```java
// 导入的都是Server端暴露出的
import cn.itedus.lottery.rpc.IActivityBooth;
import cn.itedus.lottery.rpc.req.ActivityReq;
import cn.itedus.lottery.rpc.res.ActivityRes;

@Reference(interfaceClass = IActivityBooth.class, url = "dubbo://127.0.0.1:20880")
private IActivityBooth activityBooth;

@Test
public void test_rpc() {
    ActivityReq req = new ActivityReq();
    req.setActivityId(927L);
    ActivityRes result = activityBooth.queryActivityById(req);
    logger.info("测试结果：{}", JSON.toJSONString(result));
}

```
3. 配置

使用的是Dubbo的广播模式，N/A是直连模式

# Dubbo 广播方式配置
```yaml
dubbo:
  application:
    name: Lottery
    version: 1.0.0
  registry:
    address: multicast://224.5.6.7:1234
  protocol:
    name: dubbo
    port: 20880
  provider:
    timeout: 50000
```

---

# 四、抽奖活动策略表设计

---

# 五、抽奖策略领域模块开发

**项目中所有的map都用**`**ConcurrentHashMap**`**存储保证线程安全**

分支：210814_xfg_strategy

![|500](https://cdn.nlark.com/yuque/0/2022/png/300264/1671865151318-dd67ccf8-395a-4f9b-95bf-693ffc265833.png)

- model是用到的一些dto
- repository是数据库的一些数据查询
- service下的algorithm，是具体的抽奖算法，不依赖于任何数据，只是进行抽奖
- 传入抽奖策略ID，返回奖品ID
- service下的draw，是对上面的抽奖算法的包装：传入请求参数，返回抽奖结果

## ① 抽奖算法实现：总体概率

- 必中

如果A奖品抽空后，B和C奖品的概率按照 3:5 均分，相当于B奖品中奖概率由 0.3 升为 0.375

```java
// 新的概率 = 原概率 ÷ 新的概率和（结果向上取整）
int rateVal = awardRateInfo.getAwardRate().divide(differenceDenominator, 2, BigDecimal.ROUND_UP).multiply(new BigDecimal(100)).intValue();
```

即一般的话所有奖品总概率加起来是1，但如果不是1的话，那么也会按照已有的概率总和进行等比例划分

```java
@Override
public String randomDraw(Long strategyId, List<String> excludeAwardIds) {

    BigDecimal differenceDenominator = BigDecimal.ZERO;

    // 排除掉不在抽奖范围的奖品ID集合
    List<AwardRateInfo> differenceAwardRateList = new ArrayList<>();
    List<AwardRateInfo> awardRateIntervalValList = awardRateInfoMap.get(strategyId);
    for (AwardRateInfo awardRateInfo : awardRateIntervalValList) {
        String awardId = awardRateInfo.getAwardId();
        if (excludeAwardIds.contains(awardId)) {
            continue;
        }
        differenceAwardRateList.add(awardRateInfo);
        // 加入到概念总和，重新计算剔除后的概率值
        differenceDenominator = differenceDenominator.add(awardRateInfo.getAwardRate());
    }

    // 前置判断，如果筛选完后的列表只有0个或1个奖品，直接返回（就是100%）
    if (differenceAwardRateList.size() == 0) return "";
    if (differenceAwardRateList.size() == 1) return differenceAwardRateList.get(0).getAwardId();

    // 获取随机概率值
    SecureRandom secureRandom = new SecureRandom();
    int randomVal = secureRandom.nextInt(100) + 1;

    // 循环获取奖品
    String awardId = "";
    int cursorVal = 0;
    for (AwardRateInfo awardRateInfo : differenceAwardRateList) {
        // 新的概率 = 原概率 ÷ 新的概率和（结果向上取整）
        int rateVal = awardRateInfo.getAwardRate().divide(differenceDenominator, 2, BigDecimal.ROUND_UP).multiply(new BigDecimal(100)).intValue();
        if (randomVal <= (cursorVal + rateVal)) {
            awardId = awardRateInfo.getAwardId();
            break;
        }
        cursorVal += rateVal;
    }

    // 返回中奖结果
    return awardId;
}
```

## ① 抽奖算法实现：单项概率

- 如果总奖品概率和不为1的话，可能不中奖。有两种不中奖的情况

- 一种是排除了一部分奖项，如把4，5等奖排除，那么就可能不中奖，即原来随机到4，5等奖的改为未中奖
- 二是原来的奖品总概率之和就不为1，这个原理要去看`rateTuple[]`的初始化已经抽奖时的随机

### 1. 进行rateTuple[]的初始化

注意，初始化数组方法需要用`synchronize`修饰，防止多次初始化

- 将策略Id和概率数组绑定起来
- **设置一个浮标，用来填充不同的概率，如果概率总和为1，那么加起来就循环100次**
- 首先要从总的中奖列表中排除掉那些被排除掉的奖品，这些奖品会涉及到概率的值重新计算。
- 如果排除后剩下的奖品列表小于等于1，则可以直接返回对应信息
- 接下来就使用随机数工具生产一个100内的随值与奖品列表中的值进行循环比对，算法时间复杂度`**O(n)**`

- 使用`SecureRandom`生成随机数，seed是一个`AtomicLong`：`this.seed = new AtomicLong();`

- `SecureRandom`128位加密位数更多更安全
- seed是从OS上的随机种子，更难猜到
- **seed是线程安全的**

- **一个概率值就是一个循环的节点，如0.2，就是循环到20**
- **数组内填充的是**`**奖品ID**`

```java
/**
 * 斐波那契散列增量，逻辑：黄金分割点：(√5 - 1) / 2 = 0.6180339887，Math.pow(2, 32) * 0.6180339887 = 0x61c88647
 */
private final int HASH_INCREMENT = 0x61c88647;

/**
 * 数组初始化长度 128，保证数据填充时不发生碰撞的最小初始化值
 * 为什么是18，这就是HashMap为什么是2的幂次方，因为最后 hashcode & n-1  ，散列更均匀 ⭐️⭐️⭐️⭐️⭐️
 */
private final int RATE_TUPLE_LENGTH = 128;

/**
 * 存放概率与奖品对应的散列结果，strategyId -> rateTuple
	// 注意这里用 ConcurrentHashMap 去存放 ⭐️⭐️⭐️⭐️⭐️
 */
protected Map<Long, String[]> rateTupleMap = new ConcurrentHashMap<>();

/**
 * 奖品区间概率值，strategyId -> [awardId->begin、awardId->end]
 */
protected Map<Long, List<AwardRateVO>> awardRateInfoMap = new ConcurrentHashMap<>();


@Override
public void initRateTuple(Long strategyId, List<AwardRateInfo> awardRateInfoList) {
    // 保存奖品概率信息
    awardRateInfoMap.put(strategyId, awardRateInfoList);

    // RATE_TUPLE_LENGTH：128
    String[] rateTuple = rateTupleMap.computeIfAbsent(strategyId, k -> new String[RATE_TUPLE_LENGTH]);

    int cursorVal = 0;
    for (AwardRateInfo awardRateInfo : awardRateInfoList) {
        // 这里使用 BigDecimal
        int rateVal = awardRateInfo.getAwardRate().multiply(new BigDecimal(100)).intValue();

        // 循环填充概率范围值
        for (int i = cursorVal + 1; i <= (rateVal + cursorVal); i++) {
            rateTuple[hashIdx(i)] = awardRateInfo.getAwardId();
        }

        cursorVal += rateVal;

    }
}

/**
 * 斐波那契（Fibonacci）散列法，计算哈希索引下标值
 *
》
 * @param val 值
 * @return 索引
 */
protected int hashIdx(int val) {
    int hashCode = val * HASH_INCREMENT + HASH_INCREMENT;
    return hashCode & (RATE_TUPLE_LENGTH - 1);  // 例：11111 & 10  -> 11
}
```

### 2. 单项抽奖

- 获取策略对应的元组
- 随机出一个索引
- 用这个索引 `hashIdx(randomVal)`去获得`rateTuple`对应位置的奖项，时间复杂度`O(1)`

⭐️⭐️⭐️

随机出的索引一定是[1,100]中的一个；

如果所有的奖项概率和为1，那么rateTuple一定是由[1,100]逐个去hashIdx后填充的，索引一定会获得奖品

但是如果所有的奖项概率和不为1，比如为0.6。那么rateTuple则是由[1,60]逐个hashIdx后填充的，如果随机索引超过60，则很可能为null

```java
@Override
public String randomDraw(Long strategyId, List<String> excludeAwardIds) {

    // 获取策略对应的元祖
    String[] rateTuple = super.rateTupleMap.get(strategyId);
    assert rateTuple != null;

    // 随机索引
    int randomVal = new SecureRandom().nextInt(100) + 1;
    int idx = super.hashIdx(randomVal);

    // 返回结果
    String awardId = rateTuple[idx];
    if (excludeAwardIds.contains(awardId)) return "未中奖";

    //  如果 awardId是null的情况，因为rateTuple并不是一定会填满，randomId可能会随机到一个不存在的位置上
    // ⭐️️️️⭐️️️️⭐️️️️⭐️️️️⭐️️️️
    //  rateTuple[]如果所有的奖品概率加起来是1的话，那么就是从1-100都随机了一次，randomVal的idx一定会落到这个数组上非null的位置
    //  但是，如果所有奖品概率加起来不为1，那么只能随机到[1, xx]，那么randomVal是可以随机到[1,100]的，就可能落在null上了
    if(awardId==null) return "未中奖";
    return awardId;
}
```

## 斐波那契散列
```java
public void test_idx() {
    // 存储idx出现次数
    Map<Integer, Integer> map = new HashMap<>();
    Map<Integer, Integer> map2 = new HashMap<>();

    // 1. 黄金分割点：(√5 - 1) / 2 = 0.6180339887     
    // 2. 1.618:1 == 1:0.618
    // 3. Math.pow(2, 32) * 0.6180339887 = 0x61c88647
    int HASH_INCREMENT = 0x61c88647;
    int hashCode = 0;
    for (int i = 1; i <= 100; i++) {
        hashCode = i * HASH_INCREMENT + HASH_INCREMENT;
        int idx = hashCode & (128 - 1);

        // 都是花活
        map.merge(idx, 1, Integer::sum);

        System.out.println("斐波那契散列：" + idx + " 普通散列：" + (String.valueOf(i).hashCode() & (128 - 1)));
        map2.merge((String.valueOf(i).hashCode() & (128 - 1)), 1, Integer::sum);
    }

    // 打印idx出现次数
    // 1就是出现1次，2就是出现了两次
    System.out.println(map);
    // 普通散列碰撞几率更大
    System.out.println(map2);
}
```

斐波那契散列碰撞几率更小

![|1000](https://cdn.nlark.com/yuque/0/2023/png/300264/1672574026490-9833152c-ac1a-4b41-8279-c85d8b53d8d3.png)

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1676791523825-7d462661-1da7-45f4-b855-57032a96d1ed.png)弄清楚各层的意义⭐️⭐️⭐️

## 策略模式的体现

整体概率抽奖、单体抽奖都继承了`BaseAlgorithm`，他们都加了`@Component`注解，然后在一个配置类`DrawConfig`中，注入到一个Map

用户只需要传入抽奖策略名称，即可选择哪一项，这就是策略模式

```java
public class DrawConfig {

    @Resource
    private IDrawAlgorithm entiretyRateRandomDrawAlgorithm;

    @Resource
    private IDrawAlgorithm singleRateRandomDrawAlgorithm;

    /** 抽奖策略组 */
    protected static Map<Integer, IDrawAlgorithm> drawAlgorithmGroup = new ConcurrentHashMap<>();

    @PostConstruct
    public void init() {
        drawAlgorithmGroup.put(Constants.StrategyMode.ENTIRETY.getCode(), entiretyRateRandomDrawAlgorithm);
        drawAlgorithmGroup.put(Constants.StrategyMode.SINGLE.getCode(), singleRateRandomDrawAlgorithm);
    }

}
```

使用`@PostConstruct`在Bean注入后进行初始化

## 模板模式的体现

抽奖过程，就是一个模板模式

模板模式的核心：一个抽象abstract类，定义了一个过程，其中一些过程调用本类的抽象方法，子类只用实现这些抽象方法，主方法就是按照这个去调用了

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1678366981463-9d9b9ae6-da7c-4123-a4a9-9b1e8dfb74a5.png)

1. 获取抽奖策略对应的奖品
2. `initTruple[]`初始到内存（只是单体概率需要）
3. 获取不在抽奖列表的奖品
4. 执行抽奖
5. 包装结果

---

# 六、模板模式：处理抽奖流程

使用模板方法设计模式优化类 DrawExecImpl 抽奖过程方法实现，主要以**抽象类** AbstractDrawBase 编排定义流程，定义抽象方法由类 DrawExecImpl 做具体实现的方式进行处理

抽奖模式就是按照一个模板定义了，然后子类都按照这个流程去做。本章节最大的目标在于把抽奖流程标准化，需要考虑的一条思路线包括：

1. 根据入参策略ID获取抽奖策略配置
2. 校验和处理抽奖策略的数据初始化到内存
3. **获取那些被排除掉的抽奖列表，这些奖品可能是已经奖品库存为空，或者因为风控策略不能给这个用户薅羊毛的奖品**

1. **整体抽奖会排除，然后再计算全体的概率**
2. **单体抽奖 如何抽取到了直接返回null**

4. **执行抽奖算法**
5. 包装中奖结果

**注意：只是3、4需要重写（有多种方式**

## 0.定义模板接口`IDrawExec`

```java
public interface IDrawExec {

    /**
     * 抽奖方法
     * @param req 抽奖参数；用户ID、策略ID
     * @return    中奖结果
     */
    DrawResult doDrawExec(DrawReq req);

}
```

## 1. 定义抽象模板过程类`AbstractDrawBase`

```java
// DrawStrategySupport提供数据DAO的支持，IDrawExec是只有一个doDrawExec方法的接口
public abstract class AbstractDrawBase extends DrawStrategySupport implements IDrawExec {

    @Override
    public DrawResult doDrawExec(DrawReq req) {
        // 1. 获取抽奖策略
        StrategyRich strategyRich = super.queryStrategyRich(req.getStrategyId());
        Strategy strategy = strategyRich.getStrategy();

        // 2. 校验抽奖策略是否已经初始化到内存
        this.checkAndInitRateData(req.getStrategyId(), strategy.getStrategyMode(), strategyRich.getStrategyDetailList());

        // 3. 获取不在抽奖范围内的列表，包括：奖品库存为空、风控策略、临时调整等
        List<String> excludeAwardIds = this.queryExcludeAwardIds(req.getStrategyId());

        // 4. 执行抽奖算法
        String awardId = this.drawAlgorithm(req.getStrategyId(), drawAlgorithmGroup.get(strategy.getStrategyMode()), excludeAwardIds);

        // 5. 包装中奖结果
        return buildDrawResult(req.getuId(), req.getStrategyId(), awardId);
    }

    // 获取不在抽奖范围内的列表，包括：奖品库存为空、风控策略、临时调整等，这类数据是含有业务逻辑的，所以需要由具体的实现方决定
    protected abstract List<String> queryExcludeAwardIds(Long strategyId);

	// 执行抽奖算法
    protected abstract String drawAlgorithm(Long strategyId, IDrawAlgorithm drawAlgorithm, List<String> excludeAwardIds);

    // 校验抽奖策略是否已经初始化到内存
    private void checkAndInitRateData(Long strategyId, Integer strategyMode, List<StrategyDetail> strategyDetailList) {

        ...
    }

    // 包装抽奖结果
    private DrawResult buildDrawResult(String uId, Long strategyId, String awardId) {
        ...

        return new DrawResult(uId, strategyId, Constants.DrawState.SUCCESS.getCode(), drawAwardInfo);
    }

}
```

在上面可以看到，模板类提供了5个步骤

1. 获取抽奖策略

1. 调用父类DrawStrategySupport获得数据

2. 检验抽奖策略是否已经初始化

1. 调用本类`checkAndInitRateData()`，这个方法是`private`，子类不可以重写

3. 获取需要排除掉的列表

1. **调用本类**`_**queryExcludeAwardIds()**_`**，是**`**protected**`**，子类可以重写**
2. **这个就是模板模式的体现了**

4. 执行抽奖算法

1. **调用本类**`_**drawAlgorithm()**_`**，是**`**protected**`**，子类可以重写**

5. 返回结果

1. 调用本类`buildDrawResult()`，是private，子类不可重写

这5个步骤都调用了本类的方法，如果想要提供模板过程就把方法暴露出去（protected），由子类重写

_**之所以定义这2个抽象方法，是因为这2个方法可能随着实现方有不同的方式变化，不适合定义成通用的方法。**_

## 2. 子类继承模板`DrawExecImpl`

**类**`**DrawExecImpl**`**继承了模板类**`**AbstractDrawBase**`**，对其中的一些方法进行重写。这样，当调用模板时，执行的是模板类的流程，但是具体的方法则是执行的**`**DrawExecImpl**`

- **在**`drawAlgorithm`更新`activity_detail`库存⭐️⭐️⭐️

```java
// 注入Spring容器
@Service("drawExec")
public class DrawExecImpl extends AbstractDrawBase {

    private Logger logger = LoggerFactory.getLogger(DrawExecImpl.class);

    @Override
    protected List<String> queryExcludeAwardIds(Long strategyId) {
        List<String> awardList = strategyRepository.queryNoStockStrategyAwardList(strategyId);
        logger.info("执行抽奖策略 strategyId：{}，无库存排除奖品列表ID集合 awardList：{}", strategyId, JSON.toJSONString(awardList));
        return awardList;
    }

    @Override
    protected String drawAlgorithm(Long strategyId, IDrawAlgorithm drawAlgorithm, List<String> excludeAwardIds) {
        // 执行抽奖，扣除`activity_detail`库存⭐️⭐️⭐️
        String awardId = drawAlgorithm.randomDraw(strategyId, excludeAwardIds);

        // 判断抽奖结果
        if (null == awardId) {
            return null;
        }

        /*
         * 扣减库存，暂时采用数据库行级锁的方式进行扣减库存，后续优化为 Redis 分布式锁扣减 decr/incr
         * 注意：通常数据库直接锁行记录的方式并不能支撑较大体量的并发，但此种方式需要了解，因为在分库分表下的正常数据流量下的个人数据记录中，是可以使用行级锁的，因为他只影响到自己的记录，不会影响到其他人
         */
        boolean isSuccess = strategyRepository.deductStock(strategyId, awardId);

        // 返回结果，库存扣减成功返回奖品ID，否则返回NULL 「在实际的业务场景中，如果中奖奖品库存为空，则会发送兜底奖品，比如各类券」
        return isSuccess ? awardId : null;
    }

}
```

- 重写了`queryExcludeAwardIds()`

- 后续可能添加进来一些其他的需要检验的项

- `drawAlgorithm()`

- 后续可能需要将库存更新由MySQL更新为Redis

## 3. 使用模板类

- 注入`DrawExecImpl`类，但是**类型是模板接口类型**`**IDrawExec**`

```java
// @Resource按名称注入，注入的是 DrawExecImpl 类：@Service("drawExec")
@Resource
private IDrawExec drawExec;

@Test
public void test_drawExec() {
    drawExec.doDrawExec(new DrawReq("小傅哥", 10001L));
}
```

spring去容器中找`IDrwaExec`类型的，注解名字为`drawExec`的bean，找到了就将`DrawExecImpl`注入进来

然后调用`DrawExecImpl对象`的`doDrawExec()`，走的是模板类，然后具体的方式使用的是`DrawExecImpl`重写的方法

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672573750637-1d897c9f-d97f-4956-b3b4-2938fa96a987.png)

---

# 七、简单工厂模式：搭建发奖领域

工厂模式

## 1. 普通工厂

就是建立一个工厂类，对实现了同一接口的一些类进行实例的创建，用户传入对应的类type，然后返回对应的

```java
public class SendFactory {  
    public Sender produce(String type) {  
        if ("mail".equals(type)) {  
            return new MailSender();  
        } else if ("sms".equals(type)) {  
            return new SmsSender();  
        } else {  
            System.out.println("请输入正确的类型!");  
            return null;  
        }  
    }  
}
```

## 2. 静态工厂

只有在调用时才去new，直接调用对应的工厂方法

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1680436369011-2dba1344-163a-45d1-a370-bcd252c1fd1b.png)

## 3. 工厂方法模式

通过依赖注入的思想，还是只有一个工厂接口，只不过最后多态到对应的对象
```java

/**
* 抽象工厂
**/
public interface CoffeeFactory {
    Coffee createCoffee();
}

/**
* 具体工厂
* 
* 抽象产品为coffee，具体产品为LatteCoffee和AmericanCoffee
* 这种工厂模式可以通过不同的具体工厂创建出不同的具体产品
**/
@Bean("LatteCoffeeFactory")
public class LatteCoffeeFactory implements CoffeeFactory {
    public Coffee createCoffee() {
        return new LatteCoffee();
    }
}
@Bean("AmericanCoffeeFactory")
public class AmericanCoffeeFactory implements CoffeeFactory {
    public Coffee createCoffee() {
        return new AmericanCoffee();
    }
}
```

## 3. 抽象工厂模式

简单工厂&静态工厂两者不符合开闭原则，即如果要创建新的工厂还需要再去改动

抽象工厂是在工厂上又出一个接口，每个商品都有自己的工厂类去实现这个接口

**即想要啥工厂，就创建啥商品的工厂**
```java
// 工厂接口
public interface Provider {  
    public Sender produce();  
} 
// 对应工厂类
public class SendMailFactory implements Provider {  
      
    @Override  
    public Sender produce(){  
        return new MailSender();  
    }  
}  
public class SendSmsFactory implements Provider{  
  
    @Override  
    public Sender produce() {  
        return new SmsSender();  
    }  
}  

// 调用
public class Test {  
    public static void main(String[] args) {  
        Provider provider = new SendMailFactory();  
        Sender sender = provider.produce();  
        sender.Send();  
    }  
}  
```

本项目中是一个简单工厂：传入常量，返回对应的对象类

需求：抽奖得到奖品，奖品类型不同：优惠券、兑换码、实物等，每一个都有各自的处理，所以_**使用工厂模式来发放对应的奖品**_

- 介绍：定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，**工厂模式使其创建过程延迟到子类进行**。
- 和策略模式一样，也是利用一个Map将各种奖品的Bean都放进去，用户传入奖品代码即返回对应奖品的对象，由其本身去执行发奖

![800](https://cdn.nlark.com/yuque/0/2022/png/300264/1672387371184-12b95ad9-7412-4cb2-8544-5a941997937b.png)

## 1. 项目结构

![](https://cdn.nlark.com/yuque/0/2022/png/300264/1672387751421-b80504a6-23fe-439a-8a74-b38148e7e75a.png)

## 2. 执行流程

先执行之前的抽奖算法`drawExec()`获得具体的商品，然后将商品类型传入工厂，返回具体的商品类，由**商品类进行抽奖**

**发送了奖品就更新**`**user_stragety_export**`**的发奖状态**

![](https://cdn.nlark.com/yuque/0/2022/jpeg/300264/1672389541990-a31e6893-c7a9-44d4-b561-48a6b81e2bdc.jpeg)

## 3. UML类图

![](https://cdn.nlark.com/yuque/0/2022/jpeg/300264/1672390954837-0d38e9a8-1c4d-4387-9a2c-983b5abd34e5.jpeg)

## 2. 测试使用
```java
@Test
public void test_award() {
    // 执行抽奖
    DrawResult drawResult = drawExec.doDrawExec(new DrawReq("小傅哥", 10001L));

    // 判断抽奖结果
    Integer drawState = drawResult.getDrawState();
    if (Constants.DrawState.FAIL.getCode().equals(drawState)) {
        logger.info("未中奖 DrawAwardInfo is null");
        return;
    }

    // 封装发奖参数，orderId：2109313442431 为模拟ID，需要在用户参与领奖活动时生成
    DrawAwardInfo drawAwardInfo = drawResult.getDrawAwardInfo();
    GoodsReq goodsReq = new GoodsReq(drawResult.getuId(), "2109313442431", drawAwardInfo.getAwardId(), drawAwardInfo.getAwardName(), drawAwardInfo.getAwardContent());

    // 根据 awardType 从抽奖工厂中获取对应的发奖服务
    IDistributionGoods distributionGoodsService = distributionGoodsFactory.getDistributionGoodsService(drawAwardInfo.getAwardType());
    DistributionRes distributionRes = distributionGoodsService.doDistribution(goodsReq);

    logger.info("测试结果：{}", JSON.toJSONString(distributionRes));
}
```

**在具体的商品类中，发放了商品还要将抽奖记录存入数据库，这部分后期做**

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672573787438-b470ff7f-163c-4003-a33e-80b8847e7749.png)

---

# 八、状态模式：活动领域的配置

这一页主要搞清楚数据存储 & DAO层操作

所有的领域domain存储都用的相同的数据存储架构

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677046689572-5f73a637-24af-4cc7-9243-cf7d28a09d29.png)

- 所有的对数据库的本质上的操作都是在`lottery-infrastructure`
- 领域类的封装都是在`lotter-domain`

- domain内有一个个的领域，每一个领域内对数据库的操作都封装在`repository`内，即是一些对业务的服务
- 比如`cn.itedus.lottery.domain.activity.repository`下就是对于`activity`的数据仓储服务接口封装`IActivityRepository`
- 这个`IActivityRepository`只是一个接口，真正的实现要交给`lottery.infrastructure.repository`的具体实现类，实现这个接口

- 对于`lottery-infrastructure`有三个文件夹：

- `dao`：负责对数据库的本质上的操作，由它来写（Mapper映射mybatis.xml）
- `po`：数据库实体类
- `repository`：每一个`domain`下的`repository`接口的具体实现类都在此

![|500](https://cdn.nlark.com/yuque/0/2022/png/300264/1672407845513-1f72b4b9-3842-4295-88da-2d04fe0d91fa.png)

## 状态模式

[https://blog.csdn.net/qq_35784669/article/details/121238278](https://blog.csdn.net/qq_35784669/article/details/121238278)

**有什么状态：未提交、待审核、审核通过、活动进行中、关闭**

**功能：用类来写的自动状态机，通过判断前置状态以限制状态流转**

**具体地，用** `**Abstractstate**` **定义状态，每个子类单独控制状态之间的流转许可，**`**IStateHandler**` **提供具体状态流转的操作服务**

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672571243562-43a2ff07-ecfa-4d5f-adff-5117642ff4bc.png)

活动的状态有上述多种：

1. 定义一个基类`AbstractState`，内部有状态流转的方法

```java
public abstract class AbstractState {

    @Resource
    protected IActivityRepository activityRepository;

    /**
     * 活动提审
     *
     * @param activityId   活动ID
     * @param currentState 当前状态
     * @return 执行结果
     */
    public abstract Result arraignment(Long activityId, Enum<Constants.ActivityState> currentState);

    /**
     * 审核通过
     *
     * @param activityId   活动ID
     * @param currentState 当前状态
     * @return 执行结果
     */
    public abstract Result checkPass(Long activityId, Enum<Constants.ActivityState> currentState);

    /**
     * 审核拒绝
     *
     * @param activityId   活动ID
     * @param currentState 当前状态
     * @return 执行结果
     */
    public abstract Result checkRefuse(Long activityId, Enum<Constants.ActivityState> currentState);

    /**
     * 撤审撤销
     *
     * @param activityId   活动ID
     * @param currentState 当前状态
     * @return 执行结果
     */
    public abstract Result checkRevoke(Long activityId, Enum<Constants.ActivityState> currentState);

    /**
     * 活动关闭
     *
     * @param activityId   活动ID
     * @param currentState 当前状态
     * @return 执行结果
     */
    public abstract Result close(Long activityId, Enum<Constants.ActivityState> currentState);

    /**
     * 活动开启
     *
     * @param activityId   活动ID
     * @param currentState 当前状态
     * @return 执行结果
     */
    public abstract Result open(Long activityId, Enum<Constants.ActivityState> currentState);

    /**
     * 活动执行
     *
     * @param activityId   活动ID
     * @param currentState 当前状态
     * @return 执行结果
     */
    public abstract Result doing(Long activityId, Enum<Constants.ActivityState> currentState);

}
```

2. 对于每一种具体的状态都给其封装成一个Java Bean，继承自`AbstractState`，实现其方法，以活动关闭状态`CloseState`为例

1. _**在这些方法中所有的入参都是一样的，**_`_**activityId**_`_**(活动ID)、**_`_**currentStatus**_`_**(当前状态)，只有他们的具体实现是不同的（根据状态来的）。**_
```java
@Component
public class CloseState extends AbstractState {

    @Override
    public Result arraignment(Long activityId, Enum<Constants.ActivityState> currentState) {
        return Result.buildResult(Constants.ResponseCode.UN_ERROR, "活动关闭不可提审");
    }

    @Override
    public Result checkPass(Long activityId, Enum<Constants.ActivityState> currentState) {
        return Result.buildResult(Constants.ResponseCode.UN_ERROR, "活动关闭不可审核通过");
    }

    @Override
    public Result checkRefuse(Long activityId, Enum<Constants.ActivityState> currentState) {
        return Result.buildResult(Constants.ResponseCode.UN_ERROR, "活动关闭不可审核拒绝");
    }

    @Override
    public Result checkRevoke(Long activityId, Enum<Constants.ActivityState> currentState) {
        return Result.buildResult(Constants.ResponseCode.UN_ERROR, "活动关闭不可撤销审核");
    }

    @Override
    public Result close(Long activityId, Enum<Constants.ActivityState> currentState) {
        return Result.buildResult(Constants.ResponseCode.UN_ERROR, "活动关闭不可重复关闭");
    }

    @Override
    public Result open(Long activityId, Enum<Constants.ActivityState> currentState) {
        boolean isSuccess = activityRepository.alterStatus(activityId, currentState, Constants.ActivityState.OPEN);
        return isSuccess ? Result.buildResult(Constants.ResponseCode.SUCCESS, "活动开启完成") : Result.buildErrorResult("活动状态变更失败");
    }

    @Override
    public Result doing(Long activityId, Enum<Constants.ActivityState> currentState) {
        return Result.buildResult(Constants.ResponseCode.UN_ERROR, "活动关闭不可变更活动中");
    }

}
```

3. 然后将这七个状态封装进一个配置类（注入）
```java
public class StateConfig {
	// 注入
    @Resource
    private ArraignmentState arraignmentState;
    @Resource
    private CloseState closeState;
    @Resource
    private DoingState doingState;
    @Resource
    private EditingState editingState;
    @Resource
    private OpenState openState;
    @Resource
    private PassState passState;
    @Resource
    private RefuseState refuseState;

    protected Map<Enum<Constants.ActivityState>, AbstractState> stateGroup = new ConcurrentHashMap<>();

    @PostConstruct
    public void init() {
        stateGroup.put(Constants.ActivityState.ARRAIGNMENT, arraignmentState);
        stateGroup.put(Constants.ActivityState.CLOSE, closeState);
        stateGroup.put(Constants.ActivityState.DOING, doingState);
        stateGroup.put(Constants.ActivityState.EDIT, editingState);
        stateGroup.put(Constants.ActivityState.OPEN, openState);
        stateGroup.put(Constants.ActivityState.PASS, passState);
        stateGroup.put(Constants.ActivityState.REFUSE, refuseState);
    }

}
```

4. 对外暴露出状态处理接口`IStateHandler`
```java
public interface IStateHandler {

    /**
     * 提审
     * @param activityId    活动ID
     * @param currentStatus 当前状态
     * @return              审核结果
     */
    Result arraignment(Long activityId, Enum<Constants.ActivityState> currentStatus);

    /**
     * 审核通过
     * @param activityId    活动ID
     * @param currentStatus 当前状态
     * @return              审核结果
     */
    Result checkPass(Long activityId, Enum<Constants.ActivityState> currentStatus);

    /**
     * 审核拒绝
     * @param activityId    活动ID
     * @param currentStatus 当前状态
     * @return              审核结果
     */
    Result checkRefuse(Long activityId, Enum<Constants.ActivityState> currentStatus);

    /**
     * 撤销审核
     * @param activityId    活动ID
     * @param currentStatus 当前状态
     * @return              审核结果
     */
    Result checkRevoke(Long activityId, Enum<Constants.ActivityState> currentStatus);

    /**
     * 关闭
     * @param activityId    活动ID
     * @param currentStatus 当前状态
     * @return              审核结果
     */
    Result close(Long activityId, Enum<Constants.ActivityState> currentStatus);

    /**
     * 开启
     * @param activityId    活动ID
     * @param currentStatus 当前状态
     * @return              审核结果
     */
    Result open(Long activityId, Enum<Constants.ActivityState> currentStatus);

    /**
     * 运行活动中
     * @param activityId    活动ID
     * @param currentStatus 当前状态
     * @return              审核结果
     */
    Result doing(Long activityId, Enum<Constants.ActivityState> currentStatus);
    
}

```
5. 定义一个具体的状态处理类，调用状态处理方法（内部就是根据状态来调用对应的方法）
```java
@Service
public class StateHandlerImpl extends StateConfig implements IStateHandler {

    @Override
    public Result arraignment(Long activityId, Enum<Constants.ActivityState> currentStatus) {
        return stateGroup.get(currentStatus).arraignment(activityId, currentStatus);
    }

    @Override
    public Result checkPass(Long activityId, Enum<Constants.ActivityState> currentStatus) {
        return stateGroup.get(currentStatus).checkPass(activityId, currentStatus);
    }

    @Override
    public Result checkRefuse(Long activityId, Enum<Constants.ActivityState> currentStatus) {
        return stateGroup.get(currentStatus).checkRefuse(activityId, currentStatus);
    }

    @Override
    public Result checkRevoke(Long activityId, Enum<Constants.ActivityState> currentStatus) {
        return stateGroup.get(currentStatus).checkRevoke(activityId, currentStatus);
    }

    @Override
    public Result close(Long activityId, Enum<Constants.ActivityState> currentStatus) {
        return stateGroup.get(currentStatus).close(activityId, currentStatus);
    }

    @Override
    public Result open(Long activityId, Enum<Constants.ActivityState> currentStatus) {
        return stateGroup.get(currentStatus).open(activityId, currentStatus);
    }

    @Override
    public Result doing(Long activityId, Enum<Constants.ActivityState> currentStatus) {
        return stateGroup.get(currentStatus).doing(activityId, currentStatus);
    }

}
```

这里用到`extends StateConfig`，其实也可以换为内部类使用：`new StateConfig().get(currentStatus)`

_**根据传入的状态码来调用对应的状态类**_

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672573095969-06df33b5-ec2b-4931-b9c3-aa14ac94c5c7.png)

**因为****活动创建****有多步，采用**`**@Transactional****(rollbackFor = Exception.class)**`**作为事务控制（这个不用分库分表，所以不用使用DBRouter的事务机制）**

- 添加活动
- 添加奖品
- 添加对应策略

@Transactional(rollbackFor = Exception.class)
@Override
public void createActivity(ActivityConfigReq req) {
    logger.info("创建活动配置开始，activityId：{}", req.getActivityId());
    ActivityConfigRich activityConfigRich = req.getActivityConfigRich();
    try {
        // 添加活动配置
        ActivityVO activity = activityConfigRich.getActivity();
        activityRepository.addActivity(activity);

        // 添加奖品配置
        List<AwardVO> awardList = activityConfigRich.getAwardList();
        activityRepository.addAward(awardList);

        // 添加策略配置
        StrategyVO strategy = activityConfigRich.getStrategy();
        activityRepository.addStrategy(strategy);

        // 添加策略明细配置
        List<StrategyDetailVO> strategyDetailList = activityConfigRich.getStrategy().getStrategyDetailList();
        activityRepository.addStrategyDetailList(strategyDetailList);

        logger.info("创建活动配置完成，activityId：{}", req.getActivityId());
    } catch (DuplicateKeyException e) {
        logger.error("创建活动配置失败，唯一索引冲突 activityId：{} reqJson：{}", req.getActivityId(), JSON.toJSONString(req), e);
        throw e;
    }
}

---

# 九、ID生成策略·领域开发

## 1. @Configuration + @Bean

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672579504379-dc5ac13f-6f58-47b9-a284-0cfd65aebb13.png)

## 2. 三种ID生成策略

**这三种都加上**`**synchronized**`

在 domain 领域包下新增支撑领域，ID 的生成服务就放到这个领域下实现。

关于 ID 的生成因为有三种不同 ID 用于在不同的场景下；

- 订单号：唯一、大量、订单创建时使用、分库分表
- 活动号：唯一、少量、活动创建时使用、单库单表
- 策略号：唯一、少量、活动创建时使用、单库单表

策略模式，是因为外部的调用方会需要根据不同的场景来选择出适合的ID生成策略，而策略模式就非常适合这一场景的使用，有三种ID生成策略

三种ID生成策略都_**需要实现一个**_`_**IIdGenerator**_`_**，用于实现多态**_

public interface IIdGenerator {

    /**
     * 获取ID，目前有两种实现方式
     * 1. 雪花算法，用于生成单号
     * 2. 日期算法，用于生成活动编号类，特性是生成数字串较短，但指定时间内不能生成太多
     * 3. 随机算法，用于生成策略ID
     *
     * @return ID
     */
    long nextId();

}

### ① `RandomNumeric`纯随机数，随机Id生成器，用lang3中的工具实现

@Component
public class RandomNumeric implements IIdGenerator {

    @Override
    public long nextId() {
        return Long.parseLong(RandomStringUtils.randomNumeric(11));
    }

}

### ② 日期算法，打乱排序：2020年为准 + 小时 + 周期 + 日 + 三位随机数

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672580467901-c77ab043-39b6-4a19-b9b6-0d3c00d965c1.png)

年月日小时要打乱顺序，为什么？防止黑客直接拼接，即更安全

`String.format("%03d", new Random().nextInt(1000))`：让字符串固定位，即这个数有可能是一位或者2位，那么给他固定成三位（0补齐）

@Component
public class ShortCode implements IIdGenerator {

    @Override
    public synchronized long nextId() {
        Calendar calendar = Calendar.getInstance();
        int year = calendar.get(Calendar.YEAR);
        int week = calendar.get(Calendar.WEEK_OF_YEAR);
        int day = calendar.get(Calendar.DAY_OF_WEEK);
        int hour = calendar.get(Calendar.HOUR_OF_DAY);

        // 打乱排序：2020年为准 + 小时 + 周期 + 日 + 三位随机数
        StringBuilder idStr = new StringBuilder();
        idStr.append(year - 2020);
        idStr.append(hour);
        idStr.append(String.format("%02d", week));
        idStr.append(day);
        idStr.append(String.format("%03d", new Random().nextInt(1000)));

        return Long.parseLong(idStr.toString());
    }

}

### ③ 雪花算法

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672580488651-aac75f2f-4b5b-44fd-aa4f-ac8bb8a45a0a.png)

@Component
public class SnowFlake implements IIdGenerator {

    private Snowflake snowflake;

    @PostConstruct	// 注意这里先构造好了，即采用配置的方式先初始化了 ⭐️⭐️⭐️
    public void init() {
        // 0 ~ 31 位，可以采用配置的方式使用
        long workerId;
        try {
            workerId = NetUtil.ipv4ToLong(NetUtil.getLocalhostStr());
        } catch (Exception e) {
            workerId = NetUtil.getLocalhostStr().hashCode();
        }

        // IP右移16位，然后取低5位（31变为二进制就是11111）
        workerId = workerId >> 16 & 31;

        long dataCenterId = 1L;
        snowflake = IdUtil.createSnowflake(workerId, dataCenterId);
    }

    // 注意是synchronized的⭐️⭐️⭐️
    @Override
    public synchronized long nextId() {
        // 12位，自增序列号，一共2^12-1=4096，即一毫秒内可以生成4096个自增序列
        return snowflake.nextId();
    }

}

## 3. 策略模式

1. _**用@Configuration、@Bean的方式**_，将**缓存三种策略的hashMap直接注册到了IOC容器里**。

1. 三种策略都是加了`@Component`，在方法内（作为`idGeanarator()`的参数传入）需要用到时Spring会去容器找的

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672580980647-c33047de-d686-4277-b6c6-5fe222cca0b9.png)![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672580849619-b4ae5e90-1410-42a8-b34b-5230db9c5255.png)

2. 第五章也用了策略模式，方式不太一样，它在 `**DrawConfig**` **类里定义了**`**hashMap**`**字段来缓存抽奖策略，通过继承类（**`**static**`**）的方式从hashMap里取出具体策略使用**

本章使用的策略模式，感觉更简洁更解耦！

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672580937677-9b67e0c0-9614-4630-91fb-5261a9d1a2ae.png)

**像这种直接从Spring容器取，传入的都是之前写好的常量**

---

# 十、自研分库分表中间件

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1681026455585-3d45631f-7846-433f-a0f0-05b789af612e.png)

## 如何将配置文件读取到内存

- 配置文件写着分表数分库数，需要先读取进来配置到一个Map中
- **实现**`**EnvironmentAware**`**接口，重写**`**setEnvironment(Environment environment****)**`**，在Spring启动时即可读取配置文件值**
- **这个配置文件是在lottery工程，因为DBRouter只是作为一个starter导入进来**
- **将配置文件的多个数据库配置读取进来，然后挨个构建spring的jdbc的管理器**

mini-db-router:
  jdbc:
    datasource:
      dbCount: 2
      tbCount: 4
      default: db00
      routerKey: uId
      list: db01,db02
      db00:
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://127.0.0.1:3306/lottery?useUnicode=true&characterEncoding=utf8&autoReconnect=true&zeroDateTimeBehavior=convertToNull&serverTimezone=UTC&useSSL=true
        username: root
        password: 1234
      db01:
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://127.0.0.1:3306/lottery_01?useUnicode=true&characterEncoding=utf8&autoReconnect=true&zeroDateTimeBehavior=convertToNull&serverTimezone=UTC&useSSL=true
        username: root
        password: 1234
      db02:
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://127.0.0.1:3306/lottery_02?useUnicode=true&characterEncoding=utf8&autoReconnect=true&zeroDateTimeBehavior=convertToNull&serverTimezone=UTC&useSSL=true
        username: root
        password: 1234

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1678696613981-6e77b90b-fe01-479f-970e-36e113a6aadf.png)`org.springframework.boot.autoconfigure.EnableAutoConfiguration=cn.bugstack.middleware.db.router.config.DataSourceAutoConfig`

这样将配置类自动注入：`DataSourceAutoConfig`，然后会去看`@ConditionOnMission`，将这个`JointPoint`类加载到容器

/**
 * @description: 数据源配置解析
 */
@Configuration
public class DataSourceAutoConfig implements EnvironmentAware {

    /**
     * 数据源配置组
     */
    private Map<String, Map<String, Object>> dataSourceMap = new HashMap<>();

    /**
     * 默认数据源配置
     */
    private Map<String, Object> defaultDataSourceConfig;

    /**
     * 分库数量
     */
    private int dbCount;

    /**
     * 分表数量
     */
    private int tbCount;

    /**
     * 路由字段
     */
    private String routerKey;

    @Bean(name = "db-router-point")
    @ConditionalOnMissingBean
    public DBRouterJoinPoint point(DBRouterConfig dbRouterConfig, IDBRouterStrategy dbRouterStrategy) {
        return new DBRouterJoinPoint(dbRouterConfig, dbRouterStrategy);
    }

    @Bean
    public DBRouterConfig dbRouterConfig() {
        return new DBRouterConfig(dbCount, tbCount, routerKey);
    }

    @Bean
    public Interceptor plugin() {
        return new DynamicMybatisPlugin();
    }

    @Bean
    public DataSource dataSource() {
        // 创建数据源
        Map<Object, Object> targetDataSources = new HashMap<>();
        for (String dbInfo : dataSourceMap.keySet()) {
            Map<String, Object> objMap = dataSourceMap.get(dbInfo);
            targetDataSources.put(dbInfo, new DriverManagerDataSource(objMap.get("url").toString(), objMap.get("username").toString(), objMap.get("password").toString()));
        }

        // 设置数据源
        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        dynamicDataSource.setTargetDataSources(targetDataSources);
        dynamicDataSource.setDefaultTargetDataSource(new DriverManagerDataSource(defaultDataSourceConfig.get("url").toString(), defaultDataSourceConfig.get("username").toString(), defaultDataSourceConfig.get("password").toString()));

        return dynamicDataSource;
    }

    @Bean
    public IDBRouterStrategy dbRouterStrategy(DBRouterConfig dbRouterConfig) {
        return new DBRouterStrategyHashCode(dbRouterConfig);
    }

    @Bean
    public TransactionTemplate transactionTemplate(DataSource dataSource) {
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
        dataSourceTransactionManager.setDataSource(dataSource);

        TransactionTemplate transactionTemplate = new TransactionTemplate();
        transactionTemplate.setTransactionManager(dataSourceTransactionManager);
        transactionTemplate.setPropagationBehaviorName("PROPAGATION_REQUIRED");
        return transactionTemplate;
    }

    @Override
    public void setEnvironment(Environment environment) {
        String prefix = "mini-db-router.jdbc.datasource.";

        dbCount = Integer.valueOf(environment.getProperty(prefix + "dbCount"));
        tbCount = Integer.valueOf(environment.getProperty(prefix + "tbCount"));
        routerKey = environment.getProperty(prefix + "routerKey");

        // 分库分表数据源
        String dataSources = environment.getProperty(prefix + "list");
        assert dataSources != null;
        for (String dbInfo : dataSources.split(",")) {
            Map<String, Object> dataSourceProps = PropertyUtil.handle(environment, prefix + dbInfo, Map.class);
            dataSourceMap.put(dbInfo, dataSourceProps);
        }

        // 默认数据源
        String defaultData = environment.getProperty(prefix + "default");
        defaultDataSourceConfig = PropertyUtil.handle(environment, prefix + defaultData, Map.class);

    }

}

## 根据什么路由？

1. 用户ID uid 作为路由key
2. 使用 **斐波那契散列**

## 如何动态配置的多数据源？

1. **继承**`**extends AbstractRoutingDataSource**`
2. **实现**`**determineCurrentLookupKey()**`**：根据返回值去配置Map内取对应的数据源配置**
3. **然后****每次切换数据源****都会过一遍这个方法，设置数据源**

/**
 * @description: 动态数据源获取，每当切换数据源，都要从这个里面进行获取
 */
public class DynamicDataSource extends AbstractRoutingDataSource {

    @Override
    protected Object determineCurrentLookupKey() {
        return "db" + DBContextHolder.getDBKey();
    }

}

## 如何进行分库？

[https://blog.csdn.net/qq_26950567/article/details/119786124](https://blog.csdn.net/qq_26950567/article/details/119786124)

这个类是Java的jdbc的，这是SPI机制，由MySQL驱动去实现

spring中有一个抽象类`AbstractRoutingDataSource`，重写其`determineCurrentLookupKey()`将获得数据源，如：![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677423848767-16a412d0-b377-4d1a-877a-1d87bd24e316.png)

DBRouter将`dbKey``tbKey`放在两个ThreadLocal中：![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677423951588-faeafb27-6dbf-4a66-a6db-6aaeebde4b9d.png)

凡是加了`@DBRouter`的注解都是分库了：

1. 使用 **AOP @Aspect** 切面拦截，调用`IDBRouterStrategy.dbRouter(Uid)`进行路由（即随机出一个dbKey和tbKey）
2. 然后在获取数据库连接，会调用上面的`DynamicDataSource`配置到动态源，AbstractRoutingDataSource，会获取当前数据库配置源：就会走重写的`determineCurrentLookupKey()`
3. 完成 **分库**

/**
 * 所有需要分库分表的操作，都需要使用自定义注解进行拦截，拦截后读取方法中的入参字段，根据字段进行路由操作。
 * 1. dbRouter.key() 确定根据哪个字段进行路由
 * 2. getAttrValue 根据数据库路由字段，从入参中读取出对应的值。比如路由 key 是 uId，那么就从入参对象 Obj 中获取到 uId 的值。
 * 3. dbRouterStrategy.doRouter(dbKeyAttr) 路由策略根据具体的路由值进行处理
 * 4. 路由处理完成比，就是放行。 jp.proceed();
 * 5. 最后 dbRouterStrategy 需要执行 clear 因为这里用到了 ThreadLocal 需要手动清空。关于 ThreadLocal 内存泄漏介绍 https://t.zsxq.com/027QF2fae
 */
@Around("aopPoint() && @annotation(dbRouter)")
public Object doRouter(ProceedingJoinPoint jp, DBRouter dbRouter) throws Throwable {
    String dbKey = dbRouter.key();
    if (StringUtils.isBlank(dbKey) && StringUtils.isBlank(dbRouterConfig.getRouterKey())) {
        throw new RuntimeException("annotation DBRouter key is null！");
    }
    dbKey = StringUtils.isNotBlank(dbKey) ? dbKey : dbRouterConfig.getRouterKey();
    // 路由属性
    String dbKeyAttr = getAttrValue(dbKey, jp.getArgs());
    
    // 路由策略
    dbRouterStrategy.doRouter(dbKeyAttr);
    
    // 返回结果
    // 内部mybatis执行时，会执行 determineLookUpKey 导航到对应的DataSource
    try {
        return jp.proceed();
    } finally {
       dbRouterStrategy.clear();
    }
}

## 如何进行分表/如何拦截Mybatis

区分分库 分表

[https://blog.csdn.net/qq_34761108/article/details/79824983/](https://blog.csdn.net/qq_34761108/article/details/79824983/)

- 实现`Interceptor`接口，重写`intercept`方法

Mybatis自定义插件针对Mybatis四大对象（Executor、StatementHandler 、ParameterHandler 、ResultSetHandler ）进行拦截，具体拦截方式为：

- `**Executor**`：拦截执行器的方法(log记录)
- `**StatementHandler**` ：拦截Sql语法构建的处理

- method 为拦截的方法，`**method** = "**prepare**"`拦截预处理方法

- `**ParameterHandler**` ：拦截参数的处理
- `**ResultSetHandler**` ：拦截结果集的处理

使用反射

/**
 * @description: Mybatis 拦截器，通过对 SQL 语句的拦截处理，修改分表信息（使用正则+反射）
 */
// 
@Intercepts({@Signature(type = StatementHandler.class, method = "prepare", args = {Connection.class, Integer.class})})
public class DynamicMybatisPlugin implements Interceptor {

    // 这个正则表达式匹配以 "from"、"into" 或者 "update" 开头，后面跟着至少一个空格，然后紧跟着一个或多个字母数字字符（即一个单词）。
    private Pattern pattern = Pattern.compile("(from|into|update)[\\s]{1,}(\\w{1,})", Pattern.CASE_INSENSITIVE);

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 获取StatementHandler
        StatementHandler statementHandler = (StatementHandler) invocation.getTarget();
        MetaObject metaObject = MetaObject.forObject(statementHandler, SystemMetaObject.DEFAULT_OBJECT_FACTORY, SystemMetaObject.DEFAULT_OBJECT_WRAPPER_FACTORY, new DefaultReflectorFactory());
        MappedStatement mappedStatement = (MappedStatement) metaObject.getValue("delegate.mappedStatement");

        // 获取自定义注解判断是否进行分表操作
        String id = mappedStatement.getId();
        String className = id.substring(0, id.lastIndexOf("."));
        Class<?> clazz = Class.forName(className);
        DBRouterStrategy dbRouterStrategy = clazz.getAnnotation(DBRouterStrategy.class);
        // // 加上DBRouter是分库，分表只有 user_strategt_export 分表
        if (null == dbRouterStrategy || !dbRouterStrategy.splitTable()){
            return invocation.proceed();
        }

        // 获取SQL 并 替换SQL
        BoundSql boundSql = statementHandler.getBoundSql();
        String sql = boundSql.getSql();

        // 替换SQL表名 USER 为 USER_03
        Matcher matcher = pattern.matcher(sql);
        String tableName = null;
        if (matcher.find()) {
            tableName = matcher.group().trim();
        }
        assert null != tableName;
        String replaceSql = matcher.replaceAll(tableName + "_" + DBContextHolder.getTBKey());

        // 通过反射修改SQL语句
        Field field = boundSql.getClass().getDeclaredField("sql");
        field.setAccessible(true);
        field.set(boundSql, replaceSql);
        field.setAccessible(false);

        return invocation.proceed();
    }

}

// ExamplePlugin.java
@Intercepts({@Signature(
    type= Executor.class,
    method = "update",
    args = {MappedStatement.class,Object.class})})
    public class ExamplePlugin implements Interceptor {
        public Object intercept(Invocation invocation) throws Throwable {
            Object target = invocation.getTarget(); //被代理对象
            Method method = invocation.getMethod(); //代理方法
            Object[] args = invocation.getArgs(); //方法参数
            // do something ...... 方法拦截前执行代码块
            Object result = invocation.proceed();
            // do something .......方法拦截后执行代码块
            return result;
        }
        public Object plugin(Object target) {
            return Plugin.wrap(target, this);
        }
        public void setProperties(Properties properties) {
        }
    }  

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677046353474-f3316ed0-7052-4320-9af8-be1e6cca3d48.png)

### 时序图

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677047163315-ede7f5bf-8e64-4d99-819f-714872088a3c.png)

---

# 十一、「编程式事务」领取活动领域开发

[https://t.zsxq.com/qzFAAuB](https://t.zsxq.com/qzFAAuB)

- 声明式事务：在AOP基础上，对方法前后进行拦截，方法前开始事务，方法后提交事务
- 编程式事务：利用`TransactionTemplate`主动进行事务的开启

**没有用到分库分表直接用的就是声明事务：**`**@Transaction**`

**用到分库分表的都是**`**TransactionTemplate.execute()**`**：，需要先进行dbRouter.deRouter() 否则只会数据源变动事务会失效**

## 1. 自研组件DBRouter 扩展事务

- 问题：如果一个场景需要在同一个事务下，连续操作不同的DAO操作，那么就会涉及到在 DAO 上使用注解 @DBRouter(key = "uId") 反复切换路由的操作。虽然都是一个数据源，但这样切换后，事务就没法处理了。

- 每次进行DAO都会进行`doRouter()`：因为在`@DBRouter`切面拦截会进行

- 解决：这里选择了一个较低的成本的解决方案，就是把数据源的切换放在事务处理前，而事务操作也通过编程式编码进行处理。_具体可以参考 db-router-spring-boot-starter 源码_

### ① 拆解路由策略，单独提供路由方法

- 把路由算法拆解出来，无论是切面中还是硬编码，都通过这个方法进行计算路由

public interface IDBRouterStrategy {

    void doRouter(String dbKeyAttr);
    void clear();
}

// doRouter接口实现
@Override
public void doRouter(String dbKeyAttr) {
    int size = dbRouterConfig.getDbCount() * dbRouterConfig.getTbCount();

    // 扰动函数；在 JDK 的 HashMap 中，对于一个元素的存放，需要进行哈希散列。而为了让散列更加均匀，所以添加了扰动函数。扩展学习；https://mp.weixin.qq.com/s/CySTVqEDK9-K1MRUwBKRCg
    int idx = (size - 1) & (dbKeyAttr.hashCode() ^ (dbKeyAttr.hashCode() >>> 16));

    // 库表索引；相当于是把一个长条的桶，切割成段，对应分库分表中的库编号和表编号
    int dbIdx = idx / dbRouterConfig.getTbCount() + 1;
    int tbIdx = idx - dbRouterConfig.getTbCount() * (dbIdx - 1);

    // 设置到 ThreadLocal
    DBContextHolder.setDBKey(String.format("%02d", dbIdx));
    DBContextHolder.setTBKey(String.format("%03d", tbIdx));
    logger.debug("数据库路由 dbIdx：{} tbIdx：{}",  dbIdx, tbIdx);
}

### ② 配置事务对象

- 创建路由策略对象，便于切面和硬编码注入使用。
- 创建事务对象，用于编程式事务引入

@Bean
public TransactionTemplate transactionTemplate(DataSource dataSource) {
    DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
    dataSourceTransactionManager.setDataSource(dataSource);

    TransactionTemplate transactionTemplate = new TransactionTemplate();
    transactionTemplate.setTransactionManager(dataSourceTransactionManager);
    transactionTemplate.setPropagationBehaviorName("PROPAGATION_REQUIRED");
    return transactionTemplate;
}

## 2. 活动领取模板抽象类（模板模式

- 抽象类 BaseActivityPartake 继承数据支撑类并实现接口方法 IActivityPartake#doPartake
- 在领取活动 doPartake 方法中，先是通过父类提供的数据服务，获取到活动账单，再定义三个抽象方法：活动信息校验处理、扣减活动库存、领取活动，依次顺序解决活动的领取操作。

@Override
public PartakeResult doPartake(PartakeReq req) {

    // 1. 查询是否存在未执行抽奖领取活动单【user_take_activity 存在 state = 0，领取了但抽奖过程失败的，可以直接返回领取结果继续抽奖】
    UserTakeActivityVO userTakeActivityVO = this.queryNoConsumedTakeActivityOrder(req.getActivityId(), req.getuId());
    if (null != userTakeActivityVO) {
        return buildPartakeResult(userTakeActivityVO.getStrategyId(), userTakeActivityVO.getTakeId());
    }

    // 2. 查询活动账单
    ActivityBillVO activityBillVO = super.queryActivityBill(req);

    // 3. 活动信息校验处理【活动库存、状态、日期、个人参与次数】
    Result checkResult = this.checkActivityBill(req, activityBillVO);
    if (!Constants.ResponseCode.SUCCESS.getCode().equals(checkResult.getCode())) {
        return new PartakeResult(checkResult.getCode(), checkResult.getInfo());
    }

    // 4. 扣减活动库存【目前为直接对配置库中的 lottery.activity 直接操作表扣减库存，后续优化为Redis扣减】
    Result subtractionActivityResult = this.subtractionActivityStock(req);
    if (!Constants.ResponseCode.SUCCESS.getCode().equals(subtractionActivityResult.getCode())) {
        return new PartakeResult(subtractionActivityResult.getCode(), subtractionActivityResult.getInfo());
    }

    // 5. 插入领取活动信息【个人用户把活动信息写入到用户表】
    Long takeId = idGeneratorMap.get(Constants.Ids.SnowFlake).nextId();
    Result grabResult = this.grabActivity(req, activityBillVO, takeId);
    if (!Constants.ResponseCode.SUCCESS.getCode().equals(grabResult.getCode())) {
        return new PartakeResult(grabResult.getCode(), grabResult.getInfo());
    }

    return buildPartakeResult(activityBillVO.getStrategyId(), takeId);
}

    /**
     * 活动信息校验处理，把活动库存、状态、日期、个人参与次数
     *
     * @param partake 参与活动请求
     * @param bill    活动账单
     * @return 校验结果
     */
    protected abstract Result checkActivityBill(PartakeReq partake, ActivityBillVO bill);

    /**
     * 扣减活动库存
     *
     * @param req 参与活动请求
     * @return 扣减结果
     */
    protected abstract Result subtractionActivityStock(PartakeReq req);

    /**
     * 领取活动
     *
     * @param partake 参与活动请求
     * @param bill    活动账单
     * @return 领取结果
     */
    protected abstract Result grabActivity(PartakeReq partake, ActivityBillVO bill);

第4步和第5步要使用「MySQL-Redis」事务一致性，后面通过Redis分布式锁做到

## 3. 领取活动编程式事务处理

- dbRouter.doRouter(partake.getuId()); 是编程式处理分库分表，如果在不需要使用事务的场景下，直接使用注解配置到DAO方法上即可。_两个方式不能混用_
- transactionTemplate.execute 是编程式事务，用的就是路由中间件提供的事务对象，通过这样的方式也可以更加方便的处理细节的回滚，而不需要抛异常处理。

第5步的`this.grabActivity()`需要使用事务保证一致性：操作![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677053212749-603516a1-fb3a-47f7-a4ed-4858a0679e32.png)这两个表

protected Result grabActivity(PartakeReq partake, ActivityBillVO bill, Long takeId) {
    try {
        // 先进行数据路由散列，保证之后用的都是同一个Connection
        dbRouter.doRouter(partake.getuId());
        // transactionTemplate编程式事务
        return transactionTemplate.execute(status -> {
            try {
                // 扣减个人已参与次数
                int updateCount = userTakeActivityRepository.subtractionLeftCount(bill.getActivityId(), bill.getActivityName(), bill.getTakeCount(), bill.getUserTakeLeftCount(), partake.getuId());
                if (0 == updateCount) {
                    status.setRollbackOnly();
                    logger.error("领取活动，扣减个人已参与次数失败 activityId：{} uId：{}", partake.getActivityId(), partake.getuId());
                    return Result.buildResult(Constants.ResponseCode.NO_UPDATE);
                }

                // 写入领取活动记录
                userTakeActivityRepository.takeActivity(bill.getActivityId(), bill.getActivityName(), bill.getStrategyId(), bill.getTakeCount(), bill.getUserTakeLeftCount(), partake.getuId(), partake.getPartakeDate(), takeId);
            } catch (DuplicateKeyException e) {
                status.setRollbackOnly();
                logger.error("领取活动，唯一索引冲突 activityId：{} uId：{}", partake.getActivityId(), partake.getuId(), e);
                return Result.buildResult(Constants.ResponseCode.INDEX_DUP);
            }
            return Result.buildSuccessResult();
        });
    } finally {
        dbRouter.clear();
    }
}

## 声明式编程 @Transaction

---

# 十二、在应用层编排抽奖过程

**在**`**lottery-application**`**应用启动模块，编排整个的抽奖过程：**

1. **领取活动**

1. **首次领取 发送MQ 执行库存扣除**
2. **如果还有已领取未消费的 就不领取 ，用这个旧的**

2. **执行抽奖**
3. **结果落库**
4. **发送MQ，触发发奖过程**
5. **返回结果**

整个抽奖过程也是按照DDD结构来的，抽象成一个`process`包，它的实现其实就是一层浅封装（封装之前实现的各个模块）

## 1. 编排流程

对于每一个流程节点编排的内容，**都是在领域层开发完成的，而应用层只是做最为简单的且很薄的一层**。_其实这块也很符合目前很多低代码的使用场景，通过界面可视化控制流程编排，生成代码_

@Override
public DrawProcessResult doDrawProcess(DrawProcessReq req) {
    // 1. 领取活动
    PartakeResult partakeResult = activityPartake.doPartake(new PartakeReq(req.getuId(), req.getActivityId()));
	// 如果有已领取但还没使用的就先用这个旧的
    if (!Constants.ResponseCode.SUCCESS.getCode().equals(partakeResult.getCode()) && !Constants.ResponseCode.NOT_CONSUMED_TAKE.getCode().equals(partakeResult.getCode())) {
        return new DrawProcessResult(partakeResult.getCode(), partakeResult.getInfo());
    }

    // 2. 首次成功领取活动，发送 MQ 消息
    if (Constants.ResponseCode.SUCCESS.getCode().equals(partakeResult.getCode())) {
        ActivityPartakeRecordVO activityPartakeRecord = new ActivityPartakeRecordVO();
        activityPartakeRecord.setuId(req.getuId());
        activityPartakeRecord.setActivityId(req.getActivityId());
        activityPartakeRecord.setStockCount(partakeResult.getStockCount());
        activityPartakeRecord.setStockSurplusCount(partakeResult.getStockSurplusCount());
        // 发送 MQ 消息：LotteryActivityPartakeRecord 记录抽奖行为
        kafkaProducer.sendLotteryActivityPartakeRecord(activityPartakeRecord);
    }

    Long strategyId = partakeResult.getStrategyId();
    Long takeId = partakeResult.getTakeId();

    // 3. 执行抽奖
    DrawResult drawResult = drawExec.doDrawExec(new DrawReq(req.getuId(), strategyId));
    if (Constants.DrawState.FAIL.getCode().equals(drawResult.getDrawState())) {
        return new DrawProcessResult(Constants.ResponseCode.LOSING_DRAW.getCode(), Constants.ResponseCode.LOSING_DRAW.getInfo());
    }
    DrawAwardVO drawAwardVO = drawResult.getDrawAwardInfo();

    // 4. 结果落库
    DrawOrderVO drawOrderVO = buildDrawOrderVO(req, strategyId, takeId, drawAwardVO);
    Result recordResult = activityPartake.recordDrawOrder(drawOrderVO);
    if (!Constants.ResponseCode.SUCCESS.getCode().equals(recordResult.getCode())) {
        return new DrawProcessResult(recordResult.getCode(), recordResult.getInfo());
    }

    // 5. 发送MQ，触发发奖流程：发奖 调用奖品工厂模式
    InvoiceVO invoiceVO = buildInvoiceVO(drawOrderVO);
    ListenableFuture<SendResult<String, Object>> future = kafkaProducer.sendLotteryInvoice(invoiceVO);
    future.addCallback(new ListenableFutureCallback<SendResult<String, Object>>() {

        @Override
        public void onSuccess(SendResult<String, Object> stringObjectSendResult) {
            // 4.1 MQ 消息发送完成，更新数据库表 user_strategy_export.mq_state = 1
            activityPartake.updateInvoiceMqState(invoiceVO.getuId(), invoiceVO.getOrderId(), Constants.MQState.COMPLETE.getCode());
        }

        @Override
        public void onFailure(Throwable throwable) {
            // 4.2 MQ 消息发送失败，更新数据库表 user_strategy_export.mq_state = 2 【等待定时任务扫码补偿MQ消息】
            activityPartake.updateInvoiceMqState(invoiceVO.getuId(), invoiceVO.getOrderId(), Constants.MQState.FAIL.getCode());
        }

    });

    // 6. 返回结果
    return new DrawProcessResult(Constants.ResponseCode.SUCCESS.getCode(), Constants.ResponseCode.SUCCESS.getInfo(), drawAwardVO);
}

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677056276591-8a95282b-46bf-4a2f-8e50-f1848bc0dbdd.png)

## 幂等性保证 ⭐️⭐️⭐️

### 1. 用户参与活动只参与一次 `user_take_activity`

在`user_take_activity`用一个state字段保证，0就是无参与，1是参与了，这样只消费一次（看见1就不消费了）

### 2. 用户发奖只发一次`user_export_strategy`

因为活动只参与一次，所以活动take_id应该是唯一的，如果有重复的活动进来了 那么就会报重复 也不能插入

上面保证了一次（只消费一次），然后用`takeId`保证唯一索引，这个takeId 是`user_take_activity`内的，只会生成一次（领取活动的单号作为ID），如果再来重复消费，他不会重新生成，而是报错回滚

---

# 十三、组合模式：规则引擎量化人群参与活动

## 1. 组合模式

组合模式：[https://baijiahao.baidu.com/s?id=1711869421433076203&wfr=spider&for=pc](https://baijiahao.baidu.com/s?id=1711869421433076203&wfr=spider&for=pc)

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672927987359-0c2d7a4e-2b61-4fe0-896c-3618f4d874c6.png)

`Component`是一个抽象类

- 接口`Interface`用来约束行为
- 抽象`abstract`用来定义能力
- 对于`abstract`，_**方法如果是**_`_**abstract**_`_**修饰，那么子类必须实现**_，所以可以使用`public/protected`则子类可有选择的实现重写
- 抽象方法必须存在于抽象类，但是抽象类可以有具体方法，子类直接使用（就和普通的父类一样）
- 抽象类不能`new`，接口也是，都是让new 子类

## 2. 数据库设计

先看视频：[https://wx.zsxq.com/dweb2/index/topic_detail/581111448222454](https://wx.zsxq.com/dweb2/index/topic_detail/581111448222454)

Lottery对规则就是用到了组合模式：存在于三个表内

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1672928909123-2d23b576-3b19-4a44-a649-dc921923bc32.png)

## 三、应用场景

说是过滤器，其实说法不对，而是一个**分类器：**

- **即用户进入，根据用户的信息（性别、年龄）给给到对应的活动上**
- **这个过程是一个决策的过程，所以叫决策树**

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1673265176729-12709f87-2bcd-4486-966c-7163e0d5829b.png)

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1673275573789-82bd0a0d-f445-4c66-afc0-0b59873dba67.png)

## 四、功能开发

重新构建一个`rule`领域，领域服务包含：

- `logic`：负责具体的节点过滤
- `engine`：负责整体的筛选，即从根节点到叶子结点的活动这个流程控制

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1673265655072-735847ab-e8a1-47f6-aa6a-8ad6c2177675.png)

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1673265756935-b79b1301-ceba-4cd3-be01-74c7e3c295a9.png)

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1673273032843-3a5b33cc-f4de-4d0e-84c5-48ed77791aa9.png)

### `engine`领域服务：编排整个过滤的流程

// 规则配置，导入规则实体，存入一个Map：logicFilterMap
public class EngineConfig {

    protected static Map<String, LogicFilter> logicFilterMap = new ConcurrentHashMap<>();

    @Resource
    private UserAgeFilter userAgeFilter;
    @Resource
    private UserGenderFilter userGenderFilter;

    @PostConstruct
    public void init() {
        logicFilterMap.put("userAge", userAgeFilter);
        logicFilterMap.put("userGender", userGenderFilter);
    }

}

public interface EngineFilter {

    /**
     * 规则过滤器接口
     *
     * @param matter      规则决策值
     * @return            规则决策结果
     */
    EngineResult process(final DecisionMatterReq matter);

}

/**
 * 完成整个规则过滤的过程
 *
 * @param treeRuleRich     	整个规则树
 * @param matter     		决策值，内部包含一个map，如 <age,25> <gender,male>
 * @return            		规则决策结果，即过滤到了最后的叶子结点，内含「活动ID」，过滤完成
 */
protected TreeNodeVO engineDecisionMaker(TreeRuleRich treeRuleRich, DecisionMatterReq matter) {
    TreeRootVO treeRoot = treeRuleRich.getTreeRoot();
    Map<Long, TreeNodeVO> treeNodeMap = treeRuleRich.getTreeNodeMap();

    // 获得规则树 树根节点
    Long rootNodeId = treeRoot.getTreeRootNodeId();
    TreeNodeVO treeNodeInfo = treeNodeMap.get(rootNodeId);

    // 节点类型[NodeType]；1 非叶子结点、2 叶子节点
    // STEM根茎，即非叶子节点
    // 如果节点是非叶子结点，那么就要执行对应的规则过滤了
    // 循环退出时，是到了叶子结点了
    while (Constants.NodeType.STEM.equals(treeNodeInfo.getNodeType())) {
        // 先取得该节点对应的过滤规则实体
        String ruleKey = treeNodeInfo.getRuleKey();
        LogicFilter logicFilter = logicFilterMap.get(ruleKey);

        // 获得该过滤规则的对照值
        String matterValue = logicFilter.matterValue(matter);

        //进行过滤，获得下一层的节点Id
        Long nextNode = logicFilter.filter(matterValue, treeNodeInfo.getTreeNodeLineInfoList());
        // 获得下一层节点对应的过滤规则实体
        treeNodeInfo = treeNodeMap.get(nextNode);
        logger.info("决策树引擎=>{} userId：{} treeId：{} treeNode：{} ruleKey：{} matterValue：{}", treeRoot.getTreeName(), matter.getUserId(), matter.getTreeId(), treeNodeInfo.getTreeNodeId(), ruleKey, matterValue);
    }

    // 最后返回的是 叶子结点，即过滤到了对应的活动ID
    return treeNodeInfo;
}

@Service("ruleEngineHandle")
public class RuleEngineHandle extends EngineBase {

    @Resource
    private IRuleRepository ruleRepository;

    // 对内部过滤进行了一层封装
    @Override
    public EngineResult process(DecisionMatterReq matter) {
        // 获得完整的「决策规则树」
        TreeRuleRich treeRuleRich = ruleRepository.queryTreeRuleRich(matter.getTreeId());
        if (null == treeRuleRich) {
            throw new RuntimeException("Tree Rule is null!");
        }

        // 获得决策节点
        // matter是一个Map，由内部对应的 UserAgeFilter UserGenderFilter进行对比
        TreeNodeVO treeNodeInfo = engineDecisionMaker(treeRuleRich, matter);

        // 决策结果
        return new EngineResult(matter.getUserId(), treeNodeInfo.getTreeId(), treeNodeInfo.getTreeNodeId(), treeNodeInfo.getNodeValue());
    }

}

### `logic`领域服务：负责具体的每个规则过滤

login负责具体的节点过滤服务，即对每个节点进行对应的过滤：对性别判别，对年龄判别。

注入到了`EngineConfig`

public interface LogicFilter {

    /**
     * 逻辑决策器
     * @param matterValue          决策值
     * @param treeNodeLineInfoList 决策节点
     * @return                     下一个节点Id
     */
    Long filter(String matterValue, List<TreeNodeLineVO> treeNodeLineInfoList);

    /**
     * 获取决策值
     *
     * @param decisionMatter 决策物料
     * @return               决策值
     */
    String matterValue(DecisionMatterReq decisionMatter);

}

public abstract class BaseLogic implements LogicFilter {

    @Override
    public Long filter(String matterValue, List<TreeNodeLineVO> treeNodeLineInfoList) {
        // treeNodeLineInfoList 只包含两个TreeNodeLineVO，因为是一个节点的 左右子树的连线
        // 所以这两个一定会返回一个结果为 true
        for (TreeNodeLineVO nodeLine : treeNodeLineInfoList) {
            if (decisionLogic(matterValue, nodeLine)) {
                return nodeLine.getNodeIdTo();
            }
        }
        return Constants.Global.TREE_NULL_NODE;
    }

    /**
     * 获取规则比对值
     * @param decisionMatter 决策物料
     * @return 比对值
     */
    @Override
    public abstract String matterValue(DecisionMatterReq decisionMatter);

    // 具体的某个规则实体 获得过滤结果
    // 数据库使用1-7字段进行一一对应 
    // 要和数据库的设计对应上 ⭐️⭐️⭐️⭐️⭐️
    private boolean decisionLogic(String matterValue, TreeNodeLineVO nodeLine) {
        switch (nodeLine.getRuleLimitType()) {
            case Constants.RuleLimitType.EQUAL:
                return matterValue.equals(nodeLine.getRuleLimitValue());
            case Constants.RuleLimitType.GT:
                return Double.parseDouble(matterValue) > Double.parseDouble(nodeLine.getRuleLimitValue());
            case Constants.RuleLimitType.LT:
                return Double.parseDouble(matterValue) < Double.parseDouble(nodeLine.getRuleLimitValue());
            case Constants.RuleLimitType.GE:
                return Double.parseDouble(matterValue) >= Double.parseDouble(nodeLine.getRuleLimitValue());
            case Constants.RuleLimitType.LE:
                return Double.parseDouble(matterValue) <= Double.parseDouble(nodeLine.getRuleLimitValue());
            default:
                return false;
        }
    }

}

@Component
public class UserAgeFilter extends BaseLogic {

    // 自己完全可以自定义一些其他校验规则

    @Override
    public String matterValue(DecisionMatterReq decisionMatter) {
        return decisionMatter.getValMap().get("age").toString();
    }

}

@Component
public class UserGenderFilter extends BaseLogic {

    @Override
    public String matterValue(DecisionMatterReq decisionMatter) {
        return decisionMatter.getValMap().get("gender").toString();
    }
    
}

### 构建规则树`queryTreeRuleRich` ⭐️⭐️⭐️

**原理：先查询所有属于这颗数的Node，然后for每一个Node，看这个Node的NodeLine（通过传入TreeID & IdFrom）**

@Override
public TreeRuleRich queryTreeRuleRich(Long treeId) {

    // 规则树
    RuleTree ruleTree = ruleTreeDao.queryRuleTreeByTreeId(treeId);
    TreeRootVO treeRoot = new TreeRootVO();
    treeRoot.setTreeId(ruleTree.getId());
    treeRoot.setTreeRootNodeId(ruleTree.getTreeRootNodeId());
    treeRoot.setTreeName(ruleTree.getTreeName());

    // 树节点->树连接线
    Map<Long, TreeNodeVO> treeNodeMap = new HashMap<>();

    // 根据 rootId 查询出这棵树的所有节点
    List<RuleTreeNode> ruleTreeNodeList = ruleTreeNodeDao.queryRuleTreeNodeList(treeId);
    for (RuleTreeNode treeNode : ruleTreeNodeList) {
        List<TreeNodeLineVO> treeNodeLineInfoList = new ArrayList<>();
        // 如果节点是非叶子节点
        if (Constants.NodeType.STEM.equals(treeNode.getNodeType())) {
            // 查询此节点的两个左右子节点（根据TreeId 和 NodeIdFrom一定能查出两个左右孩子）
            RuleTreeNodeLine ruleTreeNodeLineReq = new RuleTreeNodeLine();
            ruleTreeNodeLineReq.setTreeId(treeId);
            ruleTreeNodeLineReq.setNodeIdFrom(treeNode.getId());
            List<RuleTreeNodeLine> ruleTreeNodeLineList = ruleTreeNodeLineDao.queryRuleTreeNodeLineList(ruleTreeNodeLineReq);

            // 遍历这两个左右子孩子的线
            for (RuleTreeNodeLine nodeLine : ruleTreeNodeLineList) {
                TreeNodeLineVO treeNodeLineInfo = new TreeNodeLineVO();
                treeNodeLineInfo.setNodeIdFrom(nodeLine.getNodeIdFrom());
                treeNodeLineInfo.setNodeIdTo(nodeLine.getNodeIdTo());
                treeNodeLineInfo.setRuleLimitType(nodeLine.getRuleLimitType());
                treeNodeLineInfo.setRuleLimitValue(nodeLine.getRuleLimitValue());
                treeNodeLineInfoList.add(treeNodeLineInfo);
            }
        }
        // 将此节点的信息，尤其是此节点的两条连接子节点的线都保存进去
        TreeNodeVO treeNodeInfo = new TreeNodeVO();
        treeNodeInfo.setTreeId(treeNode.getTreeId());
        treeNodeInfo.setTreeNodeId(treeNode.getId());
        treeNodeInfo.setNodeType(treeNode.getNodeType());
        treeNodeInfo.setNodeValue(treeNode.getNodeValue());
        treeNodeInfo.setRuleKey(treeNode.getRuleKey());
        treeNodeInfo.setRuleDesc(treeNode.getRuleDesc());
        treeNodeInfo.setTreeNodeLineInfoList(treeNodeLineInfoList);

        // 保存每个节点
        treeNodeMap.put(treeNode.getId(), treeNodeInfo);
    }

    TreeRuleRich treeRuleRich = new TreeRuleRich();
    treeRuleRich.setTreeRoot(treeRoot);
    treeRuleRich.setTreeNodeMap(treeNodeMap);

    return treeRuleRich;
}

需要理清的点

1. 思考决策树对应的数据库表设计，分出哪些表，增加哪些字段
2. **规则树的构建（难点）**
3. 规则的对比，如何将代码和数据库对应上——通过![](https://cdn.nlark.com/yuque/0/2023/png/300264/1673274478765-c671fbb8-1743-419f-87cf-ddd31556fb49.png)
4. 弄清楚对于单个节点、单个节点的两条子树的连线、整体的结果 是怎么做到的

---

# 十四、防腐层：门面接口封装和对象转换

到此为止，基本的接口已经开发完成，但都是一些独立的接口，只是上一层浅封装

现在使用门面接口封装，_**对外提供接口服务**_

- _**DTO层的作用更多是防污操作，如有些数据不想要，有些数据不想暴露，就需要进行一层转换操作**_
- **尽量把系统提供者的模型转换为系统使用者的模型（而不引入中间第三者模型）**

**上一节说的 规则过滤，要单独抽象出一个对外的接口，即先请求量化接口获的用户应该参与的活动，然后再拿着活动ID去抽奖**

@Controller
public class LotteryActivityBooth implements ILotteryActivityBooth {

    private Logger logger = LoggerFactory.getLogger(LotteryActivityBooth.class);

    @Resource
    private IActivityProcess activityProcess;

    @Resource
    private IMapping<DrawAwardVO, AwardDTO> awardMapping;

    // 和下面相比少了过滤人群量化
    @Override
    public DrawRes doDraw(DrawReq drawReq) {
        try {

            logger.info("抽奖，开始 uId：{} activityId：{}", drawReq.getuId(), drawReq.getActivityId());

            // 1. 执行抽奖
            DrawProcessResult drawProcessResult = activityProcess.doDrawProcess(new DrawProcessReq(drawReq.getuId(), drawReq.getActivityId()));
            if (!Constants.ResponseCode.SUCCESS.getCode().equals(drawProcessResult.getCode())) {
                logger.error("抽奖，失败(抽奖过程异常) uId：{} activityId：{}", drawReq.getuId(), drawReq.getActivityId());
                return new DrawRes(drawProcessResult.getCode(), drawProcessResult.getInfo());
            }

            // 2. 数据转换
            DrawAwardVO drawAwardVO = drawProcessResult.getDrawAwardVO();
            AwardDTO awardDTO = awardMapping.sourceToTarget(drawAwardVO);
            awardDTO.setActivityId(drawReq.getActivityId());

            // 3. 封装数据
            DrawRes drawRes = new DrawRes(Constants.ResponseCode.SUCCESS.getCode(), Constants.ResponseCode.SUCCESS.getInfo());
            drawRes.setAwardDTO(awardDTO);

            logger.info("抽奖，完成 uId：{} activityId：{} drawRes：{}", drawReq.getuId(), drawReq.getActivityId(), JSON.toJSONString(drawRes));

            return drawRes;
        } catch (Exception e) {
            logger.error("抽奖，失败 uId：{} activityId：{} reqJson：{}", drawReq.getuId(), drawReq.getActivityId(), JSON.toJSONString(drawReq), e);
            return new DrawRes(Constants.ResponseCode.UN_ERROR.getCode(), Constants.ResponseCode.UN_ERROR.getInfo());
        }
    }

    @Override
    public DrawRes doQuantificationDraw(QuantificationDrawReq quantificationDrawReq) {
        try {
            logger.info("量化人群抽奖，开始 uId：{} treeId：{}", quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId());

            // 1. 执行规则引擎，获取用户可以参与的活动号
            RuleQuantificationCrowdResult ruleQuantificationCrowdResult = activityProcess.doRuleQuantificationCrowd(new DecisionMatterReq(quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId(), quantificationDrawReq.getValMap()));
            if (!Constants.ResponseCode.SUCCESS.getCode().equals(ruleQuantificationCrowdResult.getCode())) {
                logger.error("量化人群抽奖，失败(规则引擎执行异常) uId：{} treeId：{}", quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId());
                return new DrawRes(ruleQuantificationCrowdResult.getCode(), ruleQuantificationCrowdResult.getInfo());
            }

            // 2. 执行抽奖
            Long activityId = ruleQuantificationCrowdResult.getActivityId();
            DrawProcessResult drawProcessResult = activityProcess.doDrawProcess(new DrawProcessReq(quantificationDrawReq.getuId(), activityId));
            if (!Constants.ResponseCode.SUCCESS.getCode().equals(drawProcessResult.getCode())) {
                logger.error("量化人群抽奖，失败(抽奖过程异常) uId：{} treeId：{}", quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId());
                return new DrawRes(drawProcessResult.getCode(), drawProcessResult.getInfo());
            }

            // 3. 数据转换
            DrawAwardVO drawAwardVO = drawProcessResult.getDrawAwardVO();
            AwardDTO awardDTO = awardMapping.sourceToTarget(drawAwardVO);
            awardDTO.setActivityId(activityId);

            // 4. 封装数据
            DrawRes drawRes = new DrawRes(Constants.ResponseCode.SUCCESS.getCode(), Constants.ResponseCode.SUCCESS.getInfo());
            drawRes.setAwardDTO(awardDTO);

            logger.info("量化人群抽奖，完成 uId：{} treeId：{} drawRes：{}", quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId(), JSON.toJSONString(drawRes));

            return drawRes;
        } catch (Exception e) {
            logger.error("量化人群抽奖，失败 uId：{} treeId：{} reqJson：{}", quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId(), JSON.toJSONString(quantificationDrawReq), e);
            return new DrawRes(Constants.ResponseCode.UN_ERROR.getCode(), Constants.ResponseCode.UN_ERROR.getInfo());
        }
    }

}

---

# 十五、搭建MQ + kafka 环境

[https://www.cnblogs.com/sujing/p/10960832.html](https://www.cnblogs.com/sujing/p/10960832.html)

**消息队列作用：**

- **解耦**
- **异步**
- **削峰**

**为什么选择Kafka？**

- **零拷贝技术**

- **RabbitMQ也是支持零拷贝，但是它是基于**`**mmap**`**，因为它有延迟功能，所以必须内存要能操作到**
- **Kafka零拷贝是基于**`**sendfile**`**， 即传递文件描述符，而不是传递文件本身，这个IO是可以忽略不记得**

- `**sendfile**`**机制类似百度网盘分享的连接，但是文件本身还在百度网盘存着**
- **RabbitMQ不能这样，因为****死信机制****，如果文件没有在内存中，那么操作起来很不方便**
- **Kafka在通过回调函数解决这个问题**

@Component
public class KafkaProducer {

    private Logger logger = LoggerFactory.getLogger(KafkaProducer.class);

    @Resource
    private KafkaTemplate<String, Object> kafkaTemplate;

    public static final String TOPIC_TEST = "Hello-Kafka";

    public static final String TOPIC_GROUP = "test-consumer-group";

    public void send(Object obj) {
        String obj2String = JSON.toJSONString(obj);
        logger.info("准备发送消息为：{}", obj2String);

        // 发送消息
        ListenableFuture<SendResult<String, Object>> future = kafkaTemplate.send(TOPIC_TEST, obj);
        
        future.addCallback(new ListenableFutureCallback<SendResult<String, Object>>() {
            @Override
            public void onFailure(Throwable throwable) {
                //发送失败的处理
                logger.info(TOPIC_TEST + " - 生产者 发送消息失败：" + throwable.getMessage());
            }

            @Override
            public void onSuccess(SendResult<String, Object> stringObjectSendResult) {
                //成功的处理
                logger.info(TOPIC_TEST + " - 生产者 发送消息成功：" + stringObjectSendResult.toString());
            }
        });
    }

}

@Component
public class KafkaConsumer {

    private Logger logger = LoggerFactory.getLogger(KafkaConsumer.class);

    @KafkaListener(topics = KafkaProducer.TOPIC_TEST, groupId = KafkaProducer.TOPIC_GROUP)
    public void topicTest(ConsumerRecord<?, ?> record, Acknowledgment ack, @Header(KafkaHeaders.RECEIVED_TOPIC) String topic) {
        Optional<?> message = Optional.ofNullable(record.value());
        if (message.isPresent()) {
            Object msg = message.get();
            logger.info("topic_test 消费了： Topic:" + topic + ",Message:" + msg);
            ack.acknowledge();	// 消费成功 返回ACK（默认1）
        }
    }

}

@Resource
private KafkaProducer kafkaProducer;

@Test
public void test_send() throws InterruptedException {

    InvoiceVO invoice = new InvoiceVO();
	invoice.setuId("fustack");

    kafkaProducer.sendLotteryInvoice(invoice);

    while (true){
        Thread.sleep(10000);
    }
}

---

# 十六、使用MQ解耦抽奖、发货流程

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677139035056-6d4be89c-cf91-4864-9f45-d5548b2638cb.png)

- _**主要是：不用在抽完奖就更新发奖状态**_

## 1. 流程说明

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677140263006-ece28cd0-eb05-4a27-9eb9-eeccf6e6689d.png)

- 从用户发起抽奖到中奖后开始，就是MQ处理发奖的流程
- 因为 MQ 消息的发送是不具备事务性的，也就是说你在发送 MQ 时可能会失败，哪怕成功率是3个9到4个9，也会有一定的概率触发发送失败。所以我们**在 MQ 发送完成后需要知道是否发送成功，进行库表状态更新（**onFailure设置状态为2）**，如果发送失败则需要使用 worker 来补偿 MQ 发送**。（使用`xxl-job`
- 最后 MQ 发送完成到消费，也是可能有失败的，比如处理失败、更新库表失败等，但无论是什么失败都需要保证 MQ 进行重试处理。而保证 MQ 消息重试的前提就是服务的幂等性，否则你在重试的过程中就造成了流程异常，比如更新次数多了、数据库插入多了、给用户发奖多了等等，尤其是发生资损是更可怕的。

## 2. MQ服务

关于 MQ 的使用，无论是 Kafka 还是 RocketMQ，基本方式都是类似的，一个生产消息，一个监听消息，只不过在一些符合各自业务场景下，做了细分的优化，但这并不会影响你的使用。

### 2.1 生产消息

- 我们会把所有的生产消息都放到 KafkaProducer 中，并对外提供一个可以发送 MQ 消息的方法。
- 因为我们配置的类型转换为 StringDeserializer 所以发送消息的方式是 JSON 字符串，当然这个编解码器是可以重写的，满足你发送其他类型的数据。

@Component
public class KafkaProducer {

    private Logger logger = LoggerFactory.getLogger(KafkaProducer.class);

    @Resource
    private KafkaTemplate<String, Object> kafkaTemplate;

    /**
     * MQ主题：中奖发货单
     */
    public static final String TOPIC_INVOICE = "lottery_invoice";

    /**
     * 发送中奖物品发货单消息
     *
     * @param invoice 发货单
     */
    public ListenableFuture<SendResult<String, Object>> sendLotteryInvoice(InvoiceVO invoice) {
        String objJson = JSON.toJSONString(invoice);
        logger.info("发送MQ消息 topic：{} bizId：{} message：{}", TOPIC_INVOICE, invoice.getuId(), objJson);
        return kafkaTemplate.send(TOPIC_INVOICE, objJson);
    }

}

### 2.2 消费消息

- 每一个 MQ 消息的消费都会有一个对应的 XxxListener 来处理消息体，如果你使用一些其他的 MQ 可能还会看到一些抽象类来处理 MQ 消息集合。
- 在这个 LotteryInvoiceListener 消息监听类中，主要就是_**通过消息中的发奖类型获取到对应的奖品发货工厂，处理奖品的发送操作**_。
- 在奖品发送操作中，已经补全了 DistributionBase#updateUserAwardState 更新奖品发送状态的操作。

@Component
public class LotteryInvoiceListener {

    private Logger logger = LoggerFactory.getLogger(LotteryInvoiceListener.class);

    @Resource
    private DistributionGoodsFactory distributionGoodsFactory;

    @KafkaListener(topics = "lottery_invoice", groupId = "lottery")
    public void onMessage(ConsumerRecord<?, ?> record, Acknowledgment ack, @Header(KafkaHeaders.RECEIVED_TOPIC) String topic) {
        Optional<?> message = Optional.ofNullable(record.value());

        // 1. 判断消息是否存在
        if (!message.isPresent()) {
            return;
        }

        // 2. 处理 MQ 消息
        try {
            // 1. 转化对象（或者你也可以重写Serializer<T>）
            InvoiceVO invoiceVO = JSON.parseObject((String) message.get(), InvoiceVO.class);

            // 2. 获取发送奖品工厂，执行发奖
            IDistributionGoods distributionGoodsService = distributionGoodsFactory.getDistributionGoodsService(invoiceVO.getAwardType());
            DistributionRes distributionRes = distributionGoodsService.doDistribution(new GoodsReq(invoiceVO.getuId(), invoiceVO.getOrderId(), invoiceVO.getAwardId(), invoiceVO.getAwardName(), invoiceVO.getAwardContent()));

            Assert.isTrue(Constants.AwardState.SUCCESS.getCode().equals(distributionRes.getCode()), distributionRes.getInfo());

            // 3. 打印日志
            logger.info("消费MQ消息，完成 topic：{} bizId：{} 发奖结果：{}", topic, invoiceVO.getuId(), JSON.toJSONString(distributionRes));

            // 4. 消息消费完成
            ack.acknowledge();
        } catch (Exception e) {
            // 发奖环节失败，消息重试。所有到环节，发货、更新库，都需要保证幂等。
            logger.error("消费MQ消息，失败 topic：{} message：{}", topic, message.get());
            throw e;
        }
    }

}

因为奖品发送不需要实时

## 3. 抽奖流程解耦

- 将MQ添加到抽奖流程中
- 在 ActivityProcessImpl#doDrawProcess 方法中，主要补全的就是关于 MQ 的处理，这里我们会调动 kafkaProducer.sendLotteryInvoice 发送一个中奖结果的发货单。

- **发送 MQ 消息，第一步领取活动就扣取了Redis的库存，在这一步进行MySQL数据同步，所以后面有可能Redis不成功，但是MySQL已经落库了 ⭐️⭐️⭐️**

- 消息发送完毕后进行回调处理，更新数据库中 MQ 发送的状态，如果有 MQ 发送失败则更新数据库 mq_state = 2 **这里还有可能在更新库表状态的时候失败**，但没关系这些都会被 worker 补偿处理掉，一种是发送 MQ 失败，另外一种是 MQ 状态为 0 但很久都没有发送 MQ 那么也可以触发发送。

查询**发送MQ失败（2）**和**超时30分钟（0）**，未发送MQ的数据:

// 消息发送状态（0未发送、1发送成功、2发送失败）

// 表user_strategy_export：**WHERE mq_state = 2 OR ( mq_state = 0 AND now() - create_time > 1800000 )**

- 现在从用户领取活动、执行抽奖、结果落库，到 发送MQ处理后续发奖的流程就解耦了，**因为用户只需要知道自己中奖了，但发奖到货是可以等待的**，毕竟发送虚拟商品的等待时间并不会很长，而实物商品走物流就更可以接收了。所以对于这样的流程进行解耦是非常有必要的，否则_你的程序逻辑会让用户在界面等待更久的时间_。

public DrawProcessResult doDrawProcess(DrawProcessReq req) {
    // 1. 领取活动
    
    // 2. 执行抽奖

    // 3. 结果落库

    // 4. 发送MQ，触发发奖流程
    InvoiceVO invoiceVO = buildInvoiceVO(drawOrderVO);
    ListenableFuture<SendResult<String, Object>> future = kafkaProducer.sendLotteryInvoice(invoiceVO);
    future.addCallback(new ListenableFutureCallback<SendResult<String, Object>>() {
        @Override
        public void onSuccess(SendResult<String, Object> stringObjectSendResult) {
            // 4.1 MQ 消息发送完成，更新数据库表 user_strategy_export.mq_state = 1
                //  在这里是更新状态，而插入这一条是第三步结果落库中
            activityPartake.updateInvoiceMqState(invoiceVO.getuId(), invoiceVO.getOrderId(), Constants.MQState.COMPLETE.getCode());
        }
        @Override
        public void onFailure(Throwable throwable) {
            // 4.2 MQ 消息发送失败，更新数据库表 user_strategy_export.mq_state = 2 【等待定时任务扫码补偿MQ消息】
            activityPartake.updateInvoiceMqState(invoiceVO.getuId(), invoiceVO.getOrderId(), Constants.MQState.FAIL.getCode());
        }
    });

    // 5. 返回结果
    return new DrawProcessResult(Constants.ResponseCode.SUCCESS.getCode(), Constants.ResponseCode.SUCCESS.getInfo(), drawAwardVO);
}

## 4. 流程图

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677139616225-69182c97-2acd-485a-96de-1e1fd0b82118.png)

---

# 十七、引入`xxl-job`处理活动状态扫描

XXL-JOB是一个分布式任务调度平台

xxl-job用来充当worker，扫描抽奖活动状态，把审核通过的活动扫描为活动中，把已过期活动中的状态扫描为关闭。后续章节我们还会使用分布式任务调度系统解决其他场景问题。

## 1. xxl-job 基础配置

1. 引入pom
2. 配置yml

# xxl-job
# 官网：https://github.com/xuxueli/xxl-job/
# 地址：http://localhost:7397/xxl-job-admin 【需要先启动 xxl-job】
# 账号：admin
# 密码：123456
xxl:
  job:
    admin:
      addresses: http://127.0.0.1:7397/xxl-job-admin
    executor:
      address:
      appname: lottery-job
      ip:
      port: 9999
      logpath: /Users/fuzhengwei/itstack/data/applogs/xxl-job/jobhandler
      logretentiondays: 50
    accessToken:

## 2. 初始化任务执行器

这里需要启动一个任务执行器，通过配置 @Bean 对象的方式交给 Spring 进行管理。

/**
 * @description: XXL-JOB 配置
 * @author: 小傅哥，微信：fustack
 * @date: 2021/11/6
 * @github: https://github.com/fuzhengwei
 * @Copyright: 公众号：bugstack虫洞栈 | 博客：https://bugstack.cn - 沉淀、分享、成长，让自己和他人都能有所收获！
 */
@Configuration
public class LotteryXxlJobConfig {

    private Logger logger = LoggerFactory.getLogger(LotteryXxlJobConfig.class);

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.accessToken}")
    private String accessToken;

    @Value("${xxl.job.executor.appname}")
    private String appname;

    @Value("${xxl.job.executor.address}")
    private String address;

    @Value("${xxl.job.executor.ip}")
    private String ip;

    @Value("${xxl.job.executor.port}")
    private int port;

    @Value("${xxl.job.executor.logpath}")
    private String logPath;

    @Value("${xxl.job.executor.logretentiondays}")
    private int logRetentionDays;

    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        logger.info(">>>>>>>>>>> xxl-job config init.");

        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);

        return xxlJobSpringExecutor;
    }

    /**********************************************************************************************
     * 针对多网卡、容器内部署等情况，可借助 "spring-cloud-commons" 提供的 "InetUtils" 组件灵活定制注册IP；
     *
     *      1、引入依赖：
     *          <dependency>
     *             <groupId>org.springframework.cloud</groupId>
     *             <artifactId>spring-cloud-commons</artifactId>
     *             <version>${version}</version>
     *         </dependency>
     *
     *      2、配置文件，或者容器启动变量
     *          spring.cloud.inetutils.preferred-networks: 'xxx.xxx.xxx.'
     *
     *      3、获取IP
     *          String ip_ = inetUtils.findFirstNonLoopbackHostInfo().getIpAddress();
     **********************************************************************************************/

}

## 3. 开发活动扫描任务

**在任务扫描中，主要把已经审核通过的活动和已过期的活动中状态进行变更操作；**

- **审核通过 -> 扫描为活动中**
- **活动中已过期时间 -> 扫描为活动关闭**

/**
 * @description: 抽奖业务，任务配置
 */
@Component
public class LotteryXxlJob {

    private Logger logger = LoggerFactory.getLogger(LotteryXxlJob.class);

    @Resource
    private IActivityDeploy activityDeploy;

    @Resource
    private IStateHandler stateHandler;

    @XxlJob("lotteryActivityStateJobHandler")
    public void lotteryActivityStateJobHandler() throws Exception {
        logger.info("扫描活动状态 Begin");

        List<ActivityVO> activityVOList = activityDeploy.scanToDoActivityList(0L);
        if (activityVOList.isEmpty()){
            logger.info("扫描活动状态 End 暂无符合需要扫描的活动列表");
            return;
        }

        while (!activityVOList.isEmpty()) {
            for (ActivityVO activityVO : activityVOList) {
                Integer state = activityVO.getState();
                switch (state) {
                    // 活动状态为审核通过，在临近活动开启时间前，审核活动为活动中。在使用活动的时候，需要依照活动状态核时间两个字段进行判断和使用。
                    case 4:
                        Result state4Result = stateHandler.doing(activityVO.getActivityId(), Constants.ActivityState.PASS);
                        logger.info("扫描活动状态为活动中 结果：{} activityId：{} activityName：{} creator：{}", JSON.toJSONString(state4Result), activityVO.getActivityId(), activityVO.getActivityName(), activityVO.getCreator());
                        break;
                    // 扫描时间已过期的活动，从活动中状态变更为关闭状态【这里也可以细化为2个任务来处理，也可以把时间判断放到数据库中操作】
                    case 5:
                        // 看是否过期了
                        if (activityVO.getEndDateTime().before(new Date())){
                            Result state5Result = stateHandler.close(activityVO.getActivityId(), Constants.ActivityState.DOING);
                            logger.info("扫描活动状态为关闭 结果：{} activityId：{} activityName：{} creator：{}", JSON.toJSONString(state5Result), activityVO.getActivityId(), activityVO.getActivityName(), activityVO.getCreator());
                        }
                        break;
                    default:
                        break;
                }
            }

            // 获取集合中最后一条记录，继续扫描后面10条记录
            ActivityVO activityVO = activityVOList.get(activityVOList.size() - 1);
            activityVOList = activityDeploy.scanToDoActivityList(activityVO.getId());
        }

        logger.info("扫描活动状态 End");

    }

}

## 4. 在xxl-job管理界面配置任务调度执行器

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677144599180-f4d28131-fc2d-41a0-a0dd-4e2387a827df.png)

## 5. 配置任务

这里我们把已经开发了的任务 `LotteryXxlJob#lotteryActivityStateJobHandler` 配置到任务调度中心，如下：

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677144622468-9736ed2f-0c4d-484c-ac7a-43ad3a931201.png)

---

# 十八、扫描库表补偿发货单MQ消息

分布式任务调度，扫描抽奖发货单消息状态，对于未发送MQ或者发送失败的MQ，进行补偿发送处理

在开始参与活动时，先看一下`user_take_activity`有没有此用户参与的还没用的（状态为0）

## 1. 任务流程

- 我们的任务流程，完成的就是整个抽奖活动中，关于中奖结果落库后，进行MQ后。出现问题时，进行补偿消息发送处理的部分。
- 在MQ消息补偿的过程中，会把发送失败的消息和迟迟没有发送的消息，都进行补偿，已保障全流程的可靠性。

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677154622285-c6010daa-ae64-413e-b215-a9560b07de4e.png)

## 2. 功能实现

### 2.1 路由组件提供必要方法

因为要扫描所有的库和库表，所以要提供获得分库、分表的数量 用来for循环

- 新增加方法：setDBKey、setTBKey、dbCount、tbCount

@Override
public void setDBKey(int dbIdx) {
    DBContextHolder.setDBKey(String.format("%02d", dbIdx));
}

@Override
public void setTBKey(int tbIdx) {
    DBContextHolder.setTBKey(String.format("%03d", tbIdx));
}

@Override
public int dbCount() {
    return dbRouterConfig.getDbCount();
}

@Override
public int tbCount() {
    return dbRouterConfig.getTbCount();
}

@Override
public void clear(){
    DBContextHolder.clearDBKey();
    DBContextHolder.clearTBKey();
}

### 2.2 消息补偿任务

@XxlJob("lotteryOrderMQStateJobHandler")
public void lotteryOrderMQStateJobHandler() throws Exception {
    // 验证参数
    String jobParam = XxlJobHelper.getJobParam();
    if (null == jobParam) {
        logger.info("扫描用户抽奖奖品发放MQ状态[Table = 2*4] 错误 params is null");
        return;
    }

    // 获取分布式任务配置参数信息 参数配置格式：1,2,3 也可以是指定扫描一个，也可以配置多个库，按照部署的任务集群进行数量配置，均摊分别扫描效率更高
    String[] params = jobParam.split(",");
    logger.info("扫描用户抽奖奖品发放MQ状态[Table = 2*4] 开始 params：{}", JSON.toJSONString(params));
    if (params.length == 0) {
        logger.info("扫描用户抽奖奖品发放MQ状态[Table = 2*4] 结束 params is null");
        return;
    }

    // 获取分库分表配置下的分表数
    int tbCount = dbRouter.tbCount();
    // 循环获取指定扫描库
    for (String param : params) {
        // 获取当前任务扫描的指定分库
        int dbCount = Integer.parseInt(param);
        // 判断配置指定扫描库数，是否存在
        if (dbCount > dbRouter.dbCount()) {
            logger.info("扫描用户抽奖奖品发放MQ状态[Table = 2*4] 结束 dbCount not exist");
            continue;
        }

        // 循环扫描对应表
        for (int i = 0; i < tbCount; i++) {
            // 扫描库表数据 ⭐️⭐️⭐️⭐️⭐️
            List<InvoiceVO> invoiceVOList = activityPartake.scanInvoiceMqState(dbCount, i);
            logger.info("扫描用户抽奖奖品发放MQ状态[Table = 2*4] 扫描库：{} 扫描表：{} 扫描数：{}", dbCount, i, invoiceVOList.size());
            // 补偿 MQ 消息
            for (InvoiceVO invoiceVO : invoiceVOList) {
                ListenableFuture<SendResult<String, Object>> future = kafkaProducer.sendLotteryInvoice(invoiceVO);
                future.addCallback(new ListenableFutureCallback<SendResult<String, Object>>() {
                    @Override
                    public void onSuccess(SendResult<String, Object> stringObjectSendResult) {
                        // MQ 消息发送完成，更新数据库表 user_strategy_export.mq_state = 1
                        activityPartake.updateInvoiceMqState(invoiceVO.getuId(), invoiceVO.getOrderId(), Constants.MQState.
                    }
                    @Override
                    public void onFailure(Throwable throwable) {
                        // MQ 消息发送失败，更新数据库表 user_strategy_export.mq_state = 2 【等待定时任务扫码补偿MQ消息】
                        activityPartake.updateInvoiceMqState(invoiceVO.getuId(), invoiceVO.getOrderId(), Constants.MQState.
                    }
                });
            }
        }

    }
    logger.info("扫描用户抽奖奖品发放MQ状态[Table = 2*4] 完成 param：{}", JSON.toJSONString(params));
}

上面五⭐️处，是扫描发货单 MQ 状态，把未发送 MQ 的单子扫描出来，做补偿

@Override
public List<InvoiceVO> scanInvoiceMqState(int dbCount, int tbCount) {
    try {
        // 设置路由
        dbRouter.setDBKey(dbCount);
        dbRouter.setTBKey(tbCount);
        // 查询数据
        return userTakeActivityRepository.scanInvoiceMqState();
    } finally {
        dbRouter.clear();
    }
}

下图是调用的mapper SQL，查询state=2或者超时的

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677155235238-43dcdbaa-2b3e-4333-b1b0-64d4d19ffd8c.png)

## 3. Xxl-job任务配置

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677155308091-e023649c-7d66-4f04-b94e-457cd1d17424.png)

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677155418597-c2809fcd-d74b-46db-bb9f-505224e609ac.png)

---

# 十九、设计滑动库存分布式锁处理活动秒杀：防止超卖

用的什么Redis：

Spring Data Redis，底层是封装了 jedis

- 属于秒杀场景，目的是防止超卖
- 将原来加载活动号上的锁（粗粒度）如 100001、100002，改加到活动库存（细粒度）：以滑动库存剩余编号的方式进行加锁，例如 100001_1、100001_2、100001_3
- 增加缓存扣减库存后，发送 MQ 消息进行异步更新数据库中活动库存，做最终数据一致性处理。_这一部分如果你的系统并发体量较大，还需要把 MQ 的数据不要直接对库更新，而是更新到缓存中，再由任务最阶段同步，以此减少对数据库表的操作_

## 1. 扣减流程

活动领取完成后，其实**这个时候只是把缓存的库存扣掉了，****但数据库中的库存并没有扣减（Redis扣除前，就已经更新MySQL库存了）****，所以我们需要发送一个 MQ 消息，这个消息是执行发奖，****发奖成功的回调 更新数据库表 user_strategy_export.mq_state = 1**。因为 MQ 可以消峰因此在降低 MQ 分片的情况下，消费效率有所下降，并不会对数据库造成压力，保证最终数据一致性即可。_但也有例外，所以我们提到可以使用定时任务来更新数据库库存_

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677231941805-f4b90f02-59fb-452e-81ad-3d58f2b71af7.png)

## 2. 功能开发

### 2.1 滑动库存锁设计

原来如果只加到活动号上，则粒度太大，比如库存还有100，用户A先获得了这个活动，那么之后的用户都不能再获得了，直到A释放

现在改加到获得库存上，即对于某个具体的库存加一把锁：activity_100、activity_99 这样粒度就更小了。**「活动ID+库存扣减后的值」作为加锁的key**

## 滑动库存初始化

在管理员创建完活动后，进行Redis的初始化

@Override
public void addActivity(ActivityVO activity) {
    Activity req = new Activity();
    BeanUtils.copyProperties(activity, req);
    activityDao.insert(req);
    
    // 设置活动库存 KEY
    redisUtil.set(Constants.RedisKey.KEY_LOTTERY_ACTIVITY_STOCK_COUNT(activity.getActivityId()), 0);
}

// KEY_LOTTERY_ACTIVITY_STOCK_COUNT(id,0) 会调用下面这个方法：
public static String KEY_LOTTERY_ACTIVITY_STOCK_COUNT(Long activityId) {
    return LOTTERY_ACTIVITY_STOCK_COUNT + activityId;
}

// 然后调将此 key设为0  如：LOTTERY_ACTIVITY_STOCK_COUNT_10001
redisUtil.set(LOTTERY_ACTIVITY_STOCK_COUNT_10001, 0);

## 2.2 库存扣减操作

注意生成的是两个key：

1. 一个是每次来调用获得 先获得当前的库存：redis.get(`**KEY_LOTTERY_ACTIVITY_STOCK_COUNT****_10001**`, **40**) (40是已经消耗的）
2. 然后对比MySQL内的库存，如果小于则继续，否则库存不足
3. 二是生成 已用的库存Key `**KEY_LOTTERY_ACTIVITY_STOCK_COUNT_****TOKEN_41**`
4. 然后**对这个token加上分布式锁**，注意有时间 350L（毫秒）
5. 最后（抽奖彻底执行完毕），将这个 `**KEY_LOTTERY_ACTIVITY_STOCK_COUNT_TOKEN_****41**` **恢复，但是** `**KEY_LOTTERY_ACTIVITY_STOCK_COUNT_10001**` **不恢复**

`**stockUsedCount**`**代表的是已消耗的库存，而不是剩余的**

- 代码中的注释就是整个操作流程，创建 Key、占用库存、判断库存、**以占用库存的编号做为加锁key**、调用 setNx 加一个分布式锁，最终完成整个秒杀扣减库存的动作。
- 这里还有一些其他的实现方式，例如：lua脚本、zk、jvm层，**mysql（实现分布式锁**：阿里搜索引擎面试）都可以处理，但经过验证 lua 脚本会有一定的耗时，并发较高时会有问题。
- 秒杀的设计原则是保证不超卖，但不一定保证100个库存完全消耗掉，因为可能会有一些锁失败的情况，如用户A获得了`100001_90`用户B获得了`100001_91`，如果用户A释放锁失败没有把90再减回去，那么90这个库存就不能再用了

@Override
public StockResult subtractionActivityStockByRedis(String uId, Long activityId, Integer stockCount) {
    // 需要传入MySQL的实际库存数，优化成在Redis内存储的一个常量
    
    //  1. 获取抽奖活动库存 Key
    String stockKey = Constants.RedisKey.KEY_LOTTERY_ACTIVITY_STOCK_COUNT(activityId);

    // 2. 扣减库存，目前占用库存数
    // 采用的是增加式，代表「已消耗库存」 stockCount 为止
    Integer stockUsedCount = (int) redisUtil.incr(stockKey, 1);

    // 3. 超出库存判断，进行恢复原始库存
    // 这个恢复库存是恢复的库存，而不是释放锁
    if (stockUsedCount > stockCount) {
        redisUtil.decr(stockKey, 1);
        return new StockResult(Constants.ResponseCode.OUT_OF_STOCK.getCode(), Constants.ResponseCode.OUT_OF_STOCK.getInfo());
    }

    // 4. 以活动库存占用编号，生成对应加锁Key，细化锁的颗粒度
    String stockTokenKey = Constants.RedisKey.KEY_LOTTERY_ACTIVITY_STOCK_COUNT_TOKEN(activityId, stockUsedCount);

    // 5. 使用 Redis.setNx 加一个分布式锁
    boolean lockToken = redisUtil.setNx(stockTokenKey, 350L);
    if (!lockToken) {
        logger.info("抽奖活动{}用户秒杀{}扣减库存，分布式锁失败：{}", activityId, uId, stockTokenKey);
        return new StockResult(Constants.ResponseCode.ERR_TOKEN.getCode(), Constants.ResponseCode.ERR_TOKEN.getInfo());
    }

    return new StockResult(Constants.ResponseCode.SUCCESS.getCode(), Constants.ResponseCode.SUCCESS.getInfo(), stockTokenKey, stockCount - stockUsedCount);
}

## 2.2 并发锁删除处理

删除滑动库存锁分两种情况：

1. 一种是落库没成功，那么需要把库存再还回去

1. 注意是非原子的，没必要原子性，保证不超卖即可

2. 第二种就是落库成功了，就不能还库存了，只释放掉滑动库存锁即可（不释放对数据没影响，影响是性能）

两种情况都需要释放掉滑动库存锁

@Override
public void recoverActivityCacheStockByRedis(Long activityId, String tokenKey, String code) {
    if (!Constants.ResponseCode.SUCCESS.getCode().equals(code)){
        // 1.获取抽奖活动库存 Key
        String stockkey = Constants.RedisKey.KEY_LOTTERY_ACTIVITY_STOCK_COUNT(activityId);
        //2. 回滾库存操作，此操作非原子性的
        redisUtil.decr(stockkey,1);
        return;
    }

    // 删除分布式锁 Key
    redisUtil.del(tokenKey);
}

## 2.3 活动秒杀流程处理

`BaseActivityPartake#doPartake`

扣减库存后，在各个以下的流程节点中，如果有流程失败则进行缓存库存的恢复操作。

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677233188891-3ae4d296-5e1c-4051-8348-50b1992ff97e.png)

## 2.4 发送MQ消息，处理数据一致性 ⭐️⭐️⭐️

由于我们使用 Redis 代替数据库库存，那么在缓存的库存处理后，还需要把数据库中的库存处理为和缓存一致，这样在后续运营这部分数据时才能保证一定的运营可靠性。

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677233875893-2c9008ca-2de6-47fa-94ed-6cb7b4aba763.png)

## 2.5 活动领取记录 MQ 消费

- 消费 MQ 消息的流程就比较简单了，接收到 MQ 进行更新数据库处理即可。不过我们这里**更新数据库并不是直接对数据库进行库存扣减操作，而是把从缓存拿到的库存最新镜像更新到数据库中**。它的 SQL = `UPDATE activity SET stock_surplus_count = #{stockSurplusCount} WHERE activity_id = #{activityId} AND stock_surplus_count > #{stockSurplusCount}`
- 更新数据库库存【实际场景业务体量较大，可能也会由于MQ消费引起并发，对数据库产生压力，所以如果并发量较大，可以把库存记录缓存中，并使用定时任务进行处理缓存和数据库库存同步，减少对数据库的操作次数】

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1677234214833-09f7e3af-e0a1-48a1-b2f3-42e874a29495.png)

---

# 个人优化

面试的时候，要说一部分是看着写的，一部分是自己想的优化的：

- 最开始数据都存在MySQL，优化后存入到Redis

- 即一开始库存是直接扣的MySQL，后续优化成Redis
- 然后就解决缓存击穿、传统、雪崩

- 加了些什么？校验器，最开始只有性别，加了个年龄

---

# 末、完整的流程

![](https://cdn.nlark.com/yuque/0/2023/png/300264/1678771557494-20dcaf13-2808-45e2-a5c0-63364a91ac10.png)

@Override
public DrawProcessResult doDrawProcess(DrawProcessReq req) {
    // 1. 领取活动
    PartakeResult partakeResult = activityPartake.doPartake(new PartakeReq(req.getuId(), req.getActivityId()));
    if (!Constants.ResponseCode.SUCCESS.getCode().equals(partakeResult.getCode()) && !Constants.ResponseCode.NOT_CONSUMED_TAKE.getCode().equals(partakeResult.getCode())) {
        return new DrawProcessResult(partakeResult.getCode(), partakeResult.getInfo());
    }

    // 2. 首次成功领取活动，发送 MQ 消息
    if (Constants.ResponseCode.SUCCESS.getCode().equals(partakeResult.getCode())) {
        ActivityPartakeRecordVO activityPartakeRecord = new ActivityPartakeRecordVO();
        activityPartakeRecord.setuId(req.getuId());
        activityPartakeRecord.setActivityId(req.getActivityId());
        activityPartakeRecord.setStockCount(partakeResult.getStockCount());
        activityPartakeRecord.setStockSurplusCount(partakeResult.getStockSurplusCount());
        // 发送 MQ 消息
        kafkaProducer.sendLotteryActivityPartakeRecord(activityPartakeRecord);
    }

    Long strategyId = partakeResult.getStrategyId();
    Long takeId = partakeResult.getTakeId();

    // 3. 执行抽奖
    DrawResult drawResult = drawExec.doDrawExec(new DrawReq(req.getuId(), strategyId));
    if (Constants.DrawState.FAIL.getCode().equals(drawResult.getDrawState())) {
        return new DrawProcessResult(Constants.ResponseCode.LOSING_DRAW.getCode(), Constants.ResponseCode.LOSING_DRAW.getInfo());
    }
    DrawAwardVO drawAwardVO = drawResult.getDrawAwardInfo();

    // 4. 结果落库
    DrawOrderVO drawOrderVO = buildDrawOrderVO(req, strategyId, takeId, drawAwardVO);
    Result recordResult = activityPartake.recordDrawOrder(drawOrderVO);
    if (!Constants.ResponseCode.SUCCESS.getCode().equals(recordResult.getCode())) {
        return new DrawProcessResult(recordResult.getCode(), recordResult.getInfo());
    }

    // 5. 发送MQ，触发发奖流程
    InvoiceVO invoiceVO = buildInvoiceVO(drawOrderVO);
    ListenableFuture<SendResult<String, Object>> future = kafkaProducer.sendLotteryInvoice(invoiceVO);
    future.addCallback(new ListenableFutureCallback<SendResult<String, Object>>() {

        @Override
        public void onSuccess(SendResult<String, Object> stringObjectSendResult) {
            // 4.1 MQ 消息发送完成，更新数据库表 user_strategy_export.mq_state = 1
            activityPartake.updateInvoiceMqState(invoiceVO.getuId(), invoiceVO.getOrderId(), Constants.MQState.COMPLETE.getCode());
        }

        @Override
        public void onFailure(Throwable throwable) {
            // 4.2 MQ 消息发送失败，更新数据库表 user_strategy_export.mq_state = 2 【等待定时任务扫码补偿MQ消息】
            activityPartake.updateInvoiceMqState(invoiceVO.getuId(), invoiceVO.getOrderId(), Constants.MQState.FAIL.getCode());
        }

    });

    // 6. 返回结果
    return new DrawProcessResult(Constants.ResponseCode.SUCCESS.getCode(), Constants.ResponseCode.SUCCESS.getInfo(), drawAwardVO);
}

添加了量化的步骤之后：

@Override
public DrawRes doQuantificationDraw(QuantificationDrawReq quantificationDrawReq) {
    try {
        logger.info("量化人群抽奖，开始 uId：{} treeId：{}", quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId());

        // 1. 执行规则引擎，获取用户可以参与的活动号
        // 相比上面，就是多了这一步
        RuleQuantificationCrowdResult ruleQuantificationCrowdResult = activityProcess.doRuleQuantificationCrowd(new DecisionMatterReq(quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId(), quantificationDrawReq.getValMap()));
        if (!Constants.ResponseCode.SUCCESS.getCode().equals(ruleQuantificationCrowdResult.getCode())) {
            logger.error("量化人群抽奖，失败(规则引擎执行异常) uId：{} treeId：{}", quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId());
            return new DrawRes(ruleQuantificationCrowdResult.getCode(), ruleQuantificationCrowdResult.getInfo());
        }

        // 2. 执行抽奖
        Long activityId = ruleQuantificationCrowdResult.getActivityId();
        // 调用的是上面的doDrawProcess()
        DrawProcessResult drawProcessResult = activityProcess.doDrawProcess(new DrawProcessReq(quantificationDrawReq.getuId(), activityId));
        if (!Constants.ResponseCode.SUCCESS.getCode().equals(drawProcessResult.getCode())) {
            logger.error("量化人群抽奖，失败(抽奖过程异常) uId：{} treeId：{}", quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId());
            return new DrawRes(drawProcessResult.getCode(), drawProcessResult.getInfo());
        }

        // 3. 数据转换
        DrawAwardVO drawAwardVO = drawProcessResult.getDrawAwardVO();
        AwardDTO awardDTO = awardMapping.sourceToTarget(drawAwardVO);
        awardDTO.setActivityId(activityId);

        // 4. 封装数据
        DrawRes drawRes = new DrawRes(Constants.ResponseCode.SUCCESS.getCode(), Constants.ResponseCode.SUCCESS.getInfo());
        drawRes.setAwardDTO(awardDTO);

        logger.info("量化人群抽奖，完成 uId：{} treeId：{} drawRes：{}", quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId(), JSON.toJSONString(drawRes));

        return drawRes;
    } catch (Exception e) {
        logger.error("量化人群抽奖，失败 uId：{} treeId：{} reqJson：{}", quantificationDrawReq.getuId(), quantificationDrawReq.getTreeId(), JSON.toJSONString(quantificationDrawReq), e);
        return new DrawRes(Constants.ResponseCode.UN_ERROR.getCode(), Constants.ResponseCode.UN_ERROR.getInfo());
    }
}