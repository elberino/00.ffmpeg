======================================================

**técnica de mapping**
video1.avi -i video2.avi -filter complex “[0:v:0] [0:a:0] [1:v:0] [1:a:0] concat=n=2:v=1:a=1 [v] [a]” -map “[v]” -map “[a]” output_video.avi

// técnica dos 3 arqivos
ffmpeg -i file1.mp4 -i file2.mp4 -i file3.mp4 \
-filter_complex "[0:v] [1:v] [2:v]
concat=n=3:v=1:a=1 [v] [a]" \
-map "[vv]" -map "[aa]" mergedVideo.mp4

**Concat demuxer**
# criar arquivo seguindo modelo
file '/path/to/file1.wav'
file '/path/to/file2.wav'
file '/path/to/file3.wav'
$ ffmpeg -f concat -i list.txt -c copy out.mp4
# concat modo safe zero
$ ffmpeg -f concat -safe 0 -i files.txt -c copy output.mkv

**Trim 2 segments and export in the same output**
ffmpeg -i video \
       -vf "select='between(t,4,6.5)+between(t,17,26)+between(t,74,91)',
            setpts=N/FRAME_RATE/TB" \
       -af "aselect='between(t,4,6.5)+between(t,17,26)+between(t,74,91)',
            asetpts=N/SR/TB" out.mp4

**conversaão twitter**
ffmpeg -i video.mp4 -r 25 -crf 23 -c:v libx264 -b:v 1M -vf scale="trunc(oh*a/2)*2:360" -y video-tt_360.mp4
ffmpeg -i "video.mp4" -ss 00:25.1 -t 50 -r 30 -crf 23 -c:v libx264 -b:v 1.5M -vf scale="trunc(oh*a/2)*2:480" -an -y  video-tt_480.mp4

**template -t *.mp4, sem audio**
 ffmpeg -i comp_.mp4 -f mp4 -vcodec libx264 -preset fast -profile:v main -an -ss 00:00 -t 00:00 output_comp_.mp4

**template -t *.mp4, com audio**
 ffmpeg -i comp_.mp4 -f mp4 -vcodec libx264 -preset fast -profile:v main -a:c -ss 00:00 -t 00:00 output_comp_.mp4

**skip frames**
ffmpeg -i in1.mp4 -vf fps=fps=1.5 -an -y output.mp4
ffmpeg -i output.mp4 -vf eq=saturation=1.5 -y output2.mp4

**This will extract all I-frames from frame #0 to frame #300**
ffmpeg -i in.mp4 -vf select='eq(pict_type,I)*between(n,0,300)' -vsync 0 i%03d.png
ffmpeg -i 2.flv -vf "select=eq(pict_type\,I)" -vsync vfr frame-%02d.png

**produzir frames**
ffmpeg -i input.mp4 -vf fps=1 out%d.png (testar com esse nome: frame%06d.png)
ffmpeg -i input.mp4 -vf fps=3 out%d.png// 3 frames por segundos exp
ffmpeg -i input.mp4 -vf fps=1/60 out%d.png // 1 frame por minuto
ffmpeg -i input.mp4 -vf fps=1/600 out%d.png // 1 frame a cada 10min
ffmpeg -i test-%09d.png -vcodec libx264 -pix_fmt yuv420p test-merged-qt.mp4

**sequencia de imagem pra video, especificando fps final**
ffmpeg -framerate 12 -start_number 9685 -i DSC_%04d.JPG -c:v libx264 -r 24 output-v5.mp4

**sequencia de imagem pra video, especificando fps final + range de frames**
# comando -frames:v é o que define a abra
ffmpeg -framerate 12 -start_number 9685 -i DSC_%04d.JPG -frames:v 9800 -c:v libx264 -r 24  output-v5.mp4

**rename**
rename s'/^DSC/DSC_/' *.JPG
rename s'/^MOV/MOV_/' *.MPG

**remover .THM**
rm *.THM

===========================================

ffmpeg -i inputVideo.mp4 -ss 00:03 -to 00:08 -c:v libx264 -crf 30 trim_opseek_encode.mp4

// re-scale + crop
ffmpeg -i input.mp4 -filter_complex
"[0:v]trim=start=0:end=3,setpts=PTS-STARTPTS,split[va1][vb1];
 [0:v]trim=start=5:end=10,setpts=PTS-STARTPTS,split[va2][vb2];
 [0:a]atrim=start=0:end=3,asetpts=PTS-STARTPTS,asplit[aa1][ab1];
 [0:a]atrim=start=5:end=10,asetpts=PTS-STARTPTS,asplit[aa2][ab2];
 [va1]scale=700:700:force_original_aspect_ratio=increase,crop=700:700[700_1];
 [va2]scale=700:700:force_original_aspect_ratio=increase,crop=700:700[700_2];
 [vb1]scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920[1920_1];
 [vb2]scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920[1920_2]"
-map "[700_1]"  -map "[aa1]" 1a.mp4
-map "[700_2]"  -map "[aa2]" 1b.mp4
-map "[1920_1]" -map "[ab1]" 2a.mp4
-map "[1920_2]" -map "[ab2]" 2b.mp4

ffmpeg -i in.mp4 -vf select='eq(pict_type,I)*between(t\,0\,300)' -vsync 0 i%03d.png
ffmpeg -i in.m4v -vf 'select='eq(between*(t\,0\,300))' -vsync 0 i%03d.png

