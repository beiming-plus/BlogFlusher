# BlogFlusher
博客刷访问量工具，其他流量工具也可以使用，稍微改造一下就可以，这是针对博客阅读访问量的工具


----------


**觉得博客一直也是无人问津，感觉很沮丧，于是想到前人说的一句话，既然没有轮子，那就自己造一个轮子吧**

 1.这样的刷流量，从技术上说，是不可能被后台检测的，根本就不可能，因为他就是一个真实的代理访问，再加上模拟文章访问和点击时间，基本就是无解。
 2. 发出来，是感觉挺好玩的，之前看到同学写游戏的挂机外挂，感觉好腻害，自己闲着那就写一个博客流量的外挂吧，哈哈哈。
 3. 觉得大家还是友好使用，不要贪心，适可而止，什么事情都没有一步登天的，都需要脚踏实地，一步一步的走下去。
 4. 有不明白的同学可以私聊我，我还是很乐意解答的，有喜欢Java的同学可以私聊我，然后我们加一下QQ好友，一起学习，最近学完了Java基础和Oracle数据库，下一步学一点点前端知识，现在只会写一些Android的前端知识，感觉前端都需要了解一下，不然怎么成为真正的后台工程师，啊哈哈哈。

首先，我问了一下同学以及百度了一下，看到有人说爬虫，有人说改装一下压力测试，我感觉都不靠谱，这些怎么说都是基于自己的IP进行访问，如果我是后台人员的话，这样的测试我是绝对不允许的。

于是乎，想了想，决定再搜一下，哈哈哈，自己还是能力有限，毕竟最近刚刚学习完Java基础，很多还不会，请原谅是小黑框，不是界面，我不打算学Java的界面，也就没有看了，小黑框运行，请谅解。

查的的结果是利用代理，正好网上有一个大象代理网站，感觉还是可以，购买一万个代理才5块钱，感觉很合算。（此处可不是一个代理地址只能访问一次哦，我们可以用很多次的，下面我会详细进行介绍）

废话说的太多了，接下来开始正文。


----------

**1.引用jar包**

jsoup-1.8.1.jar
log4j-1.2.17.jar

jar包，我会同整个工程文件一起上传，大家自己下载。

**2.MyIp.java类**

也就是代理地址类，我们得到的代理地址是[地址：端口]形式的，很好理解，代码一看就知道了。

```
package com.xiaheshun.domain;

public class MyIp {

	private String address;
	
	private String port;

	public String getAddress() {
		return address;
	}

	public void setAddress(String address) {
		this.address = address;
	}

	public String getPort() {
		return port;
	}

	public void setPort(String port) {
		this.port = port;
	}

	@Override
	public String toString() {
		return address + ":" + port;
	}
}
```

**3.Main.java**

主方法类，运行类，大概的功能步骤就是：

 1. 获取的大象代理购买的地址-->装到list集合中，便于重复使用
 2. 随机函数模拟真实文章阅读的随机性，避免所有的文章访问量是一样的
 3. 随机函数模拟真实文章阅读的时间间隔，避免所有的访问量快速且过大，不符合真实的人操作状态 
 4. 死循环执行，轮训访问本地文本中需要访问的文章地址

