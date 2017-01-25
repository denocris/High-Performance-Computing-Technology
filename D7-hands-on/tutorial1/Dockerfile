FROM ubuntu:16.04
MAINTAINER Stefano Piani <stefano.piani@exact-lab.it>

RUN apt-get update && apt-get install -y build-essential libatlas-base-dev libmpich-dev gfortran

ADD http://www.netlib.org/benchmark/hpl/hpl-2.1.tar.gz /

RUN tar -xvf hpl-2.1.tar.gz && rm hpl-2.1.tar.gz
ADD Make.Linux_ATHLON_CBLAS /hpl-2.1/
RUN cd /hpl-2.1 && make arch=Linux_ATHLON_CBLAS
RUN touch /hpl-2.1/bin/Linux_ATHLON_CBLAS/HPL.out
ADD HPL.dat /hpl-2.1/bin/Linux_ATHLON_CBLAS/HPL.dat

CMD i=`grep -c ^processor /proc/cpuinfo`; cd /hpl-2.1/bin/Linux_ATHLON_CBLAS; mpirun -np $i ./xhpl > HPL.out
