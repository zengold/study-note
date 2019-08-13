# Springboot 配置文件注入值到静态变量

需要使用配置文件中的值，使用@Value注入到静态变量中。springboot是无法直接完成这样的事情的，会导致静态变量的值为null，而百度上很多显示使用静态变量的set方法是走不通的，以下办法可以：

```java
/**
 * 实例自身节点信息
 */
@Component
@Data
public class OwnNode {

	@Value("${node.IP}")
	private String ip;

	@Value("${node.leader}")
	private boolean leader;

	@PostConstruct
	public void init() {
		setOwnNodeConfig(this);
	}

	public static Node node;

	public static void setOwnNodeConfig(OwnNode ownNode) {
		node = new Node(ownNode.getIp(), UUIDTool.getUUID(), ownNode.isLeader());
	}

}
```

