# (9条消息) SpringMVC源码之Handler注册、获取以及请求controller中方法_曹自标的博客-CSDN博客
#### 总结

1.  对 requestMappingHandlerMapping 进行 initializeBean 时 register Handler
2.  http 开始请求时，initHandlerMappings，DispatcherServlet 中 handlerMappings 赋值完成
3.  最后在 DispatcherServlet#doDispatch（）中，用对应的 HandlerAdapter 和 Handler 通过反射去请求 controller 中方法

#### 对 requestMappingHandlerMapping 进行 initializeBean 时 register Handler

调用链：

> AbstractApplicationContext#refresh() --> AbstractApplicationContext#finishBeanFactoryInitialization() --> DefaultListableBeanFactory#preInstantiateSingletons() --> AbstractBeanFactory#getBean() --> AbstractBeanFactory#doGetBean() --> AbstractAutowireCapableBeanFactory#createBean() --> AbstractAutowireCapableBeanFactory#doCreateBean() --> AbstractAutowireCapableBeanFactory#initializeBean() --> AbstractAutowireCapableBeanFactory#invokeInitMethods() --> RequestMappingHandlerMapping#afterPropertiesSet() --> AbstractHandlerMethodMapping#afterPropertiesSet() --> AbstractHandlerMethodMapping#initHandlerMethods() --> AbstractHandlerMethodMapping#processCandidateBean --> AbstractHandlerMethodMapping#detectHandlerMethods() --> RequestMappingHandlerMapping#registerHandlerMethod() --> AbstractHandlerMethodMapping#registerHandlerMethod()

在 AbstractHandlerMethodMapping#initHandlerMethods（）中先获取所有的 beanName，再挑选出符合条件的进行处理

```
protected void initHandlerMethods() {
    //获取容器中所有beanName
	for (String beanName : getCandidateBeanNames()) {
		if (!beanName.startsWith(SCOPED_TARGET_NAME_PREFIX)) {
			processCandidateBean(beanName);
		}
	}
	handlerMethodsInitialized(getHandlerMethods());
}

```

判断是 Handler 的才继续调用 detectHandlerMethods 方法

```
protected void processCandidateBean(String beanName) {
	Class<?> beanType = null;
    ......
	if (beanType != null && isHandler(beanType)) {
		detectHandlerMethods(beanName);
	}
}

```

满足 handler 的条件是（RequestMappingHandlerMapping#isHandler（））：@Controller 或 @RequestMapping 进行注解

```
@Override
protected boolean isHandler(Class<?> beanType) {
	return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
			AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
}

```

满足条件的，进行注册  
RequestMappingHandlerMapping#registerHandlerMethod()

```
@Override
protected void registerHandlerMethod(Object handler, Method method, RequestMappingInfo mapping) {
	super.registerHandlerMethod(handler, method, mapping);
	updateConsumesCondition(mapping, method);
}

```

AbstractHandlerMethodMapping#registerHandlerMethod()

```
protected void registerHandlerMethod(Object handler, Method method, T mapping) {
	this.mappingRegistry.register(mapping, handler, method);
}

```

完成注册。AbstractHandlerMethodMapping#register（）

```
public void register(T mapping, Object handler, Method method) {
	this.readWriteLock.writeLock().lock();
	try {
		HandlerMethod handlerMethod = createHandlerMethod(handler, method);
		validateMethodMapping(handlerMethod, mapping);

		Set<String> directPaths = AbstractHandlerMethodMapping.this.getDirectPaths(mapping);
		for (String path : directPaths) {
			this.pathLookup.add(path, mapping);
		}

		String name = null;
		if (getNamingStrategy() != null) {
			name = getNamingStrategy().getName(handlerMethod, mapping);
			addMappingName(name, handlerMethod);
		}

		CorsConfiguration config = initCorsConfiguration(handler, method, mapping);
		if (config != null) {
			config.validateAllowCredentials();
			this.corsLookup.put(handlerMethod, config);
		}

		this.registry.put(mapping, new MappingRegistration<>(mapping, handlerMethod, directPaths, name));
	}
	finally {
		this.readWriteLock.writeLock().unlock();
	}
}

```

#### initHandlerMappings

调用链：

> Standardwrapper#initServlet() --> HttpServletBean#init() --> FrameworkServlet#initServletBean() --> FrameworkServlet#initWebApplicationContext() --> DispatcherServlet#onRefresh() --> DispatcherServlet#initStrategies() -->  
> DispatcherServlet#initHandlerMappings()

http 请求时，先 initHandlerMappings.  
matchingBeans 会存储获取到所有符合条件的，再给 handlerMappings 赋值

