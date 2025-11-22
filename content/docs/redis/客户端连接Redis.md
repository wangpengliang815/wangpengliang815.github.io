测试代码：https://github.com/wangpengliang815/CodeSnippet/tree/master/CommonLib/CommonLib

参考文档：https://www.bookstack.cn/books/StackExchange.Redis-docs-cn

<!-- more -->

# 客户端

- ServiceStack.Redis 是商业版，免费版有限制
- StackExchange.Redis 是免费版，但是内核在 .NETCore 运行有问题经常 Timeout，暂无法解决
- csRedis于2016年开始支持.NETCore一直迭代至今，实现了低门槛、高性能，和分区高级玩法的.NETCore redis-cli SDK



# ServiceStack.Redis

商业版，免费版有限制。放弃。

# StackExchange.Redis

## 安装

`StackExchange.Redis` 是 C# 操作 `Redis` 数据库的客户端。在 `NuGet` 中搜索 `StackExchange.Redis` ，直接点击按钮安装即可。

## 封装

`RedisHelper`

```csharp
namespace CommonLib
{
    using Newtonsoft.Json;

    using StackExchange.Redis;

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading.Tasks;

    /// <summary>
    /// 基于StackExchange.Redis封装的Helper
    /// 
    /// </summary>
    public class RedisHelper
    {
        /// <summary>
        /// 自定义缓存Key前缀
        /// </summary>
        /// <value>
        /// The custom key prefix.
        /// </value>
        private string CustomKeyPrefix { get; }

        /// <summary>
        /// ConnectionMultiplexer对象
        /// </summary>
        private readonly ConnectionMultiplexer conn;

        /// <summary>
        /// Redis数据库对象
        /// </summary>
        public readonly IDatabase db;

        /// <summary>
        /// Initializes a new instance of the <see cref="RedisHelper"/> class.
        /// </summary>
        /// <param name="dbSerialNumber">The database serial number.</param>
        /// <param name="redisConnectionString">The redis connection string.</param>
        /// <param name="customKeyPrefix">The custom key prefix.</param>
        public RedisHelper(int dbSerialNumber
            , string redisConnectionString
            , string customKeyPrefix = null)
        {
            CustomKeyPrefix = customKeyPrefix;
#if customSingleton 
            RedisConnectionMultiplexerHelper.RedisConnectionString = redisConnectionString;
            conn = RedisConnectionMultiplexerHelper.Instance;
#endif
            conn = ConnectionMultiplexer.Connect(redisConnectionString);
            db = conn.GetDatabase(dbSerialNumber);
        }

        /// <summary>
        /// 保存单个k/v
        /// </summary>
        /// <param name="key">The key.</param>
        /// <param name="value">The value.</param>
        /// <param name="expiry">The expiry.</param>
        /// <returns></returns>
        public bool StringSet(string key, string value, TimeSpan? expiry = default)
        {
            key = GenerateCustomKey(key);
            return db.StringSet(key, value, expiry);
        }

        /// <summary>
        /// 保存多个k/v
        /// </summary>
        /// <param name="keyValues">The key values.</param>
        /// <returns></returns>
        public bool StringSet(List<KeyValuePair<RedisKey, RedisValue>> keyValues)
        {
            List<KeyValuePair<RedisKey, RedisValue>> newkeyValues = keyValues
                .Select(p => new KeyValuePair<RedisKey, RedisValue>(GenerateCustomKey(p.Key), p.Value))
                .ToList();
            return db.StringSet(newkeyValues.ToArray());
        }

        /// <summary>
        /// 以Json格式保存对象
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <param name="object">The object.</param>
        /// <param name="expiry">The expiry.</param>
        /// <returns></returns>
        public bool StringSet<T>(string key, T @object, TimeSpan? expiry = default)
        {
            key = GenerateCustomKey(key);
            string json = ConvertToJson(@object);
            return db.StringSet(key, json, expiry);
        }

        /// <summary>
        /// 获取单个Key
        /// </summary>
        /// <param name="key">The key.</param>
        /// <returns></returns>
        public string StringGet(string key)
        {
            key = GenerateCustomKey(key);
            return db.StringGet(key);
        }

        /// <summary>
        /// 获取多个Key
        /// </summary>
        /// <param name="listKeys">The list keys.</param>
        /// <returns></returns>
        public RedisValue[] StringGet(List<string> listKeys)
        {
            List<string> redisKeys = listKeys.Select(GenerateCustomKey).ToList();
            return db.StringGet(ConvertToRedisKeys(redisKeys));
        }

        /// <summary>
        /// 获取value转换为Object
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <returns></returns>
        public T StringGet<T>(string key)
        {
            key = GenerateCustomKey(key);
            return ConvertToObject<T>(db.StringGet(key));
        }

        /// <summary>
        /// Increment,如果key存在,则:value+=value
        /// </summary>
        /// <param name="key">The key.</param>
        /// <param name="incrementVal">The increment value.</param>
        /// <returns></returns>
        public double StringIncrement(string key, double incrementVal = 1)
        {
            key = GenerateCustomKey(key);
            return db.StringIncrement(key, incrementVal);
        }

        /// <summary>
        /// Decrement,如果key存在,则:value-=value
        /// </summary>
        /// <param name="key"></param>
        /// <param name="decrementVal">The decrement value.</param>
        /// <returns>减少后的值</returns>
        public double StringDecrement(string key, double decrementVal = 1)
        {
            key = GenerateCustomKey(key);
            return db.StringDecrement(key, decrementVal);
        }

        /// <summary>
        /// 判断hash中某个key是否已经被缓存
        /// </summary>
        /// <param name="key">The key.</param>
        /// <param name="hashFieldName">Name of the hash field.</param>
        /// <returns></returns>
        public bool HashExists(string key, string hashFieldName)
        {
            key = GenerateCustomKey(key);
            return db.HashExists(key, hashFieldName);
        }

        /// <summary>
        /// 存储数据到hash表
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <param name="hashFieldName">Name of the hash field.</param>
        /// <param name="hashValue">The hash value.</param>
        /// <returns></returns>
        public bool HashSet<T>(string key, string hashFieldName, T hashValue)
        {
            key = GenerateCustomKey(key);
            string json = ConvertToJson(hashValue);
            return db.HashSet(key, hashFieldName, json);
        }

        /// <summary>
        /// 移除hash中的某值
        /// </summary>
        /// <param name="key">The key.</param>
        /// <param name="hashFieldName">Name of the hash field.</param>
        /// <returns></returns>
        public bool HashDelete(string key, string hashFieldName)
        {
            key = GenerateCustomKey(key);
            return db.HashDelete(key, hashFieldName);
        }

        /// <summary>
        /// 移除hash中的多个值
        /// </summary>
        /// <param name="key">The key.</param>
        /// <param name="hashFieldNames">The hash field names.</param>
        /// <returns></returns>
        public long HashDelete(string key, List<RedisValue> hashFieldNames)
        {
            key = GenerateCustomKey(key);
            return db.HashDelete(key, hashFieldNames.ToArray());
        }

        /// <summary>
        /// 从hash表获取数据
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <param name="hashFieldName">Name of the hash field.</param>
        /// <returns></returns>
        public T HashGet<T>(string key, string hashFieldName)
        {
            key = GenerateCustomKey(key);
            string value = db.HashGet(key, hashFieldName);
            return ConvertToObject<T>(value);
        }

        /// <summary>
        /// hash为数字增长value
        /// </summary>
        /// <param name="key">The key.</param>
        /// <param name="hashFieldName">Name of the hash field.</param>
        /// <param name="incrementVal">The increment value.</param>
        /// <returns></returns>
        public double HashIncrement(string key, string hashFieldName, double incrementVal = 1)
        {
            key = GenerateCustomKey(key);
            return db.HashIncrement(key, hashFieldName, incrementVal);
        }

        /// <summary>
        /// hash为数字减少value
        /// </summary>
        /// <param name="key">The key.</param>
        /// <param name="hashFieldName">Name of the hash field.</param>
        /// <param name="decrementVal">The decrement value.</param>
        /// <returns></returns>
        public double HashDecrement(string key, string hashFieldName, double decrementVal = 1)
        {
            key = GenerateCustomKey(key);
            return db.HashDecrement(key, hashFieldName, decrementVal);
        }

        /// <summary>
        /// 获取hashkey所有rediskey
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <returns></returns>
        public List<T> HashKeys<T>(string key)
        {
            key = GenerateCustomKey(key);
            RedisValue[] values = db.HashKeys(key);
            return ConvetToList<T>(values);
        }

        /// <summary>
        /// 移除指定List内部的值
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <param name="value">The value.</param>
        public void ListRemove<T>(string key, T value)
        {
            key = GenerateCustomKey(key);
            db.ListRemove(key, ConvertToJson(value));
        }

        /// <summary>
        /// 获取指定key的List
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <returns></returns>
        public List<T> ListRange<T>(string key)
        {
            key = GenerateCustomKey(key);
            RedisValue[] values = db.ListRange(key);
            return ConvetToList<T>(values);
        }

        /// <summary>
        /// 入队：列表从最右边插入一个元素
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <param name="value">The value.</param>
        /// <returns></returns>
        public long ListRightPush<T>(string key, T value)
        {
            key = GenerateCustomKey(key);
            return db.ListRightPush(key, ConvertToJson(value));
        }

        /// <summary>
        /// 出队：列表从最右边获取一个元素
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <returns></returns>
        public T ListRightPop<T>(string key)
        {
            key = GenerateCustomKey(key);
            RedisValue value = db.ListRightPop(key);
            return ConvertToObject<T>(value);
        }

        /// <summary>
        /// 入栈：列表从最左边插入一个元素
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <param name="value">The value.</param>
        /// <returns></returns>
        public long ListLeftPush<T>(string key, T value)
        {
            key = GenerateCustomKey(key);
            return db.ListLeftPush(key, ConvertToJson(value));
        }

        /// <summary>
        /// 出栈：列表从最左边获取一个元素
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <returns></returns>
        public T ListLeftPop<T>(string key)
        {
            key = GenerateCustomKey(key);
            var value = db.ListLeftPop(key);
            return ConvertToObject<T>(value);
        }

        /// <summary>
        /// 获取集合中的数量
        /// </summary>
        /// <param name="key">The key.</param>
        /// <returns></returns>
        public long ListLength(string key)
        {
            key = GenerateCustomKey(key);
            return db.ListLength(key);
        }

        /// <summary>
        /// Sorteds the set add.
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <param name="value">The value.</param>
        /// <param name="score">The score.</param>
        /// <returns></returns>
        public bool SortedSetAdd<T>(string key, T value, double score)
        {
            key = GenerateCustomKey(key);
            return db.SortedSetAdd(key, ConvertToJson<T>(value), score);
        }

        /// <summary>
        /// Sorteds the set remove.
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <param name="value">The value.</param>
        /// <returns></returns>
        public bool SortedSetRemove<T>(string key, T value)
        {
            key = GenerateCustomKey(key);
            return db.SortedSetRemove(key, ConvertToJson(value));
        }

        /// <summary>
        /// Sorteds the set range by rank.
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">The key.</param>
        /// <param name="start">The start.</param>
        /// <param name="stop">The stop.</param>
        /// <param name="order">The order.</param>
        /// <returns></returns>
        public List<T> SortedSetRangeByRank<T>(string key, long start, long stop, Order order = Order.Ascending)
        {
            key = GenerateCustomKey(key);
            var values = db.SortedSetRangeByRank(key, start, stop, order);
            return ConvetToList<T>(values);
        }

        /// <summary>
        /// Sorteds the length of the set.
        /// </summary>
        /// <param name="key">The key.</param>
        /// <returns></returns>
        public long SortedSetLength(string key)
        {
            key = GenerateCustomKey(key);
            return db.SortedSetLength(key);
        }

        /// <summary>
        /// 删除单个key
        /// </summary>
        /// <param name="key">The key.</param>
        /// <returns></returns>
        public bool KeyDelete(string key)
        {
            key = GenerateCustomKey(key);
            return db.KeyDelete(key);
        }

        /// <summary>
        /// 删除多个key
        /// </summary>
        /// <param name="keys">The keys.</param>
        /// <returns></returns>
        public long KeyDelete(List<string> keys)
        {
            List<string> newKeys = keys.Select(GenerateCustomKey).ToList();
            return db.KeyDelete(ConvertToRedisKeys(newKeys));
        }

        /// <summary>
        /// 判断key是否存储
        /// </summary>
        /// <param name="key">The key.</param>
        /// <returns></returns>
        public bool KeyExists(string key)
        {
            key = GenerateCustomKey(key);
            return db.KeyExists(key);
        }

        /// <summary>
        /// 重命名key
        /// </summary>
        /// <param name="key">The key.</param>
        /// <param name="newKey">The new key.</param>
        /// <returns></returns>
        public bool KeyRename(string key, string newKey)
        {
            key = GenerateCustomKey(key);
            return db.KeyRename(key, newKey);
        }

        /// <summary>
        /// 设置Key过期时间
        /// </summary>
        /// <param name="key">The key.</param>
        /// <param name="expiry">The expiry.</param>
        /// <returns></returns>
        public bool KeyExpire(string key, TimeSpan? expiry = default)
        {
            key = GenerateCustomKey(key);
            return db.KeyExpire(key, expiry);
        }

        /// <summary>
        /// Redis订阅
        /// </summary>
        /// <param name="subChannel">The sub channel.</param>
        /// <param name="handler">The handler.</param>
        public void Subscribe(string subChannel, Action<RedisChannel, RedisValue> handler = null)
        {
            ISubscriber sub = conn.GetSubscriber();
            sub.Subscribe(subChannel, (channel, message) =>
            {
                if (handler == null)
                {
                    Console.WriteLine(subChannel + " 订阅收到消息：" + message);
                }
                else
                {
                    handler(channel, message);
                }
            });
        }

        /// <summary>
        /// Redis发布
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="channel">The channel.</param>
        /// <param name="message">The message.</param>
        /// <returns></returns>
        public long Publish<T>(string channel, T message)
        {
            ISubscriber sub = conn.GetSubscriber();
            return sub.Publish(channel, ConvertToJson(message));
        }

        /// <summary>
        /// Redis取消订阅
        /// </summary>
        /// <param name="channel">The channel.</param>
        public void Unsubscribe(string channel)
        {
            ISubscriber sub = conn.GetSubscriber();
            sub.Unsubscribe(channel);
        }

        /// <summary>
        /// Redis取消全部订阅
        /// </summary>
        public void UnsubscribeAll()
        {
            ISubscriber sub = conn.GetSubscriber();
            sub.UnsubscribeAll();
        }

        /// <summary>
        /// 创建一个Redis事务
        /// </summary>
        /// <param name="dbSerialNumber">The database serial number.</param>
        /// <returns></returns>
        public ITransaction CreateTransaction(int dbSerialNumber)
        {
            return conn.GetDatabase(dbSerialNumber).CreateTransaction();
        }

        public async Task<bool> StringSetAsync(string key, string value, TimeSpan? expiry = default)
        {
            key = GenerateCustomKey(key);
            return await db.StringSetAsync(key, value, expiry);
        }

        public async Task<bool> StringSetAsync(List<KeyValuePair<RedisKey, RedisValue>> keyValues)
        {
            List<KeyValuePair<RedisKey, RedisValue>> newkeyValues = keyValues
                .Select(p => new KeyValuePair<RedisKey, RedisValue>(GenerateCustomKey(p.Key), p.Value))
                .ToList();
            return await db.StringSetAsync(newkeyValues.ToArray());
        }

        public async Task<bool> StringSetAsync<T>(string key, T @object, TimeSpan? expiry = default)
        {
            key = GenerateCustomKey(key);
            string json = ConvertToJson(@object);
            return await db.StringSetAsync(key, json, expiry);
        }

        public async Task<string> StringGetAsync(string key)
        {
            key = GenerateCustomKey(key);
            return await db.StringGetAsync(key);
        }

        public async Task<RedisValue[]> StringGetAsync(List<string> listKeys)
        {
            List<string> redisKeys = listKeys.Select(GenerateCustomKey).ToList();
            return await db.StringGetAsync(ConvertToRedisKeys(redisKeys));
        }

        public async Task<T> StringGetAsync<T>(string key)
        {
            key = GenerateCustomKey(key);
            return ConvertToObject<T>(await db.StringGetAsync(key));
        }

        public async Task<double> StringIncrementAsync(string key, double incrementVal = 1)
        {
            key = GenerateCustomKey(key);
            return await db.StringIncrementAsync(key, incrementVal);
        }

        public async Task<double> StringDecrementAsync(string key, double decrementVal = 1)
        {
            key = GenerateCustomKey(key);
            return await db.StringDecrementAsync(key, decrementVal);
        }

        public async Task<bool> HashExistsAsync(string key, string dataKey)
        {
            key = GenerateCustomKey(key);
            return await db.HashExistsAsync(key, dataKey);
        }

        public async Task<bool> HashSetAsync<T>(string key, string hashFieldName, T hashValue)
        {
            key = GenerateCustomKey(key);
            string json = ConvertToJson(hashValue);
            return await db.HashSetAsync(key, hashFieldName, json);
        }

        public async Task<bool> HashDeleteAsync(string key, string hashFieldName)
        {
            key = GenerateCustomKey(key);
            return await db.HashDeleteAsync(key, hashFieldName);
        }

        public async Task<long> HashDeleteAsync(string key, List<RedisValue> hashFieldNames)
        {
            key = GenerateCustomKey(key);
            return await db.HashDeleteAsync(key, hashFieldNames.ToArray());
        }

        public async Task<T> HashGetAsync<T>(string key, string hashFieldName)
        {
            key = GenerateCustomKey(key);
            string value = await db.HashGetAsync(key, hashFieldName);
            return ConvertToObject<T>(value);
        }

        public async Task<double> HashIncrementAsync(string key, string hashFieldName, double incrementVal = 1)
        {
            key = GenerateCustomKey(key);
            return await db.HashIncrementAsync(key, hashFieldName, incrementVal);
        }

        public async Task<double> HashDecrementAsync(string key, string hashFieldName, double decrementVal = 1)
        {
            key = GenerateCustomKey(key);
            return await db.HashDecrementAsync(key, hashFieldName, decrementVal);
        }

        public async Task<List<T>> HashKeysAsync<T>(string key)
        {
            key = GenerateCustomKey(key);
            RedisValue[] values = await db.HashKeysAsync(key);
            return ConvetToList<T>(values);
        }

        public async Task<long> ListRemoveAsync<T>(string key, T value)
        {
            key = GenerateCustomKey(key);
            return await db.ListRemoveAsync(key, ConvertToJson(value));
        }

        public async Task<List<T>> ListRangeAsync<T>(string key)
        {
            key = GenerateCustomKey(key);
            RedisValue[] values = await db.ListRangeAsync(key);
            return ConvetToList<T>(values);
        }

        public async Task<long> ListRightPushAsync<T>(string key, T value)
        {
            key = GenerateCustomKey(key);
            return await db.ListRightPushAsync(key, ConvertToJson(value));
        }

        public async Task<T> ListRightPopAsync<T>(string key)
        {
            key = GenerateCustomKey(key);
            RedisValue value = await db.ListRightPopAsync(key);
            return ConvertToObject<T>(value);
        }

        public async Task<long> ListLeftPushAsync<T>(string key, T value)
        {
            key = GenerateCustomKey(key);
            return await db.ListLeftPushAsync(key, ConvertToJson(value));
        }

        public async Task<T> ListLeftPopAsync<T>(string key)
        {
            key = GenerateCustomKey(key);
            var value = await db.ListLeftPopAsync(key);
            return ConvertToObject<T>(value);
        }

        public async Task<long> ListLengthAsync(string key)
        {
            key = GenerateCustomKey(key);
            return await db.ListLengthAsync(key);
        }

        public async Task<bool> SortedSetAddAsync<T>(string key, T value, double score)
        {
            key = GenerateCustomKey(key);
            return await db.SortedSetAddAsync(key, ConvertToJson<T>(value), score);
        }

        public async Task<bool> SortedSetRemoveAsync<T>(string key, T value)
        {
            key = GenerateCustomKey(key);
            return await db.SortedSetRemoveAsync(key, ConvertToJson(value));
        }

        public async Task<List<T>> SortedSetRangeByRankAsync<T>(string key, long start, long stop, Order order = Order.Ascending)
        {
            key = GenerateCustomKey(key);
            var values = await db.SortedSetRangeByRankAsync(key, start, stop, order);
            return ConvetToList<T>(values);
        }

        public async Task<long> SortedSetLengthAsync(string key)
        {
            key = GenerateCustomKey(key);
            return await db.SortedSetLengthAsync(key);
        }

        public async Task<bool> KeyDeleteAsync(string key)
        {
            key = GenerateCustomKey(key);
            return await db.KeyDeleteAsync(key);
        }

        public async Task<long> KeyDeleteAsync(List<string> keys)
        {
            List<string> newKeys = keys.Select(GenerateCustomKey).ToList();
            return await db.KeyDeleteAsync(ConvertToRedisKeys(newKeys));
        }

        public async Task<bool> KeyExistsAsync(string key)
        {
            key = GenerateCustomKey(key);
            return await db.KeyExistsAsync(key);
        }

        public async Task<bool> KeyRenameAsync(string key, string newKey)
        {
            key = GenerateCustomKey(key);
            return await db.KeyRenameAsync(key, newKey);
        }

        public async Task<bool> KeyExpireAsync(string key, TimeSpan? expiry = default)
        {
            key = GenerateCustomKey(key);
            return await db.KeyExpireAsync(key, expiry);
        }

        public async Task SubscribeAsync(string subChannel, Action<RedisChannel, RedisValue> handler = null)
        {
            ISubscriber sub = conn.GetSubscriber();
            await sub.SubscribeAsync(subChannel, (channel, message) =>
             {
                 if (handler == null)
                 {
                     Console.WriteLine(subChannel + " 订阅收到消息：" + message);
                 }
                 else
                 {
                     handler(channel, message);
                 }
             });
        }

        public async Task<long> PublishAsync<T>(string channel, T message)
        {
            ISubscriber sub = conn.GetSubscriber();
            return await sub.PublishAsync(channel, ConvertToJson(message));
        }

        public async Task UnsubscribeAsync(string channel)
        {
            ISubscriber sub = conn.GetSubscriber();
            await sub.UnsubscribeAsync(channel);
        }

        public async Task UnsubscribeAllAsync()
        {
            ISubscriber sub = conn.GetSubscriber();
            await sub.UnsubscribeAllAsync();
        }

        /// <summary>
        /// 生成自定义Key,格式(CustomKeyPrefix+"_"+Key)
        /// </summary>
        /// <param name="oldKey"></param>
        /// <returns></returns>
        private string GenerateCustomKey(string oldKey)
        {
            if (!string.IsNullOrWhiteSpace(CustomKeyPrefix))
            {
                return $"{CustomKeyPrefix}_{oldKey}";
            }
            return oldKey;
        }

        /// <summary>
        /// 对象序列化
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="value"></param>
        /// <returns></returns>
        private static string ConvertToJson<T>(T value)
        {
            return value is string ? value.ToString() : JsonConvert.SerializeObject(value);
        }

        /// <summary>
        /// RedisValue转Object
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="value"></param>
        /// <returns></returns>
        private static T ConvertToObject<T>(RedisValue value)
        {
            if (typeof(T).Name.Equals(typeof(string).Name))
            {
                return JsonConvert.DeserializeObject<T>($"'{value}'");
            }
            return JsonConvert.DeserializeObject<T>(value);
        }

        /// <summary>
        /// RedisValue转List
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="values">The values.</param>
        /// <returns></returns>
        private static List<T> ConvetToList<T>(RedisValue[] values)
        {
            List<T> result = new();
            foreach (RedisValue item in values)
            {
                T @object = ConvertToObject<T>(item);
                result.Add(@object);
            }
            return result;
        }

        /// <summary>
        /// 转换为RedisKey
        /// </summary>
        /// <param name="redisKeys"></param>
        /// <returns></returns>
        private static RedisKey[] ConvertToRedisKeys(List<string> redisKeys)
        {
            return redisKeys.Select(redisKey => (RedisKey)redisKey).ToArray();
        }
    }

    /// <summary>
    /// 单例获取RedisConnectionMultiplexer
    /// </summary>
    static class RedisConnectionMultiplexerHelper
    {
        /// <summary>
        /// Redis连接字符串
        /// </summary>
        public static string RedisConnectionString { get; set; }

        private static readonly object locker = new();

        /// <summary>
        /// ConnectionMultiplexer对象
        /// </summary>
        private static ConnectionMultiplexer connectionMultiplexerInstance;

        /// <summary>
        /// 单例获取
        /// </summary>
        public static ConnectionMultiplexer Instance
        {
            get
            {
                if (connectionMultiplexerInstance == null)
                {
                    lock (locker)
                    {
                        if (connectionMultiplexerInstance == null || !connectionMultiplexerInstance.IsConnected)
                        {
                            connectionMultiplexerInstance = GetConnectionMultiplexer();
                        }
                    }
                }
                return connectionMultiplexerInstance;
            }
        }

        private static ConnectionMultiplexer GetConnectionMultiplexer()
        {
            if (string.IsNullOrWhiteSpace(RedisConnectionString))
            {
                throw new Exception("RedisConnectionString IsNullOrWhiteSpace!");
            }
            var connect = ConnectionMultiplexer.Connect(RedisConnectionString);

            connect.ConnectionFailed += MuxerConnectionFailed;
            connect.ConnectionRestored += MuxerConnectionRestored;
            connect.ErrorMessage += MuxerErrorMessage;
            connect.ConfigurationChanged += MuxerConfigurationChanged;
            connect.HashSlotMoved += MuxerHashSlotMoved;
            connect.InternalError += MuxerInternalError;
            return connect;
        }

        /// <summary>
        /// 配置更改时
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private static void MuxerConfigurationChanged(object sender, EndPointEventArgs e)
        {
            Console.WriteLine($"Configuration changed: {e.EndPoint}");
        }

        /// <summary>
        /// 发生错误时
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private static void MuxerErrorMessage(object sender, RedisErrorEventArgs e)
        {
            Console.WriteLine($"ErrorMessage:{e.Message}");
        }

        /// <summary>
        /// 重新建立连接之前的错误
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private static void MuxerConnectionRestored(object sender, ConnectionFailedEventArgs e)
        {
            Console.WriteLine($"ConnectionRestored:{e.EndPoint}");
        }

        /// <summary>
        /// 连接失败,如果重新连接成功将不会收到这个通知
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private static void MuxerConnectionFailed(object sender, ConnectionFailedEventArgs e)
        {
            Console.WriteLine($"重新连接：Endpoint failed:{e.EndPoint},{e.FailureType},{e.Exception.Message}");
        }

        /// <summary>
        /// 更改集群
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private static void MuxerHashSlotMoved(object sender, HashSlotMovedEventArgs e)
        {
            Console.WriteLine($"HashSlotMoved:NewEndPoint={e.NewEndPoint},OldEndPoint={ e.OldEndPoint}");
        }

        /// <summary>
        /// Redis类库错误
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private static void MuxerInternalError(object sender, InternalErrorEventArgs e)
        {
            Console.WriteLine($"InternalError:Message={e.Exception.Message}");
        }
    }
}

```

