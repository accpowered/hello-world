# hello-world
Training on usage of repository

A new line is added to test commit

以下是实现所需功能的HTML和JavaScript代码：

```html
<!DOCTYPE html>
<html>
<head>
    <title>数字跳转工具</title>
</head>
<body>
    <input type="text" id="numberInput" placeholder="请输入6-8位数字">
    <button onclick="handleRedirect()">立即跳转</button>

    <script>
        function handleRedirect() {
            // 获取用户输入内容
            const input = document.getElementById('numberInput').value.trim();
            
            // 验证输入格式
            if (!/^\d{6,8}$/.test(input)) {
                alert("请输入6-8位的纯数字");
                return;
            }

            // 转换为数字并验证数值范围
            const number = parseInt(input, 10);
            if (number < 100000 || number > 99999999) {
                alert("数字范围应为100000到99999999");
                return;
            }

            // 计算ID值
            const calculatedID = number - 50;
            
            // 验证计算结果
            if (calculatedID < 0) {
                alert("计算结果不能为负数");
                return;
            }

            // 构建并打开URL
            window.open(`https://abc.xyz/def.aspx?ID=${calculatedID}`, '_blank');
        }
    </script>

    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        input {
            padding: 8px;
            margin-right: 10px;
            width: 200px;
        }
        button {
            padding: 8px 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover {
            background-color: #45a049;
        }
    </style>
</body>
</html>
```

代码特点说明：
1. 输入验证：
   - 使用正则表达式 `/^\d{6,8}$/` 确保输入为6-8位纯数字
   - 数值范围验证（100000-99999999）
   - 计算结果非负验证

2. 用户体验：
   - 清晰的错误提示
   - 输入框和按钮的基础样式
   - 按钮悬停效果

3. 安全性：
   - 自动去除输入内容前后空格
   - 显式指定数字解析的十进制格式
   - 使用模板字符串构建URL防止注入

使用方法：
1. 在输入框输入6-8位数字
2. 点击按钮后：
   - 如果输入合法 → 新标签页打开目标URL
   - 如果输入不合法 → 显示对应错误提示

注意：实际使用时请将URL中的`https://abc.xyz/def.aspx`替换为真实的目标地址。