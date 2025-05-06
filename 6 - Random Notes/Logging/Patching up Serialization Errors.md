
**PROBLEM:** `System.NotSupportedException: Serialization and deserialization of 'System.Reflection.MethodBase' instances is not supported.`

```C#
builder.Services.AddLoggingDebugConsoleAndFile(fileConfig =>
{
    fileConfig.MinimumLogLevel = Logging.LogLevel.HyperVerbose;
	// other properties...
	
	//  TypeInfoResolver - helps omit the problem
    fileConfig.JsonSerializerOptions = new JsonSerializerOptions
    {
        WriteIndented = false,
        ReferenceHandler = ReferenceHandler.IgnoreCycles,
        TypeInfoResolver = new DefaultJsonTypeInfoResolver
        {
            Modifiers = { IgnoreTargetSiteModifier } // Add our custom modifier
        }
    };
});
```

```C#
 // --- JSON TypeInfo Modifier to ignore Exception.TargetSite ---
 static void IgnoreTargetSiteModifier(JsonTypeInfo typeInfo)
 {
     if (typeInfo.Type != typeof(Exception) && !typeInfo.Type.IsSubclassOf(typeof(Exception)))
     {
         return; // Only modify Exception types
     }

     foreach (var propertyInfo in typeInfo.Properties)
     {
         // Check if the property is Exception.TargetSite
         if (propertyInfo.Name == nameof(Exception.TargetSite))
         {
             // Mark it as ignored for serialization
             propertyInfo.ShouldSerialize = (obj, value) => false;
         }
     }
 }
```

This occurs because System.Text.Json cannot serialize certain complex properties within the Exception object (like TargetSite, which involves reflection).