# CSRedisCore

`csRedis` 对所有方法名称进行了调整，使其和 `redis-cli`保持一致，如果熟悉 `redis-cli` 的命令可以直接上手，学习成本降低很多。

## 安装

Github：https://github.com/2881099/csRedis

`csRedisCore` 是 C# 操作 `Redis` 数据库的另一个客户端。在 `NuGet` 中搜索 `csRedisCore` ，直接点击按钮安装即可。

## 初始化

使用连接字符串创建redis实例，可使用以下两种方式

1. 使用`RedisHelper` ： `RedisHelper.Initialization(new csRedisClient("127.0.0.1:6379"))` 
2. 使用`csRedisClient`：`csRedisClient var csRedis = new csRedisClient("127.0.0.1:6379");` 

然后就可以进行操作了。

## 字符串(string)

关于字符串的`value`：

- `value` 可以用来存储任意格式的数据，如 `json`、`jpg` 甚至视频文件
- `value` 最大容量为 `512M`
- `value` 可以存储3种类型的值：字节串（byte string）/整数（int）/浮点数（double）string / 对象

其中，整数的取值范围和系统的长整数取值范围相同。

- 在32位的操作系统上，整数就是32位的
- 在64位操作系统上，整数就是64位有符号整数
- 浮点数的取值范围和IEEE 754标准的双精度浮点数相同

