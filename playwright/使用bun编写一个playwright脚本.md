
### 1. 环境准备
1. **安装依赖**  
   • 安装Bun（若未安装）：
     ```bash
     curl -fsSL https://bun.sh/install | bash
     ```
   • 在项目中初始化Bun并安装Playwright：
     ```bash
     bun init
     bun install playwright
     ```
   • 安装浏览器驱动（确保使用本机Chrome）：
     ```bash
     bunx playwright install chrome
     ```

---

### 2. 脚本编写
```javascript
// main.ts
import { chromium } from 'playwright';

async function runPlaywrightScript() {
  // 启动本机Chrome（非Chromium）
  const browser = await chromium.launch({
    channel: 'chrome',    // 指定使用已安装的Chrome
    headless: false,      // 显示浏览器界面
    slowMo: 100           // 操作间隔（毫秒，便于观察）
  });

  const page = await browser.newPage();
  
  try {
    // 导航至目标页面
    await page.goto('https://www.baidu.com');
    
    // 模拟用户输入及点击操作
    await page.fill('#kw', 'Python自动化');  // 输入框ID为#kw
    await page.click('#su');                 // 搜索按钮ID为#su
    
    // 等待结果加载并截图
    await page.waitForSelector('.result-op');
    await page.screenshot({ path: 'search_result.png' });
    
    console.log('脚本执行完成，截图已保存');
  } catch (error) {
    console.error('执行出错:', error);
  } finally {
    await browser.close();  // 确保浏览器关闭
  }
}

runPlaywrightScript();
```

---

### 3. 关键配置解析
• **浏览器选择**：通过`channel: 'chrome'`调用系统安装的Chrome而非内置Chromium。
• **交互可见性**：设置`headless: false`让操作过程可视化，适合调试。
• **元素定位**：使用开发者工具（F12）获取元素选择器（如`#kw`为百度输入框ID）。
• **异常处理**：通过`try/catch`捕获执行错误，确保资源释放。

---

### 4. 执行脚本
```bash
bun run main.ts
```

---

### 5. 扩展功能
• **文件操作**：支持上传/下载文件（需配置`acceptDownloads: true`）。
• **移动端模拟**：通过`device`参数模拟手机浏览器：
  ```javascript
  const context = await browser.newContext({
    userAgent: 'Mozilla/5.0 (iPhone; CPU iPhone OS 14_0 like Mac OS X) AppleWebKit/605.1.15'
  });
  ```
• **API测试**：结合`page.route()`拦截网络请求，直接验证接口响应。

---

### 常见问题
• **驱动缺失**：若报错`Browser not found`，需运行`bunx playwright install`。
• **选择器失效**：使用`page.waitForSelector()`确保元素加载完成。

通过以上步骤，可快速实现基于Bun和Playwright的浏览器自动化脚本。如需更多高级功能（如多标签页管理、跨域操作），可参考[Playwright官方文档](https://playwright.dev/docs/intro)。