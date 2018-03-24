---
title: HashiCorp Vault AppRole Workflow
---

[HashiCorp Vault](https://www.vaultproject.io/)可以帮助管理应用中的机密信息，如数据库密码，API key等。使用HashiCorp Vault的服务也是需要认证的。在开发模式下，使用token就可以。而在产品环境，就要使用到AppRole或者CubbyHole。

# HashiCorp Vault AppRole
AppRole模式指app使用Role ID和Secret ID登陆后，Vault根据为role设定的policy产生token，app使用这个token来访问Vault里面的内容，对于内容的访问权限由role的policy来指定。  
[AppRole Authentication](https://www.vaultproject.io/guides/identity/authentication.html)介绍了使用AppRole的workflow。结合实际运用场景，我写了一些测试代码来实践AppRole的用法。代码管理在[GitHub](https://github.com/chen2liang4/vault-practice)上。

# AppRole WorkFlow
通过一个例子来了解下如何在实际中使用AppRole， 整个过程会涉及到3个参与者：
1. Vault Admin  
Vault管理员在这个流程中负责为应用创建role，制定policy等。
2. Jenkins  
Jenkins负责编译和部署应用。
3. FooApp  
FooApp是需要发布使用的一块app，其需要的机密信息保存在vault的'secret/FooApp'路径下面。

## 基本Workflow
![AppRole auth method workflow](https://www.vaultproject.io/assets/images/vault-approle-workflow-28b761b2.png)  
[完整代码](https://github.com/chen2liang4/vault-practice/blob/master/test_approle_basic_workflow.py)  
### Vault Admin为应用创建role和Policy  
```python
def setup_app(self, app_name):
    policy = """
    path "auth/approle/login" {
        capabilities = ["read", "create"]
    
    path "secret/%s" {
        capabilities = ["read"]
    }
    """ % app_name
    policy_name = '%s_role_policy' % app_name

    self.vault_client.set_policy(policy_name, policy)
    self.vault_client.write('auth/approle/role/%s' % app_name, policies=policy_name, token_num_uses=1)

self.setup_app(fooApp)
```
这段代码创建一个fooApp的role，并创建policy让其可以登陆以及读取'secret/fooApp'下的内容。

### Jenkins得到Role ID并创建Secret ID  
图中Admin负责从第2到第5步骤，实际交给部署人员或流程更合理一些。Vault Admin只负责管理Vault。Jenkins在部署的时候，根据Role ID创建Secret ID。Role ID和Secret ID就像账号和密码，每次部署的时候产生新的Secret ID，可以达到更新密码的效果。
```python
def get_role(self, app_name):
     resp = self.vault_client.read('auth/approle/role/%s/role-id' % app_name)
     role_id = resp['data']['role_id']
     resp = self.vault_client.write('auth/approle/role/%s/secret-id' % app_name)
     secret_id = resp['data']['secret_id']
     return role_id, secret_id
```
这个方法中使用app name来得到Role ID，并产生Secret ID。  
例子中jenkins使用Vault Admin的权限来操作Vault，实际可分配更小的权限来进行。 

### Jenkins创建secret id    
同时，Jenkins把Secret信息也写入Vault，供App使用。  
```python
def write_app_secret(self, app_name, confidential):
    self.vault_client.write("secret/%s" % app_name, value=confidential)

self.write_app_secret('fooApp', 'lwsrfj23-239khsd-sdf3')
```
这段代码创建了新的Secret ID，并在'secert/approle'下面写入了一个kv值: value=lwsrfj23-239khsd-sdf3。

### Jenkins把Role ID和Secret ID部署到fooApp中  
根据[The Twelve-Factor APP](https://12factor.net/zh_cn/config)建议，配置信息放到环境中。最简单的方法是把Role ID和Secret ID放到环境变量中。但环境变量容易获取，另外一个方法就是写入文件，放到只有应用程序用户能访问的目录，如.ssh/usr/role_id。  
另外还可考虑把Role ID和Secret ID在配置的时候写入配置文件，或者编译到应用中。

### App启动后使用Role ID和Secret ID登陆得到token  
App先登录，使用得到的token可以读取policy指定的内容。
```python
def login_vault(self):
    client = hvac.Client(url=self.vault_addr)
    resp = client.write('auth/approle/login', role_id=self.role_id, secret_id=self.secret_id)
    self.vault_token = resp['auth']['client_token']

def read_secret(self):
    client = hvac.Client(url=self.vault_addr, token=self.vault_token)
    resp = client.read('secret/%s' % APP_NAME)
    return resp['data']['value']
```
### 优点
1. 使用Role可以方便地管理各个应用的Secret信息和权限。
2. AppRole要求登录认证，并有记录。
3. Token的生命周期可以设定，或者指定使用次数。如果发现token泄露，Vault Admin可以立刻使token失效。应用只需重新登录得到最新有效的token，无需重新部署。例子中的policy就设定了token只能使用一次，token_nums_use=1。

### 缺点
1. 需要权衡Role ID和Secret ID的安全性。
2. Role ID和Secret ID是放在一起的，丢失的话有安全隐患。

## 使用Wrapping Token的Workflow
![AppRole auth method workflow](https://www.vaultproject.io/assets/images/vault-approle-workflow2-9e6751e3.png)  
[完整代码](https://github.com/chen2liang4/vault-practice/blob/master/test_approle_advanced_workflow.py)  
在这个工作流中，不再把Secret ID部署到app中。而是把Secret ID保存在Vault中，给app一个Wrapping Token去获取Secret ID。Wrapping Token是Vault的一个特性，可以将信息保存在一个叫CubbyHole的地方，只有使用Wrapping Token才能取得里面的内容，即使Admin都没有权限。  
图中的Trusted Entity就是例子中的Jenkins，也可以是Kubernates这样的工具。  
使用Wrapping token非常简单，Jenkins的get_role方法需要调整成这样：
```python
def get_role(self, app_name):
    resp = self.vault_client.read('auth/approle/role/%s/role-id' % app_name)
    role_id = resp['data']['role_id']
    resp = self.vault_client.write('auth/approle/role/%s/secret-id' % app_name, wrap_ttl=1)
    wrapping_token = resp['wrap_info']['token']
    return role_id, wrapping_token
```
上面代码指定了Wrapping token有效期只有1秒。  
对于app来说，也需要在登录前增加一步获取Secret ID。  
```python
def get_secret_id(self):
    client = hvac.Client(url=self.vault_addr)
    resp = client.unwrap(self.wrapping_token)
    self.secret_id = resp['data']['secret_id']
```
### 优点
1. Role ID和Secret ID分离。
2. Secret ID保存在CubbyHole中，只有使用Wrapping token才能得到。

### 缺点
1. Wrapping token是一次性使用的。也就是说app如果需要多次读取Secret信息，则需要自己保存Secret ID。当app重启，也需要重新生成Wrapping token并注入app。

## 使用Role Token的工作流
![AppRole auth method workflow](/images/vault-approle-evolved-workflow.JPG)  
[完整代码](https://github.com/chen2liang4/vault-practice/blob/master/test_approle_evolved_workflow.py)  
针对Wrapping token只能使用一次的情况，我们可以使用一个可重复使用的token来替换。 
### Vault Admin需要分别为Secret和Role创建Policy  
```python
def setup_app(self, app_name):
    secret_policy_name = '%s_secret_policy' % app_name
    secret_policy = """
    path "secret/%s" {
        capabilities = ["read"]
    }
    """ % app_name
    role_policy_name = '%s_role_policy' % app_name
    role_policy = """
    path "auth/approle/role/%s/secret-id" {
        capabilities = ["read", "create", "update"]
    }
    """ % app_name
    self.vault_client.set_policy(secret_policy_name, secret_policy)
    self.vault_client.set_policy(role_policy_name, role_policy)
    self.vault_client.write('auth/approle/role/%s' % app_name,
                            policies=secret_policy_name,
                            secret_id_num_uses=1,
                            token_num_uses=1)
```
这次我们设置生成的Secret ID，以及role登录后返回的token都是一次性使用。  

### Jenkins的get role时就不再生成Secret ID和Wrapping token，而是可多次使用生成Secret ID的token，可以称之为Role Token
```python
def get_role(self, app_name):
    resp = self.vault_client.read('auth/approle/role/%s/role-id' % app_name)
    role_id = resp['data']['role_id']
    resp = self.vault_client.create_token(policies=['%s_role_policy' % app_name])
    role_token = resp['auth']['client_token']
    return role_id, role_token
```

### App使用Role Token得到Secret ID，然后登录读取Secret
```python
def get_secret_id(self):
    client = hvac.Client(url=self.vault_addr, token=self.role_token)
    resp = client.write('auth/approle/role/%s/secret-id' % APP_NAME)
    self.secret_id = resp['data']['secret_id']
```

这种工作流解决了Wrapping token一次性使用的问题，Secret ID也是使用时才生成，而且是一次性使用的，和放在CubbyHole中相比安全性也不差。因为每次登录都要先生成Secret，性能上有所影响。

## Push模式Workflow
![AppRole auth method workflow](/images/vault-approle-ultimate-workflow.JPG)  
上面的workflow中都在App中常驻了敏感信息，如Secret ID，Wrapping token或者Role token。我们可以建立一个token provide服务，当app需要的时候发出请求，返回Wrapping token或role token。问题又转换成了token provide服务如何验证请求的合法性。如果再分配一个token或者什么凭证，就成了“蛋生鸡-鸡生蛋”的问题。  
换个思路思考下，token provide服务可以在收到请求后，不返回token，而是将token注入到app中，就像Jenkins部署Role ID一样。这样就不用去验证请求的合法性，因为不用担心给伪造者返回token。  
这个方案的坏处就是需要去实现一个这样的服务。


### 参考资料
[AppRole pulling authentication](https://www.vaultproject.io/guides/identity/authentication.html)
[Reading Vault Secrets in your Jenkins pipeline](http://nicolas.corrarello.com/general/vault/security/ci/2017/04/23/Reading-Vault-Secrets-in-your-Jenkins-pipeline.html)
[Vault Python client hvac](https://github.com/ianunruh/hvac/blob/master/hvac/tests/test_integration.py)
[Cubbyhole mode](https://www.hashicorp.com/blog/cubbyhole-authentication-principles)
