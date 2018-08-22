# Coding Practices & Conduct for DevOps

## RBAC In Helm Chart with K8S

1、RBAC 和 ServiceAccount 配置应该在单独的密钥下进行。他们是不同的东西。将 YAML 中的这两个概念拆分出来可以消除混淆并使其更清晰。

2、为 ServiceAccount 创建 helper模板

注: *serviceAccount.name 应设置为由 chart 创建的访问控制资源使用的 ServiceAccount 的名称。如果 serviceAccount.create 为 true，则应该创建一个带有该名称的 ServiceAccount。如果名称未设置，则使用该 fullname 模板生成名称，如果 serviceAccount.create 为 false，则不应创建该名称，但它仍应与相同的资源相关联，以便稍后通过手动创建的 RBAC 资源将引用它从而功能正常。如果 serviceAccount.create 为 false 且名称未指定，则使用默认的 ServiceAccount*

~~~YAML
rbac:
  # Specifies whether RBAC resources should be created
  create: true

serviceAccount:
  # Specifies whether a ServiceAccount should be created
  create: true
  # The name of the ServiceAccount to use.
  # If not set and create is true, a name is generated using the fullname template
  name
~~~

## 命名约束

1、变量名称应该以小写字母开头，单词应该用 camelcase 分隔： `chickenNoodleSoup: true` ,不可以使用 破折号连接，如：`chicken-noodle-soup: true`

2、Chart 名称应该是小写字母和数字组成，可以用破折号（-）分隔**,如：`nginx-ingress.yaml`

+ *请注意，Helm 的所有内置变量都以大写字母开头，以便将它们与用户定义的 value 区分开来，如：.Release.Name， .Capabilities.KubeVersion*

3、模板的名称不得使用camelcase命名规则

+ 模板文件名应该使用横线符号（my-example-configmap.yaml），而不是 camelcase
+ 模板文件名应该反映名称中的资源种类，gb28181-pod.yaml， recording-svc.yaml

## 版本

1、只要有可能，Helm 使用 SemVer 2 来表示版本号。（请注意，Docker 镜像 tag 不一定遵循 SemVer，因此被视为该规则的一个例外。）**

2、标签不允许 + 标志作为值。**

3、一个 Chart.yaml 文件可以指定一个 tillerVersion**

+ 最佳做法是默认为格式 >=2.4.0，其中 2.4.0 引入了 chart 中使用的新功能的版本

~~~YAML
name: mychart
version: 0.2.0
tillerVersion: ">=2.4.0"
~~~

## 格式化

1、YAML 文件应该使用两个空格缩进 Never tabs**
2、尽量使用并列的命名规则

并列方式：

~~~YAML
serverName: nginx
serverPort: 80
~~~

嵌套规则：

~~~YAML
server:
  name: nginx
  port: 80
~~~

3、模板指令在大括号之后和大括号之前应该有空格：如 {{- print "bar" -}}、 {{ print "foo" }}

Repsitory字段的URL如果有可能，优先使用`https://` 前缀

## 整型的使用

1、为了避免整型转换问题，最好将整型存储为字符串，并在模板中使用 {{int $value}} 将字符串转换为整数。

## 在使用注释

为每一行和每一个YAML文件做注释，如：

~~~YAML
  # serverHost is the host name for the webserver
serverHost: example
 # serverPort is the HTTP listener port for the webserver
serverPort: 8080

{{- /*
mychart.shortname provides a 6 char truncated version of the release name.
*/ -}}
{{ define "mychart.shortname" -}}
{{ .Release.Name | trunc 6 }}
{{- end -}}

~~~

## JSON的使用

JSON是YAML的子集，表达List时候可以使用Json：

~~~YAML
arguments:
  - "--dirname"
  - "/foo"
~~~

可以表达为：

~~~JSON
arguments: ["--dirname", "/foo"]
~~~

## CRD的使用

区分声明和使用：

+ 有一个 CRD 的声明。这是一个 YAML 文件，kind 类型为 CustomResourceDefinition
+ 然后有资源使用 CRD。CRD 定义 foo.example.com/v1。任何拥有 apiVersion: example.com/v1 和种类 Foo 的资源都是使用 CRD 的资源。

对于 CRD，声明必须在该 CRDs 种类的任何资源可以使用之前进行注册。注册过程有时需要几秒钟。
1、一种方法是将 CRD 定义放在一个 chart 中，然后将所有使用该 CRD 的资源放入另一个 chart 中。
2、将两者打包在一起，在 CRD 定义中添加一个 pre-install 钩子，以便在执行 chart 的其余部分之前完全安装它。

+ *请注意，如果使用pre-install hook创建CRD ，则该CRD定义在helm delete运行时不会被删除。*

## 其他建议

1、确保镜像仓库地址和标签和Value.yaml中配置，可以容易地换镜像。
2、确保镜像和标签可以在 values.yaml 中定义为两个单独的字段
3、PodTemplates 应声明选择器(selector)