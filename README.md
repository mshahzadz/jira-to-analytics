

# README #
 
## OVERVIEW ##
The purpose of this software is to extract flow data from Jira and put that data into a proper format for use with the ActionableAgile&trade; Analytics tool (for more information on the ActionableAgile&trade; Analytics tool, please visit [https://www.actionableagile.com](https://www.actionableagile.com) or sign up for a free trial of the tool at [https://www.actionableagile.com/cms/analytics-free-trial-signup.html](https://www.actionableagile.com/cms/analytics-free-trial-signup.html).)  

This program reads in a well-formed config file (see The Config File section below), connects to Jira and extracts data using the Jira REST API according to the parameters specified in the config file (see the Extraction Procedure Section below), and writes out a CSV file or JSON file that can be directly loaded into ActionableAgile&trade; Analytics (see The Output File section below).  

This software is written in Go and has been complied into executables for use with these Operating Systems:  Windows, Linux, Mac OS.

##RUNNING THE EXECUTABLE
The executable supports the following command-line flags, all of which are optional:

- -i specifies input config file name (defaults to config.yaml)
- -o specifies output file name (must end with .csv or .json, defaults to data.csv)
- -v displays the version number
- -q displays the query used
- -u displays unused Jira stages
- -h displays help (this list)

For example, to run the Linux version using a config file named myconfig.yaml and creating mydata.csv:
- jira_to_analytics.linux -i myconfig.yaml -o mydata.csv

To run the Windows version using config.yaml (the default) and writing jira.json:
- jira_to_analytics.win64.exe -o jira.json

To run the Mac version using all the defaults:
- jira_to_analytics.mac64
  
##THE CONFIG FILE##
In order for this utility to run properly, you must create a config file that contains the parameters of your Jira instance, and the necessary details of your workflow.  The config file is what tells the executuable what Jira instance to connect to, what data to grab, and how to format the resultant file to be uploaded into the ActionableAgile Analytics tool.

The config file we use conforms to the YAML format standard (http://yaml.org/spec/) and is completely case sensitive.  You can find an example config file here: [https://github.com/ActionableAgile/jira-to-analytics/blob/master/config.yaml](https://github.com/ActionableAgile/jira-to-analytics/blob/master/config.yaml).  Feel free to follow along with that example as we run through the details of each section of the file.

The file itself is broken up into the four sections:  

Connection  
Criteria  
Workflow  
Attributes  

### The Connection Section ###
The Connection Section of the config file is simply named "Connection" (without the quotes).  Each line of the this section contains the name of a connection property followed by a colon (:) followed by the required value.  This section has two required fields:

- 	Domain: the url to the domain where your Jira instance is hosted
- 	Username: the username you use to login to Jira

And one optional fields:

- 	Password: the password you use to login to Jira

If you do not supply a password in the config file, you will be prompted for a password at runtime.

An example of what this section might look like is:

Connection:  
&nbsp;&nbsp;&nbsp;&nbsp;Domain: https://www.myjiradomain.com  
&nbsp;&nbsp;&nbsp;&nbsp;Username: MyUsername  
&nbsp;&nbsp;&nbsp;&nbsp;Password: MyPassword  

**NOTE**:  We only support Basic Authentication with Jira at this time

### The Criteria Section ###
The Criteria Section of the config file is simply named "Criteria" (without the quotes) and contains optional Jira attributes that can use to control your data set. Each line in this section contains the name of the Jira attribute you want in your data followed by a colon (:) followed by its corresponding value in your Jira instance.  The fields in this section that we support are:

- 	Projects: a comma-separated list of the names of the Jira Projects you are querying
- 	Types: a comma-separated list of the names of the Jira issuetypes you want to extract
- 	Filters: a comma-separated list of the names of the filters you want to apply

An example of what this section might look like would be:

Criteria:  
&nbsp;&nbsp;&nbsp;&nbsp;Project: My Project1, My Project2  
&nbsp;&nbsp;&nbsp;&nbsp;Filters: Filter1, Filter2  
&nbsp;&nbsp;&nbsp;&nbsp;Types: Epic, User Story

**NOTE**:  The fields in this section are optional

### The Workflow Section ###
The Workflow Section of the config file is simply named "Workflow" (without the quotes) and contains all the information needed to configure your workflow data.  Each line of the this section contains the name of the workflow column as you want it to appear in the data file, followed by a colon (:) followed by a comma-separated list of all the Jira statuses that you want to map to that column.  For example, a row in your Workflow section that looks like:

Dev: Development Active, Development Done, Staging

will tell the software to look for the Jira statuses "Development Active", "Development Done", and "Staging" and map those statuses to a column in the output data file called "Dev".  The simplest form of this section is to have a one-to-one mapping of all Jira statuses to a corresponding column name in the data file.  For example, assume your Jira workflow is ToDo, Doing, Done.  Then a simple Workflow section of the config file that produces a data file that captures each of those statuses in a respectively named column would be:

Workflow:  
&nbsp;&nbsp;&nbsp;&nbsp;ToDo: ToDo  
&nbsp;&nbsp;&nbsp;&nbsp;Doing: Doing  
&nbsp;&nbsp;&nbsp;&nbsp;Done: Done

Sometimes Jira issues are created with a certain status, and there is no event that corresponds to a move into that status, so there is no date at which the work item entered the corresponding workflow stage. You can designate that an item be created in a certain workflow stage by adding (Created) to the list of Jira statuses. For example, in the previous example if you wanted to designate that items enter the ToDo workflow stage when they are created, you would change the workflow section of the config file as follows:

Workflow:  
&nbsp;&nbsp;&nbsp;&nbsp;ToDo: ToDo, (Created)  
&nbsp;&nbsp;&nbsp;&nbsp;Doing: Doing  
&nbsp;&nbsp;&nbsp;&nbsp;Done: Done

Again, please refer to the sample config file for an example of what the workflow section looks like. 

**NOTE**:  The Workflow section requires a minimum of TWO (2) workflow steps, and the names of the workflow steps must be specified in the order you want them to appear in the output CSV.  The expectation is that all Jira issue types that are requested will follow the exact same workflow steps in the exact same order.


### The Attributes Section ###
The Attributes Section of the config file is simply named "Attributes" (without the quotes) and is another optional section that includes name-value pairs that you want included in your extracted data set. They may be Jira custom fields that are unique to your Jira instance, or certain standard Jira fields that we support. Each line in this section contains the name you want to appear as the attribute column name in the CSV file, followed by a colon, followed by the name of a Jira custom field or a supported standard field, like this:

- 	CSV Column Name: ID of the custom field with optional dot notation (see following paragraph)
- 	CSV Column Name: Supported field name

In the returned JSON, a custom field may contain a single string, a struct, an array of strings, or an array of structs. If a struct is present (whether singly or in an array), we use the "value" field by default. If you want to use a different field, say, "color", you can specify it with the dot notation: customfield_10010.color

If the returned JSON contains an array, the content of each element is extracted normally. If there are multiple non-empty values, all the values are joined with an escaped comma and surrounded by square brackets like this: [red\,green\,blue]

Here are the standard Jira fields that you can use:  
-  status
-  issuetype
-  priority
-  resolution
-  project
-  labels
-  fixVersions
-  components

An example of what this section might look like is:

Attributes:  
&nbsp;&nbsp;&nbsp;&nbsp;Team: customfield_10000  
&nbsp;&nbsp;&nbsp;&nbsp;Regulatory Requirement: customfield_10001  
&nbsp;&nbsp;&nbsp;&nbsp;Status: status  
&nbsp;&nbsp;&nbsp;&nbsp;Type: issuetype  
&nbsp;&nbsp;&nbsp;&nbsp;Level: priority  

These fields will show up as filter attributes in the generated data file (please visit [https://www.actionableagile.com/format-data-file/](https://www.actionableagile.com/format-data-file/) for more information).

**NOTE**:  None of the fields in this section is required--in fact, this section as a whole is optional.

## EXTRACTION PROCEDURE ##
The program will read in the properly formatted config file (see The Config File section above) and attempt to connect to Jira using the url and authentication parameters specified (or will prompt you for a password if you did not specify one).  When a connection is established, the software will extract data using Jira’s REST API according to the parameters specified in the config file.  REST calls are “batched” so as to stay under Jira’s “maxResult” size limit as well as to minimize the chances of server timeout errors when retrieving large datasets.  If a non-fatal error is encountered, the extraction program will retry up to five time before terminating.  The program ignores any Jira issue types that have workflow stages not specified in the config and it handles Jira issues that have moved backward and forward through the workflow.
If all goes well, the extraction program will write out a CSV or JSON file that contains all extracted data to the same directory where the program is running.

## THE OUTPUT FILE ##
The output file follows the format required by the ActionableAgile Analytics tool as specified here:  [https://www.actionableagile.com/format-data-file/](https://www.actionableagile.com/format-data-file/). 

If the output file is a CSV file, it can be loaded directly into the ActionableAgile Analytics tool from the Home tab using the Load Data button.

If the output file is a JSON file, it can be loaded with Analytics via the URL parameter url=your-url/filename.json. For example, if you are hosting Analytics from www.mysite.com/Analytics and the JSON file is named data.json in the same directory as Analytics, the full url would be www.mysite.com/Analytics?url=data.json or www.mysite.com/Analytics?url=www.mysite.com/data.json. If you are using the SaaS version of Analytics from https://www.actionableagile.com and want to serve the JSON file from your own website, you must enable Cross-Origin Resource Sharing (CORS) for the URLs that serve your JSON.

## INFO FOR WINDOWS USERS ##
Download jira_to_analytics.win64.exe and config.yaml from the releases page on github ([https://github.com/actionableagile/jira-to-analytics/releases](https://github.com/actionableagile/jira-to-analytics/releases)) and put both files in the same directory.  Which directory you choose doesn’t matter as long as they are co-located.  Edit the config file and customize it for your specific Jira instance according to the instructions in this README.  You can either launch the exe by double clicking it in an explorer view or open a command prompt and run it from there.  If running from a command prompt simply type the name of the exe in and hit enter (no additional command line parameters are needed).  If the program succeeds, the output data file will be written in the same directory as the exe.

## INFO FOR LINUX USERS ##
Download jira_to_analytics.linux64 and config.yaml from the releases page on github ([https://github.com/actionableagile/jira-to-analytics/releases](https://github.com/actionableagile/jira-to-analytics/releases)) and put both files in the same directory.  Which directory you choose doesn’t matter as long as they are co-located. Edit the config file and customize it for your specific Jira instance according to the instructions in this README.  Open a terminal and cd to the directory containing the files. Make the linux64 file executable by typing chmod u+x jira_to_analytics.linux64. Run it by typing ./jira_to_analytics.linux64. If the program succeeds, the output data file will be written in the same directory as the executable.

## INFO FOR MAC USERS ##
Download jira_to_analytics.mac64 and config.yaml from the releases page on github ([https://github.com/actionableagile/jira-to-analytics/releases](https://github.com/actionableagile/jira-to-analytics/releases)) and put both files in the same directory.  Which directory you choose doesn’t matter as long as they are co-located. Edit the config file and customize it for your specific Jira instance according to the instructions in this README.  Open a terminal and cd to the directory containing the files. Make the mac64 file executable by typing chmod u+x jira_to_analytics.mac64. Run it by typing ./jira_to_analytics.mac64. If the program succeeds, the output data file will be written in the same directory as the executable.
