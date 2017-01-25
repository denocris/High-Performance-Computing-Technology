How to build the image
------------------------
To build the image in the local docker repository, use the following command::

    docker build -t IMAGENAME IMAGEPATH


How to run the container
-------------------------
To run the container, use the following command::

    docker run --name NAMEOFCONTAINER -v /PATH/OF/HPL.dat:/hpl-2.1/bin/Linux_ATHLON_CBLAS/HPL.dat -v /PATH/OF/HPL.out:/hpl-2.1/bin/Linux_ATHLON_CBLAS/HPL.out IMAGENAME
