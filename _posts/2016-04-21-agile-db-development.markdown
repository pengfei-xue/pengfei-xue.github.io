---
layout: post
title: 'agile db development'
date: 2016-04-05
author: Pengfei.X
version: 0.1
categories: [ci, ]
---


我们使用Sqitch来管理数据库变更，目前，我们可以从任何一个线上数据库的备份，通过应用Sqitch Plan变更到最新的
状态。Sqitch提供了丰富的变更管理命令，常用的包括`add`, `deploy`, `revert`, `verify`, `rebase`, `tag`。

## 什么是Sqitch Plan

Sqitch Plan包括了所有的数据库变更计划，Sqitch会根据Plan，去相应的目录(deploy, revert, verify)找数据库变更的脚本。
需要特别支出的是，变更在sqitch plan中的位置，在上线后，是绝对不能再改的。如果改掉位置，Sqitch就会不知道数据库

目前处于什么状态。项目的Sqitch Plan位于项目的sqls目录中，`sqls/sqitch.plan`，里面的内容如下所示:

	%syntax-version=1.0.0
	%project=gaia
	%uri=http://git.xxx.cc/backend/gaia
	init-run 2015-09-21T15:20:55Z Pengfei.X <penhy#.com> # init run with sqitch
	create_table_shoping_cart 2015-09-22T02:50:36Z Pengfei.X <pephy#.com> # add table shopping cart
	add_installment_refund_in_service 2015-09-24T17:48:28Z Jiin.Wang <#@.com> # add xxx in api_service
	alter_table_add_wiki_in_service 2015-09-24T17:59:07Z Jin.Wang <#@.com> # add wiki in service
	add_uniq_index_for_api_shopcart 2015-09-24T15:45:35Z Pengfei.X <#@#.com> # uniq index on person_id, service_id and service_item_key
	...
	...
	@release-5.9.1-201603141632 2016-03-14T08:32:58Z Pengfei.X <pengphy#.com> # add tag for release-5.9.1-201603141632
	api_2016_03_16_alter_table_order_add_pay_time 2016-03-16T10:31:38Z Jin.Wang <waiabin@wsuo.com> # add pay_time in api_order
	api_2016_03_09_create_table_campaign_coupon_and_share 2016-03-09T06:32:27Z Chen <zxx@#.com> # api_2016_03_09_create_table_campaign_coupon_and_share


## 如何写Sqitch Plan

到Gaia目录`sqls`下，执行命令`sqitch add your-plan-name -n "description for this schema change"`。如下所示，

	Created deploy/your-plan-name.sql
	Created revert/your-plan-name.sql
	Created verify/your-plan-name.sql
	Added "your-plan-name" to sqitch.plan


执行完命令后，Sqtich会在`sqitch.plan`中增加一行变更信息，并且在目录`deploy`,`verfiy`,`revert`中增加相应的
sql脚本。

`deploy/your-plan-name.sql`中，编写数据库变更的具体内容，比如增加表的操作，增加字段的操作等等。

	-- Deploy gaia:create_table_api_doctorwhitelist to mysql
	-- xx名单
	BEGIN;
	CREATE TABLE `api_doctorwhitelist` (
	  `id` int(11) NOT NULL AUTO_INCREMENT,
	  `doctor_id`  VARCHAR(100) NOT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;
	ALTER TABLE `api_doctorwhitelist` ADD CONSTRAINT `api_doctorwhitelist_doctor_id_ref_api_doctor` FOREIGN KEY (`doctor_id`) REFERENCES `api_doctor` (`id`) ON DELETE CASCADE;
	COMMIT;

`verify/your-plan-name.sql`中编写，验证上述变更的脚本。

	-- Verify gaia:create_table_api_doctorwhitelist on mysql
	BEGIN;
	SELECT `id`, `doctor_id` FROM `api_doctorwhitelist` LIMIT 1;
	ROLLBACK;

`revert/your-plan-name.sql`中，增加撤销操作，比如删掉刚才创建的表，删除新增的字段等。

	-- Revert gaia:create_table_api_doctorwhitelist from mysql
	BEGIN;
	DROP TABLE `api_doctorwhitelist`;
	COMMIT;


## 如何变更，验证以及回滚数据库变更

到Gaia的sqls目录下，运行`sqitch deploy -t target_db`，Sqtich会对比自己维护的变更记录与sqitch.plan文件中的变更，然后判断出，
自上次成功变更后，新假如的变更有哪些，然后依次运行新的变更脚本，同时运行验证脚本，确保变更成功；此期间，如果应用某个
变更失败，或者验证的时候失败，Sqitch会自动运行revert中相关的回滚脚本。

如果是需要回滚到特定版本，那么可以使用`sqitch revert --to-change target-change-name target-db`。`revert`命令会找到对应的回滚
脚本，并且依次应用。如果是想重新应用某个变更到最新的变更，那么可以使用`rebase`命令，这个命令会先应用`revert`然后，在运行
`deploy`命令。


## 开发环境需要注意事项

我们在使用`Sqitch deploy`命令过程中，经常会遇到`can find xxxx`，找不到某个变更的情况，这个是因为，我们改变了Sqitch plan中变
跟的次序，导致，Sqitch在确认数据库目前所处的状态时，发生错误。产生这个问题的最大可能是我们经常需要在多个分支做开发，避
免不了在某个分支A上加了新的变更Ca，在另外一个分支B上加了变更Cb，这个时候如果合并分支，那么sqitch.plan，文件就会产生冲突，
解决冲突的原则是，让sqitch.plan中已经应用在生产环境的变更的次序保持不变。
在测试，或者开发环境中，如果碰到这个问题，我们可以使用`rebase`命令，让数据回滚到最近一个没有冲突的变更点，然后重新应用
变更脚本，让数据库更新到最新的状态。需要指出的是，我们应当在自己的开发机上测试自己的Sqitch脚本，包括`deploy, verify, revert`。
特别是，注意自己revert脚本的正确性，如果写的不当，那么可能导致整个表，或者数据库被删，切记。

*修改完sqitch.plan之后，同步一下本地的master地址，diff master分支sqitch.plan，确认修改的内容不会影响已经部署的plan的次序*


## 上线过程中注意事项

上线的时候，我们必须要先运行`sqitch status`这个命令，检查一下数据库的状态。在执行`deploy`命令时，需要使用`screen`或者`tmux`等
工具，避免出现由于ssh链接意外中断，而导致数据库变更值进行了一半的情况。
上线完成之后，我们需要给Sqtich plan加一个tag，tag的命令使用如下所示(教程)：

	> sqitch tag v1.0.0-dev1 -n 'Tag v1.0.0-dev1.'Tagged "change_pass" with @v1.0.0-dev1
	> git commit -am 'Tag the database with v1.0.0-dev1.'
	[master 0595297] Tag the database with v1.0.0-dev1.
	 1 file changed, 1 insertion(+)
	> git tag v1.0.0-dev1 -am 'Tag v1.0.0-dev1'

打上tag之后，会在Sqitch plan中加一行，tag标记，上线完成后，我们把master分支合并回其它的分支，其它分支在开发过程中，如果
遇到冲突的情况，就知道，把新增的变更加到最近一个tag的后面(之前，我们遇到的情况是，不知道哪些变更已经应用到线上了，所以，
需要查看master分支上的文件，才能得知)，然后调整相关变更的位置。


参考

1. http://sqitch.org/
2. https://metacpan.org/pod/sqitchtutorial-mysql
3. http://www.pgcon.org/2013/schedule/events/615.en.html
