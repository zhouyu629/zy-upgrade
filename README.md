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
| theme        | String  | 是   | 主题，默认为green，目前支持主题：gree、red、blue、yellow、pink。如需要其他主题可联系作者定制                                    |
| updateurl    | String  | 是   | 升级检测的 URL，一般需要用 php 或 java、c#、python、nodejs 等写一个服务端，作为升级检测的 url                      |
| h5preview    | Boolean | 否   | 开发过程中是否支持 hbuilderx 或浏览器预览，方便开发测试                                                            |
| oldversion   | Stirng  | 否   | H5 预览模式下可以传一个 oldverison，主要显示在升级界面上，不传则不显示原版本，APP模式下会自动检测旧版本，不需要传此参数 |
| oldcode      | Number  | 否   | H5 预览模式下传的 versioncode，有的开发者可能会用 versioncode 做升级判断，更简便一点。传或不传取决于你自己         |
| appstoreflag | Boolean | 否   | 是否支持 appstroe 升级。如果支持则服务器端返回的不是安装包地址，而是 appstore 地址，app 将会打开 appstore 进行升级 |
| autocheckupdate | Boolean | 否 | 默认为false，如果为true才会页面mounted时自动检测。比如在关于页面，只有用户点击检测新版本时，才会触发升级操作，这里设置为false即可。进入APP第一个页面，设置为true，则会自动检测新版本 |
| noticeflag | Boolean | 否 | 默认为false，如果设置为true，当没有新版本时，会通过showupdateTips方法通知父组件，父组件可以提示已是最新版本  |

###### 点击按钮检测更新示例
```
<zy-update :updateurl="updateurl" :noticeflag="true" :h5preview="true" oldversion="1.0.3" 
		ref="zyupgrade" :showupdateTips="showupdatetips"></zy-update>
```
页面调用子组件方法要引用（ref），子组件调用父页面方法要发射（emit）<br />
注意：首先要给插件一个ref="zyupgrade"，这样检测版本按钮时，才能调用插件时原版本检测方法(check_update)：
```
checkupdate:function(){
	this.$refs.zyupgrade.check_update()
}
```
另：noticeflag="true",这样插件才会通知当前页面没有最新版本（有最新版本会自动弹出升级对话框）。
	需要在当前页面绑定一个事件，用于显示通知：:showupdateTips="showupdatetips"，接下来需要做的就是在当前页面定义showupdatetips方法

```
showupdatetips:function(flag){
	if(flag == 0){
		uni.showToast({
			title: '已经是最新版本了',
			icon:'none'
		});
	}
}
```
没有版本需要升级时，会传flag=0出来。页面怎么提醒，就随你开心了

## 服务端版检测说明

为了方便开发，我把所有业务全部封到插件里了，入参如下，服务端可以根据口味选择服用：

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
		"size": 0,
		"wgt_flag": 0,
		"wgt_url": "http://xxxx/Uploads/app/backorder_1.0.2.wgt"
	}
}
```

- code=100 代表接口调用成功，其他 code 代表失败。<br />
- msg 为返回说明，跟我其他项目返回体格式保持一致，目前版本升级逻辑客户端没有用到，可以保持为空。<br />
- data 是真正的返回体：<br />
  - update_flag: 0 不需要升级，1 需要升级<br />
  - forceupdate: 0 不强制，1 强制升级（这个字段以后可能会优化到 update_flag 里）<br />
  - update_url: 升级安装包或 appstore 地址，注意是全路径<br />
  - update_tips：升级说明，不用我提醒大家换行应该用\n 了吧？ <br />
  - version：当前最新的版本。<br />
  - size：备用吧，有些服务器启用了 gzip 或者直接返回流数据，会造成客户端获取不到安装包大小，进度条将会失效。用这个字段来计算进度。<br />
  - wgt_flag: 1.0.3后新增参数，1为增量升级，0为非增量升级
  - wgt_url: 1.0.3后新增参数，如果wgt_flag=1，则这个参数就是增量包的地址，注意是全路径

###### 服务端代码示例

一般服务端会有一个版本控制的表，升级只需要把 apk、ipa 或 appstore 地址保存到表里即可。拿到客户端入参后，与数据库最新版本进行比较，判断是否需要升级。具体判断业务逻辑由服务端开发人员自己实现，这里简要给一下返回体。

- php

代码写的比较硬

```
$version = input("version");
// 一系列数据库判断逻辑，省略
$update_flag = $result?1:0;
$data = array(
	"update_flag"=> $update_flag,
	"update_url"=>"http://xxxx/Uploads/app/backorder_1.0.2.apk",
	"forceupdate"=>0,
	"wgt_flag"=>1,
    "wgt_url"=>"http://192.168.10.7:88/Public/Uploads/app/backorder_1.0.4.wgt",
	"update_tips"=>"优化拆包上传后备注丢失问题\ntest",
	"version"=>"1.0.2",
	"size"=>0
);
return json(array("code"=>100,"msg"=>"","data"=>$data));
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
	public static <T> ResponseData<T> out(CodeEnum codeEnum,T data){
        //如果data类型是List,则会强制转换为LinkedHasMap，客户端不能自动转换为List对象，需要再次转
        return new ResponseData<T>(codeEnum,data);
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
