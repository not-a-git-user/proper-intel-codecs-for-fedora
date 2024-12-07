# Intel codecs for fedora(41)
Intel codecs are required to be setup from fedora > 40. Here is a guide on how to improve performance in fedora with intel cpu.

This is a guide inspired by [discussion.fedoraproject.org](https://discussion.fedoraproject.org/t/intel-graphics-best-practices-and-settings-for-hardware-acceleration/69944). credits [Irska Lee](https://discussion.fedoraproject.org/u/inslee/summary)
## Step 1. Install RPMFusion repos
```
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
```
```
sudo dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm 
```
## Step 2. Install multimedia packages, Intel tools and other tools
```
sudo dnf4 groupinstall multimedia
```
```
sudo dnf install intel-media-driver libva libva-utils gstreamer1-vaapi ffmpeg intel-gpu-tools mesa-dri-drivers mpv libva-intel-driver libva-intel-hybrid-driver intel-media-driver gstreamer1-vaapi intel_gpu_top gstreamer1-plugins-bad-free gstreamer1-plugins-bad-free-extras vainfo
```

## Step 3. Add environment variable for Libva
Add the following line to your `.bashrc`

```export LIBVA_DRIVER_NAME=iHD```

## Step 4. Enable Intel GuC and HuC and Framebuffer compression
Add kernel parameters to load GuC and HuC (contrary to popular belief, they are not enabled by default except on Intel gen12+ platforms in the kernel) by editing the following:
```
sudo nano /etc/modprobe.d/i915.conf
```
add the following lines for <= 10th gen
```
options i915 enable_guc=2
options i915 enable_fbc=1
```  
or add the following lines for > 10th gen
```
options i915 enable_guc=3
options i915 enable_fbc=1
```  

Then rebuild your intramfs with the following:
```
sudo dracut --force
```
### Then reboot (important)
## Step 5. Make sure everything is working
1. Test VA-API support with `vainfo` and compare against this [ArchWiki section](https://wiki.archlinux.org/title/Hardware_video_acceleration#Verifying_VA-API)

2. Check to make sure GuC is enabled:
   
   `sudo dmesg | grep guc`

   Should return something like `GuC firmware i915/tgl_guc_62.0.0.bin version 62.0`


3. Check to make sure HuC is enabled:
   
   `sudo dmesg | grep huc`

   Should return something like `HuC firmware i915/tgl_huc_7.9.3.bin version 7.9 authenticated:yes`
   
4. Test VA-API support:
    1. Download a test [video file](https://test-videos.co.uk/bigbuckbunny/mp4-h264)
    2. Open a new terminal window and run 
    ```sudo intel_gpu_top```
    3. In a separate terminal window, run ```mpv --hwdec=auto <video file>```
    4. Go back to the intel_gpu_top terminal window and check to make sure the “Video” bar shows activity, this indicates that video is being accelerated properly. If only render/3d is showing activity, you went wrong somewhere and you need to debug it yourself.

Voila, its done, you can now proceed to setup hwaccel for your specific browser
PS: cpupower is another great tool to get every bit of performance from your pc
