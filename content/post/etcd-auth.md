+++
title = 'Etcd Auth模块解析'
date = 2025-05-14T11:44:21+08:00
draft = false
+++


Auth模块
本文基于etcd 3.6版本源码进行分析
auth模块文件列表如下：
```
auth/
    jwt.go         token的jwt实现
    metrics.go     metric统计，内容是auth的版本号
    nop.go         无token provider实现
    options.go     token参数选项
    range_perm_cache.go.   权限区间缓存
    simple_token.go   简单token
    store.go       用户、角色、权限的持久化存储逻辑    
```
auth模块调用过程:
```
    tp, err := auth.NewTokenProvider(cfg.Logger, cfg.AuthToken,
        func(index uint64) <-chan struct{} {
            return srv.applyWait.Wait(index)
        },
        time.Duration(cfg.TokenTTL)*time.Second,
    )
```
## 1. Token签发逻辑
Etcd token分为3种，一个是简单token、一个是jwt token、一个是无token实现。
### 简单token
```
type simpleTokenTTLKeeper struct { // size=48 (0x30)
    tokens          map[string]time.Time
    donec           chan struct{}
    stopc           chan struct{}
    deleteTokenFunc func(string)
    mu              *sync.Mutex
    simpleTokenTTL  time.Duration
}
func (tm *simpleTokenTTLKeeper) addSimpleToken(token string)
func (tm *simpleTokenTTLKeeper) deleteSimpleToken(token string)
func (tm *simpleTokenTTLKeeper) resetSimpleToken(token string)
func (tm *simpleTokenTTLKeeper) run()
func (tm *simpleTokenTTLKeeper) stop()


type tokenSimple struct {
    lg                *zap.Logger
    indexWaiter       func(uint64) <-chan struct{}
    simpleTokenKeeper *simpleTokenTTLKeeper
    simpleTokensMu    sync.Mutex
    simpleTokens      map[string]string // token -> username
    simpleTokenTTL    time.Duration
}
func (t *tokenSimple) assign(ctx context.Context, username string, rev uint64) (string, error)
func (t *tokenSimple) assignSimpleTokenToUser(username string, token string)
func (t *tokenSimple) disable()
func (t *tokenSimple) enable()
func (t *tokenSimple) genTokenPrefix() (string, error)
func (t *tokenSimple) info(ctx context.Context, token string, revision uint64) (*AuthInfo, bool)
func (t *tokenSimple) invalidateUser(username string)
func (t *tokenSimple) isValidSimpleToken(ctx context.Context, token string) bool
```
- 定时器默认每秒从内存中遍历淘汰过期token
- 不持久化，token存储在内存中，服务重启即失效
- Go routine单线程运行

### Jwt Token
```
type tokenJWT struct {
    lg         *zap.Logger
    signMethod jwt.SigningMethod
    key        any
    ttl        time.Duration
    verifyOnly bool
}
func (t *tokenJWT) assign(ctx context.Context, username string, revision uint64) (string, error)
func (t *tokenJWT) disable()
func (t *tokenJWT) enable()
func (t *tokenJWT) genTokenPrefix() (string, error)
func (t *tokenJWT) info(ctx context.Context, token string, rev uint64) (*AuthInfo, bool)
func (t *tokenJWT) invalidateUser(string)
```
- 支持不同签名参数选择
- 无需存储token
- 无法失效用户，invalidateUser，实现逻辑是空
### 无token实现
```
type tokenNop struct{}

func (t *tokenNop) enable()                         {}
func (t *tokenNop) disable()                        {}
func (t *tokenNop) invalidateUser(string)           {}
func (t *tokenNop) genTokenPrefix() (string, error) { return "", nil }
func (t *tokenNop) info(ctx context.Context, token string, rev uint64) (*AuthInfo, bool) {
    return nil, false
}

func (t *tokenNop) assign(ctx context.Context, username string, revision uint64) (string, error) {
    return "", ErrAuthFailed
}

func newTokenProviderNop() (*tokenNop, error) {
    return &tokenNop{}, nil
}
```
- 无任何实现逻辑

## 2. Metrics统计
metrics的普罗统计指标，仅包含authRevision
```
var (
    currentAuthRevision = prometheus.NewGaugeFunc(
        prometheus.GaugeOpts{
            Namespace: "etcd_debugging",
            Subsystem: "auth",
            Name:      "revision",
            Help:      "The current revision of auth store.",
        },
        func() float64 {
            reportCurrentAuthRevMu.RLock()
            defer reportCurrentAuthRevMu.RUnlock()
            return reportCurrentAuthRev()
        },
    )
    // overridden by auth store initialization
    reportCurrentAuthRevMu sync.RWMutex
    reportCurrentAuthRev   = func() float64 { return 0 }
)

func init() {
    prometheus.MustRegister(currentAuthRevision)
}
```
- 统计auth的revision数目

