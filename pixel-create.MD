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

 SQL:
```SQL
ALTER PROCEDURE [tracking].[CreatePixelExist]
@RetailerId int,
@RetailerRegistryCode nvarchar(40)
AS
BEGIN

if exists(select CreatePixelId from tracking.CreatePixel where retailerId = @RetailerId and AltRetailerRegistryCode = @RetailerRegistryCode)

select 1;
else
select 0;

END
```

#### SavePixelCreate
 执行存储过程： tracking.InsertCreatePixel

 SQL:
 ```SQL
ALTER PROCEDURE [tracking].[InsertCreatePixel]
       @RetailerId int,
       @RetailerRegistryCode nvarchar(70),
       @RegistryClickId bigint,
       @UserId bigint,
       @SessionId nvarchar(40),
       @MachineId nvarchar(40),
       @PixelUrl nvarchar(2048),
       @PageUrl nvarchar(500),
       @RegistryTrackingCookie nvarchar(2000),
       @UserAgent nvarchar(200),
       @CreatedBy nvarchar(40),
	   @UUID varchar(50),
	   @CreatePixelId bigint output,
	   @OriginalMachineId nvarchar(40) output
AS
BEGIN

	if @RegistryClickId > 0
	begin
		select @OriginalMachineId = MachineId from tracking.RegistryClick WHERE RegistryClickId = @RegistryClickId;
		if @OriginalMachineId is not null and @MachineId <> @OriginalMachineId
			set @MachineId = @OriginalMachineId
	end

	insert into tracking.CreatePixel
	(
		RetailerId, RetailerRegistryCode, RegistryClickId,
		UserId, SessionId, MachineId, PixelUrl, PageUrl, RegistryTrackingCookie, UserAgent, CreatedDate, CreatedBy, AltRetailerRegistryCode, UUID
	)
	values
	(
		@RetailerId, @RetailerRegistryCode, @RegistryClickId,
		@UserId, @SessionId, @MachineId, @PixelUrl, @PageUrl, @RegistryTrackingCookie, @UserAgent, GETDATE(), @CreatedBy, @RetailerRegistryCode, @UUID
	)
	select @CreatePixelId = @@identity;

END
 ```

 OutParams:
  - CreatePixelId
  - OriginalMachineId

  存储过程执行完成后：
    给param的id赋值，然后检查MachineId和OriginalMachineId是否相同，如果不同，则将OriginalMachineId值写入Cookie，

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

