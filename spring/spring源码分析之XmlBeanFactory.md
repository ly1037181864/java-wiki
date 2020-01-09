##XmlBeanFactory加载

- 构建spring容器
```text
XmlBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("spring-test.xml"));
```

- 容器初始化流程
![avatar](../imags/spring/gradle-03.png)

- 初始化详解
1.构建XmlBeanFactory实例时，spring默认构建了XmlBeanDefinitionReader对象，并将xml的解析委托给了XmlBeanDefinitionReader对象去处理
```text
#初始化XmlBeanFactory时就构建好XmlBeanDefinitionReader对象
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);
```
```text
public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
    super(parentBeanFactory);
    //加载BeanDefinition定义
    this.reader.loadBeanDefinitions(resource);
}
```
```text
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
    //EncodedResource对资源进行编码处理，国际化
    return loadBeanDefinitions(new EncodedResource(resource));
}
```
EncodedResource中最重要的就是getReader方法，根据编码格式来获取资源文件，即所谓的国际化
```text
public Reader getReader() throws IOException {
    if (this.charset != null) {
        return new InputStreamReader(this.resource.getInputStream(), this.charset);
    }
    else if (this.encoding != null) {
        return new InputStreamReader(this.resource.getInputStream(), this.encoding);
    }
    else {
        return new InputStreamReader(this.resource.getInputStream());
    }
}
```
spring需要在容器初始化的时候对配置文件即xml进行解析，即将xml文件转换成Document对象，以便后续的解析
```text
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
        throws BeanDefinitionStoreException {

    try {
        //将给定资源转化成Document对象，以便后续的xml元素解析
        Document doc = doLoadDocument(inputSource, resource);
        //解析并注册BeanDefinition
        int count = registerBeanDefinitions(doc, resource);
        ...
    }
    ...
}
```
spring将Document的解析委托给BeanDefinitionDocumentReader处理
```text
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    //对元素的解析委托给BeanDefinitionDocumentReader处理
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    int countBefore = getRegistry().getBeanDefinitionCount();
    //解析并注册BeanDefinition
    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    //记录本次加载的BeanDefinition个数
    return getRegistry().getBeanDefinitionCount() - countBefore;
}
```
BeanDefinitionParserDelegate为元素解析策略，在spring的配置文件中存在各种各样的元素标签，有spring默认的元素标签，如beans、alias、import、bean等，
以及用户自定义的元素标签，最典型的就是事务标签及数据源jdbc相关的标签等它不属于spring的默认标签，因此需要根据xml的元素来决定解析的元素的具体策略，
在实际的使用中BeanDefinitionParserDelegate则是被用来处理自定义标签
```text
protected void doRegisterBeanDefinitions(Element root) {
		BeanDefinitionParserDelegate parent = this.delegate;
		//创建默认的元素解析策略，实际上BeanDefinitionParserDelegate是被用来处理自定义的元素标签
		this.delegate = createDelegate(getReaderContext(), root, parent);

		//默认的名称空间还是用户自定义的名称空间
		if (this.delegate.isDefaultNamespace(root)) {
			//解析profile标签 该标签用来指定配置文件是dev环境的还是test环境还是prod生产环境
			String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
			if (StringUtils.hasText(profileSpec)) {
				String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
						profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
				// We cannot use Profiles.of(...) since profile expressions are not supported
				// in XML config. See SPR-12458 for details.
				if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
					if (logger.isDebugEnabled()) {
						logger.debug("Skipped XML bean definition file due to specified profiles [" + profileSpec +
								"] not matching: " + getReaderContext().getResource());
					}
					return;
				}
			}
		}

		//xml解析前处理，钩子方法，留给子类实现
		preProcessXml(root);
		//解析BeanDefinition配置
		parseBeanDefinitions(root, this.delegate);
		//xml解析后处理，钩子方法，留给子类实现
		postProcessXml(root);

		this.delegate = parent;
	}
```
`PROFILE_ATTRIBUTE`标签即`profile`属性，该属性是用来指定不同的环境作用域的，同如下示例，我们可以根据不同的环境来指定不同的配置信息
```text
<beans profile="development">
    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
      <property name="driverClass" value="${jdbc.driver}" />
      <property name="url" value="${jdbc.url}" />
      <property name="username" value="${jdbc.username}" />
      <property name="password" value="${jdbc.password}" />
    </bean>
</beans>

<beans profile="test">
    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
      <property name="driverClass" value="${jdbc.driver}" />
      <property name="url" value="${jdbc.url}" />
      <property name="username" value="${jdbc.username}" />
      <property name="password" value="${jdbc.password}" />
    </bean>
</beans>
```
在spring解析xml配置文件时，又需要根据实际情况，针对spring默认的元素一级用户自定义的元素进行不同的策略解析，而这个过程则交由
BeanDefinitionParserDelegate去处理
```text
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    //判断元素是否是默认名称空间如beans等标签
    if (delegate.isDefaultNamespace(root)) {
        NodeList nl = root.getChildNodes();
        for (int i = 0; i < nl.getLength(); i++) {
            Node node = nl.item(i);
            if (node instanceof Element) {
                Element ele = (Element) node;
                if (delegate.isDefaultNamespace(ele)) {
                    //spring默认的标签解析
                    parseDefaultElement(ele, delegate);
                }
                else {
                    //用户自定义的解析器如事务标签解析，aop标签等
                    delegate.parseCustomElement(ele);
                }
            }
        }
    }
    else {
        //用户自定义的解析器如事务标签解析，aop标签等
        delegate.parseCustomElement(root);
    }
}
```
在parseDefaultElement方法中，spring会递归循环的去处理import、alias、bean、beans等标签，最终会调用processBeanDefinition针对bean的解析
```text
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    //对import 标签的处理
    if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
        importBeanDefinitionResource(ele);
    }
    //对alias标签的处理
    else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
        processAliasRegistration(ele);
    }
    //对bean标签的处理，同时将bean的信息注册到伪IOC容器中去
    else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
        processBeanDefinition(ele, delegate);
    }
    //对beans标签的处理
    else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
        // recurse
        //递归解析
        doRegisterBeanDefinitions(ele);
    }
}
```
importBeanDefinitionResource对import标签解析时，首先是获取到resource元素，解析给定资源的路径定义，然后判断是否是spring表达式,
并对其进行处理，然后去判断该路径是相对路径还是绝对路径，最后根据不同的资源路径定义来加载bean的定义信息
```text
protected void importBeanDefinitionResource(Element ele) {
    String location = ele.getAttribute(RESOURCE_ATTRIBUTE);
    ...

    // Resolve system properties: e.g. "${user.dir}"
    //解析系统属性
    location = getReaderContext().getEnvironment().resolveRequiredPlaceholders(location);

    Set<Resource> actualResources = new LinkedHashSet<>(4);

    // Discover whether the location is an absolute or relative URI
    //判定location 是绝对URI 还是相对URI
    boolean absoluteLocation = false;
    try {
        absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute();
    }
    ...

    // Absolute or relative?
    if (absoluteLocation) {
        try {
            //如果是绝对URI 则直接根据地址加载对应的配置文件
            int importCount = getReaderContext().getReader().loadBeanDefinitions(location, actualResources);
            if (logger.isTraceEnabled()) {
                logger.trace("Imported " + importCount + " bean definitions from URL location [" + location + "]");
            }
        }
        ...
    }
    else {
        // No URL -> considering resource location as relative to the current file.
        //如果是相对地址则根据相对地址计算出绝对地址
        try {
            int importCount;
            //Resource 存在多个子实现类，女H VfsResource 、FileSystemResource 等，
            //而每个resource 的createRelative方式实现都不一样，所以这里先使用子类的方法尝试解析
            //如果在构建XmlBeanFactory使用的是ClasspathResource则createRelative的逻辑是applyRelativePath根据原资源文件路径来定位相对位置
            Resource relativeResource = getReaderContext().getResource().createRelative(location);
            if (relativeResource.exists()) {
                //r囡构建是默认使用的是XmlBeanFactory，则走XmlBeanDefinitionReader
                importCount = getReaderContext().getReader().loadBeanDefinitions(relativeResource);
                actualResources.add(relativeResource);
            }
            //如果解析不成功， 则使用默认的解析器ResourcePatternResolver进行解析
            else {
                String baseLocation = getReaderContext().getResource().getURL().toString();
                importCount = getReaderContext().getReader().loadBeanDefinitions(
                        StringUtils.applyRelativePath(baseLocation, location), actualResources);
            }
            ...
        }
       ...
    }
    ...
}
```
```text
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">

	<import resource="classpath:applicationContext-datasource.xml" />
	<import resource="classpath:applicationContext-dao-config.xml" />

</beans>
```
spring针对别名的处理比较简单，只是获取到别名注册即可
```text
protected void processAliasRegistration(Element ele) {
		String name = ele.getAttribute(NAME_ATTRIBUTE);
		String alias = ele.getAttribute(ALIAS_ATTRIBUTE);
		boolean valid = true;
		...
		if (valid) {
			try {
				//注册别名
				getReaderContext().getRegistry().registerAlias(name, alias);
			}
			...
		}
	}
```
别名的标签，一般是有特定场景需求，针对同一个bean可以有不同的别名，然后在不同的地方根据别名来获取同一个bean，大部分场景中
这种用法比较少，知道即可
```text
<bean id="some" class="src.com.Some"/>
<alias name="some" alias="someJava,oneBean,twoBean"/>
```
```text
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    //bean的属性解析
    //委托BeanDefinitionParserDelegate 类的parseBeanDefinitionElement 方法进行元素解析
    //bdHolder 实例已经包含了配置文件中配置的各种属性了，例如class 、name、id 、alias 之类的
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        /**
         * decorateBeanDefinitionIfRequired如果必要就对BeanDefinition进行包装
         * 若存在默认标签的子节点下有自定义属性， 还需要再次对自定义标签进行解析
         * <bean>
         *     <myname></myname>
         * </bean>
         * 需要注意的是这里的包装处理与之前的对bean的处理parseDefaultElement和parseCustomElement方法
         * 不同，前者是针对bean元素的子元素的自定义解析，而后者则是针对与bean元素同级别的元素的自定义解析，如最常用的
         * 事务属性注解
         */
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        try {
            // Register the final decorated instance.
            //注册BeanDefinition
            BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
        }
        catch (BeanDefinitionStoreException ex) {
            getReaderContext().error("Failed to register bean definition with name '" +
                    bdHolder.getBeanName() + "'", ele, ex);
        }
        // Send registration event.
        //发出响应事件，通知相关的监昕器，这个bean 已经加载完成了
        getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }
}
```
```text
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, @Nullable BeanDefinition containingBean) {
    ...

    //检查bean的唯一性
    if (containingBean == null) {
        checkNameUniqueness(beanName, aliases, ele);
    }

    //元素解析
    AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
    if (beanDefinition != null) {
        if (!StringUtils.hasText(beanName)) {
            try {
                //如果不存在beanName 那么根据Spring中提供的命名规则为当前bean 生成对应的beanName
                if (containingBean != null) {
                    beanName = BeanDefinitionReaderUtils.generateBeanName(
                            beanDefinition, this.readerContext.getRegistry(), true);
                }
                else {
                    beanName = this.readerContext.generateBeanName(beanDefinition);
                    // Register an alias for the plain bean class name, if still possible,
                    // if the generator returned the class name plus a suffix.
                    // This is expected for Spring 1.2/2.0 backwards compatibility.
                    String beanClassName = beanDefinition.getBeanClassName();
                    if (beanClassName != null &&
                            beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
                            !this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
                        aliases.add(beanClassName);
                    }
                }
                ...
            }
            ...
        }
        String[] aliasesArray = StringUtils.toStringArray(aliases);
        //将bean的信息保存到BeanDefinitionHolder中
        return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
    }

    return null;
}
```
parseBeanDefinitionElement方法中针对bean的各种属性解析，并返回AbstractBeanDefinition定义对象
```text
public AbstractBeanDefinition parseBeanDefinitionElement(
			Element ele, String beanName, @Nullable BeanDefinition containingBean) {

    this.parseState.push(new BeanEntry(beanName));

    String className = null;
    //class标签解析
    if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
        //获取class信息
        className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
    }
    String parent = null;
    //获取parent标签
    if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
        parent = ele.getAttribute(PARENT_ATTRIBUTE);
    }

    try {
        //创建用于承载属性的AbstractBeanDefinition 类型的GenericBeanDefinition
        //设置父bean
        AbstractBeanDefinition bd = createBeanDefinition(className, parent);

        //硬编码解析默认bean的各种属性
        parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
        //提取description
        bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));

        /**
         * 解析元数据
         * <bean id="myTestBean" class="bean.MyTestBean">
         *      <meta key="testStr" value="aaaaaaaa"/>
         * </bean>
         */
        parseMetaElements(ele, bd);
        //解析 lockup-method 属性 提供可插拔式开发 案例见《spring源码深度解析》
        parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
        //解析replace-method属性  案例见《spring源码深度解析》
        parseReplacedMethodSubElements(ele, bd.getMethodOverrides());

        //解析构造函数参数
        parseConstructorArgElements(ele, bd);
        //解析properties子元素
        parsePropertyElements(ele, bd);
        //解析qualifier子元素
        parseQualifierElements(ele, bd);

        bd.setResource(this.readerContext.getResource());
        bd.setSource(extractSource(ele));

        return bd;
    }
    catch (ClassNotFoundException ex) {
        error("Bean class [" + className + "] not found", ele, ex);
    }
    catch (NoClassDefFoundError err) {
        error("Class that bean class [" + className + "] depends on not found", ele, err);
    }
    catch (Throwable ex) {
        error("Unexpected failure during bean definition parsing", ele, ex);
    }
    finally {
        this.parseState.pop();
    }

    return null;
}
```
parent属性解析，为某个bean指定父bean，也即我们常说的继承关系
```text
<bean id="abstractServiceThread" class="com.project.schedual.ServiceThread" abstract="true">
    <property name="baseDao" ref="baseDAO"></property>
</bean>
<bean id="docReceiveFlowThread" parent="abstractServiceThread">
    <property name="svc" ref="docReceiveFlowService"></property>
</bean>
```
