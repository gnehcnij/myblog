# Bean的创建

## 一、使用@Autowire、@Resource注解进行属性注入

**相关类：**

```java
@Service
public class AService {

	public AService() {
		System.out.println("AService constructor");
	}
}

@Service
public class BService {

	public BService() {
		System.out.println("BService constructor");
	}
}

@Service
public class BookService {
    
    @Autowired
    private AService aService;
    
    @Resource
    private BService bService;
    
    public BookService() {
		System.out.println("BookService constructor");
	}
    
}

// 配置类
@Configuration
@ComponentScan(value = "com.gnehcnij.service")
public class CircularAutowiredConfig {
}

// 测试
public class CircularAutowiredTest {

	@Test
	public void test01() {
		AnnotationConfigApplicationContext ioc = new AnnotationConfigApplicationContext(CircularAutowiredConfig.class);
		System.out.println("IOC创建完成");
	}

}
```
### 1. createBean()细节

```java
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {

    RootBeanDefinition mbdToUse = mbd;

    // Make sure bean class is actually resolved at this point, and
    // clone the bean definition in case of a dynamically resolved Class
    // which cannot be stored in the shared merged bean definition.
    // 1
    Class<?> resolvedClass = resolveBeanClass(mbd, beanName);

    // Prepare method overrides.
    mbdToUse.prepareMethodOverrides();

    // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
    // 2
    Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
    if (bean != null) {
        return bean;
    }
	// 3
    Object beanInstance = doCreateBean(beanName, mbdToUse, args);
    return beanInstance;

}
```

1. 解析类
2. 在类的实例化前利用后置处理器执行**PostProcessor的postProcessBeforeInstantiation()方法**，**PostProcessor的类型为：InstantiationAwareBeanPostProcessor**
```java
/**
 * Apply before-instantiation post-processors, resolving whether there is a
 * before-instantiation shortcut for the specified bean.
 * @param beanName the name of the bean
 * @param mbd the bean definition for the bean
 * @return the shortcut-determined bean instance, or {@code null} if none
 */
@Nullable
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        // Make sure bean class is actually resolved at this point.
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
            Class<?> targetType = determineTargetType(beanName, mbd);
            if (targetType != null) {
                bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                if (bean != null) {
                    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                }
            }
        }
        mbd.beforeInstantiationResolved = (bean != null);
    }
    return bean;
}

/**
 * Apply InstantiationAwareBeanPostProcessors to the specified bean definition
 * (by class and name), invoking their {@code postProcessBeforeInstantiation} methods.
 * <p>Any returned object will be used as the bean instead of actually instantiating
 * the target bean. A {@code null} return value from the post-processor will
 * result in the target bean being instantiated.
 * @param beanClass the class of the bean to be instantiated
 * @param beanName the name of the bean
 * @return the bean object to use instead of a default instance of the target bean, or {@code null}
 * @see InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation
 */
@Nullable
protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
            if (result != null) {
                return result;
            }
        }
    }
    return null;
}
```
3. 调用**doCreateBean()**，真正创建Bean

### 2. doCreateBean()细节

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
    throws BeanCreationException {

    // Instantiate the bean.
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        // 1
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        // 2
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    final Object bean = instanceWrapper.getWrappedInstance();
    Class<?> beanType = instanceWrapper.getWrappedClass();
    if (beanType != NullBean.class) {
        mbd.resolvedTargetType = beanType;
    }

    // Allow post-processors to modify the merged bean definition.
    synchronized (mbd.postProcessingLock) {
        if (!mbd.postProcessed) {
            // 3
            applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            mbd.postProcessed = true;
        }
    }

    // Eagerly cache singletons to be able to resolve circular references
    // even when triggered by lifecycle interfaces like BeanFactoryAware.
    // 4
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                      isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }

    // Initialize the bean instance.
    Object exposedObject = bean;
    // 5 属性填充
    populateBean(beanName, mbd, instanceWrapper);
    // 6 初始化
    exposedObject = initializeBean(beanName, exposedObject, mbd);

    if (earlySingletonExposure) {
        Object earlySingletonReference = getSingleton(beanName, false);
        if (earlySingletonReference != null) {
            if (exposedObject == bean) {
                exposedObject = earlySingletonReference;
            }
            else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
                String[] dependentBeans = getDependentBeans(beanName);
                Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
                for (String dependentBean : dependentBeans) {
                    if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                        actualDependentBeans.add(dependentBean);
                    }
                }
            }
        }
    }

    // Register bean as disposable.
    registerDisposableBeanIfNecessary(beanName, bean, mbd);

    return exposedObject;
}
```

1. 如果当前创建的Bean是单例的，先尝试从缓存（**factoryBeanInstanceCache**）里面“取”；“取”不到则执行步骤2。
2. 调用**createBeanInstance(beanName, mbd, args)**去创建Bean的实例。
3. 执行**BeanDefinition**的**PostProcessor**去修改Bean的定义信息。
```java
/**
 * Apply MergedBeanDefinitionPostProcessors to the specified bean definition,
 * invoking their {@code postProcessMergedBeanDefinition} methods.
 * @param mbd the merged bean definition for the bean
 * @param beanType the actual type of the managed bean instance
 * @param beanName the name of the bean
 * @see MergedBeanDefinitionPostProcessor#postProcessMergedBeanDefinition
 */