代码很简单，而且注释的很简单，大家一看就明白了。
```
package com.xiaheshun.main;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Random;
import org.apache.log4j.Logger;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import com.xiaheshun.domain.MyIp;

public class Main {
	private static Logger logger = Logger.getLogger(Main.class);
	//本地博客地址文本中的文章数量
	private static int blogUrlSize = 0;
	//本地博客地址文本装入HashMap中
	private static HashMap<Integer, String> LocalBlogUrl = null;
	//本地博客地址访问统计
	private static HashMap<Integer, Integer> LocalBlogUrlCount = null;
	//访问总量统计
	private static int count = 0;

	public static void main(String[] args) throws InterruptedException, IOException {
		
		//比如你的订单号是123456789，每次你想提取200个代理进行使用，就应该是
		//http://tvp.daxiangdaili.com/ip/?tid=123456789&num=200&delay=5
		String url = "http://tvp.daxiangdaili.com/ip/?tid=订单号&num=每次所取的网址数量&delay=5";
		List<MyIp> ipList = getIp(url);
		while(true){
			LocalBlogUrl = getLocalBlogUrl();
			blogUrlSize = LocalBlogUrl.size();
			LocalBlogUrlCount = initLocalBlogUrlCount();
			for(final MyIp myIp : ipList) {
		        System.setProperty("http.maxRedirects", "50");
		        System.getProperties().setProperty("proxySet", "true");
		        System.getProperties().setProperty("http.proxyHost", myIp.getAddress());
		        System.getProperties().setProperty("http.proxyPort", myIp.getPort());
		        while(true){
		        	try {
			        	int id = randomBlogUrl();
						Document doc = Jsoup.connect(LocalBlogUrl.get(id))
						  				.userAgent("Mozilla")
						  				.cookie("auth", "token")
						  				.timeout(3000)
						  				.get();
						if(doc != null) {
							count++;
							LocalBlogUrlCount.put(id, LocalBlogUrlCount.get(id)+1);
							System.out.print("ID： "+id+"\tAddress： " + (LocalBlogUrl.get(id) + "\t成功刷新次数: " + count + "\t") + "Proxy： " + myIp.toString() + "\t");
						}
					} catch (IOException e) {
						logger.error(myIp.getAddress() + ":" + myIp.getPort() + "报错");
					}
			        
			        sleepThread(randomClick());
			        show();
		        }
			}
		}
		
	}
	
	//访问文章的随机函数，用来模拟真实的访问量操作，以免所有的文章访问量都是一样的，很明显是刷的，此操作随机访问文章，制造访问假象
	public static int randomBlogUrl(){
		int id = new Random().nextInt(blogUrlSize);
		return id;
	}
	
	//时间的随机函数，用来模拟真实的访问量操作，以防被博客后台识别，模拟操作60-200秒内的随机秒数,
	public static int randomClick(){
		int time = (new Random().nextInt(200)) +60;
		return time;
	}
	
	//获取在【大象代理】中购买的IP，装入ArrayList<MyIp>中
	public static List<MyIp> getIp(String url) {
		List<MyIp> ipList = null;
		try {
			//1.向ip代理地址发起get请求，拿到代理的ip
			Document doc = Jsoup.connect(url)
			  .userAgent("Mozilla")
			  .cookie("auth", "token")
			  .timeout(3000)
			  .get();
			
			//2,将得到的ip地址解析除字符串
			String ipStr = doc.body().text().trim().toString();
			
			//3.用正则表达式去切割所有的ip
			String[] ips = ipStr.split("\\s+");
			
			//4.循环遍历得到的ip字符串，封装成MyIp的bean
			ipList = new ArrayList<MyIp>();
			for(final String ip : ips) {
				MyIp myIp = new MyIp();
				String[] temp = ip.split(":");
				myIp.setAddress(temp[0].trim());
				myIp.setPort(temp[1].trim());
				ipList.add(myIp);
			}
		} catch (IOException e) {
			System.out.println("加载文档出错");
		}		
		return ipList;
	}
	
	//获取本地BlogUrl.txt文本中的博客地址，并装入hashMap中，key=Integer,value=博客地址
	public static HashMap<Integer,String> getLocalBlogUrl() throws IOException{
		
		File filePath = new File(System.getProperty("user.dir"),"BlogUrl.txt");
		FileReader in = new FileReader(filePath);
		BufferedReader br = new BufferedReader(in);
		HashMap<Integer, String> hashMap = new HashMap<Integer, String>();
		String line = null;
		int id = 0;
		while((line = br.readLine())!=null){
			hashMap.put(id++, line);
		}
		br.close();
		return hashMap;
	} 
	
	//休眠进程，单位是分钟,CSDN的规则好像是：每个IP访问一个博客地址的时间间隔是5-15分钟，计数一次
	public static void sleepThread(int s) throws InterruptedException{
		long ms = s * 1000;
        Thread.sleep(ms);
        System.out.println("睡眠： " + s + "s");
	}
	
	//访问统计
	public static HashMap<Integer, Integer> initLocalBlogUrlCount(){
		HashMap<Integer, Integer> temp = new HashMap<Integer, Integer>();
		for(int i  = 0; i<blogUrlSize; i++){
			temp.put(i, 0);
		}
		return temp;
	}
	
	//展示访问统计总量
	public static void show(){
		System.out.println("访问量统计：");
		for(int i = 0; i<LocalBlogUrlCount.size(); i++){
			System.out.print("博客【" + i + "】:" + LocalBlogUrlCount.get(i) + "次\t");
		}
		System.out.println();
		System.out.println("总计："+count+"次");
		System.out.println();
	}
	
}
```