## 3. 权限实现逻辑
### 功能列表
用户、角色、权限的功能接口如下
```
type AuthInfo struct {
    Username string
    Revision uint64
}

// AuthenticateParamIndex is used for a key of context in the parameters of Authenticate()
type AuthenticateParamIndex struct{}

// AuthenticateParamSimpleTokenPrefix is used for a key of context in the parameters of Authenticate()
type AuthenticateParamSimpleTokenPrefix struct{}

// authstore定义了存储的接口
type AuthStore interface {
    // 启动auth特性
    AuthEnable() error

    // 关闭auth特性
    AuthDisable()

    // auth是否打开
    IsAuthEnabled() bool

    // 用户名密码认证
    Authenticate(ctx context.Context, username, password string) (*pb.AuthenticateResponse, error)

    // Recover recovers the state of auth store from the given backend
    Recover(be AuthBackend)

    // 添加新用户
    UserAdd(r *pb.AuthUserAddRequest) (*pb.AuthUserAddResponse, error)

    // 删除用户
    UserDelete(r *pb.AuthUserDeleteRequest) (*pb.AuthUserDeleteResponse, error)

    // 修改用户密码
    UserChangePassword(r *pb.AuthUserChangePasswordRequest) (*pb.AuthUserChangePasswordResponse, error)

    // 角色绑定用户
    UserGrantRole(r *pb.AuthUserGrantRoleRequest) (*pb.AuthUserGrantRoleResponse, error)

    // 查询某个用户的详细信息
    UserGet(r *pb.AuthUserGetRequest) (*pb.AuthUserGetResponse, error)

    // 撤销用户角色
    UserRevokeRole(r *pb.AuthUserRevokeRoleRequest) (*pb.AuthUserRevokeRoleResponse, error)

    // 创建新角色
    RoleAdd(r *pb.AuthRoleAddRequest) (*pb.AuthRoleAddResponse, error)

    // 角色添加新权限
    RoleGrantPermission(r *pb.AuthRoleGrantPermissionRequest) (*pb.AuthRoleGrantPermissionResponse, error)

    // 获取角色的详细信息
    RoleGet(r *pb.AuthRoleGetRequest) (*pb.AuthRoleGetResponse, error)

    // 撤销角色中的权限点
    RoleRevokePermission(r *pb.AuthRoleRevokePermissionRequest) (*pb.AuthRoleRevokePermissionResponse, error)

    // 删除角色
    RoleDelete(r *pb.AuthRoleDeleteRequest) (*pb.AuthRoleDeleteResponse, error)

    // 查询用户列表
    UserList(r *pb.AuthUserListRequest) (*pb.AuthUserListResponse, error)

    // 查询角色列表
    RoleList(r *pb.AuthRoleListRequest) (*pb.AuthRoleListResponse, error)

    // 检查用户的put权限
    IsPutPermitted(authInfo *AuthInfo, key []byte) error

    // 检查用户的range权限
    IsRangePermitted(authInfo *AuthInfo, key, rangeEnd []byte) error

    // 检查用户的delete_range权限
    IsDeleteRangePermitted(authInfo *AuthInfo, key, rangeEnd []byte) error

    // 检查用户的admin操作
    IsAdminPermitted(authInfo *AuthInfo) error

    // 生成简单token的前缀，jwt不适用
    GenTokenPrefix() (string, error)

    // 获取当前authstore的修订版本号
    Revision() uint64

    // 检查用户名密码是否匹配
    CheckPassword(username, password string) (uint64, error)

    // 关闭authStore资源
    Close() error

    // 从RPC上下文获取auth信息
    AuthInfoFromCtx(ctx context.Context) (*AuthInfo, error)

    // 从TLS RPC上下文获取auth信息
    AuthInfoFromTLS(ctx context.Context) *AuthInfo

    // 产生一个可用作root凭据的token
    WithRoot(ctx context.Context) context.Context

    // 检查用户是否有这个角色
    HasRole(user, role string) bool

    // 获取哈希加密的授权密码的强度
    BcryptCost() int
}


type authStore struct {
    // atomic operations; need 64-bit align, or 32-bit tests will crash
    revision uint64

    lg        *zap.Logger
    be        AuthBackend
    enabled   bool
    enabledMu sync.RWMutex

    // rangePermCache needs to be protected by rangePermCacheMu
    // rangePermCacheMu needs to be write locked only in initialization phase or configuration changes
    // Hot paths like Range(), needs to acquire read lock for improving performance
    //
    // Note that BatchTx and ReadTx cannot be a mutex for rangePermCache because they are independent resources
    // see also: https://github.com/etcd-io/etcd/pull/13920#discussion_r849114855
    rangePermCache   map[string]*unifiedRangePermissions // username -> unifiedRangePermissions
    rangePermCacheMu sync.RWMutex

    tokenProvider TokenProvider
    bcryptCost    int // the algorithm cost / strength for hashing auth passwords
}
```
- etcd的持久化存储是boltdb
- 在boltdb存储层面，etcd封装了一层backend，实现了事务、压缩、MVCC ，快照机制。
- Authstore的后端存储，用了etcd封装的boltdb能力，主要是以下几个方法:
```
type AuthBackend interface {
    CreateAuthBuckets()    // 创建auth的bucket    
    ForceCommit()    // 强制提交
    ReadTx() AuthReadTx    // 读事务
    BatchTx() AuthBatchTx    // 操作事务    

    GetUser(string) *authpb.User    // 获取用户
    GetAllUsers() []*authpb.User    // 获取用户列表
    GetRole(string) *authpb.Role    // 获取角色    
    GetAllRoles() []*authpb.Role    // 获取角色列表
}
```

