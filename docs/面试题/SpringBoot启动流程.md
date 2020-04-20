

### SpringApplication

```java
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

逐步分析：

### SpringApplication.run(Application.class, args);

```java
	public static ConfigurableApplicationContext run(Object source, String... args) {
		return run(new Object[] { source }, args);
	}
    public static ConfigurableApplicationContext run(Object[] sources, String[] args) {
		return new SpringApplication(sources).run(args);
	}
```

以上可以看到new一个SpringApplication然后run

### new SpringApplication(sources)

```java
	public SpringApplication(Object... sources) {
		initialize(sources);// source = Application.class
	}
	private void initialize(Object[] sources) {
		if (sources != null && sources.length > 0) {
            // 将Application.class赋值给变量sources
			this.sources.addAll(Arrays.asList(sources));
		}
        // 这是一个boolean，判断是否是web环境
		this.webEnvironment = deduceWebEnvironment();
        // 拿到classpath下所有的ApplicationContextInitializer的实现类，赋值给initializers变量
		setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
        // 拿到classpath下所有的ApplicationListener的实现类，赋值给listeners变量
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
        // 将main方法所在的类赋值给mainApplicationClass，这个方法采用获取堆栈的方式，可以学习下
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

```java
	private static final String[] WEB_ENVIRONMENT_CLASSES = { "javax.servlet.Servlet",
			"org.springframework.web.context.ConfigurableWebApplicationContext" };
	// 判断是否是web环境
	private boolean deduceWebEnvironment() {
        // 如果不存在上述两个就不是web环境，原因是ConfigurableWebApplicationContext这个类在spring-web包下，如果没有引入spring-boot-web-starter就不会有这个包，有点自动化配置的意思了
		for (String className : WEB_ENVIRONMENT_CLASSES) {
			if (!ClassUtils.isPresent(className, null)) {
				return false;
			}
		}
		return true;
	}
```

```java
	private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type) {
		return getSpringFactoriesInstances(type, new Class<?>[] {});
	}
	// type表示要加载的类，parameterTypes为构造参数的类型，args为构造参数
	private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
		// 使用appclassload去加载type的实现类，因为一般传入的type都是接口
		Set<String> names = new LinkedHashSet<String>(
				SpringFactoriesLoader.loadFactoryNames(type, classLoader));
        // 对获取的实现类实例化，并排序
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
				classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
```

```java
	// 自动化配置的核心方法
	// factoryClasss是接口
	public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
		String factoryClassName = factoryClass.getName();
		try {
            // 通过传入接口的全限定类名去配置文件中获取相应的值，获取到的值就是他的实现类
            // 这里会将所有包下的/META-INF/spring.factories文件全部拿到
			Enumeration<URL> urls = (classLoader != null ? classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			List<String> result = new ArrayList<String>();
            // 循环遍历这些文件取出key为factoryClass的值，返回
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
				String factoryClassNames = properties.getProperty(factoryClassName);
				result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
			}
			return result;
		}
		catch (IOException ex) {
		}
	}
```

至此SpringApplication对象已经构建完成，之后调用run方法

### run(args)

```java
	public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();// 这个类似锁，但没有使用真正的锁，他保证了初始化的时候只会有一个线程在进行，同时对初始化的次数和时间进行保存
		stopWatch.start();
		ConfigurableApplicationContext context = null;// 上下文
		FailureAnalyzers analyzers = null;
        // 配置系统变量java.awt.headless为true
		configureHeadlessProperty();
        // 加载classpath下SpringApplicationRunListener的实现类并实例化最后封装到SpringApplicationRunListeners中，其实就是个监听器能统一调用的类
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();// 调用监听器的starting方法，至于里面的实现类下面分析
		try {
            // 将参数封装起来，一般使用不会有参数，有空在分析吧
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
            // 环境封装
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
            // 打印日志，就是每次启动都会有springboot的字样
			Banner printedBanner = printBanner(environment);
            // 创建上下文
			context = createApplicationContext();
            // 失败分析器
			analyzers = new FailureAnalyzers(context);
            // 准备上下文
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner);
            // 刷新上下文
			refreshContext(context);
            // 刷新上下文之后
			afterRefresh(context, applicationArguments);
			listeners.finished(context, null);// 触发监听器
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass)
						.logStarted(getApplicationLog(), stopWatch);
			}
			return context;
		}
		catch (Throwable ex) {
			handleRunFailure(context, listeners, analyzers, ex);
			throw new IllegalStateException(ex);
		}
	}
```

### 参数封装

```java
	private final Source source;

	private final String[] args;

	public DefaultApplicationArguments(String[] args) {
		Assert.notNull(args, "Args must not be null");
		this.source = new Source(args);
		this.args = args;
	}
	Source(String[] args) {
        super(args);
    }
	public SimpleCommandLinePropertySource(String... args) {
		super(new SimpleCommandLineArgsParser().parse(args));
	}
	// 这里就是解析参数，将--开头的参数以key-value的形式保存，否则就保存value
	public CommandLineArgs parse(String... args) {
		CommandLineArgs commandLineArgs = new CommandLineArgs();
		for (String arg : args) {
			if (arg.startsWith("--")) {
				String optionText = arg.substring(2, arg.length());
				String optionName;
				String optionValue = null;
				if (optionText.contains("=")) {
					optionName = optionText.substring(0, optionText.indexOf("="));
					optionValue = optionText.substring(optionText.indexOf("=")+1, optionText.length());
				}
				else {
					optionName = optionText;
				}
				if (optionName.isEmpty() || (optionValue != null && optionValue.isEmpty())) {
					throw new IllegalArgumentException("Invalid argument syntax: " + arg);
				}
				commandLineArgs.addOptionArg(optionName, optionValue);
			}
			else {
				commandLineArgs.addNonOptionArg(arg);
			}
		}
		return commandLineArgs;
	}
	// COMMAND_LINE_PROPERTY_SOURCE_NAME = commandLineArgs
	public CommandLinePropertySource(T source) {
		super(COMMAND_LINE_PROPERTY_SOURCE_NAME, source);
	}// 在往上就不复制了
```

经过上述逻辑参数的结构是这样的

```json
DefaultApplicationArguments : {
	source : {
		commandLineArgs : {
			optionArgs : Map<String, List>,// 这里就是保存--开头的参数的，list是因为参数可能重复
			nonOptionArgs : List// 这里就是保存非--开头的参数
		}
	},
	args : args//原参数数据
}

```



### 环境准备

```java
	private ConfigurableEnvironment prepareEnvironment(
			SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments) {
		// Create and configure the environment
		ConfigurableEnvironment environment = getOrCreateEnvironment();
        // 配置环境
		configureEnvironment(environment, applicationArguments.getSourceArgs());
		listeners.environmentPrepared(environment);// 触发监听器
		if (!this.webEnvironment) {// 这里就不管了，不是web环境
			environment = new EnvironmentConverter(getClassLoader())
					.convertToStandardEnvironmentIfNecessary(environment);
		}
		return environment;
	}
	private ConfigurableEnvironment getOrCreateEnvironment() {
		if (this.environment != null) {
			return this.environment;
		}
		if (this.webEnvironment) {// 由于是web环境，就直接返回StandardServletEnvironment
			return new StandardServletEnvironment();
		}
		return new StandardEnvironment();
	}
```

每次看这个环境也不知道这个东西到底能干啥，今天就看看这个environment能干啥，分析StandardEnvironment的父类，其实其内部就维护了四个变量

```java
	// 活跃的profile
	private final Set<String> activeProfiles = new LinkedHashSet<String>();
	// 默认的profile，这个里面就是一个default字符串
	private final Set<String> defaultProfiles = new LinkedHashSet<String>(getReservedDefaultProfiles());
	// MutablePropertySources里面维护了propertySourceList，propertySource其实就是source的父类由此可以想象到他就是保存参数的
	private final MutablePropertySources propertySources = new MutablePropertySources(this.logger);
	// 就是对propertySources的一个封装
	private final ConfigurablePropertyResolver propertyResolver =
			new PropertySourcesPropertyResolver(this.propertySources);
```

配置环境，看到上述对环境解析，下面的方法就不难理解了

```java
	protected void configureEnvironment(ConfigurableEnvironment environment,
			String[] args) {
		configurePropertySources(environment, args);
		configureProfiles(environment, args);
	}
	protected void configurePropertySources(ConfigurableEnvironment environment,
			String[] args) {
		MutablePropertySources sources = environment.getPropertySources();
		if (this.defaultProperties != null && !this.defaultProperties.isEmpty()) {
			sources.addLast(
					new MapPropertySource("defaultProperties", this.defaultProperties));
		}
		if (this.addCommandLineProperties && args.length > 0) {
			String name = CommandLinePropertySource.COMMAND_LINE_PROPERTY_SOURCE_NAME;
			if (sources.contains(name)) {
				PropertySource<?> source = sources.get(name);
				CompositePropertySource composite = new CompositePropertySource(name);
				composite.addPropertySource(new SimpleCommandLinePropertySource(
						name + "-" + args.hashCode(), args));
				composite.addPropertySource(source);
				sources.replace(name, composite);
			}
			else {
				sources.addFirst(new SimpleCommandLinePropertySource(args));
			}
		}
	}
	protected void configureProfiles(ConfigurableEnvironment environment, String[] args) {
		environment.getActiveProfiles(); // ensure they are initialized
		// But these ones should go first (last wins in a property key clash)
		Set<String> profiles = new LinkedHashSet<String>(this.additionalProfiles);
		profiles.addAll(Arrays.asList(environment.getActiveProfiles()));
		environment.setActiveProfiles(profiles.toArray(new String[profiles.size()]));
	}
```

### 创建上下文

```java
	public static final String DEFAULT_CONTEXT_CLASS = "org.springframework.context."
			+ "annotation.AnnotationConfigApplicationContext";
	public static final String DEFAULT_WEB_CONTEXT_CLASS = "org.springframework."
			+ "boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext";
	protected ConfigurableApplicationContext createApplicationContext() {
		Class<?> contextClass = this.applicationContextClass;
		if (contextClass == null) {
			try {
				contextClass = Class.forName(this.webEnvironment
						? DEFAULT_WEB_CONTEXT_CLASS : DEFAULT_CONTEXT_CLASS);
			}
			catch (ClassNotFoundException ex) {
			}
		}
		return (ConfigurableApplicationContext) BeanUtils.instantiate(contextClass);
	}
```

由于我们是web环境，所以context就是AnnotationConfigApplicationContext

### 准备上下文

```java
	private void prepareContext(ConfigurableApplicationContext context,
			ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments, Banner printedBanner) {
		context.setEnvironment(environment);// 配置环境
		postProcessApplicationContext(context);// 设置beanname解析器和资源加载器，但是都没用
		applyInitializers(context);// 调用上面实例化的ApplicationContextInitializer的initialize方法
		listeners.contextPrepared(context);// 触发监听器
		if (this.logStartupInfo) {// 打印日志
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}

		// 将上面封装好的参数和日志输出器添加到容器中
		context.getBeanFactory().registerSingleton("springApplicationArguments",
				applicationArguments);
		if (printedBanner != null) {
			context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
		}

		// Load the sources
		Set<Object> sources = getSources();
		Assert.notEmpty(sources, "Sources must not be empty");
		load(context, sources.toArray(new Object[sources.size()]));
		listeners.contextLoaded(context);// 触发监听器
	}
```

```java
	protected void load(ApplicationContext context, Object[] sources) {
        // 创建beandefinition加载器，并将资源加载进去，资源就是sources也就是Application.class
		BeanDefinitionLoader loader = createBeanDefinitionLoader(
				getBeanDefinitionRegistry(context), sources);
		if (this.beanNameGenerator != null) {
			loader.setBeanNameGenerator(this.beanNameGenerator);
		}
		if (this.resourceLoader != null) {
			loader.setResourceLoader(this.resourceLoader);
		}
		if (this.environment != null) {
			loader.setEnvironment(this.environment);
		}
        // 加载资源
		loader.load();
	}
```

### 刷新上下文

```java
    private void refreshContext(ConfigurableApplicationContext context) {
        refresh(context);
        if (this.registerShutdownHook) {
            try {
                context.registerShutdownHook();
            }
            catch (AccessControlException ex) {
                // Not allowed in some environments.
            }
        }
    }
	protected void refresh(ApplicationContext applicationContext) {
		Assert.isInstanceOf(AbstractApplicationContext.class, applicationContext);
        // 这里还是调用AbstractApplicationContext的refresh方法
		((AbstractApplicationContext) applicationContext).refresh();
	}
```
### 刷新上下文之后

```java
    protected void afterRefresh(ConfigurableApplicationContext context,
            ApplicationArguments args) {
        callRunners(context, args);
    }
	private void callRunners(ApplicationContext context, ApplicationArguments args) {
		List<Object> runners = new ArrayList<Object>();
		runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
		runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
		AnnotationAwareOrderComparator.sort(runners);
		for (Object runner : new LinkedHashSet<Object>(runners)) {
			if (runner instanceof ApplicationRunner) {
				callRunner((ApplicationRunner) runner, args);
			}
			if (runner instanceof CommandLineRunner) {
				callRunner((CommandLineRunner) runner, args);
			}
		}
	}
	private void callRunner(ApplicationRunner runner, ApplicationArguments args) {
		try {
			(runner).run(args);
		}
		catch (Exception ex) {
			throw new IllegalStateException("Failed to execute ApplicationRunner", ex);
		}
	}

	private void callRunner(CommandLineRunner runner, ApplicationArguments args) {
		try {
			(runner).run(args.getSourceArgs());
		}
		catch (Exception ex) {
			throw new IllegalStateException("Failed to execute CommandLineRunner", ex);
		}
	}
```
上述可以看到刷新上下文之后就是调用容器中的ApplicationRunner和CommandLineRunner的run方法，这里可以用来扩展