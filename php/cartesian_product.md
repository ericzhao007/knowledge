# 笛卡尔积算法
> 最近接了个私活，要搞sku，自然免不了直面笛卡尔积，搞起

### 前言
虽然对于cv程序员来说，这都不是事。
但是场景都不同，难免面临着结构的不同，这个时候如果再搞错了只会越来越乱，本着掌握着原则我们尝试使用自己的方式实现。
ps：百度上的虽然可行，但是一脸懵逼。

### 实现思路
假设一个如下的二维数组
```php
<?php
$arr = [
    ['西瓜', '苹果', '香蕉', '梨'],
    ['水果刀', '菜刀', '壁纸刀'],
    ['甜', '酸', '辣', '臭', '香']
];
```
我们知道笛卡尔积的结果数量一定是a*b*c，也就是 4*3*5，所以第一步我们的想法是只要有这么多次循环保证，我们只需要保证循环内的元素组合就行，所以我们代码设计如下
```php
<?php
foreach($arr as $key => $item){
    foreach($item as $k => $v){
        // ...do something
    }
}
```
下面我们来编写内部逻辑，思考一下如何可以实现两两组合并且不重样呢。

我们来参考一个现实的例子，比如数数，我们假设三组数据对应着一个三位数，如从000,只要我们顺着往上数000、001、002、003、004...，那么最终的每个数字都是不同的，但是对于某一位数字来说，到了最大值，也就是9就意味当前的数字需要重置，下一位数字需要进1。以这样的逻辑带入到我们的例子，每一位的最大值组合起来就是 4 - 3 - 5。
那么我们只要维持数数字的逻辑，当某一位达到它的最大值就置0然后下一位加1，直到所有位数都满了为止。
```php
<?php
// 记录下每组的数量，最终total_counters 就是我们的 4 - 3 - 5
// 同时需要一个记录进度的数组，我们的起始值为 0 - 0 - 0
$total_counters = [];
$current_counters = [];
foreach($arr as $key => $item){
    $total_counters[$key] = count($item) - 1; // 减1因为我们是从0开始数的
    $current_counters[$key] = 0;
}
```
这样我们的准备工作就完成了，接下来继续实现数数字的逻辑，还记得前面的两层循环吗，这里我们拿过来
```php
<?php
$lines = [];
foreach($arr as $key => $item){
    foreach($item as $k => $v){
        $keys = [];
        // 开始数数        
        foreach($current_counters as $ck => $cv){ // 每次循环代表着一位数
            $total_count = $total_counters[$ck]; //  当前位数的最大值
            if($cv < $total_count){ // 如果还没数到顶
                // 这里我们是从左边开始数的，相对于数数字的逻辑是反过来的，比如：正常来说是008,009,010,而我们倒过来就是 800, 900, 010
                // 就是说数数是从右边到最大值置0进1，而我们的场景是从左边到最大值置0进1
                $current_counters[$ck] += 1; // 加一位
                // 所以这里小于当前位的是左，大于当前位的是右
                for($n = 0;$n < $ck;$n++){
                    $current_counters[$n] = 0;
                }
                // 最终得到了我们这次循环的位数
                $keys = $current_counters;
                break;
            }
        }        
        // 将$keys 转换成对应的值 元素key是位数，元素值是对应位数的第几个值
        $line = [];
        foreach($keys as $kk => $kv){
            $line[$kk] = $arr[$kk][$kv];
        }
        $lines[] = $line;
    }
}
```
现在我们运行一下看一下效果
```json
[
    [
        "苹果",
        "水果刀",
        "甜"
    ],
    [
        "香蕉",
        "水果刀",
        "甜"
    ],
    [
        "梨",
        "水果刀",
        "甜"
    ],
    [
        "西瓜",
        "菜刀",
        "甜"
    ],
    [
        "苹果",
        "菜刀",
        "甜"
    ],
    [
        "香蕉",
        "菜刀",
        "甜"
    ],
    [
        "梨",
        "菜刀",
        "甜"
    ],
    [
        "西瓜",
        "壁纸刀",
        "甜"
    ],
    [
        "苹果",
        "壁纸刀",
        "甜"
    ],
    [
        "香蕉",
        "壁纸刀",
        "甜"
    ],
    [
        "梨",
        "壁纸刀",
        "甜"
    ],
    [
        "西瓜",
        "水果刀",
        "酸"
    ]
]
```

