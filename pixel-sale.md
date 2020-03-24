# pixel

## Sale

### 整体代码逻辑

先对传入的Schema进行验证，然后检查要创建的SalesPixel数据是否存在于数据库中，如果存在，抛出异常 Duplicate registry，
如果不存在，则写入数据库。

Code
```csharp
if (!this.TryValidateParam(param))
{
    return;
}

// Task 192566
if (!this.DataProvider.SalesPixelRecordExist(param.RetailerId, param.OrderNumber))
{
    this.DataProvider.SavePixelSale(param);
}
else
{
    Response.AppendCookie(new HttpCookie("sstest", "test_samesite") { SameSite = SameSiteMode.Lax, Secure = true, Domain = "local.track.net" });
    this.AddHeaderError(param, "Duplicate order error");
}

this.ReturnPixel(param);
```


### TryValidateParam(param)
  验证传入的object的Schema结构，有任何错误，抛出400异常，return; 我们用JOI验证即可

#### param结构
PixelSaleParam ：
```csharp
public long Id { get; set; }
public long UserId { get; set; }
public string UUID { get; set; }
public string SessionId { get; set; }
public string MachineId { get; set; }
public string UserAgent { get; set; }
private const string CREATED_BY = "SalesPixel";
[Range(1, int.MaxValue, ErrorMessage = "Missing or invalid retailer_uid")]
public int RetailerId { get; set; }
[Required(ErrorMessage = "Missing orderNumber")]
[StringLength(40, ErrorMessage = "OrderNumber is greater than 40 characters")]
public String OrderNumber { get; set; }
[Range(typeof(DateTime), "1/1/1970", "12/31/9999", ErrorMessage = "Missing or invalid saleDate")]
public DateTime OrderDate { get; set; }
[Range(0.001, double.MaxValue, ErrorMessage = "Missing or invalid merchTotal")]
public decimal SalesTotal { get; set; }
public string RetailerRegistryCodes { get; set; }
public long RegistryClickId { get; set; }
public String RegistryTrackingCookie { get; private set; }
public string PixelUrl { get; set; }
public string PageUrl { get; set; }
public String CreatedBy
```


### SalesPixelRecordExist
验证要创建的SalesPixel数据是否存在于数据库中

#### CreatePixelRecordExist
 执行存储过程： tracking.SalesPixelExist

 SQL:
```SQL
ALTER PROCEDURE [tracking].[SalesPixelExist]
       @RetailerId int,
       @OrderNumber nvarchar(40)
AS
BEGIN

       if exists(select SalesPixelId from tracking.SalesPixel where RetailerId= @RetailerId and OrderNumber = @OrderNumber)
              select 1;
       else
              select 0;

END
```

#### SavePixelCreate
 执行存储过程： tracking.InsertSalesPixel

 SQL:
 ```SQL
ALTER PROCEDURE [tracking].[InsertSalesPixel]
       @RetailerId int,
       @OrderNumber nvarchar(40),
       @OrderDate datetime,
       @SalesTotal money,
       @RetailerRegistryCodes nvarchar(512),
       @RegistryClickId bigint,
       @UserId bigint,
       @SessionId nvarchar(40),
       @MachineId nvarchar(50),
       @PixelUrl nvarchar(2048),
       @PageUrl nvarchar(500),
       @RegistryTrackingCookie nvarchar(2000),
       @UserAgent nvarchar(200),
       @CreatedBy nvarchar(40),
       @UUID varchar(50),
       @SalesPixelId bigint output,
       @OriginalMachineId nvarchar(40) output
AS
BEGIN

    if @RegistryClickId > 0
    begin
        select @OriginalMachineId = MachineId from tracking.RegistryClick WHERE RegistryClickId = @RegistryClickId;
        if @OriginalMachineId is not null and @MachineId <> @OriginalMachineId
            set @MachineId = @OriginalMachineId
    end

    insert into tracking.SalesPixel
    (
        RetailerId, OrderNumber, OrderDate, SalesTotal, RetailerRegistryCodes,
        RegistryClickId, UserId, SessionId, MachineId,
        PixelUrl, PageUrl, RegistryTrackingCookie, UserAgent, CreatedDate, CreatedBy, UUID
    )
    values
    (
        @RetailerId, @OrderNumber, @OrderDate, @SalesTotal, @RetailerRegistryCodes,
        @RegistryClickId, @UserId, @SessionId, @MachineId,
        @PixelUrl, @PageUrl, @RegistryTrackingCookie, @UserAgent, getdate(), @CreatedBy, @UUID
    )
    select @SalesPixelId = @@identity;

END
 ```

 OutParams:
  - CreatePixelId
  - OriginalMachineId

  存储过程执行完成后：
    给param的id赋值，然后检查MachineId和OriginalMachineId是否相同，如果不同，则将OriginalMachineId值写入Cookie， 同时写log.

```csharp
    param.Id = DataUtility.ObjectToValue<long>(cmd.Parameters["@SalesPixelId"].Value, 0);

    var oldMachineId = DataUtility.ObjectToValue<string>(cmd.Parameters["@OriginalMachineId"].Value, null);
    if (!string.IsNullOrWhiteSpace(oldMachineId) && oldMachineId != param.MachineId)
    {
        LoggerProvider.Current.Error(string.Format("incorrect machineId:{0}, clickId:{1}, salesPixelId: {2}", param.MachineId, param.RegistryClickId, param.Id));
        if (HttpContext.Current != null) HttpContext.Current.Response.Cookies[Constant.MACHINE_ID_COOKIE_NAME].Value = oldMachineId;
    }

    return param.Id;
```