**gif: alta qualidade**
ffmpeg -ss 61.0 -t 2.5 -i StickAround.mp4 -filter_complex "[0:v] palettegen" palette.png
ffmpeg -ss 61.0 -t 2.5 -i StickAround.mp4 -i palette.png -filter_complex "[0:v][1:v] paletteuse" prettyStickAround.gif

**gif: preservando transparencia**
ffmpeg -i input.gif -vf -filter_complex "[0:v] scale=320:240,split [a][b];\
[a] palettegen=reserve_transparent=on:transparency_color=ffffff p]; \
[b][p] paletteuse" output_converted.gif

**gif: zerando pts**
ffmpeg -y -i .gif  -filter_complex /
"fps=15,setsar=1,palettegen" palette.png
ffmpeg -y  -i .gif /
-i palette.png -filter_complex "[0]fps=15,setsar=1[x];[x][1:v]paletteuse" /
_480p.gif

**gif: paleta e gerar gif no mesmo comando**
ffmpeg -i "noise-pigeon-2329.mp4" -filter:v " \
fps=18,scale=420:-1:flags=lanczos,split[s0][s1]; \
[s0]palettegen[p]; \
[s1][p]paletteuse" ../exp/noise-pigeon-2329.gif -y -hide_banner

**gif: seleção de tempos**
ffmpeg -i output_video_no_music3.mp4 -filter:v " \
select='between(t\,0,3.75)+between(t\,8.25,9)+between(t\,13,14)+between(t\,16,20)'
,fps=15,scale=480:-1:flags=lanczos,split[s0][s1]; \
[s0]palettegen[p]; \
[s1][p]paletteuse" NAME.gif -y -hide_banner

ffmpeg -i video \
       -vf "select='between(t,4,6.5)+between(t,17,26)+between(t,74,91)',
            setpts=N/FRAME_RATE/TB" \
       -af "aselect='between(t,4,6.5)+between(t,17,26)+between(t,74,91)',
            asetpts=N/SR/TB" out.mp4

ffmpeg -i test.mp4 -filter_complex \
'[0:v] trim=start=5:end=10, setpts=PTS-STARTPTS [v0]; \
 [0:a]atrim=start=5:end=10,asetpts=PTS-STARTPTS [a0];'\
 -map [v0] -map [a0]  output.mp4

ffmpeg -i video.mp4 \
-filter_complex "atrim=start=2:end=11, asetpts=N/SR/TB" out-atrim.mp3

**mapping 1 video sem audio com 1 arquivo de audio**
ffmpeg -y -i out-aselect-intro.mp4 -i out-aselect-intro.mp3 \
-map 0:0 -map 1:0 merged_intro.mp4


**imagem estatica para video de 15s**
ffmpeg -loop 1 -i image.png -c:v libx264 -t 15 -pix_fmt yuv420p -vf scale=320:240 out.mp4
    The -t 15 makes it 15 seconds long.
    The -vf scale=320:240 sets the width/height.


**fade out ffmpeg**
ffmpeg -i input.mp4 \
    -filter_complex \
        "[0:v]fade=t=out:st=1798:d=1[v]; \
         [0:a]afade=t=out:st=1798:d=1[a]" \
     -map "[v]" -map "[a]" output.mp4

**Thumbnail**
$ ffmpeg -i video.mp4 -ss 00:00:14.435 -vframes 1 imagem.png

**seq de img para gif**
ffmpeg -framerate 12 -start_number 0100 -i DSC_%04d.JPG -vframes 000 \
 -filter:v " \
fps=15,scale=480:-1:flags=lanczos,split[s0][s1]; \
[s0]palettegen[p]; \
[s1][p]paletteuse" \
just-pigeon.gif -y -hide_banner

**pts no filtro**
trim=start=0:end=3,setpts=PTS-STARTPTS


ffmpeg -i in.gif -filter:v "crop=out_w:out_h:x:y" out.gif
reply that: https://stackoverflow.com/questions/69010256/can-ffmpeg-crop-gif-image-under-1-second

**resize/crop**
ffmpeg -i INPUT.mov -vf "scale=356:278:flags=spline,unsharp=2.5:2.5:1.5" -b:v 256k OUTPUT.mov

ffmpeg -i video.mp4 -r 30 -crf 23 -c:v libx264 -b:v 1.2M -vf scale="trunc(oh*a/2)*2:400" -y video-out.mp4

ffmpeg -i input.avi -filter:v scale=720:-1 -c:a copy output.mkv

//altura
scale="trunc(oh*a/2)*2:720"

...or a given width (1280 in this example):

//largura
scale="1280:trunc(ow/a/2)*2"



// my code
ffmpeg -framerate 6 -start_number 183 -i DSC_%5d.JPG -vframes 30 -filter:v " \
 8813  2022-05-22 20:57:55 fps=12,scale=480:-1:flags=lanczos,crop=360:360,setsar=1,split[s0][s1]; \
 8814  2022-05-22 20:57:55 [s0]palettegen[p]; \
 8815  2022-05-22 20:57:55 [s1][p]paletteuse" ../just-pigeon-3-cropped.gif -y -hide_banner
 
// ref crop crop=ih:ih:20:10
