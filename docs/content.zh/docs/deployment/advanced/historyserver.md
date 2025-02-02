---
title: "History Server"
weight: 3
type: docs
aliases:
  - /zh/deployment/advanced/historyserver.html
  - /zh/monitoring/historyserver.html
---
<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

# History Server

Flink 提供了 history server，可以在相应的 Flink 集群关闭之后查询已完成作业的统计信息。

此外，它暴露了一套 REST API，该 API 接受 HTTP 请求并返回 JSON 格式的数据。



<a name="overview"></a>

## 概览

HistoryServer 允许查询 JobManager 存档的已完成作业的状态和统计信息。

在配置 HistoryServer *和* JobManager 之后，你可以使用相应的脚本来启动和停止 HistoryServer：

```shell
# 启动或者停止 HistoryServer
bin/historyserver.sh (start|start-foreground|stop)
```

默认情况下，此服务器绑定到 `localhost` 的 `8082` 端口。

目前，只能将 HistoryServer 作为独立的进程运行。

<a name="configuration"></a>

## 配置参数

配置项 `jobmanager.archive.fs.dir` 和 `historyserver.archive.fs.refresh-interval` 需要根据 `作业存档目录` 和 `刷新作业存档目录的时间间隔` 进行调整。

**JobManager**

已完成作业的存档在 JobManager 上进行，将已存档的作业信息上传到文件系统目录中。你可以在 [Flink 配置文件]({{< ref "docs/deployment/config#flink-configuration-file" >}})中通过 `jobmanager.archive.fs.dir` 设置一个目录存档已完成的作业。

```yaml
# 上传已完成作业信息的目录
jobmanager.archive.fs.dir: hdfs:///completed-jobs
```

**HistoryServer**

可以通过 `historyserver.archive.fs.dir` 设置 HistoryServer 监视以逗号分隔的目录列表。定期轮询已配置的目录以查找新的存档；轮询间隔可以通过 `historyserver.archive.fs.refresh-interval` 来配置。

```yaml
# 监视以下目录中已完成的作业
historyserver.archive.fs.dir: hdfs:///completed-jobs

# 每 10 秒刷新一次
historyserver.archive.fs.refresh-interval: 10000
```

所包含的存档被下载缓存在本地文件系统中。本地目录通过 `historyserver.web.tmpdir` 配置。

请查看配置页面以获取[配置选项的完整列表]({{< ref "docs/deployment/config" >}}#history-server)。

## 日志集成

Flink 本身并不提供已完成作业的日志收集功能。
但是，如果你已经有了日志收集与浏览服务，可以配置 HistoryServer 与其集成。
（通过[`historyserver.log.jobmanager.url-pattern`]({{< ref "docs/deployment/config" >}}#historyserver-log-jobmanager-url-pattern)
和 [`historyserver.log.taskmanager.url-pattern`]({{< ref "docs/deployment/config" >}}#historyserver-log-taskmanager-url-pattern)）。
如此一来，你可以从 HistoryServer 的 WebUI 直接链接到相关 JobManager / TaskManager 的日志。

```yaml
# HistoryServer 会将 <jobid> 替换为对应作业的 ID
historyserver.log.jobmanager.url-pattern: http://my.log-browsing.url/<jobid>

# HistoryServer 会将 <jobid> 和 <tmid> 替换为对应作业和 TaskManager 的 ID
historyserver.log.taskmanager.url-pattern: http://my.log-browsing.url/<jobid>/<tmid>
```

<a name="available-requests"></a>

## 可用的请求

以下是可用且带有示例 JSON 响应的请求列表。所有请求格式样例均为 `http://hostname:8082/jobs`，下面我们仅列出了 URLs 的 *path* 部分。
尖括号中的值为变量，例如作业 `7684be6004e4e955c2a558a9bc463f65` 的 
`http://hostname:port/jobs/<jobid>/exceptions` 请求须写为 `http://hostname:port/jobs/7684be6004e4e955c2a558a9bc463f65/exceptions`。

  - `/config`
  - `/jobs/overview`
  - `/jobs/<jobid>`
  - `/jobs/<jobid>/vertices`
  - `/jobs/<jobid>/config`
  - `/jobs/<jobid>/exceptions`
  - `/jobs/<jobid>/accumulators`
  - `/jobs/<jobid>/vertices/<vertexid>`
  - `/jobs/<jobid>/vertices/<vertexid>/subtasktimes`
  - `/jobs/<jobid>/vertices/<vertexid>/taskmanagers`
  - `/jobs/<jobid>/vertices/<vertexid>/accumulators`
  - `/jobs/<jobid>/vertices/<vertexid>/subtasks/accumulators`
  - `/jobs/<jobid>/vertices/<vertexid>/subtasks/<subtasknum>`
  - `/jobs/<jobid>/vertices/<vertexid>/subtasks/<subtasknum>/attempts/<attempt>`
  - `/jobs/<jobid>/vertices/<vertexid>/subtasks/<subtasknum>/attempts/<attempt>/accumulators`
  - `/jobs/<jobid>/plan`
  - `/jobs/<jobid>/jobmanager/config`
  - `/jobs/<jobid>/jobmanager/environment`
  - `/jobs/<jobid>/jobmanager/log-url`
  - `/jobs/<jobid>/taskmanagers/<taskmanagerid>/log-url`

{{< top >}}
