# MD5
Message Digest Algorithm MD5（消息摘要算法第五版）为计算机安全领域广泛使用的一种散列函数，用以提供消息的完整性保护。

## 特点
1. 压缩性：任意长度字符串根据散列哈希算法得到一个16进制32位字符长度的字符串
2. 容易计算：从原数据计算出MD5值很容易。
3. 抗修改性：对原数据进行任何改动，哪怕只修改1个字节，所得到的MD5值都有很大区别。
4. 强抗碰撞：已知原数据和其MD5值，想找到一个具有相同MD5值的数据（即伪造数据）是非常困难的。
5. 不可逆的算法：由A字符串加密成B字符串，B是永远无法反推出A的。

### 缺点
只对密码进行 md5 加密是不够的，因为相同的明文，加密出来的结果是一样的，所以容易被反推。

### 改进
在密码后面加上一段很长的字符，再计算 md5 ，那反推出原始密码就变得非常困难了。加上的这段长字符，我们称为盐（Salt），通过这种方式加密的结果，我们称为密码加盐。

## 工具类
```java
public class Md5Utils {
	/**
	 * 对给定的字符使用md5进行加密，返回加密以后的字符串
	 * 
	 * @param origin 要加密的字符串
	 * @return 加密以后的
	 */
	public static String getMd5(String origin) {
		// 1) 使用静态方法，创建MessageDigest对象
		try {
			MessageDigest md = MessageDigest.getInstance("MD5");
			// 2) 将字符串使用utf-8进行编码，得到字节数组
			byte[] input = origin.getBytes("utf-8");
			// 3) 使用digest(input)对字节数组进行md5的哈希计算，得到加密以后的字节数组，长度是16个字节。
			byte[] num = md.digest(input);
			// 4) 使用类BigInteger(1, 加密后的字节数组)，将这个二进制数组转成无符号的大整数
			BigInteger big = new BigInteger(1, num); // 1 正数， -1表示负数
			// 5) 将这个大整数转成16进制字符串，参数为多少进制
			return big.toString(16);
		} catch (Exception e) {
			e.printStackTrace();
			throw new RuntimeException(e);
		}
	}
}
```

## 测试类
```java
    /**
     * 明文               密文    (彩虹表，撞库)
     * 123      202cb962ac59075b964b07152d234b70
     *
     * 1、md5加密算法不可逆，确保安全性
     * 2、md5算法固定，所以同一个密码按照md5加密后的结果一样,不安全
     * 3、如何解决？  密码字段 = md5(用户名+密码+Salt)
     */
    @Test
    public void testMd5() {
        // 用户名、密码
        String username = "Tom";
        String password = "123";
        // 随机盐
        String uuid = UUID.randomUUID().toString();

        // 加密： 对用户名、密码、随机盐(salt)
        String md5 = Md5Utils.getMd5(username+password+uuid);
        System.out.println("md5 = " + md5);
    }
```