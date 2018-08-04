My Secure Nginx
---


nginx.conf を /etc/nginx/ 直下に配置  
com.example.com とかを /etc/nginx.conf/ 配下に配置  
SSL 類の設定は任意の場所(/etc/nginx/ssl/ とか)に配置  


# セキュリティチェック
[observatory](https://observatory.mozilla.org/)で総合チェック  
[CSP Evaluator](https://csp-evaluator.withgoogle.com/)でCSPチェック  
[Report URI](https://report-uri.com/)で設定のレポーティング  


# Nginxのインストールと起動(Ubuntu)
```sh
# 最新版Nginxをリポジトリに追加
curl http://nginx.org/keys/nginx_signing.key | sudo apt-key add -
VCNAME=`cat /etc/lsb-release | grep DISTRIB_CODENAME | cut -d= -f2` && sudo -E sh -c "echo \"deb http://nginx.org/packages/ubuntu/ $VCNAME nginx\" >> /etc/apt/sources.list"
VCNAME=`cat /etc/lsb-release | grep DISTRIB_CODENAME | cut -d= -f2` && sudo -E sh -c "echo \"deb-src http://nginx.org/packages/ubuntu/ $VCNAME nginx\" >> /etc/apt/sources.list"

# インストール
apt update && apt upgrade
apt install nginx

# 起動
service nginx start
service nginx status
```


# サーバパラメータチューニング
```sh
# ulimit -n の値を65536にする(デフォルト 1024)
sudo vim /etc/security/limits.conf
### 以下を追記
root soft nofile 65536
root hard nofile 65536
* soft nofile 65536
* hard nofile 65536
###

# カーネルのパラメータチューニング
sudo vim /etc/sysctl.conf
### 以下を追記
net.core.somaxconn = 1024
net.core.netdev_max_backlog = 5000
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_wmem = 4096 12582912 16777216
net.ipv4.tcp_rmem = 4096 12582912 16777216
net.ipv4.tcp_max_syn_backlog = 8096
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 10240 65535
###

# 再起動して有効化
sudo reboot

```


# AppArmorの有効化(Ubuntu)
```sh
# AppArmor (Ubuntu版SELinuxみたいなもの)のユーティリティをインストール
apt install apparmor apparmor-profiles apparmor-utils

# Nginx用の設定を作るためログをとる
cd /etc/apparmor.d/
aa-autodep nginx
aa-complain nginx

# Nginxの再起動
service nginx restart

# ログからNginxの許可設定を作成
aa-logprof
aa-enforce nginx

# AppArmorの有効化
service apparmor restart
service nginx restart
apparmor_status
```


# 参考
- [MDN web docs](https://developer.mozilla.org/ja/docs/Web)  


