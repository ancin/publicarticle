# SRS install
git clone https://github.com/ossrs/srs
cd srs/trunk
./configure && make

## srs config
edit this file as follows:
conf/rtmp.conf
listen              1935;
max_connections     1000;
vhost __defaultVhost__ {
}

## start srs
./objs/srs -c conf/rtmp.conf

# 推流
ffmpeg -f dshow -i video="Integrated Camera" -vcodec libx264  -preset:v ultrafast -tune:v zerolatency -f flv rtmp://xxxxx:1935/live/mooc

## 低延迟
 ffmpeg -f dshow -i video="Integrated Camera"  -g 10 -f flv rtmp://localhost:1935/live/mooc

 ffmpeg -re -rtbufsize 1000M -f dshow -i video="Integrated Camera"  -g 10 -f flv rtmp://XXXX:1935/live/mooc

 # 问题
  错误信息：real-time buffer [Integrated Camera] [video input] too full or near too full
    添加参数 -rtbufsize 1000M 

    还可以参考git
    https://github.com/rdp/screen-capture-recorder-to-video-windows-free/issues/37

    FFsplit use this for streaming:

ffmpeg.exe" -y -f dshow -f dshow -i video="FFsource" -f dshow -i audio="FFsplit Playback Mixer" -vcodec libx264 -preset veryfast -g 60 -maxrate 2000k -bufsize 2000k -b:v 2000k -x264opts fullrange=on:colorprim=bt709:transfer=bt709:colormatrix=bt709:nal-hrd=cbr -pix_fmt yuv420p -acodec libvo_aacenc -ab 128k -ac 2 -ar 44100 -r 30 -f mpegts \\.\pipe\midpipeIN

fmpeg.exe" -analyzeduration 500000 -y -f mpegts -i \\.\pipe\midpipeOUT -vcodec copy -acodec libvo_aacenc -ab 128k -ac 2 -ar 44100 -flags +bitexact -metadata encoder="FFsplit 0.7.0.23" -f flv "rtmp://127.0.0.1:4759/app/live_50837324_SypWjj7g2x5Lezd5PbCSPG7HtjTlz4 flashver=FMLE/3.0\20(compatible;\20FMSc\201.0)"

dont know why there is 2x audio compression.. but
for problems in first post try use -re
im using this:
"ffmpeg -loglevel warning -re -rtbufsize 1500M -f dshow -i video="UScreenCapture" -f dshow -i audio="virtual-audio-capturer" -c:v libx264 -force_key_frames expr:gte(t,n_forced*2) -b:v 1000k -minrate 1000k -maxrate 1000k -bufsize:v 1000k -preset:v veryfast -pix_fmt yuv420p -tune film -c:a libmp3lame -q:a 2 -ar 44100 -ac 2 -f flv "rtmp://localhost:1935/app/live_27039710_6ONjNDTVXQ5NTYKUINo"

and is ok,



