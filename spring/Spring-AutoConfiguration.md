# AutoConfiguration 설정과 원리

RedisAutoConfiguration로 이해하는 AutoConfiguration 

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate")
	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
		RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean
	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
			throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

}
```

### @ConditionalOnMissingBean

특정 빈이 사전에 생성되지 않는 경우 조건이 만족됨.

@Bean과 함께 사용된다면 이미 생성된 빈이 없을 때 해당 빈을 생성한다는 의미로 보면 된다.

-> @ConditionalOnMissingBean+@Bean 의 조합일때

사용자가 특정 빈을 생성하도록 설정해놨다면, 일반적으로 AutoConfiguration의 생성 순서가 마지막에 오도록 AutoConfiguration이 잘 짜여있기 때문에 우리가 설정한 bean이 먼저 생성되고 해당 AutoConfiguration은 필터링 되어 중복생성되는 상황을 막는다.

반대고 사용자가 해당 빈을 설정하지 않았다면 AutoConfiguration에서는 해당 빈을 자동생성하게 된다.



### @ConditionalOnClass

classPath에 특정 클래스가 존재할 때만 조건이 만족된다.

작업을 위해 필수적으로 필요한 의존성이 등록되어 있는지 체크할 때 사용할 수 있다.

RedisAutoConfiguration 클래스의 경우 RedisOperations 파일이 classPath에 존재할 때 조건이 만족하여 RedisAutoConfiguration이 동작하게 된다

### @ConditionalOnBean

RedisAutoConfiguration에는 해당 어노테이션이 사용되지 않았지만 정리해봄.

특정 빈이 이미 생성되어 있는 경우에만 조건이 만족한다. 작업을 위해 필수적으로 필요한 빈이 미리 생성되어 있는지 체크할 때 사용할 수 있다.

예를들어 JdbcTemplate를 생성하기 위해 DataSource가 필요하다.

```java
@Bean 
@ConditionalOnBean(name={"dataSource"}) 
public JdbcTemplate jdbcTemplate(DataSource dataSource) {
  returnnew JdbcTemplate(dataSource); 
}
```

dataSource라고 정의된 빈이 존재할 때만 JdbcTemplate 빈을 생성한다. 만약에 dataSource가 존재하지 않는다면 JdbcTemplate을 만들 수도 없을 뿐더러 만들 필요가 없기 때문에 AutoConfiguration과정에서 JdbcTemplate을 빈으로 생성하지 않는다.

### @EnableConfigurationProperties

RedisProperties를 이용해서 관련 프로퍼티 정보를 읽어온다.

### @Configuration(proxyBeanMethods = false)

- lite mode 적용 여부 (false: lite mode 적용,cglib 사용하지 않는 상태에서 설정이 진행됨)
- lite mode란? : http://wonwoo.ml/index.php/post/2000



### @Import

설정 상속(?) 어노테이션

상속(extends)을 사용해도 되지만 다중 상속이 불가능할 뿐만 아니라

설정 정보 중에 오버라이드 하면 안되는 정보도 있어 final을 메서드에 붙이면 에러가 난다고 한다.

RedisAutoConfiguration 클래스는 LettuceConnectionConfiguration와 JedisConnectionConfiguration 설정정보를 상속받는다고 할 수 있다.



## LettuceConnectionConfiguration.java

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisClient.class)
class LettuceConnectionConfiguration extends RedisConnectionConfiguration {

	LettuceConnectionConfiguration(RedisProperties properties,
			ObjectProvider<RedisSentinelConfiguration> sentinelConfigurationProvider,
			ObjectProvider<RedisClusterConfiguration> clusterConfigurationProvider) {
		super(properties, sentinelConfigurationProvider, clusterConfigurationProvider);
	}

	@Bean(destroyMethod = "shutdown")
	@ConditionalOnMissingBean(ClientResources.class)
	DefaultClientResources lettuceClientResources() {
		return DefaultClientResources.create();
	}

	@Bean
	@ConditionalOnMissingBean(RedisConnectionFactory.class)
	LettuceConnectionFactory redisConnectionFactory(
			ObjectProvider<LettuceClientConfigurationBuilderCustomizer> builderCustomizers,
			ClientResources clientResources) throws UnknownHostException {
		LettuceClientConfiguration clientConfig = getLettuceClientConfiguration(builderCustomizers, clientResources,
				getProperties().getLettuce().getPool());
		return createLettuceConnectionFactory(clientConfig);
	}

	private LettuceConnectionFactory createLettuceConnectionFactory(LettuceClientConfiguration clientConfiguration) {
		if (getSentinelConfig() != null) {
			return new LettuceConnectionFactory(getSentinelConfig(), clientConfiguration);
		}
		if (getClusterConfiguration() != null) {
			return new LettuceConnectionFactory(getClusterConfiguration(), clientConfiguration);
		}
		return new LettuceConnectionFactory(getStandaloneConfig(), clientConfiguration);
	}

	private LettuceClientConfiguration getLettuceClientConfiguration(
			ObjectProvider<LettuceClientConfigurationBuilderCustomizer> builderCustomizers,
			ClientResources clientResources, Pool pool) {
		LettuceClientConfigurationBuilder builder = createBuilder(pool);
		applyProperties(builder);
		if (StringUtils.hasText(getProperties().getUrl())) {
			customizeConfigurationFromUrl(builder);
		}
		builder.clientResources(clientResources);
		builderCustomizers.orderedStream().forEach((customizer) -> customizer.customize(builder));
		return builder.build();
	}

	private LettuceClientConfigurationBuilder createBuilder(Pool pool) {
		if (pool == null) {
			return LettuceClientConfiguration.builder();
		}
		return new PoolBuilderFactory().createBuilder(pool);
	}

	private LettuceClientConfigurationBuilder applyProperties(
			LettuceClientConfiguration.LettuceClientConfigurationBuilder builder) {
		if (getProperties().isSsl()) {
			builder.useSsl();
		}
		if (getProperties().getTimeout() != null) {
			builder.commandTimeout(getProperties().getTimeout());
		}
		if (getProperties().getLettuce() != null) {
			RedisProperties.Lettuce lettuce = getProperties().getLettuce();
			if (lettuce.getShutdownTimeout() != null && !lettuce.getShutdownTimeout().isZero()) {
				builder.shutdownTimeout(getProperties().getLettuce().getShutdownTimeout());
			}
		}
		if (StringUtils.hasText(getProperties().getClientName())) {
			builder.clientName(getProperties().getClientName());
		}
		return builder;
	}

	private void customizeConfigurationFromUrl(LettuceClientConfiguration.LettuceClientConfigurationBuilder builder) {
		ConnectionInfo connectionInfo = parseUrl(getProperties().getUrl());
		if (connectionInfo.isUseSsl()) {
			builder.useSsl();
		}
	}

	/**
	 * Inner class to allow optional commons-pool2 dependency.
	 */
	private static class PoolBuilderFactory {

		LettuceClientConfigurationBuilder createBuilder(Pool properties) {
			return LettucePoolingClientConfiguration.builder().poolConfig(getPoolConfig(properties));
		}

		private GenericObjectPoolConfig<?> getPoolConfig(Pool properties) {
			GenericObjectPoolConfig<?> config = new GenericObjectPoolConfig<>();
			config.setMaxTotal(properties.getMaxActive());
			config.setMaxIdle(properties.getMaxIdle());
			config.setMinIdle(properties.getMinIdle());
			if (properties.getTimeBetweenEvictionRuns() != null) {
				config.setTimeBetweenEvictionRunsMillis(properties.getTimeBetweenEvictionRuns().toMillis());
			}
			if (properties.getMaxWait() != null) {
				config.setMaxWaitMillis(properties.getMaxWait().toMillis());
			}
			return config;
		}

	}

}
```

