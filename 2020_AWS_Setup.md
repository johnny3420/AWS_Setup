# Notes and procedure on creating instance
**Need to work in N. Virginia**

# AWS_Setup

## Base Instance Creation
Contents of Base Instance
  1. Free-tier Ubuntu 18.04 LTS AMI
  2. 2 Core, 4 Gb RAM
  3. 30Gb EBS storage
  4. Security, All TCP
  5. Elastic IP (Since been released)

```
sudo apt-get update
sudo apt-get upgrade
sudo nano /etc/ssh/sshd_config
##Change PasswordAuthentication to yes from no, then save and exit.
sudo /etc/init.d/ssh restart
sudo passwd ubuntu #"Genomics"
sudo apt install xfce4 xfce4-goodies libssl-dev
cd /usr/local/src
sudo wget https://bintray.com/tigervnc/stable/download_file?file_path=tigervnc-1.10.1.x86_64.tar.gz
sudo tar -xzvf download_file\?file_path\=tigervnc-1.10.1.x86_64.tar.gz
sudo rm download_file\?file_path\=tigervnc-1.10.1.x86_64.tar.gz
cd /usr/local/bin
sudo cp -s ../src/tigervnc-1.10.1.x86_64/usr/bin/* .
vncserver #Genomics
vncserver -kill :1
mv ~/.vnc/xstartup ~/.vnc/xstartup.bak
nano ~/.vnc/xstartup
```
```
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
```

Exited ~/.vnc/xstartup and now in terminal

```
sudo chmod +x ~/.vnc/xstartup
vncserver
sudo nano ~/.config/xfce4/xfconf/xfce-perchannel-xml/xfce4-keyboard-shortcuts.xml
```

change 

```
<property name="&lt;Super&gt;Tab" type="string" value="switch_window_key"/>
```

to

```
<property name="&lt;Super&gt;Tab" type="empty"/>

```

Exit to terminal

```
sudo nano /etc/systemd/system/vncserver@.service
```

Add

```
[Unit]
Description=Start Tiger vncserver at startup
After=syslog.target network.target

[Service]
Type=forking
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu

PIDFile=/home/ubuntu/.vnc/%H:%i.pid
ExecStartPre=-/usr/local/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/local/bin/vncserver :%i
ExecStop=/usr/local/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target
```

Exit /etc/systemd/system/vncserver@.service and back in terminal
Edit config file to include encryption and other settings

```
cd ~/.vnc
myIP=$(hostname --ip-address)
openssl req -x509 -newkey rsa -days 365 -nodes -config /usr/lib/ssl/openssl.cnf -keyout vnc-server-private.pem -out vnc-server.pem -subj '/CN=${myIP}' -addext 'subjectAltName=IP:${myIP}'
nano config
```
Add
```
SecurityTypes=tlsvnc,X509Vnc
X509Cert=/home/ubuntu/.vnc/vnc-server.pem
X509Key=/home/ubuntu/.vnc/vnc-server-private.pem
depth=24
geometry=1280x800
```


```
sudo systemctl daemon-reload
vncserver -kill :1
sudo systemctl enable vncserver@1.service
sudo systemctl start vncserver@1
sudo systemctl status vncserver@1 # to check
```
Copy `vnc-server.pem` to own computer

```
sudo shutdown -r now
```

### Installing Necessary Libraries and Programs

