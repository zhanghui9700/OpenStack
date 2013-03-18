我们为什么写这本书
========================

2013年2月份，多位作者在5天内集合完成了本书的编写。在编写期间他们消耗了大量的咖啡应和德克萨斯奥斯丁市订到最好的外卖。编写本书的原因是能将我们多年的部署和维护OpenStack的经验贡献和分享给其他人。同时在长时间维护OpenStack的关键岗位工作后，我们也希望能有一份文档成为日常管理员的常备手册，用于指导日常的运维工作。本文档也能在部署决策时提供更多技术细节信息。

编写本书出于2个目标。第一：在设计和建立正式OpenStack环境时提供指导。在阅读完本文档后对于需要收集什么信息，如何组织计算，网络，存储资源和相关软件包有更好的理解第二：提供日常系统管理运维工作的指导。

本书是采用一种称为Book Sprint的方式编写的。这是帮助快速编写书籍的一种开发方式。详细信息可以参考Book Sprint网站(http://www.booksprints.net)

为什么选择OpenStack

OpenStack可以建立一个IaaS的云服务(Infrastructure as a Service，基础架构即服务)。如果你打算在现有资源上建立起相应的云服务方案，OpenStack是一个很好的选择。OpenStack使用开源方式建立，并设计可运行于普通的硬件平台上。事实上，现有的很多OpenStack初始验证系统就是搭建在各种组合的服务器和网络环境上，这些服务器和网络环境就是维护人员手头能使用的空闲资源所拼凑起来的。OpenStack也具有高扩展性，通过简单的增加计算和存储资源，你可以在不影响服务的同时逐渐增加云容量。HP和Racksapce等组织已使用基于OpenStack搭建了大规模的公有云服务。

如果你刚接触OpenStack，很快会发现OpenStack不是一个打包软件集合并通过简单安装后就可以运行。相反，OpenStack是一个系统，通过集成一批不同的技术来构建云。这种构建方式提供了最大限度的灵活性，但刚开始使用时也会被众多的配置选项所混淆。

致谢和感激

OpenStack基金会为本书编写赞助了赴奥斯丁市的机票，住宿(包括风暴后一晚有惊无险的断电事件)和可口食物。在这间奥斯丁市Rackspace的办公室内，我们在一周内募集了1万美元用于书籍的编写。本书所有作者来自于OpenStack基金会。你可以通过基金会网站http://openstack.org/join加入OpenStack基金会。

开始编写的第一天我们在白板上贴满了不同颜色的即时贴。我们用这种方式开始将原来含糊混沌的想法转变成了清晰的章节。

在编写过程中，我们融入了积累的经验和在团队成员中间激发出的想法。在编写的间隙，我们不断回顾本书结构并进一步修正。经过努力工作使得本书能呈现在你的面前。

团队成员包括: Diane Fleming - Diane不知疲倦为OpenStack API文档工作，并愿意继续在本项目上提供帮助。

Tom Fifield - Tom曾在粒子物理学实验(如：欧洲强子对撞机ATLAS项目)中学习了关于计算的可测量性。现在他使用OpenStack的生产环境用于支持澳大利亚的公共研究部门。Tom居住于澳大利亚墨尔本市，他在闲暇时参与OpenStack的文档工作。

Anne Gentle - Anne是OpenStack文档协调员，同时也同Open Street Maps小组一起工作。她作为个人志愿者参与了2011年Google Doc Summit。 Anne曾经在FLOSS Manuals’ Adam Hyde的指导下使用Sprints的方式工作过。Anne居住在德克萨斯的奥斯丁市。

Lorin Hochstein - Lorin本来是一位学者，现在已转变成一个软件开发/运维人员。目前他作为云服务首席架构师任职于Nimbis Services。在这家公司他使用OpenStack环境用于部署科学计算相关应用。他从很早的Cactus版就开始接触OpenStack。在就职于Nimbis Services之前，Lorin工作于南加州大学信息科学委员会(USCISI)，主要工作是OpenStack上扩展高性能计算。

Adam Hyde - Adam指导了本书Book Sprint的编写方式。Adam是Book Sprint方法论的发明者也是该方法上最有经验的指导者。关于Book Sprint方法请参考http://www.booksprints.net/。Adam也是FLOSS Manuals的创始人。FLOSS Manuals是一个拥有3000名开发者的社区，该社区为免费软件提供免费编写的说明书。Adam还是Booktype项目的创始人和项目经理，Booktype是书籍撰写，编辑，出版(在线和印刷)的一个开源项目。

Jonathan Proulx - Jonathan是麻省理工学员计算机科学和人工智能实验室的高级系统管理员。为了在研究项目中尽可能充分利用计算资源，他很早就开始使用OpenStack。为了更好的学习他开始参与OpenStack文档的贡献和校验工作。

Everett Toews - Everett Toews就职于Rackspace，工作方向是使得OpenStack和Rackspace更易于使用。他具有多种角色，开发人员，OpenStack倡导者和运维人员。他开发web应用，在研讨会上教学，在全球不同地方发表演说也为大学，商业公司部署OpenStack的生产环境。

Joe Topjian - 在Cybera Joe设计并部署了多个云系统。Cybera是加拿大亚伯达省一个非盈利组织提供基础架构以支持企业家和本地研究员。Joe在维护这些云系统的同时也累计了丰富的troubleshooting技巧。

我们感谢所有Rackspace公司热情的员工。Emma Richards(Rackspace客户关系)照顾了我们的午餐订单，同时不介意大堆从墙上掉落带粘性的即时贴。Betsy Hagemeier(热情的总经理助理)帮我们安排了房间并安顿下来。Rackspace的称作“胜利者”的不动产小组响应超快。(译者：Rackspace还有个Real Estate Team，难道炒房的:)) Adam Powell(IT部)提供了每日必须的宽带，同时为屏幕不够的作者提供了额外的第二台显示器。封面设计师Chris Duncan将周五的一封询问邮件(什么时候能完成什么之类的询问)变成了周五的任务完成邮件，完成结果包括源文件和开源的字体。周三晚上我们同奥斯丁OpenStack Meetup group举行了一小时愉快聚会，感谢Rackspace员工 Katie Schmidt组织了这次聚会。没有你们的支持和鼓励，我们将无法完成本书。

