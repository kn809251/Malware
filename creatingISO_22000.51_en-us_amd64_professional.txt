@echo off

rem script:	   @rgadguard and abbodi406

setlocal EnableExtensions
setlocal EnableDelayedExpansion
set "params=%*"
cd /d "%~dp0" && ( if exist "%temp%\getadmin.vbs" del "%temp%\getadmin.vbs" ) && fsutil dirty query %systemdrive% 1>nul 2>nul || (  echo Set UAC = CreateObject^("Shell.Application"^) : UAC.ShellExecute "cmd.exe", "/k cd ""%~sdp0"" && %~s0 %params%", "", "runas", 1 >> "%temp%\getadmin.vbs" && "%temp%\getadmin.vbs" && exit /B )

if not exist "%cd%\bin\wimlib-imagex.exe" goto :ConvertLite

:Info
for /f "tokens=3 delims=: " %%b in ('dism /english /online /Get-Intl ^| find /i "System locale"') do (
	call bin\lang-uup.cmd -en
	if /i %%b==ru-RU call bin\lang-uup.cmd -ru
)
set "file_main=%~n0"
set "aria2=bin\aria2c.exe"
set "rand=%random%"
set "down_temp=%rand%\down"
set "aria2Script=%rand%\aria2_script.txt"
set "BuildInfo=22000.51"
set "lang_def=en-us"
set "destDir=uup/!BuildInfo!/!lang_def!/amd64"
set "updateId=e012464d-2c1e-4c36-9051-caa99ff6f213"
mkdir %rand%

%aria2% -x16 -s16 -d"%rand%" -o"aria2_script.txt" "https://uup.rg-adguard.net/api/GetFiles?id=!updateId!&lang=!lang_def!&edition=professional&txt=yes"
if %ERRORLEVEL% GTR 0 goto DOWNERR
for %%i in ("%aria2Script%") do (if /i %%~zi LEQ 10 goto ERRAPI)

:Tools
set update_en=OFF
set update_num=0
for /f "tokens=1 skip=2" %%a in ('find /N "Windows10.0-KB" "%aria2Script%"') do (
	set /a update_num+=1
)
for /f "tokens=6 delims=[]. " %%G in ('ver') do (
	if %%G LEQ 9599 set update_num=0
)
if /i %update_num% neq 0 (set update_en=ON)

:DownFiles
title %lang_titke_download%...
if not exist "%destDir%\good" (
	%aria2% -x16 -s16 -j5 -c -R -d"%destDir%" -i"%aria2Script%"
	if %ERRORLEVEL% GTR 0 goto DOWNERR
	echo.>%destDir%\good
)
erase /q /s "%aria2Script%" >NUL 2>&1

color 0a
call bin\convert-UUP.cmd %cd%\uup\!BuildInfo!\!lang_def!\amd64 %update_en%
goto EOF

:ConvertLite
for /f "tokens=3 delims=:. " %%f in ('bitsadmin.exe /CREATE /DOWNLOAD "Download convert_lite" ^| findstr "Created job"') do set GUID=%%f
title Download convert_lite...
bitsadmin /transfer %GUID% /download /priority foreground https://uup.rg-adguard.net/dl/convert_lite.cab "%cd%\convert_lite.cab"
if NOT EXIST convert_lite.cab goto DOWNERR
expand convert_lite.cab -f:* %cd% >nul
del /f /q convert_lite.cab >nul 2>&1
goto :Info

:DOWNERR
color 0c
echo We have encountered an error while downloading files.
erase /q /s "%aria2Script%" >NUL 2>&1
pause
goto EOF

:ERRAPI
color 0c
echo Error getting links to download UUP files.
echo Try again a few minutes later.
erase /q /s "%aria2Script%" >NUL 2>&1
pause
goto EOF

:EOF
exit
