---
title: AspNetCore work with Azure KeyVault
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
aspnetcore-version: 3.0
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

## AspNetCore KeyVault Configuration Provider

### appsettings.json 配置
```
{
  "KeyVault": {
    "ClientId": "<Your ClientId>",
    "ClientSecret": "<Your ClientSecret>",
    "DnsName": "<Your KeyVault DNS Name>"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
}
```

### Program.cs
```
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var configRoot = new ConfigurationBuilder()
                    .SetBasePath(hostingContext.HostingEnvironment.ContentRootPath)
                    .AddJsonFile("appsettings.json", false, true)
                    .AddJsonFile($"appsettings.{hostingContext.HostingEnvironment.EnvironmentName}.json", optional: true)
                    .Build();

                config.AddAzureKeyVault(configRoot.GetValue<string>("KeyVault:DnsName"), configRoot.GetValue<string>("KeyVault:ClientId"), configRoot.GetValue<string>("KeyVault:ClientSecret"), new DefaultKeyVaultSecretManager());
            })
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

### 重载配置

```
public HomeController(
    IConfiguration configuration,
    IOptionsSnapshot<CustomOptions> options,
    ILogger<HomeController> logger)
{
    _configuration = configuration as IConfigurationRoot;
    _options = options;
    _logger = logger;
}

public IActionResult Reload()
{
    _configuration.Reload();

    return RedirectToAction("Index");
}
```

### 注意事项

假设配置对象为 `Logging.LogLevel.Default`

在Azure App Service的Configuration中重写，节点名应该为： `Logging:LogLevel:Default`

在Key Vault中对应的Secret Name应该是： `Logging--LogLevel--Default`


## 使用KeyVault Keys进行加密/解密

### KeyVaultOptions

```
public class KeyVaultOptions
{
    public string ClientId { get; set; }

    public string ClientSecret { get; set; }

    public string DnsName { get; set; }

    public string CryptographyIdentity { get; set; }
}
```

### CryptographyProvider

```
public interface ICryptographyProvider
{
    Task<string> EncryptAsync(string plaintext);

    Task<string> DecryptAsync(string ciphertext);
}

public class KeyVaultCryptographtProvider : ICryptographyProvider
{
    public KeyVaultCryptographtProvider(IOptionsSnapshot<KeyVaultOptions> keyVaultOptions)
    {
        _options = keyVaultOptions;
        _client = new KeyVaultClient(async (string authority, string resource, string scope) =>
        {
            var authContext = new AuthenticationContext(authority);
            var clientCred = new ClientCredential(_options.Value?.ClientId, _options.Value?.ClientSecret);
            var result = await authContext.AcquireTokenAsync(resource, clientCred);
            if (result == null)
            {
                throw new InvalidOperationException("Failed to retrieve access token for Key Vault");
            }

            return result.AccessToken;
        });
    }

    private readonly IKeyVaultClient _client;
    private readonly IOptionsSnapshot<KeyVaultOptions> _options;

    private readonly string _algorithm = JsonWebKeyEncryptionAlgorithm.RSAOAEP;

    public async Task<string> DecryptAsync(string ciphertext)
    {
        var encryptedBytes = Convert.FromBase64String(ciphertext);
        var decryptResult = await _client.DecryptAsync(_options.Value?.CryptographyIdentity, _algorithm, encryptedBytes);
        if (decryptResult != null)
        {
            return Encoding.UTF8.GetString(decryptResult.Result);
        }

        return null;
    }

    public async Task<string> EncryptAsync(string plaintext)
    {
        var plaintextBytes = Encoding.UTF8.GetBytes(plaintext);
        var encryptResult = await _client.EncryptAsync(_options.Value?.CryptographyIdentity, _algorithm, plaintextBytes);
        return encryptResult == null ? null : Convert.ToBase64String(encryptResult.Result);
    }
}

```