---
title: EF Core Migrations and Serilog in ASPNet Core
description: Posting a note about using EF Core Migrations while using Serilog in a ASPNet Core application.
tags: 'dotnet, csharp, aspnet, efcore'
published: true
id: 512734
date: '2020-11-12T05:58:21.112Z'
---
Posting a note about using EF Core Migrations while using Serilog in a ASPNet Core application.

A common `Program.cs` in a aspnet core 3.1 application using Serilog might look something like this:

```
 public static void Main(string[] args)
        {
            try
            {
                using IHost host = Host.CreateDefaultBuilder(args)
                    .AddJsonFile("extra-config-file.json")
                    .ConfigureWebHostDefaults(webBuilder => webBuilder.UseStartup<Startup>())
                    .Build();
                host.Run();
            }
            catch (Exception ex)
            {
                if (Log.Logger == null || Log.Logger.GetType().Name == "SilentLogger")
                {
                    Log.Logger = new LoggerConfiguration()
                        .MinimumLevel.Debug()
                        .WriteTo.Console()
                        .CreateLogger();
                }

                Log.Fatal(ex, "Host terminated unexpectedly");
            }
            finally
            {
                Log.CloseAndFlush();
            }
        }
``` 

This is modified from the typical file created by the aspnet core project template. The changes are to enable Serilog to exit gracefully if the application crashes or otherwise stops unexpectedly.

This can cause issues when trying to use the dotnet ef CLI. Typically you will encounter this if the DbContext is in a different project that the startup project.

If the `dotnet ef` command fails, try running it again with the `-v` switch. You may see a message printed that says:
> No static method 'CreateHostBuilder(string[])' was found on class 'Program'

The tool is looking for a method that exists in the typical Program.cs file. Here is an example of a typical file without Serilog:
```
 public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }
 
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            bHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>();
    }
```
Notice the additional static `CreateHostBuilder` function. This was refactored out of the example using Serilog. To fix the issue with the CLI tools, simply refactor this method back into the file. The first example would end up looking something like this:

```
 public class Program
    {
        public static IHostBuilder CreateWebHostBuilder(string[] args) 
            => Host.CreateDefaultBuilder(args)
                 .AddJsonFile("extra-config-file.json")
                 .ConfigureWebHostDefaults(webBuilder => webBuilder.UseStartup<Startup>());


        public static void Main(string[] args)
        {
            try
            {

                using var host = CreateWebHostBuilder(args).Build();
                host.Run();
            }
            catch (Exception ex)
            {
                if (Log.Logger == null || Log.Logger.GetType().Name == "SilentLogger")
                {
                    Log.Logger = new LoggerConfiguration()
                        .MinimumLevel.Debug()
                        .WriteTo.Console()
                        .CreateLogger();
                }

                Log.Fatal(ex, "Host terminated unexpectedly");
            }
            finally
            {
                Log.CloseAndFlush();
            }
        }

    }

```

This allows Entity Framework Core CLI tools to find and execute this method. Now your DbContext should be able to get the connection string from your configuration just as it does at runtime.

