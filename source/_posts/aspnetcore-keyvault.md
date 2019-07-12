---
title: aspnetcore-keyvault
date: 2019-07-12 11:47:43
tags:
    - aspnetcore
    - azure
    - key vault
---

# 背景

AspNetCore 使用Azure Key Vault 存储机密配置信息

# 环境

````
aspnetcore-version: 2.1+
azure-cloud: Azure China
````

# Source Code

```
https://github.com/zirain/AspNetCore-Playground/tree/master/src/Azure/Azure.KeyVault.Web
```

# 步骤

## 创建Key Vault

按照 [Azure 文档](https://docs.azure.cn/zh-cn/key-vault/quick-create-net) 创建资源

## 配置Access Policy

参考 [Stackoverflow](https://stackoverflow.com/questions/51124843/keyvaulterrorexception-operation-returned-an-invalid-status-code-forbidden) 设置Key Vault的Acess Policy

## AspNetCore Starup

```
        private readonly string AAD_CLIENT_ID = "<your-client-id>";
        private readonly string AAD_CLIENT_SECRET = "<your-client-secret>";
        private readonly string KEYVAULT_DNS_NAME = "you-keyvault-dns-name";

        public Startup(IWebHostEnvironment environment)
        {

            var kvClient = new KeyVaultClient(
                 async (string authority, string resource, string scope) =>
                 {
                     var authContext = new AuthenticationContext(authority);
                     var clientCred = new ClientCredential(AAD_CLIENT_ID, AAD_CLIENT_SECRET);
                     var result = await authContext.AcquireTokenAsync(resource, clientCred);
                     if (result == null)
                     {
                         throw new InvalidOperationException("Failed to retrieve access token for Key Vault");
                     }

                     return result.AccessToken;
                 }
             );
             
            Configuration = new ConfigurationBuilder()
                .SetBasePath(environment.ContentRootPath)
                .AddJsonFile("appsettings.json", false, true)
                .AddJsonFile($"appsettings.{environment.EnvironmentName}.json", optional: true)
                .AddAzureKeyVault(KEYVAULT_DNS_NAME, kvClient, new DefaultKeyVaultSecretManager())
                .AddEnvironmentVariables()
                .Build();
        }

        public IConfiguration Configuration { get; }
````

### 注意事项

假设配置对象为 `Logging.LogLevel.Default`

在Azure App Service的Configuration中重写，节点名应该为： `Logging:LogLevel:Default`

在Key Vault中对应的Secret Name应该是： `Logging--LogLevel--Default`