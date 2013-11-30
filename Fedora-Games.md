### Stuff in Fedora Repos
What I install to have a bunch of games to test/debug with:

    sudo dnf install vegastrike sumwars ember xonotic supertuxkart extremetuxracer alienarena bzflag maniadrive neverball tremulous

note that some have very large (~1GiB) data files.  Check [[Status|Status]] to see what actually works.

### XBMC

Since it is not yet in rpmfusion for ARM:

    yum-config-manager --add-repo http://people.freedesktop.org/~robclark/xbmc-repo/xbmc.repo
    yum install xbmc

### Misc
In theory, Aleph One engine + marathon from here:

    http://juanmabc.fedorapeople.org/packages/alephone/alephone-1.0.0.20120514-1.fc18.src.rpm
     -> rpmbuild --rebuild alephone-1.0.0.20120514-1.fc18.src.rpm
    http://alephone.cebix.net/downloads/AlephOne-Infinity-1.0-1.noarch.rpm
    http://alephone.cebix.net/downloads/AlephOne-Marathon2-1.0-1.noarch.rpm
    http://alephone.cebix.net/downloads/AlephOne-M1A1-1.0-1.noarch.rpm

but seems to be missing libmad-devel and smpeg-devel (or at least rpmfusion seems to be completely missing for f20).
