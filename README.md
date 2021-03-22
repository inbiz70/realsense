1. 패키지 설치
$ sudo apt-get update
$ sudo apt-get upgrade

$ sudo apt-get install git cmake

# libgtk 와 libglfw3 는 테스트 프로그램 빌드시 필요함.
$ sudo apt-get install libusb-1.0-0-dev pkg-config libgtk-3-dev
$ sudo apt-get install libglfw3-dev

# 컴파일러가 설치되어 있지 않다면 설치
$ sudo apt-get install make
$ sudo apt-get install g++ gcc
혹시 gtk 설치 시 아래와 같은 에러가 발생한다면...
dpkg-deb (subprocess): decompressing archive member: lzma error: compressed data is corrupt
dpkg-deb: error: subprocess <decompress> returned error exit status 2
dpkg: error processing archive /tmp/apt-dpkg-install-HsLtVm/42-libcairo-script-interpreter2:
 subprocess dpkg-deb --fsys-tarfile returned error exit status 2

 You might want to run 'apt --fix-broken install' to correct these.
 The following packages have unmet dependencies:
  libcairo2-dev : Depends: libcairo-script-interpreter2 (= 1.14.8-1) but it is not going to d
 E: Unmet dependencies. Try 'apt --fix-broken install' with no packages (or specify a soluti.
다음과 같은 방법으로  libcairo-script-interpreter2를 설치하면 된다.
$ sudo apt-get clean
$ sudo apt-get install libcairo-script-interpreter2


2. 소스 코드 다운로드
$ git clone https://github.com/Maghoumi/librealsense


3. 코드 수정
다운로드한 코드는 32비트 arm 보드에서 빌드한다면 아무런 문제가 없을 것이다.
하지만 64비트 arm 보드라면 다음 소스 파일에서 아래와 같이 define을 추가해줘야 한다.
$ nano src/image.cpp
#include <algorithm>

#define __arm__ //추가!!!

#ifndef __arm__
    #include <tmmintrin.h> // For SSE3 intrinsics used in unpack_yuy2_sse
#else
    inline uint8_t clamp_byte(int v) { return v < 0 ? 0 : v > 255 ? 255 : v; }
#endif
만약 위의 부분이 제대로 수정이 안되었다면
다음과 같은 에러를 보게 될 것이다.
src/image.cpp:14:74: fatal error: tmmintrin.h: No such file or directory
compilation terminated.
Makefile:96: recipe for target 'obj/image.o' failed


4. 빌드
$ cd librealsense
$ make -j4 BACKEND=LIBUVC
$ sudo make install
$ sudo ldconfig
빌드 시 아래와 같은 에러가 발생한다면
Package glu was not found in the pkg-config search path.
Perhaps you should add the directory containing `glu.pc'
to the PKG_CONFIG_PATH environment variable
No package 'glu' found
In file included from examples/c-tutorial-3-pointcloud.c:20:0:
/usr/include/GLFW/glfw3.h:177:22: fatal error: GL/glu.h: No such file or directory
   #include <GL/glu.h>
                      ^
compilation terminated.
In file included from examples/cpp-tutorial-3-pointcloud.cpp:14:0:
/usr/include/GLFW/glfw3.h:177:22: fatal error: GL/glu.h: No such file or directory
   #include <GL/glu.h>
                      ^
compilation terminated.
g++ examples/cpp-multicam.cpp -std=c++11 -Iinclude -Llib -lrealsense -lm `pkg-config --cflags --libsm
Package glu was not found in the pkg-config search path.
Perhaps you should add the directory containing `glu.pc'
to the PKG_CONFIG_PATH environment variable
No package 'glu' found
/tmp/ccMonJLN.o: In function `main':
c-tutorial-2-streams.c:(.text+0x1c0): undefined reference to `glfwInit'
c-tutorial-2-streams.c:(.text+0x1d6): undefined reference to `glfwCreateWindow'
c-tutorial-2-streams.c:(.text+0x1de): undefined reference to `glfwMakeContextCurrent'
c-tutorial-2-streams.c:(.text+0x1e4): undefined reference to `glfwPollEvents'
c-tutorial-2-streams.c:(.text+0x1fc): undefined reference to `glClear'
c-tutorial-2-streams.c:(.text+0x208): undefined reference to `glPixelZoom'
c-tutorial-2-streams.c:(.text+0x214): undefined reference to `glRasterPos2f'
c-tutorial-2-streams.c:(.text+0x240): undefined reference to `glPixelTransferf'
c-tutorial-2-streams.c:(.text+0x26a): undefined reference to `glDrawPixels'
c-tutorial-2-streams.c:(.text+0x27a): undefined reference to `glPixelTransferf'
c-tutorial-2-streams.c:(.text+0x286): undefined reference to `glRasterPos2f'
c-tutorial-2-streams.c:(.text+0x2ac): undefined reference to `glDrawPixels'
c-tutorial-2-streams.c:(.text+0x2bc): undefined reference to `glRasterPos2f'
c-tutorial-2-streams.c:(.text+0x2e2): undefined reference to `glDrawPixels'
c-tutorial-2-streams.c:(.text+0x302): undefined reference to `glRasterPos2f'
c-tutorial-2-streams.c:(.text+0x328): undefined reference to `glDrawPixels'
c-tutorial-2-streams.c:(.text+0x32e): undefined reference to `glfwSwapBuffers'
c-tutorial-2-streams.c:(.text+0x334): undefined reference to `glfwWindowShouldClose'
Makefile:82: recipe for target 'bin/c-tutorial-3-pointcloud' failed
make: *** [bin/c-tutorial-3-pointcloud] Error 1
make: *** Waiting for unfinished jobs....
collect2: error: ld returned 1 exit status
Makefile:85: recipe for target 'bin/cpp-tutorial-3-pointcloud' failed
make: *** [bin/cpp-tutorial-3-pointcloud] Error 1
Makefile:82: recipe for target 'bin/c-tutorial-2-streams' failed
make: *** [bin/c-tutorial-2-streams] Error 1
In file included from examples/example.hpp:5:0,
                 from examples/cpp-multicam.cpp:5:
/usr/include/GLFW/glfw3.h:177:22: fatal error: GL/glu.h: No such file or directory
   #include <GL/glu.h>
                      ^
compilation terminated.
Makefile:85: recipe for target 'bin/cpp-multicam' failed
make: *** [bin/cpp-multicam] Error 1
다음 명령어로 libglu1-mesa-dev을 설치해주자.
$ sudo apt-get install libglu1-mesa-dev

5. 권한 설정
리얼센스를 사용하려면 권한을 설정해줘야 한다.
sudo cp config/99-realsense-libusb.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && udevadm trigger

sudo apt-get install libssl-dev

