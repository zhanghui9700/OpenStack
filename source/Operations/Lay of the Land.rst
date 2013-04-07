Lay of the Land
=====================

从这一刻起本指南中，我们假设你已经有一个OpenStack环境运行起来了。本章节将帮助建立你的工作环境并使用它来带领你漫步云端。

**命令行工具**

我们推荐Openstack命令行工具和Openstack的Dashboard两者结合使用。一些用户由于使用过其他云技术背景的，可能会使用EC2兼容的API，相对于我们需要使用到的Openstack原生的API，这些EC2兼容的API使用了不同命名习惯。

我们强烈建议你从Python Package Index(PyPI)(https://pypi.python.org/)安装命令行客户端，而不是从Ubuntu或者Fedora的软件包。客户端开发的很快，所以有可能在你安装的时候，操作系统自带发行的软件包已经过时。”pip”可以从大多数的linux发行版里通过”python-pip”这个软件包得到，这个工具是用来管理PyPi的安装包。每个Openstack项目有自己的客户端，所以根据你使用的服务选择某些或者所有以下的软件包：

    python-novaclient(nova CLI)
    python-glanceclient(glance CLI)
    python-keystoneclient(keystone CLI)
    python-cinderclient(cinder CLI)
    python-swiftclient(swift CLI)
    python-quantumclient(quantum CLI)

**安装工具**

使用root从PyPi安装（升级）：

 # pip install [--upgrade] <package-name>

卸载软件

 # pip uninstall <package-name>

如果你需要更加新的客戶端版本，可以用pip加-e參數直接從git庫安裝。你必須為需要安裝的Python egg指定名稱。例如：

 # pip install -e git+https://github.com/openstack/python-novaclient.git#egg=python-novaclient

如果你需要在你的雲上支持EC2 API的話，你還需要安裝”euca2ools”軟件包或者一些其他的EC2 API的工具，這樣你可以和你的用戶有相同的視圖。如何使用EC2 API工具超出了這本手冊的範圍，但是我們會討論如何得到使用EC2的認證。

**管理命令行工具**

以下是几种 *-manage命令行工具：

    nova-manage
    glance-manage
    keystone-manage
    cinder-manage

與上文提到的客戶端工具不同，*-manage工具必須在由root在控制節點上運行，因為這些命令需要有訪問配置文件的權限，例如/etc/nova.conf，並且需要直接查詢數據庫而不是通過Openstack的API接口。

*-manage工具存在是一个遗留问题。openstack项目最终的目标是把这些遗留的*-manage工具移植到正规的客户端工具中去。到目前为止，你仍旧需要通过ssh登录到控制节点上运行*-manage工具进行维护管理操作。

**获取证书**

如果你要使用命令行工具对你的Openstack云平台进行操作，你必须有合适的证书。目前最方便的得到认证证书的方式是使用horizon dashboard。在顶部的导航栏，点击Setting链接，进入用户配置页面，在页面里你可以为dashboard视图设置语言和时区。更重要的是，这个操作改变了左列的导航栏，里面包含了Openstack API和EC2 Credentials链接，这两个链接将会得到可以在你的shell环境中source的文件，文件包含了命令行工具所需要的servide endpoints地址和你的认证信息。

点击Openstack API链接。顶部的部分列出了你的Service Endpoints的URL地址，底部是Download Openstack RC File。为了以管理员的身份查看云平台，你可以从下拉菜单中选择admin。选择你需要的project项目，然后点击Download RC。于是会得到一个名为openrc.sh的文件，内容类似如下：

 #!/bin/bash
      
 # With the addition of Keystone, to use an openstack cloud you should  
 # authenticate against keystone, which returns a **Token** and **Service   
 # Catalog**. The catalog contains the endpoint for all services the  
 # user/tenant has access to - including nova, glance, keystone, swift.   
 #  
 # *NOTE*: Using the 2.0 *auth api* does not mean that compute api is 2.0.   
 # We use the 1.1 *compute api*  
export OS_AUTH_URL=http://203.0.113.10:5000/v2.0   
      
 # With the addition of Keystone we have standardized on the term **tenant**     
 # as the entity that owns the resources.    
export OS_TENANT_ID=98333aba48e756fa8f629c83a818ad57  
export OS_TENANT_NAME="test-project"  
      
 # In addition to the owning entity (tenant), openstack stores the entity  
 # performing the action as the **user**.  
export OS_USERNAME=test-user  
      
 # With Keystone you pass the keystone password.  
echo "Please enter your OpenStack Password: "  
read -s OS_PASSWORD_INPUT  
export OS_PASSWORD=$OS_PASSWORD_INPUT   
    

注意这个操作不会以文本保存你的密码，这样做是好事情。但是当你想用这个文件source或者运行脚本，它会提示你输入你的密码，并且保存在环境变量OS_PASSWORD中。记住这个过程需要手动交互。如果你想要一个不需要交互的操作，那么把值直接保存在脚本中。但是这样做你需要格外小心这个脚本文件的安全和权限。

EC2兼容的认证可以从左边导航栏中的"EC2 Credentials"链接下载，选择你需要的项目，点击"Download EC2 Credentials"。会得到一个提供x509证书的zip文件和一个shell脚本。和前面的openrc文件不同，这些证书包含了所有可以访问你的云平台的认证信息，所以在安全的地方创建一个新目录，在新目录中解开zip文件。你会得到cacert.pem, cert.pem, ec2rc.sh and pk.pem。ec2rc.sh文件内容类似如下：

 #!/bin/bash

      
NOVARC=$(readlink -f "${BASH_SOURCE:-${0}}" 2>/dev/null) ||\  
NOVARC=$(python -c 'import os,sys; print os.path.abspath(os.path.realpath(sys.argv[1]))' "${BASH_SOURCE:-${0}}")  
NOVA_KEY_DIR=${NOVARC%/*}
export EC2_ACCESS_KEY=df7f93ec47e84ef8a347bbb3d598449a  
export EC2_SECRET_KEY=ead2fff9f8a344e489956deacd47e818  
export EC2_URL=http://203.0.113.10:8773/services/Cloud  
export EC2_USER_ID=42 # nova does not use user id, but bundling requires it  
export EC2_PRIVATE_KEY=${NOVA_KEY_DIR}/pk.pem  
export EC2_CERT=${NOVA_KEY_DIR}/cert.pem  
export NOVA_CERT=${NOVA_KEY_DIR}/cacert.pem  
export EUCALYPTUS_CERT=${NOVA_CERT} # euca-bundle-image seems to require this set
      
alias ec2-bundle-image="ec2-bundle-image --cert $EC2_CERT --privatekey      $EC2_PRIVATE_KEY --user 42 --ec2cert $NOVA_CERT"  
alias ec2-upload-bundle="ec2-upload-bundle -a $EC2_ACCESS_KEY -s $EC2_SECRET_KEY --url $S3_URL --ec2cert $NOVA_CERT"  
    

把EC2 credentials放到你的环境里，source这个ec2rc.sh文件。

**命令行的技巧和陷阱**

命令行工具可以通过--debug标记来显示调用Openstack API的过程，例如

# nova --debug list

这个例子会显示从客户端来的HTTP请求和从endpoints的回应，这些对于对Openstack API定制化客户端有帮助

Keyring Support(https://wiki.openstack.org/wiki/KeyringSupport)可能就是一个困惑的东西，自这篇文章写的时候，有一个bug(https://bugs.launchpad.net/python-novaclient/+bug/1020238)。这个bug状态是open，然后解决，但是仍然有问题，又被重新open。这个bug的问题是在某些条件，命令行工具尝试使用一个python的钥匙作为证书的缓存，在这些情况的一个子集中，会要求在每次使用时会提示一个钥匙密码。如果你发现你遇到了这个不幸的情况，添加--no-cache标识或者设置环境变量OS_NO_CACHE=1来避免证书缓存。
Note

这会造成命令行工具在每次交互的时候都需要认证。

**cURL**

优先使用Openstack API而不是命令行工具，他是一个跑在HTTP协议上的RESTful API。多数情况你会希望直接使用API而不是命令行工具，因为某一个命令行可能会存在bug。最好的使用方式是，结合cURL(http://curl.haxx.se)和其他工具来语法分析响应的JSON，例如jq(http://stedolan.github.com/jq/)

第一件事情你必须使用你的证书来认证来获取authentication token。证书包含了用户名、密码和tenant（项目）。你可以从上面我们谈到的openrc.sh文件中得到他们。token允许你不需要再次认证就能和其他的service endpoints交互。Tokens通常存在24小时比较好，当token过期后，你将会收到一个401（未被授权的）响应，然后你可以再请求一个token。

    查看你的Openstack服务条目：

    $ curl -s -X POST http://203.0.113.10:35357/v2.0/tokens \
        -d '{"auth": {"passwordCredentials": {"username":"test-user", "password":"test-password"}, "tenantName":"test-project"}}' \
        -H "Content-type: application/json" | jq .
    	    

    通过阅读JSON的响应切身体会一下目录是如何被安排的。

    为了使后续的请求更方便，我们把token储存在环境变量中

    $ TOKEN=`curl -s -X POST http://203.0.113.10:35357/v2.0/tokens \
        -d '{"auth": {"passwordCredentials": {"username":"test-user", "password":"test-password"}, "tenantName":"test-project"}}' \
        -H "Content-type: application/json" |  jq -r .access.token.id`
    	     

    现在你可以在命令行里面使用变量$TOKEN来得到你的token值
    从服务目录中选择一个服务endpoint，譬如计算的服务，然后尝试着发出一个请求，例如列出实例。

    $ curl -s -H "X-Auth-Token: $TOKEN" http://203.0.113.10:8774/v2/98333aba48e756fa8f629c83a818ad57/servers | jq .
    	    

要查询API请求是如何组织的，可以阅读Openstack API Reference(http://api.openstack.org/api-ref.html)。可以使用jq来细细研究返回的请求响应，阅读jq Manual(http://stedolan.github.com/jq/manual/)

在cURL命令中使用-s选项可以避免显示过程信息。如果你在使用cURL时遇到困难，把这个标记去掉。同样的，加上-v选项会输出详细信息，帮助排错。在cURL中还有更多有用的功能，参考man帮助手册查询所有的选项。

**服务器和服务**

作为一个管理员，通过使用Openstack工具来让你查看你的Openstack云平台。这部分将会告诉你如何总览你的云，组成，大小和状态

首先，你可以查询哪些服务器属于你的Openstack云平台，通过运行

$ nova-manage service list | sort
      

输出类似如下：

Binary           Host              Zone Status  State Updated_At
nova-cert        cloud.example.com nova enabled  :-)  2013-02-25 19:32:38
nova-compute     c01.example.com   nova enabled  :-)  2013-02-25 19:32:35
nova-compute     c02.example.com   nova enabled  :-)  2013-02-25 19:32:32
nova-compute     c03.example.com   nova enabled  :-)  2013-02-25 19:32:36
nova-compute     c04.example.com   nova enabled  :-)  2013-02-25 19:32:32
nova-compute     c05.example.com   nova enabled  :-)  2013-02-25 19:32:41
nova-consoleauth cloud.example.com nova enabled  :-)  2013-02-25 19:32:36
nova-network     cloud.example.com nova enabled  :-)  2013-02-25 19:32:32
nova-scheduler   cloud.example.com nova enabled  :-)  2013-02-25 19:32:33
      

输出显示有5个计算节点和1个云控制结点。你可以看到一个笑脸:-)，表明这个服务是起的并运行着。如果一个服务不可用，:-)会变成XXX。这表明你应该排错一下为什么服务down了。

如果你用了nova-volume（这个服务在Folsom版本后不赞成使用），你会看到一行nova-volume的服务条目。

如果你是用了Cinder，运行如下命令将会看到

$cinder-manage host list | sort 

host              zone
c01.example.com   nova
c02.example.com   nova
c03.example.com   nova
c04.example.com   nova
c05.example.com   nova
cloud.example.com nova
      

通过这两个表，你现在已经对组成你的云平台的服务器和服务有了总的概况

你也可以使用认证服务（Keystone）查看什么服务在你的云平台可用，什么服务被配置了endpoints

以下这个命令需要你在shell中设置了合适的管理权限的变量

$ keystone service-list

+-----+----------+----------+----------------------------+
| id  | name     | type     | description                |
+-----+----------+----------+----------------------------+
| ... | cinder   | volume   | Cinder Service             |
| ... | glance   | image    | OpenStack Image Service    |
| ... | nova_ec2 | ec2      | EC2 Service                |
| ... | keystone | identity | OpenStack Identity Service |
| ... | nova     | compute  | OpenStack Compute Service  |
+-----+----------+----------+----------------------------+
      

输出显示有5个配置的服务

查看每个服务的endpoint，运行

$ keystone endpoint-list

+-----+-----------------------------------------+-/+-----------------------------------------+
| id  |            publicurl                    | \|               adminurl                  |
+-----+-----------------------------------------+-/+-----------------------------------------+
| ... | http://example.com:8774/v2/%(tenant_id)s| \| http://example.com:8774/v2/%(tenant_id)s|
| ... | http://example.com:8773/services/Cloud  | /| http://example.com:8773/services/Admin  |
| ... | http://example.com:9292/v1              | \| http://example.com:9292/v1              |
| ... | http://example.com:5000/v2.0            | /| http://example.com:35357/v2.0           |
| ... | http://example.com:8776/v1/%(tenant_id)s| \| http://example.com:8776/v1/%(tenant_id)s|
+-----+-----------------------------------------+-/+-----------------------------------------+
      

服务和endpoint之间应该是一对一的映射，注意一些服务的公共URL和管理URL是不同的URLs和端口。

**网络**

接下來，看看你的雲環境中的Fixed IP網絡

$ nova-manage network list

id IPv4        IPv6 start address DNS1 DNS2 VlanID project   uuid 
1  10.1.0.0/24 None 10.1.0.3      None None 300    2725bbd   beacb3f2
2  10.1.1.0/24 None 10.1.1.3      None None 301    none      d0b1a796
    

輸出顯示兩個被配置的網絡，每個網絡包含255個IP。第一個網絡被分配給一個專門的項目，第二個網絡沒有分配。你可以手動分配，或者他也會自動被分配當一個項目啟動第一個實例的時候

查看雲平台中的floating ip，運行：

$ nova-manage floating list

2725bbd458e2459a8c1bd36be859f43f 1.2.3.4                                 None nova vlan20
None                             1.2.3.5 48a415e7-6f07-4d33-ad00-814e60b010ff nova vlan20
    

這裡，兩個floating IP可用，第一個已經被分配，第二個空閒

**用戶和項目**

列出雲平台下的項目列表，運行

$ keystone tenant-list

+-----+----------+---------+
| id  | name     | enabled |
+-----+----------+---------+
| ... | jtopjian | True    |
| ... | alvaro   | True    |
| ... | everett  | True    |
| ... | admin    | True    |
| ... | services | True    |
| ... | jonathan | True    |
| ... | lorin    | True    |
| ... | anne     | True    |
| ... | rhulsker | True    |
| ... | tom      | True    |
| ... | adam     | True    |
+-----+----------+---------+
    

用戶列表，運行

$ keystone user-list

+-----+----------+---------+------------------------------+
| id  | name     | enabled | email                        |
+-----+----------+---------+------------------------------+
| ... | everett  | True    | everett.towne@backspace.com  |
| ... | jonathan | True    | jon@sfcu.edu                 |
| ... | nova     | True    | nova@localhost               |
| ... | rhulsker | True    | ryan.hulkster@cyberalbert.ca |
| ... | lorin    | True    | lorinhoch@nsservices.com     |
| ... | alvaro   | True    | Alvaro.Perry@cyberalbert.ca  |
| ... | anne     | True    | anne.green@backspace.com     |
| ... | admin    | True    | root@localhost               |
| ... | cinder   | True    | cinder@localhost             |
| ... | glance   | True    | glance@localhost             |
| ... | jtopjian | True    | joe.topjian@cyberalbert.com  |
| ... | adam     | True    | adam@ossmanuals.net          |
| ... | tom      | True    | fafield@univm.edu.au         |
+-----+----------+---------+------------------------------+
    

Note

有時候用戶和組是一一對應的。通常在標準的系統用戶是這樣的，例如cinder, glance, nova, swift或者一個組裡面就一個用戶。

**在運行的實例**

顯示運行中的實例列表

$ nova list --all-tenants

+-----+------------------+--------+-------------------------------------------+
| ID  | Name             | Status | Networks                                  |
+-----+------------------+--------+-------------------------------------------+
| ... | Windows          | ACTIVE | novanetwork_1=10.1.1.3, 199.116.232.39    |
| ... | cloud controller | ACTIVE | novanetwork_0=10.1.0.6; jtopjian=10.1.2.3 |
| ... | compute node 1   | ACTIVE | novanetwork_0=10.1.0.4; jtopjian=10.1.2.4 |
| ... | devbox           | ACTIVE | novanetwork_0=10.1.0.3                    |
| ... | devstack         | ACTIVE | novanetwork_0=10.1.0.5                    |
| ... | initial          | ACTIVE | nova_network=10.1.7.4, 10.1.8.4           |
| ... | lorin-head       | ACTIVE | nova_network=10.1.7.3, 10.1.8.3           |
+-----+------------------+--------+-------------------------------------------+
    

不幸的是這個命令沒法告訴你更多的關於實例的細節，例如實例跑在哪個計算節點，實例的配置等等。你可以使用下列命令來單獨查看每個實例的細節：

$ nova show <uuid>>

舉例：

# nova show 81db556b-8aa5-427d-a95c-2a9a6972f630

+-------------------------------------+-----------------------------------+
| Property                            | Value                             |
+-------------------------------------+-----------------------------------+
| OS-DCF:diskConfig                   | MANUAL                            |
| OS-EXT-SRV-ATTR:host                | c02.example.com                   |
| OS-EXT-SRV-ATTR:hypervisor_hostname | c02.example.com                   |
| OS-EXT-SRV-ATTR:instance_name       | instance-00000029                 |
| OS-EXT-STS:power_state              | 1                                 |
| OS-EXT-STS:task_state               | None                              |
| OS-EXT-STS:vm_state                 | active                            |
| accessIPv4                          |                                   |
| accessIPv6                          |                                   |
| config_drive                        |                                   |
| created                             | 2013-02-13T20:08:36Z              |
| flavor                              | m1.small (6)                      |
| hostId                              | ...                               |
| id                                  | ...                               |
| image                               | Ubuntu 12.04 cloudimg amd64 (...) |
| key_name                            | jtopjian-sandbox                  |
| metadata                            | {}                                |
| name                                | devstack                          |
| novanetwork_0 network               | 10.1.0.5                          |
| progress                            | 0                                 |
| security_groups                     | [{u'name': u'default'}]           |
| status                              | ACTIVE                            |
| tenant_id                           | ...                               |
| updated                             | 2013-02-13T20:08:59Z              |
| user_id                             | ...                               |
+-------------------------------------+-----------------------------------+
    

