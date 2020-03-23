# pixel

## Create

### TryValidateParam(param)
  验证传入的object结构，有任何错误，抛出400异常，return; 我们用JOI验证即可
  
#### param结构
PixelCreateParam ： 
```csharp
public long Id { get; set; }
public long UserId { get; set; }
public string UUID { get; set; }
public string SessionId { get; set; }
public string MachineId { get; set; }
public string UserAgent { get; set; }
private const string CREATED_BY = "CreatePixel";
[Range(1, int.MaxValue, ErrorMessage = "Missing or invalid retailer_uid")]
public int RetailerId { get; set; }
[Required(ErrorMessage = "Missing reg")]
public String RetailerRegistryCode { get; set; }
public long RegistryClickId { get; set; }
public String RegistryTrackingCookie { get; private set; }
public string PixelUrl { get; set; }
public string PageUrl { get; set; }
public String CreatedBy
```


### CreatePixelRecordExist
```csharp
if (!this.DataProvider.CreatePixelRecordExist(param.RetailerId, param.RetailerRegistryCode))
{
    this.DataProvider.SavePixelCreate(param);
}
else
{
    this.AddHeaderError(param, "Duplicate registry");
}
```

验证要创建的pixel数据是否存在于数据库中，如果存在，抛出异常 Duplicate registry，
如果不存在，则写入数据库。

#### CreatePixelRecordExist
 执行存储过程： tracking.CreatePixelExist
 参数
  - RetailerId
  - RetailerRegistryCode

#### SavePixelCreate
 执行存储过程： tracking.InsertCreatePixel
 
 AddInParams：
  - RetailerId
  - RetailerRegistryCode
  - RegistryClickId (小于等于0，则为null)
  - UserId
  - SessionId
  - MachineId
  - PixelUrl
  - PageUrl
  - RegistryTrackingCookie
  - UserAgent
  - CreatedBy
  - UUID

AddOutParams:
  - CreatePixelId
  - OriginalMachineId

  存储过程执行完成后：
    给param的id赋值，将OriginalMachineId值写入Cookie， 同时写log.
    
```csharp
    param.Id = DataUtility.ObjectToValue<long>(cmd.Parameters["@CreatePixelId"].Value, 0);

    var oldMachineId = DataUtility.ObjectToValue<string>(cmd.Parameters["@OriginalMachineId"].Value, null);
    if (!string.IsNullOrWhiteSpace(oldMachineId) && oldMachineId != param.MachineId)
    {
        LoggerProvider.Current.Error(string.Format("incorrect machineId:{0}, clickId:{1}, createPixelId: {2}", param.MachineId, param.RegistryClickId, param.Id));
        if (HttpContext.Current != null) HttpContext.Current.Response.Cookies[Constant.MACHINE_ID_COOKIE_NAME].Value = oldMachineId;
    }

    return param.Id;
```

