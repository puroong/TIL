# serverless-webpack으로 typeorm 번들링 했을 때 mysql2 포함 안되는 문제

## 원인

`TypeORM`이 `mysql2`를 **동적으로 임포트 하기 때문임**

TypeORM이 커넥션 생성하는 코드를 살펴보자

항상 index.ts의 `createConnection`으로 시작함

```typescript
export async function createConnection(optionsOrName?: any): Promise<Connection> {
    const connectionName = typeof optionsOrName === "string" ? optionsOrName : "default";
    const options = optionsOrName instanceof Object ? optionsOrName : await getConnectionOptions(connectionName);
    return getConnectionManager().create(options).connect();
}
```

ConnectionManager.ts의 create 함수

```typescript
create(options: ConnectionOptions): Connection {

    // check if such connection is already registered
    const existConnection = this.connections.find(connection => connection.name === (options.name || "default"));
    if (existConnection) {

        // if connection is registered and its not closed then throw an error
        if (existConnection.isConnected)
            throw new AlreadyHasActiveConnectionError(options.name || "default");

        // if its registered but closed then simply remove it from the manager
        this.connections.splice(this.connections.indexOf(existConnection), 1);
    }

    // create a new connection
    const connection = new Connection(options);
    this.connections.push(connection);
    return connection;
}
```

Connection.ts에서 생성자를 살펴보면

```typescript
constructor(options: ConnectionOptions) {
    this.name = options.name || "default";
    this.options = options;
    this.logger = new LoggerFactory().create(this.options.logger, this.options.logging);
    this.driver = new DriverFactory().create(this);
    this.manager = this.createEntityManager();
    this.namingStrategy = options.namingStrategy || new DefaultNamingStrategy();
    this.queryResultCache = options.cache ? new QueryResultCacheFactory(this).create() : undefined;
    this.relationLoader = new RelationLoader(this);
    this.relationIdLoader = new RelationIdLoader(this);
    this.isConnected = false;
}
```

DriverFactory.ts에 있는 create 함수

```typescript
create(connection: Connection): Driver {
    const {type} = connection.options;
    switch (type) {
        case "mysql":
            return new MysqlDriver(connection);
        case "postgres":
            return new PostgresDriver(connection);
        case "cockroachdb":
            return new CockroachDriver(connection);
        case "sap":
            return new SapDriver(connection);
        case "mariadb":
            return new MysqlDriver(connection);
        case "sqlite":
            return new SqliteDriver(connection);
        case "better-sqlite3":
            return new BetterSqlite3Driver(connection);
        case "cordova":
            return new CordovaDriver(connection);
        case "nativescript":
            return new NativescriptDriver(connection);
        case "react-native":
            return new ReactNativeDriver(connection);
        case "sqljs":
            return new SqljsDriver(connection);
        case "oracle":
            return new OracleDriver(connection);
        case "mssql":
            return new SqlServerDriver(connection);
        case "mongodb":
            return new MongoDriver(connection);
        case "expo":
            return new ExpoDriver(connection);
        case "aurora-data-api":
            return new AuroraDataApiDriver(connection);
        case "aurora-data-api-pg":
            return new AuroraDataApiPostgresDriver(connection);
        default:
            throw new MissingDriverError(type);
    }
}
```

MysqlDriver.ts의 생성자

```typescript
constructor(connection: Connection) {
        this.connection = connection;
        this.options = {
            legacySpatialSupport: true,
            ...connection.options
        } as MysqlConnectionOptions;
        this.isReplicated = this.options.replication ? true : false;

        // load mysql package
        this.loadDependencies();

        this.database = this.options.replication ? this.options.replication.master.database : this.options.database;

        // validate options to make sure everything is set
        // todo: revisit validation with replication in mind
        // if (!(this.options.host || (this.options.extra && this.options.extra.socketPath)) && !this.options.socketPath)
        //     throw new DriverOptionNotSetError("socketPath and host");
        // if (!this.options.username)
        //     throw new DriverOptionNotSetError("username");
        // if (!this.options.database)
        //     throw new DriverOptionNotSetError("database");
        // todo: check what is going on when connection is setup without database and how to connect to a database then?
        // todo: provide options to auto-create a database if it does not exist yet
    }
```

MysqlDriver.ts의 loadDependencies 함수

```typescript
 protected loadDependencies(): void {
    try {
        this.mysql = PlatformTools.load("mysql");  // try to load first supported package
        /*
            * Some frameworks (such as Jest) may mess up Node's require cache and provide garbage for the 'mysql' module
            * if it was not installed. We check that the object we got actually contains something otherwise we treat
            * it as if the `require` call failed.
            *
            * @see https://github.com/typeorm/typeorm/issues/1373
            */
        if (Object.keys(this.mysql).length === 0) {
            throw new Error("'mysql' was found but it is empty. Falling back to 'mysql2'.");
        }
    } catch (e) {
        try {
            this.mysql = PlatformTools.load("mysql2"); // try to load second supported package

        } catch (e) {
            throw new DriverPackageNotInstalledError("Mysql", "mysql");
        }
    }
}
```

PlatformTools.ts의 load 함수

```typescript
static load(name: string): any {

    // if name is not absolute or relative, then try to load package from the node_modules of the directory we are currently in
    // this is useful when we are using typeorm package globally installed and it accesses drivers
    // that are not installed globally

    try {

        // switch case to explicit require statements for webpack compatibility.

        switch (name) {

            /**
            * mongodb
            */
            case "mongodb":
                return require("mongodb");

            /**
            * hana
            */
            case "@sap/hana-client":
                return require("@sap/hana-client");

            case "hdb-pool":
                return require("hdb-pool");

            /**
            * mysql
            */
            case "mysql":
                return require("mysql");

            case "mysql2":
                return require("mysql2");

            /**
            * oracle
            */
            case "oracledb":
                return require("oracledb");

            /**
            * postgres
            */
            case "pg":
                return require("pg");

            case "pg-native":
                return require("pg-native");

            case "pg-query-stream":
                return require("pg-query-stream");

            case "typeorm-aurora-data-api-driver":
                return require("typeorm-aurora-data-api-driver");

            /**
            * redis
            */
            case "redis":
                return require("redis");

            case "ioredis":
                return require("ioredis");

            /**
                * better-sqlite3
                */
            case "better-sqlite3":
                return require("better-sqlite3");

            /**
            * sqlite
            */
            case "sqlite3":
                return require("sqlite3");

            /**
            * sql.js
            */
            case "sql.js":
                return require("sql.js");

            /**
            * sqlserver
            */
            case "mssql":
                return require("mssql");

            /**
                * react-native-sqlite
                */
            case "react-native-sqlite-storage":
                return require("react-native-sqlite-storage");
        }

    } catch (err) {
        return require(path.resolve(process.cwd() + "/node_modules/" + name));
    }

    // If nothing above matched and we get here, the package was not listed within PlatformTools
    // and is an Invalid Package.  To make it explicit that this is NOT the intended use case for
    // PlatformTools.load - it's not just a way to replace `require` all willy-nilly - let's throw
    // an error.
    throw new TypeError(`Invalid Package for PlatformTools.load: ${name}`);
}
```

를 통해  `mysql2` 패키지를 동적으로 임포트하는 것을 확인할 수 있음

## 해결 방법

`forceInclude` 옵션 사용하면 됨

```yaml
# serverless.yml
custom:
  webpack:
    includeModules:
      forceInclude:
        - module1
        - module2
```

[forceInclude 문서 링크](https://github.com/serverless-heaven/serverless-webpack#forced-inclusion)