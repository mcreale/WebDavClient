# WebDAV .NET client [![Build status](https://ci.appveyor.com/api/projects/status/xee0yxvah59ffvd3?svg=true)](https://ci.appveyor.com/project/skazantsev/webdavclient)

Asynchronous cross-platform WebDAV client for .NET Core and other runtimes. It aims to have a full support of [RFC4918](https://tools.ietf.org/html/rfc4918).

## Installation
Install WebDav.Client via [NuGet](https://www.nuget.org/packages/WebDav.Client/).
```
Install-Package WebDav.Client
```

## Supported platforms
- .NET Core 1.0+
- .NET Framework 4.5+
- Mono
- Xamarin
- UWP

For more information see [.NET Standard](https://docs.microsoft.com/en-us/dotnet/standard/net-standard).

## Usage notes
`WebDavClient` uses `HttpClient` under the hood that is why it is a good practice to [share a single instance for the lifetime of the application](https://aspnetmonsters.com/2016/08/2016-08-27-httpclientwrong/).

If you use a dependency injection container to manage dependencies it is a good practice to register `WebDavClient` as a singleton.

It's also possible to instantiate `WebDavClient` with a pre-configured instance of `HttpClient`.

When using `GetRawFile` / `GetProcessedFile` don't forget to dispose the response.

## Usage examples

**Basic usage**
``` csharp
class Example
{
    public static IWebDavClient _client = new WebDavClient();

    public void MakeCalls()
    {
        var result = await _client.Propfind("http://mywebdav/1.txt");
        if (result.IsSuccessful)
            // continue ...
        else
            // handle an error
    }
}
```

**Using BaseAddress**
``` csharp
var clientParams = new WebDavClientParams { BaseAddress = new Uri("http://mywebdav/") };
using (var client = new WebDavClient(clientParams))
{
    await client.Propfind("1.txt");
}
```

**Operations with files and directories (resources & collections)**
``` csharp
var clientParams = new WebDavClientParams { BaseAddress = new Uri("http://mywebdav/") };
using (var client = new WebDavClient(clientParams))
{
    await client.Mkcol("mydir"); // create a directory

    await client.Copy("source.txt", "dest.txt"); // copy a file

    await client.Move("source.txt", "dest.txt"); // move a file

    await client.Delete("file.txt", "dest.txt"); // delete a file

    using (var response = await client.GetRawFile("file.txt")) // get a file without processing from the server
    {
        // use response.Stream
    }

    using (var response = await client.GetProcessedFile("file.txt")) // get a file that can be processed by the server
    {
        // use response.Stream
    }

    await client.PutFile("file.xml", File.OpenRead("file.xml")); // upload a resource
}
```

**Authentication using an access token**
``` csharp
var httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Authorization = 
    new AuthenticationHeaderValue("Bearer", accessToken);
var client = new WebDavClient(httpClient);
```

**Authentication using NetworkCredential**
``` csharp
var clientParams = new WebDavClientParams
{
    BaseAddress = new Uri("http://mywebdav/"),
    Credentials = new NetworkCredential("user", "12345")
};
var client = new WebDavClient(clientParams);
```

**PROPFIND example**
``` csharp
// list files & subdirectories in 'mydir'
var result = await _client.Propfind("http://mywebdav/mydir");
if (result.IsSuccessful)
{
    foreach (var res in result.Resources)
    {
        Trace.WriteLine("Name: " + res.DisplayName);
        Trace.WriteLine("Is directory: " + res.IsCollection);
        // etc.
    }
}
```

**PROPFIND with custom properties**
``` csharp
var propfindParams = new PropfindParameters
{
    Namespaces = new [] { new NamespaceAttr("myns", "https://example.com/") },
    CustomProperties = new [] { XName.Get("myprop", "https://example.com/") }
};
var result = await client.Propfind("http://mywebdav/mydir", propfindParams);
```

**Custom headers**
``` csharp
var propfindParams = new PropfindParameters
{
    Headers = new List<KeyValuePair<string, string>>
    {
        new KeyValuePair<string, string>("User-Agent", "Not a browser")
    }
};
var result = await _client.Propfind("http://mywebdav/1.txt", propfindParams);
```

**Content-Range or other content headers**
``` csharp
// Content headers need to be set directly on HttpContent instance.
var content = new StreamContent(File.OpenRead("test.txt"));
content.Headers.ContentRange = new ContentRangeHeaderValue(0, 2);
var result = await _client.PutFile("http://mywebdav/1.txt", content);
```

**Synchronous API**
``` csharp
  // will block the current thread, so use it cautiously
  var result = _client.Propfind("1.txt").Result;
```

## License
WebDavClient is licensed under the MIT License. See [LICENSE.txt](https://github.com/skazantsev/WebDavClient/blob/master/LICENSE.txt) for more details.
