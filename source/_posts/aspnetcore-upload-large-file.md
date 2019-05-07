---
title: AspNetCore 上传大文件
date: 2019-05-07 20:34:22
tags:
    - aspnetcore
    - IIS
---

# 背景

AspNetCore 在上传较大文件时收到错误 404.13

# 环境

````
aspnetcore-version: 2.1
host-server: windows
use-iis: true
````

# 原因

[New default 30 MB (~28.6 MiB) max request body size limit](https://github.com/aspnet/Announcements/issues/267)

# IIS处理

针对Kestrel的修改，无法修改IIS的默认设置；本地开始时如果使用IISExpress 或 生产部署在IIS中依然会碰到这个问题。

# 通过web.config解决IIS的问题

`dotnet publish`命令会生成默认的web.config文件，在尽量不修改默认行为下，可做如下处理：

假设 BuildConfig=Release
新建 web.Release.config
在文件输入
````
<?xml version="1.0" encoding="utf-8"?>
<configuration  xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">

  <!-- To customize the asp.net core module uncomment and edit the following section. 
  For more info see https://go.microsoft.com/fwlink/?linkid=838655 -->
  <!--
  <system.webServer>
    <handlers>
      <remove name="aspNetCore"/>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified"/>
    </handlers>
    <aspNetCore processPath="%LAUNCHER_PATH%" arguments="%LAUNCHER_ARGS%" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" />
  </system.webServer>
  -->
	<location>
		<system.webServer>
			<security xdt:Transform="InsertIfMissing">
				<requestFiltering xdt:Transform="InsertIfMissing">
          <!-- Measured in Bytes -->
          <!-- 300MB = 314572800 = 300 * 1024 * 1024 -->
					<requestLimits xdt:Transform="InsertIfMissing" maxAllowedContentLength="314572800"/> 
				</requestFiltering>
			</security>
		</system.webServer>
	</location>
</configuration>
````