protected void applyMergedBeanDefinitionPostProcessors(RootBeanDefinition mbd, Class<?> beanType, String beanName) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof MergedBeanDefinitionPostProcessor) {
            MergedBeanDefinitionPostProcessor bdp = (MergedBeanDefinitionPostProcessor) bp;
            bdp.postProcessMergedBeanDefinition(mbd, beanType, beanName);
        }
    }
}
```
4. 判断**earlySingletonExposure**，即判断是否开启循环依赖，循环依赖仅支持单实例Bean，并且默认开启。若开启，则调用**addSingletonFactory**往缓存**singletonFactories**放入一个**singletonFactory**。这个工厂会使用**SmartInstantiationAwareBeanPostProcessor**处理Bean。
```java
/**
 * Add the given singleton factory for building the specified singleton
 * if necessary.
 * <p>To be called for eager registration of singletons, e.g. to be able to
 * resolve circular references.
 * @param beanName the name of the bean
 * @param singletonFactory the factory for the singleton object
 */
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```
5. Bean的属性赋值，依赖注入也在这里面（如使用**@Autowired、@Resource**注解）
6. 初始化Bean

### 3. createBeanInstance()细节

### 4. populateBean()细节

顾名思义，就是给Bean填充属性，属性有可能使用**@Autowired、@Resource、@Value、@Inject**注解注入。

```java
/**
 * Populate the bean instance in the given BeanWrapper with the property values
 * from the bean definition.
 * @param beanName the name of the bean
 * @param mbd the bean definition for the bean
 * @param bw the BeanWrapper with bean instance
 */
@SuppressWarnings("deprecation")  // for postProcessPropertyValues
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    // 1 final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
    if (bw == null) {
        if (mbd.hasPropertyValues()) {
            throw new BeanCreationException(
                mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
        }
        else {
            // Skip property population phase for null instance.
            return;
        }
    }

    // Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
    // state of the bean before properties are set. This can be used, for example,
    // to support styles of field injection.
    boolean continueWithPropertyPopulation = true;

    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                // 2
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                    continueWithPropertyPopulation = false;
                    break;
                }
            }
        }
    }

    if (!continueWithPropertyPopulation) {
        return;
    }

    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

    // 3 获取解析到的属性注入方式
    if (mbd.getResolvedAutowireMode() == AUTOWIRE_BY_NAME || mbd.getResolvedAutowireMode() == AUTOWIRE_BY_TYPE) {
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
        // Add property values based on autowire by name if applicable.
        if (mbd.getResolvedAutowireMode() == AUTOWIRE_BY_NAME) {
            autowireByName(beanName, mbd, bw, newPvs);
        }
        // Add property values based on autowire by type if applicable.
        if (mbd.getResolvedAutowireMode() == AUTOWIRE_BY_TYPE) {
            autowireByType(beanName, mbd, bw, newPvs);
        }
        pvs = newPvs;
    }

    boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
    boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

    PropertyDescriptor[] filteredPds = null;
    if (hasInstAwareBpps) {
        if (pvs == null) {
            pvs = mbd.getPropertyValues();
        }
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                // 4
                PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
                if (pvsToUse == null) {
                    if (filteredPds == null) {
                        filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                    }
                    pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
                    if (pvsToUse == null) {
                        return;
                    }
                }
                pvs = pvsToUse;
            }
        }
    }
    if (needsDepCheck) {
        if (filteredPds == null) {
            filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
        }
        checkDependencies(beanName, mbd, filteredPds, pvs);
    }

    if (pvs != null) {
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}
```

1. **mbd即RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName)**

2. 执行**InstantiationAwareBeanPostProcessor**的**postProcessAfterInstantiation**方法，使得在Bean的属性设置前对Bean做修改。

3. 获取解析到的属性注入方式，有以下方式。

```java
public interface AutowireCapableBeanFactory extends BeanFactory {

