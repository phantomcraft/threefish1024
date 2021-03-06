Threefish1024 v0.2
==================
++++++++++++++++++

* 06/02/2018
  First release.

Threefish1024 is an experimetal Threefish implementation for Linux 
filesystem encryption.

It is a modified version of original Threefish512 from 
https://github.com/bogdankernel/threefish_512

Functions threefish_encrypt_1024 and threefish_decrypt_1024 are based
on last version of skein hash staging driver: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/drivers/staging/skein/threefish_block.c?h=linux-4.17.y

It has 1024 bits of key size and an additional "tweak" operating mode. This mode doesn't need random counter or Initialization Vector, you can use plain IV (plain or plain64) safely as XTS mode uses.

Security:
=========

A cipher with NO CRYPTOGRAPHIC BREAK gives an attacker nothing more than
 brute-forcing an interactive password or the master key as options. 
 This is the case of Threefish.

Let's suppose that each atom in the known universe (believed to be 
something like 10^82) is converted into a supercomputer with the same 
power of OLCF-4 (which has 200 petaflops per second). Now, all those 
are driven to brute-force a random 1024 bit key; remembering that one 
year has 31556952 seconds.

Let's calculate: 2^1023/((10^82)*200000000000000000*31556952)

It would take approximately 1424165691653550624700149424118198054756664220091904450035526887686528437580893135153772751265771295189726449815807570370379795938581272222463432389156865691415380463747461140070674598114276801189901697 years to find the correct key!

This seems amazing, but never forget that no cipher is secure forever, 
some older ciphers with large key sizes are broken today, you'll never 
know if in the next week someone finds out a break that kills your 
favorite cipher once for all.

Usage:
======

Make sure you have Linux Kernel headers installed.

  Build:

    $ make

  Load:

    $ insmod ./threefish1024.ko

  Create an encrypted container in CBC, CTR or ECB mode, example:

    $ cryptsetup luksFormat /dev/loop0 -c threefish-ctr-plain64 -s 1024

    Key size must be 1024.

  Using with the tweak mode:

    $ cryptsetup luksFormat /dev/loop0 -c threefish-tweak-plain64 -s 1024

    With a detached header is always more secure.

    $ cryptsetup luksFormat /dev/loop0 --header ./myheader -c threefish-tweak-plain64 -s 1024

    Make a backup of header, open it with --header option.

Warnings:
=========

* Always use passwords with 20+ characters and a key derivation 
function (KDF) for them.

* Passwords between 20-35 characters are suitable for using with a 
1024 bits encryption if iterated with a KDF for some seconds and/or 
used with an external key file:

   Create a random key file:

      $ dd if=/dev/urandom of=./key.001 bs=1 count=128

   Create an encryped container with a password iterated for 10 secs and 
      this key file:

      $ cryptsetup luksFormat /dev/loop0 -c threefish-tweak-plain64 -s 1024 --key-file ./key001 --iter-time 10000

* This tool will not save your ass if you live in a country that has 
laws for key disclosure or prohibits the usage of encryption tools. You 
must use a detached header to achieve plausible deniability:

      $ cryptsetup luksFormat ./speed-test_disk-write.274 -c threefish-tweak-plain64 -s 1024 --header /dev/shm/header.001

   Buy an 1GB MicroSD card, move the header to the SD card filesystem, 
   wrap up with carbon paper and hide it. =)

Benchmarks:
===========

In CPUs without AES acceleration instructions, AES has a bad 
performance; if that is your case and you have a 64 bit linux system, 
Threefish is a good option.

Here the tests with Threefish with 1024 bits key + tweak and other 
common ciphers with different key sizes. AES and Camellia not included 
for using AES instructions.



           Algorithm  |  Key |  Encryption |  Decryption

     threefish-tweak    1024b   513.3 MiB/s  493.1 MiB/s
        serpent-cbc     128b    85.2 MiB/s   339.9 MiB/s
        serpent-cbc     192b    88.7 MiB/s   336.3 MiB/s
        serpent-cbc     256b    90.5 MiB/s   335.7 MiB/s
        serpent-xts     128b   326.5 MiB/s   328.3 MiB/s
        serpent-xts     192b   326.7 MiB/s   319.9 MiB/s
        serpent-xts     256b   326.5 MiB/s   326.0 MiB/s
        serpent-xts     384b   325.1 MiB/s   322.6 MiB/s
        serpent-xts     512b   327.3 MiB/s   322.6 MiB/s
        blowfish-cbc    128b   114.4 MiB/s   368.4 MiB/s
        blowfish-cbc    256b   118.0 MiB/s   360.5 MiB/s
        blowfish-cbc    384b   117.7 MiB/s   365.4 MiB/s
        blowfish-cbc    448b   118.8 MiB/s   375.2 MiB/s
        twofish-cbc     128b   162.1 MiB/s   325.2 MiB/s
        twofish-cbc     256b   170.4 MiB/s   341.4 MiB/s
        twofish-cbc     192b   172.8 MiB/s   344.2 MiB/s
        twofish-xts     256b   298.9 MiB/s   295.0 MiB/s
        twofish-xts     384b   295.2 MiB/s   298.2 MiB/s
        twofish-xts     512b   304.0 MiB/s   299.0 MiB/s
        cast5-cbc       128b   107.0 MiB/s   352.4 MiB/s
        cast6-cbc       128b   110.1 MiB/s   229.9 MiB/s
        cast6-cbc       192b   109.8 MiB/s   229.4 MiB/s
        cast6-cbc       256b   110.3 MiB/s   227.7 MiB/s
        cast6-xts       256b   207.9 MiB/s   211.2 MiB/s
        cast6-xts       384b   210.4 MiB/s   209.0 MiB/s
        cast6-xts       512b   209.1 MiB/s   209.7 MiB/s
        anubis-cbc      128b   139.8 MiB/s   154.7 MiB/s
        anubis-cbc      192b   127.6 MiB/s   139.8 MiB/s
        anubis-cbc      256b   116.5 MiB/s   126.5 MiB/s
        anubis-cbc      320b   103.7 MiB/s   116.4 MiB/s
        anubis-xts      256b   167.1 MiB/s   167.2 MiB/s
        anubis-xts      384b   149.6 MiB/s   150.6 MiB/s
        anubis-xts      512b   133.8 MiB/s   133.8 MiB/s
        anubis-xts      640b   124.1 MiB/s   124.8 MiB/s

Benchmarked on a AMD Ryzen 5 1400 CPU (no Overclock) with 2133Mhz DDR4 
memory.

Test yourself with:

  $ cryptsetup benchmark -c threefish-tweak-plain64 -s 1024

If you use a 32-bit linux system, encrypting in Threefish with 1024 bits
 key will be very slow, you should use Serpent or Twofish instead.
