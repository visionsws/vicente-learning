## 一、本地缓存介绍

缓存在日常开发中起到至关重要的作用，由于是存储在内存中，数据的读取速度是非常快的，能大量减少对数据库的访问，减少数据库的压力。

之前介绍过 Redis 这种 NoSql 作为缓存组件，它能够很好的作为分布式缓存组件提供多个服务间的缓存，但是 Redis 这种还是需要网络开销，增加时耗。本地缓存是直接从本地内存中读取，没有网络开销，例如秒杀系统或者数据量小的缓存等，比远程缓存更合适。

## 二、缓存组件 Caffeine 介绍

按 Caffeine Github 文档描述，Caffeine 是基于 JAVA 8 的高性能缓存库。并且在 spring5 (springboot 2.x) 后，spring 官方放弃了 Guava，而使用了性能更优秀的 Caffeine 作为默认缓存组件。

### 1、Caffeine 性能

可以通过下图观测到，在下面缓存组件中 Caffeine 性能是其中最好的。

![img](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbuc4ksRNP1YoibxDzgNRqfNYVCX4dCqicykaZbxoQEEQcGlMAH3PX6CBB9icxSWkgG0MBsFAobhO0J5pQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=1)

### 2、Caffeine 配置说明

| 参数              | 类型     | 描述                                                       |
| ----------------- | -------- | ---------------------------------------------------------- |
| initialCapacity   | integer  | 初始的缓存空间大小                                         |
| maximumSize       | long     | 缓存的最大条数                                             |
| maximumWeight     | long     | 缓存的最大权重                                             |
| expireAfterAccess | duration | 最后一次写入或访问后经过固定时间过期                       |
| refreshAfterWrite | duration | 最后一次写入后经过固定时间过期                             |
| refreshAfterWrite | duration | 创建缓存或者最近一次更新缓存后经过固定的时间间隔，刷新缓存 |
| weakKeys          | boolean  | 打开 key 的弱引用                                          |
| weakValues        | boolean  | 打开 value 的弱引用                                        |
| softValues        | boolean  | 打开 value 的软引用                                        |
| recordStats       | -        | 开发统计功能                                               |

**注意：**

- `weakValues` 和 `softValues` 不可以同时使用。
- `maximumSize` 和 `maximumWeight` 不可以同时使用。
- `expireAfterWrite` 和 `expireAfterAccess` 同事存在时，以 `expireAfterWrite` 为准。

### 3、软引用与弱引用

软引用：如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。

弱引用：弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存

```
// 软引用
Caffeine.newBuilder().softValues().build();

// 弱引用
Caffeine.newBuilder().weakKeys().weakValues().build();
```

## 三、SpringBoot 集成 Caffeine 两种方式

SpringBoot 有俩种使用 Caffeine 作为缓存的方式：

- 方式一：直接引入 Caffeine 依赖，然后使用 Caffeine 方法实现缓存。
- 方式二：引入 Caffeine 和 Spring Cache 依赖，使用 SpringCache 注解方法实现缓存。

下面将介绍下，这俩中集成方式都是如何实现的。

## 四、SpringBoot 集成 Caffeine 方式一

### 1、Maven 引入相关依赖

```xml
 <dependency>
            <groupId>com.github.ben-manes.caffeine</groupId>
            <artifactId>caffeine</artifactId>
        </dependency>
```

### 2、配置缓存配置类

```java
import com.github.benmanes.caffeine.cache.Cache;
import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.concurrent.TimeUnit;

@Configuration
public class CacheConfig {

    @Bean
    public Cache caffeineCache() {
        return Caffeine.newBuilder()
                // 设置最后一次写入或访问后经过固定时间过期
                .expireAfterWrite(60, TimeUnit.SECONDS)
                // 初始的缓存空间大小
                .initialCapacity(100)
                // 缓存的最大条数
                .maximumSize(1000)
                .build();
    }

}
```

### 3、定义测试的实体对象

```
import lombok.Data;
import lombok.ToString;

@Data
@ToString
public class UserInfo {
    private Integer id;
    private String name;
    private String sex;
    private Integer age;
}
```

### 4、定义服务接口类和实现类

UserInfoService

```
import mydlq.club.example.entity.UserInfo;

public interface UserInfoService {

    /**
     * 增加用户信息
     *
     * @param userInfo 用户信息
     */
    void addUserInfo(UserInfo userInfo);

    /**
     * 获取用户信息
     *
     * @param id 用户ID
     * @return 用户信息
     */
    UserInfo getByName(Integer id);

    /**
     * 修改用户信息
     *
     * @param userInfo 用户信息
     * @return 用户信息
     */
    UserInfo updateUserInfo(UserInfo userInfo);

    /**
     * 删除用户信息
     *
     * @param id 用户ID
     */
    void deleteById(Integer id);

}
```

