环境需求
git jdk1.8
安装
```
sudo yum -y install git
sudo yum -y install java-1.8.0-openjdk*
```

```
cd /home
git clone https://github.com/FISCO-BCOS/evidenceSample
cd evidenceSample/evidence_toolkit/bin/
chmod +x evidence
sh compile.sh Evidence
```

待编译的智能合约保存在evidenceSample/evidence_toolkit/contracts
在evidenceSample/evidence_toolkit/下生成output文件夹及output内文件
智能合约的wrap版在output/Evidence中

 
