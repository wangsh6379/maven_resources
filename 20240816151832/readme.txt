唯品会开放平台（vop.vip.com）SDK使用说明书

1）内容说明

┬ lib/osp-sdk.jar 			底层通讯协议jar包
├ lib/vop-sdk.jar 			服务帮助类jar包
└ lib/vop-sdk-sources.jar	服务帮助类sources包

2）使用说明

开放平台SDK由两部分组成，分别是“底层通讯协议(osp-sdk.jar)”和“服务帮助类(vop-sdk.jar)”。

3）关键字段说明

appkey:创建应用时生成，为接口调用凭证之一
appsecret：创建应用时生成，为接口调用凭证之一
accessToken：通过Oauth认证授权时生成，参考：http://vop.vip.com/doccenter/viewdoc/33
sign:调用签名，建议在异常捕获中记录该值，可以提高开放平台定位异常的效率，具体生成规则参考：http://vop.vip.com/doccenter/viewdoc/8#A4

4）调用示例

将sdk引入到项目中后，可根据以下demo测试应用访问权限和连通性，sdk提供两种系统级参数的设置方式：
 - 方式一为每个client设置不同的InvocationContext，即不同的client之间可以配置不同的appkey，appsecret，超时时间等，每个client之间的ClientInvocationContext独立存在，不会互相影响；
 - 方式二为线程内所有client设置一个通过ThreadLocal共享的InvocationContext，即线程内的所有client都是用同一个appkey，appsecret，超时时间等，当InvocationContext的值更改时，所有client都会受影响；

方式一，使用ClientInvocationContext设置客户端系统级参数（推荐）：

public static void main(String[] args) {
	//1、获取服务客户端
	AddressServiceClient client = new AddressServiceClient();
	
	try {
		//2、设置系统级调用参数
		ClientInvocationContext instance = new ClientInvocationContext();
		instance.setAppKey("appKey");//替换为你的appKey
		instance.setAppSecret("appSecret");//替换为你的appSecret
		//instance.setAccessToken("accessToken");//替换为你的accessToken，通过Oauth认证时必填
		instance.setAppURL("http://sandbox.vipapis.com/");//沙箱环境
		//instance.setAppURL("https://vop.vipapis.com/");//正式环境
		//instance.setReadTimeOut(30000);//读写超时时间，可选，默认30秒
		//instance.setConnectTimeOut(5000);//连接超时时间，可选，默认5秒
		
		//3、设置client的调用上下文
		client.setClientInvocationContext(instance);
		
		//4、调用API及返回
		List<ProvinceWarehouse> provinceWarehouse = client.getProvinceWarehouse(vipapis.address.Is_Show_GAT.SHOW_ALL);
	} catch (OspException e) {
		//4、捕获异常
		System.out.println(client.getClientInvocationContext().getLastInvocation().getSign());//获取最近一次调用的sign值
	}
}

方式二，使用ThreadLocal中的InvocationContext设置系统级参数：

public static void main(String[] args) {
	InvocationContext instance = null;
	try {
		//1、获取服务客户端
		AddressServiceClient client = new AddressServiceClient();
		
		//2、设置系统级调用参数，只需在程序开始调用前设置一次即可
		instance = InvocationContext.Factory.getInstance();
		instance.setAppKey("appKey");//替换为你的appKey
		instance.setAppSecret("appSecret");//替换为你的appSecret
		//instance.setAccessToken("accessToken");//替换为你的accessToken，通过Oauth认证时必填
		instance.setAppURL("http://sandbox.vipapis.com/");//沙箱环境
		//instance.setAppURL("https://vop.vipapis.com/");//正式环境
		//instance.setReadTimeOut(30000);//读写超时时间，可选，默认30秒
		//instance.setConnectTimeOut(5000);//连接超时时间，可选，默认5秒
		
		//3、调用API及返回
		List<ProvinceWarehouse> provinceWarehouse = client.getProvinceWarehouse(vipapis.address.Is_Show_GAT.SHOW_ALL);
	} catch (OspException e) {
		//4、捕获异常
		System.out.println(instance.getLastInvocation().getSign());//获取最近一次调用的sign值
	}
}

注：当客户端设置了ClientInvocationContext时，将不会从ThreadLocal中获取InvocationContext。仅当ClientInvocationContext未设置或设为null时，客户端会调用InvocationContext.Factory.getInstance()获取ThreadLocal中的InvocationContext。

5)FAQ

a.调用失败？

i.对于有明确返回错误信息的失败调用，请根据错误信息提示操作；
ii.对于原因不明的失败，请通过开放平台的支持中心搜索解决方案或提交问题进行报障：https://vop.vip.com/support/index

b.网络环境不稳定或超时？

i.如果是正式环境，请确认调用参数AppUrl是否为https://vop.vipapis.com
ii.若业务参数包含分页数据，请按照API在线文档（https://vop.vip.com/apicenter/index）的建议大小设置每页数据大小，一般建议每页数据在100以内
iii.其他疑难问题，请通过开放平台的支持中心提交问题进行报障：https://vop.vip.com/support/index

c.sdk如何升级？

可到开放平台下载sdk最新版本，：https://vop.vip.com/doccenter/viewdoc/20
建议下载新的sdk以后，替换所有内容（osp-sdk.jar和vop-sdk.jar），以保证sdk正常运行。
