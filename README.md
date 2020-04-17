# Artifact Evaluation


## 1. Preparation

Please download the (https://drive.google.com/file/d/1XOYXPMY1jRgmn4eosunvLqi2YMcuQGiK/view?usp=sharing "data package") (about 400 MB) and uncompress the data (about 3 GB after uncompression).

``` sh
$ tar -zxvf AE.tar.gz
```

Please ensure your environment satisfying the following requirements. In particular, to apply ProFL on subjects in Defects4J, it is preferrable to compile the project with JDK 1.7. 

``` sh
JDK 1.7, Python 3+, numpy, Maven 3+
```




## 2. Experiment Reproduction
In the directory `scripts/`, each bash script (i.e., `Table-*.sh`) runs experiments and logs results which are relevant to the tables in the paper. 
For example, executing the script `Table-5.sh` can collecting and reporting the results in Table-5; the generated results are in the file `Table-5.csv` in the directory `scripts/final-results/`.

```sh
$ cd AE/scripts
$ ./Table-5.sh
```
In detail, `Table-5.sh`, for each subject in Defect4J-1.2.0, it first runs spectrum, Muse, Metallaxis, MCBFL and ProFL techniques via executing `MBFL.py`;
secondly, it merges the results of each subject via executing `full-log.py`; lastly, it executes `table5.py` to print the results in the form of table and log them in the `scripts/final-results/` directory.

Similarly, all the data in the paper can be reproduced by executing relevant `Table-*.sh` script code.


## 3. Tool
ProFL has been implemented as a fully automated Maven plugin, 
which is integrated on the latest byte code program repair tool PraPR. 
To apply ProFL on any Java project,  one only needs to 
(1) install ProFL to the local maven repository; 
(2) configure ProFL in the pom.xml of the target subject; 
(3) execute ProFL via maven command.





### 3.1 Install ProFL
To install the ProFL into the local maven repository. 

```sh
$ cd AE/tool/proFL
$ mvn clean install -Dhttps.protocols=TLSv1.2
```

If you encounter compilation error (e.g., saying signature of a certain method does not match the supplied arguments), probably your local repository contains some of old JAR files downloaded from Maven Central Repo. Please remove them via `rm -rf ~/.m2/repository/org/pitest` before installation.


### 3.2 Configure ProFL
Insert the following code into the `pom.xml` of the target project (already inserted for the three example projects).


```xml
<plugin>
	<groupId>org.mudebug</groupId>
	<artifactId>prapr-plugin</artifactId>
	<version>2.0.3-SNAPSHOT</version>
</plugin>
```


### 3.3 Execute ProFL
Three examples(i.e., Lang-26, Chart-20, and Time-19) are provided.
Passing the example name to the python script `run-example.py` can execute ProFL for the example.

For example, to execute ProFL on Lang-26 with following instructions.

```sh
$ cd AE/tool/examples
$ python3 run-example.py Lang-26
```

It would first execute maven compile and test tasks on the project
and then execute ProFL as a maven plugin.

### 3.4 ProFL Reports
The final reports of ProFL are in the directory of the `target/prapr-reports/*YYMMDD*/`.
`profl.log` is the fault localization rank list generated by ProFL;
`sbfl.log` is the fault localization rank list generated by spectrum-based fault localization;

We summarize the expected execution information of three examples in the following.

Example Name | Bug Method | Ranked by Spectrum | Ranked by ProFL |  Execution Time|
:-: | :-: | :-: | :-: | :-:
Lang-17 | format | 17 | 1 | < 2 min|
Chart-20|ValueMarker.init | 5 | 1 | < 2 min|
Time-19| getOffsetFromLocal | 77| 1 | < 10 min|


## 4 Data Structure 
All the input or intermediate data are in `data/` directory, 
and the generated log data are  in `script` directory.

### 4.1 Input Data
* data/
	* mutatorResults-0/: APR and Mutation results (full matrix) for each subject
	* 0-partial-mutatorResults-0/: APR results (partial matrix with order 1) for each subject
	* 0-partial-mutatorResults-0/: APR results (partial matrix with order 2) for each subject
	* 0-partial-mutatorResults-0/: APR results (partial matrix with order 3) for each subject
	* FailingTests/: Failing tests for each subject
	* MethodInfo/: Method information for each subject
	* spectrum_ag/: pre-calculated SBFL (due to the large size of coverage data)
	* pk/: page-rank FL reused from previous work
	* TimeLog/: execution cost log for the first version of each subject
	
	
### 4.2 Generated Data
* script/
	* full-results/: all FL results for each version of each subject
		* o-new-1-1-max.csv: Muse_PIT
		* o-meta-4-max.csv: Metallaxis_PIT
		* o-hy-4-max.csv: MCBFL_PIT
		* o-new-1-1-max.csv: ProFL_PIT
		* a-new-1-1-max.csv: Muse_Prapr
		* a-meta-4-max.csv: Metallaxis_Prapr
		* a-hy-4-max.csv: MCBFL_Prapr
		* a-new-1-1-max.csv: ProFL_Prapr
		* a-pk-1-1-max.csv: ProFL_PRFL
		* a-pka-1-1-max.csv: ProFL_PRFLMA
	* logs/: summary of all subjects with whole matrix
	* *-part-results/: partial matrix results with different partial orders
	* *-part-logs/: summary of all subjects with partial matrix
	* final-results: final results in the form of table
	

		

## 5 Code Structure 
All code related to fault localization is in the directory `script/muflscripts-AE/` and 
the code for statistics is in the directory `script/statisticcode/`

* script/muflscripts-AE/
	* FLcode/new/: entry of various FL techniques
		* MBFL.py: entry of full matrix FL
		* part-MBFL.py: entry of partial matrix FL
	* FLcode/refine/base_f2p_advance.py: core code of suspicious score calculation
	* FLcode/preProcess/: data preprocessing
	

