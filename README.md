# zy-upgrade
uni-app 在线升级

## 安装插件

~~直接 hbuilderx 导入

## 导入插件

```
import ZyUpdate from '../../components/zy-upgrade/zy-upgrade.vue'
```

........

```
components:{
	ZyUpdate
},
```

## 控件引入

```
<zy-update theme="green" :h5preview="true" oldversion="1.0.2" :appstoreflag="false" :updateurl="update_url"></zy-update>
```

#### 参数说明

| 参数         | 类型    | 必填 | 说明                                                                                                               |
| ------------ | ------- | ---- | ------------------------------------------------------------------------------------------------------------------ |
| theme        | String  | 是   | 主题，目前支持主题：gree、red、blue、yellow、pink。如需要其他主题可联系作者定制                                    |
| updateurl    | String  | 是   | 升级检测的 URL，一般需要用 php 或 java、c#、python、nodejs 等写一个服务端，作为升级检测的 url                      |
| h5preview    | Boolean | 否   | 开发过程中是否支持 hbuilderx 或浏览器预览，方便开发测试                                                              |
| oldversion   | Stirng  | 否   | H5 预览模式下可以传一个 oldverison，主要显示在升级界面上，不传则不显示原版本，APP 会自动检测旧版本，不需要传此参数 |
| oldcode      | Number  | 否   | H5 预览模式下传的 versioncode，有的开发者可能会用 versioncode 做升级判断，更简便一点。传或不传取决于你自己         |
| appstoreflag | Boolean | 否   | 是否支持 appstroe 升级。如果支持则服务器端返回的不是安装包地址，而是 appstore 地址，app 将会打开 appstore 进行升级 |

## 服务端版检测说明

为了方便开发，我把所有业务全部封到插件里了，入参如下，服务端可以根据需要选中：

```
{
	"method": "upgrade",
	"version": "1.0.0",
	"name": "xxxx",
	"code": "10",
	"ts": "123"
	"platform": "android"
}

```

传了一个 method，服务端可以根据该字段表明是用来请求升级的，versionname 和 code 我都传了，自己选择使用哪一个来验证版本。ts 是时间戳，没有啥用。platform 为平台，android/ios。

调用时候不好意思，只能按插件规定的返回体格式了。示例如下：

```
{
    "code": 100,
	"msg": "",
	"data": {
		"update_flag": 1,
		"update_url": "http://xxxx/Uploads/app/backorder_1.0.2.apk",
		"forceupdate": 0,
		"update_tips": "优化拆包上传后备注丢失问题\n优化其他功能",
		"version": "1.0.2",
		"size": 0
	}
}
```

- code=100 代表接口调用成功，其他 code 代表失败。<br />
- msg 为返回说明，目前客户端没有用到，可以保持为空。<br />
- data 是真正的返回体：<br />
  - update_flag: 0 不需要升级，1 需要升级<br />
  - forceupdate: 0 不强制，1 强制升级（这个字段以后可能会优化到 update_flag 里）<br />
  - update_url: 升级安装包或 appstore 地址，注意是全路径<br />
  - update_tips：升级说明，不用我提醒大家换行应该用\n 了吧？ <br />
  - version：当前最新的版本。<br />
  - size：备用吧，有些服务器启用了 gzip 或者直接返回流数据，会造成客户端获取不到安装包大小，进度条将会失效。用这个字段来计算进度。<br />

###### 服务端代码示例

一般服务端会有一个版本控制的表，升级只需要把 apk、ipa 或 appstore 地址保存到表里即可。拿到客户端入参后，与数据库最新版本进行比较，判断是否需要升级。具体判断业务逻辑由服务端开发人员自己实现，这里简要给一下返回体。

- php

代码写的比较硬

```
$version = I("version");
// 一系列数据库判断逻辑，省略
$update_flag = $result?1:0;
$data = array(
	"update_flag"=> $update_flag,
	"update_url"=>"http://xxxxx/Uploads/app/backorder_1.0.2.apk",
	"forceupdate"=>0,
	"update_tips"=>"优化拆包上传后备注丢失问题\ntest",
	"version"=>"1.0.2",
	"size"=>0
);
return json(array("code"=>100,“msg"=>"","data"=>$data));
```

- java

java 或 c#应该定义一个实体做为返回体，直接实例化实体返回即可
(CodeEnum 是一个枚举，代码略)

```
public class BaseResponse {
    private String msg;
    private int code;
    protected BaseResponse(){}
    //构造函数
    protected BaseResponse(CodeEnum codeEnum){
        this.code = codeEnum.getCode();
        this.msg = codeEnum.getMsg();
    }
	// get set 略
}
```

再构造一个 ResponseData

```
public class ResponseData<T> extends BaseResponse {
    private T data;
    protected ResponseData(){}
    protected  ResponseData(CodeEnum codeEnum,T data){
        super(codeEnum);
        this.data = data;
    }
	// 其他方法略
}
```

返回升级信息直接这样就可以了，其中 UpdateEntity 是一个实体，跟返回内容的 data 字段保持一致就可以了

```
// 一系列数据库和升级判断逻辑，得到升级的实体类updateEntity
return ResponseData.out(CodeEnum.SUCCESS, updateEntity);
```

---

# 作者

十几年服务端软件开发精验，擅长 java、c#和 php 等服务端软件，写过两年 android，研究各类跨平台开发，weex、react-native 等，发现 uni-app 实在是比其他类跨平台开发方便很多。前几年接过几个私活，那时候还在用 mui。今年疫情期间学了段时间 vue，给公司用 springcloud+vue 写了一个知识库系统。目前作者主要从事物联网相关方面的工作，对于软件开发，纯属于业余好了。非科班出身，大学学的是物理，十几年横练了一身野路子，算是信息管理系统全栈工程师了吧，水平有限，请海涵，如有错误，请反馈给作者及时优化。
<br />
有问题交流，也可以稳步：
<a href="https://zhouyu629.cnblogs.com/">我的博客</a>
