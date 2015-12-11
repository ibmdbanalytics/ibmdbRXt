# ibmdbRXt

This package contains extensions to the [ibmdbR](https://cran.r-project.org/web/packages/ibmdbR/index.html)
 package used to connect R clients with IBM dashDB and DB2. Especially, it provides functionality for push down of spatial operations into the database. 

To install, you first need to install the packages *ibmdbR* and *sp* from CRAN (if they are not installed yet), e.g. by

    install.packages('ibmdbR')
    install.packages('sp')



If these two packages and their dependencies installed correctly, you can install ibmdbRXt

    install.packages('https://github.com/ibmdbanalytics/ibmdbRXt/blob/master/ibmdbRXt_1.47.1.tar.gz?raw=T',repos=NULL)