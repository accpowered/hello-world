
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
