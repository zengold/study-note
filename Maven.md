# Maven

## 打包命令

```json
# 用于将项目依赖的包进行打包
clean dependency:copy-dependencies

# 用于将项目打包
clean package

# 用于将项目部署到远程仓库里
clean deploy
```



打caas-start 的包，先打其依赖

mvn clean package -pl caas-start -am  -DskipTests



单独拿出所有依赖

mvn clean dependency:copy-dependencies