好像不太对，我们数了一下才12个。不应该是 60 = 4 * 3 * 5 吗，看来问题出在外循环上，那我们把它去掉吧，因为只要到达最大数了就知道该停了。我们改造如下：
```php
<?php
while(true){
    $keys = [];        
    foreach($current_counters as $ck => $cv){
        $total_count = $total_counters[$ck];
        // 试想一下达到最大值得时候这里会发生什么，首先就是循环完了if都没进，那么自然$keys是空得，这就是我们要找到结束条件了
        if($cv < $total_count){
            $current_counters[$ck] += 1;
            for($n = 0;$n < $ck;$n++){
                $current_counters[$n] = 0;
            }
            $keys = $current_counters;
            break;
        }
    }
    if(empty($keys)){   // 缤购，结束
        break;
    }
    $line = [];
    foreach($keys as $kk => $kv){
        $line[$kk] = $arr[$kk][$kv];
    }
    $lines[] = $line;
}
```
那我们再看一下结果：
```json
[
    [
        "苹果",
        "水果刀",
        "甜"
    ],
    [
        "香蕉",
        "水果刀",
        "甜"
    ],
    [
        "梨",
        "水果刀",
        "甜"
    ],
    // ... 中间省略了哈
    [
        "苹果",
        "壁纸刀",
        "香"
    ],
    [
        "香蕉",
        "壁纸刀",
        "香"
    ],
    [
        "梨",
        "壁纸刀",
        "香"
    ]
]
```
这次元素得个数是59，好像还差了一个，我们看得到得结果，发现第一条的苹果不太对，至少应该是从西瓜开始也就是第0位开始。我们分析代码发现
```php
<?php

// do something
        if($cv < $total_count){
            $current_counters[$ck] += 1;    // 这里第一次就是从 1 - 0 -0 开始的，漏掉了 0 - 0 - 0这个初始值
            // ...
            $keys = $current_counters;
            break;
        }
```
所以我们需要在初始化的时候存一下
```php
<?php
$total_counters = [];
$current_counters = [];        
$line = [];
foreach($arrs as $key => $item){
    $total_counters[$key] = count($item) - 1;
    $current_counters[$key] = 0;
    $line[] = $item[0]; // 把每一行的第一个元素存一下就好了
}        
$lines = [$line];
```
最终终于达到了我们要的效果了，完整代码如下 
```php
<?php
$arr = [
    ['西瓜', '苹果', '香蕉', '梨'],
    ['水果刀', '菜刀', '壁纸刀'],
    ['甜', '酸', '辣', '臭', '香']
];
$total_counters = [];
$current_counters = [];
$line = [];
foreach($arr as $key => $item){
    $total_counters[$key] = count($item) - 1; // 减1因为我们是从0开始数的
    $current_counters[$key] = 0;
    $line[$key] = 0;
}
$lines = [$line];
while(true){
    $keys = [];
    // 开始数数        
    foreach($current_counters as $ck => $cv){ // 每次循环代表着一位数
        $total_count = $total_counters[$ck]; //  当前位数的最大值
        if($cv < $total_count){ // 如果还没数到顶
            // 这里我们是从左边开始数的，相对于数数字的逻辑是反过来的，比如：正常来说是008,009,010,而我们倒过来就是 800, 900, 010
            // 就是说数数是从右边到最大值置0进1，而我们的场景是从左边到最大值置0进1
            $current_counters[$ck] += 1; // 加一位
            // 所以这里小于当前位的是左，大于当前位的是右
            for($n = 0;$n < $ck;$n++){
                $current_counters[$n] = 0;
            }
            // 最终得到了我们这次循环的位数
            $keys = $current_counters;
            break;
        }
    }        
    if(empty($keys)){
        break;
    }
    // 将$keys 转换成对应的值 元素key是位数，元素值是对应位数的第几个值
    $line = [];
    foreach($keys as $kk => $kv){
        $line[$kk] = $arr[$kk][$kv];
    }
    $lines[] = $line;            
}
```

### 总结

授人以鱼不如授人以渔，本文从思路到实现进行了逐步分析，这样才能掌握领会