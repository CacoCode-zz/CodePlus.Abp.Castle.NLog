# CodePlus.Abp.Castle.NLog

[![nuget](https://img.shields.io/nuget/v/CodePlus.Abp.Castle.NLog.svg?style=flat-square)](https://www.nuget.org/packages/CodePlus.Abp.Castle.NLog) 
[![Github Build Status](https://github.com/CacoCode/CodePlus.Abp.Castle.NLog/workflows/dotnetcorebuild/badge.svg?branch=master)](https://github.com/CacoCode/CodePlus.Abp.Castle.NLog/actions?query=workflow%3Adotnetcorebuild+branch%3Amaster)

## 概述

Abp NLog 集成Seq日志输出模块，版本跟随 ABP 版本

## Nuget

| **名称** |      **Nuget**      |
|----------|:-------------:|
| **CodePlus.Abp.Castle.NLog** | **[![NuGet](https://buildstats.info/nuget/CodePlus.Abp.Castle.NLog)](https://www.nuget.org/packages/CodePlus.Abp.Castle.NLog)** |

## 使用

1、安装Nuget Install-Package CodePlus.Abp.Castle.NLog

2、配置Nlog.config

```xml
<?xml version="1.0" encoding="utf-8"?>

<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true"
      internalLogLevel="Warn"
      internalLogFile="App_Data\Logs\nlogs.log"
      throwExceptions="true">

  <!-- 定义日志输出的根目录为web目录的上级目录 -->
  <variable name="logdir" value="${basedir}/App_Data/logs"/>
  
  <extensions>
    <add assembly="NLog.Web.AspNetCore"/>
    <add assembly="NLog.Targets.Seq"/>
  </extensions>
  <targets async="true">
    <default-target-parameters
    type="File"
    archiveAboveSize="50485760"
    maxArchiveFiles="50"
    archiveNumbering="Rolling"
    keepFileOpen="false"
    layout="${date:format=HH\:mm\:ss\:ffff}:[${level}] ${callsite} ${onexception:${exception:format=tostring} ${newline}${stacktrace}${newline}"/>

    <!--屏幕彩色打印消息-->
    <target name="console" xsi:type="ColoredConsole"
            layout="${date:format=HH\:mm\:ss\:ffff}:[${level}] ${message}"/>

    <!--默认日志-->
    <target xsi:type="File" name="defaultLog" fileName="${logdir}/${level}/${shortdate}.log" layout="${date:format=HH\:mm\:ss\:ffff}: ${message} ${onexception:${exception:format=tostring} ${newline}${stacktrace}${newline}" />

    <target name="warnLog" xsi:type="File"
            fileName="${logdir}/${level}/${shortdate}.log"
            layout="${date:format=HH\:mm\:ss\:ffff}:  ${logger}${newline}${message} ${onexception:${exception:format=tostring} ${newline}${stacktrace}${newline}" />
    
    <target name="seq" xsi:type="BufferingWrapper" bufferSize="1000" flushTimeout="2000">
      <target xsi:type="Seq" serverUrl="http://localhost:5341" apiKey="">
        <property name="ThreadId" value="${threadid}" as="number" />
        <property name="MachineName" value="${machinename}" />
        <property name="Environment" value="Development" />
        <!--https://github.com/NLog/NLog/wiki/Logger-Layout-Renderer-->
        <property name="Logger" value="${logger}" />
        <!--https://github.com/NLog/NLog/wiki/AspNet-Request-IP-Layout-Renderer-->
        <property name="IP" value="${aspnet-request-ip}" />
        <!--https://github.com/NLog/NLog/wiki/AspNetRequest-Url-Layout-Renderer-->
        <property name="Url" value="${aspnet-request-url:IncludeHost=true:IncludePort=true:IncludeQueryString=true:IncludeScheme=true}" />
        <!--HttpStatus仅最新的NLog.Web.AspNetCore包才支持，但不支持.NET Core 2.2-->
        <property name="Code" value="${aspnet-response-statuscode}" />       
      </target>
    </target>
  </targets>
  <rules>
    <logger name="*" minlevel="Warn" writeTo="seq" />
    <logger name="*" levels="Trace,Debug,Info" writeTo="console,defaultLog" />
    <logger name="*" minlevel="Warn" writeTo="console,warnLog" />
  </rules>
</nlog>
```

3、Program 添加 UseNlog

```csharp
public static IWebHost BuildWebHost(string[] args)
{
    return WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseNLog()
        .Build();
}
```

4、Startup  ConfigureServices 配置Nlog注入
```csharp
return services.AddAbp<CloudPlatformWebHostModule>(
    options => options.IocManager.IocContainer.AddFacility<LoggingFacility>(
        f => f.UseAbpNLog().WithConfig("nlog.config")
    )
);
```