	/**
	 * Constant that indicates no externally defined autowiring. Note that
	 * BeanFactoryAware etc and annotation-driven injection will still be applied.
	 * @see #createBean
	 * @see #autowire
	 * @see #autowireBeanProperties
	 */
	int AUTOWIRE_NO = 0;

	/**
	 * Constant that indicates autowiring bean properties by name
	 * (applying to all bean property setters).
	 * @see #createBean
	 * @see #autowire
	 * @see #autowireBeanProperties
	 */
	int AUTOWIRE_BY_NAME = 1;

	/**
	 * Constant that indicates autowiring bean properties by type
	 * (applying to all bean property setters).
	 * @see #createBean
	 * @see #autowire
	 * @see #autowireBeanProperties
	 */
	int AUTOWIRE_BY_TYPE = 2;

	/**
	 * Constant that indicates autowiring the greediest constructor that
	 * can be satisfied (involves resolving the appropriate constructor).
	 * @see #createBean
	 * @see #autowire
	 */
	int AUTOWIRE_CONSTRUCTOR = 3;

	/**
	 * Constant that indicates determining an appropriate autowire strategy
	 * through introspection of the bean class.
	 * @see #createBean
	 * @see #autowire
	 * @deprecated as of Spring 3.0: If you are using mixed autowiring strategies,
	 * prefer annotation-based autowiring for clearer demarcation of autowiring needs.
	 */
	@Deprecated
	int AUTOWIRE_AUTODETECT = 4;
}
```

**getResolvedAutowireMode() 细节，默认autowireMode = AUTOWIRE_NO;**

```java
/**
 * Return the resolved autowire code,
 * (resolving AUTOWIRE_AUTODETECT to AUTOWIRE_CONSTRUCTOR or AUTOWIRE_BY_TYPE).
 * @see #AUTOWIRE_AUTODETECT
 * @see #AUTOWIRE_CONSTRUCTOR
 * @see #AUTOWIRE_BY_TYPE
 */
