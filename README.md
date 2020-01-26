# Cross compiler environment for producing Solaris SPARC / PowerPC executables on an i386 host

In order to revive the Solaris 2.5.1/PPC distribution it's necessary to get a compiler going on the system. Unfortunately I have no access to a system running on PPC but I tried this method to create cross compilers for both SPARC and PPC. The produced binaries have been successfully tested on Solaris 2.5.1 SPARC - should hopefully work on PPC as well.

## Prerequisites
Solaris 9 running on i386.
Install GCC, gmake and binutils from software companion CD.
Set shell path to /opt/sfw/bin /usr/ccs/bin /usr/bin /usr/sbin /sbin /usr/ucb .

## binutils
Download and unpack: https://ftp.funet.fi/pub/gnu/gnu/binutils/binutils-2.8.1.tar.gz

Compile and install:
```
./configure --prefix=/opt/sparc --build=i386-sun-solaris2 --host=i386-sun-solaris2 --target=sparc-sun-solaris2
gmake -j4
gmake install
gmake distclean
./configure --prefix=/opt/ppc --build=i386-sun-solaris2 --host=i386-sun-solaris2 --target=powerpcle-sun-solaris2
gmake -j4
gmake install
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
./configure --prefix=/opt/sparc --build=i386-sun-solaris2 --host=i386-sun-solaris2  --target=sparc-sun-solaris2
gmake LANGUAGES=c -j4
gmake LANGUAGES=c install
gmake distclean
./configure --prefix=/opt/ppc --build=i386-sun-solaris2 --host=i386-sun-solaris2 --target=powerpcle-sun-solaris2
gmake LANGUAGES=c -j4
gmake LANGUAGES=c install
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
