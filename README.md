# NASB

Repository for research into NASB mechanics


## How to get started

### Download the Game:

If youre not on windows, you won't be able to download it with Steam. Use [https://developer.valvesoftware.com/wiki/SteamCMD](SteamCMD):

```
 ./steamcmd.sh +@sSteamCmdForcePlatformType windows
login <username>
force_install_dir <path>
app_update 1414850
```

where `<path>` = the path to install to 

### Install ilspy 

Install the `dotnet` utility, which is included with [.NET Core](https://docs.microsoft.com/en-us/dotnet/core/tools/)

Then use that to install ilspycmd:
```
dotnet tool install ilspycmd -g
```

### Decompile

The majority of the code lives in `Nickelodeon All-Star Brawl_Data/Managed/Assembly-CSharp.dll`

Run ilspy on it to disassemble it:

```
ilspycmd Assembly-CSharp.dll > ~/dis/Assembly-CSharp.dll.cs
```

### Profit

Open it in your favorite C# editor (JetBrains makes a good one, but Sublime works as well). Enjoy your reversing!
