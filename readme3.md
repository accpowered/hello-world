``` batch
@echo off
setlocal enabledelayedexpansion

:: Configuration Section
set "LOG_DIR=%~dp0Logs"   &:: Log storage directory (under script's directory)
set "TARGET_DRIVE=X:"     &:: Target drive letter

:: Create log directory
if not exist "%LOG_DIR%" mkdir "%LOG_DIR%"
set "LOG_FILE=%LOG_DIR%\%~n0_%date:~0,4%%date:~5,2%%date:~8,2%_%time:~0,2%%time:~3,2%.log"

:: Main loop: Introduction
echo Data Migration Tool (Designed by acc)
echo ========================================
echo This tool would run on following assumptions and instructions:
echo - This tool is running on old pc
echo - The new pc has been shared on local network with "C:\c" visible
echo - The old pc has a mapped drive "X:" directed to "C:\c" of the new pc
echo - Put default.txt in the same folder of this script and this script would try to migrate the paths you input in default.txt
echo - If default.txt is not found, this script would prompt you to input the folder path(s) you need to migrate
echo - Only accept folder path(s) as input, isolated file path(s) would be ignored
echo ========================================
echo.


:CLEAN_START
:: Initialize path counter
set "PATHS_COUNT=0"

:: Read saved_path.dat to see if any uncompleted copy task, only untick paths would be continued
if not exist "saved_path.dat" goto :INITIALIZE_SAVED_PATH

:READ_SAVED_PATH_ALL
set UNCOMPLETE_TASK_CNT=0
set index=0

for /f "tokens=1*" %%a in (saved_path.dat) do (
    set /a index+=1

    if "%%a"=="[x]" (
        set COMPLETE_FLAG[!index!]=1
    ) else (
        set COMPLETE_FLAG[!index!]=0
        set /a UNCOMPLETE_TASK_CNT+=1
    )
    set SOURCE_PATHS[!index!]=%%~b
    set /a PATHS_COUNT+=1
)

if %UNCOMPLETE_TASK_CNT%==0 goto :ASK_REFRESH_SAVED_PATH

:ASK_CONTINUE
type "saved_path.dat" & rem show saved_path.dat content
set /p user_input=Uncompleted task detected (as above), press any key to continue in 15 seconds(Input N to cancel):
if /i "!user_input!"=="N" goto :ASK_REFRESH_SAVED_PATH
timeout /t 15 /nobreak
color 09
goto :EXECUTE_COPY

:ASK_REFRESH_SAVED_PATH
echo Previous Tasks were completed or blank
set /p user_input2=Are you sure to start from clean? (Y/N): 
if /i !user_input2! NEQ Y (
echo Nothing changed, exit script...
pause
exit /b
)
Del saved_path.dat /q /f /s >nul
goto :CLEAN_START

:INITIALIZE_SAVED_PATH
echo.
echo Initialize logs\saved_path.dat
copy nul "saved_path.dat" >nul
echo.
goto :DEFAULT_PATH_HANDLING

:: Read default.txt and ask if user wants to use the recommended paths as migration paths
:DEFAULT_PATH_HANDLING
set "use_default="
set "default_paths="
if exist "default.txt" (
    set /p "use_default=Do you want to use the recommended paths as migration paths? (Y/N): "
    if /i "!use_default!"=="Y" (
        for /f "tokens=*" %%a in (default.txt) do (
            set "source=%%a"
            set /a PATHS_COUNT+=1
            set "SOURCE_PATHS[!PATHS_COUNT!]=!source!"
            set COMPLETE_FLAG[!PATHS_COUNT!]=0
            echo [-] !source! >> "saved_path.dat"
            )
    ) else (
        goto :INPUT_LOOP
    )
    goto :EXECUTE_COPY
)

:: Input loop
echo Manual input - Please input the folder path(s) you need to migrate [Press Enter (blank) to finish input]
echo.
:INPUT_LOOP
color 0B
set "source_path="
set /p "source_path=Enter source folder path: "

if "!source_path!"=="" goto :EXECUTE_COPY

:: Validate path existence
if not exist "!source_path!\" (
    echo [ERROR] Path does not exist: !source_path! >> "%LOG_FILE%"
    echo [ERROR] Path does not exist: !source_path!
    goto :INPUT_LOOP
)

:: Add to path list
set /a PATHS_COUNT+=1
set "SOURCE_PATHS[!PATHS_COUNT!]=!source_path!"
set COMPLETE_FLAG[!PATHS_COUNT!]=0
echo [-] !source! >> "saved_path.dat"
goto :INPUT_LOOP

:EXECUTE_COPY
if !PATHS_COUNT! LSS 1 (
    echo No valid paths provided. Exiting...
    pause
    exit /b
)

:: Execute copy operations
for /l %%i in (1,1,!PATHS_COUNT!) do (

    if !COMPLETE_FLAG[%%i]! == 0 (
        
        set "source=!SOURCE_PATHS[%%i]!"
        
        :: Extract relative path structure (remove drive letter)
        for %%d in ("!source!") do set "relative_path=%%~pnxd"
        
        :: Build target path
        set "target=!TARGET_DRIVE!!relative_path!"

        echo. >> "%LOG_FILE%"
        echo [START COPY] !source! -> !target! >> "%LOG_FILE%"
        echo Start Time: %date% %time% >> "%LOG_FILE%"
        
        :: Execute Robocopy
        echo Ready to copy to !target!
        timeout 5
        robocopy "!source!" "!target!" /E /Z /COPY:DAT /DCOPY:T /R:3 /W:5 /NP /TEE /UNILOG+:"%LOG_FILE%" /V /MT:4
        
        echo End Time: %date% %time% >> "%LOG_FILE%"
        echo ======================================== >> "%LOG_FILE%"
    )

    set COMPLETE_FLAG[%%i]=1

    :: rewrite saved_path.dat
    copy nul "saved_path.dat" >nul
    for /l %%m in (1,1,!PATHS_COUNT!) do (

        if !COMPLETE_FLAG[%%m]! == 1 (
            echo [x] !SOURCE_PATHS[%%m]! >> "saved_path.dat"
        ) else (
            echo [-] !SOURCE_PATHS[%%m]! >> "saved_path.dat"
        )
    )
)

echo.
echo Operation completed!
echo Log file location: "%LOG_FILE%"
pause
```



