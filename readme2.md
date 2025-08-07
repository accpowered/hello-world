```bash
@echo off
setlocal enabledelayedexpansion

:: Configuration Section
set "LOG_DIR=%~dp0Logs"   &:: Log storage directory (under script's directory)
set "TARGET_DRIVE=X:"     &:: Target drive letter

:: Initialize path counter
set "PATHS_COUNT=0"

:: Create log directory
if not exist "%LOG_DIR%" mkdir "%LOG_DIR%"
set "LOG_FILE=%LOG_DIR%\%~n0_%date:~0,4%%date:~5,2%%date:~8,2%_%time:~0,2%%time:~3,2%.log"

:: Main loop: Prompt for source paths
echo Data Migration Tool (Press Enter to finish input)
echo ========================================
:INPUT_LOOP
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
goto :INPUT_LOOP

:EXECUTE_COPY
if !PATHS_COUNT! LSS 1 (
    echo No valid paths provided. Exiting...
    pause
    exit /b
)

:: Execute copy operations
for /l %%i in (1,1,!PATHS_COUNT!) do (
    set "source=!SOURCE_PATHS[%%i]!"
    
    :: Extract relative path structure (remove drive letter)
    for %%d in ("!source!") do set "relative_path=%%~pnx"
    
    :: Build target path
    set "target=!TARGET_DRIVE!\!relative_path!"
    
    echo. >> "%LOG_FILE%"
    echo [START COPY] !source! -> !target! >> "%LOG_FILE%"
    echo [START COPY] !source! -> !target!
    echo Start Time: %date% %time% >> "%LOG_FILE%"
    
    :: Execute Robocopy
    robocopy "!source!" "!target!" /E /Z /COPY:DAT /DCOPY:T /R:3 /W:5 /NP /TEE /LOG+:"%LOG_FILE%" /V
    
    echo End Time: %date% %time% >> "%LOG_FILE%"
    echo ======================================== >> "%LOG_FILE%"
)

echo.
echo Operation completed!
echo Log file location: "%LOG_FILE%"
pause


```










```bash

以下是一个使用Robocopy的批处理脚本，满足您的所有需求。它会逐个询问源文件夹路径，保留目录结构复制到X盘，支持断点续传、非管理员运行，并记录详细日志：

```batch
@echo off
setlocal enabledelayedexpansion

:: 配置区域
set "LOG_DIR=%~dp0Logs"   &:: 日志存放目录（当前脚本目录下）
set "TARGET_DRIVE=X:"     &:: 目标驱动器

:: 创建日志目录
if not exist "%LOG_DIR%" mkdir "%LOG_DIR%"
set "LOG_FILE=%LOG_DIR%\%~n0_%date:~0,4%%date:~5,2%%date:~8,2%_%time:~0,2%%time:~3,2%.log"

:: 主循环：询问源路径
echo 数据迁移工具 (按回车键结束输入)
echo ========================================
:INPUT_LOOP
set "source_path="
set /p "source_path=请输入源文件夹路径: "

if "!source_path!"=="" goto :EXECUTE_COPY

:: 验证路径有效性
if not exist "!source_path!\" (
    echo [错误] 路径不存在: !source_path! >> "%LOG_FILE%"
    echo [错误] 路径不存在: !source_path!
    goto :INPUT_LOOP
)

:: 添加到路径列表
set "PATHS_COUNT=!PATHS_COUNT!1"
set "SOURCE_PATHS[!PATHS_COUNT!]=!source_path!"
goto :INPUT_LOOP

:EXECUTE_COPY
if !PATHS_COUNT! LSS 1 (
    echo 未提供有效路径，程序退出
    pause
    exit /b
)

:: 执行复制操作
for /l %%i in (1,1,!PATHS_COUNT!) do (
    set "source=!SOURCE_PATHS[%%i]!"
    
    :: 提取源路径结构（去掉盘符）
    for %%d in (!source!) do set "relative_path=%%~pnxd"
    
    :: 构建目标路径
    set "target=!TARGET_DRIVE!\!relative_path!"
    
    echo. >> "%LOG_FILE%"
    echo [开始复制] !source! -> !target! >> "%LOG_FILE%"
    echo [开始复制] !source! -> !target!
    echo 开始时间: %date% %time% >> "%LOG_FILE%"
    
    :: 执行Robocopy
    robocopy "!source!" "!target!" /E /Z /COPY:DAT /DCOPY:T /R:3 /W:5 /NP /TEE /LOG+:"%LOG_FILE%" /V
    
    echo 结束时间: %date% %time% >> "%LOG_FILE%"
    echo ======================================== >> "%LOG_FILE%"
)

