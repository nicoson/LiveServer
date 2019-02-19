# 1. 准备带有 rtmp module 的 nginx 镜像作为服务器
> docker pull tiangolo/nginx-rtmp

# 2. 运行 nginx 服务
> docker run -it -p 1935:1935 -p 8080:8080 -v $(pwd)/nginx-hls.conf:/etc/nginx/nginx.conf tiangolo/nginx-rtmp bash
> nginx

# 3. 开始推流
3.1 rtmp 推流地址
> ffmpeg -re -stream_loop -1 -i m1.mp4 -vcodec copy -acodec copy  -f flv rtmp://localhost:1935/live/abc

3.2 hls 推流地址
> ffmpeg -re -stream_loop -1 -i m1.mp4 -vcodec copy -acodec copy  -f flv rtmp://localhost:1935/hls/abc

# 4. 读取直播流
4.1 rtmp 拉流地址
> rtmp://localhost:1935/live/abc

4.2 hls 拉流地址
需要注意：hls拉流与推流的端口不同
> http://localhost:8080/hls/abc.m3u8

# 5. 运行前端 Demo 页面
> python3 -m http.server 8888

登录页面[localhost:8888/video.html][localhost:8888/video.html]

# 6. 推流外传
6.1 mac 下查看设备
>ffmpeg -f avfoundation -list_devices true -i ""

6.2 mac 下共享桌面
根据6.1中查到的设备号来确定 '-i' 的参数，此处注意：如果有两个屏幕，则运行推流的屏幕将会自动停止推流服务
> ffmpeg -f avfoundation -i "2" -vcodec libx264 -preset ultrafast -acodec libfaac -f flv rtmp://localhost:1935/live/abc 

6.3 mac 摄像头直播
> ffmpeg -f avfoundation -framerate 30 -pix_fmt uyvy422 -i "0" -s 640x360 -pix_fmt uyvy422 -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -f flv rtmp://localhost:1935/live/abc

6.4 ffmpeg 转播流
> ffmpeg -i rtmp://cyberplayerplay.kaywang.cn/cyberplayer/demo201711-L1 -c copy -f flv rtmp://localhost:1935/hls/abc
> ffmpeg -i http://d2e6xlgy8sg8ji.cloudfront.net/liveedge/eratv3/chunklist.m3u8 -c copy -bsf:a aac_adtstoasc -f flv rtmp://localhost:1935/hls/abc