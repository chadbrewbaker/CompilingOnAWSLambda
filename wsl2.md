# Compiling for Amazonlinux2 lambda instances on Windows 10
With Apple Silicon, at least for the near future Mac will be king of ARM development and Windows (with new Linux powers) will be king of x86 development. This memorializes the steps I went through to get my Windows 10 laptop ready for native binary development for the latest runtimes of AWS Lambda like Python3.8 that run on AmazonLinux2.

The mainstream landscape is choosing among: WSL, WSL2, Docker (on WSL or WSL2), Hyperv, VMWare, and Virtualbox.
* LambCI extracts AWS Lambda system binaries and packages them in a Docker container. https://github.com/lambci hackish, but slick and annointed by Amazon itself.
* There is the https://github.com/yuk7/wsldl project that allowed the creation of a bespoke AmazonLinux2 image for WSL, https://github.com/yosukes-dev/AmazonWSL , just like Ubuntu20.24.
* You should be able to cross compile from the stock Ubuntu20.24 or Debian WSL. This would involve grabbing a LambCI image, (or using their script to get your own Lambda binaries) and pointing the build chain at the binaries. This may be your path of choice if you have a Linux application with a curmudgonly build chain that demands a specific GCC or LLVM environment. 


## Upgrading to WSL2
Windows Subsytem for Linux 2 gets rid of the translation layer and replaces it with more Linux kernel code. AFAIK from what I have read this creates much better performance. Docker now has support for this which makes running Docker on Windows 10 more performant.

### Issue 1: Upgrading to Win 10 v 2004 release.
On my system there was a blocker that my Nvidia drivers had to be manually upgraded before Microsoft would allow the update to avoid a blue screen of death:
https://docs.microsoft.com/en-us/windows/release-information/status-windows-10-2004
