# CA 签发证书

该篇将使用java代码，使用已经存在的CA证书，签发证书。

由于网络上虽然容易找到各种各样的代码，但是很少有相关的流程说明，在签发的流程以及函数使用上基本无什么可用的资料，所以记录下来。

## 关于证书的基础知识

关于数字签名、数字证书可以参考 www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html

其实java自带程序，可以在命令行中使用keytool工具，来完成 证书的管理，具体参考 http://huangqiqing123.iteye.com/blog/2427042

## 正常流程

A 是 CA，B 是带签发机构，B 生成密钥对，并用 VK 签名 DN 得到 CSR，A 用 VK 签发 B 的 CSR 得到 B 的证书文件

可以使用keytool来实验一下流程：

1. 生成一个密钥对
2. 生成一个 ca 证书 （根证书）
3. 使用密钥生成证书签名请求（CSR）
4. 使用根证书签名

# 代码

## SignedCertInfo.java

```java
package cn.keyou.ca;

public class SignedCertInfo {

	/**
	 * 名称/姓氏
	 */
	private String CN;

	/**
	 * 组织单位名称
	 */
	private String OU;

	/**
	 * 组织名称
	 */
	private String O;

	/**
	 * 城市/区域名称
	 */
	private String L;

	/**
	 * 省/市/自治区名称
	 */
	private String ST;

	/**
	 * 双字母国家/地区代码，中国 - cn
	 */
	private String C;

	/**
	 * 颁发者证书库路径
	 */
	private String keyStorePath;

	/**
	 * 颁发者的所在证书库密码
	 */
	private String keyStorePass;

	/**
	 * 颁发者/签发者证书的别名
	 */
	private String issuerAlias;

	/**
	 * 颁发者/签发者 证书的密码
	 */
	private String issuerAliasPass;

	/**
	 * 证书所有者/被签发证书 的别名
	 */
	private String subjectAlias;

	/**
	 * 证书所有者/被签发证书 的密码
	 */
	private String subjectAliasPass;

	/**
	 * 证书有效天数
	 */
	private int validity;

	/**
	 * 证书所有者/被签发证书 的（输出）路径
	 */
	private String subjectPath;

	public String getCN() {
		return CN;
	}

	public void setCN(String cN) {
		CN = cN;
	}

	public String getOU() {
		return OU;
	}

	public void setOU(String oU) {
		OU = oU;
	}

	public String getO() {
		return O;
	}

	public void setO(String o) {
		O = o;
	}

	public String getL() {
		return L;
	}

	public void setL(String l) {
		L = l;
	}

	public String getST() {
		return ST;
	}

	public void setST(String sT) {
		ST = sT;
	}

	public String getC() {
		return C;
	}

	public void setC(String c) {
		C = c;
	}

	public String getKeyStorePath() {
		return keyStorePath;
	}

	public void setKeyStorePath(String keyStorePath) {
		this.keyStorePath = keyStorePath;
	}

	public String getKeyStorePass() {
		return keyStorePass;
	}

	public void setKeyStorePass(String keyStorePass) {
		this.keyStorePass = keyStorePass;
	}

	public String getIssuerAlias() {
		return issuerAlias;
	}

	public void setIssuerAlias(String issuerAlias) {
		this.issuerAlias = issuerAlias;
	}

	public String getIssuerAliasPass() {
		return issuerAliasPass;
	}

	public void setIssuerAliasPass(String issuerAliasPass) {
		this.issuerAliasPass = issuerAliasPass;
	}

	public String getSubjectAlias() {
		return subjectAlias;
	}

	public void setSubjectAlias(String subjectAlias) {
		this.subjectAlias = subjectAlias;
	}

	public String getSubjectAliasPass() {
		return subjectAliasPass;
	}

	public void setSubjectAliasPass(String subjectAliasPass) {
		this.subjectAliasPass = subjectAliasPass;
	}

	public int getValidity() {
		return validity;
	}

	public void setValidity(int validity) {
		this.validity = validity;
	}

	public String getSubjectPath() {
		return subjectPath;
	}

	public void setSubjectPath(String subjectPath) {
		this.subjectPath = subjectPath;
	}

	@Override
	public String toString() {
		return "SignedCertInfo [CN=" + CN + ", OU=" + OU + ", O=" + O + ", L=" + L + ", ST=" + ST + ", C=" + C + ", keyStorePath="
				+ keyStorePath + ", keyStorePass=" + keyStorePass + ", issuerAlias=" + issuerAlias + ", issuerAliasPass="
				+ issuerAliasPass + ", subjectAlias=" + subjectAlias + ", subjectAliasPass=" + subjectAliasPass + ", validity="
				+ validity + ", subjectPath=" + subjectPath + "]";
	}
}
```

## SignedCertInfo.java