我们也得到了来自于欧洲核子研究组织(European Organisation for Nuclear Research, CERN)的帮助和支持。在我们重新审核本书前，Tim Bell提供了本书大纲的的反馈。Sébastien Han慷慨的贡献出他的blog。Oisin Feeley在我们发出帮助要求后立刻通过邮件给出了反馈，帮助我们做了一些修改。

你需要具备的知识

我们假设你有Linux系统管理的经验并能管理多台Linux环境。具体来说，本书所使用的是Ubuntu的Linux发行版。我们也需要你有一些SQL数据库的经验，本书将涉及到安装和维护MySQL数据库，并需要运行部分SQL查询语句。我们还假设你熟悉虚拟化。

你也需要有Linux环境配置网络部分的经验，OpenStack中最复杂的一部分就是网络的配置。你需要熟悉常用的概念如：DHCP，Linux bridges，VLANs和iptables。除此之外，你还需要能找到具有专业网络硬件经验的人有配置交换机，路由等设备，在OpenStack搭建过程中可能需要他们的参与协助。

(译者：提供以下相关人员信息，有的有照片哦本书作者 Diane Fleming：http://www.linkedin.com/profile/view?id=1992535 Tom Fifield：http://www.linkedin.com/in/tomfifield Anne Gentle：http://www.linkedin.com/profile/view?id=1977812 Lorin Hochstein：http://www.linkedin.com/profile/view?id=18432136 Adam Hyde：http://www.linkedin.com/profile/view?id=34809539 Jonathan Proulx：http://www.linkedin.com/profile/view?id=166062713 Everett Toews：http://www.linkedin.com/profile/view?id=29659690 Joe Topjian：http://www.linkedin.com/profile/view?id=11683813

Rackspace员工： Emma Richards： Betsy Hagemeier：http://www.linkedin.com/profile/view?id=40772775 Adam Powell：无 Chris Duncan：无 Katie Schmidt：http://www.linkedin.com/profile/view?id=86814521

CERN： Tim Bell：http://www.linkedin.com/profile/view?id=52400128 Sébastien Han：无 Oisin Feeley：无 )
