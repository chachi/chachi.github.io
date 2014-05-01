---
layout: post
title: Using CMake to get around file size limits on Github
---

I recently ran up against Github's [hard limit on file sizes][github_limits]. They put a hard limit of 100 MB for individual files and a per-repository limit of 1GB per repository. The overall cap is understandable, but I think the 100MB file limit is too restrictive.

I'm aware that Git handles large files [poorly][large_files], but that's only if the files are changing. In my case, I was setting up a fork of the NGA's [GEOTRANS][geotrans] library. GEOTRANS has a single 150 MB data file that contains all the coordinate information needed to convert between 25 different coordinate systems. It's a small price to pay for the services of this library. After getting my first push rejected by Github's servers, I had to go back and find a workaround for the single large file.

I decided to split the file into 50MB chunks, commit those to the repository, and use CMake to rejoin the files at build time. CMake's great ```add_custom_command``` and ```add_custom_target``` functions let you define custom build steps and targets to do work other than assembling libraries and executables.

Here's the simple solution in CMake. It will be run the first time you run ```make```.

{% highlight cmake %}
set(LARGE_DATA_FILE Und_min2.5x2.5_egm2008_WGS84_TideFree_reformatted)

add_custom_command(OUTPUT data/${LARGE_DATA_FILE}
  COMMAND cat split/${LARGE_DATA_FILE}_* > ${LARGE_DATA_FILE}
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data
  )

add_custom_target(AssembleData ALL SOURCES data/${LARGE_DATA_FILE})
{% endhighlight %}

My fork of GEOTRANS, cleverly named GEOCON, can be found [here](https://github.com/chachi/GEOCON).

[github_limits]: https://help.github.com/articles/what-is-my-disk-quota
[geotrans]: http://earth-info.nga.mil/GandG/geotrans/
[large_files]: http://blog.dinaburg.org/2013/07/git-fails-on-large-files.html
