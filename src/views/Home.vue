<template>
  <div id="home">
    <video ref="video" id="video" autoplay width="704" height="480" style="-webkit-transform: scaleX(-1);">Video stream not available.</video>
    <button v-if="live.progress" @click="live_finish">STOP</button>
    <button v-else @click="live_start">START</button>
    <button @click="download">DOWNLOAD</button>
  </div>
</template>

<script>
    import axios from 'axios';
export default {
  name: 'Home',
    data(){
      return{
          stream:{
              video: null,
              constrains: {
                  video: true,
                  audio: true
              },
              recorder: null,
              record_data: [],
          },
          live:{
              intervalID: null,
              progress: false,
              intervalTime: 10000,  // 10秒
          },
      }
    },
    methods:{
        startup() {
            this.videoStart();
            this.startRecorder();
        },
        live_start(){
            // ライブ開始処理
            this.stream.recorder.start();
            this.live.progress = true;
            this.live.intervalID = setInterval(
                this.send_serve,
                this.live.intervalTime
            );
        },
        send_serve(){

            // 配信の切り替え
            this.stream.recorder.stop();
            this.stream.recorder.start();
        },
        live_finish(){
            clearInterval(this.live.intervalID);
            this.stream.recorder.stop();
            this.live.progress = false;
            console.log(this.stream.recorder);
        },
        download(){
            console.log(this.stream.record_data);
            let blob = new Blob(this.stream.record_data, { type: 'video/webm' });
            let url = window.URL.createObjectURL(blob);
            let a = document.createElement('a');
            document.body.appendChild(a);
            a.style = 'display:none';
            a.href = url;
            a.download = 'test.webm';
            a.click();
            window.URL.revokeObjectURL(url);
        },
        videoStart() {
            const self = this;

            // デバイスのカメラを取得
            navigator.mediaDevices.getUserMedia(this.stream.constrains)
                .then(function(stream) {
                    // Videoを投影
                  self.video.srcObject = stream;
                  self.video.play();
                })
                .catch(function(err) {
                    console.log("An error occured! " + err);
                });
        },
        startRecorder() {
            const self = this;

            navigator.mediaDevices.getUserMedia(this.stream.constrains)
                .then(function (stream) {
                    // レコーダーの開始
                    self.stream.recorder = new MediaRecorder(stream);

                    // 録画終了後の処理
                    self.stream.recorder.ondataavailable = function (e) {
                        self.stream.record_data.push(e.data);

                        // ファイル形式に変換
                        let video_file = new Blob(self.stream.record_data, { type: 'video/webm' });
                        console.log(video_file);
                        // サーバへのファイル送信
                        axios.post('/api/fileupload',video_file).then(response =>{
                            console.log(response);
                        });
                        self.stream.record_data = [];
                    }
                }).catch(function(err) {
                console.log("An error occured! " + err);
            });
        },
    },
    mounted() {
      this.video = this.$refs.video;
      this.startup();
    }
}
</script>