echo.
echo 操作已完成！
echo 日志文件保存位置: "%LOG_FILE%"
pause
```

### 功能说明：
1. **路径处理**：
   - 交互式询问源文件夹路径（支持多个路径输入）
   - 自动将 `C:\Folder1\Folder2` 映射为 `X:\Folder1\Folder2`
   - 自动验证路径有效性

2. **Robocopy 参数**：
   - `/E`：复制所有子目录（包括空目录）
   - `/Z`：支持断点续传（重启后可继续复制）
   - `/COPY:DAT`：复制文件数据、属性和时间戳（无需管理员权限）
   - `/DCOPY:T`：复制目录时间戳
   - `/R:3 /W:5`：失败重试3次，每次等待5秒
   - `/NP`：不显示进度百分比（保持日志整洁）
   - `/TEE`：输出同时显示在屏幕和日志
   - `/V`：生成详细日志
   - `/LOG+`：追加模式记录日志（保留历史记录）

3. **日志功能**：
   - 自动在脚本目录创建 `Logs` 文件夹
   - 日志命名格式：`脚本名_年月日_时分.log`
   - 记录每个操作的开始/结束时间
   - 包含成功/失败的文件详细信息

4. **安全特性**：
   - 无需管理员权限
   - 路径验证防止无效输入
   - 自动跳过空路径输入

### 使用示例：
```
请输入源文件夹路径: C:\Projects\Alpha
请输入源文件夹路径: D:\UserData\Documents
请输入源文件夹路径: (直接回车开始复制)

[开始复制] C:\Projects\Alpha -> X:\Projects\Alpha
[开始复制] D:\UserData\Documents -> X:\UserData\Documents
```

### 注意事项：
1. 目标驱动器必须是 `X:`（脚本第5行可修改）
2. 日志文件会保存在脚本所在目录的 `Logs` 文件夹
3. 按回车键结束路径输入并开始复制
4. 如果复制中断，重新运行脚本会继续上次的复制进度

> 提示：首次运行时如果提示`Logs`目录创建失败，请以管理员身份运行一次（后续使用无需管理员），或手动创建日志目录。


```









```python
import time
import csv
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from webdriver_manager.chrome import ChromeDriverManager
from bs4 import BeautifulSoup


def extract_table_data(html_source):
    """使用BeautifulSoup从HTML源码中提取表格数据"""
    soup = BeautifulSoup(html_source, 'html.parser')
    rows = []
    
    # 查找所有AngularJS动态生成的表格行项目
    items = soup.find_all(attrs={"ng-repeat": lambda x: x and "item in" in x})
    
    for item in items:
        # 提取当前行所有文本项
        text_items = item.find_all(attrs={"ng-repeat": "textItem in item.textItems"})
        if text_items:
            row_data = [span.get_text(strip=True) for span in text_items]
            rows.append(row_data)
    
    # 移除可能的空行
    return [row for row in rows if any(row)]


def main():
    # 配置Chrome浏览器
    driver = webdriver.Chrome(executable_path=ChromeDriverManager().install())
    url = "https://sense-demo.qlik.com/sense/app/069279ac-a7e8-4405-826d-0cd3de7d48e0/sheet/CmbWz/state/analysis/overlay/selections"

    try:
        # 打开目标网页
        driver.get(url)
        print("网页已打开，请手动滚动表格以加载数据...")
        print("完成后请按Ctrl+C停止程序")

        # 等待页面加载完成（等待表格出现）
        WebDriverWait(driver, 20).until(
            EC.presence_of_element_located((By.CSS_SELECTOR, 'span[ng-repeat*="textItem in item.textItems"]'))
        )

        # 存储提取的数据和已处理的行
        all_data = []
        processed_rows = set()
        first_iteration = True

        while True:
            # 获取当前页面源码
            html_source = driver.page_source
            # 提取表格数据
            current_rows = extract_table_data(html_source)

            # 处理新数据
            new_rows_count = 0
            for row in current_rows:
                # 将行转换为元组以便哈希
                row_tuple = tuple(row)
                if row_tuple not in processed_rows:
                    processed_rows.add(row_tuple)
                    all_data.append(row)
                    new_rows_count += 1

            # 首次运行时显示表头
            if first_iteration and all_data:
                print("检测到表头:\n", all_data[0])
                first_iteration = False

            # 显示进度信息
            if new_rows_count > 0:
                print(f"发现{new_rows_count}条新数据，总计{len(all_data)}条")

            # 等待一段时间再检查新数据
            time.sleep(1)

    except KeyboardInterrupt:
        print("\n用户中断，正在保存数据...")
    except Exception as e:
        print(f"发生错误: {str(e)}")
    finally:
        # 保存数据到CSV文件
        if all_data:
            with open('qlik_table_data.csv', 'w', newline='', encoding='utf-8') as f:
                writer = csv.writer(f)
                writer.writerows(all_data)
            print(f"数据已保存到qlik_table_data.csv，共{len(all_data)}行")
        else:
            print("未提取到任何数据")

        # 关闭浏览器
        driver.quit()


if __name__ == "__main__":
    main()
```


