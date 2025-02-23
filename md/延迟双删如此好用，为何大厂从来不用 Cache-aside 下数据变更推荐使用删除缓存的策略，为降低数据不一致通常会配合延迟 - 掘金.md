> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7447033901657096202?searchId=20250224004309EAD3AFE954AAD6D1E04D)

> Cache-aside 下数据变更推荐使用删除缓存的策略，为降低数据不一致通常会配合延迟双删策略。

**摘要：** 在绝大多数介绍缓存与数据库一致性方案的文章中，随着 Cache-aside 模式的数据变更几乎无例外的推荐使用删除缓存的策略，为进一步降低数据不一致的风险通常会配合延迟双删的策略。但是令人意外的是，在一些互联网大厂中的核心业务却很少使用这种方式。这背后的原因是什么呢？延迟双删策略有什么致命缺陷么？以及这些大厂如何选择缓存与数据库一致性保障的策略呢？如果你对此同样抱有有疑问的话，希望本文能为你答疑解惑。

* * *

当数据库（主副本）数据记录变更时，为了降低缓存数据不一致状态的持续时间，通常会选择主动 失效 / 更新 缓存数据的方式。绝大多数应用系统的设计方案中会选择通过删除缓存数据的方式使其失效。但同样会出现数据不一致的情况，具体情况参见下图： ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==) 所以延迟双删又成为了组合出现的常见模式。延迟双删最复杂的技术实现在于对延迟时间的确定上，间隔时间久的话数据不一致的状态持续时间会变长，如果间隔时间过短可能无法起到一致性保障的作用。所以基于经验会将这个时间设定在秒级，如 1-2 秒后执行第二次删除操作。

延迟双删的致命缺陷
---------

但是延迟时间最大的问题不在于此，而是两次删除缓存数据引起的缓存穿透，短时间对数据库（主副本）造成的流量与负载压力。绝大多数应用系统本身流量与负载并不高，使用缓存通常是为了提升系统性能表现，数据库（主副本）完全可以承载一段时间内的负载压力。对于此类系统延迟双删是一个完全可以接受的高性价比策略。

现实世界中的系统响应慢所带来的却是流量的加倍上涨。回想一下当你面对 App 响应慢的情况，是如何反应与对待便能明白，几乎所有用户的下意识行为都是如出一辙。

所以对于那些流量巨大的应用系统而言，短时的访问流量穿透缓存访问数据库（主副本），恐怕很难接受。为了应对这种流量穿透的情况，通常需要增加数据库（主副本）的部署规格或节点。而且这类应用系统的响应变慢的时候，会对其支持系统产生影响，如果其支持系统较多的情况下，会存在影响的增溢。相比延迟双删在技术实现上带来高效便捷而言，其对系统的影响与副作用则变得不可忽视。

Facebook（今 Meta）解决方案
--------------------

早在 2013 年由 Facebook（今 Meta）发表的论文 “Scaling Memcache at Facebook” 中便提供了其内部的解决方案，通过提供一种类似 “锁” 的 “leases”（本文译为 “租约”）机制防止并发带来的数据不一致现象。

租约机制实现方法大致如下：

> 当有多个请求抵达缓存时，缓存中并不存在该值时会返回给客户端一个 64 位的 token ，这个 token 会记录该请求，同时该 token 会和缓存键作为绑定，该 token 即为上文中租约的值，客户端在更新时需要传递这个 token ，缓存验证通过后会进行数据的存储。其他请求需要等待这个租约过期后才可申请新的租约。

