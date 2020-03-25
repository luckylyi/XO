# Track

## Manage

#### 代码逻辑
先对传入的Shema进行验证，再看Request.Items中有没有包含 _coupleRegistry_Entity_ 的值，如果没有，从数据库中找出RetaileId=参数中的R值的Retailer数据，写入Request.Items中。
用相关的数据填充对象LinkPatternParameters，并返回。
给params赋值
  param.RegistryClickId
  param.Lu (LandingUrl)
调用PixelClick API将Click数据写入数据库。
判断参数中的rebuildUrl，如果为1则调用TryUpdatePixelClick API.
跳到对应的LandingUrl

https://github.com/tkww/Replatform/blob/dev/Registry.Cloud/Registry.Web.UI/Controllers/TrackController.cs#L76-L81

#### TrackingParam Schema:
https://github.com/tkww/Replatform/blob/dev/Registry.Libraries/XO.Registry.CoreService/Tracking/TrackingParam.cs#L10-L100

#### ConvertFromTrackingParam
https://github.com/tkww/Replatform/blob/dev/Registry.Cloud/Registry.Web.UI/Controllers/TrackController.cs#L277-L334

#### GetLandingUrl
https://github.com/tkww/Replatform/blob/dev/Registry.Libraries/XO.Registry.CoreService/RetailerPartnerService.cs#L82-L133

#### InvokePixelClickAndRedirect
https://github.com/tkww/Replatform/blob/dev/Registry.Cloud/Registry.Web.UI/Controllers/TrackController.cs#L253-L275


### 相似APIs:
```
track/rqs               RQS
track/create            Create
track/other             AdditionalInfoUrl
track/sip               StoreInfoPage
track/affiliateCreate   AffiliateCreate
track/appointment       Appointment
track/event             Event
track/mdp               MDP
```
