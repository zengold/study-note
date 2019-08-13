# Spring boot 数据库密码加密

## 背景

在一般情况下，我们会将数据库的账号密码都放在配置文件中来配置数据库，但是由于数据库的密码（可能还有其他）属于敏感信息，需要保护起来，所以有了对数据库密码加密的处理方式

## 使用的技术

1. 使用的数据连接池，Druid，这个连接池是阿里开发的，[Druid中文文档](https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)
2. 使用的数据库加/解密算法（这个可以根据实际需求而定），本文使用简易的PBE和base64。对密码明文，使用PBE加密后，由于会出现不可见字符，所以再使用base64进行编码。（所以解密的时候需要先用base64解码再用PBE解密）
3. springboot

## 依据

![](.\图片\druid官方文档1.png)

![](.\图片\druid官方文档2.png)

根据 Druid 官方文档，Druid 本身提供数据库密码加密的功能，但是我们希望整合到springboot中同时这个加密方式不使用它自带（非对称密钥加密）

## 流程

可以看到上面图中需要对spring加入一个bean，那么我们在springboot中可以使用配置类的方式加入，代码如下：

```java
package cn.keyou.webapp.core.config;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.alibaba.druid.pool.DruidDataSource;

@Configuration
public class DbConfig {

	@Autowired
	PasswordDecrypt passwordDecrypt;

	@Value("${spring.datasource.type}")
	private String type;
	@Value("${spring.datasource.url}")
	private String url;
	@Value("${spring.datasource.username}")
	private String username;
	@Value("${spring.datasource.password}")
	private String password;

	@Bean
	public DataSource dataSource()  {
		DruidDataSource datasource = new DruidDataSource();
		datasource.setDbType(type);
		datasource.setUrl(url);
		datasource.setUsername(username);
		datasource.setPassword(passwordDecrypt.decryptPassword(password));
		return datasource;
	}
}
```

> datasource.setPassword(passwordDecrypt.decryptPassword(password));

这里就是连接数据库时需要填写密码的地方，由于从配置文件中读取到的数据库密码可能为明文/密文，我们需要对读取到的密码进行加/解密处理后，再交给DataSource。

密码处理代码如下：

```java
package cn.keyou.webapp.core.config;

import java.nio.charset.Charset;
import java.security.InvalidAlgorithmParameterException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.spec.InvalidKeySpecException;

import javax.crypto.BadPaddingException;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import cn.keyou.commons.utils.coding.Base64;
import cn.keyou.commons.utils.extend.DataBaseCipher;

@Component
public class PasswordDecrypt {

	private static final Logger log = LoggerFactory.getLogger(PasswordDecrypt.class);

	private static final Charset UTF8 = Charset.forName("UTF-8");

	public String decryptPassword(String encryptPassword) {
		if (StringUtils.isEmpty(encryptPassword)) {
			log.error("Database password must not be empty");
			throw new IllegalArgumentException("Database password must not be empty");
		}
		if (encryptPassword.startsWith("ENC(") && encryptPassword.endsWith(")")) {
			byte[] fromBase64 = Base64.decodeToByte(encryptPassword);
			String password = null;
			try {
				password = new String(DataBaseCipher.decryptPBE(fromBase64), UTF8);
			} catch (InvalidKeyException | InvalidKeySpecException | NoSuchAlgorithmException | NoSuchPaddingException
					| InvalidAlgorithmParameterException | IllegalBlockSizeException | BadPaddingException e) {
				throw new IllegalStateException("Database password error or decode password error.");
			}
			return password;
		} else {
			return encryptPassword;
		}
	}
}
```

留意上面代码可以看到我们对数据库密码的明文/密文做了如下规定：使用密文时，需要写成 “ENC(密文)”的格式，如果不存在该格式，则视为明文。

其中使用到的加密算法，编码算法都是从网上找到的，并没有什么特别的地方。

但是需要注意的是，使用的加密算法中某些值是否需要固定的问题，就像PBE需要将口令和盐值固定，不然每次生成的密文与盐值、口令绑定，再使用其他盐值与口令解密时就会出错。

