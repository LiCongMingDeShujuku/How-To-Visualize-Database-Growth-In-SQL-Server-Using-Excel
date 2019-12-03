![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 如何在SQL Server使用Excel可视化数据库增长
#### How To Visualize Database Growth In SQL Server Using Excel
**发布-日期: 2015年7月30日 (评论)**

![#](images/01-How-To-Visualize-Database-Growth-In-SQL-Server-Using-Excel.png?raw=true "#")

## Contents

- [中文](#中文)
- [English](#English)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
如何使用Excel可视化数据库增长图表。这是一个直接的“How To”帮助你入门。你会喜欢这个。我保证很容易。请注意以下图表。2行RED和BLUE。RED表示当前的压缩率，并且由于尚未为数据库配置压缩，因此它们表示与BLUE中通常的备份大小相同的级别。如你看到的; 它们通过数据库的历史并行，直到我启用了压缩的2015年7月。


## English
How to visualize a database growth chart using Excel. Here’s a straight up ‘How To’ write up to get you started. You’ll love this. It’s easy I promise. Notice the following graph. 2 lines RED and BLUE. RED represents the current compression ratio, and because compression hasn’t been configured for the database they represent the same level as the usual backup size which is in BLUE. As you can see; they are parallel through the history of the database until it gets to July 2015 where I enabled compression.


![#](images/01-How-To-Visualize-Database-Growth-In-SQL-Server-Using-Excel.png?raw=true "#")
 
Below is some SQL logic from a former post about trending your database growth using backups (https://mikesdatawork.wordpress.com/2015/07/27/get-database-size-trending-on-all-databases/) 

下面是之前的帖子中一篇关于使用备份来趋势化数据库增长的SQL逻辑
(https://mikesdatawork.wordpress.com/2015/07/27/get-database-size-trending-on-all-databases/) 

This was inspired by the original author Erin Stellato at SQL Skills (http://www.sqlskills.com/blogs/erin/trending-database-growth-from-backups/). Check that out for more info :)

这是受到SQL Skills原作者Erin Stellato的启发（http://www.sqlskills.com/blogs/erin/trending-database-growth-from-backups/）。 查询以了解更多信息:)

By the way; you can enable backup compression with the following statement.

另外; 你可以使用以下语句启用备份压缩。


---
## Logic
```SQL
exec master..sp_configure 'show advanced options' 		1 reconfigure;
exec master..sp_configure 'backup compression default',	1 reconfigure;
go

```

I adjusted the SQL logic so it could be run against all databases on a server ( or group of servers if registered under a single folder in SSMS ). I’ve also included some modifications from ‘Jeffs’ post within the original article. Basically I added an extra 365 days to it. From there you can get a pretty good idea how to modify the logic to your liking and reach back as far as needed without too much hassle.

我调整了SQL逻辑，因此它可以针对服务器上的所有数据库运行（如果在SSMS中的单个文件夹下注册，则可以运行服务器组）。我还在原始文章中的'Jeffs'帖子中加入了一些修改，基本上我额外增加了365天，你可以很好地了解如何根据自己的喜好修改逻辑，并在没有太多麻烦的情况下尽可能地返回。

---
## Logic
```SQL
use master;
set nocount on
declare @get_dbsize_history 	varchar(max)
declare @first_day_of_year 		varchar(25)
set 	@first_day_of_year 		= (select dateadd(year, datediff(year, 0, getdate()), 0)) -365 --> Go back to the first of the year plus 1 extra year (365 days)
set 	@get_dbsize_history 	= ''
select 	@get_dbsize_history 	= @get_dbsize_history +
'
select
''database'' = cast(upper([database_name]) as varchar(20))
, ''month'' = convert(varchar(7),[backup_start_date],120)
, ''backup size gb'' = str(avg([backup_size]/1024/1024/1024),5,2)
, ''compressed bu size'' = str(avg([compressed_backup_size]/1024/1024/1024),5,2)
, ''compression ratio'' = str(avg([backup_size]/[compressed_backup_size]),5,2)
from
	msdb.dbo.backupset
where
	[database_name] = ''' + upper(name) + '''
	and [type] = ''d''
	and backup_finish_date > ''' + @first_day_of_year + '''
	and database_name in (select name from master.sys.databases)
group by
	[database_name]
	, convert(varchar(7),[backup_start_date],120)
order by
	[database_name],convert(varchar(7),[backup_start_date],120);' + char(10) + char(10)
from
	sys.databases
where
	database_id > 4

exec (@get_dbsize_history)
```

So here’s what you do. You find a server that has a variety of backup sizes through the history you’re looking at. These make the best examples to illustrate growth history. Copy the results from SSMS (using a single database history) and paste them straight into Excel. Nothing crazy, or difficult. In this example we are using a server called “MyServerName”, and a single database called “MyDatabase” which is basically about 1.5 years history. It might help to insert an extra row and add the Column titles.

这就是你可以做的事情。你可以在所查看的历史记录中找到具有各种备份大小的服务器。 这些是举例说明增长历史的最佳例子。从SSMS复制结果（使用单个数据库历史记录）并将其直接粘贴到Excel中。没有什么让人抓狂或困难的。在此示例中，我们使用名为“MyServerName”的服务器和名为“MyDatabase”的单个数据库，该数据库基本上是大约1年半的历史数据。插入额外的行并添加列标题可能会有所帮助。

![#](images/02-How-To-Visualize-Database-Growth-In-SQL-Server-Using-Excel.png?raw=true "#")
 
Perform the following actions.
1. Highlight 3 columns Month, Backup Size, Compressed
2. Click ‘Insert’.
3. Click ‘Line’ chart.
4. Select ‘3-D Line’ chart.

执行以下操作。
1. 突出显示3列月份，备份大小，压缩
2. 单击“Insert’”。
3. 单击“Line’”图表。
4. 选择“3-D Line”图表。

![#](images/03-How-To-Visualize-Database-Growth-In-SQL-Server-Using-Excel.png?raw=true "#") 
Change the charting.
1. Click ‘Chart Tools’, and select the ‘Design’ tab.
2. Select ‘Style 42’ below.
Feel free to drag the chart out a bit to make it more visible.

更改图表。
1. 单击“‘Chart Tools”，然后选择“Design”选项卡。
2. 选择下面的“Style 42”。
随意拖动图表以使其更加明显。

![#](images/04-How-To-Visualize-Database-Growth-In-SQL-Server-Using-Excel.png?raw=true "#")
 
Next we want to change the chart layout slightly. Perform the following actions.
1. Select layout ‘6’.
2. Double click on the inner corner of the chart to get the format properties.
3. Select ‘3-D Rotation’ on the left, and change the [X Rotation] to 30.
4. Click ‘Close’.

接下来我们要稍微改变图表布局。 执行以下操作。
1. 选择布局'6'。
2. 双击图表的内角以获取格式属性。
3. 选择左侧的“3-D Rotation”，然后将[X Rotation]更改为30。
4. 单击“Close’”。

![#](images/05-How-To-Visualize-Database-Growth-In-SQL-Server-Using-Excel.png?raw=true "#")
 
Looking at the chart it seems like we are almost done. Lets punch up those lines a bit and add some width. To do so; perform the following actions.
1. Double click the line to bring up the Data Point properties.
2. Click on ‘Series Options’ on the left, and drag the handle all the way to the left towards ‘No Gap’.

看图表似乎我们差不多完成了。让我们稍微打一下这些线并增加一些宽度。这样做; 执行以下操作。
1. 双击该行以显示数据点属性。
2. 单击左侧的“Series Options”，然后将手柄一直向左拖动到“No Gap”。

![#](images/06-How-To-Visualize-Database-Growth-In-SQL-Server-Using-Excel.png?raw=true "#")
 
Now just change the titles and you’re done. Showing your capacity and storage requirements will be much easier now not just with backups, but with any standing database growth or otherwise. This can also be used to track performance over time from sys.dm_os_performance_stats etc.

现在只需更改标题了，然后就完成了。显示容量和存储要求现在将变得更加容易，不仅仅是备份，还有任何常数据库增长或其他方面。这也可用于跟踪sys.dm_os_performance_stats等随时间推移的性能。

![#](images/07-How-To-Visualize-Database-Growth-In-SQL-Server-Using-Excel.png?raw=true "#")
 


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

