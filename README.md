# Kustomize使用教程

## 1. Kustomize是什么

### 1.1 Kustomize简介

```shell
	kustomize是一个通过kustomization文件定制kubernetes对象的工具，它可以通过一些资源生成一些新的资源，也可以定制不同的资源的集合，根据各种资源的Gernerator生成对应的资源的yaml，例如configmap、secrets等等···
	
	kubernetes在1.14版本之后，其内部已集成了kustomize，不需要额外手动安装。
	
	kustomize在github上目前有8K+star，超300位贡献者，在gitops领域中常常用到。
	github地址：https://github.com/kubernetes-sigs/kustomize
	相关网站：
		https://kustomize.io/
		https://github.com/kubernetes-sigs/kustomize
		https://kubernetes.io/zh/docs/tasks/manage-kubernetes-objects/kustomization/
```

### 1.2 Kustomize用途

```shell
	kustomize用途有多种，包含:
	生成资源、全局性字段更改、资源管理提交、资源patch提交等基础使用方法；
	高级的概念和用法有基准（Bases）与覆盖（Overlays）
	实际windows应用中，我们经常都只使用生成资源这一个用途，这花费一个篇章进行记录；其他的如全局性字段更改、资源管理提交、资源patch提交都只是添头用途，放在一个章节内记录；而在gitops中会使用到base/overlays，也会在后文补充记录。
```

## 2. Windows下使用kustomize

### 2.1 文件下载

```shell
	kustomize在windows下也可以使用，在github上下载windows的kustomize二进制文件。
	下载地址：
		https://github.com/kubernetes-sigs/kustomize/releases/tag/kustomize%2Fv4.5.4
	PS：kustomize_v4.5.4_windows_amd64.tar文件已下载，放置\Kustomize&Helm\Kustomize\windows下
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image1.jpg)

### 2.2 文件安装

```shell
	1）下载的kustomize_v4.5.4_windows_amd64.tar是二进制文件压缩包，解压后为二进制文件；
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image2.jpg)

```shell
	2）将二进制文件存放在/usr/bin目录下，方便在windows上使用kustomize。
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image3.jpg)

### 2.3 Kustomize使用

```shell
	windows常用的资源生成器有ConfigMapGenerator、secretGenerator，我们编写kustomization.yaml时指定资源生成器的类型，之后便可引用不同类型的kustomization.yaml生成不同的资源类型。
```

#### 2.3.1 二进制kustomize相关命令

```shell
$ kustomize.exe --help
Usage:
  kustomize [command]

Available Commands:
  build                     Build a kustomization target from a directory or URL.
  cfg                       Commands for reading and writing configuration.
  completion                Generate shell completion script
  create                    Create a new kustomization in the current directory
  edit                      Edits a kustomization file
  fn                        Commands for running functions against configuration.
  help                      Help about any command
  version                   Prints the kustomize version
```

#### 2.3.2 相关示例文件

```shell
	PS：示例用到的文件已放置在\Kustomize&Helm\Kustomize\examples文件夹下。
```

#### 2.3.3 ConfigMap Generator

```shell
	相关文件在\Kustomize&Helm\Kustomize\examples\ConfigMap Generator下	
	两个demo：
	demo1是根据已有的一个配置文件生成一份configmap；
	demo2是根据已有的多个配置文件生成一份configmap。
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image4.jpg)

##### （一）Demo1：单配置文件生成Configmap

```shell
	一个配置文件：application.properties
	一个kustomization.yaml文件
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image5.jpg)

**1）查看已有的单个配置文件**

```shell
$ cat application.properties
FOO=Bar
```

**2）查看kustomization.yaml**

```shell
$ cat kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
configMapGenerator:
- name: example1-configmap
  files:
  - application.properties
  
PS：kustomization.yaml需要我们填写两个地方：
	1.[configMapGenerator].[name]
	2.[configMapGenerator].[files]
	
	apiVersion: kustomize.config.k8s.io/v1beta1
	kind: Kustomization	
	configMapGenerator:
	- name: 【example1-configmap】#自定义的configmap名字
	  files:
	  - 【application.properties】#生成configmap时用到的源配置文件名称
