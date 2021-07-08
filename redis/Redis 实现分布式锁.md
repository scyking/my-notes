# Redis 实现分布式锁

标签（空格分隔）： redis

---

[toc]

## 分布式锁

- 互斥性，任意时刻仅一个客户端能获取锁
- 安全性，锁只能被持有锁的客户端删除
- 死锁，无法释放锁，无法获取锁
- 容错，某个redis节点宕机，客户端仍能获取锁或释放锁

## 创建锁

- XX：是否存在key，存在则set成功
- NX：是否存在key，存在则set失败
- PX：key过期时间，单位毫秒
- EX：key过期时间，单位秒

```
public boolean setnx(String key, String val) {
    Jedis jedis = null;
    try {
        jedis = jedisPool.getResource();
        if (jedis == null) {
            return false;
        }
        return jedis.set(key, val, "NX", "PX", 1000 * 60).equalsIgnoreCase("ok");
    } catch (Exception ex) {
        //
    } finally {
        if (jedis != null) {
            jedis.close();
        }
    }
    return false;
}
```

## 释放锁
> 使用lua脚本，根据val判断其是否存在，如果存在旧del；

```
public int delnx(String key, String val) {
    Jedis jedis = null;
    try {
        jedis = jedisPool.getResource();
        if (jedis == null) {
            return 0;
        }
        //if redis.call('get','orderkey')=='1111' then return redis.call('del','orderkey') else return 0 end
        StringBuilder sbScript = new StringBuilder();
        sbScript.append("if redis.call('get','")
            .append(key).append("')").append("=='").append(val).append("'")
            .append(" then ").append(" return redis.call('del','").append(key).append("')")
            .append(" else ").append(" return 0").append(" end");
        return Integer.valueOf(jedis.eval(sbScript.toString()).toString());
    } catch (Exception ex) {
        //
    } finally {
        if (jedis != null) {
            jedis.close();
        }
    }
    return 0;
}
```

## 问题

不可重入




