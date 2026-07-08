```properties
# ==========================================  
# 通用参数校验 (Validation)# ==========================================  
error.param.format=参数格式错误，请检查数据类型后重试  
error.validation=参数校验失败  
error.enum.format=无效的枚举类型, 可选值: [{0}]  
error.not_found=[{0}] 不存在  
error.method.not_support=方法 {0} 不受支持。支持的方法：[{1}]  
error.param.blank={0} 是必须的  
error.param.range=长度必须在 [{0}] 至 [{1}] 个字符之间  
error.param.min=数值不能小于 {0}error.param.max=数值不能超过 {0}error.password.mismatch=原密码错误  
error.password.same=新密码不能与原密码相同  
  
error.query.format=参数 [{0}] 错误, 请检查后重试  
error.password.weak=密码强度不足，请包含大小写字母、数字及特殊符号  
  
# ==========================================  
# 认证与授权错误提示  
# ==========================================  
# 通用认证失败 (兜底)  
error.auth.unauthorized=认证失败，请重新登录  
  
# 登录阶段  
error.auth.credentials=用户名或密码错误  
  
# 重置密码  
error.auth.password.failed=修改密码失败  
  
# Token 错误 (合并到 Token 级别)  
error.auth.token.missing=缺少认证 Tokenerror.auth.token.invalid=Token 无效或签名验证失败  
error.auth.token.expired=Token 已过期，请重新登录  
error.auth.token.refresh.invalid=刷新 Token 无效，请重新登录  
error.auth.token.access.invalid=访问 Token 无效  
  
# Session 错误  
error.auth.session.invalid=会话失效或已被撤销，请重新登录  
error.auth.session.expired=会话已过期，请重新登录  
  
# 风控安全  
error.auth.fingerprint.missing=缺少设备指纹信息  
error.auth.device.mismatch=检测到设备环境变化，为保障安全请重新登录  
  
# 状态错误  
error.auth.disabled=账号不可用  
error.auth.no_roles=未分配角色，请联系管理员  
error.auth.forbidden=您没有权限访问该资源，如需帮助请联系管理员。  
  
# 校验码错误  
error.auth.captcha.failed=校验码生成失败  
error.auth.captcha.verify=校验码验证失败  
error.auth.captcha.often=校验码请求过于频繁  
error.auth.captcha.banned=您被禁止登陆, 请15分钟后再试  
error.auth.captcha.expired=校验码过期  
  
# ==========================================  
# 业务模块错误 (按模块划分，结构化处理)  
# ==========================================  
  
# --- 用户管理 ---error.user.not_found=用户不存在  
error.user.duplicate.username=用户名已存在  
error.user.assign.role.fail=用户角色分配失败，请检查角色数据是否有效后重试  
error.user.insert.failed=添加用户失败  
error.user.update.failed=更新用户失败  
error.user.delete.failed=删除用户失败  
error.user.delete.constraint=用户关联了其它信息，无法删除  
  
# --- 令牌管理 ---error.token.revoked=撤销令牌失败  
error.token.issued=令牌签发失败  
  
# --- 角色管理 ---error.role.not_found=角色不存在  
error.role.duplicate_code=角色代码已存在  
error.role.insert.failed=添加角色失败  
error.role.update.failed=更新角色失败  
error.role.delete.failed=删除角色失败  
error.role.assign.menus.fail=角色分配菜单权限失败  
error.role.assign.buttons.fail=角色分配按钮权限失败  
error.role.assign.resources.fail=角色分配资源权限失败  
  
# --- 菜单管理 ---error.menu.not_found=菜单不存在  
error.menu.insert.failed=添加菜单失败  
error.menu.update.failed=更新菜单失败  
error.menu.delete.failed=删除菜单失败  
error.menu.delete.constraint=该菜单下存在子菜单，请先删除子菜单！  
  
# --- 按钮管理 ---error.button.not_found=按钮不存在  
error.button.duplicate_code=按钮的代码已存在  
error.button.insert.failed=添加按钮失败  
error.button.update.failed=更新按钮失败  
error.button.delete.failed=删除按钮失败  
  
# --- 资源管理 ---error.resource.not_found=资源不存在  
error.resource.duplicate_url_method=资源的权限已存在  
error.resource.insert.failed=添加资源失败  
error.resource.update.failed=更新资源失败  
error.resource.delete.failed=删除资源失败  
  
# --- 字典管理 ---error.dict_type.not_found=字典类型不存在  
error.dict_type.duplicate_type=字典类型的类别已存在  
error.dict_type.insert.failed=添加字典类型失败  
error.dict_type.update.failed=更新字典类型失败  
error.dict_type.delete.failed=删除字典类型失败  
  
# --- 字典数据 ---error.dict_data.not_found=字典数据不存在  
error.dict_data.duplicate_value=字典类型的数据已存在  
error.dict_data.insert.failed=添加字典数据失败  
error.dict_data.update.failed=更新字典数据失败  
error.dict_data.delete.failed=删除字典数据失败  
  
  
# ==========================================  
# 系统异常  
# ==========================================  
error.db.execute=数据库操作异常  
error.db.mapping=无效的 [{0}] 参数值: [{1}]  
error.internal=系统内部错误  
error.external=操作失败，请检查输入内容  
error.business=业务逻辑错误
```