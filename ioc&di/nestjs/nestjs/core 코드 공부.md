# @nestjs/core 코드 공부

create (nest-factory.ts)

```typescript
public async create<T extends INestApplication = INestApplication>(
    module: any,
    serverOrOptions?: AbstractHttpAdapter | NestApplicationOptions,
    options?: NestApplicationOptions,
): Promise<T> {
    const [httpServer, appOptions] = this.isHttpServer(serverOrOptions)
        ? [serverOrOptions, options]
        : [this.createHttpAdapter(), serverOrOptions];

    const applicationConfig = new ApplicationConfig();
    const container = new NestContainer(applicationConfig);
    this.setAbortOnError(serverOrOptions, options);
    this.applyLogger(appOptions);
    await this.initialize(module, container, applicationConfig, httpServer);

    const instance = new NestApplication(
        container,
        httpServer,
        applicationConfig,
        appOptions,
    );
    const target = this.createNestInstance(instance);
    return this.createAdapterProxy<T>(target, httpServer);
}
```

1. httpServer와 옵션을 각각 변수에 저장함

> serverOrOptions는 express/fastify 인스턴스 또는 옵션인데 
> 
> serverOptions이 옵션인 경우, createHttpAdapter로 ExpressAdapter 인스턴스를 생성함

2. ApplicationConfig 인스턴스를 생성함 (전역 pipe, guard, transformer 등에 대한 정보 가지고 있음)

3. NestContainer 인스턴스를 생성함 (module 정보 가지고 있음)

4. 옵션으로 주어진 abortOnError값을 abortOnError 필드에 저장함

5. 옵션에 로깅 설정이 주어지지 않았다면 `Logger.overrideLogger`를 호출함

6. initialize 시작
    * container.setHttpAdapter(httpServer)
    * httpServer.init()
    * dependenciesScanner.scan(module)
        * container.createCoreModule // InternalCoreModule에 ExternalContextCreator, ModulesContainer, HttpAdapterHost
        * scanForModules(module) // 
        * registerCoreModuleRef
    * instanceLoader.createInstanceOfDependencies
        * container.getModules()
        * createPrototypes(modules);
        * createInstances