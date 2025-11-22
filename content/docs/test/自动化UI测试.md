`Selenium`可以用来做`自动化UI测试`，支持多语言，这里使用`C#`编写测试脚本。
<!-- more -->

# .NetFramework

`Selenium.Support`，`Selenium.WebDriver`,`Selenium.RC`

# Dotnet core

核心库：`Selenium.Support`，`Selenium.WebDriver`,`Selenium.Chrome.WebDriver`

注意：Selenium.RC包在dotnetcore会存在不兼容的问题，无法正常找到`chromedriver.exe` 文件。

**Exception**

> The chromedriver.exe file does not exist in the current directory or in a directory on the PATH environment variable. The driver can be downloaded at http://chromedriver.storage.googleapis.com/index.html.”


**解决方法**

> nuget Selenium.Chrome.WebDriver。将ChromeDriver.exe放入构建目录中


**注意**：.netcore2.1中引入了`Selenium.Chrome.WebDriver`包依旧会出现上面的异常

# 浏览器驱动下载

需下载本机对应版本，不区分32/64

[https://npm.taobao.org/mirrors/chromedriver/](https://npm.taobao.org/mirrors/chromedriver/)

# PageObject

解决的问题：版本迭代太快，UI层元素的属性经常变换，导致维护人员需要花大把的时间去维护代码，为了节省维护成本及时间，就可以利用PageObject 这种设计模式，它就大大的减少了维护时间。

创建对应浏览器配置，创建WebDriver对象：


```csharp
public abstract class TestBase
{
    protected IWebDriver driver;

    public void WebDriverInit(string url)
    {
        ChromeOptions options = new ChromeOptions();
        // 窗口最大化
        options.AddArgument("start-maximized");
        driver = new ChromeDriver(options);
        driver.Navigate().GoToUrl(url);
        Thread.Sleep(1000);
    }
}
```

ClockInPage，页面模型，封装页面标签属性以及对应操作：


```csharp
public class ClockInPage
{
    public static class PagePath
    {
        public static readonly string input_loginName_id = "tb_LoginID_Account_589f1fc067ff4031";
        public static readonly string input_passWord_id = "tb_Password";
        public static readonly string submit_login_Name = "ctl10";
    }

    public void LoginClick(IWebDriver driver, string logiName, string passWord)
    {
        driver.FindElement(By
            .Id(PagePath.input_loginName_id)).SendKeys(logiName);
        driver.FindElement(By
            .Id(PagePath.input_passWord_id)).SendKeys(passWord);
        driver.FindElement(By
            .Name(PagePath.submit_login_Name)).Click();
        Thread.Sleep(1000);
    }
}
```

ClockInTests，测试方法,采用DDT数据驱动方式提供测试数据：


```csharp
[TestCategory("ui\\CodedUITest")]
[TestClass()]
public class ClockInTests : TestBase
{
    [TestCleanup]
    public void Clean()
    {
        driver.Dispose();
    }

    [DataTestMethod]
    [DataRow("http://11.11.141.10/OMSP//Login/JSS.aspx", "wangpengliang", "Wpl-19950815", true)]
    [DataRow("http://11.11.141.10/OMSP//Login/JSS.aspx", "111111", "222222", false)]
    public void ClockIn_Login_01_Normal(string url, string loginName, string passWord, bool expect)
    {
        WebDriverInit(url);
        ClockInPage page = new ClockInPage();
        page.LoginClick(driver, loginName, passWord);
        Assert.AreEqual(expect, driver.Url != url);
    }
}
```