UserInfoServiceImpl

```
import com.github.benmanes.caffeine.cache.Cache;
import lombok.extern.slf4j.Slf4j;
import mydlq.club.example.entity.UserInfo;
import mydlq.club.example.service.UserInfoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;
import java.util.HashMap;

@Slf4j
@Service
public class UserInfoServiceImpl implements UserInfoService {

    /**
     * 模拟数据库存储数据
     */
    private HashMap userInfoMap = new HashMap<>();

    @Autowired
    Cache<String, Object> caffeineCache;

    @Override
    public void addUserInfo(UserInfo userInfo) {
        log.info("create");
        userInfoMap.put(userInfo.getId(), userInfo);
        // 加入缓存
        caffeineCache.put(String.valueOf(userInfo.getId()),userInfo);
    }

    @Override
    public UserInfo getByName(Integer id) {
        // 先从缓存读取
        caffeineCache.getIfPresent(id);
        UserInfo userInfo = (UserInfo) caffeineCache.asMap().get(String.valueOf(id));
        if (userInfo != null){
            return userInfo;
        }
        // 如果缓存中不存在，则从库中查找
        log.info("get");
        userInfo = userInfoMap.get(id);
        // 如果用户信息不为空，则加入缓存
        if (userInfo != null){
            caffeineCache.put(String.valueOf(userInfo.getId()),userInfo);
        }
        return userInfo;
    }

    @Override
    public UserInfo updateUserInfo(UserInfo userInfo) {
        log.info("update");
        if (!userInfoMap.containsKey(userInfo.getId())) {
            return null;
        }
        // 取旧的值
        UserInfo oldUserInfo = userInfoMap.get(userInfo.getId());
        // 替换内容
        if (!StringUtils.isEmpty(oldUserInfo.getAge())) {
            oldUserInfo.setAge(userInfo.getAge());
        }
        if (!StringUtils.isEmpty(oldUserInfo.getName())) {
            oldUserInfo.setName(userInfo.getName());
        }
        if (!StringUtils.isEmpty(oldUserInfo.getSex())) {
            oldUserInfo.setSex(userInfo.getSex());
        }
        // 将新的对象存储，更新旧对象信息
        userInfoMap.put(oldUserInfo.getId(), oldUserInfo);
        // 替换缓存中的值
        caffeineCache.put(String.valueOf(oldUserInfo.getId()),oldUserInfo);
        return oldUserInfo;
    }

    @Override
    public void deleteById(Integer id) {
        log.info("delete");
        userInfoMap.remove(id);
        // 从缓存中删除
        caffeineCache.asMap().remove(String.valueOf(id));
    }

}
```

### 5、测试的 Controller 类

```
import mydlq.club.example.entity.UserInfo;
import mydlq.club.example.service.UserInfoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping
public class UserInfoController {

    @Autowired
    private UserInfoService userInfoService;

    @GetMapping("/userInfo/{id}")
    public Object getUserInfo(@PathVariable Integer id) {
        UserInfo userInfo = userInfoService.getByName(id);
        if (userInfo == null) {
            return "没有该用户";
        }
        return userInfo;
    }

    @PostMapping("/userInfo")
    public Object createUserInfo(@RequestBody UserInfo userInfo) {
        userInfoService.addUserInfo(userInfo);
        return "SUCCESS";
    }

    @PutMapping("/userInfo")
    public Object updateUserInfo(@RequestBody UserInfo userInfo) {
        UserInfo newUserInfo = userInfoService.updateUserInfo(userInfo);
        if (newUserInfo == null){
            return "不存在该用户";
        }
        return newUserInfo;
    }

    @DeleteMapping("/userInfo/{id}")
    public Object deleteUserInfo(@PathVariable Integer id) {
        userInfoService.deleteById(id);
        return "SUCCESS";
    }

}
```

## 五、SpringBoot 集成 Caffeine 方式二

### 1、Maven 引入相关依赖

```
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>

        <dependency>
            <groupId>com.github.ben-manes.caffeine</groupId>
            <artifactId>caffeine</artifactId>
        </dependency>
```

### 2、配置缓存配置类

