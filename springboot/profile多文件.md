### profile

### 1 格式

### application-{env}.properties/yml,默认使用application.properties

### 2 激活

* 在application.properties里指定spring.profiles.active=${env}
* yml文件里使用文档块方式
* 命令行方式激活, --spring.profiles.active=${env}
* jvm参数 -Dspring.profiles.active=${env}
* maven 编译时指定参数 -P

