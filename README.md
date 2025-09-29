# apptainer-freesurfer-freeview

## Environment
- apptainer version 1.4.2
- freesurfer 7.4.1
- Host server(Ubuntu 24.04)
- SSH via MobaXterm on Windows 11

This is just an Apptainer build recipe to allow for the freeview to work in graphics X11.


Idea based off of this pull request:
https://github.com/ReproNim/neurodocker/pull/351/files

Based of this issue:
https://github.com/ReproNim/neurodocker/issues/343


Aside - Not really sure why this was never fully resolved in the base image for docker/apptainer - seems like they want us using the 'neurodesktop' way of setup. Since CentOs is now deprecated, this may only work so long as the vault.centos.org repos are up.


```apptainer build freesurfer.sif freesurfer-7-4-1.def```

`cat freesurfer-7-4-1.def`
```docker
Bootstrap: docker
From: freesurfer/freesurfer:7.4.1

%help
    This container provides FreeSurfer for neuroimaging processing. version 7.4.1

%setup
        cat <<END >${APPTAINER_ROOTFS}/usr/local/freesurfer/.license
<<<CUT AND PASTE LICENSE HERE>>>
END
%post
    cat <<EOF > /etc/yum.repos.d/CentOS-Base.repo
[baseos]
name=CentOS Stream 8 - BaseOS
baseurl=http://vault.centos.org/8-stream/BaseOS/x86_64/os/
gpgcheck=0
enabled=1

[appstream]
name=CentOS Stream 8 - AppStream
baseurl=http://vault.centos.org/8-stream/AppStream/x86_64/os/
gpgcheck=0
enabled=1
[extras]
name=CentOS Stream 8 - Extras
baseurl=http://vault.centos.org/8-stream/extras/x86_64/os/
gpgcheck=0
enabled=1

[extras-common]
name=CentOS Stream 8 - Extras common packages
baseurl=http://vault.centos.org/8-stream/extras/x86_64/extras-common/
gpgcheck=0
enabled=1


EOF

    dnf clean all
    dnf makecache

    dnf -y install epel-release
    dnf remove -y mesa*  # Remove all mesa/GL to start clean
    dnf -y install \
        bc \
        libgomp \
        libXmu \
        libXt \
        tcsh \
        perl \
        qt5-qtdeclarative \
        qt5-qtbase-gui \
        qt5-qtx11extras \
        libXScrnSaver \
        libXft \
        libjpeg-turbo \
        epel-release
    dnf install -y mesa-dri-drivers mesa-libGL mesa-libEGL mesa-libGLU
    dnf clean all

    # Version FreeSurfer
    export FREESURFER_VERSION=7.4.1
%runscript
    exec "$@"
```


Tested commands:   

`apptainer shell --env DISPLAY=$DISPLAY --nvccli --fakeroot freesurfer.sif`   
`apptainer exec freesurfer.sif freeview`

Not entirely sure if we need to pass the display env to the container but tried anyway and seemed okay.
