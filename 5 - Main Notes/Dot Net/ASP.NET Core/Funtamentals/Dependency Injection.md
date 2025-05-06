Source: https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection

Date: 24.04.2025 13:08

Tags: 

1. Which appsettings file is used in Debug?

When you run your ASP.NET Core application in Debug mode (typically triggered by Visual Studio or by setting the ASPNETCORE_ENVIRONMENT environment variable to Development), the framework automatically loads configuration sources in a specific order. By default, this includes:

1. appsettings.json: This file is always loaded first. It should contain your base configuration settings.

2. appsettings.Development.json: This file is loaded after appsettings.json if the environment is Development. Any settings in this file will override settings with the same name from appsettings.json.

3. Other sources like User Secrets (in Development), Environment Variables, and Command-line arguments are loaded afterwards, potentially overriding settings from the JSON files.

So, in Debug mode, you are effectively using the combined settings from appsettings.json and appsettings.Development.json, with the Development file taking precedence for any overlapping keys.
