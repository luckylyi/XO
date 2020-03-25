# pixel

## Click

### 整体代码逻辑

先对传入的Schema进行验证，通过后将Click写入数据库，更新OriginalSourceClickId，写入Cookie，返回Pixel数据(一个pixel的图片)。

Code
```csharp
if (!this.TryValidateParam(param))
{
    return;
}

var currentClickId = this.DataProvider.SavePixelClick(param);

RegistryTrackingCookie trackingCookie = new RegistryTrackingCookie { Os = param.Cosc };
trackingCookie.UpdateLastRegistryClickId(currentClickId);
RegistryTrackingCookieService.SetResponseCookies(trackingCookie, HttpContext);

this.ReturnPixel(param);
```


### TryValidateParam(param)
  验证传入的object的Schema结构，有任何错误，抛出400异常，return; 我们用JOI验证即可

#### param结构
PixelClickParam ：
```csharp
public long Id { get; set; }
public long UserId { get; set; }
public string UUID { get; set; }
public string SessionId { get; set; }
public string MachineId { get; set; }
public string UserAgent { get; set; }
private const string CREATED_BY = "PageView";
/// SourceType
public String St { get; set; }

/// SourceSection
public String Ss { get; set; }

/// SourcePlacement
public String Sp
{
    get { return sp; }
    set
    {
        if (string.IsNullOrEmpty(value) || value.IndexOf('#') < 0)
            sp = value;
        else
            sp = value.Substring(0, value.IndexOf('#'));

    }
}

/// SourceUrl
public String Su { get; set; }

/// LandingType
public String Lt { get; set; }

/// LandingUrl
public String Lu { get; set; }

/// OriginalSourceClickId
public String Cosc { get; private set; }

/// CoupleRegistryId
public long R { get; set; }

/// RetailerId
public int Rt { get; set; }

public string JumpUrl { get; set; }
/// If value is 0, it will retrieve a correct value via LandingUrl host when it was saved.
public int LandingSiteId { get; private set; }

/// CreatedBy
public String Cb { get; set; }

/// CoupleRegistryItem Id
public long Ri { get; set; }

/// CoupleId
public long C { get; set; }

public string IPAddress { get; private set; }
```

#### SavePixelClick
 执行存储过程： tracking.InsertRegistryClick

 SQL:
 ```SQL
ALTER Procedure [tracking].[InsertRegistryClick]
    @SourceType NVarchar(40),
    @SourceSection NVarchar(40),
    @SourcePlacement NVarchar(40),
    @SourceAffiliateId Int,
    @SourceUrl NVarchar(1024),
    @LandingType NVarchar(40),
    @LandingSiteId Int,
    @LandingSiteHost NVarchar(100) = Null,
    @LandingUrl NVarchar(1024),
    @CoupleRegistryId BigInt,
    @RetailerId Int,
    @UserId BigInt,
    @SessionId NVarchar(40),
    @MachineId NVarchar(40),
    @CreatedBy NVarchar(40),
    @JumpUrl NVarchar(500),
    @UserAgent NVarchar(200),
    @CoupleRegistryItemId BigInt,
    @CoupleId BigInt,
    @OriginalSourceClickId BigInt,
    @IPAddress NVarchar(16),
    @UUID Varchar(50),
    @RegistryClickId BigInt Output,
    @OriginalMachineId NVarchar(40) Output,
    @OriginalClickExist Bit Output
As
Begin

    If Len(@LandingSiteHost) > 0
        Select @LandingSiteId = SiteId From tracking.LandingSite Where CharIndex(@LandingSiteHost, [Site]) > 0;

    If @LandingSiteId <= 0
        Set @LandingSiteId = Null;

    Declare @OriginalSourceType NVarchar(40), @OriginalSourceSection NVarchar(40),
            @OriginalSourcePlacement NVarchar(40), @OriginalSourceAffiliateId NVarchar(20), @OriginalSourceUrl NVarchar(1024)

    Set @OriginalClickExist = 0;
    If @OriginalSourceClickId > 0
    Begin
        Select
            @OriginalClickExist = 1,
            @OriginalSourceType = SourceType,
            @OriginalSourceSection = SourceSection,
            @OriginalSourcePlacement = SourcePlacement,
            @OriginalSourceAffiliateId = SourceAffiliateId,
            @OriginalSourceUrl = SourceUrl,
            @OriginalMachineId = MachineId
        From tracking.RegistryClick Where RegistryClickId = @OriginalSourceClickId;

        If @OriginalClickExist <> 1
            Set @OriginalSourceClickId = Null

        If @OriginalClickExist = 1 And @OriginalMachineId Is Not Null And @MachineId <> @OriginalMachineId
            Set @MachineId = @OriginalMachineId
    End

    Insert Into RegistryClick
    (
        SourceType, SourceSection, SourcePlacement, SourceAffiliateId, SourceUrl, LandingType, LandingSiteId, LandingUrl,
        OriginalSourceClickId, OriginalSourceType, OriginalSourceSection, OriginalSourcePlacement, OriginalSourceAffiliateId, OriginalSourceUrl,
        CoupleRegistryId, RetailerId, UserId, SessionId, MachineId, CreatedDate, CreatedBy, JumpUrl, UserAgent, CoupleRegistryItemId, CoupleId, IPAddress,ModifiedBy,ModifiedDate,UUID
    )
    Values
    (
        @SourceType, @SourceSection, @SourcePlacement, @SourceAffiliateId, @SourceUrl, @LandingType, @LandingSiteId, @LandingUrl,
        @OriginalSourceClickId, IsNull(@OriginalSourceType, @SourceType), IsNull(@OriginalSourceSection, @SourceSection), IsNull(@OriginalSourcePlacement, @SourcePlacement), IsNull(@OriginalSourceAffiliateId, @SourceAffiliateId), IsNull(@OriginalSourceUrl, @SourceUrl),
        @CoupleRegistryId, @RetailerId, @UserId, @SessionId, @MachineId, GetDate(), @CreatedBy, @JumpUrl, @UserAgent, @CoupleRegistryItemId, @CoupleId, @IPAddress , @CreatedBy,GetDate(),@UUID
    )
    Select @RegistryClickId = @@Identity;

End
 ```


#### ReturnPixel
```csharp
private void ReturnPixel(TrackingParam param)
{
    Response.StatusCode = (int)HttpStatusCode.OK;
    Response.ContentType = "image/gif; charset=UTF-8";
    Response.Cache.SetLastModified(DateTime.Now);
    Response.Cache.SetCacheability(HttpCacheability.NoCache);
    Response.Expires = -1500;
    Response.Cache.SetNoStore();
    Response.ExpiresAbsolute = DateTime.Now.AddYears(-1);

    byte[] pixel = Convert.FromBase64String(Constant.PIXEL_BYTE_BASR64_RAW_STRING); //R0lGODlhAQABAIAAANvf7wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==
    Response.AppendHeader("Content-Length", pixel.Length.ToString());
    Response.BinaryWrite(pixel);
}
```