```

**3）使用kustomize**

```yaml
同级目录下执行 kustomize build .
看执行完kustomize自动生成configmap的yaml到屏幕

$ kustomize.exe build .
apiVersion: v1
data:
  application.properties: |
    FOO=Bar
kind: ConfigMap
metadata:
  name: example1-configmap-g4hk9g2ff8
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image6.jpg)

```shell
也可直接重定向到同级目录下的新文件内
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image7.jpg)

**4）kustomization.yaml优化**

```yaml
	观察kustomize生成的configmap我们发现，该configmap的名字最后边带了一串随机编码："example1-configmap-g4hk9g2ff8"，这是为了区分configmap的版本，所以对名字加了随机编码
	
$ cat application-configmap.yaml
apiVersion: v1
data:
  application.properties: |
    FOO=Bar
kind: ConfigMap
metadata:
  name: example1-configmap-g4hk9g2ff8

	
	在实际生产上我们往往不需要名字带随机编码，因此我们可以关闭这一特性，使用我们自定义的名称，在kustomization.yaml添加generatorOptions字段，新的kustomization.yaml如下：

$ cat kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
configMapGenerator:
- name: example1-configmap
  files:
  - application.properties
generatorOptions:
  disableNameSuffixHash: true  
```

**5）使用新kustomization.yaml生成configmap**

```yaml
$ kustomize.exe build .
apiVersion: v1
data:
  application.properties: |
    FOO=Bar
kind: ConfigMap
metadata:
  name: example1-configmap
  
	观察发现该configmap的name不再带随意hash值
```

##### （二）Demo2：多配置文件生成Configmap

```shell
	多个配置文件：
	config.conf
	config.json
	kustomization.yaml
	mysqlconfig.json
	redisconfig.json
	tars.json
	一个kustomization.yaml文件
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image8.jpg)

**1）查看kustomization.yaml**

```shell
$ cat kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
configMapGenerator:
- name: example2-configmap
  files:
  - config.conf
  - config.json
  - mysqlconfig.json
  - redisconfig.json
  - tars.json
generatorOptions:
  disableNameSuffixHash: true
```

**2）使用kustomize**

```yaml
同级目录下执行 kustomize build .
看执行完kustomize自动生成configmap的yaml到屏幕

$ kustomize.exe build .
apiVersion: v1
data:
  config.conf: |
    <tars>
            <application>
                    <server>
                            app=TAdaptor
                            server=AlarmSyncData
                            local=tcp -h 0.0.0.0 -p 10021 -t 30000
                            logpath=/tmp/log
                            <TAdaptor.AlarmSyncData.MainTarsObjAdapter>
                                    allow
                                    endpoint=tcp -h 0.0.0.0 -p 11120 -t 60000
                                    handlegroup=TAdaptor.AlarmSyncData.MainTarsObjAdapter
                                    maxconns=200000
                                    protocol=tars
                                    queuecap=10000
                                    queuetimeout=60000
                                    servant=TAdaptor.AlarmSyncData.MainTarsObj
                                    shmcap=0
                                    shmkey=0
                                    threads=1
                            </TAdaptor.AlarmSyncData.MainTarsObjAdapter>
                            <TAdaptor.AlarmSyncData.CgiObjAdapter>
                                    allow
                                    endpoint=tcp -h 0.0.0.0 -p 8080 -t 60000
                                    handlegroup=TAdaptor.AlarmSyncData.CgiObjAdapter
                                    maxconns=200000
                                    protocol=tars
                                    queuecap=10000
                                    queuetimeout=60000
                                    servant=TAdaptor.AlarmSyncData.CgiObj
                                    shmcap=0
                                    shmkey=0
                                    threads=1
                            </TAdaptor.AlarmSyncData.CgiObjAdapter>
                    </server>
            </application>
    </tars>
  config.json: |-
    {
      "env": "dev",
      "enableZipkin": false,
      "enableModcall": false,
      "deployEnv": "local",
      "gidMappingHost": "TAdaptor.GIDMappingService.GIDMappingObj@tcp -h tadaptor-gidmapping -p 50092 -t 60000",
      "powerCapacityUrl": "http://10.10.10.10:32515/getRackClm",
      "powerDataHistoryUrl": "http://10.10.10.10/queryHistoryIndicator",
      "disableSyncPowerData": true,
      "disableMdcSyncPowerData": true,
      "mdcSyncOverpowerConfig": {
        "rackConfigCronSpec": "0 */10 * * * *",
        "rackDataCronSpec": "0 */5 * * * *",
        "rackDataMinuteRange": 10,
        "clmConfigCronSpec": "0 */10 * * * *",
        "clmDataCronSpec": "0 */5 * * * *",
        "clmDataMinuteRange": 10
      }
    }
  mysqlconfig.json: |-
    [
      {
        "name": "t_adaptor",
        "host": "10.10.10.10",
        "port": "3306",
        "user": "idc",
        "password": "idc",
        "database": "t_adaptor_326",
        "logLevel": "INFO"
      }
    ]
  redisconfig.json: |-
    [
      {
        "name": "t_adaptor",
        "host": "10.10.10.10",
        "port": "6379",
        "user": "",
        "password": "ceodGpPI8yCbgBtj",
        "database": ""
      },
      {
        "name": "t_adaptor_test",
        "host": "10.10.10.10",
        "port": "6379",
        "user": "root",
        "password": "Aop!@#2014",
        "database": ""
      }
    ]
  tars.json: |-
    {
    }
kind: ConfigMap
metadata:
  name: example2-configmap
```