### 权限定义
权限类型分为3种：
- READ  值为0
- WRITE  值为1
- READWRITE 值为2
```
User {
  name bytes       -> 用户名
  password  bytes   -> 密码
  roles  []string      -> 这个用户拥有的角色列表
  options     -> 选项（是否设置密码）
}

Role {
  name      bytes       -> 角色名
  keyPermissions []Permission  -> 针对 key 的权限列表
}

Permission {
  permType  enum     -> 权限类型（READ / WRITE / READWRITE）
  key       bytes      -> 起始 key
  range_end  bytes   -> 结束 key
}
```
从这个模型定义来看：
- 用户关联多个角色，使用数组存储角色列表。
- 角色关联权限数组
- 权限包含权限key，权限类型（3种），结束key。

### 权限区间缓存
etcd的权限key，更多的场景是区间逻辑。区间内key按字典序排列。所以使用区间树来进行组织。
缓存定义:
```
as.rangePermCache = make(map[string]*unifiedRangePermissions)  // map结构，key是用户名，value是下面的结构体指针。

type unifiedRangePermissions struct {
    readPerms  adt.IntervalTree        // 读权限点区间树
    writePerms adt.IntervalTree        // 写权限点区间数
}
```
在每次触发用户角色、或者权限变更时，都会重新构建缓存。
在以下函数中会触发重建：
`Recover、UserAdd、UserDelete、UserChangePassword、UserGrantRole、UserRevokeRole、RoleRevokePermission、RoleDelete、RoleGrantPermission`
refreshRangePermCache构建逻辑
- 遍历系统中的所有用户    时间复杂度：O(N)
- 合并角色中的所有权限，按unifiedRangePermissions结构体组织。
  - 遍历该用户的所有角色   时间复杂度：O(N)
  - 遍历角色中的所有权限 时间复杂度：O(N)
  - 插入区间树        O(logN)
### etcd区间树定义
- etcd 的 key 是字节序列，正常的 keys 是从 []byte{} 逐渐增长到各种内容
- etcd的key区间，权限范围是一个半开区间[Key，RangeEnd）
  - 包含从key开始的所有key
  - 直到RangeEnd（不包含RangeEnd本身）
- 整个key空间（即root角色）
  - 起点用空字节切片
  - 终点用字节 0x00（[]byte{0}）表示“终点是所有正常 key 的最大可能范围的下一个字节”，也就是说 从空 key 到所有合法 key 的最末尾

## 4. 总结分析
- 认证分为2种方式，用户名密码认证、证书认证。 密码使用blowfish hash加密存储。如果频繁的用户名密码登录，会有较大的性能开销。
- 每次修改用户、角色、权限信息，都会触发权限区间缓存重建。重建时间复杂度较高，当随着用户量和角色数目增加，呈指数级上升。这块也会出现性能瓶颈。
- Etcd权限模型使用RBAC，权限key类型对应的值分为读写、读、写。