```
sudo -i
apt-get install chromium-browser
apt-get install curl 
apt-get install libcurl4-openssl-dev #needed for bioconductor
apt-get install libboost-iostreams-dev
apt-get install libgsl0-dev
apt-get install libmysql++-dev
apt-get install libboost-graph-dev
apt-get install libgl1-mesa-dev #for rgl
apt-get install libglu1-mesa-dev #for rgl
apt-get install libmysqlclient-dev #for R mysql
apt-get install libfontconfig1-dev
apt-get install libmpfr-dev
apt-get install libgmp-dev
apt-get install openmpi-bin 
apt-get install libopenmpi-dev
apt-get install ncbi-blast+ 
apt-get install ncbi-blast+-legacy
apt-get install mysql-client 
apt-get install mysql-server #password for root = Genomics
apt-get install libssl-dev
apt-get install libudunits2-dev
apt-get install libxml2-dev
apt-get install cargo
apt-get install libcairo2-dev
apt-get install libmagick++-dev 
apt-get install libxslt1-dev
apt-get install libgeos-dev
apt-get install libgdal-dev
apt-get install libpq-dev
apt-get install texlive-latex-extra 
apt-get install texlive-fonts-recommended
apt-get install dkms
apt-get install liblist-moreutils-perl 
apt-get install libstatistics-descriptive-perl
apt-get install libstatistics-r-perl 
apt-get install perl-doc 
apt-get install libtext-table-perl
apt-get install gdebi-core
apt-get install cmake
sudo apt-get install cython
apt-get install gedit gir1.2-gtksource-3.0
apt-get install emboss
apt install default-jdk
apt install openjdk-8-jdk
apt-get install ruby-dev nodejs
apt-get install python-pip python-numpy python-scipy
apt-get install python3-pip python3-numpy python3-scipy
pip install --upgrade pip
pip3 install --upgrade pip
exit
```

