# Storm.ImageProcessor.Web

This library extends the Azure blob storage functionality within the excellent [ImageProcessor](http://imageprocessor.org) by [James jackson-South](https://about.me/james_south) to support management of configuration settings via the ```<appSettings/>``` configuration section.

## Purpose

The allow a project to benefit from the ImageProcessor library and its ability to offload media to external blob storage accounts while removing the need for sensitive, environment specific information to be stored within library specific configuration files.

## Implementation

The library provides derived implementations of the ```AzureBlobCache``` and ```CloudImageService``` that look to read and augment the standard configuration section settings with values provided within the ```<appSettings/>``` section.

## Installation

There are 2 separate nuget packages available, one specifically for ImageProcessor and the other specific to the file system abstraction within Umbraco.  If you are using this within an Umbraco project you will need both packages installed:

```powershell
Install-Package Storm.ImageProcessor.Web
```

## Configuring Storm.ImageProcessor.Web

Once installed via nuget both the ```cache.config``` and ```security.config``` files will be updated.  Each will contain specific cache and service configuration to ensure image processor uses the derived implementations found within this library.

### config/imageprocessor/cache.config

```xml
    <cache name="FromConfigAzureBlobCache" type="Storm.ImageProcessor.Web.Plugins.AzureBlobCache.FromConfigAzureBlobCache, Storm.ImageProcessor.Web" maxDays="365">
      <settings>
        <setting key="CachedBlobContainer" value="cache" />
        <setting key="UseCachedContainerInUrl" value="true" />
        <setting key="CachedCDNTimeout" value="1000" />
        <setting key="StreamCachedImage" value="false" />
        <setting key="SourceBlobContainer" value="media" />
      </settings>
    </cache>
```
Once configured you can set the `currentCache` value to the name of your custom cache.

The same settings keys can be set within the configuration, however additionally the ```web.config``` will be searched for updates to any supplied values, if a value is found within the ```<appSettings/>``` then its value will override any set in the ```cache.config```:

```xml
  <appSettings>
    <add key="AzureBlobCache.CachedStorageAccount" value="DefaultEndpointsProtocol=https;AccountName=[CacheAccountName];AccountKey=[CacheAccountKey]" />
    <add key="AzureBlobCache.CachedCDNRoot" value="[CdnRootUrl]" />
  </appSettings>
```

### config/imageprocessor/security.config

```xml
    <!-- Minimal required configuration -->
    <service prefix="media/" name="CloudImageService" type="Storm.ImageProcessor.Web.Services.FromConfigCloudImageService, Storm.ImageProcessor.Web" />    
    
    <!-- Optional configuration -->
    <service prefix="media/" name="CloudImageService" type="Storm.ImageProcessor.Web.Services.FromConfigCloudImageService, Storm.ImageProcessor.Web">
      <settings>
        <setting key="MaxBytes" value="8194304"/> <!-- sets the default value -->
        <setting key="AppSettingsPrefix" value="MyPrefix"/> <!-- alters the detault appsettings prefix from 'CloudImageService' to 'MyPrefix' -->
      </settings>
    </service>
```

The same settings keys can be set within the configuration, however additionally the ```web.config``` will be searched for updates to any supplied values, if a value is found within the ```<appSettings/>``` then its value will override any set in the ```cache.config```:

```xml
  <appSettings>
    <add key="CloudImageService.Host" value="https://[account].blob.core.windows.net/media" />
    <add key="MyPrefix.Host" value="https://[account].blob.core.windows.net/media" /> <!-- if 'AppSettingsPrefix' altered as shown above -->
  </appSettings>
```