```java
package cn.keyou.ca;

public class SignedCertInfo {

	/**
	 * 名称/姓氏
	 */
	private String CN;

	/**
	 * 组织单位名称
	 */
	private String OU;

	/**
	 * 组织名称
	 */
	private String O;

	/**
	 * 城市/区域名称
	 */
	private String L;

	/**
	 * 省/市/自治区名称
	 */
	private String ST;

	/**
	 * 双字母国家/地区代码，中国 - cn
	 */
	private String C;

	/**
	 * 颁发者证书库路径
	 */
	private String keyStorePath;

	/**
	 * 颁发者的所在证书库密码
	 */
	private String keyStorePass;

	/**
	 * 颁发者/签发者证书的别名
	 */
	private String issuerAlias;

	/**
	 * 颁发者/签发者 证书的密码
	 */
	private String issuerAliasPass;

	/**
	 * 证书所有者/被签发证书 的别名
	 */
	private String subjectAlias;

	/**
	 * 证书所有者/被签发证书 的密码
	 */
	private String subjectAliasPass;

	/**
	 * 证书有效天数
	 */
	private int validity;

	/**
	 * 证书所有者/被签发证书 的（输出）路径
	 */
	private String subjectPath;

	public String getCN() {
		return CN;
	}

	public void setCN(String cN) {
		CN = cN;
	}

	public String getOU() {
		return OU;
	}

	public void setOU(String oU) {
		OU = oU;
	}

	public String getO() {
		return O;
	}

	public void setO(String o) {
		O = o;
	}

	public String getL() {
		return L;
	}

	public void setL(String l) {
		L = l;
	}

	public String getST() {
		return ST;
	}

	public void setST(String sT) {
		ST = sT;
	}

	public String getC() {
		return C;
	}

	public void setC(String c) {
		C = c;
	}

	public String getKeyStorePath() {
		return keyStorePath;
	}

	public void setKeyStorePath(String keyStorePath) {
		this.keyStorePath = keyStorePath;
	}

	public String getKeyStorePass() {
		return keyStorePass;
	}

	public void setKeyStorePass(String keyStorePass) {
		this.keyStorePass = keyStorePass;
	}

	public String getIssuerAlias() {
		return issuerAlias;
	}

	public void setIssuerAlias(String issuerAlias) {
		this.issuerAlias = issuerAlias;
	}

	public String getIssuerAliasPass() {
		return issuerAliasPass;
	}

	public void setIssuerAliasPass(String issuerAliasPass) {
		this.issuerAliasPass = issuerAliasPass;
	}

	public String getSubjectAlias() {
		return subjectAlias;
	}

	public void setSubjectAlias(String subjectAlias) {
		this.subjectAlias = subjectAlias;
	}

	public String getSubjectAliasPass() {
		return subjectAliasPass;
	}

	public void setSubjectAliasPass(String subjectAliasPass) {
		this.subjectAliasPass = subjectAliasPass;
	}

	public int getValidity() {
		return validity;
	}

	public void setValidity(int validity) {
		this.validity = validity;
	}

	public String getSubjectPath() {
		return subjectPath;
	}

	public void setSubjectPath(String subjectPath) {
		this.subjectPath = subjectPath;
	}

	@Override
	public String toString() {
		return "SignedCertInfo [CN=" + CN + ", OU=" + OU + ", O=" + O + ", L=" + L + ", ST=" + ST + ", C=" + C + ", keyStorePath="
				+ keyStorePath + ", keyStorePass=" + keyStorePass + ", issuerAlias=" + issuerAlias + ", issuerAliasPass="
				+ issuerAliasPass + ", subjectAlias=" + subjectAlias + ", subjectAliasPass=" + subjectAliasPass + ", validity="
				+ validity + ", subjectPath=" + subjectPath + "]";
	}
}
```

## SignCertTest.java

```java
package cn.keyou.ca;

import org.junit.Test;

public class SignCertTest {

	@Test
	public void signCert() throws Exception {
		SignedCertInfo signedCertInfo = new SignedCertInfo();
		signedCertInfo.setCN("tjwoods");
		signedCertInfo.setOU("java");
		signedCertInfo.setO("keyou");
		signedCertInfo.setL("guangzhou");
		signedCertInfo.setST("guangdong");
		signedCertInfo.setC("cn");
		signedCertInfo.setKeyStorePath("C:\\Users\\2018041611\\Desktop\\cert\\tjwoods.keystore");
		signedCertInfo.setKeyStorePass("123456");
		signedCertInfo.setIssuerAlias("root");
		signedCertInfo.setIssuerAliasPass("123456");
		signedCertInfo.setSubjectAlias("testcert");
		signedCertInfo.setSubjectAliasPass("123456");
		signedCertInfo.setSubjectPath("tjwoods.crt"); // 存储签发证书的路径
		signedCertInfo.setValidity(365 * 2);// 有效期,单位:天

		System.out.println(signedCertInfo);

		// 签发证书
		SignCert.signCert(signedCertInfo);
	}

}
```

