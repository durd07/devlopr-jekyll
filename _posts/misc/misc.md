---
title: MISC
---

### MISC

freenode nickname felixdu passwd felixdu_passwd email felix.du@nokia-sbell.com

## How to reset the SMC of your Mac
https://support.apple.com/en-us/HT201295
Shut down your Mac.
On your built-in keyboard, press and hold all of these keys:
Shift  on the left side of your keyboard
Control  on the left side of your keyboard
Option (Alt)  on the left side of your keyboard
While holding all three keys, press and hold the power button as well.

## Docker pull proxy
```sh
sudo HTTP_PROXY=http://10.158.100.3:8080/ docker pull jekyll/jekyll:4.2.0
```

## Docker Self Signed Certificate Registry
### Setup domain name for your host
```
yum -y install bind-utils ## install nsupdate/dig
[felixdu@Fedora ~]# nsupdate
> update delete  felixdu.hz.dynamic.nsn-net.net
> update add felixdu.hz.dynamic.nsn-net.net 3600 A 10.182.70.198 ## floating ip
> send
> quit
```
### Generate the Certificates
Linux系统下生成证书

生成秘钥key,运行:
```
$ openssl genrsa -des3 -out server.key 2048
```
会有两次要求输入密码,输入同一个即可
输入密码
然后你就获得了一个server.key文件.
以后使用此文件(通过openssl提供的命令或API)可能经常回要求输入密码,如果想去除输入密码的步骤可以使用以下命令:
```
$ openssl rsa -in server.key -out server.key
```
创建服务器证书的申请文件server.csr,运行:
```
openssl req -new -key server.key -out server.csr
```
其中Country Name填CN,Common Name填主机名也可以不填,如果不填浏览器会认为不安全.(例如你以后的url为https://abcd/xxxx….这里就可以填abcd),其他的都可以不填.
创建CA证书:
```
openssl req -new -x509 -key server.key -out ca.crt -days 3650
```
此时,你可以得到一个ca.crt的证书,这个证书用来给自己的证书签名.
创建自当前日期起有效期为期十年的服务器证书server.crt：
```
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey server.key -CAcreateserial -out server.crt
```
ls你的文件夹,可以看到一共生成了5个文件:
```
ca.crt   ca.srl    server.crt   server.csr   server.key
```
其中,server.crt和server.key就是你的nginx需要的证书文件.

### Start Registry
```sh
docker run -d \
  --restart=always \
  --name registry \
  -v "$(pwd)"/certs:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/server.key \
  -p 443:443 \
  registry:2
```

### Config Docker to use this registry
copy your ca.crt to `/etc/docker/certs.d/felixdu.hz.dynamic.nsn-net.net/ca.crt`

wget http://felixdu.hz.dynamic.nsn-net.net/uploads/ca.crt .

### Try
```
docker tag ubuntu:latest felixdu.hz.dynamic.nsn-net.net/ubuntu:latest
docker push felixdu.hz.dynamic.nsn-net.net/ubuntu:latest
```

### Docker Registry APIs
[https://docs.docker.com/registry/spec/api/](https://docs.docker.com/registry/spec/api/)

1. Get image list of private docker registry
   ```
   GET https://felixdu.hz.dynamic.nsn-net.net/v2/_catalog
   curl -k --silent https://felixdu.hz.dynamic.nsn-net.net/v2/_catalog
   ```

2. Get image tags of private docker registry
   ```
   GET https://felixdu.hz.dynamic.nsn-net.net/v2/<name>/tags/list
   curl -k --silent https://felixdu.hz.dynamic.nsn-net.net/v2/<name>/tags/list
   ```

3. Delete image from private docker registry
   ```
   DELETE /v2/<name>/manifests/<reference>
   ```

   reference can be found from
   ```
   curl -k -v --silent -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X GET https://felixdu.hz.dynamic.nsn-net.net/v2/<name>/manifests/<tag> 2>&1 | grep -i Docker-Content-Digest
   ```

## Jekyll Homepage
```sh
docker run -td --name jekyll --network host -v /home/felixdu/src/felixdu-wiki/:/wiki -v /srv/www/felixdu-wiki:/srv/www felixdu.hz.dynamic.nsn-net.net/jekyll:latest bash
```

## Win10 计算器
```
1、管理员身份运行powershell
2、Get-AppXPackage -AllUsers | Foreach {Add-AppxPackage -DisableDevelopmentMode -Register “$($_.InstallLocation)\AppXManifest.xml”}
3、由于可能会装上其他软件，开始菜单的最近添加中出现计算器即可停止运行powershell
```

## Win10 NTP Server Reset
```sh
net stop w32time
w32tm /unregister
w32tm /register
net start w32time
w32tm /resync
```

## Converting One Note to Markdown
This is an example of how to convert notes from OneNote into Markdown to use in other, less annoying Note applications. I am using PowerShell on Windows here, but other shell/scripting environments would work as well. If you want separate `.md` files, you'll need to export your OneNote pages separately. Exporting a section, or a selection of pages creates a single `.docx` file. 

- Download and install [Pandoc](https://pandoc.org/)
- Export each of your note pages to a `.docx` (Word) format using OneNote export from the File menu
- Gather all of these `.docx` files into a directory
- Open directory in File Explorer
- Open Powershell from the File Explorer using File -> Open Windows Powersell
- Run the following command:

```
ForEach ($result in Get-ChildItem | select Name, BaseName) { pandoc.exe -f docx -t markdown_strict -i $result.Name -o "$($result.BaseName).md" --wrap=none --atx-headers }

- `markdown-strict` is the type of Markdown. Other variants exist in the Pandoc [documentation](https://pandoc.org/MANUAL.html#markdown-variants)
- `--wrap=none` ensures that text in the new `.md` files doesn't get wrapped to new lines after 80 characters
- `--atx-headers` makes headers in the new `.md` files appear as `# h1`, `## h2` and so on
```
