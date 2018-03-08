在本节中，我们详细介绍了常见场景中MSP配置的最佳实践。
1. 组织/公司和 MSP 之间的映射
我们建议组织和MSP之间存在一对一的映射。 如果选择不同的映射类型的映射，则需要考虑以下内容：
  一个组织采用多种 MSP .
  这对应于一个组织的情况，包括由 MSP 代表的各种各样的部门，无论是出于管理独立的原因，还是出于隐私的原因。 在这种情况下，peer 只能由单个 MSP 拥有，并且不会将具有来自其他 MSP 的身份的 peer 识别为同一组织的 peer 。 这意味着 peer 可以通过 gossip organization-scoped 与一组作为同一个部门成员的 peer 共享数据，而不是与组成实际组织的全集共享数据。
  多个组织使用单个 MSP
  这对应于由 membership 架构类似的管理的组织联盟的情况。需要知道的是，peer 会将组织范围的消息传播给具有同一个 MSP 身份的 peer ，而不管它们是否属于同一个实际组织。这是 MSP 定义的粒度和/或 peer 的配置的一个限制。在 Fabric 的未来版本中,这种情况将会改变---当我们转向(i) identity channel 包含所有与网络信息有关的 membership (ii) peer 的“信任区域”概念是可配置的，peer 的管理员(其 MSP 成员应被 peer 认为是合法的)在 peer 建立时指定该信任区域以接收组织范围的消息。
2. 一个组织有不同的部门（称为组织单位），组织要向部门授予对不同渠道的访问权限。
  有两种解决方法:
  定义一个 MSP 以适应所有组织成员的 membership .该 MSP 的配置将由一系列根 CA ( root CAs )，中间 CA ( intermedia CAs )和管理证书( admin certificates )组成; membership identity 将包括 organization unit(OU) .然后可以定义策略以识别特定 OU 的成员，并且这些策略可以构成 chaincode 的读取/写入策略或 chaincode 的背书策略.这种方法的局限性在于，自由交流的 peer 将把具有属于其本地 MSP 的成员身份的 peer 认作同一组织的成员，并因此将组织范围的数据（例如他们的状态）与它们进行沟通。
  定义一个 MSP 来表示每个部门.这将涉及到为每个部门指定一组根 CA ，中间 CA 和管理证书的证书，以便跨 MSP 不存在重叠的认证路径.这意味着每个子部门采用不同的中间 CA .这样子的缺点是管理多个 MSP 而不是一个 MSP ，但这避免了以前的方法中存在的问题.还可以通过 leveraging 利用 MSP 配置的 OU 扩展来为每个部门定义一个 MSP
3. 将客户端与同一组织的 peer 分离
  在许多情况下，要求身份的“类型”可以从身份本身检索(例如，可能 endorsement 需要被确保是由 peer 而不是扮演着 orderer 的客户端或节点单独派生的).
  对这些要求的支持是很有限的。
  允许这种分离的一种方法是为每个节点类型创建一个单独的中间 CA - 一个用于 client ，一个用于 peer/orderer ; 并配置两个不同的 MSP - 一个用于 client ，一个用于 peer/orderer .该组织应该访问的 channel 将需要包括两个 MSP ，而背书策略将仅 leverage 指向 peer 的 MSP .这将最终导致组织映射到两个 MSP 实例，并将对 peer 和 client 交互的方式产生一定的后果. Gossip would not be drastically impacted ,因为同一组织的所有 peer 仍然属于一个 MSP . peer 可以将某些系统 chaincode 的执行限于基于策略的本地 MSP .举例来说, peer 将只会执行" join channel "请求,如果请求被它们的 local MSP 的 admin (只能作为一个 client )签名了的(终端用户应该位于该请求的起始).我们可以解决这个不一致的情况，如果我们接受作为 peer/orderer 的 MSP 的成员的 client 又是作为 MSP 的 administrator .
  该方法要考虑的另一点是，peer 根据其 local MSP 中的请求发起者的 membership 来授权事件注册请求.显然，由于请求的发起者是 client ，请求发起者始终注定属于与请求的 peer 不同的 MSP ，并且 peer 将拒绝该请求
4. Admin and CA certificates.
   将 MSP admin certificates 设置得和其他任何证书(被 MSP 认为是 root of trust 或者说是中间 CAs )不同,是很重要的.这是一种常见（安全）的做法，将成员组成部分的管理职责与颁发新证书和/或对现有证书的验证分开。
5. 将中间 CAs 设置为黑名单
  如前面部分所述，通过重新配置机制（本地 MSP 实例的手动重新配置，以及通过正确构造的 channel 的 MSP 实例的 config_update 消息）来实现 MSP 的重新配置( reconfiguration ).
  显然，有两种方法可以确保在 MSP 中考虑的中间 CA 不再被认为是 MSP 的身份验证.
  1.将 MSP 重新配置为不再将该中间 CA 的证书包含在可信中间 CA 证书列表中. 对于本地配置的 MSP ，这意味着此 CA 的证书将从中间程序文件夹中删除。
  2.重新配置 MSP 以包含由 root of trust 产生的 CRL ，该 CRL 会 denounces 提及的中间 CA 证书
  在目前的 MSP 实现中，我们只支持方法（1），因为它更简单，不需要将不再考虑的中间 CA 列入黑名单。
\**5. CAs and TLS CAs
  需要在不同的文件夹中声明 MSP 身份的 root CA 和 MSP TLS 证书的 root CA（和相对 intermedia CA ）. 这是为了避免不同类别的证书之间的混淆。  Fabric 不禁止为 MSP 身份和 TLS 证书重复使用相同的 CA ，但最佳实践建议避免在生产中使用相同的 CA .
