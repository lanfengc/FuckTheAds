### 1、访问下面的链接，然后根据自己的操作系统下载对应版本的Edge浏览器
https://www.microsoft.com/zh-cn/edge
### 2、运行下载的安装程序，等待安装完成后，打开Edge浏览器
### 3、点击Edge右上角的三个点，打开菜单，选择“扩展”，打开扩展管理器
### 4、点击左侧的“获取Microsoft Edge扩展”，打开扩展商店
### 5、在左上角的输入框内输入“TamperMonkey”，然后回车，等待搜索结果出现
### 6、选择出现的第一个名字完全相符的插件（如下图示），点击右侧的“获取”按钮
![avatar][imageTamperMonkey]
### 7、在弹出对话框中点击“添加扩展”按钮，等待扩展安装成功
### 8、在地址栏右侧，点击“TamperMonkey”扩展图标，在弹出菜单中选择“添加新脚本”
![avatar][imageTamperMonkeyIcon]
### 9、删除弹出的“新建用户脚本”下方的编辑区中的所有内容，然后复制下面的代码，粘贴进入，保存即可。
### 10、打开爱奇艺、腾讯视频即可看到广告快速闪过，VIP自动解析了。

~~~ Javascript
// ==UserScript==
// @name         视频广告加速/VIP自动解析
// @namespace    http://www.mdmsoft.cn/
// @version      1.0
// @description  看个教程，腾讯 优酷视频广告贼烦人，竟然有120秒，不干掉它真是对不起自己程序猿的职业!
// @author       @MdmSoft
// @match        https://v.qq.com/*
// @match        https://v.youku.com/*
// @match        https://www.iqiyi.com/*
// @grant        none
// ==/UserScript==


//解析接口API地址
var jiexiApiArray = ["https://jx.du2.cc/?url=",
                     "https://jx.618g.com/?url=",
                     "https://www.8090g.cn/jiexi/?url=",
                     "https://8090.ylybz.cn/jiexi/?url=",
                     "https://jiexi.bm6ig.cn/?url="]
