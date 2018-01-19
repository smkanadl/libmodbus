REM Requires PATH to point to vcpgk and nuget and NUGET_SOURCE defined to the nuget server url

vcpkg.exe export libmodbus:x86-windows libmodbus:x64-windows --nuget --nuget-id="libmodbus" --nuget-version="@LIBMODBUS_VERSION@"
IF ERRORLEVEL 1 GOTO End

NuGet.exe push .build\*.nupkg -Source %NUGET_SOURCE%
IF ERRORLEVEL 1 GOTO End

:End
exit %ERRORLEVEL%
