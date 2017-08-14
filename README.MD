# Compose and ASP.NET Core with SQL Server

## Build Docker Image

```
$ docker run -v ${PWD}:/app --workdir /app microsoft/aspnetcore-build:lts dotnet new mvc --auth Individual

Unable to find image 'microsoft/aspnetcore-build:lts' locally
lts: Pulling from microsoft/aspnetcore-build
ad74af05f5a2: Already exists
2b032b8bbe8b: Pull complete
a9a5b35f6ead: Pull complete
837d8daed36e: Pull complete
03ef1b1c8edc: Pull complete
1be1cfd14977: Pull complete
d41b351c258c: Pull complete
f0ab5fe4cfb6: Pull complete
5ee42508f0b8: Pull complete
89f31ec34e1b: Pull complete
Digest: sha256:7cbfc52455129acab5c26b20ed1d093a82c6fe456e4d99efcc978d46722b8a5b
Status: Downloaded newer image for microsoft/aspnetcore-build:lts
Content generation time: 1143.9801 ms
The template "ASP.NET Core Web App" created successfully.
```

## Create a Dockerfile

```
FROM microsoft/aspnetcore-build:lts
COPY . /app
WORKDIR /app
RUN ["dotnet", "restore"]
RUN ["dotnet", "build"]
EXPOSE 80/tcp
RUN chmod +x ./entrypoint.sh
CMD /bin/bash ./entrypoint.sh
```

## Add SQL Server Package

app.csproj
```
<ItemGroup>
    ......
    <PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="1.1.2" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite.Design" Version="1.1.2" PrivateAssets="All" />
    ......
</ItemGroup>
```


```
<ItemGroup>
    ......
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="1.1.2" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer.Design" Version="1.1.2" PrivateAssets="All" />
    ......
  </ItemGroup>
```


## Modify connection to inject SQL Server Service

```
[...]
public void ConfigureServices(IServiceCollection services)
{
    // Database connection string.
    // Make sure to update the Password value below from "your_password" to your actual password.
    var connection = @"Server=db;Database=master;User=sa;Password=Passw0rd;";

    // This line uses 'UseSqlServer' in the 'options' parameter
    // with the connection string defined above.
    services.AddDbContext<ApplicationDbContext>(
        options => options.UseSqlServer(connection));

    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    services.AddMvc();

    // Add application services.
    services.AddTransient<IEmailSender, AuthMessageSender>();
    services.AddTransient<ISmsSender, AuthMessageSender>();
}
[...]
```

## docker-compose build

```
$ docker-compose build
```

Make sure you allocate at least 4GB of memory to Docker Engine.


## docker-compose up

```
$ docker-compose up
Creating network "aspnetcoresql_default" with the default driver
Creating aspnetcoresql_db_1 ...
Creating aspnetcoresql_db_1 ... done
Creating aspnetcoresql_web_1 ...
Creating aspnetcoresql_web_1 ... error

ERROR: for aspnetcoresql_web_1  Cannot start service web: driver failed programming external connectivity on endpoint aspnetcoresql_web_1 (bb3582651071ae2855c9a62716542de2595c6356c93c95d848680db2de965b78): Bind for 0.0.0.0:8000 failed: port is already allocated

ERROR: for web  Cannot start service web: driver failed programming external connectivity on endpoint aspnetcoresql_web_1 (bb3582651071ae2855c9a62716542de2595c6356c93c95d848680db2de965b78): Bind for 0.0.0.0:8000 failed: port is already allocated
ERROR: Encountered errors while bringing up the project.
```


