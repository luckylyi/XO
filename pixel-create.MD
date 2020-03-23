# pixel

## Create

### TryValidateParam(param)
  验证object结构，有任何错误，抛出400异常，return; JOI即可


### CreatePixelRecordExist

if (!this.DataProvider.CreatePixelRecordExist(param.RetailerId, param.RetailerRegistryCode))
{
    this.DataProvider.SavePixelCreate(param);
}
else
{
    this.AddHeaderError(param, "Duplicate registry");
}

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

    param.Id = DataUtility.ObjectToValue<long>(cmd.Parameters["@CreatePixelId"].Value, 0);

    var oldMachineId = DataUtility.ObjectToValue<string>(cmd.Parameters["@OriginalMachineId"].Value, null);
    if (!string.IsNullOrWhiteSpace(oldMachineId) && oldMachineId != param.MachineId)
    {
        LoggerProvider.Current.Error(string.Format("incorrect machineId:{0}, clickId:{1}, createPixelId: {2}", param.MachineId, param.RegistryClickId, param.Id));
        if (HttpContext.Current != null) HttpContext.Current.Response.Cookies[Constant.MACHINE_ID_COOKIE_NAME].Value = oldMachineId;
    }

    return param.Id;