```batch
@echo off
setlocal enabledelayedexpansion

REM 设置默认虚拟环境路径（假设虚拟环境在脚本同一目录下）
set "VENV_PATH=%~dp0abc"

REM 检查虚拟环境是否存在
if not exist "%VENV_PATH%\Scripts\activate.bat" (
    echo 错误: 找不到虚拟环境
    echo 预期路径: %VENV_PATH%
    pause
    exit /b 1
)

REM 询问用户选择工作目录
echo 请选择Jupyter Notebook的工作目录:
echo 1 - 使用当前脚本所在目录
echo 2 - 使用我的文档目录
echo 3 - 使用桌面
echo 4 - 自定义路径
set /p "choice=请输入选项数字 (1-4): "

REM 根据用户选择设置工作目录
if "!choice!"=="1" (
    set "NOTEBOOK_DIR=%~dp0"
) else if "!choice!"=="2" (
    set "NOTEBOOK_DIR=%USERPROFILE%\Documents"
) else if "!choice!"=="3" (
    set "NOTEBOOK_DIR=%USERPROFILE%\Desktop"
) else if "!choice!"=="4" (
    set /p "NOTEBOOK_DIR=请输入完整路径: "
) else (
    echo 无效选择，使用默认目录(脚本所在目录)
    set "NOTEBOOK_DIR=%~dp0"
)

REM 检查目录是否存在，如果不存在则创建
if not exist "!NOTEBOOK_DIR!" (
    echo 目录不存在，正在创建...
    mkdir "!NOTEBOOK_DIR!"
)

REM 切换到指定目录
cd /d "!NOTEBOOK_DIR!"

REM 激活虚拟环境
call "%VENV_PATH%\Scripts\activate.bat"

REM 检查jupyter是否安装
where jupyter >nul 2>nul
if errorlevel 1 (
    echo 错误: 虚拟环境中未找到jupyter
    echo 请运行: pip install jupyter
    pause
    exit /b 1
)

echo 正在启动Jupyter Notebook...
echo 工作目录: !NOTEBOOK_DIR!
jupyter notebook

pause


```