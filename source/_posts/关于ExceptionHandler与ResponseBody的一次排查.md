---
title: 关于ExceptionHandler与ResponseBody的一次排查
tags:
  - springmvc
  - springboot
  - ExceptionHandler
  - ResponseBody
categories:
  - 问题排查
date: 2019-07-11 14:41:49
---
## 问题
工作中，在使用`@ExceptionHandler`作为`controller`全局异常处理的过程中，发现`@ResponseBody`注解并不能像往常一样返回json格式的数据，而是返回了xml格式的数据。

## 结论

### 错误认识
- `@ResponseBody`不意味着json格式响应，而是指controller的返回结果直接写回response，无需经过ModelAndView。见[spring文档](https://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-ann-responsebody)
> The @ResponseBody annotation is similar to @RequestBody. This annotation can be placed on a method and indicates that the return type should be written straight to the HTTP response body (and not placed in a Model, or interpreted as a view name)

### 新认识
- `HttpMessageConverter`才是进行数据格式转换的关键
- 经过试错发现，在request header中加入`Accept application/json`可以使原问题得到正确的json响应


### 遗留问题
- 工程中其他请求的request header中没有`Accept application/json`也能得到json响应，是为什么？


## 记录下排查过程
1. 怀疑`@ResponseBody`注解未生效
	- 查阅后得知spring3.1版本已修复
	- （知道自己对`@ResponseBody`有误解后，就明白这并不是个问题，xml响应也是注解的结果）
2.  百思不得其解，在方法中使用`HttpServletResponse response`直接输出json响应暴力解决。仔细想想，本质上是一致的。
```
	response.setContentType("application/json;charset=UTF-8");
	response.getWriter().write(gson.toJson(re));
```
3. 排除spring版本影响
	- 工程依赖过多，更改版本错误重重，临时用springboot搭建了web，使用4.3.9.RELEASE（原工程版本）和5.1.8.RELEASE，配合`@RestController`，无需`Accept application/json`就能得到json响应




## 关于遗留问题
工程中其他请求的request header中没有`Accept application/json`也能得到json响应，是为什么？

差别在于，异常处理与正常返回用到的RequestResponseBodyMethodProcessor不是同一个实例，异常的是在ExceptionHandlerExceptionResolver中的，正常的是在RequestMappingHandlerAdapter.
前者比之后者，缺少springmvc-config.xml中自定义的MessageConverter，例如MappingJackson2HttpMessageConverter。

```java
		<bean
			class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
			<property name="supportedMediaTypes">
				<list>
					<value>application/json;charset=UTF-8</value>
					<value>text/html;charset=UTF-8</value>
				</list>
			</property>
			<property name="objectMapper">
				<bean class="com.jd.common.xss.CustomObjectMapper" />
			</property>
		</bean>
```
在writeWithMessageConverters方法中，异常resolver返回的b有7个，排序后最前的是`application/xml`，正常resolver返回9个，排序最前是上面自定义的`application/json;charset=UTF-8`，因此，最终返回结果有格式上的差异。
AbstractMessageConverterMethodProcessor#writeWithMessageConverters
```java
		HttpServletRequest request = inputMessage.getServletRequest();
		// 根据request的accept得到请求支持的返回格式 a
		List<MediaType> requestedMediaTypes = getAcceptableMediaTypes(request);
		// 得到controller方法支持的返回格式 b <-- 差别在这里
		List<MediaType> producibleMediaTypes = getProducibleMediaTypes(request, valueType, declaredType);

		if (outputValue != null && producibleMediaTypes.isEmpty()) {
			throw new IllegalArgumentException("No converter found for return value of type: " + valueType);
		}

		// 对比a和b，得到所有兼容的返回格式 c
		Set<MediaType> compatibleMediaTypes = new LinkedHashSet<MediaType>();
		for (MediaType requestedType : requestedMediaTypes) {
			for (MediaType producibleType : producibleMediaTypes) {
				if (requestedType.isCompatibleWith(producibleType)) {
					compatibleMediaTypes.add(getMostSpecificMediaType(requestedType, producibleType));
				}
			}
		}
		if (compatibleMediaTypes.isEmpty()) {
			if (outputValue != null) {
				throw new HttpMediaTypeNotAcceptableException(producibleMediaTypes);
			}
			return;
		}

		// 将c排序
		List<MediaType> mediaTypes = new ArrayList<MediaType>(compatibleMediaTypes);
		MediaType.sortBySpecificityAndQuality(mediaTypes);

		// 从c中得到排序较靠前的某个Concrete返回格式 d
		MediaType selectedMediaType = null;
		for (MediaType mediaType : mediaTypes) {
			if (mediaType.isConcrete()) {
				selectedMediaType = mediaType;
				break;
			}
			else if (mediaType.equals(MediaType.ALL) || mediaType.equals(MEDIA_TYPE_APPLICATION)) {
				selectedMediaType = MediaType.APPLICATION_OCTET_STREAM;
				break;
			}
		}

		// 以下根据d得到对应的messageConverter，输出相应格式的结果
		if (selectedMediaType != null) {
			selectedMediaType = selectedMediaType.removeQualityValue();
			for (HttpMessageConverter<?> messageConverter : this.messageConverters) {
				if (messageConverter instanceof GenericHttpMessageConverter) {
					if (((GenericHttpMessageConverter) messageConverter).canWrite(
							declaredType, valueType, selectedMediaType)) {
						outputValue = (T) getAdvice().beforeBodyWrite(outputValue, returnType, selectedMediaType,
								(Class<? extends HttpMessageConverter<?>>) messageConverter.getClass(),
								inputMessage, outputMessage);
						if (outputValue != null) {
							addContentDispositionHeader(inputMessage, outputMessage);
							((GenericHttpMessageConverter) messageConverter).write(
									outputValue, declaredType, selectedMediaType, outputMessage);
							if (logger.isDebugEnabled()) {
								logger.debug("Written [" + outputValue + "] as \"" + selectedMediaType +
										"\" using [" + messageConverter + "]");
							}
						}
						return;
					}
				}
				else if (messageConverter.canWrite(valueType, selectedMediaType)) {
					outputValue = (T) getAdvice().beforeBodyWrite(outputValue, returnType, selectedMediaType,
							(Class<? extends HttpMessageConverter<?>>) messageConverter.getClass(),
							inputMessage, outputMessage);
					if (outputValue != null) {
						addContentDispositionHeader(inputMessage, outputMessage);
						((HttpMessageConverter) messageConverter).write(outputValue, selectedMediaType, outputMessage);
						if (logger.isDebugEnabled()) {
							logger.debug("Written [" + outputValue + "] as \"" + selectedMediaType +
									"\" using [" + messageConverter + "]");
						}
					}
					return;
				}
			}
		}
```


再来看看ExceptionHandlerExceptionResolver和RequestMappingHandlerAdapter中的RequestResponseBodyMethodProcessor为什么有不一样的messageConverters呢？

跟踪不到setMessageConverters的直接调用，两个类的setMessageConverters都分别调用了2次，一次7个，一次9个，AnnotationDrivenBeanDefinitionParser#parse也分别执行了两次，一次7，一次9，可以推测是这个类触发了eher和rmhq的实例化.

```java
	public void setMessageConverters(List<HttpMessageConverter<?>> messageConverters) {
		this.messageConverters = messageConverters;
	}
```

那么，问题又回到，为什么异常处理的时候，用到的eher实例是7个的那个呢？

看不动了，缓缓。需要从上至下了解spring的启动。