### Installing latest R
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/'
sudo apt-get update
sudo apt-get install r-base
sudo apt-get install r-base-dev
```

### Installing Rstudio and make launcher
```
wget https://download1.rstudio.org/desktop/bionic/amd64/rstudio-1.2.5033-amd64.deb
sudo gdebi rstudio-1.2.5033-amd64.deb
rm rstudio-1.2.5033-amd64.deb
```

### Installing R packages within Rstudio under /home/ubuntu/R/x86_64-pc-linux-gnu-library/3.6 [Default]
```
R
install.packages(c('swirl','ggplot2','genetics','hwde','seqinr','qtl','evaluate','formatR','highr','markdown','yaml','htmltools','caTools','bitops','knitr','rmarkdown','devtools','shiny','pvclust','gplots','cluster','igraph','scatterplot3d','ape','SNPassoc','rsconnect','dplyr','tidyverse','learnr'), dependencies=T)
devtools::install_github(repo = "cran/PSMix")
# Install Biocondutor packages    
install.packages("BiocManager")
BiocManager::install(c("Rsubread","snpStats","rtracklayer","goseq","impute","multtest","VariantAnnotation","chopsticks","edgeR"))
install.packages('LDheatmap')
```

### Installing git viewer

```
sudo apt-get install gitg
```

### Install git-it desktop

```
cd /usr/local/src
sudo wget https://github.com/jlord/git-it-electron/releases/download/4.4.0/Git-it-Linux-x64.zip
sudo unzip Git-it-Linux-x64.zip
sudo rm Git-it-Linux-x64.zip
cd
```
### Installing Jalview

```
sudo apt-get install jalview
```

### Installing Others

```
sudo apt-get install seaview
sudo apt-get install clustalw clustalx kalign t-coffee muscle mafft probcons
```

### Installing BWA

```
cd /usr/local/src
sudo git clone https://github.com/lh3/bwa.git
cd bwa; sudo make
cd /usr/local/bin
sudo ln -s /usr/local/src/bwa/bwa .
```

### Installing Bowtie

```
cd /usr/local/src
sudo wget https://sourceforge.net/projects/bowtie-bio/files/bowtie/1.2.3/bowtie-1.2.3-linux-x86_64.zip/download
sudo unzip download
sudo rm download
cd /usr/local/bin
sudo ln -s /usr/local/src/bowtie-1.2.3-linux-x86_64/bowtie .
sudo ln -s /usr/local/src/bowtie-1.2.3-linux-x86_64/bowtie-build .
sudo ln -s /usr/local/src/bowtie-1.2.3-linux-x86_64/bowtie-inspect .
```

### Installing Bowtie2

```
cd /usr/local/src
sudo wget https://sourceforge.net/projects/bowtie-bio/files/bowtie2/2.4.1/bowtie2-2.4.1-linux-x86_64.zip
sudo unzip bowtie2-2.4.1-linux-x86_64.zip
sudo rm bowtie2-2.4.1-linux-x86_64.zip
cd /usr/local/bin
sudo ln -s /usr/local/src/bowtie2-2.4.1-linux-x86_64/bowtie2 .
sudo ln -s /usr/local/src/bowtie2-2.4.1-linux-x86_64/bowtie2-build .
sudo ln -s /usr/local/src/bowtie2-2.4.1-linux-x86_64/bowtie2-inspect .
```

### Installing Tophat

```
cd /usr/local/src
sudo wget https://ccb.jhu.edu/software/tophat/downloads/tophat-2.1.1.Linux_x86_64.tar.gz
sudo tar xvfz tophat-2.1.1.Linux_x86_64.tar.gz
sudo rm tophat-2.1.1.Linux_x86_64.tar.gz
cd /usr/local/bin
sudo ln -s /usr/local/src/tophat-2.1.1.Linux_x86_64/tophat .
```

### Installing Samtools

```
cd /usr/local/src
sudo wget https://github.com/samtools/samtools/releases/download/1.10/samtools-1.10.tar.bz2
sudo tar xvfj samtools-1.10.tar.bz2
sudo rm samtools-1.10.tar.bz2
cd samtools-1.10
sudo ./configure --prefix=/usr/local/src/samtools-1.10
sudo make
sudo make install
cd /usr/local/bin
sudo ln -s /usr/local/src/samtools-1.10/bin/samtools .
```

### Installing Bedtools2

```
cd /usr/local/src
sudo wget https://github.com/arq5x/bedtools2/releases/download/v2.29.2/bedtools-2.29.2.tar.gz
sudo tar -zxvf bedtools-2.29.2.tar.gz
sudo rm bedtools-2.29.2.tar.gz
cd bedtools2
sudo make
cd /usr/local/bin
sudo ln -s /usr/local/src/bedtools2/bin/* .
```

### Installing Fastqc

```
cd /usr/local/src
sudo wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip
sudo unzip fastqc_v0.11.9.zip
sudo rm fastqc_v0.11.9.zip
cd FastQC/
sudo chmod 0755 fastqc
cd /usr/local/bin
sudo cp -s ../src/FastQC/fastqc .
```

### Installing seqtk

```
cd /usr/local/src
sudo git clone https://github.com/lh3/seqtk.git
cd seqtk
sudo make
cd /usr/local/bin
sudo ln -s /usr/local/src/seqtk/seqtk .
```

### Installing Cufflinks

```
cd /usr/local/src
sudo wget http://cole-trapnell-lab.github.io/cufflinks/assets/downloads/cufflinks-2.2.1.Linux_x86_64.tar.gz
sudo tar -xvzf cufflinks-2.2.1.Linux_x86_64.tar.gz
sudo rm cufflinks-2.2.1.Linux_x86_64.tar.gz
cd /usr/local/bin
sudo ln -s /usr/local/src/cufflinks-2.2.1.Linux_x86_64/cuff* .
sudo ln -s /usr/local/src/cufflinks-2.2.1.Linux_x86_64/g* .
```

### Markdown viewer: Remarkable

```
cd /usr/local/src
sudo wget https://remarkableapp.github.io/files/remarkable_1.87_all.deb
sudo gdebi remarkable_1.87_all.deb
sudo rm remarkable_1.87_all.deb
```

### Installing MACS2

```
sudo pip3 install MACS2
```

### Installing Cytoscape

```
cd /usr/local/src
sudo wget https://github.com/cytoscape/cytoscape/releases/download/3.7.2/Cytoscape_3_7_2_unix.sh
sudo chmod 755 Cytoscape_3_7_2_unix.sh
sudo ./Cytoscape_3_7_1_unix.sh
```

### Installing FreeBayes

```
cd /usr/local/src
sudo wget https://github.com/ekg/freebayes/releases/download/v1.3.1/freebayes-v1.3.1
sudo chmod +x freebayes-v1.3.1
cd /usr/local/bin
sudo ln -s /usr/local/src/freebayes-v1.3.1 freebayes
```

### Installing GATK 4.1.0.0

```
cd /usr/local/src
sudo wget https://github.com/broadinstitute/gatk/releases/download/4.1.5.0/gatk-4.1.5.0.zip
sudo unzip gatk-4.1.5.0.zip
sudo rm gatk-4.1.5.0.zip
cd /usr/local/bin
sudo ln -s /usr/local/src/gatk-4.1.5.0/gatk .
```

### Installing Atom

```
sudo wget https://atom.io/download/deb
sudo mv deb atom.deb
sudo gdebi atom.deb
sudo rm atom.deb
```

Add atom packages:

* git-plud
* markdown-pdf
* line-ending-converter
* language-markdown
* Sublime-Style-Column-Selection

In atom, go to preferences and disable the 'whitespace' package

### fastStructure

```
cd /usr/local/src/
sudo git clone https://github.com/rajanil/fastStructure
cd /usr/local/src/fastStructure/
sudo git fetch
sudo git merge origin/master
sudo updatedb
sudo locate gsl_sf_psi.h
sudo locate libgslcblas.so
sudo locate libgsl.so
set -x LDFLAGS "-L/usr/lib/x86_64-linux-gnu"
set -x CFLAGS "-I/usr/include"
set -x LD_LIBRARY_PATH /usr/lib/x86_64-linux-gnu
cd vars
sudo python setup.py build_ext --inplace
cd ../
sudo python setup.py build_ext --inplace
cd /usr/local/bin
sudo nano fastStructure
```
add in following

```
#!/bin/bash
python /usr/local/src/fastStructure/structure.py
```
make executable

```
sudo chmod 755 fastStructure
```

### igv
go to [igv downloads](http://software.broadinstitute.org/software/igv/download) and download the binary version (currently 2.8.0)

```
cd /usr/local/src
sudo wget https://data.broadinstitute.org/igv/projects/downloads/2.8/IGV_Linux_2.8.0.zip
sudo unzip IGV_Linux_2.8.0.zip 
sudo rm IGV_Linux_2.8.0.zip 
sudo ln -s /usr/local/src/IGV_Linux_2.8.0/igv.sh /usr/local/bin
```

### STAR

Download from https://github.com/alexdobin/STAR

```
cd /usr/local/src
sudo wget https://github.com/alexdobin/STAR/archive/2.7.3a.tar.gz
sudo tar -xzf 2.7.3a.tar.gz
sudo rm 2.7.3a.tar.gz
cd /usr/local/bin
sudo ln -s /usr/local/src/STAR-2.7.3a/bin/Linux_x86_64/STAR .
```

### Add class data to image

```
cd
wget http://malooflab.phytonetworks.org/media/maloof-lab/filer_public/8f/d5/8fd59de6-e311-4d50-8320-acc58402982f/bis180l_class_data_2020tar.gz
tar xzcf bis180l_class_data_2020tar.gz
rm bis180l_class_data_2020tar.gz
```

## Added script to change student's VNC connection to secure

## Installing MAFFT
```
wget https://mafft.cbrc.jp/alignment/software/mafft_7.450-1_amd64.deb
sudo dpkg -i mafft_7.450-1_amd64.deb
rm mafft_7.450-1_amd64.deb
```

## Installing FastTree
```
cd /usr/local/bin
sudo wget http://microbesonline.org/fasttree/FastTree
sudo chmod +x FastTree
```

## Installing Dendroscope
```
cd /usr/local/bin
sudo wget https://software-ab.informatik.uni-tuebingen.de/download/dendroscope/Dendroscope_unix_3_7_2.sh
sudo chmod +x Dendroscope_unix_3_7_2.sh
# run installer accept all defaults
./Dendroscope_unix_3_7_2.sh
# place icon on dock
```

## Install MEGA
```
# GUI includes CC
wget https://www.megasoftware.net/do_force_download/megax_10.1.7-1_amd64.deb
sudo dpkg -i megax_10.1.7-1_amd64.deb
rm megax_10.1.7-1_amd64.deb

```
## MISC
Added Rstudio, Atom, and IGV icons to the dock
Unchecked locale box on Atom::spell-check
Added COVID data to data folder

