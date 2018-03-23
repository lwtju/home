---
layout: post
title: 用redis实现的一个简单任务处理代码
key: 20171016
tags: php redis queue
---

## 场景需求

业务上需要执行定时任务，起初是用redis的list实现的，但pop的问题是任务执行出错、进程crash、机器故障突发时错误处理不及时，无法任务回溯


## 解决过程

直接上代码吧

    use \Yaf\Registry;

	class JobService extends BaseService
	{
		private $ttl;
		private $listName;
		private $dbConfig;

		public function __construct($dbConfig = '', $listName = '', $ttl = 300)
		{
			if(empty($listName)) {
				$this->listName = Registry::get('config')->queName;
			}else{
				$this->listName = $listName;
			}

			$this->ttl = $ttl;
			$this->dbConfig = $dbConfig ? : 'defaultName';
		}

		public function addJob($jobName, $jobTime)
		{
			//getRedisIns由BaseService内实现
			return $this->getRedisIns($this->dbConfig)->zadd($this->listName, $jobTime, $jobName);
		}

		/*获取最新一条任务*/
		public function getJob()
		{
			$queName = $this->listName;
			$arrRet = $this->getRedisIns($this->dbConfig)->zrange($queName, 0, 0, "withscores");
			if (empty($arrRet)) {
				return false;
			}

			$arrTmp = array_keys($arrRet);
			$key = $arrTmp[0];
			//判断此任务执行时间已到
			if ($arrRet[$key] > time()) {
				return false;
			}
			//保证任务只被一个处理程序读取
			$flagKey = $key . ':flag:' . $this->listName;
			$flag = $this->getRedisIns($this->dbConfig)->incr($flagKey);
			if ((int)$flag != 1) {
				return false;
			}
			$this->getRedisIns($this->dbConfig)->expire($flagKey, (int)($this->ttl) / 2);

			//读取之后为此任务延迟ttl时间
			if (false !== $this->getRedisIns($this->dbConfig)->zadd($queName, time() + $this->ttl, $key)) {
				return $key;
			}

			return false;
		}

		public function delJob($jobName)
		{
			return $this->getRedisIns($this->dbConfig)->zrem($this->listName, $jobName);
		}
	}
    