```
@Configuration
public class CacheConfig {

    /**
     * 配置缓存管理器
     *
     * @return 缓存管理器
     */
    @Bean("caffeineCacheManager")
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
                // 设置最后一次写入或访问后经过固定时间过期
                .expireAfterAccess(60, TimeUnit.SECONDS)
                // 初始的缓存空间大小
                .initialCapacity(100)
                // 缓存的最大条数
                .maximumSize(1000));
        return cacheManager;
    }

}
```

### 

###  

### 3、定义测试的实体对象



```
@Data
@ToString
public class UserInfo {
    private Integer id;
    private String name;
    private String sex;
    private Integer age;
}
```

### 4、定义服务接口类和实现类

服务接口

```
import mydlq.club.example.entity.UserInfo;

public interface UserInfoService {

    /**
     * 增加用户信息
     *
     * @param userInfo 用户信息
     */
    void addUserInfo(UserInfo userInfo);

    /**
     * 获取用户信息
     *
     * @param id 用户ID
     * @return 用户信息
     */
    UserInfo getByName(Integer id);

    /**
     * 修改用户信息
     *
     * @param userInfo 用户信息
     * @return 用户信息
     */
    UserInfo updateUserInfo(UserInfo userInfo);

    /**
     * 删除用户信息
     *
     * @param id 用户ID
     */
    void deleteById(Integer id);

}
```

服务实现类

```java
import lombok.extern.slf4j.Slf4j;
import mydlq.club.example.entity.UserInfo;
import mydlq.club.example.service.UserInfoService;
import org.springframework.cache.annotation.CacheConfig;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;
import java.util.HashMap;

@Slf4j
@Service
@CacheConfig(cacheNames = "caffeineCacheManager")
public class UserInfoServiceImpl implements UserInfoService {

    /**
     * 模拟数据库存储数据
     */
    private HashMap userInfoMap = new HashMap<>();

    @Override
    @CachePut(key = "#userInfo.id")
    public void addUserInfo(UserInfo userInfo) {
        log.info("create");
        userInfoMap.put(userInfo.getId(), userInfo);
    }

    @Override
    @Cacheable(key = "#id")
    public UserInfo getByName(Integer id) {
        log.info("get");
        return userInfoMap.get(id);
    }

    @Override
    @CachePut(key = "#userInfo.id")
    public UserInfo updateUserInfo(UserInfo userInfo) {
        log.info("update");
        if (!userInfoMap.containsKey(userInfo.getId())) {
            return null;
        }
        // 取旧的值
        UserInfo oldUserInfo = userInfoMap.get(userInfo.getId());
        // 替换内容
        if (!StringUtils.isEmpty(oldUserInfo.getAge())) {
            oldUserInfo.setAge(userInfo.getAge());
        }
        if (!StringUtils.isEmpty(oldUserInfo.getName())) {
            oldUserInfo.setName(userInfo.getName());
        }
        if (!StringUtils.isEmpty(oldUserInfo.getSex())) {
            oldUserInfo.setSex(userInfo.getSex());
        }
        // 将新的对象存储，更新旧对象信息
        userInfoMap.put(oldUserInfo.getId(), oldUserInfo);
        // 返回新对象信息
        return oldUserInfo;
    }

    @Override
    @CacheEvict(key = "#id")
    public void deleteById(Integer id) {
        log.info("delete");
        userInfoMap.remove(id);
    }

} 
```

### 5、测试的 Controller 类

```java
import mydlq.club.example.entity.UserInfo;
import mydlq.club.example.service.UserInfoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping
public class UserInfoController {

    @Autowired
    private UserInfoService userInfoService;

    @GetMapping("/userInfo/{id}")
    public Object getUserInfo(@PathVariable Integer id) {
        UserInfo userInfo = userInfoService.getByName(id);
        if (userInfo == null) {
            return "没有该用户";
        }
        return userInfo;
    }

    @PostMapping("/userInfo")
    public Object createUserInfo(@RequestBody UserInfo userInfo) {
        userInfoService.addUserInfo(userInfo);
        return "SUCCESS";
    }

    @PutMapping("/userInfo")
    public Object updateUserInfo(@RequestBody UserInfo userInfo) {
        UserInfo newUserInfo = userInfoService.updateUserInfo(userInfo);
        if (newUserInfo == null){
            return "不存在该用户";
        }
        return newUserInfo;
    }

    @DeleteMapping("/userInfo/{id}")
    public Object deleteUserInfo(@PathVariable Integer id) {
        userInfoService.deleteById(id);
        return "SUCCESS";
    }

}
```

### 引用文章
[SpringBoot 使用 Caffeine 本地缓存](http://www.mydlq.club/article/56/)

