This is related to [this issue](https://github.com/SAFE-Stack/SAFE-template/issues/621), around a breaking change somewhere after .NET 8.0.206 that is impacted (as far as I can tell) from the following convergence:

1. You are using the `Microsoft.NET.Sdk.Web` SDK.
1. You are using an SDK later than 8.0.206 (I've used up to 401 and can reliably reproduce this).
1. You are using the Paket tool as your .NET package management client.

If all of these things are true and you `dotnet watch build` or `run` from the folder of the project, things go wrong. I suspect it's something that's changed since 8.0.206 (or the Sdk.Web) that has somehow broken Paket.

We have seen this with multiple customers of SAFE Stack, which uses Paket and ASP .NET Core - when people upgraded to 8.0.3xx, things started going wrong with no code changes at all.

This repository contains a working reproduction of the issue.

Here are some steps to reproduce:

1. Install a version of .NET 8.0 after 8.0.206 (I'm using 8.0.401).
1. Make a clean folder and `cd` to it.
1. `dotnet new globaljson`
1. Set it to e.g.

```json
{
    "sdk": {
        "version": "8.0.401"
    }
}
```

1. `dotnet new console -lang F#`
1. `dotnet tool install paket --create-manifest-if-needed`
1. `dotnet paket add FSharp.Core`

Now, observe the following behaviours:

### dotnet watch build ❌

```cmd
dotnet watch � Started
  Determining projects to restore...
  Unhandled exception. System.IO.FileNotFoundException: Could not load file or assembly 'System.Runtime, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a'. The system cannot find the file specified.
  File name: 'System.Runtime, Version=6.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a'
     at System.Reflection.RuntimeAssembly.GetType(QCallAssembly assembly, String name, Boolean throwOnError, Boolean ignoreCase, ObjectHandleOnStack type, ObjectHandleOnStack keepAlive, ObjectHandleOnStack assemblyLoadContext)
     at System.Reflection.RuntimeAssembly.GetType(String name, Boolean throwOnError, Boolean ignoreCase)
     at System.Reflection.Assembly.GetType(String name, Boolean throwOnError)
     at System.StartupHookProvider.CallStartupHook(StartupHookNameOrPath startupHook)
     at System.StartupHookProvider.ProcessStartupHooks()


D:\code\runthru\.paket\Paket.Restore.targets(171,3): error MSB3073: The command "dotnet paket restore" exited with code -532462766. [D:\code\runthru\runthru.fsproj]

Build FAILED.
```

### `dotnet watch run` ❌
```cmd
error MSB3073: The command "dotnet paket restore" exited with code -532462766
```

## Workarounds
* Don't use `watch`; `dotnet build` and `dotnet run` on their own work. ✅
* Change `global.json` to pin to 8.0.206. ✅
* Change the `fsproj` SDK to `Microsoft.NET.Sdk` ✅
* Add `--no-restore` to either `watch` or `build`.  ✅
