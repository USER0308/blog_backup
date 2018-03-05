最近 VPS 上的 ss 一直连接不上，初步怀疑是服务器被墙了，想更换服务器 IP 地址，但又不想重头配置服务器环境。

向 digitalocean.com 发起 ticket，询问更换 IP 的方法，以下是他们的回复

```

Hi there,

You can use the following procedure to get a new IP address for your droplet:
1. Create a snapshot image of your droplet from the control panel.
2. Create a new droplet from your snapshot.
3. Destroy your original droplet.

Let us know if there's anything else we can assist you with, we'd be more than happy to help.

Happy sailing,
Angler Dane
DigitalOcean Platform Support

```

更重要的是，这种方法保留了之前的数据，但有些配置涉及到 IP 的还是要更改。
