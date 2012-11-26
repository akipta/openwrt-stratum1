CC=mips-openwrt-linux-uclibc-gcc
CFLAGS=-O2

all: libntpcompat.a

libntpcompat.a: ntpcompat.o
	ar rcs libntpcompat.a ntpcompat.o
	
ntpcompat.o: ntpcompat.c

clean:
	rm -f libntpcompat.a ntpcompat.o
