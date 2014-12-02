---
layout: post
title: "performance tuning chapter two"
description: "Performance tuning -- 小记"
category: performance tuning, tuning tools
tags: [clustering log collector, logstash]
---
{% include JB/setup %}

半年时间又过去了， 每当回顾下有什么可以记录下来的时候，又是只有performance tuning，于是上篇blog的姊妹篇诞生了，继续写写关于tuning tools的新发现。

### grep "error" in clustering logs
说到这个话题，不得不提的是log的重要性，如何写出好的log，这又是另一个大的话题，这篇blog不讨论了。在performance 测试的过程中，查找error log的工作是必不可少的，通常的做法是不断的grep加上|>，我们大部分时候也是这么做，但总觉得不爽，因为我们通常有多个个traffic节点，而每个节点上去grep很麻烦，也很费时（当然前提是我们没有把traffic的日志收集在一起）。而前段时间看到这个这个--Logstash，一个简单的，基于日志行记录的，实时日志收集管理工具，这里我特意加上了“简单”和“实时”，这是我希望用以区别其他类似的日志收集管理工具。字面上很容易理解，用它就是做日志收集，确实我们就是这么用的，但它真正的魅力则是：它本身是一个很简单的框架工具，而我们可以借助它搭建很多实用的工具出来。对于logstash的介绍和使用，这里也不会描述，可以直接去官网了解，或是在slideshare上搜下。这里我先列一个对我们很实用的使用方式--收集error日志：用法很简单，每个traffic节点上先按关键字把日志过滤出来，再传送到统一的地方汇总，这样我们只用坐在电脑前喝杯咖啡，tail这个日志文件，就可以看到每个traffic节点上我们关心的那些日志了。因为它是实时的，我们会在第一时间得到需要的信息，另外，每个traffic所产生的日志，我们都会加上机器的hostname/ip在原始日志前面加以区分不同机器产生的日志。而做完这个收集工具，大概只需要1-2个番茄钟的时间，并且测试的同事可以自己完成，不需要写代码，只用配置，和平时的grep近乎类似，我想，这才是这个工具真正迷人的地方。

### ./logstash > nagios
其实，这一段只是搭车附送而已，精华部分上面已经说完了。这个工具是基于上面的错误日志收集工具，再次使用logstash让我们的稳定性测试更自动化，那就得用到nagios这个监控工具了，而logstash也支持nagios。需求大致如下：因为稳定性测试一般都要跑很长一段时间，而且是白天黑夜不停的跑，那么我们希望如果出现了问题，可以第一时间通知到测试人员，那么可以基于之前的搜集工具，加上logstash的metric插件，实时的对输出日志统计：如1(10,15)分钟出了多少条记录，再加上nagios output，如果1分钟出现的错误日志达到30条，那么就让nagios告警吧，测试人员收到告警后，就可以快速保留问题现场，而不至于数据丢失。