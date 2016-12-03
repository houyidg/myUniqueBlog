
# 开发和学习编程中的备忘录 #
> 我觉得这几天学习到很多东西，那是可贵的经验，但是自己文笔又不好，只能按简单的列表一条一条记录…………

1. 最近Android端有个小需求，是要通过（qq、微信、浏览器）扫描二维码，访问获取的网址，该网址放回的结果由服务端注入一段js脚本和就可以携带数据且启动Android/ios里面的app，这时第三方app里面js与原生app交互存在一个协议，这个协议配置：

	- Android项目的manifests文件里找到启动入口activity配置节点添加
	
			<intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data
                    android:host="myhost"
                    android:scheme="myscheme"/>
            </intent-filter>	

	- 访问网页配置
	
			<!DOCTYPE html>
			<html>  
			<body>
			<h1>Test Scheme</h1> 
			<!--自动加载隐藏页面跳转-->
			<iframe src="myscheme://myhost" style="display:none"></iframe>
			<!--手动点击跳转-->
			<a href="myscheme://myhost?param1=47271229870055424&param2=100386004045">Click</a>
			</body>  
			</html>
	
	 
	- 在启动activity中获取网页传递数据有点特殊：
	
		  	 Uri uri = getIntent().getData();
       		 if (uri == null) {
          		  return;
       		 }
            String param1 = uri.getQueryParameter("param1");
            String param2 = uri.getQueryParameter("param2");

	- 同时在开发过程中遇到了Android手机上浏览器不能完成那种效果的问题，最终发现是由于这个Android机的浏览器设置页面的访问标示是desktop导致服务端没有走Android机判断的那个分支，这里get到了Android机的浏览器可以通过链接电脑的chrome浏览器进行调试，同时在console控制台可以随便粘贴一段js语句回车就可以打印结果，感觉很高端，很实用。Android机浏览器网页程序连接chrome浏览器调试如下：
	
			1、连接手机adb至电脑，打开chrome输入“chrome://inspect”找到自己的Android机连接
			2、访问Android机浏览器网页程序，可以在chrome浏览console看见手机浏览器网页程序运行的日志
			
	
2. 基本完成公司Android项目之后，我就开始着手于优化项目，由于我觉得Android程序的优化是一个很较高大上的skill，一般人很少关注这方面，在之前就通过android studio monitors工具检查分析过当前app反复执行和手动回收占内存量、方法执行效率、对象引用链的分析。在这里，由于项目性能较好，我只使用Android studio的lint发现了有很多优化点，比如：多余的资源未引用、在布局中多余的嵌套以及缺少属性带来的性能消耗。最终又发现了handler内存泄露的问题，就是在activity里面定义一个非静态handler成员变量，在lint之后会提示“This Handler class should be static or leaks might occur”，handler类应该是静态的否则也许会发生内存泄露，这个问题比较繁多不赘述，找到一篇较好的资料[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/1106/1922.html](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/1106/1922.html)，我项目中的写法如下

		private static   abstract class MyHandler<T> extends android.os.Handler{
        private final WeakReference<T> objWeakReference;
        public MyHandler(T obj){
            objWeakReference = new WeakReference<T>(obj);
        }
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            if(objWeakReference==null){
                return;
            }
            T t = objWeakReference.get();
            if(t==null){
                return;
            }
            mHandleMsg(msg,t);
        }
        	public abstract void mHandleMsg(Message msg, T t);
	    }

	    private  MyHandler<CodeFragment> mHandler = new MyHandler<CodeFragment>(this) {
	        @Override
	        public void mHandleMsg(Message msg, CodeFragment fragment) {
	            int what = msg.what;
	                if (what == WHAT_MRESULT_SUCCESS) {
	                   
	                }else if (what == WHAT_MRESULT_FAIL) {
	                 
	                }
	        }
	    };
但使用之后，我一直存在一个疑问为什么静态内部类不持有外部类的this引用，于是我又查阅到class文件内部的结构，才解决我的疑问附上静态内部类和非静态内部类class结构分析[http://blog.csdn.net/zhangjg_blog/article/details/20000769](http://blog.csdn.net/zhangjg_blog/article/details/20000769)

3.公司还有一个用reactnative开发的项目，目前正在开发，由于使用到新的编程语言，并且很少有较好一点的资料，只有去官网学习，无奈自己经验不足和英文水平不足，在学习过程遇到了不少的麻烦，走了很多弯路，但是自己选择的路，即使再难也要坚持下来！以下是开发过程问题：
	
- 经验！经验！经验！ 在学习前沿新的开发技术过程中，往往都会有很多坑，比如：我在window下开发reactnative，在给项目更新reactnative版本的时候，需要开启Android studio，一老碰到AS更新卡，显示不及时，gradle配置较复杂等问题！

- 现在又不想写了，把自己开发react-native项目中遇到的BUG问题和解决方案文档上传了，可能这些解决方案不适用所有情况（自己亲身经历过以前方法改的BUG，导致后来引入新的lib又出现新的BUG，悲剧）！