```
private void initHandlerMappings(ApplicationContext context) {
	this.handlerMappings = null;

	if (this.detectAllHandlerMappings) {
		// Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
		Map<String, HandlerMapping> matchingBeans =
				BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
		if (!matchingBeans.isEmpty()) {
			this.handlerMappings = new ArrayList<>(matchingBeans.values());
			// We keep HandlerMappings in sorted order.
			AnnotationAwareOrderComparator.sort(this.handlerMappings);
		}
	}
}

```

其中 BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false); 会先调用 AbstractApplicationContext#getBeansOfType(), 再调用 DefaultListableBeanFactory#getBeansOfType()

先获取 beanName requestMappingHandlerMapping，再根据 beanName（requestMappingHandlerMapping）从容器获取 beanInstance。最后 put 到 result 中返回

```
@Override
@SuppressWarnings("unchecked")
public <T> Map<String, T> getBeansOfType(
		@Nullable Class<T> type, boolean includeNonSingletons, boolean allowEagerInit) throws BeansException {
    //获取requestMappingHandlerMapping
	String[] beanNames = getBeanNamesForType(type, includeNonSingletons, allowEagerInit);
	Map<String, T> result = CollectionUtils.newLinkedHashMap(beanNames.length);
	for (String beanName : beanNames) {
		try {
		    再根据beanName（requestMappingHandlerMapping）从容器获取beanInstance
			Object beanInstance = getBean(beanName);
			if (!(beanInstance instanceof NullBean)) {
				result.put(beanName, (T) beanInstance);
			}
		}
	......
	return result;
}

```

从而在 initHandlerMappings 给 handlerMappings 赋值完成

```
this.handlerMappings = new ArrayList<>(matchingBeans.values());

```

#### doDispatch

获取当前请求的 handler 和 HandlerAdapter

DispatcherServlet#doDispatch（）

```
mappedHandler = getHandler(processedRequest);

```

DispatcherServlet#getHandler（）

```
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
	if (this.handlerMappings != null) {
		for (HandlerMapping mapping : this.handlerMappings) {
			HandlerExecutionChain handler = mapping.getHandler(request);
			if (handler != null) {
				return handler;
			}
		}
	}
	return null;
}

```

通过反射方式请求 controller 中方法：

```
// Actually invoke the handler.
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

```

doDispatch 代码附录

```
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
	HttpServletRequest processedRequest = request;
	HandlerExecutionChain mappedHandler = null;
	boolean multipartRequestParsed = false;

	WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

	try {
		ModelAndView mv = null;
		Exception dispatchException = null;

		try {
			processedRequest = checkMultipart(request);
			multipartRequestParsed = (processedRequest != request);

			// Determine handler for the current request.
			mappedHandler = getHandler(processedRequest);
			if (mappedHandler == null) {
				noHandlerFound(processedRequest, response);
				return;
			}

			// Determine handler adapter for the current request.
			HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

			// Process last-modified header, if supported by the handler.
			String method = request.getMethod();
			boolean isGet = "GET".equals(method);
			if (isGet || "HEAD".equals(method)) {
				long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
				if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
					return;
				}
			}

			if (!mappedHandler.applyPreHandle(processedRequest, response)) {
				return;
			}

			// Actually invoke the handler.
			mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

			if (asyncManager.isConcurrentHandlingStarted()) {
				return;
			}

			applyDefaultViewName(processedRequest, mv);
			mappedHandler.applyPostHandle(processedRequest, response, mv);
		}
		catch (Exception ex) {
			dispatchException = ex;
		}
		catch (Throwable err) {
			// As of 4.3, we're processing Errors thrown from handler methods as well,
			// making them available for @ExceptionHandler methods and other scenarios.
			dispatchException = new NestedServletException("Handler dispatch failed", err);
		}
		processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
	}
	catch (Exception ex) {
		triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
	}
	catch (Throwable err) {
		triggerAfterCompletion(processedRequest, response, mappedHandler,
				new NestedServletException("Handler processing failed", err));
	}
	finally {
		if (asyncManager.isConcurrentHandlingStarted()) {
			// Instead of postHandle and afterCompletion
			if (mappedHandler != null) {
				mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
			}
		}
		else {
			// Clean up any resources used by a multipart request.
			if (multipartRequestParsed) {
				cleanupMultipart(processedRequest);
			}
		}
	}
}


```

 [https://blog.csdn.net/weixin_43843104/article/details/109789004](https://blog.csdn.net/weixin_43843104/article/details/109789004)
