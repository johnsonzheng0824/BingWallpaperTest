## 用 C# 语言实现自动设置必应每日图片为桌面壁纸


**本程序实现平台：** Windows 10 / Visual Studio 2017

**这个小程序的具体实现流程如下：**

1. 利用必应的接口解析出壁纸的 Url；

2. 根据 URL 保存图片到本地；

3. 将保存好的图片设置为桌面壁纸；

4. 将程序设置为开机自启。


下面我们开始。

**一、利用必应的接口解析出壁纸的 URL**

必应每日壁纸的接口为：

> http://cn.bing.com/HPImageArchive.aspx?idx=0&n=1

我们只需解析出 URL 然后在前面加上：

> http://www.bing.com

默认地址中分辨率为 1366x768，如果需要 1080p 的，直接将 “1366x768” 替换为 “1920x1080” 即可。代码如下：

```c#
/**
 * 获取壁纸网络地址
 */
public static string getURL()
{
    string InfoUrl = "http://cn.bing.com/HPImageArchive.aspx?idx=0&n=1";
    HttpWebRequest request = (HttpWebRequest)WebRequest.Create(InfoUrl);
    request.Method = "GET"; request.ContentType = "text/html;charset=UTF-8";
    string xmlDoc;
    //使用using自动注销HttpWebResponse
    using (HttpWebResponse webResponse = (HttpWebResponse)request.GetResponse())
    {
        Stream stream = webResponse.GetResponseStream();
        using (StreamReader reader = new StreamReader(stream, Encoding.UTF8))
        {
            xmlDoc = reader.ReadToEnd();
        }
    }
    // 使用正则表达式解析标签（字符串），当然你也可以使用XmlDocument类或XDocument类
    Regex regex = new Regex("<Url>(?<MyUrl>.*?)</Url>", RegexOptions.IgnoreCase);
    MatchCollection collection = regex.Matches(xmlDoc);
    // 取得匹配项列表
    string ImageUrl = "http://www.bing.com" + collection[0].Groups["MyUrl"].Value;
    if (true)
    {
        ImageUrl = ImageUrl.Replace("1366x768", "1920x1080");
    }
    return ImageUrl;
}
```

**二、根据 Url 保存图片到本地**

```c#
public static void setWallpaper()
{
    string ImageSavePath = @"D:\BingWallpaper";
    //设置墙纸
    Bitmap bmpWallpaper;
    WebRequest webreq = WebRequest.Create(getURL());
    //Console.WriteLine(getURL());
    //Console.ReadLine();
    WebResponse webres = webreq.GetResponse();
    using (Stream stream = webres.GetResponseStream())
    {
        bmpWallpaper = (Bitmap)Image.FromStream(stream);
        //stream.Close();
        if (!Directory.Exists(ImageSavePath))
        {
            Directory.CreateDirectory(ImageSavePath);
        }
        //设置文件名为例：bing2017816.jpg
        bmpWallpaper.Save(ImageSavePath + "\\bing" + DateTime.Now.Year.ToString() + DateTime.Now.Month.ToString() + DateTime.Now.Day.ToString() + ".jpg", ImageFormat.Jpeg); //图片保存路径为相对路径，保存在程序的目录下
    }
    //保存图片
    string strSavePath = ImageSavePath + "\\bing" + DateTime.Now.Year.ToString() + DateTime.Now.Month.ToString() + DateTime.Now.Day.ToString() + ".jpg";
    setWallpaperApi(strSavePath);
}
```

**小提示：**此处 Bitmap 可能无法引入类，只需右击项目 - 添加 - 引用，选择左上角程序集 - 框架，然后从中间列表选择 System.Drawing（不好找就直接右上角搜索 Drawing），打勾、确定。

**三、将保存好的图片设置为桌面壁纸**

```
//利用系统的用户接口设置壁纸
[DllImport("user32.dll", EntryPoint = "SystemParametersInfo")]
public static extern int SystemParametersInfo(
        int uAction,
        int uParam,
        string lpvParam,
        int fuWinIni
        );
public static void setWallpaperApi(string strSavePath)
{
    SystemParametersInfo(20, 1, strSavePath, 1);
}
```

最后在 main 方法中调用 setWallpaper() 方法就可以了。

```c#
static void Main(string[] args)
{
    setWallpaper();
}
```

**四、将程序设置为开机自启并隐藏窗体**

生成程序后这时候已经可以双击运行设置壁纸了，将其设置为开机自启就行。

**方法一**

将可执行文件放到 `C:\Users\用户名\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup` 文件夹中，开机就会自启。

**方法二**

使用注册表，定位到：

> \HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\

新建字符串，名字随便起，内容为程序路径，例如：

> C:\Users\Dustray\Desktop\SetWallpaper.exe

这种方法在任务管理器的 “启动” 中启动影响显示“低”。

**五、设置启动不弹黑窗口**

其实是做了个小弊，让系统把控制台应用识别为窗体应用，感觉虽不正经但效果不错：

双击 Visual Studio 中项目列表中的 Properties，选择第一项 “应用程序”- 输出类型为 Windows 应用程序，保存，生成，大功告成！