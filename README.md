# Cross compiler environment for producing Solaris SPARC / PowerPC executables on an i386 host

In order to revive the Solaris 2.5.1/PPC distribution it's necessary to get a compiler going on the system. The produced binaries have been successfully tested on Solaris 2.5.1 SPARC and PPC.

## Prerequisites
Solaris 9 running on i386.
Install GCC, gmake and binutils from software companion CD.
Set shell path to /opt/sfw/bin /usr/ccs/bin /usr/bin /usr/sbin /sbin /usr/ucb .

## binutils
Download and unpack: https://ftp.funet.fi/pub/gnu/gnu/binutils/binutils-2.8.1.tar.gz

Compile and install:
```
./configure --prefix=/opt/sparc-cross --build=i386-sun-solaris2 --host=i386-sun-solaris2 --target=sparc-sun-solaris2
gmake -j4
gmake install
gmake distclean
./configure --prefix=/opt/ppc-cross --build=i386-sun-solaris2 --host=i386-sun-solaris2 --target=powerpcle-sun-solaris2
gmake -j4
gmake install
gmake distclean
```

## SUNWarc + SUNWhea
Copy reloc.cpio.Z from SUNWarc and SUNWhea packages on the Solaris 2.5.1 SPARC/PPC distribution CDs to SUNWhea-sparc.cpio.Z, SUNWhea-ppc.cpio.Z etc.

Unpack and shuffle around the files a bit.

```
cd /opt/sparc/sparc-sun-solaris2
gzip -cd ~/2.5.1/SUNWhea-sparc.cpio.Z | cpio -id
gzip -cd ~/2.5.1/SUNWarc-sparc.cpio.Z | cpio -id
mv usr/lib/* lib/
rmdir usr/lib/
mv usr/* .
mv ccs/lib/* lib/
rmdir ccs/lib/
rmdir ccs/
rmdir usr

cd /opt/ppc/powerpcle-sun-solaris2
gzip -cd ~/2.5.1/SUNWhea-ppc.cpio.Z | cpio -id
gzip -cd ~/2.5.1/SUNWarc-ppc.cpio.Z | cpio -id
mv usr/lib/* lib/
rmdir usr/lib/
mv usr/* .
mv ccs/lib/* lib/
rmdir ccs/lib/
rmdir ccs/
rmdir usr
```

## GCC
Download and unpack: https://ftp.funet.fi/pub/gnu/gnu/gcc/gcc-2.8.1.tar.gz

Compile and install:
```
./configure --prefix=/opt/sparc-cross --build=i386-sun-solaris2 --host=i386-sun-solaris2  --target=sparc-sun-solaris2
gmake LANGUAGES=c -j4
gmake LANGUAGES=c install
gmake distclean
./configure --prefix=/opt/ppc-cross --build=i386-sun-solaris2 --host=i386-sun-solaris2 --target=powerpcle-sun-solaris2
gmake LANGUAGES=c -j4
gmake LANGUAGES=c install
gmake distclean
```

## Compile test code
```
cat > hello.c << EOF
main()
{
 printf("Hello world from a cross compiled executable!\n");
}
EOF

/opt/sparc/sparc-sun-solaris2/bin/gcc -o hello-sparc hello.c
/opt/ppc/powerpcle-sun-solaris2/bin/gcc -o hello-ppc hello.c 
file hello-sparc hello-ppc
```
hello-sparc:    ELF 32-bit MSB executable SPARC Version 1, statically linked, not stripped
hello-ppc:      ELF 32-bit LSB executable PowerPC Version 1, statically linked, not stripped

# Building a native compiler for Solaris SPARC / PowerPC executables on an i386 host

## Prerequisites
Cross compiler created as outlined above. Put the path to the cross compiler in your shell path.

## SUNWlibm
Copy reloc.cpio.Z from SUNWlibm packages on the Solaris 2.5.1 SPARC/PPC distribution CDs to SUNWlibm-sparc.cpio.Z, SUNWlibm-ppc.cpio.Z etc.

Unpack and shuffle around the files a bit.

```
cd /opt/sparc/sparc-sun-solaris2
gzip -cd ~/2.5.1/SUNWlibm-sparc.cpio.Z | cpio -id
mv usr/lib/libp/* lib/libp/
rmdir usr/lib/libp
mv usr/lib/* lib/
rmdir usr/lib
mv usr/include/sys/* include/sys/
rmdir usr/include/sys
mv usr/include/* include/
rmdir usr/include
rmdir usr

cd /opt/ppc/powerpcle-sun-solaris2
gzip -cd ~/2.5.1/SUNWlibm-ppc.cpio.Z | cpio -id
mv usr/lib/libp/* lib/libp/
rmdir usr/lib/libp
mv usr/lib/* lib/
rmdir usr/lib
mv usr/include/sys/* include/sys/
rmdir usr/include/sys
mv usr/include/* include/
rmdir usr/include
rmdir usr
```

## binutils
Compile and install:
```
./configure --prefix=/opt/sparc-gcc --build=i386-sun-solaris2 --host=sparc-sun-solaris2
gmake -j4
gmake install
gmake distclean
./configure --prefix=/opt/ppc-gcc --build=i386-sun-solaris2 --host=powerpcle-sun-solaris2
gmake -j4
gmake install
gmake distclean
```

## GCC
Switching to version 2.95 as 2.8.1 seems to have problems compiling.

Download and unpack: https://ftp.funet.fi/pub/gnu/gnu/gcc/gcc-2.95.tar.gz

Compile and install:
```
./configure --prefix=/opt/sparc-gcc --build=i386-sun-solaris2 --host=sparc-sun-solaris2
gmake LANGUAGES=c -i -j4
gmake LANGUAGES=c -i install
gmake distclean
./configure --prefix=/opt/ppc-gcc --build=i386-sun-solaris2 --host=powerpcle-sun-solaris2
gmake LANGUAGES=c -i -j4
gmake LANGUAGES=c -i install
gmake distclean
```

## Installing on the host computer
Copy the files to /opt/ from the build host
Install the packets SUNWarc, SUNWhea, SUNWlibm and SUNWtoo from the Solaris distribution CD.

Copy values*.o to the compiler director:
```
cp /usr/ccs/lib/values*.o /opt/sparc-gcc/sparc-sun-solaris2/lib/
```

Test the compiler:
Put /opt/sparc-gcc/bin or /opt/ppc-gcc/bin in your path.

```
file `which gcc`
```
/opt/sparc-gcc/bin/gcc: ELF 32-bit MSB executable SPARC Version 1, statically linked, not stripped
```
gcc -v
```
Reading specs from /opt/sparc-gcc/lib/gcc-lib/sparc-sun-solaris2/2.95/specs
gcc driver version 2.95 19990728 (release) executing gcc version 2.8.1
```
gcc -o hello hello.c
./hello
```
Hello, world!

I've successfully built gcc-2.8.1, gzip-1.2.4a and emacs-19.34 using the cross-compiled gcc on a SPARC machine. Should be possible on PPC as well!

Feel free to contact me via mail - x25 at x25.ee if you have any questions.
