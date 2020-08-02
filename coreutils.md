AmazonLinux2 has old versions of several GNU utilities. I also wanted to write a test harness for https://github.com/chadbrewbaker/coreutils that compared new Rust versions of core utilities with the latest builds of GNU Coreutils - and ran the GNU Coreutil test suite on the new Rust binaries.

```bash
yum install -y gperf texinfo rsync help2man
```

Make a directory to house nightly builds of all the things and source it
```bash
mkdir /usr/nightly
cat ~/.bashrc_nightly
#PATH=/usr/nightly/bin:$PATH:/usr/local/bin
#export PATH
source  ~/.bashrc_nightly
```

GNU Coreutils complains about not having fresh automake.
```bash
git clone git://git.savannah.gnu.org/automake.git
cd automake/
./bootstrap
./configure --prefix=/usr/nightly
make
make install
source  ~/.bashrc_nightly
which automake
```

GNU Coreutils complains about not haveing fresh texinfo.
```bash
git clone https://git.savannah.gnu.org/git/texinfo.git
cd texinfo
bash autogen.sh
./configure --prefix=/usr/nightly
make
make install
source  ~/.bashrc_nightly
which texinfo
```

```bash 
git clone https://git.savannah.gnu.org/git/coreutils.git
cd coreutils
./bootstrap
export FORCE_UNSAFE_CONFIGURE=1 # Because the stock image runs as root
./configure --prefix=/usr/nightly
#./bootstrap #Because the README-relase told me to, not sure if needed?
grep -n Werror ./Makefile #Nerf that line to turn off build warnings as errors
make -j4
make check
# You need to run as non-root to get at a lot of the UNIX permission tests
#  make check-very-expensive   # To run the more expensive tests
make install
```







