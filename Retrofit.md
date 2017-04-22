
``` java
    mFaceApi = new Retrofit.Builder().baseUrl(PrivateConstant.FACE_URL_PREFIX)
                                     .client(okHttpClient)
                                     .addConverterFactory(GsonConverterFactory.create(gson))
                                     .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                                     .build()
                                     .create(FaceApi.class);
```

#### Retrofit.java

###### `create`方法

``` java
  public <T> T create(final Class<T> service) {

    // …

    // Proxy.newProxyInstance：该方法生成一个动态代理类实例。
    //
    // 参数：
    // ClassLoader：类装载器
    // Class[]：一组接口
    // InvocationHandler：动态代理类关联的调用处理器，这个接口只有一个 invoke 方法，用于集中处理在动态代理类对象上的方法调用，通常在该方法中实现对委托类的代理访问。
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          //获取当前平台（Android or Java8 or default）
          private final Platform platform = Platform.get();

	  /**
	   * invoke 方法负责集中处理动态代理类上的所有方法调用
	   *
	   * proxy：代理类实例
	   * method：被调用的方法对象
	   * args：调用参数
	   */
          @Override public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            // …

	    // 通过参数 method 构建一个 ServiceMethod 对象
	    // loadServiceMethod 方法会先从缓存(ConcurrentHashMap)中获取，如果没有再进行创建，然后缓存
            ServiceMethod<Object, Object> serviceMethod = (ServiceMethod<Object, Object>) loadServiceMethod(method);
	    
            // 通过 ServiceMethod 和参数 args 创建 OkHttpCall
            // OkHttpCall 是对 OkHttp 的封装，内部包含网络请求需要的所有参数
            OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
  }
```

###### `loadServiceMethod`方法

``` java
  private final Map<Method, ServiceMethod<?, ?>> serviceMethodCache = new ConcurrentHashMap<>();

  ServiceMethod<?, ?> loadServiceMethod(Method method) {
    // 从缓存(ConcurrentHashMap)中获取
    ServiceMethod<?, ?> result = serviceMethodCache.get(method);
    if (result != null) return result;

    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        // 如果没有则通过Builder模式构建一个ServiceMethod，然后缓存
        result = new ServiceMethod.Builder<>(this, method).build();
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }
```

动态代理扩展


#### ServiceMethod.java

###### `ServiceMethod`的构建

``` java
    Builder(Retrofit retrofit, Method method) {
      this.retrofit = retrofit;
      this.method = method;
      this.methodAnnotations = method.getAnnotations();
      this.parameterTypes = method.getGenericParameterTypes();
      this.parameterAnnotationsArray = method.getParameterAnnotations();
    }

    public ServiceMethod build() {
      // ...
      for (Annotation annotation : methodAnnotations) {
        // 解析方法的注解信息(GET/POST/Multipart/FormUrlEncoded ...等等)
        parseMethodAnnotation(annotation);
      }

      // ...

      int parameterCount = parameterAnnotationsArray.length;
      parameterHandlers = new ParameterHandler<?>[parameterCount];
      for (int p = 0; p < parameterCount; p++) {
        // ...
        // 解析方法传入的参数信息(Url/Path/Query/Header/Field/Part/Body ...等等)
        parameterHandlers[p] = parseParameter(p, parameterType, parameterAnnotations);
      }

      // ...

      return new ServiceMethod<>(this);
    }
```

上面的解析完成后，在 `toRequest` 内通过 `RequestBuilder` 构建一个 `OkHttp` 的 `Request` 对象
``` java
  Request toRequest(Object... args) throws IOException {
    RequestBuilder requestBuilder = new RequestBuilder(httpMethod, baseUrl, relativeUrl, headers,
        contentType, hasBody, isFormEncoded, isMultipart);

    ParameterHandler<Object>[] handlers = (ParameterHandler<Object>[]) parameterHandlers;

    int argumentCount = args != null ? args.length : 0;
    if (argumentCount != handlers.length) {
      throw new IllegalArgumentException("Argument count (" + argumentCount
          + ") doesn't match expected count (" + handlers.length + ")");
    }

    for (int p = 0; p < argumentCount; p++) {
      handlers[p].apply(requestBuilder, args[p]);
    }

    return requestBuilder.build();
  }
```

在 `toResponse` 内通过之前 `addConverterFactory` 方法传入的数据转换实现类，将 `ResponseBody` 转换为需要的泛型
``` java
  R toResponse(ResponseBody body) throws IOException {
    return responseConverter.convert(body);
  }
```

