# swoole-co-pool

[![Latest Version](https://img.shields.io/packagist/v/yurunsoft/swoole-co-pool.svg)](https://packagist.org/packages/yurunsoft/swoole-co-pool)
[![License](https://img.shields.io/github/license/Yurunsoft/swoole-co-pool.svg)](https://github.com/Yurunsoft/swoole-co-pool/blob/master/LICENSE)

## 介绍

本项目为 Swoole 协程工作池，它封装了一些实用的 Swoole 协程操作，使用起来十分一把梭。

* 协程工作池

用于需要大量协程任务的场景，它可以限定你的同时工作协程数量，并且减少协程频繁创建销毁的损耗。

* 协程批量执行器

用于同时执行多个协程，并且能够获取到他们所有的返回值。

宇润PHP全家桶群：17916227 [![点击加群](https://pub.idqqimg.com/wpa/images/group.png "点击加群")](https://jq.qq.com/?_wv=1027&k=5wXf4Zq)，如有问题会有人解答和修复。

程序员日常划水群：74401592 [![点击加群](https://pub.idqqimg.com/wpa/images/group.png "点击加群")](https://shang.qq.com/wpa/qunwpa?idkey=e2e6b49e9a648aae5285b3aba155d59107bb66fde02e229e078bd7359cac8ac3)。

## 安装

在您的composer.json中加入配置：

```json
{
    "require": {
        "yurunsoft/swoole-co-pool": "^1.3.0"
    }
}
```

然后执行`composer update`命令。

## 使用

### 协程工作池

```php
use Yurun\Swoole\CoPool\CoPool;
use Yurun\Swoole\CoPool\Interfaces\ICoTask;
use Yurun\Swoole\CoPool\Interfaces\ITaskParam;

$coCount = 10; // 同时工作协程数
$queueLength = 1024; // 队列长度
$pool = new CoPool($coCount, $queueLength,
    // 定义任务匿名类，当然你也可以定义成普通类，传入完整类名
    new class implements ICoTask
    {
        /**
         * 执行任务
         *
         * @param ITaskParam $param
         * @return mixed
         */
        public function run(ITaskParam $param)
        {
            // 执行任务
            return true; // 返回任务执行结果，非必须
        }

    }
);
$pool->run();

$data = 1; // 可以传递任何参数

// 增加任务，并挂起协程等待返回任务执行结果
$result = $pool->addTask($data);

// 增加任务，异步回调
$result = $pool->addTaskAsync($data, function(ITaskParam $param, $data){
    // 异步回调
});
```

### 批量执行协程

每个方法都在单独的协程中被执行，然后可以统一获取到结果。

```php
$batch = new CoBatch([
    function(){
        return 'imi';
    },
    'a' =>  function(){
        return 'niu';
    },
    'b' =>  function(){
        return 'bi';
    },
]);
$results = $batch->exec();
// $timeout = -1; // 支持超时
// $limit = -1; // 限制同时工作协程数量
// $results = $batch->exec($timeout, $limit);
var_dump($results);
// $results 值为:
// [
//     'imi',
//     'a' =>  'niu',
//     'b' =>  'bi',
// ]
```

快捷函数：

```php
use function Yurun\Swoole\Coroutine\batch;
batch([
    function(){
        return 'imi';
    },
    'a' =>  function(){
        return 'niu';
    },
    'b' =>  function(){
        return 'bi';
    },
]);
// batch($callables, $timeout, $limit);
```

### 执行单个协程并等待返回值

```php
use function Yurun\Swoole\Coroutine\goWait;
$result = goWait(function(){
    \Swoole\Coroutine::sleep(1);
    return 'wait result';
});
echo $result; // wait result
```

### 通道容器

```php
use Yurun\Swoole\CoPool\ChannelContainer;

go(function(){
    $channelContainer = new ChannelContainer;

    $id = 'abc';
    $data = [
        'time'  =>  time(),
    ];

    go(function() use($id, $data, $channelContainer){
        echo 'Wait 3 seconds...', PHP_EOL;
        \Swoole\Coroutine::sleep(3);
        $channelContainer->push($id, $data);
    });
    var_dump($channelContainer->pop($id));

});
```

## 代码示例

详见 `example` 目录

## 捐赠

<img src="https://raw.githubusercontent.com/Yurunsoft/swoole-co-pool/master/res/pay.png"/>

开源不求盈利，多少都是心意，生活不易，随缘随缘……