## 简单操作

```csharp
/*    字符串操作键值对    */
csRedis.Set("name", "wangpengliang");
csRedis.Set("name2", "wangkaining");
csRedis.Set("name", "wangpengliang2");

// 根据键获取对应的值
csRedis.Get("name");

// 移除元素
csRedis.Del("name2");
```

在对同一个键多次赋值时，该键的值是最后一次赋值时的值，实例中 `hello` 对应的值最终为`wangpengliang2`。

由于 `Redis` 可以对字符串的类型进行“识别”，所以除了对字符串进行增、删、查、之外，我们还可以对整数类型进行自增、自减操作，对字节串的一部分进行读取或者写入。

```csharp
/*    数值操作    */
csRedis.Set("num-key", "24");

// value += 5
csRedis.IncrBy("num-key",5); 
// output -> 29

// value -= 10
csRedis.IncrBy("num-key", -10); 
// output -> 19
/*    字节串操作    */
csRedis.Set("string-key", "hello ");

// 在指定key的value末尾追加字符串
csRedis.Append("string-key", "world"); 
// output -> "hello world"

// 获取从指定范围所有字符构成的子串（start:3,end:7）
csRedis.GetRange("string-key",3,7)  
// output ->  "lo wo"
    
// 用新字符串从指定位置覆写原value（index:4）
csRedis.SetRange("string-key", 4, "aa"); 
// output -> "hellaaword"
```