public int getResolvedAutowireMode() { 
    if (this.autowireMode == AUTOWIRE_AUTODETECT) {
        // Work out whether to apply setter autowiring or constructor autowiring.
        // If it has a no-arg constructor it's deemed to be setter autowiring,
        // otherwise we'll try constructor autowiring.
        Constructor<?>[] constructors = getBeanClass().getConstructors();
        for (Constructor<?> constructor : constructors) {
            if (constructor.getParameterCount() == 0) {
                return AUTOWIRE_BY_TYPE;
            }
        }
        return AUTOWIRE_CONSTRUCTOR;
    }
    else {
        return this.autowireMode;
    }
}
```

4. 运用后置处理器设置Bean的属性，即利用对应的后置处理器执行**postProcessProperties()**方法为Bean进行属性注入。

   - **CommonAnnotationBeanPostProcessor**处理**@Resource**注解的属性注入。
   - **AutowiredAnnotationBeanPostProcessor**处理**@Autowired、@Value、JSR-330's @Inject**注解的属性注入。

### 5. initializeBean()细节
```java
/**
 * Initialize the given bean instance, applying factory callbacks
 * as well as init methods and bean post processors.
 * <p>Called from {@link #createBean} for traditionally defined beans,
 * and from {@link #initializeBean} for existing bean instances.
 * @param beanName the bean name in the factory (for debugging purposes)
 * @param bean the new bean instance we may need to initialize
 * @param mbd the bean definition that the bean was created with
 * (can also be {@code null}, if given an existing bean instance)
 * @return the initialized bean instance (potentially wrapped)
 * @see BeanNameAware
 * @see BeanClassLoaderAware
 * @see BeanFactoryAware
 * @see #applyBeanPostProcessorsBeforeInitialization
 * @see #invokeInitMethods
 * @see #applyBeanPostProcessorsAfterInitialization
 */
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
    // 1 
    if (System.getSecurityManager() != null) {
        AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
            // 2
            invokeAwareMethods(beanName, bean);
            return null;
        }, getAccessControlContext());
    }
    else {
        // 2
        invokeAwareMethods(beanName, bean);
    }

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        // 3
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }

    // 4
    invokeInitMethods(beanName, wrappedBean, mbd);
    
    if (mbd == null || !mbd.isSynthetic()) {
        // 5
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }

    return wrappedBean;
}
```

1. 获取安全管理器**（当运行未知的Java程序的时候，该程序可能有恶意代码（删除系统文件、重启系统等），为了防止运行恶意代码对系统产生影响，需要对运行的代码的权限进行控制，这时候就要启用Java安全管理器）**，不管有没有安全管理器，都会执行步骤2。

2. 执行通知方法，即回调实现了接口**BeanNameAware、BeanClassLoaderAware、BeanFactoryAware**对应的**setXXX()**方法；**XXXAware**接口都会有一个**setXXX()**方法。
```java
private void invokeAwareMethods(final String beanName, final Object bean) {
    if (bean instanceof Aware) {
        if (bean instanceof BeanNameAware) {
            ((BeanNameAware) bean).setBeanName(beanName);
        }
        if (bean instanceof BeanClassLoaderAware) {
            ClassLoader bcl = getBeanClassLoader();
            if (bcl != null) {
                ((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
            }
        }
        if (bean instanceof BeanFactoryAware) {
            ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
        }
    }
}
```

3. 对给定的Bean运用后置处理器，执行后置处理器他们自己的**postProcessBeforeInitialization()**方法，返回包装后的Bean，即返回加工后的Bean；此过程在Bean的初始化前进行。
```java
@Override
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
    throws BeansException {

    Object result = existingBean;
	// 挨个执行postProcessBeforeInitialization(result, beanName)
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessBeforeInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```
4. 执行初始化方法**（实现了InitializingBean接口、或者定义了自定义的init()方法）**，也就是给Bean设置属性；方法是检查Bean是否实现了**InitializingBean**接口、或者定义了自定义的**init()**方法，有的话就执行对应的初始化方法。
```java
protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
    throws Throwable {

    // 判断Bean是否实现了InitializingBean接口
    boolean isInitializingBean = (bean instanceof InitializingBean);
    if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
        if (logger.isTraceEnabled()) {
            logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
        }
        if (System.getSecurityManager() != null) {
            try {
                AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
                    ((InitializingBean) bean).afterPropertiesSet();
                    return null;
                }, getAccessControlContext());
            }
            catch (PrivilegedActionException pae) {
                throw pae.getException();
            }
        }
        else {
            // 不管有没有安全管理器都执行InitializingBean接口规定的afterPropertiesSet()方法
            ((InitializingBean) bean).afterPropertiesSet();
        }
    }

    // 如果Bean没有实现InitializingBean接口
    if (mbd != null && bean.getClass() != NullBean.class) {
        String initMethodName = mbd.getInitMethodName();
        // 如果Bean有自定义init()方法
        if (StringUtils.hasLength(initMethodName) &&
            !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
            !mbd.isExternallyManagedInitMethod(initMethodName)) {
            // 使用反射执行
            invokeCustomInitMethod(beanName, bean, mbd);
        }
    }
}
```
6. 对给定的Bean运用后置处理器，执行后置处理器他们自己的**postProcessAfterInitialization()**方法，返回包装后的Bean，即返回加工后的Bean；此过程在Bean的初始化后进行。
```java
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    throws BeansException {

    Object result = existingBean;
    // postProcessAfterInitialization(result, beanName)
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessAfterInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```

## 二、使用构造函数
```java
@Service
public class AService {

	public AService() {
		System.out.println("AService constructor");
	}
}

@Service
public class BService {

	public BService() {
		System.out.println("BService constructor");
	}
}

@Service
public class BookService {
    
    private final AService aService;
	private final BService bService;

	public BookService(AService aService, BService bService) {
		this.aService = aService;
		this.bService = bService;
		System.out.println("BookService constructor......");
	}
    
}

// 配置类
@Configuration
@ComponentScan(value = "com.gnehcnij.service")
public class CircularAutowiredConfig {
}

// 测试
public class CircularAutowiredTest {

	@Test
	public void test01() {
		AnnotationConfigApplicationContext ioc = new AnnotationConfigApplicationContext(CircularAutowiredConfig.class);
		System.out.println("IOC创建完成");
	}

}
```

这种方式的注入与使用注解的方式的注入时机不同，后者实在**populateBean()**方法进行的，而前者是在**doCreateBean()细节中的步骤2**进行的，即在**createBeanInstance(beanName, mbd, args)**方法进行的。

### 1. createBeanInstance()细节

**使用合适的实例化策略为给定的Bean创建一个实例，并包装成BeanWrapper返回，实例化策略有工厂方法、构造器注入、简单的实例化，参数args是用于构造函数或工厂方法调用的显式参数。**

```java
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
    // Make sure bean class is actually resolved at this point.
    Class<?> beanClass = resolveBeanClass(mbd, beanName);

    if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                        "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
    }
    Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
    if (instanceSupplier != null) {
		// 1
        return obtainFromSupplier(instanceSupplier, beanName);
    }
    if (mbd.getFactoryMethodName() != null) {
		// 2 获取工厂方法名字
        return instantiateUsingFactoryMethod(beanName, mbd, args);
    }

    // Shortcut when re-creating the same bean...
    boolean resolved = false;
    boolean autowireNecessary = false;
    if (args == null) {
        synchronized (mbd.constructorArgumentLock) {
		    // 3
            if (mbd.resolvedConstructorOrFactoryMethod != null) {
                resolved = true;
                autowireNecessary = mbd.constructorArgumentsResolved;
            }
        }
    }
    if (resolved) {
        // 4
        if (autowireNecessary) {
            return autowireConstructor(beanName, mbd, null, null);
        }
        // 5
        else {
            return instantiateBean(beanName, mbd);
        }
    }

    // Candidate constructors for autowiring?
    // 6
    Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
    if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
        mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
        // 7
        return autowireConstructor(beanName, mbd, ctors, args);
    }

    // Preferred constructors for default construction?
    // 8
    ctors = mbd.getPreferredConstructors();
    if (ctors != null) {
        return autowireConstructor(beanName, mbd, ctors, null);
    }

    // No special handling: simply use no-arg constructor.
    // 9
    return instantiateBean(beanName, mbd);
}
```

1. 如果有**supplier**，那么先从**supplier**中获取Bean实例。

2. 如果有工厂方法，则使用工厂方法获取Bean实例。

3. 检查缓存**resolvedConstructorOrFactoryMethod**中对于当前要实例化的Bean是由已经有了解析好了的构造器或者是工厂方法，第一次创建Bean的实例缓存中为空，则执行步骤6；

4. autowireNecessary

5. instantiateBean()

6. 从所有已经注册在ioc容器中的后置处理器（类型为SmartInstantiationAwareBeanPostProcessor）中确定将要使用的候选构造器，最终**AutowiredAnnotationBeanPostProcessor**有返回。

```java
@Nullable
protected Constructor<?>[] determineConstructorsFromBeanPostProcessors(@Nullable Class<?> beanClass, String beanName)
    throws BeansException {

    if (beanClass != null && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                Constructor<?>[] ctors = ibp.determineCandidateConstructors(beanClass, beanName);
                if (ctors != null) {
                    return ctors;
                }
            }
        }
    }
    return null;
}
```

7. 如果**有候选构造器**或**autowireMode = AUTOWIRE_CONSTRUCTOR**或**有构造器参数值**或**参数args不为空**，则注入构造器，否则执行步骤8。

8. 看看是否有**默认构造的首选构造函数**，如果有则注入构造器，否则执行步骤9；步骤7、8的注入构造器注入都是调用**ConstructorResolver.autowireConstructor()**方法。

9. 使用无参构造器实例化Bean。

### 2. autowireConstructor()细节