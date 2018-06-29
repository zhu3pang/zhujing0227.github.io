---
layout: post
title: 利用Redis的Sorted-Set实现一个优先任务队列
tags: 杂七杂八
category: 杂七杂八
#eye_catch: http://jekyllrb.com/img/logo-2x.png
---

<!--more-->
<!--more-->
使用zset实现一个优先任务队列实际上只需要下面三个命令, `ZADD、ZRANGE、ZREMRANGEBYRANK`, ZADD用于往队列添加任务, ZRANGE和ZREMRANGEBYRANK用于取任务. 取任务是分两步操作的, 不是原子操作, 并发下会出现任务重复抓取的情况. redis客户端里可以使用 MULTI / EXEC 包围的事务, 在Java里可以将ZRANGE和ZREMRANGEBYRANK两步操作放在lua脚本里执行, 下面给出使用spring-data-redis的代码及测试样例.

```shell
ZADD key score member [[score member] [score member] ...]

MULTI
ZRANGE key 0 0 WITHSCORES
ZREMRANGEBYRANK task_list 0 0
EXEC
```

- spring-redis+lua脚本实现代码
```Java
/**
 * @author zhuji on 2018/6/29
 */

@Component
@SuppressWarnings({"rawtypes", "unchecked"})
public class RedisPriorityQueue {

    private final RedisTemplate redisTemplate;

    @Autowired
    public RedisPriorityQueue(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }


    public void zadd(String key, double score, Object member) {
        redisTemplate.boundZSetOps(key).add(member, score);
    }

    public <T> List<T> zpop(String key) {
        String script = "local items = redis.call('ZREVRANGE', KEYS[1], 0, 0)\n" +
                "if next(items) ~= nil then\n" +
                "   redis.call('zrem', KEYS[1], unpack(items))\n" +
                "end\n" +
                "return items";

        return (List<T>) redisTemplate.execute(new DefaultRedisScript<>(script, List.class), Lists.newArrayList(key));
    }

    public <T> List<T> zpopSome(String key, long start, long end) {
        String script = "local items = redis.call('ZREVRANGE', KEYS[1], %d, %d)\n" +
                "if next(items) ~= nil then\n" +
                "   redis.call('zrem', KEYS[1], unpack(items))\n" +
                "end\n" +
                "return items";
        script = String.format(script, start, end);

        return (List<T>) redisTemplate.execute(new DefaultRedisScript<>(script, List.class), Lists.newArrayList(key));
    }

    public long size(String key) {
        return Optional.ofNullable(redisTemplate.boundZSetOps(key).zCard()).orElse(0L);
    }
}
```

- 测试
```Java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class RedisPriorityQueueTest {

    @Autowired
    private RedisPriorityQueue redisPriorityQueue;

    @Test
    public void zadd() {
        redisPriorityQueue.zadd("test_zset", 1, "a");
        redisPriorityQueue.zadd("test_zset", 2, "b");
        redisPriorityQueue.zadd("test_zset", 3, "c");
        redisPriorityQueue.zadd("test_zset", 4, "d");
        assertEquals(redisPriorityQueue.size("test_zset"), 4);
    }

    @Test
    public void zpop() {
        List<String> list = redisPriorityQueue.zpop("test_zset");
        assertEquals(list.size(), 1);
        assertEquals(list.get(0), "d");
    }

    @Test
    public void zpopSome() {
        List<String> list = redisPriorityQueue.zpopSome("test_zset", 0, 2);
        assertEquals(list.size(), 3);
        assertEquals(redisPriorityQueue.size("test_zset"), 1);
        System.out.println(list);
    }
}
```