## 非正常情况

- 对字节串进行自增、自减操作时，redis会报错
- 使用`Append`、`SetRange`方法对 `value` 写入时，字节串的长度可能不够用，这时redis会使用空字符(`null` ) 将 `value`扩充到指定长度，然后再进行写入操作

## 列表(List)

> The speed of adding a new element with the [LPUSH](https://redis.io/commands/lpush) command to the head of a list with ten elements is the same as adding an element to the head of list with 10 million elements.
>
> 译：使用`LPUSH`命令，向包含10个元素的列表添加新元素的速度等于向包含一千万个元素的列表添加新元素的速度。

- 列表可以有序的存储多个字符串操作（字符串可以重复）
- 列表是通过链表来实现的，所以它添加新元素的速度非常快

```csharp
// 从右端推入元素
csRedis.RPush("my-list", "item1", "item2", "item3", "item4"); 
// 从右端弹出元素
csRedis.RPop("my-list");
// 从左端推入元素
csRedis.LPush("my-list","LeftPushItem");
// 从左端弹出元素
csRedis.LPop("my-list");

// 遍历链表元素（start:0,end:-1即可返回所有元素）
foreach (var item in csRedis.LRange("my-list", 0, -1))
{
    Console.WriteLine(item);
}
// 按索引值获取元素（当索引值大于链表长度，返回空值，不会报错）
Console.WriteLine($"{csRedis.LIndex("my-list", 1)}"); 

// 修剪指定范围内的元素（start:4,end:10）
csRedis.LTrim("my-list", 4, 10);
```

除了对列表中的元素进行以上简单的处理之外，还可以将一个列表中的元素复制到另一个列表中。在语义上，列表的左端默认为“头部”，列表的右端为“尾部”。

```csharp
// 将my-list最后一个元素弹出并压入another-list的头部
csRedis.RPopLPush("my-list", "another-list");
```

## 集合(Set)

集合以无序的方式存储**各不相同**的元素，也就是说在集合中的每个元素的`Key`都不重复。在 `Redis`中可以快速地对集合执行添加、移除等操作。

```csharp
// 实际上只插入了两个元素("item1","item2")
csRedis.SAdd("my-set", "item1", "item1", "item2"); 

// 集合的遍历
foreach (var member in csRedis.SMembers("my-set"))
{
    Console.WriteLine($"集合成员：{member.ToString()}");
}

// 判断元素是否存在
string member = "item1";
Console.WriteLine($"{member}是否存在:{csRedis.SIsMember("my-set", member)}"); 
// output -> True

// 移除元素
csRedis.SRem("my-set", member);
Console.WriteLine($"{member}是否存在:{csRedis.SIsMember("my-set", member)}"); 
// output ->  False

// 随机移除一个元素
csRedis.SPop("my-set");
```

以上是对一个集合中的元素进行操作，除此之外还可以对两个集合进行交、并、差操作。

```csharp
csRedis.SAdd("set-a", "item1", "item2", "item3","item4","item5");
csRedis.SAdd("set-b", "item2", "item5", "item6", "item7");

// 差集
csRedis.SDiff("set-a", "set-b"); 
// output -> "item1", "item3","item4"

// 交集
csRedis.SInter("set-a", "set-b"); 
// output -> "item2","item5"

// 并集
csRedis.SUnion("set-a", "set-b");
// output -> "item1","item2","item3","item4","item5","item6","item7"
```

另外还可以用 `SDiffStore`,`SInterStore`,`SUnionStore` 将操作后的结果存储在新的集合中。

## 散列(HashMap)

在 `Redis` 中可以使用散列将多个键值对存储在一个 `key` 上，从而达到将一系列相关数据存放在一起的目的。

```csharp
// 向散列添加元素
csRedis.HSet("ArticleID:10001", "Title", "了解简单的Redis数据结构");
csRedis.HSet("ArticleID:10001", "Author", "xscape");
csRedis.HSet("ArticleID:10001", "PublishTime", "2019-01-01");

// 根据Key获取散列中的元素
csRedis.HGet("ArticleID:10001", "Title");

// 获取散列中的所有元素
foreach (var item in csRedis.HGetAll("ArticleID:10001"))
{
    Console.WriteLine(item.Value);
}
```

`HGet`和`HSet`方法执行一次只能处理一个键值对，而`HMGet`和`HMSet`是他们的多参数版本，一次可以处理多个键值对。

```csharp
var keys = new string[] { "Title","Author","publishTime"};
csRedis.HMGet("ID:10001", keys);
```

虽然使用`HGetAll`可以取出所有的value，但是**有时候散列包含的值可能非常大，容易造成服务器的堵塞**，为了避免这种情况，我们可以使用`HKeys`取到散列的所有键(*`HVals可以取出所有值`*)，然后再使用 `HGet`方法一个一个地取出键对应的值。

```csharp
foreach (var item in csRedis.HKeys("ID:10001"))
{
	Console.WriteLine($"{item} - {csRedis.HGet("ID:10001", item)}");
}
```

和处理字符串一样，也可以对散列中的值进行自增、自减操作，原理同字符串是一样的。

```csharp
csRedis.HSet("ArticleID:10001", "votes", "257");
csRedis.HIncrBy("ID:10001", "votes", 40);
// output -> 297
```

## 有序集合

有序集合可以看作是可排序的散列，不过有序集合的 `val` 成为`score` 分值，集合内的元素就是基于`score` 进行排序的，`score` 以双精度浮点数的格式存储。

```csharp
// 向有序集合添加元素
csRedis.ZAdd("Quiz", (79, "Math"));
csRedis.ZAdd("Quiz", (98, "English"));
csRedis.ZAdd("Quiz", (87, "Algorithm"));
csRedis.ZAdd("Quiz", (84, "Database"));
csRedis.ZAdd("Quiz", (59, "Operation System"));

//返回集合中的元素数量
csRedis.ZCard("Quiz");

// 获取集合中指定范围(90~100)的元素集合
csRedis.ZRangeByScore("Quiz",90,100);

// 获取集合所有元素并升序排序
csRedis.ZRangeWithScores("Quiz", 0, -1);

// 移除集合中的元素
csRedis.ZRem("Quiz", "Math");
```

## 事务

事务可以保证一个客户端在执行多条命令时，不被其他客户端打断，这跟关系型数据库的事务是不一样的。事务需要使用 `MULTI` 和 `EXEC` 命令，`Redis` 会将被`MULTI` 和 `EXEC`所包围的代码依次执行，当该事务结束之后才会处理其他客户端的命令。

**管道(pipeline)**

`Redis` 的事务是通过 `pipeline` 实现的，使用时客户端会自动调用 `MULTI` 和 `EXEX` 命令，**将多条命令打包并一次性地发送给 `Redis`，然后 `Redis`再将命令的执行结果全部打包并一次性返回给客户端**，这样有效的减少了Redis与客户端的通信次数，提升执行多次命令时的性能。

```csharp
 /// <summary>
        /// 事务处理
        /// </summary>
 [TestMethod]
 public void Transaction()
        {
            string strKey = Guid.NewGuid().ToString();
            string strval = Faker.Name.FullName();

            string intKey = Guid.NewGuid().ToString();
            int intVal = Faker.RandomNumber.Next(1, 10);

#if one
            // 第一种方式:正常情况下redis命令批处理,返回[true,true]
            object[] result = csRedis.StartPipe()
                .Set(strKey, strval)
                .Set(intKey, intVal)
                .EndPipe();
            Console.WriteLine(JsonConvert.SerializeObject(result));
#endif

#if two
            // 第二种方式：对string类型进行IncrBy操作,会抛出异常但不影响执行,返回[false, false, null]; Redis中多了两条数据
            object[] result2 = csRedis.StartPipe()
                .Set(strKey, strval)
                .Set(intKey, intVal)
                .IncrBy(strKey, 5)
                .EndPipe();
            Console.WriteLine(JsonConvert.SerializeObject(result2));
#endif

#if three
            // 第三种方式：对string类型进行IncrBy操作,错误的命令不被执行,Redis中多了两条数据
            using var rc = csRedis.Nodes.First().Value.Get();
            rc.Value.Multi();
            rc.Value.SAdd(strKey, strval);
            rc.Value.SAdd(intKey, intVal);
            rc.Value.SAdd(strKey, 5);
            // 此时报错：EXEC  ---> CSRedis.RedisException: WRONGTYPE Operation against a key holding the wrong kind of value
            rc.Value.Exec();
#endif
#if four
            // 第四种方式：对string类型进行IncrBy操作,错误的命令不被执行,Redis中多了两条数据
            var tran = csRedis.StartPipe();
            tran.Set(strKey, strval);
            tran.Set(intKey, intVal);
            tran.IncrBy(strKey, 5);
            var b = tran.EndPipe();
#endif
        }
```

## Key的过期

Redis允许为key设置有效期，当key过期之后，key就不存在了。

```csharp
[TestMethod]
public void KeyExpire()
{
    string key = "name";
    string value = "wangpengliang";
    csRedis.Set(key, value);

    csRedis.Expire(key, 3);
    string result = csRedis.Get(key);
    Assert.AreEqual(value, result);
    Thread.Sleep(4000);
    result = csRedis.Get(key);
    Assert.AreEqual(null, result);
}
```

## 分区

多个服务节点共同分担存储，与官方的分区、集群、高可用方案不同。

> 例如：缓存数据达到500G，如果使用一台redis-server服务器光靠内存存储将非常吃力，使用硬盘又影响性能。
> 可以使用此功能自动管理N台redis-server服务器分担存储，每台服务器只需约 (500/N)G 内存，且每台服务器匀可以配置官方高可用架构。

```
var rds = new CSRedis.CSRedisClient(null,
  "127.0.0.1:6371,password=123,defaultDatabase=11,poolsize=10,ssl=false,writeBuffer=10240,prefix=key前辍", 
  "127.0.0.1:6372,password=123,defaultDatabase=12,poolsize=11,ssl=false,writeBuffer=10240,prefix=key前辍",
  "127.0.0.1:6373,password=123,defaultDatabase=13,poolsize=12,ssl=false,writeBuffer=10240,prefix=key前辍",
  "127.0.0.1:6374,password=123,defaultDatabase=14,poolsize=13,ssl=false,writeBuffer=10240,prefix=key前辍");
//实现思路：根据key.GetHashCode() % 节点总数量，确定连向的节点
//也可以自定义规则(第一个参数设置)

rds.MSet("key1", 1, "key2", 2, "key3", 3, "key4", 4);
rds.MGet("key1", "key2", "key3", "key4");
```

## 发布订阅

```
//普通订阅
rds.Subscribe(
  ("chan1", msg => Console.WriteLine(msg.Body)),
  ("chan2", msg => Console.WriteLine(msg.Body)));

//模式订阅（通配符）
rds.PSubscribe(new[] { "test*", "*test001", "test*002" }, msg => {
  Console.WriteLine($"PSUB   {msg.MessageId}:{msg.Body}    {msg.Pattern}: chan:{msg.Channel}");
});
//模式订阅已经解决的难题：
//1、分区的节点匹配规则，导致通配符最大可能匹配全部节点，所以全部节点都要订阅
//2、本组 "test*", "*test001", "test*002" 订阅全部节点时，需要解决同一条消息不可执行多次

//发布
rds.Publish("chan1", "123123123");
//无论是分区或普通模式，rds.Publish 都可以正常通信
```

## 缓存壳

```
        /// <summary>
        /// 缓存壳
        /// </summary>
        [TestMethod]
        public void CacheShell()
        {
            string value = "wangpengliang";
#if debug
            // 一般的缓存代码，如不封装比较繁琐
            var cacheValue = csRedis.Get("name");
            // 如果已被缓存
            if (!string.IsNullOrEmpty(cacheValue))
            {
                try
                {
                    // 
                }
                catch
                {
                    //出错时删除key
                    csRedis.Del("name");
                    throw;
                }
            }
            else
            {
                csRedis.Set("name", value, 10);
            }
#endif
            // 判断key=name是否已存在,存在返回value,不存在设置
            string t1 = csRedis.CacheShell("name", 10, () => value);
            Assert.AreEqual(value, t1);
            string t2 = csRedis.CacheShell("name", 10, () => "wangpengliang2");
            Assert.AreEqual(value, t2);
            string t3 = csRedis.CacheShell("name2", 10, () => "wangpengliang2");
            Assert.AreEqual("wangpengliang2", t3);
        }
```

## 管道

使用管道模式，打包多条命令一起执行，从而提高性能。

```
var ret1 = rds.StartPipe().Set("a", "1").Get("a").EndPipe();
var ret2 = rds.StartPipe(p => p.Set("a", "1").Get("a"));

var ret3 = rds.StartPipe().Get("b").Get("a").Get("a").EndPipe();
//与 rds.MGet("b", "a", "a") 性能相比，经测试差之毫厘
```

## 多数据库

> 如果确定一定以及肯定非要有切换数据库的需求，请看以下代码：通过定义多个CSRedisClient实现

```
/// <summary>
/// 多数据库,使用多个CSRedisClient实现
/// </summary>
[TestMethod]
public void MultiDatabase()
{
    // 实际使用必须要单例
    CSRedisClient[] redis = new CSRedisClient[14];
    for (int i = 0; i < redis.Length; i++)
    {
        redis[i] = new CSRedisClient($"{redisConnection},defaultDatabase={i}");
    }

    redis[0].Set("db0", "db0");
    string t1 = redis[0].Get("db0");
    Assert.AreEqual("db0", t1);


    redis[1].Set("db1", "db1");
    string t2 = redis[1].Get("db1");
    Assert.AreEqual("db1", t2);

    redis[2].Set("db2", "db2");
    string t3 = redis[2].Get("db2");
    Assert.AreEqual("db2", t3);
}
```