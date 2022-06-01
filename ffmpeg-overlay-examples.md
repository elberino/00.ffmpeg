##### BEST EXAMPLES: OVERLAY #####
==============================================================================

// video merge 
ffmpeg -y -i overlayed_mig19_j6_42%.mp4 -i output_mig19_j6_loopped_true.webm -filter_complex\ 
"[1:v]setpts=PTS-STARTPTS+2/TB,[ovr];[0:v][ovr]overlay=enable=gte(t\,2):eof_action=pass,format=yuva420p[vid]" \
-map "[vid]" -map 0:a? -c:v libx264 -crf 25 -c:a copy -movflags +faststart chinese-airforce_mig19-j6.mp4
_   _   _   _   _   _   _   _   _   _   _   _   _   _   _   
Don't forget to add the / after t in gte(t,10) – Tina J Jan 4 '19 at 0:14

_   _   _   _   _   _   _   _   _   _   _   _   _   _   _   

-filter_complex "[0:v][1:v]overlay=eof_action=pass,format=yuv420p[out]"

 overlay=x=10:y=10...

[ffmpeg doc](https://ffmpeg.org/ffmpeg-filters.html#overlay)

==============================================================================

ffmpeg -i in1 -i in2 -filter_complex
       "[1]format=yuva444p,colorchannelmixer=aa=0.5[in2];[0][in2]overlay"
       out

where 0.5 sets 50% transparency for the 2nd input. The format filter is needed to make sure that the 2nd video has an alpha channel.

To resize in2 to match in1, use

ffmpeg -i in1 -i in2 -filter_complex
       "[1]format=yuva444p,colorchannelmixer=aa=0.5[in2];
        [in2][0]scale2ref[in2][in1];[in1][in2]overlay"
       out

_   _   _   _   _   _   _   _   _   _   _   _   _   _   _   

Instead of colorchannelmixer you could use a lighter filter like lut and set the alpha channel to 255*{alpha percentage} like so: lut=a=255*0.5

_   _   _   _   _   _   _   _   _   _   _   _   _   _   _   

Changing the format to YUV all the time can be very taxing, especially if the input format is RGB, so you could let ffmpeg choose the best format with alpha to use out of a provided list, like so format=yuva420p|yuva444p|yuva422p|rgba|abgr|bgra|gbrap|ya8; here we ask ffmpeg to choose best format between all variants of yuv, rgb and/or grayscale. To see all supported pixel formats, use ffmpeg -pix_fmts

==============================================================================

ffmpeg -y \
-i videoin.mp4 \
-i anim01.gif \
-filter_complex [1:v]scale=1080:1920[ovrl] [0:v][ovrl]overlay=main_w-overlay_w:main_h-overlay_h \
-frames:v 900 \
-codec:a copy \
-codec:v libx264 \
-preset ultrafast \
video.mp4



ffmpeg -y -i xxx.mp4 -ignore_loop 0 -i xxx.gif -filter_complex "[1:v]scale=1080:1920[ovrl];[0:v][ovrl]overlay=0:0" -frames:v 900 -codec:a copy -codec:v libx264 -max_muxing_queue_size 2048 video.mp4

==============================================================================

// ✅ deu certo sobrepor png em mp4
ffmpeg -y -i "sonoplasta_250521_cpi-beat_520px.mkv" -i "vlcsnap_newsonoplasta.png" \
-filter_complex "[1:0] setsar=sar=1,format=rgba [1sared]; [0:0]format=rgba [0rgbd]; [0rgbd][1sared]blend=all_mode='addition':repeatlast=1:all_opacity=1,format=yuva422p10le" \
-c:v libx264 -preset slow -tune film -crf 23 \
-c:a aac -strict -2 -ac 2 -b:a 192k \
-pix_fmt yuv420p "mundinhosono_teste_2.mp4"

==============================================================================

ffmpeg -i input.gif -i logo.png -filter_complex "[0]scale=512:512:force_original_aspect_ratio=decrease,pad=512:512:-1:-1:color=black@0[bg];[bg][1]overlay=format=auto,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" output.gif

For pad filter color=black@0 will be transparent.
==============================================================================

// sobreposição de gif estilo marca d'agua
[❓] You have to generate the palette for the same visual content that you wish to encode to GIF.

ffmpeg -v error -i image.gif -i watermark.gif -filter_complex  '[1:v]scale=80:30[wm];[0:v][wm]overlay=(main_w-overlay_w) - main_h/30:(main_h-overlay_h) - main_h/30, palettegen' palette.png -y;

ffmpeg -v error -i image.gif -i watermark.gif -i palette.png -filter_complex '[1:v]scale=80:30[wm];[0:v][wm]overlay=(main_w-overlay_w) - main_h/30:(main_h-overlay_h) - main_h/30[vid];[vid][2]paletteuse' -an image-watermark.gif -y

==============================================================================

How to overlay two videos with blend filter in ffmpeg



I need to do a lot of videos with the next specifications:

    A background video (bg.mp4)
    Overlay a sequence of png images img1.png to img300.png (img%d.png) with a rate of 30 fps
    Overlay a video with dust effects using a blend-lighten filter (dust.mp4)
    Scale all the inputs to 1600x900 and if the input have not the aspect ratio, then crop them.
    Specify the duration of the output-video to 10 sec (is the duration of image sequence at 30fps).

I've being doing a lot of test with different commands but always shows error.





Well, I think I got it in the next command:

ffmpeg -ss 00:00:18.300 -i music.mp3 -loop 1 -i bg.mp4 -i ac%d.png -i dust.mp4 -filter_complex "[1:0]scale=1600:ih*1200/iw, crop=1600:900[a];[a][2:0] overlay=0:0[b]; [3:0] scale=1600:ih*1600/iw, crop=1600:900,setsar=1[c]; [b][c] blend=all_mode='overlay':all_opacity=0.2" -shortest -y output.mp4

I'm going to explain in order to share what I've found:

    Declaring the inputs:

ffmpeg -ss 00:00:18.300 -i music.mp3 -loop 1 -i bg.mp4 -i ac%d.png -i dust.mp4

    Adding the filter complex. First part: [1,0] is the second element of the inputs (bg.mp4) and scaling to get the max values, and then cropping with the size I need, the result of this opperation, is in the [a] element.

[1:0]scale=1600:ih*1600/iw, crop=1600:900, setsar=1[a];

    Second Part: Put the PNGs sequence over the resized video (bg.mp4, now [a]) and saving the resunt in the [b] element.

[a][2:0] overlay=0:0[b];

    Scaling and cropping the fourth input (overlay.mp4) and saving in the [c] element.

[3:0]scale=1600:ih*1600/iw, crop=1600:900,setsar=1[c];

    Mixing the first result with the overlay video with an "overlay" blending mode, and with an opacity of 0.1 because the video has gray tones and makes the result so dark.

[b][c] blend=all_mode='overlay':all_opacity=0.1

That's all. If anyone can explay how this scaling filter works, I would thank a lot!

==============================================================================

ffmpeg - blend video with images
https://video.stackexchange.com/questions/27615/ffmpeg-blend-video-with-images



The blend filter requires both inputs to have the same resolution. You can use the scale2ref filter to resize the overlay.

ffmpeg -i dogs.mp4 -i red.jpg -filter_complex '[1:v][0:v]scale2ref[ckout][vid];[vid][ckout]blend=all_mode='multiply'[out]' -map '[out]' output.mp4

==============================================================================

ffmpeg -c:v libvpx-vp9 -i transparent.webm -i bg.mp4 -filter_complex "[0:v]format=yuva420p [a]; [1:v]format=yuv420p [b]; [a][b]blend=all_mode='normal':shortest=1:all_opacity=1,format=yuv420p" output.mp4

==============================================================================
Try to convert colorspace before output with format=yuva422p10le (maybe similar colorspace, but this works for me), and maybe you need to convert input colorspace of overlay (or both movies) to rgba (format=rgba). Like this:

ffmpeg -i ttt.jpg -vf "movie=fix2.png [fix]; [in] setsar=sar=1,format=rgba [inf]; [inf][fix] blend=all_mode=overlay:all_opacity=1,format=yuva422p10le [out]" -q:v 1 -y xxx.jpg

==============================================================================

Apparently, it's a problem with color space. Something like this works:

ffmpeg -i "$1" -i "$2" \
-filter_complex "[1:0] setsar=sar=1,format=rgba [1sared]; [0:0]format=rgba [0rgbd]; [0rgbd][1sared]blend=all_mode='addition':repeatlast=1:all_opacity=1,format=yuva422p10le" \
-c:v libx264 -preset slow -tune film -crf 19 \
-c:a aac -strict -2 -ac 2 -b:a 256k \
-pix_fmt yuv420p "$3"

(substitute $1 with your actual video, $2 with your overlay, and $3 with the output file name)


==============================================================================
Vhs overlay
https://www.reddit.com/r/ffmpeg/comments/fo9b8t/vhs_overlay/
==============================================================================
Overlay a static PNG
ffmpeg -y -i video.mp4 -i overlay.png -filter_complex [0]overlay=x=0:y=0[out] -map [out] -map 0:a? test.mp4
==============================================================================
How to add a PNG overlay on a video using FFmpeg
https://www.bannerbear.com/blog/how-to-add-a-png-overlay-on-a-video-using-ffmpeg/
==============================================================================
ffmpeg -y -i overlayed_mig19_j6_42%.mp4 \
-i output_mig19_j6_mandarim.gif \
-i output_mig19_j6.gif \
-filter_complex " \
[0]setsar=sar=1,format=rgba [inf]; \
[1]setpts=PTS-STARTPTS+2/TB [fix1]; \
[2]setpts=PTS-STARTPTS+4/TB [fix2]; \
[inf][fix1]overlay=enable=gte(t\,2):eof_action=pass,format=yuva420p[inf]; \
[inf][fix2]overlay=enable=gte(t\,4):eof_action=pass,format=yuva420p[inf]" \
-map "[inf]" -c:v libx264 \
-crf 23 -pix_fmt yuva420p -c:a copy \
 -movflags +faststart -aspect 16:9 lion+text.mp4

==============================================================================
ffmpeg input.mp4 -vsync 0 -vf  %05d.png

ffmpeg -i .\transparent_%05d.png -vf palettegen=reserve_transparent=1 palette.png

ffmpeg -thread_queue_size 32 -framerate 25 -i .\transparent_%05d.png -i palette.png -lavfi paletteuse final.gif

ffmpeg -f lavfi -i color=c=36393f:s=540x303 -frames:v 1 bg.png
ffmpeg -f lavfi -i color=c=8d0d0d:s=640x360 -frames:v 1 bg.png


ffmpeg -i .\transparent_%05d.png -i bg.png -vsync 0 -filter_complex "[1:v][0:v]overlay[out]" -map "[out]" combined_%05d.png
==============================================================================
// overlay solid color over a video
ffmpeg -y -i Lion_eating.mp4 -i bg.png -filter_complex \
"[1]format=yuva444p,colorchannelmixer=aa=0.42,vibrance=intensity=1.35[in2];\
[in2][0]scale2ref[in2][in1];[in1][in2]overlay" Lion_eating_42%.mp4
==============================================================================
+++Overlay et burned subs via filter_complex+++
ffmpeg -i input.mp4 \
	   -i input.png \
	   -filter_complex "[0:0][1:0]overlay=main_w-overlay_w-10:10,subtitles=input.srt" \
	   -c:v h264 \
	   -c:a copy \
	   output.mp4

==============================================================================

y syntax looks like this:

ffmpeg -i "$introVid" -i "$introAud" -i "$mainVid" -i "$mainAud" -i "$outroVid" -i "$outroAud" -i "$logo" -i "$mainVid" -i "$mainAud" \
-filter_complex \
"[2:0]trim=0.4:60[trimV1]; \
 [3:0]atrim=0.4:60[trimA1]; \
 [trimV1][6:v]overlay=main_w-overlay_w-20:15,fade=in:s=2:d=0.5:alpha=1,fade=out:s=60:d=0.5:alpha=1[fade]; \
 [7:0]trim=60.2:72[trimV2]; [8:0]atrim=60.2:72[trimA2]; \
 [0:0] [1:0] [fade] [trimA1] [4:0] [5:0] [trimV2] [trimA2] concat=n=4:v=1:a=1[cv][a]; \
 [cv]scale=864:480:flags=gauss:interl=0[scal]" \
-map "[scal]" -map "[a]" -pix_fmt yuv420p -c:v libx264 -preset fast -y "$out"

It works mostly, but the problem is that I get a black video, with the same length then the main video, on the 3rd place. Interesting is also, when I watch the ffmpeg process, it hangs shortly on time 1:26min and then it jumps to 2:40min. Normally the complete test video have around 1:30min.

The output what I get is at the moment:

([intro][trimmed main with logo][black video][outro][credits])  the black video part is to much.


_   _   _   _   _   _   _   _   _   _   _   _   _   _   _

The trim command needed a PTS Flags: for video= setpts=PTS-STARTPTS and for audio= asetpts=PTS-STARTPTS



Without new pts, ffmpeg count the time wrong, after trimming. When some parts from a video needed to place more times, it is also not necessary to input them more times.

So here is the right syntax for that:


ffmpeg -i "$introVid" -i "$introAud" -i "$mainVid" -i "$mainAud" \ 
-i "$outroVid" -i "$outroAud" -i "$logo" -filter_complex "\
[2:v]trim=0.4:$cut,setpts=PTS-STARTPTS[trimV1]; \
[3:a]atrim=0.4:$cut,asetpts=PTS-STARTPTS[trimA1]; \
[trimV1][6:v]overlay=main_w-overlay_w-20:15, \
fade=in:0:25,fade=out:$cuf:25[fade]; \
[2:v]trim=$cut:$len,setpts=PTS-STARTPTS[trimV2]; \
[3:a]atrim=$cut:$len,asetpts=PTS-STARTPTS[trimA2]; \
[0:v] [1:a] [fade] [trimA1] [4:v] [5:a] [trimV2] [trimA2] \ 
concat=n=4:v=1:a=1[cv][a]; \
[cv]scale=864:480:flags=gauss:interl=0[scal]" \
-map "[scal]" -map "[a]" -pix_fmt yuv420p -c:v libx264 -preset fast -y "out.mp4" 

==============================================================================

ffmpeg \
-framerate 20 -loop 1 -t 1 -i text1.png \
-framerate 20 -loop 1 -t 1 -i text2.png \
-filter_complex "
[0]scale=640:360,setsar=1[a];
[1]scale=640:360,setsar=1[b];
[a][b]concat=n=2:v=1:a=0,split[a][b];
[a]palettegen=reserve_transparent=on:transparency_color=ffffff[p];
[b][p]paletteuse
" text-out.gif -hide_banner -y


==============================================================================

For example, to overlay an image over video

ffmpeg -i video.mkv -i image.png -filter_complex '[0:v][1:v]overlay[out]' -map
'[out]' out.mkv

Here [0:v] refers to the first video stream in the first input file, which is linked to the first (main) input of the overlay filter. Similarly the first video stream in the second input is linked to the second (overlay) input of overlay.

Assuming there is only one video stream in each input file, we can omit input labels, so the above is equivalent to

ffmpeg -i video.mkv -i image.png -filter_complex 'overlay[out]' -map
'[out]' out.mkv

Furthermore we can omit the output label and the single output from the filter graph will be added to the output file automatically, so we can simply write

ffmpeg -i video.mkv -i image.png -filter_complex 'overlay' out.mkv

As a special exception, you can use a bitmap subtitle stream as input: it will be converted into a video with the same size as the largest video in the file, or 720×576 if no video is present. Note that this is an experimental and temporary solution. It will be removed once libavfilter has proper support for subtitles.

For example, to hardcode subtitles on top of a DVB-T recording stored in MPEG-TS format, delaying the subtitles by 1 second:

ffmpeg -i input.ts -filter_complex \
  '[#0x2ef] setpts=PTS+1/TB [sub] ; [#0x2d0] [sub] overlay' \
  -sn -map '#0x2dc' output.mkv

(0x2d0, 0x2dc and 0x2ef are the MPEG-TS PIDs of respectively the video, audio and subtitles streams; 0:0, 0:3 and 0:7 would have worked too)

To generate 5 seconds of pure red video using lavfi color source:

ffmpeg -filter_complex 'color=c=red' -t 5 out.mkv

==============================================================================

//  take a single video, then take a 50% scaled version of that video and overlay it on to the original video
ffmpeg -i test.mov -filter_complex \
"[0]split[v1][v2];\
[v2]scale=iw/2:-1[v2];\
[v1][v2]overlay=main_w/4:main_h/4" \
==============================================================================

// aplicação de tecnica simples hue + scaling 
ffmpeg -y -i out-video_no-audio.mp4 -filter_complex "[0]split=4[c1][c2][c3][c4],fps=15,scale=-1:480,crop=400:400;\
[c1]hue=12*t[c1];\
[c2]hue=24*t,scale=iw*.75:-1[c2];\
[c3]hue=48*t,scale=iw*.5:ih*.5[c3];\
[c4]hue=96*t,scale=iw*.25:ih*.25[c4];\
[c1][c2]overlay=main_w/8:main_h/8[c2];\
[c2][c3]overlay=main_w/4:main_h/4[c3];\
[c3][c4]overlay=main_w*.375:main_h*.375;\                
[c4]crop=480:480:375:375" out-video-hue4_simples_crop400.mp4 

==============================================================================
// aplicação de tecnica hue + scaling + fica girando
ffmpeg -i out-video_no-audio.mp4 -filter_complex \
"[0]split=4[c1][c2][c3][c4];\
[c1]hue=12*t,rotate='0.1*PI*t:ow=hypot(iw,ih)+1:oh=ow:c=0x333333'[c1];\
[c2]hue=24*t,scale=iw*.75:-1,rotate='-0.1*PI*t:ow=hypot(iw,ih)+1:oh=ow:c=0x0000FF00'[c2];\
[c3]hue=48*t,scale=iw*.5:ih*.5,rotate='0.1*PI*t:ow=hypot(iw,ih)+1:oh=ow:c=0x0000FF00'[c3];\
[c4]hue=96*t,scale=iw*.25:ih*.25,hflip,rotate='-0.1*PI*t:ow=hypot(iw,ih)+1:oh=ow:c=0x0000FF00'[c4];\
[c1][c2]overlay=main_w/8:main_h/8[c2];\
[c2][c3]overlay=main_w/4:main_h/4[c3];\
[c3][c4]overlay=main_w*.375:main_h*.375[c4];\
[c4]crop=720:720:375:375" \
out-video-hue4.mp4 -y


->
