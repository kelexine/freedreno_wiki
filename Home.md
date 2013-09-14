# Welcome to the freedreno wiki!

### Technical Information:
* [[Reverse Engineering Tools|Reverse-Engineering-Tools]]
* [[Useful Information|Useful-Information]]
* [[Command Stream Format|Command-Stream-Format]]
   * The basic packet format (described in this page) is same for a3xx and a2xx although all the registers and some of the packet types differ.
* [[Adreno Tiling|Adreno-Tiling]] - how tiling works on adreno (a2xx and a3xx)
* [[A2xx Shader Instruction Set Architecture|A2XX-Shader-Instruction-Set-Architecture]]
* [[A3xx Shader Instruction Set Architecture|A3XX-Shader-Instruction-Set-Architecture]]
* [[Frequently Asked Questions|FAQ]]

### Status:
apps/games/etc that are known to work or not

#### Adreno 3xx
<table>
 <tr><th>App</th><th>Status</th></tr>
 <tr>
    <th>gnome-shell</th>
    <td>no known issues</td>
 </tr>
 <tr>
   <th>compiz</th>
   <td>should work.. not all plugins tested</td>
 </tr>
 <tr>
   <th>xbmc</th>
   <td>no known issues, good performance at 1080p, sw decode h264 seems fast enough up to 720p content</td>
 </tr>
 <tr>
   <th>xonotic-glx</th>
   <td>no known issues (15-20fps @720p)</td>
 </tr>
 <tr>
   <th>vdrift</th>
   <td>no known rendering issues.. but slow</td>
 </tr>
 <tr>
   <th>darkplaces, openarena, etc<br>(others based on q3 engine)</td>
   <td>no known issues (60fps, vsync limited @720p)</th>
 <tr>
   <th>etuxracer</th>
   <td>works except msaa (~20-30fps w/ bo-cache)</td>
 </tr>
 <tr>
   <th>supertuxkart</th>
   <td>works except msaa (~20-30fps w/ bo-cache)</td>
 </tr>
</table>
NOTES:
* fps figures are with msm drm/kms driver (otherwise we can't pageflip and have gpu stall on presentation blit)
* etuxracer and supertuxkart, in particular, trigger creation of 150-200 transient buffer objects (bo's) per frame, and heavily benefit (ie. double framerate) from caching and re-using bo's.  There is a work-in-progress patch for this in libdrm_freedreno, but will need to add an MADVISE type kernel API to do this properly.
* etuxracer and supertuxkart use an old trick using alpha-test for transparency (rather than enabling GL_BLEND).  In some cases, they use an alpha-to-coverage trick, which requires at least MSAA 2x, which is not implemented yet.  These textures will appear opaque.  (In some ways, it is an application bug, since it should handle the non-MSAA case.)
* the games with lower framerates tend to suffer due to heavy vertex shader workload because binning-pass is not implemented yet.  So vertex shaders are executed for each (approx) 256x256 tile.  This matters less for apps like window managers or xbmc, which would benefit more from compiler optimizations.

#### Adreno 2xx

### Devices: 
* [[HP TouchPad|HP-TouchPad]]
* [[Galaxy S3 LTE|Samsung-Galaxy-S-III-(LTE)]]
* [[Nexus 4|Nexus-4]]
* [[ifc6410|Ifc6410]]

NOTE: please feel free to make updates/additions to the wiki, add pages for your particular device that you have (or are trying to use) freedreno on, etc.  A wiki is a community effort.  Don't vandalize.  If you have a question, ask on #freedreno IRC channel on freenode.
