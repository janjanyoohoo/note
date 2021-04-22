# 管道



```java

@Autowired
	RedisTemplate<Object, Object> redis;
	@GetMapping("/redisTest")
	@ResponseBody
	public String test() {
		RedisUtil redisUtil = new RedisUtil(redis);
		int number = 5;
		Long start = System.currentTimeMillis();
		for (int i = 0; i < number; ++i) {
			redisUtil.set(("T" + i), (i + ""));
		}
		Long end = System.currentTimeMillis();
		System.out.println("--" + (end - start));
 
		// 1.executePipelined 重写 入参 RedisCallback 的doInRedis方法
		List<Object> resultList = redis.executePipelined(new RedisCallback<Object>() {
 
			@Override
			public String doInRedis(RedisConnection connection) throws DataAccessException {
				// 2.connection 打开管道
				connection.openPipeline();
				// 3.connection 给本次管道内添加 要一次性执行的多条命令
				for (int i = 0; i < number; ++i) {
					connection.set(("B" + i).getBytes(), (i + "").getBytes());
				}
				// 4.关闭管道 不需要close 否则拿不到返回值
				// connection.closePipeline();
				// 这里一定要返回null，最终pipeline的执行结果，才会返回给最外层
				return null;
			}
		});
		Long start1 = System.currentTimeMillis();
		System.out.println("---" + (start1 - end));
		// 5.最后对redis pipeline管道操作返回结果进行判断和业务补偿
		for (Object object : resultList) {
			System.out.println(object);
		}
		return "test:" + (end - start) + ":" + (start1 - end);
```