```
$ docker-compose up
Starting aspnetcoresql_db_1 ...
Starting aspnetcoresql_db_1 ... done
Starting aspnetcoresql_web_1 ...
Starting aspnetcoresql_web_1 ... done
Attaching to aspnetcoresql_db_1, aspnetcoresql_web_1
db_1   | This is an evaluation version.  There are [148] days left in the evaluation period.
db_1   | 2017-08-14 02:12:05.67 Server      Microsoft SQL Server 2017 (RC1) - 14.0.800.90 (X64)
db_1   |Jul 11 2017 07:03:16
db_1   |Copyright (C) 2017 Microsoft Corporation. All rights reserved.
db_1   |Developer Edition (64-bit) on Linux (Ubuntu 16.04.2 LTS)
2017-08-14 02:12:05.69 Server      UTC adjustment: 0:00
2017-08-14 02:12:05.69 Server      (c) Microsoft Corporation.
2017-08-14 02:12:05.69 Server      All rights reserved.
2017-08-14 02:12:05.70 Server      Server process ID is 4116.
2017-08-14 02:12:05.70 Server      Logging SQL Server messages in file '/var/opt/mssql/log/errorlog'.
2017-08-14 02:12:05.70 Server      Registry startup parameters:
db_1   | -d /var/opt/mssql/data/master.mdf
db_1   | -l /var/opt/mssql/data/mastlog.ldf
db_1   | -e /var/opt/mssql/log/errorlog
2017-08-14 02:12:05.71 Server      SQL Server detected 4 sockets with 1 cores per socket and 1 logical processors per socket, 4 total logical processors; using 4 logical processors based on SQL Server licensing. This is an informational message; no user action is required.
2017-08-14 02:12:05.72 Server      SQL Server is starting at normal priority base (=7). This is an informational message only. No user action is required.
2017-08-14 02:12:05.72 Server      Detected 3158 MB of RAM. This is an informational message; no user action is required.
2017-08-14 02:12:05.72 Server      Using conventional memory in the memory manager.
2017-08-14 02:12:06.01 Server      Buffer pool extension is already disabled. No action is necessary.
2017-08-14 02:12:06.24 Server      InitializeExternalUserGroupSid failed. Implied authentication will be disabled.
2017-08-14 02:12:06.25 Server      Implied authentication manager initialization failed. Implied authentication will be disabled.
2017-08-14 02:12:06.25 Server      Successfully initialized the TLS configuration. Allowed TLS protocol versions are ['1.0 1.1 1.2']. Allowed TLS ciphers are ['ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA'].
2017-08-14 02:12:06.33 Server      The maximum number of dedicated administrator connections for this instance is '1'
2017-08-14 02:12:06.33 Server      Node configuration: node 0: CPU mask: 0x000000000000000f:0 Active CPU mask: 0x000000000000000f:0. This message provides a description of the NUMA configuration for this computer. This is an informational message only. No user action is required.
2017-08-14 02:12:06.37 Server      Using dynamic lock allocation.  Initial allocation of 2500 Lock blocks and 5000 Lock Owner blocks per node.  This is an informational message only.  No user action is required.
2017-08-14 02:12:06.40 Server      In-Memory OLTP initialized on lowend machine.
2017-08-14 02:12:06.48 Server      Database Instant File Initialization: enabled. For security and performance considerations see the topic 'Database Instant File Initialization' in SQL Server Books Online. This is an informational message only. No user action is required.
2017-08-14 02:12:06.51 Server      Query Store settings initialized with enabled = 1,
2017-08-14 02:12:06.52 spid7s      Starting up database 'master'.
2017-08-14 02:12:06.54 Server      Software Usage Metrics is disabled.
2017-08-14 02:12:06.84 spid7s      106 transactions rolled forward in database 'master' (1:0). This is an informational message only. No user action is required.
2017-08-14 02:12:06.86 spid7s      0 transactions rolled back in database 'master' (1:0). This is an informational message only. No user action is required.
2017-08-14 02:12:06.87 spid7s      Recovery is writing a checkpoint in database 'master' (1). This is an informational message only. No user action is required.
2017-08-14 02:12:06.91 spid7s      Buffer pool extension is already disabled. No action is necessary.
2017-08-14 02:12:06.92 spid7s      Resource governor reconfiguration succeeded.
2017-08-14 02:12:06.92 spid7s      SQL Server Audit is starting the audits. This is an informational message. No user action is required.
2017-08-14 02:12:06.93 spid7s      SQL Server Audit has started the audits. This is an informational message. No user action is required.
2017-08-14 02:12:07.07 spid7s      SQL Trace ID 1 was started by login "sa".
2017-08-14 02:12:07.11 spid7s      Server name is '874410f09a86'. This is an informational message only. No user action is required.
2017-08-14 02:12:07.17 spid7s      Starting up database 'msdb'.
2017-08-14 02:12:07.20 spid22s     Always On: The availability replica manager is starting. This is an informational message only. No user action is required.
2017-08-14 02:12:07.23 spid9s      Starting up database 'mssqlsystemresource'.
2017-08-14 02:12:07.24 spid22s     Always On: The availability replica manager is waiting for the instance of SQL Server to allow client connections. This is an informational message only. No user action is required.
web_1  |
web_1  | Build succeeded.
web_1  |     0 Warning(s)
web_1  |     0 Error(s)
web_1  |
web_1  | Time Elapsed 00:00:04.22
2017-08-14 02:12:07.27 spid9s      The resource database build version is 14.00.800. This is an informational message only. No user action is required.
2017-08-14 02:12:07.31 spid9s      Starting up database 'model'.
2017-08-14 02:12:07.70 spid19s     A self-generated certificate was successfully loaded for encryption.
2017-08-14 02:12:07.74 spid19s     Server is listening on [ 'any' <ipv4> 1433].
2017-08-14 02:12:07.76 Server      Server is listening on [ 127.0.0.1 <ipv4> 1434].
2017-08-14 02:12:07.77 Server      Dedicated admin connection support was established for listening locally on port 1434.
2017-08-14 02:12:07.81 spid19s     SQL Server is now ready for client connections. This is an informational message; no user action is required.
2017-08-14 02:12:08.04 spid7s      9 transactions rolled forward in database 'msdb' (4:0). This is an informational message only. No user action is required.
2017-08-14 02:12:08.05 spid9s      1 transactions rolled forward in database 'model' (3:0). This is an informational message only. No user action is required.
2017-08-14 02:12:08.06 spid7s      0 transactions rolled back in database 'msdb' (4:0). This is an informational message only. No user action is required.
2017-08-14 02:12:08.07 spid9s      0 transactions rolled back in database 'model' (3:0). This is an informational message only. No user action is required.
2017-08-14 02:12:08.07 spid7s      Recovery is writing a checkpoint in database 'msdb' (4). This is an informational message only. No user action is required.
2017-08-14 02:12:08.08 spid9s      Recovery is writing a checkpoint in database 'model' (3). This is an informational message only. No user action is required.
2017-08-14 02:12:08.19 spid9s      Polybase feature disabled.
2017-08-14 02:12:08.19 spid9s      Clearing tempdb database.
2017-08-14 02:12:08.99 spid9s      Starting up database 'tempdb'.
2017-08-14 02:12:09.31 spid9s      The tempdb database has 1 data file(s).
2017-08-14 02:12:09.32 spid24s     The Service Broker endpoint is in disabled or stopped state.
2017-08-14 02:12:09.33 spid24s     The Database Mirroring endpoint is in disabled or stopped state.
2017-08-14 02:12:09.34 spid24s     Service Broker manager has started.
2017-08-14 02:12:09.44 spid7s      Recovery is complete. This is an informational message only. No user action is required.
web_1  | Done.
web_1  | SQL Server is up - executing command
web_1  | Hosting environment: Production
web_1  | Content root path: /app
web_1  | Now listening on: http://+:80
web_1  | Application started. Press Ctrl+C to shut down.
Killing aspnetcoresql_web_1 ... done
Killing aspnetcoresql_db_1 ... done
```
