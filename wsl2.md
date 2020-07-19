# Compiling for Amazonlinux2 lambda instances on Windows 10
With Apple Silicon, at least for the near future Mac will be king of ARM development and Windows (with new Linux powers) will be king of x86 development. This memorializes the steps I went through to get my Windows 10 laptop ready for native binary development for the latest runtimes of AWS Lambda like Python3.8 that run on AmazonLinux2.

The mainstream landscape is choosing among: WSL, WSL2, Docker (on WSL or WSL2), Hyperv, VMWare, and Virtualbox.
* LambCI extracts AWS Lambda system binaries and packages them in a Docker container. https://github.com/lambci hackish, but slick and annointed by Amazon itself.
* There is the https://github.com/yuk7/wsldl project that allowed the creation of a bespoke AmazonLinux2 image for WSL, https://github.com/yosukes-dev/AmazonWSL , just like Ubuntu20.24.
* You should be able to cross compile from the stock Ubuntu20.24 or Debian WSL. This would involve grabbing a LambCI image, (or using their script to get your own Lambda binaries) and pointing the build chain at the binaries. This may be your path of choice if you have a Linux application with a curmudgonly build chain that demands a specific GCC or LLVM environment. 

## Docker
Just go to https://hub.docker.com/_/amazonlinux/. 

## Using AmazonLinux2 for WSL
I'll have to benchmark later, but you would think running a WSL distro of AmazonLinux2 would be more performant/ease of use than running it under Docker on WSL2. Download and unzip from the above link from yousukes-dev.

```bash
yum update
yum groupinstall "Development Tools"
# Hit yes a few times...
gcc --version
#gcc (GCC) 7.3.1 20180712 (Red Hat 7.3.1-9)
```
That seems consistant with the release notes: https://aws.amazon.com/amazon-linux-2/release-notes/

Not going to deal with .Net Core for now, but Rust/LLVM , python3.8, and java and are popular toolchains.
```bash
 amazon-linux-extras install python3.8
 amazon-linux-extras install java-openjdk11
```
On to shaving the .Net Core yak. As an aside for Powershell lovers, it's now a thing inside Lambda: https://rollingwebsphere.home.blog/2020/01/18/aws-lambda-functions-with-modular-powershell/

Microsoft has a page on installing .Net Core on Linux: https://docs.microsoft.com/en-us/dotnet/core/install/linux.
Sadly no AmazonLinux2 packages, but many claim success running the CentOS binaries.

Microsoft hosts it's own latest generic linux build: https://dotnet.microsoft.com/download/dotnet-core/3.1

Taking a look at how LambCI does it was enlightening: https://github.com/lambci/docker-lambda/blob/master/dotnetcore3.1/build/Dockerfile
Mix of dowloading binaries extracted from a fresh Lambda and YOLO curl piped to bash.
```bash
DOTNET_ROOT=/var/lang/bin
ENV PATH=/root/.dotnet/tools:$DOTNET_ROOT:$PATH \
    LD_LIBRARY_PATH=/var/lang/lib:$LD_LIBRARY_PATH \
    AWS_EXECUTION_ENV=AWS_Lambda_dotnetcore3.1 \
    DOTNET_SDK_VERSION=3.1.301 \
curl https://lambci.s3.amazonaws.com/fs/dotnetcore3.1.tgz | tar -zx -C / && \
curl -L https://dot.net/v1/dotnet-install.sh | bash -s -- -v $DOTNET_SDK_VERSION -i $DOTNET_ROOT
```
I learned a new linux trick in Microsoft's dotnet-install script.
```bash
cat /etc/os-release
#NAME="Amazon Linux"
#VERSION="2"
#ID="amzn"
#ID_LIKE="centos rhel fedora"
#VERSION_ID="2"
#PRETTY_NAME="Amazon Linux 2"
#ANSI_COLOR="0;33"
#CPE_NAME="cpe:2.3:o:amazon:amazon_linux:2"
#HOME_URL="https://amazonlinux.com/"
```
Looks like the Microsoft script on AwsLinux2 installs their centos dotnet 3.1 packages, so this is the "standard" way to do it.


## Upgrading to WSL2
Windows Subsytem for Linux 2 gets rid of the translation layer and replaces it with more Linux kernel code. AFAIK from what I have read this creates much better performance. Docker now has support for this which makes running Docker on Windows 10 more performant.

### Issue 1: Upgrading to Win 10 v 2004 release.
On my system there was a blocker that my Nvidia drivers had to be manually upgraded before Microsoft would allow the update to avoid a blue screen of death:
https://docs.microsoft.com/en-us/windows/release-information/status-windows-10-2004

### Issue 2: Both Windows Update and Windows 10 Update Assistant crash
Rather than attempting to clear the download cache I downloaded a fresh ISO. Absurd, but you have to spoof a tablet browser in Chrome so that Microsoft will give you a temporary download link: https://www.bleepingcomputer.com/news/microsoft/how-to-download-the-windows-10-2004-iso-from-microsoft/

### Issue 3: Enabling WSL2
Instructions here: https://docs.microsoft.com/en-us/windows/wsl/install-win10

Now you are ready to install Docker Desktop and run it on WSL2. Instructions here: https://docs.docker.com/docker-for-windows/wsl/

### Updating all your WSL images
```bash
wsl -l -v
#NAME                   STATE           VERSION
#*  Ubuntu                Stopped         1
#  Ubuntu-20.04           Stopped         1
#  Debian                 Stopped         1
#  docker-desktop-data    Running         2
#  Ubuntu-18.04           Stopped         1
#  docker-desktop         Running         2
#  Amazon2                Stopped         1
```
Now run this to update each version to version 2.
```bash
wsl --set-version (distro name) 2
```

## Setting up an AmazonLinux2 container for compiling
Amazon's repository is still stuck on CMAKE version 2.x.
```bash
wget https://github.com/Kitware/CMake/releases/download/v3.18.0/cmake-3.18.0.tar.gz
tar -xvf cmake-3.18.0.tar.gz
cd cmake-3.18.0
./bootstrap ./bootstrap # --prefix=/usr/local should you want a different directory
make -j6
make install #update ~/.bashrc if required to add in path
```


### Musings on container compositionality/reproducibility
* When running Docker build, have the script store the yum/apt cache, downloaded git repos, and downloaded tarballs to an external zipfile. Then have a build that pulls from the zipfile. Voila, reproducible Docker build with tracable artifacts.
* instead of sudo make install, think about wrapping into an RPM and using the package manager - I need to try this.
 