//随机一个解析接口
//var jiexiApiUrl = jiexiApiArray[Math.round(Math.random()*jiexiApiArray.length,0)];
//使用固定的解析接口（这个比较稳定）
var jiexiApiUrl = "https://z1.m1907.cn/?jx=";
//当前页面href
var nowLocationHref = location.href;
//注入的视频播放器对象
var playerInject;
//注入播放器
function injectPlayer(divId) {
    playerInject = document.getElementById('playerInject');
    //如果没有注入过
    if (!playerInject) {
        //取得原有的播放器框架
        var player = document.getElementById(divId);
        if (player) {
            //如果获取成功
            player.innerHTML = '';
            //设置透明度
            player.style.opacity = 0.00;
            //取得原播放器框架大小
            var rectPlayer = player.getBoundingClientRect();
            //创建DIV，注入并覆盖
            playerInject = document.createElement('div');
            document.body.appendChild(playerInject);
            playerInject.outerHTML = '<div id="playerInject" style="position:absolute;top:' + rectPlayer.top + 'px;left:' + rectPlayer.left + 'px;z-index:999;visibility:visible;width:' + rectPlayer.width + 'px;height:' + rectPlayer.height + 'px;"><iframe src="' + jiexiApiUrl + window.location.href + '" style="width:100%;height:100%;z-index:999;border-width:0px;overflow-x:hidden;overflow-y:hidden;"></iframe></div>';
        }
    }
}
//更改注入的播放器大小
function resizePlayer(divId) {
    //取得原播放器DIV
    var player = document.getElementById(divId);
    playerInject = document.getElementById('playerInject');
    if(player && playerInject){
        //设置注入的播放器大小
        var playRect = player.getBoundingClientRect();
        playerInject.style.left = playRect.x + 'px';
        playerInject.style.top = (playRect.y + document.documentElement.scrollTop) + 'px';
        playerInject.style.width = playRect.width + 'px';
        playerInject.style.height = playRect.height + 'px';
    }
}
//腾讯视频注入
function injectTencent(divClassName) {
    var qqTips = document.getElementsByClassName(divClassName);
    if(qqTips && qqTips.length>0){
         for (var i = 0; i < qqTips.length; i++) {
             if (qqTips[i].style.display == '' && (qqTips[i].innerText.indexOf('开通VIP会员') != -1 || qqTips[i].innerText.indexOf('开通会员') != -1)) {
                 injectPlayer('tenvideo_player');
                 qqTips[i].style.display = 'none';
                 break;
             };
         }
    }
}
setInterval(function() {
    //如果浏览器href变更，则刷新页面
    if (nowLocationHref != location.href) {
        nowLocationHref = location.href;
        console.log('href changed to ' + nowLocationHref);
        if (playerInject) {
            console.log('injected, reload!');
            location.reload();
        }
    }
    var i,adTimes;
    if (window.location.href.indexOf('www.iqiyi.com') != -1) {
        //爱奇艺广告加速通过
        var o = document.getElementsByTagName('video');
        for (i = 0; i < o.length; i++) {
            if (o[i].src != '') {
                //爱奇艺广告倒计时DIV classname
                adTimes = document.getElementsByClassName('cupid-public-time');
                if (adTimes && adTimes.length>0 && adTimes[0].style.display == "") {
                    o[i].playbackRate = 16.0;
                }
                else{
                    o[i].playbackRate = 1.0;
                }
            }
        }
        //拿到爱奇艺播放页右侧的VIP提示
        var vipSides = document.getElementsByClassName('qy-player-side-vip');
        //拿到爱奇艺播放页进度条上方的VIP提示
        var iqyTips = document.getElementsByClassName('iqp-tip-stream');
        //拿到爱奇艺VIP的黑色遮罩（VIP视频直接不让试看的）
        var vipPops = document.getElementsByClassName('qy-player-vippay-popup');
        if(vipSides && vipSides.length>0 || iqyTips && iqyTips.length>0 || vipPops && vipPops.length>0){
            //标志位，标志两个VIP状态
            var isVipSideShown=false, isVipTipsShown=false, isVipPopShow=false;
            for (i = 0; i < vipSides.length; i++) {
                if (vipSides[i].style.display == '' && vipSides[i].innerText.indexOf('开通VIP会员') != -1) {
                    isVipSideShown = true;
                    break;
                }
            }
            //如果播放进度条上方有VIP提示
            for (i = 0; i < iqyTips.length; i++) {
                if (iqyTips[i].style.display == '' && iqyTips[i].getAttribute('data-player-hook').toLocaleLowerCase() == 'videotips') {
                    isVipTipsShown = true;
                    break;
                }
            }
            //如果直接不让试看的
            for (i = 0; i < vipPops.length; i++) {
                if ((vipPops[i].style.display == '' && vipPops[i].innerText.indexOf('试看已结束') != -1 || vipPops[i].innerText.indexOf('请先开通会员') != -1)) {
                    isVipPopShow = true;
                    //VIP提示的黑色屏蔽窗口，隐藏掉
                    vipPops[i].style.display = 'none';
                    break;
                }
            }
            //如果两个VIP提示都出现了，则表示这个视频是VIP，需要注入
            if(isVipPopShow || isVipSideShown && isVipTipsShown){
                injectPlayer('flashbox');
            }
        }
        //右下角VIP登录提示
        var qyScrolls=document.getElementsByClassName('qy-scroll-anchor');
        if(qyScrolls && qyScrolls.length>0){
            qyScrolls[0].style.display = 'none';
        }
        //注入后，跟随外部DIV变大或变小
        resizePlayer('flashbox');
    }
    else if (window.location.href.indexOf('v.qq.com') != -1) {
        //腾讯视频广告加速通过
        var videos = document.getElementsByTagName('video');
        for (i = 0; i < videos.length; i++) {
            if (videos[i].src != '') {
                //腾讯视频广告倒计时DIV classname
                adTimes = document.getElementsByClassName('txp_ad_inner');
                if (adTimes && adTimes.length>0 && adTimes[0].parentNode.className.indexOf('txp_none') != -1) {
                    //如果倒计时DIV已经没有了，则恢复视频播放速度
                    videos[i].playbackRate = 1.0;
                }
                else{
                    if(videos[i].src.indexOf('blob:https://v') == -1){
                        //如果倒计时DIV存在，且当前播放的不是视频，则直接删掉播放的链接
                        videos[i].src = '';
                    }
                    else{
                        //如果倒计时DIV存在，且当前播放是视频，则加速，直到视频广告DIV消失
                        videos[i].playbackRate = 16;
                    }
                }
            }
        }
        //根据不同情况，注入不同的DIV
        injectTencent("mod_vip_popup");
        injectTencent("txp_alert_info");
        //VIP视频，会有一个弹出框，隐藏它
        var vippops = document.getElementsByClassName('mod_vip_popup');
        if (vippops) {
            for (i = 0; i < vippops.length; i++) {
                vippops[i].style.display = 'none';
            }
        }
        //VIP试看结束后，有个黑色的遮罩层，使用这几句隐藏它
        var mask_layer = document.getElementById('mask_layer');
        if(mask_layer){
            mask_layer.style.display = 'none';
        }
        //重新设置注入的播放器大小
        resizePlayer('tenvideo_player');
    }
}, 200);
~~~

[imageTamperMonkey]:data:image/png;base64,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
[imageTamperMonkeyIcon]:data:image/png;base64,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