```python
import time
import csv
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from webdriver_manager.chrome import ChromeDriverManager
from bs4 import BeautifulSoup


def extract_table_data(html_source):
    """使用BeautifulSoup从HTML源码中提取表格数据"""
    soup = BeautifulSoup(html_source, 'html.parser')
    rows = []
    
    # 查找所有AngularJS动态生成的表格行项目
    # 查找包含可选标签和标题属性的元素（不限定标签类型）
    items = soup.find_all(attrs={
        "aria-label": lambda x: x and "Optional" in x,
        "title": lambda x: x and x.strip()
    })
    
    time.sleep(1)

    for item in items:
        # 移除aria-label检查，选择器已确保包含该属性
        # if 'aria-label' not in item.attrs or '可选' not in item['aria-label']:
        #     continue

        # 优先从title属性提取完整值
        if 'title' in item.attrs:
            row_data = [item['title'].strip()]
            #if len(row_data)==8: #only applies to pid
            rows.append(row_data)
        else:
            # fallback to text items if title not available
            text_items = item.find_all(attrs={"ng-repeat": "textItem in item.textItems"})
            if text_items:
                row_data = [','.join(span.get_text(strip=True) for span in text_items)]
                rows.append(row_data)
    
    # 移除可能的空行
    return [row for row in rows if any(row)]


def read_item_list(file_path):
    """读取txt清单文件，返回item列表"""
    with open(file_path, 'r', encoding='utf-8') as f:
        items = [line.strip() for line in f if line.strip()]
    return items


def create_driver():
    """创建并返回Chrome WebDriver实例"""
    from selenium.webdriver.chrome.service import Service
    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
    return driver


def process_item_list(driver, items):
    """处理项目清单，通过driver执行网页操作"""
    # 清空CSV文件（确保每次运行从头开始）
    with open('qlik_table_data_search.csv', 'w', newline='', encoding='utf-8') as f:
        pass
        
    for item in items:
        try:
            print(f"开始处理项目: {item}")

            # 1. 点击取消搜索当前项目
            cancel_elem = WebDriverWait(driver, 10).until(
                EC.element_to_be_clickable((By.CSS_SELECTOR, 'span[ng-repeat="textItem in item.textItems"].ng-binding.ng-scope'))
            )
            cancel_elem.click()
            time.sleep(0.5)

            # 2. 点击搜索按钮
            search_btn = WebDriverWait(driver, 10).until(
                EC.element_to_be_clickable((By.CSS_SELECTOR, 'a.qv-object-search.lui-icon.lui-icon--search.ng-scope'))
            )
            search_btn.click()
            time.sleep(0.5)

            # 3. 搜索框输入对应值
            search_input = WebDriverWait(driver, 10).until(
                EC.element_to_be_clickable((By.CSS_SELECTOR, 'input.lui-search__input'))
            )
            search_input.clear()
            search_input.send_keys(item)
            time.sleep(0.5)

            # 4. 点击确认这个值
            confirm_elem = WebDriverWait(driver, 10).until(
                EC.element_to_be_clickable((By.XPATH, f'//span[contains(@aria-label, "{item}")]'))
            )
            confirm_elem.click()
            time.sleep(0.5)

            # 5. 点击确认选择按钮
            confirm_btn = WebDriverWait(driver, 10).until(
                EC.element_to_be_clickable((By.CSS_SELECTOR, 'button.sel-toolbar-btn.sel-toolbar-confirm'))
            )
            confirm_btn.click()
            print(f"已确认选择项目: {item}")

            # 等待数据加载完成
            time.sleep(2)

            # 提取表格数据
            html_source = driver.page_source
            current_rows = extract_table_data(html_source)

            # 过滤空行并提取结果
            current_rows = [row for row in current_rows if row]  # 移除空行
            current_results = [row[0] for row in current_rows] if current_rows else []
            row_to_write = [item] + current_results
            
            # 追加写入CSV
            with open('qlik_table_data_search.csv', 'a', newline='', encoding='utf-8') as f:
                writer = csv.writer(f)
                writer.writerow(row_to_write)
            
            print(f"项目{item}处理完成，已保存{len(current_results)}条结果\n")

        except Exception as e:
            print(f"处理项目{item}时出错: {str(e)}")
            continue

def main():
    # 创建driver实例
    driver = create_driver()
    url = "https://sense-demo.qlik.com/sense/app/069279ac-a7e8-4405-826d-0cd3de7d48e0/sheet/CmbWz/state/analysis/overlay/selections"

    try:
        # 打开目标网页
        driver.get(url)

        # 等待页面加载完成（等待表格出现）
        WebDriverWait(driver, 20).until(
            EC.presence_of_element_located((By.CSS_SELECTOR, 'span[ng-repeat*="textItem in item.textItems"]'))
        )

        # 等待用户手动调整到待搜索状态
        input("请手动调整页面到待搜索状态，完成后按Enter键继续...")

        # 读取item清单
        item_file = 'items.txt'  # 可根据需要修改为实际文件路径
        items = read_item_list(item_file)
        print(f"成功读取{item_file}，共{len(items)}个项目需要处理")

        # 处理项目清单
        process_item_list(driver, items)

    except Exception as e:
        print(f"发生错误: {str(e)}")
    finally:
        # 关闭浏览器
        driver.quit()


if __name__ == "__main__":
    main()

```


