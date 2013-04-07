管理项目和用户
===================

一个OpenStack云没有用户的话,它并没有多大的价值.本章介绍如何管理用户、项目和配额.

**项目或租户？**

在OpenStack用户界面和一些文档中,有时候你会看到"项目"是指一组用户,而有时候你会也看到"租户",这两种术语是可以通用的.

这背后的原因是有历史的,OpenStack最初执行计算服务(nova)有着自己的身份验证系统,并使用的术语"项目".当认证系统分解到OpenStack身份识别服务(核心)项目,新项目中使用的术语"租户",指一个用户组.由于这一问题,一些OpenStack工具是指"项目",有些是指"租户".

在本手册中,我们将使用术语"项目",除非我们在一些示例交互工具中,使用术语"租户".

**管理项目**   
 
一个用户必须至少属于一个项目,也可以属于多个项目.因此,您应该至少添加一个项目,然后在添加用户.

**添加项目**
 
通过仪表盘来创建一个项目:

 #. 用管理员用户登录.
 #. 在左侧导航栏中选择"项目"按钮.
 #. 在右上角,点击"创建项目"按钮.

这将会弹出一个对话框,有项目名称和一个可选的描述,在底部有一个复选框来设置这个项目的状态,默认是开启的.

.. image:: ../_static/horizon-add-project.png

从上面可以看到,也可以添加项目成员和调整项目的配额.我们在讨论过这些后在做修改,当然你也可以一次性处理完上面的操作.

通过命令行创建一个项目(CLI):

通过命令行添加一个项目,你需要使用keystone工具,使用"租户"代替"项目":

``# keystone tenant-create --name=demo``

这将创建一个新项目命名为"demo".可以用 --description <tenant-description> 参数添加一些描述,这是非常有用的.也可以用 --enable false 参数创建一个禁用状态的租户,不指定是默认开启状态.

**配额**

OpenStack提供了大量配额选项,并都是针对租户的配额(而不是用户).作为一个管理用户在仪表盘中你可以看到(但不能编辑)一个"配额"导航栏的默认配额.这些默认项目配额都是在云控制器上 nova.conf 里默认的.

如果你不更改配额限制,系统会是使用以下默认配额.

                               nova.conf文件里配额配置选项的描述

========================================  ========================================
            选项默认值                      (类型) 描述   
========================================  ========================================
quota_cores=20                            (IntOpt) 允许租户使用的CPU核数   
quota_floating_ips=10                     (IntOpt) 允许租户使用的浮动IP数  
quota_gigabytes=1000                      (IntOpt) 设置租户的网络带宽  
quota_injected_file_content_bytes=10240   (IntOpt) 允许注入文件的字节数  
quota_injected_file_path_bytes=255        (IntOpt) 允许注入文件路径的字节数  
quota_injected_files=5                    (IntOpt) 允许注入文件的数量  
quota_instances=10                        (IntOpt) 允许租户创建实例的数量  
quota_key_pairs=100                       (IntOpt) 允许每个用户的密钥对数量  
quota_metadata_items=128                  (IntOpt) 允许每个实例的元数据项数量  
quota_ram=51200                           (IntOpt) 允许租户使用的内存大小  
quota_security_group_rules=20             (IntOpt) 每个安全组中规则的数量  
quota_security_groups=10                  (IntOpt) 每个租户创建安全组的数量  
quota_volumes=10                          (IntOpt) 每个租户使用逻辑卷的数量   
========================================  ========================================

