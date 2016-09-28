---
layout: post
title: 百度知道的爬虫-PHP版
categories: PHP
description: 百度知道的爬虫-PHP版
keywords: PHP,Spiders
---

PHP编码

```php
<?php
class spider
{
    private $content ;
    private $contentlen ;
    private $BestAnswer ;
    private $CurPosition ;
    function GetStart( $iStart )
    {
        return strpos( $this->content , '>' , $iStart )+1 ;
    }
    function GetContent ( $url )
    {
        $this->content = file_get_contents($url);
        $this->contentlen = strlen( $this->content ) ;
        $start = strpos( $this->content , '<title>') ;
        $start = $this->GetStart( $start ) ;
        $end = strpos( $this->content , '</title>' , $start ) ;
        $title = substr( $this->content , $start , $end-$start ) ;
        if ( strpos( $title , iconv("utf-8","gb2312//IGNORE","_百度知道") , 1 ) < 1 )
        {
            return false;
        }
        return true ;
    }
    function GetTitle()
    {
        $start = strpos( $this->content , '<title>') ;
        if ( $start > 0 )
        {
            $start = $this->GetStart( $start ) ;
            $end = strpos( $this->content , '</title>' , $start ) ;
            $this->CurPosition = $end ;
            return substr( $this->content , $start , $end-$start ) ;
        }
        return NULL ;
    }
    function GetQTitle()
    {
        $start = strpos( $this->content , 'span class="question-title"' , $this->CurPosition ) ;
        if ( $start > 0 )
        {
            $start = $this->GetStart( $start ) ;
            $end = strpos( $this->content , '</span>' , $start ) ;
            $this->CurPosition = $end ;
            return substr( $this->content , $start , $end-$start ) ;
        }
        return NULL ;
    }
    function GetClassFly()
    {
        ;
    }
    function GetQContent()
    {
        $start = strpos( $this->content , 'pre id="question-content"' , $this->CurPosition ) ;
        if ( $start > 0 )
        {
            $start = $this->GetStart( $start ) ;
            $end = strpos( $this->content , '</pre>' , $start ) ;
            $this->CurPosition = $end ;
            return substr( $this->content , $start , $end-$start ) ;
        }
        return NULL ;
    }
    function GetQsuply()
    {
        $start = strpos( $this->content , 'id="question-suply"' , $this->CurPosition ) ;
        if ( $start > 0 )
        {
            $start = $this->GetStart( $start ) ;
            $end = strpos( $this->content , '</pre>' , $start ) ;
            $this->CurPosition = $end ;
            return substr( $this->content , $start , $end-$start ) ;
        }
        return NULL ;
    }
    function GetAnswer()
    {
        $start = strpos( $this->content , 'class="reply-text mb10"' , $this->CurPosition ) ;
        if ( $start > 0 )
        {
            $start = $this->GetStart( $start ) ;
            $end = strpos( $this->content , '</pre>' , $start ) ;
            $this->CurPosition = $end ;
            return substr( $this->content , $start , $end-$start ) ;
        }
        return NULL ;
    }
}
ini_set('max_execution_time', '0');
$TestSpider = new spider() ;
$Startqid = 1000001 ;
$sndqid = 1000051 ;
$standurl = 'http://zhidao.baidu.com/question/' ;
$html = '.html' ;
$url ;
$NoUse = 0 ;
function microtime_float()
{
    list($usec, $sec) = explode(" ", microtime());
    return ((float)$usec + (float)$sec);
}
$time_start = microtime_float();
$answer ;
for ($i = $Startqid ; $i < $sndqid ; $i++ )
{
    $url = $standurl.$i.$html ;
    if ( $TestSpider->GetContent ( $url ) )
    {
        echo '<br>正在爬取编号为'.$i.'的网页<br>' ;
        $TestSpider->GetTitle() ; //得到网页标题,不用显示了
        echo '<font color="green">问题：</font><font color="red"><a target="_blank" href="'.$url.'"> '.$TestSpider->GetQTitle().'</a></font><br>' ; //得到问题题目
        echo '<font color="green">问题具体内容：</font>'.$TestSpider->GetQContent().'</font><br>' ; //得到问题内容，有可能不存在
        echo '<font color="green">问题补充说明：</font>'.$TestSpider->GetQsuply().'</font><br>' ; //问题补充说明，有可能不存在
        while ( ($answer = $TestSpider->GetAnswer()) != NULL )
        {
            echo '<font color="green">问题答案：</font>'.$answer.'</font><br>' ; //得到答案。有可能没有答案！
        }
        ob_flush() ;
        flush() ;
    }
    else
    {
        echo '<p>错误了<a target="_blank" href="'.$url.'" style= "color:#ff0000">'.$url.'</a></p>' ;
        $NoUse++ ;
    }
}
$time_end = microtime_float();
$time = $time_end - $time_start;
$i = $i-$Startqid ;
echo '<p>爬取'.$i.'个网页用时'.$time.'秒</p>其中跳过'.$NoUse.'个无效网页！' ;
```