```bash
@echo off
setlocal enabledelayedexpansion

:: ===== 用户配置区域 =====
:: 设置旧电脑IP（运行时输入）
set /p "OLD_PC_IP=请输入旧电脑的IP地址: "
:: 自动获取当前用户名
set USER_NAME=%USERNAME%
:: 设置共享文件夹名称（旧电脑上需预先共享）
set SHARE_NAME=UserData
:: 排除文件夹列表（用分号分隔）
set EXCLUDE_DIRS=Windows;Program Files;Program Files (x86);ProgramData;PerfLogs;`$Recycle.Bin;Temp;Temporary Internet Files;WinSxS;MicrosoftEdgeCache;EdgeTemp

:: ===== 路径设置 =====
set SOURCE_PATH="\\%OLD_PC_IP%\%SHARE_NAME%"
set TARGET_PATH="C:\"
set LOG_PATH="%TEMP%\Migration_Log_%date:~-4,4%%date:~-10,2%%date:~-7,2%.log"
set FAIL_LOG="%TEMP%\Failed_Files_%date:~-4,4%%date:~-10,2%%date:~-7,2%.log"
set DIFF_LOG="%TEMP%\Difference_Report_%date:~-4,4%%date:~-10,2%%date:~-7,2%.log"

:: ===== 创建排除参数 =====
set EXCLUDE_PARAM=
for %%d in (%EXCLUDE_DIRS%) do set EXCLUDE_PARAM=!EXCLUDE_PARAM! /XD "%%d"

:: ===== 主复制操作 =====
echo 正在复制数据（支持断点续传）...
robocopy %SOURCE_PATH% %TARGET_PATH% %EXCLUDE_PARAM% /E /COPY:DAT /Z /XJ /R:3 /W:5 /MT:16 /TEE /NP /TS /LOG+:%LOG_PATH% /UNILOG+

:: ===== 生成差异报告 =====
echo 生成文件差异报告...
robocopy %SOURCE_PATH% %TARGET_PATH% %EXCLUDE_PARAM% /L /NJH /NJS /NP /NS /NC /NDL /XJ /LOG:%DIFF_LOG% >nul

:: ===== 生成失败文件清单 =====
echo 生成失败文件清单...
findstr /i /c:"failed" /c:"error" /c:"extra" /c:"newer" "%LOG_PATH%" > "%FAIL_LOG%"

:: ===== 完成报告 =====
echo.
echo ============== 迁移完成 ==============
echo 完整日志: %LOG_PATH%
echo 失败文件: %FAIL_LOG%
echo 差异报告: %DIFF_LOG%
echo.
echo 提示：检查失败文件后，可重新运行本脚本继续传输
echo =====================================
pause


```