Do::

  sudo apt-get install gcc g++ git libc6-dev libffi-dev libssl-dev python-dev virtualenv

or::

  yum install gcc gcc-c++ git libffi-devel openssl-devel python-devel python-virtualenv

Create a virtualenv and clone and install::

  virtualenv elasticluster
  . elasticluster/bin/activate
  cd elasticluster
  git clone git://github.com/gc3-uzh-ch/elasticluster.git src
  cd src
  pip install -e .

Then::

  elasticluser start slurm 

  elasticluster list

  elasticluster stop slurm
