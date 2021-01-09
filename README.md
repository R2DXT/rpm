# rpm
Ставим необходимый софт
```sh
yum install -y wget yum-utils gcc redhat-lsb-core  rpmdevtools rpm-build createrepo 
```
Качаем SRPM и openssh 
```sh
wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.18.0-2.el7.ngx.src.rpm
rpm -i nginx-1.18.0-2.el7.ngx.src.rpm
wget https://www.openssl.org/source/latest.tar.gz
tar -xvf latest.tar.gz
```
Достаем и изменяем scep
```sh
yum-builddep rpmbuild/SPECS/nginx.spec -y 
vi rpmbuild/SPECS/nginx.spec добрасываем опцию  --with-openssl=/home/vagrant/openssl-1.1.1i
```
Собираем пакет
```sh
rpmbuild -bb rpmbuild/SPECS/nginx.spec
	+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/nginx-1.18.0-2.el7.ngx.x86_64
	+ exit 0
```
Пакет собран 
```sh
yum localinstall -y rpmbuild/RPMS/x86_64/nginx-1.18.0-2.el7.ngx.x86_64.rpm 
systemctl start nginx
systemctl status nginx

	Active: active (running) since Sat 2021-01-09 00:51:22 UTC; 7s ago
```
Собраем repo
```sh
mkdir /usr/share/nginx/html/repo
cp rpmbuild/RPMS/x86_64/nginx-1.18.0-2.el7.ngx.x86_64.rpm /usr/share/nginx/html/repo
wget https://rpmfind.net/linux/epel/7/x86_64/Packages/c/cowsay-3.04-4.el7.noarch.rpm -O /usr/share/nginx/html/repo/
createrepo /usr/share/nginx/html/repo
```
Добавляем autoindex on, проверяем синтаксис и перезапускаем nginx 
```sh 
vi /etc/nginx/conf.d/default.conf
nginx -t
systemctl restart nginx
```
проверяем отдачу 
```sh
curl -a http://localhost/repo/
	<html>
	<head><title>Index of /repo/</title></head>
	<body>
	<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
	<a href="repodata/">repodata/</a>                                          09-Jan-2021 01:09                   -
	<a href="cowsay-3.04-4.el7.noarch.rpm">cowsay-3.04-4.el7.noarch.rpm</a>                       09-Jan-2021 18:43               43144
	<a href="nginx-1.18.0-2.el7.ngx.x86_64.rpm">nginx-1.18.0-2.el7.ngx.x86_64.rpm</a>                  09-Jan-2021 00:53             2174672
	</pre><hr></body>
	</html>
```
Подключаем repo 
```sh
vi /etc/yum.repos.d/otus.repo
```
Добавляем необходимое и проверяем 
```sh
yum repolist enabled | grep otus
otus                                otus-linux                                 2
yum install cowsay -y 
cowsay meow!
 _______
< meow! >
 -------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
