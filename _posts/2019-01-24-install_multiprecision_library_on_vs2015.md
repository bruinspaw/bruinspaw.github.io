---
layout: post
title:  "Build MPIR and MPFR with Visual Studio 2015"
date:   2019-01-24 06:24:20 +0800
author: bruinspaw
categories: Compilation
---

# Step1: Download MPIR and MPFR from repositories:
* https://github.com/BrianGladman/mpir
* https://github.com/BrianGladman/mpfr

unzip as mpir and mpfr respectively(removing-suffix -master, MPFR compilation require the correct path).

# Step2: Build MPIR
* Open "mpir\msvc\vs15\dll_mpir_gc" project with Visual Studio 2015
* Build settings: Release, x86
* Build...
* Output files locate in: mpir\dll\Win32\Release, which is required for building MPFR

# Step3: Build MPFR
* Open "mpfr\build.vc15\dll_mpfr" project with notepad++, this project is for 2017, so
* Replace ToolsVersion="15.0" to ToolsVersion="14.0" and all "<PlatformToolset>v141</PlatformToolset>" to "<PlatformToolset>v140</PlatformToolset>"
* Open "mpfr\build.vc15\dll_mpfr" project with Visual Studio 2015
* Build settings: Release, x86
* Build...
* Output files locate in: mpfr\build.vc15\dll\Win32\Release

# Step4: Copy output files to desired directory.(say: C:\multiprecision)
* C:\multiprecision\include : header files
* C:\multiprecision\bin     : dll, should be added to system PATH
* C:\multiprecision\lib     : lib files

# Step5: MPIR example
factorial.cpp:
```
#include <iostream>
#include <mpirxx.h>

int main (int argc, char *argv[])
{
  mpz_class n;
  n =  1;
  for(int i = 2; i<= 100; i++) {
    n *= i;
	std::cout <<i << "!:" <<  n << std::endl;
  }
  return 0;  
}
```
namke Makefile:
```
factorial.exe : factorial.obj C:\multiprecision\lib\mpir.lib
	cl /Fe$@ $**

factorial.obj : factorial.cpp
	cl /EHsc -I"C:\multiprecision\include" -c factorial.cpp

clean:
	del -f *.obj factorial.exe
```
# Step6: MPFR example
series.cpp
```
#include <stdio.h>
#include <gmp.h>
#include <mpfr.h>

int main (void)
{
  unsigned int i;
  mpfr_t s, t, u;

  mpfr_init2 (t, 200);
  mpfr_set_d (t, 1.0, MPFR_RNDD);
  mpfr_init2 (s, 200);
  mpfr_set_d (s, 1.0, MPFR_RNDD);
  mpfr_init2 (u, 200);
  for (i = 1; i <= 100; i++) {
      mpfr_mul_ui (t, t, i, MPFR_RNDU);
      mpfr_set_d (u, 1.0, MPFR_RNDD);
      mpfr_div (u, u, t, MPFR_RNDD);
      mpfr_add (s, s, u, MPFR_RNDD);
    }
  printf ("Sum is ");
  mpfr_out_str (stdout, 10, 0, s, MPFR_RNDD);
  putchar ('\n');
  mpfr_clear (s);
  mpfr_clear (t);
  mpfr_clear (u);
  mpfr_free_cache ();
  return 0;
}
```
nmake Makefile:
```
series.exe : series.obj C:\multiprecision\lib\mpfr.lib
	cl /Fe$@ $**

series.obj : series.cpp
	cl /EHsc -I"C:\multiprecision\include" -c series.cpp

clean:
	del -f *.obj series.exe
```
