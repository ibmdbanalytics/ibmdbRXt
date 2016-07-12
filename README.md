# IBM In-Database Analytics for R - Extensions

This package contains extensions to the [ibmdbR](https://cran.r-project.org/web/packages/ibmdbR/index.html). In addition to the push-down functionality provided by *ibmdbR*, it allows for the push-down of geo-spatial functions into the database.

## Installation

To install ibmdbRXt, you first need to install the packages *ibmdbR* and *sp* from CRAN (if they are not installed yet), e.g. by

    install.packages('ibmdbR')
    install.packages('sp')
    
To automatically install all dependent R packages use:

	install.packages('ibmdbR', dependencies = TRUE)
	install.packages('sp', dependencies = TRUE)
	
Please watch for errors during the package installation (exit code != 0) and ensure that the all R package dependencies were installed correctly. 
You can find a short installation guide for the *sp* R package on dashDB Local / CentOS in the next section (although ibmdbRXt only uses a subset of functions of the *sp* package).

If these two packages and their dependencies were installed correctly, you can install *ibmdbRXt*

    install.packages('https://github.com/ibmdbanalytics/ibmdbRXt/blob/master/ibmdbRXt_1.47.1.tar.gz?raw=T',repos=NULL)

### Installation of the *sp* package dependencies on dashDB Local / CentOS:

The *sp* package requires the R packages *rgdal* and *rgeos*, which have dependencies to the *geos*, *proj* and *gdal* libraries.

A convenient method for installing the required libraries in CentOS 7 is using yum with a current EPEL repository:  
	
	su -c 'rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm'
	yum -y install gdal gdal-devel proj proj-devel proj-nad proj-epsg geos geos-devel

## Basic Steps

### Loading the sample data

In the samples directory, you will find two files with geometries, *customers.zip* and *floodzones.zip*. Load both datasets into tables in your database. If you use IBM dashDB, you can use the interactive load functionality for spatial files to do this. First download the zip files to a temporary directory on your computer. Then use the "Load -> Load Geospatial Data" functionality of dashDB to load the data into your database. You can specify a table name for each data set. In the following, we assume that the resulting tables are called "CUSTOMERS" and "FLOODZONES" respectively. When asked for a spatial reference system, click on "Add.." and simply give it any unique name. You can leave all other settings with their defaults. 

### First steps

First, use

    library(ibmdbRXt)
    idaConnect('BLUDB','','')
    idaInit(con)
    
to load the packages, connect to the database and initialize in-database analytics.

After you successfully established a database connection, you can examine the structure of the tables
that contain geospatial data. Each of these tables stores geospatial data (usually in specially designated  geospatial columns) alongside other associated information. To view a table that contains geospatial data, create an IDA data frame that points to such a table. Then, use *head* to display some rows of the table. 

    floodzones <- ida.data.frame('FLOODZONES')
    head(floodzones, 3)

### Analyze geometries and their relations using IDA data frames

One group of functions in the *ibmdbRXt* package analyzes geometries that are stored in one column of an IDA data frame. For example, *idaArea* calculates the areas of the geometries in the input IDA data frame.
Then it adds a column that contains the calculated areas to the input  IDA data frame. 
The following code example shows the syntax and the result of *idaArea*, thereby illustrating some typical characteristics of *ibmdbRXt* functions.

    floodzones$AREA <- idaArea(floodzones, unit='statute mile')
    head(floodzones,3)

The main group of functions in the *ibmdbRXt* package analyzes the relationship between two sets of geometries that are stored in two separate IDA data frame columns. These columns can either belong to a single IDA data frame, or they can belong to two different  IDA data frames. In the first case, such a function runs in *single mode*; in the latter case, such a function runs in *dual mode*. Whether such a function runs in *single* or *dual mode* greatly affects its syntax and its result. 

When run in *single mode*, these functions behave similarly to functions that analyze the geospatial data in a single column of an IDA data frame. They too add a column to the input IDA data frame that contains the result of the analysis. You also need to assign a name to the column that contains the results. The only difference is that you need to specify two column names as input columns.

In *dual mode*, these functions return an entire IDA data frame that contains the results of the
analysis. Moreover,  more parameters need to be specified. Therefore, the following code example
shows *idaWithin*, which calculates whether geometries are contained within other geometries, run in dual mode. In this example, *idaWithin* tests whether points are contained within polygons.
    
     floodzones <- ida.data.frame('FLOODZONES')
     customers <- ida.data.frame('CUSTOMERS')[,c('ID','NAME','GEO_DATA')]
     hit_customers <- idaWithin(customers, floodzones, by.x = customers$GEO_DATA, 
                             by.y = floodzones$GEO_DATA)


This code example generates a list of customers that live in flood zones. The first line of code
creates the IDA data frame *customers* that points to the table CUSTOMERS. However, only
certain columns in this table are needed for the analysis. Therefore, the code fragment *[,c('ID','NAME','GEO_DATA')]* specifies that only these columns are included in the IDA data frame.

Then, it specifies the data frames that this function analyzes, namely *customers* and
*floodzones*. The order of *customers* and *floodzones* determines that the function tests whether
the geometries in *customers* are contained within *floodzones*. 
Next, the *by.x* and *by.y* parameters specify the columns of the data frames that contain the analyzed
geometries. However, you only need to specify these columns if several columns of the input IDA data frames contain geospatial data. By default, the resulting IDA data frame contains only customers that are located in flood zones, and flood zones that affect customers. 

### Creating new goemetries

Another group of functions in the *ibmdbRXt* package creates new geometries. For example, *idaCentroid* creates points that represent the centroids of polygons. The following command calculates the centroids of the flood zones discussed above:

    floodzones$CENTROIDS <- idaCentroid(floodzones)

This command creates a column that contains the points that represent the centroids of the multipolygons of the input IDA data frame. This column is added to the input IDA data frame. 

### Convert geometries in IDA data frames

A fourth group of functions in the *ibmdbRXt* package converts IDA data frames with geospatial data
to *sp SpatialDataFrames*, which can be plotted. There are functions that convert points, lines, and mulitploygons. The following code example creates a plot that shows which customers live in flood zones: 

    sp_floodzones <- as.SpatialPolygonsDataFrame(floodzones, floodzones$GEO_DATA)
    plot(sp_floodzones, col='cyan')

### Further Reading

For more information about specific *ibmdbRXt* functions, see the reference documentation of the
function. To learn more about In-Database Analytics, see the reference documentation of the ibmdbR package.
See the [reference documentation of the spatial functions in dashDB](https://www-01.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.db2.luw.spatial.topics.doc/doc/csbp1001.html?lang=en) in the IBM Knowledge Center for more information about geospatial analysis in general.   
    