**4.解释一下代码中重要的部分，顺带跟大家说一下怎么购买代理**


大象代理官方网址：http://www.daxiangdaili.com/

进去之后注册一下，登录一下，然后购买第一个套餐，5块钱20000个代理的套餐。

![这里写图片描述](https://img-blog.csdn.net/20180425111450703?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpYUhlU2h1bg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

然后点击我的订单，找到你的订单，也就是这个网址：http://www.daxiangdaili.com/orders

![这里写图片描述](https://img-blog.csdn.net/2018042511174122?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpYUhlU2h1bg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```
        //比如你的订单号是123456789，每次你想提取200个代理进行使用，就应该是
		//http://tvp.daxiangdaili.com/ip/?tid=123456789&num=200&delay=5
		String url = "http://tvp.daxiangdaili.com/ip/?tid=订单号&num=每次所取的网址数量&delay=5";
```

再给地址进行替换就行了，自己也可以用它们网站跟你说的提取器进行提取，你提取之后还是这个样子，网址真的是很好懂，一看就知道了，订单号替换一下，一次提取数量替换一下。

tid 订单号
num 网址提取数量

说成这样了，应该是很好理解了。

在这里，我还要说明一下，我在这里写的程序是每次都是一次性的，也就是如果我每次提取num=2000个，那么我5块钱的套餐只能使用10次，也就是说你的程序只能运行10次，其实如果大家觉得这样太贵的话，可以对每次提取的代理地址写到文本中，代码中是写到了list集合中，大家也可以将对象序列化存储一下，下次也可以使用，也很好时间，很简单的。


**5.还要说的一些重要的事情**

希望大家不要贪心，觉得程序中的随机时间调动一下就会快一点，其实吧，我觉得大家还是本分的写好自己的博客比较好，有的时候也不一定非要别人看到，自己留个笔记也好，免得以后忘记了。

还要呀，有的同学或者就是想到让他跑快一点，我在这里温馨提醒一下下哦，太快的话是不计入访问的，我感觉是同一IP在几分钟内访问都是记做一次访问。

其实就拿我写的这个，算一下时间，基本就是一天四百多的访问量，还可以了啊，这样可以了，贪心不好。

也许还有的同学想到的是，用多线程该造一下，我在这里要提醒一下，如果只是给我的Main.java继承了一下Thread的话，那可就是等于提取多次代理地址哦，你的小钱包可是受不了的，如果非要改的话，记得要给获取代理地址专门写成一个类，然后每个线程共享一下哦， 不然太贵。


----------

**6.如何使用，总不能用eclipse运行吧**

1.工程文件目录（工程文件里面的这个，我是为了在eclipse中测试用的，实际中的整个文件夹内应包含的内容请继续向下看，到第5条说明）

![这里写图片描述](https://img-blog.csdn.net/20180425114142807?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpYUhlU2h1bg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

要注意我在这里面放了一个BlogUrl.txt文件，里面存放的是文章网址，就像下面这样

```
https://blog.csdn.net/xiaheshun/article/details/78726468
https://blog.csdn.net/xiaheshun/article/details/78776358
https://blog.csdn.net/xiaheshun/article/details/78906093
https://blog.csdn.net/xiaheshun/article/details/79571516
https://blog.csdn.net/xiaheshun/article/details/79706744
https://blog.csdn.net/xiaheshun/article/details/79843070
```

2.对程序进行打包成jar文件（也可以自己百度，很简单，我不介绍了，简单放一下截图）

![这里写图片描述](https://img-blog.csdn.net/20180425114510247?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpYUhlU2h1bg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![这里写图片描述](https://img-blog.csdn.net/20180425114623447?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpYUhlU2h1bg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

然后就OK了，导出成了jar包

3.写bat程序

Run.bat
```
c:
echo %cd%
java -jar BlogFlusher.jar
```

4.完整的应包含文件和运行的结果（BlogFlusher.jar 和 Run.bat 和 BlogUrl.txt）

![这里写图片描述](https://img-blog.csdn.net/2018042511564723?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpYUhlU2h1bg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

5.双击Run.bat即可开始运行
建议放在云主机上挂着，阿里和腾讯的云主机还是很便宜的，记得要在云主机上装上java环境哦。