其它配置选项(http://docs.openstack.org/folsom/openstack-compute/admin/content/list-of-compute-config-options.html)

最简单的方式来改变默认的项目配额就是在你的云控制器上编辑 nova.conf 文件.配额是由 nova-schedler 服务执行,所以你一旦改变默认配额选项,你必须重新启动该服务.

通过仪表盘来查看和编辑个别项目的配额:   
 #. 使用"项目"导航按钮链接获得到您现有项目的列表.    
 #. 找到要修改的项目并从下拉菜单中选择"Modify Quotas",修改相应的配额,最后点击"Save"完成修改.

通过命令行方式查看和编辑个别项目的配额,请按照下列步骤操作:     
你可以在命令行方式访问和修改配额,但这有点复杂.这是通过使用 keystone 来获取租户的ID,然后再用 nova-manage 查看.

1. 要列出一个项目的配额,必须要先使用 keystone 客户端工具找到它的ID:

 ``# keystone tenant-list | grep <tenant-name>`` 

 | 98333a1a28e746fa8c629c83a818ad57 | <tenant-name> | True | 

2. 回想一下,keystone 客户端工具使用"租户", nova 客户端工具使用"项目"为同一个概念.为了查看项目的配额,我们必须使用上面例子中获取的ID:98333a1a28e746fa8c629c83a818ad57:

 ``# nova-manage project quota 98333a1a28e746fa8c629c83a818ad57``

 | metadata_items: 128    
 | volumes: 10  
 | gigabytes: 1000  
 | ram: 6291456  
 | security_group_rules: 20  
 | instances: 1024  
 | security_groups: 10  
 | injected_file_content_bytes: 10240  
 | floating_ips: 10  
 | injected_files: 5  
 | cores: 2048  

注意:nova-manage project quota 后面必须指定ID,输入项目名称会报错.

现在给 floating_ips 数量提高至20,我们可以用 --key 和 --value 来增加 ip 数:

 ``# nova-manage project quota 98333a1a28e746fa8c629c83a818ad57 --key floating_ips --value 20``

 | metadata_items: 128   
 | volumes: 10   
 | gigabytes: 1000  
 | ram: 6291456   
 | security_group_rules: 20   
 | instances: 1024   
 | security_groups: 10  
 | injected_file_content_bytes: 10240  
 | floating_ips: 20  
 | injected_files: 5  
 | cores: 2048  


**用户管理**    

在命令行用户管理用户非常不方便.需要多条命令才能完成一个任务,并且要是用UUID,而不是象征性的名字.在实践中,人们通常不会使用命令行管理用户.幸运的是,OpenStack 仪表盘提供了一个合理的接口.此外,许多网站编写的自定义脚本也可能会适合您.

**创建用户**

要创建一个新的用户,您将需要以下信息:  

 * 用户名
 * 邮箱
 * 密码
 * 所属主要项目
 * 角色

用户名和电子邮件都是不言而喻的,虽然你的网站可能有本地习惯,但是这样便于观察.设置和更改密码的认证服务,需要管理员权限.在 Folsom 版本中,用户不能更改自己的密码.创建完用户和密码后,必须牢记分配的用户名和密码.项目必须在个创建用户之前存在.角色就是一个"会员".拿来直接用的:

 * "member": 一个典型的用户.
 * "admin": 超级管理员用户,在所有项目中,你应该谨慎使用它.

它可以定义其它角色,但很少这样做.

一旦你收集了这些信息,创建用户只是在仪表盘上的一个web表单形式,类似我们所见过的,可以发现"用户"链接在"Admin"导航栏上,然后点击右上角"创建用户"按钮.

修改用户也从"Users"的页面.如果你有大量的用户,这个页面会很拥挤.在页面的顶部有"Filter"可以用来搜索相关用户列表,与创建用户对话框相似,可以通过"Edit"或下拉菜单中的动作来修改用户信息.

**关联用户到项目**
 
许多网站运行与用户相关的只有一个项目.这是一种较为保守和简单的管理用户选择.在管理上,一个用户报告出现很明显问题的一个实例或配额,如果它们在一个项目中,用户不必担心它们的行为是哪个项目.然而,需要注意在默认情况下,任何用户都可以影响到这个项目下其他用户资源的使用额度.也可以让用户关联多个项目,这样的组织比较有意义.

在仪表盘"项目"页面可以关联现有的用户到一个额外的项目或删除它们从一个旧的项目,通过选择"项目"页面的指示板"修改用户":

.. image:: ../_static/horizon-user-project.png

在这个视图中,你可以做许多有用和危险的事情。

在标题为"All Users"表格中，将会列出这个项目所有的用户。用户过多，显示可能会很长，在顶部有过滤器可以限制输入用户名来搜索。

在这里,点击"+"将添加一个用户到项目,然后点击"-"将删除它们.

这里存在危险性,就是可能会改变成员的角色.在"Project Members"列表中的用户名后面的下拉列表中,一般情况下,这个值应该被设置为"Member"，这个例子意在说明，管理员用户这个值是"admin"。 **它是非常重要的,"admin"是全局用户,而不是每个项目,因此授予用户admin角色时候就等于赋予该用户在任何项目里管理整个云的权利** .

按照惯例,典型的应用是在一个单一的项目里,该项目创建默认设置云管理用户.如果您的管理用户使用云资源来启动和管理,强烈建议您使用单独的用户账户来管理访问权限和云正常运作,它们在不同的项目里.

**自定义授权**
 

缺省的授权设置只允许管理用户创建代表不用的项目资源.OpenStack处理两种类型的授权策略:

 * 操作为主:操作指定访问特定的操作标准,可能于特定属性的控制权.
 * 资源型:对特定资源的访问是否可能授权或根据配置的资源（目前仅适用于网络资源）的权限.从部署到实际OpenStack的服务执行不同的授权策略部署.

策略引擎读取policy.json文件的条目.这个文件的实际位置可能会有所不同,它通常是在/etc/nova/policy.json.在系统运行时,您可以更新条目,而不必重新启动服务.目前更新这些的唯一方法就是编辑策略文件

OpenStack的服务的策略引擎直接匹配测雒.一个规则表明了这些策略的元素.例如,在一个compute:create:[["rule:admin_or_owner"]]声明,这项策略是compute:create和规则是admin_or_owner.

策略是来诱发OpenStack策略引擎只要其中一个匹配一个API操作或特定OpenStack属性被使用给一个特定的操作.例如,在实例上,compute:create:策略用户每次发送一个POST /v2/{tenant_id}服务请求到OpenStack Compute API服务器.策略也可以与特定的API进行扩展.例如,如果一个用户需要一个compute_extension:rescue属性由提供程序定义的扩展属性触发操作规则测试.

一个授权策略可以由一个或多个规则组成.如果有多个规则指定,评估政策是否成功在于任何规则评估成功,如果一个API操作匹配多个策略,然后所有的策略必须评估成功.同时,授权规则是递归的.一旦一个规则匹配,规则(s)可以决定另一个规则,直到达到最后一个规则.这些定义的规则:

 * 基于角色的规则:成功提交请求的用户具有指定的角色.比如管理员提交一个实例"role:admin"是成功的.  
 * 字段规则: 如果字段指定的资源在当前请求匹配一个特定的值就评估成功.例如"field:networks:shared=True"属性共享的网络资源被设置为True.
 * 通用规则:比较属性与用户的安全凭据中提取的一种属性的资源和评估成果比较成功的.比如"tenant_id:%(tenant_id)s"是成功的,如果租户标识符在资源里等于租户标识用户提交请求.

以下是nova里一段默认policy.json文件的内容:
{
    "context_is_admin":  [["role:admin"]],

    "admin_or_owner":  [["is_admin:True"], ["project_id:%(project_id)s"]], 

**[1]**

    "default": [["rule:admin_or_owner"]],  

**[2]**

    "compute:create": [],

    "compute:create:attach_network": [],

    "compute:create:attach_volume": [],

    "compute:get_all": [],

    "admin_api": [["is_admin:True"]],

    "compute_extension:accounts": [["rule:admin_api"]],

    "compute_extension:admin_actions": [["rule:admin_api"]],

    "compute_extension:admin_actions:pause": [["rule:admin_or_owner"]],

    "compute_extension:admin_actions:unpause": [["rule:admin_or_owner"]],

    "compute_extension:admin_actions:suspend": [["rule:admin_or_owner"]],

    "compute_extension:admin_actions:resume": [["rule:admin_or_owner"]],

    ...

    "compute_extension:admin_actions:migrate": [["rule:admin_api"]],

    "compute_extension:aggregates": [["rule:admin_api"]],

    "compute_extension:certificates": [],

    "compute_extension:cloudpipe": [["rule:admin_api"]],

    ...

    "compute_extension:flavorextraspecs": [],

    "compute_extension:flavormanage": [["rule:admin_api"]],  

**[3]**

}

[1] 成功的计算规则,如果当前用户是管理员或所有者的请求中指定的资源（承租人标识符相等）.  
[2] 显示默认的策略,始终是评估API操作不匹配的策略的policy.json.  
[3] 显示一个策略,限制管理员使用管理API的能力.  

在某些情况下,某些操作应限制只有管理员才能执行.因此,作为进一步的例子,让我们考虑何样的策略文件进行修改的情况下,我们可以让用户创建自己的策略配置:
"compute_extension:flavormanage": [],

**有问题的用户(某个用户干扰了其它用户)**


在很多情况下,当用户在你的云中会破坏其他用户,有时故意和恶意,其它可能会意外.了解情况,可以让你做出更好的决定如何处理.

例如:A组的用户有非常计算密集型任务的情况下,利用大量的计算资源.这时负载的计算节点上,会影响其他用户.在这种情况下,请查看您的用户使用的情况.对这种情况,高密度计算方案是常见的,可以把您的云主机聚合或应适当的规划隔离.

另一个例子是一个用户消耗了非常大的带宽量.再次,关键是要了解用户在做什么.如果他们自然是需要大量的带宽,您可能需要限制其传输速率,以不影响其他用户或将它们移动到一个区域更多的可用带宽里.另一方面,也许用户的实例被黑客入侵,并发动DDOS攻击的成为僵尸网络的一部分.这个问题的解决方案是一样的,如果网络上的任何其他的服务器已经被黑客入侵.联系用户,使他们有时间作出反应.如果他们不回应,关闭实例.

最后一个例子是,如果一个用户反复使用云资源.联系用户,并了解他们正在尝试做的.也许他们不明白他们在做什么是不适当的或可能有问题的资源,他们正在试图访问,是造成他们请求队列或滞后的.

系统管理常常被忽视的一个关键因素是,最终用户是系统管理员存在的原因. 要了解用户他们所要做的事情,看看您的环境可以更好地帮助他们实现自己的目标.
