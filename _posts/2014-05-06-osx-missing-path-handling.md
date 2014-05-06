---
layout: post
title: opendirectoryd, CMake, and Missing PATHs
---

Recently, running ```cmake``` on my Macbook Pro under Mavericks had been taking extraordinarily long. To configure a moderately-sized project (~3000 source files, 159 CMakeLists.txt files) I had to wait 30 seconds! On the ["CMake Performance Tips"][fast_cmake] page, they mention that configuring [VTK](https://github.com/Kitware/VTK), a very large project, took 14 seconds. Clearly something was wrong.

For me, the problem seemed to mostly manifest itself in the "generating" stage of CMake's build process, which took more than half of that time. I tried profiling ```cmake``` with Instruments, but after about ten seconds, the ```cmake``` process stopped generating any CPU usage. That seemed strange, but indicated that perhaps it wasn't ```cmake``` but some other process that was causing it to run so slowly.

I ran the excellent [htop](http://hisham.hm/htop/) while configuring and found the culprit! ```opendirectoryd``` was consistently above 40% CPU usage during CMake's generation stage. Some searches led me to a related [superuser post](http://superuser.com/questions/350879/opendirectoryd-consumes-40-of-cpu) which indicated that broken symlinks to ```/home``` could cause ```opendirectoryd``` to get busy. I searched my Dropbox for any similar problems, but didn't find anything similar.

However, I do use Dropbox to sync my system configuration files (e.g. .gitconfig, .bashrc) across Mac and Linux systems. Scanning my .bashrc I found some old references to ```/home``` subdirectories in my ```$PATH```. After tidying up my ```$PATH``` by removing the missing directory, I re-ran cmake and it only took 10 seconds. Apparently ```opendirectoryd``` burns a lot of cycles looking for missing directories in your ```$PATH```, so I'm going to add checks throughout my .bashrc to ensure that a directory exists before adding it.


[fast_cmake]: http://www.cmake.org/Wiki/CMake_Performance_Tips