可结合下图辅助理解其作用机制。也可阅读[缓存与主副本数据一致性系统设计方案（下篇）](https://juejin.cn/post/7440021417506979866#heading-5 "https://juejin.cn/post/7440021417506979866#heading-5")一文中的如何解决并发数据不一致，又能避免延迟双删带来的惊群问题章节进一步了解。 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

### 简易参考实现

接下来我们以 Redis 为例，提供一个 Java 版本的简易参考实现。本文中会给出实现所涉及的关键要素与核心代码，你可以访问 [Github 项目](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FHAibiiin%2Fsystem-design-codebase%2Ftree%2Fmaster%2Fcache-aside-pattern "https://github.com/HAibiiin/system-design-codebase/tree/master/cache-aside-pattern") 来了解整个样例工程，并通过查阅 [Issue 与 commits](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FHAibiiin%2Fsystem-design-codebase%2Fissues%2F1 "https://github.com/HAibiiin/system-design-codebase/issues/1") 来了解整个样例工程的演化进程。

要想实现上述租约机制，需要关注的核心要素有三个：

1.  需要复写 Redis 数据获取操作，当 Redis 中数据不存在时增加对租约的设置；
2.  需要复写 Redis 数据设置操作，当设置 Redis 中数据时校验租约的有效性；
3.  最后是当数据库（主副本）数据变更时，删除 Redis 数据同时要连带删除租约信息。

同时为了保障 Redis 操作的原子性，我们需要借助 Lua 脚本来实现上述三点。这里以字符串类型为例，对应脚本分别如下：

#### Redis 数据获取操作

返回值的第二个属性作为判断是否需要执行数据获取的判断依据。当为 false 时表示 Redis 中无对应数据，需要从数据库中加载，同时保存了当前请求与 key 对应的租约信息。

```
local key = KEYS[1]
local token = ARGV[1]
local value = redis.call('get', key)
if not value then
    redis.replicate_commands()
    local lease_key = 'lease:'..key
    redis.call('set', lease_key, token)
    return {false, false}
else
    return {value, true}
end
```

#### Redis 数据设置操作

返回值的第二个属性作为判断是否成功执行数据设置操作的依据。该属性为 false 表示租约校验失败，未成功执行数据设置操作。同时意味着有其他进程 / 线程 执行数据查询操作并对该 key 设置了新的租约。

```
local key = KEYS[1]
local token = ARGV[1]
local value = ARGV[2]
local lease_key = 'lease:'..key
local lease_value = redis.call('get', lease_key)
if lease_value == token then
    redis.replicate_commands()
    redis.call('set', key, value)
    return {value, true}
else
    return {false, false}
end
```

#### Redis 数据删除操作

当数据库变更进程 / 线程 完成数据变更操作后，尝试删除缓存需要同时清理对应数据记录的 key 以及其关联租约 key。防止数据变更前的查询操作通过租约校验，将旧数据写入 Redis 。

```
local key = KEYS[1]
local token = ARGV[1]
local lease_key = 'lease:'..key
redis.call('del', key, leask_key)
```

该方案主要的影响在应用层实现，主要在集中在三个方面：

1.  应用层不能调用 Redis 数据类型的原始操作命令，而是改为调用 [EVAL](https://link.juejin.cn/?target=https%3A%2F%2Fredis.io%2Fdocs%2Flatest%2Fcommands%2Feval%2F "https://redis.io/docs/latest/commands/eval/") 命令；
2.  调用 Redis 返回结果数据结构的变更为数组，需要解析数组；
3.  应用层对于 Redis 的操作变复杂，需要生成租约用的 token，并根据每个阶段返回结果进行后续处理；

为应对上述三点变化，对应操作 Redis 的 Java 实现如下：

#### 封装返回结果

为便于后续操作，首先是对脚本返回结果的封装。

```
public class EvalResult {  

    String value;  
    boolean effect;  

    public EvalResult(List<?> args) {  
        value = (String) args.get(0);  
        if (args.get(1) == null) {  
            effect = false;  
        } else {  
            effect = 1 == (long) args.get(1);  
        }  
    }  
}
```

#### 组件设计

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

#### 封装 Redis 操作

因为在[样例工程](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FHAibiiin%2Fsystem-design-codebase%2Ftree%2Fmaster%2Fcache-aside-pattern "https://github.com/HAibiiin/system-design-codebase/tree/master/cache-aside-pattern")中独立出了一个 Query Engine 组件，所以需要跨组件传递 token，这里为了实现简单采用了 ThreadLocal 进行 token 的传递，具体系统可查阅[样例工程中的用例](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FHAibiiin%2Fsystem-design-codebase%2Fblob%2Fmaster%2Fcache-aside-pattern%2Fquery-engine%2Fsrc%2Fmain%2Fjava%2Fio%2Fhaibiiin%2Fgithub%2Fquery%2Fengine%2FSimpleQueryEngine.java "https://github.com/HAibiiin/system-design-codebase/blob/master/cache-aside-pattern/query-engine/src/main/java/io/haibiiin/github/query/engine/SimpleQueryEngine.java")。

```
public class LeaseWrapper extends Jedis implements CacheCommands {  
      
    private final Jedis jedis;  
    private final TokenGenerator tokenGenerator;  
    private final ThreadLocal<String> tokenHolder;  
      
    public LeaseWrapper(Jedis jedis) {  
        this.jedis = jedis;  
        this.tokenHolder = new ThreadLocal<>();  
        this.tokenGenerator = () -> UUID.randomUUID().toString();  
    }   

	@Override
    public String get(String key) {  
        String token = this.tokenGenerator.get();  
        tokenHolder.set(token);  
        Object result = this.jedis.eval(LuaScripts.leaseGet(), List.of(key), List.of(token));  
        EvalResult er = new EvalResult((List<?>) result);  
        if (er.effect()) {  
            return er.value();  
        }  
        return null;  
    }  

	@Override
    public String set(String key, String value) {  
        String token = tokenHolder.get();  
        tokenHolder.remove();  
        Object result = this.jedis.eval(LuaScripts.leaseSet(), List.of(key), List.of(token, value));  
        EvalResult er = new EvalResult((List<?>) result);  
        if (er.effect()) {  
            return er.value();  
        }  
        return null;  
    }  
      
}
```

### 补充

在上面的简易参考实现中，我们并没有实现**其他请求需要等待这个租约过期后才可申请新的租约**。该功能主要是防止惊群问题，进一步降低可能对数据库造成的访问压力。要实现该功能需要在 Redis 数据获取操作中改进脚本：

```
local key = KEYS[1]
local token = ARGV[1]
local value = redis.call('get', key)
if not value then
    redis.replicate_commands()
    local lease_key = 'lease:'..key
    local current_token = redis.call('get', lease_key)
    if not current_token or token == current_token then
	    redis.call('set', lease_key, token)
	    return {token, false}
	else
		return {current_token, false}
	end
else
    return {value, true}
end
```

同时也可以为租约数据设定一个短时 TTL，并在应用层通过对 EvalResult 的 effect 判断为 false 的情况下等待一段时间后再次执行。

上述实现的复杂点在于租约过期的时间的选取，以及超过设定时间的逻辑处理。我们可以实现类似自旋锁的机制，在最大等待时间内随时等待一个间隙向 Redis 发起查询请求，超过最大等待时间后直接查询数据库（主副本）获取数据。

Uber 解决方案
---------

在 Uber 今年 2 月份发表的一篇技术博客 [“How Uber Serves Over 40 Million Reads Per Second from Online Storage Using an Integrated Cache”](https://link.juejin.cn/?target=https%3A%2F%2Fwww.uber.com%2Fen-IN%2Fblog%2Fhow-uber-serves-over-40-million-reads-per-second-using-an-integrated-cache%2F "https://www.uber.com/en-IN/blog/how-uber-serves-over-40-million-reads-per-second-using-an-integrated-cache/") 中透露了其内部的解决方案，通过比对版本号的方式避免将旧数据写入缓存。

版本号比对机制实现方法大致如下：

> 将数据库中行记录的时间戳作为版本号，通过 Lua 脚本通过 Redis EVAL 命令提供类似 MSET 的更新操作，基于自定义编解码器提取 Redis 记录中的版本号，在执行数据设置操作时进行比对，只写入较新的数据。

其中 Redis 的数据记录对应的 Key-Value 编码格式如所示： ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

### 简易参考实现

接下来我们以 Redis 为例，提供一个 Java 版本的简易参考实现。本文中会给出实现所涉及的关键要素与核心代码，你可以访问 [Github 项目](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FHAibiiin%2Fsystem-design-codebase%2Ftree%2Fmaster%2Fcache-aside-pattern "https://github.com/HAibiiin/system-design-codebase/tree/master/cache-aside-pattern") 来了解整个样例工程，并通过查阅 [Issue 与 commits](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FHAibiiin%2Fsystem-design-codebase%2Fissues%2F3 "https://github.com/HAibiiin/system-design-codebase/issues/3") 来了解整个样例工程的演化进程。

我们这里不采取定制数据格式，而是通过额外的缓存 Key 存储数据版本，要想实现类似版本号比对机制，需要关注的核心要素有两个：

1.  需要复写 Redis 数据设置操作，当设置 Redis 中数据时校验版本号；
2.  在版本号比对通过后需要绑定版本号数据，与主数据同步写入 Redis 中。

同时为了保障 Redis 操作的原子性，我们需要借助 Lua 脚本来实现上述两点。这里以字符串类型为例，对应脚本分别如下：

#### Redis 数据设置操作

返回值的第二个属性作为判断是否成功执行数据设置操作的依据。该属性为 false 表示数据未成功写入 Redis。同时意味当前 进程 / 线程 执行写入的数据为历史数据，在次过程中数据已经发生变更并又其他数据写入。

```
local key = KEYS[1]  
local value = ARGV[1]  
local current_version = ARGV[2]  
local version_key = 'version:'..key  
local version_value = redis.call('get', version_key)  
if version_value == false or version_value < current_version then  
    redis.call('mset', version_key, current_version, key, value)
    return {value, true}
else
    return {false, false}
end
```

该方案主要的影响在应用层实现，需要在调用 Redis 的 EVAL 命令前从数据实体中提取时间戳作为版本号，同时需要保障数据实体中包含时间戳相关属性。

#### 封装 Redis 操作

结合我们的[样例工程代码](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FHAibiiin%2Fsystem-design-codebase%2Ftree%2Fmaster%2Fcache-aside-pattern "https://github.com/HAibiiin/system-design-codebase/tree/master/cache-aside-pattern")，我们通过实现 VersionWrapper 对 Redis 的操作进行如下封装。

```
public class VersionWrapper extends Jedis implements CacheCommands {  
      
    private final Jedis jedis;  
      
    public VersionWrapper(Jedis jedis) {  
        this.jedis = jedis;  
    }
 
    @Override  
    public String set(String key, String value, String version) {  
        Object result = this.jedis.eval(LuaScripts.versionSet(), List.of(key), List.of(value, version));  
        EvalResult er = new EvalResult((List<?>) result);  
        if (er.effect()) {  
            return er.value();  
        }  
        return null;  
    }
}
```

### 补充

透过该方案我们推测 Uber 采取的并非数据变更后删除缓存的策略，很可能是更新缓存的策略（在 Uber 的技术博客中也间接的提到了更新缓存的策略）。

因为整个版本号比对的方式与删除缓存的逻辑相悖。我们抛开 Uber CacheFront 的整体架构，仅仅将该方案应用在简单架构模型中。采取删除缓存的策略，可能会产生如下图所示的结果，此时应用服务 Server - 2 因为查询缓存未获取到值，而从数据库加载并写入缓存，但是此时缓存中写入的为历史旧值，而在该数据过期前或者下次数据变更前，都不会再触发更新了。 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==) 当然对于更新缓存的策略同样面临这个问题，因为当数据变更发生期间，缓存中并没有该数据记录时，通常我们不会采取主动刷新缓存的策略，那么则依然会面对上面的问题。

而 Uber 的 CacheFront 基于企业内部的 Flux 技术组件实现对缓存的异步处理，通过阅读文章我们也可以发现这个异步延迟在秒级，那么在如此长的时间间隙后，无论采用删除还是更新策略想要产生上图中的不一致现象都比较难，因为对应用系统来说，进程 / 线程阻塞 2-3 秒是很难以忍受的现象，所以通常不会出现如此漫长的阻塞与卡顿。

如果你想进一步了解如何实现与 Uber 利用 Flux 实现缓存异步处理的内容，也可阅读我们此前[缓存与主副本数据一致性系统设计方案（下篇）](https://juejin.cn/post/7440021417506979866#heading-2 "https://juejin.cn/post/7440021417506979866#heading-2")文章中更新主副本数据后更新缓存并发问题解决方案章节。

总结
--

本文并非对延迟双删的全盘否定，而是强调在特殊场景下，延迟双删策略的弊端会被放大，进而完全盖过其优势。对于那些业务体量大伴随着流量大的应用系统，必应要从中权衡取舍。

每一种策略都仅适配应用系统生命周期的一段。只不过部分企业随着业务发展逐步壮大，其研发基础设施的能力也更完善。从而为系统设计带来诸多便捷，从而使得技术决策变得与中小研发团队截然不同。

所以当我们在学习他人经验的过程中，到了落地执行环节一定要结合实际团队背景、业务需求、开发周期与资金预算进行灵活适配。如果你希望了解更多技术中立（排除特定基础设施）的系统设计方案，欢迎你关注我的账号或订阅我的[系统设计实战：常用架构与模式详解](https://juejin.cn/column/7438216368476585984 "https://juejin.cn/column/7438216368476585984")专栏，我将在其中持续更新技术中立的系统设计系列文章。如果您发现文章内容中任何不准确或遗漏的部分。非常希望您能评论指正，我将尽快修正疏漏，为大家提供优质技术内容。

> **相关阅读**

*   [缓存与主副本数据一致性系统设计方案](https://link.juejin.cn/?target=https%3A%2F%2Fhaibiiin.github.io%2Fposts%2Fsystem_design_in_action%2FThe_design_for_ensuring_data_consistency_between_cache_and_primary_replica.html "https://haibiiin.github.io/posts/system_design_in_action/The_design_for_ensuring_data_consistency_between_cache_and_primary_replica.html")
*   [System-Design-Codebase](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FHAibiiin%2Fsystem-design-codebase "https://github.com/HAibiiin/system-design-codebase")
*   [Scaling Memcache at Facebook](https://link.juejin.cn/?target=https%3A%2F%2Fresearch.facebook.com%2Fpublications%2Fscaling-memcache-at-facebook%2F "https://research.facebook.com/publications/scaling-memcache-at-facebook/")
*   [How Uber Serves Over 40 Million Reads Per Second from Online Storage Using an Integrated Cache](https://link.juejin.cn/?target=https%3A%2F%2Fwww.uber.com%2Fen-IN%2Fblog%2Fhow-uber-serves-over-40-million-reads-per-second-using-an-integrated-cache%2F "https://www.uber.com/en-IN/blog/how-uber-serves-over-40-million-reads-per-second-using-an-integrated-cache/")

> 你好，我是 HAibiiin，一名探索技术之外更多可能性的 Product Engineer。如果本篇文章对你有所启发或提供了一定价值，还请不要吝啬点赞、收藏和关注。 ![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2025/02/24/c644c701a54b449eb9eb5cf4ff252dda~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgSEFpYmlpaW4=:q75.awebp?rk3s=f64ab15b&x-expires=1740873192&x-signature=F3yyl4rE3zSUOhp%2BIs0jfSLN2Nc%3D)