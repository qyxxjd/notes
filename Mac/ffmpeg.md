#### 下载文件

```shell
ffmpeg -i http://xxx output.xxx
```

#### 视频文件转GIF

```shell
ffmpeg -i xxx.mp4 xxx.gif 

// 指定比特率，转换高质量GIF
ffmpeg -i xxx.mp4 -b 2048k xxx.gif
```

#### 合并音频和视频文件

```shell
ffmpeg -i video.mp4 -i audio.mp3 -c：v copy -c：aac -strict experimental output.mp4
```
