要创建一个新的Atomix实例，请创建一个Atomix构建器：

```java
AtomixBuilder builder = Atomix.builder();
```

构建器应该配置为本地节点配置：

```java
builder.withId("member1")
  .withAddress("10.192.19.181")
  .build();
```

除了配置本地节点信息之外，每个实例还必须配置一个发现配置，用于发现集群中的其他节点。最简单的发现形式是`BootstrapDiscoveryProvider`

```java
builder.withMembershipProvider(BootstrapDiscoveryProvider.builder()
  .withNodes(
    Node.builder()
      .withId("member1")
      .withAddress("10.192.19.181")
      .build(),
    Node.builder()
      .withId("member2")
      .withAddress("10.192.19.182")
      .build(),
    Node.builder()
      .withId("member3")
      .withAddress("10.192.19.183")
      .build())
  .build());
```

最后，必须用一个或多个分区组配置实例。公共分区组可以使用[剖面图](https://atomix.io/docs/latest/user-manual/cluster-management/partition-groups#profiles).

```java
builder.addProfiles(Profile.dataGrid());
```