```shell
	也可直接重定向到同级目录下的新文件内
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image9.jpg)

#### 2.3.4 Secret Generator

```shell
	相关文件在\Kustomize&Helm\Kustomize\examples\Secret Generator下	
	两个demo：
	demo1是根据文件生成secret；
	demo2是根据键值对生成secret。
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image10.jpg)

##### （一）Demo1：根据文件生成Secret

```shell
	一个文件：password
	一个kustomization.yaml文件
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image11.jpg)

**1）查看password文件**

```shell
$ cat password.txt
username=admin
password=secret
```

**2）查看kustomization.yaml**

```shell
$ cat kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
secretGenerator:
- name: example3-secret
  files:
  - password.txt
generatorOptions:
  disableNameSuffixHash: true
  
PS：kustomization.yaml同样需要我们填写两个地方：
	1.[configMapGenerator].[name]
	2.[configMapGenerator].[files]
	
	secretGenerator:
	- name: 【example3-secret】#自定义的secret名字
	  files:
	  - 【password.txt】#生成secret时用到的源文件名称
```

**3）使用kustomize**

```yaml
同级目录下执行 kustomize build .
看执行完kustomize自动生成secret的yaml到屏幕

$ kustomize.exe build .
apiVersion: v1
data:
  password.txt: dXNlcm5hbWU9YWRtaW4KcGFzc3dvcmQ9c2VjcmV0Cg==
kind: Secret
metadata:
  name: example3-secret
type: Opaque

PS:Secret资源清单中字段值是Base64编码加密后的："dXNlcm5hbWU9YWRtaW4KcGFzc3dvcmQ9c2VjcmV0Cg=="，不过，当在Pod中使用Secret时，kubelet为Pod及其中的容器提供的是解码后的数据
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image12.jpg)

##### （二）Demo2：根据键值对生成Secret

```shell
	一个kustomization.yaml文件
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image13.jpg)

**1）查看kustomization.yaml**

```shell
$ cat kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
secretGenerator:
- name: example4-secret
  literals:
  - username=admin
  - password=secret
```

**2）使用kustomize**

```yaml
同级目录下执行 kustomize build .
看执行完kustomize自动生成secret的yaml到屏幕

$ kustomize.exe build .
apiVersion: v1
data:
  password: c2VjcmV0
  username: YWRtaW4=
kind: Secret
metadata:
  name: example4-secret-8c5228dkb9
type: Opaque
```

![kustomize-image1](https://github.com/PlumDRain/Kustomize/blob/main/images/kustomize-image14.jpg)

## 3. Kustomize其他功能

### 3.1 全局性字段更改



### 3.2 资源管理提交&资源patch提交



## 4. 基准（Bases）与覆盖（Overlays）
