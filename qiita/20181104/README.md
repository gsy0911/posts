# はじめに

[このデモ](https://github.com/tensorflow/tfjs-models/tree/master/posenet)と[このゲーム](https://developers.gnavi.co.jp/entry/posenet/hasegawa)を基にユーザの姿勢を認識して部屋の電気をオンオフするシステムを作成した．
なお，このシステムは[Kit-Ok](https://qiita.com/Kit-Ok)さんと一緒に突然の思い付きと深夜テンションで作成したので，生暖かい目で見守ってほしい．



## 結果
↓YouTube動画で踊っています
[![this is it](https://i.ytimg.com/vi/GuR1UcTxJHk/hqdefault.jpg?sqp=-oaymwEZCPYBEIoBSFXyq4qpAwsIARUAAIhCGAFwAQ==&rs=AOn4CLDfElf_73ws5k5oiwVnYt2tYuqCXg)](https://youtu.be/GuR1UcTxJHk)

# 環境

* OS : Windows 10 + python3
* Web Camera : Logicool C270
* 赤外線リモコン : Nature Remo
* POST受付サーバ : IFTTT
* 姿勢認識 : PoseNet

# ソースコード

```html:demo.html
<!DOCTYPE html>
<html>
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.3.2/jquery.min.js"></script>
    <script src="https://unpkg.com/@tensorflow/tfjs"></script>
    <script src="https://unpkg.com/@tensorflow-models/posenet"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/stats.js/r16/Stats.js"></script>
    <script src="posenet_sample.js"></script>
  </head>
  <body>
    <video id="video" width="800px" height="600px" autoplay="1" style="position:absolute;"></video>
    <canvas id="canvas" width="800px" height="600px" style="position:absolute;"></canvas>
    <div class="ball"></div>
    <script>
        function doPost(url) {
            console.log("doPost");
            $.post({
                url: url,
                dataType: 'json',
                // type: 'post',
                contentType: false,
                processData: false,
                data:"data"
            }).promise().then(
                function(result, status) { 
                    // console.log("success")
                },
                function(result, status) {
                    location.reload();
                    // console.log("failure")
                }
            );
        }
        
        function audioOnEnd(){
            doPost(url);
        }

        function audioOffEnd(){
            doPost(url);
        }

    </script>
    <audio id="audio_on" preload="auto" controls onended="audioOnEnd()">
        <source src="on.wav" type="audio/wav">
    </audio>
    <audio id="audio_off" preload="auto" controls onended="audioOffEnd()">
            <source src="off.wav" type="audio/wav">
        </audio>
    
</body>
</html>
```


```javascript:posenet_sample.js
var head = document.getElementsByTagName("head");
var script = document.createElement("script");
script.setAttribute("src", "https://code.jquery.com/jquery-1.12.4.min.js");
script.setAttribute("type", "text/javascript");
script.addEventListener("load", function() {

const imageScaleFactor = 0.5;
const outputStride = 16;
const flipHorizontal = false;
const stats = new Stats();
const contentWidth = 800;
const contentHeight = 600;
const scoreThreshold = 0.7;
const keptThisIsIt = 10;

bindPage();

async function bindPage() {
    const net = await posenet.load(); // posenetの呼び出し
    let video;
    try {
        video = await loadVideo(); // video属性をロード
    } catch(e) {
        console.error(e);
        return;
    }
    detectPoseInRealTime(video, net);
}

// video属性のロード
async function loadVideo() {
    const video = await setupCamera(); // カメラのセットアップ
    video.play();
    return video;
}

// カメラのセットアップ
// video属性からストリームを取得する
async function setupCamera() {
    const video = document.getElementById('video');
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        const stream = await navigator.mediaDevices.getUserMedia({
            'audio': false,
            'video': true});
        video.srcObject = stream;

        return new Promise(resolve => {
            video.onloadedmetadata = () => {
                resolve(video);
            };
        });
    } else {
        const errorMessage = "This browser does not support video capture, or this device does not have a camera";
        alert(errorMessage);
        return Promise.reject(errorMessage);
    }
}

var keepThisIsItOn = 0;
var keepThisIsItOff = 0;

var audio_on = $('#audio_on');
var audio_off = $('#audio_off');


// 取得したストリームをestimateSinglePose()に渡して姿勢予測を実行
// requestAnimationFrameによってフレームを再描画し続ける
function detectPoseInRealTime(video, net) {
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const flipHorizontal = true; // since images are being fed from a webcam

    async function poseDetectionFrame() {
        stats.begin();
        const pose = await net.estimateSinglePose(video, imageScaleFactor, flipHorizontal, outputStride);
        ctx.clearRect(0, 0, contentWidth,contentHeight);

        ctx.save();
        ctx.scale(-1, 1);
        ctx.translate(-contentWidth, 0);
        ctx.drawImage(video, 0, 0, contentWidth, contentHeight);
        ctx.restore();



        if(pose.score > scoreThreshold){
            if(isThisIsItOn(pose)){
                keepThisIsItOn++;
                if(keepThisIsItOn >= keptThisIsIt){
                    url = "https://maker.ifttt.com/trigger/[event name]/with/key/[your sercret key]";
                    audio_on[0].play();
                    return;
                }
            }else if(isThisIsItOff(pose)){
                keepThisIsItOff++;
                if(keepThisIsItOff >= keptThisIsIt){
                    url = "https://maker.ifttt.com/trigger/[event name]/with/key/[your sercret key]";
                    audio_off[0].play();            
                    return;
                }
            }
            pose.keypoints.forEach(({position}) => {
                drawPoint(position,ctx);
            });

        }

        stats.end();

        requestAnimationFrame(poseDetectionFrame);
    }

    poseDetectionFrame();


}

// 与えられたKeypointをcanvasに描画する
function drawPoint(position,ctx){
    ctx.beginPath();
    ctx.arc(position.x , position.y, 10, 0, 2 * Math.PI);
    ctx.fillStyle = "pink";
    ctx.fill();
}

function isThisIsItOn(pose){
    var keypoints = pose.keypoints
    if(keypoints[5].position.y < keypoints[9].position.y && keypoints[6].position.y > keypoints[10].position.y){
        console.log("turn On")
        return true;
    }
    return false;
}

function isThisIsItOff(pose){
    var keypoints = pose.keypoints
    if(keypoints[5].position.y > keypoints[9].position.y && keypoints[6].position.y < keypoints[10].position.y){
        console.log("turn Off")
        return true;
    }
    return false;
}
});
document.head.appendChild(script);
```

# 実行手順

### 事前準備
1. IFTTTの`this`に`webhook`を，`that`に`Nature Remo`の電気コントロールを指定
2. 電気のオンとオフの両方に対して`Applet`を作成する
3. お好みのオーディオファイルを用意して，名前の変換する．（または用意しない）

### 電気のオンオフ実行

1. 上の2つのソースコードをローカルに落とす
2. ターミナルで`C:\Users\[yourname]\[落としたディレクトリ]> python -m http.server 8000`を実行してサーバーを起動する
3. ブラウザから`http://localhost:8000/demo.html`にアクセスする
4. 踊る（右手を挙げたThis Is Itの場合は「消灯」，左手を挙げたThis Is Itの場合は「点灯」に設定してある）
5. 電気が点いたり消えたりする

# 結果（再掲）

↓YouTube動画
[![this is it](https://i.ytimg.com/vi/GuR1UcTxJHk/hqdefault.jpg?sqp=-oaymwEZCPYBEIoBSFXyq4qpAwsIARUAAIhCGAFwAQ==&rs=AOn4CLDfElf_73ws5k5oiwVnYt2tYuqCXg)](https://youtu.be/GuR1UcTxJHk)

# おわりに

意外と時間がかかるかと思ったが，公開されているライブラリが優秀でほぼプログラミングすることなくスマートなお家を実現できた．

ただ，今回のプログラムには今後修正すべき点がたくさんある．
* html側にjavascripのコードが入っている
* 動作の再受付を`failure`の場合に行っている
* 関数が汚い．．．

などなど．．．挙げればきりがない．．．
まぁ，今回は取り急ぎだったので，この状態で公開した．
