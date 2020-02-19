# 文件传输规范定义
## 结构定义
### ResourceInfoPullRequest(资源总体信息请求)
    //文件资源唯一标识符(用于区别不同的资源之间)
    string id= 1 ;
### ResourceInfoPullResponse(资源总体信息响应)
    //响应编码(1:成功;2:资源不存在;3:其他失败原因)
    string code=1;
    //资源etag(代表资源有无变动的唯一码)
    string etag =2 ;
    //资源类型(1:完整资源;2:不完整资源)
    int32 type =3;
    //资源碎片信息(包头不包尾)
    //完整的资源只包含一个碎片(例如大小为1kb资源表示为0-1024)
    //不完整的资源包含多个碎片(例如大小为1kb资源可能表示为0-10,10-100,120-200,500-1000,1000-1024)
    string fragment=4;
### ResourceDataPullRequest(资源内容请求)
    //资源唯一标识符(用于区别不同的资源之间)
    string id= 1 ;
    //资源etag(用于表示文件资源是否产生变动,每次文件变动生成新的etag;第一次请求为空,后续的断点续传必须传入第一次请求的etag)
    string etag=2;
    //资源开始位置(0代表从头,包括)
    int32 start=3;
    //资源开始位置(-1代表到结束,不包括)
    int32 end=4;
### ResourceDataPullResponse(资源内容响应)
    //响应编码(1:成功;2:资源不存在;3:资源已经变动(etag不一致);3:其他失败原因)
    int32 code = 1;
    //数据内容
    bytes data = 2;
### ResourceInfoPushRequest(资源总体信息请求)
    空

### ResourceInfoPushResponse(资源总体信息响应)
    //响应编码(1:生成资源成功;2:其他失败原因)
    int32 code = 1;
    //资源唯一标识符
    string id =2 ;
### ResourceDataPushRequest(资源内容请求)
    //资源唯一标识符
    string id =1 ;
    //推送此段资源数据开始位置
    int32 start=2;
    //此段资源数据
    bytes data=3;
    //是否是最后一段资源
    bool finish=4;
### ResourceDataPushResponse(资源内容响应)
    //响应编码(1:资源推送成功;2:资源不存在;3:推送的资源数据位置重复;3:其他失败原因)
    int32 code = 1;

## 接口定义
### pullInfo(拉取资源信息)
    //拉取资源信息
    rpc pullInfo(ResourceInfoPullRequest) returns (ResourceInfoPullResponse);
### pullData(拉取资源内容)
    //拉取资源内容
    rpc pullData(ResourceDataPullRequest) returns (ResourceDataPullResponse);
### pushInfo(推送资源信息)
    //推送资源信息
    rpc pushInfo(ResourceInfoPushRequest) returns (ResourceInfoPushResponse);
### pushData(推送资源内容)
    //推送资源内容
    rpc pushData(ResourceDataPushRequest) returns (ResourceDataPushResponse);



## 拉取步骤说明
1. 调用**pullInfo(拉取资源信息)** 接口,获取资源信息,分为完整资源和不完整资源
2. 客户端根据**ResourceInfoPullResponse.fragment** 属性来获取已经存在的资源片段进行拉取,可自行分割成多个资源内容请求**pullData(拉取资源内容)** ,传输进度根据多个请求完成情况来判断,如果某个请求失败,重试即可
3. 调用**pullData(拉取资源内容)**,将请求的结果按照顺序组装起来,形成完整的资源内容
## 拉取流程图


## 推送步骤说明
1. 调用**pushInfo(推送资源信息)** 接口,生成一个资源,得到**ResourceInfoPushResponse.id** (若推送之前已经生成的资源,忽略此步骤)
2. 根据得到的**ResourceInfoPushResponse.id** 构造**ResourceDataPushRequest** 
3. 调用**pushData(推送资源内容)** 接口,推送资源数据 (推送最后一包资源内容请求中,设置**ResourceDataPushRequest.finish=true** )
## 推送流程图


## 特别说明
1. 如果服务器的资源发生变化,需要重新修改资源etag属性,此时客户端需要重新调用**pullInfo(拉取资源信息)** 来获取新的资源总体信息;之前的已经失效
2. 设计参考LSH,DASH设计理念
