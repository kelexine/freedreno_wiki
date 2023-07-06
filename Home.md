# Welcome to the freedreno wiki!

### Technical Information:
* [[Git Trees & Branches|Git-Trees-&-Branches]]
* [[Freedreno Architecture|Architecture]]
* GPU Docs:
  * [[Reverse Engineering Tools|Reverse-Engineering-Tools]]
  * [[Useful Information|Useful-Information]]
  * [[Command Stream Format|Command-Stream-Format]]
     * The basic packet format (described in this page) is same for a3xx and a2xx although all the registers and some of the packet types differ.
  * [[Adreno Tiling|Adreno-Tiling]] - how tiling works on adreno (a2xx and a3xx)
  * [[A2xx Shader Instruction Set Architecture|A2XX-Shader-Instruction-Set-Architecture]]
  * [[A3xx Shader Instruction Set Architecture|A3XX-Shader-Instruction-Set-Architecture]]
  * [Gallium Docs](http://gallium.readthedocs.org/en/latest/)
  * [[kgsl kernel driver|kgsl-kernel]]
* [[Setting up fedora filesystem|Fedora]]
* [[Frequently Asked Questions|FAQ]]
* [[TODO|TODO]]

### Status:
See [[Status|Status]].

### Distros:
Any distro which has a relatively recent mesa, and which packages [xf86-video-freedreno](http://cgit.freedesktop.org/xorg/driver/xf86-video-freedreno/) should work.  Please feel free to add more, update status, add links to instructions, etc.
* Fedora
  * status: mesa 10.2 and freedreno ddx in rawhide, 
  * [[instructions|Fedora]]
* Linaro/Ubuntu
  * status: mesa 10.2.1 and freedreno ddx
  * [release-notes](http://releases.linaro.org/14.06/ubuntu/ifc6410)
  * [instructions](https://wiki.linaro.org/Boards/IFC6410)
* Gentoo
* [[Arch|Arch]]
* [postmarketOS](https://postmarketos.org) (Alpine Linux based)
  * status: mesa 23.0.4 and freedreno ddx (see current version [here](https://pkgs.alpinelinux.org/packages?name=mesa-dri-freedreno))
  * [asus-flo](https://wiki.postmarketos.org/wiki/Google_Nexus_7_2013_(asus-flo)) and [sony-castor](https://wiki.postmarketos.org/wiki/Sony_Xperia_Z2_Tablet_(sony-castor-windy)) have at least limited freedreno support
  * see also: [devices](https://wiki.postmarketos.org/wiki/Devices), [porting guide](https://wiki.postmarketos.org/wiki/Porting_to_a_new_device), [freedreno related wiki content](https://wiki.postmarketos.org/index.php?search=freedreno)

### Devices: 
* Phones/Tablets:
  * [[HP TouchPad|HP-TouchPad]] (a2xx)
  * [[Galaxy S3 LTE|Samsung-Galaxy-S-III-(LTE)]] (a2xx)
  * [[Nexus 4|Nexus-4]] (a3xx)
  * [[Nexus 7 Flo|Nexus-7-Flo]] (a3xx)
* ARM boards:
  * [[ifc6410|Inforce-6410-Plus]] (a3xx)
  * [[ifc6540|Inforce-6540]] (a4xx)
  * [[bSTem|bStem]] (a3xx)
  * [[apq8074 dragonboard|apq8074dragonboard]] (a3xx)

### Contact:
 * mailing list: [freedreno@lists.freedesktop.org](http://lists.freedesktop.org/mailman/listinfo/freedreno)
 * IRC: #freedreno (on freenode)

NOTE: please feel free to make updates/additions to the wiki, add pages for your particular device that you have (or are trying to use) freedreno on, etc.  A wiki is a community effort.  Don't vandalize.  If you have a question, ask on #freedreno IRC channel on freenode.
