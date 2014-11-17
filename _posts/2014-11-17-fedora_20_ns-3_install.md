---
layout: post
title: "NS-3仿真平台的搭建"
description: "Fedora20环境下NS-3仿真平台的完整搭建"
category: NS-3
tags: [NS-3, 网络仿真, Linux, fedora]
---
{% include JB/setup %}
Fedora20 下 NS-3 安装及环境配置

* 1、基本库配置:

控制台执行以下命令:([参考链接](http://www.nsnam.org/wiki/Installation))

    sudo yum install gcc gcc-c++ python python-devel mercurial bzr gsl gsl-devel gtk2 gtk2-devel gdb valgrind doxygen graphviz ImageMagick python-sphinx dia texlive texlive-latex flex bison tcpdump sqlite sqlite-devel libxml2 libxml2-devel uncrustify openmpi openmpi-devel boost-devel cmake glibc-devel.i686 glibc-devel.x86_64 -y

***

* 2、下载BAKE（BAKE方式安装NS-3）

控制台在选择的目录执行以下脚本

    hg clone http://code.nsnam.org/bake
    BAKE_HOME=`pwd`/bake
    export PATH=$PATH:$BAKE_HOME
    export PYTHONPATH=$PYTHONPATH:$BAKE_HOME

然后执行

    bake.py check
    
得到执行的结果:

>  \> Python - OK   
   \> GNU C++ compiler - OK   
   \> Mercurial - OK   
   \> CVS - OK   
   \> GIT - OK   
   \> Bazaar - OK   
   \> Tar tool - OK   
   \> Unzip tool - OK   
   \> Unrar tool - OK   
   \> 7z  data compression utility - OK   
   \> XZ data compression utility - OK   
   \> Make - OK   
   \> cMake - OK   
   \> patch tool - OK   
   \> autoreconf tool - OK   
   \> Path searched for tools: /usr/lib64/qt-3.3/bin   
   \> /usr/lib64/ccache /usr/local/bin  /usr/bin/bin/usr/local/sbin /usr/sbin   
   \> /sbin /user/dcamara/home/scripts/user/dcamara/home/INRIA/Programs/bin   
   \> /user/dcamara/home/INRIA/repos/llvm/build/Debug+Asserts/bin   

如果有任何一个不是OK 使用yum安装对应的程序包。   

我这里缺少的是Unrar tool。   
Unrar tool 安装方法:

    sudo rpm -Uvh http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-stable.noarch.rpm
    sudo rpm -Uvh http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-stable.noarch.rpm
    sudo yum install unrar -y
    
* 3、设置下载版本:  

下面的命令是下载最新开发版本，如果下载其他版本可以使用ns-3.xx：

    bake.py configure -e ns-3-dev   
    bake.py show    
    
根据需要,参考输出的结果安装缺少的包。  
如果提示Pygraphviz  这个包提示的命令错误.可以执行

    sudo yum install python-devel gnome-python2 gnome-python2-gnomedesktop gnome-python2-rsvg graphviz-python pygoocanvas python-kiwi -y

最终bake.py show 输出结果为:

> [whimsyduke@localhost NS-3]$  bake.py show     

> module: gccxml-ns3 (enabled)   
　No dependencies!   
module: python-dev (enabled)   
　No dependencies!   
module: pygraphviz (enabled)   
　No dependencies!   
module: pygoocanvas (enabled)   
　No dependencies!   
module: g++ (enabled)   
　No dependencies!   
module: qt4 (enabled)   
　No dependencies!   
module: pygccxml (enabled)   
　depends on:   
　　gccxml-ns3 (optional:False)   
module: pyviz-prerequisites (enabled)   
　depends on:   
　　python-dev (optional:True)   
　　pygraphviz (optional:True)   
　　pygoocanvas (optional:True)   
module: netanim (enabled)   
　depends on:   
　　qt4 (optional:False)   
　　g++ (optional:False)   
module: pybindgen (enabled)   
　depends on:   
　　pygccxml (optional:True)   
　　python-dev (optional:True)   
module: ns-3-dev (enabled)   
　depends on:   
　　netanim (optional:True)   
　　pybindgen (optional:True)   
　　pyviz-prerequisites (optional:True)   
   
> -- System Dependencies --   
>  \> g++ - OK      
   \> pygoocanvas - OK   
   \> pygraphviz - OK   
   \> python-dev - OK   
   \> qt4 - OK   
 
确认后执行:   

    bake.py deploy
  
这样等待下载完成，要很久。
当然你也可以直接下载整合包或者其他方式下载。
***

* 4、NS-3配置

然后进入下载目录 */source/ns-3-dev*   
执行      

    ./waf configure    
    
输出结果是ns-3的配置。    

下面一一说明缺少的模块如何添加：   

* 4.1如提示:**Python API Scanning Support   : not enabled (Missing 'pygccxml' Python module)**   

说明缺少 **pygccxml** 模块。而实际上相关代码已经下载到*/source/pygccxml*下,可以看README来了解安装方法.    
进入*/source/pygccxml*目录下,执行

    sudo python setup.py install 

来安装。   
再次执行      

    ./waf configure    
    
输出结果提示仍然缺少**module:Python API Scanning Support   : not enabled (gccxml missing)**

继续安装**gccxml** 
进入*/source/gccxml*目录,执行

    cmake ../gccxml -DCMAKE_INSTALL_PREFIX:PATH=/installation/path
    make
    sudo make install

提示仍然缺少**module:Python API Scanning Support   : not enabled (gccxml missing)** ,原因是缺少gccxml包,执行安装:

    sudo yum install gccxml -y

再次执行 

    ./waf configure 

显示**Python API Scanning Support   : enabled **，解决。

* 4.2如提示：**BRITE Integration             : not enabled (BRITE not enabled (see option --with-brite))**

解决方法：[参考链接](http://www.nsnam.org/wiki/BRITE_integration_with_ns-3)
 
由于相关程序需要额外下载，这里选择下载到*/source/other*目录来放置相关的程序.

进入*/source/other*

    hg clone http://code.nsnam.org/jpelkey3/BRITE
    cd BRITE
    make

回到*/source/ns-3-dev*，执行 (我的source在/NS-3下)

    ./waf configure  --with-brite=/NS-3/NS-3/ns-3-dev/source/other/BRITE
    


输出**BRITE Integration             : enabled**,问题解决

* 4.3如提示:**NS-3 Click Integration        : not enabled (nsclick not enabled (see option --with-nsclick))**

解决方法：[参考链接](http://www.read.cs.ucla.edu/click/nsclick)

在参考链接下载*click-2.0.1.tar.gz*和*click-packages-2.0.tar.gz*    
新建*/source/other/CLICK*目录,将这两个压缩包放入并解压。  
进入*/source/other/CLICK/click-2.0.1*，执行

    ./configure --enable-userlevel --disable-linuxmodule --enable-nsclick
    make

回到*/source/ns-3-dev*,执行

    ./waf configure   --with-nsclick=/NS-3/NS-3/ns-3-dev/source/other/CLICK/click-2.0.1

输出**NS-3 Click Integration        : enabled**，问题解决。

* 4.4、如提示:**PlanetLab FdNetDevice         : not enabled (PlanetLab operating system not detected (see option --force-planetlab))**

**waf**加上参数**--force-planetlab**即可   

    ./waf configure --force-planetlab 

输出**PlanetLab FdNetDevice         : enabled**，问题解决。

* 4.5、如提示:**Network Simulation Cradle     : not enabled (NSC not found (see option --with-nsc))**

解决方法：[参考链接](http://research.wand.net.nz/software/nsc.php)   
新建目录*/source/other/NSC*,进入执行

    hg clone https://secure.wand.net.nz/mercurial/nsc
    cd nsc
    python scons.py

如果提示**Did not find libfl.a and/or flex. Please install flex and its development libraries.**，执行

    sudo yum install flex-devel flexiport  flexiport-devel -y

然后再次执行

    python scons.py

最终提示scons: done building targets. 就编译成功。

回到*/source/ns-3-dev*,执行

    ./waf configure --with-nsc=/NS-3/NS-3/ns-3-dev/source/other/NSC/nsc

输出**Network Simulation Cradle     : enabled**，问题解决。

* 4.6、如提示:**MPI Support                   : not enabled (option --enable-mpi not selected)**

解决方法：[参考链接](http://www.open-mpi.org/software/ompi/v1.8/)
*waf*参数加上**--enable-mpi**，即执行

    ./waf configure --enable-mpi

输出**MPI Support                   : not enabled (mpic++ not found)**，这是提示缺少mpic++库.

打开[http://www.open-mpi.org/software/ompi/v1.8/](http://www.open-mpi.org/software/ompi/v1.8/),下载**openmpi**,新建目录*/source/other/OPNMPI*,将下载tar包移入目录并解压.执行

    ./configure
    make
    sudo make install

再次回到*/source/ns-3-dev*,执行

    ./waf configure --enable-mpi

输出:**MPI Support                   : enabled**，问题解决。

* 4.7、如提示:**NS-3 OpenFlow Integration     : not enabled (OpenFlow not enabled (see option --with-openflow))**

解决方法:[参考链接](http://www.nsnam.org/docs/release/3.21/models/html/openflow-switch.html)

**openflow** 需要安装 **OFSID**库.
新建//source/other/OFSID目录,进入执行

    hg clone http://code.nsnam.org/openflow
    ./waf configure
    ./waf build

然后**waf**参数加上**--with-openflow=/NS-3/NS-3/ns-3-dev/source/other/OFSID/openflow**，回到/source/ns-3-dev,执行

    ./waf configure --with-openflow=/NS-3/NS-3/ns-3-dev/source/other/OFSID/openflow

输出:**NS-3 OpenFlow Integration     : enabled**，问题解决。

* 4.8、如提示:**PyViz visualizer              : not enabled (Missing python modules: pygraphviz)**

解决方案：[参考链接](http://www.nsnam.org/wiki/PyViz)

需要安装**PyGraphviz**,新建目录*/source/other/PYGRAPHVIZ*执行

    git clone https://github.com/pygraphviz/pygraphviz.git
    cd pygraphviz
    chmod +x setup.py
    ./setup.py

安装报错,提示需要安装**Graphviz**，执行

    sudo yum install graphviz-devel -y

再次执行

    ./setup.py

回到*/source/ns-3-dev*,执行

    ./waf configure

输出**PyViz visualizer              : enabled**，问题解决。
***

最终参考以上参数及其他辅助参数
*/source/ns-3-dev*,下执行

        ./waf configure --enable-examples --enable-tests --force-planetlab --enable-sudo --enable-mpi --with-brite=/NS-3/NS-3/ns-3-dev/source/other/BRITE/BRITE --with-nsclick=/NS-3/NS-3/ns-3-dev/source/other/CLICK/click-2.0.1 --with-nsc=/NS-3/NS-3/ns-3-dev/source/other/NSC/nsc --with-openflow=/NS-3/NS-3/ns-3-dev/source/other/OFSID/openflow

输出

>Setting top to                           : /NS-3/NS-3/ns-3-dev/source/ns-3-dev    
Setting out to                           : /NS-3/NS-3/ns-3-dev/source/ns-3-dev/build    
Checking for 'gcc' (c compiler)          : /usr/bin/gcc    
Checking for cc version                  : 4.8.3    
Checking for 'g++' (c++ compiler)        : /usr/bin/g++    
Checking for compilation flag -Wl,--soname=foo... support : ok    
Checking for program python                               : /usr/bin/python    
Checking for python version                               : (2, 7, 5, 'final', 0)    
Checking for library python2.7 in LIBDIR                  : yes    
Checking for program /usr/bin/python-config,python2.7-config,python-config-2.7,python2.7m-config : /usr/bin/python-config    
Checking for header Python.h                                                                     : yes    
Checking for compilation flag -fvisibility=hidden... support                                     : ok    
Checking for compilation flag -Wno-array-bounds... support                                       : ok    
Checking for pybindgen location                                                                  : ../pybindgen (guessed)    
Python module pybindgen                                                                          : 0.17.0.886    
Checking for pybindgen version                                                                   : 0.17.0.886    
Checking for types uint64_t and unsigned long equivalence                                        : yes    
Checking for types uint64_t and unsigned long long equivalence                                   : no    
Checking for the apidefs that can be used for Python bindings                                    : gcc-LP64    
Checking for internal GCC cxxabi                                                                 : complete    
Python module pygccxml                                                                           : 1.0.0    
Checking for pygccxml version                                                                    : 1.0.0    
Checking for program gccxml                                                                      : /usr/bin/gccxml    
Checking for gccxml version                                                                      : 0.9.0    
Checking boost includes                                                                          : 1_54    
Checking boost libs                                                                              : ok    
Checking for boost linkage                                                                       : ok    
Checking BRITE location                                                                          : /NS-3/NS-3/ns-3-dev/source/other/BRITE (given)    
Checking for library dl                                                                          : yes    
Checking for library brite                                                                       : yes    
Checking for libnsclick.so location                                                              : /NS-3/NS-3/ns-3-dev/source/other/CLICK/click-2.0.1 (given)    
Checking for library dl                                                                          : yes    
Checking for library nsclick                                                                     : yes    
Checking for program pkg-config                                                                  : /usr/bin/pkg-config    
Checking for 'gtk+-2.0' >= 2.12                                                                  : yes    
Checking for 'libxml-2.0' >= 2.7                                                                 : yes    
Checking for type uint128_t                                                                      : not found    
Checking for type __uint128_t                                                                    : yes    
Checking high precision implementation                                                           : 128-bit integer (default)    
Checking for header stdint.h                                                                     : yes    
Checking for header inttypes.h                                                                   : yes    
Checking for header sys/inttypes.h                                                               : not found    
Checking for header sys/types.h                                                                  : yes    
Checking for header sys/stat.h                                                                   : yes    
Checking for header dirent.h                                                                     : yes    
Checking for header stdlib.h                                                                     : yes    
Checking for header signal.h                                                                     : yes    
Checking for header pthread.h                                                                    : yes    
Checking for header stdint.h                                                                     : yes    
Checking for header inttypes.h                                                                   : yes    
Checking for header sys/inttypes.h                                                               : not found    
Checking for library rt                                                                          : yes    
Checking for header sys/ioctl.h                                                                  : yes    
Checking for header net/if.h                                                                     : yes    
Checking for header net/ethernet.h                                                               : yes    
Checking for header linux/if_tun.h                                                               : yes    
Checking for header netpacket/packet.h                                                           : yes    
Checking for NSC location                                                                        : /NS-3/NS-3/ns-3-dev/source/other/NSC/nsc (given)    
Checking for library dl                                                                          : yes    
Checking for NSC supported architecture x86_64                                                   : ok    
Checking for 'mpic++'                                                                            : yes    
Checking for OpenFlow location                                                                   : /NS-3/NS-3/ns-3-dev/source/other/OFSID/openflow (given)    
Checking for library dl                                                                          : yes    
Checking for library xml2                                                                        : yes    
Checking for library openflow                                                                    : yes    
Checking for 'sqlite3'                                                                           : yes    
Checking for header linux/if_tun.h                                                               : yes    
Python module gtk                                                                                : ok    
Python module goocanvas                                                                          : 0.14.1    
Python module pygraphviz                                                                         : 1.3rc2    
Checking for program sudo                                                                        : /usr/bin/sudo    
Checking for program valgrind                                                                    : /usr/bin/valgrind    
Checking for 'gsl'                                                                               : yes    
Checking for compilation flag -Wno-error=deprecated-d... support                                 : ok    
Checking for compilation flag -Wno-error=deprecated-d... support                                 : ok    
Checking for compilation flag -fstrict-aliasing... support                                       : ok    
Checking for compilation flag -fstrict-aliasing... support                                       : ok    
Checking for compilation flag -Wstrict-aliasing... support                                       : ok    
Checking for compilation flag -Wstrict-aliasing... support                                       : ok    
Checking for program doxygen                                                                     : /usr/bin/doxygen    
---- Summary of optional NS-3 features:    
Build profile                 : debug    
Build directory               : /NS-3/NS-3/ns-3-dev/source/ns-3-dev/build    
Python Bindings               : enabled    
Python API Scanning Support   : enabled    
BRITE Integration             : enabled    
NS-3 Click Integration        : enabled    
GtkConfigStore                : enabled    
XmlIo                         : enabled    
Threading Primitives          : enabled    
Real Time Simulator           : enabled    
File descriptor NetDevice     : enabled    
Tap FdNetDevice               : enabled    
Emulation FdNetDevice         : enabled    
PlanetLab FdNetDevice         : enabled    
Network Simulation Cradle     : enabled    
MPI Support                   : enabled    
NS-3 OpenFlow Integration     : enabled    
SQlite stats data output      : enabled    
Tap Bridge                    : enabled    
PyViz visualizer              : enabled    
Use sudo to set suid bit      : enabled    
Build tests                   : enabled    
Build examples                : enabled    
GNU Scientific Library (GSL)  : enabled    
'configure' finished successfully (4.323s)    

然后执行

    ./waf

编译
***

此外，使用

    ./waf docs
    
可以生成api文档等文档，不过因为缺软件包件会提示:

>/bin/sh: epstopdf: 未找到命令    
make: \*\*\* [source-temp/figures/software-organization.pdf] 错误 127    
epstopdf dcf-overview.eps    
/bin/sh: epstopdf: 未找到命令    
make: \*\*\* [source-temp/figures/dcf-overview.pdf] 错误 127    
epstopdf dcf-overview-with-aggregation.eps    
/bin/sh: epstopdf: 未找到命令    
make: \*\*\* [source-temp/figures/dcf-overview-with-aggregation.pdf] 错误 127    
make: 由于错误目标“html”并未重新创建。    
make: 由于错误目标“singlehtml”并未重新创建。    
make: 由于错误目标“latexpdf”并未重新创建。    

需要安装**texlive-epspdf**相关软件包。

或许还要执行

    sudo yum install doxygen-latex texlive-titlesec texlive-frame texlive-framed texlive-threeparttable texlive-wrapfig texlive-upquote -y

来安装软件包。

其他可能在NS-3中需要用到的软件:
kiwi    
tcpdump    
wireshark    
sqlite3    
gnuplot    
SciPy    
NetCDF    
Matplotlib    
Python Imaging Library    
